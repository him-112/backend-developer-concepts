# REST API Design - Interview Guide 🚀

## 🎯 What You'll Learn
This guide will help you understand RESTful APIs in simple terms, perfect for interviews and real-world coding!

## 🌟 REST API in Simple Terms

### What is REST? (Explain it like I'm 5)
Think of REST API like a **restaurant menu system**:
- **Restaurant** = Your server/backend
- **Menu** = Your API endpoints  
- **Customer** = Client/frontend
- **Waiter** = HTTP methods (GET, POST, etc.)
- **Orders** = API requests
- **Food** = Data/responses

Just like a restaurant has standard ways to order food, REST APIs have standard ways to request and receive data!

### Key Interview Points ⭐
**What interviewers want to hear:**
1. "REST is an architectural style, not a protocol"
2. "It uses standard HTTP methods for operations"
3. "Resources are identified by URLs"
4. "It's stateless - each request is independent"
5. "It follows uniform interface principles"

## 🏗️ The 6 REST Principles (Interview Essentials)

### 1. **Client-Server Separation** 
*Think: Restaurant kitchen vs dining area*
- **Client**: Handles the user interface (like the dining area)
- **Server**: Manages data and business logic (like the kitchen)
- **Why it matters**: They can evolve independently

```
❌ Bad: Frontend directly accessing database
✅ Good: Frontend → API → Database
```

### 2. **Stateless Communication**
*Think: Each restaurant order is complete*
- Every request contains ALL info needed to process it
- Server doesn't remember previous requests
- **Interview tip**: "No server-side sessions in pure REST"

```javascript
// ❌ Bad (Stateful)
GET /next-page  // Server needs to remember current page

// ✅ Good (Stateless) 
GET /users?page=2&limit=10  // All info in request
```

### 3. **Cacheable**
*Think: Memorizing frequently ordered items*
- Responses can be stored temporarily
- Improves performance
- **Interview tip**: "Use HTTP cache headers like ETag, Cache-Control"

### 4. **Uniform Interface**
*Think: Standardized ordering process*
- Same rules apply everywhere
- Use standard HTTP methods
- Consistent URL patterns

### 5. **Layered System**
*Think: Restaurant with multiple floors and kitchens*
- You can add load balancers, CDNs, etc.
- Client doesn't need to know about internal layers

### 6. **Code on Demand (Optional)**
*Think: Restaurant giving you cooking instructions*
- Server can send executable code
- Rarely used (mainly JavaScript in browsers)

## 🛠️ HTTP Methods - The Restaurant Analogy

| Method | Restaurant Action | Code Example | When to Use |
|--------|------------------|--------------|-------------|
| **GET** | "Show me the menu" | `GET /users` | Getting data |
| **POST** | "Take my order" | `POST /users` | Creating new data |
| **PUT** | "Replace my entire order" | `PUT /users/123` | Full update |
| **PATCH** | "Change just my drink" | `PATCH /users/123` | Partial update |
| **DELETE** | "Cancel my order" | `DELETE /users/123` | Remove data |

## 📝 URL Design - Make it Beautiful!

### ✅ Great URL Design
```
GET /users                    # Get all users
GET /users/123               # Get specific user
GET /users/123/orders        # Get user's orders
GET /users/123/orders/456    # Get specific order

# Filtering & Pagination
GET /users?role=admin&page=2&limit=10&sort=name
GET /products?category=electronics&price_min=100&price_max=500
```

### ❌ Poor URL Design
```
GET /getUsers              # Don't use verbs
GET /user/123/getOrders    # Don't use verbs in paths
GET /users/123/order       # Use plural nouns
GET /User                  # Use lowercase
```

### 🎯 Interview Tips for URLs:
1. **Use nouns, not verbs** (`/users` not `/getUsers`)
2. **Use plural nouns** (`/users` not `/user`) 
3. **Hierarchical structure** (`/users/123/orders`)
4. **Query parameters for filtering** (`?role=admin&status=active`)

## 🚨 HTTP Status Codes - The Restaurant Feedback

### Success Stories (2xx) ✅
- **200 OK**: "Here's your food!" (successful GET)
- **201 Created**: "Order placed successfully!" (successful POST)
- **204 No Content**: "Order cancelled!" (successful DELETE)

### Client Mistakes (4xx) ❌
- **400 Bad Request**: "I don't understand your order"
- **401 Unauthorized**: "Please login first"
- **403 Forbidden**: "You're not allowed in VIP section"
- **404 Not Found**: "We don't have that dish"
- **409 Conflict**: "Table already reserved"

### Server Problems (5xx) 💥
- **500 Internal Server Error**: "Kitchen is on fire!"
- **503 Service Unavailable**: "Restaurant is closed"

## 💡 Common Interview Questions & Answers

### Q1: "What makes an API RESTful?"
**Perfect Answer:**
"A RESTful API follows REST architectural principles:
1. Uses standard HTTP methods (GET, POST, PUT, DELETE)
2. Resources are identified by URLs
3. Stateless communication
4. Uses proper HTTP status codes
5. Has a uniform interface"

