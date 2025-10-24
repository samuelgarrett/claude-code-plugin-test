# TDD Expert

**Description**: Master test-driven development with the Red-Green-Refactor cycle for reliable, maintainable code

## Core Philosophy

Test-Driven Development (TDD) is not just about testing - it's a design methodology that leads to better code architecture, fewer bugs, and higher confidence in changes.

## The Red-Green-Refactor Cycle

### 🔴 **RED: Write a Failing Test**
Write the smallest test that would verify the next piece of functionality
- Test should fail for the right reason
- Test should be specific and focused
- Test should describe desired behavior, not implementation

### 🟢 **GREEN: Make It Pass**
Write the minimal code to make the test pass
- Don't worry about perfect code yet
- Simplest solution that works
- It's OK to hardcode initially

### 🔵 **REFACTOR: Improve the Code**
Clean up both test and production code
- Remove duplication
- Improve names
- Extract functions/classes
- Optimize if needed
- Tests must still pass

### ♻️ **REPEAT**
Continue the cycle for the next piece of functionality

## TDD Workflow

### Starting a New Feature
```
1. [ ] Write a list of test cases to implement
2. [ ] Pick the simplest test case
3. [ ] Write a failing test (RED)
4. [ ] Run the test - verify it fails
5. [ ] Write minimal code to pass (GREEN)
6. [ ] Run the test - verify it passes
7. [ ] Refactor (REFACTOR)
8. [ ] Run tests - verify still passing
9. [ ] Pick next test case
10. [ ] Repeat
```

### Test Case Brainstorming
Before coding, list out test scenarios:
```
Feature: User Login

Test Cases:
- [ ] Valid credentials → success
- [ ] Invalid password → error
- [ ] Nonexistent user → error
- [ ] Empty email → validation error
- [ ] Empty password → validation error
- [ ] SQL injection attempt → sanitized
- [ ] Rate limiting → too many attempts blocked
- [ ] Session creation on success
- [ ] Failed attempts logged
```

## Writing Good Tests

### The AAA Pattern
```javascript
test('should calculate total price with tax', () => {
  // ARRANGE: Set up test data
  const items = [
    { price: 10, quantity: 2 },
    { price: 5, quantity: 1 }
  ];
  const taxRate = 0.1;

  // ACT: Execute the code under test
  const result = calculateTotal(items, taxRate);

  // ASSERT: Verify the result
  expect(result).toBe(27.5); // (10*2 + 5*1) * 1.1
});
```

### Test Naming Conventions
```
Good test names are descriptive and follow patterns:

Pattern 1: should_ExpectedBehavior_When_StateUnderTest
  ✓ should_ReturnZero_When_CartIsEmpty
  ✓ should_ThrowError_When_PriceIsNegative

Pattern 2: When_StateUnderTest_Expect_ExpectedBehavior
  ✓ When_CartIsEmpty_Expect_ZeroTotal
  ✓ When_PriceIsNegative_Expect_Error

Pattern 3: GivenContext_WhenAction_ThenOutcome
  ✓ GivenEmptyCart_WhenCalculatingTotal_ThenReturnsZero
  ✓ GivenInvalidPrice_WhenCalculating_ThenThrowsError
```

### Test Organization
```javascript
describe('ShoppingCart', () => {
  describe('addItem', () => {
    it('should add item to empty cart', () => { });
    it('should increment quantity for existing item', () => { });
    it('should throw error for negative quantity', () => { });
  });

  describe('removeItem', () => {
    it('should remove item from cart', () => { });
    it('should do nothing if item not in cart', () => { });
  });

  describe('calculateTotal', () => {
    it('should return 0 for empty cart', () => { });
    it('should sum all item prices', () => { });
    it('should apply tax rate correctly', () => { });
  });
});
```

## Types of Tests

### 1. **Unit Tests**
Test individual functions/methods in isolation
```javascript
// Testing a pure function
describe('formatCurrency', () => {
  it('should format USD correctly', () => {
    expect(formatCurrency(1234.56, 'USD')).toBe('$1,234.56');
  });

  it('should handle zero', () => {
    expect(formatCurrency(0, 'USD')).toBe('$0.00');
  });

  it('should round to 2 decimals', () => {
    expect(formatCurrency(1.234, 'USD')).toBe('$1.23');
  });
});
```

