# WebSockets and Socket.IO Complete Mastery Tutorial

## Table of Contents
1. [Introduction to WebSockets](#introduction-to-websockets)
2. [WebSocket Protocol Deep Dive](#websocket-protocol-deep-dive)
3. [Need for WebSockets](#need-for-websockets)
4. [Implementing WebSockets with Node.js](#implementing-websockets-with-nodejs)
5. [Introduction to Socket.IO](#introduction-to-socketio)
6. [Socket.IO Events and Acknowledgments](#socketio-events-and-acknowledgments)
7. [Rooms and Namespaces in Socket.IO](#rooms-and-namespaces-in-socketio)
8. [Real-Time Communication Patterns](#real-time-communication-patterns)
9. [Server-Sent Events vs Polling vs WebSockets](#server-sent-events-vs-polling-vs-websockets)
10. [Advanced Socket.IO Patterns](#advanced-socketio-patterns)
11. [Performance Optimization](#performance-optimization)
12. [Security Best Practices](#security-best-practices)
13. [Production Deployment](#production-deployment)
14. [Real-World Projects](#real-world-projects)

---

## Introduction to WebSockets

### What are WebSockets?

WebSockets provide a **full-duplex communication channel** over a single TCP connection between client and server. Unlike HTTP, which follows a request-response model, WebSockets enable **bi-directional, real-time communication**.

**Key Characteristics:**
- Persistent connection between client and server
- Low latency communication
- Full-duplex (simultaneous two-way communication)
- Built on TCP protocol
- Uses ws:// or wss:// (secure) protocol

### HTTP vs WebSocket

```
HTTP (Request-Response):
Client  -->  [Request]   -->  Server
Client  <--  [Response]  <--  Server
(Connection closes after each request-response cycle)

WebSocket (Full-Duplex):
Client  <==  [Persistent Connection]  ==>  Server
(Both can send messages anytime over same connection)
```

### WebSocket Lifecycle

```javascript
// WebSocket connection lifecycle stages:
// 1. Handshake (HTTP Upgrade)
// 2. Open Connection
// 3. Data Transfer (bidirectional)
// 4. Close Connection

/*
State Diagram:
CONNECTING (0) --> OPEN (1) --> CLOSING (2) --> CLOSED (3)
*/
```

---

## WebSocket Protocol Deep Dive

### The WebSocket Handshake

The WebSocket connection begins with an **HTTP upgrade request**:

**Client Request:**
```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: http://example.com
```

**Server Response:**
```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

**How the Handshake Works:**
1. Client sends HTTP request with `Upgrade: websocket` header
2. Client sends a random `Sec-WebSocket-Key` (base64 encoded)
3. Server concatenates key with magic string `258EAFA5-E914-47DA-95CA-C5AB0DC85B11`
4. Server hashes result with SHA-1 and base64 encodes it
5. Server returns `Sec-WebSocket-Accept` with computed value
6. Connection upgrades from HTTP to WebSocket protocol

### WebSocket Frame Structure

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
|N|V|V|V|       |S|             |   (if payload len==126/127)   |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
|     Extended payload length continued, if payload len == 127  |
+ - - - - - - - - - - - - - - - +-------------------------------+
|                               |Masking-key, if MASK set to 1  |
+-------------------------------+-------------------------------+
| Masking-key (continued)       |          Payload Data         |
+-------------------------------- - - - - - - - - - - - - - - - +
:                     Payload Data continued ...                :
+ - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
|                     Payload Data continued ...                |
+---------------------------------------------------------------+
```

**Frame Components:**
- **FIN**: Indicates final fragment (1 = final, 0 = more fragments coming)
- **Opcode**: Frame type (0x1 = text, 0x2 = binary, 0x8 = close, 0x9 = ping, 0xA = pong)
- **MASK**: Client-to-server frames must be masked
- **Payload length**: Size of payload data

---

## Need for WebSockets

### Traditional HTTP Limitations

**Problem 1: Latency**
```javascript
// HTTP Polling - Inefficient repeated requests
setInterval(() => {
  fetch('/api/updates')  // New TCP connection each time
    .then(res => res.json())
    .then(data => updateUI(data));
}, 1000);  // Polls every second - wasteful if no updates
```

**Problem 2: Server Cannot Push Data**
```javascript
// HTTP: Server waits for client request
// Real-time apps need instant updates from server
// Examples: chat, live sports scores, stock prices
```

**Problem 3: Overhead**
```javascript
// Each HTTP request carries headers (~500-800 bytes)
// For small messages, overhead > actual data
// Example: Sending "Hi" (2 bytes) with 800 bytes of headers
```

### When to Use WebSockets

✅ **Use WebSockets for:**
- Chat applications
- Live collaborative editing (Google Docs style)
- Real-time gaming
- Live sports scores/stock tickers
- IoT device communication
- Live notifications
- Real-time dashboards

❌ **Don't Use WebSockets for:**
- Simple CRUD operations
- File uploads
- RESTful APIs
- SEO-dependent content
- One-time data fetches

### WebSocket Benefits

```javascript
// Benefit 1: Low Latency
// One persistent connection vs. multiple HTTP requests
// Message delivery: ~1-2ms vs HTTP polling: ~500-1000ms

// Benefit 2: Reduced Bandwidth
// No repeated headers for each message
// WebSocket frame overhead: 2-6 bytes vs HTTP headers: 500-800 bytes

// Benefit 3: Bi-directional
// Server can push data without client request
// Client can send data without waiting for response

// Benefit 4: Stateful Connection
// Server maintains context of conversation
// No need to re-authenticate each request
```

---

## Implementing WebSockets with Node.js

### Using Native WebSocket API (ws Library)

**Installation:**
```bash
npm install ws
```

**Basic WebSocket Server:**
```javascript
// server.js - Native WebSocket Server
const WebSocket = require('ws');

// Create WebSocket server on port 8080
const wss = new WebSocket.Server({ port: 8080 });

// Track connected clients
let clients = new Set();

// Handle new connection
wss.on('connection', (ws, req) => {
  console.log('New client connected');
  clients.add(ws);
  
  // Get client IP address
  const clientIp = req.socket.remoteAddress;
  console.log(`Client IP: ${clientIp}`);
  
  // Send welcome message
  ws.send(JSON.stringify({
    type: 'welcome',
    message: 'Connected to WebSocket server',
    timestamp: Date.now()
  }));
  
  // Handle incoming messages
  ws.on('message', (data) => {
    console.log('Received:', data.toString());
    
    try {
      const message = JSON.parse(data);
      
      // Broadcast to all connected clients
      clients.forEach(client => {
        if (client.readyState === WebSocket.OPEN) {
          client.send(JSON.stringify({
            type: 'broadcast',
            data: message,
            timestamp: Date.now()
          }));
        }
      });
    } catch (error) {
      console.error('Invalid JSON:', error);
    }
  });
  
  // Handle connection close
  ws.on('close', () => {
    console.log('Client disconnected');
    clients.delete(ws);
  });
  
  // Handle errors
  ws.on('error', (error) => {
    console.error('WebSocket error:', error);
    clients.delete(ws);
  });
  
  // Ping-pong for connection health check
  ws.isAlive = true;
  ws.on('pong', () => {
    ws.isAlive = true;
  });
});

// Heartbeat to detect broken connections
const interval = setInterval(() => {
  wss.clients.forEach((ws) => {
    if (ws.isAlive === false) {
      return ws.terminate();  // Kill dead connection
    }
    
    ws.isAlive = false;
    ws.ping();  // Send ping, expect pong
  });
}, 30000);  // Every 30 seconds

// Clean up on server shutdown
wss.on('close', () => {
  clearInterval(interval);
});

console.log('WebSocket server running on ws://localhost:8080');
```

**Client-Side Implementation (Browser):**
```html
<!DOCTYPE html>
<html>
<head>
  <title>WebSocket Client</title>
</head>
<body>
  <h1>WebSocket Chat</h1>
  <div id="messages"></div>
  <input type="text" id="messageInput" placeholder="Type a message">
  <button onclick="sendMessage()">Send</button>
  
  <script>
    // Create WebSocket connection
    const ws = new WebSocket('ws://localhost:8080');
    
    // Connection opened
    ws.addEventListener('open', (event) => {
      console.log('Connected to server');
      addMessage('System', 'Connected to WebSocket server');
    });
    
    // Listen for messages
    ws.addEventListener('message', (event) => {
      console.log('Message from server:', event.data);
      
      try {
        const data = JSON.parse(event.data);
        
        if (data.type === 'welcome') {
          addMessage('Server', data.message);
        } else if (data.type === 'broadcast') {
          addMessage('User', JSON.stringify(data.data));
        }
      } catch (error) {
        addMessage('Server', event.data);
      }
    });
    
    // Connection closed
    ws.addEventListener('close', (event) => {
      console.log('Disconnected from server');
      addMessage('System', 'Disconnected from server');
    });
    
    // Error handling
    ws.addEventListener('error', (error) => {
      console.error('WebSocket error:', error);
      addMessage('Error', 'Connection error occurred');
    });
    
    // Send message function
    function sendMessage() {
      const input = document.getElementById('messageInput');
      const message = input.value.trim();
      
      if (message && ws.readyState === WebSocket.OPEN) {
        ws.send(JSON.stringify({
          text: message,
          timestamp: Date.now()
        }));
        input.value = '';
      }
    }
    
    // Add message to UI
    function addMessage(sender, text) {
      const messagesDiv = document.getElementById('messages');
      const messageElement = document.createElement('div');
      messageElement.textContent = `${sender}: ${text}`;
      messagesDiv.appendChild(messageElement);
    }
    
    // Send on Enter key
    document.getElementById('messageInput').addEventListener('keypress', (e) => {
      if (e.key === 'Enter') {
        sendMessage();
      }
    });
  </script>
</body>
</html>
```

**WebSocket Client States:**
```javascript
// WebSocket.readyState values:
const CONNECTING = 0;  // Connection not yet established
const OPEN = 1;        // Connection is open and ready
const CLOSING = 2;     // Connection is in process of closing
const CLOSED = 3;      // Connection is closed or couldn't open

// Check connection state before sending
if (ws.readyState === WebSocket.OPEN) {
  ws.send('Hello Server');
}
```

---

## Introduction to Socket.IO

### What is Socket.IO?

Socket.IO is a **library built on top of WebSockets** that provides:
- Automatic reconnection
- Fallback to HTTP long-polling
- Event-based messaging
- Room and namespace support
- Acknowledgment callbacks
- Binary data support
- Broadcasting capabilities

**Socket.IO vs Native WebSockets:**
```javascript
// Native WebSocket: Raw protocol, manual handling
ws.send(JSON.stringify({ type: 'chat', message: 'Hello' }));

// Socket.IO: Event-based, automatic serialization
socket.emit('chat', { message: 'Hello' });
```

### Socket.IO Architecture

```
Client                          Server
  |                                |
  |  HTTP Handshake                |
  |------------------------------->|
  |  (Try WebSocket upgrade)       |
  |                                |
  |<-------------------------------|
  |  WebSocket Connection          |
  |  (or fall back to polling)     |
  |                                |
  |  emit('event', data)           |
  |------------------------------->|
  |                                |
  |<-------------------------------|
  |  emit('response', data)        |
  |                                |
```

### Basic Socket.IO Server

**Installation:**
```bash
npm install socket.io express
```

**Server Setup:**
```javascript
// server.js - Socket.IO Server with Express
const express = require('express');
const { createServer } = require('http');
const { Server } = require('socket.io');

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, {
  cors: {
    origin: '*',  // Configure properly in production
    methods: ['GET', 'POST']
  }
});

// Serve static files
app.use(express.static('public'));

// Track online users
const onlineUsers = new Map();

// Handle Socket.IO connections
io.on('connection', (socket) => {
  console.log(`User connected: ${socket.id}`);
  
  // Handle user join
  socket.on('join', (username) => {
    onlineUsers.set(socket.id, username);
    
    // Notify all clients
    io.emit('user-joined', {
      username,
      userId: socket.id,
      onlineCount: onlineUsers.size
    });
    
    console.log(`${username} joined (Total: ${onlineUsers.size})`);
  });
  
  // Handle chat messages
  socket.on('chat-message', (data) => {
    const username = onlineUsers.get(socket.id) || 'Anonymous';
    
    // Broadcast to all clients including sender
    io.emit('message', {
      username,
      text: data.text,
      timestamp: Date.now()
    });
  });
  
  // Handle private messages
  socket.on('private-message', ({ to, message }) => {
    const from = onlineUsers.get(socket.id);
    
    // Send to specific socket
    socket.to(to).emit('private-message', {
      from,
      message,
      timestamp: Date.now()
    });
  });
  
  // Handle typing indicator
  socket.on('typing', () => {
    const username = onlineUsers.get(socket.id);
    socket.broadcast.emit('user-typing', username);
  });
  
  // Handle disconnect
  socket.on('disconnect', () => {
    const username = onlineUsers.get(socket.id);
    onlineUsers.delete(socket.id);
    
    if (username) {
      io.emit('user-left', {
        username,
        onlineCount: onlineUsers.size
      });
      
      console.log(`${username} left (Total: ${onlineUsers.size})`);
    }
  });
});

const PORT = process.env.PORT || 3000;
httpServer.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

**Client Implementation:**
```html
<!DOCTYPE html>
<html>
<head>
  <title>Socket.IO Chat</title>
  <script src="/socket.io/socket.io.js"></script>
  <style>
    #messages { height: 400px; overflow-y: scroll; border: 1px solid #ccc; padding: 10px; }
    .message { margin: 5px 0; }
    .system { color: #666; font-style: italic; }
  </style>
</head>
<body>
  <h1>Socket.IO Chat Room</h1>
  
  <div id="login">
    <input type="text" id="username" placeholder="Enter username">
    <button onclick="joinChat()">Join</button>
  </div>
  
  <div id="chat" style="display: none;">
    <div id="messages"></div>
    <div id="typing"></div>
    <input type="text" id="messageInput" placeholder="Type a message">
    <button onclick="sendMessage()">Send</button>
    <div id="onlineCount"></div>
  </div>
  
  <script>
    // Connect to Socket.IO server
    const socket = io('http://localhost:3000');
    let currentUsername = '';
    
    // Connection events
    socket.on('connect', () => {
      console.log('Connected with ID:', socket.id);
    });
    
    socket.on('disconnect', () => {
      console.log('Disconnected from server');
      addSystemMessage('Disconnected from server');
    });
    
    // Join chat function
    function joinChat() {
      const username = document.getElementById('username').value.trim();
      if (username) {
        currentUsername = username;
        socket.emit('join', username);
        document.getElementById('login').style.display = 'none';
        document.getElementById('chat').style.display = 'block';
      }
    }
    
    // User joined event
    socket.on('user-joined', (data) => {
      addSystemMessage(`${data.username} joined the chat`);
      updateOnlineCount(data.onlineCount);
    });
    
    // User left event
    socket.on('user-left', (data) => {
      addSystemMessage(`${data.username} left the chat`);
      updateOnlineCount(data.onlineCount);
    });
    
    // Message received
    socket.on('message', (data) => {
      addMessage(data.username, data.text, new Date(data.timestamp));
    });
    
    // Typing indicator
    socket.on('user-typing', (username) => {
      const typingDiv = document.getElementById('typing');
      typingDiv.textContent = `${username} is typing...`;
      setTimeout(() => { typingDiv.textContent = ''; }, 2000);
    });
    
    // Send message
    function sendMessage() {
      const input = document.getElementById('messageInput');
      const text = input.value.trim();
      
      if (text) {
        socket.emit('chat-message', { text });
        input.value = '';
      }
    }
    
    // Typing indicator
    document.getElementById('messageInput').addEventListener('input', () => {
      socket.emit('typing');
    });
    
    // Send on Enter
    document.getElementById('messageInput').addEventListener('keypress', (e) => {
      if (e.key === 'Enter') sendMessage();
    });
    
    // UI helper functions
    function addMessage(username, text, timestamp) {
      const messagesDiv = document.getElementById('messages');
      const messageDiv = document.createElement('div');
      messageDiv.className = 'message';
      messageDiv.textContent = `[${timestamp.toLocaleTimeString()}] ${username}: ${text}`;
      messagesDiv.appendChild(messageDiv);
      messagesDiv.scrollTop = messagesDiv.scrollHeight;
    }
    
    function addSystemMessage(text) {
      const messagesDiv = document.getElementById('messages');
      const messageDiv = document.createElement('div');
      messageDiv.className = 'message system';
      messageDiv.textContent = text;
      messagesDiv.appendChild(messageDiv);
      messagesDiv.scrollTop = messagesDiv.scrollHeight;
    }
    
    function updateOnlineCount(count) {
      document.getElementById('onlineCount').textContent = `Online: ${count}`;
    }
  </script>
</body>
</html>
```

---

## Socket.IO Events and Acknowledgments

### Event Emitters and Listeners

**Basic Event Pattern:**
```javascript
// Server-side
io.on('connection', (socket) => {
  // Listen for custom events
  socket.on('custom-event', (data) => {
    console.log('Received:', data);
  });
  
  // Emit events
  socket.emit('welcome', { message: 'Hello!' });
});

// Client-side
socket.on('welcome', (data) => {
  console.log(data.message);
});

socket.emit('custom-event', { foo: 'bar' });
```

### Acknowledgments (Callbacks)

**Acknowledgments allow request-response patterns:**

```javascript
// Server-side - Event with acknowledgment
socket.on('save-data', (data, callback) => {
  // Process data
  const success = saveToDatabase(data);
  
  // Send acknowledgment
  if (success) {
    callback({ status: 'ok', id: 123 });
  } else {
    callback({ status: 'error', message: 'Save failed' });
  }
});

// Client-side - Send with callback
socket.emit('save-data', { name: 'John' }, (response) => {
  if (response.status === 'ok') {
    console.log('Data saved with ID:', response.id);
  } else {
    console.error('Error:', response.message);
  }
});
```

**Timeout for Acknowledgments:**
```javascript
// Socket.IO v4+ supports timeout
socket.timeout(5000).emit('request-data', (err, response) => {
  if (err) {
    console.error('Request timed out');
  } else {
    console.log('Response:', response);
  }
});
```

### Broadcasting Patterns

```javascript
// 1. Send to all clients including sender
io.emit('event', data);

// 2. Send to all clients except sender
socket.broadcast.emit('event', data);

// 3. Send to specific client
io.to(socketId).emit('event', data);

// 4. Send to all clients in a room
io.to('room-name').emit('event', data);

// 5. Send to all clients in room except sender
socket.to('room-name').emit('event', data);

// 6. Send to multiple rooms
io.to('room1').to('room2').emit('event', data);

// 7. Send to all clients in namespace
io.of('/namespace').emit('event', data);
```

**Complete Broadcasting Example:**
```javascript
// server.js - Broadcasting demo
io.on('connection', (socket) => {
  
  // Example 1: Broadcast typing indicator
  socket.on('typing', () => {
    socket.broadcast.emit('user-typing', socket.id);
  });
  
  // Example 2: Send to specific user
  socket.on('send-pm', ({ to, message }) => {
    io.to(to).emit('private-message', {
      from: socket.id,
      message
    });
  });
  
  // Example 3: Broadcast to room members
  socket.on('room-message', ({ room, message }) => {
    socket.to(room).emit('room-message', {
      from: socket.id,
      message,
      room
    });
  });
  
  // Example 4: Broadcast to everyone
  socket.on('announcement', (message) => {
    io.emit('announcement', {
      message,
      timestamp: Date.now()
    });
  });
});
```

### Volatile Events

**Volatile events are dropped if client is not ready:**
```javascript
// Server - Send volatile event (won't buffer if client disconnected)
socket.volatile.emit('position-update', { x: 100, y: 200 });

// Use case: Real-time game positions
// If client misses one position update, it's okay - next one arrives soon
setInterval(() => {
  socket.volatile.emit('game-state', getCurrentGameState());
}, 16);  // 60 FPS
```

---

## Rooms and Namespaces in Socket.IO

### Understanding Rooms

**Rooms are arbitrary channels that sockets can join and leave.** They're useful for:
- Chat rooms
- Game lobbies
- User-specific channels
- Topic-based groups

**Room Operations:**
```javascript
// Server-side room management
io.on('connection', (socket) => {
  
  // Join a room
  socket.join('room-name');
  
  // Join multiple rooms
  socket.join(['room1', 'room2', 'room3']);
  
  // Leave a room
  socket.leave('room-name');
  
  // Get rooms socket is in
  console.log(socket.rooms);  // Set { socket.id, 'room-name' }
  
  // Get all sockets in a room
  const socketsInRoom = await io.in('room-name').fetchSockets();
  console.log(`Room has ${socketsInRoom.length} members`);
  
  // Send to all in room
  io.to('room-name').emit('message', 'Hello room!');
  
  // Send to all in room except sender
  socket.to('room-name').emit('message', 'Hello others!');
});
```

**Complete Chat Room Implementation:**
```javascript
// server.js - Multi-room chat
const rooms = new Map();  // Track room metadata

io.on('connection', (socket) => {
  let currentRoom = null;
  let username = null;
  
  // Join room
  socket.on('join-room', ({ room, user }) => {
    username = user;
    currentRoom = room;
    
    // Leave previous room if any
    if (socket.rooms.size > 1) {
      const prevRoom = Array.from(socket.rooms)[1];
      socket.leave(prevRoom);
    }
    
    // Join new room
    socket.join(room);
    
    // Track room data
    if (!rooms.has(room)) {
      rooms.set(room, { users: new Set(), messages: [] });
    }
    rooms.get(room).users.add(socket.id);
    
    // Notify room members
    io.to(room).emit('user-joined', {
      username,
      room,
      userCount: rooms.get(room).users.size
    });
    
    // Send room history to new user
    socket.emit('room-history', rooms.get(room).messages);
  });
  
  // Room message
  socket.on('room-message', (message) => {
    if (currentRoom) {
      const msg = {
        username,
        text: message,
        timestamp: Date.now()
      };
      
      // Store message
      rooms.get(currentRoom).messages.push(msg);
      
      // Broadcast to room
      io.to(currentRoom).emit('message', msg);
    }
  });
  
  // Leave room on disconnect
  socket.on('disconnect', () => {
    if (currentRoom && rooms.has(currentRoom)) {
      rooms.get(currentRoom).users.delete(socket.id);
      
      io.to(currentRoom).emit('user-left', {
        username,
        room: currentRoom,
        userCount: rooms.get(currentRoom).users.size
      });
      
      // Clean up empty rooms
      if (rooms.get(currentRoom).users.size === 0) {
        rooms.delete(currentRoom);
      }
    }
  });
});
```

**Client-side Room Usage:**
```javascript
// Join a chat room
socket.emit('join-room', { room: 'javascript', user: 'Alice' });

// Listen for room events
socket.on('user-joined', (data) => {
  console.log(`${data.username} joined ${data.room}`);
});

socket.on('room-history', (messages) => {
  messages.forEach(msg => displayMessage(msg));
});

socket.on('message', (msg) => {
  displayMessage(msg);
});

// Send message to current room
socket.emit('room-message', 'Hello everyone!');
```

### Understanding Namespaces

**Namespaces provide separate communication channels with their own event handlers.**

**When to Use Namespaces:**
- Separate concerns (e.g., /admin, /user, /notifications)
- Multi-tenant applications
- Different authorization levels
- Isolated event spaces

**Creating Namespaces:**
```javascript
// server.js - Multiple namespaces
const io = require('socket.io')(httpServer);

// Default namespace (/)
io.on('connection', (socket) => {
  console.log('Connected to default namespace');
});

// Admin namespace (/admin)
const adminNamespace = io.of('/admin');
adminNamespace.on('connection', (socket) => {
  console.log('Admin connected');
  
  socket.on('admin-action', (data) => {
    // Only admin namespace receives this
    console.log('Admin action:', data);
  });
});

// User namespace (/user)
const userNamespace = io.of('/user');
userNamespace.on('connection', (socket) => {
  console.log('User connected');
  
  socket.on('user-action', (data) => {
    console.log('User action:', data);
  });
});

// Notifications namespace (/notifications)
const notifNamespace = io.of('/notifications');
notifNamespace.on('connection', (socket) => {
  socket.on('subscribe', (topics) => {
    topics.forEach(topic => socket.join(topic));
  });
});

// Broadcast to specific namespace
adminNamespace.emit('admin-announcement', 'New feature deployed');
```

**Client-side Namespace Connection:**
```javascript
// Connect to different namespaces
const defaultSocket = io('http://localhost:3000');
const adminSocket = io('http://localhost:3000/admin');
const userSocket = io('http://localhost:3000/user');
const notifSocket = io('http://localhost:3000/notifications');

// Each namespace has independent events
defaultSocket.on('message', (data) => {
  console.log('Default:', data);
});

adminSocket.on('admin-announcement', (data) => {
  console.log('Admin:', data);
});

notifSocket.emit('subscribe', ['orders', 'messages']);
notifSocket.on('notification', (data) => {
  showNotification(data);
});
```

### Dynamic Namespaces

```javascript
// Create namespaces dynamically
io.of(/^\/dynamic-\w+$/).on('connection', (socket) => {
  const namespace = socket.nsp;  // Get namespace instance
  console.log(`Connected to ${namespace.name}`);
  
  socket.on('message', (data) => {
    // Broadcast within this dynamic namespace
    namespace.emit('message', data);
  });
});

// Client connects to dynamic namespace
const dynamicSocket = io('http://localhost:3000/dynamic-room1');
```

### Rooms + Namespaces Combined

```javascript
// Namespaces and rooms work together
const gameNamespace = io.of('/game');

gameNamespace.on('connection', (socket) => {
  // Join game lobby (room) within game namespace
  socket.on('join-lobby', (lobbyId) => {
    socket.join(lobbyId);
    
    // Send to all in this lobby
    gameNamespace.to(lobbyId).emit('player-joined', socket.id);
  });
  
  // Game action within lobby
  socket.on('game-action', ({ lobby, action }) => {
    gameNamespace.to(lobby).emit('action-performed', {
      player: socket.id,
      action
    });
  });
});
```

---

## Real-Time Communication Patterns

### Pattern 1: Request-Response

```javascript
// Client requests data, server responds
// Server
socket.on('get-user-data', (userId, callback) => {
  const userData = database.getUser(userId);
  callback(userData);
});

// Client
socket.emit('get-user-data', 123, (userData) => {
  console.log('User:', userData);
});
```

### Pattern 2: Pub-Sub (Publish-Subscribe)

```javascript
// Server acts as message broker
const subscribers = new Map();  // topic -> Set of sockets

io.on('connection', (socket) => {
  // Subscribe to topics
  socket.on('subscribe', (topics) => {
    topics.forEach(topic => {
      if (!subscribers.has(topic)) {
        subscribers.set(topic, new Set());
      }
      subscribers.get(topic).add(socket);
    });
  });
  
  // Publish to topic
  socket.on('publish', ({ topic, message }) => {
    if (subscribers.has(topic)) {
      subscribers.get(topic).forEach(subscriber => {
        subscriber.emit('message', { topic, message });
      });
    }
  });
  
  // Unsubscribe
  socket.on('unsubscribe', (topics) => {
    topics.forEach(topic => {
      if (subscribers.has(topic)) {
        subscribers.get(topic).delete(socket);
      }
    });
  });
  
  // Cleanup on disconnect
  socket.on('disconnect', () => {
    subscribers.forEach(sockets => sockets.delete(socket));
  });
});
```

### Pattern 3: Broadcast with Filters

```javascript
// Broadcast to users matching criteria
io.on('connection', (socket) => {
  socket.userData = {};  // Store user metadata
  
  socket.on('set-metadata', (data) => {
    socket.userData = data;
  });
  
  socket.on('broadcast-filtered', ({ filter, message }) => {
    io.sockets.sockets.forEach((s) => {
      // Check if socket matches filter
      if (matchesFilter(s.userData, filter)) {
        s.emit('filtered-message', message);
      }
    });
  });
});

function matchesFilter(userData, filter) {
  return Object.keys(filter).every(key => userData[key] === filter[key]);
}

// Example: Send to all premium users in US
socket.emit('broadcast-filtered', {
  filter: { tier: 'premium', country: 'US' },
  message: 'Special offer for premium US users!'
});
```

### Pattern 4: Presence and Status

```javascript
// Track user online status
const onlineUsers = new Map();

io.on('connection', (socket) => {
  let userId = null;
  
  socket.on('auth', (id) => {
    userId = id;
    onlineUsers.set(userId, {
      socketId: socket.id,
      status: 'online',
      lastSeen: Date.now()
    });
    
    // Notify others user is online
    socket.broadcast.emit('user-online', userId);
  });
  
  // Update status
  socket.on('status-change', (status) => {
    if (onlineUsers.has(userId)) {
      onlineUsers.get(userId).status = status;
      io.emit('user-status', { userId, status });
    }
  });
  
  socket.on('disconnect', () => {
    if (userId && onlineUsers.has(userId)) {
      onlineUsers.delete(userId);
      io.emit('user-offline', userId);
    }
  });
});
```

### Pattern 5: Message Queue with Persistence

```javascript
// Queue messages for offline users
const messageQueue = new Map();  // userId -> messages[]

io.on('connection', (socket) => {
  let userId = null;
  
  socket.on('auth', (id) => {
    userId = id;
    
    // Send queued messages
    if (messageQueue.has(userId)) {
      const messages = messageQueue.get(userId);
      socket.emit('queued-messages', messages);
      messageQueue.delete(userId);
    }
  });
  
  socket.on('send-message', ({ to, message }) => {
    const recipient = findSocket(to);
    
    if (recipient) {
      // User online - send immediately
      recipient.emit('message', { from: userId, message });
    } else {
      // User offline - queue message
      if (!messageQueue.has(to)) {
        messageQueue.set(to, []);
      }
      messageQueue.get(to).push({
        from: userId,
        message,
        timestamp: Date.now()
      });
    }
  });
});
```

---

## Server-Sent Events vs Polling vs WebSockets

### Comparison Table

| Feature | WebSockets | Server-Sent Events (SSE) | Long Polling | Short Polling |
|---------|-----------|-------------------------|--------------|---------------|
| **Direction** | Bi-directional | Server → Client only | Bi-directional | Bi-directional |
| **Protocol** | ws:// or wss:// | HTTP/HTTPS | HTTP/HTTPS | HTTP/HTTPS |
| **Connection** | Persistent | Persistent | Persistent | New per request |
| **Overhead** | Low (2-6 bytes/frame) | Low | Medium | High |
| **Browser Support** | Modern browsers | Modern browsers | All browsers | All browsers |
| **Complexity** | Medium | Low | Medium | Low |
| **Use Case** | Real-time bi-directional | Server push only | Legacy support | Simple updates |

### Short Polling

**Client repeatedly requests updates at intervals:**

```javascript
// Client - Short Polling
function pollServer() {
  fetch('/api/updates')
    .then(res => res.json())
    .then(data => {
      updateUI(data);
    })
    .catch(err => console.error(err));
}

// Poll every 5 seconds
setInterval(pollServer, 5000);

// Problems:
// - Many unnecessary requests
// - High server load
// - Delayed updates (up to polling interval)
// - Bandwidth waste
```

**Server:**
```javascript
app.get('/api/updates', (req, res) => {
  const updates = getLatestUpdates();
  res.json(updates);
});

// Every client polls every N seconds
// 1000 clients × every 5 seconds = 200 requests/second
```

### Long Polling

**Client requests, server holds until data available:**

```javascript
// Client - Long Polling
function longPoll() {
  fetch('/api/long-poll')
    .then(res => res.json())
    .then(data => {
      updateUI(data);
      longPoll();  // Immediately reconnect
    })
    .catch(err => {
      setTimeout(longPoll, 5000);  // Retry on error
    });
}

longPoll();  // Start polling
```

**Server:**
```javascript
// Server - Long Polling
const waitingClients = [];

app.get('/api/long-poll', (req, res) => {
  // Hold request until data available
  waitingClients.push(res);
  
  // Timeout after 30 seconds
  setTimeout(() => {
    const index = waitingClients.indexOf(res);
    if (index > -1) {
      waitingClients.splice(index, 1);
      res.json({ type: 'timeout' });
    }
  }, 30000);
});

// When new data arrives
function notifyClients(data) {
  waitingClients.forEach(res => {
    res.json(data);
  });
  waitingClients.length = 0;
}

// Example: Notify when new message arrives
messageEmitter.on('new-message', (message) => {
  notifyClients({ type: 'message', data: message });
});
```

### Server-Sent Events (SSE)

**One-way server-to-client streaming over HTTP:**

```javascript
// Server - SSE
app.get('/events', (req, res) => {
  // Set SSE headers
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  
  // Send initial data
  res.write('data: Connected\n\n');
  
  // Send updates periodically
  const interval = setInterval(() => {
    const data = {
      timestamp: Date.now(),
      value: Math.random()
    };
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  }, 1000);
  
  // Cleanup on close
  req.on('close', () => {
    clearInterval(interval);
  });
});
```

**Client - SSE:**
```javascript
// Client - EventSource API
const eventSource = new EventSource('/events');

eventSource.onopen = () => {
  console.log('SSE connection opened');
};

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received:', data);
  updateUI(data);
};

eventSource.onerror = (error) => {
  console.error('SSE error:', error);
  // Browser automatically reconnects
};

// Custom event types
eventSource.addEventListener('custom-event', (event) => {
  console.log('Custom:', event.data);
});

// Close connection
eventSource.close();
```

**SSE with Named Events:**
```javascript
// Server - Named events
app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  
  // Send event with custom name
  res.write('event: notification\n');
  res.write('data: {"message": "New notification"}\n\n');
  
  // Send event with ID (for reconnection)
  res.write('id: 123\n');
  res.write('event: update\n');
  res.write('data: {"status": "updated"}\n\n');
  
  // Multiline data
  res.write('event: article\n');
  res.write('data: Line 1\n');
  res.write('data: Line 2\n');
  res.write('data: Line 3\n\n');
});
```

### When to Use Each Technology

**Use WebSockets when:**
- ✅ Bi-directional communication needed
- ✅ Real-time gaming
- ✅ Chat applications
- ✅ Collaborative editing
- ✅ Low-latency critical

**Use Server-Sent Events when:**
- ✅ Server-to-client updates only
- ✅ Live notifications
- ✅ News feeds
- ✅ Stock price updates
- ✅ Simpler than WebSockets

**Use Long Polling when:**
- ✅ Legacy browser support needed
- ✅ Firewall restrictions
- ✅ WebSocket not available

**Use Short Polling when:**
- ✅ Updates not time-critical
- ✅ Very simple implementation needed
- ✅ Low update frequency

### Performance Comparison

```javascript
// Bandwidth comparison for 1000 clients, 1 message/second

// Short Polling (5 second interval)
// - Requests/second: 200
// - Data per request: ~500 bytes (headers)
// - Bandwidth: 100 KB/s (even with no updates!)

// Long Polling
// - Active connections: 1000
// - Bandwidth: ~2 KB/s (only when data sent)
// - Server memory: Medium (hold connections)

// SSE
// - Active connections: 1000
// - Bandwidth: ~1.5 KB/s
// - Server memory: Low
// - One-way only

// WebSockets
// - Active connections: 1000
// - Bandwidth: ~0.5 KB/s (minimal frame overhead)
// - Server memory: Low
// - Bi-directional
```

---

## Advanced Socket.IO Patterns

### Middleware

**Socket.IO middleware intercepts connections:**

```javascript
// Authentication middleware
io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  
  if (isValidToken(token)) {
    // Attach user data to socket
    socket.userId = getUserIdFromToken(token);
    next();  // Allow connection
  } else {
    next(new Error('Authentication failed'));
  }
});

// Logging middleware
io.use((socket, next) => {
  console.log(`Connection attempt from ${socket.handshake.address}`);
  next();
});

// Rate limiting middleware
const connectionAttempts = new Map();

io.use((socket, next) => {
  const ip = socket.handshake.address;
  const attempts = connectionAttempts.get(ip) || 0;
  
  if (attempts > 10) {
    return next(new Error('Too many connection attempts'));
  }
  
  connectionAttempts.set(ip, attempts + 1);
  setTimeout(() => {
    connectionAttempts.delete(ip);
  }, 60000);  // Reset after 1 minute
  
  next();
});
```

**Namespace Middleware:**
```javascript
const adminNamespace = io.of('/admin');

adminNamespace.use((socket, next) => {
  const user = socket.userId;
  
  if (isAdmin(user)) {
    next();
  } else {
    next(new Error('Admin access required'));
  }
});
```

### Adapters for Scaling

**Socket.IO adapter enables multi-server deployments:**

```javascript
// Install Redis adapter
// npm install @socket.io/redis-adapter redis

const { Server } = require('socket.io');
const { createAdapter } = require('@socket.io/redis-adapter');
const { createClient } = require('redis');

const io = new Server();

// Create Redis clients
const pubClient = createClient({ host: 'localhost', port: 6379 });
const subClient = pubClient.duplicate();

Promise.all([pubClient.connect(), subClient.connect()]).then(() => {
  io.adapter(createAdapter(pubClient, subClient));
  
  // Now events work across multiple servers
  io.emit('message', 'This reaches all servers');
});

/*
Architecture with Redis Adapter:

Server 1 (Socket.IO) ←→ Redis Pub/Sub ←→ Server 2 (Socket.IO)
    ↑                                         ↑
  Client A                                 Client B

When Server 1 emits: io.emit('event', data)
→ Publishes to Redis
→ Redis broadcasts to all servers
→ Server 2 sends to its clients
→ Both Client A and Client B receive event
*/
```

### Custom Adapter

```javascript
// Create custom adapter for PostgreSQL
class PostgresAdapter {
  constructor(nsp) {
    this.nsp = nsp;
    this.rooms = new Map();
    this.sids = new Map();
  }
  
  addAll(id, rooms) {
    // Add socket to rooms
    rooms.forEach(room => {
      if (!this.rooms.has(room)) {
        this.rooms.set(room, new Set());
      }
      this.rooms.get(room).add(id);
    });
    
    this.sids.set(id, new Set(rooms));
  }
  
  del(id, room) {
    // Remove socket from room
    if (this.rooms.has(room)) {
      this.rooms.get(room).delete(id);
    }
  }
  
  broadcast(packet, opts) {
    // Broadcast to rooms via PostgreSQL
    const rooms = opts.rooms;
    const except = opts.except || new Set();
    
    rooms.forEach(room => {
      const sockets = this.rooms.get(room);
      if (sockets) {
        sockets.forEach(socketId => {
          if (!except.has(socketId)) {
            // Send via PostgreSQL NOTIFY
            sendViaPostgres(socketId, packet);
          }
        });
      }
    });
  }
}
```

### Binary Data Handling

```javascript
// Send binary data (images, files, etc.)
socket.on('upload-file', (data) => {
  const { filename, buffer } = data;
  
  // buffer is a Buffer or ArrayBuffer
  fs.writeFile(`./uploads/${filename}`, buffer, (err) => {
    if (err) {
      socket.emit('upload-error', err.message);
    } else {
      socket.emit('upload-complete', { filename });
    }
  });
});

// Client - Send file
const fileInput = document.getElementById('file-input');
fileInput.addEventListener('change', (e) => {
  const file = e.target.files[0];
  const reader = new FileReader();
  
  reader.onload = () => {
    socket.emit('upload-file', {
      filename: file.name,
      buffer: reader.result  // ArrayBuffer
    });
  };
  
  reader.readAsArrayBuffer(file);
});
```

### Connection State Recovery

**Socket.IO can recover lost connections:**

```javascript
const io = new Server({
  connectionStateRecovery: {
    // Max disconnect duration for recovery
    maxDisconnectionDuration: 2 * 60 * 1000,  // 2 minutes
    
    // Skip middlewares on recovery
    skipMiddlewares: true
  }
});

io.on('connection', (socket) => {
  if (socket.recovered) {
    // Connection was recovered
    console.log('Recovered from temporary disconnect');
  } else {
    // New connection
    console.log('New connection');
  }
});
```

### Typed Events (TypeScript)

```typescript
// Define event types
interface ServerToClientEvents {
  message: (data: { text: string; from: string }) => void;
  userJoined: (username: string) => void;
  typing: (username: string) => void;
}

interface ClientToServerEvents {
  sendMessage: (text: string) => void;
  joinRoom: (room: string) => void;
}

// Typed Socket.IO
const io: Server<ClientToServerEvents, ServerToClientEvents> = new Server();

io.on('connection', (socket) => {
  // TypeScript knows available events
  socket.on('sendMessage', (text) => {
    // text is typed as string
    socket.emit('message', { text, from: 'Server' });
  });
});
```

---

## Performance Optimization

### Connection Pooling

```javascript
// Limit maximum connections per IP
const connectionsPerIP = new Map();
const MAX_CONNECTIONS_PER_IP = 5;

io.use((socket, next) => {
  const ip = socket.handshake.address;
  const count = connectionsPerIP.get(ip) || 0;
  
  if (count >= MAX_CONNECTIONS_PER_IP) {
    return next(new Error('Connection limit reached'));
  }
  
  connectionsPerIP.set(ip, count + 1);
  
  socket.on('disconnect', () => {
    const current = connectionsPerIP.get(ip) || 0;
    connectionsPerIP.set(ip, current - 1);
  });
  
  next();
});
```

### Message Batching

```javascript
// Batch multiple messages into one
class MessageBatcher {
  constructor(socket, interval = 100) {
    this.socket = socket;
    this.interval = interval;
    this.queue = [];
    this.timer = null;
  }
  
  send(event, data) {
    this.queue.push({ event, data });
    
    if (!this.timer) {
      this.timer = setTimeout(() => {
        this.flush();
      }, this.interval);
    }
  }
  
  flush() {
    if (this.queue.length > 0) {
      this.socket.emit('batch', this.queue);
      this.queue = [];
    }
    this.timer = null;
  }
}

// Usage
const batcher = new MessageBatcher(socket);
batcher.send('update', { x: 1 });
batcher.send('update', { x: 2 });
// Both sent together after 100ms
```

### Compression

```javascript
// Enable per-message compression
const io = new Server({
  perMessageDeflate: {
    threshold: 1024,  // Compress if message > 1KB
    zlibDeflateOptions: {
      chunkSize: 1024,
      memLevel: 7,
      level: 3
    }
  }
});
```

### Memory Management

```javascript
// Prevent memory leaks from large message history
class CircularBuffer {
  constructor(maxSize = 100) {
    this.buffer = [];
    this.maxSize = maxSize;
    this.index = 0;
  }
  
  push(item) {
    if (this.buffer.length < this.maxSize) {
      this.buffer.push(item);
    } else {
      this.buffer[this.index] = item;
      this.index = (this.index + 1) % this.maxSize;
    }
  }
  
  getAll() {
    return this.buffer;
  }
}

// Use for message history
const messageHistory = new CircularBuffer(100);

socket.on('message', (msg) => {
  messageHistory.push(msg);
  io.emit('message', msg);
});
```

### Load Balancing

```javascript
// Nginx configuration for WebSocket load balancing
/*
upstream socketio {
    ip_hash;  # Sticky sessions
    server backend1:3000;
    server backend2:3000;
    server backend3:3000;
}

server {
    listen 80;
    
    location /socket.io/ {
        proxy_pass http://socketio;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
*/
```

### Monitoring

```javascript
// Monitor connection stats
setInterval(() => {
  const stats = {
    connections: io.sockets.sockets.size,
    rooms: io.sockets.adapter.rooms.size,
    memory: process.memoryUsage()
  };
  
  console.log('Stats:', stats);
  
  // Send to monitoring service
  sendToMonitoring(stats);
}, 10000);

// Track event frequency
const eventCounts = new Map();

io.on('connection', (socket) => {
  socket.onAny((eventName) => {
    const count = eventCounts.get(eventName) || 0;
    eventCounts.set(eventName, count + 1);
  });
});
```

---

## Security Best Practices

### Authentication

```javascript
// JWT-based authentication
const jwt = require('jsonwebtoken');
const SECRET_KEY = process.env.JWT_SECRET;

// Middleware to verify JWT
io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  
  if (!token) {
    return next(new Error('Authentication token required'));
  }
  
  try {
    const decoded = jwt.verify(token, SECRET_KEY);
    socket.userId = decoded.userId;
    socket.username = decoded.username;
    next();
  } catch (err) {
    next(new Error('Invalid token'));
  }
});

// Client - Send token
const socket = io('http://localhost:3000', {
  auth: {
    token: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
  }
});
```

### Input Validation

```javascript
// Validate all incoming data
const Joi = require('joi');

const messageSchema = Joi.object({
  text: Joi.string().max(500).required(),
  room: Joi.string().alphanum().max(50)
});

socket.on('message', (data) => {
  const { error, value } = messageSchema.validate(data);
  
  if (error) {
    socket.emit('error', 'Invalid message format');
    return;
  }
  
  // Process validated data
  handleMessage(value);
});
```

### Rate Limiting

```javascript
// Rate limit events per socket
class RateLimiter {
  constructor(maxRequests, windowMs) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
    this.requests = new Map();
  }
  
  isAllowed(socketId) {
    const now = Date.now();
    const userRequests = this.requests.get(socketId) || [];
    
    // Remove old requests outside window
    const validRequests = userRequests.filter(
      time => now - time < this.windowMs
    );
    
    if (validRequests.length >= this.maxRequests) {
      return false;
    }
    
    validRequests.push(now);
    this.requests.set(socketId, validRequests);
    return true;
  }
}

const limiter = new RateLimiter(10, 60000);  // 10 requests per minute

socket.on('message', (data) => {
  if (!limiter.isAllowed(socket.id)) {
    socket.emit('error', 'Rate limit exceeded');
    return;
  }
  
  handleMessage(data);
});
```

### XSS Prevention

```javascript
// Sanitize user input
const createDOMPurify = require('dompurify');
const { JSDOM } = require('jsdom');
const window = new JSDOM('').window;
const DOMPurify = createDOMPurify(window);

socket.on('message', (data) => {
  // Sanitize HTML content
  const clean = DOMPurify.sanitize(data.text);
  
  io.emit('message', {
    username: socket.username,
    text: clean
  });
});
```

### CORS Configuration

```javascript
// Proper CORS setup
const io = new Server(httpServer, {
  cors: {
    origin: ['https://example.com', 'https://app.example.com'],
    methods: ['GET', 'POST'],
    credentials: true,
    allowedHeaders: ['Authorization']
  }
});
```

### SSL/TLS

```javascript
// Use secure WebSocket (wss://)
const fs = require('fs');
const https = require('https');

const server = https.createServer({
  key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem')
});

const io = new Server(server);

server.listen(443);

// Client connects with wss://
const socket = io('wss://example.com', {
  secure: true,
  rejectUnauthorized: true
});
```

### Content Security Policy

```javascript
// Set CSP headers
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; connect-src 'self' wss://example.com"
  );
  next();
});
```

---

## Production Deployment

### Environment Configuration

```javascript
// config.js
module.exports = {
  port: process.env.PORT || 3000,
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379
  },
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000']
  },
  jwtSecret: process.env.JWT_SECRET,
  logLevel: process.env.LOG_LEVEL || 'info'
};
```

### Process Management (PM2)

```bash
# Install PM2
npm install -g pm2

# Start with PM2
pm2 start server.js --name "socketio-server" -i max

# PM2 ecosystem file
```

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'socketio-server',
    script: './server.js',
    instances: 'max',  // Use all CPU cores
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss',
    merge_logs: true
  }]
};
```

### Docker Deployment

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  socketio:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - REDIS_HOST=redis
    depends_on:
      - redis
    restart: unless-stopped
    
  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    restart: unless-stopped
    
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - socketio
    restart: unless-stopped

volumes:
  redis-data:
```

### Logging

```javascript
// Use winston for production logging
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
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

io.on('connection', (socket) => {
  logger.info('New connection', {
    socketId: socket.id,
    userId: socket.userId,
    ip: socket.handshake.address
  });
  
  socket.on('error', (error) => {
    logger.error('Socket error', {
      socketId: socket.id,
      error: error.message,
      stack: error.stack
    });
  });
});
```

### Health Checks

```javascript
// Health check endpoint
app.get('/health', (req, res) => {
  const health = {
    uptime: process.uptime(),
    connections: io.sockets.sockets.size,
    memory: process.memoryUsage(),
    timestamp: Date.now()
  };
  
  res.status(200).json(health);
});

// Readiness check
app.get('/ready', async (req, res) => {
  try {
    // Check Redis connection
    await redisClient.ping();
    res.status(200).send('Ready');
  } catch (error) {
    res.status(503).send('Not ready');
  }
});
```

### Graceful Shutdown

```javascript
// server.js - Graceful shutdown
const server = httpServer.listen(PORT);

function gracefulShutdown(signal) {
  console.log(`${signal} received, closing server gracefully...`);
  
  // Stop accepting new connections
  server.close(() => {
    console.log('HTTP server closed');
  });
  
  // Close all Socket.IO connections
  io.close(() => {
    console.log('Socket.IO connections closed');
  });
  
  // Close database connections
  closeDatabase().then(() => {
    console.log('Database connections closed');
    process.exit(0);
  });
  
  // Force shutdown after timeout
  setTimeout(() => {
    console.error('Forced shutdown');
    process.exit(1);
  }, 10000);
}

process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));
```

### Nginx Configuration

```nginx
# nginx.conf
upstream socketio_backend {
    ip_hash;
    server backend1:3000;
    server backend2:3000;
    server backend3:3000;
}

