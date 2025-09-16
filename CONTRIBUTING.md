# Contributing to AURA 2.0

🌈 Thank you for your interest in contributing to AURA 2.0! This document provides guidelines and instructions for contributing to our LGBTQIA+ dating platform.

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How to Contribute](#how-to-contribute)
- [Development Setup](#development-setup)
- [Coding Standards](#coding-standards)
- [Pull Request Process](#pull-request-process)
- [Commit Guidelines](#commit-guidelines)
- [Testing](#testing)
- [Documentation](#documentation)

## 💜 Code of Conduct

### Our Pledge

We are committed to providing a welcoming, inclusive, and harassment-free environment for everyone, regardless of:
- Gender identity and expression
- Sexual orientation
- Disability
- Physical appearance
- Body size
- Race
- Age
- Religion
- Nationality

### Expected Behavior

- Be respectful and inclusive
- Use welcoming and inclusive language
- Be collaborative and constructive
- Accept constructive criticism gracefully
- Focus on what is best for the community

### Unacceptable Behavior

- Harassment of any kind
- Discriminatory language or actions
- Personal attacks
- Publishing others' private information
- Any conduct which would be considered inappropriate in a professional setting

## 🚀 How to Contribute

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates.

**To report a bug:**

1. Use the issue template
2. Use a clear and descriptive title
3. Describe the exact steps to reproduce the problem
4. Provide specific examples
5. Include screenshots if applicable
6. Describe the expected behavior
7. Include your environment details

### Suggesting Enhancements

**To suggest an enhancement:**

1. Use the feature request template
2. Provide a clear use case
3. Explain why this enhancement would be useful
4. Consider the scope and impact
5. Include mockups or examples if possible

### Your First Code Contribution

Unsure where to begin? Look for these labels:

- `good-first-issue` - Simple issues for beginners
- `help-wanted` - Issues where we need community help
- `documentation` - Documentation improvements

## 💻 Development Setup

### Prerequisites

```bash
# Required versions
Node.js: 20.x or higher
PostgreSQL: 15.x or higher
Redis: 7.x or higher
Docker: 20.x or higher
```

### Local Development

1. **Fork and clone the repository**

```bash
git clone https://github.com/YOUR_USERNAME/Aura-2.0.git
cd Aura-2.0
```

2. **Install dependencies**

```bash
npm install
```

3. **Set up environment variables**

```bash
cp .env.example .env
# Edit .env with your local configuration
```

4. **Start Docker services**

```bash
docker-compose up -d
```

5. **Run database migrations**

```bash
npm run db:migrate
npm run db:seed
```

6. **Start development servers**

```bash
npm run dev
```

## 📝 Coding Standards

### TypeScript Style Guide

```typescript
// ✅ Good
export interface User {
  id: string;
  displayName: string;
  email: string;
  createdAt: Date;
}

export class UserService {
  constructor(private readonly repository: UserRepository) {}

  async findById(id: string): Promise<User | null> {
    return this.repository.findOne({ where: { id } });
  }
}

// ❌ Bad
export interface user {
  Id: string,
  display_name: string,
  Email: string,
  created: any
}

export class userService {
  findById(id) {
    // Missing types and async/await
  }
}
```

### React/Component Guidelines

```tsx
// ✅ Good
export const ProfileCard: React.FC<ProfileCardProps> = ({ user, onLike, onPass }) => {
  const [isLoading, setIsLoading] = useState(false);

  const handleLike = useCallback(async () => {
    setIsLoading(true);
    try {
      await onLike(user.id);
    } finally {
      setIsLoading(false);
    }
  }, [user.id, onLike]);

  return (
    <div className="profile-card">
      {/* Component content */}
    </div>
  );
};

// ❌ Bad
export function ProfileCard(props) {
  const handleLike = () => {
    props.onLike(props.user.id);
  };

  return <div>{/* Content */}</div>;
}
```

### File Structure

```
src/
├── modules/
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.service.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.spec.ts
│   │   └── dto/
│   └── users/
├── common/
│   ├── decorators/
│   ├── guards/
│   └── interceptors/
└── config/
```

## 🔄 Pull Request Process

### Before Submitting

1. **Ensure your code follows our standards**
   ```bash
   npm run lint
   npm run format
   ```

2. **Run all tests**
   ```bash
   npm test
   npm run test:e2e
   ```

3. **Update documentation if needed**

4. **Write meaningful tests for new features**

### PR Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Screenshots (if applicable)

## Checklist
- [ ] My code follows the style guidelines
- [ ] I have performed a self-review
- [ ] I have commented complex code
- [ ] I have updated documentation
- [ ] My changes generate no warnings
- [ ] I have added tests
- [ ] All tests pass locally
```

### Review Process

1. At least 2 approvals required
2. All CI checks must pass
3. No merge conflicts
4. Documentation updated
5. Follows semantic versioning

## 💬 Commit Guidelines

We use [Conventional Commits](https://www.conventionalcommits.org/)

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style (formatting, etc)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding tests
- `chore`: Maintenance tasks
- `ci`: CI/CD changes

### Examples

```bash
# Good commits
git commit -m "feat(auth): add two-factor authentication"
git commit -m "fix(chat): resolve message delivery issue"
git commit -m "docs(api): update authentication endpoints"

# Bad commits
git commit -m "fixed stuff"
git commit -m "WIP"
git commit -m "Update index.ts"
```

## 🧪 Testing

### Test Coverage Requirements

- Unit tests: 80% minimum
- Integration tests: 70% minimum
- Critical paths: 100% coverage

### Writing Tests

```typescript
// Good test example
describe('UserService', () => {
  let service: UserService;
  let repository: MockType<Repository<User>>;

  beforeEach(() => {
    // Setup
  });

  describe('createUser', () => {
    it('should create a user with valid data', async () => {
      // Arrange
      const userData = createMockUserData();
      
      // Act
      const result = await service.createUser(userData);
      
      // Assert
      expect(result).toBeDefined();
      expect(result.email).toBe(userData.email);
    });

    it('should throw error for duplicate email', async () => {
      // Test implementation
    });
  });
});
```

## 📚 Documentation

### Code Documentation

```typescript
/**
 * Calculates compatibility score between two users
 * @param user1 - First user object
 * @param user2 - Second user object
 * @returns Compatibility score between 0 and 100
 * @example
 * const score = calculateCompatibility(user1, user2);
 * console.log(`Compatibility: ${score}%`);
 */
export function calculateCompatibility(user1: User, user2: User): number {
  // Implementation
}
```

### API Documentation

- Use JSDoc comments
- Include examples
- Document all public methods
- Keep README files updated

## 🎆 Recognition

Contributors will be:
- Listed in our CONTRIBUTORS file
- Mentioned in release notes
- Invited to our contributor events
- Eligible for AURA swag!

## 💔 Questions?

Feel free to:
- Open a discussion in GitHub Discussions
- Join our Discord server
- Email us at dev@aura.app

---

**Thank you for making AURA more inclusive and amazing!** 🌈❤️