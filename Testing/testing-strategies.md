# Testing Strategies

## 🔍 Overview
Testing is crucial for reliable backend systems. This covers testing types, strategies, and best practices for backend development.

## 🧪 Types of Testing

### 1. Unit Testing
**Purpose**: Test individual functions/methods in isolation
**Scope**: Smallest testable parts of an application

```javascript
// Example function to test
function calculateTax(price, taxRate) {
  if (price < 0 || taxRate < 0) {
    throw new Error('Price and tax rate must be positive');
  }
  return price * taxRate;
}

// Unit tests
const assert = require('assert');

describe('calculateTax', () => {
  it('should calculate tax correctly', () => {
    const result = calculateTax(100, 0.1);
    assert.strictEqual(result, 10);
  });
  
  it('should throw error for negative price', () => {
    assert.throws(() => {
      calculateTax(-100, 0.1);
    }, Error);
  });
  
  it('should throw error for negative tax rate', () => {
    assert.throws(() => {
      calculateTax(100, -0.1);
    }, Error);
  });
});
```

### 2. Integration Testing
**Purpose**: Test interaction between components
**Scope**: Multiple units working together

```javascript
// Database integration test
const request = require('supertest');
const app = require('../app');

describe('User API Integration', () => {
  beforeEach(async () => {
    await db.migrate.latest();
    await db.seed.run();
  });
  
  afterEach(async () => {
    await db.migrate.rollback();
  });
  
  it('should create user and return 201', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com'
    };
    
    const response = await request(app)
      .post('/api/users')
      .send(userData)
      .expect(201);
      
    assert.strictEqual(response.body.name, userData.name);
    
    // Verify in database
    const user = await User.findOne({ email: userData.email });
    assert.ok(user);
  });
});
```

### 3. End-to-End Testing
**Purpose**: Test complete user workflows
**Scope**: Entire application flow

```javascript
// E2E API test
describe('Order Workflow E2E', () => {
  let authToken;
  let userId;
  
  before(async () => {
    // Setup test user
    const user = await request(app)
      .post('/auth/register')
      .send({
        email: 'test@example.com',
        password: 'password123'
      });
    
    userId = user.body.id;
    
    // Login to get token
    const loginResponse = await request(app)
      .post('/auth/login')
      .send({
        email: 'test@example.com',
        password: 'password123'
      });
    
    authToken = loginResponse.body.token;
  });
  
  it('should complete full order workflow', async () => {
    // 1. Create order
    const orderResponse = await request(app)
      .post('/api/orders')
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        items: [{ productId: 1, quantity: 2 }]
      })
      .expect(201);
    
    const orderId = orderResponse.body.id;
    
    // 2. Process payment
    await request(app)
      .post(`/api/orders/${orderId}/payment`)
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        paymentMethod: 'card',
        amount: 100
      })
      .expect(200);
    
    // 3. Verify order status
    const finalOrder = await request(app)
      .get(`/api/orders/${orderId}`)
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);
    
    assert.strictEqual(finalOrder.body.status, 'paid');
  });
});
```

## 🎯 Testing Patterns

### Test Doubles (Mocks, Stubs, Spies)

```javascript
// Mocking external dependencies
const sinon = require('sinon');

describe('EmailService', () => {
  let emailProvider;
  let emailService;
  
  beforeEach(() => {
    emailProvider = {
      send: sinon.stub()
    };
    emailService = new EmailService(emailProvider);
  });
  
  it('should send welcome email', async () => {
    emailProvider.send.resolves({ success: true, id: '123' });
    
    const result = await emailService.sendWelcomeEmail('user@example.com');
    
    assert.ok(emailProvider.send.calledOnce);
    assert.ok(emailProvider.send.calledWith({
      to: 'user@example.com',
      subject: 'Welcome!',
      template: 'welcome'
    }));
    assert.strictEqual(result.success, true);
  });
  
  it('should handle email service failure', async () => {
    emailProvider.send.rejects(new Error('Service unavailable'));
    
    await assert.rejects(
      () => emailService.sendWelcomeEmail('user@example.com'),
      /Service unavailable/
    );
  });
});
```

### Test Data Management

```javascript
// Test data factories
class UserFactory {
  static create(overrides = {}) {
    return {
      id: Math.floor(Math.random() * 1000),
      name: 'Test User',
      email: 'test@example.com',
      createdAt: new Date(),
      ...overrides
    };
  }
  
  static createMany(count, overrides = {}) {
    return Array.from({ length: count }, (_, i) => 
      this.create({ ...overrides, email: `test${i}@example.com` })
    );
  }
}

// Usage in tests
describe('UserService', () => {
  it('should filter active users', () => {
    const users = [
      UserFactory.create({ status: 'active' }),
      UserFactory.create({ status: 'inactive' }),
      UserFactory.create({ status: 'active' })
    ];
    
    const activeUsers = userService.filterActiveUsers(users);
    assert.strictEqual(activeUsers.length, 2);
  });
});
```

## 🔧 Testing Best Practices

