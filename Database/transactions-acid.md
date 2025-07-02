# Database Transactions & ACID - Interview Guide 🔄

## 🎯 What You'll Learn
Master database transactions and ACID properties for data consistency and reliability!

## 🌟 Transaction Fundamentals

### What is a Database Transaction?
Think of a transaction like a **bank transfer**:
- **All steps must complete** = Atomicity
- **Rules are followed** = Consistency  
- **No interference** = Isolation
- **Changes are permanent** = Durability

Either the entire transfer succeeds, or nothing happens at all!

### Key Interview Points ⭐
1. "Unit of work that must complete fully or not at all"
2. "ACID properties ensure data integrity"
3. "Essential for multi-step operations"
4. "Trade-offs between consistency and performance"
5. "Different isolation levels for different needs"

## 🔐 ACID Properties

### **A** - Atomicity
*All operations succeed or all fail*

```sql
-- ✅ Bank transfer example
BEGIN TRANSACTION;

UPDATE accounts 
SET balance = balance - 100 
WHERE account_id = 'from_account';

UPDATE accounts 
SET balance = balance + 100 
WHERE account_id = 'to_account';

-- If either update fails, both are rolled back
COMMIT;
```

```javascript
// Node.js example with database transaction
async function transferMoney(fromAccount, toAccount, amount) {
  const transaction = await db.beginTransaction();
  
  try {
    // Debit from source account
    await db.query(
      'UPDATE accounts SET balance = balance - ? WHERE id = ?',
      [amount, fromAccount],
      { transaction }
    );
    
    // Credit to destination account
    await db.query(
      'UPDATE accounts SET balance = balance + ? WHERE id = ?', 
      [amount, toAccount],
      { transaction }
    );
    
    // Both operations succeeded - commit
    await transaction.commit();
    console.log('Transfer completed successfully');
    
  } catch (error) {
    // Any failure - rollback all changes
    await transaction.rollback();
    console.error('Transfer failed, all changes reverted');
    throw error;
  }
}
```

### **C** - Consistency
*Database remains in valid state*

```sql
-- Constraint ensures consistency
ALTER TABLE accounts 
ADD CONSTRAINT positive_balance 
CHECK (balance >= 0);

-- This transaction will fail if it violates the constraint
BEGIN TRANSACTION;
UPDATE accounts SET balance = -50 WHERE account_id = 'acc1';
-- ERROR: violates check constraint "positive_balance"
ROLLBACK;
```

```javascript
// Application-level consistency checks
async function createOrder(orderData) {
  const transaction = await db.beginTransaction();
  
  try {
    // Check inventory availability
    const product = await db.query('SELECT quantity FROM products WHERE id = ?', 
      [orderData.productId], { transaction });
    
    if (product.quantity < orderData.quantity) {
      throw new Error('Insufficient inventory');
    }
    
    // Create order
    const order = await db.query('INSERT INTO orders (customer_id, product_id, quantity) VALUES (?, ?, ?)',
      [orderData.customerId, orderData.productId, orderData.quantity], 
      { transaction });
    
    // Reduce inventory
    await db.query('UPDATE products SET quantity = quantity - ? WHERE id = ?',
      [orderData.quantity, orderData.productId], 
      { transaction });
    
    await transaction.commit();
    return order;
    
  } catch (error) {
    await transaction.rollback();
    throw error;
  }
}
```

### **I** - Isolation
*Concurrent transactions don't interfere*

```sql
-- Transaction 1
BEGIN TRANSACTION;
SELECT balance FROM accounts WHERE id = 'acc1'; -- Reads 1000
UPDATE accounts SET balance = balance + 100 WHERE id = 'acc1';
-- ... other operations
COMMIT;

-- Transaction 2 (concurrent)
BEGIN TRANSACTION;
SELECT balance FROM accounts WHERE id = 'acc1'; -- Should it see 1000 or 1100?
UPDATE accounts SET balance = balance - 50 WHERE id = 'acc1';
COMMIT;
```

### **D** - Durability
*Committed changes survive system failures*

```javascript
// WAL (Write-Ahead Logging) ensures durability
class DatabaseEngine {
  async commitTransaction(transaction) {
    // 1. Write transaction log to disk first
    await this.writeLog(transaction.operations);
    
    // 2. Apply changes to database
    await this.applyChanges(transaction.operations);
    
    // 3. Mark transaction as committed in log
    await this.markCommitted(transaction.id);
    
    console.log('Transaction committed and durable');
  }
  
  async recover() {
    // On startup, replay committed transactions from log
    const uncommittedTransactions = await this.getUncommittedFromLog();
    
    for (const transaction of uncommittedTransactions) {
      if (transaction.status === 'committed') {
        await this.applyChanges(transaction.operations);
      } else {
        await this.rollbackChanges(transaction.operations);
      }
    }
  }
}
```

## 🔒 Isolation Levels

### 1. **Read Uncommitted**
*Lowest isolation, allows dirty reads*

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- Transaction 1
BEGIN;
UPDATE products SET price = 200 WHERE id = 1;
-- Not committed yet

