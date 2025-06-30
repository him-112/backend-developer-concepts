# SOLID Principles

## 🔍 Overview
SOLID is an acronym for five design principles that make software designs more understandable, flexible, and maintainable. Essential for writing clean, scalable backend code.

## 🔧 The SOLID Principles

### S - Single Responsibility Principle (SRP)
**Definition**: A class should have only one reason to change. Each class should have only one job or responsibility.

```javascript
// BAD - Multiple responsibilities
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  
  // User data responsibility
  getName() {
    return this.name;
  }
  
  getEmail() {
    return this.email;
  }
  
  // Database responsibility (VIOLATION!)
  save() {
    console.log(`Saving user ${this.name} to database`);
    // Database logic here
  }
  
  // Email responsibility (VIOLATION!)
  sendWelcomeEmail() {
    console.log(`Sending welcome email to ${this.email}`);
    // Email sending logic here
  }
  
  // Validation responsibility (VIOLATION!)
  validateEmail() {
    return this.email.includes('@') && this.email.includes('.');
  }
}

// GOOD - Single responsibility per class
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  
  getName() {
    return this.name;
  }
  
  getEmail() {
    return this.email;
  }
}

class UserRepository {
  save(user) {
    console.log(`Saving user ${user.getName()} to database`);
    // Database logic here
  }
  
  findById(id) {
    console.log(`Finding user with ID: ${id}`);
    // Database query logic
  }
  
  delete(user) {
    console.log(`Deleting user ${user.getName()}`);
    // Delete logic
  }
}

class EmailService {
  sendWelcomeEmail(user) {
    console.log(`Sending welcome email to ${user.getEmail()}`);
    // Email sending logic
  }
  
  sendPasswordResetEmail(user) {
    console.log(`Sending password reset email to ${user.getEmail()}`);
    // Password reset email logic
  }
}

class UserValidator {
  validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
  
  validateName(name) {
    return name && name.length >= 2;
  }
}

// Usage
const user = new User('John Doe', 'john@example.com');
const userRepo = new UserRepository();
const emailService = new EmailService();
const validator = new UserValidator();

if (validator.validateEmail(user.getEmail())) {
  userRepo.save(user);
  emailService.sendWelcomeEmail(user);
}
```

### O - Open/Closed Principle (OCP)
**Definition**: Classes should be open for extension but closed for modification. You should be able to add new functionality without changing existing code.

```javascript
// BAD - Modifying existing code for new shapes
class AreaCalculator {
  calculateArea(shapes) {
    let totalArea = 0;
    
    for (const shape of shapes) {
      if (shape.type === 'rectangle') {
        totalArea += shape.width * shape.height;
      } else if (shape.type === 'circle') {
        totalArea += Math.PI * shape.radius * shape.radius;
      }
      // Adding triangle requires modifying this method (VIOLATION!)
      else if (shape.type === 'triangle') {
        const s = (shape.a + shape.b + shape.c) / 2;
        totalArea += Math.sqrt(s * (s - shape.a) * (s - shape.b) * (s - shape.c));
      }
    }
    
    return totalArea;
  }
}

// GOOD - Open for extension, closed for modification
class Shape {
  calculateArea() {
    throw new Error('calculateArea must be implemented');
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }
  
  calculateArea() {
    return this.width * this.height;
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }
  
  calculateArea() {
    return Math.PI * this.radius * this.radius;
  }
}

// New shape can be added without modifying existing code
class Triangle extends Shape {
  constructor(a, b, c) {
    super();
    this.a = a;
    this.b = b;
    this.c = c;
  }
  
  calculateArea() {
    const s = (this.a + this.b + this.c) / 2;
    return Math.sqrt(s * (s - this.a) * (s - this.b) * (s - this.c));
  }
}

class AreaCalculator {
  calculateTotalArea(shapes) {
    return shapes.reduce((total, shape) => total + shape.calculateArea(), 0);
  }
}

// Usage - Adding new shapes doesn't require changing AreaCalculator
const shapes = [
  new Rectangle(5, 3),
  new Circle(4),
  new Triangle(3, 4, 5)
];

const calculator = new AreaCalculator();
console.log(calculator.calculateTotalArea(shapes));
```