### 2. **Integration Tests**
Test multiple components working together
```javascript
// Testing API endpoint with database
describe('POST /api/users', () => {
  it('should create user and return ID', async () => {
    const userData = { email: 'test@example.com', name: 'Test' };
    const response = await request(app).post('/api/users').send(userData);

    expect(response.status).toBe(201);
    expect(response.body).toHaveProperty('id');

    // Verify in database
    const user = await db.users.findById(response.body.id);
    expect(user.email).toBe(userData.email);
  });
});
```

### 3. **End-to-End Tests**
Test complete user workflows
```javascript
// Testing full user journey
describe('User Registration Flow', () => {
  it('should allow new user to register and login', async () => {
    // Visit registration page
    await page.goto('/register');

    // Fill form
    await page.fill('#email', 'newuser@example.com');
    await page.fill('#password', 'SecurePass123!');
    await page.click('button[type="submit"]');

    // Should redirect to dashboard
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('.welcome-message')).toContainText('Welcome');
  });
});
```

## Test Doubles (Mocks, Stubs, Spies)

### When to Use Mocks
```javascript
// Mock external dependencies for unit tests
describe('UserService', () => {
  it('should send welcome email on registration', async () => {
    // Mock the email service
    const mockEmailService = {
      send: jest.fn().mockResolvedValue(true)
    };

    const userService = new UserService(mockEmailService);
    await userService.registerUser({ email: 'test@example.com' });

    // Verify email was sent
    expect(mockEmailService.send).toHaveBeenCalledWith({
      to: 'test@example.com',
      template: 'welcome'
    });
  });
});
```

### Dependency Injection for Testability
```typescript
// Bad: Hard to test
class UserService {
  async createUser(data: UserData) {
    await Database.save(data); // Direct dependency
    await EmailService.send(data.email); // Hard to mock
  }
}

// Good: Easy to test
class UserService {
  constructor(
    private db: DatabaseService,
    private email: EmailService
  ) {}

  async createUser(data: UserData) {
    await this.db.save(data);
    await this.email.send(data.email);
  }
}

// Now we can inject mocks in tests
```

## TDD Anti-Patterns to Avoid

### ❌ **Testing Implementation Details**
```javascript
// Bad: Tests implementation
test('should call helper function 3 times', () => {
  const spy = jest.spyOn(obj, 'helperFunc');
  obj.mainFunc();
  expect(spy).toHaveBeenCalledTimes(3);
});

// Good: Tests behavior
test('should return correct result', () => {
  expect(obj.mainFunc()).toBe(expectedResult);
});
```

### ❌ **Overly Complex Tests**
```javascript
// Bad: Test is harder to understand than the code
test('complex test', () => {
  const data = generateComplexData();
  for (let i = 0; i < data.length; i++) {
    if (data[i].type === 'A') {
      // ... complex logic
    }
  }
  expect(result).toBe(/* computed value */);
});

// Good: Simple, focused test
test('should process type A items correctly', () => {
  const item = { type: 'A', value: 10 };
  expect(processItem(item)).toBe(20);
});
```

### ❌ **Test Interdependence**
```javascript
// Bad: Tests depend on each other
let userId;
test('create user', () => {
  userId = createUser(); // Side effect
});
test('update user', () => {
  updateUser(userId); // Depends on previous test
});

// Good: Each test is independent
test('create user', () => {
  const userId = createUser();
  expect(userId).toBeDefined();
});
test('update user', () => {
  const userId = createUser(); // Setup within test
  const result = updateUser(userId);
  expect(result).toBe(true);
});
```

## Test Coverage Targets

### Aim for High Coverage, Not 100%
- **Statements**: 80%+ (most lines executed)
- **Branches**: 75%+ (most if/else paths tested)
- **Functions**: 90%+ (most functions tested)
- **Lines**: 80%+ (most lines covered)

### What to Prioritize
1. **Critical business logic** - Must have 100% coverage
2. **Complex algorithms** - High coverage with edge cases
3. **Public APIs** - All entry points tested
4. **Error handling** - All error paths verified

### What's OK to Skip
- Trivial getters/setters
- Framework boilerplate
- Third-party library code
- Configuration files