server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    location /socket.io/ {
        proxy_pass http://socketio_backend;
        proxy_http_version 1.1;
        
        # WebSocket headers
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 7d;
        proxy_send_timeout 7d;
        proxy_read_timeout 7d;
    }
    
    location / {
        proxy_pass http://socketio_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## Real-World Projects

### Project 1: Real-Time Chat Application

**Features:**
- User authentication
- Multiple chat rooms
- Private messaging
- Typing indicators
- Message history
- Online user list
- File sharing

**Complete Implementation:**

```javascript
// server.js - Complete Chat Server
const express = require('express');
const { createServer } = require('http');
const { Server } = require('socket.io');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, {
  cors: { origin: '*' }
});

app.use(express.json());
app.use(express.static('public'));

// In-memory storage (use database in production)
const users = new Map();
const rooms = new Map();
const messages = new Map();
const onlineUsers = new Map();

const JWT_SECRET = 'your-secret-key';

// Authentication endpoints
app.post('/register', async (req, res) => {
  const { username, password } = req.body;
  
  if (users.has(username)) {
    return res.status(400).json({ error: 'Username exists' });
  }
  
  const hashedPassword = await bcrypt.hash(password, 10);
  users.set(username, {
    username,
    password: hashedPassword,
    createdAt: Date.now()
  });
  
  const token = jwt.sign({ username }, JWT_SECRET);
  res.json({ token, username });
});

app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = users.get(username);
  
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  const valid = await bcrypt.compare(password, user.password);
  if (!valid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  const token = jwt.sign({ username }, JWT_SECRET);
  res.json({ token, username });
});

// Socket.IO authentication middleware
io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  
  if (!token) {
    return next(new Error('Authentication required'));
  }
  
  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    socket.username = decoded.username;
    next();
  } catch (err) {
    next(new Error('Invalid token'));
  }
});

// Socket.IO connection handler
io.on('connection', (socket) => {
  console.log(`${socket.username} connected`);
  
  // Track online user
  onlineUsers.set(socket.username, socket.id);
  io.emit('online-users', Array.from(onlineUsers.keys()));
  
  // Join room
  socket.on('join-room', (roomName) => {
    socket.join(roomName);
    
    // Initialize room if needed
    if (!rooms.has(roomName)) {
      rooms.set(roomName, {
        name: roomName,
        members: new Set(),
        createdAt: Date.now()
      });
      messages.set(roomName, []);
    }
    
    const room = rooms.get(roomName);
    room.members.add(socket.username);
    
    // Send room history
    socket.emit('room-history', messages.get(roomName));
    
    // Notify room
    io.to(roomName).emit('user-joined-room', {
      username: socket.username,
      room: roomName,
      memberCount: room.members.size
    });
  });
  
  // Leave room
  socket.on('leave-room', (roomName) => {
    socket.leave(roomName);
    
    if (rooms.has(roomName)) {
      const room = rooms.get(roomName);
      room.members.delete(socket.username);
      
      io.to(roomName).emit('user-left-room', {
        username: socket.username,
        room: roomName,
        memberCount: room.members.size
      });
    }
  });
  
  // Room message
  socket.on('room-message', ({ room, text }) => {
    const message = {
      id: Date.now(),
      username: socket.username,
      text,
      room,
      timestamp: Date.now()
    };
    
    // Store message
    if (messages.has(room)) {
      messages.get(room).push(message);
    }
    
    // Broadcast to room
    io.to(room).emit('room-message', message);
  });
  
  // Private message
  socket.on('private-message', ({ to, text }) => {
    const message = {
      id: Date.now(),
      from: socket.username,
      to,
      text,
      timestamp: Date.now()
    };
    
    const recipientSocketId = onlineUsers.get(to);
    if (recipientSocketId) {
      // Send to recipient
      io.to(recipientSocketId).emit('private-message', message);
      // Send back to sender for confirmation
      socket.emit('private-message', message);
    } else {
      socket.emit('error', { message: 'User offline' });
    }
  });
  
  // Typing indicator
  socket.on('typing', ({ room }) => {
    socket.to(room).emit('user-typing', {
      username: socket.username,
      room
    });
  });
  
  // Stop typing
  socket.on('stop-typing', ({ room }) => {
    socket.to(room).emit('user-stop-typing', {
      username: socket.username,
      room
    });
  });
  
  // Get room list
  socket.on('get-rooms', () => {
    const roomList = Array.from(rooms.values()).map(room => ({
      name: room.name,
      memberCount: room.members.size,
      createdAt: room.createdAt
    }));
    socket.emit('room-list', roomList);
  });
  
  // Disconnect handler
  socket.on('disconnect', () => {
    console.log(`${socket.username} disconnected`);
    
    // Remove from online users
    onlineUsers.delete(socket.username);
    io.emit('online-users', Array.from(onlineUsers.keys()));
    
    // Remove from all rooms
    rooms.forEach((room, roomName) => {
      if (room.members.has(socket.username)) {
        room.members.delete(socket.username);
        io.to(roomName).emit('user-left-room', {
          username: socket.username,
          room: roomName,
          memberCount: room.members.size
        });
      }
    });
  });
});

const PORT = 3000;
httpServer.listen(PORT, () => {
  console.log(`Chat server running on http://localhost:${PORT}`);
});
```

**Client Implementation:**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Real-Time Chat</title>
  <script src="/socket.io/socket.io.js"></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: Arial, sans-serif; }
    
    #auth { max-width: 400px; margin: 100px auto; padding: 20px; }
    #auth input { width: 100%; padding: 10px; margin: 5px 0; }
    #auth button { width: 100%; padding: 10px; margin: 5px 0; }
    
    #app { display: none; height: 100vh; }
    #sidebar { width: 250px; background: #2c3e50; color: white; padding: 20px; float: left; height: 100%; }
    #main { margin-left: 250px; height: 100%; display: flex; flex-direction: column; }
    #messages { flex: 1; overflow-y: scroll; padding: 20px; }
    #input-area { padding: 20px; border-top: 1px solid #ddd; }
    #messageInput { width: calc(100% - 80px); padding: 10px; }
    #sendBtn { width: 70px; padding: 10px; }
    
    .message { margin: 10px 0; padding: 10px; background: #f5f5f5; border-radius: 5px; }
    .message.own { background: #dcf8c6; }
    .system { color: #666; font-style: italic; text-align: center; }
    .typing { color: #999; font-size: 12px; }
    
    .room { padding: 10px; margin: 5px 0; cursor: pointer; border-radius: 5px; }
    .room:hover { background: #34495e; }
    .room.active { background: #3498db; }
  </style>
</head>
<body>
  <!-- Authentication -->
  <div id="auth">
    <h2>Chat Login</h2>
    <input type="text" id="username" placeholder="Username">
    <input type="password" id="password" placeholder="Password">
    <button onclick="login()">Login</button>
    <button onclick="register()">Register</button>
  </div>
  
  <!-- Main App -->
  <div id="app">
    <div id="sidebar">
      <h3>Rooms</h3>
      <div id="roomList"></div>
      <hr>
      <h3>Online Users</h3>
      <div id="onlineList"></div>
    </div>
    
    <div id="main">
      <div id="messages"></div>
      <div class="typing" id="typing"></div>
      <div id="input-area">
        <input type="text" id="messageInput" placeholder="Type a message...">
        <button id="sendBtn" onclick="sendMessage()">Send</button>
      </div>
    </div>
  </div>
  
  <script>
    let socket;
    let currentRoom = null;
    let currentUser = null;
    let token = null;
    let typingTimeout = null;
    
    async function register() {
      const username = document.getElementById('username').value;
      const password = document.getElementById('password').value;
      
      const res = await fetch('/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password })
      });
      
      const data = await res.json();
      if (res.ok) {
        token = data.token;
        currentUser = data.username;
        connectSocket();
      } else {
        alert(data.error);
      }
    }
    
    async function login() {
      const username = document.getElementById('username').value;
      const password = document.getElementById('password').value;
      
      const res = await fetch('/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password })
      });
      
      const data = await res.json();
      if (res.ok) {
        token = data.token;
        currentUser = data.username;
        connectSocket();
      } else {
        alert(data.error);
      }
    }
    
    function connectSocket() {
      socket = io('http://localhost:3000', {
        auth: { token }
      });
      
      socket.on('connect', () => {
        console.log('Connected');
        document.getElementById('auth').style.display = 'none';
        document.getElementById('app').style.display = 'block';
        
        // Join default room
        socket.emit('join-room', 'general');
        currentRoom = 'general';
        socket.emit('get-rooms');
      });
      
      socket.on('room-list', (rooms) => {
        const roomList = document.getElementById('roomList');
        roomList.innerHTML = '';
        rooms.forEach(room => {
          const div = document.createElement('div');
          div.className = 'room';
          div.textContent = `${room.name} (${room.memberCount})`;
          div.onclick = () => joinRoom(room.name);
          if (room.name === currentRoom) div.classList.add('active');
          roomList.appendChild(div);
        });
      });
      
      socket.on('online-users', (users) => {
        const onlineList = document.getElementById('onlineList');
        onlineList.innerHTML = users.join('<br>');
      });
      
      socket.on('room-history', (msgs) => {
        document.getElementById('messages').innerHTML = '';
        msgs.forEach(msg => displayMessage(msg));
      });
      
      socket.on('room-message', displayMessage);
      
      socket.on('user-joined-room', (data) => {
        addSystemMessage(`${data.username} joined ${data.room}`);
      });
      
      socket.on('user-left-room', (data) => {
        addSystemMessage(`${data.username} left ${data.room}`);
      });
      
      socket.on('user-typing', (data) => {
        document.getElementById('typing').textContent = `${data.username} is typing...`;
      });
      
      socket.on('user-stop-typing', () => {
        document.getElementById('typing').textContent = '';
      });
    }
    
    function joinRoom(roomName) {
      if (currentRoom) {
        socket.emit('leave-room', currentRoom);
      }
      socket.emit('join-room', roomName);
      currentRoom = roomName;
      socket.emit('get-rooms');
    }
    
    function sendMessage() {
      const input = document.getElementById('messageInput');
      const text = input.value.trim();
      
      if (text && currentRoom) {
        socket.emit('room-message', { room: currentRoom, text });
        input.value = '';
        socket.emit('stop-typing', { room: currentRoom });
      }
    }
    
    function displayMessage(msg) {
      const messagesDiv = document.getElementById('messages');
      const div = document.createElement('div');
      div.className = 'message';
      if (msg.username === currentUser) div.classList.add('own');
      
      const time = new Date(msg.timestamp).toLocaleTimeString();
      div.innerHTML = `<strong>${msg.username}</strong> <small>${time}</small><br>${msg.text}`;
      
      messagesDiv.appendChild(div);
      messagesDiv.scrollTop = messagesDiv.scrollHeight;
    }
    
    function addSystemMessage(text) {
      const messagesDiv = document.getElementById('messages');
      const div = document.createElement('div');
      div.className = 'message system';
      div.textContent = text;
      messagesDiv.appendChild(div);
    }
    
    // Typing indicator
    document.addEventListener('DOMContentLoaded', () => {
      const input = document.getElementById('messageInput');
      
      input.addEventListener('input', () => {
        if (socket && currentRoom) {
          socket.emit('typing', { room: currentRoom });
          
          clearTimeout(typingTimeout);
          typingTimeout = setTimeout(() => {
            socket.emit('stop-typing', { room: currentRoom });
          }, 1000);
        }
      });
      
      input.addEventListener('keypress', (e) => {
        if (e.key === 'Enter') sendMessage();
      });
    });
  </script>
</body>
</html>
```

