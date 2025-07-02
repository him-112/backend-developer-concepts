# Profiling & Debugging - Performance Guide 🔍

## 🎯 What You'll Learn
Master profiling and debugging techniques to identify and solve performance bottlenecks!

## 🌟 Profiling Fundamentals

### What is Profiling?
Think of profiling like **detective work**:
- **CPU Profiler** = Time tracker (where time is spent)
- **Memory Profiler** = Space investigator (memory usage patterns)
- **Heap Dumps** = Crime scene photos (memory state snapshot)
- **Call Stack** = Witness testimony (execution path)

Find the culprit slowing down your application!

### Key Interview Points ⭐
1. "Profiling identifies performance bottlenecks"
2. "Different profilers for CPU, memory, and I/O"
3. "Profile in production-like environments"
4. "Always measure before and after optimizations"
5. "Focus on the biggest bottlenecks first"

## 🔍 CPU Profiling

### Node.js CPU Profiling

```javascript
// Built-in Node.js profiler
const { performance, PerformanceObserver } = require('perf_hooks');

// Basic performance measurement
function profileFunction(fn, name) {
  return function(...args) {
    const start = performance.now();
    const result = fn.apply(this, args);
    const end = performance.now();
    
    console.log(`${name} took ${end - start} milliseconds`);
    return result;
  };
}

// Usage
const slowFunction = profileFunction(function(data) {
  // Some expensive operation
  return data.map(x => x * 2).filter(x => x > 10);
}, 'Array Processing');

// Advanced profiling with PerformanceObserver
const obs = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log(`${entry.name}: ${entry.duration}ms`);
  });
});
obs.observe({ entryTypes: ['measure', 'mark'] });

// Mark important points
performance.mark('database-query-start');
await database.query('SELECT * FROM users');
performance.mark('database-query-end');
performance.measure('Database Query', 'database-query-start', 'database-query-end');

// Custom profiler class
class CPUProfiler {
  constructor() {
    this.measurements = new Map();
    this.activeTimers = new Map();
  }
  
  start(label) {
    this.activeTimers.set(label, performance.now());
  }
  
  end(label) {
    const start = this.activeTimers.get(label);
    if (start) {
      const duration = performance.now() - start;
      
      if (!this.measurements.has(label)) {
        this.measurements.set(label, {
          count: 0,
          totalTime: 0,
          minTime: Infinity,
          maxTime: 0,
          avgTime: 0
        });
      }
      
      const stats = this.measurements.get(label);
      stats.count++;
      stats.totalTime += duration;
      stats.minTime = Math.min(stats.minTime, duration);
      stats.maxTime = Math.max(stats.maxTime, duration);
      stats.avgTime = stats.totalTime / stats.count;
      
      this.activeTimers.delete(label);
      return duration;
    }
    return null;
  }
  
  getReport() {
    const report = [];
    for (const [label, stats] of this.measurements) {
      report.push({
        label,
        ...stats,
        avgTime: Math.round(stats.avgTime * 100) / 100
      });
    }
    return report.sort((a, b) => b.totalTime - a.totalTime);
  }
  
  // Decorator for automatic profiling
  profile(target, propertyName, descriptor) {
    const originalMethod = descriptor.value;
    const profiler = this;
    
    descriptor.value = function(...args) {
      const className = this.constructor.name;
      const label = `${className}.${propertyName}`;
      
      profiler.start(label);
      
      try {
        const result = originalMethod.apply(this, args);
        
        if (result && typeof result.then === 'function') {
          // Handle async methods
          return result.finally(() => profiler.end(label));
        } else {
          profiler.end(label);
          return result;
        }
      } catch (error) {
        profiler.end(label);
        throw error;
      }
    };
    
    return descriptor;
  }
}

// Usage with decorator
const profiler = new CPUProfiler();

class UserService {
  @profiler.profile
  async createUser(userData) {
    // This method will be automatically profiled
    return await database.users.create(userData);
  }
  
  @profiler.profile
  validateUserData(userData) {
    // Expensive validation logic
    return isValid;
  }
}
```

### V8 Profiler Integration

