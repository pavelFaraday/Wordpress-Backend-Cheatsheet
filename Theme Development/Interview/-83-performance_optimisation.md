# Performance Optimization

> Performance is a layered system. Infrastructure, database structure, backend logic, frontend assets and third-party integrations must work together. I approach performance architecturally, not cosmetically.

##### 🚀 1️⃣ Performance Mindset

##### 🖥 2️⃣ Hosting & Infrastructure

##### 🧊 3️⃣ Caching (Very Important)

##### 🗄 4️⃣ Database Optimization

##### 🧩 5️⃣ Theme & Code Optimization

##### 🎨 6️⃣ Frontend Optimization

##### 📊 7️⃣ Third-Party Scripts

##### 🛍 8️⃣ WooCommerce Performance

##### 📈 9️⃣ Monitoring & Testing

##### 🔐 🔟 Security & Performance

##### 🌐 1️⃣1️⃣ REST API Performance

---

# 🚀 1️⃣ Performance Mindset

> Performance starts from architecture. Optimization at the end is too late.

WordPress speed is not about installing a plugin.

It is about:

- Server
- Database
- Code
- Frontend
- Third-party scripts
- User behavior

Performance is a system.

### 📍 Where to optimize

- During planning phase
- During architecture design
- Before choosing hosting
- Before selecting theme or builder

### ⚙️ How to optimize

- Choose scalable hosting from the beginning
- Avoid heavy architecture if not required
- Plan caching strategy early
- Design database structure properly
- Avoid unnecessary plugins

### 🎯 Expected Results

- No major refactoring later
- Stable long-term architecture
- Scalable system

---

# 🖥 2️⃣ Hosting & Infrastructure

> I always check infrastructure first. No plugin can fix a slow server.

If server is weak, nothing will fix it.

## What affects speed on server level:

### 1. PHP Version

Use PHP 8.1 or higher.

Newer PHP versions are:

- Faster
- More memory efficient
- More secure

Older PHP 7.x is slower.

### 2. OPcache

OPcache stores compiled PHP in memory.

Without OPcache:
Every request compiles PHP again.

With OPcache:
PHP runs faster because it is already compiled.

### 3. Web Server

Apache vs Nginx.

Nginx is usually:

- Better at handling high traffic
- Better at static file delivery

### 4. HTTP/2 or HTTP/3

Modern protocols allow:

- Multiple requests in parallel
- Faster resource loading

### 5. Memory Limit

Complex sites need at least 256MB.

Low memory = slow execution or fatal errors.

### 📍 Where to optimize

- Hosting control panel
- Server configuration
- php.ini
- Web server config (Apache/Nginx)

### ⚙️ How to optimize

- Upgrade to PHP 8.1 or 8.2
- Enable OPcache
- Increase memory_limit to 256M+
- Enable Gzip or Brotli compression
- Enable HTTP/2 or HTTP/3
- Use SSD/NVMe storage
- Configure PHP-FPM properly

### 🎯 Expected Results

- Lower TTFB
- Faster backend processing
- Better high-traffic handling
- Reduced CPU load

---

# 🧊 3️⃣ Caching (Very Important)

> I align caching strategy with site behavior. Dynamic sites require selective caching.

Caching means:
Instead of rebuilding the page every time, we serve saved version.

There are different types:

## 1️⃣ Page Cache

Normally:
User visits page → PHP runs → DB query → HTML generated.

With page cache:
Saved HTML file is served directly.

This removes:

- PHP execution
- Database queries

Very fast.

Good for:

- Blogs
- Marketing pages

Not good for:

- Dynamic pages (cart, checkout)

## 2️⃣ Object Cache

This caches database query results.

If same query runs many times,
Redis stores result in memory.

Next time:
No DB query needed.

Important for:

- WooCommerce
- Large databases

## 3️⃣ Browser Cache

Browser stores:

- CSS
- JS
- Images

Next visit loads from local storage.

### 📍 Where to optimize

- Caching plugin settings
- Server-level cache
- Redis/Memcached service
- .htaccess or Nginx config

### ⚙️ How to optimize

- Enable page cache
- Exclude dynamic pages
- Enable object cache (Redis)
- Set proper browser cache headers
- Test cache invalidation

### 🎯 Expected Results

- Reduced DB load
- Faster delivery
- Improved scalability
- Faster repeat visits

---

# 🗄 4️⃣ Database Optimization

> Large autoloaded options affect every request.

WordPress uses MySQL.

Main problem areas:

- wp_posts
- wp_postmeta
- wp_options

## 1️⃣ Too many revisions

Each post update creates revision.

Thousands of revisions slow queries.

Limit revisions.

## 2️⃣ Large wp_postmeta table

Meta queries are slow.

If table has millions of rows,
queries become expensive.

Solution:

- Add indexes
- Use custom tables for structured data

## 3️⃣ Autoloaded Options

