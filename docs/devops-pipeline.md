# DEVOPS & CI/CD PIPELINE - AURA 2.0
## Infrastructure as Code & Automation

## 🏗️ INFRASTRUCTURE OVERVIEW

```yaml
Cloud Strategy: Multi-Cloud
Primary: AWS (us-east-1)
Secondary: Azure (Brazil South)
DR Region: AWS (sa-east-1)

Environments:
  - Development (dev)
  - Staging (staging)
  - Production (prod)
  - DR (disaster-recovery)
```

## 📦 CONTAINERIZATION

### Backend Dockerfile
```dockerfile
# backend/Dockerfile
FROM node:20-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runtime
WORKDIR /app

# Security: Run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

COPY --from=dependencies --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=build --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs package*.json ./

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

USER nodejs
EXPOSE 3000

CMD ["node", "dist/main.js"]
```

### Frontend Dockerfile
```dockerfile
# frontend/Dockerfile
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT 3000

CMD ["node", "server.js"]
```

## 🔄 CI/CD PIPELINE

### GitHub Actions Main Workflow
```yaml
# .github/workflows/main.yml
name: AURA CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  release:
    types: [created]

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: aura
  EKS_CLUSTER: aura-cluster

jobs:
  # ========== CODE QUALITY ==========
  quality:
    name: Code Quality Checks
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Run Prettier
        run: npm run format:check
      
      - name: TypeScript check
        run: npm run type-check
      
      - name: SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  # ========== TESTING ==========
  test:
    name: Test Suite
    runs-on: ubuntu-latest
    needs: quality
    
    services:
      postgres:
        image: postgis/postgis:15-3.3
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: aura_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit -- --coverage
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/aura_test
          REDIS_URL: redis://localhost:6379
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella

  # ========== SECURITY ==========
  security:
    name: Security Scanning
    runs-on: ubuntu-latest
    needs: quality
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: Run Snyk Security Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
      
      - name: OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'AURA'
          path: '.'
          format: 'HTML'

  # ========== BUILD & PUSH ==========
  build:
    name: Build and Push Images
    runs-on: ubuntu-latest
    needs: [test, security]
    if: github.event_name == 'push' && (github.ref == 'refs/heads/main' || github.ref == 'refs/heads/develop')
    
    strategy:
      matrix:
        service: [backend, frontend]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      
      - name: Build, tag, and push image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY-${{ matrix.service }}:$IMAGE_TAG \
                       -t $ECR_REGISTRY/$ECR_REPOSITORY-${{ matrix.service }}:latest \
                       -f ./${{ matrix.service }}/Dockerfile \
                       ./${{ matrix.service }}
          
          docker push $ECR_REGISTRY/$ECR_REPOSITORY-${{ matrix.service }}:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY-${{ matrix.service }}:latest

  # ========== DEPLOY STAGING ==========
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.aura.app
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Update kubeconfig
        run: |
          aws eks update-kubeconfig --region ${{ env.AWS_REGION }} --name ${{ env.EKS_CLUSTER }}
      
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/aura-backend \
            backend=$ECR_REGISTRY/$ECR_REPOSITORY-backend:${{ github.sha }} \
            -n staging
          
          kubectl set image deployment/aura-frontend \
            frontend=$ECR_REGISTRY/$ECR_REPOSITORY-frontend:${{ github.sha }} \
            -n staging
          
          kubectl rollout status deployment/aura-backend -n staging
          kubectl rollout status deployment/aura-frontend -n staging
      
      - name: Run smoke tests
        run: npm run test:smoke -- --env=staging

  # ========== DEPLOY PRODUCTION ==========
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://aura.app
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID_PROD }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY_PROD }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Blue/Green Deployment
        run: |
          # Deploy to green environment
          kubectl set image deployment/aura-backend-green \
            backend=$ECR_REGISTRY/$ECR_REPOSITORY-backend:${{ github.sha }} \
            -n production
          
          kubectl set image deployment/aura-frontend-green \
            frontend=$ECR_REGISTRY/$ECR_REPOSITORY-frontend:${{ github.sha }} \
            -n production
          
          # Wait for green to be ready
          kubectl rollout status deployment/aura-backend-green -n production
          kubectl rollout status deployment/aura-frontend-green -n production
          
          # Run health checks
          ./scripts/health-check.sh green
          
          # Switch traffic to green
          kubectl patch service aura-backend -p '{"spec":{"selector":{"version":"green"}}}' -n production
          kubectl patch service aura-frontend -p '{"spec":{"selector":{"version":"green"}}}' -n production
          
          # Scale down blue
          kubectl scale deployment aura-backend-blue --replicas=0 -n production
          kubectl scale deployment aura-frontend-blue --replicas=0 -n production
      
      - name: Notify deployment
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Production deployment completed for ${{ github.sha }}'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

## 🌐 KUBERNETES CONFIGURATION

### Backend Deployment
```yaml
# k8s/backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aura-backend
  namespace: production
  labels:
    app: aura
    component: backend
    version: blue
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: aura
      component: backend
      version: blue
  template:
    metadata:
      labels:
        app: aura
        component: backend
        version: blue
    spec:
      containers:
      - name: backend
        image: ${ECR_REGISTRY}/aura-backend:latest
        ports:
        - containerPort: 3000
          name: http
        env:
        - name: NODE_ENV
          value: production
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: aura-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: aura-secrets
              key: redis-url
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - aura
              topologyKey: kubernetes.io/hostname
