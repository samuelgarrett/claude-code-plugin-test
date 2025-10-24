# Performance Optimizer

**Description**: Profile, measure, and optimize application performance systematically

## Performance Optimization Process

1. **Measure** - Profile to find bottlenecks
2. **Analyze** - Understand the root cause
3. **Optimize** - Implement improvements
4. **Verify** - Measure again to confirm

## Frontend Performance

### Measuring Performance
```javascript
// Using Performance API
const perfData = performance.getEntriesByType('navigation')[0];
console.log('Page load time:', perfData.loadEventEnd - perfData.fetchStart);

// Core Web Vitals
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

getCLS(console.log);  // Cumulative Layout Shift
getFID(console.log);  // First Input Delay
getFCP(console.log);  // First Contentful Paint
getLCP(console.log);  // Largest Contentful Paint
getTTFB(console.log); // Time to First Byte
```

### React Optimization
```javascript
// 1. Memoization
const MemoizedComponent = React.memo(ExpensiveComponent);

// 2. useMemo for expensive calculations
const sortedItems = useMemo(() => {
  return items.sort(compareFunction);
}, [items]);

// 3. useCallback for function references
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);

// 4. Lazy loading
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// 5. Code splitting
const routes = [
  { path: '/', component: lazy(() => import('./Home')) },
  { path: '/dashboard', component: lazy(() => import('./Dashboard')) }
];

// 6. Virtualize long lists
import { FixedSizeList } from 'react-window';
```

### Browser Performance
```javascript
// Debounce expensive operations
function debounce(func, wait) {
  let timeout;
  return function(...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, args), wait);
  };
}

const handleSearch = debounce((query) => {
  fetch(`/api/search?q=${query}`);
}, 300);

// Use Web Workers for heavy computation
const worker = new Worker('worker.js');
worker.postMessage({ data: largeDataset });
worker.onmessage = (e) => {
  console.log('Result:', e.data);
};
```

## Backend Performance

### Node.js Profiling
```bash
# CPU profiling
node --prof app.js
# Generates isolate-*.log

# Process the log
node --prof-process isolate-*.log > processed.txt

# Memory profiling
node --inspect app.js
# Open chrome://inspect in Chrome
```

### Database Optimization
```javascript
// ❌ Bad: N+1 query problem
const users = await User.findAll();
for (const user of users) {
  const posts = await Post.findAll({ where: { userId: user.id } });
  user.posts = posts;
}

// ✅ Good: Use eager loading
const users = await User.findAll({
  include: [{ model: Post }]
});

// ❌ Bad: Select all columns
const users = await User.findAll();

// ✅ Good: Select only needed columns
const users = await User.findAll({
  attributes: ['id', 'name', 'email']
});

// Add indexes
await queryInterface.addIndex('users', ['email']);
await queryInterface.addIndex('posts', ['userId', 'createdAt']);
```

### Caching
```javascript
// Redis caching
const redis = require('redis');
const client = redis.createClient();

async function getUser(id) {
  // Check cache first
  const cached = await client.get(`user:${id}`);
  if (cached) return JSON.parse(cached);

  // Fetch from database
  const user = await User.findByPk(id);

  // Store in cache (expire after 1 hour)
  await client.setex(`user:${id}`, 3600, JSON.stringify(user));

  return user;
}

// In-memory caching
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 600 });

function expensiveOperation(key) {
  const cached = cache.get(key);
  if (cached) return cached;

  const result = doExpensiveCalculation();
  cache.set(key, result);
  return result;
}
```

## Performance Testing

### Load Testing with Artillery
```yaml
# artillery.yml
config:
  target: "http://localhost:3000"
  phases:
    - duration: 60
      arrivalRate: 10      # 10 users per second
    - duration: 120
      arrivalRate: 50      # Ramp up to 50/sec
  scenarios:
    - name: "API test"
      flow:
        - get:
            url: "/api/users"
        - post:
            url: "/api/users"
            json:
              name: "Test User"
              email: "test@example.com"
```

```bash
# Run load test
artillery run artillery.yml
```

### Benchmarking
```javascript
// Simple benchmark
console.time('operation');
performOperation();
console.timeEnd('operation');

// Detailed benchmark
const Benchmark = require('benchmark');
const suite = new Benchmark.Suite;

suite
  .add('Array#forEach', function() {
    [1,2,3].forEach(x => x * 2);
  })
  .add('Array#map', function() {
    [1,2,3].map(x => x * 2);
  })
  .on('cycle', function(event) {
    console.log(String(event.target));
  })
  .on('complete', function() {
    console.log('Fastest is ' + this.filter('fastest').map('name'));
  })
  .run();
```

## Common Optimizations

### 1. Bundle Size (Webpack)
```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10
        }
      }
    }
  },
  performance: {
    maxEntrypointSize: 512000,
    maxAssetSize: 512000
  }
};

// Analyze bundle
npm install --save-dev webpack-bundle-analyzer
```

### 2. Image Optimization
```bash
# Convert to WebP
cwebp image.jpg -q 80 -o image.webp

# Optimize PNG
pngquant image.png --output image-optimized.png

# Responsive images
convert image.jpg -resize 400 image-400w.jpg
convert image.jpg -resize 800 image-800w.jpg
```

### 3. HTTP/2 & Compression
```javascript
// Express with compression
const compression = require('compression');
app.use(compression());

// Nginx configuration
# gzip compression
gzip on;
gzip_types text/plain text/css application/json application/javascript;
```

## Metrics to Track

- **First Contentful Paint (FCP)**: < 1.8s
- **Largest Contentful Paint (LCP)**: < 2.5s
- **Time to Interactive (TTI)**: < 3.8s
- **Cumulative Layout Shift (CLS)**: < 0.1
- **First Input Delay (FID)**: < 100ms

## When to Use This Skill

- Application is slow
- Optimizing page load time
- Reducing server response time
- Improving database query performance
- Analyzing bundle size

---

**Remember**: Measure first, optimize second. Don't guess where the bottleneck is!
