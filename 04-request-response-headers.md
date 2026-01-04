# Request & Response Headers

## 1. Concept & Theory

### Definition

**HTTP Headers** are key-value pairs sent between client and server with every HTTP request and response. They carry metadata about the request or response—information that is not the actual content, but describes how to handle it.

**Request Headers** are sent by the client to the server. They tell the server about the client's capabilities, authentication, and what kind of response it wants.

**Response Headers** are sent by the server to the client. They tell the client about the content being returned, caching rules, security policies, and more.

Headers are invisible to users but critical for:
- Security (preventing attacks)
- Performance (caching, compression)
- Authentication (tokens, cookies)
- Content negotiation (format, language)
- Cross-origin communication (CORS)

### The Problem it Solves

Without headers, HTTP would be a blind exchange:
- Server wouldn't know if client accepts JSON or XML
- No way to authenticate requests
- No caching = reload everything every time
- No security policies = easy to attack
- Cross-origin requests would either always work (dangerous) or never work (limiting)

Headers solve these problems by adding context to every request and response:

```
Without headers:
Client: GET /api/users
Server: Here's... something? In some format? For anyone?

With headers:
Client: GET /api/users
        Accept: application/json
        Authorization: Bearer eyJhbG...
        Origin: https://myapp.com

Server: 200 OK
        Content-Type: application/json
        Cache-Control: max-age=3600
        Access-Control-Allow-Origin: https://myapp.com
```

### Real-World Analogy

Think of **shipping a package**.

The package content is your HTTP body. But the package also has:

**Shipping Label (Request Headers)**:
- Return address (Origin)
- "Fragile" sticker (Content-Type)
- Delivery instructions (Accept)
- Signature required (Authorization)

**Delivery Receipt (Response Headers)**:
- Delivered date (Date)
- Package weight (Content-Length)
- "Keep refrigerated" (Cache-Control)
- Insurance info (Security headers)

Without these labels, the shipping company wouldn't know where the package came from, how to handle it, or where to deliver it.

---

## 2. Visual Architecture (Mermaid)

```mermaid
sequenceDiagram
    participant Browser
    participant Server

    Note over Browser,Server: Simple Request (No Preflight)
    Browser->>Server: GET /api/data<br/>Origin: https://app.com<br/>Accept: application/json
    Server->>Browser: 200 OK<br/>Content-Type: application/json<br/>Access-Control-Allow-Origin: https://app.com

    Note over Browser,Server: Preflight Request (Complex Request)
    Browser->>Server: OPTIONS /api/data<br/>Origin: https://app.com<br/>Access-Control-Request-Method: POST<br/>Access-Control-Request-Headers: Content-Type, Authorization
    Server->>Browser: 204 No Content<br/>Access-Control-Allow-Origin: https://app.com<br/>Access-Control-Allow-Methods: GET, POST<br/>Access-Control-Allow-Headers: Content-Type, Authorization
    Browser->>Server: POST /api/data<br/>Origin: https://app.com<br/>Content-Type: application/json<br/>Authorization: Bearer token
    Server->>Browser: 201 Created<br/>Access-Control-Allow-Origin: https://app.com
```

```mermaid
flowchart TB
    subgraph Request_Headers ["Common Request Headers"]
        RH1["Host: api.example.com"]
        RH2["Accept: application/json"]
        RH3["Content-Type: application/json"]
        RH4["Authorization: Bearer token"]
        RH5["Origin: https://myapp.com"]
        RH6["User-Agent: Mozilla/5.0..."]
        RH7["Cookie: session=abc123"]
        RH8["Cache-Control: no-cache"]
    end

    subgraph Response_Headers ["Common Response Headers"]
        RS1["Content-Type: application/json"]
        RS2["Content-Length: 1234"]
        RS3["Cache-Control: max-age=3600"]
        RS4["Set-Cookie: session=xyz"]
        RS5["Access-Control-Allow-Origin: *"]
        RS6["X-Content-Type-Options: nosniff"]
        RS7["Strict-Transport-Security: max-age=..."]
        RS8["X-Frame-Options: DENY"]
    end
```