---
apiVersion: v1
kind: Service
metadata:
  name: aura-backend
  namespace: production
spec:
  selector:
    app: aura
    component: backend
    version: blue
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: aura-backend-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: aura-backend
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## 🔧 TERRAFORM INFRASTRUCTURE

### AWS Infrastructure
```hcl
# terraform/aws/main.tf
terraform {
  required_version = ">= 1.0"
  
  backend "s3" {
    bucket = "aura-terraform-state"
    key    = "infrastructure/terraform.tfstate"
    region = "us-east-1"
    encrypt = true
    dynamodb_table = "terraform-locks"
  }
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.23"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# VPC Configuration
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "aura-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["${var.aws_region}a", "${var.aws_region}b", "${var.aws_region}c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = true
  enable_dns_hostnames = true
  
  tags = {
    Environment = var.environment
    Application = "AURA"
  }
}

# EKS Cluster
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  version = "19.15.3"
  
  cluster_name    = "aura-cluster"
  cluster_version = "1.28"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  enable_irsa = true
  
  eks_managed_node_groups = {
    general = {
      desired_size = 3
      min_size     = 2
      max_size     = 10
      
      instance_types = ["t3.large"]
      
      k8s_labels = {
        Environment = var.environment
        Application = "AURA"
      }
    }
  }
}

# RDS Aurora PostgreSQL
module "aurora" {
  source = "terraform-aws-modules/rds-aurora/aws"
  version = "8.3.1"
  
  name           = "aura-postgres"
  engine         = "aurora-postgresql"
  engine_version = "15.4"
  
  vpc_id               = module.vpc.vpc_id
  db_subnet_group_name = module.vpc.database_subnet_group_name
  
  master_username = "auraadmin"
  master_password = random_password.db_password.result
  
  instance_class = "db.r6g.large"
  instances = {
    1 = {}
    2 = {}
  }
  
  backup_retention_period = 7
  preferred_backup_window = "03:00-04:00"
  
  enabled_cloudwatch_logs_exports = ["postgresql"]
  
  tags = {
    Environment = var.environment
    Application = "AURA"
  }
}

# ElastiCache Redis
resource "aws_elasticache_replication_group" "redis" {
  replication_group_id       = "aura-redis"
  replication_group_description = "AURA Redis cluster"
  
  engine               = "redis"
  engine_version       = "7.0"
  node_type           = "cache.r6g.large"
  number_cache_clusters = 2
  
  automatic_failover_enabled = true
  multi_az_enabled          = true
  
  subnet_group_name = aws_elasticache_subnet_group.redis.name
  security_group_ids = [aws_security_group.redis.id]
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  
  snapshot_retention_limit = 7
  snapshot_window         = "03:00-05:00"
  
  tags = {
    Environment = var.environment
    Application = "AURA"
  }
}

# S3 Buckets
resource "aws_s3_bucket" "media" {
  bucket = "aura-media-${var.environment}"
  
  tags = {
    Environment = var.environment
    Application = "AURA"
  }
}

resource "aws_s3_bucket_versioning" "media" {
  bucket = aws_s3_bucket.media.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_encryption" "media" {
  bucket = aws_s3_bucket.media.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# CloudFront Distribution
resource "aws_cloudfront_distribution" "cdn" {
  enabled         = true
  is_ipv6_enabled = true
  comment         = "AURA CDN"
  
  origin {
    domain_name = aws_s3_bucket.media.bucket_regional_domain_name
    origin_id   = "S3-${aws_s3_bucket.media.id}"
    
    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.oai.cloudfront_access_identity_path
    }
  }
  
  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD", "OPTIONS"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-${aws_s3_bucket.media.id}"
    
    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }
    
    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
  }
  
  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }
  
  viewer_certificate {
    cloudfront_default_certificate = true
  }
  
  tags = {
    Environment = var.environment
    Application = "AURA"
  }
}
```

