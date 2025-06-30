# Object-Oriented Programming (OOP) Concepts

## 🔍 Overview
Object-Oriented Programming is a programming paradigm based on objects that contain data (attributes) and code (methods). Essential for backend development and frequently tested in interviews.

## 🏗️ Core OOP Principles

### 1. Encapsulation
**Definition**: Bundling data and methods that operate on that data within a single unit (class), and restricting access to internal details.

```javascript
// BAD - No encapsulation
let user = {
  name: 'John',
  email: 'john@example.com',
  balance: 1000
};

// Anyone can modify balance directly
user.balance = -500; // Invalid state!

// GOOD - Encapsulation with private fields
class BankAccount {
  #balance; // Private field
  #accountNumber;
  
  constructor(initialBalance, accountNumber) {
    this.#balance = initialBalance;
    this.#accountNumber = accountNumber;
  }
  
  // Public methods to interact with private data
  deposit(amount) {
    if (amount <= 0) {
      throw new Error('Deposit amount must be positive');
    }
    this.#balance += amount;
    return this.#balance;
  }
  
  withdraw(amount) {
    if (amount <= 0) {
      throw new Error('Withdrawal amount must be positive');
    }
    if (amount > this.#balance) {
      throw new Error('Insufficient funds');
    }
    this.#balance -= amount;
    return this.#balance;
  }
  
  getBalance() {
    return this.#balance;
  }
  
  // Private method
  #validateTransaction(amount, type) {
    if (amount <= 0) {
      throw new Error(`${type} amount must be positive`);
    }
  }
}

const account = new BankAccount(1000, '12345');
console.log(account.getBalance()); // 1000
account.deposit(500); // 1500
// account.#balance = -100; // Error - cannot access private field
```

### 2. Inheritance
**Definition**: Creating new classes based on existing classes, inheriting properties and methods while adding or overriding functionality.

```javascript
// Base class
class Animal {
  constructor(name, species) {
    this.name = name;
    this.species = species;
  }
  
  makeSound() {
    return `${this.name} makes a sound`;
  }
  
  eat(food) {
    return `${this.name} eats ${food}`;
  }
  
  getInfo() {
    return `${this.name} is a ${this.species}`;
  }
}

// Derived class
class Dog extends Animal {
  constructor(name, breed) {
    super(name, 'Canine'); // Call parent constructor
    this.breed = breed;
  }
  
  // Override parent method
  makeSound() {
    return `${this.name} barks: Woof!`;
  }
  
  // Add new method specific to Dog
  fetch() {
    return `${this.name} fetches the ball`;
  }
  
  // Override getInfo to include breed
  getInfo() {
    return `${super.getInfo()} (${this.breed})`;
  }
}

class Cat extends Animal {
  constructor(name, breed) {
    super(name, 'Feline');
    this.breed = breed;
  }
  
  makeSound() {
    return `${this.name} meows: Meow!`;
  }
  
  climb() {
    return `${this.name} climbs the tree`;
  }
}

// Usage
const dog = new Dog('Buddy', 'Golden Retriever');
const cat = new Cat('Whiskers', 'Persian');

console.log(dog.makeSound()); // "Buddy barks: Woof!"
console.log(cat.makeSound()); // "Whiskers meows: Meow!"
console.log(dog.getInfo()); // "Buddy is a Canine (Golden Retriever)"
```

### 3. Polymorphism
**Definition**: The ability of objects of different types to be treated as objects of a common base type, while still maintaining their specific behavior.