---

## 3. How It Works (Step-by-Step)

### Common Request Headers

#### 1. Host
```
Host: api.example.com
```
- **Required** in HTTP/1.1
- Tells the server which website you're requesting
- One server can host multiple domains (virtual hosting)

#### 2. Accept
```
Accept: application/json, text/html;q=0.9, */*;q=0.8
```
- Tells server what **content types** client can handle
- `q=` is quality/preference (0 to 1)
- Server uses this for **content negotiation**

#### 3. Content-Type
```
Content-Type: application/json; charset=utf-8
```
- Describes the format of the **request body**
- Common values:
  - `application/json` - JSON data
  - `application/x-www-form-urlencoded` - Form data
  - `multipart/form-data` - File uploads
  - `text/plain` - Plain text

#### 4. Authorization
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```
- Carries **authentication credentials**
- Common schemes:
  - `Bearer <token>` - JWT or OAuth token
  - `Basic <base64>` - Base64 encoded username:password
  - `Digest` - Challenge-response auth

#### 5. Origin
```
Origin: https://myapp.com
```
- Sent with **cross-origin requests**
- Server uses this to decide if request is allowed (CORS)
- Cannot be faked by JavaScript

#### 6. User-Agent
```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0.0.0
```
- Identifies the client (browser, version, OS)
- Used for analytics, compatibility, bot detection
- Can be spoofed (don't rely on it for security)

#### 7. Cookie
```
Cookie: session=abc123; theme=dark; tracking_id=xyz
```
- Sends **cookies** stored by the browser
- Automatic for same-origin requests
- Controlled by `withCredentials` for cross-origin

#### 8. Cache-Control (Request)
```
Cache-Control: no-cache
```
- `no-cache` - Validate with server before using cache
- `no-store` - Don't cache at all
- `max-age=0` - Treat cache as stale

---

### Common Response Headers

#### 1. Content-Type
```
Content-Type: application/json; charset=utf-8
```
- Describes the format of the **response body**
- Browser uses this to parse the response correctly
- **Critical**: Wrong Content-Type can cause security issues

#### 2. Content-Length
```
Content-Length: 1234
```
- Size of response body in **bytes**
- Helps client know when response is complete
- Used for download progress indicators

#### 3. Cache-Control (Response)
```
Cache-Control: public, max-age=3600, must-revalidate
```
- **public** - Can be cached by browsers and CDNs
- **private** - Only browser can cache (user-specific data)
- **max-age=3600** - Cache for 1 hour (seconds)
- **no-store** - Never cache (sensitive data)
- **must-revalidate** - Check with server when stale

#### 4. Set-Cookie
```
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=86400
```
- Creates or updates a **cookie** in the browser
- Attributes:
  - `HttpOnly` - JavaScript cannot access (prevents XSS theft)
  - `Secure` - Only sent over HTTPS
  - `SameSite=Strict|Lax|None` - Cross-site behavior
  - `Max-Age` - Expiration in seconds
  - `Path` - URL scope for the cookie

#### 5. ETag
```
ETag: "33a64df551425fcc55e4d42a148795d9"
```
- **Fingerprint** of the resource content
- Used for cache validation
- Client sends `If-None-Match: "33a64df..."` on next request
- If unchanged, server returns `304 Not Modified`

#### 6. Location
```
Location: /api/users/123
```
- Used with **3xx redirects** and **201 Created**
- Tells client where to find the resource

---

### Same-Origin Policy (SOP)

**Same-Origin Policy** is a browser security feature that restricts how documents or scripts from one origin can interact with resources from another origin.

**Origin** = Protocol + Host + Port

```
https://example.com:443/path/page.html
  │         │        │