---

### Project 2: Collaborative Whiteboard

**Features:**
- Real-time drawing
- Multiple users
- Color selection
- Clear canvas
- User cursors

```javascript
// whiteboard-server.js
const express = require('express');
const { createServer } = require('http');
const { Server } = require('socket.io');

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer);

app.use(express.static('public'));

const canvasState = [];
const users = new Map();

io.on('connection', (socket) => {
  console.log('User connected:', socket.id);
  
  // Send current canvas state
  socket.emit('canvas-state', canvasState);
  
  // Send current users
  socket.emit('users', Array.from(users.values()));
  
  // Drawing event
  socket.on('draw', (data) => {
    canvasState.push(data);
    socket.broadcast.emit('draw', data);
  });
  
  // Clear canvas
  socket.on('clear', () => {
    canvasState.length = 0;
    io.emit('clear');
  });
  
  // User info
  socket.on('user-info', (info) => {
    users.set(socket.id, { ...info, id: socket.id });
    io.emit('users', Array.from(users.values()));
  });
  
  // Cursor position
  socket.on('cursor', (pos) => {
    socket.broadcast.emit('cursor', {
      id: socket.id,
      ...pos
    });
  });
  
  // Disconnect
  socket.on('disconnect', () => {
    users.delete(socket.id);
    io.emit('users', Array.from(users.values()));
    io.emit('cursor-leave', socket.id);
  });
});

httpServer.listen(3000, () => {
  console.log('Whiteboard server running on http://localhost:3000');
});
```

