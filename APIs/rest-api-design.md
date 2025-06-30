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