Protocol   Host    Port
```

**Same Origin Examples:**
| URL A | URL B | Same Origin? |
|-------|-------|--------------|
| https://example.com/a | https://example.com/b | ✅ Yes |
| https://example.com | https://api.example.com | ❌ No (different host) |
| https://example.com | http://example.com | ❌ No (different protocol) |
| https://example.com:443 | https://example.com:8080 | ❌ No (different port) |

**What SOP Blocks:**
1. JavaScript cannot read responses from different origins
2. JavaScript cannot access cookies from different origins
3. JavaScript cannot access DOM of different origin iframes

**What SOP Allows:**
1. Embedding images, scripts, stylesheets from any origin
2. Submitting forms to any origin
3. Making requests (but not reading responses)

---

### CORS (Cross-Origin Resource Sharing)

**CORS** is a mechanism that allows servers to relax the Same-Origin Policy. It uses HTTP headers to tell browsers which cross-origin requests are allowed.

#### Simple Requests (No Preflight)

A request is "simple" if ALL of these are true:
- Method is GET, HEAD, or POST
- Only simple headers: Accept, Accept-Language, Content-Language, Content-Type
- Content-Type is only: text/plain, multipart/form-data, or application/x-www-form-urlencoded

```
Browser                                  Server
   │                                        │
   │  GET /api/data                         │
   │  Origin: https://app.com               │
   │ ────────────────────────────────────── │
   │                                        │
   │  200 OK                                │
   │  Access-Control-Allow-Origin: *        │
   │ ◄──────────────────────────────────── │
```

#### Preflight Requests

Any request that doesn't meet "simple" criteria triggers a **preflight** OPTIONS request:

**Triggers for Preflight:**
- Methods: PUT, PATCH, DELETE
- Custom headers: Authorization, X-Custom-Header
- Content-Type: application/json

```
Browser                                  Server
   │                                        │
   │  OPTIONS /api/data (Preflight)         │
   │  Origin: https://app.com               │
   │  Access-Control-Request-Method: POST   │
   │  Access-Control-Request-Headers:       │
   │    Content-Type, Authorization         │
   │ ────────────────────────────────────── │
   │                                        │
   │  204 No Content                        │
   │  Access-Control-Allow-Origin: *        │
   │  Access-Control-Allow-Methods:         │
   │    GET, POST, PUT, DELETE              │
   │  Access-Control-Allow-Headers:         │
   │    Content-Type, Authorization         │
   │  Access-Control-Max-Age: 86400         │
   │ ◄──────────────────────────────────── │
   │                                        │
   │  POST /api/data (Actual Request)       │
   │  Origin: https://app.com               │
   │  Content-Type: application/json        │
   │  Authorization: Bearer token           │
   │ ────────────────────────────────────── │
   │                                        │
   │  201 Created                           │
   │  Access-Control-Allow-Origin: *        │
   │ ◄──────────────────────────────────── │
```

#### CORS Headers

| Header | Direction | Purpose |
|--------|-----------|---------|
| `Origin` | Request | Identifies requesting origin |
| `Access-Control-Allow-Origin` | Response | Which origins are allowed |
| `Access-Control-Allow-Methods` | Response | Which HTTP methods are allowed |
| `Access-Control-Allow-Headers` | Response | Which headers are allowed |
| `Access-Control-Allow-Credentials` | Response | Allow cookies/auth headers |
| `Access-Control-Max-Age` | Response | Cache preflight for N seconds |
| `Access-Control-Expose-Headers` | Response | Which headers JS can read |

---

### Security Headers

#### 1. Strict-Transport-Security (HSTS)
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```
- Forces browser to use **HTTPS only**
- `max-age` - Remember for this many seconds
- `includeSubDomains` - Apply to all subdomains
- `preload` - Include in browser's built-in HSTS list
- **Prevents**: SSL stripping attacks, accidental HTTP requests

