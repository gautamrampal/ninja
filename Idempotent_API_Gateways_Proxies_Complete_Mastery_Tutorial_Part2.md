# Idempotent API, API Gateways and Proxies - Complete Mastery Tutorial (Part 2)

> **Continuation: Advanced Patterns, Security, and Production Deployment**

---

## Request/Response Transformation and Routing (Continued)

### Request Body Transformation

```javascript
// Transform request body format
app.post('/api/legacy/users', express.json(), async (req, res, next) => {
  // Transform modern API format to legacy format
  const legacyFormat = {
    user_name: req.body.username,        // camelCase to snake_case
    email_address: req.body.email,
    phone_number: req.body.phoneNumber,
    creation_date: new Date().toISOString()
  };
  
  // Replace body with transformed data
  req.body = legacyFormat;
  next();
});

// XML to JSON transformation
const xml2js = require('xml2js');

app.post('/api/xml-endpoint', express.text({ type: 'application/xml' }), async (req, res, next) => {
  try {
    const parser = new xml2js.Parser();
    const jsonData = await parser.parseStringPromise(req.body);
    
    // Replace XML body with JSON
    req.body = jsonData;
    req.headers['content-type'] = 'application/json';
    
    next();
  } catch (error) {
    res.status(400).json({ error: 'Invalid XML' });
  }
});
```

### Response Transformation

```javascript
// Transform backend response to client format
app.get('/api/users/:id', async (req, res) => {
  // Get data from backend service
  const backendResponse = await fetch(`http://backend/users/${req.params.id}`);
  const backendData = await backendResponse.json();
  
  // Transform response format
  const clientFormat = {
    id: backendData.user_id,
    profile: {
      name: backendData.full_name,
      email: backendData.email_address,
      phone: backendData.phone_number
    },
    meta: {
      createdAt: backendData.creation_date,
      updatedAt: backendData.last_modified
    }
  };
  
  res.json(clientFormat);
});

// Response wrapping middleware
app.use((req, res, next) => {
  const originalJson = res.json.bind(res);
  
  res.json = function(data) {
    // Wrap all responses in standard envelope
    const wrapped = {
      success: true,
      timestamp: new Date().toISOString(),
      data: data,
      meta: {
        requestId: req.headers['x-request-id'],
        responseTime: Date.now() - req.startTime
      }
    };
    
    return originalJson(wrapped);
  };
  
  req.startTime = Date.now();
  next();
});
```

### NGINX Request Transformation

```nginx
# /etc/nginx/nginx.conf
server {
    listen 80;
    
    # Add custom headers to requests
    location /api/ {
        proxy_pass http://backend;
        
        # Add/modify request headers
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Request-Time $msec;
        proxy_set_header X-Request-ID $request_id;
        
        # Remove headers
        proxy_set_header Authorization "";  # Remove auth for internal service
    }
    
    # Rewrite URL paths
    location /v1/users/ {
        rewrite ^/v1/users/(.*)$ /api/users/$1 break;
        proxy_pass http://backend;
    }
    
    # Transform query parameters to headers
    location /api/search {
        set $api_key "";
        if ($arg_api_key) {
            set $api_key $arg_api_key;
        }
        
        proxy_set_header X-API-Key $api_key;
        proxy_pass http://backend;
    }
}
```

### Content Negotiation

```javascript
// Serve different formats based on Accept header
app.get('/api/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  // Check Accept header
  const acceptHeader = req.headers.accept || 'application/json';
  
  if (acceptHeader.includes('application/xml')) {
    // Return XML
    const xml = `
      <?xml version="1.0"?>
      <user>
        <id>${user.id}</id>
        <name>${user.name}</name>
        <email>${user.email}</email>
      </user>
    `;
    res.set('Content-Type', 'application/xml');
    return res.send(xml);
  }
  
  if (acceptHeader.includes('text/csv')) {
    // Return CSV
    const csv = `id,name,email\n${user.id},${user.name},${user.email}`;
    res.set('Content-Type', 'text/csv');
    res.set('Content-Disposition', `attachment; filename="user-${user.id}.csv"`);
    return res.send(csv);
  }
  
  // Default to JSON
  res.json(user);
});
```

### Dynamic Routing

```javascript
// Route requests to different backends based on criteria
class DynamicRouter {
  constructor() {
    this.routes = [];
  }
  
  /**
   * Add routing rule
   * @param {object} rule - Routing rule configuration
   */
  addRule(rule) {
    this.routes.push(rule);
  }
  
  /**
   * Find matching backend for request
   * @param {object} req - Express request object
   * @returns {string} Backend URL
   */
  findBackend(req) {
    for (const rule of this.routes) {
      if (this.matchesRule(req, rule)) {
        return rule.backend;
      }
    }
    
    return rule.defaultBackend;
  }
  
  matchesRule(req, rule) {
    // Match by path pattern
    if (rule.pathPattern && !req.path.match(rule.pathPattern)) {
      return false;
    }
    
    // Match by method
    if (rule.method && req.method !== rule.method) {
      return false;
    }
    
    // Match by header
    if (rule.header) {
      const headerValue = req.headers[rule.header.name.toLowerCase()];
      if (headerValue !== rule.header.value) {
        return false;
      }
    }
    
    // Match by query parameter
    if (rule.queryParam) {
      const paramValue = req.query[rule.queryParam.name];
      if (paramValue !== rule.queryParam.value) {
        return false;
      }
    }
    
    // Match by user attribute
    if (rule.userAttribute && req.user) {
      const attrValue = req.user[rule.userAttribute.name];
      if (attrValue !== rule.userAttribute.value) {
        return false;
      }
    }
    
    return true;
  }
}

// Usage
const router = new DynamicRouter();

// Route beta users to beta backend
router.addRule({
  userAttribute: { name: 'beta', value: true },
  backend: 'http://beta-backend:3000'
});

// Route mobile clients to mobile-optimized backend
router.addRule({
  header: { name: 'User-Agent', value: /Mobile/ },
  backend: 'http://mobile-backend:3000'
});

// Route high-priority requests to dedicated backend
router.addRule({
  header: { name: 'X-Priority', value: 'high' },
  backend: 'http://priority-backend:3000'
});

// Default backend
router.defaultBackend = 'http://default-backend:3000';

// Middleware
app.use((req, res, next) => {
  const backend = router.findBackend(req);
  req.backend = backend;
  next();
});
```

### A/B Testing with Gateway

```javascript
// A/B testing router
class ABTestRouter {
  constructor(experimentName, variants) {
    this.experimentName = experimentName;
    this.variants = variants;  // { variantA: { weight: 0.5, backend: '...' }, ... }
  }
  
