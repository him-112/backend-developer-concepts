# CDN & Edge Caching - Performance Guide 🌐

## 🎯 What You'll Learn
Master CDN and edge caching to deliver lightning-fast content worldwide!

## 🌟 CDN Fundamentals

### What is a CDN?
Think of a CDN like a **global chain of convenience stores**:
- **Main warehouse** = Origin server
- **Local stores** = Edge servers/PoPs (Points of Presence)
- **Customers** = Users worldwide
- **Products** = Static assets (images, CSS, JS, videos)

Instead of everyone traveling to the main warehouse, they get what they need from the nearest local store!

### Key Interview Points ⭐
1. "Distributes content across global edge servers"
2. "Reduces latency by serving from nearest location"
3. "Offloads traffic from origin servers"
4. "Provides DDoS protection and security"
5. "Essential for global web performance"

## 🚀 CDN Benefits & Use Cases

### Performance Benefits:
```javascript
// Without CDN
const withoutCDN = {
  userLocation: 'Tokyo',
  originServer: 'New York',
  distance: '10,000 km',
  latency: '150ms',
  loadTime: '3.2s'
};

// With CDN
const withCDN = {
  userLocation: 'Tokyo',
  edgeServer: 'Tokyo PoP',
  distance: '50 km',
  latency: '5ms',
  loadTime: '0.8s',
  improvement: '75% faster'
};
```

### Use Cases:
1. **Static Assets** - Images, CSS, JavaScript
2. **Video Streaming** - Adaptive bitrate streaming
3. **API Acceleration** - Caching API responses
4. **Dynamic Content** - Edge-side includes
5. **Mobile Apps** - App store distribution

## 🔧 CDN Types & Technologies

### 1. **Push CDN**
*You upload content to CDN*

