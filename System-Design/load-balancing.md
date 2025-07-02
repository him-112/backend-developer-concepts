# Load Balancing - System Design Interview Guide ⚖️

## 🎯 What You'll Learn
Master load balancing concepts to distribute traffic efficiently across multiple servers!

## 🌟 Load Balancing Fundamentals

### What is Load Balancing?
Think of load balancing like a **restaurant host**:
- **Host** = Load balancer
- **Tables** = Servers
- **Customers** = Incoming requests
- **Menu** = Available resources

The host ensures customers are seated at available tables, preventing any table from being overwhelmed!

### Key Interview Points ⭐
1. "Distributes incoming requests across multiple servers"
2. "Improves availability and fault tolerance"
3. "Different algorithms for different use cases"
4. "Can operate at different network layers"
5. "Essential for horizontal scaling"

## 🏗️ Load Balancing Algorithms

### 1. **Round Robin**
*Each server gets requests in turn*

```javascript
class RoundRobinBalancer {
  constructor(servers) {
    this.servers = servers;
    this.currentIndex = 0;
  }
  
  getServer() {
    const server = this.servers[this.currentIndex];
    this.currentIndex = (this.currentIndex + 1) % this.servers.length;
    return server;
  }
}

// Usage
const balancer = new RoundRobinBalancer(['server1', 'server2', 'server3']);
console.log(balancer.getServer()); // server1
console.log(balancer.getServer()); // server2
console.log(balancer.getServer()); // server3
console.log(balancer.getServer()); // server1 (cycles back)
```

### 2. **Weighted Round Robin**
*Servers with higher weights get more requests*

```javascript
class WeightedRoundRobinBalancer {
  constructor(servers) {
    this.servers = servers; // [{ name: 'server1', weight: 3 }, ...]
    this.currentWeights = servers.map(s => ({ ...s, currentWeight: 0 }));
  }
  
  getServer() {
    let selected = null;
    let maxWeight = 0;
    
    // Find server with highest current weight
    for (let server of this.currentWeights) {
      server.currentWeight += server.weight;
      
      if (server.currentWeight > maxWeight) {
        maxWeight = server.currentWeight;
        selected = server;
      }
    }
    
    // Decrease selected server's current weight
    selected.currentWeight -= this.getTotalWeight();
    return selected.name;
  }
  
  getTotalWeight() {
    return this.servers.reduce((sum, s) => sum + s.weight, 0);
  }
}

// Powerful servers get more requests
const servers = [
  { name: 'server1', weight: 3 }, // Gets 3x more requests
  { name: 'server2', weight: 2 }, // Gets 2x more requests  
  { name: 'server3', weight: 1 }  // Gets 1x requests
];
```

### 3. **Least Connections**
*Route to server with fewest active connections*

```javascript
class LeastConnectionsBalancer {
  constructor(servers) {
    this.servers = servers.map(name => ({
      name,
      connections: 0,
      isHealthy: true
    }));
  }
  
  getServer() {
    // Filter healthy servers only
    const healthyServers = this.servers.filter(s => s.isHealthy);
    
    if (healthyServers.length === 0) {
      throw new Error('No healthy servers available');
    }
    
    // Find server with minimum connections
    const selected = healthyServers.reduce((min, server) => 
      server.connections < min.connections ? server : min
    );
    
    selected.connections++;
    return selected.name;
  }
  
  releaseConnection(serverName) {
    const server = this.servers.find(s => s.name === serverName);
    if (server && server.connections > 0) {
      server.connections--;
    }
  }
}
```

### 4. **IP Hash**
*Route based on client IP for session persistence*

```javascript
class IPHashBalancer {
  constructor(servers) {
    this.servers = servers;
  }
  
  getServer(clientIP) {
    // Simple hash function
    const hash = this.hashCode(clientIP);
    const index = Math.abs(hash) % this.servers.length;
    return this.servers[index];
  }
  
  hashCode(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32-bit integer
    }
    return hash;
  }
}

// Same client IP always goes to same server
const balancer = new IPHashBalancer(['server1', 'server2', 'server3']);
console.log(balancer.getServer('192.168.1.100')); // Always same server
```

## 🎯 Types of Load Balancers

### 1. **Layer 4 (Transport Layer)**
*Routes based on IP and port*

```nginx
# Nginx Layer 4 Load Balancing
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

server {
    listen 80;
    proxy_pass backend;
}
```

**Pros:**
- Fast (no packet inspection)
- Protocol agnostic
- Lower latency

