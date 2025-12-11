# Node.js Complete Mastery Tutorial - Part 2

> Continuation of Node.js Complete Mastery covering EventEmitter, Streams, HTTP, and Advanced Topics

---

## Chapter 5: Event-Driven Architecture (Continued)

### The EventEmitter Class

The EventEmitter is the foundation of Node.js's event-driven architecture. Many Node.js core modules inherit from EventEmitter.

```javascript
const EventEmitter = require('events');

// Create an event emitter instance
const emitter = new EventEmitter();

// REGISTERING EVENT LISTENERS

// Method 1: on() - Add listener
emitter.on('userCreated', (user) => {
  console.log('New user created:', user.name);
});

// Method 2: addEventListener() - Alias for on()
emitter.addEventListener('userCreated', (user) => {
  console.log('User ID:', user.id);
});

// Method 3: once() - Listener fires only once
emitter.once('appStarted', () => {
  console.log('Application started!');
});

// Method 4: prependListener() - Add to beginning of listeners array
emitter.prependListener('userCreated', (user) => {
  console.log('FIRST: User created');
});

// EMITTING EVENTS

// Emit event with data
emitter.emit('userCreated', { id: 1, name: 'Alice' });
// Output:
// FIRST: User created
// New user created: Alice
// User ID: 1

// Emit event multiple times
emitter.emit('userCreated', { id: 2, name: 'Bob' });
// once() listener only fired the first time

// REMOVING LISTENERS

const handler = (data) => console.log('Handler:', data);

emitter.on('data', handler);
emitter.emit('data', 'test');  // Handler: test

// Remove specific listener
emitter.off('data', handler);  // or removeListener()
emitter.emit('data', 'test');  // No output

// Remove all listeners for event
emitter.removeAllListeners('data');

// Remove all listeners for all events
emitter.removeAllListeners();

// GET LISTENER INFORMATION

emitter.on('test', () => {});
emitter.on('test', () => {});

// Count listeners for event
console.log(emitter.listenerCount('test'));  // 2

// Get array of listeners
console.log(emitter.listeners('test'));  // [Function, Function]

// Get array of event names
console.log(emitter.eventNames());  // ['test']

// LISTENER LIMITS

// Default max listeners: 10
// Warning emitted if exceeded (memory leak detection)

emitter.setMaxListeners(20);  // Increase limit
console.log(emitter.getMaxListeners());  // 20

// Set to 0 or Infinity for unlimited
emitter.setMaxListeners(Infinity);
```

### Error Handling with Events

```javascript
const EventEmitter = require('events');
const emitter = new EventEmitter();

// ERROR EVENT IS SPECIAL

// If 'error' event is emitted with no listener, Node.js throws exception
// emitter.emit('error', new Error('Something went wrong'));
// CRASHES: Uncaught Error: Something went wrong

// ALWAYS add error event listener
emitter.on('error', (err) => {
  console.error('Error occurred:', err.message);
  // Handle error gracefully, log, retry, etc.
});

emitter.emit('error', new Error('Database connection failed'));
// Output: Error occurred: Database connection failed
// Application continues running

// PRACTICAL ERROR HANDLING PATTERN

class DatabaseConnection extends EventEmitter {
  connect(connectionString) {
    // Simulate async connection
    setTimeout(() => {
      try {
        // Simulate random connection failure
        if (Math.random() < 0.3) {
          throw new Error('Connection timeout');
        }
        
        this.emit('connected', { connectionString });
      } catch (error) {
        // Emit error event instead of throwing
        this.emit('error', error);
      }
    }, 1000);
  }
  
  query(sql) {
    setTimeout(() => {
      try {
        // Simulate query execution
        const result = { rows: [] };
        this.emit('queryComplete', result);
      } catch (error) {
        this.emit('error', error);
      }
    }, 500);
  }
}

const db = new DatabaseConnection();

// Set up error handler BEFORE any operations
db.on('error', (err) => {
  console.error('Database error:', err.message);
  // Implement retry logic, fallback, etc.
});

db.on('connected', (info) => {
  console.log('Connected to:', info.connectionString);
  db.query('SELECT * FROM users');
});

db.on('queryComplete', (result) => {
  console.log('Query result:', result);
});

db.connect('postgresql://localhost/mydb');
```

### Creating Custom Event Emitters

```javascript
const EventEmitter = require('events');

// EXTENDING EventEmitter

class UserManager extends EventEmitter {
  constructor() {
    super();  // Call parent constructor
    this.users = [];
  }
  
  createUser(userData) {
    // Emit 'beforeCreate' event (can be used to validate/modify data)
    this.emit('beforeCreate', userData);
    
    const user = {
      id: this.users.length + 1,
      ...userData,
      createdAt: new Date()
    };
    
    this.users.push(user);
    
    // Emit 'userCreated' event
    this.emit('userCreated', user);
    
    // Emit 'afterCreate' event
    this.emit('afterCreate', user);
    
    return user;
  }
  
  updateUser(userId, updates) {
    const user = this.users.find(u => u.id === userId);
    
    if (!user) {
      // Emit error event
      this.emit('error', new Error(`User ${userId} not found`));
      return null;
    }
    
    this.emit('beforeUpdate', { userId, updates });
    
    Object.assign(user, updates, { updatedAt: new Date() });
    
    this.emit('userUpdated', user);
    
    return user;
  }
  
  deleteUser(userId) {
    const index = this.users.findIndex(u => u.id === userId);
    
    if (index === -1) {
      this.emit('error', new Error(`User ${userId} not found`));
      return false;
    }
    
    const user = this.users[index];
    this.emit('beforeDelete', user);
    
    this.users.splice(index, 1);
    
    this.emit('userDeleted', user);
    
    return true;
  }
}

// USING THE CUSTOM EVENT EMITTER

const userManager = new UserManager();

// Error handling
userManager.on('error', (err) => {
  console.error('Error:', err.message);
});

// Log all user creations
userManager.on('userCreated', (user) => {
  console.log(`User created: ${user.name} (ID: ${user.id})`);
});

// Send welcome email on user creation
userManager.on('userCreated', (user) => {
  console.log(`Sending welcome email to ${user.email}`);
  // sendWelcomeEmail(user.email);
});

// Audit log on user updates
userManager.on('userUpdated', (user) => {
  console.log(`Audit: User ${user.id} updated at ${user.updatedAt}`);
  // logToAuditDatabase(user);
});

// Cleanup on user deletion
userManager.on('userDeleted', (user) => {
  console.log(`Cleaning up resources for user ${user.id}`);
  // cleanupUserResources(user.id);
});

// Data validation before creation
userManager.on('beforeCreate', (userData) => {
  if (!userData.email || !userData.email.includes('@')) {
    console.warn('Invalid email detected');
  }
});

// Test the event emitter
userManager.createUser({ name: 'Alice', email: 'alice@example.com' });
userManager.createUser({ name: 'Bob', email: 'bob@example.com' });
userManager.updateUser(1, { email: 'alice.new@example.com' });
userManager.deleteUser(2);
userManager.deleteUser(999);  // Error: User not found
```