**Client (whiteboard.html):**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Collaborative Whiteboard</title>
  <script src="/socket.io/socket.io.js"></script>
  <style>
    body { margin: 0; overflow: hidden; }
    #canvas { border: 1px solid #000; cursor: crosshair; }
    #toolbar { padding: 10px; background: #f0f0f0; }
    #toolbar button, #toolbar input { margin: 5px; }
    #users { position: fixed; right: 10px; top: 60px; background: white; padding: 10px; border: 1px solid #ccc; }
    .cursor { position: absolute; width: 10px; height: 10px; border-radius: 50%; pointer-events: none; }
  </style>
</head>
<body>
  <div id="toolbar">
    <input type="color" id="colorPicker" value="#000000">
    <input type="range" id="sizePicker" min="1" max="20" value="2">
    <button onclick="clearCanvas()">Clear</button>
    <input type="text" id="nameInput" placeholder="Your name">
  </div>
  
  <canvas id="canvas"></canvas>
  
  <div id="users">
    <h4>Online Users</h4>
    <div id="userList"></div>
  </div>
  
  <div id="cursors"></div>
  
  <script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight - 50;
    
    const socket = io();
    let drawing = false;
    let currentColor = '#000000';
    let currentSize = 2;
    
    // Canvas setup
    ctx.lineCap = 'round';
    ctx.lineJoin = 'round';
    
    // Socket events
    socket.on('connect', () => {
      console.log('Connected');
    });
    
    socket.on('canvas-state', (state) => {
      state.forEach(drawLine);
    });
    
    socket.on('draw', drawLine);
    
    socket.on('clear', () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    });
    
    socket.on('users', (users) => {
      const userList = document.getElementById('userList');
      userList.innerHTML = users.map(u => u.name || 'Anonymous').join('<br>');
    });
    
    socket.on('cursor', (data) => {
      updateCursor(data.id, data.x, data.y, data.color);
    });
    
    socket.on('cursor-leave', (id) => {
      const cursor = document.getElementById(`cursor-${id}`);
      if (cursor) cursor.remove();
    });
    
    // Drawing functions
    function startDrawing(e) {
      drawing = true;
      draw(e);
    }
    
    function stopDrawing() {
      drawing = false;
      ctx.beginPath();
    }
    
    function draw(e) {
      if (!drawing) return;
      
      const rect = canvas.getBoundingClientRect();
      const x = e.clientX - rect.left;
      const y = e.clientY - rect.top;
      
      const data = {
        x0: ctx.lastX || x,
        y0: ctx.lastY || y,
        x1: x,
        y1: y,
        color: currentColor,
        size: currentSize
      };
      
      drawLine(data);
      socket.emit('draw', data);
      
      ctx.lastX = x;
      ctx.lastY = y;
    }
    
    function drawLine(data) {
      ctx.strokeStyle = data.color;
      ctx.lineWidth = data.size;
      ctx.beginPath();
      ctx.moveTo(data.x0, data.y0);
      ctx.lineTo(data.x1, data.y1);
      ctx.stroke();
    }
    
    function clearCanvas() {
      socket.emit('clear');
    }
    
    function updateCursor(id, x, y, color) {
      let cursor = document.getElementById(`cursor-${id}`);
      if (!cursor) {
        cursor = document.createElement('div');
        cursor.id = `cursor-${id}`;
        cursor.className = 'cursor';
        document.getElementById('cursors').appendChild(cursor);
      }
      cursor.style.left = x + 'px';
      cursor.style.top = y + 'px';
      cursor.style.background = color;
    }
    
    // Event listeners
    canvas.addEventListener('mousedown', startDrawing);
    canvas.addEventListener('mouseup', stopDrawing);
    canvas.addEventListener('mousemove', (e) => {
      draw(e);
      
      const rect = canvas.getBoundingClientRect();
      socket.emit('cursor', {
        x: e.clientX - rect.left,
        y: e.clientY - rect.top,
        color: currentColor
      });
    });
    canvas.addEventListener('mouseout', stopDrawing);
    
    document.getElementById('colorPicker').addEventListener('change', (e) => {
      currentColor = e.target.value;
    });
    
    document.getElementById('sizePicker').addEventListener('input', (e) => {
      currentSize = e.target.value;
    });
    
    document.getElementById('nameInput').addEventListener('change', (e) => {
      socket.emit('user-info', { name: e.target.value });
    });
    
    // Window resize
    window.addEventListener('resize', () => {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight - 50;
    });
  </script>