```javascript
// Push CDN workflow
class PushCDN {
  constructor(config) {
    this.apiKey = config.apiKey;
    this.baseUrl = config.baseUrl;
  }
  
  async uploadAsset(file, path) {
    const formData = new FormData();
    formData.append('file', file);
    formData.append('path', path);
    
    const response = await fetch(`${this.baseUrl}/upload`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.apiKey}`
      },
      body: formData
    });
    
    return await response.json();
  }
  
  async deployToEdge(assetId) {
    // Deploy to all edge locations
    const response = await fetch(`${this.baseUrl}/deploy/${assetId}`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.apiKey}`
      }
    });
    
    return await response.json();
  }
}

// Usage for product images
const cdn = new PushCDN({
  apiKey: 'your-api-key',
  baseUrl: 'https://api.yourcdn.com'
});

const productImage = await cdn.uploadAsset(imageFile, '/products/img-123.jpg');
await cdn.deployToEdge(productImage.id);
```

### 2. **Pull CDN**
*CDN fetches content from your origin*

```javascript
// Pull CDN configuration
const pullCDNConfig = {
  originServer: 'https://api.yoursite.com',
  edgeLocations: [
    'us-east-1',
    'us-west-1', 
    'eu-west-1',
    'ap-southeast-1'
  ],
  cachingRules: [
    {
      path: '/images/*',
      ttl: 86400, // 24 hours
      headers: ['Cache-Control: public, max-age=86400']
    },
    {
      path: '/api/static/*',
      ttl: 3600, // 1 hour
      headers: ['Cache-Control: public, max-age=3600']
    },
    {
      path: '/api/user/*',
      ttl: 0, // No caching for user-specific data
      headers: ['Cache-Control: no-cache']
    }
  ]
};

// CDN request flow
class CDNRequestFlow {
  async handleRequest(request) {
    const cacheKey = this.generateCacheKey(request);
    
    // Check edge cache
    let response = await this.checkEdgeCache(cacheKey);
    
    if (!response) {
      // Cache miss - check regional cache
      response = await this.checkRegionalCache(cacheKey);
      
      if (!response) {
        // Still miss - fetch from origin
        response = await this.fetchFromOrigin(request);
        
        // Cache at regional level
        await this.storeInRegionalCache(cacheKey, response);
      }
      
      // Cache at edge level
      await this.storeInEdgeCache(cacheKey, response);
    }
    
    return response;
  }
  
  generateCacheKey(request) {
    const url = new URL(request.url);
    const params = new URLSearchParams(url.search);
    
    // Sort parameters for consistent caching
    const sortedParams = [...params.entries()].sort();
    const queryString = new URLSearchParams(sortedParams).toString();
    
    return `${url.pathname}${queryString ? '?' + queryString : ''}`;
  }
}
```

## 🔒 Cache Headers & Control

### 1. **Cache-Control Headers**

```javascript
// Express.js cache control
app.get('/api/products', (req, res) => {
  // Public cache for 1 hour
  res.set('Cache-Control', 'public, max-age=3600');
  res.set('ETag', generateETag(products));
  res.json(products);
});

app.get('/api/user/profile', (req, res) => {
  // Private, no caching
  res.set('Cache-Control', 'private, no-cache');
  res.json(userProfile);
});

app.get('/static/images/:id', (req, res) => {
  // Cache for 1 year with immutable content
  res.set('Cache-Control', 'public, max-age=31536000, immutable');
  res.sendFile(imagePath);
});

// Advanced cache control
class CacheController {
  static getHeaders(type, data) {
    const headers = {};
    
    switch(type) {
      case 'static-assets':
        headers['Cache-Control'] = 'public, max-age=31536000, immutable';
        headers['Vary'] = 'Accept-Encoding';
        break;
        
      case 'api-data':
        headers['Cache-Control'] = 'public, max-age=3600';
        headers['ETag'] = this.generateETag(data);
        break;
        
      case 'user-specific':
        headers['Cache-Control'] = 'private, max-age=300';
        break;
        
      case 'no-cache':
        headers['Cache-Control'] = 'no-cache, no-store, must-revalidate';
        headers['Pragma'] = 'no-cache';
        headers['Expires'] = '0';
        break;
    }
    
    return headers;
  }
  
  static generateETag(data) {
    const crypto = require('crypto');
    return crypto.createHash('md5').update(JSON.stringify(data)).digest('hex');
  }
}
```

### 2. **Conditional Requests**

```javascript
// Handle ETags and conditional requests
app.get('/api/data', (req, res) => {
  const data = getCurrentData();
  const etag = generateETag(data);
  
  // Check if client has current version
  if (req.headers['if-none-match'] === etag) {
    return res.status(304).send(); // Not Modified
  }
  
  res.set('ETag', etag);
  res.set('Cache-Control', 'public, max-age=3600');
  res.json(data);
});

// Handle Last-Modified
app.get('/api/articles/:id', async (req, res) => {
  const article = await getArticle(req.params.id);
  const lastModified = article.updatedAt.toUTCString();
  
  // Check if client has current version
  if (req.headers['if-modified-since'] === lastModified) {
    return res.status(304).send();
  }
  
  res.set('Last-Modified', lastModified);
  res.set('Cache-Control', 'public, max-age=1800');
  res.json(article);
});
```

## 🌐 Edge Computing & Functions

### 1. **Edge Functions**

```javascript
// Cloudflare Workers example
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request));
});

async function handleRequest(request) {
  const url = new URL(request.url);
  
  // A/B testing at the edge
  if (url.pathname === '/homepage') {
    return handleHomepageAB(request);
  }
  
  // Image optimization at the edge
  if (url.pathname.startsWith('/images/')) {
    return handleImageOptimization(request);
  }
  
  // API response caching
  if (url.pathname.startsWith('/api/')) {
    return handleAPICaching(request);
  }
  
  // Default: pass through to origin
  return fetch(request);
}

async function handleHomepageAB(request) {
  const country = request.cf.country;
  const variant = country === 'US' ? 'variant-a' : 'variant-b';
  
  // Check cache
  const cacheKey = `homepage-${variant}`;
  const cache = caches.default;
  let response = await cache.match(cacheKey);
  
  if (!response) {
    // Fetch and modify content
    const originResponse = await fetch(request);
    let html = await originResponse.text();
    
    // Inject A/B test content
    html = html.replace('{{variant}}', variant);
    
    response = new Response(html, {
      headers: {
        'Content-Type': 'text/html',
        'Cache-Control': 'public, max-age=3600'
      }
    });
    
    // Cache the modified response
    await cache.put(cacheKey, response.clone());
  }
  
  return response;
}

async function handleImageOptimization(request) {
  const url = new URL(request.url);
  const accept = request.headers.get('Accept');
  
  // Determine optimal format
  let format = 'jpeg';
  if (accept && accept.includes('image/webp')) {
    format = 'webp';
  }
  
  // Check for optimized image in cache
  const cacheKey = `${url.pathname}-${format}`;
  const cache = caches.default;
  let response = await cache.match(cacheKey);
  
  if (!response) {
    // Fetch original image
    const originResponse = await fetch(request);
    
    if (originResponse.ok) {
      // Optimize image (pseudo-code)
      const optimizedImage = await optimizeImage(
        await originResponse.arrayBuffer(),
        format
      );
      
      response = new Response(optimizedImage, {
        headers: {
          'Content-Type': `image/${format}`,
          'Cache-Control': 'public, max-age=86400'
        }
      });
      
      await cache.put(cacheKey, response.clone());
    } else {
      response = originResponse;
    }
  }
  
  return response;
}
```

### 2. **Edge-Side Includes (ESI)**

```html
<!-- ESI template -->
<html>
<head>
  <title>My Website</title>
  <!-- Static header - cached for 24 hours -->
  <esi:include src="/fragments/header" ttl="86400"/>
</head>
<body>
  <!-- Dynamic user content - cached for 5 minutes -->
  <esi:include src="/fragments/user-nav" ttl="300"/>
  
  <!-- Static content - cached for 1 hour -->
  <esi:include src="/fragments/sidebar" ttl="3600"/>
  
  <!-- Dynamic content - not cached -->
  <esi:include src="/fragments/live-chat"/>
</body>
</html>
```

```javascript
// ESI processor
class ESIProcessor {
  async processESI(template, request) {
    const esiRegex = /<esi:include src="([^"]+)"(?:\s+ttl="(\d+)")?\/>/g;
    
    const fragments = [];
    let match;
    
    while ((match = esiRegex.exec(template)) !== null) {
      const [fullMatch, src, ttl] = match;
      fragments.push({
        placeholder: fullMatch,
        src: src,
        ttl: parseInt(ttl) || 0
      });
    }
    
    // Fetch all fragments in parallel
    const fragmentPromises = fragments.map(async (fragment) => {
      const cacheKey = `esi:${fragment.src}`;
      
      // Check cache first
      let content = await this.getFromCache(cacheKey);
      
      if (!content) {
        // Fetch from origin
        const response = await fetch(`${request.origin}${fragment.src}`);
        content = await response.text();
        
        // Cache if TTL specified
        if (fragment.ttl > 0) {
          await this.setCache(cacheKey, content, fragment.ttl);
        }
      }
      
      return {
        placeholder: fragment.placeholder,
        content: content
      };
    });
    
    const resolvedFragments = await Promise.all(fragmentPromises);
    
    // Replace placeholders with content
    let result = template;
    for (const fragment of resolvedFragments) {
      result = result.replace(fragment.placeholder, fragment.content);
    }
    
    return result;
  }
}
```

## 📊 CDN Performance Optimization

### 1. **Monitoring & Analytics**

```javascript
class CDNAnalytics {
  constructor(config) {
    this.config = config;
    this.metrics = {
      hitRate: 0,
      bandwidth: 0,
      requests: 0,
      errors: 0,
      latency: []
    };
  }
  
  async collectMetrics(timeRange = '1h') {
    const metrics = await this.fetchCDNMetrics(timeRange);
    
    return {
      hitRate: this.calculateHitRate(metrics),
      bandwidthSavings: this.calculateBandwidthSavings(metrics),
      performanceGain: this.calculatePerformanceGain(metrics),
      errorRate: this.calculateErrorRate(metrics),
      topContent: this.getTopContent(metrics),
      geographicDistribution: this.getGeoDistribution(metrics)
    };
  }
  
  calculateHitRate(metrics) {
    const totalRequests = metrics.cacheHits + metrics.cacheMisses;
    return totalRequests > 0 ? (metrics.cacheHits / totalRequests) * 100 : 0;
  }
  
  async optimizeCaching() {
    const analysis = await this.collectMetrics('24h');
    const suggestions = [];
    
    // Analyze cache hit rates
    if (analysis.hitRate < 80) {
      suggestions.push({
        type: 'cache-ttl',
        message: 'Consider increasing TTL for static assets',
        impact: 'high'
      });
    }
    
    // Analyze popular content
    analysis.topContent.forEach(item => {
      if (item.requests > 1000 && item.hitRate < 90) {
        suggestions.push({
          type: 'pre-warming',
          message: `Pre-warm cache for ${item.path}`,
          impact: 'medium'
        });
      }
    });
    
    return suggestions;
  }
}
```

### 2. **Cache Warming & Preloading**

```javascript
class CacheWarming {
  constructor(cdnConfig) {
    this.cdnConfig = cdnConfig;
    this.warmingQueue = [];
  }
  
  async warmPopularContent() {
    // Get list of popular URLs
    const popularUrls = await this.getPopularUrls();
    
    // Warm cache in all edge locations
    const warmingPromises = this.cdnConfig.edgeLocations.map(location => 
      this.warmLocation(location, popularUrls)
    );
    
    await Promise.all(warmingPromises);
  }
  
  async warmLocation(location, urls) {
    const edgeEndpoint = `https://${location}.cdn.example.com`;
    
    for (const url of urls) {
      try {
        // Request content to warm cache
        await fetch(`${edgeEndpoint}${url}`, {
          headers: {
            'Cache-Control': 'no-cache', // Force origin fetch
            'X-Cache-Warming': 'true'
          }
        });
        
        console.log(`Warmed ${url} in ${location}`);
      } catch (error) {
        console.error(`Failed to warm ${url} in ${location}:`, error);
      }
    }
  }
  
  async scheduleWarming(urls, schedule = 'daily') {
    // Add to warming queue
    this.warmingQueue.push({
      urls,
      schedule,
      nextRun: this.calculateNextRun(schedule)
    });
  }
  
  calculateNextRun(schedule) {
    const now = new Date();
    switch(schedule) {
      case 'hourly':
        return new Date(now.getTime() + 60 * 60 * 1000);
      case 'daily':
        return new Date(now.getTime() + 24 * 60 * 60 * 1000);
      default:
        return new Date(now.getTime() + 60 * 60 * 1000);
    }
  }
}
```

## 💡 Common Interview Questions & Answers

### Q1: "What's the difference between CDN and edge computing?"
**Answer:**
"CDN focuses on content delivery - caching static assets closer to users. Edge computing runs code at edge locations:
- **CDN**: Cache static files, reduce bandwidth, improve load times
- **Edge Computing**: Execute functions, process data, real-time logic at the edge
- **Modern approach**: Combines both - CDN with edge functions for dynamic content optimization"

### Q2: "How do you handle cache invalidation in a CDN?"
**Answer:**
"Several strategies:
1. **Purge API** - Invalidate specific URLs or patterns
2. **Versioned URLs** - Change URL when content changes
3. **Cache tags** - Group related content for bulk invalidation
4. **Surrogate keys** - Use custom headers for grouping
5. **Time-based** - Set appropriate TTL values
I prefer versioned URLs for assets and purge API for content updates."

### Q3: "How do you optimize images for global delivery?"
**Answer:**
"Multi-layered approach:
1. **Format optimization** - WebP for modern browsers, JPEG fallback
2. **Responsive images** - Multiple sizes for different devices
3. **Lazy loading** - Load images as needed
4. **Edge optimization** - Transform images at CDN edge
5. **Compression** - Optimize quality vs file size
6. **Progressive loading** - Progressive JPEG for faster perceived loading"

### Q4: "How do you measure CDN performance?"
**Answer:**
"Key metrics:
- **Cache hit rate** (>85% is good)
- **Time to First Byte (TTFB)** 
- **Page load time** improvement
- **Bandwidth savings**
- **Error rates** (4xx, 5xx)
- **Geographic performance** distribution
Use Real User Monitoring (RUM) and synthetic testing for comprehensive analysis."

## 🚀 Best Practices

### 1. **Content Optimization**
```javascript
// Asset optimization pipeline
class AssetOptimizer {
  async optimizeForCDN(assets) {
    const optimized = await Promise.all(
      assets.map(async (asset) => {
        switch(asset.type) {
          case 'image':
            return await this.optimizeImage(asset);
          case 'css':
            return await this.optimizeCSS(asset);
          case 'js':
            return await this.optimizeJS(asset);
          default:
            return asset;
        }
      })
    );
    
    return optimized;
  }
  
  async optimizeImage(image) {
    // Generate multiple formats and sizes
    const formats = ['webp', 'jpeg'];
    const sizes = [320, 640, 1024, 1920];
    
    const variants = [];
    for (const format of formats) {
      for (const size of sizes) {
        const optimized = await this.resizeAndCompress(image, size, format);
        variants.push({
          ...optimized,
          url: `${image.path}-${size}w.${format}`
        });
      }
    }
    
    return { ...image, variants };
  }
}
```

## 🔧 Quick Checklist

- [ ] Understand CDN vs edge computing differences
- [ ] Know cache headers and TTL strategies
- [ ] Implement proper cache invalidation
- [ ] Optimize assets for global delivery
- [ ] Monitor CDN performance metrics
- [ ] Use edge functions for dynamic optimization

## 🎉 You're CDN & Edge Caching Ready!

**Key Message**: CDN and edge caching provide **global performance optimization** by bringing content and computation closer to users!

**Interview Golden Rule**: Always discuss the trade-offs between cache duration, content freshness, and performance when explaining CDN strategies.

Perfect! Now you can confidently discuss CDN and edge caching! 🌐 