**Cons:**
- No content-based routing
- Limited health checks

### 2. **Layer 7 (Application Layer)**
*Routes based on application data (HTTP headers, URLs)*

```nginx
# Nginx Layer 7 Load Balancing
upstream api_servers {
    server api1.example.com;
    server api2.example.com;
}

upstream web_servers {
    server web1.example.com;
    server web2.example.com;
}

server {
    listen 80;
    
    location /api/ {
        proxy_pass http://api_servers;
    }
    
    location / {
        proxy_pass http://web_servers;
    }
}
```

**Pros:**
- Content-based routing
- Advanced health checks
- SSL termination

**Cons:**
- Higher latency
- More resource intensive

## 🔧 Load Balancer Implementation

### Express.js Load Balancer:
```javascript
const express = require('express');
const httpProxy = require('http-proxy-middleware');

class LoadBalancer {
  constructor() {
    this.servers = [
      'http://localhost:3001',
      'http://localhost:3002',
      'http://localhost:3003'
    ];
    this.current = 0;
    this.app = express();
    this.setupRoutes();
  }
  
  setupRoutes() {
    this.app.use('*', (req, res, next) => {
      const target = this.getNextServer();
      
      const proxy = httpProxy({
        target,
        changeOrigin: true,
        onError: (err, req, res) => {
          console.error(`Error proxying to ${target}:`, err.message);
          this.handleServerError(target, req, res);
        }
      });
      
      proxy(req, res, next);
    });
  }
  
  getNextServer() {
    const server = this.servers[this.current];
    this.current = (this.current + 1) % this.servers.length;
    return server;
  }
  
  handleServerError(failedServer, req, res) {
    // Remove failed server temporarily
    console.log(`Server ${failedServer} is down, trying next...`);
    
    // Try next server
    const nextServer = this.getNextServer();
    const retryProxy = httpProxy({
      target: nextServer,
      changeOrigin: true
    });
    
    retryProxy(req, res);
  }
}

const lb = new LoadBalancer();
lb.app.listen(3000, () => {
  console.log('Load balancer running on port 3000');
});
```

## 🏥 Health Checks

### Active Health Checks:
```javascript
class HealthCheckBalancer {
  constructor(servers) {
    this.servers = servers.map(url => ({
      url,
      isHealthy: true,
      failures: 0
    }));
    
    this.startHealthChecks();
  }
  
  startHealthChecks() {
    setInterval(() => {
      this.servers.forEach(server => this.checkHealth(server));
    }, 30000); // Check every 30 seconds
  }
  
  async checkHealth(server) {
    try {
      const response = await fetch(`${server.url}/health`, {
        timeout: 5000
      });
      
      if (response.ok) {
        server.isHealthy = true;
        server.failures = 0;
        console.log(`✅ ${server.url} is healthy`);
      } else {
        this.handleFailure(server);
      }
    } catch (error) {
      this.handleFailure(server);
    }
  }
  
  handleFailure(server) {
    server.failures++;
    console.log(`❌ ${server.url} failed health check (${server.failures}/3)`);
    
    if (server.failures >= 3) {
      server.isHealthy = false;
      console.log(`🔴 Marking ${server.url} as unhealthy`);
    }
  }
  
  getHealthyServers() {
    return this.servers.filter(s => s.isHealthy);
  }
}
```

### Passive Health Checks:
```javascript
class PassiveHealthCheck {
  constructor() {
    this.serverStats = new Map();
  }
  
  recordResponse(serverUrl, success, responseTime) {
    if (!this.serverStats.has(serverUrl)) {
      this.serverStats.set(serverUrl, {
        requests: 0,
        failures: 0,
        avgResponseTime: 0
      });
    }
    
    const stats = this.serverStats.get(serverUrl);
    stats.requests++;
    
    if (!success) {
      stats.failures++;
    }
    
    // Update average response time
    stats.avgResponseTime = (stats.avgResponseTime + responseTime) / 2;
    
    // Mark unhealthy if error rate > 50%
    const errorRate = stats.failures / stats.requests;
    if (errorRate > 0.5 && stats.requests > 10) {
      console.log(`🔴 ${serverUrl} marked unhealthy (${errorRate * 100}% error rate)`);
      return false; // Server is unhealthy
    }
    
    return true; // Server is healthy
  }
}
```

## 💡 Common Interview Questions & Answers