### Event-Driven Patterns

```javascript
// PATTERN 1: PUB/SUB (Publish-Subscribe)

class EventBus extends EventEmitter {}
const eventBus = new EventBus();

// Subscribers (multiple components can subscribe to same event)

// Logger module
eventBus.on('orderPlaced', (order) => {
  console.log(`[LOG] Order ${order.id} placed by user ${order.userId}`);
});

// Email module
eventBus.on('orderPlaced', (order) => {
  console.log(`[EMAIL] Sending order confirmation to ${order.email}`);
});

// Inventory module
eventBus.on('orderPlaced', (order) => {
  console.log(`[INVENTORY] Reducing stock for order ${order.id}`);
});

// Analytics module
eventBus.on('orderPlaced', (order) => {
  console.log(`[ANALYTICS] Tracking order ${order.id}`);
});

// Publisher
function placeOrder(orderData) {
  const order = {
    id: Date.now(),
    ...orderData,
    placedAt: new Date()
  };
  
  // Emit event - all subscribers notified
  eventBus.emit('orderPlaced', order);
  
  return order;
}

placeOrder({ userId: 123, email: 'user@example.com', items: [] });

// PATTERN 2: EVENT CHAINING

class Pipeline extends EventEmitter {
  process(data) {
    this.emit('start', data);
  }
}

const pipeline = new Pipeline();

// Stage 1: Validation
pipeline.on('start', (data) => {
  console.log('Stage 1: Validating data');
  if (data.isValid) {
    pipeline.emit('validated', data);
  } else {
    pipeline.emit('error', new Error('Invalid data'));
  }
});

// Stage 2: Transformation
pipeline.on('validated', (data) => {
  console.log('Stage 2: Transforming data');
  const transformed = { ...data, transformed: true };
  pipeline.emit('transformed', transformed);
});

// Stage 3: Storage
pipeline.on('transformed', (data) => {
  console.log('Stage 3: Storing data');
  pipeline.emit('stored', data);
});

// Final stage
pipeline.on('stored', (data) => {
  console.log('Pipeline complete:', data);
});

pipeline.on('error', (err) => {
  console.error('Pipeline error:', err.message);
});

pipeline.process({ isValid: true, value: 42 });

// PATTERN 3: REQUEST-RESPONSE PATTERN

class RequestHandler extends EventEmitter {
  request(command, data) {
    const requestId = Date.now();
    
    // Emit request
    this.emit('request', { requestId, command, data });
    
    // Return promise that resolves when response received
    return new Promise((resolve, reject) => {
      const responseHandler = (response) => {
        if (response.requestId === requestId) {
          this.off('response', responseHandler);
          this.off('error', errorHandler);
          
          if (response.error) {
            reject(response.error);
          } else {
            resolve(response.data);
          }
        }
      };
      
      const errorHandler = (error) => {
        if (error.requestId === requestId) {
          this.off('response', responseHandler);
          this.off('error', errorHandler);
          reject(error);
        }
      };
      
      this.on('response', responseHandler);
      this.on('error', errorHandler);
      
      // Timeout after 5 seconds
      setTimeout(() => {
        this.off('response', responseHandler);
        this.off('error', errorHandler);
        reject(new Error('Request timeout'));
      }, 5000);
    });
  }
}

const handler = new RequestHandler();

// Process requests
handler.on('request', (request) => {
  console.log('Processing request:', request.command);
  
  setTimeout(() => {
    // Simulate processing and send response
    handler.emit('response', {
      requestId: request.requestId,
      data: { result: 'success', processed: request.data }
    });
  }, 1000);
});

// Make requests
async function makeRequests() {
  try {
    const result1 = await handler.request('processData', { value: 100 });
    console.log('Result 1:', result1);
    
    const result2 = await handler.request('calculate', { x: 5, y: 10 });
    console.log('Result 2:', result2);
  } catch (error) {
    console.error('Request failed:', error.message);
  }
}

makeRequests();

// PATTERN 4: MEMORY LEAK PREVENTION

class StreamProcessor extends EventEmitter {
  constructor() {
    super();
    this.processing = false;
  }
  
  start() {
    this.processing = true;
    this.process();
  }
  
  stop() {
    this.processing = false;
    // Clean up all listeners to prevent memory leaks
    this.removeAllListeners();
  }
  
  process() {
    if (!this.processing) return;
    
    this.emit('data', { timestamp: Date.now() });
    
    setTimeout(() => this.process(), 100);
  }
}

const processor = new StreamProcessor();

// Weak reference pattern for temporary listeners
let dataHandler = (data) => {
  console.log('Data:', data.timestamp);
};

processor.on('data', dataHandler);
processor.start();

// Clean up after 1 second
setTimeout(() => {
  processor.stop();
  dataHandler = null;  // Allow garbage collection
  console.log('Processor stopped and cleaned up');
}, 1000);
```