  /**
   * Assign user to variant
   * @param {string} userId - User identifier
   * @returns {object} Variant configuration
   */
  getVariant(userId) {
    // Consistent hashing - same user always gets same variant
    const hash = this.hashString(`${this.experimentName}:${userId}`);
    const normalizedHash = hash / 0xFFFFFFFF;  // 0-1
    
    let cumulative = 0;
    for (const [variantName, config] of Object.entries(this.variants)) {
      cumulative += config.weight;
      if (normalizedHash < cumulative) {
        return { variant: variantName, ...config };
      }
    }
    
    // Fallback to first variant
    const firstVariant = Object.keys(this.variants)[0];
    return { variant: firstVariant, ...this.variants[firstVariant] };
  }
  
  hashString(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32bit integer
    }
    return Math.abs(hash);
  }
}

// Usage
const abTest = new ABTestRouter('checkout-flow-v2', {
  control: { weight: 0.5, backend: 'http://backend-v1:3000' },
  variant: { weight: 0.5, backend: 'http://backend-v2:3000' }
});

app.use('/api/checkout', authenticate, (req, res, next) => {
  const userId = req.user.id;
  const assignment = abTest.getVariant(userId);
  
  // Add variant to request for logging/analytics
  req.abTestVariant = assignment.variant;
  req.backend = assignment.backend;
  
  // Add header for client
  res.setHeader('X-AB-Test-Variant', assignment.variant);
  
  next();
});
```

### Protocol Translation

```javascript
// REST to GraphQL translation
const { graphql, buildSchema } = require('graphql');

const schema = buildSchema(`
  type User {
    id: ID!
    name: String!
    email: String!
  }
  
  type Query {
    user(id: ID!): User
  }
`);

app.get('/api/users/:id', async (req, res) => {
  // Translate REST request to GraphQL query
  const query = `
    query {
      user(id: "${req.params.id}") {
        id
        name
        email
      }
    }
  `;
  
  const result = await graphql(schema, query);
  
  if (result.errors) {
    return res.status(500).json({ errors: result.errors });
  }
  
  res.json(result.data.user);
});

// gRPC to REST translation
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

const packageDefinition = protoLoader.loadSync('user.proto');
const userProto = grpc.loadPackageDefinition(packageDefinition).user;

const grpcClient = new userProto.UserService(
  'localhost:50051',
  grpc.credentials.createInsecure()
);

app.get('/api/users/:id', (req, res) => {
  // Translate REST to gRPC call
  grpcClient.GetUser({ id: req.params.id }, (error, response) => {
    if (error) {
      return res.status(500).json({ error: error.message });
    }
    
    res.json(response);
  });
});
```

---

## API Versioning and Documentation

### URL-Based Versioning

```javascript
// Version 1 API
app.use('/api/v1/users', (req, res) => {
  res.json({
    id: 1,
    name: 'John Doe',
    email: 'john@example.com'
  });
});

// Version 2 API - different response structure
app.use('/api/v2/users', (req, res) => {
  res.json({
    user: {
      id: 1,
      profile: {
        fullName: 'John Doe',
        contactEmail: 'john@example.com'
      }
    }
  });
});
```

### Header-Based Versioning

```javascript
// Version from Accept header
app.get('/api/users/:id', async (req, res) => {
  const version = req.headers['accept-version'] || '1.0';
  
  const user = await User.findById(req.params.id);
  
  if (version === '1.0') {
    return res.json({
      id: user.id,
      name: user.name,
      email: user.email
    });
  }
  
  if (version === '2.0') {
    return res.json({
      user: {
        id: user.id,
        profile: {
          fullName: user.name,
          contactEmail: user.email
        },
        metadata: {
          createdAt: user.createdAt,
          updatedAt: user.updatedAt
        }
      }
    });
  }
  
  res.status(400).json({ error: 'Unsupported API version' });
});
```

### Content-Type Versioning

```javascript
// Version via content type
app.get('/api/users/:id', async (req, res) => {
  const contentType = req.headers.accept;
  
  const user = await User.findById(req.params.id);
  
  if (contentType.includes('application/vnd.api.v1+json')) {
    return res.json({ /* v1 format */ });
  }
  
  if (contentType.includes('application/vnd.api.v2+json')) {
    return res.json({ /* v2 format */ });
  }
  
  // Default version
  res.json(user);
});
```

### Gateway-Level Version Routing

```nginx
# NGINX version routing
server {
    listen 80;
    
    # Route based on version in path
    location ~ ^/api/v1/(.*)$ {
        proxy_pass http://backend-v1/$1;
    }
    
    location ~ ^/api/v2/(.*)$ {
        proxy_pass http://backend-v2/$1;
    }
    
    # Route based on custom header
    location /api/ {
        set $backend "backend-v1";
        
        if ($http_api_version = "2.0") {
            set $backend "backend-v2";
        }
        
        proxy_pass http://$backend;
    }
}
```

### OpenAPI/Swagger Documentation

```javascript
// Generate OpenAPI documentation
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const swaggerOptions = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'Microservices API',
      version: '1.0.0',
      description: 'Comprehensive API documentation',
      contact: {
        name: 'API Support',
        email: 'support@api.com'
      }
    },
    servers: [
      {
        url: 'https://api.example.com',
        description: 'Production server'
      },
      {
        url: 'https://staging-api.example.com',
        description: 'Staging server'
      }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        },
        apiKey: {
          type: 'apiKey',
          in: 'header',
          name: 'X-API-Key'
        }
      }
    }
  },
  apis: ['./routes/*.js']
};

const swaggerSpec = swaggerJsdoc(swaggerOptions);

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));

/**
 * @swagger
 * /api/users:
 *   get:
 *     summary: Retrieve list of users
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 10
 *         description: Number of users to return
 *       - in: query
 *         name: offset
 *         schema:
 *           type: integer
 *           default: 0
 *         description: Number of users to skip
 *     responses:
 *       200:
 *         description: List of users
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 users:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 *                 total:
 *                   type: integer
 *       401:
 *         description: Unauthorized
 */
app.get('/api/users', authenticate, async (req, res) => {
  // Implementation
});

/**
 * @swagger
 * components:
 *   schemas:
 *     User:
 *       type: object
 *       required:
 *         - id
 *         - email
 *       properties:
 *         id:
 *           type: string
 *           description: User ID
 *         email:
 *           type: string
 *           format: email
 *         name:
 *           type: string
 */
```

### API Deprecation Strategy

```javascript
// Deprecation warning middleware
function deprecationWarning(version, sunsetDate, alternativeEndpoint) {
  return (req, res, next) => {
    res.setHeader('Warning', `299 - "Deprecated API version ${version}"`);
    res.setHeader('Sunset', sunsetDate);
    res.setHeader('Link', `<${alternativeEndpoint}>; rel="alternate"`);
    
    // Log deprecation usage for analytics
    console.warn(`Deprecated API v${version} accessed: ${req.path}`);
    
    next();
  };
}