#### 2. Content-Security-Policy (CSP)
```
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example.com; style-src 'self' 'unsafe-inline'; img-src *; connect-src 'self' https://api.example.com
```
- Controls which resources can be loaded
- Directives:
  - `default-src` - Fallback for all resource types
  - `script-src` - JavaScript sources
  - `style-src` - CSS sources
  - `img-src` - Image sources
  - `connect-src` - XHR/Fetch/WebSocket targets
  - `frame-ancestors` - Who can embed this page
- **Prevents**: XSS attacks, data injection

#### 3. X-Content-Type-Options
```
X-Content-Type-Options: nosniff
```
- Prevents browser from **MIME sniffing**
- Browser trusts the Content-Type header exactly
- **Prevents**: Attacks where attacker uploads malicious file disguised as image

#### 4. X-Frame-Options
```
X-Frame-Options: DENY
```
- Controls if page can be embedded in iframe
- Values:
  - `DENY` - Never allow framing
  - `SAMEORIGIN` - Only same origin can frame
  - `ALLOW-FROM uri` - Only specific URI (deprecated)
- **Prevents**: Clickjacking attacks
- Note: `Content-Security-Policy: frame-ancestors` is the modern replacement

#### 5. X-XSS-Protection
```
X-XSS-Protection: 1; mode=block
```
- Enables browser's built-in XSS filter
- Values:
  - `0` - Disable filter
  - `1` - Enable filter
  - `1; mode=block` - Block page if XSS detected
- Note: Deprecated in modern browsers (CSP is better)

#### 6. Referrer-Policy
```
Referrer-Policy: strict-origin-when-cross-origin
```
- Controls how much referrer info to send
- Values:
  - `no-referrer` - Never send referrer
  - `same-origin` - Only for same-origin requests
  - `strict-origin` - Send origin only, HTTPS→HTTP blocked
  - `strict-origin-when-cross-origin` - Full URL same-origin, origin only cross-origin

#### 7. Permissions-Policy (formerly Feature-Policy)
```
Permissions-Policy: camera=(), microphone=(), geolocation=(self)
```
- Controls which browser features can be used
- `()` - Disable completely
- `(self)` - Only this origin
- `(*)` - Allow all
- Features: camera, microphone, geolocation, fullscreen, payment, etc.

---

## 4. Code Implementation (MERN Context)

### Project Structure

```
src/
├── server.js
├── app.js
├── middleware/
│   ├── cors.js
│   ├── securityHeaders.js
│   └── cacheControl.js
├── routes/
│   └── apiRoutes.js
└── controllers/
    └── dataController.js
```

### middleware/securityHeaders.js
```javascript
// middleware/securityHeaders.js
// Sets security headers on all responses.
// These headers protect against common web vulnerabilities.

const securityHeaders = (req, res, next) => {
    // HSTS - Force HTTPS for 1 year
    // Only set in production (HTTPS required)
    if (process.env.NODE_ENV === 'production') {
        res.setHeader(
            'Strict-Transport-Security',
            'max-age=31536000; includeSubDomains'
        );
    }

    // Prevent MIME type sniffing
    // Browser will trust Content-Type header exactly
    res.setHeader('X-Content-Type-Options', 'nosniff');

    // Prevent clickjacking by disabling iframes
    res.setHeader('X-Frame-Options', 'DENY');

    // XSS filter (legacy, but doesn't hurt)
    res.setHeader('X-XSS-Protection', '1; mode=block');

    // Control referrer information
    res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');

    // Content Security Policy
    // Customize based on your app's needs
    res.setHeader(
        'Content-Security-Policy',
        [
            "default-src 'self'",
            "script-src 'self'",
            "style-src 'self' 'unsafe-inline'",
            "img-src 'self' data: https:",
            "font-src 'self'",
            "connect-src 'self'",
            "frame-ancestors 'none'"
        ].join('; ')
    );

    // Disable browser features we don't need
    res.setHeader(
        'Permissions-Policy',
        'camera=(), microphone=(), geolocation=()'
    );

    next();
};

module.exports = securityHeaders;
```