---

## Chapter 6: Streams in Node.js

### Understanding Streams

Streams are objects that allow reading or writing data in chunks, rather than loading everything into memory at once. This is crucial for handling large files and real-time data.

#### Stream Types

```javascript
/*
FOUR TYPES OF STREAMS:

1. Readable - Read data from source (fs.createReadStream, http.IncomingMessage)
2. Writable - Write data to destination (fs.createWriteStream, http.ServerResponse)
3. Duplex - Both readable and writable (net.Socket, TCP sockets)
4. Transform - Modify data while reading/writing (zlib.createGzip, crypto)

All streams are instances of EventEmitter
*/
```

### Readable Streams

```javascript
const fs = require('fs');
const { Readable } = require('stream');

// READING FILES WITH STREAMS

// Without streams (BAD for large files)
// const data = fs.readFileSync('large-file.txt', 'utf8');
// console.log(data);  // Entire file loaded into memory!

// With streams (GOOD for large files)
const readStream = fs.createReadStream('large-file.txt', {
  encoding: 'utf8',
  highWaterMark: 64 * 1024  // Buffer size: 64 KB chunks
});

// Listen to stream events
readStream.on('data', (chunk) => {
  console.log('Received chunk:', chunk.length, 'bytes');
  // Process chunk while rest of file is still loading
});

readStream.on('end', () => {
  console.log('File reading complete');
});

readStream.on('error', (err) => {
  console.error('Error reading file:', err.message);
});

readStream.on('close', () => {
  console.log('Stream closed');
});

// PAUSE AND RESUME

readStream.on('data', (chunk) => {
  console.log('Got chunk');
  
  // Pause stream to slow down reading
  readStream.pause();
  
  // Resume after processing (simulate slow processing)
  setTimeout(() => {
    readStream.resume();
  }, 1000);
});

// CREATING CUSTOM READABLE STREAM

class NumberStream extends Readable {
  constructor(max, options) {
    super(options);
    this.max = max;
    this.current = 1;
  }
  
  // _read is called when stream needs more data
  _read(size) {
    if (this.current <= this.max) {
      // Push data to stream
      this.push(String(this.current) + '\n');
      this.current++;
    } else {
      // No more data - push null to signal end
      this.push(null);
    }
  }
}

const numberStream = new NumberStream(10);

numberStream.on('data', (chunk) => {
  console.log('Number:', chunk.toString().trim());
});

numberStream.on('end', () => {
  console.log('Number stream ended');
});

// READABLE MODES

// Flowing mode - data automatically provided
const flowingStream = fs.createReadStream('file.txt');
flowingStream.on('data', (chunk) => {
  // Automatically receives data
});

// Paused mode - must explicitly call read()
const pausedStream = fs.createReadStream('file.txt');
pausedStream.on('readable', () => {
  let chunk;
  while ((chunk = pausedStream.read()) !== null) {
    console.log('Read chunk:', chunk.length);
  }
});
```

### Writable Streams

```javascript
const fs = require('fs');
const { Writable } = require('stream');

// WRITING FILES WITH STREAMS

// Create writable stream
const writeStream = fs.createWriteStream('output.txt', {
  encoding: 'utf8'
});

// Write data to stream
writeStream.write('First line\n');
writeStream.write('Second line\n');
writeStream.write('Third line\n');

// Signal end of writing
writeStream.end('Final line\n');

// Or combine: write and end in one call
// writeStream.end('Final line\n');

// Listen to events
writeStream.on('finish', () => {
  console.log('All writes completed');
});

writeStream.on('error', (err) => {
  console.error('Write error:', err.message);
});

// BACKPRESSURE HANDLING

// write() returns false when internal buffer is full
const success = writeStream.write('data');

if (!success) {
  console.log('Buffer full, waiting...');
  
  // Wait for 'drain' event before writing more
  writeStream.once('drain', () => {
    console.log('Buffer drained, can write more');
    writeStream.write('more data');
  });
}

// PRACTICAL EXAMPLE: Efficient file copy

function copyFile(source, destination) {
  const readStream = fs.createReadStream(source);
  const writeStream = fs.createWriteStream(destination);
  
  let totalBytes = 0;
  
  readStream.on('data', (chunk) => {
    totalBytes += chunk.length;
    const canContinue = writeStream.write(chunk);
    
    // Handle backpressure
    if (!canContinue) {
      console.log('Pausing read - write buffer full');
      readStream.pause();
    }
  });
  
  // Resume reading when write buffer drains
  writeStream.on('drain', () => {
    console.log('Resuming read - write buffer drained');
    readStream.resume();
  });
  
  readStream.on('end', () => {
    writeStream.end();
    console.log(`Copied ${totalBytes} bytes`);
  });
  
  readStream.on('error', (err) => {
    console.error('Read error:', err);
    writeStream.end();
  });
  
  writeStream.on('error', (err) => {
    console.error('Write error:', err);
    readStream.destroy();
  });
}

// copyFile('large-file.txt', 'copy.txt');

// CREATING CUSTOM WRITABLE STREAM

class Logger extends Writable {
  constructor(options) {
    super(options);
    this.logs = [];
  }
  
  // _write is called for each chunk
  _write(chunk, encoding, callback) {
    const log = {
      timestamp: new Date(),
      message: chunk.toString()
    };
    
    this.logs.push(log);
    console.log(`[${log.timestamp.toISOString()}] ${log.message}`);
    
    // Call callback when write is complete
    callback();
  }
  
  // _final is called before stream closes
  _final(callback) {
    console.log(`Total logs written: ${this.logs.length}`);
    callback();
  }
}

const logger = new Logger();

logger.write('Application started\n');
logger.write('Processing request\n');
logger.write('Request completed\n');
logger.end();
```