// Apply to deprecated endpoints
app.use(
  '/api/v1/users',
  deprecationWarning('1.0', '2024-12-31', '/api/v2/users'),
  v1UsersHandler
);
```

---

## Authentication and Authorization at Gateway Level

### JWT Authentication

```javascript
const jwt = require('jsonwebtoken');

/**
 * JWT authentication middleware
 */
function jwtAuth(req, res, next) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({
      error: 'Unauthorized',
      message: 'No token provided'
    });
  }
  
  const token = authHeader.substring(7);
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Attach user to request
    req.user = decoded;
    
    // Add user context to headers for backend services
    req.headers['x-user-id'] = decoded.userId;
    req.headers['x-user-role'] = decoded.role;
    req.headers['x-user-email'] = decoded.email;
    
    next();
    
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({
        error: 'Unauthorized',
        message: 'Token expired'
      });
    }
    
    return res.status(401).json({
      error: 'Unauthorized',
      message: 'Invalid token'
    });
  }
}

// Apply to routes
app.use('/api/protected/', jwtAuth);

// Issue JWT tokens
app.post('/api/auth/login', async (req, res) => {
  const { email, password } = req.body;
  
  // Verify credentials
  const user = await User.findOne({ email });
  if (!user || !await user.verifyPassword(password)) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Generate JWT
  const token = jwt.sign(
    {
      userId: user.id,
      email: user.email,
      role: user.role
    },
    process.env.JWT_SECRET,
    {
      expiresIn: '1h',
      issuer: 'api-gateway',
      audience: 'microservices'
    }
  );
  
  // Generate refresh token
  const refreshToken = jwt.sign(
    { userId: user.id },
    process.env.REFRESH_TOKEN_SECRET,
    { expiresIn: '7d' }
  );
  
  res.json({
    accessToken: token,
    refreshToken: refreshToken,
    expiresIn: 3600
  });
});
```

### API Key Authentication

```javascript
class ApiKeyManager {
  constructor(redis) {
    this.redis = redis;
  }
  
  /**
   * Generate new API key
   */
  async generateKey(userId, permissions = []) {
    const apiKey = `ak_${crypto.randomBytes(32).toString('hex')}`;
    
    await this.redis.hmset(`apikey:${apiKey}`, {
      userId,
      permissions: JSON.stringify(permissions),
      createdAt: Date.now(),
      lastUsed: Date.now()
    });
    
    return apiKey;
  }
  
  /**
   * Validate API key
   */
  async validateKey(apiKey) {
    const data = await this.redis.hgetall(`apikey:${apiKey}`);
    
    if (!data || !data.userId) {
      return null;
    }
    
    // Update last used timestamp
    await this.redis.hset(`apikey:${apiKey}`, 'lastUsed', Date.now());
    
    return {
      userId: data.userId,
      permissions: JSON.parse(data.permissions || '[]')
    };
  }
  
  /**
   * Revoke API key
   */
  async revokeKey(apiKey) {
    await this.redis.del(`apikey:${apiKey}`);
  }
}

// Middleware
const apiKeyManager = new ApiKeyManager(redis);

async function apiKeyAuth(req, res, next) {
  const apiKey = req.headers['x-api-key'];
  
  if (!apiKey) {
    return res.status(401).json({ error: 'API key required' });
  }
  
  const keyData = await apiKeyManager.validateKey(apiKey);
  
  if (!keyData) {
    return res.status(401).json({ error: 'Invalid API key' });
  }
  
  req.user = keyData;
  next();
}
```

### OAuth 2.0 Integration

```javascript
const oauth2 = require('simple-oauth2');

const oauth2Client = oauth2.create({
  client: {
    id: process.env.OAUTH_CLIENT_ID,
    secret: process.env.OAUTH_CLIENT_SECRET
  },
  auth: {
    tokenHost: 'https://oauth-provider.com',
    tokenPath: '/oauth/token',
    authorizePath: '/oauth/authorize'
  }
});

// OAuth middleware
async function oauthAuth(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    // Verify token with OAuth provider
    const response = await fetch('https://oauth-provider.com/oauth/verify', {
      headers: {
        Authorization: `Bearer ${token}`
      }
    });
    
    if (!response.ok) {
      return res.status(401).json({ error: 'Invalid token' });
    }
    
    const userData = await response.json();
    req.user = userData;
    
    next();
    
  } catch (error) {
    res.status(401).json({ error: 'Authentication failed' });
  }
}
```

### Role-Based Access Control (RBAC)

```javascript
/**
 * RBAC middleware
 * @param {string[]} allowedRoles - Roles that can access the endpoint
 */
function requireRole(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    const userRole = req.user.role;
    
    if (!allowedRoles.includes(userRole)) {
      return res.status(403).json({
        error: 'Forbidden',
        message: `Requires one of: ${allowedRoles.join(', ')}`
      });
    }
    
    next();
  };
}

// Usage
app.get('/api/admin/users', jwtAuth, requireRole('admin', 'superadmin'), async (req, res) => {
  // Only admins can access
});

app.post('/api/posts', jwtAuth, requireRole('user', 'admin'), async (req, res) => {
  // Users and admins can create posts
});
```

### Permission-Based Access Control

```javascript
class PermissionManager {
  constructor() {
    this.permissions = {
      'users:read': ['user', 'admin'],
      'users:write': ['admin'],
      'users:delete': ['admin'],
      'posts:read': ['guest', 'user', 'admin'],
      'posts:write': ['user', 'admin'],
      'posts:delete': ['user', 'admin'],
      'admin:access': ['admin']
    };
  }
  
  /**
   * Check if role has permission
   */
  hasPermission(role, permission) {
    const allowedRoles = this.permissions[permission];
    return allowedRoles && allowedRoles.includes(role);
  }
}

const permissionManager = new PermissionManager();

/**
 * Permission middleware
 * @param {string} permission - Required permission
 */
function requirePermission(permission) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    if (!permissionManager.hasPermission(req.user.role, permission)) {
      return res.status(403).json({
        error: 'Forbidden',
        message: `Requires permission: ${permission}`
      });
    }
    
    next();
  };
}

// Usage
app.get('/api/users', jwtAuth, requirePermission('users:read'), getUsers);
app.post('/api/users', jwtAuth, requirePermission('users:write'), createUser);
app.delete('/api/users/:id', jwtAuth, requirePermission('users:delete'), deleteUser);
```

### Kong Authentication Plugins

```yaml
# kong.yml
services:
  - name: user-service
    url: http://user-service:3000
    routes:
      - name: user-routes
        paths:
          - /api/users
    plugins:
      # JWT authentication
      - name: jwt
        config:
          claims_to_verify:
            - exp
          key_claim_name: iss
          secret_is_base64: false
      
      # ACL (Access Control List)
      - name: acl
        config:
          allow:
            - admin
            - user
      
      # Rate limiting per consumer
      - name: rate-limiting
        config:
          minute: 100
          policy: local