### middleware/cors.js
```javascript
// middleware/cors.js
// Custom CORS middleware with fine-grained control.
// For production, always specify allowed origins explicitly.

const corsConfig = {
    // Allowed origins - use environment variable in production
    allowedOrigins: [
        'http://localhost:3000',
        'http://localhost:5173',
        'https://myapp.com',
        'https://www.myapp.com'
    ],

    // Allowed HTTP methods
    allowedMethods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],

    // Allowed request headers
    allowedHeaders: [
        'Content-Type',
        'Authorization',
        'X-Requested-With',
        'Accept',
        'Origin'
    ],

    // Headers that JavaScript can read from response
    exposedHeaders: ['X-Total-Count', 'X-Page-Count'],

    // Allow cookies and auth headers
    credentials: true,

    // Cache preflight for 24 hours
    maxAge: 86400
};

const cors = (req, res, next) => {
    const origin = req.headers.origin;

    // Check if origin is allowed
    if (corsConfig.allowedOrigins.includes(origin)) {
        res.setHeader('Access-Control-Allow-Origin', origin);
    }

    // Allow credentials (cookies, authorization headers)
    if (corsConfig.credentials) {
        res.setHeader('Access-Control-Allow-Credentials', 'true');
    }

    // Expose custom headers to JavaScript
    if (corsConfig.exposedHeaders.length > 0) {
        res.setHeader(
            'Access-Control-Expose-Headers',
            corsConfig.exposedHeaders.join(', ')
        );
    }

    // Handle preflight OPTIONS request
    if (req.method === 'OPTIONS') {
        res.setHeader(
            'Access-Control-Allow-Methods',
            corsConfig.allowedMethods.join(', ')
        );

        res.setHeader(
            'Access-Control-Allow-Headers',
            corsConfig.allowedHeaders.join(', ')
        );

        res.setHeader('Access-Control-Max-Age', corsConfig.maxAge);

        // End preflight request
        return res.status(204).end();
    }

    next();
};

// Alternative: Using the 'cors' npm package (simpler)
// const cors = require('cors');
// app.use(cors({
//     origin: ['https://myapp.com'],
//     methods: ['GET', 'POST', 'PUT', 'DELETE'],
//     allowedHeaders: ['Content-Type', 'Authorization'],
//     credentials: true
// }));

module.exports = cors;
```

### middleware/cacheControl.js
```javascript
// middleware/cacheControl.js
// Sets appropriate caching headers based on route type.

// No caching for API responses (dynamic data)
const noCache = (req, res, next) => {
    res.setHeader('Cache-Control', 'no-store, no-cache, must-revalidate');
    res.setHeader('Pragma', 'no-cache');
    res.setHeader('Expires', '0');
    next();
};

// Cache for static assets (1 year with revalidation)
const cacheStatic = (req, res, next) => {
    // Cache for 1 year, but revalidate
    res.setHeader('Cache-Control', 'public, max-age=31536000, immutable');
    next();
};

// Short cache for semi-dynamic content (5 minutes)
const cacheShort = (req, res, next) => {
    res.setHeader('Cache-Control', 'public, max-age=300, must-revalidate');
    next();
};

// Private cache (only browser, not CDN) - for user-specific data
const cachePrivate = (req, res, next) => {
    res.setHeader('Cache-Control', 'private, max-age=60');
    next();
};

module.exports = { noCache, cacheStatic, cacheShort, cachePrivate };
```