```javascript
// Base class
class Shape {
  constructor(name) {
    this.name = name;
  }
  
  // Abstract method - should be overridden
  calculateArea() {
    throw new Error('calculateArea must be implemented by subclass');
  }
  
  calculatePerimeter() {
    throw new Error('calculatePerimeter must be implemented by subclass');
  }
  
  getInfo() {
    return `${this.name}: Area = ${this.calculateArea()}, Perimeter = ${this.calculatePerimeter()}`;
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super('Rectangle');
    this.width = width;
    this.height = height;
  }
  
  calculateArea() {
    return this.width * this.height;
  }
  
  calculatePerimeter() {
    return 2 * (this.width + this.height);
  }
}

class Circle extends Shape {
  constructor(radius) {
    super('Circle');
    this.radius = radius;
  }
  
  calculateArea() {
    return Math.PI * this.radius * this.radius;
  }
  
  calculatePerimeter() {
    return 2 * Math.PI * this.radius;
  }
}

class Triangle extends Shape {
  constructor(a, b, c) {
    super('Triangle');
    this.a = a;
    this.b = b;
    this.c = c;
  }
  
  calculateArea() {
    // Using Heron's formula
    const s = (this.a + this.b + this.c) / 2;
    return Math.sqrt(s * (s - this.a) * (s - this.b) * (s - this.c));
  }
  
  calculatePerimeter() {
    return this.a + this.b + this.c;
  }
}

// Polymorphism in action
function printShapeInfo(shapes) {
  shapes.forEach(shape => {
    // Same method call, different behavior based on object type
    console.log(shape.getInfo());
  });
}

const shapes = [
  new Rectangle(5, 3),
  new Circle(4),
  new Triangle(3, 4, 5)
];

printShapeInfo(shapes);
// Rectangle: Area = 15, Perimeter = 16
// Circle: Area = 50.27, Perimeter = 25.13
// Triangle: Area = 6, Perimeter = 12
```

### 4. Abstraction
**Definition**: Hiding complex implementation details while exposing only essential features and functionality.

```javascript
// Abstract base class for payment processing
class PaymentProcessor {
  constructor(amount) {
    this.amount = amount;
    this.transactionId = this.generateTransactionId();
  }
  
  // Template method - defines the algorithm structure
  processPayment() {
    this.validatePayment();
    this.authenticateUser();
    const result = this.executePayment();
    this.logTransaction(result);
    this.sendConfirmation(result);
    return result;
  }
  
  // Common methods
  validatePayment() {
    if (this.amount <= 0) {
      throw new Error('Payment amount must be positive');
    }
  }
  
  generateTransactionId() {
    return 'TXN_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
  
  logTransaction(result) {
    console.log(`Transaction ${this.transactionId}: ${result.success ? 'SUCCESS' : 'FAILED'}`);
  }
  
  // Abstract methods - must be implemented by subclasses
  authenticateUser() {
    throw new Error('authenticateUser must be implemented');
  }
  
  executePayment() {
    throw new Error('executePayment must be implemented');
  }
  
  sendConfirmation(result) {
    throw new Error('sendConfirmation must be implemented');
  }
}

// Concrete implementation for credit card
class CreditCardProcessor extends PaymentProcessor {
  constructor(amount, cardNumber, cvv) {
    super(amount);
    this.cardNumber = cardNumber;
    this.cvv = cvv;
  }
  
  authenticateUser() {
    // Simulate card authentication
    if (this.cardNumber.length !== 16 || this.cvv.length !== 3) {
      throw new Error('Invalid card details');
    }
    console.log('Credit card authenticated');
  }
  
  executePayment() {
    // Simulate payment processing
    console.log(`Processing credit card payment of $${this.amount}`);
    return {
      success: true,
      transactionId: this.transactionId,
      method: 'credit_card'
    };
  }
  
  sendConfirmation(result) {
    console.log(`Credit card payment confirmation sent for transaction ${result.transactionId}`);
  }
}

// Concrete implementation for PayPal
class PayPalProcessor extends PaymentProcessor {
  constructor(amount, email, password) {
    super(amount);
    this.email = email;
    this.password = password;
  }
  
  authenticateUser() {
    // Simulate PayPal authentication
    if (!this.email.includes('@') || this.password.length < 6) {
      throw new Error('Invalid PayPal credentials');
    }
    console.log('PayPal user authenticated');
  }
  
  executePayment() {
    console.log(`Processing PayPal payment of $${this.amount}`);
    return {
      success: true,
      transactionId: this.transactionId,
      method: 'paypal'
    };
  }
  
  sendConfirmation(result) {
    console.log(`PayPal confirmation email sent to ${this.email}`);
  }
}

// Usage - Client code doesn't need to know implementation details
function processCustomerPayment(processor) {
  try {
    return processor.processPayment();
  } catch (error) {
    console.error('Payment failed:', error.message);
    return { success: false, error: error.message };
  }
}

const creditCardPayment = new CreditCardProcessor(100, '1234567890123456', '123');
const paypalPayment = new PayPalProcessor(75, 'user@example.com', 'securepass');

processCustomerPayment(creditCardPayment);
processCustomerPayment(paypalPayment);
```

