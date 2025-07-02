# Dependency Injection - Design Patterns Guide 💉

## 🎯 What You'll Learn
Master dependency injection to write loosely coupled, testable, and maintainable code!

## 🌟 Dependency Injection Fundamentals

### What is Dependency Injection?
Think of dependency injection like a **restaurant kitchen**:
- **Chef** = Class that needs dependencies
- **Ingredients** = Dependencies (services, data)
- **Kitchen Manager** = DI Container
- **Recipe** = Configuration

Instead of the chef going to get ingredients, the kitchen manager provides everything the chef needs!

### Key Interview Points ⭐
1. "Inversion of Control principle"
2. "Dependencies are provided from outside"
3. "Promotes loose coupling and testability"
4. "Three main types: Constructor, Setter, Interface"
5. "Essential for clean architecture"

## 🏗️ Types of Dependency Injection

### 1. **Constructor Injection**
*Dependencies provided through constructor*

```javascript
// ❌ Bad: Hard-coded dependencies
class OrderService {
  constructor() {
    this.emailService = new EmailService(); // Tight coupling
    this.database = new PostgresDatabase(); // Hard to test
  }
  
  async processOrder(order) {
    await this.database.save(order);
    await this.emailService.sendConfirmation(order.email);
  }
}

// ✅ Good: Dependencies injected
class OrderService {
  constructor(emailService, database) {
    this.emailService = emailService;
    this.database = database;
  }
  
  async processOrder(order) {
    await this.database.save(order);
    await this.emailService.sendConfirmation(order.email);
  }
}

// Usage
const emailService = new EmailService();
const database = new PostgresDatabase();
const orderService = new OrderService(emailService, database);
```

### 2. **Setter Injection**
*Dependencies provided through setter methods*

```javascript
class PaymentService {
  setPaymentGateway(gateway) {
    this.paymentGateway = gateway;
  }
  
  setLogger(logger) {
    this.logger = logger;
  }
  
  async processPayment(amount, card) {
    this.logger?.info(`Processing payment: $${amount}`);
    return await this.paymentGateway.charge(amount, card);
  }
}

// Usage
const paymentService = new PaymentService();
paymentService.setPaymentGateway(new StripeGateway());
paymentService.setLogger(new ConsoleLogger());
```

### 3. **Interface Injection**
*Dependencies provided through interface methods*

```javascript
// Define interfaces
class DatabaseInterface {
  async save(data) {
    throw new Error('Must implement save method');
  }
  
  async findById(id) {
    throw new Error('Must implement findById method');
  }
}

class EmailInterface {
  async send(to, subject, body) {
    throw new Error('Must implement send method');
  }
}

// Implementations
class MongoDatabase extends DatabaseInterface {
  async save(data) {
    // MongoDB implementation
    console.log('Saving to MongoDB:', data);
  }
  
  async findById(id) {
    console.log('Finding in MongoDB:', id);
    return { id, data: 'sample' };
  }
}

class GmailService extends EmailInterface {
  async send(to, subject, body) {
    console.log(`Sending via Gmail to ${to}: ${subject}`);
  }
}

// Service using interfaces
class UserService {
  constructor(database, emailService) {
    if (!(database instanceof DatabaseInterface)) {
      throw new Error('Database must implement DatabaseInterface');
    }
    if (!(emailService instanceof EmailInterface)) {
      throw new Error('Email service must implement EmailInterface');
    }
    
    this.database = database;
    this.emailService = emailService;
  }
  
  async createUser(userData) {
    const user = await this.database.save(userData);
    await this.emailService.send(
      user.email, 
      'Welcome!', 
      'Thanks for joining!'
    );
    return user;
  }
}
```

## 🔧 DI Container Implementation