# Create consumer with JWT credential
consumers:
  - username: webapp
    jwt_credentials:
      - algorithm: HS256
        key: webapp-key
        secret: super-secret-key
    acls:
      - group: user
```

---

## Circuit Breaking and Fallback Mechanisms

### Circuit Breaker Pattern

```javascript
/**
 * Circuit Breaker Implementation
 * States: CLOSED (normal) -> OPEN (failing) -> HALF_OPEN (testing)
 */
class CircuitBreaker {
  constructor(options = {}) {
    this.failureThreshold = options.failureThreshold || 5;
    this.successThreshold = options.successThreshold || 2;
    this.timeout = options.timeout || 60000;  // 60 seconds
    this.monitoringPeriod = options.monitoringPeriod || 10000;  // 10 seconds
    
    this.state = 'CLOSED';
    this.failureCount = 0;
    this.successCount = 0;
    this.nextAttempt = Date.now();
    this.stats = {
      totalRequests: 0,
      successfulRequests: 0,
      failedRequests: 0,
      rejectedRequests: 0
    };
  }
  
  /**
   * Execute function with circuit breaker protection
   * @param {Function} fn - Async function to execute
   * @param {Function} fallback - Fallback function if circuit is open
   * @returns {Promise<any>}
   */
  async execute(fn, fallback = null) {
    this.stats.totalRequests++;
    
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        this.stats.rejectedRequests++;
        
        if (fallback) {
          return await fallback();
        }
        
        throw new Error('Circuit breaker is OPEN');
      }
      
      // Transition to HALF_OPEN
      this.state = 'HALF_OPEN';
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
      
    } catch (error) {
      this.onFailure();
      
      if (fallback) {
        return await fallback();
      }
      
      throw error;
    }
  }
  
  onSuccess() {
    this.stats.successfulRequests++;
    this.failureCount = 0;
    
    if (this.state === 'HALF_OPEN') {
      this.successCount++;
      
      if (this.successCount >= this.successThreshold) {
        this.state = 'CLOSED';
        this.successCount = 0;
        console.log('Circuit breaker transitioned to CLOSED');
      }
    }
  }
  
  onFailure() {
    this.stats.failedRequests++;
    this.failureCount++;
    this.successCount = 0;
    
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
      console.error(`Circuit breaker OPEN. Next attempt at ${new Date(this.nextAttempt)}`);
    }
  }
  
  getStats() {
    return {
      state: this.state,
      ...this.stats,
      successRate: this.stats.totalRequests > 0 
        ? (this.stats.successfulRequests / this.stats.totalRequests * 100).toFixed(2) + '%'
        : '0%'
    };
  }
  
  reset() {
    this.state = 'CLOSED';
    this.failureCount = 0;
    this.successCount = 0;
  }
}

// Usage
const userServiceBreaker = new CircuitBreaker({
  failureThreshold: 5,
  successThreshold: 2,
  timeout: 60000
});

app.get('/api/users/:id', async (req, res) => {
  try {
    const user = await userServiceBreaker.execute(
      // Primary function
      async () => {
        const response = await fetch(`http://user-service/users/${req.params.id}`);
        if (!response.ok) throw new Error('Service unavailable');
        return await response.json();
      },
      // Fallback function
      async () => {
        // Return cached data or default response
        const cached = await redis.get(`user:${req.params.id}`);
        if (cached) return JSON.parse(cached);
        
        return {
          id: req.params.id,
          name: 'Unknown',
          message: 'Service temporarily unavailable'
        };
      }
    );
    
    res.json(user);
    
  } catch (error) {
    res.status(503).json({
      error: 'Service Unavailable',
      message: error.message
    });
  }
});

// Health endpoint showing circuit breaker states
app.get('/health/circuit-breakers', (req, res) => {
  res.json({
    userService: userServiceBreaker.getStats()
  });
});
```

### Advanced Circuit Breaker with Metrics

```javascript
class AdvancedCircuitBreaker extends CircuitBreaker {
  constructor(options = {}) {
    super(options);
    this.errorRateThreshold = options.errorRateThreshold || 0.5;  // 50%
    this.volumeThreshold = options.volumeThreshold || 10;
    this.requestLog = [];
  }
  
  async execute(fn, fallback = null) {
    const startTime = Date.now();
    
    try {
      const result = await super.execute(fn, fallback);
      this.recordRequest(true, Date.now() - startTime);
      return result;
      
    } catch (error) {
      this.recordRequest(false, Date.now() - startTime);
      throw error;
    }
  }
  
  recordRequest(success, duration) {
    const now = Date.now();
    
    this.requestLog.push({
      success,
      duration,
      timestamp: now
    });
    
    // Keep only recent requests within monitoring period
    const cutoff = now - this.monitoringPeriod;
    this.requestLog = this.requestLog.filter(r => r.timestamp > cutoff);
    
    // Check error rate
    this.checkErrorRate();
  }
  
  checkErrorRate() {
    const totalRequests = this.requestLog.length;
    
    if (totalRequests < this.volumeThreshold) {
      return;  // Not enough data
    }
    
    const failedRequests = this.requestLog.filter(r => !r.success).length;
    const errorRate = failedRequests / totalRequests;
    
    if (errorRate >= this.errorRateThreshold && this.state === 'CLOSED') {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
      console.error(`Circuit breaker OPEN due to high error rate: ${(errorRate * 100).toFixed(2)}%`);
    }
  }
  
  getMetrics() {
    const totalRequests = this.requestLog.length;
    const successful = this.requestLog.filter(r => r.success).length;
    const failed = totalRequests - successful;
    
    const durations = this.requestLog.map(r => r.duration);
    const avgDuration = durations.length > 0
      ? durations.reduce((a, b) => a + b, 0) / durations.length
      : 0;
    
    return {
      state: this.state,
      totalRequests,
      successful,
      failed,
      errorRate: totalRequests > 0 ? (failed / totalRequests * 100).toFixed(2) + '%' : '0%',
      avgResponseTime: avgDuration.toFixed(2) + 'ms'
    };
  }
}
```

### Bulkhead Pattern

```javascript
/**
 * Bulkhead pattern - limit concurrent requests to prevent resource exhaustion
 */
class Bulkhead {
  constructor(maxConcurrent = 10) {
    this.maxConcurrent = maxConcurrent;
    this.currentConcurrent = 0;
    this.queue = [];
    this.stats = {
      accepted: 0,
      rejected: 0,
      queued: 0
    };
  }
  
  async execute(fn) {
    // Reject if at capacity and queue is full
    if (this.currentConcurrent >= this.maxConcurrent) {
      this.stats.rejected++;
      throw new Error('Bulkhead capacity exceeded');
    }
    
    this.currentConcurrent++;
    this.stats.accepted++;
    
    try {
      const result = await fn();
      return result;
    } finally {
      this.currentConcurrent--;
    }
  }
  
