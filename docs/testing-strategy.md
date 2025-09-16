# TESTING STRATEGY - AURA 2.0
## Comprehensive Testing Approach

## 🎯 TESTING PHILOSOPHY

```yaml
Principles:
  - Quality over speed
  - Automate everything possible
  - Test early, test often
  - User safety is non-negotiable
  - Performance is a feature
  
Coverage Goals:
  - Unit Tests: 80%
  - Integration Tests: 70%
  - E2E Critical Paths: 100%
  - Security: 100%
```

## 📊 TESTING PYRAMID

```
            /\
           /  \        Manual Testing (5%)
          /    \       - Exploratory
         /      \      - Usability
        /--------\     - Acceptance
       /          \
      /    E2E     \   End-to-End Tests (15%)
     /   Testing    \  - Critical user journeys
    /                \ - Cross-browser/device
   /------------------\
  /                    \
 /    Integration      \ Integration Tests (30%)
/      Testing          \- API endpoints
/                        \- Database operations
/--------------------------\- External services
/                          \
/       Unit Testing        \ Unit Tests (50%)
/                            \- Business logic
/                              \- Components
/                                \- Utilities
```

## 🧪 TESTING TYPES

### 1. UNIT TESTING

#### Backend (Jest + NestJS)
```typescript
// user.service.spec.ts
describe('UserService', () => {
  let service: UserService;
  let repository: Repository<User>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UserService,
        {
          provide: getRepositoryToken(User),
          useClass: Repository,
        },
      ],
    }).compile();

    service = module.get<UserService>(UserService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
  });

  describe('createUser', () => {
    it('should create a user with hashed password', async () => {
      const createUserDto = {
        email: 'test@example.com',
        password: 'Test123!',
        displayName: 'Test User',
      };

      const hashedPassword = await bcrypt.hash(createUserDto.password, 10);
      const savedUser = { ...createUserDto, password: hashedPassword, id: '1' };

      jest.spyOn(repository, 'save').mockResolvedValue(savedUser);

      const result = await service.createUser(createUserDto);

      expect(result).toEqual(expect.objectContaining({
        email: createUserDto.email,
        displayName: createUserDto.displayName,
      }));
      expect(result.password).not.toEqual(createUserDto.password);
    });

    it('should throw error for duplicate email', async () => {
      jest.spyOn(repository, 'findOne').mockResolvedValue({} as User);

      await expect(service.createUser(mockDto)).rejects.toThrow(
        ConflictException,
      );
    });
  });
});
```

#### Frontend (Jest + React Testing Library)
```tsx
// ProfileCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { ProfileCard } from '@/components/ProfileCard';

describe('ProfileCard', () => {
  const mockUser = {
    id: '1',
    displayName: 'Maria Silva',
    age: 28,
    bio: 'Love music and travel',
    photos: [{ url: 'photo.jpg', isPrimary: true }],
    distance: 5.2,
  };

  const mockOnLike = jest.fn();
  const mockOnPass = jest.fn();

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should render user information correctly', () => {
    render(
      <ProfileCard
        user={mockUser}
        onLike={mockOnLike}
        onPass={mockOnPass}
      />
    );

    expect(screen.getByText('Maria Silva, 28')).toBeInTheDocument();
    expect(screen.getByText('Love music and travel')).toBeInTheDocument();
    expect(screen.getByText('5.2 km away')).toBeInTheDocument();
  });

  it('should call onLike when like button clicked', () => {
    render(
      <ProfileCard
        user={mockUser}
        onLike={mockOnLike}
        onPass={mockOnPass}
      />
    );

    fireEvent.click(screen.getByLabelText('Like'));
    expect(mockOnLike).toHaveBeenCalledWith(mockUser.id);
  });

  it('should handle swipe gestures', () => {
    const { container } = render(
      <ProfileCard
        user={mockUser}
        onLike={mockOnLike}
        onPass={mockOnPass}
      />
    );

    // Simulate swipe right
    fireEvent.touchStart(container.firstChild, {
      touches: [{ clientX: 0, clientY: 0 }],
    });
    fireEvent.touchEnd(container.firstChild, {
      touches: [{ clientX: 100, clientY: 0 }],
    });

    expect(mockOnLike).toHaveBeenCalled();
  });
});
```