### Simple DI Container:
```javascript
class DIContainer {
  constructor() {
    this.services = new Map();
    this.singletons = new Map();
  }
  
  // Register a service
  register(name, definition, options = {}) {
    this.services.set(name, {
      definition,
      singleton: options.singleton || false,
      dependencies: options.dependencies || []
    });
  }
  
  // Resolve a service
  resolve(name) {
    const serviceConfig = this.services.get(name);
    
    if (!serviceConfig) {
      throw new Error(`Service ${name} not found`);
    }
    
    // Return singleton if already created
    if (serviceConfig.singleton && this.singletons.has(name)) {
      return this.singletons.get(name);
    }
    
    // Resolve dependencies first
    const dependencies = serviceConfig.dependencies.map(dep => 
      this.resolve(dep)
    );
    
    // Create instance
    const ServiceClass = serviceConfig.definition;
    const instance = new ServiceClass(...dependencies);
    
    // Store singleton
    if (serviceConfig.singleton) {
      this.singletons.set(name, instance);
    }
    
    return instance;
  }
  
  // Auto-wire based on constructor parameters
  autoWire(ServiceClass) {
    const paramNames = this.getConstructorParams(ServiceClass);
    const dependencies = paramNames.map(name => this.resolve(name));
    return new ServiceClass(...dependencies);
  }
  
  getConstructorParams(func) {
    const funcStr = func.toString();
    const match = funcStr.match(/constructor\s*\(([^)]*)\)/);
    if (!match) return [];
    
    return match[1]
      .split(',')
      .map(param => param.trim())
      .filter(param => param.length > 0);
  }
}

// Usage example
const container = new DIContainer();

// Register services
container.register('database', MongoDatabase);
container.register('emailService', GmailService);
container.register('logger', ConsoleLogger, { singleton: true });

container.register('userService', UserService, {
  dependencies: ['database', 'emailService']
});

container.register('orderService', OrderService, {
  dependencies: ['database', 'emailService', 'logger']
});

// Resolve services
const userService = container.resolve('userService');
const orderService = container.resolve('orderService');
```

### Advanced DI Container with Decorators:
```javascript
// Decorator for dependency injection
function Injectable(dependencies = []) {
  return function(target) {
    target.dependencies = dependencies;
    return target;
  };
}

function Inject(token) {
  return function(target, propertyKey, parameterIndex) {
    const existingTokens = Reflect.getMetadata('design:paramtypes', target) || [];
    existingTokens[parameterIndex] = token;
    Reflect.defineMetadata('design:paramtypes', existingTokens, target);
  };
}

// Usage with decorators
@Injectable(['database', 'emailService'])
class UserService {
  constructor(
    @Inject('database') database,
    @Inject('emailService') emailService
  ) {
    this.database = database;
    this.emailService = emailService;
  }
}
```

## 🧪 Testing with Dependency Injection

### Easy Mocking:
```javascript
// Mock implementations for testing
class MockDatabase extends DatabaseInterface {
  constructor() {
    super();
    this.data = new Map();
  }
  
  async save(data) {
    const id = Math.random().toString(36);
    this.data.set(id, { ...data, id });
    return { ...data, id };
  }
  
  async findById(id) {
    return this.data.get(id);
  }
}

class MockEmailService extends EmailInterface {
  constructor() {
    super();
    this.sentEmails = [];
  }
  
  async send(to, subject, body) {
    this.sentEmails.push({ to, subject, body, sentAt: new Date() });
    return true;
  }
  
  getSentEmails() {
    return this.sentEmails;
  }
}

// Test suite
describe('UserService', () => {
  let userService;
  let mockDatabase;
  let mockEmailService;
  
  beforeEach(() => {
    mockDatabase = new MockDatabase();
    mockEmailService = new MockEmailService();
    userService = new UserService(mockDatabase, mockEmailService);
  });
  
  test('should create user and send welcome email', async () => {
    const userData = { name: 'John', email: 'john@example.com' };
    
    const user = await userService.createUser(userData);
    
    expect(user.id).toBeDefined();
    expect(user.name).toBe('John');
    
    const sentEmails = mockEmailService.getSentEmails();
    expect(sentEmails).toHaveLength(1);
    expect(sentEmails[0].to).toBe('john@example.com');
    expect(sentEmails[0].subject).toBe('Welcome!');
  });
});
```

## 🚀 Advanced Patterns

### 1. **Factory Pattern with DI**
```javascript
class ServiceFactory {
  constructor(container) {
    this.container = container;
  }
  
  createPaymentService(type) {
    switch(type) {
      case 'stripe':
        return this.container.resolve('stripePaymentService');
      case 'paypal':
        return this.container.resolve('paypalPaymentService');
      default:
        throw new Error(`Unknown payment type: ${type}`);
    }
  }
}

// Register factory
container.register('serviceFactory', ServiceFactory, {
  dependencies: ['container']
});
```

### 2. **Aspect-Oriented Programming**
```javascript
function Logging(target, propertyName, descriptor) {
  const method = descriptor.value;
  
  descriptor.value = function(...args) {
    console.log(`Calling ${propertyName} with args:`, args);
    const result = method.apply(this, args);
    console.log(`${propertyName} returned:`, result);
    return result;
  };
  
  return descriptor;
}

class UserService {
  constructor(database, emailService) {
    this.database = database;
    this.emailService = emailService;
  }
  
  @Logging
  async createUser(userData) {
    // Method implementation
  }
}
```