## 📊 MONITORING & OBSERVABILITY

### Prometheus Configuration
```yaml
# monitoring/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

rule_files:
  - "alerts.yml"

scrape_configs:
  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
      - role: endpoints
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https

  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
```

### Alert Rules
```yaml
# monitoring/alerts.yml
groups:
  - name: aura_alerts
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
          service: aura
        annotations:
          summary: "High error rate detected"
          description: "Error rate is above 5% for {{ $labels.instance }}"
      
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, http_request_duration_seconds_bucket) > 0.5
        for: 5m
        labels:
          severity: warning
          service: aura
        annotations:
          summary: "High response time detected"
          description: "95th percentile response time is above 500ms"
      
      - alert: PodCrashLooping
        expr: rate(kube_pod_container_status_restarts_total[1h]) > 3
        for: 5m
        labels:
          severity: critical
          service: aura
        annotations:
          summary: "Pod is crash looping"
          description: "Pod {{ $labels.pod }} has restarted {{ $value }} times in the last hour"
      
      - alert: HighMemoryUsage
        expr: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.9
        for: 5m
        labels:
          severity: warning
          service: aura
        annotations:
          summary: "High memory usage"
          description: "Container {{ $labels.container }} is using > 90% of memory limit"
      
      - alert: DatabaseConnectionsHigh
        expr: pg_stat_activity_count > 80
        for: 5m
        labels:
          severity: warning
          service: aura
        annotations:
          summary: "Database connections high"
          description: "PostgreSQL has {{ $value }} active connections"
```

## 🔐 SECRETS MANAGEMENT

### AWS Secrets Manager
```yaml
# scripts/manage-secrets.sh
#!/bin/bash

# Create secrets in AWS Secrets Manager
aws secretsmanager create-secret \
  --name aura/production/database \
  --secret-string '{
    "username":"auraadmin",
    "password":"${DB_PASSWORD}",
    "host":"aura-db.cluster-xxx.us-east-1.rds.amazonaws.com",
    "port":"5432",
    "database":"aura_production"
  }'

# Sync secrets to Kubernetes
kubectl create secret generic aura-secrets \
  --from-literal=database-url="postgresql://..." \
  --from-literal=redis-url="redis://..." \
  --from-literal=jwt-secret="${JWT_SECRET}" \
  --from-literal=stripe-key="${STRIPE_KEY}" \
  --namespace=production

# Rotate secrets
aws secretsmanager rotate-secret \
  --secret-id aura/production/database \
  --rotation-lambda-arn arn:aws:lambda:...
```

## 🚀 DEPLOYMENT STRATEGIES

### Blue-Green Deployment Script
```bash
#!/bin/bash
# scripts/blue-green-deploy.sh

set -e

ENVIRONMENT=${1:-staging}
VERSION=${2:-latest}
CURRENT_COLOR=$(kubectl get service aura-backend -o jsonpath='{.spec.selector.version}' -n $ENVIRONMENT)
NEW_COLOR=$([ "$CURRENT_COLOR" == "blue" ] && echo "green" || echo "blue")

echo "Deploying version $VERSION to $NEW_COLOR environment"

# Deploy new version
kubectl set image deployment/aura-backend-$NEW_COLOR \
  backend=aura-backend:$VERSION \
  -n $ENVIRONMENT

kubectl set image deployment/aura-frontend-$NEW_COLOR \
  frontend=aura-frontend:$VERSION \
  -n $ENVIRONMENT

# Wait for rollout
kubectl rollout status deployment/aura-backend-$NEW_COLOR -n $ENVIRONMENT
kubectl rollout status deployment/aura-frontend-$NEW_COLOR -n $ENVIRONMENT

# Run health checks
for i in {1..10}; do
  if curl -f https://$NEW_COLOR.$ENVIRONMENT.aura.app/health; then
    echo "Health check passed"
    break
  fi
  echo "Health check attempt $i failed, retrying..."
  sleep 10
done

# Switch traffic
kubectl patch service aura-backend \
  -p '{"spec":{"selector":{"version":"'$NEW_COLOR'"}}}' \
  -n $ENVIRONMENT

kubectl patch service aura-frontend \
  -p '{"spec":{"selector":{"version":"'$NEW_COLOR'"}}}' \
  -n $ENVIRONMENT

echo "Traffic switched to $NEW_COLOR"

# Scale down old version
sleep 60  # Wait for connections to drain
kubectl scale deployment aura-backend-$CURRENT_COLOR --replicas=0 -n $ENVIRONMENT
kubectl scale deployment aura-frontend-$CURRENT_COLOR --replicas=0 -n $ENVIRONMENT

echo "Deployment complete! Old version ($CURRENT_COLOR) scaled down"
```

