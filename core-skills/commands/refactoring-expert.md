# Refactoring Expert

**Description**: Systematically improve code structure and quality while preserving behavior

## Core Philosophy

Refactoring is the disciplined technique of restructuring existing code without changing its external behavior. You improve code quality incrementally, safely, and confidently.

## When to Refactor

### 🟢 **Good Times to Refactor**
- Before adding new features (clean up the area first)
- When you see code smells
- During code review (both as author and reviewer)
- When you touch old code
- When tests are all passing

### 🔴 **Bad Times to Refactor**
- When tests are failing
- While debugging (fix first, refactor later)
- Under tight deadline pressure (unless refactoring speeds you up)
- When you don't have tests

## The Golden Rule

**🔐 Preserve Behavior**: Refactoring must not change what the code does - only how it does it.

### Safety Protocol
```
1. ✅ Ensure tests pass before refactoring
2. 🔄 Make small, incremental changes
3. ✅ Run tests after each change
4. 💾 Commit after each successful refactor
5. 🔄 Repeat
```

## Common Code Smells

### 1. **Duplicated Code**
```javascript
// ❌ Bad: Duplicated logic
function calculateOrderTotal(order) {
  let total = 0;
  for (const item of order.items) {
    total += item.price * item.quantity;
  }
  total = total * 1.1; // Tax
  return total;
}

function calculateCartTotal(cart) {
  let total = 0;
  for (const item of cart.items) {
    total += item.price * item.quantity;
  }
  total = total * 1.1; // Tax
  return total;
}

// ✅ Good: Extracted common logic
function calculateItemsSubtotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

function applyTax(amount) {
  return amount * 1.1;
}

function calculateOrderTotal(order) {
  return applyTax(calculateItemsSubtotal(order.items));
}

function calculateCartTotal(cart) {
  return applyTax(calculateItemsSubtotal(cart.items));
}
```

### 2. **Long Methods**
```javascript
// ❌ Bad: 100-line method doing everything
function processUserRegistration(userData) {
  // Validate
  if (!userData.email) throw new Error('Email required');
  if (!userData.email.includes('@')) throw new Error('Invalid email');
  if (!userData.password) throw new Error('Password required');
  if (userData.password.length < 8) throw new Error('Password too short');

  // Hash password
  const salt = generateSalt();
  const hash = hashPassword(userData.password, salt);

  // Create user
  const user = {
    id: generateId(),
    email: userData.email.toLowerCase(),
    passwordHash: hash,
    createdAt: new Date()
  };

  // Save to database
  database.users.insert(user);

  // Send welcome email
  emailService.send({
    to: user.email,
    template: 'welcome',
    data: { name: userData.name }
  });

  // Log analytics
  analytics.track('user_registered', { userId: user.id });

  return user;
}

// ✅ Good: Broken into focused functions
function processUserRegistration(userData) {
  const validatedData = validateUserData(userData);
  const user = createUser(validatedData);
  await saveUser(user);
  await sendWelcomeEmail(user);
  trackRegistration(user);
  return user;
}

function validateUserData(userData) {
  validateEmail(userData.email);
  validatePassword(userData.password);
  return userData;
}

function createUser(userData) {
  return {
    id: generateId(),
    email: userData.email.toLowerCase(),
    passwordHash: hashUserPassword(userData.password),
    createdAt: new Date()
  };
}

// ... other focused functions
```

### 3. **Large Classes (God Objects)**
```typescript
// ❌ Bad: Class doing too much
class UserManager {
  createUser() { }
  deleteUser() { }
  validateEmail() { }
  hashPassword() { }
  sendEmail() { }
  logAnalytics() { }
  generateReport() { }
  exportToCSV() { }
}

// ✅ Good: Single Responsibility Principle
class UserService {
  createUser() { }
  deleteUser() { }
}

class UserValidator {
  validateEmail() { }
  validatePassword() { }
}

class AuthService {
  hashPassword() { }
  verifyPassword() { }
}

class EmailService {
  sendEmail() { }
}

class AnalyticsService {
  logEvent() { }
}

class ReportService {
  generateReport() { }
  exportToCSV() { }
}
```

