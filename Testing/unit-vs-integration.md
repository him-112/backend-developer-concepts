# Unit vs Integration Testing - Interview Guide 🧪

## 🎯 What You'll Learn
Master the differences between unit and integration testing to build reliable software!

## 🌟 Testing Fundamentals

### Testing Pyramid Overview
Think of testing like **quality control in manufacturing**:
- **Unit Tests** = Component inspection (individual parts)
- **Integration Tests** = Assembly testing (parts working together)
- **E2E Tests** = Final product testing (complete user journey)
- **Manual Tests** = Human quality assurance

Test early, test often, test at the right level!

### Key Interview Points ⭐
1. "Unit tests are fast, isolated, and test single functions"
2. "Integration tests verify components work together"
3. "Follow the testing pyramid: more unit, fewer integration, minimal E2E"
4. "Different testing strategies for different scenarios"
5. "Test behavior, not implementation details"

## 🔬 Unit Testing

### What are Unit Tests?
*Test individual functions or methods in isolation*

```javascript
// Example function to test
class Calculator {
  add(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
      throw new Error('Arguments must be numbers');
    }
    return a + b;
  }
  
  divide(a, b) {
    if (b === 0) {
      throw new Error('Cannot divide by zero');
    }
    return a / b;
  }
}

// Unit tests with Jest
describe('Calculator', () => {
  let calculator;
  
  beforeEach(() => {
    calculator = new Calculator();
  });
  
  describe('add method', () => {
    it('should add two positive numbers correctly', () => {
      const result = calculator.add(2, 3);
      expect(result).toBe(5);
    });
    
    it('should add negative numbers correctly', () => {
      const result = calculator.add(-2, -3);
      expect(result).toBe(-5);
    });
    
    it('should throw error for non-numeric inputs', () => {
      expect(() => calculator.add('2', 3)).toThrow('Arguments must be numbers');
      expect(() => calculator.add(2, null)).toThrow('Arguments must be numbers');
    });
  });
  
  describe('divide method', () => {
    it('should divide numbers correctly', () => {
      const result = calculator.divide(10, 2);
      expect(result).toBe(5);
    });
    
    it('should throw error when dividing by zero', () => {
      expect(() => calculator.divide(10, 0)).toThrow('Cannot divide by zero');
    });
  });
});
```

### Unit Testing Best Practices

```javascript
// ✅ Good unit test example
describe('UserService', () => {
  let userService;
  let mockDatabase;
  let mockEmailService;
  
  beforeEach(() => {
    // Create mocks for dependencies
    mockDatabase = {
      findUser: jest.fn(),
      createUser: jest.fn(),
      updateUser: jest.fn()
    };
    
    mockEmailService = {
      sendWelcomeEmail: jest.fn()
    };
    
    userService = new UserService(mockDatabase, mockEmailService);
  });
  
  describe('createUser', () => {
    it('should create user and send welcome email when valid data provided', async () => {
      // Arrange
      const userData = { email: 'test@example.com', name: 'John Doe' };
      const createdUser = { id: 1, ...userData };
      
      mockDatabase.findUser.mockResolvedValue(null); // User doesn't exist
      mockDatabase.createUser.mockResolvedValue(createdUser);
      mockEmailService.sendWelcomeEmail.mockResolvedValue(true);
      
      // Act
      const result = await userService.createUser(userData);
      
      // Assert
      expect(result).toEqual(createdUser);
      expect(mockDatabase.findUser).toHaveBeenCalledWith(userData.email);
      expect(mockDatabase.createUser).toHaveBeenCalledWith(userData);
      expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledWith(userData.email);
    });
    
    it('should throw error when user already exists', async () => {
      // Arrange
      const userData = { email: 'test@example.com', name: 'John Doe' };
      const existingUser = { id: 1, ...userData };
      
      mockDatabase.findUser.mockResolvedValue(existingUser);
      
      // Act & Assert
      await expect(userService.createUser(userData))
        .rejects.toThrow('User already exists');
      
      expect(mockDatabase.createUser).not.toHaveBeenCalled();
      expect(mockEmailService.sendWelcomeEmail).not.toHaveBeenCalled();
    });
  });
});

// ❌ Bad unit test example
describe('UserService Bad Example', () => {
  it('should work', async () => {
    // Testing multiple things at once
    // Using real database
    // Not descriptive test name
    const userService = new UserService(realDatabase, realEmailService);
    const result = await userService.createUser({ email: 'test@example.com' });
    expect(result).toBeTruthy(); // Vague assertion
  });
});
```

### Mocking and Stubbing