### 3. **Configuration-Based DI**
```javascript
const config = {
  services: {
    database: {
      class: 'MongoDatabase',
      singleton: true,
      config: {
        connectionString: 'mongodb://localhost:27017/myapp'
      }
    },
    emailService: {
      class: 'GmailService',
      dependencies: ['logger'],
      config: {
        apiKey: process.env.GMAIL_API_KEY
      }
    },
    userService: {
      class: 'UserService',
      dependencies: ['database', 'emailService']
    }
  }
};

class ConfigurableDIContainer extends DIContainer {
  loadFromConfig(config) {
    for (const [name, serviceConfig] of Object.entries(config.services)) {
      const ServiceClass = this.getClassByName(serviceConfig.class);
      
      this.register(name, ServiceClass, {
        singleton: serviceConfig.singleton,
        dependencies: serviceConfig.dependencies,
        config: serviceConfig.config
      });
    }
  }
  
  getClassByName(className) {
    // Dynamic class loading logic
    const classes = {
      MongoDatabase,
      GmailService,
      UserService
      // ... other classes
    };
    
    return classes[className];
  }
}
```

## 💡 Common Interview Questions & Answers

### Q1: "What are the benefits of dependency injection?"
**Answer:**
"DI provides several key benefits:
1. **Testability** - Easy to mock dependencies for unit testing
2. **Loose coupling** - Classes don't depend on concrete implementations
3. **Flexibility** - Easy to swap implementations without changing code
4. **Single Responsibility** - Classes focus on their core logic, not dependency management
5. **Configuration** - Dependencies can be configured externally"

### Q2: "What's the difference between DI and Service Locator pattern?"
**Answer:**
"In DI, dependencies are pushed to the class (Inversion of Control). In Service Locator, the class pulls dependencies from a central registry. DI is preferred because:
- Dependencies are explicit in constructor
- No hidden dependencies on service locator
- Better testability
- Follows Dependency Inversion Principle"

### Q3: "How do you handle circular dependencies?"
**Answer:**
"Several strategies:
1. **Interface segregation** - Split interfaces to break cycles
2. **Event-driven communication** - Use events instead of direct calls
3. **Lazy initialization** - Use factories or lazy loading
4. **Redesign** - Often indicates poor design, consider refactoring
5. **Proxy pattern** - Use proxies to break the cycle"

### Q4: "When should you not use dependency injection?"
**Answer:**
"Avoid DI when:
- Simple applications with few dependencies
- Performance-critical code where DI overhead matters
- Value objects that are immutable and stateless
- Framework code that needs direct control
- When it adds unnecessary complexity without benefits"

## 🔧 Best Practices

### 1. **Constructor Injection Preferred**
```javascript
// ✅ Preferred - Constructor injection
class OrderService {
  constructor(database, emailService) {
    this.database = database;
    this.emailService = emailService;
  }
}

// ❌ Avoid - Setter injection (optional dependencies only)
class OrderService {
  setDatabase(database) {
    this.database = database;
  }
}
```

### 2. **Depend on Abstractions**
```javascript
// ✅ Good - Depend on interface
class UserService {
  constructor(repository) { // IUserRepository
    this.repository = repository;
  }
}

// ❌ Bad - Depend on concrete class
class UserService {
  constructor(mongoUserRepository) { // Concrete implementation
    this.repository = mongoUserRepository;
  }
}
```

### 3. **Validate Dependencies**
```javascript
class UserService {
  constructor(database, emailService) {
    if (!database) {
      throw new Error('Database is required');
    }
    if (!emailService) {
      throw new Error('Email service is required');
    }
    
    this.database = database;
    this.emailService = emailService;
  }
}
```

## 🎯 Popular DI Frameworks

### Node.js / JavaScript:
- **InversifyJS** - Full-featured DI container
- **Awilix** - Lightweight, functional DI
- **TypeDI** - Decorator-based DI for TypeScript
- **TSyringe** - Microsoft's lightweight DI

### Java:
- **Spring Framework** - Most popular Java DI
- **Google Guice** - Lightweight DI framework
- **CDI** - Java EE standard

### .NET:
- **Built-in DI** - .NET Core built-in container
- **Autofac** - Popular third-party container
- **Ninject** - Lightweight DI framework

## 🔧 Quick Checklist

- [ ] Understand the three types of DI
- [ ] Know how to implement a simple DI container
- [ ] Explain benefits for testing and maintainability
- [ ] Handle circular dependencies
- [ ] Follow best practices (constructor injection, interfaces)
- [ ] Know popular DI frameworks

## 🎉 You're Dependency Injection Ready!

**Key Message**: Dependency injection promotes **loose coupling, testability, and maintainability** by inverting control of dependency management!

**Interview Golden Rule**: Always emphasize how DI makes code more testable and maintainable by removing hard-coded dependencies.

Perfect! Now you can confidently discuss dependency injection patterns! 💉 