</body>
</html>
```

---

### Project 3: Live Gaming Leaderboard

```javascript
// leaderboard-server.js
const express = require('express');
const { createServer } = require('http');
const { Server } = require('socket.io');

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer);

app.use(express.static('public'));

// Game state
const players = new Map();
const gameRooms = new Map();

io.on('connection', (socket) => {
  console.log('Player connected:', socket.id);
  
  // Player joins game
  socket.on('join-game', ({ playerName, gameId }) => {
    // Create game room if not exists
    if (!gameRooms.has(gameId)) {
      gameRooms.set(gameId, {
        players: new Map(),
        started: false,
        startTime: null
      });
    }
    
    const game = gameRooms.get(gameId);
    
    // Add player to game
    const player = {
      id: socket.id,
      name: playerName,
      score: 0,
      level: 1,
      status: 'waiting'
    };
    
    game.players.set(socket.id, player);
    players.set(socket.id, { ...player, gameId });
    
    socket.join(gameId);
    
    // Send game state
    socket.emit('game-joined', {
      gameId,
      player
    });
    
    // Broadcast updated leaderboard
    broadcastLeaderboard(gameId);
  });
  
  // Start game
  socket.on('start-game', (gameId) => {
    if (gameRooms.has(gameId)) {
      const game = gameRooms.get(gameId);
      game.started = true;
      game.startTime = Date.now();
      
      game.players.forEach(player => {
        player.status = 'playing';
      });
      
      io.to(gameId).emit('game-started', {
        startTime: game.startTime
      });
      
      broadcastLeaderboard(gameId);
    }
  });
  
  // Update score
  socket.on('update-score', ({ points, level }) => {
    const playerData = players.get(socket.id);
    if (!playerData) return;
    
    const game = gameRooms.get(playerData.gameId);
    const player = game.players.get(socket.id);
    
    player.score += points;
    if (level) player.level = level;
    
    // Broadcast updated leaderboard
    broadcastLeaderboard(playerData.gameId);
    
    // Check for achievements
    checkAchievements(player, playerData.gameId);
  });
  
  // Game over
  socket.on('game-over', () => {
    const playerData = players.get(socket.id);
    if (!playerData) return;
    
    const game = gameRooms.get(playerData.gameId);
    const player = game.players.get(socket.id);
    player.status = 'finished';
    
    broadcastLeaderboard(playerData.gameId);
    
    // Check if all players finished
    const allFinished = Array.from(game.players.values())
      .every(p => p.status === 'finished');
    
    if (allFinished) {
      io.to(playerData.gameId).emit('all-finished', {
        winner: getWinner(game)
      });
    }
  });
  
  // Disconnect
  socket.on('disconnect', () => {
    const playerData = players.get(socket.id);
    if (playerData) {
      const game = gameRooms.get(playerData.gameId);
      if (game) {
        game.players.delete(socket.id);
        broadcastLeaderboard(playerData.gameId);
      }
      players.delete(socket.id);
    }
  });
});