### 2. INTEGRATION TESTING

#### API Testing (Supertest)
```typescript
// auth.e2e-spec.ts
describe('AuthController (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  describe('/auth/register (POST)', () => {
    it('should register a new user', async () => {
      const newUser = {
        email: 'newuser@test.com',
        password: 'SecurePass123!',
        displayName: 'New User',
        birthDate: '1995-01-01',
        genderIdentity: 'woman_cis',
        sexualOrientation: 'lesbian',
      };

      const response = await request(app.getHttpServer())
        .post('/auth/register')
        .send(newUser)
        .expect(201);

      expect(response.body).toHaveProperty('access_token');
      expect(response.body).toHaveProperty('user');
      expect(response.body.user.email).toBe(newUser.email);
    });

    it('should not register with invalid email', async () => {
      const invalidUser = {
        email: 'not-an-email',
        password: 'SecurePass123!',
      };

      await request(app.getHttpServer())
        .post('/auth/register')
        .send(invalidUser)
        .expect(400);
    });
  });

  describe('/auth/login (POST)', () => {
    it('should login with valid credentials', async () => {
      const response = await request(app.getHttpServer())
        .post('/auth/login')
        .send({
          email: 'test@test.com',
          password: 'TestPass123!',
        })
        .expect(200);

      expect(response.body).toHaveProperty('access_token');
      expect(response.body).toHaveProperty('refresh_token');
    });

    it('should fail with wrong password', async () => {
      await request(app.getHttpServer())
        .post('/auth/login')
        .send({
          email: 'test@test.com',
          password: 'WrongPassword',
        })
        .expect(401);
    });
  });

  afterAll(async () => {
    await app.close();
  });
});
```

#### Database Testing (TestContainers)
```typescript
// database.integration.spec.ts
import { GenericContainer, StartedTestContainer } from 'testcontainers';
import { Client } from 'pg';

describe('Database Integration', () => {
  let container: StartedTestContainer;
  let client: Client;

  beforeAll(async () => {
    container = await new GenericContainer('postgis/postgis:15-3.3')
      .withExposedPorts(5432)
      .withEnv('POSTGRES_USER', 'test')
      .withEnv('POSTGRES_PASSWORD', 'test')
      .withEnv('POSTGRES_DB', 'aura_test')
      .start();

    client = new Client({
      host: container.getHost(),
      port: container.getMappedPort(5432),
      user: 'test',
      password: 'test',
      database: 'aura_test',
    });

    await client.connect();
  });

  it('should handle geospatial queries', async () => {
    await client.query(`
      CREATE TABLE test_locations (
        id SERIAL PRIMARY KEY,
        location GEOGRAPHY(POINT, 4326)
      )
    `);

    // Insert test data
    await client.query(
      `INSERT INTO test_locations (location) VALUES 
       (ST_MakePoint(-46.6333, -23.5505)),
       (ST_MakePoint(-46.6350, -23.5520))`,
    );

    // Query nearby locations
    const result = await client.query(
      `SELECT id, ST_Distance(location, ST_MakePoint(-46.6340, -23.5510)) as distance
       FROM test_locations
       WHERE ST_DWithin(location, ST_MakePoint(-46.6340, -23.5510), 1000)
       ORDER BY distance`,
    );

    expect(result.rows).toHaveLength(2);
    expect(result.rows[0].distance).toBeLessThan(result.rows[1].distance);
  });

  afterAll(async () => {
    await client.end();
    await container.stop();
  });
});
```

### 3. END-TO-END TESTING

