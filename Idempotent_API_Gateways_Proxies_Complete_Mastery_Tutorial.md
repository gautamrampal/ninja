# Idempotent API, API Gateways and Proxies - Complete Mastery Tutorial

> **From Zero to Ninja: Master API Design Patterns, Gateway Architecture, and Proxy Systems**

---

## Table of Contents

1. [Introduction](#introduction)
2. [Understanding Idempotency and Its Use Cases](#understanding-idempotency-and-its-use-cases)
3. [Implementing an Idempotent API](#implementing-an-idempotent-api)
4. [NGINX as a Reverse Proxy and Load Balancer](#nginx-as-a-reverse-proxy-and-load-balancer)
5. [API Gateway Patterns and Implementation](#api-gateway-patterns-and-implementation)
6. [Rate Limiting and Throttling in API Gateways](#rate-limiting-and-throttling-in-api-gateways)
7. [Request/Response Transformation and Routing](#request-response-transformation-and-routing)
8. [API Versioning and Documentation](#api-versioning-and-documentation)
9. [Authentication and Authorization at Gateway Level](#authentication-and-authorization-at-gateway-level)
10. [Circuit Breaking and Fallback Mechanisms](#circuit-breaking-and-fallback-mechanisms)
11. [Caching Strategies at the Proxy Layer](#caching-strategies-at-the-proxy-layer)
12. [Cross-Origin Resource Sharing (CORS) Configuration](#cross-origin-resource-sharing-cors-configuration)
13. [Advanced Patterns and Best Practices](#advanced-patterns-and-best-practices)
14. [Production Deployment Strategies](#production-deployment-strategies)
15. [Conclusion](#conclusion)

---

## Introduction

### What You'll Master

This tutorial provides a complete guide to building robust, scalable API infrastructure using idempotent design patterns, API gateways, and reverse proxies. You'll learn how to design APIs that are resilient to network failures, implement sophisticated gateway patterns, and build production-grade proxy systems.

### Why This Matters

- **Reliability**: Idempotent APIs prevent duplicate operations in distributed systems
- **Scalability**: API Gateways centralize cross-cutting concerns and enable horizontal scaling
- **Security**: Proxy layers provide defense-in-depth and centralized access control
- **Performance**: Strategic caching and load balancing optimize response times
- **Maintainability**: Centralized routing and transformation simplify microservices architecture

### Prerequisites

- Understanding of HTTP protocol and RESTful APIs
- Basic knowledge of web servers and networking
- Familiarity with at least one backend programming language
- Understanding of JSON and API design principles

---

## Understanding Idempotency and Its Use Cases

### What is Idempotency?

**Idempotency** is a property of operations where performing the same operation multiple times produces the same result as performing it once. In API design, this means that making the same API call multiple times has the same effect as making it once.

### Mathematical Definition

```
f(f(x)) = f(x)
```

For all values of `x`, applying function `f` multiple times yields the same result as applying it once.

### HTTP Methods and Idempotency

| HTTP Method | Idempotent | Safe | Use Case |
|-------------|-----------|------|----------|
| GET | Yes | Yes | Retrieve resources |
| HEAD | Yes | Yes | Retrieve headers only |
| PUT | Yes | No | Replace/Update resource |
| DELETE | Yes | No | Remove resource |
| POST | No | No | Create resource |
| PATCH | No | No | Partial update |
| OPTIONS | Yes | Yes | Check capabilities |

### Why Idempotency Matters

#### 1. Network Reliability

```javascript
// Without idempotency - dangerous!
async function transferMoney(fromAccount, toAccount, amount) {
  // If network fails after debit but before credit,
  // retry causes double debit
  await debit(fromAccount, amount);
  await credit(toAccount, amount);
}

// With idempotency - safe!
async function transferMoneyIdempotent(transferId, fromAccount, toAccount, amount) {
  // Check if transfer already processed
  if (await isTransferProcessed(transferId)) {
    return await getTransferResult(transferId);
  }
  
  // Process with idempotency key
  const result = await processTransfer(transferId, fromAccount, toAccount, amount);
  await storeTransferResult(transferId, result);
  return result;
}
```

#### 2. Retry Safety

In distributed systems, automatic retries are common. Idempotent operations can be safely retried without side effects.

```javascript
// Client with automatic retry
async function makePayment(paymentData) {
  const idempotencyKey = generateUUID();
  
  let retries = 3;
  while (retries > 0) {
    try {
      return await fetch('/api/payments', {
        method: 'POST',
        headers: {
          'Idempotency-Key': idempotencyKey,  // Same key for all retries
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(paymentData)
      });
    } catch (error) {
      retries--;
      if (retries === 0) throw error;
      await sleep(1000);  // Wait before retry
    }
  }
}
```

#### 3. Distributed Systems Consistency

```javascript
// Order processing in microservices
async function createOrder(orderId, orderData) {
  // Each service uses the same orderId as idempotency key
  
  // 1. Inventory service - idempotent reserve
  await inventoryService.reserve(orderId, orderData.items);
  
  // 2. Payment service - idempotent charge
  await paymentService.charge(orderId, orderData.total);
  
  // 3. Shipping service - idempotent create shipment
  await shippingService.createShipment(orderId, orderData.address);
  
  // If any step fails and retries, idempotency prevents duplicates
}
```

### Common Use Cases

#### 1. Payment Processing

```javascript
// Stripe-style idempotent payment API
app.post('/api/payments', async (req, res) => {
  const idempotencyKey = req.headers['idempotency-key'];
  
  if (!idempotencyKey) {
    return res.status(400).json({ error: 'Idempotency-Key required' });
  }
  
  // Check if payment already processed
  const existingPayment = await db.payments.findOne({ idempotencyKey });
  if (existingPayment) {
    return res.json(existingPayment);  // Return cached result
  }
  
  // Process new payment
  const payment = await processPayment(req.body);
  await db.payments.insert({ ...payment, idempotencyKey });
  
  res.json(payment);
});
```

#### 2. Order Creation

```javascript
// E-commerce order creation with idempotency
app.post('/api/orders', async (req, res) => {
  const { cartId, userId } = req.body;
  const idempotencyKey = req.headers['idempotency-key'] || `${userId}-${cartId}`;
  
  // Check for existing order
  const existingOrder = await Order.findOne({ idempotencyKey });
  if (existingOrder) {
    return res.json({ order: existingOrder, created: false });
  }
  
  // Create new order atomically
  const order = await Order.create({
    userId,
    items: await Cart.getItems(cartId),
    idempotencyKey,
    status: 'pending'
  });
  
  // Clear cart
  await Cart.clear(cartId);
  
  res.status(201).json({ order, created: true });
});
```

#### 3. Resource Provisioning

```javascript
// Cloud resource provisioning (AWS-style)
async function provisionServer(provisioningId, config) {
  // Check if server already provisioned
  const existing = await servers.findByProvisioningId(provisioningId);
  if (existing) {
    return { server: existing, provisioned: false };
  }
  
  // Create server with idempotency protection
  const server = await createServer({
    ...config,
    provisioningId,
    tags: { idempotency_key: provisioningId }
  });
  
  return { server, provisioned: true };
}
```

### Idempotency Keys: Best Practices

#### 1. Key Generation Strategies

```javascript
// UUID v4 - Cryptographically random
const idempotencyKey = crypto.randomUUID();

// Timestamp + User ID
const idempotencyKey = `${userId}-${Date.now()}-${Math.random()}`;

// Hash of request content
const idempotencyKey = crypto
  .createHash('sha256')
  .update(JSON.stringify(requestBody))
  .digest('hex');

// Client-generated with business logic
const idempotencyKey = `order-${userId}-${cartId}-${timestamp}`;
```

#### 2. Storage and Expiration

```javascript
// Redis-based idempotency key storage
class IdempotencyManager {
  constructor(redisClient) {
    this.redis = redisClient;
    this.ttl = 24 * 60 * 60; // 24 hours
  }
  
  async check(key) {
    // Check if key exists and get stored result
    const stored = await this.redis.get(`idempotency:${key}`);
    return stored ? JSON.parse(stored) : null;
  }
  
  async store(key, result) {
    // Store result with expiration
    await this.redis.setex(
      `idempotency:${key}`,
      this.ttl,
      JSON.stringify(result)
    );
  }
  
  async lockKey(key, timeout = 60) {
    // Distributed lock to prevent concurrent processing
    const lockKey = `idempotency:lock:${key}`;
    const acquired = await this.redis.set(
      lockKey,
      '1',
      'EX', timeout,
      'NX'  // Only set if not exists
    );
    return acquired === 'OK';
  }
  
  async unlockKey(key) {
    await this.redis.del(`idempotency:lock:${key}`);
  }
}
```

#### 3. Idempotency Middleware

```javascript
// Express middleware for idempotent POST requests
function idempotencyMiddleware(idempotencyManager) {
  return async (req, res, next) => {
    // Only apply to non-idempotent methods
    if (!['POST', 'PATCH'].includes(req.method)) {
      return next();
    }
    
    const idempotencyKey = req.headers['idempotency-key'];
    if (!idempotencyKey) {
      return res.status(400).json({
        error: 'Idempotency-Key header required for POST/PATCH'
      });
    }
    
    // Check for existing result
    const existing = await idempotencyManager.check(idempotencyKey);
    if (existing) {
      return res.status(existing.status).json(existing.body);
    }
    
    // Acquire lock to prevent concurrent processing
    const locked = await idempotencyManager.lockKey(idempotencyKey);
    if (!locked) {
      return res.status(409).json({
        error: 'Request with this idempotency key is being processed'
      });
    }
    
    // Override res.json to store result
    const originalJson = res.json.bind(res);
    res.json = async function(body) {
      await idempotencyManager.store(idempotencyKey, {
        status: res.statusCode,
        body
      });
      await idempotencyManager.unlockKey(idempotencyKey);
      return originalJson(body);
    };
    
    next();
  };
}

// Usage
app.use(idempotencyMiddleware(new IdempotencyManager(redisClient)));
```

### Time Complexity Analysis

| Operation | Time Complexity | Space Complexity |
|-----------|----------------|------------------|
| Generate UUID key | O(1) | O(1) |
| Check Redis key | O(1) | O(1) |
| Store result | O(1) | O(n) where n = result size |
| Hash-based key | O(n) where n = input size | O(1) |

---

## Implementing an Idempotent API

### Complete Implementation Example: Payment API

```javascript
// models/Payment.js
const mongoose = require('mongoose');

const paymentSchema = new mongoose.Schema({
  idempotencyKey: {
    type: String,
    required: true,
    unique: true,
    index: true
  },
  userId: {
    type: String,
    required: true,
    index: true
  },
  amount: {
    type: Number,
    required: true
  },
  currency: {
    type: String,
    default: 'USD'
  },
  status: {
    type: String,
    enum: ['pending', 'processing', 'completed', 'failed'],
    default: 'pending'
  },
  paymentMethod: String,
  transactionId: String,
  metadata: Object,
  error: String,
  createdAt: {
    type: Date,
    default: Date.now,
    expires: 86400 // Auto-delete after 24 hours
  }
});

module.exports = mongoose.model('Payment', paymentSchema);
```

```javascript
// services/PaymentService.js
const Payment = require('../models/Payment');
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

class PaymentService {
  /**
   * Process payment with idempotency protection
   * @param {string} idempotencyKey - Unique key for this payment
   * @param {object} paymentData - Payment details
   * @returns {Promise<object>} Payment result
   */
  async processPayment(idempotencyKey, paymentData) {
    const { userId, amount, currency, paymentMethod, metadata } = paymentData;
    
    // Step 1: Check for existing payment
    let payment = await Payment.findOne({ idempotencyKey });
    
    if (payment) {
      // Payment already processed or in progress
      if (payment.status === 'completed') {
        return {
          payment,
          cached: true,
          message: 'Payment already processed'
        };
      }
      
      if (payment.status === 'processing') {
        // Wait for ongoing processing (implement with polling or websocket)
        return {
          payment,
          processing: true,
          message: 'Payment is being processed'
        };
      }
      
      if (payment.status === 'failed') {
        // Allow retry for failed payments
        payment.status = 'pending';
        payment.error = null;
      }
    } else {
      // Step 2: Create new payment record
      payment = new Payment({
        idempotencyKey,
        userId,
        amount,
        currency,
        paymentMethod,
        metadata,
        status: 'pending'
      });
      await payment.save();
    }
    
    // Step 3: Update status to processing (prevents concurrent processing)
    payment.status = 'processing';
    await payment.save();
    
    try {
      // Step 4: Process with payment provider
      const charge = await stripe.charges.create({
        amount: amount * 100, // Stripe uses cents
        currency,
        source: paymentMethod,
        description: `Payment for user ${userId}`,
        metadata: {
          userId,
          idempotencyKey,
          ...metadata
        }
      }, {
        idempotencyKey // Stripe also supports idempotency
      });
      
      // Step 5: Update payment record
      payment.status = 'completed';
      payment.transactionId = charge.id;
      await payment.save();
      
      return {
        payment,
        cached: false,
        message: 'Payment processed successfully'
      };
      
    } catch (error) {
      // Step 6: Handle errors
      payment.status = 'failed';
      payment.error = error.message;
      await payment.save();
      
      throw error;
    }
  }
  
  /**
   * Refund payment with idempotency
   * @param {string} paymentId - Original payment ID
   * @param {string} idempotencyKey - Unique key for this refund
   * @returns {Promise<object>} Refund result
   */
  async refundPayment(paymentId, idempotencyKey) {
    const payment = await Payment.findById(paymentId);
    
    if (!payment) {
      throw new Error('Payment not found');
    }
    
    if (payment.status !== 'completed') {
      throw new Error('Cannot refund incomplete payment');
    }
    
    // Check if already refunded (using metadata)
    if (payment.metadata?.refunded) {
      return {
        payment,
        cached: true,
        message: 'Payment already refunded'
      };
    }
    
    // Process refund with Stripe
    const refund = await stripe.refunds.create({
      charge: payment.transactionId
    }, {
      idempotencyKey
    });
    
    // Update payment record
    payment.metadata = {
      ...payment.metadata,
      refunded: true,
      refundId: refund.id,
      refundedAt: new Date()
    };
    await payment.save();
    
    return {
      payment,
      refund,
      message: 'Payment refunded successfully'
    };
  }
}

module.exports = new PaymentService();
```

```javascript
// routes/payments.js
const express = require('express');
const router = express.Router();
const PaymentService = require('../services/PaymentService');
const { validatePayment } = require('../middleware/validation');
const { authenticate } = require('../middleware/auth');

/**
 * Create payment - Idempotent endpoint
 * POST /api/payments
 * Headers: Idempotency-Key (required)
 */
router.post('/', authenticate, validatePayment, async (req, res) => {
  try {
    const idempotencyKey = req.headers['idempotency-key'];
    
    // Validate idempotency key
    if (!idempotencyKey) {
      return res.status(400).json({
        error: 'Idempotency-Key header is required'
      });
    }
    
    if (idempotencyKey.length < 16 || idempotencyKey.length > 255) {
      return res.status(400).json({
        error: 'Idempotency-Key must be between 16 and 255 characters'
      });
    }
    
    // Process payment
    const result = await PaymentService.processPayment(idempotencyKey, {
      userId: req.user.id,
      amount: req.body.amount,
      currency: req.body.currency || 'USD',
      paymentMethod: req.body.paymentMethod,
      metadata: req.body.metadata
    });
    
    // Return appropriate status code
    const statusCode = result.cached ? 200 : 201;
    
    res.status(statusCode).json({
      success: true,
      data: result.payment,
      cached: result.cached,
      message: result.message
    });
    
  } catch (error) {
    console.error('Payment processing error:', error);
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
});

/**
 * Refund payment - Idempotent endpoint
 * POST /api/payments/:paymentId/refund
 */
router.post('/:paymentId/refund', authenticate, async (req, res) => {
  try {
    const idempotencyKey = req.headers['idempotency-key'] || 
                          `refund-${req.params.paymentId}-${Date.now()}`;
    
    const result = await PaymentService.refundPayment(
      req.params.paymentId,
      idempotencyKey
    );
    
    res.json({
      success: true,
      data: result.payment,
      cached: result.cached,
      message: result.message
    });
    
  } catch (error) {
    console.error('Refund error:', error);
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
});

/**
 * Get payment by ID - Naturally idempotent (GET)
 * GET /api/payments/:paymentId
 */
router.get('/:paymentId', authenticate, async (req, res) => {
  try {
    const payment = await Payment.findById(req.params.paymentId);
    
    if (!payment) {
      return res.status(404).json({
        success: false,
        error: 'Payment not found'
      });
    }
    
    // Check authorization
    if (payment.userId !== req.user.id && !req.user.isAdmin) {
      return res.status(403).json({
        success: false,
        error: 'Unauthorized'
      });
    }
    
    res.json({
      success: true,
      data: payment
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
});

module.exports = router;
```

### Making POST Idempotent with Database Constraints

```javascript
// Using MongoDB unique index for idempotency
const orderSchema = new mongoose.Schema({
  orderId: {
    type: String,
    required: true,
    unique: true  // Ensures no duplicate orders
  },
  userId: String,
  items: Array,
  total: Number,
  status: String
});

// Atomic upsert operation
async function createOrder(orderId, orderData) {
  try {
    // findOneAndUpdate with upsert is atomic and idempotent
    const order = await Order.findOneAndUpdate(
      { orderId },  // Query
      {
        $setOnInsert: {  // Only set on insert, not update
          ...orderData,
          createdAt: new Date()
        }
      },
      {
        upsert: true,  // Create if doesn't exist
        new: true,     // Return the document
        runValidators: true
      }
    );
    
    return order;
    
  } catch (error) {
    if (error.code === 11000) {
      // Duplicate key - order already exists
      return await Order.findOne({ orderId });
    }
    throw error;
  }
}
```

### Idempotent PUT Implementation

```javascript
// PUT is naturally idempotent - replaces entire resource
app.put('/api/users/:userId', async (req, res) => {
  const { userId } = req.params;
  const userData = req.body;
  
  // Replace entire user document
  const user = await User.findOneAndUpdate(
    { userId },
    userData,  // Complete replacement
    {
      upsert: true,      // Create if doesn't exist
      new: true,         // Return updated document
      overwrite: true    // Replace entire document
    }
  );
  
  res.json(user);
});

// Calling this multiple times with same data produces same result
// PUT /api/users/123 { "name": "John", "email": "john@example.com" }
// Result is always the same regardless of how many times called
```

### Idempotent DELETE Implementation

```javascript
// DELETE is naturally idempotent
app.delete('/api/users/:userId', async (req, res) => {
  const { userId } = req.params;
  
  const result = await User.deleteOne({ userId });
  
  if (result.deletedCount === 0) {
    // Already deleted or never existed - still successful
    return res.status(204).send();  // No content
  }
  
  res.status(204).send();
});

// Calling DELETE multiple times:
// 1st call: Deletes the user, returns 204
// 2nd call: User already deleted, returns 204
// Result: Same end state (user doesn't exist)
```

### Handling Non-Idempotent PATCH

```javascript
// PATCH can be non-idempotent without proper design
// Bad: Increment operation (non-idempotent)
app.patch('/api/accounts/:id/increment', async (req, res) => {
  // Each call increases balance - NOT idempotent!
  const account = await Account.findOneAndUpdate(
    { _id: req.params.id },
    { $inc: { balance: req.body.amount } },  // Dangerous!
    { new: true }
  );
  res.json(account);
});

// Good: Set absolute value with version check (idempotent-ish)
app.patch('/api/accounts/:id', async (req, res) => {
  const { version, balance } = req.body;
  
  // Optimistic locking ensures consistency
  const account = await Account.findOneAndUpdate(
    {
      _id: req.params.id,
      version: version  // Only update if version matches
    },
    {
      $set: { balance },
      $inc: { version: 1 }
    },
    { new: true }
  );
  
  if (!account) {
    return res.status(409).json({
      error: 'Version conflict - data was modified'
    });
  }
  
  res.json(account);
});
```

### Client-Side Implementation

```javascript
// React component with idempotent API calls
import { useState } from 'react';
import { v4 as uuidv4 } from 'uuid';

function PaymentForm() {
  const [processing, setProcessing] = useState(false);
  const [result, setResult] = useState(null);
  
  const handlePayment = async (paymentData) => {
    // Generate idempotency key once
    const idempotencyKey = uuidv4();
    setProcessing(true);
    
    try {
      const response = await fetch('/api/payments', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Idempotency-Key': idempotencyKey
        },
        body: JSON.stringify(paymentData)
      });
      
      const data = await response.json();
      
      if (data.cached) {
        console.log('Payment already processed');
      }
      
      setResult(data);
      
    } catch (error) {
      // Safe to retry with same idempotency key
      console.error('Payment failed, can retry safely');
      throw error;
      
    } finally {
      setProcessing(false);
    }
  };
  
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      handlePayment({ amount: 100, currency: 'USD' });
    }}>
      <button type="submit" disabled={processing}>
        {processing ? 'Processing...' : 'Pay Now'}
      </button>
    </form>
  );
}
```

---

## NGINX as a Reverse Proxy and Load Balancer

### Introduction to NGINX

NGINX is a high-performance web server, reverse proxy, and load balancer. It's designed to handle thousands of concurrent connections with low memory footprint.

### Basic Reverse Proxy Configuration

```nginx
# /etc/nginx/nginx.conf
# Basic reverse proxy setup

worker_processes auto;  # Auto-detect CPU cores
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;  # Max connections per worker
    use epoll;                # Efficient connection processing (Linux)
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Logging format
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
    
    # Performance optimizations
    sendfile on;           # Efficient file transfers
    tcp_nopush on;         # Optimize packet sending
    tcp_nodelay on;        # Don't buffer data
    keepalive_timeout 65;  # Keep connections alive
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript 
               application/json application/javascript application/xml+rss;
    
    # Upstream backend servers
    upstream backend {
        # Define backend server pool
        server 127.0.0.1:3000;
    }
    
    # Server block
    server {
        listen 80;
        server_name example.com;
        
        location / {
            # Proxy to backend
            proxy_pass http://backend;
            
            # Proxy headers
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Timeouts
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }
    }
}
```

### Load Balancing Strategies

#### 1. Round Robin (Default)

```nginx
# Distributes requests evenly across servers
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}

# Request 1 -> backend1
# Request 2 -> backend2
# Request 3 -> backend3
# Request 4 -> backend1 (cycles back)
```

#### 2. Least Connections

```nginx
# Routes to server with fewest active connections
upstream backend {
    least_conn;  # Enable least connections algorithm
    
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}

# If backend1 has 5 connections, backend2 has 3, backend3 has 7
# Next request goes to backend2 (least connections)
```

#### 3. IP Hash (Session Persistence)

```nginx
# Same client IP always goes to same server
upstream backend {
    ip_hash;  # Enable IP-based hashing
    
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}

# Client IP 192.168.1.100 -> always backend2
# Client IP 192.168.1.101 -> always backend1
# Useful for session persistence
```

#### 4. Weighted Load Balancing

```nginx
# Distribute based on server capacity
upstream backend {
    server backend1.example.com weight=3;  # Receives 3x more requests
    server backend2.example.com weight=2;  # Receives 2x more requests
    server backend3.example.com weight=1;  # Receives 1x requests
}

# Out of 6 requests:
# backend1: 3 requests
# backend2: 2 requests
# backend3: 1 request
```

#### 5. Generic Hash

```nginx
# Hash based on custom variable
upstream backend {
    hash $request_uri consistent;  # Hash by URL path
    
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}

# Same URL always goes to same server
# /api/users -> backend1
# /api/products -> backend2
```

### Health Checks and Failover

```nginx
upstream backend {
    # Server with custom parameters
    server backend1.example.com max_fails=3 fail_timeout=30s;
    server backend2.example.com max_fails=3 fail_timeout=30s;
    server backend3.example.com backup;  # Only used if others fail
    
    # max_fails=3: Mark as down after 3 failed attempts
    # fail_timeout=30s: Try again after 30 seconds
    # backup: Backup server only used when others are unavailable
}

server {
    listen 80;
    
    location / {
        proxy_pass http://backend;
        proxy_next_upstream error timeout http_502 http_503 http_504;
        # Automatically retry on these error codes
    }
}
```

### Advanced Proxy Configuration

```nginx
# /etc/nginx/sites-available/api-gateway
server {
    listen 80;
    listen 443 ssl http2;
    server_name api.example.com;
    
    # SSL configuration
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    # Client body size limit
    client_max_body_size 10M;
    
    # Rate limiting (covered in detail later)
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
    
    # API v1 endpoints
    location /api/v1/ {
        limit_req zone=api_limit burst=20 nodelay;
        
        proxy_pass http://backend_v1/;
        proxy_http_version 1.1;
        
        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Buffering configuration
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
        proxy_busy_buffers_size 8k;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
    
    # Static files (no proxying)
    location /static/ {
        alias /var/www/static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}

# Backend upstream pools
upstream backend_v1 {
    least_conn;
    server 10.0.1.10:3000 max_fails=3 fail_timeout=30s;
    server 10.0.1.11:3000 max_fails=3 fail_timeout=30s;
    server 10.0.1.12:3000 max_fails=3 fail_timeout=30s;
    
    # Keep alive connections to backend
    keepalive 32;
}
```

### Load Balancing with Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - backend1
      - backend2
      - backend3
  
  backend1:
    build: ./backend
    environment:
      - PORT=3000
      - INSTANCE=1
  
  backend2:
    build: ./backend
    environment:
      - PORT=3000
      - INSTANCE=2
  
  backend3:
    build: ./backend
    environment:
      - PORT=3000
      - INSTANCE=3
```

```nginx
# nginx.conf for Docker
http {
    upstream backend {
        # Docker service names as hostnames
        server backend1:3000;
        server backend2:3000;
        server backend3:3000;
    }
    
    server {
        listen 80;
        
        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
        }
    }
}
```

### Monitoring and Logging

```nginx
# Extended logging format with timing information
log_format detailed '$remote_addr - $remote_user [$time_local] '
                     '"$request" $status $body_bytes_sent '
                     '"$http_referer" "$http_user_agent" '
                     'rt=$request_time '
                     'uct="$upstream_connect_time" '
                     'uht="$upstream_header_time" '
                     'urt="$upstream_response_time"';

server {
    access_log /var/log/nginx/access.log detailed;
    
    location /api/ {
        proxy_pass http://backend;
        
        # Add response time header
        add_header X-Response-Time $request_time always;
        add_header X-Upstream-Server $upstream_addr always;
    }
}
```

### NGINX Performance Tuning

```nginx
# /etc/nginx/nginx.conf - Production optimizations
user nginx;
worker_processes auto;  # One worker per CPU core
worker_rlimit_nofile 100000;  # Increase file descriptor limit

events {
    worker_connections 4096;  # Increase from default 1024
    use epoll;                # Linux-specific optimization
    multi_accept on;          # Accept multiple connections at once
}

http {
    # Connection keep-alive
    keepalive_timeout 65;
    keepalive_requests 100;
    
    # File I/O optimizations
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    
    # Reduce timeouts for slow clients
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;
    
    # Buffer sizes
    client_body_buffer_size 128k;
    client_max_body_size 10m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 8k;
    
    # Proxy optimizations
    proxy_connect_timeout 60;
    proxy_send_timeout 60;
    proxy_read_timeout 60;
    proxy_buffer_size 4k;
    proxy_buffers 8 4k;
    proxy_busy_buffers_size 8k;
    proxy_temp_file_write_size 64k;
    
    # Enable caching
    proxy_cache_path /var/cache/nginx levels=1:2 
                     keys_zone=api_cache:10m 
                     max_size=1g 
                     inactive=60m 
                     use_temp_path=off;
}
```

---

## API Gateway Patterns and Implementation

### What is an API Gateway?

An **API Gateway** is a server that acts as a single entry point for a collection of microservices. It handles request routing, composition, protocol translation, and various cross-cutting concerns.

### API Gateway Responsibilities

```
┌─────────────┐
│   Clients   │
└──────┬──────┘
       │
       │ Single Entry Point
       ▼
┌─────────────────────────────────┐
│       API Gateway               │
│                                 │
│  ┌───────────────────────────┐ │
│  │  Authentication           │ │
│  │  Authorization            │ │
│  │  Rate Limiting            │ │
│  │  Request Routing          │ │
│  │  Load Balancing           │ │
│  │  Caching                  │ │
│  │  Request/Response Transform│ │
│  │  Protocol Translation     │ │
│  │  API Versioning           │ │
│  │  Logging & Monitoring     │ │
│  └───────────────────────────┘ │
└────────┬────────┬────────┬─────┘
         │        │        │
    ┌────▼───┐ ┌─▼────┐ ┌─▼─────┐
    │ Users  │ │Orders│ │Payment│
    │Service │ │Service│ │Service│
    └────────┘ └──────┘ └───────┘
```

### Implementing API Gateway with Kong

#### Installation and Setup

```bash
# Using Docker Compose
# docker-compose.yml
version: '3.8'

services:
  kong-database:
    image: postgres:13
    environment:
      POSTGRES_USER: kong
      POSTGRES_DB: kong
      POSTGRES_PASSWORD: kong
    volumes:
      - kong_data:/var/lib/postgresql/data
  
  kong-migration:
    image: kong:3.4
    command: kong migrations bootstrap
    depends_on:
      - kong-database
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_PASSWORD: kong
  
  kong:
    image: kong:3.4
    depends_on:
      - kong-database
      - kong-migration
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_PASSWORD: kong
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    ports:
      - "8000:8000"  # Proxy port
      - "8443:8443"  # Proxy SSL port
      - "8001:8001"  # Admin API port
  
  konga:
    image: pantsel/konga
    depends_on:
      - kong
    environment:
      NODE_ENV: production
    ports:
      - "1337:1337"  # Konga UI

volumes:
  kong_data:
```

```bash
# Start Kong
docker-compose up -d

# Verify Kong is running
curl http://localhost:8001/
```

#### Configuring Services and Routes

```bash
# Add a service
curl -i -X POST http://localhost:8001/services \
  --data name=user-service \
  --data url='http://user-service:3000'

# Add a route to the service
curl -i -X POST http://localhost:8001/services/user-service/routes \
  --data 'paths[]=/api/users' \
  --data 'methods[]=GET' \
  --data 'methods[]=POST'

# Test the route
curl http://localhost:8000/api/users
```

#### Kong Configuration via Declarative Config

```yaml
# kong.yml - Declarative configuration
_format_version: "3.0"

# Define services (backend APIs)
services:
  - name: user-service
    url: http://user-service:3000
    routes:
      - name: user-routes
        paths:
          - /api/users
        methods:
          - GET
          - POST
          - PUT
          - DELETE
    plugins:
      - name: rate-limiting
        config:
          minute: 100
          policy: local
      - name: cors
        config:
          origins:
            - "*"
          methods:
            - GET
            - POST
          headers:
            - Accept
            - Authorization
          credentials: true
  
  - name: order-service
    url: http://order-service:3000
    routes:
      - name: order-routes
        paths:
          - /api/orders
    plugins:
      - name: jwt
        config:
          claims_to_verify:
            - exp
      - name: rate-limiting
        config:
          minute: 50
  
  - name: payment-service
    url: http://payment-service:3000
    routes:
      - name: payment-routes
        paths:
          - /api/payments
    plugins:
      - name: jwt
      - name: rate-limiting
        config:
          minute: 20
      - name: request-size-limiting
        config:
          allowed_payload_size: 1

# Global plugins
plugins:
  - name: correlation-id
    config:
      header_name: X-Correlation-ID
      generator: uuid
      echo_downstream: true
  
  - name: prometheus
    config:
      per_consumer: true

# Consumers (API clients)
consumers:
  - username: mobile-app
    keyauth_credentials:
      - key: mobile-app-key-12345
    plugins:
      - name: rate-limiting
        config:
          minute: 1000
  
  - username: web-app
    jwt_credentials:
      - algorithm: HS256
        key: web-app-key
        secret: super-secret-key
```

```bash
# Apply declarative configuration
docker exec -it kong kong config db_import /path/to/kong.yml
```

### Implementing API Gateway with AWS API Gateway

#### Using AWS CDK (TypeScript)

```typescript
// lib/api-gateway-stack.ts
import * as cdk from 'aws-cdk-lib';
import * as apigateway from 'aws-cdk-lib/aws-apigateway';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as iam from 'aws-cdk-lib/aws-iam';
import { Construct } from 'constructs';

export class ApiGatewayStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    
    // Lambda function for user service
    const userServiceLambda = new lambda.Function(this, 'UserServiceLambda', {
      runtime: lambda.Runtime.NODEJS_18_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda/user-service'),
      environment: {
        TABLE_NAME: 'Users'
      }
    });
    
    // Lambda function for order service
    const orderServiceLambda = new lambda.Function(this, 'OrderServiceLambda', {
      runtime: lambda.Runtime.NODEJS_18_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda/order-service')
    });
    
    // Create API Gateway
    const api = new apigateway.RestApi(this, 'ApiGateway', {
      restApiName: 'Microservices API',
      description: 'API Gateway for microservices',
      deployOptions: {
        stageName: 'prod',
        throttlingBurstLimit: 100,
        throttlingRateLimit: 50,
        loggingLevel: apigateway.MethodLoggingLevel.INFO,
        dataTraceEnabled: true,
        metricsEnabled: true
      },
      defaultCorsPreflightOptions: {
        allowOrigins: apigateway.Cors.ALL_ORIGINS,
        allowMethods: apigateway.Cors.ALL_METHODS,
        allowHeaders: [
          'Content-Type',
          'Authorization',
          'X-Api-Key'
        ]
      }
    });
    
    // API Key for authentication
    const apiKey = api.addApiKey('ApiKey', {
      apiKeyName: 'ClientApiKey'
    });
    
    // Usage plan for rate limiting
    const usagePlan = api.addUsagePlan('UsagePlan', {
      name: 'Standard',
      throttle: {
        rateLimit: 100,
        burstLimit: 200
      },
      quota: {
        limit: 10000,
        period: apigateway.Period.MONTH
      }
    });
    
    usagePlan.addApiKey(apiKey);
    usagePlan.addApiStage({
      stage: api.deploymentStage
    });
    
    // User service endpoints
    const usersResource = api.root.addResource('users');
    
    // GET /users
    usersResource.addMethod(
      'GET',
      new apigateway.LambdaIntegration(userServiceLambda),
      {
        apiKeyRequired: true,
        requestParameters: {
          'method.request.querystring.limit': false,
          'method.request.querystring.offset': false
        },
        requestValidator: new apigateway.RequestValidator(
          this,
          'GetUsersValidator',
          {
            restApi: api,
            validateRequestParameters: true
          }
        )
      }
    );
    
    // POST /users
    usersResource.addMethod(
      'POST',
      new apigateway.LambdaIntegration(userServiceLambda),
      {
        apiKeyRequired: true,
        requestValidator: new apigateway.RequestValidator(
          this,
          'CreateUserValidator',
          {
            restApi: api,
            validateRequestBody: true
          }
        ),
        requestModels: {
          'application/json': new apigateway.Model(this, 'CreateUserModel', {
            restApi: api,
            contentType: 'application/json',
            schema: {
              type: apigateway.JsonSchemaType.OBJECT,
              required: ['email', 'name'],
              properties: {
                email: {
                  type: apigateway.JsonSchemaType.STRING,
                  format: 'email'
                },
                name: {
                  type: apigateway.JsonSchemaType.STRING,
                  minLength: 1
                }
              }
            }
          })
        }
      }
    );
    
    // GET /users/{userId}
    const userResource = usersResource.addResource('{userId}');
    userResource.addMethod(
      'GET',
      new apigateway.LambdaIntegration(userServiceLambda, {
        requestTemplates: {
          'application/json': `{
            "userId": "$input.params('userId')",
            "action": "getUser"
          }`
        }
      }),
      {
        apiKeyRequired: true
      }
    );
    
    // Order service endpoints
    const ordersResource = api.root.addResource('orders');
    ordersResource.addMethod(
      'GET',
      new apigateway.LambdaIntegration(orderServiceLambda),
      {
        apiKeyRequired: true,
        authorizationType: apigateway.AuthorizationType.IAM
      }
    );
    
    ordersResource.addMethod(
      'POST',
      new apigateway.LambdaIntegration(orderServiceLambda),
      {
        apiKeyRequired: true,
        authorizationType: apigateway.AuthorizationType.IAM
      }
    );
    
    // Output API URL and key
    new cdk.CfnOutput(this, 'ApiUrl', {
      value: api.url,
      description: 'API Gateway URL'
    });
    
    new cdk.CfnOutput(this, 'ApiKeyId', {
      value: apiKey.keyId,
      description: 'API Key ID'
    });
  }
}
```

### Custom API Gateway with Express.js

```javascript
// api-gateway/index.js
const express = require('express');
const httpProxy = require('http-proxy-middleware');
const rateLimit = require('express-rate-limit');
const jwt = require('jsonwebtoken');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

const app = express();

// Middleware
app.use(helmet());  // Security headers
app.use(cors());    // CORS handling
app.use(express.json());
app.use(morgan('combined'));  // Logging

// Rate limiting
const limiter = rateLimit({
  windowMs: 1 * 60 * 1000,  // 1 minute
  max: 100,  // Limit each IP to 100 requests per minute
  message: 'Too many requests from this IP'
});

app.use('/api/', limiter);

// JWT authentication middleware
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

// Service discovery (in production, use Consul, etcd, etc.)
const services = {
  users: process.env.USER_SERVICE_URL || 'http://localhost:3001',
  orders: process.env.ORDER_SERVICE_URL || 'http://localhost:3002',
  payments: process.env.PAYMENT_SERVICE_URL || 'http://localhost:3003'
};

// Request transformation middleware
function transformRequest(serviceName) {
  return (req, res, next) => {
    // Add custom headers
    req.headers['x-gateway-time'] = Date.now();
    req.headers['x-service-name'] = serviceName;
    req.headers['x-correlation-id'] = req.headers['x-correlation-id'] || 
                                      require('uuid').v4();
    
    // Add user context
    if (req.user) {
      req.headers['x-user-id'] = req.user.id;
      req.headers['x-user-role'] = req.user.role;
    }
    
    next();
  };
}

// Circuit breaker implementation
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.failureCount = 0;
    this.threshold = threshold;
    this.timeout = timeout;
    this.state = 'CLOSED';  // CLOSED, OPEN, HALF_OPEN
    this.nextAttempt = Date.now();
  }
  
  async execute(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }
  
  onFailure() {
    this.failureCount++;
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
    }
  }
}

const circuitBreakers = {
  users: new CircuitBreaker(),
  orders: new CircuitBreaker(),
  payments: new CircuitBreaker()
};

// Proxy middleware with circuit breaker
function createProxy(serviceName, serviceUrl) {
  const circuitBreaker = circuitBreakers[serviceName];
  
  return async (req, res, next) => {
    try {
      await circuitBreaker.execute(async () => {
        const proxy = httpProxy.createProxyMiddleware({
          target: serviceUrl,
          changeOrigin: true,
          pathRewrite: {
            [`^/api/${serviceName}`]: ''
          },
          onProxyReq: (proxyReq, req) => {
            // Forward custom headers
            Object.keys(req.headers).forEach(key => {
              if (key.startsWith('x-')) {
                proxyReq.setHeader(key, req.headers[key]);
              }
            });
          },
          onProxyRes: (proxyRes, req, res) => {
            // Add gateway headers to response
            proxyRes.headers['x-gateway'] = 'custom-gateway';
            proxyRes.headers['x-service'] = serviceName;
          },
          onError: (err, req, res) => {
            console.error(`Proxy error for ${serviceName}:`, err);
            res.status(502).json({
              error: 'Bad Gateway',
              service: serviceName,
              message: err.message
            });
          }
        });
        
        return proxy(req, res, next);
      });
    } catch (error) {
      res.status(503).json({
        error: 'Service Unavailable',
        service: serviceName,
        message: 'Circuit breaker is OPEN'
      });
    }
  };
}

// Routes
// User service - public
app.use(
  '/api/users',
  transformRequest('users'),
  createProxy('users', services.users)
);

// Order service - authenticated
app.use(
  '/api/orders',
  authenticate,
  transformRequest('orders'),
  createProxy('orders', services.orders)
);

// Payment service - authenticated
app.use(
  '/api/payments',
  authenticate,
  transformRequest('payments'),
  createProxy('payments', services.payments)
);

// Health check
app.get('/health', (req, res) => {
  const health = {
    gateway: 'healthy',
    services: {}
  };
  
  Object.keys(circuitBreakers).forEach(service => {
    health.services[service] = {
      state: circuitBreakers[service].state,
      failures: circuitBreakers[service].failureCount
    };
  });
  
  res.json(health);
});

// Start server
const PORT = process.env.PORT || 8000;
app.listen(PORT, () => {
  console.log(`API Gateway running on port ${PORT}`);
  console.log('Services:');
  Object.keys(services).forEach(service => {
    console.log(`  ${service}: ${services[service]}`);
  });
});
```

### API Composition Pattern

```javascript
// Aggregation endpoint - combines data from multiple services
app.get('/api/user-profile/:userId', authenticate, async (req, res) => {
  try {
    const { userId } = req.params;
    
    // Parallel requests to multiple services
    const [userResponse, ordersResponse, preferencesResponse] = await Promise.all([
      fetch(`${services.users}/users/${userId}`),
      fetch(`${services.orders}/users/${userId}/orders?limit=5`),
      fetch(`${services.preferences}/users/${userId}/preferences`)
    ]);
    
    // Parse responses
    const [user, orders, preferences] = await Promise.all([
      userResponse.json(),
      ordersResponse.json(),
      preferencesResponse.json()
    ]);
    
    // Compose aggregated response
    res.json({
      user,
      recentOrders: orders,
      preferences
    });
    
  } catch (error) {
    console.error('Profile aggregation error:', error);
    res.status(500).json({ error: 'Failed to load profile' });
  }
});
```

### Time Complexity Analysis

| Gateway Operation | Time Complexity | Notes |
|------------------|----------------|-------|
| Route matching | O(1) - O(log n) | Hash map or trie lookup |
| Authentication | O(1) | JWT verification (constant time) |
| Rate limiting | O(1) | Redis/memory counter |
| Request proxying | O(1) | Network I/O (not CPU-bound) |
| Circuit breaker check | O(1) | State machine lookup |
| Service composition | O(k) | k = number of services |

---

## Rate Limiting and Throttling in API Gateways

### Understanding Rate Limiting vs Throttling

**Rate Limiting**: Restricts the number of requests a client can make in a given time window.

**Throttling**: Slows down request processing when limits are approached or exceeded.

### Rate Limiting Algorithms

#### 1. Fixed Window Counter

```javascript
// Simple fixed window implementation
class FixedWindowRateLimiter {
  constructor(limit, windowMs) {
    this.limit = limit;        // Max requests per window
    this.windowMs = windowMs;  // Time window in milliseconds
    this.requests = new Map(); // clientId -> { count, resetTime }
  }
  
  /**
   * Check if request is allowed
   * @param {string} clientId - Client identifier
   * @returns {object} { allowed: boolean, remaining: number, resetTime: number }
   */
  isAllowed(clientId) {
    const now = Date.now();
    const clientData = this.requests.get(clientId);
    
    // No previous requests or window expired
    if (!clientData || now >= clientData.resetTime) {
      this.requests.set(clientId, {
        count: 1,
        resetTime: now + this.windowMs
      });
      
      return {
        allowed: true,
        remaining: this.limit - 1,
        resetTime: now + this.windowMs
      };
    }
    
    // Within current window
    if (clientData.count < this.limit) {
      clientData.count++;
      return {
        allowed: true,
        remaining: this.limit - clientData.count,
        resetTime: clientData.resetTime
      };
    }
    
    // Limit exceeded
    return {
      allowed: false,
      remaining: 0,
      resetTime: clientData.resetTime
    };
  }
}

// Usage
const limiter = new FixedWindowRateLimiter(100, 60000); // 100 req/min

app.use((req, res, next) => {
  const clientId = req.ip;
  const result = limiter.isAllowed(clientId);
  
  // Add rate limit headers
  res.setHeader('X-RateLimit-Limit', 100);
  res.setHeader('X-RateLimit-Remaining', result.remaining);
  res.setHeader('X-RateLimit-Reset', result.resetTime);
  
  if (!result.allowed) {
    return res.status(429).json({
      error: 'Too Many Requests',
      retryAfter: Math.ceil((result.resetTime - Date.now()) / 1000)
    });
  }
  
  next();
});
```

**Time Complexity**: O(1)  
**Space Complexity**: O(n) where n = number of unique clients

**Limitation**: Burst at window boundaries  
Example: Client makes 100 requests at 11:59:59 and 100 more at 12:00:01 = 200 requests in 2 seconds

#### 2. Sliding Window Log

```javascript
// More accurate but memory-intensive
class SlidingWindowLogLimiter {
  constructor(limit, windowMs) {
    this.limit = limit;
    this.windowMs = windowMs;
    this.requests = new Map(); // clientId -> [timestamps]
  }
  
  isAllowed(clientId) {
    const now = Date.now();
    const windowStart = now - this.windowMs;
    
    // Get existing timestamps
    let timestamps = this.requests.get(clientId) || [];
    
    // Remove timestamps outside current window
    timestamps = timestamps.filter(ts => ts > windowStart);
    
    // Check limit
    if (timestamps.length < this.limit) {
      timestamps.push(now);
      this.requests.set(clientId, timestamps);
      
      return {
        allowed: true,
        remaining: this.limit - timestamps.length
      };
    }
    
    return {
      allowed: false,
      remaining: 0,
      oldestTimestamp: timestamps[0]
    };
  }
  
  // Cleanup old entries periodically
  cleanup() {
    const now = Date.now();
    const windowStart = now - this.windowMs;
    
    for (const [clientId, timestamps] of this.requests) {
      const filtered = timestamps.filter(ts => ts > windowStart);
      if (filtered.length === 0) {
        this.requests.delete(clientId);
      } else {
        this.requests.set(clientId, filtered);
      }
    }
  }
}

// Cleanup every minute
setInterval(() => limiter.cleanup(), 60000);
```

**Time Complexity**: O(n) where n = number of requests in window  
**Space Complexity**: O(n × m) where n = clients, m = requests per client

**Advantage**: No burst issues at boundaries  
**Disadvantage**: Higher memory usage

#### 3. Sliding Window Counter (Hybrid)

```javascript
// Best of both worlds - memory efficient and accurate
class SlidingWindowCounterLimiter {
  constructor(limit, windowMs) {
    this.limit = limit;
    this.windowMs = windowMs;
    this.windows = new Map(); // clientId -> { current, previous, currentStart }
  }
  
  isAllowed(clientId) {
    const now = Date.now();
    const currentWindowStart = Math.floor(now / this.windowMs) * this.windowMs;
    const previousWindowStart = currentWindowStart - this.windowMs;
    
    let windowData = this.windows.get(clientId);
    
    // Initialize or reset windows
    if (!windowData || windowData.currentStart !== currentWindowStart) {
      windowData = {
        current: 0,
        previous: windowData?.currentStart === previousWindowStart ? 
                  windowData.current : 0,
        currentStart: currentWindowStart
      };
      this.windows.set(clientId, windowData);
    }
    
    // Calculate weighted count
    const timeInCurrentWindow = now - currentWindowStart;
    const percentageInCurrent = timeInCurrentWindow / this.windowMs;
    const percentageInPrevious = 1 - percentageInCurrent;
    
    const weightedCount = 
      windowData.current + 
      (windowData.previous * percentageInPrevious);
    
    // Check limit
    if (weightedCount < this.limit) {
      windowData.current++;
      return {
        allowed: true,
        remaining: Math.floor(this.limit - weightedCount - 1)
      };
    }
    
    return {
      allowed: false,
      remaining: 0
    };
  }
}
```

**Time Complexity**: O(1)  
**Space Complexity**: O(n) where n = number of unique clients

#### 4. Token Bucket Algorithm

```javascript
// Allows bursts but maintains average rate
class TokenBucketLimiter {
  constructor(capacity, refillRate) {
    this.capacity = capacity;      // Max tokens (burst size)
    this.refillRate = refillRate;  // Tokens per second
    this.buckets = new Map();      // clientId -> { tokens, lastRefill }
  }
  
  isAllowed(clientId, tokensNeeded = 1) {
    const now = Date.now();
    let bucket = this.buckets.get(clientId);
    
    // Initialize bucket
    if (!bucket) {
      bucket = {
        tokens: this.capacity,
        lastRefill: now
      };
      this.buckets.set(clientId, bucket);
    }
    
    // Refill tokens based on elapsed time
    const elapsedSeconds = (now - bucket.lastRefill) / 1000;
    const tokensToAdd = elapsedSeconds * this.refillRate;
    bucket.tokens = Math.min(
      this.capacity,
      bucket.tokens + tokensToAdd
    );
    bucket.lastRefill = now;
    
    // Check if enough tokens available
    if (bucket.tokens >= tokensNeeded) {
      bucket.tokens -= tokensNeeded;
      return {
        allowed: true,
        tokens: bucket.tokens
      };
    }
    
    return {
      allowed: false,
      tokens: bucket.tokens,
      waitTime: (tokensNeeded - bucket.tokens) / this.refillRate
    };
  }
}

// Usage: 100 token capacity, refill at 10/second
const limiter = new TokenBucketLimiter(100, 10);

// Allows burst of 100 requests, then 10/second sustained
```

**Time Complexity**: O(1)  
**Space Complexity**: O(n)

**Use Case**: APIs that need to allow occasional bursts but control average rate

#### 5. Leaky Bucket Algorithm

```javascript
// Smooths out bursty traffic
class LeakyBucketLimiter {
  constructor(capacity, leakRate) {
    this.capacity = capacity;    // Queue size
    this.leakRate = leakRate;    // Requests processed per second
    this.queues = new Map();     // clientId -> { queue, lastLeak }
  }
  
  isAllowed(clientId) {
    const now = Date.now();
    let queueData = this.queues.get(clientId);
    
    // Initialize queue
    if (!queueData) {
      queueData = {
        size: 0,
        lastLeak: now
      };
      this.queues.set(clientId, queueData);
    }
    
    // Leak (process) requests
    const elapsedSeconds = (now - queueData.lastLeak) / 1000;
    const leaked = elapsedSeconds * this.leakRate;
    queueData.size = Math.max(0, queueData.size - leaked);
    queueData.lastLeak = now;
    
    // Check if queue has space
    if (queueData.size < this.capacity) {
      queueData.size++;
      return {
        allowed: true,
        queueSize: queueData.size
      };
    }
    
    return {
      allowed: false,
      queueSize: queueData.size
    };
  }
}
```

### Redis-Based Distributed Rate Limiting

```javascript
// Production-ready distributed rate limiter using Redis
const Redis = require('ioredis');

class RedisRateLimiter {
  constructor(redisClient, limit, windowSeconds) {
    this.redis = redisClient;
    this.limit = limit;
    this.windowSeconds = windowSeconds;
  }
  
  /**
   * Check rate limit using sliding window
   * @param {string} key - Rate limit key (e.g., "user:123" or "ip:192.168.1.1")
   * @returns {Promise<object>}
   */
  async checkLimit(key) {
    const now = Date.now();
    const windowStart = now - (this.windowSeconds * 1000);
    const rateLimitKey = `rate_limit:${key}`;
    
    // Use Redis pipeline for atomic operations
    const pipeline = this.redis.pipeline();
    
    // Remove old timestamps outside window
    pipeline.zremrangebyscore(rateLimitKey, 0, windowStart);
    
    // Count requests in current window
    pipeline.zcard(rateLimitKey);
    
    // Add current request timestamp
    pipeline.zadd(rateLimitKey, now, `${now}-${Math.random()}`);
    
    // Set expiration
    pipeline.expire(rateLimitKey, this.windowSeconds);
    
    // Execute pipeline
    const results = await pipeline.exec();
    
    // Parse results
    const count = results[1][1]; // zcard result
    
    const allowed = count < this.limit;
    const remaining = Math.max(0, this.limit - count - 1);
    
    return {
      allowed,
      remaining,
      limit: this.limit,
      resetTime: now + (this.windowSeconds * 1000)
    };
  }
  
  /**
   * Fixed window counter (more efficient)
   */
  async checkLimitFixed(key) {
    const now = Date.now();
    const windowStart = Math.floor(now / (this.windowSeconds * 1000));
    const rateLimitKey = `rate_limit:${key}:${windowStart}`;
    
    // Increment counter
    const count = await this.redis.incr(rateLimitKey);
    
    // Set expiration on first request in window
    if (count === 1) {
      await this.redis.expire(rateLimitKey, this.windowSeconds);
    }
    
    const allowed = count <= this.limit;
    const remaining = Math.max(0, this.limit - count);
    
    return {
      allowed,
      remaining,
      limit: this.limit,
      resetTime: (windowStart + 1) * this.windowSeconds * 1000
    };
  }
  
  /**
   * Token bucket using Redis
   */
  async checkLimitTokenBucket(key, capacity, refillRate) {
    const now = Date.now();
    const bucketKey = `token_bucket:${key}`;
    
    // Lua script for atomic token bucket operation
    const script = `
      local capacity = tonumber(ARGV[1])
      local refillRate = tonumber(ARGV[2])
      local now = tonumber(ARGV[3])
      
      local bucket = redis.call('HMGET', KEYS[1], 'tokens', 'lastRefill')
      local tokens = tonumber(bucket[1]) or capacity
      local lastRefill = tonumber(bucket[2]) or now
      
      local elapsedSeconds = (now - lastRefill) / 1000
      local tokensToAdd = elapsedSeconds * refillRate
      tokens = math.min(capacity, tokens + tokensToAdd)
      
      local allowed = 0
      if tokens >= 1 then
        tokens = tokens - 1
        allowed = 1
      end
      
      redis.call('HMSET', KEYS[1], 'tokens', tokens, 'lastRefill', now)
      redis.call('EXPIRE', KEYS[1], 3600)
      
      return {allowed, tokens}
    `;
    
    const result = await this.redis.eval(
      script,
      1,
      bucketKey,
      capacity,
      refillRate,
      now
    );
    
    return {
      allowed: result[0] === 1,
      tokens: result[1]
    };
  }
}

// Express middleware
function createRateLimitMiddleware(redisClient, options = {}) {
  const {
    limit = 100,
    windowSeconds = 60,
    keyGenerator = (req) => req.ip,
    skip = (req) => false,
    handler = (req, res) => {
      res.status(429).json({
        error: 'Too Many Requests',
        message: 'Rate limit exceeded'
      });
    }
  } = options;
  
  const limiter = new RedisRateLimiter(redisClient, limit, windowSeconds);
  
  return async (req, res, next) => {
    if (skip(req)) {
      return next();
    }
    
    try {
      const key = keyGenerator(req);
      const result = await limiter.checkLimit(key);
      
      // Add rate limit headers
      res.setHeader('X-RateLimit-Limit', result.limit);
      res.setHeader('X-RateLimit-Remaining', result.remaining);
      res.setHeader('X-RateLimit-Reset', result.resetTime);
      
      if (!result.allowed) {
        res.setHeader('Retry-After', Math.ceil(
          (result.resetTime - Date.now()) / 1000
        ));
        return handler(req, res);
      }
      
      next();
      
    } catch (error) {
      console.error('Rate limit error:', error);
      // Fail open - allow request if rate limiting fails
      next();
    }
  };
}

// Usage
const redis = new Redis();

app.use('/api/', createRateLimitMiddleware(redis, {
  limit: 100,
  windowSeconds: 60,
  keyGenerator: (req) => {
    // Rate limit by user ID if authenticated, otherwise by IP
    return req.user?.id || req.ip;
  },
  skip: (req) => {
    // Skip rate limiting for admin users
    return req.user?.role === 'admin';
  }
}));
```

### NGINX Rate Limiting

```nginx
# /etc/nginx/nginx.conf
http {
    # Define rate limit zones
    
    # Zone 1: Limit by IP address - 10 requests per second
    limit_req_zone $binary_remote_addr zone=by_ip:10m rate=10r/s;
    
    # Zone 2: Limit by API key - 100 requests per second
    limit_req_zone $http_x_api_key zone=by_api_key:10m rate=100r/s;
    
    # Zone 3: Limit by user ID - 50 requests per second
    limit_req_zone $http_x_user_id zone=by_user:10m rate=50r/s;
    
    # Connection limit zone
    limit_conn_zone $binary_remote_addr zone=conn_limit:10m;
    
    server {
        listen 80;
        
        # Limit concurrent connections
        limit_conn conn_limit 10;  # Max 10 concurrent connections per IP
        
        # Public API - strict rate limit
        location /api/public/ {
            limit_req zone=by_ip burst=20 nodelay;
            # burst=20: Allow burst of 20 requests above rate
            # nodelay: Don't delay requests within burst
            
            proxy_pass http://backend;
        }
        
        # Authenticated API - higher limit
        location /api/ {
            limit_req zone=by_api_key burst=50 nodelay;
            limit_req zone=by_user burst=30 nodelay;
            
            # Return 429 with custom message
            limit_req_status 429;
            
            proxy_pass http://backend;
        }
        
        # Download endpoint - connection limit
        location /downloads/ {
            limit_conn conn_limit 5;  # Max 5 concurrent downloads per IP
            limit_rate 1m;            # Limit download speed to 1MB/s
            
            proxy_pass http://backend;
        }
    }
    
    # Custom error page for rate limiting
    error_page 429 /rate_limit_error.html;
    location = /rate_limit_error.html {
        internal;
        return 429 '{"error": "Too Many Requests", "retry_after": 60}';
        add_header Content-Type application/json;
    }
}
```

### Kong Rate Limiting Plugin

```bash
# Add rate limiting plugin to a service
curl -i -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=rate-limiting" \
  --data "config.second=10" \
  --data "config.minute=100" \
  --data "config.hour=1000" \
  --data "config.policy=redis" \
  --data "config.redis_host=redis" \
  --data "config.redis_port=6379" \
  --data "config.fault_tolerant=true" \
  --data "config.hide_client_headers=false"

# Different limits for different consumers
curl -i -X POST http://localhost:8001/consumers/premium-user/plugins \
  --data "name=rate-limiting" \
  --data "config.minute=1000" \
  --data "config.hour=10000"
```

### Advanced Rate Limiting Patterns

#### 1. Adaptive Rate Limiting

```javascript
// Adjust rate limits based on system load
class AdaptiveRateLimiter {
  constructor(baseLimit, windowSeconds) {
    this.baseLimit = baseLimit;
    this.windowSeconds = windowSeconds;
    this.systemLoad = 0;  // 0-1 (0 = no load, 1 = max load)
  }
  
  updateSystemLoad(cpuUsage, memoryUsage, activeConnections, maxConnections) {
    // Calculate weighted system load
    this.systemLoad = (
      cpuUsage * 0.4 +
      memoryUsage * 0.3 +
      (activeConnections / maxConnections) * 0.3
    );
  }
  
  getCurrentLimit() {
    // Reduce limit as system load increases
    if (this.systemLoad > 0.9) {
      return Math.floor(this.baseLimit * 0.5);  // 50% of base
    } else if (this.systemLoad > 0.7) {
      return Math.floor(this.baseLimit * 0.7);  // 70% of base
    } else if (this.systemLoad > 0.5) {
      return Math.floor(this.baseLimit * 0.85); // 85% of base
    }
    return this.baseLimit;
  }
  
  async checkLimit(clientId) {
    const currentLimit = this.getCurrentLimit();
    // Use any rate limiting algorithm with adjusted limit
    // ...
  }
}
```

#### 2. Quota-Based Rate Limiting

```javascript
// Monthly quota with daily limits
class QuotaRateLimiter {
  constructor(redis) {
    this.redis = redis;
  }
  
  async checkQuota(userId, tier) {
    const limits = {
      free: { monthly: 1000, daily: 50 },
      pro: { monthly: 100000, daily: 5000 },
      enterprise: { monthly: Infinity, daily: 50000 }
    };
    
    const userLimits = limits[tier];
    const now = new Date();
    const monthKey = `quota:${userId}:${now.getFullYear()}-${now.getMonth()}`;
    const dayKey = `quota:${userId}:${now.toISOString().split('T')[0]}`;
    
    // Check both monthly and daily limits
    const [monthlyCount, dailyCount] = await Promise.all([
      this.redis.incr(monthKey),
      this.redis.incr(dayKey)
    ]);
    
    // Set expiration
    if (monthlyCount === 1) {
      await this.redis.expire(monthKey, 32 * 24 * 60 * 60); // 32 days
    }
    if (dailyCount === 1) {
      await this.redis.expire(dayKey, 26 * 60 * 60); // 26 hours
    }
    
    const monthlyAllowed = monthlyCount <= userLimits.monthly;
    const dailyAllowed = dailyCount <= userLimits.daily;
    
    return {
      allowed: monthlyAllowed && dailyAllowed,
      monthly: {
        used: monthlyCount,
        limit: userLimits.monthly,
        remaining: Math.max(0, userLimits.monthly - monthlyCount)
      },
      daily: {
        used: dailyCount,
        limit: userLimits.daily,
        remaining: Math.max(0, userLimits.daily - dailyCount)
      }
    };
  }
}
```

---

## Request/Response Transformation and Routing

### Request Transformation

#### 1. Header Manipulation

```javascript
// Add, modify, or remove headers
app.use((req, res, next) => {
  // Add custom headers
  req.headers['x-request-id'] = require('uuid').v4();
  req.headers['x-request-time'] = Date.now();
  
  // Remove sensitive headers before proxying
  delete req.headers['cookie'];
  delete req.headers['authorization'];
  
  // Modify headers
  if (req.headers['content-type'] === 'text/plain') {
    req.headers['content-type'] = 'application/json';
  }
  
  next();
});
```

#### 2. Body Transformation

```javascript
// Transform request body format
app.post('/api/legacy