### Q2: "What's the difference between PUT and PATCH?"
**Perfect Answer:**
"PUT replaces the entire resource - like rewriting a whole document. PATCH updates only specific fields - like editing just one paragraph. PUT is idempotent (same result every time), PATCH might not be."

```javascript
// PUT - Replace entire user
PUT /users/123
{
  "name": "John Doe",
  "email": "john@example.com",
  "role": "admin"
}

// PATCH - Update only name
PATCH /users/123
{
  "name": "John Smith"
}
```

### Q3: "How do you handle errors in REST APIs?"
**Perfect Answer:**
"Use appropriate HTTP status codes and include helpful error messages:

```javascript
// 400 Bad Request
{
  "error": "Validation failed",
  "message": "Email is required",
  "code": "VALIDATION_ERROR"
}

// 404 Not Found
{
  "error": "Resource not found",
  "message": "User with ID 123 does not exist"
}
```

### Q4: "How do you design REST APIs for scalability?"
**Perfect Answer:**
"Several strategies:
1. **Pagination**: `GET /users?page=1&limit=20`
2. **Filtering**: `GET /users?role=admin&status=active`
3. **Caching**: Use ETag headers and Cache-Control
4. **Rate limiting**: Prevent API abuse
5. **Stateless design**: No server-side sessions"

## 🎯 Real-World Example: E-commerce API

Let's design a simple e-commerce API:

```javascript
// Products
GET    /products              # List all products
GET    /products/123          # Get specific product
POST   /products              # Create new product (admin only)
PUT    /products/123          # Update entire product
PATCH  /products/123          # Update specific fields
DELETE /products/123          # Remove product

// Categories
GET    /categories            # List all categories
GET    /categories/electronics/products  # Products in category

// Shopping Cart
GET    /users/123/cart        # Get user's cart
POST   /users/123/cart/items  # Add item to cart
PUT    /users/123/cart/items/456  # Update item quantity
DELETE /users/123/cart/items/456  # Remove item from cart

// Orders
GET    /users/123/orders      # Get user's orders
POST   /users/123/orders      # Place new order
GET    /orders/789            # Get specific order (admin/owner only)
```

## 🚀 Best Practices for Interviews

### 1. **Always mention these in interviews:**
- "I always use proper HTTP status codes"
- "I design stateless APIs"
- "I use nouns in URLs, not verbs"
- "I implement proper error handling"
- "I consider caching and pagination"

### 2. **Show you understand security:**
```javascript
// Authentication
GET /users/profile
Headers: { Authorization: "Bearer jwt-token-here" }

// Input validation
POST /users
{
  "email": "valid@example.com",  // Validate email format
  "password": "StrongPass123!"   // Validate password strength
}
```

### 3. **Demonstrate real-world thinking:**
- "I'd add rate limiting to prevent abuse"
- "I'd use HTTPS in production"
- "I'd implement proper logging for debugging"
- "I'd consider API versioning for future changes"

## 🔧 Quick Interview Preparation Checklist

Before your interview, make sure you can explain:
- [ ] What REST stands for and its 6 principles
- [ ] HTTP methods and when to use each
- [ ] How to design good URLs
- [ ] Common HTTP status codes
- [ ] Difference between PUT and PATCH
- [ ] How to handle errors
- [ ] Basic security considerations
- [ ] Pagination and filtering strategies

## 🎉 You're Ready!

Remember: **REST is about being consistent, predictable, and easy to use**. When in doubt, think about how you'd want to use the API yourself!

**Interview tip**: Always give examples when explaining concepts. Interviewers love practical demonstrations of your knowledge!

# REST API Design

## 🔍 Overview
REST (Representational State Transfer) is an architectural style for designing networked applications. Understanding REST principles is essential for backend developers.

## 📋 REST Principles

### 1. Client-Server Architecture
- Separation of concerns between client and server
- Server manages data and business logic
- Client handles user interface and user experience

### 2. Stateless
- Each request must contain all information needed to process it
- Server doesn't store client context between requests
- Session state is kept on the client

### 3. Cacheable
- Responses should be cacheable when appropriate
- Improves performance and scalability
- Use proper HTTP headers for cache control

### 4. Uniform Interface
- Consistent API design across all endpoints
- Use standard HTTP methods and status codes
- Resource identification through URIs

### 5. Layered System
- Architecture can be composed of hierarchical layers
- Each layer doesn't know about layers beyond its immediate layer
- Allows for load balancers, gateways, proxies

### 6. Code on Demand (Optional)
- Server can extend client functionality by transferring executable code
- Rarely used in practice (JavaScript in web browsers)

## 🛣️ HTTP Methods & Their Usage