### Duplex and Transform Streams

```javascript
const { Duplex, Transform } = require('stream');
const fs = require('fs');

// DUPLEX STREAM - Both readable and writable

class MyDuplex extends Duplex {
  constructor(options) {
    super(options);
    this.data = [];
  }
  
  // Implement _read for readable side
  _read(size) {
    if (this.data.length > 0) {
      this.push(this.data.shift());
    } else {
      this.push(null);  // No more data
    }
  }
  
  // Implement _write for writable side
  _write(chunk, encoding, callback) {
    // Store data that can be read later
    this.data.push(chunk);
    callback();
  }
}

const duplex = new MyDuplex();

// Write to duplex stream
duplex.write('Hello ');
duplex.write('World');
duplex.end();

// Read from duplex stream
duplex.on('data', (chunk) => {
  console.log('Read:', chunk.toString());
});

// TRANSFORM STREAM - Modify data during transmission

// Example 1: Uppercase transformer
class UpperCaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    // Transform data to uppercase
    this.push(chunk.toString().toUpperCase());
    callback();
  }
}

const upperCase = new UpperCaseTransform();

// Pipe data through transform
process.stdin
  .pipe(upperCase)
  .pipe(process.stdout);

// Example 2: CSV to JSON transformer
class CSVToJSON extends Transform {
  constructor(options) {
    super(options);
    this.headers = null;
    this.lineNumber = 0;
  }
  
  _transform(chunk, encoding, callback) {
    const lines = chunk.toString().split('\n');
    
    for (const line of lines) {
      if (!line.trim()) continue;
      
      if (this.lineNumber === 0) {
        // First line is headers
        this.headers = line.split(',').map(h => h.trim());
      } else {
        // Convert CSV line to JSON
        const values = line.split(',').map(v => v.trim());
        const obj = {};
        
        this.headers.forEach((header, i) => {
          obj[header] = values[i];
        });
        
        this.push(JSON.stringify(obj) + '\n');
      }
      
      this.lineNumber++;
    }
    
    callback();
  }
}

const csvToJson = new CSVToJSON();

// Usage
fs.createReadStream('data.csv')
  .pipe(csvToJson)
  .pipe(fs.createWriteStream('data.json'));

// Example 3: Compression transform
const zlib = require('zlib');

// Compress file
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));

// Decompress file
fs.createReadStream('input.txt.gz')
  .pipe(zlib.createGunzip())
  .pipe(fs.createWriteStream('output.txt'));

// Example 4: Encryption transform
const crypto = require('crypto');

// Encrypt
const encryptKey = crypto.randomBytes(32);
const encryptIv = crypto.randomBytes(16);
const cipher = crypto.createCipheriv('aes-256-cbc', encryptKey, encryptIv);

fs.createReadStream('plain.txt')
  .pipe(cipher)
  .pipe(fs.createWriteStream('encrypted.txt'));

// Decrypt
const decipher = crypto.createDecipheriv('aes-256-cbc', encryptKey, encryptIv);

fs.createReadStream('encrypted.txt')
  .pipe(decipher)
  .pipe(fs.createWriteStream('decrypted.txt'));
```

### Piping Streams

```javascript
const fs = require('fs');
const zlib = require('zlib');

// BASIC PIPING

// Read file and write to another file
fs.createReadStream('input.txt')
  .pipe(fs.createWriteStream('output.txt'));

// CHAINING PIPES

// Read → Compress → Write
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));

// MULTIPLE TRANSFORMS

// Read → Uppercase → Compress → Write
fs.createReadStream('input.txt')
  .pipe(new UpperCaseTransform())
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('output.txt.gz'));

// ERROR HANDLING IN PIPES

// BAD: Error in one stream doesn't automatically handle others
fs.createReadStream('nonexistent.txt')
  .pipe(fs.createWriteStream('output.txt'));
// If read fails, write stream may remain open!

// GOOD: Handle errors on each stream
const read = fs.createReadStream('input.txt');
const write = fs.createWriteStream('output.txt');

read.on('error', (err) => {
  console.error('Read error:', err.message);
  write.end();  // Close write stream
});

write.on('error', (err) => {
  console.error('Write error:', err.message);
  read.destroy();  // Destroy read stream
});

read.pipe(write);

// BETTER: Use pipeline() for automatic error handling
const { pipeline } = require('stream');

pipeline(
  fs.createReadStream('input.txt'),
  zlib.createGzip(),
  fs.createWriteStream('output.txt.gz'),
  (err) => {
    if (err) {
      console.error('Pipeline failed:', err.message);
    } else {
      console.log('Pipeline succeeded');
    }
  }
);

// STREAM UTILITIES

// stream.finished() - Know when stream is done
const { finished } = require('stream');

const stream = fs.createReadStream('large-file.txt');

finished(stream, (err) => {
  if (err) {
    console.error('Stream failed', err);
  } else {
    console.log('Stream completed successfully');
  }
});

// stream.pipeline() with promises
const { promisify } = require('util');
const pipelineAsync = promisify(pipeline);

async function compressFile(input, output) {
  try {
    await pipelineAsync(
      fs.createReadStream(input),
      zlib.createGzip(),
      fs.createWriteStream(output)
    );
    console.log('File compressed successfully');
  } catch (err) {
    console.error('Compression failed:', err);
  }
}

compressFile('large-file.txt', 'large-file.txt.gz');
```