function broadcastLeaderboard(gameId) {
  const game = gameRooms.get(gameId);
  if (!game) return;
  
  const leaderboard = Array.from(game.players.values())
    .sort((a, b) => b.score - a.score)
    .map((player, index) => ({
      rank: index + 1,
      ...player
    }));
  
  io.to(gameId).emit('leaderboard-update', leaderboard);
}

function checkAchievements(player, gameId) {
  const achievements = [];
  
  if (player.score >= 1000 && player.score < 1010) {
    achievements.push({ type: 'score', value: 1000 });
  }
  
  if (player.level >= 10 && player.level < 11) {
    achievements.push({ type: 'level', value: 10 });
  }
  
  if (achievements.length > 0) {
    io.to(gameId).emit('achievement', {
      player: player.name,
      achievements
    });
  }
}

function getWinner(game) {
  return Array.from(game.players.values())
    .sort((a, b) => b.score - a.score)[0];
}

httpServer.listen(3000, () => {
  console.log('Leaderboard server running on http://localhost:3000');
});
```

---

## Debugging and Troubleshooting

### Common Issues and Solutions

**1. CORS Errors**
```javascript
// Problem: CORS policy blocking WebSocket connection

// Solution: Configure CORS properly
const io = new Server(httpServer, {
  cors: {
    origin: ['http://localhost:3000', 'https://yourdomain.com'],
    methods: ['GET', 'POST'],
    credentials: true
  }
});
```

**2. Connection Not Upgrading**
```javascript
// Problem: Connection stuck on polling, not upgrading to WebSocket