```javascript
// Different types of test doubles
class TestDoubles {
  // Dummy: Objects passed around but never used
  static createDummyUser() {
    return { id: 1, name: 'dummy', email: 'dummy@example.com' };
  }
  
  // Stub: Provides predefined responses
  static createUserStub() {
    return {
      findUser: () => Promise.resolve({ id: 1, name: 'John' }),
      createUser: () => Promise.resolve({ id: 2, name: 'Jane' })
    };
  }
  
  // Mock: Verifies interactions and can provide responses
  static createUserMock() {
    return {
      findUser: jest.fn().mockResolvedValue({ id: 1, name: 'John' }),
      createUser: jest.fn().mockResolvedValue({ id: 2, name: 'Jane' }),
      deleteUser: jest.fn().mockResolvedValue(true)
    };
  }
  
  // Spy: Wrapper around real object to observe interactions
  static createUserSpy(realUserService) {
    return {
      findUser: jest.spyOn(realUserService, 'findUser'),
      createUser: jest.spyOn(realUserService, 'createUser')
    };
  }
}

// Advanced mocking patterns
describe('Advanced Mocking', () => {
  it('should mock external dependencies', async () => {
    // Mock external HTTP calls
    const mockFetch = jest.fn();
    global.fetch = mockFetch;
    
    mockFetch.mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve({ data: 'test' })
    });
    
    const apiService = new APIService();
    const result = await apiService.fetchData('/api/users');
    
    expect(mockFetch).toHaveBeenCalledWith('/api/users');
    expect(result).toEqual({ data: 'test' });
  });
  
  it('should mock different responses for multiple calls', () => {
    const mockRandom = jest.spyOn(Math, 'random');
    
    mockRandom
      .mockReturnValueOnce(0.1) // First call
      .mockReturnValueOnce(0.5) // Second call
      .mockReturnValueOnce(0.9); // Third call
    
    const service = new RandomService();
    
    expect(service.getRandomValue()).toBe(0.1);
    expect(service.getRandomValue()).toBe(0.5);
    expect(service.getRandomValue()).toBe(0.9);
    
    mockRandom.mockRestore();
  });
});
```

## 🔗 Integration Testing

### What are Integration Tests?
*Test how multiple components work together*

```javascript
// Integration test example - testing API endpoints
const request = require('supertest');
const app = require('../app');

describe('User API Integration Tests', () => {
  beforeEach(async () => {
    // Set up test database
    await setupTestDatabase();
  });
  
  afterEach(async () => {
    // Clean up test database
    await cleanupTestDatabase();
  });
  
  describe('POST /api/users', () => {
    it('should create user and return 201 status', async () => {
      const userData = {
        email: 'test@example.com',
        name: 'John Doe',
        password: 'securePassword123'
      };
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);
      
      expect(response.body).toMatchObject({
        id: expect.any(Number),
        email: userData.email,
        name: userData.name
      });
      
      // Verify user was actually created in database
      const createdUser = await db.users.findByEmail(userData.email);
      expect(createdUser).toBeTruthy();
      expect(createdUser.email).toBe(userData.email);
    });
    
    it('should return 400 for invalid email format', async () => {
      const userData = {
        email: 'invalid-email',
        name: 'John Doe',
        password: 'securePassword123'
      };
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(400);
      
      expect(response.body).toMatchObject({
        error: 'Invalid email format'
      });
    });
    
    it('should return 409 when user already exists', async () => {
      const userData = {
        email: 'existing@example.com',
        name: 'John Doe',
        password: 'securePassword123'
      };
      
      // Create user first
      await db.users.create(userData);
      
      // Try to create same user again
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(409);
      
      expect(response.body).toMatchObject({
        error: 'User already exists'
      });
    });
  });
  
  describe('GET /api/users/:id', () => {
    it('should return user when valid ID provided', async () => {
      const user = await db.users.create({
        email: 'test@example.com',
        name: 'John Doe'
      });
      
      const response = await request(app)
        .get(`/api/users/${user.id}`)
        .expect(200);
      
      expect(response.body).toMatchObject({
        id: user.id,
        email: user.email,
        name: user.name
      });
    });
    
    it('should return 404 when user not found', async () => {
      const response = await request(app)
        .get('/api/users/99999')
        .expect(404);
      
      expect(response.body).toMatchObject({
        error: 'User not found'
      });
    });
  });
});

// Database integration tests
describe('Database Integration Tests', () => {
  describe('UserRepository', () => {
    let userRepository;
    
    beforeAll(async () => {
      await setupTestDatabase();
      userRepository = new UserRepository(db);
    });
    
    afterAll(async () => {
      await cleanupTestDatabase();
    });
    
    beforeEach(async () => {
      await clearUsers();
    });
    
    it('should create and retrieve user', async () => {
      const userData = {
        email: 'test@example.com',
        name: 'John Doe'
      };
      
      const createdUser = await userRepository.create(userData);
      expect(createdUser.id).toBeDefined();
      
      const retrievedUser = await userRepository.findById(createdUser.id);
      expect(retrievedUser).toMatchObject(userData);
    });
    
    it('should update user information', async () => {
      const user = await userRepository.create({
        email: 'test@example.com',
        name: 'John Doe'
      });
      
      const updatedData = { name: 'Jane Doe' };
      const updatedUser = await userRepository.update(user.id, updatedData);
      
      expect(updatedUser.name).toBe('Jane Doe');
      expect(updatedUser.email).toBe('test@example.com'); // Unchanged
    });
    
    it('should handle concurrent user creation', async () => {
      const userData = {
        email: 'test@example.com',
        name: 'John Doe'
      };
      
      // Try to create same user concurrently
      const promises = [
        userRepository.create(userData),
        userRepository.create(userData)
      ];
      
      const results = await Promise.allSettled(promises);
      
      // One should succeed, one should fail
      const successful = results.filter(r => r.status === 'fulfilled');
      const failed = results.filter(r => r.status === 'rejected');
      
      expect(successful).toHaveLength(1);
      expect(failed).toHaveLength(1);
    });
  });
});
```