### Real-World Stream Applications

```javascript
// APPLICATION 1: Large File Processing

const fs = require('fs');
const readline = require('readline');

// Process large log file line by line
async function processLogFile(filename) {
  const fileStream = fs.createReadStream(filename);
  
  const rl = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity  // Handle both \r\n and \n
  });
  
  let errorCount = 0;
  let warningCount = 0;
  
  for await (const line of rl) {
    if (line.includes('ERROR')) errorCount++;
    if (line.includes('WARN')) warningCount++;
  }
  
  console.log(`Errors: ${errorCount}, Warnings: ${warningCount}`);
}

processLogFile('app.log');

// APPLICATION 2: HTTP File Upload/Download

const http = require('http');

// File download server
http.createServer((req, res) => {
  // Stream file to client (efficient for large files)
  const fileStream = fs.createReadStream('video.mp4');
  
  res.writeHead(200, {
    'Content-Type': 'video/mp4',
    'Content-Disposition': 'attachment; filename=video.mp4'
  });
  
  // Pipe file directly to response
  fileStream.pipe(res);
  
  fileStream.on('error', (err) => {
    res.writeHead(500);
    res.end('Error reading file');
  });
}).listen(3000);

// File upload handler
http.createServer((req, res) => {
  if (req.method === 'POST') {
    const writeStream = fs.createWriteStream('uploaded-file.dat');
    
    // Stream request body directly to file
    req.pipe(writeStream);
    
    writeStream.on('finish', () => {
      res.end('File uploaded successfully');
    });
    
    writeStream.on('error', (err) => {
      res.writeHead(500);
      res.end('Upload failed');
    });
  }
}).listen(3001);

// APPLICATION 3: Real-time Data Processing

const { Transform } = require('stream');

class MetricsCollector extends Transform {
  constructor(options) {
    super(options);
    this.requestCount = 0;
    this.totalBytes = 0;
  }
  
  _transform(chunk, encoding, callback) {
    this.requestCount++;
    this.totalBytes += chunk.length;
    
    // Every 10 requests, emit metrics
    if (this.requestCount % 10 === 0) {
      console.log(`Requests: ${this.requestCount}, Bytes: ${this.totalBytes}`);
    }
    
    this.push(chunk);  // Pass data through
    callback();
  }
  
  _flush(callback) {
    console.log(`Final - Requests: ${this.requestCount}, Bytes: ${this.totalBytes}`);
    callback();
  }
}

// Use in HTTP server
http.createServer((req, res) => {
  const metrics = new MetricsCollector();
  
  req
    .pipe(metrics)
    .pipe(res);
}).listen(3002);
```

---

## Chapter 7: Buffers and Binary Data

### Understanding Buffers

Buffers are Node.js's way of handling binary data. They represent fixed-size chunks of memory allocated outside the V8 heap.

```javascript
// CREATING BUFFERS

// Method 1: Create empty buffer of specific size
const buf1 = Buffer.alloc(10);  // 10 bytes, filled with 0
console.log(buf1);  // <Buffer 00 00 00 00 00 00 00 00 00 00>

// Method 2: Create uninitialized buffer (faster but contains old data)
const buf2 = Buffer.allocUnsafe(10);  // May contain sensitive data!
console.log(buf2);  // <Buffer ?? ?? ?? ?? ?? ?? ?? ?? ?? ??>

// Method 3: Create from string
const buf3 = Buffer.from('Hello World', 'utf8');
console.log(buf3);  // <Buffer 48 65 6c 6c 6f 20 57 6f 72 6c 64>

// Method 4: Create from array of bytes
const buf4 = Buffer.from([72, 101, 108, 108, 111]);  // "Hello"
console.log(buf4.toString());  // Hello

// Method 5: Create from another buffer (copy)
const buf5 = Buffer.from(buf3);
console.log(buf5.toString());  // Hello World

// WORKING WITH BUFFERS

// Read and write individual bytes
const buf = Buffer.alloc(5);

buf[0] = 72;   // 'H'
buf[1] = 101;  // 'e'
buf[2] = 108;  // 'l'
buf[3] = 108;  // 'l'
buf[4] = 111;  // 'o'

console.log(buf.toString());  // Hello

// Buffer properties
console.log(buf.length);  // 5
console.log(buf.byteLength);  // 5 (same as length)

// BUFFER METHODS

// toString() - Convert to string
const textBuf = Buffer.from('Node.js');
console.log(textBuf.toString());  // Node.js
console.log(textBuf.toString('hex'));  // 4e6f64652e6a73
console.log(textBuf.toString('base64'));  // Tm9kZS5qcw==

// write() - Write to buffer
const writeBuf = Buffer.alloc(10);
writeBuf.write('Hello');
console.log(writeBuf.toString());  // Hello

// Specify offset and encoding
writeBuf.write('World', 5, 'utf8');
console.log(writeBuf.toString());  // HelloWorld (truncated to 10 bytes)

// slice() - Create view of buffer (shares memory!)
const original = Buffer.from('Hello World');
const slice = original.slice(0, 5);
console.log(slice.toString());  // Hello

// Modifying slice affects original!
slice[0] = 74;  // 'J'
console.log(original.toString());  // Jello World

// subarray() - Same as slice() (newer API)
const sub = original.subarray(6, 11);
console.log(sub.toString());  // World

// copy() - Copy buffer data
const source = Buffer.from('Hello');
const target = Buffer.alloc(10);

source.copy(target, 0);  // Copy to target starting at index 0
console.log(target.toString());  // Hello

// concat() - Concatenate buffers
const buf1 = Buffer.from('Hello ');
const buf2 = Buffer.from('World');
const combined = Buffer.concat([buf1, buf2]);
console.log(combined.toString());  // Hello World

// equals() - Compare buffers
const bufA = Buffer.from('ABC');
const bufB = Buffer.from('ABC');
const bufC = Buffer.from('XYZ');

console.log(bufA.equals(bufB));  // true
console.log(bufA.equals(bufC));  // false

// compare() - Lexicographic comparison
console.log(bufA.compare(bufB));  // 0 (equal)
console.log(bufA.compare(bufC));  // -1 (bufA < bufC)
console.log(bufC.compare(bufA));  // 1 (bufC > bufA)

// fill() - Fill buffer with value
const fillBuf = Buffer.alloc(10);
fillBuf.fill('ab');
console.log(fillBuf.toString());  // ababababab

fillBuf.fill(0);  // Fill with zeros
console.log(fillBuf);  // <Buffer 00 00 00 00 00 00 00 00 00 00>

// indexOf() - Find value in buffer
const searchBuf = Buffer.from('Hello World');
console.log(searchBuf.indexOf('World'));  // 6
console.log(searchBuf.indexOf('xyz'));    // -1 (not found)

// includes() - Check if buffer contains value
console.log(searchBuf.includes('World'));  // true
console.log(searchBuf.includes('xyz'));    // false
```