### L - Liskov Substitution Principle (LSP)
**Definition**: Objects of a superclass should be replaceable with objects of its subclasses without breaking the application.

```javascript
// BAD - Violates LSP
class Bird {
  fly() {
    return 'Flying in the sky';
  }
}

class Eagle extends Bird {
  fly() {
    return 'Eagle soars high';
  }
}

class Penguin extends Bird {
  fly() {
    // Penguins can't fly! This breaks LSP
    throw new Error('Penguins cannot fly');
  }
}

// This function expects all birds to fly
function makeBirdFly(bird) {
  return bird.fly(); // Will throw error for Penguin!
}

// GOOD - Proper hierarchy respecting LSP
class Bird {
  constructor(name) {
    this.name = name;
  }
  
  eat() {
    return `${this.name} is eating`;
  }
  
  makeSound() {
    return `${this.name} makes a sound`;
  }
}

class FlyingBird extends Bird {
  fly() {
    return `${this.name} is flying`;
  }
}

class SwimmingBird extends Bird {
  swim() {
    return `${this.name} is swimming`;
  }
}

class Eagle extends FlyingBird {
  fly() {
    return `${this.name} soars majestically`;
  }
  
  hunt() {
    return `${this.name} hunts for prey`;
  }
}

class Penguin extends SwimmingBird {
  swim() {
    return `${this.name} swims gracefully underwater`;
  }
  
  slide() {
    return `${this.name} slides on ice`;
  }
}

// Functions work with appropriate bird types
function makeFlyingBirdFly(flyingBird) {
  return flyingBird.fly(); // Works for all FlyingBird subclasses
}

function makeSwimmingBirdSwim(swimmingBird) {
  return swimmingBird.swim(); // Works for all SwimmingBird subclasses
}

const eagle = new Eagle('Golden Eagle');
const penguin = new Penguin('Emperor Penguin');

console.log(makeFlyingBirdFly(eagle)); // Works fine
console.log(makeSwimmingBirdSwim(penguin)); // Works fine
```

### I - Interface Segregation Principle (ISP)
**Definition**: No client should be forced to depend on methods it does not use. Create specific interfaces rather than one general-purpose interface.

```javascript
// BAD - Fat interface
class MultiFunctionDevice {
  print() {
    throw new Error('print must be implemented');
  }
  
  scan() {
    throw new Error('scan must be implemented');
  }
  
  fax() {
    throw new Error('fax must be implemented');
  }
  
  copy() {
    throw new Error('copy must be implemented');
  }
}

// Simple printer forced to implement unused methods
class SimplePrinter extends MultiFunctionDevice {
  print() {
    return 'Printing document';
  }
  
  // Forced to implement methods it doesn't need (VIOLATION!)
  scan() {
    throw new Error('SimplePrinter cannot scan');
  }
  
  fax() {
    throw new Error('SimplePrinter cannot fax');
  }
  
  copy() {
    throw new Error('SimplePrinter cannot copy');
  }
}

// GOOD - Segregated interfaces
class Printer {
  print() {
    throw new Error('print must be implemented');
  }
}

class Scanner {
  scan() {
    throw new Error('scan must be implemented');
  }
}

class FaxMachine {
  fax() {
    throw new Error('fax must be implemented');
  }
}

class Copier {
  copy() {
    throw new Error('copy must be implemented');
  }
}

// Simple printer only implements what it needs
class SimplePrinter extends Printer {
  print() {
    return 'Printing document';
  }
}

// All-in-one printer implements multiple interfaces
class AllInOnePrinter extends Printer {
  constructor() {
    super();
    this.scanner = new BasicScanner();
    this.faxMachine = new BasicFaxMachine();
  }
  
  print() {
    return 'All-in-one printing';
  }
  
  scan() {
    return this.scanner.scan();
  }
  
  fax() {
    return this.faxMachine.fax();
  }
  
  copy() {
    const scanned = this.scan();
    const printed = this.print();
    return `Copied: ${scanned} -> ${printed}`;
  }
}

class BasicScanner extends Scanner {
  scan() {
    return 'Scanning document';
  }
}

class BasicFaxMachine extends FaxMachine {
  fax() {
    return 'Sending fax';
  }
}
```