#### Web E2E (Playwright)
```typescript
// user-journey.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Journey - From Registration to First Match', () => {
  test('complete user journey', async ({ page }) => {
    // 1. Registration
    await page.goto('https://localhost:3001');
    await page.click('text=Sign Up');
    
    await page.fill('input[name="email"]', 'testuser@example.com');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.fill('input[name="displayName"]', 'Test User');
    await page.fill('input[name="birthDate"]', '1995-01-01');
    
    await page.selectOption('select[name="genderIdentity"]', 'woman_cis');
    await page.selectOption('select[name="sexualOrientation"]', 'lesbian');
    
    await page.click('button[type="submit"]');
    
    // 2. Email Verification
    await expect(page).toHaveURL(/\/verify-email/);
    // Simulate email verification
    await page.goto('https://localhost:3001/verify-email?token=test-token');
    
    // 3. Complete Profile
    await expect(page).toHaveURL(/\/onboarding/);
    await page.fill('textarea[name="bio"]', 'Love hiking and coffee');
    
    // Upload photos
    await page.setInputFiles('input[type="file"]', 'test-assets/photo1.jpg');
    
    // Select interests
    await page.click('text=Music');
    await page.click('text=Travel');
    await page.click('text=Coffee');
    
    await page.click('text=Complete Profile');
    
    // 4. Choose Mode
    await expect(page).toHaveURL(/\/mode-selection/);
    await page.click('text=ALMA - Deep Connections');
    
    // 5. Start Swiping
    await expect(page).toHaveURL(/\/alma/);
    
    // Wait for profiles to load
    await page.waitForSelector('.profile-card');
    
    // Swipe right on first profile
    await page.locator('.profile-card').first().swipe({ direction: 'right' });
    
    // Check for match
    const matchModal = page.locator('.match-celebration');
    if (await matchModal.isVisible()) {
      await expect(matchModal).toContainText("It's a Match!");
      await page.click('text=Send a Message');
      
      // 6. Send First Message
      await page.fill('input[name="message"]', 'Hi! Love your profile 😊');
      await page.click('button[aria-label="Send"]');
      
      await expect(page.locator('.message-bubble')).toContainText('Hi! Love your profile');
    }
  });
});
```

#### Mobile E2E (Detox)
```javascript
// firstMatch.e2e.js
describe('First Match Flow', () => {
  beforeAll(async () => {
    await device.launchApp({
      newInstance: true,
      permissions: { notifications: 'YES', location: 'always' },
    });
  });

  it('should complete onboarding and make first match', async () => {
    // Login
    await element(by.id('email-input')).typeText('test@example.com');
    await element(by.id('password-input')).typeText('TestPass123!');
    await element(by.id('login-button')).tap();

    // Wait for main screen
    await waitFor(element(by.id('mode-selector')))
      .toBeVisible()
      .withTimeout(5000);

    // Select FLASH mode
    await element(by.id('flash-mode-button')).tap();

    // Wait for grid to load
    await waitFor(element(by.id('user-grid')))
      .toBeVisible()
      .withTimeout(3000);

    // Double tap to like first user
    await element(by.id('grid-user-0')).multiTap(2);

    // Check if match occurred
    try {
      await waitFor(element(by.id('match-modal')))
        .toBeVisible()
        .withTimeout(2000);
      
      await expect(element(by.text("It's a Match!"))).toBeVisible();
      await element(by.id('start-chat-button')).tap();
      
      // Send message
      await element(by.id('message-input')).typeText('Hey there!');
      await element(by.id('send-button')).tap();
      
      await expect(element(by.text('Hey there!'))).toBeVisible();
    } catch (e) {
      // No match, continue
    }
  });
});
```

### 4. PERFORMANCE TESTING

