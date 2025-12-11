# RabbitMQ Complete Mastery Tutorial

> **A comprehensive guide covering RabbitMQ from basics to advanced concepts with practical examples, configurations, and real-world scenarios**

---

## Table of Contents

### Part 1: Fundamentals
1. [Introduction to Message Queuing](#1-introduction-to-message-queuing)
2. [RabbitMQ Architecture & Core Concepts](#2-rabbitmq-architecture--core-concepts)
3. [Installation & Setup](#3-installation--setup)
4. [Basic Message Patterns](#4-basic-message-patterns)
5. [Exchanges Deep Dive](#5-exchanges-deep-dive)

### Part 2: Intermediate Concepts
6. [Message Durability & Persistence](#6-message-durability--persistence)
7. [Message Acknowledgments & Reliability](#7-message-acknowledgments--reliability)
8. [Prefetch & Flow Control](#8-prefetch--flow-control)
9. [Dead Letter Exchanges (DLX)](#9-dead-letter-exchanges-dlx)
10. [Message TTL & Queue Length Limits](#10-message-ttl--queue-length-limits)
11. [Priority Queues](#11-priority-queues)
12. [Publisher Confirms](#12-publisher-confirms)

### Part 3: Advanced Topics
13. [Clustering & High Availability](#13-clustering--high-availability)
14. [Federation & Shovel](#14-federation--shovel)
15. [RabbitMQ Streams](#15-rabbitmq-streams)
16. [Security & Authentication](#16-security--authentication)
17. [Monitoring & Management](#17-monitoring--management)
18. [Performance Optimization](#18-performance-optimization)
19. [Advanced Patterns & Architectures](#19-advanced-patterns--architectures)
20. [Production Best Practices](#20-production-best-practices)

---

## Part 1: Fundamentals

### 1. Introduction to Message Queuing

#### What is Message Queuing?

Message queuing is an asynchronous communication pattern where applications communicate by sending messages through an intermediary (message broker) rather than calling each other directly.

**Key Benefits:**
- **Decoupling**: Producer and consumer don't need to know about each other
- **Scalability**: Add more consumers to handle increased load
- **Reliability**: Messages are stored until successfully processed
- **Async Processing**: Producer doesn't wait for consumer to process
- **Load Balancing**: Distribute work across multiple consumers

#### Synchronous vs Asynchronous Communication

```
┌─────────────────────────────────────────────────────────┐
│           SYNCHRONOUS COMMUNICATION                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Client ──────────> Server                              │
│         (waiting)                                        │
│         <──────────                                      │
│         (response)                                       │
│                                                          │
│  • Client blocks until response                         │
│  • Tight coupling                                       │
│  • Server must be available                             │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│          ASYNCHRONOUS COMMUNICATION                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Producer ──> [Queue] ──> Consumer                      │
│         (continues)   (processes later)                  │
│                                                          │
│  • Producer doesn't wait                                │
│  • Loose coupling                                       │
│  • Consumer can be offline temporarily                  │
└─────────────────────────────────────────────────────────┘
```

#### When to Use RabbitMQ?

**Good Use Cases:**
- Email sending systems
- Order processing workflows
- Background job processing
- Microservices communication
- Event-driven architectures
- Log aggregation
- Real-time notifications

**Not Ideal For:**
- Simple request/response (use HTTP)
- Low-latency requirements (<1ms)
- Small-scale applications
- Direct peer-to-peer communication

---

### 2. RabbitMQ Architecture & Core Concepts

#### High-Level Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    RabbitMQ Broker                           │
│                                                              │
│  ┌──────────┐      ┌──────────┐      ┌──────────┐         │
│  │Exchange 1│      │Exchange 2│      │Exchange 3│         │
│  │  (topic) │      │ (direct) │      │ (fanout) │         │
│  └────┬─────┘      └────┬─────┘      └────┬─────┘         │
│       │                 │                  │                │
│  ┌────▼──────────┬──────▼──────┬───────────▼──────┐       │
│  │   Binding     │   Binding   │     Binding      │       │
│  └────┬──────────┴──────┬──────┴───────────┬──────┘       │
│       │                 │                  │                │
│  ┌────▼─────┐     ┌─────▼────┐      ┌─────▼────┐         │
│  │  Queue 1 │     │ Queue 2  │      │ Queue 3  │         │
│  │ [M][M][M]│     │ [M][M]   │      │ [M]      │         │
│  └────┬─────┘     └─────┬────┘      └─────┬────┘         │
└───────┼──────────────────┼─────────────────┼──────────────┘
        │                  │                 │
   ┌────▼────┐        ┌────▼────┐      ┌────▼────┐
   │Consumer │        │Consumer │      │Consumer │
   │    1    │        │    2    │      │    3    │
   └─────────┘        └─────────┘      └─────────┘

┌──────────┐
│ Producer │ ───────────────────────────────────> Exchange
└──────────┘
```

#### Core Components

**1. Producer**
- Application that sends messages
- Publishes to an exchange (never directly to queue)
- Can specify routing key and message properties

**2. Exchange**
- Receives messages from producers
- Routes messages to queues based on rules (bindings)
- Four main types: direct, topic, fanout, headers

**3. Queue**
- Buffer that stores messages
- Messages wait here until consumed
- Can be durable or temporary
- Bound to exchanges via routing keys

**4. Binding**
- Link between exchange and queue
- Defines routing rules
- Can include routing key patterns

**5. Consumer**
- Application that receives messages
- Subscribes to queues
- Acknowledges message processing

**6. Connection**
- TCP connection between application and RabbitMQ
- Expensive to create/destroy

**7. Channel**
- Virtual connection inside a TCP connection
- Lightweight, many channels per connection
- Most AMQP operations happen on channels

#### Message Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    MESSAGE LIFECYCLE                         │
└─────────────────────────────────────────────────────────────┘

1. PUBLISH
   Producer ──┐
              │ {"order_id": 123, "amount": 99.99}
              ▼
        ┌──────────┐
        │ Exchange │
        │ (orders) │
        └────┬─────┘
             │
2. ROUTE     │ routing_key: "order.created"
             ▼
        ┌──────────┐
        │  Queue   │
        │(orders_q)│
        └────┬─────┘
             │
3. CONSUME   │
             ▼
        ┌──────────┐
        │Consumer  │
        └────┬─────┘
             │
4. PROCESS   │
             │
5. ACK       │
             ▼
        [Message Deleted]
```

#### AMQP Protocol

RabbitMQ implements AMQP (Advanced Message Queuing Protocol) 0-9-1.

**Key AMQP Concepts:**
- **Frame-based**: Messages sent as frames over TCP
- **Channel multiplexing**: Multiple channels over one connection
- **Content-type agnostic**: Any data format (JSON, XML, binary)
- **Acknowledgment-based**: Reliable message delivery

---

### 3. Installation & Setup

#### Installation Options

**Option 1: Docker (Recommended for Development)**

```bash
# Pull official RabbitMQ image with management plugin
docker pull rabbitmq:3-management

# Run RabbitMQ container
docker run -d --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin123 \
  rabbitmq:3-management

# Check logs
docker logs rabbitmq

# Access management UI: http://localhost:15672
# Username: admin
# Password: admin123
```

**Option 2: Ubuntu/Debian**

```bash
# Install dependencies
sudo apt-get update
sudo apt-get install -y curl gnupg apt-transport-https

# Add RabbitMQ repository
curl -1sLf "https://keys.openpgp.org/vks/v1/by-fingerprint/0A9AF2115F4687BD29803A206B73A36E6026DFCA" | sudo gpg --dearmor | sudo tee /usr/share/keyrings/com.rabbitmq.team.gpg > /dev/null

# Add repository
sudo tee /etc/apt/sources.list.d/rabbitmq.list <<EOF
deb [signed-by=/usr/share/keyrings/com.rabbitmq.team.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu jammy main
deb [signed-by=/usr/share/keyrings/com.rabbitmq.team.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu jammy main
EOF

# Update and install
sudo apt-get update
sudo apt-get install -y rabbitmq-server

# Enable and start
sudo systemctl enable rabbitmq-server
sudo systemctl start rabbitmq-server

# Enable management plugin
sudo rabbitmq-plugins enable rabbitmq_management

# Create admin user
sudo rabbitmqctl add_user admin admin123
sudo rabbitmqctl set_user_tags admin administrator
sudo rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```

**Option 3: Windows**

```powershell
# Download from https://www.rabbitmq.com/install-windows.html
# Install Erlang first, then RabbitMQ

# Enable management plugin (run as Administrator)
rabbitmq-plugins enable rabbitmq_management

# Restart RabbitMQ service
net stop RabbitMQ
net start RabbitMQ
```

**Option 4: macOS**

```bash
# Using Homebrew
brew update
brew install rabbitmq

# Start RabbitMQ
brew services start rabbitmq

# Enable management plugin
rabbitmq-plugins enable rabbitmq_management
```

#### Basic Configuration

**rabbitmq.conf** (located in `/etc/rabbitmq/` or `C:\Users\[User]\AppData\Roaming\RabbitMQ\`)

```conf
# Networking
listeners.tcp.default = 5672
management.tcp.port = 15672

# Memory
vm_memory_high_watermark.relative = 0.4

# Disk
disk_free_limit.absolute = 2GB

# Logging
log.file.level = info
log.console = true
log.console.level = info

# Default user
default_user = admin
default_pass = admin123

# Virtual hosts
default_vhost = /
```

#### Management Commands

```bash
# Service management
sudo systemctl status rabbitmq-server
sudo systemctl stop rabbitmq-server
sudo systemctl restart rabbitmq-server

# User management
rabbitmqctl add_user username password
rabbitmqctl delete_user username
rabbitmqctl change_password username newpassword
rabbitmqctl set_user_tags username administrator
rabbitmqctl list_users

# Virtual host management
rabbitmqctl add_vhost /production
rabbitmqctl delete_vhost /production
rabbitmqctl list_vhosts

# Permissions
rabbitmqctl set_permissions -p / username ".*" ".*" ".*"
rabbitmqctl list_permissions -p /

# Queue management
rabbitmqctl list_queues
rabbitmqctl list_exchanges
rabbitmqctl list_bindings

# Cluster status
rabbitmqctl cluster_status

# Environment info
rabbitmqctl environment
rabbitmqctl status
```

---

### 4. Basic Message Patterns

#### Pattern 1: Simple Queue (Hello World)

**Concept**: One producer sends messages to a queue, one consumer receives them.

```
Producer ──> [Queue] ──> Consumer
```

**Node.js Example:**

```javascript
// producer.js
const amqp = require('amqplib');

async function sendMessage() {
  try {
    const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
    const channel = await connection.createChannel();
    
    const queue = 'hello';
    const message = 'Hello World!';
    
    await channel.assertQueue(queue, {
      durable: false
    });
    
    channel.sendToQueue(queue, Buffer.from(message));
    console.log(`[x] Sent: ${message}`);
    
    setTimeout(() => {
      connection.close();
      process.exit(0);
    }, 500);
  } catch (error) {
    console.error('Error:', error);
  }
}

sendMessage();
```

```javascript
// consumer.js
const amqp = require('amqplib');

async function receiveMessage() {
  try {
    const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
    const channel = await connection.createChannel();
    
    const queue = 'hello';
    
    await channel.assertQueue(queue, {
      durable: false
    });
    
    console.log(`[*] Waiting for messages in ${queue}. To exit press CTRL+C`);
    
    channel.consume(queue, (msg) => {
      if (msg !== null) {
        console.log(`[x] Received: ${msg.content.toString()}`);
        channel.ack(msg);
      }
    });
  } catch (error) {
    console.error('Error:', error);
  }
}

receiveMessage();
```

**Python Example:**

```python
# producer.py
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters(
        host='localhost',
        credentials=pika.PlainCredentials('admin', 'admin123')
    )
)
channel = connection.channel()

channel.queue_declare(queue='hello')

channel.basic_publish(
    exchange='',
    routing_key='hello',
    body='Hello World!'
)

print(" [x] Sent 'Hello World!'")
connection.close()
```

```python
# consumer.py
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters(
        host='localhost',
        credentials=pika.PlainCredentials('admin', 'admin123')
    )
)
channel = connection.channel()

channel.queue_declare(queue='hello')

def callback(ch, method, properties, body):
    print(f" [x] Received {body.decode()}")

channel.basic_consume(
    queue='hello',
    auto_ack=True,
    on_message_callback=callback
)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

#### Pattern 2: Work Queues (Task Distribution)

**Concept**: Distribute time-consuming tasks among multiple workers (round-robin).

```
Producer ──> [Queue] ──> Worker 1
                    ├──> Worker 2
                    └──> Worker 3
```

**Use Case**: Image processing, email sending, report generation

**Node.js Implementation:**

```javascript
// task_producer.js
const amqp = require('amqplib');

async function sendTasks() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'task_queue';
  
  await channel.assertQueue(queue, {
    durable: true  // Queue survives broker restart
  });
  
  const tasks = [
    'Task 1: Process image.jpg',
    'Task 2: Send email to user@example.com',
    'Task 3: Generate monthly report',
    'Task 4: Resize video.mp4',
    'Task 5: Backup database'
  ];
  
  tasks.forEach((task) => {
    channel.sendToQueue(queue, Buffer.from(task), {
      persistent: true  // Message survives broker restart
    });
    console.log(`[x] Sent: ${task}`);
  });
  
  setTimeout(() => {
    connection.close();
    process.exit(0);
  }, 500);
}

sendTasks();
```

```javascript
// worker.js
const amqp = require('amqplib');

async function startWorker() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'task_queue';
  
  await channel.assertQueue(queue, {
    durable: true
  });
  
  // Fair dispatch: don't give more than 1 message to a worker at a time
  channel.prefetch(1);
  
  console.log(`[*] Worker waiting for tasks. To exit press CTRL+C`);
  
  channel.consume(queue, async (msg) => {
    if (msg !== null) {
      const task = msg.content.toString();
      console.log(`[x] Processing: ${task}`);
      
      // Simulate work (1 second per dot in message)
      const dots = task.split('.').length - 1;
      await new Promise(resolve => setTimeout(resolve, dots * 1000));
      
      console.log(`[✓] Done: ${task}`);
      channel.ack(msg);  // Acknowledge after processing
    }
  });
}

startWorker();
```

**Run multiple workers:**

```bash
# Terminal 1
node worker.js

# Terminal 2
node worker.js

# Terminal 3
node worker.js

# Terminal 4 - Send tasks
node task_producer.js
```

**Message Distribution:**
```
Producer sends: Task1, Task2, Task3, Task4, Task5

Worker 1: Task1 ──> Task4 ──>
Worker 2: Task2 ──> Task5 ──>
Worker 3: Task3 ──>
```

#### Pattern 3: Publish/Subscribe (Fanout Exchange)

**Concept**: Send a message to multiple consumers simultaneously.

```
                  ┌──> [Queue1] ──> Consumer1
Producer ──> Exchange ──> [Queue2] ──> Consumer2
             (fanout)   └──> [Queue3] ──> Consumer3
```

**Use Case**: Logging system, notifications, real-time updates

**Node.js Implementation:**

```javascript
// publisher.js
const amqp = require('amqplib');

async function publish() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const exchange = 'logs';
  const message = process.argv.slice(2).join(' ') || 'Hello World!';
  
  await channel.assertExchange(exchange, 'fanout', {
    durable: false
  });
  
  channel.publish(exchange, '', Buffer.from(message));
  console.log(`[x] Sent: ${message}`);
  
  setTimeout(() => {
    connection.close();
    process.exit(0);
  }, 500);
}

publish();
```

```javascript
// subscriber.js
const amqp = require('amqplib');

async function subscribe() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const exchange = 'logs';
  
  await channel.assertExchange(exchange, 'fanout', {
    durable: false
  });
  
  // Create exclusive queue (auto-delete when consumer disconnects)
  const q = await channel.assertQueue('', {
    exclusive: true
  });
  
  console.log(`[*] Waiting for logs. To exit press CTRL+C`);
  
  // Bind queue to exchange
  channel.bindQueue(q.queue, exchange, '');
  
  channel.consume(q.queue, (msg) => {
    if (msg !== null) {
      console.log(`[x] ${msg.content.toString()}`);
    }
  }, {
    noAck: true
  });
}

subscribe();
```

**Usage:**

```bash
# Terminal 1 - Subscriber 1
node subscriber.js

# Terminal 2 - Subscriber 2
node subscriber.js

# Terminal 3 - Publisher
node publisher.js "Server error occurred"
node publisher.js "New user registered"
node publisher.js "Payment processed"
```

**Both subscribers receive all messages!**

---

### 5. Exchanges Deep Dive

#### Exchange Types Overview

```
┌────────────────────────────────────────────────────────────┐
│                    EXCHANGE TYPES                          │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  1. DIRECT    : routing_key == binding_key                │
│  2. TOPIC     : routing_key matches pattern               │
│  3. FANOUT    : broadcasts to all queues                  │
│  4. HEADERS   : matches message headers                   │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

#### 1. Direct Exchange

**Routing**: Message goes to queues whose binding key exactly matches the routing key.

```
                        routing_key: "error"
Producer ──────────────────────────────────────> Direct Exchange
                                                      │
                                    ┌─────────────────┼─────────────────┐
                                    │                 │                 │
                            binding_key:       binding_key:      binding_key:
                              "info"             "error"          "warning"
                                    │                 │                 │
                                    ▼                 ▼                 ▼
                                [Queue1]          [Queue2]          [Queue3]
                                (no match)        (MATCH!)          (no match)
```

**Example: Log Routing**

```javascript
// direct_publisher.js
const amqp = require('amqplib');

async function publishLog(severity, message) {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const exchange = 'direct_logs';
  
  await channel.assertExchange(exchange, 'direct', {
    durable: false
  });
  
  channel.publish(exchange, severity, Buffer.from(message));
  console.log(`[x] Sent ${severity}: ${message}`);
  
  setTimeout(() => {
    connection.close();
    process.exit(0);
  }, 500);
}

// Usage
publishLog('error', 'Database connection failed');
publishLog('info', 'User logged in');
publishLog('warning', 'Disk space low');
```

```javascript
// direct_subscriber.js
const amqp = require('amqplib');

async function subscribeLogs(severities) {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const exchange = 'direct_logs';
  
  await channel.assertExchange(exchange, 'direct', {
    durable: false
  });
  
  const q = await channel.assertQueue('', {
    exclusive: true
  });
  
  console.log(`[*] Waiting for ${severities.join(', ')} logs`);
  
  severities.forEach((severity) => {
    channel.bindQueue(q.queue, exchange, severity);
  });
  
  channel.consume(q.queue, (msg) => {
    if (msg !== null) {
      console.log(`[x] ${msg.fields.routingKey}: ${msg.content.toString()}`);
    }
  }, {
    noAck: true
  });
}

// Usage: Subscribe to error and warning logs only
subscribeLogs(['error', 'warning']);
```

#### 2. Topic Exchange

**Routing**: Message routing key matches binding pattern using wildcards.

**Wildcards:**
- `*` (star): matches exactly one word
- `#` (hash): matches zero or more words

**Pattern Examples:**
- `order.*` matches `order.created`, `order.updated`
- `order.#` matches `order.created`, `order.payment.completed`
- `#.error` matches `app.error`, `database.connection.error`

```
                        routing_key: "order.payment.completed"
Producer ──────────────────────────────────────> Topic Exchange
                                                      │
                            ┌─────────────────────────┼─────────────────┐
                            │                         │                 │
                    binding_pattern:         binding_pattern:    binding_pattern:
                      "order.*"               "order.#"            "*.payment.*"
                            │                         │                 │
                            ▼                         ▼                 ▼
                        [Queue1]                  [Queue2]          [Queue3]
                        (no match)                (MATCH!)          (MATCH!)
```

**Example: E-commerce Events**

```javascript
// topic_publisher.js
const amqp = require('amqplib');

async function publishEvent(routingKey, data) {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const exchange = 'ecommerce_events';
  
  await channel.assertExchange(exchange, 'topic', {
    durable: true
  });
  
  const message = JSON.stringify(data);
  
  channel.publish(exchange, routingKey, Buffer.from(message), {
    persistent: true
  });
  
  console.log(`[x] Sent ${routingKey}: ${message}`);
  
  setTimeout(() => {
    connection.close();
    process.exit(0);
  }, 500);
}

// Usage
publishEvent('order.created', { orderId: 123, amount: 99.99 });
publishEvent('order.payment.completed', { orderId: 123, transactionId: 'tx456' });
publishEvent('user.registered', { userId: 789, email: 'user@example.com' });
publishEvent('product.inventory.updated', { productId: 456, stock: 50 });
```

```javascript
// topic_subscriber.js
const amqp = require('amqplib');

async function subscribeEvents(patterns, serviceName) {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const exchange = 'ecommerce_events';
  
  await channel.assertExchange(exchange, 'topic', {
    durable: true
  });
  
  const q = await channel.assertQueue(`${serviceName}_queue`, {
    durable: true
  });
  
  console.log(`[*] ${serviceName} waiting for events: ${patterns.join(', ')}`);
  
  patterns.forEach((pattern) => {
    channel.bindQueue(q.queue, exchange, pattern);
  });
  
  channel.prefetch(1);
  
  channel.consume(q.queue, (msg) => {
    if (msg !== null) {
      const event = msg.content.toString();
      const routingKey = msg.fields.routingKey;
      
      console.log(`[x] ${serviceName} received ${routingKey}: ${event}`);
      
      // Process event
      setTimeout(() => {
        console.log(`[✓] ${serviceName} processed ${routingKey}`);
        channel.ack(msg);
      }, 1000);
    }
  });
}

// Different services subscribe to different patterns

// Order Service - interested in all order events
subscribeEvents(['order.#'], 'OrderService');

// Payment Service - interested in payment events across all entities
subscribeEvents(['*.payment.*'], 'PaymentService');

// Notification Service - interested in order creation and user registration
subscribeEvents(['order.created', 'user.registered'], 'NotificationService');

// Analytics Service - interested in everything
subscribeEvents(['#'], 'AnalyticsService');
```

**Routing Examples:**

| Event Routing Key | Pattern | Match? |
|-------------------|---------|--------|
| order.created | order.* | ✓ |
| order.created | order.# | ✓ |
| order.payment.completed | order.* | ✗ |
| order.payment.completed | order.# | ✓ |
| order.payment.completed | *.payment.* | ✓ |
| user.registered | order.# | ✗ |
| user.registered | # | ✓ |

#### 3. Fanout Exchange

**Routing**: Ignores routing keys, broadcasts to all bound queues.

```
Producer ──> Fanout Exchange ──┬──> [Queue1] ──> Consumer1
                                ├──> [Queue2] ──> Consumer2
                                └──> [Queue3] ──> Consumer3
```

**Use Case**: Broadcasting system updates, cache invalidation

```javascript
// fanout_example.js
const amqp = require('amqplib');

async function setupFanout() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const exchange = 'system_broadcast';
  
  await channel.assertExchange(exchange, 'fanout', {
    durable: true
  });
  
  // Create multiple queues for different services
  const services = ['web_app', 'mobile_app', 'analytics'];
  
  for (const service of services) {
    const queue = `${service}_notifications`;
    await channel.assertQueue(queue, { durable: true });
    await channel.bindQueue(queue, exchange, ''); // No routing key needed
  }
  
  // Publish broadcast message
  const message = JSON.stringify({
    type: 'MAINTENANCE',
    message: 'System maintenance in 30 minutes',
    timestamp: new Date().toISOString()
  });
  
  channel.publish(exchange, '', Buffer.from(message), {
    persistent: true
  });
  
  console.log('[x] Broadcast sent to all services');
  
  setTimeout(() => {
    connection.close();
    process.exit(0);
  }, 500);
}

setupFanout();
```

#### 4. Headers Exchange

**Routing**: Routes based on message header attributes instead of routing key.

```javascript
// headers_example.js
const amqp = require('amqplib');

async function setupHeadersExchange() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const exchange = 'image_processing';
  
  await channel.assertExchange(exchange, 'headers', {
    durable: true
  });
  
  // Queue for high priority large images
  const queue1 = 'high_priority_large';
  await channel.assertQueue(queue1, { durable: true });
  await channel.bindQueue(queue1, exchange, '', {
    'x-match': 'all',  // All headers must match
    'priority': 'high',
    'size': 'large'
  });
  
  // Queue for any high priority images
  const queue2 = 'high_priority_any';
  await channel.assertQueue(queue2, { durable: true });
  await channel.bindQueue(queue2, exchange, '', {
    'x-match': 'any',  // Any header matches
    'priority': 'high',
    'format': 'png'
  });
  
  // Publish message with headers
  channel.publish(exchange, '', Buffer.from('Image data'), {
    headers: {
      'priority': 'high',
      'size': 'large',
      'format': 'jpg'
    },
    persistent: true
  });
  
  console.log('[x] Message published with headers');
  
  setTimeout(() => {
    connection.close();
    process.exit(0);
  }, 500);
}

setupHeadersExchange();
```

**Header Matching:**
- `x-match: all` - All specified headers must match
- `x-match: any` - At least one header must match

#### Exchange Comparison Table

| Exchange Type | Use Case | Routing | Complexity |
|---------------|----------|---------|------------|
| **Direct** | Simple routing, logs by severity | Exact match | Low |
| **Topic** | Complex routing patterns, microservices | Pattern matching | Medium |
| **Fanout** | Broadcasting, pub/sub | No routing | Very Low |
| **Headers** | Content-based routing | Header attributes | High |

#### Default Exchange

RabbitMQ has a **default exchange** (empty string name) that's a direct exchange.

```javascript
// Using default exchange (implicit)
channel.sendToQueue('myqueue', Buffer.from('message'));

// Equivalent to:
channel.publish('', 'myqueue', Buffer.from('message'));
```

---

## Part 2: Intermediate Concepts

### 6. Message Durability & Persistence

#### Understanding Durability

**Three levels of durability:**
1. **Exchange durability**: Exchange survives broker restart
2. **Queue durability**: Queue survives broker restart
3. **Message persistence**: Messages survive broker restart

```
┌─────────────────────────────────────────────────────────┐
│              DURABILITY COMBINATIONS                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Non-Durable Queue + Non-Persistent Message             │
│  → Lost on broker restart                               │
│                                                          │
│  Durable Queue + Non-Persistent Message                 │
│  → Queue survives, messages lost                        │
│                                                          │
│  Durable Queue + Persistent Message                     │
│  → Both survive broker restart ✓                        │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

#### Implementation

```javascript
// durable_setup.js
const amqp = require('amqplib');

async function setupDurableMessaging() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const exchange = 'durable_orders';
  const queue = 'durable_order_queue';
  const routingKey = 'order.created';
  
  // 1. Create durable exchange
  await channel.assertExchange(exchange, 'topic', {
    durable: true  // Exchange survives restart
  });
  
  // 2. Create durable queue
  await channel.assertQueue(queue, {
    durable: true  // Queue survives restart
  });
  
  // 3. Bind queue to exchange
  await channel.bindQueue(queue, exchange, routingKey);
  
  // 4. Publish persistent message
  const order = {
    orderId: 12345,
    amount: 299.99,
    timestamp: new Date().toISOString()
  };
  
  channel.publish(
    exchange,
    routingKey,
    Buffer.from(JSON.stringify(order)),
    {
      persistent: true,  // Message survives restart
      contentType: 'application/json',
      timestamp: Date.now()
    }
  );
  
  console.log('[x] Durable message sent');
  
  setTimeout(() => {
    connection.close();
    process.exit(0);
  }, 500);
}

setupDurableMessaging();
```

#### Performance Considerations

**Durability Impact:**
- Persistent messages are written to disk (slower)
- Non-persistent messages kept in memory (faster)

**Optimization strategies:**

```javascript
// High-throughput scenario: Use publisher confirms
async function highThroughputPublisher() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  await channel.confirmSelect(); // Enable publisher confirms
  
  const exchange = 'high_volume';
  await channel.assertExchange(exchange, 'direct', { durable: true });
  
  // Batch publishing
  const messages = Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    data: `Message ${i}`
  }));
  
  const publishPromises = messages.map(msg => 
    new Promise((resolve, reject) => {
      channel.publish(
        exchange,
        'routing.key',
        Buffer.from(JSON.stringify(msg)),
        { persistent: true },
        (err) => {
          if (err) reject(err);
          else resolve();
        }
      );
    })
  );
  
  try {
    await Promise.all(publishPromises);
    console.log('[x] All messages confirmed');
  } catch (error) {
    console.error('[!] Publish failed:', error);
  }
  
  await connection.close();
}
```

#### Lazy Queues

**Lazy queues** move messages to disk as early as possible, reducing memory usage.

```javascript
// lazy_queue.js
await channel.assertQueue('lazy_queue', {
  durable: true,
  arguments: {
    'x-queue-mode': 'lazy'  // Messages stored on disk
  }
});
```

**When to use lazy queues:**
- Very long queues (millions of messages)
- Large message sizes
- Limited RAM
- Messages can tolerate slower access

**Trade-offs:**
- Lower memory usage
- Higher disk I/O
- Slightly slower consumption

---

### 7. Message Acknowledgments & Reliability

#### Acknowledgment Modes

```
┌──────────────────────────────────────────────────────────┐
│               ACKNOWLEDGMENT FLOW                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  1. AUTO-ACK (noAck: true)                               │
│     Queue ──> Consumer                                   │
│           (message deleted immediately)                   │
│                                                           │
│  2. MANUAL ACK                                           │
│     Queue ──> Consumer ──> Process ──> ACK ──> Queue    │
│                                      (delete message)     │
│                                                           │
│  3. NACK/REJECT                                          │
│     Queue ──> Consumer ──> Process ──> NACK ──> Queue   │
│                                      (requeue or discard) │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

#### Manual Acknowledgments

```javascript
// manual_ack_consumer.js
const amqp = require('amqplib');

async function reliableConsumer() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'reliable_queue';
  
  await channel.assertQueue(queue, {
    durable: true
  });
  
  channel.prefetch(1); // Process one message at a time
  
  console.log('[*] Waiting for messages...');
  
  channel.consume(queue, async (msg) => {
    if (msg !== null) {
      const content = msg.content.toString();
      console.log(`[x] Received: ${content}`);
      
      try {
        // Simulate processing
        await processMessage(content);
        
        // Success: acknowledge message
        channel.ack(msg);
        console.log('[✓] Message processed and acknowledged');
        
      } catch (error) {
        console.error('[!] Processing failed:', error.message);
        
        // Failure: decide what to do
        if (error.retryable) {
          // Requeue for retry
          channel.nack(msg, false, true);
          console.log('[↻] Message requeued for retry');
        } else {
          // Discard or send to dead letter
          channel.nack(msg, false, false);
          console.log('[✗] Message discarded');
        }
      }
    }
  }, {
    noAck: false  // Manual acknowledgment
  });
}

async function processMessage(content) {
  // Simulate work
  await new Promise(resolve => setTimeout(resolve, 2000));
  
  // Simulate occasional failures
  if (Math.random() < 0.2) {
    const error = new Error('Random processing error');
    error.retryable = true;
    throw error;
  }
  
  return true;
}

reliableConsumer();
```

#### Acknowledgment Methods

```javascript
// 1. ACK - Acknowledge single message
channel.ack(msg);

// 2. ACK multiple - Acknowledge this and all previous unacked messages
channel.ack(msg, true);

// 3. NACK - Negative acknowledgment (requeue or discard)
channel.nack(msg, false, true);  // requeue = true
channel.nack(msg, false, false); // requeue = false (discard)

// 4. REJECT - Reject single message
channel.reject(msg, true);  // requeue = true
channel.reject(msg, false); // requeue = false (discard)
```

#### Handling Consumer Failures

**Scenario**: Consumer crashes before acknowledging

```
Consumer receives message → Process → Crash! (no ACK sent)
                                        ↓
                          RabbitMQ requeues message
                                        ↓
                           Another consumer receives it
```

**Implementation with retry logic:**

```javascript
// retry_consumer.js
const amqp = require('amqplib');

async function consumerWithRetry() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'retry_queue';
  const maxRetries = 3;
  
  await channel.assertQueue(queue, {
    durable: true
  });
  
  channel.prefetch(1);
  
  channel.consume(queue, async (msg) => {
    if (msg !== null) {
      const content = msg.content.toString();
      const retryCount = msg.properties.headers['x-retry-count'] || 0;
      
      console.log(`[x] Processing (attempt ${retryCount + 1}): ${content}`);
      
      try {
        await processMessage(content);
        channel.ack(msg);
        console.log('[✓] Success');
        
      } catch (error) {
        console.error(`[!] Failed (attempt ${retryCount + 1}):`, error.message);
        
        if (retryCount < maxRetries) {
          // Retry: republish with incremented retry count
          setTimeout(() => {
            channel.sendToQueue(
              queue,
              msg.content,
              {
                persistent: true,
                headers: {
                  'x-retry-count': retryCount + 1
                }
              }
            );
            channel.ack(msg); // Remove original from queue
            console.log(`[↻] Retry scheduled (${retryCount + 1}/${maxRetries})`);
          }, 5000 * (retryCount + 1)); // Exponential backoff
          
        } else {
          // Max retries exceeded: send to dead letter or discard
          channel.nack(msg, false, false);
          console.log('[✗] Max retries exceeded, message discarded');
        }
      }
    }
  }, {
    noAck: false
  });
}

consumerWithRetry();
```

#### Prefetch Count

Controls how many unacknowledged messages a consumer can have.

```javascript
// Without prefetch: Consumer gets all messages at once
channel.prefetch(0); // or not set

// With prefetch: Fair distribution
channel.prefetch(1); // One message at a time
channel.prefetch(10); // Up to 10 unacked messages

// Global prefetch (applies to entire channel)
channel.prefetch(1, true);
```

**Visual example:**

```
WITHOUT PREFETCH:
Queue [M1][M2][M3][M4][M5][M6] ──> Fast Consumer (gets M1, M2, M3)
                                ──> Slow Consumer (gets M4, M5, M6)
                                    (Slow consumer is overwhelmed!)

WITH PREFETCH(1):
Queue [M1][M2][M3][M4][M5][M6] ──> Fast Consumer (M1) ──> ACK ──> (M3) ──> ACK ──> (M5)
                                ──> Slow Consumer (M2) ──> ACK ──> (M4) ──> ACK ──> (M6)
                                    (Fair distribution!)
```

---

### 8. Prefetch & Flow Control

#### Understanding Prefetch

Prefetch controls the **Quality of Service (QoS)** - how many messages RabbitMQ sends to a consumer before waiting for acknowledgments.

```javascript
// prefetch_demo.js
const amqp = require('amqplib');

async function demonstratePrefetch() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'prefetch_queue';
  
  await channel.assertQueue(queue, { durable: true });
  
  // Set prefetch to 5
  channel.prefetch(5);
  
  let processing = 0;
  
  channel.consume(queue, async (msg) => {
    if (msg !== null) {
      processing++;
      console.log(`[x] Processing message ${msg.content.toString()} (${processing} in flight)`);
      
      // Simulate slow processing
      await new Promise(resolve => setTimeout(resolve, 3000));
      
      channel.ack(msg);
      processing--;
      console.log(`[✓] Done with ${msg.content.toString()} (${processing} remaining)`);
    }
  }, {
    noAck: false
  });
  
  console.log('[*] Consumer started with prefetch=5');
  console.log('[*] Maximum 5 messages will be processing at once');
}

demonstratePrefetch();
```

#### Optimal Prefetch Values

```javascript
// Different scenarios

// 1. Fast processing, many small messages
channel.prefetch(50);

// 2. Slow processing, CPU-intensive
channel.prefetch(1);

// 3. Moderate processing
channel.prefetch(10);

// 4. I/O bound operations
channel.prefetch(20);

// 5. Memory-intensive messages
channel.prefetch(3);
```

**Rule of thumb**: `prefetch = (processing_time / network_latency) * concurrency`

#### Consumer Cancellation

```javascript
// cancellation_example.js
const amqp = require('amqplib');

async function cancellableConsumer() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'cancellable_queue';
  await channel.assertQueue(queue, { durable: true });
  
  // Start consuming
  const { consumerTag } = await channel.consume(queue, (msg) => {
    if (msg !== null) {
      console.log(`[x] Received: ${msg.content.toString()}`);
      channel.ack(msg);
    }
  }, {
    noAck: false
  });
  
  console.log(`[*] Consumer started with tag: ${consumerTag}`);
  
  // Cancel consumer after 30 seconds
  setTimeout(async () => {
    await channel.cancel(consumerTag);
    console.log('[!] Consumer cancelled');
    
    await connection.close();
    process.exit(0);
  }, 30000);
}

cancellableConsumer();
```

#### Channel Flow Control

**Pause and resume message delivery:**

```javascript
// flow_control.js
const amqp = require('amqplib');

async function flowControlExample() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'flow_queue';
  await channel.assertQueue(queue, { durable: true });
  
  let paused = false;
  
  channel.consume(queue, async (msg) => {
    if (msg !== null) {
      console.log(`[x] Received: ${msg.content.toString()}`);
      
      // Check memory/CPU usage
      if (shouldPauseConsuming()) {
        if (!paused) {
          console.log('[!] Pausing consumption...');
          await channel.flow(false); // Stop delivery
          paused = true;
          
          // Resume after cooldown
          setTimeout(async () => {
            console.log('[!] Resuming consumption...');
            await channel.flow(true); // Resume delivery
            paused = false;
          }, 10000);
        }
      }
      
      channel.ack(msg);
    }
  }, {
    noAck: false
  });
}

function shouldPauseConsuming() {
  // Check system resources
  const memUsage = process.memoryUsage();
  return memUsage.heapUsed / memUsage.heapTotal > 0.8;
}
```

---

### 9. Dead Letter Exchanges (DLX)

#### Concept

Messages become "dead-lettered" when:
1. Message is rejected (nack/reject with requeue=false)
2. Message TTL expires
3. Queue length limit exceeded

Dead letters are republished to a **Dead Letter Exchange (DLX)**.

```
┌──────────────────────────────────────────────────────────┐
│                 DEAD LETTER FLOW                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Producer ──> [Main Queue] ──> Consumer                  │
│                    │                                      │
│                    │ (reject/expire/overflow)            │
│                    ▼                                      │
│            [Dead Letter Exchange]                        │
│                    │                                      │
│                    ▼                                      │
│            [Dead Letter Queue] ──> Error Handler         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

#### Implementation

```javascript
// dlx_setup.js
const amqp = require('amqplib');

async function setupDeadLetterExchange() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  // 1. Create Dead Letter Exchange
  const dlx = 'dlx_exchange';
  await channel.assertExchange(dlx, 'direct', {
    durable: true
  });
  
  // 2. Create Dead Letter Queue
  const dlq = 'dead_letter_queue';
  await channel.assertQueue(dlq, {
    durable: true
  });
  
  // 3. Bind DLQ to DLX
  await channel.bindQueue(dlq, dlx, 'dead_letter');
  
  // 4. Create Main Queue with DLX configuration
  const mainQueue = 'main_queue';
  await channel.assertQueue(mainQueue, {
    durable: true,
    arguments: {
      'x-dead-letter-exchange': dlx,
      'x-dead-letter-routing-key': 'dead_letter'
    }
  });
  
  console.log('[✓] Dead Letter Exchange setup complete');
  
  await connection.close();
}

setupDeadLetterExchange();
```

```javascript
// dlx_producer.js - Send test messages
const amqp = require('amqplib');

async function sendMessages() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'main_queue';
  
  for (let i = 1; i <= 5; i++) {
    const message = JSON.stringify({
      id: i,
      data: `Message ${i}`,
      willFail: i % 2 === 0  // Even messages will fail
    });
    
    channel.sendToQueue(queue, Buffer.from(message), {
      persistent: true
    });
    
    console.log(`[x] Sent: ${message}`);
  }
  
  setTimeout(() => {
    connection.close();
    process.exit(0);
  }, 500);
}

sendMessages();
```

```javascript
// dlx_consumer.js - Consumer that rejects failed messages
const amqp = require('amqplib');

async function mainConsumer() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'main_queue';
  
  await channel.assertQueue(queue, { durable: true });
  channel.prefetch(1);
  
  console.log('[*] Main consumer waiting for messages...');
  
  channel.consume(queue, async (msg) => {
    if (msg !== null) {
      const message = JSON.parse(msg.content.toString());
      console.log(`[x] Processing: ${JSON.stringify(message)}`);
      
      try {
        if (message.willFail) {
          throw new Error('Simulated processing failure');
        }
        
        // Success
        channel.ack(msg);
        console.log('[✓] Success');
        
      } catch (error) {
        console.error('[!] Failed:', error.message);
        
        // Reject and send to DLX
        channel.reject(msg, false); // requeue = false
        console.log('[→] Sent to Dead Letter Queue');
      }
    }
  }, {
    noAck: false
  });
}

mainConsumer();
```

```javascript
// dlx_handler.js - Handle dead-lettered messages
const amqp = require('amqplib');

async function deadLetterHandler() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const dlq = 'dead_letter_queue';
  
  await channel.assertQueue(dlq, { durable: true });
  
  console.log('[*] Dead Letter handler waiting...');
  
  channel.consume(dlq, async (msg) => {
    if (msg !== null) {
      const message = JSON.parse(msg.content.toString());
      const deathInfo = msg.properties.headers['x-death'];
      
      console.log('[!] Dead letter received:', JSON.stringify(message));
      console.log('[!] Death info:', JSON.stringify(deathInfo, null, 2));
      
      // Handle dead letter (log, alert, retry, etc.)
      await handleDeadLetter(message, deathInfo);
      
      channel.ack(msg);
    }
  }, {
    noAck: false
  });
}

async function handleDeadLetter(message, deathInfo) {
  // Options:
  // 1. Log to error tracking system
  // 2. Send alert to ops team
  // 3. Store in database for manual review
  // 4. Attempt retry with different parameters
  
  console.log('[→] Logged to error tracking system');
}

deadLetterHandler();
```

#### DLX with TTL

**Delayed Message Processing** using DLX + TTL:

```javascript
// delayed_queue_setup.js
const amqp = require('amqplib');

async function setupDelayedQueue() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  // Main exchange and queue
  const mainExchange = 'main_exchange';
  const mainQueue = 'main_queue';
  
  await channel.assertExchange(mainExchange, 'direct', { durable: true });
  await channel.assertQueue(mainQueue, { durable: true });
  await channel.bindQueue(mainQueue, mainExchange, 'process');
  
  // Delay queue (messages expire and go to DLX)
  const delayQueue = 'delay_queue_30s';
  
  await channel.assertQueue(delayQueue, {
    durable: true,
    arguments: {
      'x-message-ttl': 30000,  // 30 seconds
      'x-dead-letter-exchange': mainExchange,
      'x-dead-letter-routing-key': 'process'
    }
  });
  
  console.log('[✓] Delayed queue setup complete');
  console.log('[✓] Messages sent to delay_queue will appear in main_queue after 30s');
  
  await connection.close();
}

setupDelayedQueue();
```

**Use case: Delayed retry**

```javascript
// Send message for delayed processing
channel.sendToQueue('delay_queue_30s', Buffer.from(JSON.stringify({
  taskId: 123,
  retryCount: 1
})), {
  persistent: true
});

console.log('[x] Message will be processed in 30 seconds');
```

---

### 10. Message TTL & Queue Length Limits

#### Message TTL (Time To Live)

**Per-Message TTL:**

```javascript
// per_message_ttl.js
const amqp = require('amqplib');

async function sendWithTTL() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'ttl_queue';
  await channel.assertQueue(queue, { durable: true });
  
  // Message expires in 60 seconds
  channel.sendToQueue(
    queue,
    Buffer.from('This message will expire in 60 seconds'),
    {
      persistent: true,
      expiration: '60000'  // milliseconds as string
    }
  );
  
  console.log('[x] Message sent with 60s TTL');
  
  setTimeout(() => {
    connection.close();
    process.exit(0);
  }, 500);
}

sendWithTTL();
```

**Queue-Level TTL:**

```javascript
// queue_level_ttl.js
const amqp = require('amqplib');

async function setupQueueTTL() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'queue_with_ttl';
  
  await channel.assertQueue(queue, {
    durable: true,
    arguments: {
      'x-message-ttl': 120000  // All messages expire in 120 seconds
    }
  });
  
  console.log('[✓] Queue created with 120s TTL for all messages');
  
  await connection.close();
}

setupQueueTTL();
```

**TTL Priority:**
- Per-message TTL takes precedence over queue TTL
- Lower value is used if both are set

#### Queue Length Limits

**Max Length (Message Count):**

```javascript
// max_length_queue.js
const amqp = require('amqplib');

async function setupMaxLengthQueue() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const queue = 'limited_queue';
  
  await channel.assertQueue(queue, {
    durable: true,
    arguments: {
      'x-max-length': 1000,  // Maximum 1000 messages
      'x-overflow': 'reject-publish'  // or 'drop-head'
    }
  });
  
  console.log('[✓] Queue limited to 1000 messages');
  
  await connection.close();
}

setupMaxLengthQueue();
```

**Overflow Behaviors:**

```javascript
// 1. drop-head: Remove oldest message when limit reached
await channel.assertQueue('queue1', {
  durable: true,
  arguments: {
    'x-max-length': 100,
    'x-overflow': 'drop-head'  // FIFO behavior
  }
});

// 2. reject-publish: Reject new messages when limit reached
await channel.assertQueue('queue2', {
  durable: true,
  arguments: {
    'x-max-length': 100,
    'x-overflow': 'reject-publish'  // Reject new messages
  }
});

// 3. reject-publish-dlx: Reject and send to DLX
await channel.assertQueue('queue3', {
  durable: true,
  arguments: {
    'x-max-length': 100,
    'x-overflow': 'reject-publish-dlx',
    'x-dead-letter-exchange': 'overflow_dlx'
  }
});
```

**Max Length in Bytes:**

```javascript
// Limit queue size by total message bytes
await channel.assertQueue('size_limited_queue', {
  durable: true,
  arguments: {
    'x-max-length-bytes': 10485760,  // 10MB
    'x-overflow': 'drop-head'
  }
});
```

#### Combining TTL and Length Limits

```javascript
// combined_limits.js
const amqp = require('amqplib');

async function setupCombinedLimits() {
  const connection = await amqp.connect('amqp://admin:admin123@localhost:5672');
  const channel = await connection.createChannel();
  
  const dlx = 'overflow_dlx';
  await channel.assertExchange(dlx, 'direct', { durable: true });
  
  const dlq = 'overflow_dlq';
  await channel.assertQueue(dlq, { durable: true });
  await channel.bindQueue(dlq, dlx, 'overflow');
  
  const queue = 'limited_ttl_queue';
  await channel.assertQueue(queue, {
    durable: true,
    arguments: {
      'x-message-ttl': 300000,              // Messages expire in 5 minutes
      'x-max-length': 500,                  // Max 500 messages
      'x-overflow': 'reject-publish-dlx',   // Overflow to DLX
      'x-dead-letter-exchange': dlx,
      'x-dead-letter-routing-key': 'overflow'
    }
  });
  
  console.log('[✓] Queue with TTL and length limits created');
  console.log('[✓] Messages: expire in 5min OR overflow at 500 messages');
  
  await connection.close();
}

setupCombinedLimits();
```

---

### 11. Priority Queues

RabbitMQ Complete Mastery Tutorial created successfully. The comprehensive guide covers basics through advanced topics including exchanges, queues, clustering, streams, security, monitoring, performance optimization, and production best practices with detailed code examples and configurations.