autoload = yes loads on every request.

Large options increase memory usage.

```sql
SELECT option_name, LENGTH(option_value)
FROM wp_options
WHERE autoload = 'yes'
ORDER BY LENGTH(option_value) DESC;
```

### 📍 Where to optimize

- Database tables
- MySQL configuration
- Slow query logs
- wp_options table

### ⚙️ How to optimize

- Limit revisions
- Remove orphaned metadata
- Clean expired transients
- Add indexes to meta keys
- Move heavy structured data to custom tables
- Audit autoloaded options
- Optimize tables regularly

### 🎯 Expected Results

- Faster queries
- Lower memory usage
- Better scalability
- Reduced server load

---

# 🧩 5️⃣ Theme & Code Optimization

> Performance issues usually come from custom code.

Most performance problems come from bad custom code.

## Common Mistakes:

### ❌ Too many WP_Query calls

Each query = DB call.

### ❌ Nested loops

Loop inside loop increases processing time.

### ❌ SELECT \*

Select only needed fields.

### ❌ Global variables abuse

Increases memory usage.

## Good Practices:

- Reduce duplicate queries
- Cache expensive queries
- Remove unused hooks
- Dequeue unused assets
- Load assets conditionally

### 📍 Where to optimize

- Theme templates
- functions.php
- Custom plugins
- WP_Query usage
- Hooks and filters

### ⚙️ How to optimize

- Reduce unnecessary WP_Query calls
- Avoid nested loops
- Cache heavy queries
- Remove unused actions
- Load scripts conditionally

### 🎯 Expected Results

- Fewer DB queries
- Faster PHP execution
- Cleaner architecture

---

# 🎨 6️⃣ Frontend Optimization

> I optimize based on Core Web Vitals.

User feels speed based on frontend.

## Images

Compress and convert to WebP.
Use correct dimensions.
Enable lazy loading.

## CSS & JS

Minify files.
Remove unused CSS.
Defer non-critical JS.
Avoid render blocking scripts.

## Core Web Vitals

LCP → main content speed
CLS → layout stability
INP → interaction speed

### 📍 Where to optimize

- Media library
- Theme assets
- Enqueue system
- CSS and JS files

### ⚙️ How to optimize

- Compress images
- Use WebP
- Minify CSS/JS
- Inline critical CSS
- Defer JS
- Remove unused styles

### 🎯 Expected Results

- Faster LCP
- Smaller page size
- Better Core Web Vitals
- Improved SEO

---

# 📊 7️⃣ Third-Party Scripts

> Third-party scripts often impact performance more than theme itself.

Examples:

- Analytics
- Facebook Pixel
- Chat widgets

Each adds requests and JS execution.

Solution:

- Load async
- Delay execution
- Remove unused tools

### 📍 Where to optimize

- Header scripts
- Footer scripts
- Tag manager

### ⚙️ How to optimize

- Delay scripts
- Use async/defer
- Load conditionally
- Audit with DevTools

### 🎯 Expected Results

- Faster interaction
- Reduced JS blocking
- Better frontend performance

---

# 🛍 8️⃣ WooCommerce Performance

> WooCommerce requires different caching logic.

WooCommerce is heavy.

Problems:

- Cart fragments
- Heavy queries
- Session handling

Fixes:

- Disable cart fragments if unnecessary
- Cache product queries
- Optimize checkout

### 📍 Where to optimize

- Cart system
- Checkout page
- Product queries

### ⚙️ How to optimize

- Reduce AJAX calls
- Use object cache
- Optimize dynamic filters

### 🎯 Expected Results

- Faster product pages
- Faster checkout
- Higher conversion

---

# 📈 9️⃣ Monitoring & Testing

> I measure first, then optimize.

Use:

- Lighthouse
- PageSpeed Insights
- GTmetrix
- Query Monitor
- Chrome DevTools

Check:

- TTFB
- LCP
- DB queries
- Waterfall

### 📍 Where to optimize

- Before deployment
- After changes
- During traffic spikes

### ⚙️ How to optimize

- Compare before/after metrics
- Analyze bottlenecks
- Monitor server resources

### 🎯 Expected Results

- Data-driven improvements
- Identified weak points
- Continuous optimization

---

# 🔐 🔟 Security & Performance

Bots slow sites.

Solutions:

- Disable XML-RPC
- Limit login attempts
- Use firewall
- Block malicious bots

### 📍 Where to optimize

- WordPress settings
- Server firewall
- Login system

### ⚙️ How to optimize

- Enable rate limiting
- Block suspicious IPs
- Reduce brute-force attempts

### 🎯 Expected Results

- Lower server load
- Improved stability

---

# 🌐 1️⃣1️⃣ REST API Performance

If using REST:

- Cache responses
- Optimize queries
- Validate permissions early

### 📍 Where to optimize

- Custom REST endpoints
- AJAX calls
- Headless frontend