-- Transaction 2 (different session)
SELECT price FROM products WHERE id = 1; -- Sees 200 (dirty read)
```

### 2. **Read Committed**
*Prevents dirty reads*

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Transaction 1
BEGIN;
UPDATE products SET price = 200 WHERE id = 1;
-- Not committed yet

-- Transaction 2
SELECT price FROM products WHERE id = 1; -- Sees old value (100)

-- Transaction 1
COMMIT;

-- Transaction 2
SELECT price FROM products WHERE id = 1; -- Now sees 200
```

### 3. **Repeatable Read**
*Prevents dirty reads and non-repeatable reads*

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Transaction 1
BEGIN;
SELECT price FROM products WHERE id = 1; -- Reads 100

-- Transaction 2
UPDATE products SET price = 200 WHERE id = 1;
COMMIT;

-- Transaction 1
SELECT price FROM products WHERE id = 1; -- Still sees 100 (repeatable read)
COMMIT;
```

### 4. **Serializable**
*Highest isolation, prevents all anomalies*

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Transactions are executed as if they were serial
-- Complete isolation but lowest concurrency
```

## 🔧 Transaction Implementation

### Node.js with Sequelize:
```javascript
const { Sequelize, DataTypes } = require('sequelize');

// Transaction with explicit commit/rollback
async function transferWithExplicitTransaction(fromId, toId, amount) {
  const transaction = await sequelize.transaction();
  
  try {
    const fromAccount = await Account.findByPk(fromId, { transaction });
    const toAccount = await Account.findByPk(toId, { transaction });
    
    if (fromAccount.balance < amount) {
      throw new Error('Insufficient funds');
    }
    
    await fromAccount.decrement('balance', { by: amount, transaction });
    await toAccount.increment('balance', { by: amount, transaction });
    
    await transaction.commit();
    return { success: true, message: 'Transfer completed' };
    
  } catch (error) {
    await transaction.rollback();
    return { success: false, error: error.message };
  }
}

// Transaction with automatic handling
async function transferWithAutoTransaction(fromId, toId, amount) {
  return await sequelize.transaction(async (transaction) => {
    const fromAccount = await Account.findByPk(fromId, { transaction });
    const toAccount = await Account.findByPk(toId, { transaction });
    
    if (fromAccount.balance < amount) {
      throw new Error('Insufficient funds');
    }
    
    await fromAccount.decrement('balance', { by: amount, transaction });
    await toAccount.increment('balance', { by: amount, transaction });
    
    return { success: true, message: 'Transfer completed' };
    // Automatic commit on success, rollback on error
  });
}
```

### Python with SQLAlchemy:
```python
from sqlalchemy.orm import sessionmaker
from contextlib import contextmanager

@contextmanager
def database_transaction():
    session = SessionLocal()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()

def transfer_money(from_account_id, to_account_id, amount):
    with database_transaction() as session:
        # Lock accounts to prevent concurrent modifications
        from_account = session.query(Account).filter(
            Account.id == from_account_id
        ).with_for_update().first()
        
        to_account = session.query(Account).filter(
            Account.id == to_account_id
        ).with_for_update().first()
        
        if from_account.balance < amount:
            raise ValueError("Insufficient funds")
        
        from_account.balance -= amount
        to_account.balance += amount
        
        # Transaction automatically committed on success
        return {"success": True, "message": "Transfer completed"}
```

## ⚡ Advanced Transaction Concepts

### 1. **Savepoints**
*Partial rollback within transactions*

```sql
BEGIN TRANSACTION;

INSERT INTO orders (id, customer_id) VALUES (1, 100);
SAVEPOINT order_created;

INSERT INTO order_items (order_id, product_id, quantity) VALUES (1, 1, 2);
SAVEPOINT items_added;

-- This might fail
UPDATE inventory SET quantity = quantity - 2 WHERE product_id = 1;

-- Rollback to savepoint if inventory update fails
ROLLBACK TO SAVEPOINT items_added;

-- Order still exists, but no items
COMMIT;
```

```javascript
async function complexOrder(orderData) {
  const transaction = await db.beginTransaction();
  
  try {
    // Create order
    const order = await createOrder(orderData, transaction);
    await transaction.setSavepoint('order_created');
    
    // Add items (might partially fail)
    for (const item of orderData.items) {
      try {
        await addOrderItem(order.id, item, transaction);
        await transaction.setSavepoint(`item_${item.id}_added`);
      } catch (error) {
        // Rollback to previous item
        await transaction.rollbackToSavepoint(`item_${item.id}_added`);
        console.log(`Failed to add item ${item.id}, continuing...`);
      }
    }
    
    await transaction.commit();
    return order;
    
  } catch (error) {
    await transaction.rollback();
    throw error;
  }
}
```

### 2. **Distributed Transactions (2PC)**
*Transactions across multiple databases*