## TDD Best Practices

### 1. **Write Tests First** (Truly!)
```
❌ Bad: Write code first, then add tests
✓ Good: Write failing test, then make it pass
```

### 2. **One Assert Per Test** (Usually)
```javascript
// Prefer focused tests
test('should create user with correct email', () => {
  const user = createUser({ email: 'test@example.com' });
  expect(user.email).toBe('test@example.com');
});

test('should create user with hashed password', () => {
  const user = createUser({ password: 'secret' });
  expect(user.password).not.toBe('secret');
  expect(user.password).toMatch(/^\$2[aby]\$/);
});
```

### 3. **Fast Tests**
```
- Unit tests: < 10ms each
- Integration tests: < 100ms each
- E2E tests: < 5s each

If slower, consider:
- Using test doubles
- Running in parallel
- Optimizing setup/teardown
```

### 4. **Reliable Tests**
```
Tests should:
- ✓ Pass consistently (no flakiness)
- ✓ Fail for the right reasons
- ✓ Be deterministic
- ✓ Not depend on external state
- ✓ Clean up after themselves
```

### 5. **Readable Tests**
```javascript
// Tests are documentation - make them clear
test('should allow user to checkout with valid cart', () => {
  // Arrange: Create a realistic scenario
  const cart = createCart();
  cart.addItem({ id: 1, name: 'Widget', price: 9.99 });
  const user = createUser({ hasPaymentMethod: true });

  // Act: Perform the action
  const result = checkout(cart, user);

  // Assert: Verify expected outcome
  expect(result.status).toBe('success');
  expect(result.orderId).toBeDefined();
});
```

## TDD Workflow with Todo System

```
When implementing a feature with TDD:

Feature: Shopping Cart

[ ] Write test cases list
[ ] RED: Test for adding first item
[ ] GREEN: Implement add item (simple)
[ ] REFACTOR: Clean up code
[ ] RED: Test for adding duplicate item
[ ] GREEN: Implement quantity increment
[ ] REFACTOR: Extract logic
[ ] RED: Test for removing item
[ ] GREEN: Implement remove item
[ ] REFACTOR: Final cleanup
[ ] Verify all tests pass
[ ] Check coverage report
```

## When to Use This Skill

- When starting any new feature or function
- When fixing bugs (write test that reproduces bug first)
- When refactoring (tests ensure behavior doesn't change)
- When code quality and reliability are critical
- When you want to move fast with confidence

## Example: TDD Session

```
Task: Implement a function to validate email addresses

TEST LIST:
- [ ] Valid email with standard format
- [ ] Invalid email without @
- [ ] Invalid email without domain
- [ ] Empty string
- [ ] Null/undefined input
- [ ] Email with special characters
- [ ] Email with subdomain

---

🔴 RED - Test 1:
```javascript
test('should return true for valid email', () => {
  expect(isValidEmail('user@example.com')).toBe(true);
});
// ❌ FAILS: isValidEmail is not defined
```

🟢 GREEN - Minimal implementation:
```javascript
function isValidEmail(email) {
  return email === 'user@example.com'; // Hardcoded!
}
// ✅ PASSES
```

🔴 RED - Test 2:
```javascript
test('should return false for email without @', () => {
  expect(isValidEmail('userexample.com')).toBe(false);
});
// ❌ FAILS: Returns true (because of hardcode)
```

🟢 GREEN - Real implementation:
```javascript
function isValidEmail(email) {
  return email.includes('@');
}
// ✅ PASSES both tests
```

🔴 RED - Test 3:
```javascript
test('should return false for empty string', () => {
  expect(isValidEmail('')).toBe(false);
});
// ❌ FAILS
```

🟢 GREEN - Improved:
```javascript
function isValidEmail(email) {
  return email && email.includes('@') && email.includes('.');
}
// ✅ PASSES all tests
```

🔵 REFACTOR - Polish:
```javascript
function isValidEmail(email) {
  if (!email) return false;
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}
// ✅ PASSES all tests, cleaner implementation
```

Continue with remaining test cases...
```

---

**Remember**: TDD feels slower at first, but pays dividends in fewer bugs, better design, and faster debugging. Trust the process!