| Method | Purpose | Idempotent | Safe | Request Body | Response Body |
|--------|---------|------------|------|--------------|---------------|
| **GET** | Retrieve resource | ✅ | ✅ | ❌ | ✅ |
| **POST** | Create resource | ❌ | ❌ | ✅ | ✅ |
| **PUT** | Create/Update resource | ✅ | ❌ | ✅ | ✅ |
| **PATCH** | Partial update | ❌ | ❌ | ✅ | ✅ |
| **DELETE** | Remove resource | ✅ | ❌ | ❌ | ✅/❌ |
| **HEAD** | Get headers only | ✅ | ✅ | ❌ | ❌ |
| **OPTIONS** | Get allowed methods | ✅ | ✅ | ❌ | ✅ |

## 🗂️ Resource Naming Conventions

### Good Examples
```
# Collections (plural nouns)
GET /users
GET /orders
GET /products

# Specific resources
GET /users/123
GET /orders/456
GET /products/789

# Nested resources
GET /users/123/orders
GET /orders/456/items
GET /categories/electronics/products

# Query parameters for filtering
GET /users?role=admin&status=active
GET /products?category=electronics&price_min=100
```

### Bad Examples
```
# Avoid verbs in URLs
❌ GET /getUsers
❌ POST /createUser
❌ DELETE /deleteUser

# Avoid inconsistent naming
❌ GET /user (should be plural)
❌ GET /Users (avoid camelCase)
❌ GET /user_list (avoid underscores)
```

## 📊 HTTP Status Codes

### Success (2xx)
- **200 OK**: Request successful
- **201 Created**: Resource created successfully
- **202 Accepted**: Request accepted for processing
- **204 No Content**: Successful with no response body

### Client Errors (4xx)
- **400 Bad Request**: Invalid request syntax
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Access denied
- **404 Not Found**: Resource not found
- **409 Conflict**: Resource conflict
- **422 Unprocessable Entity**: Validation errors
- **429 Too Many Requests**: Rate limit exceeded

### Server Errors (5xx)
- **500 Internal Server Error**: Generic server error
- **502 Bad Gateway**: Invalid response from upstream
- **503 Service Unavailable**: Server temporarily unavailable
- **504 Gateway Timeout**: Upstream server timeout

## 💻 Code Examples

### User Management API
```javascript
const express = require('express');
const app = express();

// GET /users - Get all users
app.get('/users', async (req, res) => {
  try {
    const { page = 1, limit = 10, role, status } = req.query;
    
    const filters = {};
    if (role) filters.role = role;
    if (status) filters.status = status;
    
    const users = await User.find(filters)
      .limit(limit * 1)
      .skip((page - 1) * limit)
      .select('-password');
      
    const total = await User.countDocuments(filters);
    
    res.json({
      users,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total,
        pages: Math.ceil(total / limit)
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// GET /users/:id - Get specific user
app.get('/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id).select('-password');
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(user);
  } catch (error) {
    if (error.name === 'CastError') {
      return res.status(400).json({ error: 'Invalid user ID' });
    }
    res.status(500).json({ error: error.message });
  }
});

// POST /users - Create new user
app.post('/users', async (req, res) => {
  try {
    const { email, password, name, role } = req.body;
    
    // Validation
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(409).json({ error: 'Email already exists' });
    }
    
    const user = new User({
      email,
      password: await hashPassword(password),
      name,
      role: role || 'user'
    });
    
    const savedUser = await user.save();
    
    // Don't return password
    const { password: _, ...userResponse } = savedUser.toObject();
    
    res.status(201).json(userResponse);
  } catch (error) {
    if (error.name === 'ValidationError') {
      return res.status(422).json({ 
        error: 'Validation failed',
        details: error.errors
      });
    }
    res.status(500).json({ error: error.message });
  }
});

// PUT /users/:id - Update entire user
app.put('/users/:id', async (req, res) => {
  try {
    const { email, name, role, status } = req.body;
    
    const user = await User.findByIdAndUpdate(
      req.params.id,
      { email, name, role, status, updatedAt: new Date() },
      { new: true, runValidators: true }
    ).select('-password');
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(user);
  } catch (error) {
    if (error.name === 'ValidationError') {
      return res.status(422).json({ 
        error: 'Validation failed',
        details: error.errors
      });
    }
    res.status(500).json({ error: error.message });
  }
});

// PATCH /users/:id - Partial update
app.patch('/users/:id', async (req, res) => {
  try {
    const updates = req.body;
    const allowedUpdates = ['name', 'email', 'role', 'status'];
    const actualUpdates = {};
    
    // Filter allowed fields
    Object.keys(updates).forEach(key => {
      if (allowedUpdates.includes(key)) {
        actualUpdates[key] = updates[key];
      }
    });
    
    if (Object.keys(actualUpdates).length === 0) {
      return res.status(400).json({ error: 'No valid fields to update' });
    }
    
    actualUpdates.updatedAt = new Date();
    
    const user = await User.findByIdAndUpdate(
      req.params.id,
      actualUpdates,
      { new: true, runValidators: true }
    ).select('-password');
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// DELETE /users/:id - Delete user
app.delete('/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.status(204).send();
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

## 🔒 API Security Best Practices

### 1. Authentication & Authorization
```javascript
// JWT middleware
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }
  
  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid token' });
    }
    req.user = user;
    next();
  });
};