### controllers/dataController.js
```javascript
// controllers/dataController.js
// Demonstrates reading request headers and setting response headers.

const dataController = {
    // GET /api/data - Read and respond with proper headers
    getData: async (req, res) => {
        // Reading common request headers
        const clientInfo = {
            origin: req.headers.origin || 'Same origin',
            userAgent: req.headers['user-agent'],
            acceptLanguage: req.headers['accept-language'],
            contentType: req.headers['content-type'],
            authorization: req.headers.authorization ? 'Present' : 'Missing',
            customHeader: req.headers['x-custom-header'] || 'Not set'
        };

        // Check what format client accepts
        const acceptHeader = req.headers.accept || '';
        let responseFormat = 'json';

        if (acceptHeader.includes('text/html')) {
            responseFormat = 'html';
        } else if (acceptHeader.includes('text/plain')) {
            responseFormat = 'text';
        }

        // Set response headers
        res.setHeader('X-Response-Time', `${Date.now()}ms`);
        res.setHeader('X-Custom-Header', 'Hello from server');

        // ETag for caching
        const data = { message: 'Hello World', timestamp: Date.now() };
        const etag = `"${Buffer.from(JSON.stringify(data)).toString('base64')}"`;
        res.setHeader('ETag', etag);

        // Check if client has cached version
        const clientEtag = req.headers['if-none-match'];
        if (clientEtag === etag) {
            return res.status(304).end(); // Not Modified
        }

        // Respond based on Accept header
        if (responseFormat === 'json') {
            res.setHeader('Content-Type', 'application/json');
            return res.json({
                success: true,
                data,
                clientInfo
            });
        }

        res.setHeader('Content-Type', 'text/plain');
        res.send(`Message: ${data.message}`);
    },

    // POST /api/data - Handle content-type and set Location
    createData: async (req, res) => {
        const contentType = req.headers['content-type'];

        // Validate content type
        if (!contentType || !contentType.includes('application/json')) {
            return res.status(415).json({
                success: false,
                message: 'Unsupported Media Type. Use Content-Type: application/json'
            });
        }

        // Create resource
        const newResource = {
            id: Date.now().toString(),
            ...req.body,
            createdAt: new Date()
        };

        // Set Location header for new resource
        res.setHeader('Location', `/api/data/${newResource.id}`);

        res.status(201).json({
            success: true,
            data: newResource
        });
    }
};

module.exports = dataController;
```

### routes/apiRoutes.js
```javascript
// routes/apiRoutes.js
// API routes with appropriate headers middleware.

const express = require('express');
const router = express.Router();
const dataController = require('../controllers/dataController');
const { noCache, cachePrivate } = require('../middleware/cacheControl');

// Public data - short cache
router.get('/data', dataController.getData);

// User-specific data - private cache
router.get('/user/profile', cachePrivate, (req, res) => {
    res.json({ user: 'data' });
});

// Mutations - no cache
router.post('/data', noCache, dataController.createData);
router.put('/data/:id', noCache, dataController.createData);
router.delete('/data/:id', noCache, (req, res) => {
    res.status(204).end();
});

module.exports = router;
```

### app.js
```javascript
// app.js
// Express app with security and CORS headers.

const express = require('express');
const securityHeaders = require('./middleware/securityHeaders');
const cors = require('./middleware/cors');
const apiRoutes = require('./routes/apiRoutes');

const app = express();

// Apply security headers to ALL responses
app.use(securityHeaders);

// Handle CORS for all routes
app.use(cors);

// Parse JSON bodies
app.use(express.json());

// Log request headers (development only)
if (process.env.NODE_ENV !== 'production') {
    app.use((req, res, next) => {
        console.log('\n--- Request Headers ---');
        console.log('Method:', req.method);
        console.log('Path:', req.path);
        console.log('Origin:', req.headers.origin);
        console.log('Content-Type:', req.headers['content-type']);
        console.log('Authorization:', req.headers.authorization ? 'Present' : 'Missing');
        next();
    });
}

// API routes
app.use('/api', apiRoutes);

// 404 handler
app.use((req, res) => {
    res.status(404).json({ message: 'Not found' });
});

// Error handler
app.use((err, req, res, next) => {
    console.error(err);
    res.status(500).json({ message: 'Internal server error' });
});

module.exports = app;
```