```javascript
// Using clinic.js for Node.js profiling
// npm install -g clinic
// clinic doctor -- node app.js
// clinic flame -- node app.js

// Or using 0x for flame graphs
// npm install -g 0x
// 0x -- node app.js

// Custom V8 CPU profiler
const v8Profiler = require('v8-profiler-next');

class V8CPUProfiler {
  constructor() {
    this.profiles = new Map();
  }
  
  startProfiling(name) {
    v8Profiler.startProfiling(name, true);
  }
  
  stopProfiling(name, saveToFile = false) {
    const profile = v8Profiler.stopProfiling(name);
    
    if (saveToFile) {
      const fs = require('fs');
      profile.export((error, result) => {
        fs.writeFileSync(`${name}-${Date.now()}.cpuprofile`, result);
        profile.delete();
      });
    }
    
    return profile;
  }
  
  // Middleware for Express.js
  middleware() {
    return (req, res, next) => {
      const profileName = `${req.method}-${req.url}-${Date.now()}`;
      
      this.startProfiling(profileName);
      
      res.on('finish', () => {
        const profile = this.stopProfiling(profileName);
        
        // Analyze slow requests
        if (res.responseTime > 1000) {
          this.saveProfile(profile, profileName);
        }
        
        profile.delete();
      });
      
      next();
    };
  }
}
```

## 🧠 Memory Profiling

### Memory Leak Detection

```javascript
// Memory usage monitoring
class MemoryProfiler {
  constructor() {
    this.snapshots = [];
    this.leakDetectionEnabled = false;
  }
  
  takeSnapshot() {
    const usage = process.memoryUsage();
    const snapshot = {
      timestamp: Date.now(),
      rss: usage.rss, // Resident Set Size
      heapTotal: usage.heapTotal,
      heapUsed: usage.heapUsed,
      external: usage.external,
      arrayBuffers: usage.arrayBuffers
    };
    
    this.snapshots.push(snapshot);
    
    // Keep only last 100 snapshots
    if (this.snapshots.length > 100) {
      this.snapshots.shift();
    }
    
    return snapshot;
  }
  
  detectMemoryLeaks() {
    if (this.snapshots.length < 10) return null;
    
    // Check if memory consistently grows
    const recent = this.snapshots.slice(-10);
    const growth = recent.map((snapshot, i) => {
      if (i === 0) return 0;
      return snapshot.heapUsed - recent[i-1].heapUsed;
    });
    
    const avgGrowth = growth.reduce((sum, g) => sum + g, 0) / growth.length;
    const positiveGrowth = growth.filter(g => g > 0).length;
    
    // Potential memory leak if memory grows in 80% of samples
    if (positiveGrowth >= 8 && avgGrowth > 1024 * 1024) { // 1MB average growth
      return {
        severity: 'warning',
        message: 'Potential memory leak detected',
        avgGrowthMB: Math.round(avgGrowth / 1024 / 1024 * 100) / 100,
        growthRate: positiveGrowth / recent.length
      };
    }
    
    return null;
  }
  
  startMonitoring(intervalMs = 30000) {
    this.leakDetectionEnabled = true;
    
    const monitor = () => {
      if (!this.leakDetectionEnabled) return;
      
      this.takeSnapshot();
      const leak = this.detectMemoryLeaks();
      
      if (leak) {
        console.warn('Memory Leak Detection:', leak);
      }
      
      setTimeout(monitor, intervalMs);
    };
    
    monitor();
  }
  
  stopMonitoring() {
    this.leakDetectionEnabled = false;
  }
  
  getMemoryReport() {
    if (this.snapshots.length === 0) return null;
    
    const latest = this.snapshots[this.snapshots.length - 1];
    const earliest = this.snapshots[0];
    
    return {
      current: {
        heapUsedMB: Math.round(latest.heapUsed / 1024 / 1024 * 100) / 100,
        heapTotalMB: Math.round(latest.heapTotal / 1024 / 1024 * 100) / 100,
        rssMB: Math.round(latest.rss / 1024 / 1024 * 100) / 100
      },
      growth: {
        heapGrowthMB: Math.round((latest.heapUsed - earliest.heapUsed) / 1024 / 1024 * 100) / 100,
        timespan: latest.timestamp - earliest.timestamp
      }
    };
  }
}

// Heap dump analysis
const v8 = require('v8');
const fs = require('fs');

class HeapAnalyzer {
  static takeHeapSnapshot(filename) {
    const snapshotStream = v8.getHeapSnapshot();
    const fileStream = fs.createWriteStream(filename);
    snapshotStream.pipe(fileStream);
    
    return new Promise((resolve, reject) => {
      fileStream.on('finish', resolve);
      fileStream.on('error', reject);
    });
  }
  
  static async compareHeapSnapshots(before, after) {
    // This would typically use a heap analysis library
    // For demonstration, showing the concept
    
    const beforeStats = await this.analyzeHeapSnapshot(before);
    const afterStats = await this.analyzeHeapSnapshot(after);
    
    return {
      objectGrowth: afterStats.objects - beforeStats.objects,
      sizeGrowth: afterStats.size - beforeStats.size,
      newObjectTypes: afterStats.types.filter(t => !beforeStats.types.includes(t))
    };
  }
}
```

