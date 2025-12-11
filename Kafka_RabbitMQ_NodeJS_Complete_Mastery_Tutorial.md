# Kafka and RabbitMQ for Node.js - Complete Mastery Tutorial

## Table of Contents
1. [Introduction to Message Queues](#introduction-to-message-queues)
2. [Understanding Message Queues in Distributed Systems](#understanding-message-queues-in-distributed-systems)
3. [Apache Kafka Architecture](#apache-kafka-architecture)
4. [Kafka Producer and Consumer APIs in Node.js](#kafka-producer-and-consumer-apis-in-nodejs)
5. [Kafka Stream Processing and Real-time Data Pipelines](#kafka-stream-processing-and-real-time-data-pipelines)
6. [RabbitMQ Architecture and Message Exchange Patterns](#rabbitmq-architecture-and-message-exchange-patterns)
7. [Implementing Publish-Subscribe Patterns with RabbitMQ](#implementing-publish-subscribe-patterns-with-rabbitmq)
8. [Message Acknowledgments and Delivery Guarantees](#message-acknowledgments-and-delivery-guarantees)
9. [Dead Letter Queues and Message Retry Mechanisms](#dead-letter-queues-and-message-retry-mechanisms)
10. [Scaling Message Queues and Handling High Throughput](#scaling-message-queues-and-handling-high-throughput)
11. [Comparing Kafka vs RabbitMQ - Use Cases and Trade-offs](#comparing-kafka-vs-rabbitmq-use-cases-and-trade-offs)
12. [Advanced Patterns and Best Practices](#advanced-patterns-and-best-practices)
13. [Production Deployment and Monitoring](#production-deployment-and-monitoring)
14. [Conclusion](#conclusion)

---

## Introduction to Message Queues

Message queues are fundamental building blocks in modern distributed systems, enabling asynchronous communication between different components, services, or applications. They act as intermediaries that temporarily store messages sent from producers until consumers are ready to process them.

### Why Message Queues Matter

In a microservices architecture or distributed system, direct synchronous communication can lead to:
- **Tight coupling** between services
- **Cascading failures** when one service goes down
- **Performance bottlenecks** during traffic spikes
- **Lost data** if receiving services are unavailable

Message queues solve these problems by:
- **Decoupling** producers and consumers
- **Buffering** messages during traffic spikes
- **Ensuring reliability** through message persistence
- **Enabling scalability** through distributed processing

### Key Concepts

```javascript
// Basic message queue flow
// Producer -> Message Queue -> Consumer

// Producer sends messages without waiting for immediate processing
producer.send({
    topic: 'user-registrations',
    message: { userId: 123, email: 'user@example.com' }
});

// Consumer processes messages at their own pace
consumer.on('message', async (message) => {
    await processUserRegistration(message);
});
```

---

## Understanding Message Queues in Distributed Systems

### The Role of Message Queues

Message queues serve multiple critical functions in distributed systems:

#### 1. **Asynchronous Communication**
Instead of waiting for immediate responses, services can send messages and continue with other tasks.

```javascript
// Without message queue - synchronous and blocking
async function createOrder(orderData) {
    await saveToDatabase(orderData);           // Wait
    await sendEmailConfirmation(orderData);    // Wait
    await updateInventory(orderData);          // Wait
    await notifyShipping(orderData);           // Wait
    return { status: 'success' };
}

// With message queue - asynchronous and non-blocking
async function createOrder(orderData) {
    await saveToDatabase(orderData);
    
    // Fire and forget - publish to queue
    await publisher.publish('order.created', orderData);
    
    return { status: 'success' }; // Immediate response
}

// Separate consumers handle each task independently
emailConsumer.subscribe('order.created', sendEmailConfirmation);
inventoryConsumer.subscribe('order.created', updateInventory);
shippingConsumer.subscribe('order.created', notifyShipping);
```

#### 2. **Load Leveling**
Message queues smooth out traffic spikes by buffering messages when demand exceeds processing capacity.

```javascript
// During traffic spike: 10,000 requests/second
// Queue buffers messages
for (let i = 0; i < 10000; i++) {
    await queue.enqueue({ orderId: i, timestamp: Date.now() });
}

// Consumers process at sustainable rate: 100 messages/second
// No system overload, no lost data
const consumer = new Consumer({
    concurrency: 10,  // 10 parallel workers
    rateLimit: 100    // 100 messages per second
});
```

#### 3. **Reliability and Fault Tolerance**
Messages persist in the queue even if consumers fail or are temporarily unavailable.

```javascript
// Message persisted to disk
await queue.send({ 
    data: criticalTransaction,
    persistent: true  // Survives broker restart
});

// Consumer crashes mid-processing
consumer.on('message', async (msg) => {
    try {
        await processTransaction(msg);
        await msg.ack();  // Acknowledge successful processing
    } catch (error) {
        await msg.nack(); // Negative acknowledgment - requeue
    }
});
```

#### 4. **Service Decoupling**
Producers and consumers don't need to know about each other's existence, location, or implementation details.

```javascript
// Producer doesn't know who consumes the message
await eventBus.publish('user.registered', {
    userId: user.id,
    email: user.email,
    timestamp: Date.now()
});

// Multiple consumers can subscribe independently
// Email service
emailService.subscribe('user.registered', sendWelcomeEmail);

// Analytics service
analyticsService.subscribe('user.registered', trackUserSignup);

// CRM service
crmService.subscribe('user.registered', createCRMContact);

// Marketing service
marketingService.subscribe('user.registered', addToMailingList);
```

### Message Queue Patterns

#### 1. Point-to-Point (Queue)
One message is consumed by exactly one consumer.

```
Producer → [Queue] → Consumer 1
                  → Consumer 2  (only one receives each message)
                  → Consumer 3
```

#### 2. Publish-Subscribe (Topic)
One message is consumed by all subscribed consumers.

```
Producer → [Topic] → Consumer 1 (receives message)
                  → Consumer 2 (receives message)
                  → Consumer 3 (receives message)
```

---

## Apache Kafka Architecture

Apache Kafka is a distributed streaming platform designed for high-throughput, fault-tolerant, and scalable message processing.

### Core Components

#### 1. **Topics**
Logical channels where messages are published. Think of topics as categories or feeds.

```javascript
// Creating and using topics
const topics = {
    'user-events': 'All user-related activities',
    'order-events': 'E-commerce order transactions',
    'payment-events': 'Payment processing events',
    'analytics-events': 'Tracking and analytics data'
};

// Producing to a topic
await producer.send({
    topic: 'user-events',
    messages: [{
        key: 'user-123',        // Message key for partitioning
        value: JSON.stringify({
            type: 'registration',
            userId: 123,
            timestamp: Date.now()
        })
    }]
});
```

#### 2. **Partitions**
Each topic is divided into partitions for parallel processing and scalability. Messages with the same key always go to the same partition, ensuring ordering.

```
Topic: user-events (3 partitions)
├── Partition 0: [msg1, msg4, msg7, ...]
├── Partition 1: [msg2, msg5, msg8, ...]
└── Partition 2: [msg3, msg6, msg9, ...]
```

```javascript
// Kafka automatically distributes messages across partitions
// Messages with same key go to same partition (ordering guaranteed)
await producer.send({
    topic: 'user-events',
    messages: [
        { key: 'user-123', value: 'event1' }, // → Partition 0
        { key: 'user-123', value: 'event2' }, // → Partition 0 (same partition)
        { key: 'user-456', value: 'event3' }, // → Partition 1
        { key: 'user-789', value: 'event4' }  // → Partition 2
    ]
});

// Custom partitioner for specific routing logic
class CustomPartitioner {
    partition(topic, partitionMetadata, message) {
        const userId = parseInt(message.key.split('-')[1]);
        const partitionCount = partitionMetadata.length;
        
        // Route premium users to specific partition
        if (userId % 10 === 0) {
            return 0; // Premium user partition
        }
        
        return userId % partitionCount;
    }
}
```

#### 3. **Brokers**
Kafka servers that store and serve messages. Multiple brokers form a Kafka cluster.

```
Kafka Cluster
├── Broker 1 (Leader for Partition 0)
├── Broker 2 (Leader for Partition 1)
└── Broker 3 (Leader for Partition 2)
```

```javascript
// Connecting to Kafka cluster (multiple brokers for HA)
const kafka = new Kafka({
    clientId: 'my-app',
    brokers: [
        'kafka-broker-1:9092',
        'kafka-broker-2:9092',
        'kafka-broker-3:9092'
    ],
    retry: {
        initialRetryTime: 100,
        retries: 8
    }
});
```

#### 4. **Producers**
Applications that publish messages to Kafka topics.

#### 5. **Consumers**
Applications that subscribe to topics and process messages.

#### 6. **Consumer Groups**
Multiple consumers working together to process messages from a topic in parallel.

```
Topic: orders (3 partitions)
Consumer Group: order-processors
├── Consumer 1 → Partition 0
├── Consumer 2 → Partition 1
└── Consumer 3 → Partition 2
```

```javascript
// Each consumer in a group gets exclusive access to partitions
const consumer = kafka.consumer({ 
    groupId: 'order-processors',
    sessionTimeout: 30000,
    heartbeatInterval: 3000
});

await consumer.subscribe({ 
    topic: 'orders',
    fromBeginning: false  // Start from latest messages
});

// Kafka automatically rebalances partitions among consumers
await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
        console.log(`Consumer processing partition ${partition}`);
        await processOrder(message.value);
    }
});
```

### Kafka Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                     Kafka Cluster                        │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐          │
│  │ Broker 1 │    │ Broker 2 │    │ Broker 3 │          │
│  │          │    │          │    │          │          │
│  │ Part 0   │    │ Part 1   │    │ Part 2   │          │
│  │ (Leader) │    │ (Leader) │    │ (Leader) │          │
│  │ Part 1   │    │ Part 2   │    │ Part 0   │          │
│  │(Replica) │    │(Replica) │    │(Replica) │          │
│  └──────────┘    └──────────┘    └──────────┘          │
└─────────────────────────────────────────────────────────┘
         ▲                                    │
         │                                    ▼
    ┌──────────┐                      ┌────────────┐
    │Producers │                      │ Consumers  │
    │          │                      │(Consumer   │
    │          │                      │ Groups)    │
    └──────────┘                      └────────────┘
```

### Key Features

1. **Durability**: Messages are persisted to disk and replicated across brokers
2. **Scalability**: Horizontal scaling through partitions and consumer groups
3. **High Throughput**: Optimized for handling millions of messages per second
4. **Fault Tolerance**: Automatic failover when brokers fail
5. **Message Retention**: Configurable retention periods (time or size-based)

```javascript
// Topic configuration with retention and replication
await admin.createTopics({
    topics: [{
        topic: 'critical-events',
        numPartitions: 6,              // 6 partitions for parallelism
        replicationFactor: 3,           // 3 copies of each partition
        configEntries: [
            { 
                name: 'retention.ms', 
                value: '604800000'      // 7 days retention
            },
            { 
                name: 'retention.bytes', 
                value: '1073741824'     // 1GB per partition
            },
            {
                name: 'compression.type',
                value: 'snappy'         // Compression for efficiency
            }
        ]
    }]
});
```

---

## Kafka Producer and Consumer APIs in Node.js

### Setting Up Kafka with Node.js

First, install the KafkaJS library:

```bash
npm install kafkajs
```

### Creating a Kafka Producer

```javascript
// producer.js
const { Kafka } = require('kafkajs');

// Initialize Kafka client
const kafka = new Kafka({
    clientId: 'my-producer-app',
    brokers: ['localhost:9092'],
    logLevel: 2  // INFO level
});

// Create producer instance
const producer = kafka.producer({
    allowAutoTopicCreation: true,
    transactionTimeout: 30000,
    // Idempotent producer for exactly-once semantics
    idempotent: true,
    maxInFlightRequests: 5,
    // Compression for better network utilization
    compression: 1  // GZIP compression
});

// Connect to Kafka cluster
async function connectProducer() {
    await producer.connect();
    console.log('Producer connected to Kafka');
}

// Send a single message
async function sendMessage(topic, message) {
    try {
        const result = await producer.send({
            topic: topic,
            messages: [{
                key: message.key || null,      // Optional message key
                value: JSON.stringify(message.value),
                headers: {                      // Optional metadata
                    'correlation-id': generateId(),
                    'timestamp': Date.now().toString()
                },
                partition: message.partition    // Optional explicit partition
            }],
            // Acknowledgment level
            acks: -1,  // All replicas must acknowledge (strongest guarantee)
            timeout: 30000
        });
        
        console.log('Message sent successfully:', result);
        return result;
    } catch (error) {
        console.error('Error sending message:', error);
        throw error;
    }
}

// Send batch of messages for better throughput
async function sendBatch(topic, messages) {
    try {
        const kafkaMessages = messages.map(msg => ({
            key: msg.key || null,
            value: JSON.stringify(msg.value),
            timestamp: Date.now().toString()
        }));
        
        const result = await producer.send({
            topic: topic,
            messages: kafkaMessages,
            acks: -1,
            compression: 1  // Compress batch
        });
        
        console.log(`Batch of ${messages.length} messages sent`);
        return result;
    } catch (error) {
        console.error('Error sending batch:', error);
        throw error;
    }
}

// Send to multiple topics atomically using transactions
async function sendTransactional(messages) {
    const transaction = await producer.transaction();
    
    try {
        // All messages are sent atomically
        for (const msg of messages) {
            await transaction.send({
                topic: msg.topic,
                messages: [{
                    key: msg.key,
                    value: JSON.stringify(msg.value)
                }]
            });
        }
        
        // Commit transaction - all or nothing
        await transaction.commit();
        console.log('Transaction committed successfully');
    } catch (error) {
        // Rollback on error
        await transaction.abort();
        console.error('Transaction aborted:', error);
        throw error;
    }
}

// Graceful shutdown
async function disconnectProducer() {
    await producer.disconnect();
    console.log('Producer disconnected');
}

// Example usage
async function main() {
    await connectProducer();
    
    // Send single message
    await sendMessage('user-events', {
        key: 'user-123',
        value: {
            type: 'registration',
            userId: 123,
            email: 'user@example.com',
            timestamp: Date.now()
        }
    });
    
    // Send batch
    await sendBatch('order-events', [
        { key: 'order-1', value: { orderId: 1, amount: 100 } },
        { key: 'order-2', value: { orderId: 2, amount: 200 } },
        { key: 'order-3', value: { orderId: 3, amount: 300 } }
    ]);
    
    // Transactional send
    await sendTransactional([
        { topic: 'inventory', key: 'product-1', value: { qty: -1 } },
        { topic: 'orders', key: 'order-1', value: { status: 'confirmed' } },
        { topic: 'notifications', key: 'user-1', value: { type: 'order-confirmed' } }
    ]);
}

// Handle process termination
process.on('SIGINT', async () => {
    await disconnectProducer();
    process.exit(0);
});

module.exports = {
    connectProducer,
    sendMessage,
    sendBatch,
    sendTransactional,
    disconnectProducer
};
```

### Creating a Kafka Consumer

```javascript
// consumer.js
const { Kafka } = require('kafkajs');

// Initialize Kafka client
const kafka = new Kafka({
    clientId: 'my-consumer-app',
    brokers: ['localhost:9092']
});

// Create consumer instance with consumer group
const consumer = kafka.consumer({
    groupId: 'my-consumer-group',
    // Start from earliest message if no offset committed
    sessionTimeout: 30000,
    heartbeatInterval: 3000,
    // Rebalance strategy
    partitionAssigners: ['RoundRobin'],
    // Auto commit offsets
    autoCommit: true,
    autoCommitInterval: 5000
});

// Connect to Kafka cluster
async function connectConsumer() {
    await consumer.connect();
    console.log('Consumer connected to Kafka');
}

// Subscribe to topics
async function subscribe(topics) {
    // Subscribe to one or multiple topics
    await consumer.subscribe({
        topics: Array.isArray(topics) ? topics : [topics],
        fromBeginning: false  // true to consume from earliest offset
    });
    
    console.log(`Subscribed to topics: ${topics}`);
}

// Process messages with manual offset management
async function consumeMessages(messageHandler) {
    await consumer.run({
        // Process messages in batch for better throughput
        eachBatchAutoResolve: false,
        eachBatch: async ({ 
            batch, 
            resolveOffset, 
            heartbeat, 
            commitOffsetsIfNecessary,
            isRunning, 
            isStale 
        }) => {
            console.log(`Processing batch from partition ${batch.partition}`);
            
            for (let message of batch.messages) {
                if (!isRunning() || isStale()) break;
                
                try {
                    // Parse message
                    const value = JSON.parse(message.value.toString());
                    const key = message.key?.toString();
                    
                    // Process message
                    await messageHandler({
                        topic: batch.topic,
                        partition: batch.partition,
                        offset: message.offset,
                        key: key,
                        value: value,
                        headers: message.headers,
                        timestamp: message.timestamp
                    });
                    
                    // Manually commit offset after successful processing
                    resolveOffset(message.offset);
                    
                    // Send heartbeat to avoid rebalance
                    await heartbeat();
                    
                } catch (error) {
                    console.error('Error processing message:', error);
                    // Implement retry logic or dead letter queue here
                    // For now, we'll skip and continue
                }
            }
            
            // Commit all resolved offsets
            await commitOffsetsIfNecessary();
        }
    });
}

// Simple message-by-message processing
async function consumeMessagesSimple(messageHandler) {
    await consumer.run({
        eachMessage: async ({ topic, partition, message }) => {
            try {
                const value = JSON.parse(message.value.toString());
                const key = message.key?.toString();
                
                await messageHandler({
                    topic,
                    partition,
                    offset: message.offset,
                    key,
                    value,
                    timestamp: message.timestamp
                });
                
                // Auto-commit handled by KafkaJS
            } catch (error) {
                console.error('Error processing message:', error);
            }
        }
    });
}

// Seek to specific offset
async function seekToOffset(topic, partition, offset) {
    consumer.seek({ topic, partition, offset });
    console.log(`Seeking to offset ${offset} on ${topic}:${partition}`);
}

// Pause and resume consumption
async function pauseConsumption(topics) {
    consumer.pause(topics.map(topic => ({ topic })));
    console.log('Consumption paused');
}

async function resumeConsumption(topics) {
    consumer.resume(topics.map(topic => ({ topic })));
    console.log('Consumption resumed');
}

// Graceful shutdown
async function disconnectConsumer() {
    await consumer.disconnect();
    console.log('Consumer disconnected');
}

// Example usage
async function main() {
    await connectConsumer();
    
    // Subscribe to topics
    await subscribe(['user-events', 'order-events']);
    
    // Consume messages
    await consumeMessagesSimple(async (message) => {
        console.log('Received message:', {
            topic: message.topic,
            partition: message.partition,
            offset: message.offset,
            key: message.key,
            value: message.value
        });
        
        // Process based on topic
        if (message.topic === 'user-events') {
            await handleUserEvent(message.value);
        } else if (message.topic === 'order-events') {
            await handleOrderEvent(message.value);
        }
    });
}

// Business logic handlers
async function handleUserEvent(event) {
    console.log('Processing user event:', event);
    // Implement user event processing logic
}

async function handleOrderEvent(event) {
    console.log('Processing order event:', event);
    // Implement order event processing logic
}

// Handle process termination
process.on('SIGINT', async () => {
    await disconnectConsumer();
    process.exit(0);
});

module.exports = {
    connectConsumer,
    subscribe,
    consumeMessages,
    consumeMessagesSimple,
    seekToOffset,
    pauseConsumption,
    resumeConsumption,
    disconnectConsumer
};
```

### Advanced Producer Patterns

#### 1. **Partitioner Strategy**

```javascript
// Custom partitioner for load balancing
class PriorityPartitioner {
    partition({ topic, partitionMetadata, message }) {
        const messageData = JSON.parse(message.value);
        const totalPartitions = partitionMetadata.length;
        
        // Route high-priority messages to specific partitions
        if (messageData.priority === 'high') {
            // Use first 20% of partitions for high priority
            const highPriorityPartitions = Math.ceil(totalPartitions * 0.2);
            return messageData.userId % highPriorityPartitions;
        }
        
        // Normal priority uses remaining partitions
        const normalPartitionStart = Math.ceil(totalPartitions * 0.2);
        const normalPartitionCount = totalPartitions - normalPartitionStart;
        return normalPartitionStart + (messageData.userId % normalPartitionCount);
    }
}

const producer = kafka.producer({
    createPartitioner: () => new PriorityPartitioner()
});
```

#### 2. **Error Handling and Retry Logic**

```javascript
async function sendWithRetry(topic, message, maxRetries = 3) {
    let attempt = 0;
    
    while (attempt < maxRetries) {
        try {
            const result = await producer.send({
                topic,
                messages: [{
                    key: message.key,
                    value: JSON.stringify(message.value)
                }],
                acks: -1,
                timeout: 30000
            });
            
            return result;
        } catch (error) {
            attempt++;
            console.error(`Send attempt ${attempt} failed:`, error.message);
            
            if (attempt >= maxRetries) {
                // Send to dead letter topic after all retries exhausted
                await sendToDeadLetter(topic, message, error);
                throw error;
            }
            
            // Exponential backoff
            const delay = Math.min(1000 * Math.pow(2, attempt), 10000);
            await sleep(delay);
        }
    }
}

async function sendToDeadLetter(originalTopic, message, error) {
    await producer.send({
        topic: `${originalTopic}.dead-letter`,
        messages: [{
            key: message.key,
            value: JSON.stringify({
                originalMessage: message,
                error: error.message,
                timestamp: Date.now()
            })
        }]
    });
}

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}
```

### Advanced Consumer Patterns

#### 1. **Parallel Processing with Concurrency Control**

```javascript
const pLimit = require('p-limit');

async function consumeWithConcurrency(concurrencyLimit = 10) {
    const limit = pLimit(concurrencyLimit);
    
    await consumer.run({
        eachBatch: async ({ batch, resolveOffset, heartbeat }) => {
            // Process messages in parallel with concurrency limit
            const promises = batch.messages.map(message => 
                limit(async () => {
                    try {
                        const value = JSON.parse(message.value.toString());
                        await processMessage(value);
                        resolveOffset(message.offset);
                        await heartbeat();
                    } catch (error) {
                        console.error('Processing error:', error);
                    }
                })
            );
            
            await Promise.all(promises);
        }
    });
}
```

#### 2. **Consumer with Circuit Breaker**

```javascript
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
            console.log('Circuit breaker opened');
        }
    }
}

// Usage in consumer
const breaker = new CircuitBreaker(5, 60000);

await consumer.run({
    eachMessage: async ({ message }) => {
        try {
            await breaker.execute(async () => {
                const value = JSON.parse(message.value.toString());
                await externalServiceCall(value);
            });
        } catch (error) {
            if (error.message === 'Circuit breaker is OPEN') {
                // Pause consumption temporarily
                await pauseConsumption(['user-events']);
                setTimeout(() => resumeConsumption(['user-events']), 60000);
            }
        }
    }
});
```

---

## Kafka Stream Processing and Real-time Data Pipelines

Kafka Streams enables real-time processing of data as it flows through Kafka topics. While Node.js doesn't have an official Kafka Streams library, we can implement stream processing patterns using KafkaJS.

### Stream Processing Concepts

```javascript
// Simple stream processing: Read from one topic, transform, write to another
const { Kafka } = require('kafkajs');

class StreamProcessor {
    constructor(kafka, inputTopic, outputTopic, processorFn) {
        this.kafka = kafka;
        this.inputTopic = inputTopic;
        this.outputTopic = outputTopic;
        this.processorFn = processorFn;
        
        this.consumer = kafka.consumer({ 
            groupId: `stream-processor-${inputTopic}` 
        });
        this.producer = kafka.producer();
    }
    
    async start() {
        await this.consumer.connect();
        await this.producer.connect();
        
        await this.consumer.subscribe({ 
            topic: this.inputTopic,
            fromBeginning: false 
        });
        
        await this.consumer.run({
            eachMessage: async ({ message }) => {
                try {
                    // Parse input message
                    const inputValue = JSON.parse(message.value.toString());
                    
                    // Transform using processor function
                    const outputValue = await this.processorFn(inputValue);
                    
                    // Send to output topic
                    if (outputValue !== null && outputValue !== undefined) {
                        await this.producer.send({
                            topic: this.outputTopic,
                            messages: [{
                                key: message.key,
                                value: JSON.stringify(outputValue),
                                timestamp: Date.now().toString()
                            }]
                        });
                    }
                } catch (error) {
                    console.error('Stream processing error:', error);
                }
            }
        });
    }
    
    async stop() {
        await this.consumer.disconnect();
        await this.producer.disconnect();
    }
}

// Example: User registration stream processor
const kafka = new Kafka({
    clientId: 'stream-app',
    brokers: ['localhost:9092']
});

const registrationProcessor = new StreamProcessor(
    kafka,
    'raw-user-registrations',
    'processed-user-registrations',
    async (input) => {
        // Transform: enrich user data
        return {
            userId: input.userId,
            email: input.email,
            emailDomain: input.email.split('@')[1],
            registrationDate: new Date(input.timestamp).toISOString(),
            userType: input.email.endsWith('.edu') ? 'student' : 'regular',
            processed: true
        };
    }
);

await registrationProcessor.start();
```

### Windowed Aggregations

```javascript
// Tumbling window aggregation: Count events per time window
class TumblingWindowAggregator {
    constructor(kafka, inputTopic, outputTopic, windowSizeMs) {
        this.kafka = kafka;
        this.inputTopic = inputTopic;
        this.outputTopic = outputTopic;
        this.windowSizeMs = windowSizeMs;
        this.windowData = new Map();
        
        this.consumer = kafka.consumer({ 
            groupId: `window-aggregator-${inputTopic}` 
        });
        this.producer = kafka.producer();
        
        // Flush windows periodically
        this.flushInterval = setInterval(
            () => this.flushExpiredWindows(),
            windowSizeMs / 2
        );
    }
    
    getWindowKey(timestamp) {
        const windowStart = Math.floor(timestamp / this.windowSizeMs) * this.windowSizeMs;
        return windowStart;
    }
    
    async start() {
        await this.consumer.connect();
        await this.producer.connect();
        
        await this.consumer.subscribe({ topic: this.inputTopic });
        
        await this.consumer.run({
            eachMessage: async ({ message }) => {
                const value = JSON.parse(message.value.toString());
                const timestamp = parseInt(message.timestamp);
                
                const windowKey = this.getWindowKey(timestamp);
                
                if (!this.windowData.has(windowKey)) {
                    this.windowData.set(windowKey, {
                        windowStart: windowKey,
                        windowEnd: windowKey + this.windowSizeMs,
                        count: 0,
                        sum: 0,
                        events: []
                    });
                }
                
                const window = this.windowData.get(windowKey);
                window.count++;
                window.sum += value.amount || 0;
                window.events.push(value);
            }
        });
    }
    
    async flushExpiredWindows() {
        const now = Date.now();
        
        for (const [windowKey, windowData] of this.windowData.entries()) {
            // Flush windows that have ended
            if (windowData.windowEnd < now) {
                await this.producer.send({
                    topic: this.outputTopic,
                    messages: [{
                        key: windowKey.toString(),
                        value: JSON.stringify({
                            windowStart: new Date(windowData.windowStart).toISOString(),
                            windowEnd: new Date(windowData.windowEnd).toISOString(),
                            count: windowData.count,
                            sum: windowData.sum,
                            average: windowData.count > 0 ? windowData.sum / windowData.count : 0
                        })
                    }]
                });
                
                this.windowData.delete(windowKey);
            }
        }
    }
    
    async stop() {
        clearInterval(this.flushInterval);
        await this.flushExpiredWindows();
        await this.consumer.disconnect();
        await this.producer.disconnect();
    }
}

// Example: Aggregate order amounts per 5-minute window
const orderAggregator = new TumblingWindowAggregator(
    kafka,
    'order-events',
    'order-aggregates',
    5 * 60 * 1000  // 5 minutes
);

await orderAggregator.start();
```

### Stream Joins

```javascript
// Join two streams based on common key
class StreamJoiner {
    constructor(kafka, leftTopic, rightTopic, outputTopic, joinWindowMs) {
        this.kafka = kafka;
        this.leftTopic = leftTopic;
        this.rightTopic = rightTopic;
        this.outputTopic = outputTopic;
        this.joinWindowMs = joinWindowMs;
        
        // Store for pending joins
        this.leftStore = new Map();
        this.rightStore = new Map();
        
        this.leftConsumer = kafka.consumer({ groupId: 'joiner-left' });
        this.rightConsumer = kafka.consumer({ groupId: 'joiner-right' });
        this.producer = kafka.producer();
    }
    
    async start() {
        await this.leftConsumer.connect();
        await this.rightConsumer.connect();
        await this.producer.connect();
        
        // Consume from left topic
        await this.leftConsumer.subscribe({ topic: this.leftTopic });
        this.leftConsumer.run({
            eachMessage: async ({ message }) => {
                await this.processLeft(message);
            }
        });
        
        // Consume from right topic
        await this.rightConsumer.subscribe({ topic: this.rightTopic });
        this.rightConsumer.run({
            eachMessage: async ({ message }) => {
                await this.processRight(message);
            }
        });
    }
    
    async processLeft(message) {
        const key = message.key.toString();
        const value = JSON.parse(message.value.toString());
        const timestamp = parseInt(message.timestamp);
        
        // Check if matching right message exists
        if (this.rightStore.has(key)) {
            const rightData = this.rightStore.get(key);
            
            // Check if within join window
            if (Math.abs(timestamp - rightData.timestamp) <= this.joinWindowMs) {
                await this.emitJoin(key, value, rightData.value);
                this.rightStore.delete(key);
                return;
            }
        }
        
        // Store for future join
        this.leftStore.set(key, { value, timestamp });
        
        // Clean up old entries
        this.cleanupStore(this.leftStore, timestamp);
    }
    
    async processRight(message) {
        const key = message.key.toString();
        const value = JSON.parse(message.value.toString());
        const timestamp = parseInt(message.timestamp);
        
        // Check if matching left message exists
        if (this.leftStore.has(key)) {
            const leftData = this.leftStore.get(key);
            
            // Check if within join window
            if (Math.abs(timestamp - leftData.timestamp) <= this.joinWindowMs) {
                await this.emitJoin(key, leftData.value, value);
                this.leftStore.delete(key);
                return;
            }
        }
        
        // Store for future join
        this.rightStore.set(key, { value, timestamp });
        
        // Clean up old entries
        this.cleanupStore(this.rightStore, timestamp);
    }
    
    async emitJoin(key, leftValue, rightValue) {
        await this.producer.send({
            topic: this.outputTopic,
            messages: [{
                key: key,
                value: JSON.stringify({
                    ...leftValue,
                    ...rightValue,
                    joinedAt: Date.now()
                })
            }]
        });
    }
    
    cleanupStore(store, currentTimestamp) {
        for (const [key, data] of store.entries()) {
            if (currentTimestamp - data.timestamp > this.joinWindowMs) {
                store.delete(key);
            }
        }
    }
}

// Example: Join user profile updates with user activity
const userJoiner = new StreamJoiner(
    kafka,
    'user-profiles',      // Left: user profile data
    'user-activities',    // Right: user activity data
    'enriched-activities', // Output: joined data
    60000                 // 1-minute join window
);

await userJoiner.start();
```

### Real-time Analytics Pipeline

```javascript
// Complete pipeline: Ingest -> Filter -> Transform -> Aggregate -> Output
class AnalyticsPipeline {
    constructor(kafka) {
        this.kafka = kafka;
        this.stages = [];
    }
    
    // Add processing stage
    addStage(name, processFn) {
        this.stages.push({ name, processFn });
        return this;
    }
    
    // Build and start pipeline
    async start(inputTopic, outputTopic) {
        const consumer = this.kafka.consumer({ 
            groupId: `pipeline-${inputTopic}` 
        });
        const producer = this.kafka.producer();
        
        await consumer.connect();
        await producer.connect();
        await consumer.subscribe({ topic: inputTopic });
        
        await consumer.run({
            eachMessage: async ({ message }) => {
                try {
                    let data = JSON.parse(message.value.toString());
                    
                    // Process through each stage
                    for (const stage of this.stages) {
                        data = await stage.processFn(data);
                        
                        // If stage returns null, skip message
                        if (data === null || data === undefined) {
                            console.log(`Message filtered at stage: ${stage.name}`);
                            return;
                        }
                    }
                    
                    // Emit final result
                    await producer.send({
                        topic: outputTopic,
                        messages: [{
                            key: message.key,
                            value: JSON.stringify(data)
                        }]
                    });
                    
                } catch (error) {
                    console.error('Pipeline error:', error);
                }
            }
        });
        
        this.consumer = consumer;
        this.producer = producer;
    }
    
    async stop() {
        await this.consumer.disconnect();
        await this.producer.disconnect();
    }
}

// Example: E-commerce analytics pipeline
const kafka = new Kafka({
    clientId: 'analytics-pipeline',
    brokers: ['localhost:9092']
});

const pipeline = new AnalyticsPipeline(kafka)
    // Stage 1: Filter - only completed orders
    .addStage('filter', (order) => {
        return order.status === 'completed' ? order : null;
    })
    // Stage 2: Transform - calculate order metrics
    .addStage('transform', (order) => {
        return {
            orderId: order.id,
            customerId: order.customerId,
            totalAmount: order.items.reduce((sum, item) => 
                sum + item.price * item.quantity, 0),
            itemCount: order.items.length,
            timestamp: order.completedAt,
            category: order.items[0]?.category || 'other'
        };
    })
    // Stage 3: Enrich - add customer segment
    .addStage('enrich', async (order) => {
        const customerSegment = await getCustomerSegment(order.customerId);
        return {
            ...order,
            customerSegment
        };
    })
    // Stage 4: Classify - categorize order value
    .addStage('classify', (order) => {
        return {
            ...order,
            valueCategory: order.totalAmount > 1000 ? 'high' :
                          order.totalAmount > 100 ? 'medium' : 'low'
        };
    });

await pipeline.start('raw-orders', 'analytics-orders');

// Helper function
async function getCustomerSegment(customerId) {
    // Mock implementation - in real scenario, query database or cache
    return customerId % 3 === 0 ? 'premium' : 'regular';
}
```

---

## RabbitMQ Architecture and Message Exchange Patterns

RabbitMQ is a message broker that implements the Advanced Message Queuing Protocol (AMQP). It provides flexible routing and delivery guarantees.

### Core Components

#### 1. **Producer**
Application that sends messages to RabbitMQ.

#### 2. **Queue**
Buffer that stores messages until consumers are ready to process them.

#### 3. **Consumer**
Application that receives and processes messages from queues.

#### 4. **Exchange**
Routes messages to queues based on routing rules. This is the key difference from Kafka.

#### 5. **Binding**
Link between an exchange and a queue with optional routing key.

### RabbitMQ Architecture Diagram

```
Producer → Exchange → Binding → Queue → Consumer
            (Routes)  (Rules)  (Stores)  (Processes)
```

### Exchange Types

RabbitMQ supports four main exchange types:

#### 1. **Direct Exchange**
Routes messages to queues based on exact routing key match.

```
Producer --[routing_key: "error"]--> Direct Exchange
                                         |
                    ┌────────────────────┼────────────────────┐
                    |                    |                    |
             [key: "error"]        [key: "warning"]    [key: "info"]
                    |                    |                    |
                Error Queue          Warning Queue        Info Queue
```

#### 2. **Topic Exchange**
Routes based on pattern matching of routing keys.

```
Producer --[routing_key: "user.email.sent"]--> Topic Exchange
                                                     |
                        ┌────────────────────────────┼──────────────┐
                        |                            |              |
                   [pattern: "user.*"]      [pattern: "*.email.*"]  [pattern: "#"]
                        |                            |              |
                   User Queue                   Email Queue      All Queue
```

#### 3. **Fanout Exchange**
Broadcasts messages to all bound queues (pub/sub pattern).

```
Producer --> Fanout Exchange
                  |
        ┌─────────┼─────────┐
        |         |         |
    Queue 1   Queue 2   Queue 3
        |         |         |
    Consumer  Consumer  Consumer
    (all receive the same message)
```

#### 4. **Headers Exchange**
Routes based on message header attributes instead of routing key.

### Setting Up RabbitMQ with Node.js

```bash
npm install amqplib
```

### Basic RabbitMQ Producer

```javascript
// rabbitmq-producer.js
const amqp = require('amqplib');

class RabbitMQProducer {
    constructor(url = 'amqp://localhost') {
        this.url = url;
        this.connection = null;
        this.channel = null;
    }
    
    async connect() {
        try {
            // Create connection
            this.connection = await amqp.connect(this.url);
            
            // Create channel
            this.channel = await this.connection.createChannel();
            
            console.log('Connected to RabbitMQ');
            
            // Handle connection errors
            this.connection.on('error', (err) => {
                console.error('Connection error:', err);
            });
            
            this.connection.on('close', () => {
                console.log('Connection closed');
            });
            
        } catch (error) {
            console.error('Failed to connect:', error);
            throw error;
        }
    }
    
    // Send to direct exchange
    async sendDirect(exchange, routingKey, message, options = {}) {
        try {
            // Ensure exchange exists
            await this.channel.assertExchange(exchange, 'direct', {
                durable: true  // Survive broker restart
            });
            
            // Publish message
            const published = this.channel.publish(
                exchange,
                routingKey,
                Buffer.from(JSON.stringify(message)),
                {
                    persistent: true,  // Message survives broker restart
                    contentType: 'application/json',
                    timestamp: Date.now(),
                    ...options
                }
            );
            
            if (published) {
                console.log(`Message sent to ${exchange} with key ${routingKey}`);
            } else {
                console.warn('Message not published - channel buffer full');
            }
            
        } catch (error) {
            console.error('Error sending message:', error);
            throw error;
        }
    }
    
    // Send to topic exchange
    async sendTopic(exchange, routingKey, message) {
        await this.channel.assertExchange(exchange, 'topic', {
            durable: true
        });
        
        this.channel.publish(
            exchange,
            routingKey,
            Buffer.from(JSON.stringify(message)),
            { persistent: true }
        );
        
        console.log(`Topic message sent: ${routingKey}`);
    }
    
    // Send to fanout exchange (broadcast)
    async broadcast(exchange, message) {
        await this.channel.assertExchange(exchange, 'fanout', {
            durable: true
        });
        
        this.channel.publish(
            exchange,
            '',  // Routing key ignored for fanout
            Buffer.from(JSON.stringify(message)),
            { persistent: true }
        );
        
        console.log('Broadcast message sent');
    }
    
    // Send to queue directly (default exchange)
    async sendToQueue(queue, message, options = {}) {
        // Ensure queue exists
        await this.channel.assertQueue(queue, {
            durable: true,        // Queue survives broker restart
            maxLength: 10000,     // Maximum queue size
            messageTtl: 86400000  // Message TTL: 24 hours
        });
        
        const sent = this.channel.sendToQueue(
            queue,
            Buffer.from(JSON.stringify(message)),
            {
                persistent: true,
                ...options
            }
        );
        
        if (sent) {
            console.log(`Message sent to queue: ${queue}`);
        }
        
        return sent;
    }
    
    // Send with confirmation
    async sendWithConfirm(queue, message) {
        // Enable publisher confirms
        await this.channel.confirmChannel();
        
        return new Promise((resolve, reject) => {
            this.channel.sendToQueue(
                queue,
                Buffer.from(JSON.stringify(message)),
                { persistent: true },
                (err) => {
                    if (err) {
                        reject(err);
                    } else {
                        console.log('Message confirmed by broker');
                        resolve();
                    }
                }
            );
        });
    }
    
    async close() {
        if (this.channel) {
            await this.channel.close();
        }
        if (this.connection) {
            await this.connection.close();
        }
        console.log('RabbitMQ connection closed');
    }
}

// Example usage
async function main() {
    const producer = new RabbitMQProducer();
    await producer.connect();
    
    // Direct exchange example
    await producer.sendDirect('logs_exchange', 'error', {
        level: 'error',
        message: 'Database connection failed',
        timestamp: Date.now()
    });
    
    // Topic exchange example
    await producer.sendTopic('events_exchange', 'user.registration.completed', {
        userId: 123,
        email: 'user@example.com'
    });
    
    // Fanout exchange example
    await producer.broadcast('notifications_exchange', {
        type: 'system-maintenance',
        message: 'System will be down for maintenance'
    });
    
    // Direct queue example
    await producer.sendToQueue('tasks', {
        taskId: 1,
        type: 'email',
        data: { to: 'user@example.com', subject: 'Welcome' }
    });
}

process.on('SIGINT', async () => {
    await producer.close();
    process.exit(0);
});

module.exports = RabbitMQProducer;
```

### Basic RabbitMQ Consumer

```javascript
// rabbitmq-consumer.js
const amqp = require('amqplib');

class RabbitMQConsumer {
    constructor(url = 'amqp://localhost') {
        this.url = url;
        this.connection = null;
        this.channel = null;
    }
    
    async connect() {
        this.connection = await amqp.connect(this.url);
        this.channel = await this.connection.createChannel();
        
        // Set prefetch count - number of unacked messages per consumer
        await this.channel.prefetch(10);
        
        console.log('Consumer connected to RabbitMQ');
    }
    
    // Consume from queue
    async consumeQueue(queue, handler, options = {}) {
        // Ensure queue exists
        await this.channel.assertQueue(queue, {
            durable: true
        });
        
        console.log(`Waiting for messages in queue: ${queue}`);
        
        await this.channel.consume(
            queue,
            async (msg) => {
                if (msg !== null) {
                    try {
                        const content = JSON.parse(msg.content.toString());
                        
                        console.log('Received message:', content);
                        
                        // Process message
                        await handler(content, msg);
                        
                        // Acknowledge successful processing
                        this.channel.ack(msg);
                        
                    } catch (error) {
                        console.error('Error processing message:', error);
                        
                        // Negative acknowledgment
                        // requeue: false - send to DLX if configured
                        this.channel.nack(msg, false, false);
                    }
                }
            },
            {
                noAck: false,  // Manual acknowledgment
                ...options
            }
        );
    }
    
    // Consume from direct exchange
    async consumeDirect(exchange, routingKeys, handler) {
        await this.channel.assertExchange(exchange, 'direct', {
            durable: true
        });
        
        // Create exclusive queue for this consumer
        const q = await this.channel.assertQueue('', {
            exclusive: true  // Deleted when consumer disconnects
        });
        
        // Bind queue to exchange with routing keys
        for (const key of routingKeys) {
            await this.channel.bindQueue(q.queue, exchange, key);
            console.log(`Bound to ${exchange} with key: ${key}`);
        }
        
        await this.channel.consume(q.queue, async (msg) => {
            if (msg) {
                const content = JSON.parse(msg.content.toString());
                await handler(content, msg.fields.routingKey);
                this.channel.ack(msg);
            }
        });
    }
    
    // Consume from topic exchange
    async consumeTopic(exchange, patterns, handler) {
        await this.channel.assertExchange(exchange, 'topic', {
            durable: true
        });
        
        const q = await this.channel.assertQueue('', { exclusive: true });
        
        // Bind with topic patterns
        for (const pattern of patterns) {
            await this.channel.bindQueue(q.queue, exchange, pattern);
            console.log(`Subscribed to pattern: ${pattern}`);
        }
        
        await this.channel.consume(q.queue, async (msg) => {
            if (msg) {
                const content = JSON.parse(msg.content.toString());
                await handler(content, msg.fields.routingKey);
                this.channel.ack(msg);
            }
        });
    }
    
    // Consume from fanout exchange
    async consumeFanout(exchange, handler) {
        await this.channel.assertExchange(exchange, 'fanout', {
            durable: true
        });
        
        const q = await this.channel.assertQueue('', { exclusive: true });
        
        await this.channel.bindQueue(q.queue, exchange, '');
        console.log('Subscribed to fanout exchange');
        
        await this.channel.consume(q.queue, async (msg) => {
            if (msg) {
                const content = JSON.parse(msg.content.toString());
                await handler(content);
                this.channel.ack(msg);
            }
        });
    }
    
    async close() {
        if (this.channel) await this.channel.close();
        if (this.connection) await this.connection.close();
        console.log('Consumer disconnected');
    }
}

// Example usage
async function main() {
    const consumer = new RabbitMQConsumer();
    await consumer.connect();
    
    // Consume from direct queue
    await consumer.consumeQueue('tasks', async (message) => {
        console.log('Processing task:', message);
        await processTask(message);
    });
    
    // Consume from direct exchange
    await consumer.consumeDirect('logs_exchange', ['error', 'warning'], 
        async (message, routingKey) => {
            console.log(`Log [${routingKey}]:`, message);
        }
    );
    
    // Consume from topic exchange
    await consumer.consumeTopic('events_exchange', ['user.*', '*.email.*'],
        async (message, routingKey) => {
            console.log(`Event [${routingKey}]:`, message);
        }
    );
    
    // Consume from fanout exchange
    await consumer.consumeFanout('notifications_exchange',
        async (message) => {
            console.log('Notification:', message);
        }
    );
}

async function processTask(task) {
    // Simulate task processing
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log('Task completed:', task.taskId);
}

module.exports = RabbitMQConsumer;
```

---

## Implementing Publish-Subscribe Patterns with RabbitMQ

The publish-subscribe pattern allows multiple consumers to receive the same message. This is ideal for event-driven architectures.

### Simple Pub/Sub with Fanout Exchange

```javascript
// pubsub-publisher.js
const amqp = require('amqplib');

class Publisher {
    constructor() {
        this.exchangeName = 'events';
    }
    
    async connect() {
        this.connection = await amqp.connect('amqp://localhost');
        this.channel = await this.connection.createChannel();
        
        // Declare fanout exchange
        await this.channel.assertExchange(this.exchangeName, 'fanout', {
            durable: false  // Exchange won't survive broker restart
        });
    }
    
    publish(event) {
        const message = JSON.stringify(event);
        this.channel.publish(this.exchangeName, '', Buffer.from(message));
        console.log('Published:', event);
    }
    
    async close() {
        await this.channel.close();
        await this.connection.close();
    }
}

// Usage
const publisher = new Publisher();
await publisher.connect();

publisher.publish({ type: 'user-login', userId: 123 });
publisher.publish({ type: 'order-placed', orderId: 456 });
```

```javascript
// pubsub-subscriber.js
const amqp = require('amqplib');

class Subscriber {
    constructor(subscriberName) {
        this.subscriberName = subscriberName;
        this.exchangeName = 'events';
    }
    
    async connect() {
        this.connection = await amqp.connect('amqp://localhost');
        this.channel = await this.connection.createChannel();
        
        await this.channel.assertExchange(this.exchangeName, 'fanout', {
            durable: false
        });
        
        // Create exclusive queue for this subscriber
        const q = await this.channel.assertQueue('', {
            exclusive: true
        });
        
        // Bind queue to exchange
        await this.channel.bindQueue(q.queue, this.exchangeName, '');
        
        console.log(`${this.subscriberName} waiting for events...`);
        
        // Consume messages
        this.channel.consume(q.queue, (msg) => {
            const event = JSON.parse(msg.content.toString());
            console.log(`${this.subscriberName} received:`, event);
            this.handleEvent(event);
        }, { noAck: true });
    }
    
    handleEvent(event) {
        // Process event based on type
        if (event.type === 'user-login') {
            console.log(`${this.subscriberName}: User logged in`);
        } else if (event.type === 'order-placed') {
            console.log(`${this.subscriberName}: Order placed`);
        }
    }
}

// Create multiple subscribers
const emailService = new Subscriber('EmailService');
const analyticsService = new Subscriber('AnalyticsService');
const notificationService = new Subscriber('NotificationService');

await emailService.connect();
await analyticsService.connect();
await notificationService.connect();

// All three subscribers receive all events
```

### Topic-Based Pub/Sub

```javascript
// topic-publisher.js
class TopicPublisher {
    constructor() {
        this.exchangeName = 'topic_events';
    }
    
    async connect() {
        this.connection = await amqp.connect('amqp://localhost');
        this.channel = await this.connection.createChannel();
        
        await this.channel.assertExchange(this.exchangeName, 'topic', {
            durable: true
        });
    }
    
    publish(routingKey, event) {
        this.channel.publish(
            this.exchangeName,
            routingKey,
            Buffer.from(JSON.stringify(event)),
            { persistent: true }
        );
        console.log(`Published to ${routingKey}:`, event);
    }
}

// Usage
const publisher = new TopicPublisher();
await publisher.connect();

// Routing keys follow a pattern: entity.action.context
publisher.publish('user.created.email', { 
    userId: 123, 
    email: 'user@example.com' 
});

publisher.publish('user.created.analytics', { 
    userId: 123, 
    source: 'web' 
});

publisher.publish('order.placed.notification', { 
    orderId: 456, 
    userId: 123 
});

publisher.publish('order.placed.inventory', { 
    orderId: 456, 
    items: [1, 2, 3] 
});
```

```javascript
// topic-subscriber.js
class TopicSubscriber {
    constructor(serviceName, patterns) {
        this.serviceName = serviceName;
        this.patterns = patterns;
        this.exchangeName = 'topic_events';
    }
    
    async connect() {
        this.connection = await amqp.connect('amqp://localhost');
        this.channel = await this.connection.createChannel();
        
        await this.channel.assertExchange(this.exchangeName, 'topic', {
            durable: true
        });
        
        // Create queue for this service
        const q = await this.channel.assertQueue(`${this.serviceName}_queue`, {
            durable: true
        });
        
        // Bind with multiple patterns
        for (const pattern of this.patterns) {
            await this.channel.bindQueue(q.queue, this.exchangeName, pattern);
            console.log(`${this.serviceName} subscribed to: ${pattern}`);
        }
        
        // Consume messages
        await this.channel.consume(q.queue, async (msg) => {
            const event = JSON.parse(msg.content.toString());
            const routingKey = msg.fields.routingKey;
            
            console.log(`${this.serviceName} [${routingKey}]:`, event);
            
            await this.handleEvent(routingKey, event);
            this.channel.ack(msg);
        });
    }
    
    async handleEvent(routingKey, event) {
        // Service-specific event handling
        console.log(`${this.serviceName} processing ${routingKey}`);
    }
}

// Email service: subscribes to all email-related events
const emailService = new TopicSubscriber('EmailService', [
    '*.*.email'  // Matches: user.created.email, order.shipped.email, etc.
]);

// Analytics service: subscribes to all analytics events
const analyticsService = new TopicSubscriber('AnalyticsService', [
    '*.*.analytics',
    'user.*'     // Also interested in all user events
]);

// Inventory service: subscribes to order-related events
const inventoryService = new TopicSubscriber('InventoryService', [
    'order.*.inventory',
    'product.*.inventory'
]);

await emailService.connect();
await analyticsService.connect();
await inventoryService.connect();
```

### Advanced Pub/Sub Pattern: Event Sourcing

```javascript
// event-store.js
class EventStore {
    constructor() {
        this.exchangeName = 'event_store';
        this.allEventsQueue = 'all_events';
    }
    
    async connect() {
        this.connection = await amqp.connect('amqp://localhost');
        this.channel = await this.connection.createChannel();
        
        // Topic exchange for event sourcing
        await this.channel.assertExchange(this.exchangeName, 'topic', {
            durable: true
        });
        
        // Persistent queue for all events
        await this.channel.assertQueue(this.allEventsQueue, {
            durable: true,
            arguments: {
                'x-max-length': 1000000  // Store up to 1M events
            }
        });
        
        // Bind to capture all events
        await this.channel.bindQueue(this.allEventsQueue, this.exchangeName, '#');
    }
    
    // Append event to store
    async appendEvent(aggregateType, aggregateId, eventType, eventData) {
        const event = {
            aggregateType,
            aggregateId,
            eventType,
            eventData,
            timestamp: Date.now(),
            eventId: generateUUID()
        };
        
        const routingKey = `${aggregateType}.${aggregateId}.${eventType}`;
        
        this.channel.publish(
            this.exchangeName,
            routingKey,
            Buffer.from(JSON.stringify(event)),
            { persistent: true }
        );
        
        return event.eventId;
    }
    
    // Subscribe to specific aggregate's events
    async subscribeToAggregate(aggregateType, aggregateId, handler) {
        const pattern = `${aggregateType}.${aggregateId}.#`;
        
        const q = await this.channel.assertQueue('', { exclusive: true });
        await this.channel.bindQueue(q.queue, this.exchangeName, pattern);
        
        await this.channel.consume(q.queue, async (msg) => {
            const event = JSON.parse(msg.content.toString());
            await handler(event);
            this.channel.ack(msg);
        });
    }
}

// Usage: Order aggregate with event sourcing
const eventStore = new EventStore();
await eventStore.connect();

// Append events
await eventStore.appendEvent('Order', 'order-123', 'OrderCreated', {
    orderId: 'order-123',
    customerId: 'customer-456',
    items: [{ productId: 1, quantity: 2 }]
});

await eventStore.appendEvent('Order', 'order-123', 'OrderPaid', {
    orderId: 'order-123',
    amount: 200,
    paymentMethod: 'credit-card'
});

await eventStore.appendEvent('Order', 'order-123', 'OrderShipped', {
    orderId: 'order-123',
    trackingNumber: 'TRACK123'
});

// Subscribe to order events
await eventStore.subscribeToAggregate('Order', 'order-123', (event) => {
    console.log('Order event:', event.eventType, event.eventData);
});

function generateUUID() {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, (c) => {
        const r = Math.random() * 16 | 0;
        const v = c === 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
    });
}
```

---

## Message Acknowledgments and Delivery Guarantees

Both Kafka and RabbitMQ provide mechanisms to ensure reliable message delivery. Understanding acknowledgments is crucial for building robust systems.

### Kafka Acknowledgments

#### Producer Acknowledgments (acks)

```javascript
// acks = 0: Fire and forget (no acknowledgment)
const producer1 = kafka.producer({
    acks: 0  // Fastest but no guarantee
});

await producer1.send({
    topic: 'logs',
    messages: [{ value: 'log message' }],
    acks: 0  // Producer doesn't wait for any acknowledgment
});
// Pros: Highest throughput
// Cons: Messages may be lost if broker fails

// acks = 1: Leader acknowledgment
const producer2 = kafka.producer({
    acks: 1  // Default, balanced approach
});

await producer2.send({
    topic: 'events',
    messages: [{ value: 'event data' }],
    acks: 1  // Wait for leader broker acknowledgment
});
// Pros: Good balance between performance and reliability
// Cons: Data lost if leader fails before replication

// acks = -1 or 'all': All replicas acknowledgment
const producer3 = kafka.producer({
    acks: -1,  // Strongest guarantee
    idempotent: true  // Exactly-once semantics
});

await producer3.send({
    topic: 'transactions',
    messages: [{ value: 'transaction data' }],
    acks: -1  // Wait for all in-sync replicas
});
// Pros: Strongest durability guarantee
// Cons: Highest latency
```

#### Consumer Acknowledgments (Offset Management)

```javascript
// Auto-commit offsets (default)
const consumer1 = kafka.consumer({
    groupId: 'auto-commit-group',
    autoCommit: true,
    autoCommitInterval: 5000  // Commit every 5 seconds
});

await consumer1.run({
    eachMessage: async ({ message }) => {
        console.log('Processing:', message.value.toString());
        // Offset auto-committed periodically
    }
});
// Pros: Simple, automatic
// Cons: Risk of message loss or duplication on failure

// Manual commit per message
const consumer2 = kafka.consumer({
    groupId: 'manual-commit-group',
    autoCommit: false  // Disable auto-commit
});

await consumer2.run({
    eachMessage: async ({ message, topic, partition, heartbeat }) => {
        try {
            // Process message
            await processMessage(message.value.toString());
            
            // Manually commit offset after successful processing
            await consumer2.commitOffsets([{
                topic,
                partition,
                offset: (parseInt(message.offset) + 1).toString()
            }]);
            
            await heartbeat();  // Keep consumer alive
            
        } catch (error) {
            console.error('Processing failed:', error);
            // Don't commit - message will be reprocessed
        }
    }
});
// Pros: Precise control, no message loss
// Cons: More complex, potential for duplicates on retry

// Batch commit for better performance
const consumer3 = kafka.consumer({
    groupId: 'batch-commit-group',
    autoCommit: false
});

await consumer3.run({
    eachBatchAutoResolve: false,
    eachBatch: async ({ batch, resolveOffset, commitOffsetsIfNecessary }) => {
        for (const message of batch.messages) {
            try {
                await processMessage(message.value.toString());
                // Mark offset as resolved
                resolveOffset(message.offset);
            } catch (error) {
                console.error('Failed:', error);
                break;  // Stop processing batch on error
            }
        }
        
        // Commit all resolved offsets in batch
        await commitOffsetsIfNecessary();
    }
});
```

### RabbitMQ Acknowledgments

#### Producer Confirmations

```javascript
// Publisher confirms for reliability
const amqp = require('amqplib');

class ReliablePublisher {
    async connect() {
        this.connection = await amqp.connect('amqp://localhost');
        
        // Create confirm channel instead of regular channel
        this.channel = await this.connection.createConfirmChannel();
        
        await this.channel.assertQueue('tasks', { durable: true });
    }
    
    // Send with confirmation
    async sendWithConfirm(queue, message) {
        return new Promise((resolve, reject) => {
            this.channel.sendToQueue(
                queue,
                Buffer.from(JSON.stringify(message)),
                { persistent: true },
                (err, ok) => {
                    if (err) {
                        console.error('Message rejected by broker:', err);
                        reject(err);
                    } else {
                        console.log('Message confirmed by broker');
                        resolve(ok);
    }
                }
            );
        });
    }
    
    // Wait for all outstanding confirmations
    async waitForConfirms() {
        return this.channel.waitForConfirms();
    }
}

// Usage
const publisher = new ReliablePublisher();
await publisher.connect();

try {
    await publisher.sendWithConfirm('tasks', { taskId: 1, data: 'important task' });
    console.log('Message successfully delivered and confirmed');
} catch (error) {
    console.error('Message delivery failed:', error);
    // Implement retry logic
}
```

#### Consumer Acknowledgments

```javascript
class ReliableConsumer {
    async connect() {
        this.connection = await amqp.connect('amqp://localhost');
        this.channel = await this.connection.createChannel();
        
        // Limit unacknowledged messages
        await this.channel.prefetch(10);
        
        await this.channel.assertQueue('tasks', { durable: true });
    }
    
    // Manual acknowledgment
    async consume() {
        await this.channel.consume('tasks', async (msg) => {
            if (msg) {
                try {
                    const message = JSON.parse(msg.content.toString());
                    console.log('Processing:', message);
                    
                    // Simulate processing
                    await processTask(message);
                    
                    // Acknowledge successful processing
                    this.channel.ack(msg);
                    console.log('Message acknowledged');
                    
                } catch (error) {
                    console.error('Processing failed:', error);
                    
                    // Negative acknowledgment with requeue
                    this.channel.nack(msg, false, true);
                    // Parameters: msg, allUpTo, requeue
                }
            }
        }, {
            noAck: false  // Manual acknowledgment mode
        });
    }
    
    // Acknowledge multiple messages
    async consumeWithBatchAck() {
        let messageCount = 0;
        let lastMsg = null;
        
        await this.channel.consume('tasks', async (msg) => {
            if (msg) {
                try {
                    await processTask(JSON.parse(msg.content.toString()));
                    messageCount++;
                    lastMsg = msg;
                    
                    // Acknowledge every 10 messages
                    if (messageCount >= 10) {
                        this.channel.ack(lastMsg, true);  // Ack all up to this message
                        messageCount = 0;
                    }
                    
                } catch (error) {
                    // Reject single message
                    this.channel.nack(msg, false, false);
                }
            }
        }, { noAck: false });
    }
    
    // Reject without requeue (send to DLX)
    async consumeWithDLX() {
        await this.channel.consume('tasks', async (msg) => {
            if (msg) {
                try {
                    await processTask(JSON.parse(msg.content.toString()));
                    this.channel.ack(msg);
                } catch (error) {
                    console.error('Permanent failure, sending to DLX');
                    
                    // Reject without requeue - goes to dead letter exchange
                    this.channel.nack(msg, false, false);
                }
            }
        }, { noAck: false });
    }
}

async function processTask(task) {
    // Simulate task processing
    await new Promise(resolve => setTimeout(resolve, 100));
    
    // Simulate occasional failures
    if (Math.random() < 0.1) {
        throw new Error('Random processing error');
    }
}
```

### Delivery Guarantees Comparison

#### At-Most-Once Delivery
Messages may be lost but never duplicated.

```javascript
// Kafka: acks = 0, autoCommit = true
const producer = kafka.producer({ acks: 0 });
const consumer = kafka.consumer({ autoCommit: true });

// RabbitMQ: noAck = true
await channel.consume('queue', (msg) => {
    processMessage(msg);
    // No acknowledgment - message might be lost on crash
}, { noAck: true });
```

#### At-Least-Once Delivery
Messages never lost but may be duplicated. **Most common pattern.**

```javascript
// Kafka: acks = -1, manual commit after processing
const producer = kafka.producer({ acks: -1 });

await consumer.run({
    eachMessage: async ({ message, topic, partition }) => {
        await processMessage(message);  // May be processed multiple times
        await consumer.commitOffsets([{ topic, partition, offset: message.offset }]);
    }
});

// RabbitMQ: Manual ack after processing
await channel.consume('queue', async (msg) => {
    await processMessage(msg);  // May be processed multiple times
    channel.ack(msg);
}, { noAck: false });
```

#### Exactly-Once Delivery
Messages processed exactly once (most complex).

```javascript
// Kafka: Idempotent producer + transactional processing
const producer = kafka.producer({
    idempotent: true,
    transactionalId: 'my-transactional-producer'
});

const consumer = kafka.consumer({
    groupId: 'exactly-once-group',
    autoCommit: false,
    isolation: 'read_committed'  // Only read committed messages
});

await producer.connect();

// Transactional send
const transaction = await producer.transaction();
try {
    await transaction.send({
        topic: 'output',
        messages: [{ value: 'processed data' }]
    });
    
    // Commit transaction atomically with consumer offset
    await transaction.sendOffsets({
        consumerGroupId: 'exactly-once-group',
        topics: [{ topic: 'input', partitions: [{ partition: 0, offset: '10' }] }]
    });
    
    await transaction.commit();
} catch (error) {
    await transaction.abort();
}

// RabbitMQ: Requires idempotency in application logic
// Use message IDs and deduplication
const processedIds = new Set();

await channel.consume('queue', async (msg) => {
    const messageId = msg.properties.messageId;
    
    // Check if already processed
    if (processedIds.has(messageId)) {
        channel.ack(msg);  // Already processed, just ack
        return;
    }
    
    await processMessage(msg);
    processedIds.add(messageId);
    channel.ack(msg);
}, { noAck: false });
```

---

## Dead Letter Queues and Message Retry Mechanisms

Dead Letter Queues (DLQs) store messages that cannot be processed successfully, enabling graceful error handling and retry mechanisms.

### RabbitMQ Dead Letter Exchange

```javascript
// Setup DLX and DLQ
const amqp = require('amqplib');

class DLXSetup {
    async setup() {
        const connection = await amqp.connect('amqp://localhost');
        const channel = await connection.createChannel();
        
        // 1. Create Dead Letter Exchange
        await channel.assertExchange('dlx', 'direct', { durable: true });
        
        // 2. Create Dead Letter Queue
        await channel.assertQueue('dead_letter_queue', {
            durable: true
        });
        
        // 3. Bind DLQ to DLX
        await channel.bindQueue('dead_letter_queue', 'dlx', 'failed');
        
        // 4. Create main queue with DLX configuration
        await channel.assertQueue('main_queue', {
            durable: true,
            arguments: {
                'x-dead-letter-exchange': 'dlx',
                'x-dead-letter-routing-key': 'failed',
                'x-message-ttl': 300000,  // 5 minutes TTL
                'x-max-length': 10000      // Max queue length
            }
        });
        
        return { connection, channel };
    }
}

// Producer sends to main queue
async function sendMessage(channel, message) {
    channel.sendToQueue(
        'main_queue',
        Buffer.from(JSON.stringify(message)),
        {
            persistent: true,
            messageId: generateUUID()
        }
    );
}

// Consumer with retry logic
class RetryConsumer {
    constructor(channel) {
        this.channel = channel;
        this.maxRetries = 3;
    }
    
    async consume() {
        await this.channel.consume('main_queue', async (msg) => {
            if (msg) {
                try {
                    const message = JSON.parse(msg.content.toString());
                    const retryCount = (msg.properties.headers?.['x-retry-count'] || 0);
                    
                    console.log(`Processing (attempt ${retryCount + 1}):`, message);
                    
                    // Simulate processing
                    await processMessage(message);
                    
                    // Success - acknowledge
                    this.channel.ack(msg);
                    
                } catch (error) {
                    console.error('Processing failed:', error);
                    
                    const retryCount = (msg.properties.headers?.['x-retry-count'] || 0);
                    
                    if (retryCount < this.maxRetries) {
                        // Retry - send back to queue with incremented retry count
                        await this.retry(msg, retryCount + 1);
                        this.channel.ack(msg);  // Remove from original queue
                    } else {
                        // Max retries exceeded - send to DLQ
                        console.log('Max retries exceeded, sending to DLQ');
                        this.channel.nack(msg, false, false);  // Goes to DLX
                    }
                }
            }
        }, { noAck: false });
    }
    
    async retry(msg, retryCount) {
        const delay = Math.min(1000 * Math.pow(2, retryCount), 30000);  // Exponential backoff
        
        console.log(`Retrying in ${delay}ms (attempt ${retryCount})`);
        
        // Send to delayed retry queue
        this.channel.sendToQueue(
            'main_queue',
            msg.content,
            {
                persistent: true,
                headers: {
                    'x-retry-count': retryCount,
                    'x-first-death-reason': msg.properties.headers?.['x-first-death-reason'] || 'processing-error'
                },
                expiration: delay.toString()  // Delay before reprocessing
            }
        );
    }
}

// Monitor DLQ and handle failed messages
class DLQMonitor {
    constructor(channel) {
        this.channel = channel;
    }
    
    async monitor() {
        await this.channel.consume('dead_letter_queue', async (msg) => {
            if (msg) {
                const message = JSON.parse(msg.content.toString());
                const headers = msg.properties.headers;
                
                console.log('Message in DLQ:', {
                    message,
                    retryCount: headers['x-retry-count'],
                    deathReason: headers['x-first-death-reason'],
                    originalQueue: headers['x-first-death-queue']
                });
                
                // Log to monitoring system
                await logToMonitoring(message, headers);
                
                // Send alert
                await sendAlert(`Message failed after ${headers['x-retry-count']} retries`);
                
                this.channel.ack(msg);
            }
        }, { noAck: false });
    }
}

async function processMessage(message) {
    // Simulate processing with occasional failures
    if (Math.random() < 0.7) {
        throw new Error('Simulated processing error');
    }
}

async function logToMonitoring(message, headers) {
    console.log('Logged to monitoring system');
}

async function sendAlert(message) {
    console.log('Alert sent:', message);
}
```

### Kafka Retry Pattern with Multiple Topics

```javascript
// Kafka doesn't have native DLQ, but we can implement retry pattern
class KafkaRetryHandler {
    constructor(kafka) {
        this.kafka = kafka;
        this.producer = kafka.producer();
        this.maxRetries = 3;
        this.retryDelays = [1000, 5000, 15000];  // Delays in ms
    }
    
    async setup() {
        await this.producer.connect();
        
        const admin = this.kafka.admin();
        await admin.connect();
        
        // Create retry topics
        await admin.createTopics({
            topics: [
                { topic: 'orders', numPartitions: 3 },
                { topic: 'orders.retry.1', numPartitions: 3 },
                { topic: 'orders.retry.2', numPartitions: 3 },
                { topic: 'orders.retry.3', numPartitions: 3 },
                { topic: 'orders.dead-letter', numPartitions: 1 }
            ]
        });
        
        await admin.disconnect();
    }
    
    // Main topic consumer
    async consumeMain() {
        const consumer = this.kafka.consumer({ groupId: 'order-processor' });
        await consumer.connect();
        await consumer.subscribe({ topic: 'orders' });
        
        await consumer.run({
            eachMessage: async ({ message }) => {
                await this.processWithRetry(message, 0);
            }
        });
    }
    
    // Retry topic consumers
    async consumeRetry(retryLevel) {
        const consumer = this.kafka.consumer({ 
            groupId: `order-retry-${retryLevel}` 
        });
        await consumer.connect();
        await consumer.subscribe({ topic: `orders.retry.${retryLevel}` });
        
        await consumer.run({
            eachMessage: async ({ message }) => {
                // Wait for retry delay
                const metadata = JSON.parse(message.headers?.metadata?.toString() || '{}');
                const scheduledTime = parseInt(metadata.scheduledTime || '0');
                const now = Date.now();
                
                if (now < scheduledTime) {
                    // Not ready yet, wait
                    await new Promise(resolve => setTimeout(resolve, scheduledTime - now));
                }
                
                await this.processWithRetry(message, retryLevel);
            }
        });
    }
    
    async processWithRetry(message, retryLevel) {
        try {
            const value = JSON.parse(message.value.toString());
            console.log(`Processing (retry ${retryLevel}):`, value);
            
            // Simulate processing
            await processOrder(value);
            
            console.log('Processing successful');
            
        } catch (error) {
            console.error(`Processing failed (retry ${retryLevel}):`, error.message);
            
            if (retryLevel < this.maxRetries) {
                // Send to retry topic
                await this.sendToRetry(message, retryLevel + 1);
            } else {
                // Send to dead letter topic
                await this.sendToDeadLetter(message, error);
            }
        }
    }
    
    async sendToRetry(message, retryLevel) {
        const delay = this.retryDelays[retryLevel - 1];
        const scheduledTime = Date.now() + delay;
        
        await this.producer.send({
            topic: `orders.retry.${retryLevel}`,
            messages: [{
                key: message.key,
                value: message.value,
                headers: {
                    metadata: JSON.stringify({
                        retryLevel,
                        scheduledTime,
                        originalTopic: 'orders'
                    })
                }
            }]
        });
        
        console.log(`Scheduled retry ${retryLevel} at ${new Date(scheduledTime)}`);
    }
    
    async sendToDeadLetter(message, error) {
        await this.producer.send({
            topic: 'orders.dead-letter',
            messages: [{
                key: message.key,
                value: message.value,
                headers: {
                    error: error.message,
                    failedAt: Date.now().toString(),
                    originalTopic: 'orders'
                }
            }]
        });
        
        console.log('Message sent to dead letter queue');
    }
}

// Dead letter queue monitor
async function monitorDeadLetter(kafka) {
    const consumer = kafka.consumer({ groupId: 'dlq-monitor' });
    await consumer.connect();
    await consumer.subscribe({ topic: 'orders.dead-letter' });
    
    await consumer.run({
        eachMessage: async ({ message }) => {
            const value = JSON.parse(message.value.toString());
            const error = message.headers?.error?.toString();
            const failedAt = message.headers?.failedAt?.toString();
            
            console.log('Failed message:', {
                data: value,
                error,
                failedAt: new Date(parseInt(failedAt))
            });
            
            // Send to monitoring/alerting system
            await alertOps(value, error);
        }
    });
}

async function processOrder(order) {
    // Simulate processing with failures
    if (Math.random() < 0.6) {
        throw new Error('Order processing failed');
    }
}

async function alertOps(data, error) {
    console.log('ALERT: Message processing failed permanently:', error);
}
```

---

## Scaling Message Queues and Handling High Throughput

### Kafka Scaling Strategies

#### 1. **Partition-based Parallelism**

```javascript
// Create topic with multiple partitions for parallel processing
const admin = kafka.admin();
await admin.connect();

await admin.createTopics({
    topics: [{
        topic: 'high-throughput-events',
        numPartitions: 12,  // More partitions = more parallelism
        replicationFactor: 3,
        configEntries: [
            { name: 'compression.type', value: 'snappy' },
            { name: 'max.message.bytes', value: '1048576' }  // 1MB
        ]
    }]
});

// Producer: Distribute messages across partitions
const producer = kafka.producer({
    allowAutoTopicCreation: false,
    compression: 1,  // GZIP compression
    // Batch settings for throughput
    batch: {
        size: 16384,        // 16KB batch size
        maxBytes: 1048576   // 1MB max batch
    },
    linger: 10  // Wait up to 10ms to batch messages
});

await producer.connect();

// Send with key for partition assignment
for (let i = 0; i < 10000; i++) {
    await producer.send({
        topic: 'high-throughput-events',
        messages: [{
            key: `user-${i % 1000}`,  // Consistent hashing by user ID
            value: JSON.stringify({ eventId: i, data: '...' })
        }]
    });
}
```

#### 2. **Consumer Group Scaling**

```javascript
// Scale horizontally with consumer groups
// Each consumer in group processes different partitions

class ScalableConsumer {
    constructor(kafka, consumerId) {
        this.kafka = kafka;
        this.consumerId = consumerId;
        this.consumer = kafka.consumer({
            groupId: 'scalable-processor-group',
            sessionTimeout: 30000,
            heartbeatInterval: 3000,
            maxBytesPerPartition: 1048576,  // 1MB per partition
            maxWaitTimeInMs: 1000
        });
    }
    
    async start() {
        await this.consumer.connect();
        await this.consumer.subscribe({ 
            topic: 'high-throughput-events',
            fromBeginning: false 
        });
        
        console.log(`Consumer ${this.consumerId} started`);
        
        await this.consumer.run({
            // Process messages in parallel within batch
            eachBatchAutoResolve: false,
            eachBatch: async ({ 
                batch, 
                resolveOffset, 
                heartbeat, 
                commitOffsetsIfNecessary 
            }) => {
                console.log(`Consumer ${this.consumerId} processing partition ${batch.partition}`);
                
                // Process messages in parallel with concurrency limit
                const concurrency = 10;
                for (let i = 0; i < batch.messages.length; i += concurrency) {
                    const chunk = batch.messages.slice(i, i + concurrency);
                    
                    await Promise.all(chunk.map(async (message) => {
                        try {
                            await this.processMessage(message);
                            resolveOffset(message.offset);
                        } catch (error) {
                            console.error('Processing error:', error);
                        }
                    }));
                    
                    await heartbeat();
                }
                
                await commitOffsetsIfNecessary();
            }
        });
    }
    
    async processMessage(message) {
        const value = JSON.parse(message.value.toString());
        // Simulate processing
        await new Promise(resolve => setTimeout(resolve, 10));
    }
}

// Launch multiple consumers
const consumers = [];
for (let i = 0; i < 12; i++) {  // Match number of partitions
    const consumer = new ScalableConsumer(kafka, i);
    consumers.push(consumer.start());
}

await Promise.all(consumers);
// Kafka automatically distributes partitions among consumers
// Adding more consumers (up to partition count) increases throughput
```

#### 3. **Producer Batching and Compression**

```javascript
// Optimize producer for high throughput
const highThroughputProducer = kafka.producer({
    // Batching configuration
    batch: {
        size: 65536,        // 64KB batch size
        maxBytes: 1048576   // 1MB maximum batch
    },
    linger: 50,  // Wait up to 50ms to batch more messages
    
    // Compression
    compression: 2,  // Snappy compression (fast)
    
    // Parallelism
    maxInFlightRequests: 5,
    
    // Reliability
    idempotent: true,
    acks: 1  // Balance between throughput and reliability
});

await highThroughputProducer.connect();

// Batch send for maximum throughput
async function sendBatch(messages) {
    const batches = [];
    const batchSize = 1000;
    
    for (let i = 0; i < messages.length; i += batchSize) {
        batches.push(messages.slice(i, i + batchSize));
    }
    
    await Promise.all(batches.map(batch => 
        highThroughputProducer.send({
            topic: 'high-throughput-events',
            messages: batch.map(msg => ({
                value: JSON.stringify(msg)
            }))
        })
    ));
}

// Send 100,000 messages
const messages = Array.from({ length: 100000 }, (_, i) => ({
    id: i,
    timestamp: Date.now(),
    data: 'event data'
}));

const startTime = Date.now();
await sendBatch(messages);
const duration = Date.now() - startTime;

console.log(`Sent ${messages.length} messages in ${duration}ms`);
console.log(`Throughput: ${Math.round(messages.length / (duration / 1000))} msg/sec`);
```

### RabbitMQ Scaling Strategies

#### 1. **Multiple Queues with Consistent Hashing**

```javascript
// Shard across multiple queues for parallel processing
class ShardedQueue {
    constructor(connection, numShards = 10) {
        this.connection = connection;
        this.numShards = numShards;
        this.queuePrefix = 'sharded_queue';
    }
    
    async setup() {
        this.channel = await this.connection.createChannel();
        
        // Create multiple queues
        for (let i = 0; i < this.numShards; i++) {
            await this.channel.assertQueue(`${this.queuePrefix}_${i}`, {
                durable: true,
                arguments: {
                    'x-max-length': 100000  // Limit per shard
                }
            });
        }
    }
    
    // Hash-based routing
    getShardId(key) {
        let hash = 0;
        for (let i = 0; i < key.length; i++) {
            hash = ((hash << 5) - hash) + key.charCodeAt(i);
            hash = hash & hash;  // Convert to 32-bit integer
        }
        return Math.abs(hash) % this.numShards;
    }
    
    async publish(key, message) {
        const shardId = this.getShardId(key);
        const queue = `${this.queuePrefix}_${shardId}`;
        
        this.channel.sendToQueue(
            queue,
            Buffer.from(JSON.stringify(message)),
            { persistent: true }
        );
    }
    
    async consume(shardId, handler) {
        const queue = `${this.queuePrefix}_${shardId}`;
        
        await this.channel.prefetch(100);  // Process 100 at a time
        
        await this.channel.consume(queue, async (msg) => {
            if (msg) {
                await handler(JSON.parse(msg.content.toString()));
                this.channel.ack(msg);
            }
        }, { noAck: false });
    }
}

// Usage
const connection = await amqp.connect('amqp://localhost');
const shardedQueue = new ShardedQueue(connection, 10);
await shardedQueue.setup();

// Publish to shards
for (let i = 0; i < 10000; i++) {
    await shardedQueue.publish(`user-${i}`, {
        userId: i,
        action: 'login'
    });
}

// Consume from each shard in parallel
for (let i = 0; i < 10; i++) {
    shardedQueue.consume(i, async (message) => {
        console.log(`Shard ${i} processing:`, message);
    });
}
```

#### 2. **Prefetch and Parallel Processing**

```javascript
class HighThroughputConsumer {
    constructor(connection, queueName, concurrency = 50) {
        this.connection = connection;
        this.queueName = queueName;
        this.concurrency = concurrency;
    }
    
    async start() {
        this.channel = await this.connection.createChannel();
        
        // Set high prefetch for batching
        await this.channel.prefetch(this.concurrency);
        
        await this.channel.assertQueue(this.queueName, { durable: true });
        
        // Consume with parallel processing
        await this.channel.consume(this.queueName, async (msg) => {
            if (msg) {
                // Don't await - process asynchronously
                this.processAsync(msg);
            }
        }, { noAck: false });
    }
    
    async processAsync(msg) {
        try {
            const message = JSON.parse(msg.content.toString());
            
            // Simulate processing
            await this.processMessage(message);
            
            this.channel.ack(msg);
        } catch (error) {
            console.error('Processing error:', error);
            this.channel.nack(msg, false, true);
        }
    }
    
    async processMessage(message) {
        // Fast processing simulation
        await new Promise(resolve => setTimeout(resolve, 10));
    }
}

const consumer = new HighThroughputConsumer(connection, 'high_volume_queue', 100);
await consumer.start();
```

#### 3. **Publisher Connection Pooling**

```javascript
// Connection pool for high-volume publishing
class PublisherPool {
    constructor(url, poolSize = 10) {
        this.url = url;
        this.poolSize = poolSize;
        this.channels = [];
        this.currentIndex = 0;
    }
    
    async initialize() {
        // Create multiple connections and channels
        for (let i = 0; i < this.poolSize; i++) {
            const connection = await amqp.connect(this.url);
            const channel = await connection.createConfirmChannel();
            this.channels.push(channel);
        }
        
        console.log(`Initialized ${this.poolSize} publisher channels`);
    }
    
    // Round-robin channel selection
    getChannel() {
        const channel = this.channels[this.currentIndex];
        this.currentIndex = (this.currentIndex + 1) % this.poolSize;
        return channel;
    }
    
    async publish(queue, message) {
        const channel = this.getChannel();
        
        return new Promise((resolve, reject) => {
            channel.sendToQueue(
                queue,
                Buffer.from(JSON.stringify(message)),
                { persistent: true },
                (err) => {
                    if (err) reject(err);
                    else resolve();
                }
            );
        });
    }
    
    async publishBatch(queue, messages) {
        const promises = messages.map(msg => this.publish(queue, msg));
        await Promise.all(promises);
    }
}

// Usage: High-throughput publishing
const pool = new PublisherPool('amqp://localhost', 10);
await pool.initialize();

const messages = Array.from({ length: 100000 }, (_, i) => ({
    id: i,
    data: `message ${i}`
}));

const startTime = Date.now();
await pool.publishBatch('high_volume_queue', messages);
const duration = Date.now() - startTime;

console.log(`Published ${messages.length} messages in ${duration}ms`);
console.log(`Throughput: ${Math.round(messages.length / (duration / 1000))} msg/sec`);
```

---

## Comparing Kafka vs RabbitMQ - Use Cases and Trade-offs

### Architecture Comparison

| Aspect | Kafka | RabbitMQ |
|--------|-------|----------|
| **Model** | Distributed commit log | Message broker |
| **Routing** | Topic-based, partition-based | Flexible exchange routing |
| **Message Ordering** | Per-partition ordering | Per-queue ordering |
| **Message Retention** | Time/size-based, persistent | Until consumed (or TTL) |
| **Replay** | Yes, consumers control offset | No, messages deleted after ack |
| **Throughput** | Very high (millions/sec) | High (tens of thousands/sec) |
| **Latency** | Low (milliseconds) | Very low (microseconds) |
| **Protocol** | Custom binary protocol | AMQP, MQTT, STOMP |

### When to Use Kafka

#### 1. **Event Streaming and Log Aggregation**
```javascript
// Kafka excels at high-volume event streaming
const eventLogger = kafka.producer();

// Collect application logs from thousands of servers
for (const server of servers) {
    await eventLogger.send({
        topic: 'application-logs',
        messages: [{
            key: server.id,
            value: JSON.stringify({
                serverId: server.id,
                level: 'INFO',
                message: 'Request processed',
                timestamp: Date.now()
            })
        }]
    });
}

// Process logs in real-time with multiple consumers
const logProcessor = kafka.consumer({ groupId: 'log-analyzers' });
await logProcessor.subscribe({ topic: 'application-logs' });
await logProcessor.run({
    eachMessage: async ({ message }) => {
        await analyzeLog(JSON.parse(message.value.toString()));
    }
});
```

**Why Kafka?**
- High throughput for massive log volumes
- Durable storage allows replay for debugging
- Multiple consumers can process same logs differently
- Partitioning enables parallel processing

#### 2. **Real-time Analytics Pipelines**
```javascript
// Stream processing for real-time metrics
const analyticsStream = new StreamProcessor(
    kafka,
    'user-clicks',
    'click-analytics',
    async (click) => {
        return {
            userId: click.userId,
            page: click.page,
            timestamp: click.timestamp,
            sessionDuration: calculateSessionDuration(click),
            category: classifyClick(click)
        };
    }
);

await analyticsStream.start();
```

**Why Kafka?**
- Built for stream processing
- Windowed aggregations
- Complex event processing
- Integration with stream processing frameworks

#### 3. **Event Sourcing**
```javascript
// Kafka as event store
const orderEvents = kafka.producer();

// Append events (never deleted)
await orderEvents.send({
    topic: 'order-events',
    messages: [
        { value: JSON.stringify({ type: 'OrderCreated', data: {...} }) },
        { value: JSON.stringify({ type: 'OrderPaid', data: {...} }) },
        { value: JSON.stringify({ type: 'OrderShipped', data: {...} }) }
    ]
});

// Rebuild state by replaying events
const consumer = kafka.consumer({ groupId: 'state-rebuilder' });
await consumer.subscribe({ topic: 'order-events', fromBeginning: true });
```

**Why Kafka?**
- Persistent, immutable event log
- Can replay from any point
- Multiple projections from same events
- Audit trail built-in

### When to Use RabbitMQ

#### 1. **Task Queues and Job Processing**
```javascript
// RabbitMQ excels at task distribution
const taskQueue = await amqp.connect('amqp://localhost');
const channel = await taskQueue.createChannel();

await channel.assertQueue('image-processing', { durable: true });

// Enqueue tasks
await channel.sendToQueue('image-processing', 
    Buffer.from(JSON.stringify({
        imageUrl: 'https://example.com/image.jpg',
        operations: ['resize', 'compress', 'watermark']
    })),
    { persistent: true }
);

// Workers process tasks
await channel.consume('image-processing', async (msg) => {
    const task = JSON.parse(msg.content.toString());
    await processImage(task);
    channel.ack(msg);  // Task removed after completion
}, { noAck: false });
```

**Why RabbitMQ?**
- Simple task distribution
- Messages deleted after processing (no storage bloat)
- Advanced routing for task prioritization
- Lower latency for task pickup

#### 2. **Request-Reply Patterns**
```javascript
// RPC pattern with RabbitMQ
class RPCClient {
    constructor(channel) {
        this.channel = channel;
        this.responseQueue = null;
        this.pendingRequests = new Map();
    }
    
    async initialize() {
        const q = await this.channel.assertQueue('', { exclusive: true });
        this.responseQueue = q.queue;
        
        await this.channel.consume(this.responseQueue, (msg) => {
            const correlationId = msg.properties.correlationId;
            const resolve = this.pendingRequests.get(correlationId);
            if (resolve) {
                resolve(JSON.parse(msg.content.toString()));
                this.pendingRequests.delete(correlationId);
            }
        }, { noAck: true });
    }
    
    async call(method, params) {
        return new Promise((resolve) => {
            const correlationId = generateUUID();
            this.pendingRequests.set(correlationId, resolve);
            
            this.channel.sendToQueue('rpc_queue', 
                Buffer.from(JSON.stringify({ method, params })),
                {
                    correlationId,
                    replyTo: this.responseQueue
                }
            );
        });
    }
}

const rpcClient = new RPCClient(channel);
await rpcClient.initialize();

const result = await rpcClient.call('calculateSum', [5, 10]);
console.log('Result:', result);  // 15
```

**Why RabbitMQ?**
- Native request-reply support
- Low latency responses
- Built-in correlation IDs
- Temporary reply queues

#### 3. **Complex Routing Scenarios**
```javascript
// Advanced routing with headers exchange
await channel.assertExchange('notifications', 'headers', { durable: true });

// Create queues with specific routing rules
await channel.assertQueue('urgent_notifications');
await channel.bindQueue('urgent_notifications', 'notifications', '', {
    'priority': 'high',
    'type': 'alert',
    'x-match': 'all'  // Must match all headers
});

await channel.assertQueue('email_notifications');
await channel.bindQueue('email_notifications', 'notifications', '', {
    'channel': 'email',
    'x-match': 'any'  // Match any header
});

// Publish with headers
channel.publish('notifications', '', 
    Buffer.from(JSON.stringify({ message: 'Critical error!' })),
    {
        headers: {
            priority: 'high',
            type: 'alert',
            channel: 'email'
        }
    }
);
```

**Why RabbitMQ?**
- Flexible routing (direct, topic, fanout, headers)
- Complex message filtering
- Dynamic routing logic
- Multiple routing patterns in one system

### Performance Comparison

```javascript
// Kafka benchmark
async function benchmarkKafka() {
    const kafka = new Kafka({
        clientId: 'benchmark',
        brokers: ['localhost:9092']
    });
    
    const producer = kafka.producer();
    await producer.connect();
    
    const messageCount = 100000;
    const startTime = Date.now();
    
    for (let i = 0; i < messageCount; i++) {
        await producer.send({
            topic: 'benchmark',
            messages: [{ value: `message ${i}` }]
        });
    }
    
    const duration = Date.now() - startTime;
    console.log(`Kafka: ${messageCount} messages in ${duration}ms`);
    console.log(`Throughput: ${Math.round(messageCount / (duration / 1000))} msg/sec`);
}

// RabbitMQ benchmark
async function benchmarkRabbitMQ() {
    const connection = await amqp.connect('amqp://localhost');
    const channel = await connection.createChannel();
    
    await channel.assertQueue('benchmark', { durable: true });
    
    const messageCount = 100000;
    const startTime = Date.now();
    
    for (let i = 0; i < messageCount; i++) {
        await new Promise((resolve, reject) => {
            channel.sendToQueue(
                'benchmark',
                Buffer.from(`message ${i}`),
                { persistent: true },
                (err) => err ? reject(err) : resolve()
            );
        });
    }
    
    const duration = Date.now() - startTime;
    console.log(`RabbitMQ: ${messageCount} messages in ${duration}ms`);
    console.log(`Throughput: ${Math.round(messageCount / (duration / 1000))} msg/sec`);
}

// Typical results:
// Kafka: 100000 messages in 5000ms → 20,000 msg/sec
// RabbitMQ: 100000 messages in 15000ms → 6,667 msg/sec
```

### Decision Matrix

| Requirement | Choose Kafka | Choose RabbitMQ |
|-------------|--------------|------------------|
| High throughput (>100k msg/sec) | ✓ | |
| Low latency (<10ms) | | ✓ |
| Message replay needed | ✓ | |
| Complex routing | | ✓ |
| Temporary messages | | ✓ |
| Stream processing | ✓ | |
| Event sourcing | ✓ | |
| Task queues | | ✓ |
| Request-reply | | ✓ |
| Multi-protocol support | | ✓ |
| Horizontal scalability | ✓ | Moderate |
| Operational simplicity | | ✓ |

### Hybrid Approach

```javascript
// Use both Kafka and RabbitMQ in the same system
class HybridMessageBus {
    constructor(kafka, rabbitmq) {
        this.kafka = kafka;
        this.rabbitmq = rabbitmq;
    }
    
    // Events go to Kafka for streaming/replay
    async publishEvent(eventType, eventData) {
        const producer = this.kafka.producer();
        await producer.send({
            topic: 'events',
            messages: [{
                value: JSON.stringify({ eventType, eventData, timestamp: Date.now() })
            }]
        });
    }
    
    // Commands go to RabbitMQ for task processing
    async sendCommand(commandType, commandData) {
        const channel = await this.rabbitmq.createChannel();
        await channel.assertQueue('commands', { durable: true });
        
        channel.sendToQueue('commands',
            Buffer.from(JSON.stringify({ commandType, commandData })),
            { persistent: true }
        );
    }
}

// Example usage
const messageBus = new HybridMessageBus(kafka, rabbitmqConnection);

// Events: Use Kafka
await messageBus.publishEvent('UserRegistered', { userId: 123 });
await messageBus.publishEvent('OrderPlaced', { orderId: 456 });

// Commands: Use RabbitMQ
await messageBus.sendCommand('SendEmail', { to: 'user@example.com' });
await messageBus.sendCommand('ProcessPayment', { amount: 100 });
```

---

## Advanced Patterns and Best Practices

### 1. Idempotent Message Processing

```javascript
// Ensure messages are processed exactly once, even with retries
class IdempotentProcessor {
    constructor() {
        this.processedIds = new Set();
        // In production, use Redis or database
        this.redis = require('redis').createClient();
    }
    
    async processMessage(messageId, handler) {
        // Check if already processed
        const alreadyProcessed = await this.redis.get(`processed:${messageId}`);
        
        if (alreadyProcessed) {
            console.log(`Message ${messageId} already processed, skipping`);
            return;
        }
        
        try {
            // Process message
            await handler();
            
            // Mark as processed (with TTL)
            await this.redis.setex(`processed:${messageId}`, 86400, 'true');
            
        } catch (error) {
            console.error('Processing failed:', error);
            throw error;  // Will retry
        }
    }
}

// Kafka usage
const processor = new IdempotentProcessor();

await consumer.run({
    eachMessage: async ({ message }) => {
        const messageId = message.headers?.messageId?.toString();
        
        await processor.processMessage(messageId, async () => {
            const data = JSON.parse(message.value.toString());
            await updateDatabase(data);
            await sendNotification(data);
        });
    }
});
```

### 2. Saga Pattern for Distributed Transactions

```javascript
// Implement saga pattern using message queues
class OrderSaga {
    constructor(kafka) {
        this.kafka = kafka;
        this.producer = kafka.producer();
    }
    
    async createOrder(orderData) {
        const sagaId = generateUUID();
        
        // Step 1: Create order
        await this.producer.send({
            topic: 'order-saga',
            messages: [{
                value: JSON.stringify({
                    sagaId,
                    step: 'create-order',
                    data: orderData
                })
            }]
        });
        
        return sagaId;
    }
    
    async listen() {
        const consumer = this.kafka.consumer({ groupId: 'order-saga-coordinator' });
        await consumer.subscribe({ topic: 'order-saga' });
        
        await consumer.run({
            eachMessage: async ({ message }) => {
                const { sagaId, step, data } = JSON.parse(message.value.toString());
                
                try {
                    await this.executeStep(sagaId, step, data);
                } catch (error) {
                    await this.compensate(sagaId, step, data);
                }
            }
        });
    }
    
    async executeStep(sagaId, step, data) {
        switch (step) {
            case 'create-order':
                await createOrderRecord(data);
                // Trigger next step
                await this.producer.send({
                    topic: 'order-saga',
                    messages: [{
                        value: JSON.stringify({
                            sagaId,
                            step: 'reserve-inventory',
                            data
                        })
                    }]
                });
                break;
                
            case 'reserve-inventory':
                await reserveInventory(data.items);
                await this.producer.send({
                    topic: 'order-saga',
                    messages: [{
                        value: JSON.stringify({
                            sagaId,
                            step: 'process-payment',
                            data
                        })
                    }]
                });
                break;
                
            case 'process-payment':
                await processPayment(data.payment);
                await this.producer.send({
                    topic: 'order-saga',
                    messages: [{
                        value: JSON.stringify({
                            sagaId,
                            step: 'complete-order',
                            data
                        })
                    }]
                });
                break;
                
            case 'complete-order':
                await completeOrder(sagaId);
                console.log(`Saga ${sagaId} completed successfully`);
                break;
        }
    }
    
    async compensate(sagaId, failedStep, data) {
        console.log(`Compensating saga ${sagaId} from step ${failedStep}`);
        
        // Rollback in reverse order
        switch (failedStep) {
            case 'process-payment':
                await releaseInventory(data.items);
                // Fall through
            case 'reserve-inventory':
                await cancelOrder(data.orderId);
                break;
        }
        
        console.log(`Saga ${sagaId} compensated`);
    }
}

// Stub functions
async function createOrderRecord(data) { console.log('Order created'); }
async function reserveInventory(items) { console.log('Inventory reserved'); }
async function processPayment(payment) { console.log('Payment processed'); }
async function completeOrder(sagaId) { console.log('Order completed'); }
async function releaseInventory(items) { console.log('Inventory released'); }
async function cancelOrder(orderId) { console.log('Order cancelled'); }
```

### 3. Circuit Breaker Pattern

```javascript
// Protect downstream services with circuit breaker
class CircuitBreakerConsumer {
    constructor(consumer, threshold = 5, timeout = 60000) {
        this.consumer = consumer;
        this.threshold = threshold;
        this.timeout = timeout;
        this.failures = 0;
        this.state = 'CLOSED';  // CLOSED, OPEN, HALF_OPEN
        this.nextAttempt = Date.now();
    }
    
    async consume(handler) {
        await this.consumer.run({
            eachMessage: async ({ message }) => {
                if (this.state === 'OPEN') {
                    if (Date.now() < this.nextAttempt) {
                        console.log('Circuit breaker OPEN, skipping message');
                        // Optionally: send to retry queue
                        return;
                    }
                    this.state = 'HALF_OPEN';
                    console.log('Circuit breaker HALF_OPEN, testing');
                }
                
                try {
                    await handler(message);
                    this.onSuccess();
                } catch (error) {
                    this.onFailure();
                    throw error;
                }
            }
        });
    }
    
    onSuccess() {
        this.failures = 0;
        if (this.state === 'HALF_OPEN') {
            this.state = 'CLOSED';
            console.log('Circuit breaker CLOSED');
        }
    }
    
    onFailure() {
        this.failures++;
        if (this.failures >= this.threshold) {
            this.state = 'OPEN';
            this.nextAttempt = Date.now() + this.timeout;
            console.log('Circuit breaker OPEN');
        }
    }
}
```

### 4. Message Deduplication

```javascript
// Prevent duplicate message processing
class DeduplicationFilter {
    constructor(ttlSeconds = 3600) {
        this.ttlSeconds = ttlSeconds;
        this.redis = require('redis').createClient();
    }
    
    async isDuplicate(messageId) {
        const key = `dedup:${messageId}`;
        const exists = await this.redis.get(key);
        
        if (exists) {
            return true;
        }
        
        // Mark as seen
        await this.redis.setex(key, this.ttlSeconds, '1');
        return false;
    }
}

// Usage
const dedup = new DeduplicationFilter();

await consumer.run({
    eachMessage: async ({ message }) => {
        const messageId = message.key?.toString() || 
                         message.headers?.messageId?.toString();
        
        if (await dedup.isDuplicate(messageId)) {
            console.log('Duplicate message, skipping');
            return;  // Acknowledge but don't process
        }
        
        await processMessage(message);
    }
});
```

### 5. Rate Limiting

```javascript
// Rate limit message processing
class RateLimitedConsumer {
    constructor(messagesPerSecond) {
        this.messagesPerSecond = messagesPerSecond;
        this.tokens = messagesPerSecond;
        this.lastRefill = Date.now();
    }
    
    async consume(consumer, handler) {
        await consumer.run({
            eachMessage: async ({ message, pause }) => {
                // Refill tokens
                this.refillTokens();
                
                // Wait if no tokens available
                while (this.tokens < 1) {
                    await new Promise(resolve => setTimeout(resolve, 100));
                    this.refillTokens();
                }
                
                // Consume token
                this.tokens--;
                
                // Process message
                await handler(message);
            }
        });
    }
    
    refillTokens() {
        const now = Date.now();
        const timePassed = (now - this.lastRefill) / 1000;
        const tokensToAdd = timePassed * this.messagesPerSecond;
        
        this.tokens = Math.min(
            this.messagesPerSecond,
            this.tokens + tokensToAdd
        );
        this.lastRefill = now;
    }
}

// Process max 100 messages per second
const rateLimited = new RateLimitedConsumer(100);
await rateLimited.consume(consumer, async (message) => {
    await processMessage(message);
});
```

---

## Production Deployment and Monitoring

### Kafka Production Configuration

```javascript
// Production-ready Kafka setup
const productionKafka = new Kafka({
    clientId: 'production-app',
    brokers: [
        'kafka-1.prod.example.com:9092',
        'kafka-2.prod.example.com:9092',
        'kafka-3.prod.example.com:9092'
    ],
    // SSL/TLS configuration
    ssl: {
        rejectUnauthorized: true,
        ca: [fs.readFileSync('./ca-cert.pem', 'utf-8')],
        key: fs.readFileSync('./client-key.pem', 'utf-8'),
        cert: fs.readFileSync('./client-cert.pem', 'utf-8')
    },
    // SASL authentication
    sasl: {
        mechanism: 'scram-sha-256',
        username: process.env.KAFKA_USERNAME,
        password: process.env.KAFKA_PASSWORD
    },
    // Connection management
    connectionTimeout: 30000,
    requestTimeout: 30000,
    retry: {
        initialRetryTime: 300,
        retries: 8,
        maxRetryTime: 30000,
        multiplier: 2,
        retryForever: false
    },
    // Logging
    logLevel: logLevel.ERROR
});

// Production producer
const producer = productionKafka.producer({
    idempotent: true,
    maxInFlightRequests: 5,
    transactionTimeout: 60000,
    acks: -1,
    compression: 2,  // Snappy
    retry: {
        retries: 10,
        initialRetryTime: 100
    }
});

// Production consumer
const consumer = productionKafka.consumer({
    groupId: 'production-consumer-group',
    sessionTimeout: 30000,
    rebalanceTimeout: 60000,
    heartbeatInterval: 3000,
    maxBytesPerPartition: 1048576,
    minBytes: 1,
    maxBytes: 10485760,
    maxWaitTimeInMs: 5000,
    retry: {
        retries: 10
    },
    autoCommit: false
});
```

### RabbitMQ Production Configuration

```javascript
// Production-ready RabbitMQ setup
const productionRabbitMQ = await amqp.connect({
    protocol: 'amqps',
    hostname: 'rabbitmq.prod.example.com',
    port: 5671,
    username: process.env.RABBITMQ_USERNAME,
    password: process.env.RABBITMQ_PASSWORD,
    vhost: '/production',
    // TLS options
    ca: [fs.readFileSync('./ca.pem')],
    cert: fs.readFileSync('./cert.pem'),
    key: fs.readFileSync('./key.pem'),
    // Connection pooling
    heartbeat: 60,
    frameMax: 0,
    channelMax: 0
});

// Connection error handling
productionRabbitMQ.on('error', (err) => {
    console.error('RabbitMQ connection error:', err);
    // Implement reconnection logic
});

productionRabbitMQ.on('close', () => {
    console.log('RabbitMQ connection closed');
    // Reconnect
    setTimeout(reconnect, 5000);
});

// Channel with prefetch
const channel = await productionRabbitMQ.createChannel();
await channel.prefetch(100);

// Queue configuration
await channel.assertQueue('production-queue', {
    durable: true,
    arguments: {
        'x-max-length': 1000000,
        'x-max-length-bytes': 10737418240,  // 10GB
        'x-message-ttl': 86400000,  // 24 hours
        'x-dead-letter-exchange': 'dlx-exchange',
        'x-queue-mode': 'lazy'  // Store messages on disk
    }
});
```

### Monitoring and Metrics

```javascript
// Prometheus metrics for Kafka
const prometheus = require('prom-client');

const messagesSent = new prometheus.Counter({
    name: 'kafka_messages_sent_total',
    help: 'Total number of messages sent to Kafka',
    labelNames: ['topic']
});

const messagesProcessed = new prometheus.Counter({
    name: 'kafka_messages_processed_total',
    help: 'Total number of messages processed',
    labelNames: ['topic', 'status']
});

const processingDuration = new prometheus.Histogram({
    name: 'kafka_message_processing_duration_seconds',
    help: 'Message processing duration',
    labelNames: ['topic'],
    buckets: [0.001, 0.01, 0.1, 1, 10]
});

// Instrumented producer
class MonitoredProducer {
    constructor(kafka) {
        this.producer = kafka.producer();
    }
    
    async send(topic, messages) {
        const start = Date.now();
        
        try {
            await this.producer.send({ topic, messages });
            messagesSent.labels(topic).inc(messages.length);
        } catch (error) {
            console.error('Send failed:', error);
            throw error;
        } finally {
            const duration = (Date.now() - start) / 1000;
            processingDuration.labels(topic).observe(duration);
        }
    }
}

// Instrumented consumer
class MonitoredConsumer {
    constructor(kafka, topic) {
        this.consumer = kafka.consumer({ groupId: 'monitored-group' });
        this.topic = topic;
    }
    
    async start(handler) {
        await this.consumer.subscribe({ topic: this.topic });
        
        await this.consumer.run({
            eachMessage: async ({ message }) => {
                const start = Date.now();
                
                try {
                    await handler(message);
                    messagesProcessed.labels(this.topic, 'success').inc();
                } catch (error) {
                    messagesProcessed.labels(this.topic, 'error').inc();
                    throw error;
                } finally {
                    const duration = (Date.now() - start) / 1000;
                    processingDuration.labels(this.topic).observe(duration);
                }
            }
        });
    }
}

// Expose metrics endpoint
const express = require('express');
const app = express();

app.get('/metrics', async (req, res) => {
    res.set('Content-Type', prometheus.register.contentType);
    res.end(await prometheus.register.metrics());
});

app.listen(9090, () => {
    console.log('Metrics server listening on port 9090');
});
```

### Health Checks

```javascript
// Health check endpoint
class HealthChecker {
    constructor(kafka, rabbitmq) {
        this.kafka = kafka;
        this.rabbitmq = rabbitmq;
    }
    
    async checkKafka() {
        try {
            const admin = this.kafka.admin();
            await admin.connect();
            await admin.listTopics();
            await admin.disconnect();
            return { status: 'healthy', latency: 0 };
        } catch (error) {
            return { status: 'unhealthy', error: error.message };
        }
    }
    
    async checkRabbitMQ() {
        try {
            const channel = await this.rabbitmq.createChannel();
            await channel.close();
            return { status: 'healthy' };
        } catch (error) {
            return { status: 'unhealthy', error: error.message };
        }
    }
    
    async getHealth() {
        const [kafka, rabbitmq] = await Promise.all([
            this.checkKafka(),
            this.checkRabbitMQ()
        ]);
        
        return {
            status: kafka.status === 'healthy' && rabbitmq.status === 'healthy' 
                ? 'healthy' : 'unhealthy',
            services: { kafka, rabbitmq },
            timestamp: new Date().toISOString()
        };
    }
}

const healthChecker = new HealthChecker(kafka, rabbitmqConnection);

app.get('/health', async (req, res) => {
    const health = await healthChecker.getHealth();
    const statusCode = health.status === 'healthy' ? 200 : 503;
    res.status(statusCode).json(health);
});
```

---

## Conclusion

Kafka and RabbitMQ are powerful message queue systems, each with distinct strengths:

### Kafka is ideal for:
- **High-throughput event streaming** (millions of messages per second)
- **Event sourcing and audit logs** (immutable, replayable event store)
- **Real-time analytics pipelines** (stream processing)
- **Log aggregation** from distributed systems
- **Scenarios requiring message replay** and historical data access

### RabbitMQ excels at:
- **Traditional task queues** with guaranteed delivery
- **Complex routing scenarios** (multiple exchange types)
- **Low-latency messaging** (microsecond range)
- **Request-reply patterns** (RPC)
- **Temporary messages** that should be deleted after processing
- **Systems requiring multiple protocols** (AMQP, MQTT, STOMP)

### Key Takeaways

1. **Choose based on use case**: Don't default to one; evaluate your specific requirements
2. **Understand delivery guarantees**: Configure acks and commits appropriately
3. **Implement retry and error handling**: Dead letter queues are essential
4. **Monitor and measure**: Instrument your producers and consumers
5. **Test failure scenarios**: Ensure your system handles broker failures gracefully
6. **Consider hybrid approaches**: Use both systems where appropriate

### Best Practices Summary

- **Always use manual acknowledgments** in production for reliability
- **Implement idempotent processing** to handle duplicate messages
- **Use circuit breakers** to protect downstream services
- **Monitor lag and throughput** continuously
- **Configure proper retention policies** based on your needs
- **Secure your brokers** with TLS and authentication
- **Test disaster recovery procedures** regularly

Both Kafka and RabbitMQ are mature, battle-tested technologies that power critical infrastructure at scale. Understanding their architectural differences, strengths, and trade-offs empowers you to build robust, scalable distributed systems.

### Further Learning

- Explore **Kafka Streams** for advanced stream processing
- Study **RabbitMQ clustering** for high availability
- Learn about **schema evolution** with Avro/Protocol Buffers
- Investigate **exactly-once semantics** in depth
- Practice **capacity planning** and performance tuning
- Understand **operational considerations** for production deployments

Mastering message queues is a journey. Start with simple use cases, gradually increase complexity, and always measure the impact of your configurations. Happy messaging!

---

**Tutorial Complete** - You now have a comprehensive understanding of Kafka and RabbitMQ with Node.js, from basic concepts to production-ready implementations.