### 4. **Long Parameter Lists**
```javascript
// ❌ Bad: Too many parameters
function createUser(
  email,
  password,
  firstName,
  lastName,
  age,
  country,
  city,
  zipCode,
  preferences
) {
  // ...
}

// ✅ Good: Use objects for related data
function createUser(userProfile, userAddress, userPreferences) {
  // ...
}

// Or even better: Single object
function createUser(userData) {
  const { profile, address, preferences } = userData;
  // ...
}
```

### 5. **Primitive Obsession**
```typescript
// ❌ Bad: Using primitives for domain concepts
function processPayment(amount: number, currency: string, userId: string) {
  // What if currency is 'abc'? What if userId is not a valid ID?
}

// ✅ Good: Use value objects
class Money {
  constructor(
    readonly amount: number,
    readonly currency: Currency
  ) {}
}

class UserId {
  constructor(readonly value: string) {
    if (!this.isValid(value)) throw new Error('Invalid user ID');
  }
  private isValid(value: string): boolean {
    return /^[0-9a-f]{24}$/.test(value);
  }
}

function processPayment(amount: Money, userId: UserId) {
  // Type-safe and validated
}
```

### 6. **Deep Nesting**
```javascript
// ❌ Bad: Nested conditionals
function processOrder(order) {
  if (order) {
    if (order.items && order.items.length > 0) {
      if (order.user) {
        if (order.user.paymentMethod) {
          // Process order
        } else {
          throw new Error('No payment method');
        }
      } else {
        throw new Error('No user');
      }
    } else {
      throw new Error('Empty order');
    }
  } else {
    throw new Error('No order');
  }
}

// ✅ Good: Guard clauses
function processOrder(order) {
  if (!order) throw new Error('No order');
  if (!order.items || order.items.length === 0) throw new Error('Empty order');
  if (!order.user) throw new Error('No user');
  if (!order.user.paymentMethod) throw new Error('No payment method');

  // Process order (now at top level)
  return processPayment(order);
}
```

### 7. **Magic Numbers/Strings**
```javascript
// ❌ Bad
if (user.status === 2) {
  setTimeout(retry, 3000);
}

// ✅ Good
const UserStatus = {
  ACTIVE: 1,
  SUSPENDED: 2,
  DELETED: 3
};
const RETRY_DELAY_MS = 3000;

if (user.status === UserStatus.SUSPENDED) {
  setTimeout(retry, RETRY_DELAY_MS);
}
```

## Refactoring Techniques

### 1. **Extract Function**
```javascript
// Before
function printOwing(invoice) {
  let outstanding = 0;

  console.log('***********************');
  console.log('**** Customer Owes ****');
  console.log('***********************');

  for (const order of invoice.orders) {
    outstanding += order.amount;
  }

  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
}

// After
function printOwing(invoice) {
  printBanner();
  const outstanding = calculateOutstanding(invoice);
  printDetails(invoice, outstanding);
}

function printBanner() {
  console.log('***********************');
  console.log('**** Customer Owes ****');
  console.log('***********************');
}

function calculateOutstanding(invoice) {
  return invoice.orders.reduce((sum, order) => sum + order.amount, 0);
}

function printDetails(invoice, outstanding) {
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
}
```

### 2. **Rename Variable/Function**
```javascript
// Before
function calc(d) {
  return d * 0.1;
}

// After
function calculateTax(amount) {
  const TAX_RATE = 0.1;
  return amount * TAX_RATE;
}
```

### 3. **Replace Conditional with Polymorphism**
```javascript
// Before
function getSpeed(bird) {
  switch (bird.type) {
    case 'EuropeanSwallow':
      return getBaseSpeed(bird);
    case 'AfricanSwallow':
      return getBaseSpeed(bird) - getLoadFactor(bird);
    case 'NorwegianBlueParrot':
      return (bird.isNailed) ? 0 : getBaseSpeed(bird);
    default:
      throw new Error('Unknown bird');
  }
}

// After
class Bird {
  getSpeed() {
    throw new Error('Must implement getSpeed');
  }
}

class EuropeanSwallow extends Bird {
  getSpeed() {
    return this.getBaseSpeed();
  }
}

class AfricanSwallow extends Bird {
  getSpeed() {
    return this.getBaseSpeed() - this.getLoadFactor();
  }
}

class NorwegianBlueParrot extends Bird {
  getSpeed() {
    return this.isNailed ? 0 : this.getBaseSpeed();
  }
}
```