## 🐛 Advanced Debugging Techniques

### Performance Debugging Middleware

```javascript
class PerformanceDebugger {
  constructor() {
    this.slowOperations = [];
    this.thresholds = {
      database: 100,    // 100ms
      api: 200,         // 200ms
      total: 1000       // 1 second
    };
  }
  
  // Express middleware for request debugging
  debugMiddleware() {
    return async (req, res, next) => {
      const startTime = Date.now();
      const debug = {
        url: req.url,
        method: req.method,
        timestamp: new Date().toISOString(),
        operations: []
      };
      
      // Intercept database queries
      this.interceptDatabase(debug);
      
      // Intercept external API calls
      this.interceptHTTP(debug);
      
      res.on('finish', () => {
        const totalTime = Date.now() - startTime;
        debug.totalDuration = totalTime;
        debug.statusCode = res.statusCode;
        
        // Log slow requests
        if (totalTime > this.thresholds.total) {
          this.logSlowOperation(debug);
        }
      });
      
      req.debug = debug;
      next();
    };
  }
  
  interceptDatabase(debug) {
    const originalQuery = db.query;
    
    db.query = function(sql, params) {
      const start = Date.now();
      const operation = {
        type: 'database',
        sql: sql.substring(0, 100) + '...',
        startTime: start
      };
      
      debug.operations.push(operation);
      
      return originalQuery.call(this, sql, params)
        .then(result => {
          operation.duration = Date.now() - start;
          operation.rowCount = Array.isArray(result) ? result.length : 1;
          return result;
        })
        .catch(error => {
          operation.duration = Date.now() - start;
          operation.error = error.message;
          throw error;
        });
    };
  }
  
  interceptHTTP(debug) {
    const originalFetch = global.fetch;
    
    global.fetch = function(url, options = {}) {
      const start = Date.now();
      const operation = {
        type: 'http',
        url: url.substring(0, 100),
        method: options.method || 'GET',
        startTime: start
      };
      
      debug.operations.push(operation);
      
      return originalFetch(url, options)
        .then(response => {
          operation.duration = Date.now() - start;
          operation.status = response.status;
          return response;
        })
        .catch(error => {
          operation.duration = Date.now() - start;
          operation.error = error.message;
          throw error;
        });
    };
  }
  
  logSlowOperation(debug) {
    console.warn('Slow Request Detected:', {
      url: debug.url,
      totalDuration: debug.totalDuration,
      slowOperations: debug.operations.filter(op => 
        op.duration > this.thresholds[op.type]
      )
    });
    
    this.slowOperations.push(debug);
    
    // Keep only last 50 slow operations
    if (this.slowOperations.length > 50) {
      this.slowOperations.shift();
    }
  }
  
  getSlowOperationReport() {
    return {
      count: this.slowOperations.length,
      patterns: this.analyzePatterns(),
      recommendations: this.getRecommendations()
    };
  }
  
  analyzePatterns() {
    const patterns = {};
    
    this.slowOperations.forEach(op => {
      const route = op.url.split('?')[0]; // Remove query params
      
      if (!patterns[route]) {
        patterns[route] = {
          count: 0,
          avgDuration: 0,
          totalDuration: 0,
          commonSlowOps: {}
        };
      }
      
      const pattern = patterns[route];
      pattern.count++;
      pattern.totalDuration += op.totalDuration;
      pattern.avgDuration = pattern.totalDuration / pattern.count;
      
      // Track common slow operations
      op.operations.forEach(operation => {
        if (operation.duration > this.thresholds[operation.type]) {
          const key = `${operation.type}:${operation.sql || operation.url || 'unknown'}`;
          pattern.commonSlowOps[key] = (pattern.commonSlowOps[key] || 0) + 1;
        }
      });
    });
    
    return patterns;
  }
}
```