#### Load Testing (K6)
```javascript
// load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';

const errorRate = new Rate('errors');

export const options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up
    { duration: '5m', target: 100 }, // Stay at 100 users
    { duration: '2m', target: 500 }, // Ramp up to 500
    { duration: '5m', target: 500 }, // Stay at 500
    { duration: '2m', target: 1000 }, // Ramp to 1000
    { duration: '5m', target: 1000 }, // Stay at 1000
    { duration: '5m', target: 0 }, // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests under 500ms
    errors: ['rate<0.01'], // Error rate under 1%
  },
};

const BASE_URL = 'https://api.aura.app';

export default function () {
  // Login
  const loginRes = http.post(`${BASE_URL}/auth/login`, JSON.stringify({
    email: 'loadtest@example.com',
    password: 'LoadTest123!',
  }), {
    headers: { 'Content-Type': 'application/json' },
  });

  check(loginRes, {
    'login successful': (r) => r.status === 200,
    'has token': (r) => r.json('access_token') !== '',
  });

  errorRate.add(loginRes.status !== 200);

  const token = loginRes.json('access_token');
  const headers = {
    Authorization: `Bearer ${token}`,
    'Content-Type': 'application/json',
  };

  // Get FLASH grid
  const gridRes = http.get(`${BASE_URL}/modes/flash/grid`, { headers });
  
  check(gridRes, {
    'grid loaded': (r) => r.status === 200,
    'has users': (r) => r.json('data.users').length > 0,
    'response time OK': (r) => r.timings.duration < 500,
  });

  // Simulate user action (like)
  const users = gridRes.json('data.users');
  if (users && users.length > 0) {
    const randomUser = users[Math.floor(Math.random() * users.length)];
    
    const likeRes = http.post(`${BASE_URL}/actions/like`, JSON.stringify({
      target_user_id: randomUser.id,
      mode: 'FLASH',
    }), { headers });

    check(likeRes, {
      'like successful': (r) => r.status === 200,
    });
  }

  sleep(1); // Think time
}
```

### 5. SECURITY TESTING

#### Security Audit Checklist
```yaml
Authentication:
  - [ ] Password complexity enforced
  - [ ] Bcrypt/Argon2 for hashing
  - [ ] JWT properly signed
  - [ ] Refresh token rotation
  - [ ] Session timeout
  - [ ] Account lockout after failures

Authorization:
  - [ ] RBAC implemented
  - [ ] Resource ownership verified
  - [ ] Admin actions logged
  - [ ] Privilege escalation prevented

Input Validation:
  - [ ] SQL injection prevented
  - [ ] XSS protection
  - [ ] CSRF tokens
  - [ ] File upload validation
  - [ ] Rate limiting

Data Protection:
  - [ ] TLS 1.3 enforced
  - [ ] PII encrypted at rest
  - [ ] Sensitive data masked in logs
  - [ ] Secure headers (HSTS, CSP)

Compliance:
  - [ ] LGPD compliant
  - [ ] GDPR ready
  - [ ] Data retention policies
  - [ ] Right to deletion
```

#### Penetration Testing Script
```bash
#!/bin/bash
# security-scan.sh

echo "Starting AURA Security Scan..."

# OWASP ZAP Scan
echo "Running OWASP ZAP..."
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t https://api.aura.app \
  -r zap-report.html

# Nikto Web Scanner
echo "Running Nikto..."
nikto -h https://api.aura.app -output nikto-report.txt

# SQLMap for SQL Injection
echo "Testing SQL Injection..."
sqlmap -u "https://api.aura.app/users?id=1" \
  --batch --random-agent --level=5 --risk=3

# Nmap Port Scan
echo "Running Nmap..."
nmap -sV -sC -O -A api.aura.app -oN nmap-report.txt

# SSL/TLS Test
echo "Testing SSL/TLS..."
testssl --fulltest https://api.aura.app

# Dependency Check
echo "Checking dependencies..."
npm audit
snyk test

echo "Security scan complete. Check reports for details."
```

## 📱 MOBILE SPECIFIC TESTING

### iOS Testing
```swift
// PhotoVerificationTests.swift
import XCTest
@testable import AURA

class PhotoVerificationTests: XCTestCase {
    var sut: PhotoVerificationService!
    
    override func setUp() {
        super.setUp()
        sut = PhotoVerificationService()
    }
    
    func testFaceDetection() async throws {
        let image = UIImage(named: "test-face")!
        
        let result = await sut.verifyPhoto(image)
        
        XCTAssertTrue(result.hasFace)
        XCTAssertTrue(result.confidence > 0.9)
        XCTAssertEqual(result.faceCount, 1)
    }
    
    func testMultipleFacesRejection() async throws {
        let image = UIImage(named: "test-multiple-faces")!
        
        let result = await sut.verifyPhoto(image)
        
        XCTAssertFalse(result.isValid)
        XCTAssertEqual(result.error, .multipleFacesDetected)
    }
}
```