### 1. AAA Pattern (Arrange, Act, Assert)
```javascript
describe('OrderService', () => {
  it('should calculate total with tax', () => {
    // Arrange
    const order = {
      items: [
        { price: 100, quantity: 2 },
        { price: 50, quantity: 1 }
      ]
    };
    const taxRate = 0.1;
    
    // Act
    const total = orderService.calculateTotal(order, taxRate);
    
    // Assert
    assert.strictEqual(total, 275); // (100*2 + 50*1) * 1.1
  });
});
```

### 2. Test Isolation
```javascript
describe('UserRepository', () => {
  beforeEach(async () => {
    // Each test starts with clean state
    await db.raw('TRUNCATE TABLE users CASCADE');
  });
  
  it('should save user', async () => {
    const user = await userRepository.save({
      name: 'John',
      email: 'john@example.com'
    });
    
    assert.ok(user.id);
  });
  
  it('should find user by email', async () => {
    // This test is isolated from the previous one
    await userRepository.save({
      name: 'Jane',
      email: 'jane@example.com'
    });
    
    const user = await userRepository.findByEmail('jane@example.com');
    assert.strictEqual(user.name, 'Jane');
  });
});
```

### 3. Error Case Testing
```javascript
describe('PaymentService', () => {
  it('should handle payment failures gracefully', async () => {
    const paymentProvider = {
      charge: sinon.stub().rejects(new Error('Card declined'))
    };
    
    const paymentService = new PaymentService(paymentProvider);
    
    const result = await paymentService.processPayment({
      amount: 100,
      cardToken: 'invalid_token'
    });
    
    assert.strictEqual(result.success, false);
    assert.strictEqual(result.error, 'Payment failed: Card declined');
  });
});
```

## 📊 Test Coverage

### Measuring Coverage
```javascript
// package.json
{
  "scripts": {
    "test": "mocha",
    "test:coverage": "nyc mocha",
    "test:coverage:report": "nyc report --reporter=html"
  },
  "nyc": {
    "include": ["src/**/*.js"],
    "exclude": ["src/**/*.test.js"],
    "reporter": ["text", "html"],
    "check-coverage": true,
    "lines": 80,
    "functions": 80,
    "branches": 80,
    "statements": 80
  }
}
```

### Coverage Goals
- **Lines**: 80-90% (percentage of code lines executed)
- **Functions**: 90%+ (percentage of functions called)
- **Branches**: 70-80% (percentage of code branches taken)
- **Statements**: 80-90% (percentage of statements executed)

## 🚀 Advanced Testing Concepts

### Contract Testing
```javascript
// API contract test
const Pact = require('@pact-foundation/pact');

describe('User Service Contract', () => {
  const provider = new Pact({
    consumer: 'UserAPI',
    provider: 'UserService',
    port: 1234
  });
  
  before(() => provider.setup());
  after(() => provider.finalize());
  
  it('should get user by ID', async () => {
    await provider.addInteraction({
      state: 'user with ID 1 exists',
      uponReceiving: 'a request for user 1',
      withRequest: {
        method: 'GET',
        path: '/users/1'
      },
      willRespondWith: {
        status: 200,
        headers: { 'Content-Type': 'application/json' },
        body: {
          id: 1,
          name: 'John Doe',
          email: 'john@example.com'
        }
      }
    });
    
    const response = await userClient.getUser(1);
    assert.strictEqual(response.name, 'John Doe');
  });
});
```

### Performance Testing
```javascript
// Load testing with artillery
const { performance } = require('perf_hooks');

describe('Performance Tests', () => {
  it('should handle 100 concurrent requests', async () => {
    const startTime = performance.now();
    
    const promises = Array.from({ length: 100 }, () =>
      request(app).get('/api/users/1').expect(200)
    );
    
    await Promise.all(promises);
    
    const endTime = performance.now();
    const duration = endTime - startTime;
    
    // Should complete within 5 seconds
    assert.ok(duration < 5000, `Took ${duration}ms, expected < 5000ms`);
  });
});
```

## ❓ Interview Questions

1. **Q: Difference between unit and integration tests?**
   A: Unit tests test individual components in isolation. Integration tests test components working together.

2. **Q: What are test doubles and when to use them?**
   A: Mocks, stubs, spies replace dependencies. Use when external services are slow, expensive, or unreliable.

3. **Q: How do you test async code?**
   A: Use async/await in tests, mock promises, test both success and error cases.

4. **Q: What is TDD?**
   A: Test-Driven Development - write tests first, then code to pass tests. Red-Green-Refactor cycle.

5. **Q: How do you test database interactions?**
   A: Use test database, transactions for isolation, factories for test data, mock for unit tests.

## 💡 Testing Best Practices

### Test Organization
- Group related tests in describe blocks
- Use descriptive test names
- Keep tests simple and focused
- Test one thing at a time

### Test Maintenance
- Update tests when code changes
- Remove redundant tests
- Keep test data minimal
- Use factories for complex objects

### CI/CD Integration
- Run tests on every commit
- Fail builds on test failures
- Generate coverage reports
- Run different test types in parallel

---

**Key Takeaway**: Good tests give confidence to refactor and deploy. Write tests that are fast, reliable, and maintainable. Focus on testing behavior, not implementation. 