// Role-based authorization
const requireRole = (roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
};

// Usage
app.delete('/users/:id', authenticateToken, requireRole(['admin']), deleteUser);
```

### 2. Input Validation
```javascript
const { body, validationResult } = require('express-validator');

const validateUser = [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }).matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
  body('name').isLength({ min: 2, max: 50 }).trim().escape(),
  
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(422).json({
        error: 'Validation failed',
        details: errors.array()
      });
    }
    next();
  }
];

app.post('/users', validateUser, createUser);
```

### 3. Rate Limiting
```javascript
const rateLimit = require('express-rate-limit');

const createAccountLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour window
  max: 5, // limit each IP to 5 requests per windowMs
  message: {
    error: 'Too many accounts created from this IP, please try again after an hour.'
  }
});

app.post('/users', createAccountLimiter, createUser);
```

## 📄 API Documentation

### OpenAPI/Swagger Example
```yaml
paths:
  /users:
    get:
      summary: Get all users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 10
        - name: role
          in: query
          schema:
            type: string
            enum: [admin, user, moderator]
      responses:
        '200':
          description: List of users
          content:
            application/json:
              schema:
                type: object
                properties:
                  users:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
```

## ❓ Common Interview Questions

1. **Q: What makes an API RESTful?**
   A: Following REST principles: stateless communication, uniform interface, resource-based URLs, proper use of HTTP methods and status codes.

2. **Q: What's the difference between PUT and PATCH?**
   A: PUT replaces the entire resource, PATCH applies partial updates. PUT is idempotent, PATCH may not be.

3. **Q: How do you handle API versioning?**
   A: URL versioning (/api/v1/users), header versioning (Accept: application/vnd.api.v1+json), or query parameters (?version=1).

4. **Q: What's the difference between 401 and 403 status codes?**
   A: 401 means unauthenticated (need to login), 403 means unauthorized (authenticated but insufficient permissions).

5. **Q: How do you implement pagination in REST APIs?**
   A: Use offset/limit or cursor-based pagination with proper metadata in the response.

## 🚀 Advanced Concepts

### HATEOAS (Hypermedia as the Engine of Application State)
```json
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "_links": {
    "self": { "href": "/users/123" },
    "orders": { "href": "/users/123/orders" },
    "edit": { "href": "/users/123", "method": "PUT" },
    "delete": { "href": "/users/123", "method": "DELETE" }
  }
}
```

### Content Negotiation
```javascript
app.get('/users/:id', (req, res) => {
  const user = getUserById(req.params.id);
  
  res.format({
    'application/json': () => {
      res.json(user);
    },
    'application/xml': () => {
      res.send(toXML(user));
    },
    'text/html': () => {
      res.render('user', { user });
    },
    default: () => {
      res.status(406).send('Not Acceptable');
    }
  });
});
```

### Error Handling
```javascript
// Consistent error response format
const errorHandler = (err, req, res, next) => {
  const error = {
    message: err.message,
    status: err.status || 500,
    timestamp: new Date().toISOString(),
    path: req.path
  };
  
  if (process.env.NODE_ENV === 'development') {
    error.stack = err.stack;
  }
  
  res.status(error.status).json({ error });
};

app.use(errorHandler);
```

---

**Key Takeaway**: Good REST API design is about consistency, predictability, and following web standards. Focus on resource-oriented URLs, proper HTTP methods, meaningful status codes, and comprehensive documentation.

## 🎯 Richardson Maturity Model

The Richardson Maturity Model defines four levels of REST API maturity:

### Level 0: The Swamp of POX (Plain Old XML)
- Single endpoint for all operations
- Usually uses POST for everything
- RPC-style communication

```javascript
// Level 0 - Bad example
POST /api/endpoint
{
  "method": "getUser",
  "userId": 123
}