## 🏛️ Advanced OOP Concepts

### Composition vs Inheritance
**Composition**: "Has-a" relationship - building complex objects by combining simpler ones.

```javascript
// Composition approach
class Engine {
  constructor(horsepower, type) {
    this.horsepower = horsepower;
    this.type = type;
  }
  
  start() {
    return `${this.type} engine started (${this.horsepower} HP)`;
  }
  
  stop() {
    return `${this.type} engine stopped`;
  }
}

class GPS {
  constructor() {
    this.isActive = false;
  }
  
  activate() {
    this.isActive = true;
    return 'GPS activated';
  }
  
  getLocation() {
    return this.isActive ? 'Current location: 40.7128° N, 74.0060° W' : 'GPS not active';
  }
}

class MusicSystem {
  constructor() {
    this.volume = 0;
    this.isPlaying = false;
  }
  
  play(song) {
    this.isPlaying = true;
    return `Playing: ${song}`;
  }
  
  setVolume(level) {
    this.volume = level;
    return `Volume set to ${level}`;
  }
}

// Car uses composition instead of inheritance
class Car {
  constructor(make, model, engineType, horsepower) {
    this.make = make;
    this.model = model;
    this.engine = new Engine(horsepower, engineType); // Composition
    this.gps = new GPS(); // Composition
    this.musicSystem = new MusicSystem(); // Composition
  }
  
  startCar() {
    const engineStatus = this.engine.start();
    const gpsStatus = this.gps.activate();
    return `${this.make} ${this.model} started. ${engineStatus}. ${gpsStatus}`;
  }
  
  navigate() {
    return this.gps.getLocation();
  }
  
  playMusic(song) {
    return this.musicSystem.play(song);
  }
}

const car = new Car('Toyota', 'Camry', 'V6', 300);
console.log(car.startCar());
console.log(car.navigate());
console.log(car.playMusic('Highway to Hell'));
```

### Interface Implementation (using Duck Typing)
```javascript
// Define expected interface behavior
class Flyable {
  fly() {
    throw new Error('fly() method must be implemented');
  }
}

class Swimmable {
  swim() {
    throw new Error('swim() method must be implemented');
  }
}

// Duck implementation
class Duck {
  constructor(name) {
    this.name = name;
  }
  
  fly() {
    return `${this.name} flies in the sky`;
  }
  
  swim() {
    return `${this.name} swims in the water`;
  }
  
  quack() {
    return `${this.name} says quack!`;
  }
}

// Airplane implementation
class Airplane {
  constructor(model) {
    this.model = model;
  }
  
  fly() {
    return `${this.model} flies at 30,000 feet`;
  }
}

// Fish implementation
class Fish {
  constructor(species) {
    this.species = species;
  }
  
  swim() {
    return `${this.species} swims underwater`;
  }
}

// Functions that work with any object implementing the interface
function makeFly(flyableObject) {
  return flyableObject.fly();
}

function makeSwim(swimmableObject) {
  return swimmableObject.swim();
}

const duck = new Duck('Donald');
const plane = new Airplane('Boeing 747');
const fish = new Fish('Salmon');

console.log(makeFly(duck)); // Donald flies in the sky
console.log(makeFly(plane)); // Boeing 747 flies at 30,000 feet
console.log(makeSwim(duck)); // Donald swims in the water
console.log(makeSwim(fish)); // Salmon swims underwater
```

