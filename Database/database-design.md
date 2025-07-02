# Database Design - Interview Guide 🗄️

## 🎯 What You'll Learn
Master database design principles to create efficient, scalable data architectures!

## 🌟 Database Design Fundamentals

### What is Good Database Design?
Think of database design like **organizing a library**:
- **Tables** = Different sections (Fiction, Non-fiction, etc.)
- **Columns** = Book properties (Title, Author, ISBN)
- **Relationships** = How books connect (Same author, series)
- **Indexes** = Card catalog for quick lookups

### Key Interview Points ⭐
1. "Normalization reduces redundancy"
2. "Denormalization can improve performance"
3. "Indexes speed up queries but slow down writes"
4. "Consider access patterns when designing"
5. "ACID properties ensure data integrity"

## 🏗️ Database Design Process

### 1. **Requirements Analysis**
```
Ask these questions:
- What data do we need to store?
- How will data be accessed?
- What are the relationships?
- What are the performance requirements?
- How much data growth is expected?
```

### 2. **Conceptual Design (ER Diagram)**
```
Entities: User, Post, Comment
Relationships:
- User (1) → (M) Post
- Post (1) → (M) Comment
- User (1) → (M) Comment
```

### 3. **Logical Design (Tables)**
```sql
-- Users table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Posts table
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  title VARCHAR(200) NOT NULL,
  content TEXT,
  status VARCHAR(20) DEFAULT 'published',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Comments table
CREATE TABLE comments (
  id SERIAL PRIMARY KEY,
  post_id INTEGER REFERENCES posts(id),
  user_id INTEGER REFERENCES users(id),
  content TEXT NOT NULL,
  parent_id INTEGER REFERENCES comments(id), -- For nested comments
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 📐 Normalization & Denormalization

### Normalization Forms:

**1NF (First Normal Form):**
- Eliminate duplicate columns
- Each cell contains single value

**2NF (Second Normal Form):**
- Must be 1NF
- Remove partial dependencies

**3NF (Third Normal Form):**
- Must be 2NF
- Remove transitive dependencies

```sql
-- ❌ Unnormalized (violates 1NF)
CREATE TABLE orders_bad (
  id INT,
  customer_name VARCHAR(100),
  products VARCHAR(500), -- Multiple products in one field
  prices VARCHAR(200)    -- Multiple prices in one field
);

-- ✅ Normalized (3NF)
CREATE TABLE customers (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER REFERENCES customers(id),
  order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  price DECIMAL(10,2) NOT NULL
);

CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER REFERENCES orders(id),
  product_id INTEGER REFERENCES products(id),
  quantity INTEGER NOT NULL,
  price DECIMAL(10,2) NOT NULL -- Price at time of order
);
```

### When to Denormalize:
```sql
-- Denormalized for performance (common in analytics)
CREATE TABLE user_stats (
  user_id INTEGER PRIMARY KEY,
  username VARCHAR(50),
  total_posts INTEGER DEFAULT 0,
  total_comments INTEGER DEFAULT 0,
  last_activity TIMESTAMP,
  -- Redundant data for faster queries
);

-- Update triggers to maintain consistency
CREATE OR REPLACE FUNCTION update_user_stats()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE user_stats 
  SET total_posts = total_posts + 1,
      last_activity = CURRENT_TIMESTAMP
  WHERE user_id = NEW.user_id;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

## 🔑 Indexing Strategy

### Types of Indexes:
```sql
-- Primary index (automatically created)
CREATE TABLE users (
  id SERIAL PRIMARY KEY -- Automatically indexed
);

-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index
CREATE INDEX idx_posts_user_status ON posts(user_id, status);

-- Partial index
CREATE INDEX idx_active_users ON users(id) WHERE status = 'active';

-- Functional index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
```

### Index Best Practices:
```sql
-- ✅ Good: Index on frequently queried columns
CREATE INDEX idx_orders_created_at ON orders(created_at);
CREATE INDEX idx_users_username ON users(username);

-- ❌ Bad: Over-indexing (slows down writes)
CREATE INDEX idx_users_every_column ON users(id, username, email, created_at, updated_at);

-- ✅ Good: Composite index with proper order
CREATE INDEX idx_posts_user_date ON posts(user_id, created_at DESC);
-- Supports: WHERE user_id = ? ORDER BY created_at DESC
```

## 🔄 Relationships & Constraints

### One-to-Many:
```sql
-- User has many posts
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(200) NOT NULL
);
```

### Many-to-Many:
```sql
-- Users can follow many users
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE user_follows (
  follower_id INTEGER REFERENCES users(id),
  following_id INTEGER REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (follower_id, following_id),
  CHECK (follower_id != following_id) -- Can't follow yourself
);
```