### Binary Data Operations

```javascript
// READING AND WRITING INTEGERS

const buf = Buffer.alloc(8);

// Write integers
buf.writeUInt8(255, 0);      // 1 byte unsigned at offset 0
buf.writeUInt16BE(65535, 1); // 2 bytes big-endian at offset 1
buf.writeUInt32LE(4294967295, 3);  // 4 bytes little-endian at offset 3

console.log(buf);

// Read integers
console.log(buf.readUInt8(0));      // 255
console.log(buf.readUInt16BE(1));   // 65535
console.log(buf.readUInt32LE(3));   // 4294967295

// READING AND WRITING FLOATS

const floatBuf = Buffer.alloc(8);

floatBuf.writeFloatBE(3.14159, 0);
floatBuf.writeDoubleBE(2.718281828, 4);

console.log(floatBuf.readFloatBE(0));   // 3.14159...
console.log(floatBuf.readDoubleBE(4));  // 2.718281828

// ENDIANNESS

// Big Endian (BE): Most significant byte first
// Little Endian (LE): Least significant byte first

const number = 0x12345678;

const beBuf = Buffer.alloc(4);
beBuf.writeUInt32BE(number, 0);
console.log(beBuf);  // <Buffer 12 34 56 78>

const leBuf = Buffer.alloc(4);
leBuf.writeUInt32LE(number, 0);
console.log(leBuf);  // <Buffer 78 56 34 12>

// PRACTICAL EXAMPLE: Binary Protocol

class BinaryProtocol {
  // Encode message: [length (4 bytes)] [message (variable)]
  static encode(message) {
    const msgBuf = Buffer.from(message, 'utf8');
    const lenBuf = Buffer.alloc(4);
    
    lenBuf.writeUInt32BE(msgBuf.length, 0);
    
    return Buffer.concat([lenBuf, msgBuf]);
  }
  
  // Decode message
  static decode(buffer) {
    const length = buffer.readUInt32BE(0);
    const message = buffer.slice(4, 4 + length).toString('utf8');
    
    return {
      message,
      remaining: buffer.slice(4 + length)
    };
  }
}

// Usage
const encoded = BinaryProtocol.encode('Hello World');
console.log(encoded);  // <Buffer 00 00 00 0b 48 65 6c 6c 6f 20 57 6f 72 6c 64>

const decoded = BinaryProtocol.decode(encoded);
console.log(decoded.message);  // Hello World
```

### Working with File Buffers

```javascript
const fs = require('fs');

// READ FILE AS BUFFER

// Synchronous
const fileBuffer = fs.readFileSync('image.png');
console.log(fileBuffer);  // <Buffer 89 50 4e 47 ...>
console.log(fileBuffer.length, 'bytes');

// Asynchronous
fs.readFile('image.png', (err, data) => {
  if (err) throw err;
  console.log('File size:', data.length);
  
  // Check PNG signature (first 8 bytes)
  const pngSignature = Buffer.from([0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A]);
  const fileSignature = data.slice(0, 8);
  
  if (pngSignature.equals(fileSignature)) {
    console.log('Valid PNG file');
  } else {
    console.log('Not a PNG file');
  }
});

// WRITE BUFFER TO FILE

const imageData = Buffer.alloc(1000);  // Simulate image data
// ... fill buffer with actual image data ...

fs.writeFileSync('output.dat', imageData);

// BUFFER STREAM PROCESSING

const stream = fs.createReadStream('large-file.dat');

let totalBytes = 0;

stream.on('data', (chunk) => {
  totalBytes += chunk.length;
  console.log(`Received ${chunk.length} bytes`);
  // Process buffer chunk
});

stream.on('end', () => {
  console.log(`Total: ${totalBytes} bytes`);
});
```

### Buffer Pool and Memory Management

```javascript
// Buffer.allocUnsafe() is faster but may contain old data

// BAD: Security risk with allocUnsafe
function getUserData() {
  const buf = Buffer.allocUnsafe(100);
  // Buffer may contain fragments of old sensitive data!
  return buf;
}

// GOOD: Use alloc for sensitive data
function getUserDataSecure() {
  const buf = Buffer.alloc(100);  // Filled with zeros
  return buf;
}

// GOOD: Or fill allocUnsafe immediately
function getUserDataFast() {
  const buf = Buffer.allocUnsafe(100);
  buf.fill(0);  // Clear any old data
  return buf;
}

// BUFFER POOLING

// Small buffers (<= 4KB) are allocated from a pre-allocated slab
// This reduces memory fragmentation and improves performance

const small1 = Buffer.allocUnsafe(100);  // From pool
const small2 = Buffer.allocUnsafe(200);  // From pool
const large = Buffer.allocUnsafe(10000); // Not pooled

// MEMORY CONSIDERATIONS

// Buffers are allocated outside V8 heap
// Not automatically garbage collected like JavaScript objects
// Large buffers can cause memory pressure

function processLargeFile() {
  const huge Buffer = Buffer.alloc(1024 * 1024 * 1024);  // 1GB!
  // This bypasses V8's memory limit but can crash the process
  
  // Better: Use streams for large data
}
```