### Service Integration Tests

```javascript
// Testing service interactions
describe('Order Service Integration Tests', () => {
  let orderService;
  let userService;
  let inventoryService;
  let paymentService;
  
  beforeEach(() => {
    // Use real services but with test databases
    userService = new UserService(testDb);
    inventoryService = new InventoryService(testDb);
    paymentService = new PaymentService(testPaymentProvider);
    orderService = new OrderService(userService, inventoryService, paymentService);
  });
  
  it('should process complete order workflow', async () => {
    // Create test user
    const user = await userService.create({
      email: 'test@example.com',
      name: 'John Doe'
    });
    
    // Add inventory
    const product = await inventoryService.addProduct({
      name: 'Test Product',
      price: 100,
      quantity: 10
    });
    
    // Create order
    const orderData = {
      userId: user.id,
      items: [{
        productId: product.id,
        quantity: 2
      }]
    };
    
    const order = await orderService.createOrder(orderData);
    
    // Verify order was created
    expect(order.id).toBeDefined();
    expect(order.status).toBe('pending');
    expect(order.total).toBe(200);
    
    // Verify inventory was reserved
    const updatedProduct = await inventoryService.getProduct(product.id);
    expect(updatedProduct.reservedQuantity).toBe(2);
    
    // Process payment
    await orderService.processPayment(order.id, {
      cardNumber: '4111111111111111',
      expiryDate: '12/25',
      cvv: '123'
    });
    
    // Verify order status updated
    const paidOrder = await orderService.getOrder(order.id);
    expect(paidOrder.status).toBe('paid');
    
    // Verify inventory was decremented
    const finalProduct = await inventoryService.getProduct(product.id);
    expect(finalProduct.quantity).toBe(8);
    expect(finalProduct.reservedQuantity).toBe(0);
  });
  
  it('should handle payment failure gracefully', async () => {
    const user = await userService.create({
      email: 'test@example.com',
      name: 'John Doe'
    });
    
    const product = await inventoryService.addProduct({
      name: 'Test Product',
      price: 100,
      quantity: 10
    });
    
    const order = await orderService.createOrder({
      userId: user.id,
      items: [{ productId: product.id, quantity: 2 }]
    });
    
    // Simulate payment failure
    await expect(orderService.processPayment(order.id, {
      cardNumber: '4000000000000002', // Declined card
      expiryDate: '12/25',
      cvv: '123'
    })).rejects.toThrow('Payment declined');
    
    // Verify order status
    const failedOrder = await orderService.getOrder(order.id);
    expect(failedOrder.status).toBe('payment_failed');
    
    // Verify inventory was released
    const product2 = await inventoryService.getProduct(product.id);
    expect(product2.quantity).toBe(10);
    expect(product2.reservedQuantity).toBe(0);
  });
});
```

## 📊 Testing Strategies

### Testing Pyramid Implementation