// Check:
// 1. Server supports WebSocket (check Node.js version)
// 2. Proxy/Load balancer allows WebSocket upgrade
// 3. No firewall blocking WebSocket

// Force WebSocket only (debugging)
const socket = io('http://localhost:3000', {
  transports: ['websocket'],  // Skip polling
  upgrade: false
});
```

**3. Memory Leaks**
```javascript
// Problem: Memory usage keeps growing

// Solution: Clean up event listeners
socket.on('disconnect', () => {
  // Remove all listeners
  socket.removeAllListeners();
  
  // Clean up references
  users.delete(socket.id);
  rooms.forEach(room => room.members.delete(socket.id));
});

// Use weak references for large objects
const cache = new WeakMap();
```

**4. Events Not Firing**
```javascript
// Problem: Events emitted but not received

// Debug: Log all events
socket.onAny((eventName, ...args) => {
  console.log('Event:', eventName, args);
});

// Check event names match exactly (case-sensitive)
socket.emit('user-join');  // ❌ Won't match
socket.on('user-joined');  // ❌

socket.emit('user-joined');  // ✅
socket.on('user-joined');    // ✅
```

**5. Reconnection Issues**
```javascript
// Problem: Client doesn't reconnect after disconnect

// Configure reconnection
const socket = io('http://localhost:3000', {
  reconnection: true,
  reconnectionAttempts: 5,
  reconnectionDelay: 1000,
  reconnectionDelayMax: 5000
});

