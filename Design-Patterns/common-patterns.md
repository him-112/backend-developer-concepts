# Common Backend Design Patterns

## 🔍 Overview
Design patterns are reusable solutions to common problems in software design. Understanding these patterns is crucial for writing maintainable, scalable backend code.

## 🏭 Creational Patterns

### 1. Singleton Pattern
**Purpose**: Ensure a class has only one instance and provide global access to it.
**Use Cases**: Database connections, logging, configuration, caching

```javascript
class DatabaseConnection {
  constructor() {
    if (DatabaseConnection.instance) {
      return DatabaseConnection.instance;
    }
    this.connection = null;
    this.isConnected = false;
    DatabaseConnection.instance = this;
  }
  
  async connect() {
    if (!this.isConnected) {
      this.connection = await createConnection(config);
      this.isConnected = true;
    }
    return this.connection;
  }
}
```

### 2. Factory Pattern
**Purpose**: Create objects without specifying their exact classes.
**Use Cases**: Creating different types of database connections, payment processors

```javascript
class PaymentProcessorFactory {
  static createProcessor(type) {
    switch (type.toLowerCase()) {
      case 'stripe':
        return new StripeProcessor();
      case 'paypal':
        return new PayPalProcessor();
      default:
        throw new Error(`Unknown payment processor: ${type}`);
    }
  }
}
```

## 🏗️ Structural Patterns

### 3. Decorator Pattern
**Purpose**: Add new functionality to objects dynamically without altering their structure.
**Use Cases**: Middleware, logging, authentication, caching

```javascript
class CachingDecorator {
  constructor(service) {
    this.service = service;
    this.cache = new Map();
  }
  
  async getUser(id) {
    if (this.cache.has(id)) {
      return this.cache.get(id);
    }
    
    const result = await this.service.getUser(id);
    this.cache.set(id, result);
    return result;
  }
}
```

## 🎭 Behavioral Patterns

### 4. Observer Pattern
**Purpose**: Define a one-to-many dependency between objects for notifications.
**Use Cases**: Event systems, pub/sub messaging, model-view architectures

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = [];
    }
    this.events[eventName].push(callback);
  }
  
  emit(eventName, data) {
    if (this.events[eventName]) {
      this.events[eventName].forEach(callback => callback(data));
    }
  }
}
```

### 5. Strategy Pattern
**Purpose**: Define a family of algorithms and make them interchangeable.
**Use Cases**: Payment processing, validation strategies, pricing strategies

```javascript
class PricingContext {
  constructor(strategy) {
    this.strategy = strategy;
  }
  
  setStrategy(strategy) {
    this.strategy = strategy;
  }
  
  calculatePrice(basePrice, context) {
    return this.strategy.calculate(basePrice, context);
  }
}

class StudentDiscountStrategy {
  calculate(price, context) {
    return price * 0.8; // 20% discount
  }
}
```

## ❓ Common Interview Questions

1. **Q: When would you use the Singleton pattern?**
   A: For shared resources like database connections, but be aware of testing difficulties and tight coupling.

2. **Q: How does Strategy pattern differ from State pattern?**
   A: Strategy changes algorithms based on context, State changes behavior based on internal state.

3. **Q: What's the difference between Observer and Pub/Sub?**
   A: Observer has direct coupling, Pub/Sub uses a message broker for loose coupling.

4. **Q: When would you choose Factory over direct instantiation?**
   A: When object creation is complex or when you need to choose implementation at runtime.

---

**Key Takeaway**: Use patterns to solve actual problems, not for the sake of using patterns. The goal is maintainable, scalable code. 