### Self-Referencing:
```sql
-- Hierarchical comments
CREATE TABLE comments (
  id SERIAL PRIMARY KEY,
  post_id INTEGER REFERENCES posts(id),
  parent_id INTEGER REFERENCES comments(id), -- Self-reference
  content TEXT NOT NULL,
  level INTEGER DEFAULT 0 -- For easy querying
);
```

## 💡 Common Interview Questions & Answers

### Q1: "What's the difference between normalization and denormalization?"
**Answer:** "Normalization reduces data redundancy by organizing data into separate tables, following normal forms (1NF, 2NF, 3NF). Denormalization intentionally introduces redundancy to improve query performance, often used in read-heavy applications or data warehouses."

### Q2: "How do you design a database for a social media platform?"
**Answer:** "Key entities: Users, Posts, Comments, Likes, Follows. Relationships: User (1:M) Posts, Post (1:M) Comments, User (M:M) Follows. Consider indexes on user_id, created_at for timeline queries. Denormalize follower counts for performance."

### Q3: "When would you use a composite index?"
**Answer:** "When queries filter by multiple columns together. Order matters - put the most selective column first. For example, `(user_id, created_at)` for queries like 'get posts by user ordered by date'."

### Q4: "How do you handle soft deletes?"
**Answer:** "Add `deleted_at` timestamp column, set to NULL for active records. Create partial index `WHERE deleted_at IS NULL` for performance. Update queries to filter out deleted records."

```sql
-- Soft delete implementation
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP NULL;

-- Index for active users only
CREATE INDEX idx_active_users ON users(id) WHERE deleted_at IS NULL;

-- Queries filter out deleted
SELECT * FROM users WHERE deleted_at IS NULL;
```

## 🎯 Design Patterns

### 1. **Audit Trail Pattern:**
```sql
CREATE TABLE user_audit (
  id SERIAL PRIMARY KEY,
  user_id INTEGER,
  action VARCHAR(50), -- INSERT, UPDATE, DELETE
  old_values JSONB,
  new_values JSONB,
  changed_by INTEGER,
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2. **Polymorphic Associations:**
```sql
-- Comments can belong to posts or other comments
CREATE TABLE comments (
  id SERIAL PRIMARY KEY,
  commentable_id INTEGER NOT NULL,
  commentable_type VARCHAR(50) NOT NULL, -- 'post' or 'comment'
  content TEXT NOT NULL,
  INDEX idx_commentable (commentable_id, commentable_type)
);
```

### 3. **Event Sourcing:**
```sql
CREATE TABLE events (
  id SERIAL PRIMARY KEY,
  aggregate_id UUID NOT NULL,
  event_type VARCHAR(100) NOT NULL,
  event_data JSONB NOT NULL,
  version INTEGER NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(aggregate_id, version)
);
```

## 🚀 Scalability Considerations

### Partitioning:
```sql
-- Time-based partitioning for logs
CREATE TABLE logs (
  id SERIAL,
  user_id INTEGER,
  action VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) PARTITION BY RANGE (created_at);

-- Create monthly partitions
CREATE TABLE logs_2024_01 PARTITION OF logs
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE logs_2024_02 PARTITION OF logs
FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

### Read Replicas:
```sql
-- Master-slave setup
-- Write operations go to master
-- Read operations can go to slaves

-- Connection routing in application
const masterDB = new Database('master-host');
const slaveDB = new Database('slave-host');

function writeOperation(query) {
  return masterDB.query(query);
}

function readOperation(query) {
  return slaveDB.query(query);
}
```

## 🔧 Best Practices Checklist

### **Design Phase:**
- [ ] Understand requirements thoroughly
- [ ] Identify entities and relationships
- [ ] Choose appropriate data types
- [ ] Apply normalization principles
- [ ] Consider denormalization for performance

### **Implementation Phase:**
- [ ] Create proper indexes
- [ ] Add foreign key constraints
- [ ] Implement data validation
- [ ] Plan for data growth
- [ ] Consider backup/recovery strategy

### **Optimization Phase:**
- [ ] Monitor query performance
- [ ] Analyze slow queries
- [ ] Optimize indexes
- [ ] Consider partitioning
- [ ] Plan for scaling

## 🎯 Interview Success Tips

### **Always Mention:**
- "Design based on access patterns"
- "Balance between normalization and performance"
- "Indexes improve reads but slow writes"
- "Consider data growth and scalability"
- "Maintain data integrity with constraints"

### **Show Real-World Knowledge:**
- "Use EXPLAIN to analyze query plans"
- "Monitor database metrics"
- "Consider connection pooling"
- "Plan for backup and disaster recovery"

## 🎉 You're Database Design Ready!

**Key Message**: Good database design is about **balancing normalization, performance, and scalability** based on your application's specific needs!

**Interview Golden Rule**: Always ask about requirements first, then explain your design decisions and trade-offs.

Perfect! Now you can confidently discuss database design! 🗄️ 