## 🔧 OOP Design Patterns in Backend

### Factory Pattern
```javascript
class DatabaseConnection {
  connect() {
    throw new Error('connect() must be implemented');
  }
}

class MySQLConnection extends DatabaseConnection {
  connect() {
    return 'Connected to MySQL database';
  }
  
  query(sql) {
    return `Executing MySQL query: ${sql}`;
  }
}

class PostgreSQLConnection extends DatabaseConnection {
  connect() {
    return 'Connected to PostgreSQL database';
  }
  
  query(sql) {
    return `Executing PostgreSQL query: ${sql}`;
  }
}

class MongoDBConnection extends DatabaseConnection {
  connect() {
    return 'Connected to MongoDB database';
  }
  
  find(collection, query) {
    return `Finding documents in ${collection}: ${JSON.stringify(query)}`;
  }
}

// Factory
class DatabaseFactory {
  static createConnection(type) {
    switch (type.toLowerCase()) {
      case 'mysql':
        return new MySQLConnection();
      case 'postgresql':
        return new PostgreSQLConnection();
      case 'mongodb':
        return new MongoDBConnection();
      default:
        throw new Error(`Unknown database type: ${type}`);
    }
  }
}

// Usage
const mysqlDB = DatabaseFactory.createConnection('mysql');
const postgresDB = DatabaseFactory.createConnection('postgresql');
const mongoDB = DatabaseFactory.createConnection('mongodb');

console.log(mysqlDB.connect());
console.log(postgresDB.connect());
console.log(mongoDB.connect());
```

## ❓ Interview Questions

1. **Q: What are the four pillars of OOP?**
   A: Encapsulation (data hiding), Inheritance (code reuse), Polymorphism (multiple forms), Abstraction (hiding complexity).

2. **Q: Difference between composition and inheritance?**
   A: Inheritance is "is-a" relationship, composition is "has-a". Composition is more flexible and avoids inheritance hierarchy problems.

3. **Q: What is method overriding vs overloading?**
   A: Overriding: subclass provides different implementation of parent method. Overloading: multiple methods with same name but different parameters.

4. **Q: When to use abstract classes vs interfaces?**
   A: Abstract classes for shared code and "is-a" relationships. Interfaces for contracts and "can-do" relationships.

5. **Q: What is the Diamond Problem?**
   A: Multiple inheritance issue where a class inherits from two classes that have a common base class, causing ambiguity.

6. **Q: How does encapsulation improve security?**
   A: By hiding internal data and providing controlled access through methods, preventing unauthorized or invalid modifications.

## 💡 Best Practices

### Class Design
- **Single Responsibility**: Each class should have one reason to change
- **Favor Composition**: Use composition over inheritance when possible
- **Program to Interfaces**: Depend on abstractions, not concretions
- **Keep It Simple**: Don't over-engineer with unnecessary complexity

### Method Design
- **Small Methods**: Each method should do one thing well
- **Meaningful Names**: Use descriptive names for classes, methods, and variables
- **Consistent Parameters**: Keep parameter lists short and consistent
- **Return Consistently**: Methods should have predictable return types

### Error Handling
- **Fail Fast**: Validate inputs early and throw meaningful errors
- **Use Exceptions**: For exceptional cases, not control flow
- **Clean Up**: Properly dispose of resources in finally blocks

---

**Key Takeaway**: OOP helps organize code into reusable, maintainable components. Master the four pillars and understand when to apply each concept. Good OOP design leads to more robust and scalable backend systems. 