### D - Dependency Inversion Principle (DIP)
**Definition**: High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details.

```javascript
// BAD - High-level module depends on low-level module
class MySQLDatabase {
  save(user) {
    console.log(`Saving ${user.name} to MySQL database`);
  }
  
  find(id) {
    console.log(`Finding user ${id} in MySQL database`);
    return { id, name: 'John Doe' };
  }
}

class UserService {
  constructor() {
    this.database = new MySQLDatabase(); // Direct dependency (VIOLATION!)
  }
  
  createUser(userData) {
    // Business logic
    const user = { ...userData, createdAt: new Date() };
    
    // Tightly coupled to MySQL
    this.database.save(user);
    return user;
  }
  
  getUser(id) {
    return this.database.find(id);
  }
}

// GOOD - Depend on abstractions
class DatabaseInterface {
  save(user) {
    throw new Error('save must be implemented');
  }
  
  find(id) {
    throw new Error('find must be implemented');
  }
}

class MySQLDatabase extends DatabaseInterface {
  save(user) {
    console.log(`Saving ${user.name} to MySQL database`);
  }
  
  find(id) {
    console.log(`Finding user ${id} in MySQL database`);
    return { id, name: 'John Doe' };
  }
}

class PostgreSQLDatabase extends DatabaseInterface {
  save(user) {
    console.log(`Saving ${user.name} to PostgreSQL database`);
  }
  
  find(id) {
    console.log(`Finding user ${id} in PostgreSQL database`);
    return { id, name: 'Jane Smith' };
  }
}

class MongoDatabase extends DatabaseInterface {
  save(user) {
    console.log(`Saving ${user.name} to MongoDB`);
  }
  
  find(id) {
    console.log(`Finding user ${id} in MongoDB`);
    return { id, name: 'Bob Wilson' };
  }
}

// Dependency injection
class UserService {
  constructor(database) {
    if (!(database instanceof DatabaseInterface)) {
      throw new Error('Database must implement DatabaseInterface');
    }
    this.database = database; // Injected dependency
  }
  
  createUser(userData) {
    // Business logic
    const user = { ...userData, createdAt: new Date() };
    
    // Works with any database implementation
    this.database.save(user);
    return user;
  }
  
  getUser(id) {
    return this.database.find(id);
  }
}

// Usage - Can easily switch database implementations
const mysqlDB = new MySQLDatabase();
const postgresDB = new PostgreSQLDatabase();
const mongoDB = new MongoDatabase();

const userServiceMySQL = new UserService(mysqlDB);
const userServicePostgres = new UserService(postgresDB);
const userServiceMongo = new UserService(mongoDB);

// All work the same way
userServiceMySQL.createUser({ name: 'Alice' });
userServicePostgres.createUser({ name: 'Bob' });
userServiceMongo.createUser({ name: 'Charlie' });
```

## 🏗️ Real-World Backend Example