POST /api/endpoint
{
  "method": "createUser",
  "userData": {...}
}
```

### Level 1: Resources
- Multiple endpoints for different resources
- Still uses limited HTTP methods

```javascript
// Level 1
POST /api/users/getUser
POST /api/users/createUser
POST /api/orders/getOrder
```

### Level 2: HTTP Verbs
- Proper use of HTTP methods
- Correct status codes
- Most REST APIs operate at this level

```javascript
// Level 2
GET /api/users/123
POST /api/users
PUT /api/users/123
DELETE /api/users/123
```

### Level 3: Hypermedia Controls (HATEOAS)
- Self-describing API responses
- Links guide client interactions
- True REST implementation

```json
{
  "id": 123,
  "name": "John Doe",
  "status": "active",
  "_links": {
    "self": { "href": "/users/123" },
    "edit": { "href": "/users/123", "method": "PUT" },
    "deactivate": { "href": "/users/123/deactivate", "method": "POST" },
    "orders": { "href": "/users/123/orders" }
  }
}
```

## 🔄 REST vs GraphQL vs SOAP

| Aspect | REST | GraphQL | SOAP |
|--------|------|---------|------|
| **Protocol** | HTTP | HTTP/WebSocket | HTTP/SMTP/TCP |
| **Data Format** | JSON/XML | JSON | XML |
| **Flexibility** | Multiple endpoints | Single endpoint | Contract-based |
| **Caching** | HTTP caching | Complex | Limited |
| **Learning Curve** | Easy | Moderate | Hard |
| **Over-fetching** | Common | Eliminated | Common |
| **Real-time** | WebSockets/SSE | Subscriptions | WS-* standards |

### When to Use Each:

**REST**: 
- Simple CRUD operations
- Well-defined resource relationships
- Need HTTP caching
- Stateless operations

**GraphQL**:
- Complex data relationships
- Multiple client types (mobile, web)
- Need to minimize over-fetching
- Real-time subscriptions

**SOAP**:
- Enterprise systems
- Strict security requirements
- Formal contracts needed
- Legacy system integration

## 📊 Advanced Pagination Strategies

### 1. Offset-Based Pagination
```javascript
// Basic offset pagination
app.get('/users', async (req, res) => {
  const { page = 1, limit = 10 } = req.query;
  const offset = (page - 1) * limit;
  
  const users = await User.find()
    .skip(offset)
    .limit(parseInt(limit));
    
  const total = await User.countDocuments();
  
  res.json({
    data: users,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total,
      totalPages: Math.ceil(total / limit),
      hasNext: page * limit < total,
      hasPrev: page > 1
    }
  });
});
```

### 2. Cursor-Based Pagination (Better for large datasets)
```javascript
app.get('/users', async (req, res) => {
  const { cursor, limit = 10 } = req.query;
  
  const query = cursor ? { _id: { $gt: cursor } } : {};
  
  const users = await User.find(query)
    .sort({ _id: 1 })
    .limit(parseInt(limit) + 1); // +1 to check if there's next page
    
  const hasNext = users.length > limit;
  if (hasNext) users.pop(); // Remove extra record
  
  const nextCursor = hasNext ? users[users.length - 1]._id : null;
  
  res.json({
    data: users,
    pagination: {
      nextCursor,
      hasNext,
      limit: parseInt(limit)
    }
  });
});
```

### 3. Keyset Pagination (Time-based)
```javascript
app.get('/posts', async (req, res) => {
  const { since, until, limit = 20 } = req.query;
  
  let query = {};
  if (since) query.createdAt = { $gt: new Date(since) };
  if (until) query.createdAt = { ...query.createdAt, $lt: new Date(until) };
  
  const posts = await Post.find(query)
    .sort({ createdAt: -1 })
    .limit(parseInt(limit));
    
  res.json({
    data: posts,
    pagination: {
      since: posts.length > 0 ? posts[0].createdAt : null,
      until: posts.length > 0 ? posts[posts.length - 1].createdAt : null
    }
  });
});
```

## 🌐 CORS (Cross-Origin Resource Sharing)

### Understanding CORS
```javascript
const cors = require('cors');