---

## Chapter 8: File System Operations

### Reading Files

```javascript
const fs = require('fs');
const fsPromises = require('fs').promises;

// SYNCHRONOUS FILE READING (Blocks execution)

// Read entire file
try {
  const data = fs.readFileSync('file.txt', 'utf8');
  console.log(data);
} catch (err) {
  console.error('Error reading file:', err.message);
}

// Read as buffer
const buffer = fs.readFileSync('image.png');
console.log(buffer);

// ASYNCHRONOUS FILE READING (Callback-based)

fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Error:', err.message);
    return;
  }
  console.log(data);
});

// PROMISE-BASED FILE READING (Modern approach)

async function readFileAsync() {
  try {
    const data = await fsPromises.readFile('file.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error('Error:', err.message);
  }
}

readFileAsync();

// READ FILE IN CHUNKS (Large files)

const stream = fs.createReadStream('large-file.txt', {
  encoding: 'utf8',
  highWaterMark: 64 * 1024  // 64KB chunks
});

stream.on('data', (chunk) => {
  console.log('Chunk size:', chunk.length);
  // Process chunk
});

stream.on('end', () => {
  console.log('File reading complete');
});

stream.on('error', (err) => {
  console.error('Stream error:', err.message);
});
```

### Writing Files

```javascript
const fs = require('fs');
const fsPromises = require('fs').promises;

// SYNCHRONOUS WRITING

fs.writeFileSync('output.txt', 'Hello World', 'utf8');

// Append mode
fs.appendFileSync('log.txt', 'New log entry\n', 'utf8');

// ASYNCHRONOUS WRITING

fs.writeFile('output.txt', 'Hello World', 'utf8', (err) => {
  if (err) throw err;
  console.log('File written successfully');
});

// PROMISE-BASED WRITING

async function writeFileAsync() {
  try {
    await fsPromises.writeFile('output.txt', 'Hello World', 'utf8');
    console.log('File written');
  } catch (err) {
    console.error('Error:', err.message);
  }
}

writeFileAsync();

// STREAM WRITING (Large data)

const writeStream = fs.createWriteStream('large-output.txt');

for (let i = 0; i < 1000000; i++) {
  writeStream.write(`Line ${i}\n`);
}

writeStream.end();

writeStream.on('finish', () => {
  console.log('Writing complete');
});

// ATOMIC WRITES (Write to temp, then rename)

async function atomicWrite(filename, data) {
  const tempFile = `${filename}.tmp`;
  
  try {
    await fsPromises.writeFile(tempFile, data);
    await fsPromises.rename(tempFile, filename);
    console.log('Atomic write successful');
  } catch (err) {
    // Clean up temp file on error
    try {
      await fsPromises.unlink(tempFile);
    } catch (unlinkErr) {
      // Ignore if temp file doesn't exist
    }
    throw err;
  }
}

atomicWrite('important.json', JSON.stringify({ key: 'value' }));
```

### File and Directory Operations

```javascript
const fs = require('fs');
const fsPromises = require('fs').promises;
const path = require('path');

// CHECK IF FILE/DIRECTORY EXISTS

// Using fs.existsSync (synchronous)
if (fs.existsSync('file.txt')) {
  console.log('File exists');
}

// Using fs.access (asynchronous, preferred)
fs.access('file.txt', fs.constants.F_OK, (err) => {
  if (err) {
    console.log('File does not exist');
  } else {
    console.log('File exists');
  }
});

// Promise-based
async function checkExists(filepath) {
  try {
    await fsPromises.access(filepath);
    return true;
  } catch {
    return false;
  }
}

// GET FILE INFORMATION

fs.stat('file.txt', (err, stats) => {
  if (err) throw err;
  
  console.log('File size:', stats.size);
  console.log('Is file:', stats.isFile());
  console.log('Is directory:', stats.isDirectory());
  console.log('Created:', stats.birthtime);
  console.log('Modified:', stats.mtime);
  console.log('Accessed:', stats.atime);
  console.log('Mode:', stats.mode);
});

// RENAME/MOVE FILE

fs.rename('old-name.txt', 'new-name.txt', (err) => {
  if (err) throw err;
  console.log('File renamed');
});

// Move to different directory
fs.rename('file.txt', 'backup/file.txt', (err) => {
  if (err) throw err;
  console.log('File moved');
});

// DELETE FILE

fs.unlink('file-to-delete.txt', (err) => {
  if (err) throw err;
  console.log('File deleted');
});

// COPY FILE

fs.copyFile('source.txt', 'destination.txt', (err) => {
  if (err) throw err;
  console.log('File copied');
});

// Copy with flags
fs.copyFile('source.txt', 'dest.txt', fs.constants.COPYFILE_EXCL, (err) => {
  if (err) throw err;  // Fails if dest.txt already exists
  console.log('File copied (exclusive)');
});

// DIRECTORY OPERATIONS

// Create directory
fs.mkdir('new-directory', (err) => {
  if (err) throw err;
  console.log('Directory created');
});

// Create nested directories
fs.mkdir('path/to/nested/dir', { recursive: true }, (err) => {
  if (err) throw err;
  console.log('Nested directories created');
});

// Read directory contents
fs.readdir('.', (err, files) => {
  if (err) throw err;
  console.log('Files:', files);
});

// Read directory with file types
fs.readdir('.', { withFileTypes: true }, (err, entries) => {
  if (err) throw err;
  
  entries.forEach(entry => {
    console.log(entry.name, entry.isDirectory() ? '[DIR]' : '[FILE]');
  });
});

// Remove directory
fs.rmdir('empty-directory', (err) => {
  if (err) throw err;
  console.log('Directory removed');
});

// Remove directory recursively
fs.rm('directory-with-files', { recursive: true, force: true }, (err) => {
  if (err) throw err;
  console.log('Directory and contents removed');
});
```