### ⚙️ How to optimize

- Cache REST responses
- Reduce unnecessary API calls
- Optimize DB queries inside endpoints
- Return only required data

### 🎯 Expected Results

- Faster dynamic content
- Reduced server processing
- Better scalability

---

---

---

---

---

# 🔥 1️⃣ A WordPress site has good hosting and caching, but TTFB is still high. What do you check?

### ✅ Strong Answer:

First, I confirm TTFB from multiple tools to avoid false readings.

> **TTFB** - How long it takes from the moment a user requests a page until the browser receives the first byte of response from the server.

When someone opens a website:

1. Browser sends request to server
2. Server processes the request
3. Server sends back response

TTFB measures step 2 + start of 3
It does NOT measure full page load.

It measures:

- Server processing time
- PHP execution
- Database queries
- Backend logic

After this, I check:

- Database query count using Query Monitor
- Slow queries in MySQL slow query log
- Autoloaded options size in wp_options
- External API calls during page load
- Heavy hooks running on init or wp

If TTFB is high even with cache disabled for testing, I suspect:

- Large autoloaded options
- Expensive custom WP_Query
- External API blocking execution

> TTFB problems usually come from backend logic or database, not frontend.

---

# 🔥 2️⃣ How would you optimize a wp_postmeta table with millions of rows?

### ✅ Strong Answer:

wp_postmeta becomes slow when:

- Heavy meta_query is used
- No proper indexes
- Filtering large datasets

My approach:

1. Add index to frequently queried meta_key
2. Avoid complex meta_query with OR conditions
3. Move structured data to custom table
4. Replace meta filtering with taxonomy when possible

> When wp_postmeta grows significantly, architectural changes are required.

---

# 🔥 3️⃣ Why can too many autoloaded options slow down a site?

### ✅ Strong Answer:

All autoload = yes options load on every request.

Even if page doesn’t need them.

If total autoload size is 3MB,
every request loads 3MB into memory.

That increases:

- Memory usage
- Execution time
- TTFB

I audit using SQL and reduce unnecessary autoload entries.

---

# 🔥 4️⃣ How do you debug performance issues in WooCommerce checkout?

### ✅ Strong Answer:

Checkout is dynamic and cannot be fully cached.

I check:

- Number of queries during checkout
- External payment API response time
- Cart fragments AJAX calls
- Session handling
- Third-party plugins adding filters

I use:

- Query Monitor
- DevTools Network tab
- Disable plugins one by one

> WooCommerce checkout performance is usually plugin-related.

---

# 🔥 5️⃣ What is the difference between Page Cache and Object Cache?

### ✅ Strong Answer:

Page Cache:

- Stores full HTML output
- Skips PHP and DB execution
- Best for static content

Object Cache:

- Stores database query results
- Reduces repeated DB calls
- Good for dynamic content

Page cache improves frontend delivery.
Object cache improves backend efficiency.

---

# 🔥 6️⃣ A site loads fast on first visit but becomes slow under traffic spike. Why?

### ✅ Strong Answer:

Possible reasons:

- No object cache
- Database locking
- CPU bottleneck
- PHP-FPM process limits
- No proper server scaling

Under traffic:

- More concurrent DB queries
- Memory exhaustion
- Process queue overload

Solution:

- Enable Redis
- Increase PHP-FPM workers
- Optimize DB queries
- Use CDN

> Scalability problems are infrastructure-related.

---

# 🔥 7️⃣ How do third-party scripts affect Core Web Vitals?

### ✅ Strong Answer:

They increase:

- JavaScript execution time
- Blocking time
- Network requests

Impact on:

- INP (interaction delay)
- LCP (if loaded early)
- CLS (if injected dynamically)

Solution:

- Defer or async load
- Delay until user interaction
- Remove unnecessary tools

---

# 🔥 8️⃣ When would you create a custom database table instead of using postmeta?

### ✅ Strong Answer:

When:

- Dataset is large
- Complex filtering required
- Structured relational data
- Performance critical queries

postmeta is flexible but not optimized for large relational datasets.

Custom tables allow:

- Proper indexing
- Better query performance
- Structured schema

---

# 🔥 9️⃣ Why is disabling unused plugins important for performance?

### ✅ Strong Answer:

Even inactive plugins may:

- Load files
- Register hooks
- Increase memory usage

Active plugins:

- Add queries
- Add hooks
- Load scripts

Each plugin increases execution overhead.

> WordPress performance is often plugin-driven.

---

# 🔥 🔟 How do you approach performance optimization step by step?

### ✅ Strong Answer:

1. Measure baseline performance
2. Check TTFB
3. Analyze DB queries
4. Check autoload size
5. Review theme queries
6. Audit third-party scripts
7. Optimize images and assets
8. Retest and compare

> I never optimize blindly. I measure, improve and validate results.

---

---

---

---

---