// Basic CORS setup
app.use(cors({
  origin: ['http://localhost:3000', 'https://myapp.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));

// Custom CORS for specific routes
const corsOptions = {
  origin: (origin, callback) => {
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
    
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  }
};

app.use('/api/public', cors(corsOptions));
```

### Preflight Requests
```javascript
// Handle OPTIONS requests for preflight
app.options('*', (req, res) => {
  res.header('Access-Control-Allow-Origin', req.headers.origin);
  res.header('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE,OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Content-Type,Authorization');
  res.header('Access-Control-Max-Age', '86400'); // 24 hours
  res.status(200).send();
});
```

## 🏗️ Microservices Communication Patterns

### 1. Synchronous Communication
```javascript
// Service-to-service HTTP calls
const axios = require('axios');

class UserService {
  static async getUser(userId) {
    try {
      const response = await axios.get(`${USER_SERVICE_URL}/users/${userId}`, {
        timeout: 5000,
        headers: { 'Authorization': `Bearer ${serviceToken}` }
      });
      return response.data;
    } catch (error) {
      if (error.code === 'ECONNABORTED') {
        throw new Error('User service timeout');
      }
      throw error;
    }
  }
}
```

### 2. Asynchronous Communication (Event-Driven)
```javascript
// Event publisher
const EventEmitter = require('events');
const publishEvent = (eventType, data) => {
  eventBus.emit(eventType, {
    ...data,
    timestamp: new Date(),
    service: 'user-service'
  });
};

// User created event
app.post('/users', async (req, res) => {
  const user = await User.create(req.body);
  
  // Publish event for other services
  publishEvent('user.created', {
    userId: user._id,
    email: user.email,
    role: user.role
  });
  
  res.status(201).json(user);
});

// Event subscriber in another service
eventBus.on('user.created', async (eventData) => {
  // Create user profile, send welcome email, etc.
  await ProfileService.createProfile(eventData.userId);
  await EmailService.sendWelcomeEmail(eventData.email);
});
```

### 3. API Gateway Pattern
```javascript
// Simple API Gateway implementation
const httpProxy = require('http-proxy-middleware');

const services = {
  '/api/users': 'http://user-service:3001',
  '/api/orders': 'http://order-service:3002',
  '/api/payments': 'http://payment-service:3003'
};

// Route requests to appropriate services
Object.keys(services).forEach(path => {
  app.use(path, httpProxy({
    target: services[path],
    changeOrigin: true,
    pathRewrite: {
      [`^${path}`]: ''
    },
    onError: (err, req, res) => {
      res.status(503).json({ 
        error: 'Service temporarily unavailable' 
      });
    }
  }));
});
```

## 🧪 API Testing Strategies

### 1. Unit Testing API Endpoints
```javascript
const request = require('supertest');
const app = require('../app');

describe('User API', () => {
  describe('GET /users/:id', () => {
    it('should return user when valid id provided', async () => {
      const userId = '507f1f77bcf86cd799439011';
      
      const response = await request(app)
        .get(`/users/${userId}`)
        .set('Authorization', `Bearer ${validToken}`)
        .expect(200);
        
      expect(response.body).toHaveProperty('_id', userId);
      expect(response.body).toHaveProperty('email');
      expect(response.body).not.toHaveProperty('password');
    });
    
    it('should return 404 when user not found', async () => {
      const response = await request(app)
        .get('/users/507f1f77bcf86cd799439999')
        .set('Authorization', `Bearer ${validToken}`)
        .expect(404);
        
      expect(response.body).toHaveProperty('error', 'User not found');
    });
    
    it('should return 401 when no token provided', async () => {
      await request(app)
        .get('/users/507f1f77bcf86cd799439011')
        .expect(401);
    });
  });
});
```

### 2. Integration Testing
```javascript
describe('User Registration Flow', () => {
  it('should create user, send email, and create profile', async () => {
    const userData = {
      email: 'test@example.com',
      password: 'SecurePass123',
      name: 'Test User'
    };
    
    // Create user
    const response = await request(app)
      .post('/users')
      .send(userData)
      .expect(201);
      
    const userId = response.body._id;
    
    // Verify user was created
    const user = await User.findById(userId);
    expect(user).toBeTruthy();
    
    // Verify email was queued
    expect(emailQueue.jobs).toHaveLength(1);
    
    // Verify profile was created
    const profile = await Profile.findOne({ userId });
    expect(profile).toBeTruthy();
  });
});
```

### 3. Contract Testing (API Documentation Testing)
```javascript
const swaggerJSDoc = require('swagger-jsdoc');
const { validate } = require('swagger-parser');

describe('API Documentation', () => {
  it('should have valid OpenAPI specification', async () => {
    const spec = swaggerJSDoc(swaggerOptions);
    await expect(validate(spec)).resolves.toBeTruthy();
  });
  
  it('should match actual API responses with schema', async () => {
    const response = await request(app)
      .get('/users/507f1f77bcf86cd799439011')
      .set('Authorization', `Bearer ${validToken}`);
      
    // Validate response against OpenAPI schema
    const isValid = validateResponseSchema(response.body, 'User');
    expect(isValid).toBe(true);
  });
});
```

## ⚡ API Performance Optimization

### 1. Response Compression
```javascript
const compression = require('compression');

app.use(compression({
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  },
  level: 6, // Compression level (1-9)
  threshold: 1024 // Only compress responses > 1KB
}));
```

### 2. API Response Caching
```javascript
const redis = require('redis');
const client = redis.createClient();

const cacheMiddleware = (duration = 300) => {
  return async (req, res, next) => {
    const key = `cache:${req.method}:${req.originalUrl}`;
    
    try {
      const cached = await client.get(key);
      if (cached) {
        return res.json(JSON.parse(cached));
      }
      
      // Store original res.json
      const originalJson = res.json;
      res.json = function(data) {
        // Cache the response
        client.setex(key, duration, JSON.stringify(data));
        return originalJson.call(this, data);
      };
      
      next();
    } catch (error) {
      next();
    }
  };
};

// Usage
app.get('/users', cacheMiddleware(600), getUsers); // Cache for 10 minutes
```

### 3. Database Query Optimization
```javascript
// Use projection to limit fields
app.get('/users', async (req, res) => {
  const users = await User.find(
    {},
    'name email role createdAt' // Only return specific fields
  ).lean(); // Return plain objects, not Mongoose documents
  
  res.json(users);
});

// Use aggregation for complex queries
app.get('/user-stats', async (req, res) => {
  const stats = await User.aggregate([
    { $match: { status: 'active' } },
    { $group: {
      _id: '$role',
      count: { $sum: 1 },
      avgAge: { $avg: '$age' }
    }},
    { $sort: { count: -1 } }
  ]);
  
  res.json(stats);
});
```

### 4. Connection Pooling
```javascript
const mongoose = require('mongoose');

// Optimize connection pool
mongoose.connect(process.env.MONGODB_URI, {
  maxPoolSize: 10, // Maximum number of connections
  minPoolSize: 2,  // Minimum number of connections
  maxIdleTimeMS: 30000, // Close connections after 30s of inactivity
  serverSelectionTimeoutMS: 5000, // How long to try connecting
  bufferMaxEntries: 0 // Disable mongoose buffering
});
```

### 5. Request/Response Size Optimization
```javascript
// Limit request body size
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ limit: '10mb', extended: true }));