### Advanced File Operations

```javascript
const fs = require('fs');
const fsPromises = require('fs').promises;
const path = require('path');

// WATCHING FILES AND DIRECTORIES

// Watch file for changes
const watcher = fs.watch('file.txt', (eventType, filename) => {
  console.log(`Event: ${eventType}, File: ${filename}`);
  // eventType: 'rename' or 'change'
});

// Stop watching
// watcher.close();

// Watch directory
fs.watch('directory', { recursive: true }, (eventType, filename) => {
  console.log(`${filename} ${eventType}d`);
});

// More reliable watching with watchFile
fs.watchFile('file.txt', { interval: 1000 }, (curr, prev) => {
  console.log(`File modified: ${prev.mtime} → ${curr.mtime}`);
});

// Stop watching
// fs.unwatchFile('file.txt');

// FILE PERMISSIONS

// Change mode (permissions)
fs.chmod('file.txt', 0o755, (err) => {
  if (err) throw err;
  console.log('Permissions changed');
});

// Change owner (requires privileges)
fs.chown('file.txt', 1000, 1000, (err) => {
  if (err) throw err;
  console.log('Owner changed');
});

// SYMBOLIC LINKS

// Create symbolic link
fs.symlink('target.txt', 'link.txt', (err) => {
  if (err) throw err;
  console.log('Symbolic link created');
});

// Read symbolic link
fs.readlink('link.txt', (err, linkString) => {
  if (err) throw err;
  console.log('Link points to:', linkString);
});

// HARD LINKS

fs.link('original.txt', 'hardlink.txt', (err) => {
  if (err) throw err;
  console.log('Hard link created');
});

// FILE DESCRIPTORS (Low-level operations)

// Open file
fs.open('file.txt', 'r', (err, fd) => {
  if (err) throw err;
  
  // Read from specific position
  const buffer = Buffer.alloc(100);
  fs.read(fd, buffer, 0, 100, 0, (err, bytesRead, buffer) => {
    if (err) throw err;
    console.log('Read:', buffer.slice(0, bytesRead).toString());
    
    // Close file descriptor
    fs.close(fd, (err) => {
      if (err) throw err;
      console.log('File closed');
    });
  });
});

// Write to specific position
fs.open('file.txt', 'r+', (err, fd) => {
  if (err) throw err;
  
  const data = Buffer.from('INSERT');
  fs.write(fd, data, 0, data.length, 10, (err, written) => {
    if (err) throw err;
    console.log(`Wrote ${written} bytes`);
    
    fs.close(fd, (err) => {
      if (err) throw err;
    });
  });
});

// PRACTICAL EXAMPLES

// Recursively list all files in directory
async function getAllFiles(dirPath, arrayOfFiles = []) {
  const files = await fsPromises.readdir(dirPath);
  
  for (const file of files) {
    const fullPath = path.join(dirPath, file);
    const stat = await fsPromises.stat(fullPath);
    
    if (stat.isDirectory()) {
      arrayOfFiles = await getAllFiles(fullPath, arrayOfFiles);
    } else {
      arrayOfFiles.push(fullPath);
    }
  }
  
  return arrayOfFiles;
}

getAllFiles('.').then(files => {
  console.log('All files:', files);
});

// Find files by extension
async function findFilesByExtension(dirPath, ext) {
  const allFiles = await getAllFiles(dirPath);
  return allFiles.filter(file => path.extname(file) === ext);
}

findFilesByExtension('.', '.js').then(jsFiles => {
  console.log('JavaScript files:', jsFiles);
});

// Calculate directory size
async function getDirectorySize(dirPath) {
  const files = await getAllFiles(dirPath);
  let totalSize = 0;
  
  for (const file of files) {
    const stats = await fsPromises.stat(file);
    totalSize += stats.size;
  }
  
  return totalSize;
}

getDirectorySize('.').then(size => {
  console.log(`Directory size: ${(size / 1024 / 1024).toFixed(2)} MB`);
});

// Safe file operations with error handling
async function safeFileOperation(filepath, operation) {
  try {
    return await operation(filepath);
  } catch (err) {
    if (err.code === 'ENOENT') {
      console.error('File not found:', filepath);
    } else if (err.code === 'EACCES') {
      console.error('Permission denied:', filepath);
    } else if (err.code === 'EISDIR') {
      console.error('Is a directory:', filepath);
    } else {
      console.error('File operation error:', err.message);
    }
    throw err;
  }
}

// Usage
safeFileOperation('file.txt', fsPromises.readFile);
```

---

**Tutorial continues with HTTP, Network Programming, Process Management, Clustering, Performance Optimization, and Production Best Practices...**

---

*This is Part 2 of the Node.js Complete Mastery Tutorial. See Part 1 (NodeJS_Complete_Mastery_Tutorial.md) for Chapters 1-4.*

*Total Coverage: V8 Engine, LibUV, Event Loop, Module Systems, Events, Streams, Buffers, File System Operations, and continuing with HTTP, Networking, and Advanced Topics.*
