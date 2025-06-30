# System Architecture Patterns

## 🔍 Overview
System architecture patterns define how to structure and organize software systems for scalability and maintainability.

## 🏢 Monolithic Architecture

### Characteristics
- Single deployable unit
- All components tightly coupled
- Shared database
- Traditional approach

### Pros & Cons
✅ Simple development and deployment
✅ Easy testing and debugging
✅ No network latency between components
❌ Hard to scale individual components
❌ Technology lock-in
❌ Large codebase becomes unwieldy

### Example
```javascript
class ECommerceApp {
  async createOrder(userId, items) {
    const transaction = await db.beginTransaction();
    try {
      const user = await this.userService.getUser(userId, transaction);
      const order = await this.orderService.create(user, items, transaction);
      await this.paymentService.process(order, transaction);
      await transaction.commit();
      return order;
    } catch (error) {
      await transaction.rollback();
      throw error;
    }
  }
}
```

## 🔧 Microservices Architecture

### Characteristics
- Multiple small, independent services
- Each service has its own database
- Communicate over network
- Decentralized governance

### Pros & Cons
✅ Independent scaling and deployment
✅ Technology diversity
✅ Fault isolation
✅ Team independence
❌ Distributed system complexity
❌ Network latency
❌ Data consistency challenges
❌ Operational overhead

### Example
```javascript
// User Service
class UserService {
  async createUser(userData) {
    const user = await this.userDB.create(userData);
    await this.eventBus.publish('user.created', user);
    return user;
  }
}

// Order Service
class OrderService {
  async createOrder(orderData) {
    // Call other services via HTTP
    const user = await this.userServiceClient.getUser(orderData.userId);
    const order = await this.orderDB.create(orderData);
    await this.eventBus.publish('order.created', order);
    return order;
  }
}
```

## 🌐 Event-Driven Architecture

### Characteristics
- Components communicate through events
- Asynchronous processing
- Loose coupling
- Event producers and consumers

### Example
```javascript
class EventBus {
  async publish(eventType, data) {
    const event = {
      id: uuid(),
      type: eventType,
      data,
      timestamp: new Date().toISOString()
    };
    
    const handlers = this.subscribers.get(eventType) || [];
    await Promise.all(handlers.map(handler => handler(event)));
  }
  
  subscribe(eventType, handler) {
    if (!this.subscribers.has(eventType)) {
      this.subscribers.set(eventType, []);
    }
    this.subscribers.get(eventType).push(handler);
  }
}

// Usage
eventBus.subscribe('order.created', async (event) => {
  await paymentService.processPayment(event.data);
});
```

## 📱 Serverless Architecture

### Characteristics
- Functions as a Service (FaaS)
- Event-driven execution
- Automatic scaling
- Pay-per-use model

### Pros & Cons
✅ No server management
✅ Automatic scaling
✅ Cost effective for variable workloads
❌ Cold start latency
❌ Vendor lock-in
❌ Limited execution time

### Example
```javascript
// AWS Lambda function
exports.processOrder = async (event) => {
  const orderData = JSON.parse(event.body);
  
  try {
    const order = await dynamodb.put({
      TableName: 'Orders',
      Item: { id: uuid(), ...orderData }
    }).promise();
    
    // Trigger payment processing
    await sns.publish({
      TopicArn: process.env.PAYMENT_TOPIC,
      Message: JSON.stringify(order)
    }).promise();
    
    return {
      statusCode: 201,
      body: JSON.stringify({ success: true })
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  }
};
```

## 🔄 CQRS Pattern

### Concept
Separate read and write operations into different models.

### Example
```javascript
// Command (Write) Model
class UserCommandService {
  async createUser(userData) {
    const user = await this.writeDB.save(userData);
    await this.eventBus.publish('user.created', user);
    return user.id;
  }
}

// Query (Read) Model  
class UserQueryService {
  constructor() {
    this.eventBus.subscribe('user.created', this.updateReadModel.bind(this));
  }
  
  async getUser(userId) {
    return await this.readDB.findById(userId); // Optimized for reads
  }
  
  async updateReadModel(event) {
    await this.readDB.upsert(event.data);
  }
}
```

## ❓ Interview Questions

1. **Q: Monolith vs Microservices - when to choose what?**
   A: Monolith for small teams/simple domains. Microservices for large teams, complex domains, independent scaling needs.

2. **Q: Main challenges with microservices?**
   A: Network latency, data consistency, testing complexity, operational overhead.

3. **Q: How handle data consistency in microservices?**
   A: Event-driven architecture, saga patterns, eventual consistency.

4. **Q: What is event-driven architecture?**
   A: Components communicate through events instead of direct calls, enabling loose coupling and scalability.

5. **Q: When use serverless?**
   A: Variable workloads, event-driven processing, quick prototyping, cost optimization.

## 💡 Best Practices

### General Principles
- Start simple (monolith), evolve to complexity (microservices)
- Design for failure - use circuit breakers, timeouts, retries
- Implement proper monitoring and logging
- Use API gateways for microservices
- Plan data consistency strategy early

### Migration Strategy
```javascript
// Strangler Fig Pattern - gradually replace monolith
class APIGateway {
  async route(request) {
    if (this.shouldUseMicroservice(request.path)) {
      return await this.forwardToMicroservice(request);
    } else {
      return await this.forwardToMonolith(request);
    }
  }
}
```

---

**Key Takeaway**: Choose architecture based on team size, domain complexity, and scalability needs. Start simple and evolve as requirements grow. 