// Field selection middleware
const selectFields = (req, res, next) => {
  if (req.query.fields) {
    req.selectFields = req.query.fields.split(',').join(' ');
  }
  next();
};

app.get('/users', selectFields, async (req, res) => {
  const query = User.find();
  
  if (req.selectFields) {
    query.select(req.selectFields);
  }
  
  const users = await query.exec();
  res.json(users);
});
```

## 🔐 Idempotency Deep Dive

### Understanding Idempotency
```javascript
// Idempotent key middleware
const idempotencyMiddleware = (req, res, next) => {
  const idempotencyKey = req.headers['idempotency-key'];
  
  if (['POST', 'PATCH'].includes(req.method) && !idempotencyKey) {
    return res.status(400).json({
      error: 'Idempotency-Key header required for this operation'
    });
  }
  
  req.idempotencyKey = idempotencyKey;
  next();
};

// Payment endpoint with idempotency
app.post('/payments', idempotencyMiddleware, async (req, res) => {
  const { idempotencyKey } = req;
  
  // Check if this request was already processed
  const existingPayment = await Payment.findOne({ idempotencyKey });
  if (existingPayment) {
    return res.status(200).json(existingPayment);
  }
  
  try {
    const payment = await processPayment(req.body);
    payment.idempotencyKey = idempotencyKey;
    await payment.save();
    
    res.status(201).json(payment);
  } catch (error) {
    // Store failed attempts to prevent retries with same key
    await FailedPayment.create({
      idempotencyKey,
      error: error.message,
      requestData: req.body
    });
    
    res.status(400).json({ error: error.message });
  }
});
```

## 🎯 Advanced Interview Questions & Answers

### Technical Scenarios

**Q: How would you design a RESTful API for a social media platform where users can follow each other?**

A: 
```javascript
// User relationships
GET /users/:id/followers        // Get user's followers
GET /users/:id/following        // Get who user follows
POST /users/:id/follow          // Follow a user
DELETE /users/:id/follow        // Unfollow a user

// Alternative resource-based approach
GET /relationships?follower=123&following=456  // Check relationship
POST /relationships             // Create relationship
DELETE /relationships/:id       // Remove relationship

// Feed endpoints
GET /users/:id/feed            // User's timeline
GET /users/:id/posts           // User's own posts
```

**Q: How do you handle eventual consistency in distributed systems?**

A: Implement saga patterns, event sourcing, and compensation actions:

```javascript
// Order processing saga
class OrderSaga {
  async processOrder(orderData) {
    const sagaId = generateId();
    
    try {
      // Step 1: Reserve inventory
      const reservation = await inventoryService.reserve(orderData.items);
      await this.saveStep(sagaId, 'inventory_reserved', reservation);
      
      // Step 2: Process payment
      const payment = await paymentService.charge(orderData.payment);
      await this.saveStep(sagaId, 'payment_processed', payment);
      
      // Step 3: Create order
      const order = await orderService.create(orderData);
      await this.saveStep(sagaId, 'order_created', order);
      
      return order;
    } catch (error) {
      // Compensate previous steps
      await this.compensate(sagaId);
      throw error;
    }
  }
  
  async compensate(sagaId) {
    const steps = await this.getSteps(sagaId);
    
    for (const step of steps.reverse()) {
      switch (step.type) {
        case 'inventory_reserved':
          await inventoryService.release(step.data.reservationId);
          break;
        case 'payment_processed':
          await paymentService.refund(step.data.paymentId);
          break;
      }
    }
  }
}
```

**Q: How do you implement API rate limiting at scale?**

A: Use distributed rate limiting with Redis:

```javascript
const redis = require('redis');
const client = redis.createClient();

class DistributedRateLimiter {
  async isAllowed(key, limit, windowMs) {
    const multi = client.multi();
    const now = Date.now();
    const window = Math.floor(now / windowMs);
    const redisKey = `rate_limit:${key}:${window}`;
    
    multi.incr(redisKey);
    multi.expire(redisKey, Math.ceil(windowMs / 1000));
    
    const results = await multi.exec();
    const current = results[0][1];
    
    return {
      allowed: current <= limit,
      current,
      limit,
      resetTime: (window + 1) * windowMs
    };
  }
}