```javascript
// Test configuration for different levels
module.exports = {
  // Jest configuration
  testMatch: [
    '**/tests/unit/**/*.test.js',      // Unit tests
    '**/tests/integration/**/*.test.js', // Integration tests
    '**/tests/e2e/**/*.test.js'        // E2E tests
  ],
  
  // Different test environments
  projects: [
    {
      displayName: 'unit',
      testMatch: ['**/tests/unit/**/*.test.js'],
      testEnvironment: 'node',
      setupFilesAfterEnv: ['<rootDir>/tests/setup/unit.js']
    },
    {
      displayName: 'integration',
      testMatch: ['**/tests/integration/**/*.test.js'],
      testEnvironment: 'node',
      setupFilesAfterEnv: ['<rootDir>/tests/setup/integration.js'],
      globalSetup: '<rootDir>/tests/setup/database.js'
    }
  ],
  
  // Coverage thresholds
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/services/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90
    }
  }
};

// Test utilities
class TestHelpers {
  // Factory for creating test data
  static createUser(overrides = {}) {
    return {
      email: 'test@example.com',
      name: 'John Doe',
      age: 30,
      ...overrides
    };
  }
  
  static createOrder(overrides = {}) {
    return {
      userId: 1,
      items: [{ productId: 1, quantity: 2 }],
      total: 100,
      ...overrides
    };
  }
  
  // Async test utilities
  static async waitFor(condition, timeout = 5000) {
    const start = Date.now();
    
    while (Date.now() - start < timeout) {
      if (await condition()) {
        return true;
      }
      await new Promise(resolve => setTimeout(resolve, 100));
    }
    
    throw new Error('Condition not met within timeout');
  }
  
  // Database utilities
  static async cleanDatabase() {
    await db.users.deleteMany({});
    await db.orders.deleteMany({});
    await db.products.deleteMany({});
  }
  
  static async seedDatabase() {
    const users = await db.users.insertMany([
      TestHelpers.createUser({ email: 'user1@example.com' }),
      TestHelpers.createUser({ email: 'user2@example.com' })
    ]);
    
    const products = await db.products.insertMany([
      { name: 'Product 1', price: 100, quantity: 10 },
      { name: 'Product 2', price: 200, quantity: 5 }
    ]);
    
    return { users, products };
  }
}
```

## 💡 Common Interview Questions & Answers

### Q1: "What's the difference between unit and integration tests?"
**Answer:**
"Key differences:
- **Unit Tests**: Test individual components in isolation, fast, use mocks/stubs
- **Integration Tests**: Test component interactions, slower, use real dependencies
- **Scope**: Unit tests focus on single function, integration tests on workflows
- **Purpose**: Unit tests catch logic bugs, integration tests catch interface bugs
- **Speed**: Unit tests run in milliseconds, integration tests in seconds"

### Q2: "When would you use mocks vs real dependencies?"
**Answer:**
"Use mocks for:
- **Unit tests** - To isolate the component being tested
- **External services** - APIs, payment gateways, email services
- **Slow operations** - Database calls, file I/O
- **Non-deterministic behavior** - Random values, current time

Use real dependencies for:
- **Integration tests** - To test actual interactions
- **Simple utilities** - Math functions, validators
- **Critical paths** - Core business logic workflows"

### Q3: "How do you structure your test files?"
**Answer:**
"Organized test structure:
```
tests/
├── unit/
│   ├── services/
│   ├── utils/
│   └── models/
├── integration/
│   ├── api/
│   ├── database/
│   └── services/
├── e2e/
├── setup/
└── fixtures/
```
- Group by test type first, then by feature
- Mirror source code structure
- Shared setup and test data in separate files"

### Q4: "What's the ideal test coverage percentage?"
**Answer:**
"Coverage guidelines:
- **80-90%** is a good target for most projects
- **100%** coverage doesn't guarantee bug-free code
- **Focus on critical paths** first (payment, auth, data loss scenarios)
- **Quality over quantity** - Better to have fewer, well-written tests
- **Different thresholds** for different components (core services higher)"

## 🚀 Best Practices

### Test Organization

```javascript
// Good test structure
describe('UserService', () => {
  // Group related tests
  describe('user creation', () => {
    describe('when valid data provided', () => {
      it('should create user successfully');
      it('should send welcome email');
      it('should return user without password');
    });
    
    describe('when invalid data provided', () => {
      it('should reject invalid email format');
      it('should reject weak passwords');
      it('should reject missing required fields');
    });
  });
  
  describe('user authentication', () => {
    // More grouped tests...
  });
});

// Test naming convention
describe('Component/Service Name', () => {
  describe('method/feature being tested', () => {
    describe('context/condition', () => {
      it('should expected behavior', () => {
        // Arrange
        // Act  
        // Assert
      });
    });
  });
});
```

## 🔧 Quick Checklist

- [ ] Follow the testing pyramid (more unit, fewer integration)
- [ ] Use descriptive test names that explain behavior
- [ ] Test edge cases and error conditions
- [ ] Mock external dependencies in unit tests
- [ ] Use real dependencies in integration tests
- [ ] Maintain good test coverage on critical paths
- [ ] Keep tests fast and independent

## 🎉 You're Testing Ready!

**Key Message**: Good testing strategy uses **unit tests for speed and coverage**, **integration tests for confidence**!

**Interview Golden Rule**: Always explain the trade-offs between test types and when to use each approach.

Perfect! Now you can confidently discuss testing strategies! 🧪