### server.js
```javascript
// server.js
// Entry point for the Express application.

const app = require('./app');

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
    console.log(`API: http://localhost:${PORT}/api/data`);
});
```

---

## 5. Code Explanation

### Why Custom CORS Instead of `cors` Package?

The `cors` npm package is excellent for most cases:

```javascript
const cors = require('cors');
app.use(cors({
    origin: 'https://myapp.com',
    credentials: true
}));
```

Custom middleware is useful when you need:
- Dynamic origin validation (check database)
- Logging of CORS requests
- Different rules for different routes
- Learning how CORS actually works

### Why `Access-Control-Allow-Origin` Can't Be Multiple Values

A common mistake:
```javascript
// WRONG - This doesn't work!
res.setHeader('Access-Control-Allow-Origin', 'https://a.com, https://b.com');
```

The header only accepts:
- A single origin: `https://myapp.com`
- A wildcard: `*` (but then credentials don't work)

To allow multiple origins, check dynamically:
```javascript
const allowed = ['https://a.com', 'https://b.com'];
if (allowed.includes(req.headers.origin)) {
    res.setHeader('Access-Control-Allow-Origin', req.headers.origin);
}
```

### Why Preflight Exists

Preflight protects servers that existed before CORS:

1. Old server expects only same-origin requests
2. Attacker's page sends cross-origin DELETE request
3. Without preflight, server would process it!

Preflight asks the server first: "Do you even understand CORS?"
- Old server ignores OPTIONS → request blocked
- Modern server responds with CORS headers → request allowed

### ETag and Conditional Requests

ETags enable efficient caching:

```
First Request:
GET /api/users → 200 OK, ETag: "abc123", Body: {...}

Second Request:
GET /api/users
If-None-Match: "abc123"
↓
If unchanged: 304 Not Modified (no body, saves bandwidth)
If changed: 200 OK, ETag: "def456", Body: {...}
```

### Content-Type Matters for Security

If you serve user-uploaded files:
```javascript
// DANGEROUS: Browser might execute as HTML
res.send(userUploadedContent);

// SAFE: Force browser to treat as plain text
res.setHeader('Content-Type', 'text/plain');
res.setHeader('X-Content-Type-Options', 'nosniff');
res.send(userUploadedContent);
```

---

## 6. Senior Level Insights

### Best Practices

1. **Always Set Security Headers**
   ```javascript
   // Use helmet package in production
   const helmet = require('helmet');
   app.use(helmet());
   ```

2. **Never Use `Access-Control-Allow-Origin: *` with Credentials**
   ```javascript
   // WRONG - Browser will block this
   res.setHeader('Access-Control-Allow-Origin', '*');
   res.setHeader('Access-Control-Allow-Credentials', 'true');

   // RIGHT - Specific origin
   res.setHeader('Access-Control-Allow-Origin', 'https://myapp.com');
   res.setHeader('Access-Control-Allow-Credentials', 'true');
   ```

3. **Cache Preflight Responses**
   ```javascript
   res.setHeader('Access-Control-Max-Age', '86400'); // 24 hours
   ```

4. **Use Vary Header for Caching**
   ```javascript
   // Tell CDN that response varies by Origin
   res.setHeader('Vary', 'Origin');
   ```

5. **Set Content-Type for All Responses**
   ```javascript
   app.use((req, res, next) => {
       res.setHeader('Content-Type', 'application/json');
       next();
   });
   ```

### Common Mistakes

1. **Exposing Authorization Header in Error Messages**
   ```javascript
   // BAD: Logs sensitive header
   console.log('Request headers:', req.headers);

   // GOOD: Redact sensitive values
   const safeHeaders = { ...req.headers };
   if (safeHeaders.authorization) safeHeaders.authorization = '[REDACTED]';
   console.log('Request headers:', safeHeaders);
   ```

2. **Forgetting CORS for Error Responses**
   ```javascript
   // BAD: No CORS headers on error
   app.use((err, req, res, next) => {
       res.status(500).json({ error: 'Failed' });
   });

   // GOOD: CORS headers set before error handler runs
   ```

3. **Cache-Control on Sensitive Data**
   ```javascript
   // BAD: CDN caches user-specific data
   res.setHeader('Cache-Control', 'public, max-age=3600');
   res.json({ user: 'private data' });

   // GOOD: Private cache only
   res.setHeader('Cache-Control', 'private, no-cache');
   ```

4. **Missing Content-Type on Error**
   ```javascript
   // BAD: Browser might misinterpret
   res.status(400).send('{"error": "bad"}');

   // GOOD: Explicit Content-Type
   res.status(400).json({ error: 'bad' });
   ```

5. **Trusting User-Agent for Security**
   ```javascript
   // BAD: User-Agent can be spoofed
   if (req.headers['user-agent'].includes('Bot')) {
       return res.status(403).send('Bots not allowed');
   }
   ```

### Interview Question

**Q: Explain what happens during a CORS preflight request. Why does it exist?**

**Ideal Answer:**

"A CORS preflight is an OPTIONS request that browsers automatically send before certain cross-origin requests. It happens when a request is considered 'not simple'—meaning it uses methods like PUT or DELETE, includes custom headers like Authorization, or sends JSON as Content-Type.

**What happens:**

1. Browser detects the request would be cross-origin and non-simple
2. Browser sends an OPTIONS request with:
   - `Origin`: The requesting origin
   - `Access-Control-Request-Method`: The actual method (e.g., DELETE)
   - `Access-Control-Request-Headers`: Custom headers being sent
3. Server responds with:
   - `Access-Control-Allow-Origin`: Allowed origin(s)
   - `Access-Control-Allow-Methods`: Allowed methods
   - `Access-Control-Allow-Headers`: Allowed headers
   - `Access-Control-Max-Age`: How long to cache this preflight
4. If server's response permits the request, browser sends the actual request
5. If not permitted, browser blocks and throws a CORS error

**Why it exists:**

Preflight protects legacy servers that were built before CORS existed. These servers only expected same-origin requests. Without preflight, a malicious site could send DELETE requests to old APIs that would execute them. Preflight asks permission first—old servers that don't understand CORS will either not respond to OPTIONS or won't include the right headers, so the browser blocks the request.

**Optimization:**

Preflight adds latency (extra round trip). To minimize impact:
1. Use `Access-Control-Max-Age` to cache preflight results (up to 24 hours)
2. Design APIs to use simple requests where possible
3. Combine endpoints to reduce total requests"

---

## Summary

HTTP headers are the invisible backbone of web communication. They control security, caching, authentication, and cross-origin behavior.

**Request Headers to Know:**
| Header | Purpose |
|--------|---------|
| `Authorization` | Send auth token |
| `Content-Type` | Format of request body |
| `Accept` | What formats client wants |
| `Origin` | For CORS |
| `Cookie` | Send cookies |

**Response Headers to Know:**
| Header | Purpose |
|--------|---------|
| `Content-Type` | Format of response |
| `Cache-Control` | Caching rules |
| `Set-Cookie` | Create cookies |
| `Access-Control-*` | CORS permissions |
| `Strict-Transport-Security` | Force HTTPS |
| `Content-Security-Policy` | Prevent XSS |

**Key Concepts:**
- **Same-Origin Policy**: Browsers block cross-origin reads by default
- **CORS**: Server tells browser which cross-origin requests are OK
- **Preflight**: OPTIONS request before non-simple cross-origin requests
- **Security Headers**: Protect against XSS, clickjacking, MIME sniffing

**Golden Rules:**
1. Always set security headers in production
2. Never use `*` with credentials
3. Cache preflight responses for performance
4. Use `helmet` package for security headers
5. Set `Content-Type` on every response