### Database Query Analysis

```javascript
class QueryAnalyzer {
  constructor() {
    this.queries = [];
    this.nPlusOneDetector = new Map();
  }
  
  analyzeQuery(sql, duration, context = {}) {
    const analysis = {
      sql,
      duration,
      timestamp: Date.now(),
      issues: [],
      suggestions: [],
      context
    };
    
    // Detect potential issues
    this.detectNPlusOne(analysis);
    this.detectSlowQueries(analysis);
    this.detectMissingIndexes(analysis);
    this.detectCartesianProducts(analysis);
    
    this.queries.push(analysis);
    
    // Keep only recent queries
    if (this.queries.length > 1000) {
      this.queries.shift();
    }
    
    return analysis;
  }
  
  detectNPlusOne(analysis) {
    const pattern = this.extractQueryPattern(analysis.sql);
    const context = analysis.context.route || 'unknown';
    
    const key = `${context}:${pattern}`;
    
    if (!this.nPlusOneDetector.has(key)) {
      this.nPlusOneDetector.set(key, {
        count: 0,
        firstSeen: Date.now(),
        lastSeen: Date.now()
      });
    }
    
    const detector = this.nPlusOneDetector.get(key);
    detector.count++;
    detector.lastSeen = Date.now();
    
    // If same query pattern executed multiple times in short period
    if (detector.count > 5 && detector.lastSeen - detector.firstSeen < 1000) {
      analysis.issues.push('Potential N+1 query detected');
      analysis.suggestions.push('Consider using JOIN or eager loading');
    }
  }
  
  detectSlowQueries(analysis) {
    if (analysis.duration > 100) {
      analysis.issues.push(`Slow query (${analysis.duration}ms)`);
      
      if (analysis.sql.includes('SELECT *')) {
        analysis.suggestions.push('Avoid SELECT *, specify needed columns');
      }
      
      if (!analysis.sql.includes('LIMIT') && analysis.sql.includes('SELECT')) {
        analysis.suggestions.push('Consider adding LIMIT clause');
      }
    }
  }
  
  detectMissingIndexes(analysis) {
    // Simple heuristic - check for WHERE clauses without obvious indexes
    const whereMatch = analysis.sql.match(/WHERE\s+(\w+)\s*=/i);
    if (whereMatch && analysis.duration > 50) {
      const column = whereMatch[1];
      if (!['id', 'uuid'].includes(column.toLowerCase())) {
        analysis.suggestions.push(`Consider adding index on column: ${column}`);
      }
    }
  }
  
  detectCartesianProducts(analysis) {
    // Check for JOINs without proper ON clauses
    const joinCount = (analysis.sql.match(/JOIN/gi) || []).length;
    const onCount = (analysis.sql.match(/ON/gi) || []).length;
    
    if (joinCount > onCount) {
      analysis.issues.push('Potential cartesian product - missing JOIN conditions');
    }
  }
  
  extractQueryPattern(sql) {
    // Normalize SQL to detect patterns
    return sql
      .replace(/\d+/g, '?')           // Replace numbers with ?
      .replace(/'[^']*'/g, '?')       // Replace strings with ?
      .replace(/\s+/g, ' ')           // Normalize whitespace
      .trim()
      .toLowerCase();
  }
  
  getQueryReport() {
    const totalQueries = this.queries.length;
    const slowQueries = this.queries.filter(q => q.duration > 100);
    const avgDuration = this.queries.reduce((sum, q) => sum + q.duration, 0) / totalQueries;
    
    return {
      totalQueries,
      slowQueries: slowQueries.length,
      avgDuration: Math.round(avgDuration * 100) / 100,
      topSlowQueries: slowQueries
        .sort((a, b) => b.duration - a.duration)
        .slice(0, 10),
      commonIssues: this.getCommonIssues()
    };
  }
  
  getCommonIssues() {
    const issues = {};
    
    this.queries.forEach(query => {
      query.issues.forEach(issue => {
        issues[issue] = (issues[issue] || 0) + 1;
      });
    });
    
    return Object.entries(issues)
      .sort(([,a], [,b]) => b - a)
      .slice(0, 5);
  }
}
```