  getStats() {
    return {
      maxConcurrent: this.maxConcurrent,
      currentConcurrent: this.currentConcurrent,
      ...this.stats
    };
  }
}

// Usage
const paymentBulkhead = new Bulkhead(5);  // Max 5 concurrent payment requests

app.post('/api/payments', async (req, res) => {
  try {
    const result = await paymentBulkhead.execute(async () => {
      return await processPayment(req.body);
    });
    
    res.json(result);
    
  } catch (error) {
    if (error.message === 'Bulkhead capacity exceeded') {
      return res.status(503).json({
        error: 'Service Busy',
        message: 'Too many concurrent requests, please try again'
      });
    }
    
    res.status(500).json({ error: error.message });
  }
});
```

### Timeout Pattern

```javascript
/**
 * Timeout wrapper
 * @param {Function} fn - Async function to execute
 * @param {number} timeoutMs - Timeout in milliseconds
 * @returns {Promise<any>}
 */
async function withTimeout(fn, timeoutMs) {
  return Promise.race([
    fn(),
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Request timeout')), timeoutMs)
    )
  ]);
}

// Usage
app.get('/api/slow-service', async (req, res) => {
  try {
    const result = await withTimeout(
      async () => await fetch('http://slow-service/data'),
      5000  // 5 second timeout
    );
    
    res.json(result);
    
  } catch (error) {
    if (error.message === 'Request timeout') {
      return res.status(504).json({ error: 'Gateway Timeout' });
    }
    
    res.status(500).json({ error: error.message });
  }
});
```

### Retry Pattern with Exponential Backoff

```javascript
/**
 * Retry with exponential backoff
 * @param {Function} fn - Function to retry
 * @param {object} options - Retry options
 * @returns {Promise<any>}
 */
async function retryWithBackoff(fn, options = {}) {
  const {
    maxRetries = 3,
    initialDelay = 1000,
    maxDelay = 10000,
    backoffMultiplier = 2,
    shouldRetry = (error) => true
  } = options;
  
  let lastError;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
      
    } catch (error) {
      lastError = error;
      
      if (attempt === maxRetries || !shouldRetry(error)) {
        throw error;
      }
      
      // Calculate delay with exponential backoff
      const delay = Math.min(
        initialDelay * Math.pow(backoffMultiplier, attempt),
        maxDelay
      );
      
      // Add jitter to prevent thundering herd
      const jitter = Math.random() * 0.1 * delay;
      const totalDelay = delay + jitter;
      
      console.log(`Retry attempt ${attempt + 1}/${maxRetries} after ${totalDelay.toFixed(0)}ms`);
      
      await new Promise(resolve => setTimeout(resolve, totalDelay));
    }
  }
  
  throw lastError;
}

// Usage
app.get('/api/unreliable-service', async (req, res) => {
  try {
    const data = await retryWithBackoff(
      async () => {
        const response = await fetch('http://unreliable-service/data');
        if (!response.ok) throw new Error('Service error');
        return await response.json();
      },
      {
        maxRetries: 3,
        initialDelay: 1000,
        shouldRetry: (error) => {
          // Only retry on network errors or 5xx status codes
          return error.message.includes('network') || 
                 error.message.includes('Service error');
        }
      }
    );
    
    res.json(data);
    
  } catch (error) {
    res.status(503).json({ error: 'Service unavailable after retries' });
  }
});
```

---

## Caching Strategies at the Proxy Layer

### In-Memory Cache

```javascript
class MemoryCache {
  constructor(ttl = 60000) {  // Default 60 seconds
    this.cache = new Map();
    this.ttl = ttl;
  }
  
  /**
   * Get cached value
   * @param {string} key
   * @returns {any|null}
   */
  get(key) {
    const item = this.cache.get(key);
    
    if (!item) return null;
    
    // Check expiration
    if (Date.now() > item.expiresAt) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value;
  }
  
  /**
   * Set cache value
   * @param {string} key
   * @param {any} value
   * @param {number} customTtl - Optional custom TTL
   */
  set(key, value, customTtl = null) {
    this.cache.set(key, {
      value,
      expiresAt: Date.now() + (customTtl || this.ttl)
    });
  }
  
  /**
   * Delete cache entry
   */
  delete(key) {
    this.cache.delete(key);
  }
  
  /**
   * Clear entire cache
   */
  clear() {
    this.cache.clear();
  }
  
  /**
   * Cleanup expired entries
   */
  cleanup() {
    const now = Date.now();
    for (const [key, item] of this.cache.entries()) {
      if (now > item.expiresAt) {
        this.cache.delete(key);
      }
    }
  }
}

// Periodic cleanup
const cache = new MemoryCache();
setInterval(() => cache.cleanup(), 60000);

// Cache middleware
function cacheMiddleware(ttl = 60000) {
  return (req, res, next) => {
    // Only cache GET requests
    if (req.method !== 'GET') {
      return next();
    }
    
    const cacheKey = req.originalUrl;
    const cached = cache.get(cacheKey);
    
    if (cached) {
      res.setHeader('X-Cache', 'HIT');
      return res.json(cached);
    }
    
    // Override res.json to cache response
    const originalJson = res.json.bind(res);
    res.json = function(data) {
      cache.set(cacheKey, data, ttl);
      res.setHeader('X-Cache', 'MISS');
      return originalJson(data);
    };
    
    next();
  };
}

// Apply caching
app.get('/api/products', cacheMiddleware(30000), async (req, res) => {
  // Expensive database query
  const products = await Product.find();
  res.json(products);
});
```

### Redis Cache

```javascript
class RedisCache {
  constructor(redisClient, defaultTtl = 60) {
    this.redis = redisClient;
    this.defaultTtl = defaultTtl;  // In seconds
  }
  
  async get(key) {
    const value = await this.redis.get(key);
    return value ? JSON.parse(value) : null;
  }
  
  async set(key, value, ttl = null) {
    const serialized = JSON.stringify(value);
    await this.redis.setex(key, ttl || this.defaultTtl, serialized);
  }
  
  async delete(key) {
    await this.redis.del(key);
  }
  
  /**
   * Cache-aside pattern
   */
  async getOrSet(key, fetchFn, ttl = null) {
    // Try to get from cache
    let value = await this.get(key);
    
    if (value !== null) {
      return { value, cached: true };
    }
    
    // Fetch from source
    value = await fetchFn();
    
    // Store in cache
    await this.set(key, value, ttl);
    
    return { value, cached: false };
  }
}

const redisCache = new RedisCache(redis, 300);  // 5 minutes default

app.get('/api/users/:id', async (req, res) => {
  const cacheKey = `user:${req.params.id}`;
  
  const { value: user, cached } = await redisCache.getOrSet(
    cacheKey,
    async () => await User.findById(req.params.id),
    600  // 10 minutes TTL
  );
  
  res.setHeader('X-Cache', cached ? 'HIT' : 'MISS');
  res.json(user);
});