### 4. **Replace Temp with Query**
```javascript
// Before
function getPrice(quantity, itemPrice) {
  const basePrice = quantity * itemPrice;
  const discountFactor = 0.98;

  if (basePrice > 1000) {
    discountFactor = 0.95;
  }

  return basePrice * discountFactor;
}

// After
function getPrice(quantity, itemPrice) {
  return getBasePrice(quantity, itemPrice) * getDiscountFactor(quantity, itemPrice);
}

function getBasePrice(quantity, itemPrice) {
  return quantity * itemPrice;
}

function getDiscountFactor(quantity, itemPrice) {
  return getBasePrice(quantity, itemPrice) > 1000 ? 0.95 : 0.98;
}
```

### 5. **Introduce Parameter Object**
```javascript
// Before
function drawRectangle(x, y, width, height, color, borderWidth, borderColor) {
  // ...
}

// After
class Rectangle {
  constructor(position, dimensions, style) {
    this.position = position; // { x, y }
    this.dimensions = dimensions; // { width, height }
    this.style = style; // { color, borderWidth, borderColor }
  }
}

function drawRectangle(rectangle) {
  // ...
}
```

## Refactoring Workflow

### Small Steps Protocol
```
1. [ ] Identify code smell or improvement opportunity
2. [ ] Ensure all tests pass (baseline)
3. [ ] Make one small refactoring change
4. [ ] Run tests - verify still passing
5. [ ] Commit with descriptive message
6. [ ] Repeat
```

### Example Session
```
Feature: Refactor user authentication code

[ ] Review current implementation
[ ] Ensure all tests pass
[ ] Extract validation logic → separate function
[ ] Run tests ✓
[ ] Commit "Extract email validation"
[ ] Extract password hashing → separate function
[ ] Run tests ✓
[ ] Commit "Extract password hashing"
[ ] Replace conditional with strategy pattern
[ ] Run tests ✓
[ ] Commit "Introduce authentication strategy pattern"
[ ] Add missing tests for edge cases
[ ] Run tests ✓
[ ] Commit "Add tests for auth edge cases"
[ ] Final review and cleanup
```

## Refactoring Checklist

### Before Refactoring
- [ ] Do you have good test coverage?
- [ ] Are all tests currently passing?
- [ ] Have you committed recent changes?
- [ ] Do you understand the code's current behavior?
- [ ] Have you identified the specific smell/issue?

### During Refactoring
- [ ] Making small, incremental changes?
- [ ] Running tests after each change?
- [ ] Committing after each successful step?
- [ ] Preserving existing behavior?
- [ ] Improving code clarity/maintainability?

### After Refactoring
- [ ] Do all tests still pass?
- [ ] Is the code more readable?
- [ ] Is the code more maintainable?
- [ ] Have you removed dead code?
- [ ] Have you updated documentation if needed?
- [ ] Can you explain why the new version is better?

## Tools & Techniques

### Automated Refactoring (IDE Features)
- Rename symbol
- Extract function/method
- Extract variable
- Inline variable/function
- Move file/class
- Change signature

### Manual Refactoring (Requires Care)
- Restructure logic
- Change algorithms
- Redesign architecture
- Split/merge classes

### Refactoring with Confidence
```
High Test Coverage → Refactor Fearlessly
Low Test Coverage → Write Tests First, Then Refactor
No Tests → Add Tests, Then Refactor Carefully
```

## Refactoring Patterns

### The Strangler Fig Pattern
For large legacy systems:
```
1. Identify a piece of functionality to refactor
2. Create new implementation alongside old
3. Redirect calls to new implementation
4. Once stable, remove old implementation
5. Repeat for next piece

This allows incremental refactoring of large systems
```

### The Branch by Abstraction
```
1. Create abstraction interface
2. Implement interface with old code
3. Replace direct calls with abstraction calls
4. Create new implementation of interface
5. Switch to new implementation
6. Remove old implementation
```

## When to Stop Refactoring

### Diminishing Returns
- Code is clear and maintainable
- Tests are comprehensive and passing
- Further changes would be cosmetic
- You're gold-plating

### Scope Creep
- Refactoring unrelated areas
- Changing behavior (no longer refactoring!)
- Rewriting instead of refactoring

---

**Remember**: Refactoring is not rewriting. It's disciplined improvement in small, safe steps. Leave the codebase better than you found it - the Boy Scout Rule!