### Android Testing
```kotlin
// MatchingAlgorithmTest.kt
@RunWith(AndroidJUnit4::class)
class MatchingAlgorithmTest {
    
    @get:Rule
    val instantTaskExecutorRule = InstantTaskExecutorRule()
    
    private lateinit var algorithm: MatchingAlgorithm
    private lateinit var database: AuraDatabase
    
    @Before
    fun setup() {
        val context = ApplicationProvider.getApplicationContext<Context>()
        database = Room.inMemoryDatabaseBuilder(
            context,
            AuraDatabase::class.java
        ).allowMainThreadQueries().build()
        
        algorithm = MatchingAlgorithm(database)
    }
    
    @Test
    fun testCompatibilityScoring() = runBlocking {
        val user1 = TestData.createUser(
            interests = listOf("music", "travel", "coffee")
        )
        val user2 = TestData.createUser(
            interests = listOf("music", "books", "coffee")
        )
        
        val score = algorithm.calculateCompatibility(user1, user2)
        
        assertThat(score).isGreaterThan(0.6f) // 2/3 interests match
        assertThat(score).isLessThan(1.0f)
    }
}
```

## 🎭 USABILITY TESTING

### User Testing Protocol
```yaml
Participants: 20 users from target demographic

Tasks:
  1. Complete registration and profile setup
  2. Find and use FLASH mode
  3. Make 5 swipes in ALMA mode
  4. Send a message to a match
  5. Join a group in VIBE mode
  6. Report a fake profile
  7. Upgrade to premium

Metrics:
  - Task completion rate
  - Time to complete
  - Error rate
  - User satisfaction (SUS score)
  - Qualitative feedback

Success Criteria:
  - Completion rate > 90%
  - SUS score > 68
  - No critical usability issues
```

## 🔄 CONTINUOUS TESTING

### CI/CD Pipeline Tests
```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: cd backend && npm ci
      
      - name: Run linting
        run: cd backend && npm run lint
      
      - name: Run unit tests
        run: cd backend && npm run test:unit
      
      - name: Run integration tests
        run: cd backend && npm run test:integration
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./backend/coverage/lcov.info

  frontend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      
      - name: Install dependencies
        run: cd frontend && npm ci
      
      - name: Run tests
        run: cd frontend && npm test
      
      - name: Run E2E tests
        run: cd frontend && npm run test:e2e

  mobile-tests:
    strategy:
      matrix:
        platform: [ios, android]
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run iOS tests
        if: matrix.platform == 'ios'
        run: |
          cd mobile/ios
          xcodebuild test -scheme AURA -destination 'platform=iOS Simulator,name=iPhone 14'
      
      - name: Run Android tests
        if: matrix.platform == 'android'
        run: |
          cd mobile/android
          ./gradlew test

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Snyk Security Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

## 📊 TEST METRICS & REPORTING

### Dashboard Metrics
```yaml
Code Quality:
  - Test coverage percentage
  - Technical debt ratio
  - Code smell count
  - Vulnerability count

Test Execution:
  - Pass/fail rate
  - Test execution time
  - Flaky test count
  - Test stability trend

Defects:
  - Open bugs by severity
  - Bug discovery rate
  - Mean time to fix
  - Escape rate to production

Performance:
  - API response times
  - Page load speeds
  - Crash rate
  - Memory usage
```

## 🚀 TEST AUTOMATION ROADMAP

### Phase 1: Foundation (Weeks 1-4)
- Setup test frameworks
- Create test data factories
- Implement basic unit tests
- Setup CI/CD integration

### Phase 2: Expansion (Weeks 5-8)
- Add integration tests
- Implement E2E critical paths
- Add performance baselines
- Setup monitoring

### Phase 3: Maturity (Weeks 9-12)
- Achieve 80% coverage
- Implement visual regression
- Add chaos testing
- Complete security scanning

---

*"Quality is not an act, it is a habit."* - Aristotle

**Testing Team Lead: QA Department**
*Last Updated: January 2024*