// Sliding window rate limiter
class SlidingWindowRateLimiter {
  async isAllowed(key, limit, windowMs) {
    const now = Date.now();
    const cutoff = now - windowMs;
    
    const multi = client.multi();
    multi.zremrangebyscore(`sliding:${key}`, 0, cutoff);
    multi.zcard(`sliding:${key}`);
    multi.zadd(`sliding:${key}`, now, `${now}-${Math.random()}`);
    multi.expire(`sliding:${key}`, Math.ceil(windowMs / 1000));
    
    const results = await multi.exec();
    const current = results[1][1];
    
    return {
      allowed: current < limit,
      current: current + 1,
      limit
    };
  }
}
```

**Q: How do you handle file uploads in a RESTful API?**

A: 
```javascript
const multer = require('multer');
const AWS = require('aws-sdk');

// Configure multer for multipart uploads
const upload = multer({
  limits: {
    fileSize: 10 * 1024 * 1024, // 10MB limit
    files: 5 // Max 5 files
  },
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
    
    if (allowedTypes.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error('Invalid file type'), false);
    }
  }
});

// Direct upload endpoint
app.post('/upload', upload.array('files'), async (req, res) => {
  try {
    const uploadPromises = req.files.map(async (file) => {
      const key = `uploads/${Date.now()}-${file.originalname}`;
      
      const params = {
        Bucket: process.env.S3_BUCKET,
        Key: key,
        Body: file.buffer,
        ContentType: file.mimetype,
        ACL: 'public-read'
      };
      
      const result = await s3.upload(params).promise();
      
      return {
        originalName: file.originalname,
        url: result.Location,
        size: file.size,
        type: file.mimetype
      };
    });
    
    const files = await Promise.all(uploadPromises);
    res.json({ files });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Presigned URL approach (better for large files)
app.post('/upload-url', async (req, res) => {
  const { fileName, fileType } = req.body;
  
  const key = `uploads/${Date.now()}-${fileName}`;
  const params = {
    Bucket: process.env.S3_BUCKET,
    Key: key,
    ContentType: fileType,
    Expires: 300 // 5 minutes
  };
  
  const uploadUrl = s3.getSignedUrl('putObject', params);
  
  res.json({
    uploadUrl,
    key,
    publicUrl: `https://${process.env.S3_BUCKET}.s3.amazonaws.com/${key}`
  });
});
```

**Q: How do you implement real-time features in a REST API?**

A: Combine REST with WebSockets or Server-Sent Events:

```javascript
// WebSocket integration
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

// Connection management
const connections = new Map();

wss.on('connection', (ws, req) => {
  const userId = getUserIdFromToken(req.headers.authorization);
  connections.set(userId, ws);
  
  ws.on('close', () => {
    connections.delete(userId);
  });
});

// REST endpoint that triggers real-time updates
app.post('/messages', async (req, res) => {
  const message = await Message.create({
    ...req.body,
    senderId: req.user.id
  });
  
  // Send real-time notification
  const recipientConnection = connections.get(req.body.recipientId);
  if (recipientConnection) {
    recipientConnection.send(JSON.stringify({
      type: 'new_message',
      data: message
    }));
  }
  
  res.status(201).json(message);
});

// Server-Sent Events approach
app.get('/events', (req, res) => {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
    'Access-Control-Allow-Origin': '*'
  });
  
  const userId = req.user.id;
  
  // Send periodic updates
  const interval = setInterval(() => {
    res.write(`data: ${JSON.stringify({ 
      type: 'heartbeat', 
      timestamp: Date.now() 
    })}\n\n`);
  }, 30000);
  
  // Clean up on disconnect
  req.on('close', () => {
    clearInterval(interval);
  });
});
```

## 📚 Key Concepts Summary

### REST Fundamentals
- **Stateless**: Each request contains all necessary information
- **Resource-based**: URLs represent resources, not actions
- **HTTP Methods**: Use appropriate verbs (GET, POST, PUT, DELETE, PATCH)
- **Status Codes**: Return meaningful HTTP status codes
- **Idempotency**: GET, PUT, DELETE should be idempotent

### Best Practices Checklist
- ✅ Use nouns for resources, not verbs
- ✅ Implement proper error handling and consistent error responses
- ✅ Version your APIs (prefer URL versioning for simplicity)
- ✅ Use HTTPS in production
- ✅ Implement rate limiting and request validation
- ✅ Document your API thoroughly (OpenAPI/Swagger)
- ✅ Use pagination for large datasets
- ✅ Implement proper authentication and authorization
- ✅ Follow semantic versioning for API changes
- ✅ Monitor API performance and usage metrics

### Performance Optimization
- Use response compression
- Implement caching strategies
- Optimize database queries
- Use connection pooling
- Implement request/response size limits
- Use CDN for static assets
- Consider API gateway for microservices

### Security Essentials
- Validate all inputs
- Use HTTPS/TLS
- Implement proper authentication (JWT/OAuth)
- Use CORS appropriately
- Implement rate limiting
- Never expose sensitive data
- Use security headers
- Log security events

---

**Final Key Takeaway**: REST is about building scalable, maintainable APIs that follow web standards. Focus on consistency, proper use of HTTP, comprehensive documentation, and always consider security and performance from the start. 