### Q1: "What's the difference between Layer 4 and Layer 7 load balancing?"
**Answer:**
"Layer 4 operates at the transport layer, routing based on IP addresses and ports. It's faster and protocol-agnostic. Layer 7 operates at the application layer, making routing decisions based on content like HTTP headers or URLs. It enables advanced features like SSL termination and content-based routing but has higher latency."

### Q2: "How do you handle server failures in load balancing?"
**Answer:**
"I implement health checks - both active (periodic ping to health endpoints) and passive (monitoring response times and error rates). Failed servers are temporarily removed from rotation. I also implement circuit breakers and retry logic with exponential backoff for transient failures."

### Q3: "When would you use different load balancing algorithms?"
**Answer:**
"Round Robin for uniform servers and traffic. Weighted Round Robin for servers with different capacities. Least Connections for varying request processing times. IP Hash for session persistence. Geographic routing for global applications with region-specific data."

### Q4: "How do you maintain session persistence?"
**Answer:**
"Several approaches:
1. **Sticky sessions** - Route same client to same server (IP hash)
2. **Shared session store** - Store sessions in Redis/database
3. **Stateless design** - Use JWT tokens, no server-side sessions
4. **Session replication** - Sync sessions across servers
I prefer stateless design for scalability."

## 🚀 Advanced Load Balancing

### Global Load Balancing:
```javascript
// DNS-based global load balancing
const regions = {
  'us-east': ['server1.us-east.com', 'server2.us-east.com'],
  'us-west': ['server1.us-west.com', 'server2.us-west.com'],
  'europe': ['server1.eu.com', 'server2.eu.com']
};

function getServerByLocation(clientIP) {
  const region = getRegionFromIP(clientIP);
  const servers = regions[region] || regions['us-east']; // fallback
  return servers[Math.floor(Math.random() * servers.length)];
}
```

### Auto-Scaling Integration:
```javascript
class AutoScalingBalancer {
  constructor() {
    this.servers = [];
    this.targetUtilization = 70; // Target 70% CPU
    this.minServers = 2;
    this.maxServers = 10;
  }
  
  async checkScaling() {
    const avgUtilization = await this.getAverageUtilization();
    
    if (avgUtilization > 80 && this.servers.length < this.maxServers) {
      await this.scaleUp();
    } else if (avgUtilization < 30 && this.servers.length > this.minServers) {
      await this.scaleDown();
    }
  }
  
  async scaleUp() {
    const newServer = await this.launchServer();
    this.servers.push(newServer);
    console.log(`📈 Scaled up: Added ${newServer.url}`);
  }
  
  async scaleDown() {
    const serverToRemove = this.servers.pop();
    await this.terminateServer(serverToRemove);
    console.log(`📉 Scaled down: Removed ${serverToRemove.url}`);
  }
}
```

## 🔧 Best Practices

### Configuration Example:
```yaml
# HAProxy Configuration
global
    daemon
    maxconn 4096

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend web_frontend
    bind *:80
    default_backend web_servers

backend web_servers
    balance roundrobin
    option httpchk GET /health
    
    server web1 192.168.1.10:8080 check
    server web2 192.168.1.11:8080 check
    server web3 192.168.1.12:8080 check
```

### Monitoring Metrics:
```javascript
const metrics = {
  requestsPerSecond: 0,
  activeConnections: 0,
  responseTime: {
    avg: 0,
    p95: 0,
    p99: 0
  },
  serverHealth: new Map(),
  errorRate: 0
};
```

## 🎯 Interview Success Tips

### **Always Mention:**
- "Choose algorithm based on use case"
- "Health checks prevent routing to failed servers"
- "Consider session persistence requirements"
- "Monitor and measure performance"
- "Plan for auto-scaling"

### **Show Real-World Knowledge:**
- "HAProxy and Nginx for production"
- "Cloud load balancers (ALB, NLB)"
- "Geographic load balancing"
- "SSL termination at load balancer"
- "DDoS protection integration"

## 🔧 Quick Checklist

- [ ] Understand different algorithms
- [ ] Know Layer 4 vs Layer 7 differences
- [ ] Explain health check strategies
- [ ] Describe session persistence options
- [ ] Discuss auto-scaling integration
- [ ] Know monitoring best practices

## 🎉 You're Load Balancing Ready!

**Key Message**: Load balancing is about **efficiently distributing traffic** to ensure high availability, performance, and fault tolerance!

**Interview Golden Rule**: Always discuss trade-offs between different algorithms and explain how you'd choose based on specific requirements.

Perfect! Now you can confidently discuss load balancing strategies! ⚖️ 