### Rollback Script
```bash
#!/bin/bash
# scripts/rollback.sh

set -e

ENVIRONMENT=${1:-staging}
CURRENT_COLOR=$(kubectl get service aura-backend -o jsonpath='{.spec.selector.version}' -n $ENVIRONMENT)
PREVIOUS_COLOR=$([ "$CURRENT_COLOR" == "blue" ] && echo "green" || echo "blue")

echo "Rolling back from $CURRENT_COLOR to $PREVIOUS_COLOR"

# Scale up previous version
kubectl scale deployment aura-backend-$PREVIOUS_COLOR --replicas=3 -n $ENVIRONMENT
kubectl scale deployment aura-frontend-$PREVIOUS_COLOR --replicas=3 -n $ENVIRONMENT

# Wait for pods to be ready
kubectl rollout status deployment/aura-backend-$PREVIOUS_COLOR -n $ENVIRONMENT
kubectl rollout status deployment/aura-frontend-$PREVIOUS_COLOR -n $ENVIRONMENT

# Switch traffic back
kubectl patch service aura-backend \
  -p '{"spec":{"selector":{"version":"'$PREVIOUS_COLOR'"}}}' \
  -n $ENVIRONMENT

kubectl patch service aura-frontend \
  -p '{"spec":{"selector":{"version":"'$PREVIOUS_COLOR'"}}}' \
  -n $ENVIRONMENT

# Scale down current (failed) version
kubectl scale deployment aura-backend-$CURRENT_COLOR --replicas=0 -n $ENVIRONMENT
kubectl scale deployment aura-frontend-$CURRENT_COLOR --replicas=0 -n $ENVIRONMENT

echo "Rollback complete!"
```

## 📈 PERFORMANCE OPTIMIZATION

### CDN Configuration
```nginx
# nginx/cdn.conf
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=STATIC:10m inactive=7d use_temp_path=off;

server {
    listen 443 ssl http2;
    server_name cdn.aura.app;
    
    ssl_certificate /etc/ssl/certs/aura.crt;
    ssl_certificate_key /etc/ssl/private/aura.key;
    
    location ~* \.(jpg|jpeg|png|gif|webp|ico|css|js|svg|woff|woff2)$ {
        proxy_pass http://origin.aura.app;
        proxy_cache STATIC;
        proxy_cache_valid 200 7d;
        proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
        proxy_cache_background_update on;
        proxy_cache_lock on;
        
        add_header Cache-Control "public, max-age=604800, immutable";
        add_header X-Cache-Status $upstream_cache_status;
    }
}
```

## 🔄 BACKUP & DISASTER RECOVERY

### Backup Script
```bash
#!/bin/bash
# scripts/backup.sh

set -e

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/$TIMESTAMP"

# Database backup
echo "Backing up PostgreSQL..."
pg_dump $DATABASE_URL | gzip > $BACKUP_DIR/postgres_$TIMESTAMP.sql.gz

# Redis backup
echo "Backing up Redis..."
redis-cli --rdb $BACKUP_DIR/redis_$TIMESTAMP.rdb

# Upload to S3
echo "Uploading to S3..."
aws s3 cp $BACKUP_DIR s3://aura-backups/$TIMESTAMP/ --recursive

# Clean old local backups
find /backups -mtime +7 -delete

echo "Backup completed: $TIMESTAMP"
```

---

**DevOps Team Lead: Infrastructure Department**
*Last Updated: January 2024*