```javascript
class TwoPhaseCommitCoordinator {
  constructor(participants) {
    this.participants = participants; // Array of database connections
  }
  
  async executeDistributedTransaction(operations) {
    const transactionId = this.generateTransactionId();
    
    try {
      // Phase 1: Prepare
      const prepareResults = await Promise.all(
        this.participants.map(async (participant, index) => {
          const transaction = await participant.beginTransaction();
          
          try {
            await operations[index](transaction);
            await transaction.prepare(); // Prepare to commit
            return { participant, transaction, ready: true };
          } catch (error) {
            await transaction.rollback();
            return { participant, transaction, ready: false, error };
          }
        })
      );
      
      // Check if all participants are ready
      const allReady = prepareResults.every(result => result.ready);
      
      if (allReady) {
        // Phase 2: Commit
        await Promise.all(
          prepareResults.map(result => result.transaction.commit())
        );
        return { success: true, message: 'Distributed transaction committed' };
      } else {
        // Phase 2: Abort
        await Promise.all(
          prepareResults
            .filter(result => result.ready)
            .map(result => result.transaction.rollback())
        );
        return { success: false, message: 'Distributed transaction aborted' };
      }
      
    } catch (error) {
      // Abort all transactions
      await this.abortAll(transactionId);
      throw error;
    }
  }
}
```

## 💡 Common Interview Questions & Answers

### Q1: "Explain the ACID properties with examples"
**Answer:**
"ACID ensures data integrity:
- **Atomicity**: All operations in a transaction succeed or all fail. Like a bank transfer - both debit and credit must complete.
- **Consistency**: Database stays valid. Constraints are enforced, like account balance can't go negative.
- **Isolation**: Concurrent transactions don't interfere. One transaction's uncommitted changes aren't visible to others.
- **Durability**: Committed changes survive system crashes through write-ahead logging."

### Q2: "What problems do different isolation levels solve?"
**Answer:**
"Each level prevents specific anomalies:
- **Read Uncommitted**: No protection, allows dirty reads
- **Read Committed**: Prevents dirty reads but allows non-repeatable reads
- **Repeatable Read**: Prevents dirty and non-repeatable reads but allows phantom reads
- **Serializable**: Prevents all anomalies but has lowest concurrency
Choose based on consistency needs vs. performance requirements."

### Q3: "How do you handle long-running transactions?"
**Answer:**
"Several strategies:
1. **Break into smaller transactions** when possible
2. **Use application-level locking** for coordination
3. **Implement timeout mechanisms** to prevent hanging
4. **Use optimistic locking** instead of pessimistic
5. **Consider eventual consistency** for non-critical operations
6. **Monitor and alert** on long-running transactions"

### Q4: "What's the CAP theorem and how does it relate to ACID?"
**Answer:**
"CAP theorem states you can only guarantee 2 of 3: Consistency, Availability, Partition tolerance. ACID focuses on consistency in single-node systems. In distributed systems, you often trade consistency for availability (eventual consistency) or partition tolerance. ACID is strong consistency, while many distributed systems use eventual consistency."

## 🚀 Best Practices

### 1. **Keep Transactions Short**
```javascript
// ❌ Bad - Long transaction
async function badTransaction() {
  const transaction = await db.beginTransaction();
  
  // This holds locks for too long
  await processLargeDataSet(transaction); // Takes 5 minutes
  await sendEmails(transaction); // Takes 2 minutes
  await generateReports(transaction); // Takes 3 minutes
  
  await transaction.commit();
}

// ✅ Good - Short transaction
async function goodTransaction() {
  // Do expensive operations outside transaction
  const processedData = await processLargeDataSet();
  const emailData = await prepareEmails();
  const reportData = await prepareReports();
  
  // Quick transaction for data changes only
  const transaction = await db.beginTransaction();
  try {
    await saveProcessedData(processedData, transaction);
    await transaction.commit();
  } catch (error) {
    await transaction.rollback();
    throw error;
  }
  
  // Continue with non-transactional operations
  await sendEmails(emailData);
  await generateReports(reportData);
}
```

### 2. **Handle Deadlocks**
```javascript
async function handleDeadlock(operation, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      if (error.code === 'DEADLOCK_DETECTED' && attempt < maxRetries) {
        const delay = Math.min(1000 * Math.pow(2, attempt), 5000); // Exponential backoff
        console.log(`Deadlock detected, retrying in ${delay}ms (attempt ${attempt})`);
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}

// Usage
await handleDeadlock(async () => {
  return await transferMoney(fromAccount, toAccount, amount);
});
```

## 🔧 Quick Checklist

- [ ] Understand all ACID properties with examples
- [ ] Know different isolation levels and their trade-offs
- [ ] Explain transaction implementation patterns
- [ ] Handle deadlocks and long-running transactions
- [ ] Understand distributed transactions
- [ ] Know when to use different consistency models

## 🎉 You're Transaction Ready!

**Key Message**: Transactions with ACID properties ensure **data integrity and consistency** in concurrent, failure-prone environments!

**Interview Golden Rule**: Always discuss the trade-offs between consistency, performance, and availability when explaining transaction choices.

Perfect! Now you can confidently discuss database transactions and ACID properties! 🔄 