## 💡 Common Interview Questions & Answers

### Q1: "How do you identify performance bottlenecks?"
**Answer:**
"Systematic approach to find bottlenecks:
1. **Profile first** - Use CPU and memory profilers
2. **Measure everything** - Database queries, API calls, functions
3. **Look for patterns** - N+1 queries, memory leaks, hot spots
4. **Use APM tools** - Application Performance Monitoring
5. **Focus on biggest impact** - 80/20 rule, optimize slowest parts first"

### Q2: "What's the difference between CPU and memory profiling?"
**Answer:**
"Different aspects of performance:
- **CPU Profiling**: Shows where time is spent, function call frequency, hot paths
- **Memory Profiling**: Shows memory allocation patterns, leaks, heap usage
- **Use CPU profiling for**: Slow functions, algorithmic issues
- **Use memory profiling for**: Memory leaks, excessive allocations, GC pressure"

### Q3: "How do you debug memory leaks in Node.js?"
**Answer:**
"Multi-step process:
1. **Monitor heap growth** - Track memory usage over time
2. **Take heap snapshots** - Before and after operations
3. **Compare snapshots** - Find objects that aren't being garbage collected
4. **Common causes**: Event listeners not removed, closures holding references, global variables
5. **Tools**: Node.js --inspect, clinic.js, heap dumps"

### Q4: "How do you profile production applications safely?"
**Answer:**
"Safe production profiling strategies:
1. **Sampling profilers** - Low overhead, statistical sampling
2. **Conditional profiling** - Only profile slow requests
3. **Load balancer rotation** - Profile one instance at a time
4. **Feature flags** - Enable profiling for specific users/requests
5. **Off-peak profiling** - During low traffic periods"

## 🚀 Best Practices

### Production-Safe Profiling

```javascript
class ProductionProfiler {
  constructor() {
    this.enabled = false;
    this.samplingRate = 0.01; // 1% of requests
    this.maxProfileTime = 30000; // 30 seconds max
  }
  
  shouldProfile(req) {
    if (!this.enabled) return false;
    
    // Only profile slow requests or sampled requests
    return Math.random() < this.samplingRate || 
           req.headers['x-enable-profiling'] === 'true';
  }
  
  safeProfile(name, operation) {
    if (!this.shouldProfile()) {
      return operation();
    }
    
    const timeout = setTimeout(() => {
      console.warn(`Profiling timeout for ${name}`);
    }, this.maxProfileTime);
    
    try {
      const result = this.profileOperation(name, operation);
      clearTimeout(timeout);
      return result;
    } catch (error) {
      clearTimeout(timeout);
      throw error;
    }
  }
}
```

## 🔧 Quick Checklist

- [ ] Set up CPU profiling for performance hotspots
- [ ] Monitor memory usage and detect leaks
- [ ] Profile database queries and detect N+1 problems
- [ ] Use production-safe profiling techniques
- [ ] Implement automated performance regression detection
- [ ] Create alerts for performance degradation

## 🎉 You're Profiling & Debugging Ready!

**Key Message**: Effective profiling and debugging help you **find and fix performance bottlenecks** systematically!

**Interview Golden Rule**: Always emphasize measuring before optimizing and using production-safe profiling techniques.

Perfect! Now you can confidently discuss profiling and debugging! 🔍