```javascript
// E-commerce Order Processing System following SOLID principles

// Abstractions (Interfaces)
class PaymentProcessor {
  processPayment(amount, paymentData) {
    throw new Error('processPayment must be implemented');
  }
}

class NotificationService {
  sendNotification(recipient, message) {
    throw new Error('sendNotification must be implemented');
  }
}

class OrderRepository {
  save(order) {
    throw new Error('save must be implemented');
  }
  
  findById(id) {
    throw new Error('findById must be implemented');
  }
}

// Concrete implementations
class CreditCardProcessor extends PaymentProcessor {
  processPayment(amount, paymentData) {
    console.log(`Processing $${amount} via credit card ending in ${paymentData.lastFour}`);
    return { success: true, transactionId: 'cc_' + Date.now() };
  }
}

class PayPalProcessor extends PaymentProcessor {
  processPayment(amount, paymentData) {
    console.log(`Processing $${amount} via PayPal account ${paymentData.email}`);
    return { success: true, transactionId: 'pp_' + Date.now() };
  }
}

class EmailNotification extends NotificationService {
  sendNotification(recipient, message) {
    console.log(`Sending email to ${recipient}: ${message}`);
  }
}

class SMSNotification extends NotificationService {
  sendNotification(recipient, message) {
    console.log(`Sending SMS to ${recipient}: ${message}`);
  }
}

class DatabaseOrderRepository extends OrderRepository {
  save(order) {
    console.log(`Saving order ${order.id} to database`);
  }
  
  findById(id) {
    console.log(`Finding order ${id} in database`);
    return { id, status: 'pending' };
  }
}

// Business logic (follows SRP and DIP)
class OrderService {
  constructor(orderRepo, paymentProcessor, notificationService) {
    this.orderRepository = orderRepo;
    this.paymentProcessor = paymentProcessor;
    this.notificationService = notificationService;
  }
  
  processOrder(orderData, paymentData, customerContact) {
    // Single responsibility: process orders
    const order = {
      id: 'order_' + Date.now(),
      ...orderData,
      status: 'processing',
      createdAt: new Date()
    };
    
    // Process payment
    const paymentResult = this.paymentProcessor.processPayment(
      order.total, 
      paymentData
    );
    
    if (paymentResult.success) {
      order.status = 'paid';
      order.transactionId = paymentResult.transactionId;
      
      // Save order
      this.orderRepository.save(order);
      
      // Send confirmation
      this.notificationService.sendNotification(
        customerContact,
        `Order ${order.id} confirmed! Transaction: ${paymentResult.transactionId}`
      );
    } else {
      order.status = 'failed';
      this.notificationService.sendNotification(
        customerContact,
        `Order ${order.id} failed. Please try again.`
      );
    }
    
    return order;
  }
}

// Usage with dependency injection
const orderRepo = new DatabaseOrderRepository();
const creditCardProcessor = new CreditCardProcessor();
const emailNotification = new EmailNotification();

const orderService = new OrderService(
  orderRepo,
  creditCardProcessor,
  emailNotification
);

const order = orderService.processOrder(
  { items: ['laptop', 'mouse'], total: 1200 },
  { lastFour: '1234', cvv: '123' },
  'customer@example.com'
);

// Easy to swap implementations
const smsNotification = new SMSNotification();
const paypalProcessor = new PayPalProcessor();

const orderServiceSMS = new OrderService(
  orderRepo,
  paypalProcessor,
  smsNotification
);
```

## ❓ Interview Questions

1. **Q: What does SOLID stand for?**
   A: Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion.

2. **Q: How does SRP improve code maintainability?**
   A: Each class has one reason to change, making it easier to understand, test, and modify without affecting other functionality.

3. **Q: Give an example of violating the Open/Closed principle.**
   A: Modifying existing class code to add new functionality instead of extending it through inheritance or composition.

4. **Q: How does Dependency Inversion help with testing?**
   A: You can inject mock dependencies, making unit testing easier by isolating the class under test.

5. **Q: What's the difference between LSP and polymorphism?**
   A: Polymorphism allows different implementations; LSP ensures substitutability without breaking functionality.

## 💡 Benefits of SOLID Principles

### Code Quality
- **Maintainable**: Easy to modify and extend
- **Testable**: Can isolate and test individual components
- **Readable**: Clear responsibilities and dependencies
- **Flexible**: Can adapt to changing requirements

### Development Process
- **Parallel Development**: Teams can work on different components
- **Reduced Coupling**: Changes in one area don't break others
- **Easier Debugging**: Problems are isolated to specific components
- **Better Collaboration**: Clear interfaces make integration easier

---

**Key Takeaway**: SOLID principles guide you to write better object-oriented code. They're not rules but guidelines that help create more maintainable, flexible, and testable software. Apply them judiciously - not every small class needs all principles. 