socket.on('disconnect', (reason) => {
  console.log('Disconnected:', reason);
  if (reason === 'io server disconnect') {
    // Server forcefully disconnected, manually reconnect
    socket.connect();
  }
});
```

### Debugging Tools

```javascript
// Enable Socket.IO debug mode
// Client-side
localStorage.debug = 'socket.io-client:socket';

// Server-side
process.env.DEBUG = 'socket.io:*';

// Then run: DEBUG=socket.io:* node server.js
```

**Chrome DevTools:**
1. Network tab → Filter: WS (WebSocket)
2. See all WebSocket frames sent/received
3. Inspect frame data

**Socket.IO Admin UI:**
```bash
npm install @socket.io/admin-ui
```

```javascript
const { instrument } = require('@socket.io/admin-ui');

instrument(io, {
  auth: false  // Enable authentication in production
});

// Access at: https://admin.socket.io
// Connect to: http://localhost:3000
```

---

## Testing WebSocket Applications

### Unit Testing with Jest

```bash
npm install --save-dev jest socket.io-client
```

```javascript
// server.test.js
const { createServer } = require('http');
const { Server } = require('socket.io');
const Client = require('socket.io-client');

describe('Socket.IO Server', () => {
  let io, serverSocket, clientSocket;
  
  beforeAll((done) => {
    const httpServer = createServer();
    io = new Server(httpServer);
    httpServer.listen(() => {
      const port = httpServer.address().port;
      clientSocket = new Client(`http://localhost:${port}`);
      
      io.on('connection', (socket) => {
        serverSocket = socket;
      });
      
      clientSocket.on('connect', done);
    });
  });
  
  afterAll(() => {
    io.close();
    clientSocket.close();
  });
  
  test('should communicate', (done) => {
    clientSocket.on('hello', (arg) => {
      expect(arg).toBe('world');
      done();
    });
    serverSocket.emit('hello', 'world');
  });
  
  test('should work with acknowledgements', (done) => {
    serverSocket.on('test-ack', (cb) => {
      cb('got it');
    });
    
    clientSocket.emit('test-ack', (arg) => {
      expect(arg).toBe('got it');
      done();
    });
  });
});
```

### Integration Testing

```javascript
// integration.test.js
const request = require('supertest');
const { app, io } = require('./server');
const Client = require('socket.io-client');

describe('Chat Integration Tests', () => {
  let client1, client2;
  
  beforeEach((done) => {
    client1 = new Client('http://localhost:3000');
    client2 = new Client('http://localhost:3000');
    
    let connected = 0;
    const checkDone = () => {
      connected++;
      if (connected === 2) done();
    };
    
    client1.on('connect', checkDone);
    client2.on('connect', checkDone);
  });
  
  afterEach(() => {
    client1.close();
    client2.close();
  });
  
  test('should broadcast message to other clients', (done) => {
    client2.on('message', (msg) => {
      expect(msg.text).toBe('Hello');
      done();
    });
    
    client1.emit('message', { text: 'Hello' });
  });
  
  test('should handle room join', (done) => {
    client2.on('user-joined', (data) => {
      expect(data.room).toBe('test-room');
      done();
    });
    
    client2.emit('join-room', 'test-room');
    setTimeout(() => {
      client1.emit('join-room', 'test-room');
    }, 100);
  });
});
```

### Load Testing

```javascript
// load-test.js using artillery
// npm install -g artillery

// Create config.yml
/*
config:
  target: "http://localhost:3000"
  phases:
    - duration: 60
      arrivalRate: 10
  engines:
    socketio: {}

scenarios:
  - name: "Connect and send messages"
    engine: socketio
    flow:
      - emit:
          channel: "join"
          data: "user"
      - think: 1
      - emit:
          channel: "message"
          data: "Hello"
      - think: 2
*/

// Run: artillery run config.yml
```

---

## Best Practices Summary

### ✅ Do's

1. **Always validate input**
   ```javascript
   socket.on('message', (data) => {
     if (typeof data !== 'string' || data.length > 500) {
       return socket.emit('error', 'Invalid message');
     }
     // Process message
   });
   ```

2. **Use namespaces for separation**
   ```javascript
   const adminNs = io.of('/admin');
   const userNs = io.of('/user');
   ```

3. **Implement authentication**
   ```javascript
   io.use((socket, next) => {
     const token = socket.handshake.auth.token;
     if (isValid(token)) next();
     else next(new Error('Auth failed'));
   });
   ```

4. **Handle errors gracefully**
   ```javascript
   socket.on('error', (error) => {
     logger.error('Socket error:', error);
     socket.emit('error', 'Something went wrong');
   });
   ```

5. **Clean up on disconnect**
   ```javascript
   socket.on('disconnect', () => {
     cleanupUserData(socket.id);
     socket.removeAllListeners();
   });
   ```

### ❌ Don'ts

1. **Don't trust client data**
   ```javascript
   // ❌ Bad
   socket.on('delete-user', (userId) => {
     database.deleteUser(userId);  // Anyone can delete any user!
   });
   
   // ✅ Good
   socket.on('delete-user', (userId) => {
     if (socket.userId === userId && hasPermission(socket.userId)) {
       database.deleteUser(userId);
     }
   });
   ```

2. **Don't store sensitive data in socket object**
   ```javascript
   // ❌ Bad
   socket.password = 'secret123';
   
   // ✅ Good
   socket.userId = 'user123';  // Reference only
   ```

3. **Don't emit in loops without batching**
   ```javascript
   // ❌ Bad
   for (let i = 0; i < 1000; i++) {
     socket.emit('update', i);
   }
   
   // ✅ Good
   socket.emit('batch-update', Array.from({ length: 1000 }, (_, i) => i));
   ```

4. **Don't forget to handle reconnection**
   ```javascript
   // ✅ Always configure reconnection
   const socket = io(url, {
     reconnection: true,
     reconnectionAttempts: 5
   });
   ```

5. **Don't use Socket.IO for large file transfers**
   ```javascript
   // ❌ Bad - Use HTTP upload instead
   socket.emit('file', hugeFileBuffer);
   
   // ✅ Good - Use dedicated upload endpoint
   await fetch('/upload', { method: 'POST', body: formData });
   ```

---

## Conclusion

WebSockets and Socket.IO enable powerful real-time applications with low latency and efficient bi-directional communication. This tutorial covered:

✅ WebSocket protocol fundamentals  
✅ Native WebSocket implementation  
✅ Socket.IO features and patterns  
✅ Rooms and namespaces  
✅ Real-time communication strategies  
✅ Performance optimization  
✅ Security best practices  
✅ Production deployment  
✅ Real-world project implementations  

### Next Steps

1. Build a chat application from scratch
2. Implement a real-time dashboard with live data
3. Create a multiplayer game using Socket.IO
4. Deploy to production with load balancing
5. Explore Socket.IO adapters for scaling (Redis, MongoDB)
6. Learn about WebRTC for peer-to-peer communication

### Resources

- [Socket.IO Documentation](https://socket.io/docs/)
- [WebSocket MDN Reference](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
- [RFC 6455 - WebSocket Protocol](https://tools.ietf.org/html/rfc6455)
- [Socket.IO GitHub](https://github.com/socketio/socket.io)

---

**Happy Real-Time Coding! 🚀**