// Cache invalidation on updates
app.put('/api/users/:id', async (req, res) => {
  const user = await User.findByIdAndUpdate(req.params.id, req.body, { new: true });
  
  // Invalidate cache
  await redisCache.delete(`user:${req.params.id}`);
  
  res.json(user);
});
```

### NGINX Caching

```nginx
http {
    # Define cache path
    proxy_cache_path /var/cache/nginx 
                     levels=1:2 
                     keys_zone=api_cache:10m 
                     max_size=1g 
                     inactive=60m 
                     use_temp_path=off;
    
    server {
        listen 80;
        
        # Cache GET requests
        location /api/products {
            proxy_cache api_cache;
            proxy_cache_methods GET;
            proxy_cache_key \"$request_uri\";
            proxy_cache_valid 200 10m;
            proxy_cache_valid 404 1m;
            proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
            proxy_cache_lock on;
            
            # Cache headers
            add_header X-Cache-Status $upstream_cache_status;
            
            proxy_pass http://backend;
        }
        
        # Cache with custom key including auth
        location /api/user-specific {
            proxy_cache api_cache;
            proxy_cache_key \"$request_uri|$http_authorization\";
            proxy_cache_valid 200 5m;
            
            add_header X-Cache-Status $upstream_cache_status;
            
            proxy_pass http://backend;
        }
        
        # Bypass cache for certain conditions
        location /api/dynamic {
            set $skip_cache 0;
            
            # Don't cache POST requests
            if ($request_method = POST) {
                set $skip_cache 1;
            }
            
            # Don't cache if nocache query param
            if ($arg_nocache) {
                set $skip_cache 1;
            }
            
            proxy_cache_bypass $skip_cache;
            proxy_no_cache $skip_cache;
            
            proxy_cache api_cache;
            proxy_pass http://backend;
        }
        
        # Cache purge endpoint (NGINX Plus)
        location ~ /purge(/.*) {
            allow 127.0.0.1;
            deny all;
            proxy_cache_purge api_cache \"$1\";
        }
    }
}
```

### Cache Invalidation Strategies

```javascript
/**
 * Tag-based cache invalidation
 */
class TaggedCache {
  constructor(redis) {
    this.redis = redis;
  }
  
  /**
   * Set cache with tags
   */
  async set(key, value, tags = [], ttl = 300) {
    await this.redis.setex(key, ttl, JSON.stringify(value));
    
    // Store key in each tag's set
    for (const tag of tags) {
      await this.redis.sadd(`tag:${tag}`, key);
      await this.redis.expire(`tag:${tag}`, ttl);
    }
  }
  
  /**
   * Get cached value
   */
  async get(key) {
    const value = await this.redis.get(key);
    return value ? JSON.parse(value) : null;
  }
  
  /**
   * Invalidate all keys with tag
   */
  async invalidateTag(tag) {
    const keys = await this.redis.smembers(`tag:${tag}`);
    
    if (keys.length > 0) {
      await this.redis.del(...keys);
      await this.redis.del(`tag:${tag}`);
    }
    
    return keys.length;
  }
}

const taggedCache = new TaggedCache(redis);

// Cache user with tags
app.get('/api/users/:id', async (req, res) => {
  const cacheKey = `user:${req.params.id}`;
  let user = await taggedCache.get(cacheKey);
  
  if (!user) {
    user = await User.findById(req.params.id);
    await taggedCache.set(
      cacheKey,
      user,
      ['users', `user:${user.id}`, `org:${user.orgId}`],
      600
    );
  }
  
  res.json(user);
});

// Invalidate all user caches when organization changes
app.put('/api/organizations/:id', async (req, res) => {
  const org = await Organization.findByIdAndUpdate(req.params.id, req.body);
  
  // Invalidate all caches tagged with this org
  const invalidated = await taggedCache.invalidateTag(`org:${org.id}`);
  console.log(`Invalidated ${invalidated} cache entries`);
  
  res.json(org);
});
```

---

## Cross-Origin Resource Sharing (CORS) Configuration

### Basic CORS Setup

```javascript
const cors = require('cors');

// Simple CORS - allow all origins (development only)
app.use(cors());

// Production CORS configuration
const corsOptions = {
  origin: function (origin, callback) {
    const allowedOrigins = [
      'https://example.com',
      'https://www.example.com',
      'https://app.example.com'
    ];
    
    // Allow requests with no origin (mobile apps, curl, etc.)
    if (!origin) return callback(null, true);
    
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,  // Allow cookies
  optionsSuccessStatus: 200,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-Requested-With',
    'X-API-Key'
  ],
  exposedHeaders: [
    'X-Total-Count',
    'X-Page-Number',
    'X-Page-Size'
  ],
  maxAge: 86400  // 24 hours
};

app.use(cors(corsOptions));
```

### Dynamic CORS

```javascript
// Dynamic CORS based on environment
app.use((req, res, next) => {
  const origin = req.headers.origin;
  
  // Development: allow localhost
  if (process.env.NODE_ENV === 'development') {
    if (origin && origin.includes('localhost')) {
      res.setHeader('Access-Control-Allow-Origin', origin);
      res.setHeader('Access-Control-Allow-Credentials', 'true');
    }
  }
  // Production: strict whitelist
  else {
    const allowedOrigins = process.env.ALLOWED_ORIGINS.split(',');
    if (origin && allowedOrigins.includes(origin)) {
      res.setHeader('Access-Control-Allow-Origin', origin);
      res.setHeader('Access-Control-Allow-Credentials', 'true');
    }
  }
  
  // Handle preflight
  if (req.method === 'OPTIONS') {
    res.setHeader('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE,PATCH');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type,Authorization');
    res.setHeader('Access-Control-Max-Age', '86400');
    return res.sendStatus(204);
  }
  
  next();
});
```

### NGINX CORS

```nginx
server {
    listen 80;
    
    # CORS configuration
    location /api/ {
        # Preflight requests
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '$http_origin';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, PATCH, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization, X-Requested-With';
            add_header 'Access-Control-Max-Age' 86400;
            add_header 'Content-Length' 0;
            add_header 'Content-Type' 'text/plain';
            return 204;
        }
        
        # Regular requests
        add_header 'Access-Control-Allow-Origin' '$http_origin' always;
        add_header 'Access-Control-Allow-Credentials' 'true' always;
        add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range' always;
        
        proxy_pass http://backend;
    }
}
```

---

## Production Deployment Strategies

### Docker Deployment

```dockerfile
# Dockerfile for API Gateway
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

# Production image
FROM node:18-alpine

RUN apk add --no-cache dumb-init

ENV NODE_ENV=production

WORKDIR /app

COPY --from=builder /app .

USER node

EXPOSE 8000

CMD [\"dumb-init\", \"node\", \"server.js\"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  gateway:
    build: .
    ports:
      - \"8000:8000\"
    environment:
      - NODE_ENV=production
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - redis
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
        max_attempts: 3
    healthcheck:
      test: [\"CMD\", \"wget\", \"--quiet\", \"--tries=1\", \"--spider\", \"http://localhost:8000/health\"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
  
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

volumes:
  redis_data:
```

### Kubernetes Deployment

```yaml
# gateway-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  labels:
    app: api-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: gateway
        image: myregistry/api-gateway:latest
        ports:
        - containerPort: 8000
        env:
        - name: NODE_ENV
          value: \"production\"
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: gateway-secrets
              key: redis-url
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: gateway-secrets
              key: jwt-secret
        resources:
          requests:
            memory: \"128Mi\"
            cpu: \"100m\"
          limits:
            memory: \"256Mi\"
            cpu: \"200m\"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
spec:
  selector:
    app: api-gateway
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-gateway
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Monitoring and Observability

```javascript
// Prometheus metrics
const prometheus = require('prom-client');

// Create metrics registry
const register = new prometheus.Registry();

// Default metrics (CPU, memory, etc.)
prometheus.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestTotal = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

const activeConnections = new prometheus.Gauge({
  name: 'http_active_connections',
  help: 'Number of active HTTP connections'
});

register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(activeConnections);

// Metrics middleware
app.use((req, res, next) => {
  const start = Date.now();
  activeConnections.inc();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    
    httpRequestDuration.observe(
      {
        method: req.method,
        route: req.route?.path || req.path,
        status_code: res.statusCode
      },
      duration
    );
    
    httpRequestTotal.inc({
      method: req.method,
      route: req.route?.path || req.path,
      status_code: res.statusCode
    });
    
    activeConnections.dec();
  });
  
  next();
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

### Distributed Tracing

```javascript
// OpenTelemetry tracing
const opentelemetry = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');

const exporter = new JaegerExporter({
  endpoint: 'http://jaeger:14268/api/traces'
});

const sdk = new opentelemetry.NodeSDK({
  traceExporter: exporter,
  instrumentations: [getNodeAutoInstrumentations()]
});

sdk.start();

// Manual span creation
const { trace } = require('@opentelemetry/api');

app.get('/api/complex-operation', async (req, res) => {
  const tracer = trace.getTracer('api-gateway');
  
  const span = tracer.startSpan('complex-operation', {
    attributes: {
      'user.id': req.user?.id,
      'http.method': req.method,
      'http.url': req.originalUrl
    }
  });
  
  try {
    // Step 1
    const step1Span = tracer.startSpan('fetch-user-data', { parent: span });
    const userData = await fetchUserData();
    step1Span.end();
    
    // Step 2
    const step2Span = tracer.startSpan('fetch-order-data', { parent: span });
    const orderData = await fetchOrderData();
    step2Span.end();
    
    res.json({ user: userData, orders: orderData });
    
  } catch (error) {
    span.recordException(error);
    span.setStatus({ code: SpanStatusCode.ERROR });
    res.status(500).json({ error: error.message });
    
  } finally {
    span.end();
  }
});
```

### Structured Logging

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'api-gateway' },
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

// Logging middleware
app.use((req, res, next) => {
  const requestId = req.headers['x-request-id'] || require('uuid').v4();
  req.requestId = requestId;
  
  logger.info('Incoming request', {
    requestId,
    method: req.method,
    path: req.path,
    ip: req.ip,
    userAgent: req.headers['user-agent']
  });
  
  const start = Date.now();
  
  res.on('finish', () => {
    logger.info('Request completed', {
      requestId,
      method: req.method,
      path: req.path,
      statusCode: res.statusCode,
      duration: Date.now() - start
    });
  });
  
  next();
});
```

---

## Conclusion

### What You've Mastered

Congratulations! You've completed a comprehensive journey through idempotent API design, API gateways, and proxy systems. Let's recap what you've learned:

#### 1. Idempotency Fundamentals
- Understanding idempotency and why it matters in distributed systems
- HTTP methods and their idempotent properties
- Implementing idempotent operations with idempotency keys
- Storage strategies for idempotency tracking
- Making POST requests idempotent
- Client-side implementation patterns

#### 2. Reverse Proxies and Load Balancing
- NGINX as a high-performance reverse proxy
- Load balancing algorithms (round-robin, least connections, IP hash, weighted)
- Health checks and automatic failover
- Connection management and optimization
- SSL/TLS termination at proxy layer

#### 3. API Gateway Patterns
- Centralized entry point for microservices
- Service discovery and routing
- Request/response transformation
- Protocol translation (REST, GraphQL, gRPC)
- API composition and aggregation
- Implementation with Kong, AWS API Gateway, and custom solutions

#### 4. Rate Limiting and Throttling
- Fixed window counter algorithm
- Sliding window log and counter algorithms
- Token bucket for burst handling
- Leaky bucket for traffic smoothing
- Distributed rate limiting with Redis
- Adaptive and quota-based rate limiting

#### 5. Request/Response Transformation
- Header manipulation and enrichment
- Body format conversion (JSON, XML, CSV)
- Content negotiation
- Dynamic routing based on request attributes
- A/B testing at gateway level

#### 6. API Versioning
- URL-based versioning strategies
- Header-based versioning
- Content-type versioning
- Gateway-level version routing
- API deprecation handling

#### 7. Authentication and Authorization
- JWT authentication at gateway level
- API key management
- OAuth 2.0 integration
- Role-based access control (RBAC)
- Permission-based access control
- Kong authentication plugins

#### 8. Resilience Patterns
- Circuit breaker implementation
- Bulkhead pattern for resource isolation
- Timeout handling
- Retry with exponential backoff
- Graceful degradation
- Fallback mechanisms

#### 9. Caching Strategies
- In-memory caching with TTL
- Redis-based distributed caching
- NGINX proxy caching
- Cache invalidation patterns
- Tag-based cache management
- Cache-aside pattern

#### 10. CORS Configuration
- Understanding CORS and preflight requests
- Dynamic origin validation
- Credential handling
- NGINX CORS configuration

#### 11. Production Deployment
- Docker containerization
- Kubernetes orchestration
- Horizontal pod autoscaling
- Health checks and readiness probes
- Monitoring with Prometheus
- Distributed tracing with OpenTelemetry
- Structured logging

### Best Practices Summary

#### API Design
1. **Always use idempotency keys for non-idempotent operations**
2. **Design APIs to be naturally idempotent where possible**
3. **Use appropriate HTTP methods (PUT for updates, not POST)**
4. **Include comprehensive error handling with proper status codes**
5. **Version your APIs from day one**

#### Gateway Configuration
1. **Implement rate limiting to protect backend services**
2. **Use circuit breakers to prevent cascade failures**
3. **Cache aggressively but invalidate precisely**
4. **Authenticate at gateway, authorize at service**
5. **Transform requests/responses at gateway to decouple clients from services**

#### Performance Optimization
1. **Use connection pooling for backend connections**
2. **Enable HTTP/2 for multiplexing**
3. **Implement caching at multiple layers**
4. **Use async/non-blocking I/O**
5. **Monitor and profile continuously**

#### Security
1. **Never trust client input - validate everything**
2. **Use HTTPS everywhere in production**
3. **Implement proper CORS policies**
4. **Rate limit aggressively on public endpoints**
5. **Log security events for audit trails**

#### Reliability
1. **Design for failure - everything will fail eventually**
2. **Implement health checks and readiness probes**
3. **Use circuit breakers and bulkheads**
4. **Have fallback strategies for critical paths**
5. **Test failure scenarios regularly**

### Real-World Use Cases

#### E-Commerce Platform
```
Client Apps (Web, Mobile, Partner APIs)
        ↓
    API Gateway
    ├── Authentication (JWT)
    ├── Rate Limiting (per user tier)
    ├── Request Routing
    └── Response Caching
        ↓
    ┌─────┼─────┼─────┼─────┐
    │     │     │     │     │
  Users Orders Products Cart Payment
```

**Key Patterns:**
- Idempotent payment processing with idempotency keys
- Circuit breaker on payment service (critical path)
- Aggressive caching on product catalog
- Rate limiting per user tier (free vs. premium)
- API composition for order details (user + items + shipping)

#### Financial Services API
```
Partner Applications
        ↓
    API Gateway
    ├── mTLS Authentication
    ├── Strict Rate Limiting
    ├── Request Logging
    ├── Idempotency Enforcement
    └── Circuit Breakers
        ↓
    Banking Services
```

**Key Patterns:**
- Mandatory idempotency keys on all transactions
- Distributed rate limiting with Redis
- Complete audit logging
- Circuit breakers on external integrations
- No caching of sensitive data

#### Microservices Platform
```
Multiple Client Types
        ↓
    API Gateway
    ├── Service Discovery
    ├── Load Balancing
    ├── Protocol Translation
    └── Observability
        ↓
    50+ Microservices
```

**Key Patterns:**
- Dynamic service discovery
- Protocol translation (REST/gRPC/GraphQL)
- Distributed tracing for debugging
- Centralized authentication
- API versioning and deprecation

### Performance Benchmarks

#### Latency Impact

| Component | Added Latency | Notes |
|-----------|---------------|-------|
| NGINX Proxy | 1-5ms | Minimal overhead |
| JWT Verification | 1-3ms | CPU-bound |
| Redis Rate Limit | 1-2ms | Network-bound |
| Circuit Breaker | <1ms | In-memory check |
| Response Cache Hit | 1-2ms | Redis lookup |
| Response Cache Miss | Backend latency | Plus cache write |

#### Throughput

| Configuration | Requests/Second | Notes |
|---------------|-----------------|-------|
| Node.js Gateway (single core) | 5,000-10,000 | CPU-bound |
| Node.js Gateway (cluster) | 50,000+ | Scales linearly |
| NGINX | 50,000-100,000+ | C-based, highly optimized |
| Kong (NGINX + plugins) | 30,000-50,000 | Plugin overhead |

### Common Pitfalls and Solutions

#### Pitfall 1: Not Using Idempotency Keys
**Problem:** Duplicate charges, duplicate records
**Solution:** Enforce idempotency keys on all non-idempotent operations

#### Pitfall 2: Infinite Circuit Breaker
**Problem:** Circuit stays open forever after backend recovers
**Solution:** Implement HALF_OPEN state with success threshold

#### Pitfall 3: Cache Stampede
**Problem:** Cache expires, 1000 requests hit backend simultaneously
**Solution:** Use cache locking or stale-while-revalidate pattern

#### Pitfall 4: Gateway as Monolith
**Problem:** Gateway becomes a bottleneck and single point of failure
**Solution:** Run multiple gateway instances with load balancing

#### Pitfall 5: Over-Caching
**Problem:** Serving stale data, cache invalidation nightmares
**Solution:** Short TTLs, event-based invalidation, cache tags

### Next Steps

#### Advanced Topics to Explore
1. **Service Mesh** (Istio, Linkerd) - Network of sidecars for inter-service communication
2. **GraphQL Federation** - Unified GraphQL API across microservices
3. **Event-Driven Architecture** - Async communication patterns
4. **API Security** - OAuth 2.0, OpenID Connect, API threat protection
5. **Multi-Region Deployment** - Global load balancing, geo-routing
6. **API Analytics** - Usage tracking, billing, quota management

#### Recommended Reading
- "Building Microservices" by Sam Newman
- "Designing Data-Intensive Applications" by Martin Kleppmann
- "Release It!" by Michael T. Nygard
- NGINX documentation and best practices
- Kong documentation and plugin development
- AWS API Gateway documentation

#### Tools to Master
- **Kong** - Open-source API Gateway
- **Traefik** - Cloud-native edge router
- **Envoy** - Service proxy by Lyft
- **Istio** - Service mesh platform
- **Prometheus + Grafana** - Monitoring stack
- **Jaeger** - Distributed tracing

### Final Thoughts

API Gateways and idempotent design are not just technical patterns - they're essential building blocks for modern, distributed systems. As you build real-world applications, remember:

1. **Start simple** - Don't over-engineer initially
2. **Measure everything** - You can't improve what you don't measure
3. **Design for failure** - Systems will fail; plan for it
4. **Document thoroughly** - Your future self will thank you
5. **Security first** - Never compromise on security

You now have the knowledge to build production-grade API infrastructure. Go forth and build amazing systems!

---

## Appendix: Quick Reference

### Idempotency Key Header
```
Idempotency-Key: <unique-identifier>
```

### Common HTTP Status Codes
- `200 OK` - Successful request
- `201 Created` - Resource created
- `204 No Content` - Success with no body
- `400 Bad Request` - Invalid request
- `401 Unauthorized` - Authentication required
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource doesn't exist
- `409 Conflict` - Conflicting request
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error
- `502 Bad Gateway` - Upstream server error
- `503 Service Unavailable` - Temporary unavailability
- `504 Gateway Timeout` - Upstream timeout

### Rate Limit Headers
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1638360000
Retry-After: 60
```

### Cache Headers
```
Cache-Control: max-age=3600, public
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Expires: Wed, 21 Oct 2025 07:28:00 GMT
X-Cache: HIT
```

### CORS Headers
```
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```

### Security Headers
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

---

**End of Tutorial**

**Author:** AI Code Assistant  
**Version:** 1.0  
**Last Updated:** December 2024  
**License:** MIT

For questions, improvements, or contributions, please visit the repository or contact the maintainers.

Happy coding! 🚀