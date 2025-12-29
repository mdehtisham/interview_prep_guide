# NestJS WebSockets - Top Interview Questions

## WebSocket Fundamentals

1. What are WebSockets and how do they work?

<details>
<summary><strong>Answer</strong></summary>

**WebSockets** are a communication protocol that provides **full-duplex (bidirectional)** communication channels over a single TCP connection. Unlike HTTP where the client initiates every request, WebSockets allow both the server and client to send messages independently at any time, making them ideal for real-time applications.

### **How WebSockets Work**

**1. WebSocket Handshake (HTTP Upgrade)**

WebSockets start with an HTTP handshake to upgrade the connection:

```typescript
// CLIENT: Initial HTTP request to upgrade to WebSocket
GET ws://example.com/chat HTTP/1.1
Host: example.com
Upgrade: websocket                    // Request protocol upgrade
Connection: Upgrade                    // Keep connection alive
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==  // Random key for security
Sec-WebSocket-Version: 13              // WebSocket protocol version

// SERVER: Response accepting the upgrade
HTTP/1.1 101 Switching Protocols       // 101 status = protocol switch
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=  // Hashed key response

// After handshake: Connection is now WebSocket, not HTTP!
// Both client and server can send messages freely
```

**2. Persistent Connection**

Once established, the WebSocket connection remains open:

```typescript
// TRADITIONAL HTTP (Request-Response cycle)
// Client must initiate every interaction
Client  -->  [Request]  -->  Server
Client  <--  [Response] <--  Server
// Connection closes after response

// Client wants new data? Must make another request
Client  -->  [Request]  -->  Server
Client  <--  [Response] <--  Server

// WEBSOCKET (Persistent bidirectional)
// Connection stays open after handshake
Client  <=========>  Server

// Both can send messages anytime:
Client  -->  [Message]  -->  Server   // Client to server
Client  <--  [Message]  <--  Server   // Server to client
Client  -->  [Message]  -->  Server   // Client again
Server  -->  [Message]  -->  Client   // Server initiates

// Connection stays open until explicitly closed
```

**3. Message Framing**

WebSocket messages are sent in frames (small data chunks):

```typescript
/*
WebSocket Frame Structure:

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

FIN bit: 1 = final fragment, 0 = more fragments coming
Opcode: 0x1 = text, 0x2 = binary, 0x8 = close, 0x9 = ping, 0xA = pong
Mask: Client messages must be masked (XOR with random key)
Payload: Actual message data
*/

// Example: Sending "Hello"
// Text frame with FIN=1, opcode=0x1 (text), payload="Hello"
```

### **WebSocket Lifecycle**

```typescript
// Complete WebSocket lifecycle example

// 1. CLIENT: Create WebSocket connection
const socket = new WebSocket('ws://localhost:3000');

// 2. CONNECTION OPENED
socket.addEventListener('open', (event) => {
  console.log('WebSocket connection established');
  
  // 3. SEND MESSAGE (Client to Server)
  socket.send(JSON.stringify({
    type: 'join',
    room: 'chat-room-1',
    username: 'John'
  }));
});

// 4. RECEIVE MESSAGE (Server to Client)
socket.addEventListener('message', (event) => {
  const data = JSON.parse(event.data);
  console.log('Message from server:', data);
  
  // Process message based on type
  if (data.type === 'chat') {
    displayChatMessage(data.message);
  } else if (data.type === 'notification') {
    showNotification(data.content);
  }
});

// 5. HANDLE ERRORS
socket.addEventListener('error', (error) => {
  console.error('WebSocket error:', error);
});

// 6. CONNECTION CLOSED
socket.addEventListener('close', (event) => {
  console.log('WebSocket connection closed:', event.code, event.reason);
  
  // Common close codes:
  // 1000 = Normal closure
  // 1001 = Going away (server shutdown or user navigation)
  // 1002 = Protocol error
  // 1006 = Abnormal closure (no close frame)
  // 1011 = Server error
  
  // Implement reconnection logic if needed
  if (event.code !== 1000) {
    setTimeout(() => reconnect(), 5000);
  }
});

// 7. CLOSE CONNECTION (when needed)
function closeConnection() {
  socket.close(1000, 'User logged out'); // Clean closure
}
```

### **NestJS WebSocket Example**

```typescript
// SERVER: NestJS WebSocket Gateway
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  OnGatewayDisconnect,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({ cors: true }) // Enable CORS for WebSocket
export class ChatGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server; // Socket.IO server instance

  // 1. HANDLE NEW CONNECTION
  handleConnection(client: Socket) {
    console.log(`Client connected: ${client.id}`);
    
    // Send welcome message to the new client
    client.emit('welcome', {
      message: 'Welcome to the chat!',
      clientId: client.id,
      timestamp: new Date()
    });
    
    // Notify all other clients about new user
    client.broadcast.emit('user-joined', {
      clientId: client.id,
      timestamp: new Date()
    });
  }

  // 2. HANDLE DISCONNECTION
  handleDisconnect(client: Socket) {
    console.log(`Client disconnected: ${client.id}`);
    
    // Notify all clients about user leaving
    this.server.emit('user-left', {
      clientId: client.id,
      timestamp: new Date()
    });
  }

  // 3. HANDLE INCOMING MESSAGES
  @SubscribeMessage('send-message')
  handleMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { message: string; room?: string }
  ) {
    console.log(`Message from ${client.id}:`, data.message);
    
    // Broadcast message to all clients (including sender)
    this.server.emit('new-message', {
      clientId: client.id,
      message: data.message,
      timestamp: new Date()
    });
    
    // Return acknowledgment to sender
    return {
      success: true,
      messageId: Date.now(),
      timestamp: new Date()
    };
  }

  // 4. HANDLE JOIN ROOM
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { room: string }
  ) {
    client.join(data.room);
    console.log(`Client ${client.id} joined room: ${data.room}`);
    
    // Notify everyone in the room
    this.server.to(data.room).emit('user-joined-room', {
      clientId: client.id,
      room: data.room,
      timestamp: new Date()
    });
    
    return { success: true, room: data.room };
  }

  // 5. SEND MESSAGE TO SPECIFIC ROOM
  @SubscribeMessage('send-room-message')
  handleRoomMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { room: string; message: string }
  ) {
    // Send message only to clients in the specified room
    this.server.to(data.room).emit('room-message', {
      clientId: client.id,
      room: data.room,
      message: data.message,
      timestamp: new Date()
    });
    
    return { success: true };
  }

  // 6. SEND MESSAGE TO SPECIFIC CLIENT
  sendToClient(clientId: string, event: string, data: any) {
    this.server.to(clientId).emit(event, data);
  }

  // 7. BROADCAST TO ALL EXCEPT SENDER
  @SubscribeMessage('typing')
  handleTyping(@ConnectedSocket() client: Socket) {
    // Broadcast to everyone except the sender
    client.broadcast.emit('user-typing', {
      clientId: client.id,
      timestamp: new Date()
    });
  }
}
```

### **Production-Grade Client Implementation**

```typescript
// CLIENT: TypeScript WebSocket client with reconnection
export class WebSocketClient {
  private socket: WebSocket | null = null;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 10;
  private reconnectDelay = 1000; // Start with 1 second
  private messageQueue: any[] = []; // Queue messages when disconnected
  private isConnecting = false;
  private listeners: Map<string, Function[]> = new Map();

  constructor(private url: string) {}

  /**
   * Establish WebSocket connection with automatic reconnection
   */
  connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      if (this.socket?.readyState === WebSocket.OPEN) {
        resolve();
        return;
      }

      if (this.isConnecting) {
        reject(new Error('Connection already in progress'));
        return;
      }

      this.isConnecting = true;
      this.socket = new WebSocket(this.url);

      // Connection opened
      this.socket.onopen = (event) => {
        console.log('[WebSocket] Connected');
        this.isConnecting = false;
        this.reconnectAttempts = 0;
        this.reconnectDelay = 1000;
        
        // Send queued messages
        this.flushMessageQueue();
        
        resolve();
      };

      // Message received
      this.socket.onmessage = (event) => {
        try {
          const data = JSON.parse(event.data);
          console.log('[WebSocket] Message received:', data);
          
          // Trigger event listeners
          this.emit(data.event || 'message', data);
        } catch (error) {
          console.error('[WebSocket] Failed to parse message:', error);
        }
      };

      // Connection error
      this.socket.onerror = (error) => {
        console.error('[WebSocket] Error:', error);
        this.isConnecting = false;
        reject(error);
      };

      // Connection closed
      this.socket.onclose = (event) => {
        console.log('[WebSocket] Connection closed:', event.code, event.reason);
        this.isConnecting = false;
        this.socket = null;
        
        // Attempt reconnection if not a clean close
        if (event.code !== 1000 && this.reconnectAttempts < this.maxReconnectAttempts) {
          this.scheduleReconnect();
        }
      };
    });
  }

  /**
   * Send message (queues if disconnected)
   */
  send(event: string, data: any): void {
    const message = { event, data, timestamp: new Date() };
    
    if (this.socket?.readyState === WebSocket.OPEN) {
      this.socket.send(JSON.stringify(message));
    } else {
      console.warn('[WebSocket] Not connected, queueing message');
      this.messageQueue.push(message);
    }
  }

  /**
   * Listen to specific events
   */
  on(event: string, callback: Function): void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event)!.push(callback);
  }

  /**
   * Remove event listener
   */
  off(event: string, callback: Function): void {
    const listeners = this.listeners.get(event);
    if (listeners) {
      const index = listeners.indexOf(callback);
      if (index !== -1) {
        listeners.splice(index, 1);
      }
    }
  }

  /**
   * Emit event to all listeners
   */
  private emit(event: string, data: any): void {
    const listeners = this.listeners.get(event);
    if (listeners) {
      listeners.forEach(callback => callback(data));
    }
  }

  /**
   * Schedule reconnection with exponential backoff
   */
  private scheduleReconnect(): void {
    this.reconnectAttempts++;
    console.log(
      `[WebSocket] Reconnecting in ${this.reconnectDelay}ms (attempt ${this.reconnectAttempts}/${this.maxReconnectAttempts})`
    );
    
    setTimeout(() => {
      this.connect().catch(error => {
        console.error('[WebSocket] Reconnection failed:', error);
      });
    }, this.reconnectDelay);
    
    // Exponential backoff: 1s, 2s, 4s, 8s, ..., max 30s
    this.reconnectDelay = Math.min(this.reconnectDelay * 2, 30000);
  }

  /**
   * Send all queued messages
   */
  private flushMessageQueue(): void {
    while (this.messageQueue.length > 0 && this.socket?.readyState === WebSocket.OPEN) {
      const message = this.messageQueue.shift();
      this.socket.send(JSON.stringify(message));
    }
  }

  /**
   * Close connection gracefully
   */
  disconnect(): void {
    if (this.socket) {
      this.socket.close(1000, 'Client disconnected');
      this.socket = null;
    }
  }

  /**
   * Check connection status
   */
  isConnected(): boolean {
    return this.socket?.readyState === WebSocket.OPEN;
  }
}

// Usage example
const wsClient = new WebSocketClient('ws://localhost:3000');

await wsClient.connect();

// Listen to events
wsClient.on('new-message', (data) => {
  console.log('New message:', data);
});

wsClient.on('user-joined', (data) => {
  console.log('User joined:', data);
});

// Send messages
wsClient.send('send-message', { message: 'Hello World!' });
wsClient.send('join-room', { room: 'general' });
```

### **Key Concepts**

1. **Full-Duplex Communication**: Both client and server can send messages independently
2. **Persistent Connection**: Connection remains open (unlike HTTP request-response)
3. **Low Latency**: No need to establish new connection for each message
4. **Real-Time**: Instant message delivery (no polling required)
5. **Lightweight**: Small frame overhead compared to HTTP headers

**Interview Tip**: WebSockets provide **full-duplex, persistent connections** over TCP. They start with an **HTTP handshake** (GET request with `Upgrade: websocket` header) to upgrade to WebSocket protocol (101 Switching Protocols). Once established, **both client and server can send messages anytime** without request-response cycle. Messages are sent in **frames** (text/binary/control frames). Perfect for **real-time applications** (chat, notifications, live updates) where server needs to **push data to clients** without polling. Key difference from HTTP: **persistent bidirectional** vs **request-response**. Production considerations: implement **reconnection logic**, **message queuing** during disconnects, **exponential backoff**, and **heartbeat/ping-pong** to detect dead connections.

</details>

<details>
<summary><strong>Answer</strong></summary>

[Previous Q1 answer content will be here]

</details>

2. What is the difference between WebSockets and HTTP?

<details>
<summary><strong>Answer</strong></summary>

**WebSockets** and **HTTP** are both communication protocols, but they work fundamentally differently. HTTP is **request-response** (client-initiated, stateless), while WebSockets provide **persistent, bidirectional** communication (both client and server can initiate).

### **Key Differences**

| Feature | HTTP | WebSockets |
|---------|------|------------|
| **Communication Pattern** | Request-Response (half-duplex) | Full-Duplex (bidirectional) |
| **Connection** | New connection per request | Persistent connection |
| **Initiated By** | Always client | Both client and server |
| **State** | Stateless | Stateful |
| **Overhead** | Large headers (500-800 bytes) | Small frames (2-6 bytes) |
| **Protocol** | HTTP/1.1, HTTP/2, HTTP/3 | WebSocket (RFC 6455) |
| **URL Scheme** | `http://` or `https://` | `ws://` or `wss://` |
| **Latency** | Higher (new connection each time) | Lower (reuses connection) |
| **Use Case** | Traditional web pages, REST APIs | Real-time apps (chat, gaming, live data) |

### **1. Communication Pattern**

```typescript
// HTTP: Request-Response (Client initiates every interaction)
// CLIENT:
await fetch('https://api.example.com/users');
// 1. Client opens connection
// 2. Client sends HTTP request
// 3. Server processes request
// 4. Server sends HTTP response
// 5. Connection closes

// Want another update? Client must request again:
await fetch('https://api.example.com/users'); // New connection!

// WEBSOCKET: Bidirectional (Both can send anytime)
// CLIENT:
const socket = new WebSocket('ws://api.example.com');

// 1. Initial handshake (HTTP upgrade)
// 2. Connection stays open

// Client can send:
socket.send(JSON.stringify({ type: 'message', text: 'Hello' }));

// Server can send (without client request):
socket.onmessage = (event) => {
  console.log('Server pushed:', event.data); // No client request needed!
};

// Both can send messages anytime, connection stays open
```

### **2. Connection Lifecycle**

```typescript
// HTTP: Short-lived connections
class HttpClient {
  async getUser(id: string) {
    // 1. Establish TCP connection
    // 2. Send HTTP GET request
    const response = await fetch(`http://api.example.com/users/${id}`);
    const user = await response.json();
    // 3. Connection closes automatically
    return user;
  }
  
  async getOrders() {
    // NEW connection for each request!
    const response = await fetch('http://api.example.com/orders');
    const orders = await response.json();
    return orders;
  }
}

// WEBSOCKET: Long-lived persistent connection
class WebSocketClient {
  private socket: WebSocket;
  
  connect() {
    // 1. Establish WebSocket connection (once)
    this.socket = new WebSocket('ws://api.example.com');
    
    this.socket.onopen = () => {
      console.log('Connected - connection stays open');
    };
  }
  
  getUser(id: string) {
    // Reuse existing connection - no new connection overhead!
    this.socket.send(JSON.stringify({ type: 'getUser', id }));
  }
  
  getOrders() {
    // Same connection - very efficient!
    this.socket.send(JSON.stringify({ type: 'getOrders' }));
  }
  
  // Connection stays open until explicitly closed
  disconnect() {
    this.socket.close();
  }
}
```

### **3. Server-Initiated Communication**

```typescript
// HTTP: Server CANNOT send data without client request
// SERVER (NestJS):
@Controller('notifications')
export class NotificationsController {
  @Get()
  getNotifications(@Query('userId') userId: string) {
    // Server can only respond to client requests
    // If new notification arrives, server CANNOT push to client
    // Client must poll (repeatedly request) to get updates
    return this.notificationService.getNotifications(userId);
  }
}

// CLIENT: Must poll every few seconds to check for updates
setInterval(async () => {
  const response = await fetch('http://api.example.com/notifications?userId=123');
  const notifications = await response.json();
  // Inefficient: Most requests return empty (no new notifications)
}, 5000); // Poll every 5 seconds

// WEBSOCKET: Server CAN push data to client anytime
// SERVER (NestJS):
@WebSocketGateway()
export class NotificationsGateway {
  @WebSocketServer()
  server: Server;
  
  // Server can push notification without client request!
  sendNotification(userId: string, notification: any) {
    this.server.to(userId).emit('new-notification', notification);
    // Client receives immediately - no polling needed!
  }
}

// CLIENT: Receives notifications instantly
const socket = new WebSocket('ws://api.example.com');
socket.onmessage = (event) => {
  const notification = JSON.parse(event.data);
  displayNotification(notification); // Instant delivery!
};
```

### **4. Overhead Comparison**

```typescript
// HTTP: Large overhead per request
/*
HTTP REQUEST (sending "Hello"):

GET /api/messages HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 ...
Accept: application/json
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Cookie: session=abc123...
Content-Type: application/json
Content-Length: 17

{"message":"Hello"}

Total: ~500-800 bytes of headers + 17 bytes payload = 517-817 bytes
*/

// WEBSOCKET: Minimal overhead per message
/*
WEBSOCKET FRAME (sending "Hello"):

[FIN=1][opcode=1][mask=1][payload_len=17][mask_key=4bytes][payload]

Total: ~6 bytes of frame header + 17 bytes payload = 23 bytes

Savings: 94-97% less overhead!
*/

// Example: Sending 1000 messages
const messageSize = 17; // "Hello" in JSON
const httpOverhead = 800 + messageSize; // ~817 bytes per message
const wsOverhead = 6 + messageSize;     // ~23 bytes per message

const totalHttpBytes = 817 * 1000;      // 817,000 bytes (817 KB)
const totalWsBytes = 23 * 1000;         // 23,000 bytes (23 KB)

console.log(`HTTP: ${totalHttpBytes} bytes`);
console.log(`WebSocket: ${totalWsBytes} bytes`);
console.log(`Savings: ${((1 - totalWsBytes/totalHttpBytes) * 100).toFixed(1)}%`);
// Output: Savings: 97.2%
```

### **5. NestJS Implementation Comparison**

```typescript
// HTTP REST API (Traditional)
@Controller('chat')
export class ChatController {
  constructor(private chatService: ChatService) {}
  
  // Client must poll to get new messages
  @Get('messages')
  async getMessages(
    @Query('roomId') roomId: string,
    @Query('since') since: number // Last message timestamp
  ) {
    // Returns messages since last check
    return this.chatService.getMessagesSince(roomId, since);
  }
  
  // Client sends message via POST
  @Post('messages')
  async sendMessage(@Body() dto: { roomId: string; message: string }) {
    const message = await this.chatService.createMessage(dto);
    // Server CANNOT notify other users - they must poll!
    return message;
  }
}

// CLIENT: Must poll for new messages
let lastMessageTime = Date.now();

setInterval(async () => {
  // Poll every 3 seconds
  const response = await fetch(
    `http://api.example.com/chat/messages?roomId=room1&since=${lastMessageTime}`
  );
  const newMessages = await response.json();
  
  if (newMessages.length > 0) {
    displayMessages(newMessages);
    lastMessageTime = newMessages[newMessages.length - 1].timestamp;
  }
}, 3000);
// Problems:
// - 3 second delay for new messages
// - Wastes bandwidth (most polls return empty)
// - Server load (all clients polling constantly)

// WEBSOCKET (Real-time)
@WebSocketGateway({ cors: true })
export class ChatGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;
  
  constructor(private chatService: ChatService) {}
  
  handleConnection(client: Socket) {
    console.log(`Client connected: ${client.id}`);
  }
  
  // Client joins chat room
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { roomId: string }
  ) {
    client.join(data.roomId);
    return { success: true };
  }
  
  // Client sends message
  @SubscribeMessage('send-message')
  async handleMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { roomId: string; message: string }
  ) {
    // Save message to database
    const message = await this.chatService.createMessage(data);
    
    // Instantly push to ALL users in room (no polling!)
    this.server.to(data.roomId).emit('new-message', {
      id: message.id,
      message: data.message,
      senderId: client.id,
      timestamp: new Date()
    });
    
    return { success: true, messageId: message.id };
  }
}

// CLIENT: Receives messages instantly
const socket = io('http://localhost:3000');

// Join room
socket.emit('join-room', { roomId: 'room1' });

// Listen for new messages (instant delivery!)
socket.on('new-message', (data) => {
  displayMessage(data); // No delay, no polling!
});

// Send message
socket.emit('send-message', { roomId: 'room1', message: 'Hello!' });
```

### **6. Stateful vs Stateless**

```typescript
// HTTP: Stateless (each request independent)
@Controller('cart')
export class CartController {
  // Each request must include authentication and context
  @Get()
  async getCart(@Headers('authorization') token: string) {
    // Must validate token on EVERY request
    const userId = await this.authService.validateToken(token);
    return this.cartService.getCart(userId);
  }
  
  @Post('add')
  async addItem(
    @Headers('authorization') token: string,
    @Body() item: any
  ) {
    // Must validate token AGAIN
    const userId = await this.authService.validateToken(token);
    return this.cartService.addItem(userId, item);
  }
}

// WEBSOCKET: Stateful (connection maintains state)
@WebSocketGateway()
export class CartGateway implements OnGatewayConnection {
  // Store user context per connection
  private userSessions = new Map<string, { userId: string; authenticated: boolean }>();
  
  async handleConnection(client: Socket) {
    // Authenticate ONCE during connection
    const token = client.handshake.auth.token;
    const userId = await this.authService.validateToken(token);
    
    if (userId) {
      this.userSessions.set(client.id, { userId, authenticated: true });
      console.log(`User ${userId} authenticated`);
    } else {
      client.disconnect();
    }
  }
  
  @SubscribeMessage('get-cart')
  handleGetCart(@ConnectedSocket() client: Socket) {
    // No need to validate token - already authenticated
    const session = this.userSessions.get(client.id);
    return this.cartService.getCart(session.userId);
  }
  
  @SubscribeMessage('add-item')
  handleAddItem(
    @ConnectedSocket() client: Socket,
    @MessageBody() item: any
  ) {
    // No need to validate token - already authenticated
    const session = this.userSessions.get(client.id);
    return this.cartService.addItem(session.userId, item);
  }
  
  handleDisconnect(client: Socket) {
    this.userSessions.delete(client.id);
  }
}
```

### **7. When to Use Each**

```typescript
// ✅ USE HTTP FOR:
// - Traditional CRUD operations
// - RESTful APIs
// - File uploads/downloads
// - Caching scenarios
// - Simple request-response
// - Public APIs (easier to consume)

@Controller('users')
export class UsersController {
  @Get(':id')        // Good: Simple data fetch
  @Post()            // Good: Create resource
  @Put(':id')        // Good: Update resource
  @Delete(':id')     // Good: Delete resource
}

// ✅ USE WEBSOCKETS FOR:
// - Real-time chat
// - Live notifications
// - Collaborative editing
// - Live sports scores
// - Stock tickers
// - Online gaming
// - Live dashboards
// - IoT sensor data

@WebSocketGateway()
export class RealTimeGateway {
  @SubscribeMessage('join-chat')     // Good: Real-time chat
  @SubscribeMessage('live-update')   // Good: Push updates
  @SubscribeMessage('game-move')     // Good: Gaming
}

// ❌ DON'T USE WEBSOCKETS FOR:
// - Simple CRUD operations (overkill)
// - File transfers (HTTP better)
// - SEO-critical pages (crawlers use HTTP)
// - Infrequent updates (HTTP more efficient)
```

### **8. Latency Comparison**

```typescript
// Scenario: Send 100 small messages

// HTTP: Each message requires new connection
const httpLatency = [
  // Message 1:
  50,   // TCP handshake
  50,   // TLS handshake (if HTTPS)
  10,   // Request
  10,   // Response
  // Total: 120ms per message
];

const totalHttpTime = 120 * 100; // 12,000ms (12 seconds)

// WEBSOCKET: Reuse connection
const wsLatency = [
  // Initial connection:
  50,   // TCP handshake
  50,   // TLS handshake (if WSS)
  20,   // WebSocket upgrade
  // Total: 120ms once
  
  // Each subsequent message:
  10,   // Send
  10,   // Receive
  // Total: 20ms per message
];

const totalWsTime = 120 + (20 * 100); // 2,120ms (2.1 seconds)

console.log(`HTTP: ${totalHttpTime}ms`);
console.log(`WebSocket: ${totalWsTime}ms`);
console.log(`WebSocket is ${(totalHttpTime / totalWsTime).toFixed(1)}x faster`);
// Output: WebSocket is 5.7x faster
```

### **9. Hybrid Approach (Best of Both)**

```typescript
// Use BOTH in the same application!

// app.module.ts
@Module({
  imports: [
    // HTTP for traditional API
    UsersModule,    // REST CRUD operations
    AuthModule,     // Login/logout
    FilesModule,    // File uploads
    
    // WebSockets for real-time features
    ChatModule,     // Real-time chat
    NotificationsModule, // Push notifications
    LiveDashboardModule, // Live data updates
  ],
})
export class AppModule {}

// users.controller.ts (HTTP)
@Controller('users')
export class UsersController {
  @Get(':id')
  async getUser(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }
  
  @Put(':id')
  async updateUser(@Param('id') id: string, @Body() dto: UpdateUserDto) {
    const user = await this.usersService.update(id, dto);
    
    // Trigger WebSocket notification after HTTP update
    this.notificationsGateway.sendToUser(id, 'profile-updated', user);
    
    return user;
  }
}

// notifications.gateway.ts (WebSocket)
@WebSocketGateway()
export class NotificationsGateway {
  @WebSocketServer()
  server: Server;
  
  sendToUser(userId: string, event: string, data: any) {
    this.server.to(userId).emit(event, data);
  }
}

// CLIENT: Use both protocols
class UserService {
  // HTTP for CRUD
  async updateProfile(userId: string, updates: any) {
    const response = await fetch(`/api/users/${userId}`, {
      method: 'PUT',
      body: JSON.stringify(updates)
    });
    return response.json();
  }
}

class NotificationService {
  private socket: Socket;
  
  // WebSocket for real-time notifications
  connect() {
    this.socket = io('http://localhost:3000');
    
    this.socket.on('profile-updated', (data) => {
      // Real-time notification from server!
      showNotification('Your profile was updated');
    });
  }
}
```

**Interview Tip**: **HTTP** is **request-response** (client always initiates, stateless, new connection per request) while **WebSockets** are **persistent bidirectional** (both can initiate, stateful, single long-lived connection). **Key difference**: HTTP server **cannot push** data to client (client must poll), WebSocket server **can push** anytime. **Overhead**: HTTP has large headers (500-800 bytes), WebSocket has small frames (2-6 bytes). **Use HTTP for**: CRUD operations, REST APIs, file transfers, caching. **Use WebSockets for**: chat, live notifications, collaborative editing, gaming, live dashboards, real-time data streams. **Production**: often use **both** - HTTP for traditional API, WebSockets for real-time features. WebSockets start with **HTTP handshake** (Upgrade header), then switch to WebSocket protocol. **Latency**: WebSockets much faster for multiple messages (reuse connection vs new connection each time).

</details>

3. When should you use WebSockets vs HTTP?

<details>
<summary><strong>Answer</strong></summary>

**WebSockets** and **HTTP** serve different purposes. The decision depends on whether you need **real-time bidirectional communication** (WebSockets) or **simple request-response** (HTTP). Choose based on your application's communication pattern, latency requirements, and scalability needs.

### **Use WebSockets When:**

#### **1. Real-Time Bidirectional Communication**

When the server needs to push data to clients without them requesting it:

```typescript
// ✅ PERFECT USE CASE: Chat Application
@WebSocketGateway({ cors: true })
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('send-message')
  handleMessage(@MessageBody() data: { room: string; message: string; userId: string }) {
    // Server pushes message to all clients in room INSTANTLY
    this.server.to(data.room).emit('new-message', {
      message: data.message,
      userId: data.userId,
      timestamp: new Date(),
    });
    // No polling needed - all clients receive instantly!
  }
}

// CLIENT: Receives messages in real-time
socket.on('new-message', (data) => {
  displayMessage(data); // Instant delivery!
});

// ❌ WRONG WITH HTTP: Would require constant polling
setInterval(async () => {
  const messages = await fetch('/api/messages'); // Wasteful!
}, 1000); // Polls every second - inefficient and delayed
```

#### **2. High-Frequency Updates**

When data changes frequently (multiple times per second):

```typescript
// ✅ PERFECT: Live Stock Ticker
@WebSocketGateway()
export class StockTickerGateway {
  @WebSocketServer()
  server: Server;

  constructor() {
    // Push stock prices as they change (multiple times per second)
    setInterval(() => {
      const stockData = this.getLatestStockPrices();
      this.server.emit('stock-update', stockData);
      // Efficient: Reuses connection, minimal overhead
    }, 100); // Update every 100ms
  }
}

// ❌ WRONG WITH HTTP: Too many requests
setInterval(async () => {
  await fetch('/api/stocks'); // New connection every 100ms!
}, 100); // Terrible performance, high server load
```

#### **3. Low Latency Requirements**

When every millisecond matters:

```typescript
// ✅ PERFECT: Online Multiplayer Game
@WebSocketGateway()
export class GameGateway {
  @SubscribeMessage('player-move')
  handlePlayerMove(@MessageBody() data: { x: number; y: number; playerId: string }) {
    // Broadcast player position to all other players
    // Latency: ~10-50ms (reuses existing connection)
    this.server.emit('player-moved', data);
  }
}

// ❌ WRONG WITH HTTP: Too slow
async function sendPlayerMove(x: number, y: number) {
  await fetch('/api/game/move', {
    method: 'POST',
    body: JSON.stringify({ x, y })
  });
  // Latency: ~100-300ms (new connection + TLS handshake)
  // Game feels laggy and unresponsive!
}
```

#### **4. Server-Initiated Events**

When the server needs to notify clients proactively:

```typescript
// ✅ PERFECT: Push Notifications
@WebSocketGateway()
export class NotificationsGateway {
  @WebSocketServer()
  server: Server;

  async sendNotification(userId: string, notification: any) {
    // Server initiates - pushes to client without request
    this.server.to(userId).emit('notification', notification);
    // Client receives instantly!
  }
}

// Triggered by background job, webhook, or another service
this.notificationsGateway.sendNotification('user-123', {
  title: 'New Message',
  body: 'You have a new message from John',
});

// ❌ WRONG WITH HTTP: Client must poll
setInterval(async () => {
  const notifications = await fetch('/api/notifications');
  // Most requests return empty - wasteful!
}, 5000); // Delay: up to 5 seconds before user sees notification
```

#### **5. Collaborative Applications**

Multiple users editing the same resource:

```typescript
// ✅ PERFECT: Collaborative Document Editing (like Google Docs)
@WebSocketGateway()
export class DocumentGateway {
  @SubscribeMessage('edit-document')
  handleEdit(@MessageBody() data: { docId: string; changes: any; userId: string }) {
    // Broadcast changes to all users viewing the document
    this.server.to(data.docId).emit('document-updated', {
      changes: data.changes,
      userId: data.userId,
    });
    // All users see changes in real-time!
  }
}

// ❌ WRONG WITH HTTP: Conflicts and delays
// User A makes change, User B doesn't see it for 5 seconds
// Both edit same paragraph - conflict!
```

### **Use HTTP When:**

#### **1. CRUD Operations**

Standard create, read, update, delete operations:

```typescript
// ✅ PERFECT: RESTful API
@Controller('users')
export class UsersController {
  @Get(':id')
  async getUser(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Post()
  async createUser(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Put(':id')
  async updateUser(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.usersService.update(id, updateUserDto);
  }

  @Delete(':id')
  async deleteUser(@Param('id') id: string) {
    return this.usersService.remove(id);
  }
}

// ❌ OVERKILL WITH WEBSOCKETS: Unnecessary complexity
// WebSocket connection just for simple CRUD is wasteful
```

#### **2. Infrequent Updates**

Data that rarely changes:

```typescript
// ✅ PERFECT: User Profile
@Controller('profile')
export class ProfileController {
  @Get()
  async getProfile(@User() user: UserEntity) {
    return this.profileService.getProfile(user.id);
    // Profile changes maybe once per day - HTTP is fine
  }
}

// ❌ OVERKILL WITH WEBSOCKETS
// Maintaining persistent connection for data that changes rarely is wasteful
```

#### **3. File Uploads/Downloads**

Large file transfers:

```typescript
// ✅ PERFECT: File Upload
@Controller('files')
export class FilesController {
  @Post('upload')
  @UseInterceptors(FileInterceptor('file'))
  async uploadFile(@UploadedFile() file: Express.Multer.File) {
    return this.filesService.upload(file);
  }

  @Get('download/:id')
  async downloadFile(@Param('id') id: string, @Res() res: Response) {
    const file = await this.filesService.getFile(id);
    res.download(file.path);
  }
}

// ❌ WRONG WITH WEBSOCKETS: Not designed for large binary data
// WebSocket frames have overhead, HTTP streaming is better
```

#### **4. Caching & CDN**

Content that benefits from HTTP caching:

```typescript
// ✅ PERFECT: Static content
@Controller('articles')
export class ArticlesController {
  @Get(':id')
  @CacheControl({ maxAge: 3600 }) // Cache for 1 hour
  async getArticle(@Param('id') id: string) {
    return this.articlesService.findOne(id);
  }
}

// Benefits from:
// - Browser cache
// - CDN cache
// - HTTP cache headers (ETag, Last-Modified)

// ❌ WRONG WITH WEBSOCKETS: No caching mechanism
// Every client needs full data, can't leverage CDN
```

#### **5. Public APIs**

APIs consumed by third parties:

```typescript
// ✅ PERFECT: Public REST API
@Controller('api/v1')
export class PublicApiController {
  @Get('weather')
  async getWeather(@Query('city') city: string) {
    return this.weatherService.getWeather(city);
  }
}

// Easy for clients to use:
// curl https://api.example.com/api/v1/weather?city=London

// ❌ WRONG WITH WEBSOCKETS: Complex for third parties
// Requires WebSocket client library, connection management, etc.
```

#### **6. SEO Requirements**

Content that needs to be indexed by search engines:

```typescript
// ✅ PERFECT: Blog posts, product pages
@Controller('blog')
export class BlogController {
  @Get(':slug')
  @Render('blog-post') // Server-side rendering
  async getPost(@Param('slug') slug: string) {
    return this.blogService.getPost(slug);
  }
}

// Search engines crawl HTTP, not WebSockets
// ❌ WRONG WITH WEBSOCKETS: Search engines can't index
```

### **Decision Matrix**

```typescript
/**
 * WEBSOCKETS:
 * - Real-time updates (chat, notifications, live feeds)
 * - High-frequency updates (stock tickers, sensor data)
 * - Low latency critical (gaming, trading platforms)
 * - Server-initiated communication
 * - Collaborative editing
 * - Presence tracking (online/offline status)
 * - Live dashboards
 * - IoT streaming data
 * 
 * HTTP:
 * - CRUD operations
 * - Infrequent updates (settings, profiles)
 * - File uploads/downloads
 * - Caching needed
 * - Public APIs
 * - SEO requirements
 * - Third-party integrations
 * - Idempotent operations
 */

// Decision flow:
function shouldUseWebSocket(requirements: {
  realTime: boolean;
  serverInitiated: boolean;
  highFrequency: boolean;
  lowLatency: boolean;
  bidirectional: boolean;
}): boolean {
  return (
    requirements.realTime ||
    requirements.serverInitiated ||
    requirements.highFrequency ||
    requirements.lowLatency ||
    requirements.bidirectional
  );
}

// Examples:
shouldUseWebSocket({
  realTime: true,      // Chat app
  serverInitiated: true,
  highFrequency: true,
  lowLatency: true,
  bidirectional: true,
}); // → true (Use WebSockets)

shouldUseWebSocket({
  realTime: false,     // User profile
  serverInitiated: false,
  highFrequency: false,
  lowLatency: false,
  bidirectional: false,
}); // → false (Use HTTP)
```

### **Hybrid Approach (Production Recommendation)**

```typescript
// ✅ BEST PRACTICE: Use both in the same app

// app.module.ts
@Module({
  imports: [
    // HTTP modules
    UsersModule,       // CRUD operations
    AuthModule,        // Login/logout
    FilesModule,       // File uploads
    ProductsModule,    // Product catalog
    
    // WebSocket modules
    ChatModule,        // Real-time chat
    NotificationsModule, // Push notifications
    PresenceModule,    // Online/offline status
    LiveDataModule,    // Live dashboard
  ],
})
export class AppModule {}

// Example: E-commerce application

// HTTP for standard operations
@Controller('products')
export class ProductsController {
  @Get()
  async getProducts() {
    return this.productsService.findAll();
  }

  @Post('purchase')
  async purchase(@Body() purchaseDto: PurchaseDto) {
    const order = await this.ordersService.create(purchaseDto);
    
    // Trigger WebSocket notification after HTTP operation
    this.notificationsGateway.notifyUser(
      purchaseDto.userId,
      'order-confirmed',
      order
    );
    
    return order;
  }
}

// WebSocket for real-time features
@WebSocketGateway()
export class NotificationsGateway {
  @WebSocketServer()
  server: Server;

  notifyUser(userId: string, event: string, data: any) {
    this.server.to(userId).emit(event, data);
  }
}

// Another example: Live inventory tracking
@Controller('inventory')
export class InventoryController {
  constructor(
    private inventoryService: InventoryService,
    private inventoryGateway: InventoryGateway,
  ) {}

  // HTTP: Update inventory
  @Put(':id')
  async updateInventory(
    @Param('id') id: string,
    @Body() updateDto: UpdateInventoryDto
  ) {
    const inventory = await this.inventoryService.update(id, updateDto);
    
    // WebSocket: Broadcast to all connected clients watching this item
    this.inventoryGateway.broadcastInventoryUpdate(inventory);
    
    return inventory;
  }
}

@WebSocketGateway()
export class InventoryGateway {
  @WebSocketServer()
  server: Server;

  broadcastInventoryUpdate(inventory: Inventory) {
    // All clients watching this product get instant update
    this.server.to(`product-${inventory.productId}`).emit('inventory-updated', {
      productId: inventory.productId,
      quantity: inventory.quantity,
      timestamp: new Date(),
    });
  }
}
```

### **Performance Comparison**

```typescript
// Scenario: Display live data that updates every 2 seconds for 1 hour

// HTTP Polling:
const httpRequests = (60 * 60) / 2; // 1,800 requests
const httpOverheadPerRequest = 800; // bytes
const httpDataPerRequest = 100;     // bytes
const httpTotal = httpRequests * (httpOverheadPerRequest + httpDataPerRequest);
console.log(`HTTP: ${httpTotal / 1024 / 1024} MB`); // ~1.54 MB

// WebSocket:
const wsHandshake = 800;           // bytes (once)
const wsFrameOverhead = 6;         // bytes per message
const wsDataPerMessage = 100;      // bytes
const wsMessages = (60 * 60) / 2;  // 1,800 messages
const wsTotal = wsHandshake + (wsMessages * (wsFrameOverhead + wsDataPerMessage));
console.log(`WebSocket: ${wsTotal / 1024 / 1024} MB`); // ~0.18 MB

// WebSocket uses 88% less bandwidth!
```

### **Common Mistakes**

```typescript
// ❌ MISTAKE 1: Using WebSockets for everything
@WebSocketGateway()
export class EverythingGateway {
  @SubscribeMessage('get-user-profile') // BAD: Use HTTP GET instead
  @SubscribeMessage('upload-file')      // BAD: Use HTTP POST instead
  @SubscribeMessage('search-products')  // BAD: Use HTTP GET instead
}
// WebSockets add complexity - only use when needed!

// ✅ CORRECT: Use HTTP for CRUD
@Controller('users')
export class UsersController {
  @Get(':id')
  getUserProfile() { /* ... */ }
}

// ❌ MISTAKE 2: Using HTTP polling for real-time data
setInterval(async () => {
  const messages = await fetch('/api/messages'); // Wasteful!
}, 1000);

// ✅ CORRECT: Use WebSockets
socket.on('new-message', (message) => {
  displayMessage(message); // Instant!
});

// ❌ MISTAKE 3: Not considering scale
// WebSockets hold connections open - limits concurrent users
// HTTP is stateless - scales better horizontally

// Solution: Use both strategically
// HTTP for bulk operations, WebSockets for real-time updates
```

**Interview Tip**: Use **WebSockets** for **real-time bidirectional communication** (chat, live notifications, collaborative editing, gaming, live dashboards, high-frequency updates) where **server needs to push** data to clients instantly without polling. Use **HTTP** for **CRUD operations**, infrequent updates, file transfers, caching, public APIs, and SEO requirements. **Key factors**: if server-initiated push needed or updates are frequent (>1 per minute) → WebSockets; if client-initiated requests, infrequent updates, or need caching/SEO → HTTP. **Production best practice**: use **both** - HTTP for standard API operations, WebSockets for real-time features. **Performance**: WebSockets save 88-95% bandwidth for frequent updates by reusing connection. **Scalability**: HTTP scales better (stateless), WebSockets require connection management (use Redis adapter for multi-server).

</details>

4. What is Socket.IO?

<details>
<summary><strong>Answer</strong></summary>

**Socket.IO** is a JavaScript library that enables **real-time, bidirectional, event-based communication** between web clients and servers. It's built on top of WebSockets but provides additional features like automatic reconnection, rooms, namespaces, broadcasting, and fallback to HTTP long-polling when WebSockets aren't available.

### **What Makes Socket.IO Special**

Socket.IO is **not just a WebSocket wrapper** - it's a comprehensive real-time communication framework:

```typescript
/**
 * Socket.IO = WebSocket Protocol + Additional Features
 * 
 * Features Socket.IO adds:
 * 1. Automatic reconnection
 * 2. Packet buffering (queues messages during disconnection)
 * 3. Acknowledgments (callbacks for message delivery)
 * 4. Broadcasting to multiple clients
 * 5. Rooms (group clients into channels)
 * 6. Namespaces (separate communication channels)
 * 7. Multiplexing (multiple channels over single connection)
 * 8. Fallback to HTTP long-polling (if WebSocket unavailable)
 * 9. Binary support
 * 10. Error handling
 */
```

### **Socket.IO Components**

```typescript
// SERVER SIDE: Socket.IO Server
import { Server } from 'socket.io';

const io = new Server(3000, {
  cors: {
    origin: 'http://localhost:3001',
    credentials: true,
  },
});

// Listen for connections
io.on('connection', (socket) => {
  console.log('Client connected:', socket.id);
  
  // Listen to custom events
  socket.on('message', (data) => {
    console.log('Received:', data);
  });
  
  // Emit custom events
  socket.emit('welcome', 'Hello from server!');
  
  // Handle disconnection
  socket.on('disconnect', () => {
    console.log('Client disconnected:', socket.id);
  });
});

// CLIENT SIDE: Socket.IO Client
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000', {
  reconnection: true,
  reconnectionDelay: 1000,
  reconnectionAttempts: 10,
});

// Listen for connection
socket.on('connect', () => {
  console.log('Connected to server:', socket.id);
});

// Listen to custom events
socket.on('welcome', (message) => {
  console.log('Server says:', message);
});

// Emit custom events
socket.emit('message', 'Hello from client!');

// Handle disconnection
socket.on('disconnect', (reason) => {
  console.log('Disconnected:', reason);
});
```

### **NestJS Socket.IO Integration**

```typescript
// Install dependencies:
// npm install @nestjs/websockets @nestjs/platform-socket.io socket.io

// Gateway implementation
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  OnGatewayDisconnect,
  OnGatewayInit,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway({
  cors: {
    origin: '*', // Configure CORS
  },
  namespace: '/chat', // Optional: separate namespace
})
export class ChatGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  @WebSocketServer()
  server: Server; // Socket.IO server instance

  private logger = new Logger('ChatGateway');

  // 1. Gateway initialized
  afterInit(server: Server) {
    this.logger.log('WebSocket Gateway initialized');
  }

  // 2. Client connected
  handleConnection(client: Socket, ...args: any[]) {
    this.logger.log(`Client connected: ${client.id}`);
    
    // Send welcome message
    client.emit('welcome', {
      message: 'Welcome to the chat!',
      clientId: client.id,
    });
  }

  // 3. Client disconnected
  handleDisconnect(client: Socket) {
    this.logger.log(`Client disconnected: ${client.id}`);
  }

  // 4. Handle custom events
  @SubscribeMessage('message')
  handleMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { text: string; room?: string },
  ) {
    this.logger.log(`Message from ${client.id}: ${data.text}`);
    
    if (data.room) {
      // Send to specific room
      this.server.to(data.room).emit('message', {
        clientId: client.id,
        text: data.text,
        timestamp: new Date(),
      });
    } else {
      // Broadcast to all clients
      this.server.emit('message', {
        clientId: client.id,
        text: data.text,
        timestamp: new Date(),
      });
    }
    
    // Return acknowledgment
    return { success: true, timestamp: new Date() };
  }
}
```

### **Key Features with Examples**

#### **1. Automatic Reconnection**

```typescript
// Socket.IO automatically reconnects if connection drops
const socket = io('http://localhost:3000', {
  reconnection: true,              // Enable reconnection
  reconnectionDelay: 1000,         // Initial delay
  reconnectionDelayMax: 5000,      // Maximum delay
  reconnectionAttempts: 10,        // Max attempts
});

// Events for reconnection
socket.on('reconnect_attempt', (attemptNumber) => {
  console.log(`Reconnection attempt ${attemptNumber}`);
});

socket.on('reconnect', (attemptNumber) => {
  console.log(`Reconnected after ${attemptNumber} attempts`);
});

socket.on('reconnect_error', (error) => {
  console.error('Reconnection error:', error);
});

socket.on('reconnect_failed', () => {
  console.error('Failed to reconnect after all attempts');
});

// Native WebSocket requires manual reconnection logic!
```

#### **2. Acknowledgments (Callbacks)**

```typescript
// CLIENT: Send message with acknowledgment
socket.emit('save-data', { name: 'John', age: 30 }, (response) => {
  console.log('Server acknowledged:', response);
  // Output: Server acknowledged: { success: true, id: 123 }
});

// SERVER: Return acknowledgment
@SubscribeMessage('save-data')
handleSaveData(
  @MessageBody() data: { name: string; age: number },
  @ConnectedSocket() client: Socket,
) {
  // Save data
  const result = this.dataService.save(data);
  
  // Return acknowledgment (client receives in callback)
  return { success: true, id: result.id };
}

// Native WebSocket doesn't have built-in acknowledgments!
```

#### **3. Rooms (Group Clients)**

```typescript
// SERVER: Manage rooms
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('join-room')
  handleJoinRoom(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { room: string },
  ) {
    // Add client to room
    client.join(data.room);
    
    // Notify others in room
    client.to(data.room).emit('user-joined', {
      clientId: client.id,
      room: data.room,
    });
    
    return { success: true };
  }

  @SubscribeMessage('leave-room')
  handleLeaveRoom(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { room: string },
  ) {
    // Remove client from room
    client.leave(data.room);
    
    // Notify others in room
    client.to(data.room).emit('user-left', {
      clientId: client.id,
      room: data.room,
    });
  }

  @SubscribeMessage('room-message')
  handleRoomMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { room: string; message: string },
  ) {
    // Send message only to clients in the room
    this.server.to(data.room).emit('message', {
      clientId: client.id,
      message: data.message,
    });
  }
}

// CLIENT: Join room and send messages
socket.emit('join-room', { room: 'general' });
socket.emit('room-message', { room: 'general', message: 'Hello room!' });

// Native WebSocket requires manual room management!
```

#### **4. Namespaces (Separate Channels)**

```typescript
// SERVER: Multiple namespaces
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  // Handles connections to /chat namespace
}

@WebSocketGateway({ namespace: '/notifications' })
export class NotificationsGateway {
  // Handles connections to /notifications namespace
}

@WebSocketGateway({ namespace: '/admin' })
export class AdminGateway {
  // Handles connections to /admin namespace
}

// CLIENT: Connect to specific namespace
const chatSocket = io('http://localhost:3000/chat');
const notificationSocket = io('http://localhost:3000/notifications');
const adminSocket = io('http://localhost:3000/admin');

// Each namespace is isolated - events don't interfere
chatSocket.emit('message', 'Hello chat!');
notificationSocket.emit('subscribe', 'user-123');
adminSocket.emit('get-stats', {});

// Native WebSocket requires separate connections or manual multiplexing!
```

#### **5. Broadcasting**

```typescript
// SERVER: Different broadcasting options
@WebSocketGateway()
export class BroadcastGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('broadcast-test')
  handleBroadcast(@ConnectedSocket() client: Socket) {
    // 1. Send to all clients (including sender)
    this.server.emit('event', 'To everyone');
    
    // 2. Send to all clients EXCEPT sender
    client.broadcast.emit('event', 'To everyone except sender');
    
    // 3. Send to all clients in a room
    this.server.to('room1').emit('event', 'To room1');
    
    // 4. Send to all clients in a room EXCEPT sender
    client.to('room1').emit('event', 'To room1 except sender');
    
    // 5. Send to specific client
    this.server.to(client.id).emit('event', 'Only to you');
    
    // 6. Send to multiple rooms
    this.server.to(['room1', 'room2']).emit('event', 'To room1 and room2');
  }
}
```

#### **6. Fallback to HTTP Long-Polling**

```typescript
// Socket.IO automatically falls back if WebSocket unavailable
const socket = io('http://localhost:3000', {
  transports: ['websocket', 'polling'], // Try WebSocket first, fallback to polling
});

// Connection upgrade process:
// 1. Client connects via HTTP long-polling (always works)
// 2. Client tries to upgrade to WebSocket
// 3. If WebSocket successful, switches to WebSocket
// 4. If WebSocket fails, stays on HTTP long-polling

socket.on('connect', () => {
  console.log('Transport:', socket.io.engine.transport.name);
  // Output: "websocket" or "polling"
});

// Listen for transport upgrade
socket.io.engine.on('upgrade', (transport) => {
  console.log('Upgraded to:', transport.name);
  // Output: Upgraded to: websocket
});

// Native WebSocket fails completely if unavailable!
```

### **Production-Grade Socket.IO Setup**

```typescript
// main.ts - Configure Socket.IO with NestJS
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { IoAdapter } from '@nestjs/platform-socket.io';
import { ServerOptions } from 'socket.io';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Configure Socket.IO adapter
  app.useWebSocketAdapter(new IoAdapter(app));
  
  await app.listen(3000);
}
bootstrap();

// Custom Socket.IO adapter with advanced configuration
import { IoAdapter } from '@nestjs/platform-socket.io';
import { ServerOptions } from 'socket.io';
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

export class CustomSocketAdapter extends IoAdapter {
  createIOServer(port: number, options?: ServerOptions) {
    const server = super.createIOServer(port, {
      ...options,
      cors: {
        origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
        credentials: true,
      },
      transports: ['websocket', 'polling'],
      pingTimeout: 60000,
      pingInterval: 25000,
      upgradeTimeout: 30000,
      maxHttpBufferSize: 1e6, // 1MB
      allowEIO3: true, // Support older clients
    });
    
    // Redis adapter for multi-server deployment
    const pubClient = createClient({ url: process.env.REDIS_URL });
    const subClient = pubClient.duplicate();
    
    Promise.all([pubClient.connect(), subClient.connect()]).then(() => {
      server.adapter(createAdapter(pubClient, subClient));
    });
    
    return server;
  }
}

// Use custom adapter
app.useWebSocketAdapter(new CustomSocketAdapter(app));
```

### **Complete Chat Example**

```typescript
// chat.gateway.ts
@WebSocketGateway({ cors: true })
export class ChatGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;
  
  private users: Map<string, { id: string; username: string }> = new Map();

  handleConnection(client: Socket) {
    console.log(`Client connected: ${client.id}`);
  }

  handleDisconnect(client: Socket) {
    const user = this.users.get(client.id);
    if (user) {
      this.server.emit('user-left', { username: user.username });
      this.users.delete(client.id);
    }
  }

  @SubscribeMessage('join')
  handleJoin(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { username: string; room: string },
  ) {
    this.users.set(client.id, { id: client.id, username: data.username });
    client.join(data.room);
    
    // Notify room
    client.to(data.room).emit('user-joined', { username: data.username });
    
    // Send room users to new user
    const roomUsers = Array.from(this.users.values());
    return { users: roomUsers };
  }

  @SubscribeMessage('send-message')
  handleMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { room: string; message: string },
  ) {
    const user = this.users.get(client.id);
    
    // Broadcast to room
    this.server.to(data.room).emit('new-message', {
      id: Date.now(),
      username: user.username,
      message: data.message,
      timestamp: new Date(),
    });
    
    // Acknowledgment
    return { success: true, messageId: Date.now() };
  }

  @SubscribeMessage('typing')
  handleTyping(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { room: string },
  ) {
    const user = this.users.get(client.id);
    client.to(data.room).emit('user-typing', { username: user.username });
  }
}

// CLIENT: React component
import { useEffect, useState } from 'react';
import { io } from 'socket.io-client';

function ChatApp() {
  const [socket, setSocket] = useState(null);
  const [messages, setMessages] = useState([]);
  const [inputMessage, setInputMessage] = useState('');

  useEffect(() => {
    const newSocket = io('http://localhost:3000');
    
    newSocket.on('connect', () => {
      console.log('Connected');
      newSocket.emit('join', { username: 'John', room: 'general' });
    });
    
    newSocket.on('new-message', (message) => {
      setMessages((prev) => [...prev, message]);
    });
    
    newSocket.on('user-joined', (data) => {
      console.log(`${data.username} joined`);
    });
    
    setSocket(newSocket);
    
    return () => newSocket.close();
  }, []);

  const sendMessage = () => {
    if (socket && inputMessage.trim()) {
      socket.emit('send-message', 
        { room: 'general', message: inputMessage },
        (response) => {
          console.log('Message sent:', response);
        }
      );
      setInputMessage('');
    }
  };

  return (
    <div>
      <div>
        {messages.map((msg) => (
          <div key={msg.id}>
            <strong>{msg.username}:</strong> {msg.message}
          </div>
        ))}
      </div>
      <input
        value={inputMessage}
        onChange={(e) => setInputMessage(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
      />
      <button onClick={sendMessage}>Send</button>
    </div>
  );
}
```

**Interview Tip**: **Socket.IO** is a real-time communication library built on top of WebSockets with **additional features**: automatic reconnection, packet buffering, acknowledgments (callbacks), rooms (group clients), namespaces (separate channels), broadcasting, fallback to HTTP long-polling. **Not just WebSocket wrapper** - adds production features that native WebSocket lacks. **NestJS integration**: use `@WebSocketGateway()` decorator, `@WebSocketServer()` for server instance, `@SubscribeMessage()` for events. **Key benefits**: automatic reconnection (no manual logic needed), rooms for grouping clients (chat rooms, game lobbies), acknowledgments for reliable delivery, fallback transport (works even if WebSocket blocked). **Production**: use Redis adapter for multi-server scaling. **When to use**: almost always prefer Socket.IO over native WebSocket for production apps due to built-in features and reliability.

</details>

5. What is the difference between Socket.IO and native WebSockets?

<details>
<summary><strong>Answer</strong></summary>

**Socket.IO** and **native WebSockets** both enable real-time communication, but Socket.IO provides **additional features** and **abstractions** that native WebSockets lack. Native WebSocket is a **low-level protocol**, while Socket.IO is a **high-level library** built on top of WebSockets with automatic reconnection, rooms, namespaces, and fallback mechanisms.

### **Key Differences**

| Feature | Native WebSocket | Socket.IO |
|---------|-----------------|------------|
| **Protocol** | WebSocket (RFC 6455) | Socket.IO protocol (custom) |
| **Transport** | WebSocket only | WebSocket + HTTP long-polling (fallback) |
| **Reconnection** | Manual | Automatic |
| **Event-Based** | No (only `message` event) | Yes (custom events) |
| **Rooms** | No | Yes |
| **Namespaces** | No | Yes |
| **Broadcasting** | Manual | Built-in |
| **Acknowledgments** | No | Yes (callbacks) |
| **Binary Support** | Yes | Yes |
| **Packet Buffering** | No | Yes (queues during disconnect) |
| **Connection State** | `readyState` | `connected`, `disconnected` |
| **Browser Support** | Modern browsers | All browsers (fallback) |
| **Learning Curve** | Lower | Higher |
| **Bundle Size** | Smaller (~10KB) | Larger (~60KB client) |
| **Compatibility** | Standard WebSocket | Socket.IO clients only |

### **1. Protocol & Transport**

```typescript
// NATIVE WEBSOCKET: Uses WebSocket protocol only
const ws = new WebSocket('ws://localhost:3000');

ws.onopen = () => {
  console.log('Transport: WebSocket');
};

// If WebSocket fails, connection fails completely
// No fallback mechanism!

// SOCKET.IO: Tries multiple transports
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000', {
  transports: ['websocket', 'polling'], // Tries both
});

socket.on('connect', () => {
  console.log('Transport:', socket.io.engine.transport.name);
  // Could be "websocket" or "polling" depending on what works
});

// Automatic upgrade from polling to WebSocket
socket.io.engine.on('upgrade', (transport) => {
  console.log('Upgraded to:', transport.name);
});

// Works even if WebSocket is blocked by firewall/proxy!
```

### **2. Reconnection**

```typescript
// NATIVE WEBSOCKET: Manual reconnection required
class WebSocketClient {
  private ws: WebSocket | null = null;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 10;

  connect() {
    this.ws = new WebSocket('ws://localhost:3000');

    this.ws.onopen = () => {
      console.log('Connected');
      this.reconnectAttempts = 0;
    };

    this.ws.onclose = (event) => {
      console.log('Connection closed');
      
      // MUST implement reconnection logic manually
      if (this.reconnectAttempts < this.maxReconnectAttempts) {
        this.reconnectAttempts++;
        const delay = Math.min(1000 * Math.pow(2, this.reconnectAttempts), 30000);
        console.log(`Reconnecting in ${delay}ms...`);
        setTimeout(() => this.connect(), delay);
      }
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }
}

// Lots of boilerplate code for reconnection!

// SOCKET.IO: Automatic reconnection (built-in)
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000', {
  reconnection: true,           // Enabled by default
  reconnectionDelay: 1000,      // Initial delay
  reconnectionDelayMax: 5000,   // Max delay
  reconnectionAttempts: 10,     // Max attempts
});

// Automatic reconnection - no code needed!
// Just listen to events if you want to track it:
socket.on('reconnect', (attemptNumber) => {
  console.log(`Reconnected after ${attemptNumber} attempts`);
});

// No manual reconnection logic required!
```

### **3. Event-Based Communication**

```typescript
// NATIVE WEBSOCKET: Only generic message event
const ws = new WebSocket('ws://localhost:3000');

ws.onmessage = (event) => {
  // MUST manually parse and route messages
  const data = JSON.parse(event.data);
  
  if (data.type === 'chat') {
    handleChatMessage(data);
  } else if (data.type === 'notification') {
    handleNotification(data);
  } else if (data.type === 'update') {
    handleUpdate(data);
  }
  // Manual routing for every message type!
};

// Send: Must wrap in JSON with type
ws.send(JSON.stringify({
  type: 'chat',
  message: 'Hello'
}));

// SOCKET.IO: Custom event names (cleaner)
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000');

// Listen to specific events directly
socket.on('chat', (data) => {
  handleChatMessage(data); // Automatic routing!
});

socket.on('notification', (data) => {
  handleNotification(data);
});

socket.on('update', (data) => {
  handleUpdate(data);
});

// Send: Use event names
socket.emit('chat', { message: 'Hello' }); // Clean and clear!
```

### **4. Rooms & Namespaces**

```typescript
// NATIVE WEBSOCKET: No built-in rooms/namespaces
// Must implement manually with a lot of code

// SERVER: Manual room management
class WebSocketServer {
  private rooms: Map<string, Set<WebSocket>> = new Map();

  handleConnection(ws: WebSocket) {
    ws.on('message', (data) => {
      const message = JSON.parse(data.toString());
      
      if (message.type === 'join-room') {
        // Manually add to room
        if (!this.rooms.has(message.room)) {
          this.rooms.set(message.room, new Set());
        }
        this.rooms.get(message.room)!.add(ws);
      }
      
      if (message.type === 'room-message') {
        // Manually broadcast to room
        const room = this.rooms.get(message.room);
        if (room) {
          room.forEach((client) => {
            if (client !== ws && client.readyState === WebSocket.OPEN) {
              client.send(JSON.stringify(message));
            }
          });
        }
      }
    });
  }
}

// Lots of manual management code!

// SOCKET.IO: Built-in rooms (one line)
// SERVER:
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('join-room')
  handleJoinRoom(@ConnectedSocket() client: Socket, @MessageBody() room: string) {
    client.join(room); // That's it! One line!
  }

  @SubscribeMessage('room-message')
  handleRoomMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { room: string; message: string }
  ) {
    this.server.to(data.room).emit('message', data.message); // One line!
  }
}

// Much simpler!
```

### **5. Acknowledgments (Callbacks)**

```typescript
// NATIVE WEBSOCKET: No acknowledgments
// Must implement manually with message IDs

class WebSocketWithAcknowledgments {
  private ws: WebSocket;
  private callbacks: Map<string, Function> = new Map();

  send(event: string, data: any, callback?: Function) {
    const messageId = Math.random().toString(36);
    
    if (callback) {
      this.callbacks.set(messageId, callback);
    }
    
    this.ws.send(JSON.stringify({
      id: messageId,
      event,
      data
    }));
  }

  onmessage = (event) => {
    const message = JSON.parse(event.data);
    
    // Check if this is an acknowledgment
    if (message.ack && this.callbacks.has(message.id)) {
      const callback = this.callbacks.get(message.id);
      callback(message.data);
      this.callbacks.delete(message.id);
    }
  };
}

// Complex manual implementation!

// SOCKET.IO: Built-in acknowledgments
const socket = io('http://localhost:3000');

// Send with callback (acknowledgment)
socket.emit('save-data', { name: 'John' }, (response) => {
  console.log('Server acknowledged:', response);
  // Callback automatically called when server responds
});

// SERVER:
@SubscribeMessage('save-data')
handleSaveData(@MessageBody() data: any) {
  // Process data
  const result = this.service.save(data);
  
  // Return value is sent as acknowledgment
  return { success: true, id: result.id };
}

// Simple and clean!
```

### **6. Broadcasting**

```typescript
// NATIVE WEBSOCKET: Manual broadcasting
class WebSocketServer {
  private clients: Set<WebSocket> = new Set();

  broadcast(message: string, sender?: WebSocket) {
    this.clients.forEach((client) => {
      if (client !== sender && client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  }
}

// Must track clients manually and implement broadcast logic

// SOCKET.IO: Built-in broadcasting
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('message')
  handleMessage(@ConnectedSocket() client: Socket, @MessageBody() data: any) {
    // Broadcast to all clients
    this.server.emit('message', data);
    
    // Or broadcast to all except sender
    client.broadcast.emit('message', data);
    
    // Or broadcast to specific room
    this.server.to('room1').emit('message', data);
  }
}

// Multiple broadcasting options built-in!
```

### **7. Packet Buffering**

```typescript
// NATIVE WEBSOCKET: Messages lost if disconnected
const ws = new WebSocket('ws://localhost:3000');

ws.send('Message 1'); // Sent
ws.close();
ws.send('Message 2'); // LOST! Error thrown

// Must implement queue manually
class BufferedWebSocket {
  private ws: WebSocket;
  private messageQueue: string[] = [];

  send(message: string) {
    if (this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(message);
    } else {
      this.messageQueue.push(message); // Queue manually
    }
  }

  onopen = () => {
    // Flush queue manually
    while (this.messageQueue.length > 0) {
      this.ws.send(this.messageQueue.shift()!);
    }
  };
}

// SOCKET.IO: Automatic packet buffering
const socket = io('http://localhost:3000');

socket.emit('message', 'Message 1'); // Sent
socket.disconnect();
socket.emit('message', 'Message 2'); // Automatically queued!
socket.connect();
// Message 2 is automatically sent when reconnected

// No manual queue implementation needed!
```

### **8. Browser Compatibility**

```typescript
// NATIVE WEBSOCKET: Modern browsers only
if ('WebSocket' in window) {
  const ws = new WebSocket('ws://localhost:3000');
} else {
  // Old browser - no WebSocket support
  // Must implement HTTP long-polling fallback manually
  console.error('WebSocket not supported');
}

// SOCKET.IO: Works everywhere
const socket = io('http://localhost:3000');
// Automatically falls back to HTTP long-polling in old browsers
// Works in IE9+ (with polyfill)
```

### **9. Server Implementation Comparison**

```typescript
// NATIVE WEBSOCKET SERVER (Node.js ws library)
import { WebSocketServer } from 'ws';

const wss = new WebSocketServer({ port: 3000 });

wss.on('connection', (ws) => {
  console.log('Client connected');

  ws.on('message', (data) => {
    const message = JSON.parse(data.toString());
    
    // Manual message routing
    if (message.type === 'chat') {
      // Manual broadcast to all clients
      wss.clients.forEach((client) => {
        if (client !== ws && client.readyState === WebSocket.OPEN) {
          client.send(JSON.stringify({ type: 'chat', data: message.data }));
        }
      });
    }
  });

  ws.on('close', () => {
    console.log('Client disconnected');
  });
});

// SOCKET.IO SERVER (NestJS)
@WebSocketGateway()
export class ChatGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  handleConnection(client: Socket) {
    console.log('Client connected:', client.id);
  }

  handleDisconnect(client: Socket) {
    console.log('Client disconnected:', client.id);
  }

  @SubscribeMessage('chat')
  handleChat(@ConnectedSocket() client: Socket, @MessageBody() data: any) {
    // Automatic broadcast - much cleaner!
    client.broadcast.emit('chat', data);
  }
}
```

### **10. When to Use Each**

```typescript
// ✅ USE NATIVE WEBSOCKET WHEN:
// - Simple use case (minimal features needed)
// - Bundle size critical (need smallest possible)
// - Communicating with non-Socket.IO servers
// - Standard WebSocket protocol required
// - Maximum performance (less abstraction)

// Example: Simple data streaming
const ws = new WebSocket('ws://api.example.com/stream');
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  updateChart(data); // Simple data display
};

// ✅ USE SOCKET.IO WHEN:
// - Need automatic reconnection
// - Need rooms/namespaces
// - Need acknowledgments
// - Production app (reliability important)
// - Need fallback for restricted networks
// - Complex real-time features (chat, collaboration)

// Example: Production chat application
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer() server: Server;
  
  @SubscribeMessage('send-message')
  async handleMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { room: string; message: string }
  ) {
    // Save to database
    await this.chatService.saveMessage(data);
    
    // Broadcast to room
    this.server.to(data.room).emit('new-message', {
      message: data.message,
      senderId: client.id,
      timestamp: new Date()
    });
    
    // Return acknowledgment
    return { success: true, messageId: Date.now() };
  }
}
```

### **Bundle Size Comparison**

```typescript
// NATIVE WEBSOCKET
// Client: ~0 KB (built into browser)
// Server: ~10 KB (ws library)

// SOCKET.IO
// Client: ~60 KB (socket.io-client)
// Server: ~100 KB (@nestjs/platform-socket.io + socket.io)

// Trade-off: Socket.IO is larger but provides much more functionality
```

### **Performance Comparison**

```typescript
// Benchmark: Send 10,000 messages

// Native WebSocket:
// - Latency: ~5-10ms per message
// - Overhead: 2-6 bytes per frame
// - Total time: ~50-100ms

// Socket.IO:
// - Latency: ~8-15ms per message (slightly higher due to abstraction)
// - Overhead: 10-20 bytes per packet (includes event name, namespace)
// - Total time: ~80-150ms

// Difference: Native WebSocket is 20-30% faster
// BUT: Socket.IO's features usually worth the small overhead
```

### **Interoperability**

```typescript
// NATIVE WEBSOCKET: Standard protocol
// Can connect to any WebSocket server
const ws = new WebSocket('ws://any-standard-websocket-server.com');

// SOCKET.IO: Custom protocol
// Can ONLY connect to Socket.IO servers
const socket = io('http://socket-io-server.com');

// ❌ Won't work:
const socket = io('ws://standard-websocket-server.com');
// Socket.IO client can't connect to standard WebSocket server!

// Solution: Use native WebSocket if need to connect to standard servers
```

### **Production Recommendation**

```typescript
// RECOMMENDED: Use Socket.IO for most production apps

// Reasons:
// 1. Automatic reconnection (critical for reliability)
// 2. Rooms/namespaces (essential for scaling)
// 3. Fallback transport (works in restricted networks)
// 4. Event-based API (cleaner code)
// 5. Acknowledgments (reliable messaging)
// 6. Battle-tested (used by millions)

// Only use native WebSocket if:
// - Bundle size is absolutely critical
// - Connecting to non-Socket.IO server
// - Very simple use case (no rooms, no reconnection needed)

@Module({
  imports: [
    // Recommended for production
    ChatModule, // Uses Socket.IO
    NotificationsModule, // Uses Socket.IO
  ],
})
export class AppModule {}
```

**Interview Tip**: **Native WebSocket** is low-level protocol (only `message` event, manual reconnection, no rooms/namespaces, no acknowledgments) while **Socket.IO** is high-level library with **automatic reconnection**, **event-based API** (custom event names), **rooms** (group clients), **namespaces** (separate channels), **acknowledgments** (callbacks), and **fallback to HTTP long-polling**. **Key difference**: Socket.IO provides production-ready features out-of-the-box that you'd need to manually implement with native WebSocket. **Trade-off**: Socket.IO is larger (~60KB client vs ~0KB native) but provides much more functionality. **When to use**: Socket.IO for production apps (reliability, features), native WebSocket for simple use cases or when connecting to standard WebSocket servers. **Not compatible**: Socket.IO client can't connect to standard WebSocket server (custom protocol). **NestJS**: use `@nestjs/platform-socket.io` - integrates Socket.IO seamlessly.

</details>

## NestJS WebSocket Setup

6. How do you install WebSocket dependencies (`@nestjs/websockets`, `@nestjs/platform-socket.io`)?

<details>
<summary><strong>Answer</strong></summary>

**NestJS WebSocket** support requires installing specific packages depending on whether you want to use **Socket.IO** (recommended) or **native WebSockets** (ws library). The most common approach is using Socket.IO for its additional features like rooms, namespaces, and automatic reconnection.

### **Option 1: Socket.IO (Recommended)**

**Installation:**

```bash
# Install NestJS WebSocket packages
npm install @nestjs/websockets @nestjs/platform-socket.io

# Install Socket.IO server and client (peer dependencies)
npm install socket.io socket.io-client

# Or using yarn
yarn add @nestjs/websockets @nestjs/platform-socket.io socket.io socket.io-client

# Or using pnpm
pnpm add @nestjs/websockets @nestjs/platform-socket.io socket.io socket.io-client
```

**Package Purposes:**

```typescript
/**
 * @nestjs/websockets
 * - Core WebSocket functionality for NestJS
 * - Provides decorators: @WebSocketGateway(), @SubscribeMessage(), etc.
 * - Gateway interfaces: OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
 * - Platform-agnostic (works with Socket.IO or ws)
 */

/**
 * @nestjs/platform-socket.io
 * - Socket.IO adapter for NestJS
 * - Integrates Socket.IO with NestJS WebSocket decorators
 * - Provides IoAdapter class
 */

/**
 * socket.io
 * - Socket.IO server library
 * - Core real-time engine
 * - Peer dependency for @nestjs/platform-socket.io
 */

/**
 * socket.io-client (optional - for testing or client-side)
 * - Socket.IO client library
 * - Used for testing or Node.js clients
 * - Browser clients can use CDN or npm package
 */
```

**Complete Installation Script:**

```bash
#!/bin/bash
# install-websockets.sh

echo "Installing NestJS WebSocket dependencies..."

# Core WebSocket packages
npm install @nestjs/websockets @nestjs/platform-socket.io

# Socket.IO packages
npm install socket.io socket.io-client

# Optional: TypeScript types (usually included)
npm install --save-dev @types/socket.io @types/socket.io-client

echo "WebSocket dependencies installed successfully!"
```

### **Option 2: Native WebSockets (ws library)**

If you prefer native WebSocket protocol instead of Socket.IO:

```bash
# Install NestJS WebSocket core
npm install @nestjs/websockets

# Install ws library (native WebSocket)
npm install @nestjs/platform-ws ws

# TypeScript types
npm install --save-dev @types/ws
```

### **Verify Installation**

**Check package.json:**

```json
// package.json
{
  "name": "my-nestjs-app",
  "version": "1.0.0",
  "dependencies": {
    "@nestjs/common": "^10.0.0",
    "@nestjs/core": "^10.0.0",
    "@nestjs/platform-express": "^10.0.0",
    "@nestjs/websockets": "^10.0.0",
    "@nestjs/platform-socket.io": "^10.0.0",
    "socket.io": "^4.6.0",
    "socket.io-client": "^4.6.0",
    "reflect-metadata": "^0.1.13",
    "rxjs": "^7.8.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "@types/socket.io": "^3.0.0",
    "@types/socket.io-client": "^3.0.0",
    "typescript": "^5.0.0"
  }
}
```

**Test installation:**

```bash
# Check installed versions
npm list @nestjs/websockets
npm list @nestjs/platform-socket.io
npm list socket.io

# Output:
# @nestjs/websockets@10.x.x
# @nestjs/platform-socket.io@10.x.x  
# socket.io@4.x.x
```

### **Basic Setup After Installation**

**1. Create a WebSocket Gateway:**

```typescript
// chat.gateway.ts
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({
  cors: {
    origin: '*', // Configure for your needs
  },
})
export class ChatGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  handleConnection(client: Socket) {
    console.log(`Client connected: ${client.id}`);
  }

  handleDisconnect(client: Socket) {
    console.log(`Client disconnected: ${client.id}`);
  }

  @SubscribeMessage('message')
  handleMessage(client: Socket, payload: any): string {
    return 'Hello from server!';
  }
}
```

**2. Register Gateway in Module:**

```typescript
// chat.module.ts
import { Module } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';

@Module({
  providers: [ChatGateway],
})
export class ChatModule {}
```

**3. Import Module in App:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ChatModule } from './chat/chat.module';

@Module({
  imports: [ChatModule],
})
export class AppModule {}
```

**4. Start Application:**

```bash
npm run start:dev

# Output:
# [Nest] WebSocketsController {/}:
# [Nest] WebSocket is listening on port 3000
```

### **Advanced Configuration**

**Custom Socket.IO Adapter:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { IoAdapter } from '@nestjs/platform-socket.io';
import { ServerOptions } from 'socket.io';

// Custom adapter with advanced configuration
class CustomSocketIOAdapter extends IoAdapter {
  createIOServer(port: number, options?: ServerOptions) {
    const server = super.createIOServer(port, {
      ...options,
      cors: {
        origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
        credentials: true,
      },
      transports: ['websocket', 'polling'],
      pingTimeout: 60000,
      pingInterval: 25000,
      upgradeTimeout: 30000,
      maxHttpBufferSize: 1e6, // 1MB
      allowEIO3: true, // Support older clients
    });
    return server;
  }
}

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Use custom Socket.IO adapter
  app.useWebSocketAdapter(new CustomSocketIOAdapter(app));
  
  await app.listen(3000);
  console.log('Application with WebSocket is running on: http://localhost:3000');
}
bootstrap();
```

### **Redis Adapter for Multi-Server (Production)**

```bash
# Install Redis adapter
npm install @socket.io/redis-adapter redis
```

```typescript
// redis-socket.adapter.ts
import { IoAdapter } from '@nestjs/platform-socket.io';
import { ServerOptions } from 'socket.io';
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

export class RedisSocketIOAdapter extends IoAdapter {
  private adapterConstructor: ReturnType<typeof createAdapter>;

  async connectToRedis(): Promise<void> {
    const pubClient = createClient({ 
      url: process.env.REDIS_URL || 'redis://localhost:6379' 
    });
    const subClient = pubClient.duplicate();

    await Promise.all([pubClient.connect(), subClient.connect()]);

    this.adapterConstructor = createAdapter(pubClient, subClient);
  }

  createIOServer(port: number, options?: ServerOptions) {
    const server = super.createIOServer(port, options);
    server.adapter(this.adapterConstructor);
    return server;
  }
}

// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  const redisAdapter = new RedisSocketIOAdapter(app);
  await redisAdapter.connectToRedis();
  
  app.useWebSocketAdapter(redisAdapter);
  
  await app.listen(3000);
}
bootstrap();
```

### **Client-Side Setup**

**Browser (React/Vue/Angular):**

```bash
# Install Socket.IO client
npm install socket.io-client
```

```typescript
// React component
import { useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';

function ChatComponent() {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [messages, setMessages] = useState<string[]>([]);

  useEffect(() => {
    // Connect to WebSocket server
    const newSocket = io('http://localhost:3000', {
      transports: ['websocket'],
      reconnection: true,
    });

    newSocket.on('connect', () => {
      console.log('Connected to WebSocket server');
    });

    newSocket.on('message', (data) => {
      setMessages((prev) => [...prev, data]);
    });

    setSocket(newSocket);

    return () => {
      newSocket.close();
    };
  }, []);

  const sendMessage = (text: string) => {
    if (socket) {
      socket.emit('message', text);
    }
  };

  return (
    <div>
      {messages.map((msg, idx) => (
        <div key={idx}>{msg}</div>
      ))}
      <button onClick={() => sendMessage('Hello!')}>Send</button>
    </div>
  );
}
```

**CDN (No build step):**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Socket.IO Chat</title>
  <!-- Include Socket.IO client from CDN -->
  <script src="https://cdn.socket.io/4.6.0/socket.io.min.js"></script>
</head>
<body>
  <script>
    // Connect to WebSocket server
    const socket = io('http://localhost:3000');

    socket.on('connect', () => {
      console.log('Connected:', socket.id);
    });

    socket.on('message', (data) => {
      console.log('Message:', data);
    });

    // Send message
    socket.emit('message', 'Hello from browser!');
  </script>
</body>
</html>
```

### **Testing Setup**

```bash
# Install testing utilities
npm install --save-dev @nestjs/testing socket.io-client
```

```typescript
// chat.gateway.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { ChatGateway } from './chat.gateway';
import { INestApplication } from '@nestjs/common';
import { io, Socket } from 'socket.io-client';

describe('ChatGateway', () => {
  let app: INestApplication;
  let gateway: ChatGateway;
  let clientSocket: Socket;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      providers: [ChatGateway],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.listen(3001);

    gateway = moduleFixture.get<ChatGateway>(ChatGateway);

    // Connect test client
    clientSocket = io('http://localhost:3001', {
      transports: ['websocket'],
    });
  });

  afterAll(async () => {
    clientSocket.close();
    await app.close();
  });

  it('should connect successfully', (done) => {
    clientSocket.on('connect', () => {
      expect(clientSocket.connected).toBe(true);
      done();
    });
  });

  it('should receive message', (done) => {
    clientSocket.emit('message', 'test');
    clientSocket.on('message', (data) => {
      expect(data).toBe('Hello from server!');
      done();
    });
  });
});
```

### **Common Installation Issues**

```typescript
// ERROR 1: Peer dependency warnings
// npm WARN @nestjs/platform-socket.io@10.0.0 requires a peer of socket.io@^4.0.0

// SOLUTION: Install peer dependencies
npm install socket.io@^4.0.0

// ERROR 2: TypeScript types missing
// Cannot find module 'socket.io' or its corresponding type declarations

// SOLUTION: Install types
npm install --save-dev @types/socket.io

// ERROR 3: Version mismatch
// Error: Socket.IO server version incompatible with client

// SOLUTION: Match versions
npm install socket.io@4.6.0 socket.io-client@4.6.0

// ERROR 4: CORS issues
// Access to XMLHttpRequest has been blocked by CORS policy

// SOLUTION: Configure CORS in gateway
@WebSocketGateway({
  cors: {
    origin: 'http://localhost:3001',
    credentials: true,
  },
})
```

### **Version Compatibility**

```typescript
/**
 * Recommended versions (as of 2024):
 * 
 * @nestjs/websockets: ^10.0.0
 * @nestjs/platform-socket.io: ^10.0.0
 * socket.io: ^4.6.0
 * socket.io-client: ^4.6.0
 * 
 * Note: Keep NestJS packages at same major version
 * Note: Keep socket.io and socket.io-client at same version
 */

// Check compatibility
const compatibility = {
  nestjs: '^10.0.0',
  socketio: '^4.6.0',
  node: '>=16.0.0',
  typescript: '>=4.9.0',
};
```

**Interview Tip**: Install WebSocket dependencies with `npm install @nestjs/websockets @nestjs/platform-socket.io socket.io socket.io-client`. **Four packages needed**: `@nestjs/websockets` (core decorators/interfaces), `@nestjs/platform-socket.io` (Socket.IO adapter), `socket.io` (server), `socket.io-client` (client for testing/Node.js). **For native WebSocket**: use `@nestjs/platform-ws ws` instead of Socket.IO packages. **Setup**: create gateway with `@WebSocketGateway()`, register in module, start app. **Production**: add Redis adapter (`@socket.io/redis-adapter redis`) for multi-server scaling. **Client**: install `socket.io-client` in frontend or use CDN. **Testing**: use `socket.io-client` to create test connections. **Common issue**: peer dependency warnings (install `socket.io` explicitly), CORS errors (configure in gateway decorator). **Version matching**: keep socket.io and socket.io-client at same version.

</details>

7. What is a WebSocket Gateway in NestJS?

<details>
<summary><strong>Answer</strong></summary>

**A WebSocket Gateway** in NestJS is a class decorated with `@WebSocketGateway()` that handles WebSocket connections and events. It's the equivalent of a Controller for HTTP requests, but for WebSocket communication. Gateways provide a structured way to manage real-time bidirectional communication between the server and connected clients.

### **Core Concept**

```typescript
/**
 * WebSocket Gateway = Controller for WebSocket connections
 * 
 * Controller (HTTP)          vs    Gateway (WebSocket)
 * -------------------              --------------------
 * @Controller()                     @WebSocketGateway()
 * @Get(), @Post()                   @SubscribeMessage()
 * Request/Response                 Event-based
 * Stateless                        Stateful (maintains connection)
 * req, res objects                 Socket object
 */

// HTTP Controller
@Controller('users')
export class UsersController {
  @Get(':id')
  getUser(@Param('id') id: string) {
    return { id, name: 'John' };
  }
}

// WebSocket Gateway (similar structure)
@WebSocketGateway()
export class UsersGateway {
  @SubscribeMessage('getUser')
  handleGetUser(@MessageBody() id: string) {
    return { id, name: 'John' };
  }
}
```

### **Basic Gateway Structure**

```typescript
// chat.gateway.ts
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayInit,
  OnGatewayConnection,
  OnGatewayDisconnect,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway({ cors: true }) // Decorator makes it a gateway
export class ChatGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  // 1. Server instance (like Express app in HTTP)
  @WebSocketServer()
  server: Server;

  private logger = new Logger('ChatGateway');

  // 2. Lifecycle hook: After gateway initialized
  afterInit(server: Server) {
    this.logger.log('Gateway initialized');
  }

  // 3. Lifecycle hook: Client connects
  handleConnection(client: Socket, ...args: any[]) {
    this.logger.log(`Client connected: ${client.id}`);
  }

  // 4. Lifecycle hook: Client disconnects
  handleDisconnect(client: Socket) {
    this.logger.log(`Client disconnected: ${client.id}`);
  }

  // 5. Event handlers (like @Get, @Post in controllers)
  @SubscribeMessage('message')
  handleMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: string,
  ) {
    this.logger.log(`Message from ${client.id}: ${data}`);
    return { event: 'message', data: `Echo: ${data}` };
  }
}
```

### **Gateway Responsibilities**

```typescript
/**
 * A Gateway handles:
 * 
 * 1. Connection Management
 *    - Accept/reject connections
 *    - Track connected clients
 *    - Handle disconnections
 * 
 * 2. Event Handling
 *    - Listen to client events
 *    - Process incoming messages
 *    - Emit events to clients
 * 
 * 3. Broadcasting
 *    - Send to all clients
 *    - Send to specific rooms
 *    - Send to specific clients
 * 
 * 4. State Management
 *    - Track user sessions
 *    - Manage rooms/namespaces
 *    - Store connection metadata
 */

@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  // Store connected users (state management)
  private connectedUsers: Map<string, UserInfo> = new Map();

  // 1. CONNECTION MANAGEMENT
  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    this.connectedUsers.set(client.id, {
      userId,
      socketId: client.id,
      connectedAt: new Date(),
    });
    
    // Notify others about new connection
    client.broadcast.emit('user-online', { userId });
  }

  handleDisconnect(client: Socket) {
    const user = this.connectedUsers.get(client.id);
    if (user) {
      this.connectedUsers.delete(client.id);
      client.broadcast.emit('user-offline', { userId: user.userId });
    }
  }

  // 2. EVENT HANDLING
  @SubscribeMessage('send-message')
  handleMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { message: string; roomId: string },
  ) {
    // Process message (business logic)
    const user = this.connectedUsers.get(client.id);
    
    // 3. BROADCASTING
    this.server.to(data.roomId).emit('new-message', {
      userId: user.userId,
      message: data.message,
      timestamp: new Date(),
    });
  }
}

interface UserInfo {
  userId: string;
  socketId: string;
  connectedAt: Date;
}
```

### **Gateway vs Controller Comparison**

```typescript
// HTTP CONTROLLER (Traditional REST API)
@Controller('chat')
export class ChatController {
  constructor(private chatService: ChatService) {}

  // Stateless: Each request is independent
  @Get('messages')
  async getMessages(@Query('roomId') roomId: string) {
    return this.chatService.getMessages(roomId);
  }

  @Post('messages')
  async sendMessage(@Body() dto: SendMessageDto) {
    const message = await this.chatService.createMessage(dto);
    // Can't push to other users - they must poll!
    return message;
  }
}

// WEBSOCKET GATEWAY (Real-time)
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  constructor(private chatService: ChatService) {}

  // Stateful: Connection persists
  @SubscribeMessage('get-messages')
  async handleGetMessages(@MessageBody() roomId: string) {
    return this.chatService.getMessages(roomId);
  }

  @SubscribeMessage('send-message')
  async handleSendMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() dto: SendMessageDto,
  ) {
    const message = await this.chatService.createMessage(dto);
    
    // Push to all users in room instantly!
    this.server.to(dto.roomId).emit('new-message', message);
    
    return message;
  }
}
```

### **Gateway Registration**

```typescript
// 1. Create Gateway file
// chat.gateway.ts
@WebSocketGateway()
export class ChatGateway {
  // Implementation
}

// 2. Register in Module
// chat.module.ts
import { Module } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';
import { ChatService } from './chat.service';

@Module({
  providers: [
    ChatGateway,  // Register as provider (like controllers)
    ChatService,
  ],
})
export class ChatModule {}

// 3. Import Module in App
// app.module.ts
import { Module } from '@nestjs/common';
import { ChatModule } from './chat/chat.module';

@Module({
  imports: [ChatModule], // Gateway automatically activated
})
export class AppModule {}

// Note: Unlike Controllers, Gateways are registered as PROVIDERS, not controllers
```

### **Dependency Injection in Gateways**

```typescript
// Gateways support full dependency injection
@WebSocketGateway()
export class ChatGateway {
  constructor(
    // Inject services
    private chatService: ChatService,
    private usersService: UsersService,
    private notificationService: NotificationService,
    
    // Inject repositories
    @InjectRepository(Message)
    private messageRepository: Repository<Message>,
    
    // Inject configuration
    private configService: ConfigService,
  ) {}

  @SubscribeMessage('send-message')
  async handleMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: { message: string; userId: string },
  ) {
    // Use injected services
    const user = await this.usersService.findOne(data.userId);
    const message = await this.chatService.createMessage(data);
    
    // Save to database
    await this.messageRepository.save(message);
    
    // Send notification
    await this.notificationService.notify(user.id, 'New message');
    
    // Broadcast
    this.server.emit('new-message', message);
  }
}
```

### **Multiple Gateways in One Application**

```typescript
// You can have multiple gateways for different purposes

// 1. Chat Gateway (namespace: /chat)
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  @SubscribeMessage('send-message')
  handleChatMessage() { /* ... */ }
}

// 2. Notifications Gateway (namespace: /notifications)
@WebSocketGateway({ namespace: '/notifications' })
export class NotificationsGateway {
  @SubscribeMessage('subscribe')
  handleSubscribe() { /* ... */ }
}

// 3. Admin Gateway (namespace: /admin, different port)
@WebSocketGateway({ namespace: '/admin', port: 3001 })
export class AdminGateway {
  @SubscribeMessage('get-stats')
  handleGetStats() { /* ... */ }
}

// 4. Game Gateway (namespace: /game)
@WebSocketGateway({ namespace: '/game' })
export class GameGateway {
  @SubscribeMessage('player-move')
  handlePlayerMove() { /* ... */ }
}

// Each gateway is independent and handles its own connections
// Client connects to specific namespace:
// const chatSocket = io('http://localhost:3000/chat');
// const notificationSocket = io('http://localhost:3000/notifications');
```

### **Gateway with Guards and Interceptors**

```typescript
// Gateways support Guards, Interceptors, Pipes like Controllers

import { UseGuards, UseInterceptors, UsePipes } from '@nestjs/common';
import { WsJwtGuard } from './guards/ws-jwt.guard';
import { LoggingInterceptor } from './interceptors/logging.interceptor';
import { ValidationPipe } from '@nestjs/common';

@WebSocketGateway()
@UseGuards(WsJwtGuard) // Apply to entire gateway
@UseInterceptors(LoggingInterceptor)
export class ChatGateway {
  @SubscribeMessage('send-message')
  @UsePipes(new ValidationPipe()) // Validate incoming data
  handleMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: SendMessageDto, // Validated DTO
  ) {
    // Only authenticated users reach here
    return { success: true };
  }
}

// DTO with validation
export class SendMessageDto {
  @IsNotEmpty()
  @IsString()
  message: string;

  @IsNotEmpty()
  @IsUUID()
  roomId: string;
}
```

### **Gateway Lifecycle**

```typescript
/**
 * Gateway Lifecycle (in order):
 * 
 * 1. Gateway class instantiated (constructor)
 * 2. Dependencies injected
 * 3. afterInit() called (OnGatewayInit)
 * 4. Server starts listening
 * 5. Client connects → handleConnection() (OnGatewayConnection)
 * 6. Client sends event → @SubscribeMessage handler
 * 7. Client disconnects → handleDisconnect() (OnGatewayDisconnect)
 */

@WebSocketGateway()
export class LifecycleGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  private logger = new Logger('LifecycleGateway');

  // 1. Constructor (first)
  constructor(private someService: SomeService) {
    this.logger.log('1. Constructor called');
  }

  // 2. After Init (second)
  afterInit(server: Server) {
    this.logger.log('2. Gateway initialized, server ready');
  }

  // 3. Connection (when client connects)
  handleConnection(client: Socket) {
    this.logger.log(`3. Client connected: ${client.id}`);
  }

  // 4. Event handler (when client sends event)
  @SubscribeMessage('test')
  handleTest() {
    this.logger.log('4. Event received');
  }

  // 5. Disconnection (when client disconnects)
  handleDisconnect(client: Socket) {
    this.logger.log(`5. Client disconnected: ${client.id}`);
  }
}
```

### **Production Gateway Example**

```typescript
// Complete production-ready gateway
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayInit,
  OnGatewayConnection,
  OnGatewayDisconnect,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger, UseGuards, UsePipes, ValidationPipe } from '@nestjs/common';
import { WsJwtGuard } from './guards/ws-jwt.guard';

@WebSocketGateway({
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(','),
    credentials: true,
  },
  namespace: '/chat',
})
@UseGuards(WsJwtGuard)
export class ChatGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  @WebSocketServer()
  server: Server;

  private logger = new Logger('ChatGateway');
  private connectedClients: Map<string, ClientInfo> = new Map();

  constructor(
    private chatService: ChatService,
    private usersService: UsersService,
  ) {}

  afterInit(server: Server) {
    this.logger.log('Chat Gateway initialized');
    
    // Setup periodic cleanup
    setInterval(() => {
      this.cleanupInactiveConnections();
    }, 300000); // Every 5 minutes
  }

  async handleConnection(client: Socket) {
    try {
      const userId = client.handshake.auth.userId;
      
      // Store client info
      this.connectedClients.set(client.id, {
        userId,
        socketId: client.id,
        connectedAt: new Date(),
        lastActivity: new Date(),
      });

      // Join user's personal room
      client.join(`user:${userId}`);

      // Broadcast online status
      client.broadcast.emit('user:online', { userId });

      this.logger.log(`Client connected: ${client.id} (User: ${userId})`);
    } catch (error) {
      this.logger.error('Connection error:', error);
      client.disconnect();
    }
  }

  handleDisconnect(client: Socket) {
    const clientInfo = this.connectedClients.get(client.id);
    
    if (clientInfo) {
      // Broadcast offline status
      client.broadcast.emit('user:offline', { userId: clientInfo.userId });
      
      // Cleanup
      this.connectedClients.delete(client.id);
      
      this.logger.log(`Client disconnected: ${client.id}`);
    }
  }

  @SubscribeMessage('join-room')
  @UsePipes(new ValidationPipe())
  async handleJoinRoom(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: JoinRoomDto,
  ) {
    client.join(data.roomId);
    
    // Update last activity
    const clientInfo = this.connectedClients.get(client.id);
    if (clientInfo) {
      clientInfo.lastActivity = new Date();
    }

    // Notify room members
    client.to(data.roomId).emit('user-joined', {
      userId: clientInfo.userId,
      roomId: data.roomId,
    });

    return { success: true, roomId: data.roomId };
  }

  @SubscribeMessage('send-message')
  @UsePipes(new ValidationPipe())
  async handleSendMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: SendMessageDto,
  ) {
    const clientInfo = this.connectedClients.get(client.id);
    
    // Save message to database
    const message = await this.chatService.createMessage({
      userId: clientInfo.userId,
      roomId: data.roomId,
      content: data.message,
    });

    // Broadcast to room
    this.server.to(data.roomId).emit('new-message', {
      id: message.id,
      userId: clientInfo.userId,
      content: data.message,
      timestamp: message.createdAt,
    });

    // Update last activity
    clientInfo.lastActivity = new Date();

    return { success: true, messageId: message.id };
  }

  private cleanupInactiveConnections() {
    const now = new Date();
    const timeout = 30 * 60 * 1000; // 30 minutes

    for (const [socketId, clientInfo] of this.connectedClients.entries()) {
      if (now.getTime() - clientInfo.lastActivity.getTime() > timeout) {
        this.logger.warn(`Disconnecting inactive client: ${socketId}`);
        this.server.to(socketId).disconnectSockets();
        this.connectedClients.delete(socketId);
      }
    }
  }
}

interface ClientInfo {
  userId: string;
  socketId: string;
  connectedAt: Date;
  lastActivity: Date;
}
```

**Interview Tip**: **WebSocket Gateway** is a class with `@WebSocketGateway()` decorator that handles WebSocket connections and events - it's the **WebSocket equivalent of a Controller** (Controller handles HTTP, Gateway handles WebSocket). **Key features**: lifecycle hooks (`afterInit`, `handleConnection`, `handleDisconnect`), event handlers with `@SubscribeMessage()`, access to Socket.IO server via `@WebSocketServer()`, supports dependency injection, guards, interceptors, pipes. **Registration**: add as provider in module (not in controllers array). **Responsibilities**: connection management (accept/reject, track clients), event handling (listen/process messages), broadcasting (send to clients/rooms), state management (track sessions/rooms). **Multiple gateways**: use different namespaces (`/chat`, `/notifications`) or ports. **Lifecycle order**: constructor → afterInit → handleConnection → event handlers → handleDisconnect. **Production**: implement cleanup for inactive connections, validate DTOs with pipes, authenticate with guards, log all events.

</details>

8. What is `@WebSocketGateway()` decorator?

<details>
<summary><strong>Answer</strong></summary>

**`@WebSocketGateway()`** is a class decorator that transforms a regular TypeScript class into a WebSocket gateway, enabling it to handle WebSocket connections and events. It configures the gateway's behavior including port, namespace, CORS settings, and Socket.IO options.

### **Basic Usage**

```typescript
// Simplest form (uses default port where app is running)
@WebSocketGateway()
export class BasicGateway {
  @WebSocketServer()
  server: Server;
}

// The decorator does several things:
// 1. Registers the class as a WebSocket gateway
// 2. Sets up Socket.IO server
// 3. Enables WebSocket event handling
// 4. Integrates with NestJS dependency injection
```

### **Decorator Signature**

```typescript
// Full signature
@WebSocketGateway(port?: number, options?: GatewayMetadata)

// GatewayMetadata interface
interface GatewayMetadata {
  namespace?: string;           // Socket.IO namespace (e.g., '/chat')
  path?: string;                // Path for Socket.IO server
  cors?: CorsOptions;           // CORS configuration
  transports?: Transport[];     // Transport protocols
  adapter?: any;                // Custom adapter (e.g., Redis)
  // ... and many more Socket.IO options
}
```

### **Configuration Options**

#### **1. Port Configuration**

```typescript
// DEFAULT: Same port as HTTP server
@WebSocketGateway()
export class DefaultPortGateway {
  // Runs on same port as main app (e.g., 3000)
}

// CUSTOM PORT: Separate port for WebSocket
@WebSocketGateway(3001)
export class CustomPortGateway {
  // WebSocket server runs on port 3001
  // HTTP server might be on port 3000
}

// Client connection:
// const socket = io('http://localhost:3001');
```

#### **2. Namespace Configuration**

```typescript
// DEFAULT NAMESPACE: '/'
@WebSocketGateway()
export class DefaultNamespaceGateway {
  // Accessible at: http://localhost:3000
}

// CUSTOM NAMESPACE: Separate channel
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  // Accessible at: http://localhost:3000/chat
}

@WebSocketGateway({ namespace: '/notifications' })
export class NotificationsGateway {
  // Accessible at: http://localhost:3000/notifications
}

@WebSocketGateway({ namespace: '/admin' })
export class AdminGateway {
  // Accessible at: http://localhost:3000/admin
}

// Client connections:
// const chatSocket = io('http://localhost:3000/chat');
// const notificationSocket = io('http://localhost:3000/notifications');
// const adminSocket = io('http://localhost:3000/admin');
```

#### **3. CORS Configuration**

```typescript
// ALLOW ALL ORIGINS (development only!)
@WebSocketGateway({ cors: true })
export class AllowAllGateway {}

// SPECIFIC ORIGIN
@WebSocketGateway({
  cors: {
    origin: 'http://localhost:3001',
  },
})
export class SingleOriginGateway {}

// MULTIPLE ORIGINS
@WebSocketGateway({
  cors: {
    origin: ['http://localhost:3001', 'http://localhost:3002'],
    credentials: true, // Allow cookies
  },
})
export class MultipleOriginsGateway {}

// DYNAMIC ORIGIN (function)
@WebSocketGateway({
  cors: {
    origin: (origin, callback) => {
      // Check if origin is allowed
      const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
      if (allowedOrigins.includes(origin) || !origin) {
        callback(null, true);
      } else {
        callback(new Error('Not allowed by CORS'));
      }
    },
    credentials: true,
  },
})
export class DynamicCorsGateway {}

// PRODUCTION CONFIG
@WebSocketGateway({
  cors: {
    origin: process.env.FRONTEND_URL || 'https://myapp.com',
    methods: ['GET', 'POST'],
    allowedHeaders: ['Authorization'],
    credentials: true,
  },
})
export class ProductionGateway {}
```

#### **4. Transport Configuration**

```typescript
// DEFAULT: Both WebSocket and polling
@WebSocketGateway()
export class DefaultTransportGateway {}

// WEBSOCKET ONLY (no fallback)
@WebSocketGateway({
  transports: ['websocket'],
})
export class WebSocketOnlyGateway {
  // Faster but may fail in restricted networks
}

// POLLING ONLY (HTTP long-polling)
@WebSocketGateway({
  transports: ['polling'],
})
export class PollingOnlyGateway {
  // More compatible but slower
}

// CUSTOM ORDER (try WebSocket first, fallback to polling)
@WebSocketGateway({
  transports: ['websocket', 'polling'],
})
export class PreferWebSocketGateway {}
```

#### **5. Path Configuration**

```typescript
// DEFAULT PATH: '/socket.io'
@WebSocketGateway()
export class DefaultPathGateway {
  // Client connects to: http://localhost:3000/socket.io
}

// CUSTOM PATH
@WebSocketGateway({
  path: '/ws', // Custom path
})
export class CustomPathGateway {
  // Client connects to: http://localhost:3000/ws
}

// Client must specify custom path:
const socket = io('http://localhost:3000', {
  path: '/ws',
});
```

#### **6. Connection Timeout Configuration**

```typescript
@WebSocketGateway({
  pingTimeout: 60000,      // 60 seconds (default: 60000)
  pingInterval: 25000,     // 25 seconds (default: 25000)
  upgradeTimeout: 10000,   // 10 seconds (default: 10000)
  maxHttpBufferSize: 1e6,  // 1MB (default: 1e6)
})
export class TimeoutConfigGateway {
  // pingInterval: How often to send ping to client
  // pingTimeout: How long to wait for pong before disconnect
  // upgradeTimeout: How long to wait for WebSocket upgrade
  // maxHttpBufferSize: Max size of HTTP request during handshake
}
```

### **Complete Configuration Example**

```typescript
@WebSocketGateway({
  // Namespace for isolation
  namespace: '/chat',
  
  // CORS configuration
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3001'],
    credentials: true,
    methods: ['GET', 'POST'],
  },
  
  // Transport protocols
  transports: ['websocket', 'polling'],
  
  // Connection settings
  pingTimeout: 60000,
  pingInterval: 25000,
  
  // WebSocket upgrade settings
  upgradeTimeout: 10000,
  
  // Buffer size
  maxHttpBufferSize: 1e6, // 1MB
  
  // Allow older Engine.IO clients
  allowEIO3: true,
  
  // Cookie settings
  cookie: {
    name: 'io',
    path: '/',
    httpOnly: true,
    sameSite: 'lax',
  },
  
  // Connection state recovery (Socket.IO v4.6+)
  connectionStateRecovery: {
    maxDisconnectionDuration: 2 * 60 * 1000, // 2 minutes
    skipMiddlewares: true,
  },
})
export class ComprehensiveGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  handleConnection(client: Socket) {
    console.log(`Client connected: ${client.id}`);
  }
}
```

### **Multiple Gateways with Different Configurations**

```typescript
// 1. Public Chat Gateway (permissive CORS)
@WebSocketGateway({
  namespace: '/chat',
  cors: { origin: '*' }, // Open to all
})
export class PublicChatGateway {}

// 2. Admin Gateway (restricted, authenticated)
@WebSocketGateway({
  namespace: '/admin',
  cors: {
    origin: ['https://admin.myapp.com'],
    credentials: true,
  },
})
export class AdminGateway {}

// 3. Real-time Analytics (WebSocket only, no polling)
@WebSocketGateway({
  namespace: '/analytics',
  transports: ['websocket'],
  cors: {
    origin: ['https://dashboard.myapp.com'],
  },
})
export class AnalyticsGateway {}

// 4. IoT Devices (separate port, custom path)
@WebSocketGateway(3002, {
  path: '/iot',
  cors: false, // No CORS for IoT devices
})
export class IoTGateway {}
```

### **Environment-Based Configuration**

```typescript
// Dynamic configuration based on environment
import { ConfigService } from '@nestjs/config';

// Create factory function
export function createGatewayConfig(configService: ConfigService) {
  const isDevelopment = configService.get('NODE_ENV') === 'development';
  
  return {
    namespace: '/chat',
    cors: isDevelopment
      ? { origin: '*' } // Development: allow all
      : {
          origin: configService.get('ALLOWED_ORIGINS').split(','),
          credentials: true,
        }, // Production: restricted
    transports: isDevelopment
      ? ['websocket', 'polling'] // Development: both
      : ['websocket'], // Production: WebSocket only
  };
}

// Use in gateway
@WebSocketGateway({
  namespace: '/chat',
  cors: {
    origin: process.env.NODE_ENV === 'development' 
      ? '*' 
      : process.env.ALLOWED_ORIGINS?.split(','),
    credentials: true,
  },
})
export class EnvironmentAwareGateway {}
```

### **Decorator Registration Process**

```typescript
/**
 * What @WebSocketGateway() does internally:
 * 
 * 1. Registers class as WebSocket gateway provider
 * 2. Creates Socket.IO server with specified options
 * 3. Binds server to specified port/namespace
 * 4. Sets up event listeners (@SubscribeMessage handlers)
 * 5. Integrates with NestJS lifecycle hooks
 * 6. Enables dependency injection
 * 7. Applies middleware, guards, interceptors
 */

// Before decorator:
class ChatGateway {
  // Just a regular class
}

// After decorator:
@WebSocketGateway({ namespace: '/chat' })
class ChatGateway {
  // Now it's:
  // - Registered in NestJS IoC container
  // - Has Socket.IO server instance
  // - Listening for connections on /chat namespace
  // - Ready to handle events
}
```

### **Custom Decorator (Advanced)**

```typescript
// Create custom decorator with predefined config
import { applyDecorators } from '@nestjs/common';
import { WebSocketGateway } from '@nestjs/websockets';

export function SecureGateway(namespace: string) {
  return applyDecorators(
    WebSocketGateway({
      namespace,
      cors: {
        origin: process.env.ALLOWED_ORIGINS?.split(','),
        credentials: true,
      },
      transports: ['websocket'],
      pingTimeout: 60000,
      pingInterval: 25000,
    }),
  );
}

// Usage:
@SecureGateway('/chat')
export class ChatGateway {
  // Automatically configured with secure settings
}

@SecureGateway('/admin')
export class AdminGateway {
  // Same secure configuration
}
```

### **Common Configuration Patterns**

```typescript
// PATTERN 1: Development Gateway (permissive)
@WebSocketGateway({
  cors: true, // Allow all origins
  transports: ['websocket', 'polling'], // All transports
})
export class DevGateway {}

// PATTERN 2: Production Gateway (strict)
@WebSocketGateway({
  cors: {
    origin: process.env.FRONTEND_URL,
    credentials: true,
  },
  transports: ['websocket'], // WebSocket only
  pingTimeout: 60000,
  pingInterval: 25000,
})
export class ProdGateway {}

// PATTERN 3: Internal Gateway (no CORS)
@WebSocketGateway({
  namespace: '/internal',
  cors: false, // Same-origin only
})
export class InternalGateway {}

// PATTERN 4: Public API Gateway (rate-limited)
@WebSocketGateway({
  namespace: '/public-api',
  cors: { origin: '*' },
  maxHttpBufferSize: 100000, // 100KB limit
  pingTimeout: 30000, // Shorter timeout
})
export class PublicApiGateway {}
```

### **Testing Configuration**

```typescript
// Test with different configurations
describe('Gateway Configuration', () => {
  it('should configure CORS correctly', async () => {
    @WebSocketGateway({
      cors: {
        origin: 'http://localhost:3001',
      },
    })
    class TestGateway {}

    const module = await Test.createTestingModule({
      providers: [TestGateway],
    }).compile();

    const gateway = module.get<TestGateway>(TestGateway);
    expect(gateway).toBeDefined();
  });

  it('should use custom namespace', async () => {
    @WebSocketGateway({ namespace: '/test' })
    class TestGateway {
      @WebSocketServer()
      server: Server;
    }

    const app = await NestFactory.create(AppModule);
    await app.listen(3000);

    const socket = io('http://localhost:3000/test');
    await new Promise((resolve) => socket.on('connect', resolve));
    
    expect(socket.connected).toBe(true);
    socket.close();
  });
});
```

**Interview Tip**: **`@WebSocketGateway()`** is a class decorator that **registers a class as a WebSocket gateway** and configures Socket.IO server. **Common options**: `namespace` (separate channels like `/chat`, `/notifications`), `cors` (origin restrictions), `transports` (`['websocket', 'polling']`), `pingTimeout`/`pingInterval` (connection health), `maxHttpBufferSize` (request limit). **Default behavior**: same port as HTTP server, default namespace (`/`), both transports enabled. **CORS configuration critical**: use `cors: true` for development, restrict origins in production (`origin: process.env.FRONTEND_URL`). **Multiple gateways**: use different namespaces or ports for isolation. **Client connection**: must match namespace (`io('http://localhost:3000/chat')`). **Production**: use environment variables for dynamic config, enable `credentials: true` for cookies, use `transports: ['websocket']` for performance. **Registration**: add gateway as provider in module (not controllers array).

</details>

9. How do you configure WebSocket gateway options (port, namespace, cors)?

<details>
<summary><strong>Answer</strong></summary>

WebSocket gateway options are configured by passing configuration objects to the `@WebSocketGateway()` decorator. The most important options are **port** (server port), **namespace** (channel isolation), and **cors** (cross-origin security).

### **1. Port Configuration**

```typescript
// METHOD 1: Default port (same as HTTP server)
@WebSocketGateway()
export class DefaultPortGateway {
  // If HTTP server runs on port 3000, WebSocket also on 3000
}

// METHOD 2: Custom port (number as first parameter)
@WebSocketGateway(3001)
export class CustomPortGateway {
  // WebSocket runs on port 3001 (separate from HTTP)
}

// METHOD 3: Port in options object
@WebSocketGateway({ port: 3002 })
export class PortInOptionsGateway {}

// Usage Example:
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000); // HTTP on 3000
  // DefaultPortGateway will use 3000
  // CustomPortGateway will use 3001
}

// Client connections:
const defaultSocket = io('http://localhost:3000');
const customSocket = io('http://localhost:3001');
```

### **2. Namespace Configuration**

```typescript
// DEFAULT NAMESPACE: '/' (root)
@WebSocketGateway()
export class RootGateway {
  // Accessible at: ws://localhost:3000/
}

// CUSTOM NAMESPACE: Isolated channel
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  // Accessible at: ws://localhost:3000/chat
}

@WebSocketGateway({ namespace: '/notifications' })
export class NotificationsGateway {
  // Accessible at: ws://localhost:3000/notifications
}

@WebSocketGateway({ namespace: '/admin' })
export class AdminGateway {
  // Accessible at: ws://localhost:3000/admin
}

// Client connections (must specify namespace):
const chatSocket = io('http://localhost:3000/chat');
const notificationSocket = io('http://localhost:3000/notifications');
const adminSocket = io('http://localhost:3000/admin');

// Why use namespaces?
// 1. Logical separation (chat, notifications, admin)
// 2. Different authentication per namespace
// 3. Different middleware per namespace
// 4. Isolated rooms per namespace
// 5. Better organization for large apps
```

### **3. CORS Configuration**

```typescript
// OPTION 1: Allow all origins (DEVELOPMENT ONLY!)
@WebSocketGateway({ cors: true })
export class OpenCorsGateway {
  // ⚠️ WARNING: Security risk in production!
}

// OPTION 2: Specific origin (string)
@WebSocketGateway({
  cors: {
    origin: 'http://localhost:3001',
  },
})
export class SingleOriginGateway {
  // Only allows requests from http://localhost:3001
}

// OPTION 3: Multiple origins (array)
@WebSocketGateway({
  cors: {
    origin: [
      'http://localhost:3001',
      'http://localhost:3002',
      'https://myapp.com',
    ],
    credentials: true, // Allow cookies/auth headers
  },
})
export class MultipleOriginsGateway {}

// OPTION 4: Dynamic origin (function)
@WebSocketGateway({
  cors: {
    origin: (origin, callback) => {
      // Custom logic to allow/deny origin
      const allowedOrigins = [
        'http://localhost:3001',
        'https://myapp.com',
        'https://staging.myapp.com',
      ];
      
      if (!origin || allowedOrigins.includes(origin)) {
        callback(null, true); // Allow
      } else {
        callback(new Error('Not allowed by CORS')); // Deny
      }
    },
    credentials: true,
  },
})
export class DynamicCorsGateway {}

// OPTION 5: Regex pattern
@WebSocketGateway({
  cors: {
    origin: /\.myapp\.com$/, // Allow all *.myapp.com
    credentials: true,
  },
})
export class RegexCorsGateway {}

// OPTION 6: Environment-based (PRODUCTION PATTERN)
@WebSocketGateway({
  cors: {
    origin: process.env.NODE_ENV === 'production'
      ? process.env.FRONTEND_URL
      : 'http://localhost:3001',
    credentials: true,
    methods: ['GET', 'POST'],
    allowedHeaders: ['Authorization', 'Content-Type'],
  },
})
export class EnvironmentCorsGateway {}
```

### **Complete Configuration Example**

```typescript
// Production-ready gateway with all common options
import { WebSocketGateway, WebSocketServer } from '@nestjs/websockets';
import { Server } from 'socket.io';

@WebSocketGateway({
  // 1. PORT: Custom port (optional, defaults to HTTP port)
  // port: 3001,
  
  // 2. NAMESPACE: Logical separation
  namespace: '/chat',
  
  // 3. CORS: Cross-origin security
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(',') || [
      'http://localhost:3001',
      'https://myapp.com',
    ],
    credentials: true,           // Allow cookies/auth
    methods: ['GET', 'POST'],    // Allowed methods
    allowedHeaders: ['Authorization'], // Allowed headers
  },
  
  // 4. TRANSPORTS: Communication protocols
  transports: ['websocket', 'polling'], // Try WebSocket, fallback to polling
  
  // 5. PATH: Socket.IO endpoint path
  path: '/socket.io', // Default, can be changed
  
  // 6. TIMEOUTS: Connection health
  pingTimeout: 60000,   // 60 seconds before disconnect
  pingInterval: 25000,  // Send ping every 25 seconds
  
  // 7. UPGRADE: WebSocket handshake
  upgradeTimeout: 10000, // 10 seconds to upgrade
  
  // 8. BUFFER SIZE: Request size limit
  maxHttpBufferSize: 1e6, // 1MB max
  
  // 9. COOKIE: Session management
  cookie: {
    name: 'io',
    path: '/',
    httpOnly: true,
    sameSite: 'lax',
  },
  
  // 10. CONNECTION STATE RECOVERY (Socket.IO v4.6+)
  connectionStateRecovery: {
    maxDisconnectionDuration: 2 * 60 * 1000, // 2 minutes
    skipMiddlewares: true,
  },
  
  // 11. ALLOW EIO3: Backward compatibility
  allowEIO3: true, // Support older clients
  
  // 12. SERVE CLIENT: Serve Socket.IO client files
  serveClient: false, // Don't serve client files (use CDN)
})
export class ProductionGateway {
  @WebSocketServer()
  server: Server;
}
```

### **Configuration Patterns by Environment**

```typescript
// PATTERN 1: Development Configuration
@WebSocketGateway({
  namespace: '/dev-chat',
  cors: true, // Allow all (convenient for testing)
  transports: ['websocket', 'polling'], // Both for debugging
})
export class DevelopmentGateway {}

// PATTERN 2: Production Configuration
@WebSocketGateway({
  namespace: '/chat',
  cors: {
    origin: process.env.FRONTEND_URL, // Single trusted origin
    credentials: true,
  },
  transports: ['websocket'], // WebSocket only (faster)
  pingTimeout: 60000,
  pingInterval: 25000,
  serveClient: false, // Use CDN for client files
})
export class ProductionGateway {}

// PATTERN 3: Testing Configuration
@WebSocketGateway({
  namespace: '/test',
  cors: true,
  transports: ['websocket'], // Faster for tests
  pingTimeout: 5000, // Shorter for faster tests
})
export class TestingGateway {}

// PATTERN 4: Internal/Microservices (No CORS)
@WebSocketGateway({
  namespace: '/internal',
  cors: false, // Same-origin only (backend to backend)
})
export class InternalGateway {}
```

### **Dynamic Configuration with Config Service**

```typescript
// config.service.ts - Centralized configuration
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class WebSocketConfigService {
  constructor(private configService: ConfigService) {}

  getGatewayConfig() {
    const env = this.configService.get('NODE_ENV');
    const isDev = env === 'development';

    return {
      namespace: '/chat',
      cors: isDev
        ? { origin: '*' }
        : {
            origin: this.configService.get('ALLOWED_ORIGINS').split(','),
            credentials: true,
          },
      transports: isDev ? ['websocket', 'polling'] : ['websocket'],
      pingTimeout: 60000,
      pingInterval: 25000,
    };
  }
}

// Use in gateway (note: decorator evaluated at compile time,
// so we can't directly inject ConfigService)
// Instead, use environment variables directly:

@WebSocketGateway({
  namespace: '/chat',
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3001'],
    credentials: true,
  },
})
export class ConfigurableGateway {}
```

### **Multiple Gateways with Different Configurations**

```typescript
// 1. Public Chat (Open CORS)
@WebSocketGateway({
  namespace: '/public-chat',
  cors: { origin: '*' },
})
export class PublicChatGateway {}

// 2. Private Messages (Restricted CORS)
@WebSocketGateway({
  namespace: '/private-messages',
  cors: {
    origin: ['https://myapp.com'],
    credentials: true,
  },
})
export class PrivateMessagesGateway {}

// 3. Admin Panel (Very restricted)
@WebSocketGateway({
  namespace: '/admin',
  cors: {
    origin: ['https://admin.myapp.com'],
    credentials: true,
  },
})
export class AdminPanelGateway {}

// 4. Analytics (Separate port, WebSocket only)
@WebSocketGateway(3002, {
  namespace: '/analytics',
  cors: {
    origin: ['https://dashboard.myapp.com'],
  },
  transports: ['websocket'], // No polling
})
export class AnalyticsGateway {}
```

### **Client-Side Configuration Matching**

```typescript
// Server: chat.gateway.ts
@WebSocketGateway({
  namespace: '/chat',
  path: '/ws',
  cors: {
    origin: 'http://localhost:3001',
    credentials: true,
  },
})
export class ChatGateway {}

// Client: Must match server config
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000/chat', {
  // Match namespace: '/chat'
  path: '/ws',              // Match custom path
  withCredentials: true,   // Match credentials: true
  transports: ['websocket'], // Match transports
});
```

### **Configuration Validation**

```typescript
// Validate configuration at runtime
import { Logger } from '@nestjs/common';

@WebSocketGateway({
  namespace: '/chat',
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(','),
    credentials: true,
  },
})
export class ValidatedGateway implements OnGatewayInit {
  private logger = new Logger('ValidatedGateway');

  afterInit(server: Server) {
    // Validate configuration
    if (!process.env.ALLOWED_ORIGINS) {
      this.logger.warn('ALLOWED_ORIGINS not set! Using default.');
    }

    const origins = process.env.ALLOWED_ORIGINS?.split(',');
    this.logger.log(`Gateway initialized on namespace /chat`);
    this.logger.log(`Allowed origins: ${origins?.join(', ')}`);
    
    // Log server configuration
    this.logger.log(`Server listening on port: ${(server as any).httpServer.address().port}`);
  }
}
```

### **Common Configuration Mistakes**

```typescript
// ❌ MISTAKE 1: CORS too permissive in production
@WebSocketGateway({ cors: true })
export class InsecureGateway {} // Anyone can connect!

// ✅ CORRECT: Restrict origins
@WebSocketGateway({
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(','),
    credentials: true,
  },
})
export class SecureGateway {}

// ❌ MISTAKE 2: Missing credentials flag
@WebSocketGateway({
  cors: { origin: 'http://localhost:3001' },
  // Missing credentials: true
})
export class NoCookiesGateway {} // Cookies won't work!

// ✅ CORRECT: Enable credentials
@WebSocketGateway({
  cors: {
    origin: 'http://localhost:3001',
    credentials: true, // Allow cookies/auth headers
  },
})
export class WithCookiesGateway {}

// ❌ MISTAKE 3: Client namespace mismatch
// Server:
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {}

// Client (wrong - missing namespace):
const socket = io('http://localhost:3000'); // Won't connect!

// Client (correct):
const socket = io('http://localhost:3000/chat'); // Connects!

// ❌ MISTAKE 4: Hardcoded origins
@WebSocketGateway({
  cors: {
    origin: 'http://localhost:3001', // Breaks in production!
  },
})
export class HardcodedGateway {}

// ✅ CORRECT: Use environment variables
@WebSocketGateway({
  cors: {
    origin: process.env.FRONTEND_URL,
  },
})
export class EnvironmentGateway {}
```

### **Testing Different Configurations**

```typescript
// Test CORS configuration
describe('Gateway CORS Configuration', () => {
  it('should allow configured origin', async () => {
    process.env.ALLOWED_ORIGINS = 'http://localhost:3001';
    
    @WebSocketGateway({
      cors: {
        origin: process.env.ALLOWED_ORIGINS?.split(','),
      },
    })
    class TestGateway {}

    const module = await Test.createTestingModule({
      providers: [TestGateway],
    }).compile();

    const app = module.createNestApplication();
    await app.listen(3000);

    const socket = io('http://localhost:3000', {
      extraHeaders: {
        origin: 'http://localhost:3001',
      },
    });

    await new Promise((resolve) => socket.on('connect', resolve));
    expect(socket.connected).toBe(true);
    
    socket.close();
    await app.close();
  });
});
```

**Interview Tip**: Configure WebSocket gateway options via `@WebSocketGateway()` decorator. **Three critical options**: (1) **Port** - number as first param (`@WebSocketGateway(3001)`) or defaults to HTTP port; (2) **Namespace** - logical channels (`namespace: '/chat'`), clients must match (`io('http://localhost:3000/chat')`); (3) **CORS** - origin restrictions (`cors: { origin: process.env.FRONTEND_URL, credentials: true }`). **Common pattern**: `cors: true` for development (allow all), restricted origins array for production. **Other options**: `transports` (websocket/polling), `pingTimeout`/`pingInterval` (connection health), `path` (custom endpoint), `maxHttpBufferSize` (request limit). **Best practice**: use environment variables for dynamic config, enable `credentials: true` for cookies, use `transports: ['websocket']` in production for performance. **Multiple gateways**: use different namespaces for separation (chat, notifications, admin). **Client must match**: namespace, path, and credential settings.

</details>

## Creating Gateways

10. How do you create a WebSocket Gateway?

<details>
<summary><strong>Answer</strong></summary>

Creating a WebSocket gateway in NestJS involves **5 steps**: (1) create a class, (2) add `@WebSocketGateway()` decorator, (3) inject `@WebSocketServer()`, (4) implement lifecycle interfaces, (5) register in module. The gateway handles connections and events.

### **Step-by-Step Gateway Creation**

```typescript
// STEP 1: Create gateway file
// chat.gateway.ts

import {
  WebSocketGateway,        // 1. Gateway decorator
  WebSocketServer,         // 2. Server instance decorator
  SubscribeMessage,        // 3. Event handler decorator
  OnGatewayInit,          // 4. Lifecycle interfaces
  OnGatewayConnection,
  OnGatewayDisconnect,
  ConnectedSocket,        // 5. Parameter decorators
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

// STEP 2: Add @WebSocketGateway() decorator
@WebSocketGateway({ cors: true })
export class ChatGateway
  // STEP 3: Implement lifecycle interfaces
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  // STEP 4: Inject Socket.IO server instance
  @WebSocketServer()
  server: Server;

  private logger = new Logger('ChatGateway');

  // STEP 5: Implement afterInit (OnGatewayInit)
  afterInit(server: Server) {
    this.logger.log('Gateway initialized');
  }

  // STEP 6: Implement handleConnection (OnGatewayConnection)
  handleConnection(client: Socket, ...args: any[]) {
    this.logger.log(`Client connected: ${client.id}`);
  }

  // STEP 7: Implement handleDisconnect (OnGatewayDisconnect)
  handleDisconnect(client: Socket) {
    this.logger.log(`Client disconnected: ${client.id}`);
  }

  // STEP 8: Add event handlers with @SubscribeMessage()
  @SubscribeMessage('message')
  handleMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: string,
  ): string {
    this.logger.log(`Message from ${client.id}: ${data}`);
    return `Echo: ${data}`;
  }
}

// STEP 9: Register in module
// chat.module.ts
import { Module } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';

@Module({
  providers: [ChatGateway], // Register as provider (not controller)
})
export class ChatModule {}

// STEP 10: Import module in app
// app.module.ts
import { Module } from '@nestjs/common';
import { ChatModule } from './chat/chat.module';

@Module({
  imports: [ChatModule],
})
export class AppModule {}
```

### **Minimal Gateway (Bare Minimum)**

```typescript
// Absolute minimum to create a working gateway
import { WebSocketGateway, WebSocketServer } from '@nestjs/websockets';
import { Server } from 'socket.io';

@WebSocketGateway() // 1. Decorator
export class MinimalGateway {
  @WebSocketServer() // 2. Server instance
  server: Server;
  
  // That's it! Gateway is ready to accept connections
  // But without lifecycle hooks or event handlers, it won't do anything useful
}
```

### **Complete Production Gateway**

```typescript
// Complete production-ready gateway with all features
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayInit,
  OnGatewayConnection,
  OnGatewayDisconnect,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import {
  Logger,
  UseGuards,
  UsePipes,
  ValidationPipe,
} from '@nestjs/common';
import { WsJwtGuard } from './guards/ws-jwt.guard';
import { ChatService } from './chat.service';

// 1. GATEWAY DECORATOR with configuration
@WebSocketGateway({
  namespace: '/chat',
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(','),
    credentials: true,
  },
})
// 2. GUARDS (optional - for authentication)
@UseGuards(WsJwtGuard)
export class ChatGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  // 3. SERVER INSTANCE
  @WebSocketServer()
  server: Server;

  private logger = new Logger('ChatGateway');
  
  // 4. STATE MANAGEMENT (track connected users)
  private connectedUsers: Map<string, UserInfo> = new Map();

  // 5. DEPENDENCY INJECTION
  constructor(
    private chatService: ChatService, // Inject services
  ) {}

  // 6. LIFECYCLE: After Init
  afterInit(server: Server) {
    this.logger.log('Chat Gateway initialized');
  }

  // 7. LIFECYCLE: Connection
  async handleConnection(client: Socket, ...args: any[]) {
    try {
      // Extract user info from handshake
      const userId = client.handshake.auth.userId;
      
      // Validate user (if needed)
      if (!userId) {
        client.disconnect();
        return;
      }

      // Store connection info
      this.connectedUsers.set(client.id, {
        userId,
        socketId: client.id,
        connectedAt: new Date(),
      });

      // Join personal room
      client.join(`user:${userId}`);

      // Notify others
      client.broadcast.emit('user:online', { userId });

      this.logger.log(`User ${userId} connected (${client.id})`);
    } catch (error) {
      this.logger.error('Connection error:', error);
      client.disconnect();
    }
  }

  // 8. LIFECYCLE: Disconnection
  handleDisconnect(client: Socket) {
    const user = this.connectedUsers.get(client.id);
    
    if (user) {
      // Notify others
      client.broadcast.emit('user:offline', { userId: user.userId });
      
      // Cleanup
      this.connectedUsers.delete(client.id);
      
      this.logger.log(`User ${user.userId} disconnected`);
    }
  }

  // 9. EVENT HANDLERS
  @SubscribeMessage('send-message')
  @UsePipes(new ValidationPipe()) // Validate DTOs
  async handleSendMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: SendMessageDto,
  ) {
    const user = this.connectedUsers.get(client.id);
    
    // Business logic (save to DB)
    const message = await this.chatService.createMessage({
      userId: user.userId,
      content: data.message,
      roomId: data.roomId,
    });

    // Broadcast to room
    this.server.to(data.roomId).emit('new-message', {
      id: message.id,
      userId: user.userId,
      content: data.message,
      timestamp: message.createdAt,
    });

    return { success: true, messageId: message.id };
  }

  @SubscribeMessage('join-room')
  async handleJoinRoom(
    @ConnectedSocket() client: Socket,
    @MessageBody() roomId: string,
  ) {
    client.join(roomId);
    
    const user = this.connectedUsers.get(client.id);
    client.to(roomId).emit('user-joined', {
      userId: user.userId,
      roomId,
    });

    return { success: true, roomId };
  }

  @SubscribeMessage('leave-room')
  async handleLeaveRoom(
    @ConnectedSocket() client: Socket,
    @MessageBody() roomId: string,
  ) {
    client.leave(roomId);
    
    const user = this.connectedUsers.get(client.id);
    client.to(roomId).emit('user-left', {
      userId: user.userId,
      roomId,
    });

    return { success: true, roomId };
  }

  // 10. UTILITY METHODS (not event handlers)
  getConnectedUsers(): UserInfo[] {
    return Array.from(this.connectedUsers.values());
  }

  getUserBySocketId(socketId: string): UserInfo | undefined {
    return this.connectedUsers.get(socketId);
  }
}

// DTOs for validation
export class SendMessageDto {
  message: string;
  roomId: string;
}

interface UserInfo {
  userId: string;
  socketId: string;
  connectedAt: Date;
}
```

### **Gateway with Service Dependency Injection**

```typescript
// Example showing dependency injection in gateways

// 1. Service
// chat.service.ts
@Injectable()
export class ChatService {
  constructor(
    @InjectRepository(Message)
    private messageRepo: Repository<Message>,
  ) {}

  async createMessage(data: CreateMessageDto): Promise<Message> {
    const message = this.messageRepo.create(data);
    return this.messageRepo.save(message);
  }

  async getMessages(roomId: string): Promise<Message[]> {
    return this.messageRepo.find({ where: { roomId } });
  }
}

// 2. Gateway with injected service
// chat.gateway.ts
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  // Inject services like in controllers
  constructor(
    private chatService: ChatService,
    private usersService: UsersService,
    private notificationService: NotificationService,
  ) {}

  @SubscribeMessage('send-message')
  async handleMessage(
    @ConnectedSocket() client: Socket,
    @MessageBody() data: any,
  ) {
    // Use injected services
    const message = await this.chatService.createMessage(data);
    const user = await this.usersService.findOne(data.userId);
    
    // Send notification
    await this.notificationService.notify(user.id, 'New message');
    
    // Broadcast
    this.server.emit('new-message', message);
  }
}

// 3. Module registration
// chat.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([Message])],
  providers: [
    ChatGateway,    // Gateway
    ChatService,    // Service
  ],
})
export class ChatModule {}
```

### **Multiple Gateways in Same Module**

```typescript
// You can have multiple gateways for different purposes

// 1. Chat Gateway
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  @SubscribeMessage('send-message')
  handleChatMessage() { /* ... */ }
}

// 2. Notifications Gateway
@WebSocketGateway({ namespace: '/notifications' })
export class NotificationsGateway {
  @SubscribeMessage('subscribe')
  handleSubscribe() { /* ... */ }
}

// 3. Presence Gateway
@WebSocketGateway({ namespace: '/presence' })
export class PresenceGateway {
  handleConnection(client: Socket) { /* ... */ }
}

// Module with all gateways
@Module({
  providers: [
    ChatGateway,
    NotificationsGateway,
    PresenceGateway,
  ],
})
export class RealtimeModule {}
```

### **Gateway Generator (NestJS CLI)**

```bash
# Use NestJS CLI to generate gateway boilerplate
nest generate gateway chat
# or shorthand:
nest g gateway chat

# Creates:
# - chat.gateway.ts (gateway file)
# - chat.gateway.spec.ts (test file)
# - Updates module automatically

# With module:
nest g gateway chat --no-spec

# Generated file structure:
# src/
#   chat/
#     chat.gateway.ts
#     chat.gateway.spec.ts
```

### **Gateway File Structure (Best Practices)**

```typescript
// Organized gateway with clear sections

@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  // ============================================
  // 1. PROPERTIES
  // ============================================
  @WebSocketServer()
  server: Server;

  private logger = new Logger('ChatGateway');
  private connectedUsers: Map<string, UserInfo> = new Map();

  // ============================================
  // 2. CONSTRUCTOR (Dependency Injection)
  // ============================================
  constructor(
    private chatService: ChatService,
    private usersService: UsersService,
  ) {}

  // ============================================
  // 3. LIFECYCLE HOOKS
  // ============================================
  afterInit(server: Server) {
    this.logger.log('Gateway initialized');
  }

  handleConnection(client: Socket) {
    this.logger.log(`Client connected: ${client.id}`);
  }

  handleDisconnect(client: Socket) {
    this.logger.log(`Client disconnected: ${client.id}`);
  }

  // ============================================
  // 4. EVENT HANDLERS (@SubscribeMessage)
  // ============================================
  @SubscribeMessage('send-message')
  handleSendMessage(@ConnectedSocket() client: Socket, @MessageBody() data: any) {
    // Handle message
  }

  @SubscribeMessage('join-room')
  handleJoinRoom(@ConnectedSocket() client: Socket, @MessageBody() roomId: string) {
    // Handle join
  }

  // ============================================
  // 5. UTILITY METHODS (Private/Public)
  // ============================================
  private validateUser(userId: string): boolean {
    // Validation logic
    return true;
  }

  public getConnectedUsers(): UserInfo[] {
    return Array.from(this.connectedUsers.values());
  }
}
```

### **Testing Gateway Creation**

```typescript
// Test that gateway is created correctly
import { Test, TestingModule } from '@nestjs/testing';
import { ChatGateway } from './chat.gateway';
import { ChatService } from './chat.service';

describe('ChatGateway', () => {
  let gateway: ChatGateway;
  let service: ChatService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        ChatGateway,
        {
          provide: ChatService,
          useValue: {
            createMessage: jest.fn(),
            getMessages: jest.fn(),
          },
        },
      ],
    }).compile();

    gateway = module.get<ChatGateway>(ChatGateway);
    service = module.get<ChatService>(ChatService);
  });

  it('should be defined', () => {
    expect(gateway).toBeDefined();
  });

  it('should have server instance', () => {
    expect(gateway.server).toBeDefined();
  });

  it('should inject service', () => {
    expect(service).toBeDefined();
  });
});
```

### **Common Gateway Creation Mistakes**

```typescript
// ❌ MISTAKE 1: Missing @WebSocketGateway() decorator
export class ChatGateway { // Won't work - not a gateway!
  @WebSocketServer()
  server: Server;
}

// ✅ CORRECT:
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;
}

// ❌ MISTAKE 2: Missing @WebSocketServer() decorator
@WebSocketGateway()
export class ChatGateway {
  server: Server; // server will be undefined!
}

// ✅ CORRECT:
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server; // server will be injected
}

// ❌ MISTAKE 3: Implementing interfaces but not methods
@WebSocketGateway()
export class ChatGateway implements OnGatewayConnection {
  // Missing handleConnection() method - TypeScript error!
}

// ✅ CORRECT:
@WebSocketGateway()
export class ChatGateway implements OnGatewayConnection {
  handleConnection(client: Socket) {
    // Implementation
  }
}

// ❌ MISTAKE 4: Registering as controller instead of provider
@Module({
  controllers: [ChatGateway], // Wrong! Gateways are providers
})
export class ChatModule {}

// ✅ CORRECT:
@Module({
  providers: [ChatGateway], // Correct
})
export class ChatModule {}

// ❌ MISTAKE 5: Using @Param() instead of @MessageBody()
@WebSocketGateway()
export class ChatGateway {
  @SubscribeMessage('message')
  handleMessage(@Param('data') data: string) { // Wrong decorator!
    // data will be undefined
  }
}

// ✅ CORRECT:
@WebSocketGateway()
export class ChatGateway {
  @SubscribeMessage('message')
  handleMessage(@MessageBody() data: string) {
    // Correct
  }
}
```

**Interview Tip**: Create WebSocket gateway in **5 steps**: (1) create class file, (2) add `@WebSocketGateway()` decorator with config (`namespace`, `cors`), (3) inject `@WebSocketServer()` for Server instance, (4) implement lifecycle interfaces (`OnGatewayInit`, `OnGatewayConnection`, `OnGatewayDisconnect`) with methods (`afterInit`, `handleConnection`, `handleDisconnect`), (5) register as **provider** in module (not controller). **Event handlers**: use `@SubscribeMessage('event-name')` with `@ConnectedSocket()` for client, `@MessageBody()` for data. **Dependency injection**: inject services via constructor like controllers. **CLI command**: `nest g gateway chat` generates boilerplate. **Common mistakes**: forgetting `@WebSocketServer()` (server undefined), registering as controller instead of provider, using wrong decorators (`@Param` instead of `@MessageBody`). **Best practice**: organize code in sections (properties, constructor, lifecycle, events, utilities), use Logger for debugging, track connected users in Map, validate DTOs with pipes.

</details>
11. What interfaces should a Gateway implement (`OnGatewayInit`, `OnGatewayConnection`, `OnGatewayDisconnect`)?

<details>
<summary><strong>Answer</strong></summary>

Gateways should implement **three lifecycle interfaces** from `@nestjs/websockets`: (1) **`OnGatewayInit`** (after gateway initialized, implement `afterInit()`), (2) **`OnGatewayConnection`** (when client connects, implement `handleConnection()`), (3) **`OnGatewayDisconnect`** (when client disconnects, implement `handleDisconnect()`). These provide hooks into the WebSocket lifecycle.

### **The Three Gateway Interfaces**

```typescript
import {
  OnGatewayInit,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

// All three interfaces
@WebSocketGateway()
export class ChatGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  // 1. OnGatewayInit - Implement afterInit()
  afterInit(server: Server) {
    console.log('Gateway initialized');
  }

  // 2. OnGatewayConnection - Implement handleConnection()
  handleConnection(client: Socket, ...args: any[]) {
    console.log(`Client connected: ${client.id}`);
  }

  // 3. OnGatewayDisconnect - Implement handleDisconnect()
  handleDisconnect(client: Socket) {
    console.log(`Client disconnected: ${client.id}`);
  }
}
```

### **1. OnGatewayInit Interface**

```typescript
/**
 * OnGatewayInit - Called ONCE when gateway is fully initialized
 * Use for: Setup, configuration, logging, initialization tasks
 */

import { OnGatewayInit, WebSocketGateway, WebSocketServer } from '@nestjs/websockets';
import { Server } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway()
export class InitExampleGateway implements OnGatewayInit {
  @WebSocketServer()
  server: Server;

  private logger = new Logger('InitExampleGateway');

  // Required method: afterInit(server: Server)
  afterInit(server: Server) {
    // 1. Log initialization
    this.logger.log('Gateway initialized and ready');

    // 2. Store server reference (optional, already have @WebSocketServer)
    // this.server = server;

    // 3. Setup middleware
    server.use((socket, next) => {
      this.logger.log('Middleware: Authenticating socket');
      // Validate token, etc.
      next();
    });

    // 4. Configure server options
    server.engine.on('connection_error', (err) => {
      this.logger.error('Connection error:', err);
    });

    // 5. Initialize periodic tasks
    setInterval(() => {
      this.logger.log(`Connected clients: ${server.sockets.sockets.size}`);
    }, 60000); // Every minute

    // 6. Setup event listeners
    server.on('connection', (socket) => {
      this.logger.log(`New connection: ${socket.id}`);
    });
  }
}

// Lifecycle order:
// 1. Constructor
// 2. afterInit() ← Called once here
// 3. Server starts listening
// 4. handleConnection() called for each client
```

### **2. OnGatewayConnection Interface**

```typescript
/**
 * OnGatewayConnection - Called EVERY TIME a client connects
 * Use for: Authentication, user tracking, joining rooms, logging
 */

import { OnGatewayConnection, WebSocketGateway } from '@nestjs/websockets';
import { Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway()
export class ConnectionExampleGateway implements OnGatewayConnection {
  private logger = new Logger('ConnectionExampleGateway');
  private connectedUsers: Map<string, any> = new Map();

  // Required method: handleConnection(client: Socket, ...args: any[])
  handleConnection(client: Socket, ...args: any[]) {
    // 1. Log connection
    this.logger.log(`Client connected: ${client.id}`);

    // 2. Extract handshake data
    const userId = client.handshake.auth.userId;
    const token = client.handshake.auth.token;
    const query = client.handshake.query;

    // 3. Validate authentication
    if (!token) {
      this.logger.warn(`Unauthenticated connection: ${client.id}`);
      client.disconnect();
      return;
    }

    // 4. Store user info
    this.connectedUsers.set(client.id, {
      socketId: client.id,
      userId,
      connectedAt: new Date(),
    });

    // 5. Join personal room
    client.join(`user:${userId}`);

    // 6. Broadcast to others
    client.broadcast.emit('user:online', { userId });

    // 7. Send welcome message to client
    client.emit('welcome', {
      message: 'Connected successfully',
      socketId: client.id,
    });

    // 8. Additional args (from handshake)
    this.logger.log(`Handshake args:`, args);
  }
}
```

### **3. OnGatewayDisconnect Interface**

```typescript
/**
 * OnGatewayDisconnect - Called EVERY TIME a client disconnects
 * Use for: Cleanup, user tracking, leaving rooms, logging
 */

import { OnGatewayDisconnect, WebSocketGateway } from '@nestjs/websockets';
import { Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway()
export class DisconnectExampleGateway implements OnGatewayDisconnect {
  private logger = new Logger('DisconnectExampleGateway');
  private connectedUsers: Map<string, any> = new Map();

  // Required method: handleDisconnect(client: Socket)
  handleDisconnect(client: Socket) {
    // 1. Log disconnection
    this.logger.log(`Client disconnected: ${client.id}`);

    // 2. Get user info before cleanup
    const user = this.connectedUsers.get(client.id);

    // 3. Cleanup stored data
    this.connectedUsers.delete(client.id);

    // 4. Broadcast to others (if user exists)
    if (user) {
      client.broadcast.emit('user:offline', {
        userId: user.userId,
      });
    }

    // 5. Log disconnection reason
    const reason = client.disconnected ? 'disconnected' : 'unknown';
    this.logger.log(`Disconnect reason: ${reason}`);

    // 6. Additional cleanup
    // - Remove from database
    // - Update user status
    // - Clear timers
    // - Save session data
  }
}
```

### **All Three Together (Complete Example)**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  OnGatewayInit,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway({ cors: true })
export class CompleteLifecycleGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  @WebSocketServer()
  server: Server;

  private logger = new Logger('CompleteLifecycleGateway');
  private connectedUsers: Map<string, UserInfo> = new Map();

  // ================================================
  // 1. AFTER INIT (Called once at startup)
  // ================================================
  afterInit(server: Server) {
    this.logger.log('\u2705 Gateway initialized');
    
    // Setup middleware
    server.use((socket, next) => {
      const token = socket.handshake.auth.token;
      if (this.validateToken(token)) {
        next();
      } else {
        next(new Error('Invalid token'));
      }
    });

    // Log server info
    this.logger.log(`Server listening on namespace: ${server.name}`);
  }

  // ================================================
  // 2. CONNECTION (Called for each client)
  // ================================================
  handleConnection(client: Socket, ...args: any[]) {
    this.logger.log(`\u2795 Client connected: ${client.id}`);

    try {
      // Extract user data
      const userId = client.handshake.auth.userId;
      const username = client.handshake.auth.username;

      // Validate user
      if (!userId) {
        this.logger.warn('Missing userId, disconnecting');
        client.disconnect();
        return;
      }

      // Store user info
      this.connectedUsers.set(client.id, {
        socketId: client.id,
        userId,
        username,
        connectedAt: new Date(),
        lastActivity: new Date(),
      });

      // Join personal room
      client.join(`user:${userId}`);

      // Send welcome message
      client.emit('connected', {
        socketId: client.id,
        message: `Welcome, ${username}!`,
      });

      // Broadcast to others
      client.broadcast.emit('user:online', {
        userId,
        username,
      });

      // Log stats
      this.logger.log(
        `Total connected users: ${this.connectedUsers.size}`,
      );
    } catch (error) {
      this.logger.error('Connection error:', error);
      client.disconnect();
    }
  }

  // ================================================
  // 3. DISCONNECTION (Called for each client)
  // ================================================
  handleDisconnect(client: Socket) {
    this.logger.log(`\u2796 Client disconnected: ${client.id}`);

    // Get user info
    const user = this.connectedUsers.get(client.id);

    if (user) {
      // Calculate session duration
      const duration =
        new Date().getTime() - user.connectedAt.getTime();
      this.logger.log(
        `Session duration: ${Math.round(duration / 1000)}s`,
      );

      // Broadcast offline status
      client.broadcast.emit('user:offline', {
        userId: user.userId,
        username: user.username,
      });

      // Cleanup
      this.connectedUsers.delete(client.id);

      // Log stats
      this.logger.log(
        `Remaining connected users: ${this.connectedUsers.size}`,
      );
    }
  }

  // ================================================
  // UTILITY METHODS
  // ================================================
  private validateToken(token: string): boolean {
    // Token validation logic
    return !!token;
  }

  public getConnectedUsers(): UserInfo[] {
    return Array.from(this.connectedUsers.values());
  }
}

interface UserInfo {
  socketId: string;
  userId: string;
  username: string;
  connectedAt: Date;
  lastActivity: Date;
}
```

### **Interface Method Signatures**

```typescript
// Interface definitions from @nestjs/websockets

export interface OnGatewayInit<T = any> {
  /**
   * Called after the gateway is fully initialized
   * @param server - Socket.IO server instance
   */
  afterInit(server: T): any;
}

export interface OnGatewayConnection<T = any> {
  /**
   * Called when a client connects
   * @param client - Socket instance
   * @param args - Additional handshake arguments
   */
  handleConnection(client: T, ...args: any[]): any;
}

export interface OnGatewayDisconnect<T = any> {
  /**
   * Called when a client disconnects
   * @param client - Socket instance
   */
  handleDisconnect(client: T): any;
}
```

### **When to Implement Each Interface**

```typescript
// SCENARIO 1: Simple gateway (no lifecycle needs)
@WebSocketGateway()
export class SimpleGateway {
  // No interfaces needed - just event handlers
  
  @SubscribeMessage('message')
  handleMessage() {
    return 'Hello';
  }
}

// SCENARIO 2: Need authentication (use OnGatewayConnection)
@WebSocketGateway()
export class AuthenticatedGateway implements OnGatewayConnection {
  handleConnection(client: Socket) {
    // Validate token, reject if invalid
    if (!this.isAuthenticated(client)) {
      client.disconnect();
    }
  }
}

// SCENARIO 3: Need cleanup (use OnGatewayDisconnect)
@WebSocketGateway()
export class CleanupGateway implements OnGatewayDisconnect {
  private activeConnections = new Map();

  handleDisconnect(client: Socket) {
    // Cleanup resources
    this.activeConnections.delete(client.id);
  }
}

// SCENARIO 4: Need initialization (use OnGatewayInit)
@WebSocketGateway()
export class InitializedGateway implements OnGatewayInit {
  afterInit(server: Server) {
    // Setup middleware, logging, etc.
    console.log('Gateway ready');
  }
}

// SCENARIO 5: Production gateway (use all three)
@WebSocketGateway()
export class ProductionGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  afterInit(server: Server) {
    // Initial setup
  }

  handleConnection(client: Socket) {
    // Authenticate and track
  }

  handleDisconnect(client: Socket) {
    // Cleanup
  }
}
```

### **Lifecycle Execution Order**

```typescript
/**
 * Complete lifecycle flow:
 * 
 * 1. Application starts
 * 2. Gateway constructor called
 * 3. Dependencies injected
 * 4. afterInit() called (OnGatewayInit) ← ONCE
 * 5. Server starts listening
 * 6. Client connects
 * 7. handleConnection() called (OnGatewayConnection) ← PER CLIENT
 * 8. Client sends event
 * 9. @SubscribeMessage handler called
 * 10. Client disconnects
 * 11. handleDisconnect() called (OnGatewayDisconnect) ← PER CLIENT
 */

@WebSocketGateway()
export class LifecycleOrderGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  private logger = new Logger('LifecycleOrderGateway');

  // Step 1: Constructor
  constructor() {
    this.logger.log('1. Constructor called');
  }

  // Step 2: After init (ONCE)
  afterInit(server: Server) {
    this.logger.log('2. afterInit called (once)');
  }

  // Step 3: Connection (PER CLIENT)
  handleConnection(client: Socket) {
    this.logger.log(`3. handleConnection called for ${client.id}`);
  }

  // Step 4: Event handler (PER EVENT)
  @SubscribeMessage('test')
  handleTest() {
    this.logger.log('4. Event handler called');
  }

  // Step 5: Disconnection (PER CLIENT)
  handleDisconnect(client: Socket) {
    this.logger.log(`5. handleDisconnect called for ${client.id}`);
  }
}

// Console output:
// 1. Constructor called
// 2. afterInit called (once)
// 3. handleConnection called for abc123
// 4. Event handler called
// 5. handleDisconnect called for abc123
```

### **Error Handling in Lifecycle Methods**

```typescript
@WebSocketGateway()
export class ErrorHandlingGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  private logger = new Logger('ErrorHandlingGateway');

  afterInit(server: Server) {
    try {
      // Setup logic that might fail
      this.setupMiddleware(server);
    } catch (error) {
      this.logger.error('Error in afterInit:', error);
      // Gateway will still start, but middleware won't be set up
    }
  }

  handleConnection(client: Socket, ...args: any[]) {
    try {
      // Authentication logic that might fail
      this.authenticateClient(client);
    } catch (error) {
      this.logger.error(`Connection error for ${client.id}:`, error);
      // Disconnect client on error
      client.emit('error', { message: 'Authentication failed' });
      client.disconnect();
    }
  }

  handleDisconnect(client: Socket) {
    try {
      // Cleanup logic that might fail
      this.cleanupClient(client);
    } catch (error) {
      // Log but don't throw (client already disconnected)
      this.logger.error(`Disconnect error for ${client.id}:`, error);
    }
  }

  private setupMiddleware(server: Server) {
    // Middleware setup
  }

  private authenticateClient(client: Socket) {
    // Authentication logic
  }

  private cleanupClient(client: Socket) {
    // Cleanup logic
  }
}
```

### **Async Lifecycle Methods**

```typescript
// All lifecycle methods can be async
@WebSocketGateway()
export class AsyncLifecycleGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  constructor(private usersService: UsersService) {}

  // Async init
  async afterInit(server: Server) {
    await this.loadConfiguration();
    console.log('Configuration loaded');
  }

  // Async connection
  async handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    // Validate user from database
    const user = await this.usersService.findOne(userId);
    
    if (!user) {
      client.disconnect();
      return;
    }

    // Update user status
    await this.usersService.updateStatus(userId, 'online');
    
    client.emit('authenticated', { user });
  }

  // Async disconnection
  async handleDisconnect(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    // Update user status in database
    await this.usersService.updateStatus(userId, 'offline');
    
    // Save session data
    await this.saveSession(client.id);
  }

  private async loadConfiguration() {
    // Load config from database/file
  }

  private async saveSession(socketId: string) {
    // Save session to database
  }
}
```

**Interview Tip**: Gateways implement **three lifecycle interfaces** from `@nestjs/websockets`: (1) **`OnGatewayInit`** - implement `afterInit(server)`, called **once** at startup for setup/configuration/middleware; (2) **`OnGatewayConnection`** - implement `handleConnection(client, ...args)`, called **per client** connection for authentication/tracking/joining rooms; (3) **`OnGatewayDisconnect`** - implement `handleDisconnect(client)`, called **per client** disconnection for cleanup/status updates. **Lifecycle order**: constructor → afterInit → handleConnection → event handlers → handleDisconnect. **All optional** but recommended for production. **Can be async** for database operations. **Use cases**: OnGatewayInit (middleware setup, logging), OnGatewayConnection (auth validation, reject invalid clients with `client.disconnect()`), OnGatewayDisconnect (cleanup resources, broadcast offline status). **Error handling**: wrap in try-catch, disconnect client on connection errors, log but don't throw on disconnect errors.

</details>
12. What is the `handleConnection()` lifecycle hook?

<details>
<summary><strong>Answer</strong></summary>

**`handleConnection()`** is a lifecycle hook from the `OnGatewayConnection` interface that is called **every time a client connects** to the gateway. It receives the **Socket instance** and optional handshake arguments, allowing you to authenticate clients, track connections, join rooms, and send welcome messages.

### **Basic Signature**

```typescript
import { OnGatewayConnection, WebSocketGateway } from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway()
export class ChatGateway implements OnGatewayConnection {
  // Method signature:
  handleConnection(client: Socket, ...args: any[]): any {
    // Called when client connects
    // client: Socket instance for this connection
    // args: Additional handshake arguments
  }
}
```

### **Simple Example**

```typescript
@WebSocketGateway()
export class SimpleGateway implements OnGatewayConnection {
  handleConnection(client: Socket) {
    // 1. Log the connection
    console.log(`New client connected: ${client.id}`);
    
    // 2. Send welcome message
    client.emit('welcome', {
      message: 'Successfully connected!',
      socketId: client.id,
    });
  }
}

// When client connects:
// const socket = io('http://localhost:3000');
// Console: "New client connected: abc123"
// Client receives: { message: 'Successfully connected!', socketId: 'abc123' }
```

### **Common Use Cases**

#### **1. Authentication and Validation**

```typescript
import { Logger } from '@nestjs/common';

@WebSocketGateway()
export class AuthenticatedGateway implements OnGatewayConnection {
  private logger = new Logger('AuthenticatedGateway');

  handleConnection(client: Socket, ...args: any[]) {
    // Extract token from handshake
    const token = client.handshake.auth.token;
    const userId = client.handshake.auth.userId;

    // VALIDATION 1: Check if token exists
    if (!token) {
      this.logger.warn(`No token provided, disconnecting: ${client.id}`);
      client.emit('error', { message: 'Authentication required' });
      client.disconnect();
      return;
    }

    // VALIDATION 2: Verify token
    try {
      const isValid = this.validateToken(token);
      if (!isValid) {
        this.logger.warn(`Invalid token, disconnecting: ${client.id}`);
        client.disconnect();
        return;
      }
    } catch (error) {
      this.logger.error('Token validation error:', error);
      client.disconnect();
      return;
    }

    // VALIDATION 3: Check user exists
    if (!userId) {
      this.logger.warn(`No userId provided, disconnecting: ${client.id}`);
      client.disconnect();
      return;
    }

    // Authentication successful
    this.logger.log(`User ${userId} authenticated (${client.id})`);
    client.emit('authenticated', { userId, socketId: client.id });
  }

  private validateToken(token: string): boolean {
    // Token validation logic (JWT, etc.)
    return true;
  }
}
```

#### **2. User Tracking and State Management**

```typescript
@WebSocketGateway()
export class TrackingGateway implements OnGatewayConnection, OnGatewayDisconnect {
  // Store connected users
  private connectedUsers: Map<string, UserInfo> = new Map();
  private logger = new Logger('TrackingGateway');

  handleConnection(client: Socket) {
    // Extract user data
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;
    const userAgent = client.handshake.headers['user-agent'];
    const ip = client.handshake.address;

    // Store user info
    const userInfo: UserInfo = {
      socketId: client.id,
      userId,
      username,
      connectedAt: new Date(),
      lastActivity: new Date(),
      ip,
      userAgent,
    };

    this.connectedUsers.set(client.id, userInfo);

    // Log connection stats
    this.logger.log(
      `User ${username} connected. Total users: ${this.connectedUsers.size}`,
    );

    // Broadcast online status to others
    client.broadcast.emit('user:online', {
      userId,
      username,
      connectedAt: userInfo.connectedAt,
    });
  }

  handleDisconnect(client: Socket) {
    const user = this.connectedUsers.get(client.id);
    this.connectedUsers.delete(client.id);
    
    if (user) {
      client.broadcast.emit('user:offline', { userId: user.userId });
    }
  }

  // Utility method to get connected users
  getConnectedUsers(): UserInfo[] {
    return Array.from(this.connectedUsers.values());
  }
}

interface UserInfo {
  socketId: string;
  userId: string;
  username: string;
  connectedAt: Date;
  lastActivity: Date;
  ip: string;
  userAgent: string;
}
```

#### **3. Joining Rooms Automatically**

```typescript
@WebSocketGateway()
export class RoomGateway implements OnGatewayConnection {
  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    const teamId = client.handshake.query.teamId as string;

    // AUTO-JOIN 1: Personal room (for private messages)
    client.join(`user:${userId}`);
    console.log(`User ${userId} joined personal room`);

    // AUTO-JOIN 2: Team room (if teamId provided)
    if (teamId) {
      client.join(`team:${teamId}`);
      console.log(`User ${userId} joined team:${teamId}`);
      
      // Notify team members
      client.to(`team:${teamId}`).emit('team:member-joined', {
        userId,
        teamId,
      });
    }

    // AUTO-JOIN 3: Broadcast room (for announcements)
    client.join('broadcast');

    // Send confirmation
    client.emit('rooms:joined', {
      rooms: [`user:${userId}`, `team:${teamId}`, 'broadcast'],
    });
  }
}
```

#### **4. Sending Initial Data**

```typescript
@WebSocketGateway()
export class InitialDataGateway implements OnGatewayConnection {
  constructor(
    private chatService: ChatService,
    private usersService: UsersService,
  ) {}

  async handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;

    // Send initial data to client
    try {
      // 1. User profile
      const user = await this.usersService.findOne(userId);
      client.emit('user:profile', user);

      // 2. Unread messages count
      const unreadCount = await this.chatService.getUnreadCount(userId);
      client.emit('messages:unread-count', { count: unreadCount });

      // 3. Online friends
      const onlineFriends = await this.usersService.getOnlineFriends(userId);
      client.emit('friends:online', onlineFriends);

      // 4. Recent conversations
      const conversations = await this.chatService.getRecentConversations(userId);
      client.emit('conversations:recent', conversations);

      console.log(`Initial data sent to user ${userId}`);
    } catch (error) {
      console.error('Error sending initial data:', error);
      client.emit('error', { message: 'Failed to load initial data' });
    }
  }
}
```

### **Accessing Handshake Data**

```typescript
@WebSocketGateway()
export class HandshakeGateway implements OnGatewayConnection {
  handleConnection(client: Socket, ...args: any[]) {
    // 1. AUTH DATA (sent from client during connection)
    const token = client.handshake.auth.token;
    const userId = client.handshake.auth.userId;
    console.log('Auth:', { token, userId });

    // 2. QUERY PARAMETERS (?param=value)
    const roomId = client.handshake.query.roomId;
    const source = client.handshake.query.source;
    console.log('Query:', { roomId, source });

    // 3. HEADERS
    const userAgent = client.handshake.headers['user-agent'];
    const origin = client.handshake.headers.origin;
    console.log('Headers:', { userAgent, origin });

    // 4. CONNECTION INFO
    const ip = client.handshake.address;
    const secure = client.handshake.secure;
    console.log('Connection:', { ip, secure });

    // 5. TIME
    const connectedAt = new Date(client.handshake.time);
    console.log('Connected at:', connectedAt);

    // 6. TRANSPORT
    const transport = client.conn.transport.name; // 'websocket' or 'polling'
    console.log('Transport:', transport);

    // 7. ADDITIONAL ARGS (from ...args parameter)
    console.log('Additional args:', args);
  }
}

// Client-side code that sends handshake data:
const socket = io('http://localhost:3000', {
  auth: {
    token: 'jwt-token-here',
    userId: '123',
  },
  query: {
    roomId: 'room-456',
    source: 'mobile-app',
  },
});
```

### **Production Example with All Features**

```typescript
import {
  WebSocketGateway,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Socket } from 'socket.io';
import { Logger } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

@WebSocketGateway({
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(','),
    credentials: true,
  },
})
export class ProductionGateway
  implements OnGatewayConnection, OnGatewayDisconnect
{
  private logger = new Logger('ProductionGateway');
  private connectedUsers = new Map<string, UserSession>();

  constructor(
    private jwtService: JwtService,
    private usersService: UsersService,
    private chatService: ChatService,
  ) {}

  async handleConnection(client: Socket, ...args: any[]) {
    const startTime = Date.now();
    
    try {
      // ================================================
      // 1. EXTRACT HANDSHAKE DATA
      // ================================================
      const token = client.handshake.auth.token;
      const userId = client.handshake.auth.userId;
      const ip = client.handshake.address;
      const userAgent = client.handshake.headers['user-agent'];

      // ================================================
      // 2. VALIDATE TOKEN
      // ================================================
      if (!token) {
        this.logger.warn(`No token provided: ${client.id} from ${ip}`);
        client.emit('error', {
          code: 'AUTH_REQUIRED',
          message: 'Authentication token required',
        });
        client.disconnect();
        return;
      }

      let decodedToken;
      try {
        decodedToken = await this.jwtService.verifyAsync(token);
      } catch (error) {
        this.logger.warn(`Invalid token: ${client.id}`);
        client.emit('error', {
          code: 'AUTH_INVALID',
          message: 'Invalid authentication token',
        });
        client.disconnect();
        return;
      }

      // ================================================
      // 3. VALIDATE USER
      // ================================================
      const user = await this.usersService.findOne(decodedToken.sub);
      if (!user) {
        this.logger.warn(`User not found: ${decodedToken.sub}`);
        client.disconnect();
        return;
      }

      if (user.status === 'banned') {
        this.logger.warn(`Banned user attempted connection: ${user.id}`);
        client.emit('error', {
          code: 'USER_BANNED',
          message: 'Your account has been banned',
        });
        client.disconnect();
        return;
      }

      // ================================================
      // 4. STORE SESSION
      // ================================================
      const session: UserSession = {
        socketId: client.id,
        userId: user.id,
        username: user.username,
        connectedAt: new Date(),
        lastActivity: new Date(),
        ip,
        userAgent,
      };
      this.connectedUsers.set(client.id, session);

      // ================================================
      // 5. JOIN ROOMS
      // ================================================
      // Personal room for direct messages
      client.join(`user:${user.id}`);
      
      // User's teams
      const teams = await this.usersService.getUserTeams(user.id);
      teams.forEach(team => {
        client.join(`team:${team.id}`);
      });

      // Broadcast room
      client.join('broadcast');

      // ================================================
      // 6. UPDATE USER STATUS
      // ================================================
      await this.usersService.updateStatus(user.id, 'online');

      // ================================================
      // 7. SEND INITIAL DATA
      // ================================================
      // Welcome message
      client.emit('connected', {
        socketId: client.id,
        userId: user.id,
        message: `Welcome back, ${user.username}!`,
      });

      // Unread messages
      const unreadCount = await this.chatService.getUnreadCount(user.id);
      client.emit('messages:unread-count', { count: unreadCount });

      // Online friends
      const onlineFriends = this.getOnlineFriends(user.id);
      client.emit('friends:online', onlineFriends);

      // ================================================
      // 8. BROADCAST TO OTHERS
      // ================================================
      client.broadcast.emit('user:online', {
        userId: user.id,
        username: user.username,
      });

      // ================================================
      // 9. LOGGING & METRICS
      // ================================================
      const duration = Date.now() - startTime;
      this.logger.log(
        `User ${user.username} (${user.id}) connected from ${ip} in ${duration}ms. ` +
        `Total connections: ${this.connectedUsers.size}`,
      );

    } catch (error) {
      this.logger.error(
        `Connection error for ${client.id}:`,
        error.stack,
      );
      client.emit('error', {
        code: 'INTERNAL_ERROR',
        message: 'An error occurred during connection',
      });
      client.disconnect();
    }
  }

  handleDisconnect(client: Socket) {
    const session = this.connectedUsers.get(client.id);
    
    if (session) {
      const duration = Date.now() - session.connectedAt.getTime();
      this.logger.log(
        `User ${session.username} disconnected after ${Math.round(duration / 1000)}s`,
      );

      // Update status
      this.usersService.updateStatus(session.userId, 'offline');

      // Broadcast offline
      client.broadcast.emit('user:offline', {
        userId: session.userId,
      });

      // Cleanup
      this.connectedUsers.delete(client.id);
    }
  }

  private getOnlineFriends(userId: string): any[] {
    // Get user's friends who are currently online
    return [];
  }
}

interface UserSession {
  socketId: string;
  userId: string;
  username: string;
  connectedAt: Date;
  lastActivity: Date;
  ip: string;
  userAgent: string;
}
```

### **Error Handling Best Practices**

```typescript
@WebSocketGateway()
export class ErrorHandlingGateway implements OnGatewayConnection {
  private logger = new Logger('ErrorHandlingGateway');

  async handleConnection(client: Socket) {
    try {
      // Your connection logic
      await this.authenticateClient(client);
      
    } catch (error) {
      // Log the error
      this.logger.error(
        `Connection error for ${client.id}:`,
        error.stack,
      );

      // Send error to client (user-friendly message)
      client.emit('error', {
        message: 'Connection failed',
        code: 'CONNECTION_ERROR',
      });

      // Disconnect client
      client.disconnect();
      
      // DON'T throw - it will crash the gateway!
      // Instead, handle gracefully and disconnect
    }
  }

  private async authenticateClient(client: Socket) {
    // Authentication logic that might throw
  }
}
```

**Interview Tip**: **`handleConnection()`** is called **every time a client connects**, receives `Socket` instance and optional handshake args. **Common use cases**: (1) **Authentication** - validate `client.handshake.auth.token`, call `client.disconnect()` if invalid; (2) **User tracking** - store connected users in Map by `client.id`; (3) **Auto-join rooms** - `client.join('user:' + userId)` for personal room; (4) **Send initial data** - emit welcome message, unread counts, online friends. **Access handshake data**: `client.handshake.auth` (token/userId), `client.handshake.query` (URL params), `client.handshake.headers` (user-agent/origin), `client.handshake.address` (IP). **Best practices**: wrap in try-catch, emit error before disconnect, use async for DB operations, log connection stats, broadcast online status to others. **Validation pattern**: check token → verify user → store session → join rooms → send initial data → broadcast online. **Don't throw errors** - handle gracefully and disconnect client.

</details>
13. What is the `handleDisconnect()` lifecycle hook?

<details>
<summary><strong>Answer</strong></summary>

**`handleDisconnect()`** is a lifecycle hook from the `OnGatewayDisconnect` interface that is called **every time a client disconnects** from the gateway. It receives the **Socket instance**, allowing you to clean up resources, update user status, broadcast offline notifications, and save session data.

### **Basic Signature**

```typescript
import { OnGatewayDisconnect, WebSocketGateway } from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway()
export class ChatGateway implements OnGatewayDisconnect {
  // Method signature:
  handleDisconnect(client: Socket): any {
    // Called when client disconnects
    // client: Socket instance that disconnected
  }
}
```

### **Simple Example**

```typescript
@WebSocketGateway()
export class SimpleGateway implements OnGatewayDisconnect {
  handleDisconnect(client: Socket) {
    // Log the disconnection
    console.log(`Client disconnected: ${client.id}`);
  }
}

// When client disconnects (client.disconnect() or connection lost):
// Console: "Client disconnected: abc123"
```

### **Common Use Cases**

#### **1. Cleanup Resources and User Tracking**

```typescript
import { Logger } from '@nestjs/common';

@WebSocketGateway()
export class TrackingGateway
  implements OnGatewayConnection, OnGatewayDisconnect
{
  private logger = new Logger('TrackingGateway');
  private connectedUsers = new Map<string, UserInfo>();

  handleConnection(client: Socket) {
    // Store user when connecting
    const userId = client.handshake.auth.userId;
    this.connectedUsers.set(client.id, {
      socketId: client.id,
      userId,
      connectedAt: new Date(),
    });
    
    this.logger.log(`User connected. Total: ${this.connectedUsers.size}`);
  }

  handleDisconnect(client: Socket) {
    // CLEANUP 1: Get user info before removing
    const user = this.connectedUsers.get(client.id);
    
    if (user) {
      // Calculate session duration
      const duration = Date.now() - user.connectedAt.getTime();
      const durationSeconds = Math.round(duration / 1000);
      
      this.logger.log(
        `User ${user.userId} disconnected after ${durationSeconds}s`,
      );
    }

    // CLEANUP 2: Remove from connected users map
    this.connectedUsers.delete(client.id);

    // CLEANUP 3: Log remaining connections
    this.logger.log(
      `Remaining connected users: ${this.connectedUsers.size}`,
    );
  }
}

interface UserInfo {
  socketId: string;
  userId: string;
  connectedAt: Date;
}
```

#### **2. Broadcasting Offline Status**

```typescript
@WebSocketGateway()
export class PresenceGateway
  implements OnGatewayConnection, OnGatewayDisconnect
{
  private connectedUsers = new Map<string, any>();

  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;
    
    this.connectedUsers.set(client.id, { userId, username });
    
    // Broadcast ONLINE status to others
    client.broadcast.emit('user:online', {
      userId,
      username,
      timestamp: new Date(),
    });
  }

  handleDisconnect(client: Socket) {
    const user = this.connectedUsers.get(client.id);
    
    if (user) {
      // Broadcast OFFLINE status to others
      client.broadcast.emit('user:offline', {
        userId: user.userId,
        username: user.username,
        timestamp: new Date(),
      });
      
      console.log(`${user.username} went offline`);
    }

    // Cleanup
    this.connectedUsers.delete(client.id);
  }
}
```

#### **3. Updating Database Status**

```typescript
@WebSocketGateway()
export class DatabaseStatusGateway
  implements OnGatewayConnection, OnGatewayDisconnect
{
  constructor(
    private usersService: UsersService,
    private sessionsService: SessionsService,
  ) {}

  async handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    // Update status to ONLINE in database
    await this.usersService.updateStatus(userId, 'online');
    
    // Create session record
    await this.sessionsService.create({
      userId,
      socketId: client.id,
      connectedAt: new Date(),
    });
  }

  async handleDisconnect(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    try {
      // UPDATE 1: User status to OFFLINE
      await this.usersService.updateStatus(userId, 'offline');
      
      // UPDATE 2: Set last seen timestamp
      await this.usersService.updateLastSeen(userId, new Date());
      
      // UPDATE 3: End session in database
      await this.sessionsService.endSession(client.id, new Date());
      
      console.log(`Updated database for user ${userId}`);
    } catch (error) {
      console.error('Error updating database on disconnect:', error);
      // DON'T throw - client already disconnected
    }
  }
}
```

#### **4. Leaving Rooms and Cleanup**

```typescript
@WebSocketGateway()
export class RoomCleanupGateway
  implements OnGatewayConnection, OnGatewayDisconnect
{
  private userRooms = new Map<string, string[]>();

  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    // Join rooms
    const rooms = [`user:${userId}`, 'general', 'announcements'];
    rooms.forEach(room => client.join(room));
    
    // Track rooms
    this.userRooms.set(client.id, rooms);
  }

  handleDisconnect(client: Socket) {
    const rooms = this.userRooms.get(client.id);
    
    if (rooms) {
      // CLEANUP 1: Notify each room that user left
      rooms.forEach(room => {
        client.to(room).emit('user:left-room', {
          socketId: client.id,
          room,
          timestamp: new Date(),
        });
      });

      // CLEANUP 2: Leave all rooms (Socket.IO does this automatically,
      // but good to be explicit for custom logic)
      rooms.forEach(room => {
        client.leave(room);
      });

      // CLEANUP 3: Remove from tracking
      this.userRooms.delete(client.id);
      
      console.log(`Cleaned up ${rooms.length} rooms for ${client.id}`);
    }
  }
}
```

#### **5. Saving Session Data**

```typescript
@WebSocketGateway()
export class SessionGateway
  implements OnGatewayConnection, OnGatewayDisconnect
{
  private sessions = new Map<string, SessionData>();

  constructor(private sessionService: SessionService) {}

  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    // Initialize session
    this.sessions.set(client.id, {
      userId,
      startTime: new Date(),
      activityLog: [],
      messagesCount: 0,
    });
  }

  // Track activity during session
  @SubscribeMessage('send-message')
  handleMessage(@ConnectedSocket() client: Socket) {
    const session = this.sessions.get(client.id);
    if (session) {
      session.messagesCount++;
      session.activityLog.push({
        action: 'message',
        timestamp: new Date(),
      });
    }
  }

  async handleDisconnect(client: Socket) {
    const session = this.sessions.get(client.id);
    
    if (session) {
      // Calculate session metrics
      const endTime = new Date();
      const duration = endTime.getTime() - session.startTime.getTime();
      
      // SAVE SESSION DATA to database
      try {
        await this.sessionService.saveSession({
          userId: session.userId,
          socketId: client.id,
          startTime: session.startTime,
          endTime,
          duration,
          messagesCount: session.messagesCount,
          activityLog: session.activityLog,
        });
        
        console.log(
          `Saved session for user ${session.userId}: ` +
          `${Math.round(duration / 1000)}s, ${session.messagesCount} messages`,
        );
      } catch (error) {
        console.error('Error saving session:', error);
      }

      // CLEANUP: Remove from memory
      this.sessions.delete(client.id);
    }
  }
}

interface SessionData {
  userId: string;
  startTime: Date;
  activityLog: { action: string; timestamp: Date }[];
  messagesCount: number;
}
```

### **Production Example with All Features**

```typescript
import {
  WebSocketGateway,
  OnGatewayConnection,
  OnGatewayDisconnect,
  WebSocketServer,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway({
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(','),
    credentials: true,
  },
})
export class ProductionGateway
  implements OnGatewayConnection, OnGatewayDisconnect
{
  @WebSocketServer()
  server: Server;

  private logger = new Logger('ProductionGateway');
  private connectedUsers = new Map<string, UserSession>();

  constructor(
    private usersService: UsersService,
    private sessionsService: SessionsService,
    private analyticsService: AnalyticsService,
  ) {}

  async handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;
    
    // Store session
    this.connectedUsers.set(client.id, {
      userId,
      username,
      socketId: client.id,
      connectedAt: new Date(),
      messageCount: 0,
      lastActivity: new Date(),
    });

    // Update status
    await this.usersService.updateStatus(userId, 'online');
    
    // Broadcast online
    client.broadcast.emit('user:online', { userId, username });
    
    this.logger.log(
      `User ${username} connected. Total: ${this.connectedUsers.size}`,
    );
  }

  async handleDisconnect(client: Socket) {
    const startTime = Date.now();
    const session = this.connectedUsers.get(client.id);

    if (!session) {
      this.logger.warn(`No session found for ${client.id}`);
      return;
    }

    try {
      // ================================================
      // 1. CALCULATE SESSION METRICS
      // ================================================
      const endTime = new Date();
      const duration = endTime.getTime() - session.connectedAt.getTime();
      const durationSeconds = Math.round(duration / 1000);

      // ================================================
      // 2. UPDATE DATABASE STATUS
      // ================================================
      await Promise.all([
        // Update user status to offline
        this.usersService.updateStatus(session.userId, 'offline'),
        
        // Update last seen timestamp
        this.usersService.updateLastSeen(session.userId, endTime),
        
        // Save session to database
        this.sessionsService.saveSession({
          userId: session.userId,
          socketId: client.id,
          connectedAt: session.connectedAt,
          disconnectedAt: endTime,
          duration,
          messageCount: session.messageCount,
        }),
      ]);

      // ================================================
      // 3. BROADCAST OFFLINE STATUS
      // ================================================
      client.broadcast.emit('user:offline', {
        userId: session.userId,
        username: session.username,
        lastSeen: endTime,
      });

      // ================================================
      // 4. TRACK ANALYTICS
      // ================================================
      await this.analyticsService.trackEvent({
        event: 'user_disconnected',
        userId: session.userId,
        metadata: {
          duration,
          messageCount: session.messageCount,
        },
      });

      // ================================================
      // 5. CLEANUP MEMORY
      // ================================================
      this.connectedUsers.delete(client.id);

      // ================================================
      // 6. LOGGING
      // ================================================
      const processingTime = Date.now() - startTime;
      this.logger.log(
        `User ${session.username} (${session.userId}) disconnected. ` +
        `Session: ${durationSeconds}s, Messages: ${session.messageCount}. ` +
        `Cleanup: ${processingTime}ms. ` +
        `Remaining: ${this.connectedUsers.size}`,
      );

    } catch (error) {
      // ================================================
      // 7. ERROR HANDLING
      // ================================================
      this.logger.error(
        `Error during disconnect cleanup for ${client.id}:`,
        error.stack,
      );
      
      // Still cleanup memory even if DB update fails
      this.connectedUsers.delete(client.id);
      
      // DON'T throw - client already disconnected
    }
  }

  // Update activity timestamp and message count
  @SubscribeMessage('send-message')
  handleMessage(@ConnectedSocket() client: Socket) {
    const session = this.connectedUsers.get(client.id);
    if (session) {
      session.lastActivity = new Date();
      session.messageCount++;
    }
  }
}

interface UserSession {
  userId: string;
  username: string;
  socketId: string;
  connectedAt: Date;
  lastActivity: Date;
  messageCount: number;
}
```

### **Disconnect Reasons**

```typescript
@WebSocketGateway()
export class DisconnectReasonsGateway implements OnGatewayDisconnect {
  private logger = new Logger('DisconnectReasonsGateway');

  handleDisconnect(client: Socket) {
    // Socket.IO disconnect reasons (client.disconnected is always true here)
    
    // Common disconnect reasons:
    // - 'transport close': Connection lost (network issue)
    // - 'transport error': Transport error
    // - 'server namespace disconnect': Server called client.disconnect()
    // - 'client namespace disconnect': Client called socket.disconnect()
    // - 'ping timeout': Client didn't respond to ping
    
    this.logger.log(
      `Client ${client.id} disconnected. ` +
      `Connected: ${client.connected}, ` +
      `Disconnected: ${client.disconnected}`,
    );

    // Check if client was forcefully disconnected
    // (vs. normal disconnection)
    const wasForced = !client.connected && client.disconnected;
    
    if (wasForced) {
      this.logger.warn(`Forced disconnect: ${client.id}`);
    } else {
      this.logger.log(`Normal disconnect: ${client.id}`);
    }
  }
}
```

### **Handling Multiple Disconnects (Edge Case)**

```typescript
@WebSocketGateway()
export class SafeDisconnectGateway implements OnGatewayDisconnect {
  private processingDisconnect = new Set<string>();
  private logger = new Logger('SafeDisconnectGateway');

  async handleDisconnect(client: Socket) {
    // EDGE CASE: handleDisconnect might be called multiple times
    // (rare but possible in certain scenarios)
    
    // Guard against duplicate processing
    if (this.processingDisconnect.has(client.id)) {
      this.logger.warn(
        `Duplicate disconnect call for ${client.id}, ignoring`,
      );
      return;
    }

    // Mark as processing
    this.processingDisconnect.add(client.id);

    try {
      // Your disconnect logic here
      await this.cleanup(client);
      
    } catch (error) {
      this.logger.error('Disconnect error:', error);
      
    } finally {
      // Always remove from processing set
      this.processingDisconnect.delete(client.id);
    }
  }

  private async cleanup(client: Socket) {
    // Cleanup logic
  }
}
```

### **Automatic Reconnection Handling**

```typescript
@WebSocketGateway({
  // Enable connection state recovery (Socket.IO v4.6+)
  connectionStateRecovery: {
    maxDisconnectionDuration: 2 * 60 * 1000, // 2 minutes
    skipMiddlewares: true,
  },
})
export class ReconnectionGateway
  implements OnGatewayConnection, OnGatewayDisconnect
{
  private logger = new Logger('ReconnectionGateway');
  private temporaryDisconnects = new Map<string, NodeJS.Timeout>();

  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    // Check if this is a reconnection
    const disconnectTimer = this.temporaryDisconnects.get(userId);
    
    if (disconnectTimer) {
      // User reconnected within timeout window
      clearTimeout(disconnectTimer);
      this.temporaryDisconnects.delete(userId);
      
      this.logger.log(`User ${userId} reconnected`);
      client.emit('reconnected', { message: 'Welcome back!' });
    } else {
      // New connection
      this.logger.log(`User ${userId} connected`);
      client.emit('connected', { message: 'Connected!' });
    }
  }

  handleDisconnect(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    // Don't immediately mark as offline - give them time to reconnect
    const timeout = setTimeout(async () => {
      // After 30 seconds, mark as truly offline
      this.logger.log(`User ${userId} offline (timeout)`);
      
      // Update database
      await this.updateOfflineStatus(userId);
      
      // Broadcast offline
      this.server.emit('user:offline', { userId });
      
      // Cleanup
      this.temporaryDisconnects.delete(userId);
    }, 30000); // 30 seconds grace period

    this.temporaryDisconnects.set(userId, timeout);
    this.logger.log(`User ${userId} disconnected (grace period)`);
  }

  private async updateOfflineStatus(userId: string) {
    // Update status in database
  }
}
```

### **Error Handling Best Practices**

```typescript
@WebSocketGateway()
export class ErrorSafeGateway implements OnGatewayDisconnect {
  private logger = new Logger('ErrorSafeGateway');

  async handleDisconnect(client: Socket) {
    // ALWAYS wrap in try-catch
    try {
      // Your disconnect logic
      await this.performCleanup(client);
      
    } catch (error) {
      // Log error but DON'T throw
      this.logger.error(
        `Disconnect error for ${client.id}:`,
        error.stack,
      );
      
      // Client is already gone, throwing won't help
      // Instead, handle gracefully and continue
    }
  }

  private async performCleanup(client: Socket) {
    // Individual operations with their own error handling
    
    // Operation 1: Database update
    try {
      await this.updateDatabase(client);
    } catch (error) {
      this.logger.error('Database update failed:', error);
      // Continue with other cleanup
    }

    // Operation 2: Broadcast
    try {
      this.broadcastOffline(client);
    } catch (error) {
      this.logger.error('Broadcast failed:', error);
      // Continue with other cleanup
    }

    // Operation 3: Memory cleanup (always do this)
    try {
      this.cleanupMemory(client);
    } catch (error) {
      this.logger.error('Memory cleanup failed:', error);
      // This should never fail, but log if it does
    }
  }

  private async updateDatabase(client: Socket) {
    // Database operations
  }

  private broadcastOffline(client: Socket) {
    // Broadcasting
  }

  private cleanupMemory(client: Socket) {
    // Memory cleanup
  }
}
```

**Interview Tip**: **`handleDisconnect()`** is called **every time a client disconnects**, receives `Socket` instance. **Common use cases**: (1) **Cleanup resources** - remove from Map (`connectedUsers.delete(client.id)`); (2) **Broadcast offline** - `client.broadcast.emit('user:offline', { userId })` to notify others; (3) **Update database** - set status to 'offline', save last seen timestamp, end session record; (4) **Save session data** - store duration, message count, activity log; (5) **Leave rooms** - notify rooms that user left (Socket.IO leaves automatically). **Best practices**: wrap in try-catch (client already gone, don't throw), use async for DB operations, calculate session duration (`Date.now() - connectedAt`), log disconnect stats, handle gracefully even if operations fail. **Reconnection pattern**: use timeout before marking truly offline (30s grace period) to handle temporary disconnects. **Don't throw errors** - client already disconnected, throwing crashes gateway. **Always cleanup memory** even if other operations fail.

</details>

## Event Handling

14. How do you listen to client events using `@SubscribeMessage()`?

<details>
<summary><strong>Answer</strong></summary>

**`@SubscribeMessage()`** is a method decorator that marks a method to handle specific WebSocket events from clients. It takes an **event name** (string) and the decorated method is called whenever a client emits that event. Parameters are accessed using `@MessageBody()` (data) and `@ConnectedSocket()` (client).

### **Basic Syntax**

```typescript
import {
  WebSocketGateway,
  SubscribeMessage,
  MessageBody,
  ConnectedSocket,
} from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway()
export class ChatGateway {
  // Listen to 'message' event from client
  @SubscribeMessage('message')
  handleMessage(
    @MessageBody() data: any,        // Event data (payload)
    @ConnectedSocket() client: Socket, // Client who sent it
  ) {
    console.log(`Received from ${client.id}:`, data);
    return { event: 'message', data: `Echo: ${data}` };
  }
}

// Client-side:
// socket.emit('message', 'Hello'); // Triggers handleMessage()
// socket.on('message', (data) => console.log(data)); // Receives echo
```

### **Simple Examples**

```typescript
@WebSocketGateway()
export class SimpleEventsGateway {
  // EXAMPLE 1: Simple message
  @SubscribeMessage('ping')
  handlePing() {
    console.log('Received ping');
    return 'pong'; // Simple string response
  }

  // EXAMPLE 2: With data
  @SubscribeMessage('greet')
  handleGreet(@MessageBody() name: string) {
    return `Hello, ${name}!`;
  }

  // EXAMPLE 3: With client socket
  @SubscribeMessage('whoami')
  handleWhoAmI(@ConnectedSocket() client: Socket) {
    return {
      socketId: client.id,
      message: 'This is your socket ID',
    };
  }

  // EXAMPLE 4: Both data and client
  @SubscribeMessage('echo')
  handleEcho(
    @MessageBody() data: string,
    @ConnectedSocket() client: Socket,
  ) {
    console.log(`Echo from ${client.id}: ${data}`);
    return data;
  }
}
```

### **Parameter Decorators**

```typescript
@WebSocketGateway()
export class ParameterExamplesGateway {
  // DECORATOR 1: @MessageBody() - Get entire payload
  @SubscribeMessage('send-message')
  handleMessage(@MessageBody() data: any) {
    // data = entire payload sent by client
    console.log('Data:', data);
    return { success: true };
  }

  // DECORATOR 2: @MessageBody('property') - Get specific property
  @SubscribeMessage('update-profile')
  handleUpdate(
    @MessageBody('name') name: string,
    @MessageBody('age') age: number,
  ) {
    // Extract specific properties from payload
    console.log(`Name: ${name}, Age: ${age}`);
    return { success: true };
  }

  // DECORATOR 3: @ConnectedSocket() - Get client socket
  @SubscribeMessage('get-info')
  handleGetInfo(@ConnectedSocket() client: Socket) {
    return {
      id: client.id,
      rooms: Array.from(client.rooms),
    };
  }

  // DECORATOR 4: Combined (most common)
  @SubscribeMessage('chat-message')
  handleChatMessage(
    @MessageBody() data: { message: string; roomId: string },
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;
    console.log(`User ${userId} sent: ${data.message}`);
    return { success: true };
  }

  // WITHOUT DECORATORS: Get raw arguments (not recommended)
  @SubscribeMessage('raw')
  handleRaw(...args: any[]) {
    // args[0] = client Socket
    // args[1] = data
    const [client, data] = args;
    console.log('Raw:', client.id, data);
  }
}
```

### **Return Values**

```typescript
@WebSocketGateway()
export class ReturnValuesGateway {
  @WebSocketServer()
  server: Server;

  // RETURN 1: Nothing (void)
  @SubscribeMessage('no-response')
  handleNoResponse() {
    console.log('Received event, no response');
    // Client won't receive anything
  }

  // RETURN 2: Simple value (string, number, boolean)
  @SubscribeMessage('get-count')
  handleGetCount() {
    return 42; // Client receives: 42
  }

  // RETURN 3: Object (auto-wrapped in event)
  @SubscribeMessage('get-user')
  handleGetUser() {
    return { id: '123', name: 'John' };
    // Client receives: { id: '123', name: 'John' }
  }

  // RETURN 4: Event object (explicit event name)
  @SubscribeMessage('request-data')
  handleRequestData() {
    return {
      event: 'data-response', // Custom event name
      data: { value: 'Hello' },
    };
    // Client must listen to 'data-response' event
  }

  // RETURN 5: Observable (for streaming)
  @SubscribeMessage('stream')
  handleStream() {
    return interval(1000).pipe(
      take(5),
      map(i => ({ data: i })),
    );
    // Client receives 5 events, one per second
  }

  // RETURN 6: Promise (async)
  @SubscribeMessage('async-data')
  async handleAsyncData() {
    const data = await this.fetchData();
    return data;
  }

  private async fetchData() {
    return { message: 'Async data' };
  }
}
```

### **With DTOs and Validation**

```typescript
// DTO with validation
import { IsNotEmpty, IsString, IsUUID } from 'class-validator';

export class SendMessageDto {
  @IsNotEmpty()
  @IsString()
  message: string;

  @IsNotEmpty()
  @IsUUID()
  roomId: string;
}

// Gateway with validation
import { UsePipes, ValidationPipe } from '@nestjs/common';

@WebSocketGateway()
export class ValidatedGateway {
  // Apply validation pipe to specific handler
  @SubscribeMessage('send-message')
  @UsePipes(new ValidationPipe())
  handleSendMessage(
    @MessageBody() dto: SendMessageDto,
    @ConnectedSocket() client: Socket,
  ) {
    // dto is validated - will throw WsException if invalid
    console.log(`Valid message: ${dto.message} to room ${dto.roomId}`);
    return { success: true };
  }

  // Apply validation to entire gateway
  @UsePipes(new ValidationPipe())
  @SubscribeMessage('update-profile')
  handleUpdateProfile(@MessageBody() dto: UpdateProfileDto) {
    return { success: true };
  }
}
```

### **Real-World Examples**

#### **1. Chat Application**

```typescript
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  constructor(private chatService: ChatService) {}

  // Send message to room
  @SubscribeMessage('send-message')
  async handleSendMessage(
    @MessageBody() data: { message: string; roomId: string },
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;

    // Save to database
    const message = await this.chatService.createMessage({
      userId,
      content: data.message,
      roomId: data.roomId,
    });

    // Broadcast to room
    this.server.to(data.roomId).emit('new-message', {
      id: message.id,
      userId,
      content: data.message,
      timestamp: message.createdAt,
    });

    return { success: true, messageId: message.id };
  }

  // Join room
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    client.join(roomId);
    
    // Notify others in room
    client.to(roomId).emit('user-joined', {
      userId: client.handshake.auth.userId,
      roomId,
    });

    return { success: true, roomId };
  }

  // Typing indicator
  @SubscribeMessage('typing')
  handleTyping(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;
    
    // Broadcast to room except sender
    client.to(roomId).emit('user-typing', { userId, roomId });
  }

  // Stop typing
  @SubscribeMessage('stop-typing')
  handleStopTyping(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;
    client.to(roomId).emit('user-stop-typing', { userId, roomId });
  }
}
```

#### **2. Real-time Notifications**

```typescript
@WebSocketGateway({ namespace: '/notifications' })
export class NotificationsGateway {
  @WebSocketServer()
  server: Server;

  // Subscribe to notifications
  @SubscribeMessage('subscribe')
  handleSubscribe(
    @MessageBody() userId: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Join personal notification room
    client.join(`notifications:${userId}`);
    
    return {
      success: true,
      message: 'Subscribed to notifications',
    };
  }

  // Mark notification as read
  @SubscribeMessage('mark-read')
  async handleMarkRead(
    @MessageBody() notificationId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;
    
    await this.notificationService.markAsRead(notificationId, userId);
    
    return { success: true };
  }

  // Get unread count
  @SubscribeMessage('get-unread-count')
  async handleGetUnreadCount(@ConnectedSocket() client: Socket) {
    const userId = client.handshake.auth.userId;
    const count = await this.notificationService.getUnreadCount(userId);
    
    return { count };
  }
}
```

#### **3. Real-time Game**

```typescript
@WebSocketGateway({ namespace: '/game' })
export class GameGateway {
  private gameRooms = new Map<string, GameRoom>();

  // Create game room
  @SubscribeMessage('create-room')
  handleCreateRoom(
    @MessageBody() settings: GameSettings,
    @ConnectedSocket() client: Socket,
  ) {
    const roomId = generateId();
    
    this.gameRooms.set(roomId, {
      id: roomId,
      players: [client.id],
      settings,
      state: 'waiting',
    });

    client.join(roomId);
    
    return { roomId, message: 'Room created' };
  }

  // Join game room
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const room = this.gameRooms.get(roomId);
    
    if (!room) {
      return { error: 'Room not found' };
    }

    if (room.players.length >= room.settings.maxPlayers) {
      return { error: 'Room is full' };
    }

    room.players.push(client.id);
    client.join(roomId);

    // Notify all players
    this.server.to(roomId).emit('player-joined', {
      playerId: client.id,
      playerCount: room.players.length,
    });

    return { success: true, room };
  }

  // Player move
  @SubscribeMessage('player-move')
  handlePlayerMove(
    @MessageBody() data: { roomId: string; move: any },
    @ConnectedSocket() client: Socket,
  ) {
    // Broadcast move to other players
    client.to(data.roomId).emit('player-moved', {
      playerId: client.id,
      move: data.move,
    });

    return { success: true };
  }
}
```

### **Multiple Event Names**

```typescript
@WebSocketGateway()
export class MultipleEventsGateway {
  // Listen to multiple event names (same handler)
  @SubscribeMessage('message')
  @SubscribeMessage('msg')
  @SubscribeMessage('chat')
  handleMessage(@MessageBody() data: string) {
    // All three events trigger this handler
    return `Received: ${data}`;
  }
}
```

### **With Guards and Interceptors**

```typescript
import { UseGuards, UseInterceptors } from '@nestjs/common';
import { WsJwtGuard } from './guards/ws-jwt.guard';
import { LoggingInterceptor } from './interceptors/logging.interceptor';

@WebSocketGateway()
export class SecureGateway {
  // Apply guard to specific handler
  @SubscribeMessage('protected-action')
  @UseGuards(WsJwtGuard)
  handleProtectedAction(@ConnectedSocket() client: Socket) {
    // Only authenticated users reach here
    return { success: true };
  }

  // Apply interceptor
  @SubscribeMessage('logged-action')
  @UseInterceptors(LoggingInterceptor)
  handleLoggedAction(@MessageBody() data: any) {
    return { success: true };
  }

  // Apply both
  @SubscribeMessage('secure-logged')
  @UseGuards(WsJwtGuard)
  @UseInterceptors(LoggingInterceptor)
  handleSecureLogged(@MessageBody() data: any) {
    return { success: true };
  }
}
```

### **Async/Await with Database Operations**

```typescript
@WebSocketGateway()
export class AsyncGateway {
  constructor(private userService: UserService) {}

  @SubscribeMessage('get-profile')
  async handleGetProfile(@MessageBody() userId: string) {
    // Async operation
    const user = await this.userService.findOne(userId);
    
    if (!user) {
      throw new WsException('User not found');
    }

    return {
      id: user.id,
      name: user.name,
      email: user.email,
    };
  }

  @SubscribeMessage('update-settings')
  async handleUpdateSettings(
    @MessageBody() settings: any,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;
    
    try {
      await this.userService.updateSettings(userId, settings);
      return { success: true };
    } catch (error) {
      throw new WsException('Failed to update settings');
    }
  }
}
```

### **Error Handling**

```typescript
import { WsException } from '@nestjs/websockets';

@WebSocketGateway()
export class ErrorHandlingGateway {
  @SubscribeMessage('risky-operation')
  handleRiskyOperation(@MessageBody() data: any) {
    try {
      // Your logic
      if (!data.required) {
        throw new WsException('Missing required field');
      }

      return { success: true };
      
    } catch (error) {
      if (error instanceof WsException) {
        throw error; // Re-throw WsException
      }
      
      // Wrap other errors
      throw new WsException('Internal server error');
    }
  }
}
```

**Interview Tip**: **`@SubscribeMessage('event-name')`** is a method decorator that **listens to client events**. **Parameters**: use `@MessageBody()` for event data (entire payload or specific property), `@ConnectedSocket()` for Socket instance. **Return values**: (1) void (no response), (2) simple value/object (auto-sent to client), (3) `{ event: 'name', data: {...} }` (custom event), (4) Observable (streaming), (5) Promise (async). **Client emits**: `socket.emit('event-name', data)` triggers handler. **Client receives**: `socket.on('event-name', callback)` or auto-response. **DTOs**: use with `@UsePipes(new ValidationPipe())` for validation. **Guards**: `@UseGuards(WsJwtGuard)` for authentication. **Common pattern**: `@MessageBody()` for data, `@ConnectedSocket()` for client, save to DB, broadcast to room with `server.to(roomId).emit()`, return confirmation. **Multiple events**: stack decorators for same handler. **Error handling**: throw `WsException` for client errors. **Async**: methods can be async/await for DB operations.

</details>
15. How do you emit events to clients using `@WebSocketServer()` and `server.emit()`?

<details>
<summary><strong>Answer</strong></summary>

**`@WebSocketServer()`** is a property decorator that injects the Socket.IO **Server instance** into your gateway. Use **`server.emit()`** to send events to **all connected clients**. You can also use `server.to(room)` for room-specific broadcasts, or access individual client sockets.

### **Basic Usage**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
} from '@nestjs/websockets';
import { Server } from 'socket.io';

@WebSocketGateway()
export class ChatGateway {
  // Inject Socket.IO server instance
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('message')
  handleMessage(@MessageBody() data: string) {
    // Emit to ALL connected clients
    this.server.emit('new-message', {
      message: data,
      timestamp: new Date(),
    });
  }
}

// Client-side:
// socket.on('new-message', (data) => {
//   console.log(data.message);
// });
```

### **Server Instance Methods**

```typescript
@WebSocketGateway()
export class EmitExamplesGateway {
  @WebSocketServer()
  server: Server;

  // METHOD 1: server.emit() - Send to ALL clients
  sendToAll() {
    this.server.emit('announcement', {
      message: 'Server announcement',
      timestamp: new Date(),
    });
    // All connected clients receive this
  }

  // METHOD 2: server.to(room) - Send to specific room
  sendToRoom(roomId: string) {
    this.server.to(roomId).emit('room-message', {
      message: 'Message for room',
      roomId,
    });
    // Only clients in this room receive it
  }

  // METHOD 3: server.in(room) - Alias for to()
  sendToRoomAlias(roomId: string) {
    this.server.in(roomId).emit('room-message', {
      message: 'Message for room',
    });
    // Same as server.to(roomId)
  }

  // METHOD 4: server.except(room) - Send to all EXCEPT room
  sendExceptRoom(roomId: string) {
    this.server.except(roomId).emit('message', {
      message: 'Everyone except this room',
    });
  }

  // METHOD 5: Access specific socket by ID
  sendToSpecificSocket(socketId: string) {
    this.server.to(socketId).emit('private-message', {
      message: 'Message for you only',
    });
  }

  // METHOD 6: Multiple rooms
  sendToMultipleRooms(rooms: string[]) {
    this.server.to(rooms).emit('multi-room-message', {
      message: 'Message for multiple rooms',
    });
  }
}
```

### **Common Patterns**

#### **1. Broadcasting from Event Handler**

```typescript
@WebSocketGateway()
export class BroadcastGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('send-message')
  handleSendMessage(
    @MessageBody() data: { message: string; roomId: string },
    @ConnectedSocket() client: Socket,
  ) {
    // Broadcast to room
    this.server.to(data.roomId).emit('new-message', {
      userId: client.handshake.auth.userId,
      message: data.message,
      timestamp: new Date(),
    });

    return { success: true };
  }
}
```

#### **2. Broadcasting from Service**

```typescript
// notifications.service.ts
import { Injectable } from '@nestjs/common';
import { NotificationsGateway } from './notifications.gateway';

@Injectable()
export class NotificationsService {
  constructor(private notificationsGateway: NotificationsGateway) {}

  async sendNotification(userId: string, message: string) {
    // Save to database
    const notification = await this.saveToDb(userId, message);

    // Send via WebSocket using gateway's server
    this.notificationsGateway.server.to(`user:${userId}`).emit('notification', {
      id: notification.id,
      message,
      timestamp: new Date(),
    });
  }

  private async saveToDb(userId: string, message: string) {
    // Database logic
    return { id: '123', userId, message };
  }
}

// notifications.gateway.ts
@WebSocketGateway({ namespace: '/notifications' })
export class NotificationsGateway {
  @WebSocketServer()
  server: Server; // Public so service can access it

  @SubscribeMessage('subscribe')
  handleSubscribe(
    @MessageBody() userId: string,
    @ConnectedSocket() client: Socket,
  ) {
    client.join(`user:${userId}`);
  }
}
```

#### **3. Scheduled Broadcasts**

```typescript
import { Injectable } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';

@Injectable()
@WebSocketGateway()
export class ScheduledGateway {
  @WebSocketServer()
  server: Server;

  // Send broadcast every minute
  @Cron('*/1 * * * *')
  handleCron() {
    this.server.emit('server-time', {
      time: new Date().toISOString(),
    });
  }

  // Send broadcast at specific time
  @Cron('0 9 * * *') // 9 AM daily
  sendDailyBroadcast() {
    this.server.emit('daily-update', {
      message: 'Good morning! Here is your daily update.',
      timestamp: new Date(),
    });
  }
}
```

#### **4. Event-Driven Broadcasts**

```typescript
// order.service.ts
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class OrderService {
  constructor(private eventEmitter: EventEmitter2) {}

  async createOrder(orderData: any) {
    const order = await this.saveOrder(orderData);
    
    // Emit domain event
    this.eventEmitter.emit('order.created', {
      orderId: order.id,
      userId: order.userId,
      total: order.total,
    });

    return order;
  }

  private async saveOrder(data: any) {
    return { id: '123', userId: 'user1', total: 100 };
  }
}

// order.gateway.ts
import { OnEvent } from '@nestjs/event-emitter';

@WebSocketGateway()
export class OrderGateway {
  @WebSocketServer()
  server: Server;

  // Listen to domain event and broadcast via WebSocket
  @OnEvent('order.created')
  handleOrderCreated(payload: any) {
    // Send to specific user
    this.server.to(`user:${payload.userId}`).emit('order-created', {
      orderId: payload.orderId,
      total: payload.total,
      message: 'Your order has been created!',
    });

    // Send to admin dashboard
    this.server.to('admin-room').emit('new-order', payload);
  }
}
```

### **Emit with Acknowledgments**

```typescript
@WebSocketGateway()
export class AckGateway {
  @WebSocketServer()
  server: Server;

  async sendWithAck(socketId: string) {
    // Emit with callback (acknowledgment)
    this.server.to(socketId).emit(
      'important-message',
      { data: 'Important data' },
      (response: any) => {
        // This callback is called when client acknowledges
        console.log('Client acknowledged:', response);
      },
    );
  }
}

// Client-side:
// socket.on('important-message', (data, callback) => {
//   console.log('Received:', data);
//   callback({ received: true }); // Send acknowledgment
// });
```

### **Volatile Events (Skip if Client Busy)**

```typescript
@WebSocketGateway()
export class VolatileGateway {
  @WebSocketServer()
  server: Server;

  sendVolatileEvent() {
    // Use volatile() to skip if client's buffer is full
    this.server.volatile.emit('real-time-update', {
      data: 'Non-critical update',
    });
    // If client is busy, this event is dropped (not buffered)
  }
}
```

### **Compress Events**

```typescript
@WebSocketGateway()
export class CompressedGateway {
  @WebSocketServer()
  server: Server;

  sendLargeData() {
    // Compress large data before sending
    this.server.compress(true).emit('large-data', {
      data: '...very large payload...',
    });
  }
}
```

### **Real-World Complete Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Injectable } from '@nestjs/common';

@Injectable()
@WebSocketGateway({ namespace: '/chat', cors: true })
export class ChatGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  private connectedUsers = new Map<string, string>(); // socketId -> userId

  constructor(
    private chatService: ChatService,
    private presenceService: PresenceService,
  ) {}

  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    this.connectedUsers.set(client.id, userId);
    
    // Join personal room
    client.join(`user:${userId}`);

    // EMIT 1: Broadcast to all that user is online
    this.server.emit('user:online', {
      userId,
      timestamp: new Date(),
    });

    // EMIT 2: Send online users list to new client only
    const onlineUsers = Array.from(this.connectedUsers.values());
    client.emit('online-users', onlineUsers);
  }

  // Send message to room
  @SubscribeMessage('send-message')
  async handleSendMessage(
    @MessageBody() data: { message: string; roomId: string },
    @ConnectedSocket() client: Socket,
  ) {
    const userId = this.connectedUsers.get(client.id);

    // Save to database
    const message = await this.chatService.saveMessage({
      userId,
      content: data.message,
      roomId: data.roomId,
    });

    // EMIT: Broadcast to room
    this.server.to(data.roomId).emit('new-message', {
      id: message.id,
      userId,
      content: data.message,
      roomId: data.roomId,
      timestamp: message.createdAt,
    });

    return { success: true, messageId: message.id };
  }

  // Send typing indicator
  @SubscribeMessage('typing')
  handleTyping(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = this.connectedUsers.get(client.id);

    // EMIT: Broadcast to room except sender
    client.to(roomId).emit('user-typing', {
      userId,
      roomId,
    });
  }

  // Admin broadcast (called from service)
  async sendAdminBroadcast(message: string) {
    // EMIT: Send to all connected clients
    this.server.emit('admin-broadcast', {
      message,
      timestamp: new Date(),
      priority: 'high',
    });
  }

  // Send notification to specific user (called from service)
  async notifyUser(userId: string, notification: any) {
    // EMIT: Send to specific user's room
    this.server.to(`user:${userId}`).emit('notification', {
      ...notification,
      timestamp: new Date(),
    });
  }

  // Get connection statistics
  getStats() {
    return {
      connectedClients: this.server.sockets.sockets.size,
      rooms: Array.from(this.server.sockets.adapter.rooms.keys()),
    };
  }
}
```

### **Server Properties**

```typescript
@WebSocketGateway()
export class ServerPropertiesGateway {
  @WebSocketServer()
  server: Server;

  inspectServer() {
    // Get all connected sockets
    const sockets = this.server.sockets.sockets;
    console.log(`Connected sockets: ${sockets.size}`);

    // Get all rooms
    const rooms = this.server.sockets.adapter.rooms;
    console.log('Rooms:', Array.from(rooms.keys()));

    // Get namespace
    console.log('Namespace:', this.server.name);

    // Access specific socket by ID
    const socket = this.server.sockets.sockets.get('socket-id');
    if (socket) {
      socket.emit('direct-message', { message: 'Hello' });
    }

    // Get clients in specific room
    const roomClients = this.server.sockets.adapter.rooms.get('room-id');
    console.log('Clients in room:', roomClients?.size);
  }
}
```

### **Emit from Outside Gateway**

```typescript
// any.service.ts
import { Injectable } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';

@Injectable()
export class AnyService {
  constructor(private chatGateway: ChatGateway) {}

  async someBusinessLogic() {
    // Do business logic
    const result = await this.processData();

    // Emit event via gateway
    this.chatGateway.server.emit('data-processed', {
      result,
      timestamp: new Date(),
    });
  }

  private async processData() {
    return { data: 'processed' };
  }
}

// Make sure gateway is injectable
@Injectable()
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server; // Must be public to access from service
}

// Module registration
@Module({
  providers: [ChatGateway, AnyService],
})
export class ChatModule {}
```

### **Common Mistakes**

```typescript
// ❌ MISTAKE 1: Forgetting @WebSocketServer() decorator
@WebSocketGateway()
export class BadGateway {
  server: Server; // undefined! Missing decorator

  sendMessage() {
    this.server.emit('message', {}); // ERROR: Cannot read property 'emit' of undefined
  }
}

// ✅ CORRECT:
@WebSocketGateway()
export class GoodGateway {
  @WebSocketServer()
  server: Server; // Properly injected

  sendMessage() {
    this.server.emit('message', {});
  }
}

// ❌ MISTAKE 2: Using server before initialization
@WebSocketGateway()
export class InitGateway {
  @WebSocketServer()
  server: Server;

  constructor() {
    // server is undefined here!
    this.server.emit('message', {}); // ERROR
  }
}

// ✅ CORRECT: Use afterInit
@WebSocketGateway()
export class InitGateway implements OnGatewayInit {
  @WebSocketServer()
  server: Server;

  afterInit(server: Server) {
    // server is ready now
    this.server.emit('message', {});
  }
}

// ❌ MISTAKE 3: Trying to emit to single client with server.emit()
@WebSocketGateway()
export class WrongEmitGateway {
  @WebSocketServer()
  server: Server;

  sendToUser(userId: string) {
    this.server.emit('message', { userId }); // Wrong! Goes to ALL clients
  }
}

// ✅ CORRECT: Use server.to()
@WebSocketGateway()
export class CorrectEmitGateway {
  @WebSocketServer()
  server: Server;

  sendToUser(userId: string) {
    this.server.to(`user:${userId}`).emit('message', { userId }); // Correct!
  }
}
```

**Interview Tip**: **`@WebSocketServer()`** decorator **injects Socket.IO Server instance** into gateway property. **`server.emit(event, data)`** sends to **all connected clients**. **Common methods**: (1) `server.emit()` - all clients, (2) `server.to(room)` - specific room, (3) `server.to(socketId)` - specific client by ID, (4) `server.except(room)` - all except room, (5) `server.to([room1, room2])` - multiple rooms. **Access from service**: inject gateway, use `gateway.server.emit()`. **Must be public** property to access from other classes. **Available after** `afterInit()` lifecycle hook, **undefined in constructor**. **Real-world pattern**: save to DB first, then `server.to(roomId).emit()` to broadcast. **Event-driven**: listen to domain events with `@OnEvent()`, broadcast via WebSocket. **Scheduled**: use `@Cron()` for periodic broadcasts. **Properties**: `server.sockets.sockets` (all sockets), `server.sockets.adapter.rooms` (all rooms).

</details>
16. What is the difference between `emit()`, `broadcast()`, and `to()` methods?

<details>
<summary><strong>Answer</strong></summary>

**Three key broadcasting methods**: (1) **`emit()`** - sends event to target (client socket or all clients via server), (2) **`broadcast()`** - sends to all clients **except sender**, (3) **`to(room)`** - sends to specific room(s). Each serves different use cases for controlling message recipients.

### **Quick Comparison Table**

| Method | Sender | Target | Use Case |
|--------|--------|--------|----------|
| `client.emit()` | ❌ No | ✅ This client only | Send confirmation/response to sender |
| `client.broadcast.emit()` | ❌ No | ✅ All except sender | Notify others of sender's action |
| `server.emit()` | ✅ Yes | ✅ All clients | Global announcements |
| `client.to(room).emit()` | ❌ No | ✅ Room (except sender if in room) | Room-specific messages |
| `server.to(room).emit()` | ✅ Yes | ✅ Room (including sender if in room) | Broadcast to room from service |

### **1. emit() - Direct Emission**

```typescript
@WebSocketGateway()
export class EmitGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('message')
  handleMessage(
    @MessageBody() data: string,
    @ConnectedSocket() client: Socket,
  ) {
    // CLIENT EMIT: Send to THIS client only
    client.emit('response', {
      message: `You sent: ${data}`,
      timestamp: new Date(),
    });
    // Only the sender receives this

    // SERVER EMIT: Send to ALL clients (including sender)
    this.server.emit('announcement', {
      message: 'New message received',
    });
    // Every connected client receives this
  }
}

// Summary:
// client.emit()  → Send to sender only
// server.emit()  → Send to everyone
```

### **2. broadcast - Exclude Sender**

```typescript
@WebSocketGateway()
export class BroadcastGateway {
  @SubscribeMessage('user-action')
  handleUserAction(
    @MessageBody() action: string,
    @ConnectedSocket() client: Socket,
  ) {
    // BROADCAST: Send to all EXCEPT sender
    client.broadcast.emit('user-did-action', {
      userId: client.id,
      action,
      timestamp: new Date(),
    });
    // Sender does NOT receive this
    // All other clients DO receive it

    // Send confirmation to sender separately
    client.emit('action-confirmed', {
      action,
      success: true,
    });
  }
}

// Use case: User joins chat
// - Tell everyone else: "John joined" (broadcast)
// - Tell John: "You joined successfully" (emit to John only)
```

### **3. to() - Room-Specific**

```typescript
@WebSocketGateway()
export class RoomGateway {
  @SubscribeMessage('send-to-room')
  handleSendToRoom(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    // TO: Send to specific room
    client.to(data.roomId).emit('room-message', {
      userId: client.id,
      message: data.message,
    });
    // Only clients in this room receive it
    // SENDER does NOT receive it (if they're in the room)
  }
}

// Variations:
// client.to(room).emit()  → Room members except sender
// server.to(room).emit()  → All room members including sender
```

### **Complete Comparison Examples**

```typescript
@WebSocketGateway()
export class ComparisonGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('test-emissions')
  handleTestEmissions(@ConnectedSocket() client: Socket) {
    // Assume client is in room 'room1'
    client.join('room1');

    // ============================================
    // 1. client.emit() - SENDER ONLY
    // ============================================
    client.emit('message', { text: 'For you only' });
    // ✅ Sender receives
    // ❌ Others don't receive

    // ============================================
    // 2. client.broadcast.emit() - ALL EXCEPT SENDER
    // ============================================
    client.broadcast.emit('message', { text: 'For everyone else' });
    // ❌ Sender doesn't receive
    // ✅ All others receive

    // ============================================
    // 3. server.emit() - EVERYONE
    // ============================================
    this.server.emit('message', { text: 'For everyone including sender' });
    // ✅ Sender receives
    // ✅ All others receive

    // ============================================
    // 4. client.to(room).emit() - ROOM EXCEPT SENDER
    // ============================================
    client.to('room1').emit('message', { text: 'For room except me' });
    // ❌ Sender doesn't receive (even though in room)
    // ✅ Other room members receive

    // ============================================
    // 5. server.to(room).emit() - ALL IN ROOM
    // ============================================
    this.server.to('room1').emit('message', { text: 'For all in room' });
    // ✅ Sender receives (if in room)
    // ✅ Other room members receive

    // ============================================
    // 6. client.broadcast.to(room).emit() - ROOM EXCEPT SENDER
    // ============================================
    client.broadcast.to('room1').emit('message', { text: 'Room except me' });
    // ❌ Sender doesn't receive
    // ✅ Other room members receive
    // Same as client.to(room).emit()
  }
}
```

### **Real-World Chat Application**

```typescript
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  constructor(private chatService: ChatService) {}

  // SCENARIO 1: User sends message to room
  @SubscribeMessage('send-message')
  async handleSendMessage(
    @MessageBody() data: { message: string; roomId: string },
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;

    // Save to database
    const message = await this.chatService.saveMessage({
      userId,
      content: data.message,
      roomId: data.roomId,
    });

    // 1. EMIT: Confirm to sender
    client.emit('message-sent', {
      messageId: message.id,
      success: true,
    });

    // 2. BROADCAST TO ROOM: Notify others in room
    client.to(data.roomId).emit('new-message', {
      id: message.id,
      userId,
      content: data.message,
      roomId: data.roomId,
      timestamp: message.createdAt,
    });
    // Sender already has the message (optimistic update),
    // so only send to others
  }

  // SCENARIO 2: User joins room
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;

    // Join the room
    client.join(roomId);

    // 1. EMIT: Confirm to joiner
    client.emit('joined-room', {
      roomId,
      message: `You joined ${roomId}`,
    });

    // 2. BROADCAST TO ROOM: Notify others
    client.to(roomId).emit('user-joined', {
      userId,
      username,
      roomId,
      message: `${username} joined the room`,
    });
    // Others see "John joined", but John sees "You joined"
  }

  // SCENARIO 3: User typing indicator
  @SubscribeMessage('typing')
  handleTyping(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;

    // BROADCAST TO ROOM: Notify others (not sender)
    client.to(roomId).emit('user-typing', {
      userId,
      roomId,
    });
    // Sender doesn't need to see their own typing indicator
  }

  // SCENARIO 4: Admin broadcast (from service)
  async sendAdminAnnouncement(message: string) {
    // SERVER EMIT: Send to everyone
    this.server.emit('admin-announcement', {
      message,
      timestamp: new Date(),
      priority: 'high',
    });
    // All users see the announcement
  }

  // SCENARIO 5: User leaves room
  @SubscribeMessage('leave-room')
  handleLeaveRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;

    // BROADCAST TO ROOM: Notify others BEFORE leaving
    client.to(roomId).emit('user-left', {
      userId,
      username,
      roomId,
      message: `${username} left the room`,
    });

    // Leave the room
    client.leave(roomId);

    // EMIT: Confirm to leaver
    client.emit('left-room', {
      roomId,
      message: `You left ${roomId}`,
    });
  }
}
```

### **Chaining Methods**

```typescript
@WebSocketGateway()
export class ChainGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('complex-emit')
  handleComplexEmit(@ConnectedSocket() client: Socket) {
    // CHAIN 1: to() + to() - Multiple rooms
    client.to('room1').to('room2').emit('message', {
      text: 'Message for room1 and room2',
    });
    // Sent to both rooms, excluding sender

    // CHAIN 2: broadcast + to() - All except sender in room
    client.broadcast.to('room1').emit('message', {
      text: 'Everyone in room1 except me',
    });
    // Same as client.to('room1').emit()

    // CHAIN 3: except() + emit() - All except specific room
    this.server.except('room1').emit('message', {
      text: 'Everyone except room1',
    });

    // CHAIN 4: to() + volatile - Non-critical room message
    client.to('room1').volatile.emit('update', {
      data: 'Non-critical update',
    });
    // Dropped if client buffer is full

    // CHAIN 5: to() + compress - Compress for specific room
    this.server.to('room1').compress(true).emit('large-data', {
      data: 'Large payload',
    });
  }
}
```

### **Multiple Rooms**

```typescript
@WebSocketGateway()
export class MultiRoomGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('multi-room-message')
  handleMultiRoom(@ConnectedSocket() client: Socket) {
    // Send to multiple rooms (array)
    client.to(['room1', 'room2', 'room3']).emit('message', {
      text: 'Message for 3 rooms',
    });

    // Alternative: Chain to() calls
    client
      .to('room1')
      .to('room2')
      .to('room3')
      .emit('message', {
        text: 'Same as above',
      });
  }
}
```

### **Visual Flow Diagram**

```typescript
/**
 * CLIENT METHODS:
 * 
 * client.emit('event', data)
 *   └─> [Sender] ✅
 * 
 * client.broadcast.emit('event', data)
 *   ├─> [Sender] ❌
 *   └─> [Others] ✅
 * 
 * client.to('room').emit('event', data)
 *   ├─> [Sender] ❌
 *   ├─> [Room Members] ✅
 *   └─> [Non-Room Members] ❌
 * 
 * 
 * SERVER METHODS:
 * 
 * server.emit('event', data)
 *   └─> [Everyone] ✅
 * 
 * server.to('room').emit('event', data)
 *   ├─> [Room Members] ✅
 *   └─> [Non-Room Members] ❌
 * 
 * server.except('room').emit('event', data)
 *   ├─> [Room Members] ❌
 *   └─> [Non-Room Members] ✅
 */
```

### **Common Patterns**

```typescript
@WebSocketGateway()
export class PatternsGateway {
  @WebSocketServer()
  server: Server;

  // PATTERN 1: Confirm to sender, notify others
  @SubscribeMessage('action')
  pattern1(@ConnectedSocket() client: Socket) {
    // Sender gets confirmation
    client.emit('action-success', { success: true });
    
    // Others get notification
    client.broadcast.emit('user-action', { userId: client.id });
  }

  // PATTERN 2: Update room, exclude sender
  @SubscribeMessage('update-room')
  pattern2(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Sender already has updated state (optimistic update)
    // Only notify others
    client.to(roomId).emit('room-updated', { roomId });
  }

  // PATTERN 3: Broadcast from service (include everyone)
  async pattern3() {
    // Send to all users
    this.server.emit('system-update', {
      message: 'System maintenance in 5 minutes',
    });
  }

  // PATTERN 4: Notify specific user (from another user's action)
  @SubscribeMessage('send-friend-request')
  async pattern4(
    @MessageBody() targetUserId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const senderId = client.handshake.auth.userId;
    
    // Confirm to sender
    client.emit('request-sent', { success: true });
    
    // Notify target user
    this.server.to(`user:${targetUserId}`).emit('friend-request', {
      from: senderId,
    });
  }
}
```

### **Performance Considerations**

```typescript
@WebSocketGateway()
export class PerformanceGateway {
  @WebSocketServer()
  server: Server;

  // GOOD: Targeted emission (efficient)
  goodEmit(roomId: string) {
    this.server.to(roomId).emit('message', { data: 'Targeted' });
    // Only room members receive it
  }

  // BAD: Broadcast to all then filter on client (inefficient)
  badEmit(roomId: string) {
    this.server.emit('message', {
      data: 'For all',
      targetRoom: roomId, // Client must check if it's for them
    });
    // All clients receive it, waste of bandwidth
  }

  // GOOD: Use rooms for targeting
  efficientBroadcast(userIds: string[]) {
    // Join users to temporary room
    userIds.forEach(userId => {
      this.server.to(`user:${userId}`).emit('notification', {});
    });
  }
}
```

**Interview Tip**: **Three core methods**: (1) **`emit()`** - `client.emit()` sends to **sender only**, `server.emit()` sends to **all clients**; (2) **`broadcast()`** - `client.broadcast.emit()` sends to **all except sender**; (3) **`to(room)`** - `client.to(room)` sends to **room except sender**, `server.to(room)` sends to **all in room including sender**. **Key difference**: sender inclusion - `emit()` includes sender (when used on server), `broadcast` excludes sender, `to()` depends on context (client.to excludes, server.to includes). **Common pattern**: emit confirmation to sender (`client.emit()`), broadcast update to others (`client.broadcast.emit()` or `client.to(room).emit()`). **Multiple rooms**: `client.to(['room1', 'room2'])` or chain `to()` calls. **Chaining**: `client.to(room).volatile.emit()`, `server.except(room).emit()`. **Use cases**: chat (sender gets confirmation, others get new message), typing indicators (broadcast to others, not sender), admin announcements (server.emit to all).

</details>
17. How do you send messages to a specific client?

<details>
<summary><strong>Answer</strong></summary>

To send messages to a **specific client**, use **`server.to(socketId).emit()`** with the client's socket ID, or **`client.emit()`** directly to the connected client, or join clients to **personal rooms** (`user:userId`) and use **`server.to('user:userId').emit()`**. The personal room pattern is most common for production as it persists across reconnections.

### **Method 1: Direct Socket Emit (Within Event Handler)**

```typescript
import {
  WebSocketGateway,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway()
export class DirectEmitGateway {
  @SubscribeMessage('request-data')
  handleRequest(
    @MessageBody() data: any,
    @ConnectedSocket() client: Socket,
  ) {
    // METHOD 1: Direct emit to this specific client
    client.emit('response-data', {
      message: 'This is for you only',
      socketId: client.id,
      timestamp: new Date(),
    });
    // Only this client receives the response
  }
}

// Use case: Responding to client's request
// Client sends event, you reply directly to that client
```

### **Method 2: By Socket ID (Using Server)**

```typescript
import { WebSocketServer } from '@nestjs/websockets';
import { Server } from 'socket.io';

@WebSocketGateway()
export class SocketIdGateway {
  @WebSocketServer()
  server: Server;

  // Send to specific client by socket ID
  sendToSocketId(socketId: string, message: string) {
    this.server.to(socketId).emit('private-message', {
      message,
      timestamp: new Date(),
    });
  }

  // Example: Another user sends message to specific socket
  @SubscribeMessage('send-to-socket')
  handleSendToSocket(
    @MessageBody() data: { targetSocketId: string; message: string },
  ) {
    // Send to target socket
    this.server.to(data.targetSocketId).emit('private-message', {
      message: data.message,
      from: 'another-user',
    });

    return { success: true };
  }
}

// Limitation: Socket ID changes on reconnection!
// Not ideal for persistent user targeting
```

### **Method 3: Personal Room Pattern (RECOMMENDED)**

```typescript
@WebSocketGateway()
export class PersonalRoomGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  // On connection: Join personal room
  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    // Join room named after user ID
    client.join(`user:${userId}`);
    console.log(`User ${userId} joined personal room`);
  }

  // Send to specific user (works even after reconnection)
  async sendToUser(userId: string, message: string) {
    this.server.to(`user:${userId}`).emit('notification', {
      message,
      timestamp: new Date(),
    });
    // Targets the user, not the socket
    // If user reconnects with new socket, still works!
  }

  // Example: User sends private message to another user
  @SubscribeMessage('private-message')
  async handlePrivateMessage(
    @MessageBody() data: { toUserId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    const fromUserId = client.handshake.auth.userId;

    // Send to target user's personal room
    this.server.to(`user:${data.toUserId}`).emit('private-message', {
      from: fromUserId,
      message: data.message,
      timestamp: new Date(),
    });

    // Optionally confirm to sender
    client.emit('message-sent', { success: true });
  }
}

// Best practice: Use personal rooms for user-specific messages
// Pattern: `user:${userId}` as room name
```

### **Method 4: Get Socket from Connected Sockets Map**

```typescript
@WebSocketGateway()
export class SocketMapGateway {
  @WebSocketServer()
  server: Server;

  // Get socket by ID and emit directly
  sendToSocketById(socketId: string, message: string) {
    // Get socket from server's connected sockets
    const socket = this.server.sockets.sockets.get(socketId);
    
    if (socket) {
      socket.emit('message', {
        message,
        timestamp: new Date(),
      });
    } else {
      console.log(`Socket ${socketId} not found (disconnected?)`);
    }
  }

  // Check if socket exists before sending
  async sendSafely(socketId: string, event: string, data: any) {
    const socket = this.server.sockets.sockets.get(socketId);
    
    if (socket && socket.connected) {
      socket.emit(event, data);
      return true;
    }
    return false;
  }
}
```

### **Complete Production Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Injectable } from '@nestjs/common';

@Injectable()
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  // Track users: userId -> Set of socket IDs
  private userSockets = new Map<string, Set<string>>();

  constructor(private chatService: ChatService) {}

  // ================================================
  // CONNECTION: Setup personal room
  // ================================================
  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;

    // Join personal room (primary method)
    client.join(`user:${userId}`);

    // Track socket ID (backup method)
    if (!this.userSockets.has(userId)) {
      this.userSockets.set(userId, new Set());
    }
    this.userSockets.get(userId)!.add(client.id);

    console.log(
      `User ${userId} connected with socket ${client.id}`,
    );
  }

  handleDisconnect(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    // Cleanup tracking
    const sockets = this.userSockets.get(userId);
    if (sockets) {
      sockets.delete(client.id);
      if (sockets.size === 0) {
        this.userSockets.delete(userId);
      }
    }
  }

  // ================================================
  // METHOD 1: Send to user via personal room
  // ================================================
  async sendToUser(userId: string, event: string, data: any) {
    this.server.to(`user:${userId}`).emit(event, data);
    console.log(`Sent ${event} to user ${userId}`);
  }

  // ================================================
  // METHOD 2: Send to all user's sockets
  // ================================================
  async sendToAllUserSockets(userId: string, event: string, data: any) {
    const socketIds = this.userSockets.get(userId);
    
    if (socketIds) {
      socketIds.forEach(socketId => {
        this.server.to(socketId).emit(event, data);
      });
      console.log(
        `Sent ${event} to ${socketIds.size} sockets for user ${userId}`,
      );
    }
  }

  // ================================================
  // METHOD 3: Check if user is online
  // ================================================
  isUserOnline(userId: string): boolean {
    const sockets = this.userSockets.get(userId);
    return sockets ? sockets.size > 0 : false;
  }

  // ================================================
  // EXAMPLE: Private message
  // ================================================
  @SubscribeMessage('send-private-message')
  async handlePrivateMessage(
    @MessageBody() data: { toUserId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    const fromUserId = client.handshake.auth.userId;

    // Save to database
    const message = await this.chatService.savePrivateMessage({
      fromUserId,
      toUserId: data.toUserId,
      content: data.message,
    });

    // Send to recipient
    await this.sendToUser(data.toUserId, 'private-message', {
      id: message.id,
      from: fromUserId,
      message: data.message,
      timestamp: message.createdAt,
    });

    // Confirm to sender
    client.emit('message-sent', {
      messageId: message.id,
      success: true,
    });
  }

  // ================================================
  // EXAMPLE: Send notification from service
  // ================================================
  async notifyUser(userId: string, notification: any) {
    // Check if user is online
    if (this.isUserOnline(userId)) {
      // Send via WebSocket
      await this.sendToUser(userId, 'notification', notification);
    } else {
      // User offline, send push notification or email
      console.log(`User ${userId} offline, using alternative notification`);
    }
  }
}
```

### **Sending from Service**

```typescript
// notification.service.ts
import { Injectable } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';

@Injectable()
export class NotificationService {
  constructor(private chatGateway: ChatGateway) {}

  async sendNotification(userId: string, message: string) {
    // Save to database
    const notification = await this.saveToDb(userId, message);

    // Send via WebSocket to specific user
    await this.chatGateway.sendToUser(userId, 'notification', {
      id: notification.id,
      message,
      timestamp: new Date(),
    });

    return notification;
  }

  private async saveToDb(userId: string, message: string) {
    return { id: '123', userId, message };
  }
}

// Make sure gateway is injectable and exported
@Injectable()
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  async sendToUser(userId: string, event: string, data: any) {
    this.server.to(`user:${userId}`).emit(event, data);
  }
}

@Module({
  providers: [ChatGateway, NotificationService],
  exports: [ChatGateway], // Export to use in other modules
})
export class ChatModule {}
```

### **Multiple Devices Pattern**

```typescript
@WebSocketGateway()
export class MultiDeviceGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  // User can be connected from multiple devices
  // All devices should receive the message
  
  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    const deviceId = client.handshake.auth.deviceId;

    // Join personal room (all devices in same room)
    client.join(`user:${userId}`);

    // Also join device-specific room
    client.join(`device:${deviceId}`);

    console.log(
      `User ${userId} connected from device ${deviceId}`,
    );
  }

  // Send to all user's devices
  async sendToAllDevices(userId: string, data: any) {
    this.server.to(`user:${userId}`).emit('notification', data);
    // All devices (phone, tablet, desktop) receive it
  }

  // Send to specific device only
  async sendToDevice(deviceId: string, data: any) {
    this.server.to(`device:${deviceId}`).emit('notification', data);
    // Only this device receives it
  }
}
```

### **With Acknowledgment**

```typescript
@WebSocketGateway()
export class AckGateway {
  @WebSocketServer()
  server: Server;

  async sendToUserWithAck(
    userId: string,
    event: string,
    data: any,
  ): Promise<boolean> {
    return new Promise((resolve) => {
      // Send with acknowledgment callback
      this.server
        .to(`user:${userId}`)
        .timeout(5000) // 5 second timeout
        .emit(event, data, (err: any, responses: any[]) => {
          if (err) {
            console.error('Acknowledgment error:', err);
            resolve(false);
          } else {
            console.log('Client acknowledged:', responses);
            resolve(true);
          }
        });
    });
  }
}

// Client-side:
// socket.on('event', (data, callback) => {
//   console.log('Received:', data);
//   callback({ received: true }); // Send ack
// });
```

### **Checking if User is Online**

```typescript
@WebSocketGateway()
export class OnlineCheckGateway {
  @WebSocketServer()
  server: Server;

  // Check if any socket is in user's room
  async isUserOnline(userId: string): Promise<boolean> {
    const sockets = await this.server
      .in(`user:${userId}`)
      .fetchSockets();
    return sockets.length > 0;
  }

  // Get count of user's connected devices
  async getUserSocketCount(userId: string): Promise<number> {
    const sockets = await this.server
      .in(`user:${userId}`)
      .fetchSockets();
    return sockets.length;
  }

  // Send only if user is online
  async sendIfOnline(userId: string, event: string, data: any) {
    const isOnline = await this.isUserOnline(userId);
    
    if (isOnline) {
      this.server.to(`user:${userId}`).emit(event, data);
      return true;
    } else {
      console.log(`User ${userId} is offline`);
      return false;
    }
  }
}
```

### **Comparison of Methods**

```typescript
@WebSocketGateway()
export class ComparisonGateway {
  @WebSocketServer()
  server: Server;

  // METHOD 1: Direct client.emit()
  // ✅ Pros: Simple, works in event handlers
  // ❌ Cons: Need client reference, only for current request
  method1(client: Socket) {
    client.emit('message', { data: 'Direct' });
  }

  // METHOD 2: server.to(socketId)
  // ✅ Pros: Can send from anywhere if you have socket ID
  // ❌ Cons: Socket ID changes on reconnection
  method2(socketId: string) {
    this.server.to(socketId).emit('message', { data: 'By ID' });
  }

  // METHOD 3: Personal room (BEST)
  // ✅ Pros: Works across reconnections, user-based not socket-based
  // ✅ Pros: Can send from services, persists
  // ✅ Pros: Supports multiple devices
  // ❌ Cons: Requires setup on connection
  method3(userId: string) {
    this.server.to(`user:${userId}`).emit('message', { data: 'Room' });
  }
}
```

### **Error Handling**

```typescript
@WebSocketGateway()
export class ErrorSafeGateway {
  @WebSocketServer()
  server: Server;

  async sendToUserSafe(
    userId: string,
    event: string,
    data: any,
  ): Promise<{ success: boolean; error?: string }> {
    try {
      // Check if user room exists
      const sockets = await this.server
        .in(`user:${userId}`)
        .fetchSockets();

      if (sockets.length === 0) {
        return {
          success: false,
          error: 'User not connected',
        };
      }

      // Send message
      this.server.to(`user:${userId}`).emit(event, data);

      return { success: true };
    } catch (error) {
      console.error('Error sending to user:', error);
      return {
        success: false,
        error: error.message,
      };
    }
  }
}
```

**Interview Tip**: Send to **specific client** using **3 methods**: (1) **Direct emit** - `client.emit()` within event handler (only for current request); (2) **Socket ID** - `server.to(socketId).emit()` using socket ID (changes on reconnection); (3) **Personal room** (RECOMMENDED) - `client.join('user:' + userId)` on connection, then `server.to('user:' + userId).emit()` anytime (works across reconnections, supports multiple devices). **Personal room pattern** is production standard: join on `handleConnection()`, emit from anywhere in app including services. **Multiple devices**: all devices join same `user:userId` room, one emit reaches all. **Check online**: `await server.in('user:' + userId).fetchSockets()` returns array of sockets (length > 0 = online). **From service**: inject gateway, call `gateway.sendToUser(userId, event, data)`. **With acknowledgment**: use `server.to(room).timeout(5000).emit(event, data, callback)`. **Key advantage of rooms**: user ID doesn't change, socket ID does - rooms persist across reconnections.

</details>
18. How do you broadcast to all clients except sender?

<details>
<summary><strong>Answer</strong></summary>

To broadcast to **all clients except sender**, use **`client.broadcast.emit()`**. This sends the event to every connected client **except** the one who triggered it. You can also combine with **`to(room)`** to broadcast to room members excluding sender: **`client.broadcast.to(room).emit()`** or **`client.to(room).emit()`** (equivalent).

### **Basic Syntax**

```typescript
import {
  WebSocketGateway,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway()
export class BroadcastGateway {
  @SubscribeMessage('action')
  handleAction(
    @MessageBody() data: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Broadcast to ALL clients EXCEPT sender
    client.broadcast.emit('user-action', {
      userId: client.id,
      action: data,
      timestamp: new Date(),
    });
    // Sender does NOT receive this
    // All other connected clients DO receive it

    // Send different message to sender
    client.emit('action-confirmed', {
      success: true,
      action: data,
    });
  }
}
```

### **Common Use Cases**

#### **1. User Joins (Notify Others)**

```typescript
@WebSocketGateway()
export class JoinGateway implements OnGatewayConnection {
  handleConnection(client: Socket) {
    const username = client.handshake.auth.username;

    // Tell everyone EXCEPT the joiner
    client.broadcast.emit('user-joined', {
      username,
      userId: client.handshake.auth.userId,
      message: `${username} joined the chat`,
      timestamp: new Date(),
    });

    // Tell the joiner (different message)
    client.emit('welcome', {
      message: 'Welcome to the chat!',
      username,
    });
  }
}

// Result:
// - New user sees: "Welcome to the chat!"
// - Others see: "John joined the chat"
```

#### **2. Typing Indicator**

```typescript
@WebSocketGateway()
export class TypingGateway {
  @SubscribeMessage('typing')
  handleTyping(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const username = client.handshake.auth.username;

    // Broadcast to room EXCEPT sender
    client.broadcast.to(roomId).emit('user-typing', {
      username,
      roomId,
    });
    // Sender doesn't see their own typing indicator
    // Others in room see "John is typing..."
  }

  @SubscribeMessage('stop-typing')
  handleStopTyping(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const username = client.handshake.auth.username;

    client.broadcast.to(roomId).emit('user-stop-typing', {
      username,
      roomId,
    });
  }
}
```

#### **3. Chat Message (Optimistic Update)**

```typescript
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  constructor(private chatService: ChatService) {}

  @SubscribeMessage('send-message')
  async handleSendMessage(
    @MessageBody() data: { message: string; roomId: string },
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;

    // Save to database
    const message = await this.chatService.saveMessage({
      userId,
      content: data.message,
      roomId: data.roomId,
    });

    // PATTERN: Sender already has message (optimistic update)
    // Only send to others
    client.broadcast.to(data.roomId).emit('new-message', {
      id: message.id,
      userId,
      content: data.message,
      roomId: data.roomId,
      timestamp: message.createdAt,
    });

    // Confirm to sender (with message ID for reconciliation)
    client.emit('message-sent', {
      messageId: message.id,
      tempId: data.tempId, // Client's temporary ID
      success: true,
    });
  }
}
```

#### **4. User Leaves (Notify Others)**

```typescript
@WebSocketGateway()
export class LeaveGateway implements OnGatewayDisconnect {
  handleDisconnect(client: Socket) {
    const username = client.handshake.auth.username;
    const userId = client.handshake.auth.userId;

    // Broadcast to everyone EXCEPT the person leaving
    // (they're already disconnected anyway)
    client.broadcast.emit('user-left', {
      username,
      userId,
      message: `${username} left the chat`,
      timestamp: new Date(),
    });
  }
}
```

#### **5. User Status Change**

```typescript
@WebSocketGateway()
export class StatusGateway {
  @SubscribeMessage('status-change')
  handleStatusChange(
    @MessageBody() status: 'online' | 'away' | 'busy',
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;

    // Broadcast new status to everyone except user
    client.broadcast.emit('user-status-changed', {
      userId,
      status,
      timestamp: new Date(),
    });

    // Confirm to user
    client.emit('status-updated', {
      status,
      success: true,
    });
  }
}
```

### **Broadcast to Room Except Sender**

```typescript
@WebSocketGateway()
export class RoomBroadcastGateway {
  // TWO EQUIVALENT METHODS:

  // Method 1: client.broadcast.to(room)
  method1(client: Socket, roomId: string) {
    client.broadcast.to(roomId).emit('message', {
      text: 'Room members except sender',
    });
  }

  // Method 2: client.to(room) - Same as above!
  method2(client: Socket, roomId: string) {
    client.to(roomId).emit('message', {
      text: 'Room members except sender',
    });
  }
  // Note: client.to(room) automatically excludes sender
  // client.broadcast.to(room) is redundant but clearer
}
```

### **Complete Chat Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  OnGatewayDisconnect,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({ namespace: '/chat' })
export class CompleteChatGateway
  implements OnGatewayConnection, OnGatewayDisconnect
{
  @WebSocketServer()
  server: Server;

  private connectedUsers = new Map<string, any>();

  constructor(private chatService: ChatService) {}

  // ================================================
  // USER JOINS
  // ================================================
  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;

    // Store user info
    this.connectedUsers.set(client.id, {
      userId,
      username,
      connectedAt: new Date(),
    });

    // BROADCAST: Tell others about new user
    client.broadcast.emit('user-online', {
      userId,
      username,
      message: `${username} joined`,
    });

    // DIRECT: Send welcome to new user
    client.emit('connected', {
      message: `Welcome, ${username}!`,
      onlineUsers: Array.from(this.connectedUsers.values()),
    });
  }

  // ================================================
  // USER LEAVES
  // ================================================
  handleDisconnect(client: Socket) {
    const user = this.connectedUsers.get(client.id);
    
    if (user) {
      // BROADCAST: Tell others user left
      client.broadcast.emit('user-offline', {
        userId: user.userId,
        username: user.username,
        message: `${user.username} left`,
      });

      // Cleanup
      this.connectedUsers.delete(client.id);
    }
  }

  // ================================================
  // SEND MESSAGE
  // ================================================
  @SubscribeMessage('send-message')
  async handleSendMessage(
    @MessageBody() data: { message: string; roomId: string },
    @ConnectedSocket() client: Socket,
  ) {
    const user = this.connectedUsers.get(client.id);

    // Save message
    const message = await this.chatService.saveMessage({
      userId: user.userId,
      content: data.message,
      roomId: data.roomId,
    });

    const messageData = {
      id: message.id,
      userId: user.userId,
      username: user.username,
      content: data.message,
      roomId: data.roomId,
      timestamp: message.createdAt,
    };

    // BROADCAST: Send to room except sender
    client.broadcast.to(data.roomId).emit('new-message', messageData);

    // DIRECT: Confirm to sender
    client.emit('message-sent', {
      messageId: message.id,
      success: true,
    });
  }

  // ================================================
  // JOIN ROOM
  // ================================================
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const user = this.connectedUsers.get(client.id);

    // Join room
    client.join(roomId);

    // BROADCAST: Tell room members (except joiner)
    client.broadcast.to(roomId).emit('user-joined-room', {
      userId: user.userId,
      username: user.username,
      roomId,
      message: `${user.username} joined the room`,
    });

    // DIRECT: Confirm to joiner
    client.emit('joined-room', {
      roomId,
      message: `You joined ${roomId}`,
    });
  }

  // ================================================
  // TYPING INDICATOR
  // ================================================
  @SubscribeMessage('typing')
  handleTyping(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const user = this.connectedUsers.get(client.id);

    // BROADCAST: Tell others (not sender)
    client.broadcast.to(roomId).emit('user-typing', {
      userId: user.userId,
      username: user.username,
      roomId,
    });
    // Sender doesn't need to see own typing
  }

  @SubscribeMessage('stop-typing')
  handleStopTyping(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const user = this.connectedUsers.get(client.id);

    // BROADCAST: Tell others
    client.broadcast.to(roomId).emit('user-stop-typing', {
      userId: user.userId,
      roomId,
    });
  }
}
```

### **Broadcast to Multiple Rooms**

```typescript
@WebSocketGateway()
export class MultiRoomBroadcastGateway {
  @SubscribeMessage('update-teams')
  handleUpdateTeams(
    @MessageBody() teams: string[],
    @ConnectedSocket() client: Socket,
  ) {
    // Broadcast to multiple rooms except sender
    client.broadcast.to(teams).emit('teams-updated', {
      teams,
      timestamp: new Date(),
    });
    // Or chain:
    // client.broadcast.to('team1').to('team2').emit(...);
  }
}
```

### **Broadcast with Volatile (Skip Slow Clients)**

```typescript
@WebSocketGateway()
export class VolatileBroadcastGateway {
  @SubscribeMessage('mouse-move')
  handleMouseMove(
    @MessageBody() position: { x: number; y: number },
    @ConnectedSocket() client: Socket,
  ) {
    // Broadcast cursor position (non-critical)
    // Skip if client's buffer is full
    client.broadcast.volatile.emit('cursor-move', {
      userId: client.id,
      position,
    });
    // If client is slow/busy, this event is dropped
  }
}
```

### **Broadcast vs server.emit()**

```typescript
@WebSocketGateway()
export class ComparisonGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('test')
  handleTest(@ConnectedSocket() client: Socket) {
    // OPTION 1: client.broadcast.emit()
    // Sends to: All EXCEPT sender
    client.broadcast.emit('message', { text: 'Everyone else' });

    // OPTION 2: server.emit()
    // Sends to: EVERYONE including sender
    this.server.emit('message', { text: 'Everyone' });

    // OPTION 3: server.except(socketId).emit()
    // Same as client.broadcast.emit()
    this.server.except(client.id).emit('message', { text: 'Everyone else' });
  }
}
```

### **Broadcast Pattern for Real-Time Collaboration**

```typescript
@WebSocketGateway({ namespace: '/collab' })
export class CollabGateway {
  @SubscribeMessage('document-edit')
  handleDocumentEdit(
    @MessageBody() data: {
      documentId: string;
      changes: any;
      cursorPosition: number;
    },
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;

    // Broadcast changes to other collaborators
    client.broadcast.to(`doc:${data.documentId}`).emit('remote-changes', {
      userId,
      changes: data.changes,
      cursorPosition: data.cursorPosition,
    });
    // Editor applies changes locally (optimistic)
    // Others receive broadcast to sync
  }

  @SubscribeMessage('cursor-move')
  handleCursorMove(
    @MessageBody() data: { documentId: string; position: number },
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;

    // Broadcast cursor position (volatile - can drop)
    client.broadcast
      .to(`doc:${data.documentId}`)
      .volatile.emit('remote-cursor', {
        userId,
        position: data.position,
      });
  }
}
```

### **Common Patterns Summary**

```typescript
@WebSocketGateway()
export class PatternsSummary {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('action')
  handleAction(@ConnectedSocket() client: Socket) {
    // PATTERN 1: User action (tell others, confirm to sender)
    client.broadcast.emit('user-action', { /* ... */ });
    client.emit('action-confirmed', { /* ... */ });

    // PATTERN 2: Room action (tell room except sender)
    client.broadcast.to('roomId').emit('room-update', { /* ... */ });
    client.emit('update-confirmed', { /* ... */ });

    // PATTERN 3: Global announcement (tell everyone)
    this.server.emit('announcement', { /* ... */ });

    // PATTERN 4: Tell everyone except specific socket
    this.server.except(client.id).emit('update', { /* ... */ });
  }
}
```

**Interview Tip**: Broadcast to **all except sender** using **`client.broadcast.emit(event, data)`** - sender doesn't receive, all others do. **Common use cases**: (1) user joins (others see "John joined", John sees "Welcome"), (2) typing indicator (others see "John is typing", John doesn't), (3) chat messages with optimistic update (sender already has it, broadcast to others), (4) user leaves (notify others). **Room broadcast**: `client.broadcast.to(room).emit()` OR `client.to(room).emit()` (equivalent) - room members except sender receive. **Alternative**: `server.except(socketId).emit()` same as `client.broadcast.emit()`. **Pattern**: broadcast update to others, emit confirmation to sender separately. **Multiple rooms**: `client.broadcast.to([room1, room2])` or chain `to()`. **Volatile**: add `.volatile` for non-critical broadcasts (dropped if client busy). **Key concept**: sender handles action locally (optimistic update), broadcast synchronizes others.

</details>

## Rooms & Namespaces

19. What are Rooms in Socket.IO?

<details>
<summary><strong>Answer</strong></summary>

**Rooms** in Socket.IO are **arbitrary channels** that sockets can **join** and **leave**. They provide a way to **broadcast events to a subset of clients** without sending to everyone. Rooms are server-side only (clients don't know which rooms they're in) and are perfect for chat rooms, game lobbies, user-specific channels, and any scenario requiring targeted broadcasting.

### **Core Concept**

```typescript
/**
 * ROOMS = Groups of sockets
 * 
 * Key characteristics:
 * 1. Server-side only (clients can't see room membership)
 * 2. Socket can join multiple rooms
 * 3. Rooms are created automatically when first socket joins
 * 4. Rooms are deleted automatically when last socket leaves
 * 5. Perfect for broadcasting to specific groups
 */

// Example: Chat application
// Room "room-1" might have sockets: [socket-abc, socket-def, socket-ghi]
// Room "room-2" might have sockets: [socket-def, socket-xyz]
// Notice: socket-def is in BOTH rooms

// Emit to room-1:
// server.to('room-1').emit('message', data);
// Only socket-abc, socket-def, socket-ghi receive it
```

### **Basic Room Operations**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway()
export class RoomsBasicsGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('test-rooms')
  handleTestRooms(@ConnectedSocket() client: Socket) {
    // 1. JOIN A ROOM
    client.join('room-1');
    console.log(`${client.id} joined room-1`);

    // 2. JOIN MULTIPLE ROOMS
    client.join('room-2');
    client.join('room-3');
    // Socket is now in: room-1, room-2, room-3

    // 3. EMIT TO ROOM (from server)
    this.server.to('room-1').emit('message', {
      text: 'Message for room-1',
    });
    // All sockets in room-1 receive it

    // 4. EMIT TO ROOM (from client)
    client.to('room-1').emit('message', {
      text: 'Message for room-1 except me',
    });
    // All sockets in room-1 EXCEPT sender receive it

    // 5. GET ROOMS A SOCKET IS IN
    const rooms = Array.from(client.rooms);
    console.log(`Socket ${client.id} is in rooms:`, rooms);
    // Output: ["socket-id", "room-1", "room-2", "room-3"]
    // Note: Socket ID is also a room (personal room)

    // 6. LEAVE A ROOM
    client.leave('room-2');
    console.log(`${client.id} left room-2`);

    // 7. GET ALL ROOMS (server-wide)
    const allRooms = Array.from(this.server.sockets.adapter.rooms.keys());
    console.log('All rooms:', allRooms);

    // 8. GET SOCKETS IN A ROOM
    const room1Sockets = this.server.sockets.adapter.rooms.get('room-1');
    console.log(`room-1 has ${room1Sockets?.size} sockets`);
  }
}
```

### **Common Use Cases**

#### **1. Chat Rooms**

```typescript
@WebSocketGateway({ namespace: '/chat' })
export class ChatRoomsGateway {
  @WebSocketServer()
  server: Server;

  // Join chat room
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Join the room
    client.join(roomId);

    // Notify others in room
    client.to(roomId).emit('user-joined', {
      userId: client.handshake.auth.userId,
      username: client.handshake.auth.username,
      roomId,
    });

    // Confirm to joiner
    client.emit('joined-room', {
      roomId,
      message: `You joined ${roomId}`,
    });
  }

  // Send message to room
  @SubscribeMessage('room-message')
  handleRoomMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    // Broadcast to room (including sender)
    this.server.to(data.roomId).emit('new-message', {
      userId: client.handshake.auth.userId,
      message: data.message,
      roomId: data.roomId,
      timestamp: new Date(),
    });
  }

  // Leave room
  @SubscribeMessage('leave-room')
  handleLeaveRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Notify others BEFORE leaving
    client.to(roomId).emit('user-left', {
      userId: client.handshake.auth.userId,
      roomId,
    });

    // Leave the room
    client.leave(roomId);

    // Confirm to leaver
    client.emit('left-room', { roomId });
  }
}
```

#### **2. Personal Rooms (User Targeting)**

```typescript
@WebSocketGateway()
export class PersonalRoomsGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  // On connection: join personal room
  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;

    // Join personal room (user:userId)
    client.join(`user:${userId}`);
    console.log(`User ${userId} joined personal room`);

    // Now can target this user from anywhere:
    // server.to(`user:${userId}`).emit('notification', data);
  }

  // Send notification to specific user
  async notifyUser(userId: string, notification: any) {
    this.server.to(`user:${userId}`).emit('notification', {
      ...notification,
      timestamp: new Date(),
    });
  }

  // Send to multiple users
  async notifyUsers(userIds: string[], notification: any) {
    userIds.forEach(userId => {
      this.server.to(`user:${userId}`).emit('notification', notification);
    });
  }
}
```

#### **3. Game Lobbies**

```typescript
@WebSocketGateway({ namespace: '/game' })
export class GameLobbyGateway {
  @WebSocketServer()
  server: Server;

  private lobbies = new Map<string, GameLobby>();

  // Create lobby
  @SubscribeMessage('create-lobby')
  handleCreateLobby(
    @MessageBody() settings: any,
    @ConnectedSocket() client: Socket,
  ) {
    const lobbyId = generateId();

    // Create lobby data
    this.lobbies.set(lobbyId, {
      id: lobbyId,
      host: client.id,
      players: [client.id],
      settings,
    });

    // Join lobby room
    client.join(`lobby:${lobbyId}`);

    return { lobbyId, message: 'Lobby created' };
  }

  // Join lobby
  @SubscribeMessage('join-lobby')
  handleJoinLobby(
    @MessageBody() lobbyId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const lobby = this.lobbies.get(lobbyId);

    if (!lobby) {
      return { error: 'Lobby not found' };
    }

    // Add player
    lobby.players.push(client.id);

    // Join room
    client.join(`lobby:${lobbyId}`);

    // Notify all players in lobby
    this.server.to(`lobby:${lobbyId}`).emit('player-joined', {
      playerId: client.id,
      playerCount: lobby.players.length,
    });

    return { success: true, lobby };
  }

  // Start game
  @SubscribeMessage('start-game')
  handleStartGame(
    @MessageBody() lobbyId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const lobby = this.lobbies.get(lobbyId);

    if (lobby?.host !== client.id) {
      return { error: 'Only host can start game' };
    }

    // Notify all players in lobby
    this.server.to(`lobby:${lobbyId}`).emit('game-started', {
      lobbyId,
      timestamp: new Date(),
    });
  }
}

interface GameLobby {
  id: string;
  host: string;
  players: string[];
  settings: any;
}
```

#### **4. Team Channels**

```typescript
@WebSocketGateway()
export class TeamRoomsGateway implements OnGatewayConnection {
  constructor(private teamsService: TeamsService) {}

  // On connection: join all user's team rooms
  async handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;

    // Get user's teams from database
    const teams = await this.teamsService.getUserTeams(userId);

    // Join all team rooms
    teams.forEach(team => {
      client.join(`team:${team.id}`);
      console.log(`User ${userId} joined team:${team.id}`);
    });
  }

  // Send update to entire team
  async notifyTeam(teamId: string, update: any) {
    this.server.to(`team:${teamId}`).emit('team-update', {
      teamId,
      ...update,
      timestamp: new Date(),
    });
  }
}
```

### **Room Naming Conventions**

```typescript
@WebSocketGateway()
export class RoomNamingGateway {
  @WebSocketServer()
  server: Server;

  setupRooms(client: Socket) {
    // PATTERN 1: Entity-based rooms
    client.join('user:123');           // Personal user room
    client.join('team:456');           // Team room
    client.join('project:789');        // Project room
    client.join('document:abc');       // Document room

    // PATTERN 2: Feature-based rooms
    client.join('chat:general');       // General chat
    client.join('chat:support');       // Support chat
    client.join('notifications');      // Notifications channel
    client.join('announcements');      // Announcements channel

    // PATTERN 3: Hierarchical rooms
    client.join('org:company1');       // Organization
    client.join('org:company1:team:1'); // Team within org
    client.join('org:company1:project:5'); // Project within org

    // PATTERN 4: Temporary rooms
    client.join('lobby:game123');      // Game lobby
    client.join('session:xyz');        // Temporary session

    // PATTERN 5: Broadcast rooms
    client.join('broadcast:all');      // Everyone
    client.join('broadcast:premium');  // Premium users
  }
}
```

### **Advanced Room Operations**

```typescript
@WebSocketGateway()
export class AdvancedRoomsGateway {
  @WebSocketServer()
  server: Server;

  // Get all sockets in a room (async)
  async getSocketsInRoom(roomName: string) {
    const sockets = await this.server.in(roomName).fetchSockets();
    return sockets.map(s => ({
      id: s.id,
      data: s.data,
      rooms: Array.from(s.rooms),
    }));
  }

  // Count sockets in room
  async getRoomSize(roomName: string): Promise<number> {
    const sockets = await this.server.in(roomName).fetchSockets();
    return sockets.length;
  }

  // Check if room exists
  roomExists(roomName: string): boolean {
    const room = this.server.sockets.adapter.rooms.get(roomName);
    return room !== undefined && room.size > 0;
  }

  // Get all rooms
  getAllRooms(): string[] {
    const rooms = Array.from(this.server.sockets.adapter.rooms.keys());
    // Filter out socket IDs (which are also stored as rooms)
    return rooms.filter(room => !room.startsWith('socket-'));
  }

  // Disconnect all sockets in room
  async disconnectRoom(roomName: string) {
    const sockets = await this.server.in(roomName).fetchSockets();
    sockets.forEach(socket => socket.disconnect());
  }

  // Make all sockets in one room join another
  async moveRoom(fromRoom: string, toRoom: string) {
    const sockets = await this.server.in(fromRoom).fetchSockets();
    sockets.forEach(socket => {
      socket.leave(fromRoom);
      socket.join(toRoom);
    });
  }
}
```

### **Room Cleanup**

```typescript
@WebSocketGateway()
export class RoomCleanupGateway implements OnGatewayDisconnect {
  handleDisconnect(client: Socket) {
    // Socket.IO automatically removes socket from all rooms
    // But you might want to notify rooms

    const rooms = Array.from(client.rooms);
    
    rooms.forEach(room => {
      // Skip personal room (socket ID)
      if (room !== client.id) {
        // Notify room that user left
        client.to(room).emit('user-left-room', {
          socketId: client.id,
          room,
        });
      }
    });
  }
}
```

### **Room Broadcasting Patterns**

```typescript
@WebSocketGateway()
export class BroadcastPatternsGateway {
  @WebSocketServer()
  server: Server;

  // PATTERN 1: Broadcast to single room
  pattern1() {
    this.server.to('room1').emit('update', { data: 'Room 1 update' });
  }

  // PATTERN 2: Broadcast to multiple rooms
  pattern2() {
    this.server.to(['room1', 'room2', 'room3']).emit('update', {
      data: 'Multi-room update',
    });
    // Or chain: this.server.to('room1').to('room2').to('room3').emit()
  }

  // PATTERN 3: Broadcast to room except sender
  pattern3(client: Socket, roomId: string) {
    client.to(roomId).emit('update', { data: 'Update except sender' });
  }

  // PATTERN 4: Broadcast to all rooms except one
  pattern4(exceptRoom: string) {
    this.server.except(exceptRoom).emit('update', {
      data: 'All rooms except one',
    });
  }

  // PATTERN 5: Broadcast to room with volatile (drop if busy)
  pattern5(roomId: string) {
    this.server.to(roomId).volatile.emit('update', {
      data: 'Non-critical update',
    });
  }
}
```

### **Real-World Example: Multi-Feature App**

```typescript
@WebSocketGateway()
export class MultiFeatureGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  constructor(
    private teamsService: TeamsService,
    private projectsService: ProjectsService,
  ) {}

  async handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;

    // 1. PERSONAL ROOM (for notifications)
    client.join(`user:${userId}`);

    // 2. TEAM ROOMS (for team updates)
    const teams = await this.teamsService.getUserTeams(userId);
    teams.forEach(team => client.join(`team:${team.id}`));

    // 3. PROJECT ROOMS (for project updates)
    const projects = await this.projectsService.getUserProjects(userId);
    projects.forEach(project => client.join(`project:${project.id}`));

    // 4. BROADCAST ROOM (for announcements)
    client.join('broadcast:all');

    // 5. ROLE-BASED ROOMS (for role-specific updates)
    const userRole = client.handshake.auth.role;
    client.join(`role:${userRole}`);

    console.log(
      `User ${userId} joined ${client.rooms.size} rooms`,
    );
  }

  // Now can target users by:
  // - Personal: server.to(`user:${userId}`)
  // - Team: server.to(`team:${teamId}`)
  // - Project: server.to(`project:${projectId}`)
  // - Everyone: server.to('broadcast:all')
  // - Role: server.to(`role:${role}`)
}
```

**Interview Tip**: **Rooms** are **server-side channels** that sockets can **join/leave**. **Purpose**: broadcast to **subset of clients** without sending to everyone. **Characteristics**: (1) server-side only (clients unaware), (2) socket can be in **multiple rooms**, (3) rooms **auto-created** on first join, **auto-deleted** when empty, (4) socket ID itself is a room (personal room). **Common use cases**: chat rooms, personal user rooms (`user:userId`), team channels (`team:teamId`), game lobbies. **Operations**: `client.join(room)` to join, `client.leave(room)` to leave, `server.to(room).emit()` to broadcast. **Get rooms**: `Array.from(client.rooms)` for socket's rooms, `server.sockets.adapter.rooms` for all rooms. **Naming convention**: use prefixes (`user:`, `team:`, `project:`) for organization. **On disconnect**: Socket.IO auto-removes from all rooms. **Production pattern**: join personal room on connection (`user:userId`), enables targeting users from anywhere in app. **Multiple rooms**: `server.to([room1, room2]).emit()` broadcasts to multiple.

</details>
20. How do you join a room using `socket.join()`?

<details>
<summary><strong>Answer</strong></summary>

To join a room, call **`client.join(roomName)`** where `client` is the Socket instance. The method is **synchronous** (Socket.IO v4+), immediately adds the socket to the room, and returns **void** (or Promise in some versions). You can join multiple rooms, and rooms are created automatically if they don't exist.

### **Basic Syntax**

```typescript
import {
  WebSocketGateway,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway()
export class JoinRoomGateway {
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @MessageBody() roomName: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Join the room (synchronous in Socket.IO v4+)
    client.join(roomName);
    
    console.log(`Socket ${client.id} joined room ${roomName}`);
    
    // Confirm to client
    client.emit('joined-room', {
      room: roomName,
      message: `You joined ${roomName}`,
    });

    // Notify others in room
    client.to(roomName).emit('user-joined', {
      socketId: client.id,
      room: roomName,
    });
  }
}
```

### **Joining on Connection**

```typescript
import { OnGatewayConnection } from '@nestjs/websockets';

@WebSocketGateway()
export class AutoJoinGateway implements OnGatewayConnection {
  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;

    // AUTO-JOIN: Personal room
    client.join(`user:${userId}`);
    console.log(`${username} joined personal room`);

    // AUTO-JOIN: General room
    client.join('general');
    console.log(`${username} joined general room`);

    // AUTO-JOIN: Broadcast room
    client.join('broadcast:all');

    // Send welcome message
    client.emit('welcome', {
      message: `Welcome, ${username}!`,
      rooms: Array.from(client.rooms),
    });
  }
}
```

### **Joining Multiple Rooms**

```typescript
@WebSocketGateway()
export class MultipleRoomsGateway {
  @SubscribeMessage('join-multiple-rooms')
  handleJoinMultiple(
    @MessageBody() rooms: string[],
    @ConnectedSocket() client: Socket,
  ) {
    // METHOD 1: Loop through rooms
    rooms.forEach(room => {
      client.join(room);
      console.log(`Joined ${room}`);
    });

    // Confirm
    client.emit('joined-rooms', {
      rooms,
      message: `Joined ${rooms.length} rooms`,
    });
  }

  // Join user's teams
  async joinUserTeams(
    client: Socket,
    userId: string,
  ) {
    const teams = await this.getTeamsForUser(userId);
    
    teams.forEach(team => {
      client.join(`team:${team.id}`);
    });

    console.log(`User ${userId} joined ${teams.length} team rooms`);
  }

  private async getTeamsForUser(userId: string) {
    // Database query
    return [{ id: '1' }, { id: '2' }];
  }
}
```

### **Conditional Joining**

```typescript
@WebSocketGateway()
export class ConditionalJoinGateway {
  constructor(private roomsService: RoomsService) {}

  @SubscribeMessage('join-room')
  async handleJoinRoom(
    @MessageBody() data: { roomId: string; password?: string },
    @ConnectedSocket() client: Socket,
  ) {
    // VALIDATION 1: Check if room exists
    const room = await this.roomsService.findRoom(data.roomId);
    
    if (!room) {
      client.emit('join-error', { message: 'Room not found' });
      return;
    }

    // VALIDATION 2: Check room capacity
    const currentSize = await this.getRoomSize(data.roomId);
    
    if (currentSize >= room.maxCapacity) {
      client.emit('join-error', { message: 'Room is full' });
      return;
    }

    // VALIDATION 3: Check password (if private)
    if (room.isPrivate) {
      if (!data.password || data.password !== room.password) {
        client.emit('join-error', { message: 'Invalid password' });
        return;
      }
    }

    // VALIDATION 4: Check user permissions
    const userId = client.handshake.auth.userId;
    const hasPermission = await this.roomsService.checkPermission(
      userId,
      data.roomId,
    );
    
    if (!hasPermission) {
      client.emit('join-error', { message: 'No permission' });
      return;
    }

    // ALL VALIDATIONS PASSED: Join the room
    client.join(data.roomId);

    // Notify room members
    client.to(data.roomId).emit('user-joined', {
      userId,
      roomId: data.roomId,
    });

    // Confirm to joiner
    client.emit('joined-room', {
      roomId: data.roomId,
      roomName: room.name,
      memberCount: currentSize + 1,
    });
  }

  private async getRoomSize(roomId: string): Promise<number> {
    const sockets = await this.server.in(roomId).fetchSockets();
    return sockets.length;
  }
}
```

### **Join with Notification**

```typescript
@WebSocketGateway({ namespace: '/chat' })
export class NotifyingJoinGateway {
  @WebSocketServer()
  server: Server;

  constructor(private chatService: ChatService) {}

  @SubscribeMessage('join-room')
  async handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;

    // Save to database
    await this.chatService.addUserToRoom(userId, roomId);

    // Join the room
    client.join(roomId);

    // NOTIFY: Tell others user joined
    client.to(roomId).emit('user-joined-room', {
      userId,
      username,
      roomId,
      message: `${username} joined the room`,
      timestamp: new Date(),
    });

    // CONFIRM: Tell joiner
    client.emit('joined-room', {
      roomId,
      message: `You joined ${roomId}`,
    });

    // SEND: Room history to new member
    const messages = await this.chatService.getRecentMessages(roomId, 50);
    client.emit('room-history', {
      roomId,
      messages,
    });

    // SEND: Current room members
    const members = await this.getRoomMembers(roomId);
    client.emit('room-members', {
      roomId,
      members,
      count: members.length,
    });
  }

  private async getRoomMembers(roomId: string) {
    const sockets = await this.server.in(roomId).fetchSockets();
    return sockets.map(s => ({
      socketId: s.id,
      userId: s.handshake.auth.userId,
      username: s.handshake.auth.username,
    }));
  }
}
```

### **Join and Track**

```typescript
@WebSocketGateway()
export class TrackingJoinGateway {
  @WebSocketServer()
  server: Server;

  // Track which users are in which rooms
  private userRooms = new Map<string, Set<string>>();

  @SubscribeMessage('join-room')
  handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;

    // Join room
    client.join(roomId);

    // Track in local state
    if (!this.userRooms.has(userId)) {
      this.userRooms.set(userId, new Set());
    }
    this.userRooms.get(userId)!.add(roomId);

    console.log(
      `User ${userId} joined room ${roomId}. ` +
      `Total rooms: ${this.userRooms.get(userId)!.size}`,
    );

    return { success: true, roomId };
  }

  // Get all rooms a user is in
  getUserRooms(userId: string): string[] {
    const rooms = this.userRooms.get(userId);
    return rooms ? Array.from(rooms) : [];
  }

  // Cleanup on disconnect
  handleDisconnect(client: Socket) {
    const userId = client.handshake.auth.userId;
    this.userRooms.delete(userId);
  }
}
```

### **Join from Service**

```typescript
// room.service.ts
import { Injectable } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';

@Injectable()
export class RoomService {
  constructor(private chatGateway: ChatGateway) {}

  async assignUserToRoom(userId: string, roomId: string) {
    // Get user's socket(s)
    const sockets = await this.chatGateway.server
      .in(`user:${userId}`)
      .fetchSockets();

    // Make all user's sockets join the room
    sockets.forEach(socket => {
      socket.join(roomId);
      console.log(`Socket ${socket.id} joined ${roomId}`);
    });

    // Notify user
    this.chatGateway.server.to(`user:${userId}`).emit('assigned-to-room', {
      roomId,
      message: 'You have been assigned to a room',
    });
  }
}

// Gateway must be injectable
@Injectable()
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server; // Public for service access
}
```

### **Join with Role-Based Access**

```typescript
@WebSocketGateway()
export class RoleBasedJoinGateway {
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userRole = client.handshake.auth.role;

    // Check role permissions
    const allowedRoles = this.getRoomAllowedRoles(roomId);
    
    if (!allowedRoles.includes(userRole)) {
      client.emit('join-error', {
        message: 'Insufficient permissions',
        requiredRoles: allowedRoles,
      });
      return;
    }

    // Permission granted, join room
    client.join(roomId);

    client.emit('joined-room', { roomId, role: userRole });
  }

  private getRoomAllowedRoles(roomId: string): string[] {
    // Room configuration
    const roomRoles = {
      'admin-room': ['admin'],
      'moderator-room': ['admin', 'moderator'],
      'general': ['admin', 'moderator', 'user'],
    };

    return roomRoles[roomId] || [];
  }
}
```

### **Automatic Team/Project Rooms**

```typescript
@WebSocketGateway()
export class AutoAssignGateway implements OnGatewayConnection {
  constructor(
    private teamsService: TeamsService,
    private projectsService: ProjectsService,
  ) {}

  async handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;

    // 1. Join personal room
    client.join(`user:${userId}`);

    // 2. Join all team rooms
    const teams = await this.teamsService.getUserTeams(userId);
    teams.forEach(team => {
      client.join(`team:${team.id}`);
      console.log(`Auto-joined team:${team.id}`);
    });

    // 3. Join all project rooms
    const projects = await this.projectsService.getUserProjects(userId);
    projects.forEach(project => {
      client.join(`project:${project.id}`);
      console.log(`Auto-joined project:${project.id}`);
    });

    // 4. Join role room
    const role = client.handshake.auth.role;
    client.join(`role:${role}`);

    // 5. Join broadcast room
    client.join('broadcast:all');

    // Log total rooms
    console.log(
      `User ${userId} auto-joined ${client.rooms.size} rooms`,
    );

    // Send list of rooms to client
    client.emit('rooms-joined', {
      rooms: Array.from(client.rooms),
      count: client.rooms.size,
    });
  }
}
```

### **Verify Join Success**

```typescript
@WebSocketGateway()
export class VerifyJoinGateway {
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Join room
    client.join(roomId);

    // VERIFY: Check if socket is now in room
    const isInRoom = client.rooms.has(roomId);
    
    if (isInRoom) {
      console.log(`✓ Successfully joined ${roomId}`);
      
      client.emit('joined-room', {
        roomId,
        success: true,
        rooms: Array.from(client.rooms),
      });
    } else {
      console.error(`✗ Failed to join ${roomId}`);
      
      client.emit('join-error', {
        roomId,
        message: 'Failed to join room',
      });
    }
  }
}
```

### **Production Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway({ namespace: '/chat' })
export class ProductionJoinGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  private logger = new Logger('ProductionJoinGateway');

  constructor(
    private chatService: ChatService,
    private roomsService: RoomsService,
  ) {}

  // Auto-join on connection
  async handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;

    try {
      // Join personal room
      client.join(`user:${userId}`);
      this.logger.log(`User ${userId} joined personal room`);

      // Join previously joined rooms from DB
      const userRooms = await this.roomsService.getUserRooms(userId);
      userRooms.forEach(room => {
        client.join(room.id);
        this.logger.log(`User ${userId} rejoined ${room.id}`);
      });
    } catch (error) {
      this.logger.error('Error during auto-join:', error);
    }
  }

  // Manual join with full validation
  @SubscribeMessage('join-room')
  async handleJoinRoom(
    @MessageBody() data: { roomId: string; password?: string },
    @ConnectedSocket() client: Socket,
  ) {
    const startTime = Date.now();
    const userId = client.handshake.auth.userId;

    try {
      // Validate room
      const room = await this.roomsService.findRoom(data.roomId);
      if (!room) {
        return { success: false, error: 'Room not found' };
      }

      // Check capacity
      const currentSize = await this.getRoomSize(data.roomId);
      if (currentSize >= room.maxCapacity) {
        return { success: false, error: 'Room is full' };
      }

      // Validate password for private rooms
      if (room.isPrivate && data.password !== room.password) {
        return { success: false, error: 'Invalid password' };
      }

      // Join room
      client.join(data.roomId);

      // Save to database
      await this.roomsService.addUserToRoom(userId, data.roomId);

      // Notify others
      client.to(data.roomId).emit('user-joined-room', {
        userId,
        username: client.handshake.auth.username,
        roomId: data.roomId,
      });

      // Send history to new member
      const messages = await this.chatService.getRecentMessages(
        data.roomId,
        50,
      );

      client.emit('room-history', {
        roomId: data.roomId,
        messages,
      });

      // Log success
      const duration = Date.now() - startTime;
      this.logger.log(
        `User ${userId} joined ${data.roomId} in ${duration}ms`,
      );

      return {
        success: true,
        roomId: data.roomId,
        memberCount: currentSize + 1,
      };
    } catch (error) {
      this.logger.error('Error joining room:', error);
      return {
        success: false,
        error: 'Failed to join room',
      };
    }
  }

  private async getRoomSize(roomId: string): Promise<number> {
    const sockets = await this.server.in(roomId).fetchSockets();
    return sockets.length;
  }
}
```

**Interview Tip**: Join room with **`client.join(roomName)`** - **synchronous** in Socket.IO v4+, **immediately** adds socket to room, **returns void**. **Common patterns**: (1) join on connection (`handleConnection`) for personal room (`user:userId`), (2) join on request (`@SubscribeMessage('join-room')`) for chat rooms, (3) auto-join multiple rooms (teams, projects) from database. **Validation**: check room exists, check capacity, verify password (if private), check permissions before joining. **After joining**: notify room members (`client.to(room).emit('user-joined')`), send history to new member, confirm to joiner. **Verify success**: check `client.rooms.has(roomName)`. **Multiple rooms**: loop through array, call `join()` for each. **Tracking**: use Map to track user-room relationships, cleanup on disconnect. **From service**: fetch sockets with `server.in('user:userId').fetchSockets()`, call `socket.join(room)` on each. **Room created automatically** if doesn't exist.

</details>
21. How do you leave a room using `socket.leave()`?

<details>
<summary><strong>Answer</strong></summary>

To leave a room, call **`client.leave(roomName)`** where `client` is the Socket instance. The method is **synchronous** (Socket.IO v4+), immediately removes the socket from the room, and returns **void**. When a socket disconnects, Socket.IO **automatically** removes it from all rooms.

### **Basic Syntax**

```typescript
import {
  WebSocketGateway,
  SubscribeMessage,
  OnGatewayDisconnect,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway()
export class LeaveRoomGateway {
  @SubscribeMessage('leave-room')
  handleLeaveRoom(
    @MessageBody() roomName: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Notify others BEFORE leaving
    client.to(roomName).emit('user-leaving', {
      socketId: client.id,
      room: roomName,
      message: 'User is leaving',
    });

    // Leave the room (synchronous in Socket.IO v4+)
    client.leave(roomName);
    
    console.log(`Socket ${client.id} left room ${roomName}`);
    
    // Confirm to client
    client.emit('left-room', {
      room: roomName,
      message: `You left ${roomName}`,
    });
  }
}
```

### **Leave Multiple Rooms**

```typescript
@WebSocketGateway()
export class LeaveMultipleGateway {
  @SubscribeMessage('leave-all-rooms')
  handleLeaveAll(@ConnectedSocket() client: Socket) {
    // Get all rooms (excluding personal room which is socket ID)
    const rooms = Array.from(client.rooms).filter(
      room => room !== client.id,
    );

    // Leave each room
    rooms.forEach(room => {
      // Notify room members
      client.to(room).emit('user-left', {
        socketId: client.id,
        room,
      });

      // Leave room
      client.leave(room);
      console.log(`Left room: ${room}`);
    });

    // Confirm
    client.emit('left-all-rooms', {
      count: rooms.length,
      rooms,
    });
  }

  @SubscribeMessage('leave-rooms')
  handleLeaveRooms(
    @MessageBody() roomsToLeave: string[],
    @ConnectedSocket() client: Socket,
  ) {
    roomsToLeave.forEach(room => {
      // Check if socket is in room before leaving
      if (client.rooms.has(room)) {
        client.to(room).emit('user-left', {
          socketId: client.id,
          room,
        });
        client.leave(room);
      }
    });

    return { success: true, leftRooms: roomsToLeave };
  }
}
```

### **Automatic Leave on Disconnect**

```typescript
@WebSocketGateway({ namespace: '/chat' })
export class AutoLeaveGateway implements OnGatewayDisconnect {
  handleDisconnect(client: Socket) {
    // Socket.IO automatically removes socket from all rooms
    // But we can notify rooms before that happens

    const rooms = Array.from(client.rooms).filter(
      room => room !== client.id,
    );

    rooms.forEach(room => {
      // Notify room members
      client.to(room).emit('user-disconnected', {
        socketId: client.id,
        userId: client.handshake.auth.userId,
        room,
        timestamp: new Date(),
      });
    });

    console.log(
      `Socket ${client.id} disconnected, ` +
      `auto-left ${rooms.length} rooms`,
    );
  }
}
```

### **Conditional Leave**

```typescript
@WebSocketGateway()
export class ConditionalLeaveGateway {
  constructor(private roomsService: RoomsService) {}

  @SubscribeMessage('leave-room')
  async handleLeaveRoom(
    @MessageBody() data: { roomId: string; reason?: string },
    @ConnectedSocket() client: Socket,
  ) {
    // CHECK 1: Is socket in room?
    if (!client.rooms.has(data.roomId)) {
      return {
        success: false,
        error: 'Not in room',
      };
    }

    // CHECK 2: Can user leave? (some rooms might be mandatory)
    const room = await this.roomsService.findRoom(data.roomId);
    if (room?.isMandatory) {
      return {
        success: false,
        error: 'Cannot leave mandatory room',
      };
    }

    const userId = client.handshake.auth.userId;

    // Notify others
    client.to(data.roomId).emit('user-left-room', {
      userId,
      roomId: data.roomId,
      reason: data.reason,
    });

    // Leave room
    client.leave(data.roomId);

    // Update database
    await this.roomsService.removeUserFromRoom(userId, data.roomId);

    return {
      success: true,
      roomId: data.roomId,
      message: 'Successfully left room',
    };
  }
}
```

### **Chat Application Leave**

```typescript
@WebSocketGateway({ namespace: '/chat' })
export class ChatLeaveGateway {
  @WebSocketServer()
  server: Server;

  constructor(private chatService: ChatService) {}

  @SubscribeMessage('leave-room')
  async handleLeaveRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;

    // Verify membership
    if (!client.rooms.has(roomId)) {
      return { error: 'Not a member of this room' };
    }

    // Get room size before leaving
    const roomSizeBefore = await this.getRoomSize(roomId);

    // Broadcast leave message to room
    client.to(roomId).emit('user-left-room', {
      userId,
      username,
      roomId,
      message: `${username} left the room`,
      timestamp: new Date(),
    });

    // Leave room
    client.leave(roomId);

    // Update database
    await this.chatService.removeUserFromRoom(userId, roomId);

    // If room is now empty, clean up
    const roomSizeAfter = await this.getRoomSize(roomId);
    if (roomSizeAfter === 0) {
      await this.chatService.markRoomEmpty(roomId);
      console.log(`Room ${roomId} is now empty`);
    }

    // Confirm to leaver
    client.emit('left-room', {
      roomId,
      previousMemberCount: roomSizeBefore,
      currentMemberCount: roomSizeAfter,
    });
  }

  private async getRoomSize(roomId: string): Promise<number> {
    const sockets = await this.server.in(roomId).fetchSockets();
    return sockets.length;
  }
}
```

### **Leave with Tracking**

```typescript
@WebSocketGateway()
export class TrackedLeaveGateway {
  // Track user-room relationships
  private userRooms = new Map<string, Set<string>>();

  @SubscribeMessage('join-room')
  handleJoin(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;

    // Join room
    client.join(roomId);

    // Track
    if (!this.userRooms.has(userId)) {
      this.userRooms.set(userId, new Set());
    }
    this.userRooms.get(userId)!.add(roomId);
  }

  @SubscribeMessage('leave-room')
  handleLeave(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;

    // Leave room
    client.leave(roomId);

    // Untrack
    const userRoomSet = this.userRooms.get(userId);
    if (userRoomSet) {
      userRoomSet.delete(roomId);
      
      // If user has no rooms left, remove from map
      if (userRoomSet.size === 0) {
        this.userRooms.delete(userId);
      }
    }

    console.log(
      `User ${userId} left room ${roomId}. ` +
      `Remaining rooms: ${userRoomSet?.size || 0}`,
    );
  }

  // Get all rooms user is in
  getUserRooms(userId: string): string[] {
    const rooms = this.userRooms.get(userId);
    return rooms ? Array.from(rooms) : [];
  }
}
```

### **Kick User from Room**

```typescript
@WebSocketGateway()
export class KickUserGateway {
  @WebSocketServer()
  server: Server;

  // Admin kicks user from room
  @SubscribeMessage('kick-user')
  async handleKickUser(
    @MessageBody() data: { roomId: string; userId: string; reason?: string },
    @ConnectedSocket() client: Socket,
  ) {
    const adminUserId = client.handshake.auth.userId;

    // Verify admin permissions
    const isAdmin = await this.checkAdminPermissions(
      adminUserId,
      data.roomId,
    );
    
    if (!isAdmin) {
      return { error: 'Insufficient permissions' };
    }

    // Find user's socket(s)
    const userSockets = await this.server
      .in(`user:${data.userId}`)
      .fetchSockets();

    // Remove from room
    userSockets.forEach(socket => {
      if (socket.rooms.has(data.roomId)) {
        // Notify user they were kicked
        socket.emit('kicked-from-room', {
          roomId: data.roomId,
          reason: data.reason || 'No reason provided',
          kickedBy: adminUserId,
        });

        // Remove from room
        socket.leave(data.roomId);
      }
    });

    // Notify room members
    this.server.to(data.roomId).emit('user-kicked', {
      userId: data.userId,
      roomId: data.roomId,
      kickedBy: adminUserId,
      reason: data.reason,
    });

    return { success: true, message: 'User kicked from room' };
  }

  private async checkAdminPermissions(
    userId: string,
    roomId: string,
  ): Promise<boolean> {
    // Check if user is admin for this room
    return true; // Replace with actual logic
  }
}
```

### **Leave from Service**

```typescript
// room.service.ts
import { Injectable } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';

@Injectable()
export class RoomService {
  constructor(private chatGateway: ChatGateway) {}

  async removeUserFromRoom(userId: string, roomId: string) {
    // Get all user's sockets
    const sockets = await this.chatGateway.server
      .in(`user:${userId}`)
      .fetchSockets();

    // Remove each socket from room
    sockets.forEach(socket => {
      if (socket.rooms.has(roomId)) {
        socket.leave(roomId);
        console.log(`Socket ${socket.id} left room ${roomId}`);
      }
    });

    // Notify user
    this.chatGateway.server.to(`user:${userId}`).emit('removed-from-room', {
      roomId,
      reason: 'Removed by admin',
    });

    // Notify room
    this.chatGateway.server.to(roomId).emit('user-removed', {
      userId,
      roomId,
    });
  }
}
```

### **Temporary Room Leave**

```typescript
@WebSocketGateway()
export class TemporaryLeaveGateway {
  // Track temporary leaves
  private temporaryLeaves = new Map<string, NodeJS.Timeout>();

  @SubscribeMessage('leave-temporarily')
  handleTemporaryLeave(
    @MessageBody() data: { roomId: string; duration: number },
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;

    // Leave room
    client.leave(data.roomId);

    // Notify room
    client.to(data.roomId).emit('user-left-temporarily', {
      userId,
      duration: data.duration,
    });

    // Set auto-rejoin timer
    const timer = setTimeout(() => {
      client.join(data.roomId);
      client.emit('rejoined-room', { roomId: data.roomId });
      this.temporaryLeaves.delete(`${userId}:${data.roomId}`);
    }, data.duration);

    this.temporaryLeaves.set(`${userId}:${data.roomId}`, timer);

    return {
      success: true,
      willRejoinIn: data.duration,
    };
  }

  // Cancel temporary leave
  @SubscribeMessage('cancel-temporary-leave')
  handleCancelTemporaryLeave(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;
    const key = `${userId}:${roomId}`;
    const timer = this.temporaryLeaves.get(key);

    if (timer) {
      clearTimeout(timer);
      this.temporaryLeaves.delete(key);

      // Rejoin immediately
      client.join(roomId);

      return { success: true, message: 'Rejoined room early' };
    }

    return { success: false, message: 'No temporary leave found' };
  }
}
```

### **Verify Leave Success**

```typescript
@WebSocketGateway()
export class VerifyLeaveGateway {
  @SubscribeMessage('leave-room')
  handleLeaveRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Check if in room before leaving
    const wasInRoom = client.rooms.has(roomId);

    if (!wasInRoom) {
      return {
        success: false,
        error: 'Not in room',
      };
    }

    // Leave room
    client.leave(roomId);

    // VERIFY: Check if socket is still in room
    const stillInRoom = client.rooms.has(roomId);
    
    if (!stillInRoom) {
      console.log(`✓ Successfully left ${roomId}`);
      
      return {
        success: true,
        roomId,
        rooms: Array.from(client.rooms),
      };
    } else {
      console.error(`✗ Failed to leave ${roomId}`);
      
      return {
        success: false,
        error: 'Failed to leave room',
      };
    }
  }
}
```

### **Leave All Except Personal Room**

```typescript
@WebSocketGateway()
export class LeaveAllExceptPersonalGateway {
  @SubscribeMessage('reset-rooms')
  handleResetRooms(@ConnectedSocket() client: Socket) {
    const userId = client.handshake.auth.userId;
    const personalRoom = `user:${userId}`;

    // Get all rooms
    const allRooms = Array.from(client.rooms);

    // Leave all except socket ID and personal room
    allRooms.forEach(room => {
      if (room !== client.id && room !== personalRoom) {
        client.leave(room);
        console.log(`Left room: ${room}`);
      }
    });

    // Now socket is only in: socket ID + personal room
    return {
      success: true,
      remainingRooms: [client.id, personalRoom],
    };
  }
}
```

### **Production Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayDisconnect,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway({ namespace: '/chat' })
export class ProductionLeaveGateway implements OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  private logger = new Logger('ProductionLeaveGateway');

  constructor(
    private chatService: ChatService,
    private roomsService: RoomsService,
  ) {}

  // Manual leave
  @SubscribeMessage('leave-room')
  async handleLeaveRoom(
    @MessageBody() data: { roomId: string; reason?: string },
    @ConnectedSocket() client: Socket,
  ) {
    const startTime = Date.now();
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;

    try {
      // Validate membership
      if (!client.rooms.has(data.roomId)) {
        return {
          success: false,
          error: 'Not a member of this room',
        };
      }

      // Get room info
      const room = await this.roomsService.findRoom(data.roomId);
      if (!room) {
        return { success: false, error: 'Room not found' };
      }

      // Check if room is mandatory
      if (room.isMandatory) {
        return {
          success: false,
          error: 'Cannot leave mandatory room',
        };
      }

      // Get room size before leaving
      const sizeBefore = await this.getRoomSize(data.roomId);

      // Notify others BEFORE leaving
      client.to(data.roomId).emit('user-left-room', {
        userId,
        username,
        roomId: data.roomId,
        reason: data.reason,
        timestamp: new Date(),
      });

      // Leave room
      client.leave(data.roomId);

      // Update database
      await this.roomsService.removeUserFromRoom(userId, data.roomId);

      // Get room size after leaving
      const sizeAfter = await this.getRoomSize(data.roomId);

      // Clean up empty room
      if (sizeAfter === 0) {
        await this.roomsService.markRoomEmpty(data.roomId);
        this.logger.log(`Room ${data.roomId} is now empty`);
      }

      // Log success
      const duration = Date.now() - startTime;
      this.logger.log(
        `User ${userId} left ${data.roomId} in ${duration}ms`,
      );

      return {
        success: true,
        roomId: data.roomId,
        previousMemberCount: sizeBefore,
        currentMemberCount: sizeAfter,
      };
    } catch (error) {
      this.logger.error('Error leaving room:', error);
      return {
        success: false,
        error: 'Failed to leave room',
      };
    }
  }

  // Auto-leave on disconnect
  async handleDisconnect(client: Socket) {
    const userId = client.handshake.auth.userId;

    // Get all rooms (except personal room)
    const rooms = Array.from(client.rooms).filter(
      room => room !== client.id,
    );

    try {
      // Notify all rooms
      rooms.forEach(room => {
        client.to(room).emit('user-disconnected', {
          userId,
          room,
          timestamp: new Date(),
        });
      });

      // Update database
      await this.chatService.removeUserFromAllRooms(userId);

      this.logger.log(
        `User ${userId} disconnected, auto-left ${rooms.length} rooms`,
      );
    } catch (error) {
      this.logger.error('Error during disconnect cleanup:', error);
    }
  }

  private async getRoomSize(roomId: string): Promise<number> {
    const sockets = await this.server.in(roomId).fetchSockets();
    return sockets.length;
  }
}
```

**Interview Tip**: Leave room with **`client.leave(roomName)`** - **synchronous** in Socket.IO v4+, **immediately** removes socket from room, **returns void**. **Important**: notify room members **BEFORE** leaving (`client.to(room).emit()` while still in room, otherwise can't broadcast to it). **Auto-leave**: Socket.IO **automatically** removes socket from **all rooms** on disconnect, but you should manually notify rooms in `handleDisconnect()` hook. **Verify**: check `client.rooms.has(roomName)` returns `false` after leaving. **Multiple rooms**: loop through array, call `leave()` for each. **Get rooms**: `Array.from(client.rooms)` includes socket ID (which is also a room), filter it out. **Leave all**: iterate `client.rooms`, skip socket ID and personal room. **From service**: fetch user's sockets (`server.in('user:userId').fetchSockets()`), call `socket.leave(room)` on each. **Room cleanup**: if room becomes empty after leave, Socket.IO auto-deletes it (no manual cleanup needed). **Tracking**: maintain Map of user-room relationships, update on join/leave, cleanup on disconnect.

</details>
22. How do you emit to a specific room?

<details>
<summary><strong>Answer</strong></summary>

To emit to a specific room, use **`server.to(roomName).emit(event, data)`** to send to **all sockets in room** (including sender if sender is in room), or **`client.to(roomName).emit(event, data)`** to send to **all except sender**. The `to()` method **returns a BroadcastOperator** that allows chaining additional methods like `to()`, `except()`, `volatile`, `compress`, `timeout()`, etc.

### **Basic Syntax**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway()
export class EmitToRoomGateway {
  @WebSocketServer()
  server: Server;

  // METHOD 1: Emit to ALL in room (including sender if in room)
  emitToAllInRoom(roomId: string, event: string, data: any) {
    this.server.to(roomId).emit(event, data);
    // All sockets in room receive this
  }

  // METHOD 2: Emit to ALL EXCEPT SENDER
  @SubscribeMessage('send-to-room')
  emitExceptSender(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    // Broadcast to room, excluding sender
    client.to(data.roomId).emit('room-message', {
      message: data.message,
      from: client.id,
    });
    // Sender does NOT receive this
  }

  // METHOD 3: Emit to room + confirm to sender separately
  @SubscribeMessage('send-message')
  handleSendMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    // To room (all including sender)
    this.server.to(data.roomId).emit('new-message', {
      message: data.message,
      from: client.id,
      timestamp: new Date(),
    });

    // Separate confirmation to sender
    client.emit('message-sent', {
      roomId: data.roomId,
      success: true,
    });
  }
}
```

### **Chat Room Example**

```typescript
@WebSocketGateway({ namespace: '/chat' })
export class ChatRoomGateway {
  @WebSocketServer()
  server: Server;

  // Send message to room
  @SubscribeMessage('chat-message')
  handleChatMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;

    // Validate room membership
    if (!client.rooms.has(data.roomId)) {
      client.emit('error', { message: 'Not in room' });
      return;
    }

    // Broadcast to ALL in room (including sender)
    this.server.to(data.roomId).emit('new-message', {
      userId,
      username,
      message: data.message,
      roomId: data.roomId,
      timestamp: new Date(),
    });

    console.log(`Message sent to room ${data.roomId} by ${username}`);
  }

  // Typing indicator (to others, not sender)
  @SubscribeMessage('typing')
  handleTyping(
    @MessageBody() data: { roomId: string; isTyping: boolean },
    @ConnectedSocket() client: Socket,
  ) {
    const username = client.handshake.auth.username;

    // Broadcast to room EXCEPT sender
    client.to(data.roomId).emit('user-typing', {
      username,
      isTyping: data.isTyping,
      roomId: data.roomId,
    });
  }

  // System announcement (from server)
  announceToRoom(roomId: string, announcement: string) {
    this.server.to(roomId).emit('system-announcement', {
      message: announcement,
      type: 'system',
      timestamp: new Date(),
    });
  }
}
```

### **Multiple Rooms**

```typescript
@WebSocketGateway()
export class MultiRoomEmitGateway {
  @WebSocketServer()
  server: Server;

  // PATTERN 1: Emit to multiple rooms (array)
  emitToMultipleRooms() {
    this.server.to(['room1', 'room2', 'room3']).emit('update', {
      message: 'Update for multiple rooms',
    });
    // Sockets in ANY of these rooms receive it
  }

  // PATTERN 2: Chain to() calls
  emitToMultipleRoomsChained() {
    this.server
      .to('room1')
      .to('room2')
      .to('room3')
      .emit('update', { message: 'Chained update' });
    // Same as array method
  }

  // PATTERN 3: Emit to team rooms
  notifyAllTeams(teams: string[], notification: any) {
    const roomNames = teams.map(teamId => `team:${teamId}`);
    this.server.to(roomNames).emit('team-notification', notification);
  }

  // PATTERN 4: Emit to all rooms except one
  emitExceptRoom(excludeRoom: string) {
    this.server.except(excludeRoom).emit('broadcast', {
      message: 'Message to everyone except one room',
    });
  }
}
```

### **From Service/Controller**

```typescript
// notification.service.ts
import { Injectable } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';

@Injectable()
export class NotificationService {
  constructor(private chatGateway: ChatGateway) {}

  // Emit from service
  notifyRoom(roomId: string, notification: any) {
    this.chatGateway.server.to(roomId).emit('notification', {
      ...notification,
      timestamp: new Date(),
    });
  }

  // Notify multiple rooms
  notifyRooms(roomIds: string[], notification: any) {
    roomIds.forEach(roomId => {
      this.chatGateway.server.to(roomId).emit('notification', notification);
    });
  }

  // Notify team
  notifyTeam(teamId: string, message: string) {
    this.chatGateway.server.to(`team:${teamId}`).emit('team-update', {
      message,
      teamId,
      timestamp: new Date(),
    });
  }
}

// Gateway must be injectable and expose server
@Injectable()
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server; // Public for service access
}

// notification.controller.ts
import { Controller, Post, Body } from '@nestjs/common';
import { NotificationService } from './notification.service';

@Controller('notifications')
export class NotificationController {
  constructor(private notificationService: NotificationService) {}

  @Post('send-to-room')
  sendToRoom(@Body() data: { roomId: string; message: string }) {
    this.notificationService.notifyRoom(data.roomId, {
      message: data.message,
    });
    return { success: true };
  }
}
```

### **With Acknowledgments**

```typescript
@WebSocketGateway()
export class AckRoomGateway {
  @WebSocketServer()
  server: Server;

  // Emit with timeout and ack
  async emitToRoomWithAck(roomId: string, event: string, data: any) {
    try {
      // Set 5 second timeout
      const responses = await this.server
        .to(roomId)
        .timeout(5000)
        .emitWithAck(event, data);

      console.log(`Received ${responses.length} acknowledgments`);
      return responses;
    } catch (error) {
      console.error('Some clients did not acknowledge:', error);
      throw error;
    }
  }

  // Example: Request room members' status
  @SubscribeMessage('request-room-status')
  async handleRequestStatus(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    try {
      // Ask all room members for status
      const responses = await this.server
        .to(roomId)
        .timeout(3000)
        .emitWithAck('status-request', { requestedBy: client.id });

      client.emit('room-status-collected', {
        roomId,
        responses,
        count: responses.length,
      });
    } catch (error) {
      client.emit('status-request-failed', {
        roomId,
        error: 'Timeout or error',
      });
    }
  }
}
```

### **Volatile Emit (Non-Critical)**

```typescript
@WebSocketGateway()
export class VolatileRoomGateway {
  @WebSocketServer()
  server: Server;

  // Volatile: drop if client is busy
  @SubscribeMessage('mouse-move')
  handleMouseMove(
    @MessageBody() data: { roomId: string; x: number; y: number },
    @ConnectedSocket() client: Socket,
  ) {
    // Send to room, but drop if clients are busy
    // Good for high-frequency non-critical updates
    client.to(data.roomId).volatile.emit('cursor-move', {
      userId: client.id,
      x: data.x,
      y: data.y,
    });
  }

  // Non-volatile: always deliver
  @SubscribeMessage('important-update')
  handleImportantUpdate(
    @MessageBody() data: { roomId: string; update: string },
    @ConnectedSocket() client: Socket,
  ) {
    // Always delivered (buffered if client busy)
    this.server.to(data.roomId).emit('critical-update', {
      update: data.update,
      timestamp: new Date(),
    });
  }
}
```

### **Compressed Emit (Large Data)**

```typescript
@WebSocketGateway()
export class CompressedRoomGateway {
  @WebSocketServer()
  server: Server;

  // Emit with compression for large data
  sendLargeDataToRoom(roomId: string, largeData: any) {
    this.server
      .to(roomId)
      .compress(true) // Enable compression
      .emit('large-data', largeData);
  }

  // Example: Send file contents to room
  @SubscribeMessage('share-file')
  handleShareFile(
    @MessageBody() data: { roomId: string; fileContent: string },
    @ConnectedSocket() client: Socket,
  ) {
    this.server
      .to(data.roomId)
      .compress(true)
      .emit('file-shared', {
        fileName: data.fileName,
        content: data.fileContent,
        sharedBy: client.id,
      });
  }
}
```

### **Room Broadcasting Patterns**

```typescript
@WebSocketGateway({ namespace: '/collaboration' })
export class CollaborationGateway {
  @WebSocketServer()
  server: Server;

  // PATTERN 1: Update everyone in room
  @SubscribeMessage('document-edit')
  handleDocumentEdit(
    @MessageBody() data: { documentId: string; changes: any },
    @ConnectedSocket() client: Socket,
  ) {
    // Broadcast changes to all collaborators (including editor)
    this.server.to(`doc:${data.documentId}`).emit('document-updated', {
      changes: data.changes,
      by: client.id,
    });
  }

  // PATTERN 2: Update others, confirm to sender
  @SubscribeMessage('lock-section')
  handleLockSection(
    @MessageBody() data: { documentId: string; sectionId: string },
    @ConnectedSocket() client: Socket,
  ) {
    // Notify others (not sender)
    client.to(`doc:${data.documentId}`).emit('section-locked', {
      sectionId: data.sectionId,
      by: client.id,
    });

    // Confirm to sender
    client.emit('lock-confirmed', {
      sectionId: data.sectionId,
    });
  }

  // PATTERN 3: Chain operations
  @SubscribeMessage('multi-room-update')
  handleMultiRoomUpdate(
    @MessageBody() data: { rooms: string[]; update: any },
    @ConnectedSocket() client: Socket,
  ) {
    // Emit to multiple rooms with compression and timeout
    this.server
      .to(data.rooms)
      .compress(true)
      .timeout(5000)
      .emit('update', data.update);
  }

  // PATTERN 4: Conditional emit
  async emitToActiveRoomMembers(roomId: string, data: any) {
    // Get all sockets in room
    const sockets = await this.server.in(roomId).fetchSockets();

    // Filter active users (e.g., not idle)
    const activeSockets = sockets.filter(
      socket => socket.data.isActive === true,
    );

    // Emit only to active users
    activeSockets.forEach(socket => {
      socket.emit('active-user-update', data);
    });
  }
}
```

### **Admin/Moderator Broadcasting**

```typescript
@WebSocketGateway()
export class AdminBroadcastGateway {
  @WebSocketServer()
  server: Server;

  // Admin announcement to specific room
  @SubscribeMessage('admin-announce')
  handleAdminAnnounce(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    const role = client.handshake.auth.role;

    // Check admin permissions
    if (role !== 'admin' && role !== 'moderator') {
      client.emit('error', { message: 'Insufficient permissions' });
      return;
    }

    // Broadcast announcement to room
    this.server.to(data.roomId).emit('admin-announcement', {
      message: data.message,
      from: client.handshake.auth.username,
      role,
      timestamp: new Date(),
    });
  }

  // Broadcast to all rooms
  @SubscribeMessage('global-announcement')
  handleGlobalAnnounce(
    @MessageBody() message: string,
    @ConnectedSocket() client: Socket,
  ) {
    const role = client.handshake.auth.role;

    if (role !== 'admin') {
      client.emit('error', { message: 'Admin only' });
      return;
    }

    // Broadcast to everyone
    this.server.emit('global-announcement', {
      message,
      from: client.handshake.auth.username,
      timestamp: new Date(),
    });
  }
}
```

### **Get Room Info Before Emitting**

```typescript
@WebSocketGateway()
export class RoomInfoGateway {
  @WebSocketServer()
  server: Server;

  // Check if room has members before emitting
  async emitIfRoomNotEmpty(roomId: string, event: string, data: any) {
    const sockets = await this.server.in(roomId).fetchSockets();

    if (sockets.length > 0) {
      this.server.to(roomId).emit(event, data);
      console.log(`Emitted to ${sockets.length} sockets in ${roomId}`);
    } else {
      console.log(`Room ${roomId} is empty, not emitting`);
    }
  }

  // Get room members and emit
  async emitWithMemberInfo(roomId: string) {
    const sockets = await this.server.in(roomId).fetchSockets();

    const members = sockets.map(s => ({
      id: s.id,
      userId: s.handshake.auth.userId,
      username: s.handshake.auth.username,
    }));

    // Send member list to room
    this.server.to(roomId).emit('room-members-update', {
      roomId,
      members,
      count: members.length,
    });
  }

  // Check room size before emitting
  async emitIfRoomSizeAbove(roomId: string, minSize: number) {
    const sockets = await this.server.in(roomId).fetchSockets();

    if (sockets.length >= minSize) {
      this.server.to(roomId).emit('enough-players', {
        message: 'Game can start',
        playerCount: sockets.length,
      });
    }
  }
}
```

### **Production Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway({ namespace: '/chat' })
export class ProductionRoomEmitGateway {
  @WebSocketServer()
  server: Server;

  private logger = new Logger('ProductionRoomEmitGateway');

  constructor(private chatService: ChatService) {}

  // Send message to room with validation
  @SubscribeMessage('send-message')
  async handleSendMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    const startTime = Date.now();
    const userId = client.handshake.auth.userId;
    const username = client.handshake.auth.username;

    try {
      // VALIDATION 1: Check room membership
      if (!client.rooms.has(data.roomId)) {
        return { success: false, error: 'Not in room' };
      }

      // VALIDATION 2: Rate limiting (optional)
      // const canSend = await this.checkRateLimit(userId, data.roomId);
      // if (!canSend) {
      //   return { success: false, error: 'Rate limit exceeded' };
      // }

      // VALIDATION 3: Content validation
      if (!data.message || data.message.trim().length === 0) {
        return { success: false, error: 'Message cannot be empty' };
      }

      if (data.message.length > 1000) {
        return { success: false, error: 'Message too long' };
      }

      // Save to database
      const savedMessage = await this.chatService.saveMessage({
        userId,
        roomId: data.roomId,
        content: data.message,
      });

      // EMIT TO ROOM (all including sender)
      this.server.to(data.roomId).emit('new-message', {
        id: savedMessage.id,
        userId,
        username,
        message: data.message,
        roomId: data.roomId,
        timestamp: savedMessage.createdAt,
      });

      // Log success
      const duration = Date.now() - startTime;
      this.logger.log(
        `Message sent to room ${data.roomId} by ${username} in ${duration}ms`,
      );

      return { success: true, messageId: savedMessage.id };
    } catch (error) {
      this.logger.error('Error sending message:', error);
      return { success: false, error: 'Failed to send message' };
    }
  }

  // Emit from service/external trigger
  notifyRoom(roomId: string, notification: any) {
    this.server.to(roomId).emit('notification', {
      ...notification,
      timestamp: new Date(),
    });

    this.logger.log(`Notification sent to room ${roomId}`);
  }

  // Emit to multiple rooms
  notifyRooms(roomIds: string[], notification: any) {
    roomIds.forEach(roomId => {
      this.server.to(roomId).emit('notification', notification);
    });

    this.logger.log(`Notification sent to ${roomIds.length} rooms`);
  }
}
```

**Interview Tip**: Emit to room with **`server.to(roomName).emit(event, data)`** to send to **all in room** (including sender), or **`client.to(roomName).emit(event, data)`** to send **except sender**. **`to()` method** returns **BroadcastOperator** allowing chaining. **Multiple rooms**: `server.to([room1, room2]).emit()` or chain `to()` calls. **From anywhere**: inject gateway, use `gateway.server.to(room).emit()` from services/controllers. **With acknowledgment**: `server.to(room).timeout(5000).emitWithAck()` - returns array of responses. **Volatile**: `server.to(room).volatile.emit()` drops if client busy (non-critical updates like cursor position). **Compressed**: `server.to(room).compress(true).emit()` for large data. **Except room**: `server.except(room).emit()` sends to all except that room. **Verify room not empty**: `await server.in(room).fetchSockets()` returns array, check length before emitting. **Common patterns**: (1) broadcast to all in room (`server.to()`), (2) broadcast except sender with separate confirmation (`client.to()` + `client.emit()`), (3) from service for external triggers. **Validation**: check sender is in room before emitting (`client.rooms.has(room)`).

</details>
23. What are Namespaces in Socket.IO?

<details>
<summary><strong>Answer</strong></summary>

**Namespaces** in Socket.IO are **separate communication channels** that share the **same underlying connection**. Each namespace has its **own event handlers**, **rooms**, **middleware**, and **connection logic**. They're identified by a **path** (e.g., `/chat`, `/notifications`, `/admin`) and are perfect for **separating concerns** in your application while using a single WebSocket connection.

### **Core Concept**

```typescript
/**
 * NAMESPACES = Separate channels on same connection
 * 
 * URL Structure:
 * - Default: ws://server.com/ (or ws://server.com/socket.io)
 * - Custom: ws://server.com/chat (chat namespace)
 * - Custom: ws://server.com/notifications (notifications namespace)
 * 
 * Key characteristics:
 * 1. Each namespace has its own event handlers
 * 2. Each namespace has its own rooms
 * 3. Each namespace has its own middleware
 * 4. Clients connect to specific namespace
 * 5. Server can have multiple gateways (one per namespace)
 * 6. Multiplexing: multiple namespaces share same TCP connection
 */

// Client connects to specific namespace:
// const chatSocket = io('http://server.com/chat');
// const notifSocket = io('http://server.com/notifications');
// Both use same underlying connection!
```

### **Creating Namespaces**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

// DEFAULT NAMESPACE: / (root)
@WebSocketGateway()
export class DefaultGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('default-message')
  handleMessage(@MessageBody() data: any) {
    // Handles messages on default namespace
    return { event: 'default-response', data };
  }
}

// CUSTOM NAMESPACE: /chat
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('chat-message')
  handleChatMessage(@MessageBody() data: any) {
    // Only handles messages from /chat namespace
    return { event: 'chat-response', data };
  }
}

// CUSTOM NAMESPACE: /notifications
@WebSocketGateway({ namespace: '/notifications' })
export class NotificationsGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('subscribe')
  handleSubscribe(@ConnectedSocket() client: Socket) {
    // Only handles /notifications namespace
    client.emit('subscribed', { message: 'Subscribed to notifications' });
  }
}

// CUSTOM NAMESPACE: /admin
@WebSocketGateway({ namespace: '/admin' })
export class AdminGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('admin-command')
  handleAdminCommand(@MessageBody() command: string) {
    // Admin-only namespace
    return { event: 'command-executed', command };
  }
}
```

### **Client Connection**

```typescript
// client.ts
import { io, Socket } from 'socket.io-client';

// Connect to DEFAULT namespace (/)
const defaultSocket = io('http://localhost:3000');
// Same as: io('http://localhost:3000/');

defaultSocket.on('connect', () => {
  console.log('Connected to default namespace');
});

// Connect to CHAT namespace (/chat)
const chatSocket = io('http://localhost:3000/chat');

chatSocket.on('connect', () => {
  console.log('Connected to /chat namespace');
});

chatSocket.emit('chat-message', { message: 'Hello chat!' });

// Connect to NOTIFICATIONS namespace (/notifications)
const notifSocket = io('http://localhost:3000/notifications');

notifSocket.on('connect', () => {
  console.log('Connected to /notifications namespace');
});

notifSocket.on('notification', (data) => {
  console.log('New notification:', data);
});

// Connect to ADMIN namespace with auth
const adminSocket = io('http://localhost:3000/admin', {
  auth: {
    token: 'admin-token-here',
    role: 'admin',
  },
});

// All these sockets share the SAME underlying connection!
// This is called "multiplexing"
```

### **Real-World Multi-Namespace Application**

```typescript
// chat.gateway.ts
import { Injectable } from '@nestjs/common';
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@Injectable()
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  handleConnection(client: Socket) {
    const username = client.handshake.auth.username;
    console.log(`${username} connected to /chat`);

    // Broadcast to chat namespace only
    client.broadcast.emit('user-joined', { username });
  }

  @SubscribeMessage('send-message')
  handleMessage(
    @MessageBody() data: { room: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    // Emit to room in /chat namespace
    this.server.to(data.room).emit('new-message', {
      username: client.handshake.auth.username,
      message: data.message,
      timestamp: new Date(),
    });
  }
}

// notifications.gateway.ts
@Injectable()
@WebSocketGateway({ namespace: '/notifications' })
export class NotificationsGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    console.log(`User ${userId} connected to /notifications`);

    // Join personal notification room
    client.join(`user:${userId}`);
  }

  // Called from services
  sendNotification(userId: string, notification: any) {
    this.server.to(`user:${userId}`).emit('notification', notification);
  }
}

// admin.gateway.ts
@Injectable()
@WebSocketGateway({ namespace: '/admin' })
export class AdminGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  handleConnection(client: Socket) {
    const role = client.handshake.auth.role;
    
    // Validate admin access
    if (role !== 'admin') {
      client.disconnect();
      return;
    }

    console.log('Admin connected to /admin namespace');
  }

  @SubscribeMessage('get-stats')
  async handleGetStats() {
    // Admin-only functionality
    return {
      event: 'stats',
      data: await this.getSystemStats(),
    };
  }

  private async getSystemStats() {
    return {
      activeUsers: 150,
      activeRooms: 30,
      serverLoad: 45,
    };
  }
}

// game.gateway.ts
@Injectable()
@WebSocketGateway({ namespace: '/game' })
export class GameGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('player-move')
  handlePlayerMove(
    @MessageBody() data: { gameId: string; move: any },
    @ConnectedSocket() client: Socket,
  ) {
    // Broadcast move to game room
    client.to(`game:${data.gameId}`).emit('opponent-move', {
      playerId: client.id,
      move: data.move,
    });
  }
}
```

### **Namespace with Middleware**

```typescript
import { OnGatewayConnection } from '@nestjs/websockets';

@WebSocketGateway({ namespace: '/private' })
export class PrivateGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  // Middleware runs ONLY for /private namespace
  afterInit(server: Server) {
    server.use((socket, next) => {
      const token = socket.handshake.auth.token;

      // Validate token
      if (!token || !this.validateToken(token)) {
        return next(new Error('Authentication failed'));
      }

      // Attach user data
      socket.data.user = this.decodeToken(token);
      next();
    });

    console.log('/private namespace initialized with auth middleware');
  }

  handleConnection(client: Socket) {
    // Only authenticated users reach here
    console.log(`Authenticated user ${client.data.user.id} connected`);
  }

  private validateToken(token: string): boolean {
    // Token validation logic
    return true;
  }

  private decodeToken(token: string): any {
    return { id: '123', username: 'john' };
  }
}
```

### **Dynamic Namespaces**

```typescript
import { OnGatewayConnection } from '@nestjs/websockets';

@WebSocketGateway()
export class DynamicNamespacesGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  afterInit(server: Server) {
    // Create namespace dynamically
    const dynamicNsp = server.of(/^\/dynamic-\d+$/);

    dynamicNsp.on('connection', (socket) => {
      const namespace = socket.nsp.name;
      console.log(`Client connected to dynamic namespace: ${namespace}`);
      // e.g., /dynamic-123, /dynamic-456, etc.

      socket.on('message', (data) => {
        // Handle message in dynamic namespace
        dynamicNsp.emit('broadcast', data);
      });
    });
  }

  handleConnection(client: Socket) {
    // Default namespace connection
  }
}
```

### **Emitting Across Namespaces**

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
@WebSocketGateway()
export class CrossNamespaceGateway {
  @WebSocketServer()
  server: Server;

  // Get specific namespace
  getChatNamespace() {
    return this.server.of('/chat');
  }

  getNotificationsNamespace() {
    return this.server.of('/notifications');
  }

  // Emit to specific namespace from anywhere
  notifyAllChatUsers(message: string) {
    this.server.of('/chat').emit('announcement', { message });
  }

  notifyAllNotificationUsers(notification: any) {
    this.server.of('/notifications').emit('global-notification', notification);
  }

  // Emit to multiple namespaces
  broadcastToAll(event: string, data: any) {
    // Default namespace
    this.server.emit(event, data);

    // Chat namespace
    this.server.of('/chat').emit(event, data);

    // Notifications namespace
    this.server.of('/notifications').emit(event, data);
  }
}
```

### **Namespace Configuration**

```typescript
@WebSocketGateway({
  namespace: '/custom',
  cors: {
    origin: 'http://localhost:4200',
    credentials: true,
  },
  transports: ['websocket', 'polling'],
  pingTimeout: 10000,
  pingInterval: 5000,
})
export class ConfiguredNamespaceGateway {
  @WebSocketServer()
  server: Server;

  // This configuration applies ONLY to /custom namespace
}
```

### **Get Namespace Info**

```typescript
@WebSocketGateway({ namespace: '/analytics' })
export class AnalyticsGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('get-namespace-info')
  async handleGetInfo() {
    // Get namespace instance
    const namespace = this.server; // Current namespace

    // Get all connected sockets in this namespace
    const sockets = await namespace.fetchSockets();

    // Get all rooms in this namespace
    const rooms = Array.from(namespace.adapter.rooms.keys());

    return {
      event: 'namespace-info',
      data: {
        name: namespace.name,
        socketCount: sockets.length,
        roomCount: rooms.length,
        rooms,
      },
    };
  }

  // Get info from another namespace
  async getNamespaceStats(namespaceName: string) {
    const nsp = this.server.of(namespaceName);
    const sockets = await nsp.fetchSockets();

    return {
      name: namespaceName,
      connections: sockets.length,
    };
  }
}
```

### **Module Setup for Multiple Namespaces**

```typescript
// websockets.module.ts
import { Module } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';
import { NotificationsGateway } from './notifications.gateway';
import { AdminGateway } from './admin.gateway';
import { GameGateway } from './game.gateway';

@Module({
  providers: [
    ChatGateway,          // /chat namespace
    NotificationsGateway, // /notifications namespace
    AdminGateway,         // /admin namespace
    GameGateway,          // /game namespace
  ],
  exports: [
    ChatGateway,
    NotificationsGateway,
    AdminGateway,
    GameGateway,
  ],
})
export class WebSocketsModule {}

// Each gateway handles its own namespace independently
```

### **Use Cases for Namespaces**

```typescript
/**
 * NAMESPACE USE CASES:
 * 
 * 1. FEATURE SEPARATION
 *    /chat      - Chat functionality
 *    /video     - Video call events
 *    /game      - Game events
 *    /notifications - Notifications
 * 
 * 2. ACCESS CONTROL
 *    /public    - Public content (no auth)
 *    /private   - Authenticated users
 *    /admin     - Admin only
 *    /moderator - Moderators only
 * 
 * 3. TENANT ISOLATION
 *    /tenant-1  - Company 1 data
 *    /tenant-2  - Company 2 data
 *    /tenant-3  - Company 3 data
 * 
 * 4. VERSIONING
 *    /v1        - API v1
 *    /v2        - API v2
 *    /beta      - Beta features
 * 
 * 5. DEVICE TYPES
 *    /mobile    - Mobile clients
 *    /desktop   - Desktop clients
 *    /iot       - IoT devices
 * 
 * 6. GEOGRAPHIC
 *    /us        - US region
 *    /eu        - EU region
 *    /asia      - Asia region
 */

// Example: Feature separation
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {}

@WebSocketGateway({ namespace: '/video' })
export class VideoGateway {}

// Example: Access control
@WebSocketGateway({ namespace: '/public' })
export class PublicGateway {}

@WebSocketGateway({ namespace: '/admin' })
export class AdminGateway {}

// Example: Versioning
@WebSocketGateway({ namespace: '/v1' })
export class V1Gateway {}

@WebSocketGateway({ namespace: '/v2' })
export class V2Gateway {}
```

### **Production Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  OnGatewayDisconnect,
  OnGatewayInit,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

// CHAT NAMESPACE: /chat
@Injectable()
@WebSocketGateway({
  namespace: '/chat',
  cors: { origin: '*' },
})
export class ProductionChatGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  @WebSocketServer()
  server: Server;

  private logger = new Logger('ChatNamespace');

  afterInit() {
    this.logger.log('/chat namespace initialized');
  }

  async handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    // Join personal room in /chat namespace
    client.join(`user:${userId}`);

    // Get stats for this namespace
    const sockets = await this.server.fetchSockets();
    this.logger.log(
      `User ${userId} connected to /chat. ` +
      `Total connections: ${sockets.length}`,
    );
  }

  handleDisconnect(client: Socket) {
    this.logger.log(`Client ${client.id} disconnected from /chat`);
  }

  @SubscribeMessage('send-message')
  handleMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    this.server.to(data.roomId).emit('new-message', {
      userId: client.handshake.auth.userId,
      message: data.message,
      timestamp: new Date(),
    });
  }
}

// NOTIFICATIONS NAMESPACE: /notifications
@Injectable()
@WebSocketGateway({
  namespace: '/notifications',
  cors: { origin: '*' },
})
export class ProductionNotificationsGateway
  implements OnGatewayInit, OnGatewayConnection
{
  @WebSocketServer()
  server: Server;

  private logger = new Logger('NotificationsNamespace');

  afterInit() {
    this.logger.log('/notifications namespace initialized');
  }

  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    client.join(`user:${userId}`);
    this.logger.log(`User ${userId} subscribed to notifications`);
  }

  // Called from services
  sendNotification(userId: string, notification: any) {
    this.server.to(`user:${userId}`).emit('notification', {
      ...notification,
      timestamp: new Date(),
    });
    this.logger.log(`Notification sent to user ${userId}`);
  }
}
```

**Interview Tip**: **Namespaces** are **separate communication channels** on the **same connection** (multiplexing). **Created** with `@WebSocketGateway({ namespace: '/name' })`, **client connects** with `io('http://server.com/name')`. **Each namespace** has **independent**: (1) event handlers, (2) rooms, (3) middleware, (4) connection logic. **Use cases**: feature separation (`/chat`, `/game`), access control (`/admin`, `/public`), tenant isolation, versioning (`/v1`, `/v2`). **Multiple namespaces** share **same TCP connection** (efficient). **Access namespace**: `server.of('/name')` from any gateway. **Default namespace**: `/` (root), used when no namespace specified. **Configuration**: each namespace can have different CORS, transports, timeouts. **Module setup**: create separate gateway class for each namespace, register all in module providers. **Get info**: `server.name` for namespace name, `server.fetchSockets()` for connections in that namespace. **Rooms are namespace-specific**: room `room1` in `/chat` is separate from `room1` in `/notifications`.

</details>
24. What is the difference between Rooms and Namespaces?

<details>
<summary><strong>Answer</strong></summary>

**Rooms** and **Namespaces** serve different purposes: **Namespaces** are **separate channels** for **different features** (like `/chat`, `/game`, `/admin`), requiring **explicit client connection** to each. **Rooms** are **groups within a namespace** for **broadcasting** to subsets of clients, **managed server-side** only (clients don't explicitly connect to rooms). Think of **Namespaces** as **separate apps** and **Rooms** as **groups within an app**.

### **Quick Comparison Table**

| **Aspect** | **Namespaces** | **Rooms** |
|------------|----------------|------------|
| **Purpose** | Separate features/apps | Group clients within namespace |
| **Client Connection** | Explicit: `io('/chat')` | No explicit connection needed |
| **Server Management** | `@WebSocketGateway({ namespace })` | `socket.join(room)` / `socket.leave(room)` |
| **Visibility** | Client knows which namespace | Client doesn't know which rooms |
| **Scope** | Application-level separation | Within a namespace |
| **URL** | `ws://server.com/chat` | No URL, server-side only |
| **Event Handlers** | Independent for each namespace | Shared within namespace |
| **Middleware** | Per-namespace middleware | No room-specific middleware |
| **Use Case** | Feature separation, access control | Broadcasting to groups |
| **Example** | `/chat`, `/admin`, `/game` | `room-1`, `user:123`, `team:5` |
| **Multiple** | Socket connects to one namespace at a time | Socket can be in multiple rooms |
| **Auto-cleanup** | Manual disconnect | Auto-removed from rooms on disconnect |
| **Persistence** | Persistent (gateway always exists) | Dynamic (created/deleted automatically) |

### **Visual Comparison**

```typescript
/**
 * NAMESPACES (Application-level)
 * 
 * Client Side:                Server Side:
 * 
 * io('/')                     @WebSocketGateway()
 * io('/chat')                 @WebSocketGateway({ namespace: '/chat' })
 * io('/game')                 @WebSocketGateway({ namespace: '/game' })
 * 
 * - Each requires separate client connection
 * - Each has its own gateway class
 * - Each has independent event handlers
 * 
 * 
 * ROOMS (Within namespace)
 * 
 * Client Side:                Server Side:
 * 
 * // No explicit connection    socket.join('room-1')
 * // Client doesn't know       socket.join('room-2')
 * // which rooms it's in       socket.to('room-1').emit()
 * 
 * - No client code needed
 * - Managed server-side only
 * - Used for broadcasting to groups
 */
```

### **Namespaces Example**

```typescript
// SERVER: Three separate namespaces

// chat.gateway.ts
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('chat-message')
  handleChatMessage(@MessageBody() data: any) {
    // Only handles events from /chat namespace
    return { event: 'chat-response', data };
  }
}

// game.gateway.ts
@WebSocketGateway({ namespace: '/game' })
export class GameGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('game-move')
  handleGameMove(@MessageBody() data: any) {
    // Only handles events from /game namespace
    return { event: 'move-response', data };
  }
}

// admin.gateway.ts
@WebSocketGateway({ namespace: '/admin' })
export class AdminGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('admin-command')
  handleAdminCommand(@MessageBody() data: any) {
    // Only handles events from /admin namespace
    return { event: 'command-response', data };
  }
}

// CLIENT: Explicit connection to each namespace
const chatSocket = io('http://localhost:3000/chat');
const gameSocket = io('http://localhost:3000/game');
const adminSocket = io('http://localhost:3000/admin');

// Each socket connects to different namespace
chatSocket.emit('chat-message', { message: 'Hello' });
gameSocket.emit('game-move', { move: 'A1' });
adminSocket.emit('admin-command', { cmd: 'status' });
```

### **Rooms Example**

```typescript
// SERVER: One namespace with multiple rooms

@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  // Join room (server-side only)
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Add socket to room
    client.join(roomId);
    
    // Notify room members
    client.to(roomId).emit('user-joined', {
      socketId: client.id,
      room: roomId,
    });
  }

  // Send to room
  @SubscribeMessage('room-message')
  handleRoomMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    // Broadcast to room
    this.server.to(data.roomId).emit('new-message', {
      message: data.message,
      from: client.id,
    });
  }
}

// CLIENT: One connection, rooms managed server-side
const socket = io('http://localhost:3000/chat');

// Client requests to join room (server decides)
socket.emit('join-room', 'room-1');
socket.emit('join-room', 'room-2');

// Client sends to room
socket.emit('room-message', {
  roomId: 'room-1',
  message: 'Hello room 1',
});

// Client doesn't know which rooms it's in
// All room management is server-side
```

### **When to Use Namespaces vs Rooms**

```typescript
/**
 * USE NAMESPACES WHEN:
 * 
 * 1. SEPARATE FEATURES
 *    - Different parts of app: /chat, /game, /notifications
 *    - Want independent event handlers
 *    - Need different middleware per feature
 * 
 * 2. ACCESS CONTROL
 *    - /public (no auth), /private (auth required), /admin (admin only)
 *    - Different authentication per namespace
 * 
 * 3. VERSIONING
 *    - /v1, /v2, /beta
 *    - Maintain backward compatibility
 * 
 * 4. TENANT ISOLATION
 *    - Multi-tenant apps: /tenant-1, /tenant-2
 *    - Complete isolation between tenants
 * 
 * 
 * USE ROOMS WHEN:
 * 
 * 1. GROUPING USERS
 *    - Chat rooms within chat app
 *    - Game lobbies within game app
 *    - Team channels within workspace
 * 
 * 2. TARGETING BROADCASTS
 *    - Send to specific group of users
 *    - User can be in multiple groups
 * 
 * 3. PERSONAL CHANNELS
 *    - user:123, user:456 for per-user targeting
 *    - Device-specific: device:abc
 * 
 * 4. DYNAMIC GROUPS
 *    - Groups created/destroyed on demand
 *    - Temporary groupings
 */

// NAMESPACE EXAMPLE: Feature separation
@WebSocketGateway({ namespace: '/chat' })   // Chat feature
@WebSocketGateway({ namespace: '/game' })   // Game feature
@WebSocketGateway({ namespace: '/video' })  // Video feature

// ROOM EXAMPLE: Groups within chat
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  // Rooms: room-1, room-2, room-3, user:123, team:5, etc.
  // All within /chat namespace
}
```

### **Combined Example: Namespaces + Rooms**

```typescript
// Real-world: Use BOTH together

// NAMESPACE: /chat (for chat feature)
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    
    // JOIN ROOMS within /chat namespace
    client.join(`user:${userId}`);  // Personal room
    client.join('general');         // General room
    
    console.log(
      `User ${userId} connected to /chat namespace ` +
      `and joined 2 rooms`,
    );
  }

  @SubscribeMessage('join-chat-room')
  handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Join room within /chat namespace
    client.join(roomId);
    
    // Broadcast to room in /chat namespace
    client.to(roomId).emit('user-joined-room', {
      userId: client.handshake.auth.userId,
      room: roomId,
    });
  }

  @SubscribeMessage('send-message')
  handleMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    // Emit to room in /chat namespace
    this.server.to(data.roomId).emit('new-message', {
      message: data.message,
      from: client.id,
    });
  }
}

// NAMESPACE: /game (for game feature)
@WebSocketGateway({ namespace: '/game' })
export class GameGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('join-game-lobby')
  handleJoinLobby(
    @MessageBody() lobbyId: string,
    @ConnectedSocket() client: Socket,
  ) {
    // Join room within /game namespace
    client.join(`lobby:${lobbyId}`);
    
    // Broadcast to lobby in /game namespace
    client.to(`lobby:${lobbyId}`).emit('player-joined', {
      playerId: client.id,
    });
  }
}

// CLIENT: Connects to both namespaces
const chatSocket = io('http://localhost:3000/chat');
const gameSocket = io('http://localhost:3000/game');

// Join rooms in each namespace
chatSocket.emit('join-chat-room', 'room-1');  // Room in /chat
gameSocket.emit('join-game-lobby', 'lobby-5'); // Room in /game

// Rooms are namespace-specific:
// 'room-1' in /chat !== 'room-1' in /game
```

### **Key Differences Illustrated**

```typescript
// NAMESPACE: Client must explicitly connect
const chatSocket = io('/chat');  // ← Client code REQUIRED
const gameSocket = io('/game');  // ← Client code REQUIRED

// ROOM: No client code needed
const socket = io('/chat');      // Connect to namespace
// Server-side:
// socket.join('room-1');        // ← Server decides rooms
// socket.join('room-2');        // ← No client code needed

// NAMESPACE: Separate event handlers
@WebSocketGateway({ namespace: '/chat' })
class ChatGateway {
  @SubscribeMessage('message')  // Only for /chat
  handleChatMessage() {}
}

@WebSocketGateway({ namespace: '/game' })
class GameGateway {
  @SubscribeMessage('message')  // Only for /game
  handleGameMessage() {}        // Different handler!
}

// ROOM: Same event handlers
@WebSocketGateway({ namespace: '/chat' })
class ChatGateway {
  @SubscribeMessage('message')  // For ALL rooms in /chat
  handleMessage(               // Same handler!
    @MessageBody() data,
    @ConnectedSocket() client,
  ) {
    // Can broadcast to any room
    this.server.to('room-1').emit('msg', data);
    this.server.to('room-2').emit('msg', data);
  }
}

// NAMESPACE: Independent middleware
@WebSocketGateway({ namespace: '/admin' })
class AdminGateway {
  afterInit(server: Server) {
    server.use((socket, next) => {
      // Middleware ONLY for /admin
      if (socket.handshake.auth.role !== 'admin') {
        return next(new Error('Not admin'));
      }
      next();
    });
  }
}

// ROOM: No room-specific middleware
// Middleware applies to entire namespace
```

### **Isolation Comparison**

```typescript
// NAMESPACE ISOLATION: Complete
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  broadcast() {
    // Emits ONLY to /chat namespace
    this.server.emit('update', { data: 'chat update' });
    // /game and /admin DO NOT receive this
  }
}

// ROOM ISOLATION: Within namespace only
@WebSocketGateway({ namespace: '/chat' })
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  broadcastToRoom() {
    // Emits ONLY to room-1 in /chat namespace
    this.server.to('room-1').emit('update', { data: 'room update' });
    // room-2 in /chat does NOT receive this
    // But all sockets in room-1 DO receive it
  }
}
```

### **Performance Comparison**

```typescript
/**
 * NAMESPACES:
 * - Each namespace = separate event handlers
 * - More namespaces = more overhead
 * - Good for: < 10 namespaces
 * - Use when: features are truly separate
 * 
 * ROOMS:
 * - Lightweight (just group membership)
 * - Can have thousands of rooms
 * - Good for: unlimited rooms
 * - Use when: grouping users dynamically
 */

// NAMESPACE: More overhead
@WebSocketGateway({ namespace: '/feature1' })  // Separate gateway
@WebSocketGateway({ namespace: '/feature2' })  // Separate gateway
@WebSocketGateway({ namespace: '/feature3' })  // Separate gateway
// Each has its own event loop, handlers, etc.

// ROOM: Lightweight
@WebSocketGateway({ namespace: '/app' })
export class AppGateway {
  // Can have thousands of rooms with minimal overhead
  // room-1, room-2, ..., room-9999
  // user:1, user:2, ..., user:9999
}
```

### **Production Decision Matrix**

```typescript
/**
 * DECISION MATRIX:
 * 
 * Use NAMESPACE if:
 * ✓ Features are completely separate (chat vs game vs admin)
 * ✓ Need different authentication per feature
 * ✓ Want independent middleware
 * ✓ Different teams maintain different features
 * ✓ Need versioning (/v1, /v2)
 * 
 * Use ROOMS if:
 * ✓ Grouping users within same feature
 * ✓ Dynamic groups (created/destroyed on demand)
 * ✓ User can be in multiple groups
 * ✓ Need efficient broadcasting to subsets
 * ✓ Per-user targeting (user:123)
 * 
 * Use BOTH if:
 * ✓ Large app with multiple features (namespaces)
 * ✓ Each feature needs user grouping (rooms)
 * ✓ Example: /chat with chat-rooms, /game with game-lobbies
 */

// GOOD ARCHITECTURE:
@WebSocketGateway({ namespace: '/chat' })     // Feature = namespace
export class ChatGateway {
  // Chat rooms, user rooms, team rooms = rooms
}

@WebSocketGateway({ namespace: '/game' })     // Feature = namespace
export class GameGateway {
  // Game lobbies, match rooms = rooms
}

@WebSocketGateway({ namespace: '/notifications' }) // Feature = namespace
export class NotificationsGateway {
  // User-specific channels = rooms
}
```

**Interview Tip**: **Namespaces** = **separate channels** for **different features** (`/chat`, `/game`, `/admin`), **client explicitly connects** (`io('/chat')`), each has **independent event handlers**, **middleware**, **rooms**. **Rooms** = **groups within namespace** for **broadcasting** to subsets, **server-side only** (no client connection), **client doesn't know** which rooms it's in, **lightweight** (can have thousands). **Key difference**: Namespaces = **application-level separation** (different apps/features), Rooms = **grouping within feature** (chat rooms within chat app). **Use namespaces** for: feature separation, access control (`/admin`), versioning (`/v1`, `/v2`). **Use rooms** for: grouping users (chat rooms), per-user targeting (`user:123`), team channels. **Can use both**: namespace `/chat` with rooms `room-1`, `room-2` inside it. **Rooms are namespace-specific**: `room-1` in `/chat` ≠ `room-1` in `/game`. **Socket can**: connect to **one namespace** at a time (per connection), be in **multiple rooms** simultaneously. **Auto-cleanup**: rooms auto-deleted when empty, namespaces persist.

</details>

## Authentication

25. How do you implement authentication in WebSockets?

<details>
<summary><strong>Answer</strong></summary>

Authentication in WebSockets is implemented using **middleware** that validates credentials from the **handshake** (`socket.handshake.auth` or `socket.handshake.query`). The most common approach is sending a **JWT token** during connection, validating it in middleware in the `afterInit()` hook, and either allowing the connection (`next()`) or rejecting it (`next(new Error())`). Unlike HTTP, WebSocket authentication happens **once during handshake**, not on every event.

### **Basic Authentication Flow**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  OnGatewayInit,
  OnGatewayConnection,
  ConnectedSocket,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { UnauthorizedException } from '@nestjs/common';

@WebSocketGateway()
export class AuthGateway implements OnGatewayInit, OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  // STEP 1: Setup authentication middleware
  afterInit(server: Server) {
    server.use((socket, next) => {
      try {
        // Get token from handshake
        const token = socket.handshake.auth.token;

        if (!token) {
          throw new UnauthorizedException('No token provided');
        }

        // Validate token (simplified)
        const user = this.validateToken(token);

        if (!user) {
          throw new UnauthorizedException('Invalid token');
        }

        // Attach user to socket
        socket.data.user = user;

        // Allow connection
        next();
      } catch (error) {
        // Reject connection
        next(new Error('Authentication failed'));
      }
    });

    console.log('WebSocket authentication middleware initialized');
  }

  // STEP 2: Connection is now authenticated
  handleConnection(client: Socket) {
    // User is authenticated if they reach here
    const user = client.data.user;
    console.log(`Authenticated user ${user.id} connected`);
  }

  private validateToken(token: string): any {
    // Token validation logic (see Q26 for JWT details)
    return { id: '123', username: 'john' };
  }
}

// CLIENT: Send token during connection
const socket = io('http://localhost:3000', {
  auth: {
    token: 'your-jwt-token-here',
  },
});
```

### **JWT-Based Authentication**

```typescript
import { JwtService } from '@nestjs/jwt';
import { WsException } from '@nestjs/websockets';

@WebSocketGateway({ cors: { origin: '*' } })
export class JwtAuthGateway implements OnGatewayInit {
  @WebSocketServer()
  server: Server;

  constructor(private jwtService: JwtService) {}

  afterInit(server: Server) {
    server.use((socket, next) => {
      try {
        // METHOD 1: Token in auth object
        const token = socket.handshake.auth.token;

        // METHOD 2: Token in query params (fallback)
        // const token = socket.handshake.query.token as string;

        // METHOD 3: Token in headers (if available)
        // const token = socket.handshake.headers.authorization?.split(' ')[1];

        if (!token) {
          return next(new Error('Authentication error: No token'));
        }

        // Verify JWT token
        const payload = this.jwtService.verify(token);

        // Attach user data to socket
        socket.data.user = {
          userId: payload.sub,
          username: payload.username,
          email: payload.email,
          role: payload.role,
        };

        // Allow connection
        next();
      } catch (error) {
        console.error('WebSocket auth error:', error.message);
        next(new Error('Authentication failed'));
      }
    });
  }

  handleConnection(client: Socket) {
    const user = client.data.user;
    console.log(
      `User ${user.username} (${user.userId}) connected with role: ${user.role}`,
    );

    // Join personal room
    client.join(`user:${user.userId}`);
  }
}
```

### **Authentication Module**

```typescript
// ws-auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { AuthGateway } from './auth.gateway';

@Module({
  imports: [
    JwtModule.register({
      secret: process.env.JWT_SECRET || 'your-secret-key',
      signOptions: { expiresIn: '1d' },
    }),
  ],
  providers: [AuthGateway],
})
export class WebSocketAuthModule {}
```

### **Multiple Authentication Methods**

```typescript
@WebSocketGateway()
export class MultiAuthGateway implements OnGatewayInit {
  constructor(
    private jwtService: JwtService,
    private authService: AuthService,
  ) {}

  afterInit(server: Server) {
    server.use(async (socket, next) => {
      try {
        // METHOD 1: JWT Token
        const token = socket.handshake.auth.token;
        if (token) {
          const user = await this.validateJWT(token);
          if (user) {
            socket.data.user = user;
            return next();
          }
        }

        // METHOD 2: Session ID (from cookies)
        const sessionId = socket.handshake.auth.sessionId;
        if (sessionId) {
          const user = await this.validateSession(sessionId);
          if (user) {
            socket.data.user = user;
            return next();
          }
        }

        // METHOD 3: API Key
        const apiKey = socket.handshake.auth.apiKey;
        if (apiKey) {
          const user = await this.validateApiKey(apiKey);
          if (user) {
            socket.data.user = user;
            return next();
          }
        }

        // No valid authentication found
        throw new Error('No valid authentication provided');
      } catch (error) {
        next(new Error('Authentication failed'));
      }
    });
  }

  private async validateJWT(token: string) {
    try {
      const payload = this.jwtService.verify(token);
      return await this.authService.getUserById(payload.sub);
    } catch {
      return null;
    }
  }

  private async validateSession(sessionId: string) {
    return await this.authService.getUserBySession(sessionId);
  }

  private async validateApiKey(apiKey: string) {
    return await this.authService.getUserByApiKey(apiKey);
  }
}
```

### **Client-Side Authentication**

```typescript
// client-auth.ts
import { io } from 'socket.io-client';

// METHOD 1: Token in auth object (RECOMMENDED)
const socket1 = io('http://localhost:3000', {
  auth: {
    token: localStorage.getItem('jwt_token'),
  },
});

// METHOD 2: Token in query params
const token = localStorage.getItem('jwt_token');
const socket2 = io('http://localhost:3000', {
  query: {
    token,
  },
});

// METHOD 3: Multiple auth data
const socket3 = io('http://localhost:3000', {
  auth: {
    token: 'jwt-token',
    userId: '123',
    username: 'john',
  },
});

// Handle authentication errors
socket1.on('connect_error', (error) => {
  console.error('Connection failed:', error.message);
  
  if (error.message === 'Authentication failed') {
    // Redirect to login or refresh token
    window.location.href = '/login';
  }
});

// Reconnect with new token
function reconnectWithNewToken(newToken: string) {
  socket1.auth = { token: newToken };
  socket1.connect();
}
```

### **Token Refresh Pattern**

```typescript
@WebSocketGateway()
export class TokenRefreshGateway implements OnGatewayInit {
  constructor(
    private jwtService: JwtService,
    private authService: AuthService,
  ) {}

  afterInit(server: Server) {
    server.use(async (socket, next) => {
      try {
        const token = socket.handshake.auth.token;
        const refreshToken = socket.handshake.auth.refreshToken;

        // Try access token first
        try {
          const payload = this.jwtService.verify(token);
          socket.data.user = await this.authService.getUserById(payload.sub);
          return next();
        } catch (error) {
          // Access token expired, try refresh token
          if (refreshToken) {
            const newTokens = await this.authService.refreshTokens(
              refreshToken,
            );
            
            if (newTokens) {
              // Send new tokens to client
              socket.emit('tokens-refreshed', newTokens);
              
              socket.data.user = await this.authService.getUserById(
                newTokens.userId,
              );
              return next();
            }
          }
        }

        throw new Error('Invalid or expired token');
      } catch (error) {
        next(new Error('Authentication failed'));
      }
    });
  }
}

// CLIENT: Handle token refresh
socket.on('tokens-refreshed', (tokens) => {
  localStorage.setItem('jwt_token', tokens.accessToken);
  localStorage.setItem('refresh_token', tokens.refreshToken);
  
  // Update socket auth
  socket.auth = { token: tokens.accessToken };
});
```

### **Role-Based Access Control**

```typescript
@WebSocketGateway()
export class RBACGateway implements OnGatewayInit {
  afterInit(server: Server) {
    server.use(async (socket, next) => {
      try {
        const token = socket.handshake.auth.token;
        const payload = this.jwtService.verify(token);

        // Validate user exists
        const user = await this.authService.getUserById(payload.sub);
        if (!user) {
          throw new Error('User not found');
        }

        // Attach full user data including roles
        socket.data.user = {
          id: user.id,
          username: user.username,
          email: user.email,
          roles: user.roles,  // ['user', 'admin', 'moderator']
          permissions: user.permissions,
        };

        next();
      } catch (error) {
        next(new Error('Authentication failed'));
      }
    });
  }

  handleConnection(client: Socket) {
    const user = client.data.user;
    
    // Join role-based rooms
    user.roles.forEach(role => {
      client.join(`role:${role}`);
    });

    console.log(
      `User ${user.username} connected with roles: ${user.roles.join(', ')}`,
    );
  }
}
```

### **Namespace-Specific Authentication**

```typescript
// public.gateway.ts - No auth required
@WebSocketGateway({ namespace: '/public' })
export class PublicGateway {
  // No authentication middleware
  handleConnection(client: Socket) {
    console.log('Anonymous user connected to /public');
  }
}

// auth.gateway.ts - Auth required
@WebSocketGateway({ namespace: '/private' })
export class PrivateGateway implements OnGatewayInit {
  constructor(private jwtService: JwtService) {}

  afterInit(server: Server) {
    // Authentication ONLY for /private namespace
    server.use((socket, next) => {
      try {
        const token = socket.handshake.auth.token;
        const payload = this.jwtService.verify(token);
        socket.data.user = payload;
        next();
      } catch (error) {
        next(new Error('Authentication required for /private'));
      }
    });
  }

  handleConnection(client: Socket) {
    console.log('Authenticated user connected to /private');
  }
}

// admin.gateway.ts - Admin auth required
@WebSocketGateway({ namespace: '/admin' })
export class AdminGateway implements OnGatewayInit {
  constructor(private jwtService: JwtService) {}

  afterInit(server: Server) {
    server.use((socket, next) => {
      try {
        const token = socket.handshake.auth.token;
        const payload = this.jwtService.verify(token);

        // Check admin role
        if (payload.role !== 'admin') {
          throw new Error('Admin access required');
        }

        socket.data.user = payload;
        next();
      } catch (error) {
        next(new Error('Admin authentication failed'));
      }
    });
  }

  handleConnection(client: Socket) {
    console.log('Admin connected to /admin');
  }
}
```

### **Authentication with Database Validation**

```typescript
@WebSocketGateway()
export class DatabaseAuthGateway implements OnGatewayInit {
  constructor(
    private jwtService: JwtService,
    private userService: UserService,
  ) {}

  afterInit(server: Server) {
    server.use(async (socket, next) => {
      try {
        const token = socket.handshake.auth.token;

        // Verify JWT
        const payload = this.jwtService.verify(token);

        // VALIDATE USER IN DATABASE
        const user = await this.userService.findById(payload.sub);

        // Check if user exists
        if (!user) {
          throw new Error('User not found');
        }

        // Check if user is active
        if (!user.isActive) {
          throw new Error('User account is disabled');
        }

        // Check if token is blacklisted
        const isBlacklisted = await this.userService.isTokenBlacklisted(
          token,
        );
        if (isBlacklisted) {
          throw new Error('Token is blacklisted');
        }

        // Attach full user data
        socket.data.user = user;

        next();
      } catch (error) {
        next(new Error(`Authentication failed: ${error.message}`));
      }
    });
  }
}
```

### **Production Authentication Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  OnGatewayInit,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

@WebSocketGateway({
  cors: {
    origin: process.env.FRONTEND_URL,
    credentials: true,
  },
})
export class ProductionAuthGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  @WebSocketServer()
  server: Server;

  private logger = new Logger('WebSocketAuth');

  constructor(
    private jwtService: JwtService,
    private userService: UserService,
    private sessionService: SessionService,
  ) {}

  afterInit(server: Server) {
    server.use(async (socket, next) => {
      const startTime = Date.now();

      try {
        // Get token from auth or query
        const token =
          socket.handshake.auth.token ||
          socket.handshake.query.token as string;

        if (!token) {
          this.logger.warn('Connection attempt without token');
          return next(new Error('No authentication token provided'));
        }

        // Verify JWT
        let payload;
        try {
          payload = this.jwtService.verify(token, {
            secret: process.env.JWT_SECRET,
          });
        } catch (error) {
          this.logger.warn(`Invalid token: ${error.message}`);
          return next(new Error('Invalid or expired token'));
        }

        // Get user from database
        const user = await this.userService.findById(payload.sub);

        if (!user) {
          this.logger.warn(`User not found: ${payload.sub}`);
          return next(new Error('User not found'));
        }

        if (!user.isActive) {
          this.logger.warn(`Inactive user tried to connect: ${user.id}`);
          return next(new Error('Account is disabled'));
        }

        // Create session
        await this.sessionService.create({
          userId: user.id,
          socketId: socket.id,
          connectedAt: new Date(),
        });

        // Attach user data to socket
        socket.data.user = {
          id: user.id,
          username: user.username,
          email: user.email,
          roles: user.roles,
        };

        const duration = Date.now() - startTime;
        this.logger.log(
          `User ${user.username} authenticated in ${duration}ms`,
        );

        next();
      } catch (error) {
        this.logger.error('Authentication error:', error);
        next(new Error('Authentication failed'));
      }
    });

    this.logger.log('WebSocket authentication initialized');
  }

  async handleConnection(client: Socket) {
    const user = client.data.user;

    // Join personal room
    client.join(`user:${user.id}`);

    // Join role rooms
    user.roles.forEach(role => client.join(`role:${role}`));

    // Notify user
    client.emit('authenticated', {
      userId: user.id,
      username: user.username,
      roles: user.roles,
    });

    this.logger.log(`User ${user.username} connected (${client.id})`);
  }

  async handleDisconnect(client: Socket) {
    const user = client.data.user;

    if (user) {
      // Clean up session
      await this.sessionService.deleteBySocketId(client.id);

      this.logger.log(`User ${user.username} disconnected (${client.id})`);
    }
  }
}
```

**Interview Tip**: **Authentication** in WebSockets uses **middleware** in `afterInit()` hook: `server.use((socket, next) => { ... })`. **Client sends** credentials via `io(url, { auth: { token } })` or `query: { token }`. **Middleware** validates token from `socket.handshake.auth.token` or `socket.handshake.query.token`, **verifies JWT** with `jwtService.verify(token)`, **attaches user** to `socket.data.user`, and calls `next()` to **allow** or `next(new Error())` to **reject**. **Authentication happens once** during handshake, not per event. **Common approaches**: (1) JWT token (most common), (2) session ID, (3) API key. **After auth**: user data available in `client.data.user` in all handlers. **Namespace-specific**: each namespace can have different auth middleware. **Production**: validate token, check user exists in DB, verify account active, check token not blacklisted. **Client error**: handle `connect_error` event for auth failures. **Token refresh**: send new tokens via `socket.emit('tokens-refreshed', tokens)` after validating refresh token.

</details>
26. How do you access JWT tokens in WebSocket connections?

<details>
<summary><strong>Answer</strong></summary>

JWT tokens in WebSocket connections are accessed from the **handshake** object during authentication middleware. The token is typically sent by the client via **`socket.handshake.auth.token`** (recommended), **`socket.handshake.query.token`** (query parameter), or **`socket.handshake.headers.authorization`** (Bearer token). Once verified with `jwtService.verify(token)`, the decoded payload is attached to **`socket.data.user`** for use in all event handlers.

### **Token Access Methods**

```typescript
import { JwtService } from '@nestjs/jwt';
import {
  WebSocketGateway,
  WebSocketServer,
  OnGatewayInit,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway()
export class TokenAccessGateway implements OnGatewayInit {
  constructor(private jwtService: JwtService) {}

  afterInit(server: Server) {
    server.use((socket, next) => {
      // METHOD 1: From auth object (RECOMMENDED)
      const tokenFromAuth = socket.handshake.auth.token;
      console.log('Token from auth:', tokenFromAuth);

      // METHOD 2: From query parameters
      const tokenFromQuery = socket.handshake.query.token as string;
      console.log('Token from query:', tokenFromQuery);

      // METHOD 3: From headers (Authorization: Bearer <token>)
      const authHeader = socket.handshake.headers.authorization;
      const tokenFromHeader = authHeader?.split(' ')[1];
      console.log('Token from header:', tokenFromHeader);

      // Use first available token
      const token = tokenFromAuth || tokenFromQuery || tokenFromHeader;

      if (!token) {
        return next(new Error('No token provided'));
      }

      try {
        // Verify and decode token
        const payload = this.jwtService.verify(token);
        
        // Attach to socket
        socket.data.user = payload;
        
        next();
      } catch (error) {
        next(new Error('Invalid token'));
      }
    });
  }
}
```

### **JWT Verification and Decoding**

```typescript
import { JwtService } from '@nestjs/jwt';

@WebSocketGateway()
export class JwtVerificationGateway implements OnGatewayInit {
  constructor(private jwtService: JwtService) {}

  afterInit(server: Server) {
    server.use((socket, next) => {
      try {
        const token = socket.handshake.auth.token;

        if (!token) {
          throw new Error('No token');
        }

        // VERIFY TOKEN with secret
        const payload = this.jwtService.verify(token, {
          secret: process.env.JWT_SECRET,
        });

        console.log('Decoded JWT payload:', payload);
        /**
         * Typical payload structure:
         * {
         *   sub: '123',              // User ID (standard claim)
         *   username: 'john_doe',
         *   email: 'john@example.com',
         *   role: 'admin',
         *   iat: 1234567890,         // Issued at (timestamp)
         *   exp: 1234654290,         // Expiration (timestamp)
         * }
         */

        // Attach user data to socket
        socket.data.user = {
          userId: payload.sub,        // Standard: sub = subject (user ID)
          username: payload.username,
          email: payload.email,
          role: payload.role,
        };

        // Token is valid
        next();
      } catch (error) {
        if (error.name === 'TokenExpiredError') {
          return next(new Error('Token expired'));
        }
        if (error.name === 'JsonWebTokenError') {
          return next(new Error('Invalid token'));
        }
        next(new Error('Authentication failed'));
      }
    });
  }

  handleConnection(client: Socket) {
    // Access user data from socket
    const user = client.data.user;
    console.log(`User ${user.username} (ID: ${user.userId}) connected`);
  }
}
```

### **Client-Side: Sending JWT Token**

```typescript
// client.ts
import { io } from 'socket.io-client';

// Get token from storage
const token = localStorage.getItem('jwt_token');

// METHOD 1: Send in auth object (RECOMMENDED)
const socket1 = io('http://localhost:3000', {
  auth: {
    token: token,
  },
});
// Server accesses: socket.handshake.auth.token

// METHOD 2: Send in query parameters
const socket2 = io('http://localhost:3000', {
  query: {
    token: token,
  },
});
// Server accesses: socket.handshake.query.token

// METHOD 3: Send in headers (extraHeaders)
const socket3 = io('http://localhost:3000', {
  extraHeaders: {
    Authorization: `Bearer ${token}`,
  },
});
// Server accesses: socket.handshake.headers.authorization

// METHOD 4: Send during reconnection
const socket4 = io('http://localhost:3000', {
  auth: (cb) => {
    // Fetch token dynamically
    const freshToken = localStorage.getItem('jwt_token');
    cb({ token: freshToken });
  },
});

// Update token for reconnection
function updateToken(newToken: string) {
  socket1.auth = { token: newToken };
  socket1.connect();
}
```

### **Using @nestjs/jwt Module**

```typescript
// jwt.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { JwtAuthGateway } from './jwt-auth.gateway';

@Module({
  imports: [
    ConfigModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        secret: configService.get<string>('JWT_SECRET'),
        signOptions: {
          expiresIn: configService.get<string>('JWT_EXPIRATION', '1d'),
        },
      }),
    }),
  ],
  providers: [JwtAuthGateway],
})
export class WebSocketJwtModule {}

// jwt-auth.gateway.ts
import { JwtService } from '@nestjs/jwt';

@WebSocketGateway()
export class JwtAuthGateway implements OnGatewayInit {
  constructor(private jwtService: JwtService) {}

  afterInit(server: Server) {
    server.use((socket, next) => {
      try {
        const token = socket.handshake.auth.token;

        // Verify with injected JwtService
        const payload = this.jwtService.verify(token);

        socket.data.user = payload;
        next();
      } catch (error) {
        next(new Error('Invalid token'));
      }
    });
  }
}
```

### **Accessing User Data in Event Handlers**

```typescript
@WebSocketGateway()
export class UserDataGateway {
  @SubscribeMessage('send-message')
  handleMessage(
    @MessageBody() data: { message: string },
    @ConnectedSocket() client: Socket,
  ) {
    // ACCESS USER DATA from socket
    const user = client.data.user;

    console.log(`Message from user ${user.userId}: ${data.message}`);

    return {
      event: 'message-received',
      data: {
        from: user.username,
        message: data.message,
        timestamp: new Date(),
      },
    };
  }

  @SubscribeMessage('get-profile')
  handleGetProfile(@ConnectedSocket() client: Socket) {
    const user = client.data.user;

    return {
      event: 'profile',
      data: {
        userId: user.userId,
        username: user.username,
        email: user.email,
        role: user.role,
      },
    };
  }
}
```

### **Token Validation with Database**

```typescript
@WebSocketGateway()
export class DatabaseTokenGateway implements OnGatewayInit {
  constructor(
    private jwtService: JwtService,
    private userService: UserService,
  ) {}

  afterInit(server: Server) {
    server.use(async (socket, next) => {
      try {
        const token = socket.handshake.auth.token;

        // STEP 1: Verify JWT signature and expiration
        const payload = this.jwtService.verify(token);
        console.log('JWT verified:', payload);

        // STEP 2: Validate user exists in database
        const user = await this.userService.findById(payload.sub);

        if (!user) {
          return next(new Error('User not found'));
        }

        if (!user.isActive) {
          return next(new Error('User is not active'));
        }

        // STEP 3: Check token blacklist (optional)
        const isBlacklisted = await this.userService.isTokenBlacklisted(
          token,
        );

        if (isBlacklisted) {
          return next(new Error('Token is blacklisted'));
        }

        // STEP 4: Attach full user data (not just JWT payload)
        socket.data.user = {
          id: user.id,
          username: user.username,
          email: user.email,
          role: user.role,
          permissions: user.permissions,
          profilePicture: user.profilePicture,
        };

        next();
      } catch (error) {
        next(new Error(`Token validation failed: ${error.message}`));
      }
    });
  }
}
```

### **Token Refresh Pattern**

```typescript
@WebSocketGateway()
export class TokenRefreshGateway implements OnGatewayInit {
  constructor(
    private jwtService: JwtService,
    private authService: AuthService,
  ) {}

  afterInit(server: Server) {
    server.use(async (socket, next) => {
      const accessToken = socket.handshake.auth.token;
      const refreshToken = socket.handshake.auth.refreshToken;

      try {
        // TRY: Verify access token
        const payload = this.jwtService.verify(accessToken);
        socket.data.user = payload;
        return next();
      } catch (accessError) {
        // CATCH: Access token expired or invalid

        if (!refreshToken) {
          return next(new Error('Token expired and no refresh token'));
        }

        try {
          // Verify refresh token
          const refreshPayload = this.jwtService.verify(refreshToken, {
            secret: process.env.JWT_REFRESH_SECRET,
          });

          // Generate new access token
          const newAccessToken = this.jwtService.sign(
            {
              sub: refreshPayload.sub,
              username: refreshPayload.username,
              email: refreshPayload.email,
            },
            {
              secret: process.env.JWT_SECRET,
              expiresIn: '15m',
            },
          );

          // Send new token to client
          socket.emit('token-refreshed', {
            accessToken: newAccessToken,
          });

          // Update socket auth
          socket.handshake.auth.token = newAccessToken;
          socket.data.user = refreshPayload;

          next();
        } catch (refreshError) {
          return next(new Error('Refresh token invalid or expired'));
        }
      }
    });
  }
}

// CLIENT: Handle token refresh
socket.on('token-refreshed', ({ accessToken }) => {
  // Save new token
  localStorage.setItem('jwt_token', accessToken);
  
  // Update socket auth for reconnections
  socket.auth.token = accessToken;
  
  console.log('Token refreshed automatically');
});
```

### **Multiple Token Types**

```typescript
@WebSocketGateway()
export class MultiTokenGateway implements OnGatewayInit {
  constructor(
    private jwtService: JwtService,
    private apiKeyService: ApiKeyService,
  ) {}

  afterInit(server: Server) {
    server.use(async (socket, next) => {
      const handshake = socket.handshake;

      // TOKEN TYPE 1: JWT from auth
      const jwtToken = handshake.auth.token;
      if (jwtToken) {
        try {
          const payload = this.jwtService.verify(jwtToken);
          socket.data.user = payload;
          socket.data.authType = 'jwt';
          return next();
        } catch {}
      }

      // TOKEN TYPE 2: API Key from headers
      const apiKey = handshake.headers['x-api-key'];
      if (apiKey) {
        const user = await this.apiKeyService.validateKey(apiKey as string);
        if (user) {
          socket.data.user = user;
          socket.data.authType = 'api-key';
          return next();
        }
      }

      // TOKEN TYPE 3: Session ID from cookies
      const sessionId = handshake.headers.cookie
        ?.split('; ')
        .find(c => c.startsWith('sessionId='))
        ?.split('=')[1];

      if (sessionId) {
        const user = await this.apiKeyService.validateSession(sessionId);
        if (user) {
          socket.data.user = user;
          socket.data.authType = 'session';
          return next();
        }
      }

      next(new Error('No valid authentication'));
    });
  }

  handleConnection(client: Socket) {
    const user = client.data.user;
    const authType = client.data.authType;
    console.log(
      `User ${user.username} connected via ${authType}`,
    );
  }
}
```

### **Token from Different Sources (Fallback)**

```typescript
@WebSocketGateway()
export class FallbackTokenGateway implements OnGatewayInit {
  constructor(private jwtService: JwtService) {}

  afterInit(server: Server) {
    server.use((socket, next) => {
      // Try multiple sources
      let token: string | undefined;

      // 1. auth.token (recommended)
      token = socket.handshake.auth.token;

      // 2. query.token (fallback)
      if (!token) {
        token = socket.handshake.query.token as string;
      }

      // 3. Authorization header (Bearer)
      if (!token) {
        const authHeader = socket.handshake.headers.authorization;
        token = authHeader?.startsWith('Bearer ')
          ? authHeader.substring(7)
          : undefined;
      }

      // 4. Cookie (last resort)
      if (!token) {
        const cookies = socket.handshake.headers.cookie;
        token = cookies
          ?.split('; ')
          .find(c => c.startsWith('jwt='))
          ?.split('=')[1];
      }

      if (!token) {
        return next(new Error('No token found in any source'));
      }

      try {
        const payload = this.jwtService.verify(token);
        socket.data.user = payload;
        next();
      } catch (error) {
        next(new Error('Invalid token'));
      }
    });
  }
}
```

### **Production Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  OnGatewayInit,
  OnGatewayConnection,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { ConfigService } from '@nestjs/config';

@WebSocketGateway({
  cors: { origin: '*' },
})
export class ProductionJwtGateway
  implements OnGatewayInit, OnGatewayConnection
{
  @WebSocketServer()
  server: Server;

  private logger = new Logger('JwtAuth');

  constructor(
    private jwtService: JwtService,
    private configService: ConfigService,
    private userService: UserService,
  ) {}

  afterInit(server: Server) {
    server.use(async (socket, next) => {
      const startTime = Date.now();

      try {
        // GET TOKEN from preferred source
        const token =
          socket.handshake.auth.token ||
          socket.handshake.query.token as string;

        if (!token) {
          this.logger.warn(
            `Connection attempt without token from ${socket.handshake.address}`,
          );
          return next(new Error('No token provided'));
        }

        // VERIFY JWT
        let payload;
        try {
          payload = this.jwtService.verify(token, {
            secret: this.configService.get<string>('JWT_SECRET'),
          });
        } catch (error) {
          this.logger.warn(`Invalid token: ${error.message}`);
          
          if (error.name === 'TokenExpiredError') {
            return next(new Error('Token expired'));
          }
          return next(new Error('Invalid token'));
        }

        // VALIDATE USER
        const user = await this.userService.findById(payload.sub);

        if (!user) {
          this.logger.warn(`User not found: ${payload.sub}`);
          return next(new Error('User not found'));
        }

        if (!user.isActive) {
          this.logger.warn(`Inactive user: ${user.id}`);
          return next(new Error('Account disabled'));
        }

        // ATTACH USER DATA
        socket.data.user = {
          id: user.id,
          username: user.username,
          email: user.email,
          role: user.role,
          permissions: user.permissions,
        };

        const duration = Date.now() - startTime;
        this.logger.log(
          `User ${user.username} authenticated via JWT in ${duration}ms`,
        );

        next();
      } catch (error) {
        this.logger.error('JWT authentication error:', error);
        next(new Error('Authentication failed'));
      }
    });

    this.logger.log('JWT authentication middleware initialized');
  }

  handleConnection(client: Socket) {
    const user = client.data.user;

    // Join user's personal room
    client.join(`user:${user.id}`);

    // Emit authentication success
    client.emit('authenticated', {
      userId: user.id,
      username: user.username,
      role: user.role,
    });

    this.logger.log(
      `User ${user.username} connected (${client.id})`,
    );
  }
}
```

**Interview Tip**: **JWT tokens** accessed from **handshake** during authentication middleware. **Three common sources**: (1) **`socket.handshake.auth.token`** (recommended - client sends via `io(url, { auth: { token } })`), (2) **`socket.handshake.query.token`** (query param - client sends via `query: { token }`), (3) **`socket.handshake.headers.authorization`** (Bearer token - split by space, take second part). **Verify** with **`jwtService.verify(token)`** - returns **decoded payload** (contains `sub` (user ID), `username`, `email`, `role`, `iat`, `exp`). **Attach to socket**: `socket.data.user = payload` - available in all handlers via `client.data.user`. **Error handling**: catch `TokenExpiredError` (expired), `JsonWebTokenError` (invalid). **Module setup**: `JwtModule.register({ secret, signOptions: { expiresIn } })` in module imports, inject `JwtService` in gateway constructor. **Production**: (1) verify JWT signature, (2) validate user exists in DB, (3) check account active, (4) check token not blacklisted, (5) attach full user data. **Token refresh**: if access token expired, verify refresh token from `handshake.auth.refreshToken`, generate new access token, emit via `socket.emit('token-refreshed', { accessToken })`, client updates `socket.auth.token`. **Fallback**: try `auth.token` first, then `query.token`, then headers, then cookies.

</details>
27. How do you use Guards with WebSocket Gateways?

<details>
<summary><strong>Answer</strong></summary>

Guards in WebSocket Gateways work similarly to HTTP Guards but use the **`@UseGuards()` decorator** on the **gateway class** (for all events) or **event handler methods** (for specific events). Guards implement **`CanActivate`** interface, receive **`ExecutionContext`**, extract Socket from context with **`context.switchToWs().getClient()`**, and return **`true`** to allow or **`false`** to deny. If denied, **`WsException`** is thrown automatically.

### **Basic Guard Structure**

```typescript
import {
  CanActivate,
  ExecutionContext,
  Injectable,
} from '@nestjs/common';
import { WsException } from '@nestjs/websockets';
import { Socket } from 'socket.io';

@Injectable()
export class WsAuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // Get Socket from context
    const client: Socket = context.switchToWs().getClient();
    
    // Get user from socket (attached during auth middleware)
    const user = client.data.user;

    // CHECK: Is user authenticated?
    if (!user) {
      throw new WsException('Unauthorized');
    }

    // Allow request
    return true;
  }
}
```

### **Applying Guards**

```typescript
import {
  WebSocketGateway,
  SubscribeMessage,
  UseGuards,
  MessageBody,
  ConnectedSocket,
} from '@nestjs/websockets';
import { Socket } from 'socket.io';
import { WsAuthGuard } from './ws-auth.guard';

// METHOD 1: Guard on ENTIRE GATEWAY (all events protected)
@UseGuards(WsAuthGuard)
@WebSocketGateway()
export class ProtectedGateway {
  @SubscribeMessage('protected-event')
  handleProtectedEvent(@MessageBody() data: any) {
    // User is authenticated if they reach here
    return { event: 'response', data: 'Protected data' };
  }

  @SubscribeMessage('another-protected-event')
  handleAnother(@MessageBody() data: any) {
    // Also protected by gateway-level guard
    return { event: 'response', data };
  }
}

// METHOD 2: Guard on SPECIFIC EVENT
@WebSocketGateway()
export class MixedGateway {
  // Public event (no guard)
  @SubscribeMessage('public-event')
  handlePublic(@MessageBody() data: any) {
    return { event: 'public-response', data };
  }

  // Protected event (guard applied)
  @UseGuards(WsAuthGuard)
  @SubscribeMessage('protected-event')
  handleProtected(@MessageBody() data: any) {
    // Only authenticated users can access
    return { event: 'protected-response', data };
  }
}
```

### **Role-Based Guard**

```typescript
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { WsException } from '@nestjs/websockets';
import { Socket } from 'socket.io';

// roles.decorator.ts
import { SetMetadata } from '@nestjs/common';
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

// ws-roles.guard.ts
@Injectable()
export class WsRolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get required roles from decorator
    const requiredRoles = this.reflector.get<string[]>(
      'roles',
      context.getHandler(),
    );

    // No roles required = allow
    if (!requiredRoles) {
      return true;
    }

    // Get socket and user
    const client: Socket = context.switchToWs().getClient();
    const user = client.data.user;

    if (!user) {
      throw new WsException('Unauthorized: No user');
    }

    // Check if user has required role
    const hasRole = requiredRoles.some(role => user.roles?.includes(role));

    if (!hasRole) {
      throw new WsException(
        `Forbidden: Requires role ${requiredRoles.join(' or ')}`,
      );
    }

    return true;
  }
}

// Usage
@WebSocketGateway()
export class RoleBasedGateway {
  // Admin only
  @UseGuards(WsRolesGuard)
  @Roles('admin')
  @SubscribeMessage('admin-command')
  handleAdminCommand(@MessageBody() data: any) {
    return { event: 'admin-response', data };
  }

  // Admin or Moderator
  @UseGuards(WsRolesGuard)
  @Roles('admin', 'moderator')
  @SubscribeMessage('moderate')
  handleModerate(@MessageBody() data: any) {
    return { event: 'moderation-response', data };
  }
}
```

### **JWT Guard for WebSockets**

```typescript
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { WsException } from '@nestjs/websockets';
import { Socket } from 'socket.io';

@Injectable()
export class WsJwtGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  canActivate(context: ExecutionContext): boolean {
    try {
      const client: Socket = context.switchToWs().getClient();
      
      // Get token from handshake
      const token = client.handshake.auth.token;

      if (!token) {
        throw new WsException('No token provided');
      }

      // Verify token
      const payload = this.jwtService.verify(token);

      // Attach user to socket (if not already done)
      if (!client.data.user) {
        client.data.user = payload;
      }

      return true;
    } catch (error) {
      throw new WsException('Invalid token');
    }
  }
}

// Usage
@WebSocketGateway()
export class JwtProtectedGateway {
  @UseGuards(WsJwtGuard)
  @SubscribeMessage('secure-event')
  handleSecureEvent(
    @MessageBody() data: any,
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;
    return { event: 'secure-response', userId: user.sub };
  }
}
```

### **Custom Authorization Guard**

```typescript
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { WsException } from '@nestjs/websockets';
import { Socket } from 'socket.io';

@Injectable()
export class RoomOwnerGuard implements CanActivate {
  constructor(private roomService: RoomService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const client: Socket = context.switchToWs().getClient();
    const data = context.switchToWs().getData();

    const userId = client.data.user?.id;
    const roomId = data.roomId;

    if (!userId || !roomId) {
      throw new WsException('Missing user or room ID');
    }

    // Check if user owns the room
    const isOwner = await this.roomService.isOwner(userId, roomId);

    if (!isOwner) {
      throw new WsException('Not room owner');
    }

    return true;
  }
}

// Usage
@WebSocketGateway()
export class RoomGateway {
  @UseGuards(RoomOwnerGuard)
  @SubscribeMessage('delete-room')
  handleDeleteRoom(@MessageBody() data: { roomId: string }) {
    // Only room owner can delete
    return { event: 'room-deleted', roomId: data.roomId };
  }
}
```

### **Multiple Guards**

```typescript
import { UseGuards } from '@nestjs/common';

// Multiple guards execute in order
@WebSocketGateway()
export class MultiGuardGateway {
  // Execute: WsAuthGuard -> WsRolesGuard
  @UseGuards(WsAuthGuard, WsRolesGuard)
  @Roles('admin')
  @SubscribeMessage('admin-action')
  handleAdminAction(@MessageBody() data: any) {
    // User is authenticated AND has admin role
    return { event: 'action-response', data };
  }
}
```

### **Guard with Execution Context**

```typescript
import { ExecutionContext } from '@nestjs/common';

@Injectable()
export class DetailedGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // Get Socket
    const client: Socket = context.switchToWs().getClient();
    
    // Get event data
    const data = context.switchToWs().getData();
    
    // Get event pattern (event name)
    const pattern = context.switchToWs().getPattern();
    
    // Get handler (method reference)
    const handler = context.getHandler();
    
    // Get class (gateway instance)
    const gatewayClass = context.getClass();

    console.log('Event:', pattern);
    console.log('Data:', data);
    console.log('User:', client.data.user);

    // Perform authorization logic
    return this.authorize(client, data, pattern);
  }

  private authorize(client: Socket, data: any, event: string): boolean {
    // Custom authorization logic
    return true;
  }
}
```

### **Throttle Guard (Rate Limiting)**

```typescript
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { WsException } from '@nestjs/websockets';
import { Socket } from 'socket.io';

@Injectable()
export class WsThrottleGuard implements CanActivate {
  private requests = new Map<string, number[]>();

  canActivate(context: ExecutionContext): boolean {
    const client: Socket = context.switchToWs().getClient();
    const userId = client.data.user?.id || client.id;

    const now = Date.now();
    const windowMs = 60000; // 1 minute
    const maxRequests = 10; // 10 requests per minute

    // Get user's request timestamps
    const userRequests = this.requests.get(userId) || [];

    // Filter timestamps within window
    const recentRequests = userRequests.filter(
      timestamp => now - timestamp < windowMs,
    );

    // Check if exceeded
    if (recentRequests.length >= maxRequests) {
      throw new WsException('Rate limit exceeded. Try again later.');
    }

    // Add current request
    recentRequests.push(now);
    this.requests.set(userId, recentRequests);

    return true;
  }
}

// Usage
@WebSocketGateway()
export class RateLimitedGateway {
  @UseGuards(WsThrottleGuard)
  @SubscribeMessage('send-message')
  handleMessage(@MessageBody() data: any) {
    // Rate limited to 10 messages per minute
    return { event: 'message-sent', data };
  }
}
```

### **Permission-Based Guard**

```typescript
import { SetMetadata } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

// permissions.decorator.ts
export const RequirePermissions = (...permissions: string[]) =>
  SetMetadata('permissions', permissions);

// ws-permissions.guard.ts
@Injectable()
export class WsPermissionsGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredPermissions = this.reflector.get<string[]>(
      'permissions',
      context.getHandler(),
    );

    if (!requiredPermissions) {
      return true;
    }

    const client: Socket = context.switchToWs().getClient();
    const user = client.data.user;

    if (!user) {
      throw new WsException('Unauthorized');
    }

    const hasPermission = requiredPermissions.every(permission =>
      user.permissions?.includes(permission),
    );

    if (!hasPermission) {
      throw new WsException(
        `Missing permission: ${requiredPermissions.join(', ')}`,
      );
    }

    return true;
  }
}

// Usage
@WebSocketGateway()
export class PermissionGateway {
  @UseGuards(WsPermissionsGuard)
  @RequirePermissions('messages.delete')
  @SubscribeMessage('delete-message')
  handleDeleteMessage(@MessageBody() data: { messageId: string }) {
    return { event: 'message-deleted', messageId: data.messageId };
  }

  @UseGuards(WsPermissionsGuard)
  @RequirePermissions('users.ban')
  @SubscribeMessage('ban-user')
  handleBanUser(@MessageBody() data: { userId: string }) {
    return { event: 'user-banned', userId: data.userId };
  }
}
```

### **Guard with Exception Handling**

```typescript
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { WsException } from '@nestjs/websockets';
import { Socket } from 'socket.io';

@Injectable()
export class SafeGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    try {
      const client: Socket = context.switchToWs().getClient();
      const user = client.data.user;

      if (!user) {
        // WsException automatically caught by NestJS
        throw new WsException({
          error: 'Unauthorized',
          message: 'You must be logged in',
          code: 401,
        });
      }

      return true;
    } catch (error) {
      // Transform errors to WsException
      if (error instanceof WsException) {
        throw error;
      }
      throw new WsException({
        error: 'Authentication Error',
        message: error.message,
      });
    }
  }
}
```

### **Global vs Local Guards**

```typescript
// main.ts - Global WebSocket guard (NOT RECOMMENDED)
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Global guards apply to HTTP and WebSocket
  // But not recommended for WebSockets
  // Better to use gateway-level or event-level guards
  
  await app.listen(3000);
}
bootstrap();

// RECOMMENDED: Gateway-level
@UseGuards(WsAuthGuard)
@WebSocketGateway()
export class ProtectedGateway {}

// RECOMMENDED: Event-level
@WebSocketGateway()
export class SelectiveGateway {
  @UseGuards(WsAuthGuard)
  @SubscribeMessage('protected')
  handleProtected() {}
}
```

### **Production Example**

```typescript
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  Logger,
} from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { WsException } from '@nestjs/websockets';
import { Socket } from 'socket.io';

// Production-ready guard with logging
@Injectable()
export class ProductionWsGuard implements CanActivate {
  private logger = new Logger('WsGuard');

  constructor(
    private reflector: Reflector,
    private authService: AuthService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const startTime = Date.now();

    try {
      // Get socket and data
      const client: Socket = context.switchToWs().getClient();
      const data = context.switchToWs().getData();
      const event = context.switchToWs().getPattern();

      // Check if user is authenticated
      const user = client.data.user;
      if (!user) {
        this.logger.warn(
          `Unauthorized access attempt to ${event} from ${client.id}`,
        );
        throw new WsException('Unauthorized');
      }

      // Check roles if required
      const requiredRoles = this.reflector.get<string[]>(
        'roles',
        context.getHandler(),
      );

      if (requiredRoles) {
        const hasRole = requiredRoles.some(role =>
          user.roles?.includes(role),
        );

        if (!hasRole) {
          this.logger.warn(
            `User ${user.id} lacks role for ${event}: ` +
            `${requiredRoles.join(', ')}`,
          );
          throw new WsException(
            `Requires role: ${requiredRoles.join(' or ')}`,
          );
        }
      }

      // Check permissions if required
      const requiredPermissions = this.reflector.get<string[]>(
        'permissions',
        context.getHandler(),
      );

      if (requiredPermissions) {
        const hasPermissions = await this.authService.hasPermissions(
          user.id,
          requiredPermissions,
        );

        if (!hasPermissions) {
          this.logger.warn(
            `User ${user.id} lacks permissions for ${event}`,
          );
          throw new WsException('Insufficient permissions');
        }
      }

      const duration = Date.now() - startTime;
      this.logger.debug(
        `Guard passed for user ${user.id} on ${event} in ${duration}ms`,
      );

      return true;
    } catch (error) {
      if (error instanceof WsException) {
        throw error;
      }
      this.logger.error('Guard error:', error);
      throw new WsException('Authorization failed');
    }
  }
}

// Usage
@UseGuards(ProductionWsGuard)
@WebSocketGateway()
export class SecureGateway {
  @Roles('admin')
  @SubscribeMessage('admin-command')
  handleAdminCommand(@MessageBody() data: any) {
    // Only admins reach here
  }

  @RequirePermissions('messages.send')
  @SubscribeMessage('send-message')
  handleSendMessage(@MessageBody() data: any) {
    // Only users with 'messages.send' permission reach here
  }
}
```

**Interview Tip**: Guards in WebSockets use **`@UseGuards()` decorator** on gateway class (all events) or specific event handlers. **Guard implementation**: implement `CanActivate`, receive `ExecutionContext`, extract Socket with **`context.switchToWs().getClient()`**, return `true` (allow) or `false`/throw `WsException` (deny). **Get user**: `client.data.user` (attached during auth middleware). **Get data**: `context.switchToWs().getData()` for event payload. **Get event name**: `context.switchToWs().getPattern()`. **Common guards**: (1) Auth guard (check `user` exists), (2) Role guard (check `user.roles` contains required role), (3) Permission guard (check `user.permissions`), (4) Rate limit guard (throttle requests). **Decorators**: use `@Roles()`, `@RequirePermissions()` with **Reflector** in guard to get metadata. **Multiple guards**: `@UseGuards(AuthGuard, RoleGuard)` execute in order, all must return `true`. **WsException**: thrown when guard denies, automatically caught by NestJS, sent to client as error event. **Best practice**: use middleware for authentication (once on connection), use guards for authorization (per event).

</details>
28. How do you reject unauthorized connections?

<details>
<summary><strong>Answer</strong></summary>

To reject unauthorized connections, call **`next(new Error('message'))`** in the **authentication middleware** (in `afterInit()` hook) to **prevent connection**, or call **`client.disconnect()`** in **`handleConnection()`** to **close connection after it's established**. The middleware approach is preferred as it **rejects before connection completes**, while `disconnect()` is useful for **conditional disconnection** after initial handshake.

### **Method 1: Reject in Middleware (RECOMMENDED)**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  OnGatewayInit,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway()
export class RejectMiddlewareGateway implements OnGatewayInit {
  @WebSocketServer()
  server: Server;

  afterInit(server: Server) {
    server.use((socket, next) => {
      try {
        const token = socket.handshake.auth.token;

        // REJECT: No token
        if (!token) {
          return next(new Error('No token provided'));
        }

        // REJECT: Invalid token
        const user = this.validateToken(token);
        if (!user) {
          return next(new Error('Invalid token'));
        }

        // REJECT: Inactive user
        if (!user.isActive) {
          return next(new Error('Account disabled'));
        }

        // ALLOW: Attach user and continue
        socket.data.user = user;
        next();
      } catch (error) {
        // REJECT: Any error
        next(new Error('Authentication failed'));
      }
    });
  }

  private validateToken(token: string): any {
    // Validation logic
    return { id: '123', username: 'john', isActive: true };
  }
}

// CLIENT: Handle rejection
const socket = io('http://localhost:3000', {
  auth: { token: 'invalid-token' },
});

socket.on('connect_error', (error) => {
  console.error('Connection rejected:', error.message);
  // Output: "Connection rejected: Invalid token"
});
```

### **Method 2: Disconnect in handleConnection**

```typescript
import { OnGatewayConnection } from '@nestjs/websockets';

@WebSocketGateway()
export class RejectConnectionGateway implements OnGatewayConnection {
  handleConnection(client: Socket) {
    const user = client.data.user;

    // REJECT: No user (not authenticated)
    if (!user) {
      client.emit('error', { message: 'Unauthorized' });
      client.disconnect();
      return;
    }

    // REJECT: Banned user
    if (user.isBanned) {
      client.emit('error', { message: 'You are banned' });
      client.disconnect();
      return;
    }

    // REJECT: Max connections reached
    if (this.isUserAtMaxConnections(user.id)) {
      client.emit('error', { message: 'Too many connections' });
      client.disconnect();
      return;
    }

    // ALLOW: Connection successful
    console.log(`User ${user.username} connected`);
  }

  private isUserAtMaxConnections(userId: string): boolean {
    // Check connection count
    return false;
  }
}
```

### **JWT Token Rejection**

```typescript
import { JwtService } from '@nestjs/jwt';

@WebSocketGateway()
export class JwtRejectGateway implements OnGatewayInit {
  constructor(private jwtService: JwtService) {}

  afterInit(server: Server) {
    server.use((socket, next) => {
      const token = socket.handshake.auth.token;

      // REJECT: No token
      if (!token) {
        return next(new Error('Authentication required'));
      }

      try {
        // Verify JWT
        const payload = this.jwtService.verify(token);
        socket.data.user = payload;
        next();
      } catch (error) {
        // REJECT: Token expired
        if (error.name === 'TokenExpiredError') {
          return next(new Error('Token expired'));
        }
        // REJECT: Invalid token
        if (error.name === 'JsonWebTokenError') {
          return next(new Error('Invalid token'));
        }
        // REJECT: Other errors
        return next(new Error('Authentication failed'));
      }
    });
  }
}

// CLIENT:
socket.on('connect_error', (error) => {
  if (error.message === 'Token expired') {
    // Refresh token and reconnect
    refreshToken().then(newToken => {
      socket.auth = { token: newToken };
      socket.connect();
    });
  } else {
    // Redirect to login
    window.location.href = '/login';
  }
});
```

### **Role-Based Rejection**

```typescript
@WebSocketGateway({ namespace: '/admin' })
export class AdminOnlyGateway implements OnGatewayInit {
  constructor(private jwtService: JwtService) {}

  afterInit(server: Server) {
    server.use((socket, next) => {
      try {
        const token = socket.handshake.auth.token;
        const payload = this.jwtService.verify(token);

        // REJECT: Not admin role
        if (payload.role !== 'admin') {
          return next(new Error('Admin access only'));
        }

        socket.data.user = payload;
        next();
      } catch (error) {
        next(new Error('Authentication failed'));
      }
    });
  }

  handleConnection(client: Socket) {
    console.log('Admin connected:', client.data.user.username);
  }
}
```

### **Database Validation Rejection**

```typescript
@WebSocketGateway()
export class DatabaseValidationGateway implements OnGatewayInit {
  constructor(
    private jwtService: JwtService,
    private userService: UserService,
  ) {}

  afterInit(server: Server) {
    server.use(async (socket, next) => {
      try {
        const token = socket.handshake.auth.token;

        // Verify JWT
        const payload = this.jwtService.verify(token);

        // REJECT: User not found in database
        const user = await this.userService.findById(payload.sub);
        if (!user) {
          return next(new Error('User not found'));
        }

        // REJECT: Account disabled
        if (!user.isActive) {
          return next(new Error('Account is disabled'));
        }

        // REJECT: Email not verified
        if (!user.emailVerified) {
          return next(new Error('Email not verified'));
        }

        // REJECT: Subscription expired
        if (user.subscriptionExpired) {
          return next(new Error('Subscription expired'));
        }

        // REJECT: Token blacklisted
        const isBlacklisted = await this.userService.isTokenBlacklisted(
          token,
        );
        if (isBlacklisted) {
          return next(new Error('Token is blacklisted'));
        }

        socket.data.user = user;
        next();
      } catch (error) {
        next(new Error('Authentication failed'));
      }
    });
  }
}
```

### **Rate Limiting Rejection**

```typescript
@WebSocketGateway()
export class RateLimitGateway implements OnGatewayInit {
  private connectionAttempts = new Map<string, number[]>();

  afterInit(server: Server) {
    server.use((socket, next) => {
      const ip = socket.handshake.address;
      const now = Date.now();
      const windowMs = 60000; // 1 minute
      const maxAttempts = 5; // 5 connections per minute

      // Get IP's connection attempts
      const attempts = this.connectionAttempts.get(ip) || [];

      // Filter recent attempts
      const recentAttempts = attempts.filter(
        timestamp => now - timestamp < windowMs,
      );

      // REJECT: Too many attempts
      if (recentAttempts.length >= maxAttempts) {
        return next(
          new Error('Too many connection attempts. Try again later.'),
        );
      }

      // Record attempt
      recentAttempts.push(now);
      this.connectionAttempts.set(ip, recentAttempts);

      // Continue with authentication
      const token = socket.handshake.auth.token;
      if (!token) {
        return next(new Error('No token'));
      }

      // ... rest of auth logic
      next();
    });
  }
}
```

### **IP Whitelist/Blacklist**

```typescript
@WebSocketGateway()
export class IPFilterGateway implements OnGatewayInit {
  private readonly whitelist = ['127.0.0.1', '::1', '192.168.1.100'];
  private readonly blacklist = ['10.0.0.50'];

  afterInit(server: Server) {
    server.use((socket, next) => {
      const ip = socket.handshake.address;

      // REJECT: IP blacklisted
      if (this.blacklist.includes(ip)) {
        return next(new Error('IP address is blocked'));
      }

      // REJECT: IP not whitelisted (if whitelist is enabled)
      if (this.whitelist.length > 0 && !this.whitelist.includes(ip)) {
        return next(new Error('IP address not allowed'));
      }

      // Continue with other checks
      next();
    });
  }
}
```

### **Max Connections Per User**

```typescript
@WebSocketGateway()
export class MaxConnectionsGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  private userConnections = new Map<string, Set<string>>();
  private readonly MAX_CONNECTIONS = 3;

  afterInit(server: Server) {
    server.use((socket, next) => {
      // Authenticate first
      const token = socket.handshake.auth.token;
      if (!token) {
        return next(new Error('No token'));
      }

      try {
        const payload = this.jwtService.verify(token);
        socket.data.user = payload;
        next();
      } catch (error) {
        next(new Error('Invalid token'));
      }
    });
  }

  handleConnection(client: Socket) {
    const userId = client.data.user.sub;

    // Get user's connections
    if (!this.userConnections.has(userId)) {
      this.userConnections.set(userId, new Set());
    }

    const connections = this.userConnections.get(userId)!;

    // REJECT: Max connections reached
    if (connections.size >= this.MAX_CONNECTIONS) {
      client.emit('error', {
        message: `Maximum ${this.MAX_CONNECTIONS} connections allowed`,
      });
      client.disconnect();
      return;
    }

    // Add connection
    connections.add(client.id);
    console.log(
      `User ${userId} connected. ` +
      `Total connections: ${connections.size}`,
    );
  }

  handleDisconnect(client: Socket) {
    const userId = client.data.user?.sub;
    if (userId) {
      const connections = this.userConnections.get(userId);
      connections?.delete(client.id);
      
      if (connections?.size === 0) {
        this.userConnections.delete(userId);
      }
    }
  }
}
```

### **Graceful Rejection with Reason**

```typescript
@WebSocketGateway()
export class GracefulRejectGateway implements OnGatewayInit {
  afterInit(server: Server) {
    server.use(async (socket, next) => {
      try {
        const token = socket.handshake.auth.token;

        if (!token) {
          return this.reject(next, 'NO_TOKEN', 'Authentication required');
        }

        const payload = this.jwtService.verify(token);
        const user = await this.userService.findById(payload.sub);

        if (!user) {
          return this.reject(next, 'USER_NOT_FOUND', 'User not found');
        }

        if (!user.isActive) {
          return this.reject(next, 'ACCOUNT_DISABLED', 'Account disabled');
        }

        if (user.isBanned) {
          return this.reject(
            next,
            'BANNED',
            'You have been banned',
            { bannedUntil: user.bannedUntil },
          );
        }

        socket.data.user = user;
        next();
      } catch (error) {
        if (error.name === 'TokenExpiredError') {
          return this.reject(next, 'TOKEN_EXPIRED', 'Token expired');
        }
        return this.reject(next, 'AUTH_FAILED', 'Authentication failed');
      }
    });
  }

  private reject(
    next: (err?: Error) => void,
    code: string,
    message: string,
    data?: any,
  ) {
    const error = new Error(
      JSON.stringify({ code, message, ...data }),
    );
    next(error);
  }
}

// CLIENT: Parse rejection reason
socket.on('connect_error', (error) => {
  try {
    const parsed = JSON.parse(error.message);
    console.error('Rejection:', parsed);
    // { code: 'BANNED', message: 'You have been banned', bannedUntil: '...' }
    
    switch (parsed.code) {
      case 'TOKEN_EXPIRED':
        // Refresh token
        break;
      case 'BANNED':
        // Show ban message
        alert(`Banned until ${parsed.bannedUntil}`);
        break;
      default:
        // Redirect to login
        window.location.href = '/login';
    }
  } catch {
    console.error('Connection error:', error.message);
  }
});
```

### **Production Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  OnGatewayInit,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

@WebSocketGateway({
  cors: { origin: process.env.FRONTEND_URL },
})
export class ProductionRejectGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  @WebSocketServer()
  server: Server;

  private logger = new Logger('WebSocketAuth');
  private userConnections = new Map<string, Set<string>>();
  private connectionAttempts = new Map<string, number[]>();

  constructor(
    private jwtService: JwtService,
    private userService: UserService,
    private configService: ConfigService,
  ) {}

  afterInit(server: Server) {
    server.use(async (socket, next) => {
      const startTime = Date.now();
      const ip = socket.handshake.address;

      try {
        // RATE LIMITING
        if (!this.checkRateLimit(ip)) {
          this.logger.warn(`Rate limit exceeded for IP: ${ip}`);
          return next(new Error('Too many connection attempts'));
        }

        // GET TOKEN
        const token =
          socket.handshake.auth.token ||
          socket.handshake.query.token as string;

        if (!token) {
          this.logger.warn(`Connection attempt without token from ${ip}`);
          return next(new Error('No token provided'));
        }

        // VERIFY JWT
        let payload;
        try {
          payload = this.jwtService.verify(token, {
            secret: this.configService.get('JWT_SECRET'),
          });
        } catch (error) {
          this.logger.warn(`Invalid token from ${ip}: ${error.message}`);
          
          if (error.name === 'TokenExpiredError') {
            return next(new Error('Token expired'));
          }
          return next(new Error('Invalid token'));
        }

        // VALIDATE USER
        const user = await this.userService.findById(payload.sub);

        if (!user) {
          this.logger.warn(`User not found: ${payload.sub}`);
          return next(new Error('User not found'));
        }

        if (!user.isActive) {
          this.logger.warn(`Inactive user attempted connection: ${user.id}`);
          return next(new Error('Account is disabled'));
        }

        if (user.isBanned) {
          this.logger.warn(`Banned user attempted connection: ${user.id}`);
          return next(new Error('You are banned'));
        }

        // TOKEN BLACKLIST CHECK
        const isBlacklisted = await this.userService.isTokenBlacklisted(
          token,
        );
        if (isBlacklisted) {
          this.logger.warn(`Blacklisted token used: ${user.id}`);
          return next(new Error('Token is blacklisted'));
        }

        // Attach user
        socket.data.user = user;

        const duration = Date.now() - startTime;
        this.logger.log(
          `User ${user.username} authenticated in ${duration}ms`,
        );

        next();
      } catch (error) {
        this.logger.error('Authentication error:', error);
        next(new Error('Authentication failed'));
      }
    });

    this.logger.log('WebSocket authentication initialized');
  }

  handleConnection(client: Socket) {
    const user = client.data.user;
    const maxConnections = this.configService.get('MAX_CONNECTIONS_PER_USER', 3);

    // Get user's connections
    if (!this.userConnections.has(user.id)) {
      this.userConnections.set(user.id, new Set());
    }

    const connections = this.userConnections.get(user.id)!;

    // REJECT: Max connections
    if (connections.size >= maxConnections) {
      this.logger.warn(
        `User ${user.id} exceeded max connections (${maxConnections})`,
      );
      client.emit('error', {
        code: 'MAX_CONNECTIONS',
        message: `Maximum ${maxConnections} connections allowed`,
      });
      client.disconnect();
      return;
    }

    // Add connection
    connections.add(client.id);

    // Join personal room
    client.join(`user:${user.id}`);

    // Emit success
    client.emit('authenticated', {
      userId: user.id,
      username: user.username,
    });

    this.logger.log(
      `User ${user.username} connected (${client.id}). ` +
      `Connections: ${connections.size}/${maxConnections}`,
    );
  }

  handleDisconnect(client: Socket) {
    const user = client.data.user;
    if (user) {
      const connections = this.userConnections.get(user.id);
      connections?.delete(client.id);
      
      if (connections?.size === 0) {
        this.userConnections.delete(user.id);
      }

      this.logger.log(`User ${user.username} disconnected (${client.id})`);
    }
  }

  private checkRateLimit(ip: string): boolean {
    const now = Date.now();
    const windowMs = 60000; // 1 minute
    const maxAttempts = 10;

    const attempts = this.connectionAttempts.get(ip) || [];
    const recentAttempts = attempts.filter(
      timestamp => now - timestamp < windowMs,
    );

    if (recentAttempts.length >= maxAttempts) {
      return false;
    }

    recentAttempts.push(now);
    this.connectionAttempts.set(ip, recentAttempts);
    return true;
  }
}
```

**Interview Tip**: **Reject connections** with **`next(new Error('message'))`** in **middleware** (preferred - rejects before connection completes) or **`client.disconnect()`** in `handleConnection()` (after connection established). **Middleware rejection**: `server.use((socket, next) => { next(new Error('reason')) })` in `afterInit()` - **prevents connection**, client receives `connect_error` event. **Common rejection reasons**: (1) no token, (2) invalid/expired token, (3) user not found, (4) account disabled/banned, (5) rate limit exceeded, (6) max connections reached, (7) IP blacklisted. **Client handling**: `socket.on('connect_error', (error) => { console.error(error.message) })` - parse error message, refresh token if expired, redirect to login if unauthorized. **After connection rejection**: can send error data before disconnect with `client.emit('error', { reason })`, then `client.disconnect()`. **Production practices**: (1) rate limit by IP, (2) validate token and user in DB, (3) check account status, (4) enforce max connections per user, (5) log rejection attempts, (6) return meaningful error codes (not just strings). **Error structure**: send JSON with `code`, `message`, optional `data` fields for better client error handling.

</details>

## Message Format

29. What is the format of WebSocket messages?

<details>
<summary><strong>Answer</strong></summary>

In Socket.IO (NestJS WebSockets), messages follow the format: **`socket.emit(eventName, data, callback)`** where `eventName` is a **string**, `data` is **any serializable value** (typically an object), and `callback` is an **optional acknowledgment function**. Under the hood, Socket.IO uses a **packet structure** with type, namespace, event name, and data, but NestJS abstracts this with decorators like `@SubscribeMessage(eventName)` and `@MessageBody()` for easy handling.

### **Basic Message Structure**

```typescript
/**
 * MESSAGE FORMAT:
 * socket.emit(eventName, data, [callback])
 * 
 * eventName: string - the event identifier
 * data: any - payload (object, string, number, array, etc.)
 * callback: function - optional acknowledgment
 */

// CLIENT: Sending messages
socket.emit('message', { text: 'Hello' });
socket.emit('user-action', { action: 'click', buttonId: '123' });
socket.emit('update', 'simple string');
socket.emit('numbers', 42);
socket.emit('array', [1, 2, 3]);

// SERVER: Receiving messages
@SubscribeMessage('message')
handleMessage(@MessageBody() data: { text: string }) {
  console.log('Received:', data); // { text: 'Hello' }
}
```

### **NestJS Message Handling**

```typescript
import {
  WebSocketGateway,
  SubscribeMessage,
  MessageBody,
  ConnectedSocket,
  WsResponse,
} from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway()
export class MessageFormatGateway {
  // RECEIVE: Event with data
  @SubscribeMessage('send-message')
  handleSendMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    console.log('Event:', 'send-message');
    console.log('Data:', data);
    console.log('From:', client.id);

    // SEND: Response with event name and data
    return {
      event: 'message-received',
      data: {
        success: true,
        messageId: '123',
        timestamp: new Date(),
      },
    };
  }

  // RECEIVE: Multiple parameters
  @SubscribeMessage('multi-param')
  handleMultiParam(
    @MessageBody() data: any[],
    @ConnectedSocket() client: Socket,
  ) {
    // Client sent: socket.emit('multi-param', 'arg1', 'arg2', { key: 'value' })
    console.log('Parameters:', data);
    // Output: ['arg1', 'arg2', { key: 'value' }]
  }

  // RECEIVE: Event only (no data)
  @SubscribeMessage('ping')
  handlePing() {
    return { event: 'pong', data: { timestamp: Date.now() } };
  }
}
```

### **Common Data Formats**

```typescript
@WebSocketGateway()
export class DataFormatsGateway {
  // FORMAT 1: Object (most common)
  @SubscribeMessage('object-message')
  handleObject(@MessageBody() data: { id: string; text: string }) {
    return {
      event: 'response',
      data: {
        received: data,
        processed: true,
      },
    };
  }

  // FORMAT 2: String
  @SubscribeMessage('string-message')
  handleString(@MessageBody() message: string) {
    return {
      event: 'response',
      data: `Received: ${message}`,
    };
  }

  // FORMAT 3: Number
  @SubscribeMessage('number-message')
  handleNumber(@MessageBody() num: number) {
    return {
      event: 'response',
      data: num * 2,
    };
  }

  // FORMAT 4: Array
  @SubscribeMessage('array-message')
  handleArray(@MessageBody() items: any[]) {
    return {
      event: 'response',
      data: items.length,
    };
  }

  // FORMAT 5: Complex nested object
  @SubscribeMessage('complex-message')
  handleComplex(
    @MessageBody()
    data: {
      user: { id: string; name: string };
      message: { text: string; attachments: string[] };
      metadata: Record<string, any>;
    },
  ) {
    return { event: 'processed', data: { success: true } };
  }
}
```

### **Response Formats**

```typescript
@WebSocketGateway()
export class ResponseFormatsGateway {
  // RESPONSE 1: WsResponse object (typed)
  @SubscribeMessage('typed-response')
  handleTypedResponse(): WsResponse<any> {
    return {
      event: 'response-event',
      data: { message: 'Typed response' },
    };
  }

  // RESPONSE 2: Plain object (implicit)
  @SubscribeMessage('plain-response')
  handlePlainResponse() {
    return {
      event: 'response',
      data: 'Plain response',
    };
  }

  // RESPONSE 3: Observable (async)
  @SubscribeMessage('observable-response')
  handleObservableResponse(): Observable<WsResponse<any>> {
    return of({
      event: 'response',
      data: 'Observable response',
    }).pipe(delay(1000));
  }

  // RESPONSE 4: Promise (async)
  @SubscribeMessage('promise-response')
  async handlePromiseResponse(): Promise<WsResponse<any>> {
    await new Promise(resolve => setTimeout(resolve, 1000));
    return {
      event: 'response',
      data: 'Promise response',
    };
  }

  // RESPONSE 5: No response (void)
  @SubscribeMessage('no-response')
  handleNoResponse(@MessageBody() data: any): void {
    console.log('Received:', data);
    // No response sent to client
  }

  // RESPONSE 6: Direct emit (manual)
  @SubscribeMessage('manual-response')
  handleManualResponse(
    @MessageBody() data: any,
    @ConnectedSocket() client: Socket,
  ): void {
    // Emit directly instead of returning
    client.emit('manual-event', { data: 'Manual response' });
  }
}
```

### **Binary Data (Buffers)**

```typescript
@WebSocketGateway()
export class BinaryDataGateway {
  // Send binary data
  @SubscribeMessage('request-file')
  handleFileRequest(@ConnectedSocket() client: Socket) {
    // Send Buffer
    const buffer = Buffer.from('Hello, binary!');
    client.emit('file-data', buffer);
  }

  // Receive binary data
  @SubscribeMessage('upload-file')
  handleFileUpload(@MessageBody() data: Buffer) {
    console.log('Received buffer size:', data.length);
    return { event: 'upload-complete', data: { size: data.length } };
  }
}

// CLIENT: Binary data
socket.emit('upload-file', buffer);
socket.on('file-data', (data) => {
  console.log('Received buffer:', data);
});
```

### **Structured Message Pattern**

```typescript
// message.dto.ts
export interface MessageDTO {
  type: 'text' | 'image' | 'file';
  content: string;
  metadata?: {
    timestamp: Date;
    userId: string;
    roomId: string;
  };
}

@WebSocketGateway()
export class StructuredMessageGateway {
  @SubscribeMessage('send-structured')
  handleStructured(
    @MessageBody() message: MessageDTO,
    @ConnectedSocket() client: Socket,
  ) {
    // Validate structure
    if (!message.type || !message.content) {
      return {
        event: 'error',
        data: { error: 'Invalid message structure' },
      };
    }

    // Process message
    const processed: MessageDTO = {
      ...message,
      metadata: {
        ...message.metadata,
        timestamp: new Date(),
        userId: client.data.user.id,
      },
    };

    return {
      event: 'message-processed',
      data: processed,
    };
  }
}

// CLIENT:
socket.emit('send-structured', {
  type: 'text',
  content: 'Hello!',
  metadata: { roomId: 'room-1' },
});
```

### **Packet Structure (Under the Hood)**

```typescript
/**
 * Socket.IO Packet Structure:
 * 
 * Type 0: CONNECT
 * Type 1: DISCONNECT  
 * Type 2: EVENT (most common)
 * Type 3: ACK (acknowledgment)
 * Type 4: CONNECT_ERROR
 * Type 5: BINARY_EVENT
 * Type 6: BINARY_ACK
 * 
 * Example EVENT packet:
 * [
 *   2,                    // Type: EVENT
 *   "/chat",              // Namespace
 *   "message",            // Event name
 *   {"text": "Hello"}    // Data
 * ]
 * 
 * NestJS abstracts this with decorators
 */

// You typically don't need to work with raw packets
// But can access via socket.io internals if needed
@WebSocketGateway()
export class PacketInspectionGateway {
  afterInit(server: Server) {
    server.on('connection', (socket) => {
      socket.on('*', (packet) => {
        // Intercept all packets (requires wildcard middleware)
        console.log('Raw packet:', packet);
      });
    });
  }
}
```

**Interview Tip**: Socket.IO messages use **`emit(eventName, data, callback?)`** format. **EventName** is string identifier, **data** is any serializable value (typically object with multiple fields). **NestJS** handles with `@SubscribeMessage(eventName)` for event, `@MessageBody()` for data, `@ConnectedSocket()` for socket. **Response**: return `{ event: string, data: any }` object - NestJS automatically emits to sender. **Common formats**: objects (most common), strings, numbers, arrays, buffers (binary). **Multiple args**: client `emit('event', arg1, arg2, arg3)` → server receives as array `[arg1, arg2, arg3]` in `@MessageBody()`. **No response**: return `void` or emit manually with `client.emit()`. **Async**: return `Promise` or `Observable` for async processing. **Packet structure**: Socket.IO uses type + namespace + event + data, but NestJS abstracts this. **Serialization**: data automatically JSON stringified/parsed, supports objects, arrays, strings, numbers, nulls - **not** functions, undefined, symbols.

</details>
30. How do you handle acknowledgments (callbacks)?

<details>
<summary><strong>Answer</strong></summary>

Acknowledgments in Socket.IO are **callback functions** that confirm message delivery/processing. **Client sends**: `socket.emit('event', data, (response) => { ... })` where the callback receives the server's response. **Server responds**: either (1) **return value** from `@SubscribeMessage()` handler (NestJS automatically calls callback with it), or (2) use **`@ConnectedSocket()` + manual callback** as the last parameter in `@MessageBody()`. Acknowledgments enable **request-response** pattern over WebSockets.

### **Basic Acknowledgment**

```typescript
// SERVER: Automatic acknowledgment (return value)
import {
  WebSocketGateway,
  SubscribeMessage,
  MessageBody,
} from '@nestjs/websockets';

@WebSocketGateway()
export class AckGateway {
  @SubscribeMessage('send-message')
  handleMessage(@MessageBody() data: { message: string }) {
    // Return value automatically sent to client's callback
    return {
      event: 'message-ack',
      data: {
        success: true,
        messageId: '123',
        timestamp: new Date(),
      },
    };
  }
}

// CLIENT: Send with acknowledgment
socket.emit('send-message', { message: 'Hello' }, (response) => {
  console.log('Server response:', response);
  // Output: { success: true, messageId: '123', timestamp: '...' }
});
```

### **Manual Acknowledgment**

```typescript
import { ConnectedSocket } from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway()
export class ManualAckGateway {
  @SubscribeMessage('manual-ack')
  handleManualAck(
    @MessageBody() data: any,
    @ConnectedSocket() client: Socket,
  ) {
    // Get callback from socket.io (internal)
    // NestJS doesn't directly expose callback parameter
    // Use return value instead (automatic ack)
    
    return {
      event: 'ack-response',
      data: { received: true },
    };
  }
}
```

### **Async Acknowledgment**

```typescript
@WebSocketGateway()
export class AsyncAckGateway {
  constructor(private messageService: MessageService) {}

  // ASYNC: With Promise
  @SubscribeMessage('save-message')
  async handleSaveMessage(
    @MessageBody() data: { message: string },
  ): Promise<any> {
    // Save to database (async)
    const saved = await this.messageService.save(data.message);

    // Return ack after async operation
    return {
      event: 'message-saved',
      data: {
        success: true,
        messageId: saved.id,
      },
    };
  }

  // ASYNC: With Observable
  @SubscribeMessage('process-data')
  handleProcessData(
    @MessageBody() data: any,
  ): Observable<WsResponse<any>> {
    return from(this.messageService.process(data)).pipe(
      map(result => ({
        event: 'processed',
        data: { result },
      })),
    );
  }
}

// CLIENT:
socket.emit('save-message', { message: 'Hello' }, (response) => {
  console.log('Saved:', response);
  // Waits for async operation to complete
});
```

### **Acknowledgment with Timeout**

```typescript
// CLIENT: Timeout if server doesn't respond
const timeout = setTimeout(() => {
  console.error('Acknowledgment timeout');
}, 5000);

socket.emit('send-message', { message: 'Hello' }, (response) => {
  clearTimeout(timeout);
  console.log('Response:', response);
});

// Or use Socket.IO built-in timeout (v4.4+)
socket.timeout(5000).emit('send-message', { message: 'Hello' }, (err, response) => {
  if (err) {
    console.error('Timeout error:', err);
  } else {
    console.log('Response:', response);
  }
});
```

### **Acknowledgment with Error Handling**

```typescript
@WebSocketGateway()
export class ErrorAckGateway {
  @SubscribeMessage('risky-operation')
  async handleRiskyOperation(
    @MessageBody() data: any,
  ): Promise<any> {
    try {
      const result = await this.performRiskyOperation(data);
      
      // Success acknowledgment
      return {
        event: 'operation-success',
        data: {
          success: true,
          result,
        },
      };
    } catch (error) {
      // Error acknowledgment
      return {
        event: 'operation-error',
        data: {
          success: false,
          error: error.message,
        },
      };
    }
  }

  private async performRiskyOperation(data: any) {
    // Risky operation
    throw new Error('Something went wrong');
  }
}

// CLIENT:
socket.emit('risky-operation', { data: '...' }, (response) => {
  if (response.success) {
    console.log('Success:', response.result);
  } else {
    console.error('Error:', response.error);
  }
});
```

### **Multiple Acknowledgments Pattern**

```typescript
@WebSocketGateway()
export class MultiAckGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('batch-operation')
  async handleBatchOperation(
    @MessageBody() items: any[],
  ): Promise<any> {
    const results = [];

    for (const item of items) {
      const result = await this.processItem(item);
      results.push(result);
    }

    return {
      event: 'batch-complete',
      data: {
        processed: results.length,
        results,
      },
    };
  }

  private async processItem(item: any) {
    // Process each item
    return { id: item.id, processed: true };
  }
}
```

### **Request-Response Pattern**

```typescript
@WebSocketGateway()
export class RequestResponseGateway {
  constructor(private userService: UserService) {}

  // GET user data
  @SubscribeMessage('get-user')
  async handleGetUser(
    @MessageBody() data: { userId: string },
  ): Promise<any> {
    const user = await this.userService.findById(data.userId);

    if (!user) {
      return {
        event: 'user-response',
        data: {
          success: false,
          error: 'User not found',
        },
      };
    }

    return {
      event: 'user-response',
      data: {
        success: true,
        user: {
          id: user.id,
          username: user.username,
          email: user.email,
        },
      },
    };
  }

  // CREATE user
  @SubscribeMessage('create-user')
  async handleCreateUser(
    @MessageBody() data: { username: string; email: string },
  ): Promise<any> {
    try {
      const user = await this.userService.create(data);
      
      return {
        event: 'user-created',
        data: {
          success: true,
          userId: user.id,
        },
      };
    } catch (error) {
      return {
        event: 'user-created',
        data: {
          success: false,
          error: error.message,
        },
      };
    }
  }
}

// CLIENT: Request-response pattern
function getUser(userId: string): Promise<any> {
  return new Promise((resolve, reject) => {
    socket.emit('get-user', { userId }, (response) => {
      if (response.success) {
        resolve(response.user);
      } else {
        reject(new Error(response.error));
      }
    });
  });
}

// Usage
getUser('123')
  .then(user => console.log('User:', user))
  .catch(error => console.error('Error:', error));
```

### **Server-to-Client Acknowledgment**

```typescript
@WebSocketGateway()
export class ServerAckGateway {
  @WebSocketServer()
  server: Server;

  // Server sends message and expects acknowledgment
  async notifyUserWithAck(userId: string, notification: any) {
    const sockets = await this.server
      .to(`user:${userId}`)
      .fetchSockets();

    for (const socket of sockets) {
      // Send with acknowledgment callback
      socket.emit(
        'notification',
        notification,
        (response) => {
          console.log('Client acknowledged:', response);
        },
      );
    }
  }
}

// CLIENT: Acknowledge server message
socket.on('notification', (data, callback) => {
  console.log('Notification:', data);
  
  // Send acknowledgment
  callback({ received: true, timestamp: Date.now() });
});
```

### **Broadcast with Acknowledgments**

```typescript
@WebSocketGateway()
export class BroadcastAckGateway {
  @WebSocketServer()
  server: Server;

  // Broadcast to room and collect acknowledgments
  async broadcastWithAcks(roomId: string, message: any) {
    const sockets = await this.server.in(roomId).fetchSockets();

    const acknowledgments: Promise<any>[] = [];

    for (const socket of sockets) {
      const ackPromise = new Promise((resolve) => {
        socket.emit('broadcast-message', message, (response) => {
          resolve({
            socketId: socket.id,
            response,
          });
        });
      });
      acknowledgments.push(ackPromise);
    }

    // Wait for all acknowledgments
    const results = await Promise.all(acknowledgments);
    console.log('All clients acknowledged:', results);
    return results;
  }
}
```

### **Socket.IO v4.5+ emitWithAck**

```typescript
@WebSocketGateway()
export class EmitWithAckGateway {
  @WebSocketServer()
  server: Server;

  // Use built-in emitWithAck (Socket.IO v4.5+)
  async sendWithAck(userId: string, data: any) {
    try {
      // Automatically returns Promise that resolves with ack
      const response = await this.server
        .to(`user:${userId}`)
        .timeout(5000) // Optional timeout
        .emitWithAck('request', data);

      console.log('Response:', response);
      return response;
    } catch (error) {
      console.error('Acknowledgment failed:', error);
      throw error;
    }
  }
}
```

### **Production Example**

```typescript
import {
  WebSocketGateway,
  SubscribeMessage,
  MessageBody,
  ConnectedSocket,
} from '@nestjs/websockets';
import { Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway()
export class ProductionAckGateway {
  private logger = new Logger('AckGateway');

  constructor(
    private chatService: ChatService,
    private validationService: ValidationService,
  ) {}

  @SubscribeMessage('send-message')
  async handleSendMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ): Promise<any> {
    const startTime = Date.now();
    const userId = client.data.user.id;

    try {
      // VALIDATION
      const validation = this.validationService.validateMessage(data);
      if (!validation.valid) {
        this.logger.warn(`Invalid message from user ${userId}`);
        return {
          event: 'message-error',
          data: {
            success: false,
            error: validation.error,
          },
        };
      }

      // SAVE MESSAGE
      const saved = await this.chatService.saveMessage({
        userId,
        roomId: data.roomId,
        content: data.message,
      });

      // BROADCAST TO ROOM
      client.to(data.roomId).emit('new-message', {
        id: saved.id,
        userId,
        message: data.message,
        timestamp: saved.createdAt,
      });

      const duration = Date.now() - startTime;
      this.logger.log(
        `Message ${saved.id} saved and broadcast in ${duration}ms`,
      );

      // ACKNOWLEDGMENT
      return {
        event: 'message-sent',
        data: {
          success: true,
          messageId: saved.id,
          timestamp: saved.createdAt,
          duration,
        },
      };
    } catch (error) {
      this.logger.error('Error sending message:', error);
      
      return {
        event: 'message-error',
        data: {
          success: false,
          error: 'Failed to send message',
        },
      };
    }
  }
}

// CLIENT: Production usage with retry
class MessageClient {
  private maxRetries = 3;
  private timeout = 5000;

  async sendMessage(
    roomId: string,
    message: string,
  ): Promise<any> {
    for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
      try {
        const response = await this.sendWithTimeout(
          roomId,
          message,
        );

        if (response.success) {
          return response;
        } else {
          throw new Error(response.error);
        }
      } catch (error) {
        console.error(`Attempt ${attempt} failed:`, error);
        
        if (attempt === this.maxRetries) {
          throw new Error('Max retries reached');
        }
        
        // Wait before retry
        await new Promise(resolve =>
          setTimeout(resolve, 1000 * attempt),
        );
      }
    }
  }

  private sendWithTimeout(
    roomId: string,
    message: string,
  ): Promise<any> {
    return new Promise((resolve, reject) => {
      const timer = setTimeout(() => {
        reject(new Error('Timeout'));
      }, this.timeout);

      socket.emit(
        'send-message',
        { roomId, message },
        (response) => {
          clearTimeout(timer);
          resolve(response);
        },
      );
    });
  }
}
```

**Interview Tip**: **Acknowledgments** are **callback functions** for confirming message delivery. **Client sends**: `socket.emit('event', data, (response) => { ... })` where callback receives server's response. **Server responds**: **return value** from `@SubscribeMessage()` handler - NestJS **automatically** calls client's callback with it. **Return format**: `{ event: string, data: any }` object. **Async**: return `Promise` or `Observable` - client callback waits for async operation. **Timeout**: client uses `socket.timeout(ms).emit()` (v4.4+) or manual `setTimeout`. **Server-to-client ack**: `socket.emit('event', data, (clientResponse) => { ... })` - client listens with `socket.on('event', (data, callback) => { callback(response) })`. **Use cases**: (1) confirm message saved, (2) request-response pattern, (3) error handling, (4) retry logic. **emitWithAck**: Socket.IO v4.5+ has `server.to(room).timeout(ms).emitWithAck('event', data)` returning Promise. **Best practice**: always return `{ success: boolean, error?: string, data?: any }` format for consistent error handling. **No manual callback**: NestJS doesn't expose callback as parameter, use return value instead.

</details>
31. How do you send and receive JSON data?

<details>
<summary><strong>Answer</strong></summary>

Socket.IO **automatically serializes/deserializes JSON** by default, so you can send and receive JavaScript objects directly without manual `JSON.stringify()` or `JSON.parse()`. Simply pass objects to **`emit()`** and they'll be transmitted as JSON. On the receiving end, use **`@MessageBody()`** decorator to get the parsed object automatically.

### **Sending JSON (Server to Client)**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway()
export class JsonGateway {
  @WebSocketServer()
  server: Server;

  // Send JSON object
  sendUserData(userId: string) {
    const userData = {
      id: userId,
      username: 'john_doe',
      email: 'john@example.com',
      profile: {
        age: 30,
        location: 'New York',
        interests: ['coding', 'gaming', 'music'],
      },
      createdAt: new Date(),
      isOnline: true,
    };

    // Socket.IO automatically converts to JSON
    this.server.to(`user:${userId}`).emit('user-data', userData);
    // Client receives parsed object, no JSON.parse() needed
  }

  // Send array of objects
  sendMessageHistory(roomId: string) {
    const messages = [
      { id: 1, text: 'Hello', user: 'Alice', timestamp: new Date() },
      { id: 2, text: 'Hi there', user: 'Bob', timestamp: new Date() },
      { id: 3, text: 'How are you?', user: 'Alice', timestamp: new Date() },
    ];

    this.server.to(roomId).emit('message-history', messages);
  }

  // Send nested JSON
  sendComplexData() {
    const complexData = {
      status: 'success',
      data: {
        users: [
          { id: 1, name: 'Alice' },
          { id: 2, name: 'Bob' },
        ],
        meta: {
          total: 2,
          page: 1,
          hasMore: false,
        },
      },
      timestamp: new Date(),
    };

    this.server.emit('complex-data', complexData);
  }
}
```

### **Receiving JSON (Client to Server)**

```typescript
@WebSocketGateway()
export class ReceiveJsonGateway {
  // Receive simple object
  @SubscribeMessage('send-message')
  handleMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    // 'data' is automatically parsed from JSON
    console.log('Room:', data.roomId);
    console.log('Message:', data.message);

    return {
      event: 'message-received',
      data: {
        success: true,
        messageId: generateId(),
        timestamp: new Date(),
      },
    };
  }

  // Receive complex object with nested data
  @SubscribeMessage('create-post')
  handleCreatePost(
    @MessageBody()
    postData: {
      title: string;
      content: string;
      tags: string[];
      metadata: {
        author: string;
        category: string;
      };
    },
    @ConnectedSocket() client: Socket,
  ) {
    console.log('Title:', postData.title);
    console.log('Tags:', postData.tags);
    console.log('Author:', postData.metadata.author);

    return {
      event: 'post-created',
      data: {
        postId: generateId(),
        ...postData,
        createdAt: new Date(),
      },
    };
  }

  // Receive array
  @SubscribeMessage('bulk-update')
  handleBulkUpdate(
    @MessageBody() items: Array<{ id: string; value: number }>,
  ) {
    console.log(`Updating ${items.length} items`);
    items.forEach(item => {
      console.log(`Item ${item.id}: ${item.value}`);
    });

    return {
      event: 'bulk-update-complete',
      data: { updated: items.length },
    };
  }
}

function generateId(): string {
  return Math.random().toString(36).substr(2, 9);
}
```

### **Client-Side JSON (TypeScript/JavaScript)**

```typescript
// client.ts
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000');

// SEND JSON (automatically serialized)
socket.emit('send-message', {
  roomId: 'room-123',
  message: 'Hello World',
});

// Send complex object
socket.emit('create-post', {
  title: 'My Post',
  content: 'Post content here...',
  tags: ['javascript', 'nestjs', 'websockets'],
  metadata: {
    author: 'John Doe',
    category: 'Technology',
  },
});

// Send array
socket.emit('bulk-update', [
  { id: '1', value: 100 },
  { id: '2', value: 200 },
  { id: '3', value: 300 },
]);

// RECEIVE JSON (automatically parsed)
socket.on('user-data', (userData) => {
  // userData is already a JavaScript object
  console.log('User ID:', userData.id);
  console.log('Username:', userData.username);
  console.log('Interests:', userData.profile.interests);
});

socket.on('message-history', (messages) => {
  // messages is already an array of objects
  messages.forEach(msg => {
    console.log(`${msg.user}: ${msg.text}`);
  });
});

socket.on('complex-data', (data) => {
  console.log('Status:', data.status);
  console.log('Users:', data.data.users);
  console.log('Total:', data.data.meta.total);
});
```

### **Using DTOs (Data Transfer Objects)**

```typescript
// message.dto.ts
export class SendMessageDto {
  roomId: string;
  message: string;
  attachments?: string[];
}

export class CreatePostDto {
  title: string;
  content: string;
  tags: string[];
  metadata: {
    author: string;
    category: string;
  };
}

// gateway.ts
import { UsePipes, ValidationPipe } from '@nestjs/common';
import { IsString, IsArray, IsNotEmpty, ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';

// DTO with validation
export class ValidatedMessageDto {
  @IsString()
  @IsNotEmpty()
  roomId: string;

  @IsString()
  @IsNotEmpty()
  message: string;

  @IsArray()
  @IsString({ each: true })
  tags?: string[];
}

@WebSocketGateway()
export class DtoGateway {
  // Use DTO type
  @SubscribeMessage('send-message')
  handleMessage(
    @MessageBody() data: SendMessageDto,
    @ConnectedSocket() client: Socket,
  ) {
    // Type-safe access
    const roomId: string = data.roomId;
    const message: string = data.message;

    return {
      event: 'message-sent',
      data: { success: true },
    };
  }

  // With validation
  @UsePipes(new ValidationPipe())
  @SubscribeMessage('validated-message')
  handleValidatedMessage(
    @MessageBody() data: ValidatedMessageDto,
  ) {
    // Data is validated automatically
    return { event: 'validated', data };
  }
}
```

### **Handling Dates in JSON**

```typescript
@WebSocketGateway()
export class DateHandlingGateway {
  @SubscribeMessage('create-event')
  handleCreateEvent(
    @MessageBody()
    data: {
      title: string;
      startDate: string; // Dates come as ISO strings
      endDate: string;
    },
  ) {
    // Convert string dates to Date objects
    const event = {
      title: data.title,
      startDate: new Date(data.startDate),
      endDate: new Date(data.endDate),
      duration:
        new Date(data.endDate).getTime() - new Date(data.startDate).getTime(),
    };

    console.log('Event:', event);

    // Send back with dates (automatically converted to ISO strings)
    return {
      event: 'event-created',
      data: {
        ...event,
        createdAt: new Date(), // Sent as ISO string
      },
    };
  }
}

// CLIENT:
socket.emit('create-event', {
  title: 'Meeting',
  startDate: new Date().toISOString(), // Convert Date to string
  endDate: new Date(Date.now() + 3600000).toISOString(),
});

socket.on('event-created', (data) => {
  // Convert string back to Date
  const createdAt = new Date(data.createdAt);
  console.log('Created at:', createdAt);
});
```

### **Binary Data (Buffer, ArrayBuffer)**

```typescript
@WebSocketGateway()
export class BinaryDataGateway {
  @WebSocketServer()
  server: Server;

  // Send binary data
  sendBinaryData(userId: string) {
    const buffer = Buffer.from('Binary data here', 'utf8');
    
    // Socket.IO can send Buffer directly
    this.server.to(`user:${userId}`).emit('binary-data', buffer);
  }

  // Receive binary data
  @SubscribeMessage('upload-file')
  handleFileUpload(
    @MessageBody() data: { filename: string; content: Buffer },
  ) {
    console.log('Filename:', data.filename);
    console.log('Buffer size:', data.content.length);

    // Process buffer
    const text = data.content.toString('utf8');
    console.log('Content:', text);

    return {
      event: 'file-uploaded',
      data: { success: true, size: data.content.length },
    };
  }
}

// CLIENT:
socket.emit('upload-file', {
  filename: 'document.txt',
  content: new ArrayBuffer(1024), // Binary data
});
```

### **Large JSON Payloads**

```typescript
@WebSocketGateway()
export class LargeDataGateway {
  @WebSocketServer()
  server: Server;

  // Send large JSON with compression
  sendLargeData(userId: string) {
    const largeData = {
      items: Array.from({ length: 1000 }, (_, i) => ({
        id: i,
        name: `Item ${i}`,
        description: 'A'.repeat(100),
        metadata: {
          created: new Date(),
          updated: new Date(),
        },
      })),
    };

    // Enable compression for large payloads
    this.server
      .to(`user:${userId}`)
      .compress(true)
      .emit('large-data', largeData);
  }

  // Paginated data
  sendPaginatedData(page: number, limit: number) {
    const start = (page - 1) * limit;
    const items = Array.from({ length: limit }, (_, i) => ({
      id: start + i,
      data: `Item ${start + i}`,
    }));

    this.server.emit('paginated-data', {
      page,
      limit,
      items,
      total: 10000,
      hasMore: page * limit < 10000,
    });
  }
}
```

### **JSON Serialization Options**

```typescript
@WebSocketGateway()
export class SerializationGateway {
  @WebSocketServer()
  server: Server;

  // Custom JSON serialization
  sendCustomData() {
    const data = {
      id: 123,
      name: 'John',
      // This will be serialized as ISO string
      createdAt: new Date(),
      // undefined values are removed
      optionalField: undefined,
      // null is preserved
      nullableField: null,
    };

    this.server.emit('custom-data', data);
    // Client receives: { id: 123, name: 'John', createdAt: '2025-...', nullableField: null }
  }

  // Class instances (won't serialize methods)
  sendClassInstance() {
    class User {
      constructor(
        public id: number,
        public name: string,
      ) {}

      greet() {
        return `Hello, ${this.name}`;
      }
    }

    const user = new User(1, 'Alice');

    // Only properties are sent, methods are lost
    this.server.emit('user-data', user);
    // Client receives: { id: 1, name: 'Alice' }
  }
}
```

### **Error Handling for Invalid JSON**

```typescript
@WebSocketGateway()
export class JsonErrorGateway {
  @SubscribeMessage('process-data')
  handleProcessData(
    @MessageBody() data: any,
    @ConnectedSocket() client: Socket,
  ) {
    try {
      // Validate data structure
      if (!data || typeof data !== 'object') {
        throw new Error('Invalid data format');
      }

      if (!data.requiredField) {
        throw new Error('Missing required field');
      }

      // Process data
      return {
        event: 'data-processed',
        data: { success: true },
      };
    } catch (error) {
      return {
        event: 'error',
        data: {
          message: error.message,
          code: 'INVALID_DATA',
        },
      };
    }
  }
}
```

### **Production Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger, UsePipes, ValidationPipe } from '@nestjs/common';
import { IsString, IsNotEmpty, IsNumber, Min, Max } from 'class-validator';

// DTOs with validation
export class SendMessageDto {
  @IsString()
  @IsNotEmpty()
  roomId: string;

  @IsString()
  @IsNotEmpty()
  message: string;

  @IsString({ each: true })
  mentions?: string[];
}

export class PaginationDto {
  @IsNumber()
  @Min(1)
  page: number;

  @IsNumber()
  @Min(1)
  @Max(100)
  limit: number;
}

@WebSocketGateway({
  cors: { origin: '*' },
})
export class ProductionJsonGateway {
  @WebSocketServer()
  server: Server;

  private logger = new Logger('JsonGateway');

  constructor(
    private messageService: MessageService,
    private userService: UserService,
  ) {}

  // SEND JSON: Message with full data
  @UsePipes(new ValidationPipe())
  @SubscribeMessage('send-message')
  async handleSendMessage(
    @MessageBody() dto: SendMessageDto,
    @ConnectedSocket() client: Socket,
  ) {
    try {
      const user = client.data.user;

      // Save to database
      const message = await this.messageService.create({
        userId: user.id,
        roomId: dto.roomId,
        content: dto.message,
        mentions: dto.mentions,
      });

      // Prepare JSON response
      const messageData = {
        id: message.id,
        content: message.content,
        roomId: message.roomId,
        user: {
          id: user.id,
          username: user.username,
          avatar: user.avatar,
        },
        mentions: message.mentions || [],
        createdAt: message.createdAt,
        updatedAt: message.updatedAt,
      };

      // Broadcast to room (JSON automatically serialized)
      this.server.to(dto.roomId).emit('new-message', messageData);

      this.logger.log(
        `Message sent by ${user.username} to room ${dto.roomId}`,
      );

      return {
        event: 'message-sent',
        data: { messageId: message.id, success: true },
      };
    } catch (error) {
      this.logger.error('Error sending message:', error);
      return {
        event: 'error',
        data: {
          message: 'Failed to send message',
          code: 'SEND_ERROR',
        },
      };
    }
  }

  // RECEIVE JSON: Paginated data request
  @UsePipes(new ValidationPipe())
  @SubscribeMessage('get-messages')
  async handleGetMessages(
    @MessageBody() dto: { roomId: string; pagination: PaginationDto },
    @ConnectedSocket() client: Socket,
  ) {
    try {
      const { roomId, pagination } = dto;

      // Get paginated messages
      const result = await this.messageService.findByRoom(
        roomId,
        pagination.page,
        pagination.limit,
      );

      // Send JSON response
      client.emit('messages-data', {
        roomId,
        messages: result.messages.map(msg => ({
          id: msg.id,
          content: msg.content,
          user: {
            id: msg.user.id,
            username: msg.user.username,
          },
          createdAt: msg.createdAt,
        })),
        pagination: {
          page: pagination.page,
          limit: pagination.limit,
          total: result.total,
          hasMore: pagination.page * pagination.limit < result.total,
        },
      });
    } catch (error) {
      this.logger.error('Error getting messages:', error);
      client.emit('error', {
        message: 'Failed to get messages',
        code: 'FETCH_ERROR',
      });
    }
  }

  // SEND COMPRESSED JSON: Large data
  async sendRoomData(roomId: string) {
    const [messages, users, settings] = await Promise.all([
      this.messageService.getRecent(roomId, 100),
      this.userService.getRoomMembers(roomId),
      this.messageService.getRoomSettings(roomId),
    ]);

    const roomData = {
      roomId,
      messages,
      users,
      settings,
      timestamp: new Date(),
    };

    // Use compression for large payloads
    this.server.to(roomId).compress(true).emit('room-data', roomData);
  }
}
```

**Interview Tip**: Socket.IO **automatically handles JSON** serialization/deserialization - no need for `JSON.stringify()` or `JSON.parse()`. **Send**: `emit('event', { key: 'value' })` - object sent as JSON. **Receive**: `@MessageBody()` decorator provides **parsed object**. **Supported types**: objects, arrays, strings, numbers, booleans, null, Date (as ISO string), Buffer. **Not supported**: functions, undefined (removed), circular references (error). **Dates**: automatically converted to ISO strings, need manual `new Date(str)` conversion on receive. **Large data**: use `.compress(true)` before emit for compression. **DTOs**: use TypeScript interfaces/classes for type safety, add class-validator decorators with `@UsePipes(ValidationPipe)` for validation. **Best practices**: (1) use DTOs for structure, (2) validate with ValidationPipe, (3) compress large payloads, (4) handle Date conversions, (5) catch JSON errors. **Binary data**: Buffer/ArrayBuffer supported directly without JSON encoding.

</details>

## Error Handling

32. How do you handle errors in WebSocket connections?

<details>
<summary><strong>Answer</strong></summary>

Errors in WebSocket connections are handled using **try-catch blocks** in event handlers, **`WsException`** for throwing structured errors (automatically sent to client), **Exception Filters** for global error handling (using `@Catch()` decorator), and **client-side error events** (`connect_error`, `error`, `disconnect`). **Middleware errors** (in `afterInit()`) reject connections with `next(new Error())`, while **runtime errors** in handlers are caught and sent as error events.

### **Basic Error Handling**

```typescript
import {
  WebSocketGateway,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { WsException } from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway()
export class ErrorHandlingGateway {
  @SubscribeMessage('risky-operation')
  handleRiskyOperation(
    @MessageBody() data: any,
    @ConnectedSocket() client: Socket,
  ) {
    try {
      // Validate input
      if (!data || !data.value) {
        throw new WsException('Invalid data: value is required');
      }

      // Business logic that might throw
      const result = this.processData(data.value);

      return {
        event: 'operation-success',
        data: result,
      };
    } catch (error) {
      // WsException is automatically sent to client
      if (error instanceof WsException) {
        throw error;
      }

      // Wrap other errors
      throw new WsException({
        error: 'OPERATION_FAILED',
        message: error.message,
      });
    }
  }

  private processData(value: any) {
    if (value < 0) {
      throw new Error('Value must be positive');
    }
    return { processed: value * 2 };
  }
}

// CLIENT: Handle errors
socket.on('exception', (error) => {
  console.error('Server error:', error);
  // Output: { error: 'OPERATION_FAILED', message: '...' }
});
```

### **Connection Errors (Middleware)**

```typescript
import { OnGatewayInit } from '@nestjs/websockets';
import { Server } from 'socket.io';

@WebSocketGateway()
export class ConnectionErrorGateway implements OnGatewayInit {
  afterInit(server: Server) {
    server.use((socket, next) => {
      try {
        const token = socket.handshake.auth.token;

        if (!token) {
          // ERROR: Rejects connection
          return next(new Error('No token provided'));
        }

        // Validation that might throw
        this.validateToken(token);

        next();
      } catch (error) {
        // ERROR: Rejects connection with error message
        next(new Error(`Authentication failed: ${error.message}`));
      }
    });
  }

  private validateToken(token: string) {
    if (token === 'invalid') {
      throw new Error('Invalid token');
    }
  }
}

// CLIENT: Handle connection errors
socket.on('connect_error', (error) => {
  console.error('Connection error:', error.message);
  // Output: "Authentication failed: Invalid token"
  
  if (error.message.includes('Authentication')) {
    // Redirect to login
    window.location.href = '/login';
  }
});
```

### **Runtime Errors in Event Handlers**

```typescript
@WebSocketGateway()
export class RuntimeErrorGateway {
  @SubscribeMessage('divide')
  handleDivide(
    @MessageBody() data: { a: number; b: number },
    @ConnectedSocket() client: Socket,
  ) {
    try {
      if (data.b === 0) {
        throw new WsException({
          error: 'DIVISION_BY_ZERO',
          message: 'Cannot divide by zero',
          statusCode: 400,
        });
      }

      const result = data.a / data.b;

      return {
        event: 'divide-result',
        data: { result },
      };
    } catch (error) {
      // Log error server-side
      console.error('Division error:', error);

      // Send to client
      if (error instanceof WsException) {
        throw error;
      }

      throw new WsException('Division failed');
    }
  }

  @SubscribeMessage('fetch-user')
  async handleFetchUser(
    @MessageBody() userId: string,
    @ConnectedSocket() client: Socket,
  ) {
    try {
      // Database operation that might fail
      const user = await this.userService.findById(userId);

      if (!user) {
        throw new WsException({
          error: 'USER_NOT_FOUND',
          message: `User ${userId} not found`,
          statusCode: 404,
        });
      }

      return {
        event: 'user-data',
        data: user,
      };
    } catch (error) {
      if (error instanceof WsException) {
        throw error;
      }

      // Database errors
      throw new WsException({
        error: 'DATABASE_ERROR',
        message: 'Failed to fetch user',
        statusCode: 500,
      });
    }
  }

  private userService = {
    async findById(id: string) {
      // Simulate database query
      if (id === 'invalid') {
        throw new Error('Database connection failed');
      }
      return id === '123' ? { id, name: 'John' } : null;
    },
  };
}
```

### **Custom Error Types**

```typescript
// Custom error classes
export class ValidationError extends WsException {
  constructor(field: string, message: string) {
    super({
      error: 'VALIDATION_ERROR',
      field,
      message,
      statusCode: 400,
    });
  }
}

export class AuthorizationError extends WsException {
  constructor(action: string) {
    super({
      error: 'AUTHORIZATION_ERROR',
      message: `Not authorized to ${action}`,
      statusCode: 403,
    });
  }
}

export class NotFoundError extends WsException {
  constructor(resource: string, id: string) {
    super({
      error: 'NOT_FOUND',
      message: `${resource} with id ${id} not found`,
      statusCode: 404,
    });
  }
}

// Usage
@WebSocketGateway()
export class CustomErrorGateway {
  @SubscribeMessage('update-user')
  async handleUpdateUser(
    @MessageBody() data: { userId: string; name: string },
    @ConnectedSocket() client: Socket,
  ) {
    // Validation error
    if (!data.name || data.name.trim().length === 0) {
      throw new ValidationError('name', 'Name cannot be empty');
    }

    // Authorization check
    const currentUser = client.data.user;
    if (currentUser.id !== data.userId && currentUser.role !== 'admin') {
      throw new AuthorizationError('update other users');
    }

    // Not found error
    const user = await this.userService.findById(data.userId);
    if (!user) {
      throw new NotFoundError('User', data.userId);
    }

    // Update user
    await this.userService.update(data.userId, { name: data.name });

    return {
      event: 'user-updated',
      data: { userId: data.userId, name: data.name },
    };
  }

  private userService = {
    async findById(id: string) {
      return id === '123' ? { id, name: 'John' } : null;
    },
    async update(id: string, data: any) {
      return { id, ...data };
    },
  };
}
```

### **Error Logging**

```typescript
import { Logger } from '@nestjs/common';

@WebSocketGateway()
export class LoggedErrorGateway {
  private logger = new Logger('WebSocketErrors');

  @SubscribeMessage('process-order')
  async handleProcessOrder(
    @MessageBody() orderId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const startTime = Date.now();

    try {
      this.logger.log(
        `Processing order ${orderId} for user ${client.data.user?.id}`,
      );

      // Process order (might throw)
      const result = await this.orderService.process(orderId);

      const duration = Date.now() - startTime;
      this.logger.log(`Order ${orderId} processed in ${duration}ms`);

      return {
        event: 'order-processed',
        data: result,
      };
    } catch (error) {
      const duration = Date.now() - startTime;

      // Log error with context
      this.logger.error(
        `Failed to process order ${orderId} after ${duration}ms`,
        error.stack,
      );

      // Send error to client
      throw new WsException({
        error: 'ORDER_PROCESSING_FAILED',
        message: 'Failed to process order',
        orderId,
      });
    }
  }

  private orderService = {
    async process(id: string) {
      if (id === 'fail') {
        throw new Error('Payment gateway error');
      }
      return { orderId: id, status: 'processed' };
    },
  };
}
```

### **Client-Side Error Handling**

```typescript
// client.ts
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000', {
  auth: { token: 'my-token' },
});

// CONNECTION ERRORS
socket.on('connect_error', (error) => {
  console.error('Connection failed:', error.message);
  
  // Handle specific errors
  if (error.message.includes('Authentication')) {
    alert('Please log in again');
    window.location.href = '/login';
  } else if (error.message.includes('Rate limit')) {
    alert('Too many connection attempts. Please wait.');
  }
});

// RUNTIME ERRORS (from WsException)
socket.on('exception', (error) => {
  console.error('Server exception:', error);
  
  // Parse error structure
  const { error: errorCode, message, statusCode } = error;
  
  switch (errorCode) {
    case 'VALIDATION_ERROR':
      alert(`Validation error: ${message}`);
      break;
    case 'AUTHORIZATION_ERROR':
      alert('You are not authorized for this action');
      break;
    case 'NOT_FOUND':
      alert(message);
      break;
    default:
      alert('An error occurred');
  }
});

// DISCONNECT ERRORS
socket.on('disconnect', (reason) => {
  console.log('Disconnected:', reason);
  
  if (reason === 'io server disconnect') {
    // Server disconnected us (e.g., kicked out)
    alert('You have been disconnected by the server');
  } else if (reason === 'transport close') {
    // Connection lost
    console.log('Connection lost, will attempt to reconnect...');
  }
});

// GENERIC ERROR EVENT
socket.on('error', (error) => {
  console.error('Socket error:', error);
});

// ERROR IN SPECIFIC EVENT RESPONSE
socket.emit('divide', { a: 10, b: 0 }, (response) => {
  if (response.error) {
    console.error('Operation error:', response.error);
  } else {
    console.log('Result:', response.data);
  }
});
```

### **Error Retry Logic**

```typescript
@WebSocketGateway()
export class RetryErrorGateway {
  private retryCount = new Map<string, number>();
  private readonly MAX_RETRIES = 3;

  @SubscribeMessage('unreliable-operation')
  async handleUnreliableOperation(
    @MessageBody() data: { id: string },
    @ConnectedSocket() client: Socket,
  ) {
    const key = `${client.id}:${data.id}`;
    const retries = this.retryCount.get(key) || 0;

    try {
      // Operation that might fail
      const result = await this.performOperation(data.id);

      // Success: clear retry count
      this.retryCount.delete(key);

      return {
        event: 'operation-success',
        data: result,
      };
    } catch (error) {
      // Increment retry count
      this.retryCount.set(key, retries + 1);

      if (retries < this.MAX_RETRIES) {
        // Tell client to retry
        throw new WsException({
          error: 'OPERATION_FAILED',
          message: 'Operation failed, please retry',
          retries: retries + 1,
          maxRetries: this.MAX_RETRIES,
          canRetry: true,
        });
      } else {
        // Max retries exceeded
        this.retryCount.delete(key);
        throw new WsException({
          error: 'MAX_RETRIES_EXCEEDED',
          message: 'Operation failed after maximum retries',
          canRetry: false,
        });
      }
    }
  }

  private async performOperation(id: string) {
    // Simulate unreliable operation
    if (Math.random() < 0.5) {
      throw new Error('Random failure');
    }
    return { id, result: 'success' };
  }
}

// CLIENT: Auto-retry
function sendWithRetry(eventName: string, data: any, maxRetries = 3) {
  let retries = 0;

  function send() {
    socket.emit(eventName, data);
  }

  socket.on('exception', (error) => {
    if (error.canRetry && retries < maxRetries) {
      retries++;
      console.log(`Retrying... (${retries}/${maxRetries})`);
      setTimeout(send, 1000 * retries); // Exponential backoff
    } else {
      console.error('Operation failed:', error.message);
    }
  });

  send();
}
```

### **Production Error Handling**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
  OnGatewayInit,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';
import { WsException } from '@nestjs/websockets';

@WebSocketGateway({
  cors: { origin: '*' },
})
export class ProductionErrorGateway implements OnGatewayInit {
  @WebSocketServer()
  server: Server;

  private logger = new Logger('WebSocketGateway');

  constructor(
    private messageService: MessageService,
    private errorTracker: ErrorTrackingService,
  ) {}

  // MIDDLEWARE ERRORS (Connection-level)
  afterInit(server: Server) {
    server.use((socket, next) => {
      try {
        const token = socket.handshake.auth.token;

        if (!token) {
          this.logger.warn(
            `Connection attempt without token from ${socket.handshake.address}`,
          );
          return next(new Error('Authentication required'));
        }

        // Validation that might throw
        const user = this.validateToken(token);
        socket.data.user = user;
        next();
      } catch (error) {
        this.logger.error('Connection error:', error);
        this.errorTracker.track('CONNECTION_ERROR', error);
        next(new Error('Authentication failed'));
      }
    });
  }

  // RUNTIME ERRORS (Event-level)
  @SubscribeMessage('send-message')
  async handleSendMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    const startTime = Date.now();
    const user = client.data.user;

    try {
      // VALIDATION
      if (!data.message || data.message.trim().length === 0) {
        throw new WsException({
          error: 'VALIDATION_ERROR',
          message: 'Message cannot be empty',
          field: 'message',
          statusCode: 400,
        });
      }

      if (data.message.length > 1000) {
        throw new WsException({
          error: 'VALIDATION_ERROR',
          message: 'Message too long (max 1000 characters)',
          field: 'message',
          statusCode: 400,
        });
      }

      // AUTHORIZATION
      const isMember = await this.messageService.isRoomMember(
        user.id,
        data.roomId,
      );

      if (!isMember) {
        throw new WsException({
          error: 'AUTHORIZATION_ERROR',
          message: 'You are not a member of this room',
          statusCode: 403,
        });
      }

      // BUSINESS LOGIC (might throw)
      const message = await this.messageService.create({
        userId: user.id,
        roomId: data.roomId,
        content: data.message,
      });

      // SUCCESS: Broadcast
      this.server.to(data.roomId).emit('new-message', {
        id: message.id,
        content: message.content,
        user: { id: user.id, username: user.username },
        createdAt: message.createdAt,
      });

      const duration = Date.now() - startTime;
      this.logger.log(
        `Message sent by ${user.username} to ${data.roomId} in ${duration}ms`,
      );

      return {
        event: 'message-sent',
        data: { messageId: message.id, success: true },
      };
    } catch (error) {
      const duration = Date.now() - startTime;

      // LOG ERROR
      this.logger.error(
        `Error sending message (user: ${user?.id}, room: ${data.roomId}, ` +
        `duration: ${duration}ms):`,
        error.stack,
      );

      // TRACK ERROR (for monitoring)
      this.errorTracker.track('SEND_MESSAGE_ERROR', {
        userId: user?.id,
        roomId: data.roomId,
        error: error.message,
        duration,
      });

      // SEND TO CLIENT
      if (error instanceof WsException) {
        throw error;
      }

      // Wrap unexpected errors
      throw new WsException({
        error: 'INTERNAL_ERROR',
        message: 'Failed to send message',
        statusCode: 500,
      });
    }
  }

  private validateToken(token: string): any {
    // Token validation
    if (token === 'invalid') {
      throw new Error('Invalid token');
    }
    return { id: '123', username: 'john' };
  }
}

// Error Tracking Service
class ErrorTrackingService {
  track(errorType: string, data: any) {
    // Send to monitoring service (Sentry, DataDog, etc.)
    console.error(`[${errorType}]`, data);
  }
}

class MessageService {
  async create(data: any) {
    return {
      id: '123',
      ...data,
      createdAt: new Date(),
    };
  }

  async isRoomMember(userId: string, roomId: string) {
    return true;
  }
}
```

**Interview Tip**: **Error handling** in WebSockets: (1) **Connection errors** - use `next(new Error())` in middleware (`afterInit` hook) to reject connections, client receives `connect_error` event; (2) **Runtime errors** - throw **`WsException`** in event handlers, automatically sent to client as `exception` event; (3) **Client errors** - listen for `connect_error` (connection failed), `exception` (server errors), `disconnect` (connection lost), `error` (generic errors). **WsException structure**: `{ error: 'CODE', message: 'text', statusCode: 400 }` - provides structured error data. **Try-catch**: wrap risky operations, log errors server-side, throw WsException to client. **Custom errors**: extend WsException for specific error types (ValidationError, AuthorizationError, NotFoundError). **Logging**: use Logger service to track errors with context (user, event, duration). **Monitoring**: integrate with error tracking services (Sentry, DataDog). **Retry logic**: include retry info in error response (`{ canRetry: true, retries: 2 }`), implement exponential backoff on client. **Production**: (1) validate input, (2) check authorization, (3) wrap operations in try-catch, (4) log all errors, (5) send structured errors to client, (6) track errors for monitoring.

</details>
33. What is `WsException`?

<details>
<summary><strong>Answer</strong></summary>

**`WsException`** is NestJS's built-in exception class for WebSocket errors, extending the base `Error` class. When thrown in a WebSocket gateway event handler, it **automatically sends** the error to the client via Socket.IO's **`exception` event**. It provides a structured way to communicate errors from server to client without manual `socket.emit('error')` calls. The exception payload is **serialized as JSON** and sent to the triggering client only (not broadcast).

### **Basic Usage**

```typescript
import { WsException } from '@nestjs/websockets';
import {
  WebSocketGateway,
  SubscribeMessage,
  MessageBody,
  ConnectedSocket,
} from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway()
export class BasicWsExceptionGateway {
  @SubscribeMessage('validate-input')
  handleValidateInput(
    @MessageBody() data: { value: string },
    @ConnectedSocket() client: Socket,
  ) {
    if (!data.value) {
      // Throw WsException - automatically sent to client
      throw new WsException('Value is required');
      // Client receives on 'exception' event: "Value is required"
    }

    return {
      event: 'validation-success',
      data: { valid: true },
    };
  }
}

// CLIENT:
socket.emit('validate-input', { value: '' });

// Receives error on 'exception' event
socket.on('exception', (error) => {
  console.error('Error:', error);
  // Output: "Value is required"
});
```

### **Structured Error Payload**

```typescript
@WebSocketGateway()
export class StructuredErrorGateway {
  @SubscribeMessage('create-user')
  handleCreateUser(
    @MessageBody() data: { username: string; email: string },
  ) {
    // Throw with object payload
    if (!data.username) {
      throw new WsException({
        error: 'VALIDATION_ERROR',
        message: 'Username is required',
        field: 'username',
        statusCode: 400,
      });
    }

    if (data.username.length < 3) {
      throw new WsException({
        error: 'VALIDATION_ERROR',
        message: 'Username must be at least 3 characters',
        field: 'username',
        minLength: 3,
        statusCode: 400,
      });
    }

    if (!this.isValidEmail(data.email)) {
      throw new WsException({
        error: 'VALIDATION_ERROR',
        message: 'Invalid email format',
        field: 'email',
        statusCode: 400,
      });
    }

    return {
      event: 'user-created',
      data: { id: '123', ...data },
    };
  }

  private isValidEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
}

// CLIENT:
socket.emit('create-user', { username: 'ab', email: 'invalid' });

socket.on('exception', (error) => {
  console.error(error);
  /*
  Output:
  {
    error: 'VALIDATION_ERROR',
    message: 'Username must be at least 3 characters',
    field: 'username',
    minLength: 3,
    statusCode: 400
  }
  */

  // Handle specific errors
  if (error.error === 'VALIDATION_ERROR') {
    alert(`Validation failed: ${error.message}`);
  }
});
```

### **Error Types**

```typescript
@WebSocketGateway()
export class ErrorTypesGateway {
  @SubscribeMessage('fetch-resource')
  async handleFetchResource(
    @MessageBody() data: { resourceId: string },
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;

    // VALIDATION ERROR (400)
    if (!data.resourceId) {
      throw new WsException({
        error: 'BAD_REQUEST',
        message: 'Resource ID is required',
        statusCode: 400,
      });
    }

    // AUTHENTICATION ERROR (401)
    if (!user) {
      throw new WsException({
        error: 'UNAUTHORIZED',
        message: 'Authentication required',
        statusCode: 401,
      });
    }

    // AUTHORIZATION ERROR (403)
    const hasAccess = await this.checkAccess(user.id, data.resourceId);
    if (!hasAccess) {
      throw new WsException({
        error: 'FORBIDDEN',
        message: 'Access denied to this resource',
        statusCode: 403,
      });
    }

    // NOT FOUND ERROR (404)
    const resource = await this.findResource(data.resourceId);
    if (!resource) {
      throw new WsException({
        error: 'NOT_FOUND',
        message: `Resource ${data.resourceId} not found`,
        statusCode: 404,
      });
    }

    return {
      event: 'resource-data',
      data: resource,
    };
  }

  private async checkAccess(userId: string, resourceId: string) {
    return true; // Simplified
  }

  private async findResource(id: string) {
    return id === '123' ? { id, name: 'Resource' } : null;
  }
}
```

### **Custom Error Classes**

```typescript
// Custom error classes extending WsException
export class ValidationError extends WsException {
  constructor(field: string, message: string, constraints?: any) {
    super({
      error: 'VALIDATION_ERROR',
      message,
      field,
      constraints,
      statusCode: 400,
    });
  }
}

export class UnauthorizedError extends WsException {
  constructor(message = 'Authentication required') {
    super({
      error: 'UNAUTHORIZED',
      message,
      statusCode: 401,
    });
  }
}

export class ForbiddenError extends WsException {
  constructor(resource: string) {
    super({
      error: 'FORBIDDEN',
      message: `Access denied to ${resource}`,
      statusCode: 403,
    });
  }
}

export class NotFoundError extends WsException {
  constructor(resource: string, id: string) {
    super({
      error: 'NOT_FOUND',
      message: `${resource} with id ${id} not found`,
      resourceType: resource,
      resourceId: id,
      statusCode: 404,
    });
  }
}

export class ConflictError extends WsException {
  constructor(message: string) {
    super({
      error: 'CONFLICT',
      message,
      statusCode: 409,
    });
  }
}

export class TooManyRequestsError extends WsException {
  constructor(retryAfter: number) {
    super({
      error: 'TOO_MANY_REQUESTS',
      message: 'Rate limit exceeded',
      retryAfter, // seconds
      statusCode: 429,
    });
  }
}

export class InternalServerError extends WsException {
  constructor(message = 'Internal server error') {
    super({
      error: 'INTERNAL_ERROR',
      message,
      statusCode: 500,
    });
  }
}

// Usage
@WebSocketGateway()
export class CustomErrorsGateway {
  @SubscribeMessage('register-user')
  async handleRegisterUser(
    @MessageBody() data: { username: string; email: string },
  ) {
    // Validation error
    if (!data.username || data.username.length < 3) {
      throw new ValidationError('username', 'Username must be at least 3 characters', {
        minLength: 3,
      });
    }

    // Check if user exists
    const existingUser = await this.userService.findByUsername(data.username);
    if (existingUser) {
      throw new ConflictError(`Username ${data.username} is already taken`);
    }

    try {
      const user = await this.userService.create(data);
      return {
        event: 'user-registered',
        data: user,
      };
    } catch (error) {
      // Wrap unexpected errors
      throw new InternalServerError('Failed to register user');
    }
  }

  @SubscribeMessage('admin-action')
  handleAdminAction(@ConnectedSocket() client: Socket) {
    const user = client.data.user;

    if (!user) {
      throw new UnauthorizedError();
    }

    if (user.role !== 'admin') {
      throw new ForbiddenError('admin actions');
    }

    return { event: 'admin-action-success', data: { success: true } };
  }

  @SubscribeMessage('get-post')
  async handleGetPost(@MessageBody() postId: string) {
    const post = await this.postService.findById(postId);
    
    if (!post) {
      throw new NotFoundError('Post', postId);
    }

    return { event: 'post-data', data: post };
  }

  private userService = {
    async findByUsername(username: string) {
      return username === 'taken' ? { id: '1', username } : null;
    },
    async create(data: any) {
      return { id: '123', ...data };
    },
  };

  private postService = {
    async findById(id: string) {
      return id === '123' ? { id, title: 'Post' } : null;
    },
  };
}
```

### **Wrapping Other Errors**

```typescript
@WebSocketGateway()
export class WrappingErrorsGateway {
  @SubscribeMessage('database-operation')
  async handleDatabaseOperation(@MessageBody() data: any) {
    try {
      // Operation that might throw various errors
      const result = await this.databaseService.query(data);

      return {
        event: 'operation-success',
        data: result,
      };
    } catch (error) {
      // WsException: rethrow as is
      if (error instanceof WsException) {
        throw error;
      }

      // Database-specific errors
      if (error.code === 'ECONNREFUSED') {
        throw new WsException({
          error: 'SERVICE_UNAVAILABLE',
          message: 'Database connection failed',
          statusCode: 503,
        });
      }

      if (error.code === '23505') {
        // PostgreSQL unique violation
        throw new WsException({
          error: 'DUPLICATE_ENTRY',
          message: 'Record already exists',
          statusCode: 409,
        });
      }

      // Generic error
      throw new WsException({
        error: 'INTERNAL_ERROR',
        message: 'Operation failed',
        originalError: error.message,
        statusCode: 500,
      });
    }
  }

  private databaseService = {
    async query(data: any) {
      if (data.fail) {
        throw new Error('Database error');
      }
      return { success: true };
    },
  };
}
```

### **Error Logging with WsException**

```typescript
import { Logger } from '@nestjs/common';

@WebSocketGateway()
export class LoggedWsExceptionGateway {
  private logger = new Logger('WebSocketGateway');

  @SubscribeMessage('critical-operation')
  async handleCriticalOperation(
    @MessageBody() data: any,
    @ConnectedSocket() client: Socket,
  ) {
    const startTime = Date.now();
    const user = client.data.user;

    try {
      this.logger.log(
        `Critical operation started by user ${user?.id}: ${JSON.stringify(data)}`,
      );

      // Operation
      const result = await this.performOperation(data);

      const duration = Date.now() - startTime;
      this.logger.log(
        `Critical operation completed in ${duration}ms`,
      );

      return {
        event: 'operation-complete',
        data: result,
      };
    } catch (error) {
      const duration = Date.now() - startTime;

      // Log error details
      this.logger.error(
        `Critical operation failed after ${duration}ms:\n` +
        `User: ${user?.id}\n` +
        `Data: ${JSON.stringify(data)}\n` +
        `Error: ${error.message}`,
        error.stack,
      );

      // Send to monitoring service
      this.errorTracker.track({
        type: 'CRITICAL_OPERATION_FAILED',
        userId: user?.id,
        data,
        error: error.message,
        duration,
      });

      // Throw WsException to client
      if (error instanceof WsException) {
        throw error;
      }

      throw new WsException({
        error: 'OPERATION_FAILED',
        message: 'Critical operation failed',
        statusCode: 500,
      });
    }
  }

  private async performOperation(data: any) {
    if (data.fail) {
      throw new Error('Operation error');
    }
    return { success: true };
  }

  private errorTracker = {
    track(data: any) {
      console.error('[ERROR TRACKER]', data);
    },
  };
}
```

### **Client-Side Error Handling**

```typescript
// client.ts
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000');

// LISTEN FOR EXCEPTIONS (from WsException)
socket.on('exception', (error) => {
  console.error('Server exception:', error);

  // Handle different error types
  if (typeof error === 'string') {
    // Simple string error
    alert(`Error: ${error}`);
  } else if (typeof error === 'object') {
    // Structured error
    const { error: code, message, statusCode, field } = error;

    switch (code) {
      case 'VALIDATION_ERROR':
        console.error(`Validation failed on field '${field}': ${message}`);
        // Highlight field in UI
        break;

      case 'UNAUTHORIZED':
        console.error('Not authenticated');
        // Redirect to login
        window.location.href = '/login';
        break;

      case 'FORBIDDEN':
        console.error('Access denied:', message);
        // Show permission denied message
        break;

      case 'NOT_FOUND':
        console.error('Resource not found:', message);
        // Show 404 message
        break;

      case 'CONFLICT':
        console.error('Conflict:', message);
        // Show conflict resolution UI
        break;

      case 'TOO_MANY_REQUESTS':
        console.error('Rate limit exceeded');
        // Wait and retry
        setTimeout(() => {
          console.log('Retrying...');
        }, error.retryAfter * 1000);
        break;

      case 'INTERNAL_ERROR':
        console.error('Server error:', message);
        // Show generic error message
        break;

      default:
        console.error('Unknown error:', error);
    }
  }
});

// EMIT WITH ERROR HANDLING
function emitWithErrorHandling(event: string, data: any) {
  socket.emit(event, data);

  // Set timeout for error
  const timeoutId = setTimeout(() => {
    console.error('Operation timed out');
  }, 5000);

  // Clear timeout on response
  socket.once(`${event}-success`, () => {
    clearTimeout(timeoutId);
  });
}
```

### **Production Example**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { WsException } from '@nestjs/websockets';
import { Logger } from '@nestjs/common';

// Custom error classes
export class WsValidationError extends WsException {
  constructor(field: string, message: string) {
    super({ error: 'VALIDATION_ERROR', message, field, statusCode: 400 });
  }
}

export class WsUnauthorizedError extends WsException {
  constructor() {
    super({ error: 'UNAUTHORIZED', message: 'Authentication required', statusCode: 401 });
  }
}

export class WsForbiddenError extends WsException {
  constructor(action: string) {
    super({ error: 'FORBIDDEN', message: `Not allowed to ${action}`, statusCode: 403 });
  }
}

export class WsNotFoundError extends WsException {
  constructor(resource: string, id: string) {
    super({
      error: 'NOT_FOUND',
      message: `${resource} ${id} not found`,
      resource,
      id,
      statusCode: 404,
    });
  }
}

@WebSocketGateway({
  cors: { origin: '*' },
})
export class ProductionWsExceptionGateway {
  @WebSocketServer()
  server: Server;

  private logger = new Logger('WsException');

  constructor(
    private messageService: MessageService,
    private errorTracker: ErrorTrackingService,
  ) {}

  @SubscribeMessage('send-message')
  async handleSendMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;
    const startTime = Date.now();

    try {
      // AUTHENTICATION CHECK
      if (!user) {
        throw new WsUnauthorizedError();
      }

      // VALIDATION
      if (!data.message?.trim()) {
        throw new WsValidationError('message', 'Message cannot be empty');
      }

      if (data.message.length > 1000) {
        throw new WsValidationError(
          'message',
          'Message too long (max 1000 characters)',
        );
      }

      // AUTHORIZATION CHECK
      const isMember = await this.messageService.isRoomMember(
        user.id,
        data.roomId,
      );

      if (!isMember) {
        throw new WsForbiddenError('send messages in this room');
      }

      // BUSINESS LOGIC
      const message = await this.messageService.create({
        userId: user.id,
        roomId: data.roomId,
        content: data.message,
      });

      // SUCCESS: Broadcast
      this.server.to(data.roomId).emit('new-message', {
        id: message.id,
        content: message.content,
        user: { id: user.id, username: user.username },
        createdAt: message.createdAt,
      });

      const duration = Date.now() - startTime;
      this.logger.log(
        `Message sent by ${user.username} to ${data.roomId} in ${duration}ms`,
      );

      return {
        event: 'message-sent',
        data: { messageId: message.id },
      };
    } catch (error) {
      const duration = Date.now() - startTime;

      // LOG ERROR
      this.logger.error(
        `Error sending message (user: ${user?.id}, room: ${data.roomId}, ` +
        `duration: ${duration}ms): ${error.message}`,
      );

      // TRACK ERROR
      this.errorTracker.track('SEND_MESSAGE_ERROR', {
        userId: user?.id,
        roomId: data.roomId,
        error: error.message,
        duration,
      });

      // RETHROW WsException (auto-sent to client)
      if (error instanceof WsException) {
        throw error;
      }

      // WRAP OTHER ERRORS
      throw new WsException({
        error: 'INTERNAL_ERROR',
        message: 'Failed to send message',
        statusCode: 500,
      });
    }
  }

  @SubscribeMessage('delete-message')
  async handleDeleteMessage(
    @MessageBody() messageId: string,
    @ConnectedSocket() client: Socket,
  ) {
    try {
      const user = client.data.user;

      if (!user) {
        throw new WsUnauthorizedError();
      }

      // Find message
      const message = await this.messageService.findById(messageId);

      if (!message) {
        throw new WsNotFoundError('Message', messageId);
      }

      // Check ownership
      if (message.userId !== user.id && user.role !== 'admin') {
        throw new WsForbiddenError('delete this message');
      }

      // Delete
      await this.messageService.delete(messageId);

      // Broadcast deletion
      this.server.to(message.roomId).emit('message-deleted', {
        messageId,
      });

      return {
        event: 'message-deleted-success',
        data: { messageId },
      };
    } catch (error) {
      if (error instanceof WsException) {
        throw error;
      }

      throw new WsException({
        error: 'DELETE_FAILED',
        message: 'Failed to delete message',
        statusCode: 500,
      });
    }
  }
}

class MessageService {
  async create(data: any) {
    return { id: '123', ...data, createdAt: new Date() };
  }

  async isRoomMember(userId: string, roomId: string) {
    return true;
  }

  async findById(id: string) {
    return id === '123'
      ? { id, userId: '1', roomId: 'room1', content: 'Hello' }
      : null;
  }

  async delete(id: string) {
    return { success: true };
  }
}

class ErrorTrackingService {
  track(type: string, data: any) {
    console.error(`[${type}]`, data);
  }
}
```

**Interview Tip**: **`WsException`** is NestJS's **built-in error class** for WebSockets - when thrown in event handler, **automatically sends** error to client on **`exception` event** (no manual `socket.emit('error')` needed). **Usage**: `throw new WsException('message')` or `throw new WsException({ error: 'CODE', message: 'text', statusCode: 400 })` for structured errors. **Sent to**: only the **triggering client** (not broadcast). **Client receives**: `socket.on('exception', error => { ... })` - error is the string/object passed to WsException. **Best practice**: create **custom error classes** extending WsException (ValidationError, UnauthorizedError, ForbiddenError, NotFoundError) with predefined structure `{ error: 'CODE', message, statusCode }`. **Error wrapping**: catch non-WsException errors, log them, wrap in WsException before rethrowing to hide internal details. **Logging**: always log errors server-side with context (user ID, event name, data, duration) before throwing WsException. **Structure**: include `error` (code string), `message` (human-readable), `statusCode` (HTTP-like code), and any relevant fields (`field`, `resourceId`, `retryAfter`). **vs throw Error**: regular `throw new Error()` crashes the handler and disconnects client, while `WsException` sends error gracefully without disconnection. **Production**: (1) create custom error classes, (2) validate input → throw ValidationError, (3) check auth → throw UnauthorizedError/ForbiddenError, (4) handle not found → throw NotFoundError, (5) wrap unexpected errors in generic WsException with 500 status, (6) log all errors, (7) client handles on 'exception' event.

</details>
34. How do you use Exception Filters with WebSockets?

<details>
<summary><strong>Answer</strong></summary>

**Exception Filters** for WebSockets intercept **all exceptions** thrown in gateway handlers (including `WsException` and regular errors) to provide **centralized error handling**. Use **`@Catch()`** decorator to specify which exceptions to catch (e.g., `@Catch(WsException)`, `@Catch()` for all). Implement **`ExceptionFilter<T>`** interface with **`catch(exception, host)`** method. Access Socket.IO client via **`host.switchToWs().getClient()`** to emit errors manually. Apply filter with **`@UseFilters()`** decorator on gateway class or specific handlers.

### **Basic Exception Filter**

```typescript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
} from '@nestjs/common';
import { WsException } from '@nestjs/websockets';
import { Socket } from 'socket.io';

// Catch WsException specifically
@Catch(WsException)
export class WsExceptionFilter implements ExceptionFilter<WsException> {
  catch(exception: WsException, host: ArgumentsHost) {
    // Get Socket.IO client
    const client = host.switchToWs().getClient<Socket>();

    // Get exception data
    const error = exception.getError();
    const details = typeof error === 'object' ? error : { message: error };

    // Emit error to client
    client.emit('exception', {
      ...details,
      timestamp: new Date().toISOString(),
    });
  }
}

// Apply to gateway
@WebSocketGateway()
@UseFilters(WsExceptionFilter)
export class BasicFilterGateway {
  @SubscribeMessage('test-error')
  handleTestError() {
    throw new WsException('Something went wrong');
    // Filter catches this and emits to client
  }
}
```

### **Catch All Exceptions**

```typescript
import { Logger } from '@nestjs/common';

// Catch ALL exceptions (WsException, Error, etc.)
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private logger = new Logger('WebSocketExceptions');

  catch(exception: any, host: ArgumentsHost) {
    const client = host.switchToWs().getClient<Socket>();
    const data = host.switchToWs().getData(); // Get message data

    // Log error
    this.logger.error(
      `WebSocket exception in client ${client.id}:`,
      exception.stack || exception.message,
    );

    // Handle different exception types
    let errorResponse: any;

    if (exception instanceof WsException) {
      // WsException
      const error = exception.getError();
      errorResponse = typeof error === 'object' ? error : { message: error };
    } else if (exception instanceof Error) {
      // Regular Error
      errorResponse = {
        error: 'INTERNAL_ERROR',
        message: exception.message,
        statusCode: 500,
      };
    } else {
      // Unknown error
      errorResponse = {
        error: 'UNKNOWN_ERROR',
        message: 'An unexpected error occurred',
        statusCode: 500,
      };
    }

    // Add timestamp
    errorResponse.timestamp = new Date().toISOString();

    // Emit to client
    client.emit('exception', errorResponse);
  }
}

@WebSocketGateway()
@UseFilters(AllExceptionsFilter)
export class CatchAllGateway {
  @SubscribeMessage('throw-ws-exception')
  handleWsException() {
    throw new WsException({ error: 'WS_ERROR', message: 'WsException thrown' });
  }

  @SubscribeMessage('throw-regular-error')
  handleRegularError() {
    throw new Error('Regular Error thrown');
  }

  @SubscribeMessage('throw-unknown')
  handleUnknown() {
    throw 'String error';
  }
}
```

### **Logging and Monitoring**

```typescript
import { Logger } from '@nestjs/common';

@Catch()
export class LoggingExceptionFilter implements ExceptionFilter {
  private logger = new Logger('WebSocketErrors');

  constructor(private errorTracker: ErrorTrackingService) {}

  catch(exception: any, host: ArgumentsHost) {
    const client = host.switchToWs().getClient<Socket>();
    const data = host.switchToWs().getData();
    const pattern = host.switchToWs().getPattern(); // Event name

    // Extract user info
    const user = client.data?.user;

    // Parse error
    let errorResponse: any;
    let errorCode: string;

    if (exception instanceof WsException) {
      const error = exception.getError();
      errorResponse = typeof error === 'object' ? error : { message: error };
      errorCode = errorResponse.error || 'WS_EXCEPTION';
    } else if (exception instanceof Error) {
      errorResponse = {
        error: 'INTERNAL_ERROR',
        message: exception.message,
        statusCode: 500,
      };
      errorCode = 'INTERNAL_ERROR';
    } else {
      errorResponse = {
        error: 'UNKNOWN_ERROR',
        message: String(exception),
        statusCode: 500,
      };
      errorCode = 'UNKNOWN_ERROR';
    }

    // LOG ERROR with context
    this.logger.error(
      `[${errorCode}] WebSocket error:\n` +
      `  Event: ${pattern}\n` +
      `  Client: ${client.id}\n` +
      `  User: ${user?.id || 'anonymous'}\n` +
      `  Data: ${JSON.stringify(data)}\n` +
      `  Message: ${errorResponse.message}`,
      exception.stack,
    );

    // TRACK ERROR (send to monitoring service)
    this.errorTracker.track({
      type: errorCode,
      event: pattern,
      clientId: client.id,
      userId: user?.id,
      message: errorResponse.message,
      data,
      timestamp: new Date(),
    });

    // Add metadata
    errorResponse.timestamp = new Date().toISOString();
    errorResponse.eventName = pattern;

    // Emit to client
    client.emit('exception', errorResponse);
  }
}

class ErrorTrackingService {
  track(data: any) {
    // Send to Sentry, DataDog, etc.
    console.error('[ERROR TRACKER]', data);
  }
}
```

### **Custom Error Responses**

```typescript
@Catch()
export class CustomResponseFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const client = host.switchToWs().getClient<Socket>();

    let response: any;

    if (exception instanceof WsException) {
      const error = exception.getError();
      const errorData = typeof error === 'object' ? error : { message: error };

      // Customize based on error code
      if (errorData.error === 'VALIDATION_ERROR') {
        response = {
          type: 'validation',
          field: errorData.field,
          message: errorData.message,
          constraints: errorData.constraints,
          timestamp: new Date().toISOString(),
        };
      } else if (errorData.error === 'UNAUTHORIZED') {
        response = {
          type: 'authentication',
          message: 'Please log in to continue',
          redirectTo: '/login',
          timestamp: new Date().toISOString(),
        };
      } else if (errorData.error === 'FORBIDDEN') {
        response = {
          type: 'authorization',
          message: errorData.message,
          requiredRole: errorData.requiredRole,
          timestamp: new Date().toISOString(),
        };
      } else if (errorData.error === 'NOT_FOUND') {
        response = {
          type: 'not_found',
          resource: errorData.resource,
          id: errorData.id,
          message: errorData.message,
          timestamp: new Date().toISOString(),
        };
      } else {
        // Generic WsException
        response = {
          ...errorData,
          timestamp: new Date().toISOString(),
        };
      }
    } else {
      // Non-WsException
      response = {
        type: 'server_error',
        message: 'An unexpected error occurred',
        timestamp: new Date().toISOString(),
      };
    }

    // Emit custom response
    client.emit('error', response);
  }
}
```

### **Per-Handler Filters**

```typescript
// Filter for validation errors
@Catch(WsException)
export class ValidationFilter implements ExceptionFilter {
  catch(exception: WsException, host: ArgumentsHost) {
    const client = host.switchToWs().getClient<Socket>();
    const error = exception.getError();
    const errorData = typeof error === 'object' ? error : { message: error };

    if (errorData.error === 'VALIDATION_ERROR') {
      client.emit('validation-error', {
        field: errorData.field,
        message: errorData.message,
        value: errorData.value,
      });
    } else {
      client.emit('exception', errorData);
    }
  }
}

@WebSocketGateway()
export class PerHandlerFiltersGateway {
  // Apply filter to specific handler
  @UseFilters(ValidationFilter)
  @SubscribeMessage('create-user')
  handleCreateUser(@MessageBody() data: { username: string }) {
    if (!data.username || data.username.length < 3) {
      throw new WsException({
        error: 'VALIDATION_ERROR',
        field: 'username',
        message: 'Username must be at least 3 characters',
        value: data.username,
      });
    }

    return { event: 'user-created', data: { username: data.username } };
  }

  // This handler doesn't use the filter
  @SubscribeMessage('other-event')
  handleOther() {
    throw new WsException('Regular error');
  }
}
```

### **Multiple Filters**

```typescript
// Filter 1: Handle specific errors
@Catch(WsException)
export class WsExceptionOnlyFilter implements ExceptionFilter {
  catch(exception: WsException, host: ArgumentsHost) {
    const client = host.switchToWs().getClient<Socket>();
    const error = exception.getError();

    client.emit('ws-exception', {
      error: typeof error === 'object' ? error : { message: error },
      timestamp: new Date().toISOString(),
    });
  }
}

// Filter 2: Handle all other errors
@Catch()
export class FallbackFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const client = host.switchToWs().getClient<Socket>();

    if (!(exception instanceof WsException)) {
      client.emit('internal-error', {
        message: 'An internal error occurred',
        timestamp: new Date().toISOString(),
      });
    }
  }
}

// Apply multiple filters (evaluated in order)
@WebSocketGateway()
@UseFilters(WsExceptionOnlyFilter, FallbackFilter)
export class MultipleFiltersGateway {
  @SubscribeMessage('test')
  handleTest() {
    throw new WsException('WsException'); // Caught by WsExceptionOnlyFilter
  }

  @SubscribeMessage('test2')
  handleTest2() {
    throw new Error('Regular error'); // Caught by FallbackFilter
  }
}
```

### **Global Exception Filter**

```typescript
// main.ts or app.module.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Apply filter globally (affects all WebSocket gateways)
  app.useGlobalFilters(new AllExceptionsFilter());

  await app.listen(3000);
}
bootstrap();

// OR in AppModule with dependency injection
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: AllExceptionsFilter,
    },
  ],
})
export class AppModule {}
```

### **Extracting Context Information**

```typescript
@Catch()
export class ContextExtractionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    // Get WebSocket context
    const wsContext = host.switchToWs();

    // Get Socket.IO client
    const client = wsContext.getClient<Socket>();

    // Get message data (arguments sent by client)
    const data = wsContext.getData();

    // Get event pattern (event name)
    const pattern = wsContext.getPattern();

    // Get user from socket data
    const user = client.data?.user;

    // Get connection info
    const address = client.handshake?.address;
    const userAgent = client.handshake?.headers?.['user-agent'];

    console.log('Error context:', {
      clientId: client.id,
      userId: user?.id,
      event: pattern,
      data,
      address,
      userAgent,
      error: exception.message,
    });

    // Emit error
    client.emit('exception', {
      message: 'An error occurred',
      eventName: pattern,
      timestamp: new Date().toISOString(),
    });
  }
}
```

### **Production Exception Filter**

```typescript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  Logger,
} from '@nestjs/common';
import { WsException } from '@nestjs/websockets';
import { Socket } from 'socket.io';

@Catch()
export class ProductionExceptionFilter implements ExceptionFilter {
  private logger = new Logger('WebSocketExceptions');

  constructor(
    private errorTracker: ErrorTrackingService,
    private configService: ConfigService,
  ) {}

  catch(exception: any, host: ArgumentsHost) {
    const client = host.switchToWs().getClient<Socket>();
    const data = host.switchToWs().getData();
    const pattern = host.switchToWs().getPattern();
    const user = client.data?.user;

    // PARSE EXCEPTION
    let errorResponse: any;
    let errorCode: string;
    let statusCode: number;

    if (exception instanceof WsException) {
      const error = exception.getError();
      errorResponse = typeof error === 'object' ? error : { message: error };
      errorCode = errorResponse.error || 'WS_EXCEPTION';
      statusCode = errorResponse.statusCode || 400;
    } else if (exception instanceof Error) {
      errorResponse = {
        error: 'INTERNAL_ERROR',
        message: this.configService.isProduction()
          ? 'An internal error occurred'
          : exception.message,
        statusCode: 500,
      };
      errorCode = 'INTERNAL_ERROR';
      statusCode = 500;
    } else {
      errorResponse = {
        error: 'UNKNOWN_ERROR',
        message: 'An unexpected error occurred',
        statusCode: 500,
      };
      errorCode = 'UNKNOWN_ERROR';
      statusCode = 500;
    }

    // LOG ERROR (different levels based on status)
    const logMessage =
      `[${errorCode}] WebSocket error in event "${pattern}":\n` +
      `  Client: ${client.id}\n` +
      `  User: ${user?.id || 'anonymous'}\n` +
      `  Message: ${errorResponse.message}`;

    if (statusCode >= 500) {
      this.logger.error(logMessage, exception.stack);
    } else if (statusCode >= 400) {
      this.logger.warn(logMessage);
    } else {
      this.logger.log(logMessage);
    }

    // TRACK ERROR in monitoring service (only for 500 errors)
    if (statusCode >= 500) {
      this.errorTracker.track({
        type: errorCode,
        event: pattern,
        clientId: client.id,
        userId: user?.id,
        message: errorResponse.message,
        data,
        stack: exception.stack,
        timestamp: new Date(),
      });
    }

    // ADD METADATA
    errorResponse.timestamp = new Date().toISOString();
    errorResponse.eventName = pattern;

    // Remove sensitive data in production
    if (this.configService.isProduction() && statusCode >= 500) {
      delete errorResponse.stack;
      delete errorResponse.originalError;
    }

    // EMIT TO CLIENT
    client.emit('exception', errorResponse);
  }
}

class ErrorTrackingService {
  track(data: any) {
    // Send to Sentry, DataDog, etc.
    console.error('[ERROR TRACKER]', data);
  }
}

class ConfigService {
  isProduction() {
    return process.env.NODE_ENV === 'production';
  }
}

// Apply globally
@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: ProductionExceptionFilter,
    },
    ErrorTrackingService,
    ConfigService,
  ],
})
export class AppModule {}
```

**Interview Tip**: **Exception Filters** provide **centralized WebSocket error handling** - intercept all exceptions thrown in gateway handlers. **Implementation**: (1) create class with `@Catch()` decorator (e.g., `@Catch(WsException)` for specific, `@Catch()` for all exceptions), (2) implement `ExceptionFilter<T>` interface with `catch(exception, host)` method, (3) use `host.switchToWs().getClient<Socket>()` to get Socket.IO client, (4) manually emit error to client with `client.emit('exception', errorData)`. **Application**: `@UseFilters(MyFilter)` on gateway class (all handlers) or specific handler method. **Global**: `app.useGlobalFilters(new MyFilter())` in main.ts or `APP_FILTER` provider in AppModule. **Benefits**: (1) centralized error handling logic, (2) consistent error format, (3) logging/monitoring in one place, (4) separate concerns (handlers throw, filters handle), (5) custom error responses per error type. **Context extraction**: `host.switchToWs().getClient()` (Socket), `getData()` (message data), `getPattern()` (event name), `client.data.user` (authenticated user). **Production pattern**: (1) catch all exceptions with `@Catch()`, (2) log errors with Logger (different levels for 4xx vs 5xx), (3) track 500 errors in monitoring service (Sentry), (4) hide internal details in production (generic message for 500 errors), (5) emit structured error to client with timestamp and event name, (6) apply filter globally for consistency. **Multiple filters**: apply multiple filters in order, first matching filter handles exception. **vs try-catch**: filters provide centralized handling, try-catch is for specific handler logic. Use filters for logging, monitoring, and consistent error responses.

</details>

## Real-Time Use Cases

35. How do you implement a chat application?

<details>
<summary><strong>Answer</strong></summary>

A complete chat application requires: **(1) Gateway** with event handlers for sending/receiving messages, **(2) Rooms** for chat channels (join on connection, emit to room), **(3) Database service** to persist messages, **(4) Authentication** to identify users (JWT in middleware), **(5) Message history** loading on join, **(6) Broadcasting** new messages to all room members, **(7) User presence** (online/offline status), **(8) Typing indicators**, and **(9) Client-side** Socket.IO integration. Use **`@SubscribeMessage()`** for events (send-message, load-history, join-room), **`server.to(roomId).emit()`** for broadcasting to rooms, and **database persistence** for message history.

### **Complete Chat Gateway**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
  OnGatewayInit,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger, UseGuards } from '@nestjs/common';
import { WsException } from '@nestjs/websockets';

@WebSocketGateway({
  cors: { origin: '*' },
  namespace: '/chat',
})
export class ChatGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  @WebSocketServer()
  server: Server;

  private logger = new Logger('ChatGateway');

  constructor(
    private messageService: MessageService,
    private userService: UserService,
    private roomService: RoomService,
  ) {}

  // INITIALIZATION
  afterInit(server: Server) {
    // Authentication middleware
    server.use((socket, next) => {
      try {
        const token = socket.handshake.auth.token;

        if (!token) {
          return next(new Error('Authentication required'));
        }

        // Validate token
        const user = this.validateToken(token);
        socket.data.user = user;
        next();
      } catch (error) {
        next(new Error('Invalid token'));
      }
    });

    this.logger.log('Chat Gateway initialized');
  }

  // CONNECTION
  async handleConnection(client: Socket) {
    const user = client.data.user;
    this.logger.log(`User connected: ${user.username} (${client.id})`);

    // Set user online
    await this.userService.setOnline(user.id, true);

    // Broadcast user online status
    this.server.emit('user-status', {
      userId: user.id,
      username: user.username,
      status: 'online',
    });
  }

  // DISCONNECTION
  async handleDisconnect(client: Socket) {
    const user = client.data.user;
    this.logger.log(`User disconnected: ${user.username} (${client.id})`);

    // Set user offline
    await this.userService.setOnline(user.id, false);

    // Broadcast user offline status
    this.server.emit('user-status', {
      userId: user.id,
      username: user.username,
      status: 'offline',
      lastSeen: new Date(),
    });
  }

  // JOIN ROOM (chat channel)
  @SubscribeMessage('join-room')
  async handleJoinRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;

    try {
      // Check if user has access to room
      const isMember = await this.roomService.isMember(user.id, roomId);

      if (!isMember) {
        throw new WsException({
          error: 'FORBIDDEN',
          message: 'You are not a member of this room',
        });
      }

      // Join Socket.IO room
      client.join(roomId);

      // Load message history
      const messages = await this.messageService.getHistory(roomId, 50);

      // Get room members
      const members = await this.roomService.getMembers(roomId);

      // Send history to joining user
      client.emit('room-joined', {
        roomId,
        messages,
        members,
      });

      // Notify others in room
      client.to(roomId).emit('user-joined-room', {
        roomId,
        user: {
          id: user.id,
          username: user.username,
        },
      });

      this.logger.log(`User ${user.username} joined room ${roomId}`);

      return {
        event: 'join-room-success',
        data: { roomId },
      };
    } catch (error) {
      if (error instanceof WsException) {
        throw error;
      }
      throw new WsException('Failed to join room');
    }
  }

  // LEAVE ROOM
  @SubscribeMessage('leave-room')
  async handleLeaveRoom(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;

    client.leave(roomId);

    // Notify others
    client.to(roomId).emit('user-left-room', {
      roomId,
      user: {
        id: user.id,
        username: user.username,
      },
    });

    this.logger.log(`User ${user.username} left room ${roomId}`);

    return {
      event: 'leave-room-success',
      data: { roomId },
    };
  }

  // SEND MESSAGE
  @SubscribeMessage('send-message')
  async handleSendMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;

    try {
      // Validation
      if (!data.message || data.message.trim().length === 0) {
        throw new WsException({
          error: 'VALIDATION_ERROR',
          message: 'Message cannot be empty',
        });
      }

      if (data.message.length > 1000) {
        throw new WsException({
          error: 'VALIDATION_ERROR',
          message: 'Message too long (max 1000 characters)',
        });
      }

      // Check membership
      const isMember = await this.roomService.isMember(user.id, data.roomId);

      if (!isMember) {
        throw new WsException({
          error: 'FORBIDDEN',
          message: 'You are not a member of this room',
        });
      }

      // Save message to database
      const message = await this.messageService.create({
        userId: user.id,
        roomId: data.roomId,
        content: data.message,
      });

      // Prepare message data
      const messageData = {
        id: message.id,
        content: message.content,
        roomId: message.roomId,
        user: {
          id: user.id,
          username: user.username,
          avatar: user.avatar,
        },
        createdAt: message.createdAt,
      };

      // Broadcast to all users in room (including sender)
      this.server.to(data.roomId).emit('new-message', messageData);

      this.logger.log(
        `Message sent by ${user.username} in room ${data.roomId}`,
      );

      return {
        event: 'message-sent',
        data: { messageId: message.id },
      };
    } catch (error) {
      if (error instanceof WsException) {
        throw error;
      }
      throw new WsException('Failed to send message');
    }
  }

  // TYPING INDICATOR
  @SubscribeMessage('typing')
  handleTyping(
    @MessageBody() data: { roomId: string; isTyping: boolean },
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;

    // Broadcast typing status to others in room (not sender)
    client.to(data.roomId).emit('user-typing', {
      roomId: data.roomId,
      user: {
        id: user.id,
        username: user.username,
      },
      isTyping: data.isTyping,
    });
  }

  // LOAD MORE MESSAGES (pagination)
  @SubscribeMessage('load-more-messages')
  async handleLoadMore(
    @MessageBody() data: { roomId: string; before: Date; limit: number },
    @ConnectedSocket() client: Socket,
  ) {
    try {
      const messages = await this.messageService.getHistory(
        data.roomId,
        data.limit || 50,
        data.before,
      );

      client.emit('messages-loaded', {
        roomId: data.roomId,
        messages,
        hasMore: messages.length === data.limit,
      });
    } catch (error) {
      throw new WsException('Failed to load messages');
    }
  }

  // DELETE MESSAGE
  @SubscribeMessage('delete-message')
  async handleDeleteMessage(
    @MessageBody() data: { roomId: string; messageId: string },
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;

    try {
      // Get message
      const message = await this.messageService.findById(data.messageId);

      if (!message) {
        throw new WsException({
          error: 'NOT_FOUND',
          message: 'Message not found',
        });
      }

      // Check ownership or admin
      if (message.userId !== user.id && user.role !== 'admin') {
        throw new WsException({
          error: 'FORBIDDEN',
          message: 'You can only delete your own messages',
        });
      }

      // Delete message
      await this.messageService.delete(data.messageId);

      // Broadcast deletion
      this.server.to(data.roomId).emit('message-deleted', {
        roomId: data.roomId,
        messageId: data.messageId,
      });

      return {
        event: 'message-deleted-success',
        data: { messageId: data.messageId },
      };
    } catch (error) {
      if (error instanceof WsException) {
        throw error;
      }
      throw new WsException('Failed to delete message');
    }
  }

  private validateToken(token: string): any {
    // JWT validation (simplified)
    if (token === 'valid-token') {
      return {
        id: '123',
        username: 'john_doe',
        avatar: 'https://example.com/avatar.jpg',
        role: 'user',
      };
    }
    throw new Error('Invalid token');
  }
}
```

### **Services (Database Layer)**

```typescript
import { Injectable } from '@nestjs/common';

export interface Message {
  id: string;
  userId: string;
  roomId: string;
  content: string;
  createdAt: Date;
  updatedAt: Date;
}

@Injectable()
export class MessageService {
  // Create message
  async create(data: {
    userId: string;
    roomId: string;
    content: string;
  }): Promise<Message> {
    // Save to database (PostgreSQL, MongoDB, etc.)
    const message: Message = {
      id: this.generateId(),
      userId: data.userId,
      roomId: data.roomId,
      content: data.content,
      createdAt: new Date(),
      updatedAt: new Date(),
    };

    // await this.db.messages.create(message);
    return message;
  }

  // Get message history
  async getHistory(
    roomId: string,
    limit: number = 50,
    before?: Date,
  ): Promise<Message[]> {
    // Query database
    // const messages = await this.db.messages.find({
    //   roomId,
    //   createdAt: before ? { $lt: before } : undefined,
    // }).sort({ createdAt: -1 }).limit(limit);

    // Mock data
    return [
      {
        id: '1',
        userId: '123',
        roomId,
        content: 'Hello!',
        createdAt: new Date(),
        updatedAt: new Date(),
      },
    ];
  }

  // Find message by ID
  async findById(id: string): Promise<Message | null> {
    // return await this.db.messages.findOne({ id });
    return id === '123'
      ? {
          id,
          userId: '123',
          roomId: 'room1',
          content: 'Message',
          createdAt: new Date(),
          updatedAt: new Date(),
        }
      : null;
  }

  // Delete message
  async delete(id: string): Promise<void> {
    // await this.db.messages.delete({ id });
  }

  private generateId(): string {
    return Math.random().toString(36).substr(2, 9);
  }
}

@Injectable()
export class UserService {
  // Set user online/offline
  async setOnline(userId: string, isOnline: boolean): Promise<void> {
    // await this.db.users.update({ id: userId }, {
    //   isOnline,
    //   lastSeen: new Date(),
    // });
  }
}

@Injectable()
export class RoomService {
  // Check if user is member of room
  async isMember(userId: string, roomId: string): Promise<boolean> {
    // const membership = await this.db.roomMembers.findOne({
    //   userId,
    //   roomId,
    // });
    // return !!membership;
    return true; // Mock
  }

  // Get room members
  async getMembers(roomId: string): Promise<any[]> {
    // return await this.db.users.find({
    //   rooms: { $contains: roomId },
    // });
    return [
      { id: '123', username: 'john_doe', status: 'online' },
      { id: '456', username: 'jane_doe', status: 'offline' },
    ];
  }
}
```

### **Client-Side (React/TypeScript)**

```typescript
import { useEffect, useState, useRef } from 'react';
import { io, Socket } from 'socket.io-client';

interface Message {
  id: string;
  content: string;
  user: {
    id: string;
    username: string;
    avatar: string;
  };
  createdAt: Date;
}

export function Chat({ roomId, token }: { roomId: string; token: string }) {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [messages, setMessages] = useState<Message[]>([]);
  const [inputMessage, setInputMessage] = useState('');
  const [isTyping, setIsTyping] = useState(false);
  const [typingUsers, setTypingUsers] = useState<string[]>([]);
  const typingTimeoutRef = useRef<NodeJS.Timeout | null>(null);

  // CONNECT TO SOCKET
  useEffect(() => {
    const newSocket = io('http://localhost:3000/chat', {
      auth: { token },
    });

    setSocket(newSocket);

    return () => {
      newSocket.close();
    };
  }, [token]);

  // JOIN ROOM & SETUP LISTENERS
  useEffect(() => {
    if (!socket) return;

    // Join room
    socket.emit('join-room', roomId);

    // RECEIVE: Room joined with history
    socket.on('room-joined', (data) => {
      console.log('Joined room:', data.roomId);
      setMessages(data.messages);
    });

    // RECEIVE: New message
    socket.on('new-message', (message: Message) => {
      setMessages((prev) => [...prev, message]);
    });

    // RECEIVE: Message deleted
    socket.on('message-deleted', (data) => {
      setMessages((prev) => prev.filter((m) => m.id !== data.messageId));
    });

    // RECEIVE: User typing
    socket.on('user-typing', (data) => {
      if (data.isTyping) {
        setTypingUsers((prev) => [...new Set([...prev, data.user.username])]);
      } else {
        setTypingUsers((prev) => prev.filter((u) => u !== data.user.username));
      }
    });

    // RECEIVE: User joined/left
    socket.on('user-joined-room', (data) => {
      console.log(`${data.user.username} joined the room`);
    });

    socket.on('user-left-room', (data) => {
      console.log(`${data.user.username} left the room`);
    });

    // RECEIVE: Errors
    socket.on('exception', (error) => {
      console.error('Error:', error);
      alert(error.message);
    });

    return () => {
      socket.emit('leave-room', roomId);
      socket.off('room-joined');
      socket.off('new-message');
      socket.off('message-deleted');
      socket.off('user-typing');
      socket.off('user-joined-room');
      socket.off('user-left-room');
      socket.off('exception');
    };
  }, [socket, roomId]);

  // SEND MESSAGE
  const sendMessage = () => {
    if (!socket || !inputMessage.trim()) return;

    socket.emit('send-message', {
      roomId,
      message: inputMessage,
    });

    setInputMessage('');
    setIsTyping(false);
    socket.emit('typing', { roomId, isTyping: false });
  };

  // TYPING INDICATOR
  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setInputMessage(e.target.value);

    if (!socket) return;

    // Start typing
    if (!isTyping) {
      setIsTyping(true);
      socket.emit('typing', { roomId, isTyping: true });
    }

    // Reset timeout
    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
    }

    // Stop typing after 2 seconds
    typingTimeoutRef.current = setTimeout(() => {
      setIsTyping(false);
      socket.emit('typing', { roomId, isTyping: false });
    }, 2000);
  };

  // DELETE MESSAGE
  const deleteMessage = (messageId: string) => {
    if (!socket) return;
    socket.emit('delete-message', { roomId, messageId });
  };

  return (
    <div>
      <h2>Chat Room: {roomId}</h2>

      {/* Messages */}
      <div style={{ height: '400px', overflowY: 'auto', border: '1px solid #ccc' }}>
        {messages.map((msg) => (
          <div key={msg.id} style={{ padding: '10px', borderBottom: '1px solid #eee' }}>
            <strong>{msg.user.username}:</strong> {msg.content}
            <button onClick={() => deleteMessage(msg.id)}>Delete</button>
            <small>{new Date(msg.createdAt).toLocaleTimeString()}</small>
          </div>
        ))}
      </div>

      {/* Typing indicator */}
      {typingUsers.length > 0 && (
        <div style={{ fontStyle: 'italic', color: '#666' }}>
          {typingUsers.join(', ')} {typingUsers.length === 1 ? 'is' : 'are'} typing...
        </div>
      )}

      {/* Input */}
      <div style={{ marginTop: '10px' }}>
        <input
          type="text"
          value={inputMessage}
          onChange={handleInputChange}
          onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
          placeholder="Type a message..."
          style={{ width: '80%', padding: '10px' }}
        />
        <button onClick={sendMessage} style={{ width: '18%', padding: '10px' }}>
          Send
        </button>
      </div>
    </div>
  );
}
```

### **Module Setup**

```typescript
import { Module } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';
import { MessageService } from './message.service';
import { UserService } from './user.service';
import { RoomService } from './room.service';

@Module({
  providers: [ChatGateway, MessageService, UserService, RoomService],
})
export class ChatModule {}
```

**Interview Tip**: **Chat application** with WebSockets: **(1) Gateway** - `@WebSocketGateway({ namespace: '/chat' })` with event handlers for send-message, join-room, leave-room, typing, load-more, delete-message; **(2) Authentication** - middleware in `afterInit()` validates JWT token from `socket.handshake.auth.token`, stores user in `socket.data.user`; **(3) Rooms** - `client.join(roomId)` on join-room event, `server.to(roomId).emit('new-message', data)` broadcasts to all room members; **(4) Database** - services (MessageService, RoomService, UserService) handle persistence (create message, load history, check membership); **(5) Message flow** - client emits 'send-message' → server validates → saves to DB → broadcasts to room via `server.to(roomId).emit()`; **(6) History** - load on join with `messageService.getHistory(roomId, 50)`, pagination with 'load-more-messages' event (before: Date, limit); **(7) Typing indicators** - client emits 'typing' with isTyping boolean, server broadcasts to others with `client.to(roomId).emit()` (not sender); **(8) Presence** - `handleConnection` sets user online, broadcasts status; `handleDisconnect` sets offline; **(9) Client** - Socket.IO client connects with auth token, listens for 'new-message', 'room-joined', 'user-typing', emits 'send-message', 'join-room'; **(10) Features** - message deletion (check ownership), user joined/left notifications, error handling with WsException. **Production additions**: message editing, reactions, file uploads, read receipts, mentions, search, notifications.

</details>
36. How do you implement real-time notifications?

<details>
<summary><strong>Answer</strong></summary>

Real-time notifications use WebSockets to push updates to users instantly. Implementation requires: **(1) Notification Gateway** with user-specific rooms (`user:${userId}`), **(2) Service layer** to create notifications and trigger WebSocket events, **(3) Joining user rooms** on connection (each user joins their own room `user:${userId}`), **(4) Targeted emission** to specific users via `server.to(`user:${userId}`).emit('notification', data)`, **(5) Notification persistence** in database, **(6) Delivery tracking** (read/unread status), **(7) Different notification types** (comment, like, mention, system), and **(8) Client-side** listener to display real-time updates (toast, badge, list).

### **Notification Gateway**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  OnGatewayConnection,
  OnGatewayDisconnect,
  OnGatewayInit,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway({
  cors: { origin: '*' },
  namespace: '/notifications',
})
export class NotificationGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  @WebSocketServer()
  server: Server;

  private logger = new Logger('NotificationGateway');

  constructor(
    private notificationService: NotificationService,
    private authService: AuthService,
  ) {}

  // INITIALIZATION: Auth middleware
  afterInit(server: Server) {
    server.use((socket, next) => {
      try {
        const token = socket.handshake.auth.token;

        if (!token) {
          return next(new Error('Authentication required'));
        }

        const user = this.authService.validateToken(token);
        socket.data.user = user;
        next();
      } catch (error) {
        next(new Error('Invalid token'));
      }
    });

    this.logger.log('Notification Gateway initialized');
  }

  // CONNECTION: Join user-specific room
  async handleConnection(client: Socket) {
    const user = client.data.user;

    // Join user's personal room
    const userRoom = `user:${user.id}`;
    client.join(userRoom);

    this.logger.log(
      `User ${user.username} connected to notifications (room: ${userRoom})`,
    );

    // Send unread notification count
    const unreadCount = await this.notificationService.getUnreadCount(user.id);
    client.emit('unread-count', { count: unreadCount });

    // Send recent notifications
    const recentNotifications = await this.notificationService.getRecent(
      user.id,
      10,
    );
    client.emit('recent-notifications', recentNotifications);
  }

  // DISCONNECTION
  handleDisconnect(client: Socket) {
    const user = client.data.user;
    this.logger.log(`User ${user.username} disconnected from notifications`);
  }

  // GET NOTIFICATIONS (pagination)
  @SubscribeMessage('get-notifications')
  async handleGetNotifications(
    @MessageBody() data: { page: number; limit: number },
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;

    const notifications = await this.notificationService.getAll(
      user.id,
      data.page || 1,
      data.limit || 20,
    );

    return {
      event: 'notifications-loaded',
      data: notifications,
    };
  }

  // MARK AS READ
  @SubscribeMessage('mark-read')
  async handleMarkRead(
    @MessageBody() notificationId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;

    await this.notificationService.markAsRead(notificationId, user.id);

    // Update unread count
    const unreadCount = await this.notificationService.getUnreadCount(user.id);
    client.emit('unread-count', { count: unreadCount });

    return {
      event: 'mark-read-success',
      data: { notificationId },
    };
  }

  // MARK ALL AS READ
  @SubscribeMessage('mark-all-read')
  async handleMarkAllRead(@ConnectedSocket() client: Socket) {
    const user = client.data.user;

    await this.notificationService.markAllAsRead(user.id);

    client.emit('unread-count', { count: 0 });

    return {
      event: 'mark-all-read-success',
      data: {},
    };
  }

  // DELETE NOTIFICATION
  @SubscribeMessage('delete-notification')
  async handleDeleteNotification(
    @MessageBody() notificationId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;

    await this.notificationService.delete(notificationId, user.id);

    return {
      event: 'delete-notification-success',
      data: { notificationId },
    };
  }

  // PUBLIC METHOD: Send notification to specific user
  async sendToUser(userId: string, notification: any) {
    const userRoom = `user:${userId}`;
    
    // Emit to user's room
    this.server.to(userRoom).emit('notification', notification);

    // Update unread count
    const unreadCount = await this.notificationService.getUnreadCount(userId);
    this.server.to(userRoom).emit('unread-count', { count: unreadCount });

    this.logger.log(`Notification sent to user ${userId}`);
  }

  // PUBLIC METHOD: Send to multiple users
  async sendToUsers(userIds: string[], notification: any) {
    for (const userId of userIds) {
      await this.sendToUser(userId, notification);
    }
  }

  // PUBLIC METHOD: Broadcast to all connected users
  broadcastToAll(notification: any) {
    this.server.emit('notification', notification);
    this.logger.log('Notification broadcast to all users');
  }
}
```

### **Notification Service**

```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

export interface Notification {
  id: string;
  userId: string;
  type: 'comment' | 'like' | 'mention' | 'follow' | 'system';
  title: string;
  message: string;
  data?: any; // Extra data (e.g., postId, commentId)
  isRead: boolean;
  createdAt: Date;
}

@Injectable()
export class NotificationService {
  constructor(
    private notificationGateway: NotificationGateway,
    // @InjectRepository(Notification)
    // private notificationRepo: Repository<Notification>,
  ) {}

  // CREATE AND SEND NOTIFICATION
  async create(
    userId: string,
    type: Notification['type'],
    title: string,
    message: string,
    data?: any,
  ): Promise<Notification> {
    // Save to database
    const notification: Notification = {
      id: this.generateId(),
      userId,
      type,
      title,
      message,
      data,
      isRead: false,
      createdAt: new Date(),
    };

    // await this.notificationRepo.save(notification);

    // Send via WebSocket (real-time)
    await this.notificationGateway.sendToUser(userId, notification);

    return notification;
  }

  // CREATE MULTIPLE NOTIFICATIONS
  async createForUsers(
    userIds: string[],
    type: Notification['type'],
    title: string,
    message: string,
    data?: any,
  ) {
    const notifications = userIds.map((userId) => ({
      id: this.generateId(),
      userId,
      type,
      title,
      message,
      data,
      isRead: false,
      createdAt: new Date(),
    }));

    // await this.notificationRepo.save(notifications);

    // Send via WebSocket
    await this.notificationGateway.sendToUsers(userIds, notifications[0]);
  }

  // GET ALL NOTIFICATIONS (paginated)
  async getAll(
    userId: string,
    page: number = 1,
    limit: number = 20,
  ): Promise<Notification[]> {
    // const notifications = await this.notificationRepo.find({
    //   where: { userId },
    //   order: { createdAt: 'DESC' },
    //   skip: (page - 1) * limit,
    //   take: limit,
    // });

    // Mock data
    return [
      {
        id: '1',
        userId,
        type: 'comment',
        title: 'New Comment',
        message: 'John commented on your post',
        isRead: false,
        createdAt: new Date(),
      },
    ];
  }

  // GET RECENT NOTIFICATIONS
  async getRecent(userId: string, limit: number = 10): Promise<Notification[]> {
    // return await this.notificationRepo.find({
    //   where: { userId },
    //   order: { createdAt: 'DESC' },
    //   take: limit,
    // });
    return [];
  }

  // GET UNREAD COUNT
  async getUnreadCount(userId: string): Promise<number> {
    // return await this.notificationRepo.count({
    //   where: { userId, isRead: false },
    // });
    return 5; // Mock
  }

  // MARK AS READ
  async markAsRead(notificationId: string, userId: string): Promise<void> {
    // await this.notificationRepo.update(
    //   { id: notificationId, userId },
    //   { isRead: true },
    // );
  }

  // MARK ALL AS READ
  async markAllAsRead(userId: string): Promise<void> {
    // await this.notificationRepo.update({ userId }, { isRead: true });
  }

  // DELETE NOTIFICATION
  async delete(notificationId: string, userId: string): Promise<void> {
    // await this.notificationRepo.delete({ id: notificationId, userId });
  }

  private generateId(): string {
    return Math.random().toString(36).substr(2, 9);
  }
}
```

### **Usage Examples (Triggering Notifications)**

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class PostService {
  constructor(private notificationService: NotificationService) {}

  // When someone comments on a post
  async createComment(
    postId: string,
    authorId: string,
    content: string,
  ): Promise<void> {
    // Save comment to database
    // ...

    // Get post owner
    const post = await this.findPostById(postId);

    // Send notification to post owner
    if (post.ownerId !== authorId) {
      await this.notificationService.create(
        post.ownerId,
        'comment',
        'New Comment',
        `${authorId} commented on your post`,
        { postId, commentId: 'comment-123' },
      );
    }
  }

  // When someone likes a post
  async likePost(postId: string, userId: string): Promise<void> {
    // Save like
    // ...

    const post = await this.findPostById(postId);

    // Send notification to post owner
    if (post.ownerId !== userId) {
      await this.notificationService.create(
        post.ownerId,
        'like',
        'New Like',
        `${userId} liked your post`,
        { postId },
      );
    }
  }

  // When someone mentions users in a post
  async createPostWithMentions(
    content: string,
    mentionedUserIds: string[],
  ): Promise<void> {
    // Save post
    const postId = 'post-123';

    // Send notification to all mentioned users
    await this.notificationService.createForUsers(
      mentionedUserIds,
      'mention',
      'You were mentioned',
      'You were mentioned in a post',
      { postId },
    );
  }

  private async findPostById(id: string): Promise<any> {
    return { id, ownerId: 'owner-123' };
  }
}

@Injectable()
export class SystemService {
  constructor(private notificationService: NotificationService) {}

  // System-wide notification (maintenance, announcements)
  async sendSystemNotification(message: string): Promise<void> {
    // Get all active user IDs
    const userIds = await this.getAllUserIds();

    // Send to all users
    await this.notificationService.createForUsers(
      userIds,
      'system',
      'System Announcement',
      message,
    );
  }

  private async getAllUserIds(): Promise<string[]> {
    // Query all users
    return ['user1', 'user2', 'user3'];
  }
}
```

### **Client-Side (React/TypeScript)**

```typescript
import { useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';
import { toast } from 'react-toastify'; // For toast notifications

interface Notification {
  id: string;
  type: string;
  title: string;
  message: string;
  isRead: boolean;
  createdAt: Date;
}

export function useNotifications(token: string) {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);

  // CONNECT
  useEffect(() => {
    const newSocket = io('http://localhost:3000/notifications', {
      auth: { token },
    });

    setSocket(newSocket);

    return () => {
      newSocket.close();
    };
  }, [token]);

  // LISTEN FOR EVENTS
  useEffect(() => {
    if (!socket) return;

    // RECEIVE: Unread count
    socket.on('unread-count', (data) => {
      setUnreadCount(data.count);
    });

    // RECEIVE: Recent notifications (on connect)
    socket.on('recent-notifications', (data) => {
      setNotifications(data);
    });

    // RECEIVE: New notification (real-time)
    socket.on('notification', (notification: Notification) => {
      // Add to list
      setNotifications((prev) => [notification, ...prev]);

      // Show toast
      toast.info(`${notification.title}: ${notification.message}`);

      // Play sound
      playNotificationSound();
    });

    // RECEIVE: Notifications loaded (pagination)
    socket.on('notifications-loaded', (data) => {
      setNotifications(data);
    });

    return () => {
      socket.off('unread-count');
      socket.off('recent-notifications');
      socket.off('notification');
      socket.off('notifications-loaded');
    };
  }, [socket]);

  // MARK AS READ
  const markAsRead = (notificationId: string) => {
    if (!socket) return;
    socket.emit('mark-read', notificationId);

    // Update UI
    setNotifications((prev) =>
      prev.map((n) => (n.id === notificationId ? { ...n, isRead: true } : n)),
    );
  };

  // MARK ALL AS READ
  const markAllAsRead = () => {
    if (!socket) return;
    socket.emit('mark-all-read');

    setNotifications((prev) => prev.map((n) => ({ ...n, isRead: true })));
  };

  // DELETE NOTIFICATION
  const deleteNotification = (notificationId: string) => {
    if (!socket) return;
    socket.emit('delete-notification', notificationId);

    setNotifications((prev) => prev.filter((n) => n.id !== notificationId));
  };

  return {
    notifications,
    unreadCount,
    markAsRead,
    markAllAsRead,
    deleteNotification,
  };
}

function playNotificationSound() {
  const audio = new Audio('/notification.mp3');
  audio.play().catch((e) => console.error('Sound play failed:', e));
}

// COMPONENT USAGE
export function NotificationBell() {
  const token = 'user-token';
  const { notifications, unreadCount, markAsRead, markAllAsRead } =
    useNotifications(token);

  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      {/* Bell icon with badge */}
      <button onClick={() => setIsOpen(!isOpen)}>
        🔔
        {unreadCount > 0 && <span className="badge">{unreadCount}</span>}
      </button>

      {/* Notification dropdown */}
      {isOpen && (
        <div className="notification-dropdown">
          <div className="header">
            <h3>Notifications</h3>
            <button onClick={markAllAsRead}>Mark all as read</button>
          </div>

          <div className="notification-list">
            {notifications.length === 0 ? (
              <p>No notifications</p>
            ) : (
              notifications.map((notif) => (
                <div
                  key={notif.id}
                  className={`notification-item ${notif.isRead ? 'read' : 'unread'}`}
                  onClick={() => markAsRead(notif.id)}
                >
                  <div className="content">
                    <strong>{notif.title}</strong>
                    <p>{notif.message}</p>
                    <small>{new Date(notif.createdAt).toLocaleString()}</small>
                  </div>
                </div>
              ))
            )}
          </div>
        </div>
      )}
    </div>
  );
}
```

### **Module Setup**

```typescript
import { Module, forwardRef } from '@nestjs/common';
import { NotificationGateway } from './notification.gateway';
import { NotificationService } from './notification.service';
import { AuthService } from '../auth/auth.service';

@Module({
  providers: [NotificationGateway, NotificationService, AuthService],
  exports: [NotificationService], // Export for use in other modules
})
export class NotificationModule {}
```

**Interview Tip**: **Real-time notifications** with WebSockets: **(1) Gateway** - `@WebSocketGateway({ namespace: '/notifications' })` with authentication middleware in `afterInit()`; **(2) User rooms** - each user joins personal room `user:${userId}` on connection (`client.join(`user:${userId}`)`) for targeted notifications; **(3) Sending** - service method `notificationGateway.sendToUser(userId, notification)` emits to user's room with `server.to(`user:${userId}`).emit('notification', data)` - only that user receives it; **(4) On connect** - send unread count and recent notifications immediately to new connection; **(5) Database** - persist all notifications with fields (id, userId, type, title, message, data, isRead, createdAt), query for pagination; **(6) Triggering** - business logic (PostService, CommentService) calls `notificationService.create(userId, type, title, message, data)` which saves to DB and sends via WebSocket; **(7) Types** - comment (someone commented), like (someone liked), mention (tagged in post), follow (new follower), system (announcements); **(8) Unread count** - track with `isRead` boolean, send updated count after mark-read, display badge on client; **(9) Client** - connect to `/notifications` namespace with auth token, listen for 'notification' event (display toast), 'unread-count' (update badge), emit 'mark-read', 'mark-all-read'; **(10) Features** - mark as read, mark all as read, delete, pagination (get-notifications event with page/limit), real-time badge updates. **Production additions**: notification preferences (email/push/in-app), grouping ("John and 5 others liked your post"), delivery tracking, push notifications (web push API), email fallback for offline users, batching (debounce multiple notifications).

</details>
37. How do you implement presence tracking (online/offline status)?

<details>
<summary><strong>Answer</strong></summary>

**Presence tracking** shows users' online/offline status in real-time. Implementation: **(1) On connection** - set user status to **online** in database, broadcast to all clients; **(2) On disconnection** - set status to **offline** with `lastSeen` timestamp, broadcast; **(3) In-memory tracking** - maintain Map of `userId -> socketId[]` for fast lookup (users can have multiple connections/tabs); **(4) Heartbeat/ping** - use Socket.IO's built-in ping-pong to detect dead connections (set `pingTimeout`, `pingInterval`); **(5) Broadcast status changes** - emit `user-status` event with `{ userId, status: 'online'/'offline', lastSeen }` to all connected users or specific rooms; **(6) Query online users** - provide endpoint/event to get list of online users; **(7) Idle detection** - track user activity (last interaction time) to show "away" status.

### **Presence Tracking Gateway**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  OnGatewayConnection,
  OnGatewayDisconnect,
  OnGatewayInit,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway({
  cors: { origin: '*' },
  pingTimeout: 60000, // 60 seconds
  pingInterval: 25000, // 25 seconds
})
export class PresenceGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  @WebSocketServer()
  server: Server;

  private logger = new Logger('PresenceGateway');

  // In-memory tracking: userId -> Set<socketId>
  private userSockets = new Map<string, Set<string>>();

  // User activity tracking: userId -> lastActivityTime
  private lastActivity = new Map<string, Date>();

  constructor(
    private presenceService: PresenceService,
    private authService: AuthService,
  ) {
    // Check for idle users every minute
    setInterval(() => this.checkIdleUsers(), 60000);
  }

  afterInit(server: Server) {
    // Authentication
    server.use((socket, next) => {
      try {
        const token = socket.handshake.auth.token;
        if (!token) return next(new Error('Auth required'));

        const user = this.authService.validateToken(token);
        socket.data.user = user;
        next();
      } catch (error) {
        next(new Error('Invalid token'));
      }
    });

    this.logger.log('Presence Gateway initialized');
  }

  // USER CONNECTS
  async handleConnection(client: Socket) {
    const user = client.data.user;
    this.logger.log(`User ${user.username} connected (${client.id})`);

    // Track socket
    if (!this.userSockets.has(user.id)) {
      this.userSockets.set(user.id, new Set());
    }
    this.userSockets.get(user.id)!.add(client.id);

    // Update activity
    this.lastActivity.set(user.id, new Date());

    // Update database (set online)
    await this.presenceService.setOnline(user.id, true);

    // Broadcast online status to all users
    this.server.emit('user-status', {
      userId: user.id,
      username: user.username,
      status: 'online',
      timestamp: new Date(),
    });

    // Send list of online users to newly connected user
    const onlineUsers = await this.getOnlineUsersList();
    client.emit('online-users', onlineUsers);
  }

  // USER DISCONNECTS
  async handleDisconnect(client: Socket) {
    const user = client.data.user;
    this.logger.log(`User ${user.username} disconnected (${client.id})`);

    // Remove socket from tracking
    const userSocketSet = this.userSockets.get(user.id);
    if (userSocketSet) {
      userSocketSet.delete(client.id);

      // If no more sockets for this user, set offline
      if (userSocketSet.size === 0) {
        this.userSockets.delete(user.id);
        this.lastActivity.delete(user.id);

        // Update database (set offline)
        const lastSeen = new Date();
        await this.presenceService.setOnline(user.id, false, lastSeen);

        // Broadcast offline status
        this.server.emit('user-status', {
          userId: user.id,
          username: user.username,
          status: 'offline',
          lastSeen,
          timestamp: new Date(),
        });
      }
    }
  }

  // GET ONLINE USERS
  @SubscribeMessage('get-online-users')
  async handleGetOnlineUsers(@ConnectedSocket() client: Socket) {
    const onlineUsers = await this.getOnlineUsersList();

    return {
      event: 'online-users',
      data: onlineUsers,
    };
  }

  // UPDATE ACTIVITY (reset idle timer)
  @SubscribeMessage('activity')
  handleActivity(@ConnectedSocket() client: Socket) {
    const user = client.data.user;

    // Update last activity time
    this.lastActivity.set(user.id, new Date());

    // If user was idle, set back to online
    const currentStatus = this.presenceService.getStatus(user.id);
    if (currentStatus === 'away') {
      this.presenceService.setStatus(user.id, 'online');

      // Broadcast status change
      this.server.emit('user-status', {
        userId: user.id,
        username: user.username,
        status: 'online',
        timestamp: new Date(),
      });
    }
  }

  // CHECK USER STATUS
  @SubscribeMessage('check-user-status')
  async handleCheckStatus(
    @MessageBody() userId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const isOnline = this.userSockets.has(userId);
    const status = isOnline
      ? this.presenceService.getStatus(userId)
      : 'offline';

    const lastSeen = await this.presenceService.getLastSeen(userId);

    return {
      event: 'user-status-response',
      data: {
        userId,
        status,
        isOnline,
        lastSeen,
      },
    };
  }

  // HELPER: Get list of online users
  private async getOnlineUsersList(): Promise<any[]> {
    const onlineUserIds = Array.from(this.userSockets.keys());

    const users = await Promise.all(
      onlineUserIds.map(async (userId) => {
        const user = await this.authService.getUserById(userId);
        const status = this.presenceService.getStatus(userId);

        return {
          id: userId,
          username: user.username,
          avatar: user.avatar,
          status, // 'online' or 'away'
        };
      }),
    );

    return users;
  }

  // HELPER: Check for idle users (called periodically)
  private checkIdleUsers() {
    const now = new Date();
    const idleThreshold = 5 * 60 * 1000; // 5 minutes

    this.lastActivity.forEach((lastActivityTime, userId) => {
      const idleTime = now.getTime() - lastActivityTime.getTime();

      if (idleTime > idleThreshold) {
        // User is idle - set to 'away'
        const currentStatus = this.presenceService.getStatus(userId);

        if (currentStatus === 'online') {
          this.presenceService.setStatus(userId, 'away');

          // Broadcast away status
          this.server.emit('user-status', {
            userId,
            username: 'username', // Get from service
            status: 'away',
            timestamp: new Date(),
          });
        }
      }
    });
  }

  // PUBLIC METHOD: Check if user is online
  isUserOnline(userId: string): boolean {
    return this.userSockets.has(userId);
  }

  // PUBLIC METHOD: Get user's socket IDs
  getUserSockets(userId: string): string[] {
    return Array.from(this.userSockets.get(userId) || []);
  }
}
```

### **Presence Service**

```typescript
import { Injectable } from '@nestjs/common';

export type UserStatus = 'online' | 'away' | 'offline';

@Injectable()
export class PresenceService {
  // In-memory status cache
  private statusCache = new Map<string, UserStatus>();

  // Set user online/offline
  async setOnline(
    userId: string,
    isOnline: boolean,
    lastSeen?: Date,
  ): Promise<void> {
    // Update database
    // await this.db.users.update({
    //   where: { id: userId },
    //   data: {
    //     isOnline,
    //     lastSeen: lastSeen || new Date(),
    //   },
    // });

    // Update cache
    this.statusCache.set(userId, isOnline ? 'online' : 'offline');
  }

  // Set specific status (online, away, offline)
  setStatus(userId: string, status: UserStatus): void {
    this.statusCache.set(userId, status);

    // Optionally update database
    // await this.db.users.update({
    //   where: { id: userId },
    //   data: { status },
    // });
  }

  // Get user status
  getStatus(userId: string): UserStatus {
    return this.statusCache.get(userId) || 'offline';
  }

  // Get last seen time
  async getLastSeen(userId: string): Promise<Date | null> {
    // const user = await this.db.users.findOne({
    //   where: { id: userId },
    //   select: ['lastSeen'],
    // });
    // return user?.lastSeen || null;
    return new Date(); // Mock
  }

  // Get all online users
  async getOnlineUsers(): Promise<string[]> {
    // const users = await this.db.users.find({
    //   where: { isOnline: true },
    //   select: ['id'],
    // });
    // return users.map((u) => u.id);
    return Array.from(this.statusCache.keys()).filter(
      (userId) => this.statusCache.get(userId) !== 'offline',
    );
  }
}

class AuthService {
  validateToken(token: string): any {
    if (token === 'valid-token') {
      return { id: '123', username: 'john_doe', avatar: 'avatar.jpg' };
    }
    throw new Error('Invalid token');
  }

  async getUserById(id: string): Promise<any> {
    return { id, username: 'user_' + id, avatar: 'avatar.jpg' };
  }
}
```

### **Client-Side (React)**

```typescript
import { useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';

interface UserStatus {
  userId: string;
  username: string;
  status: 'online' | 'away' | 'offline';
  lastSeen?: Date;
}

export function usePresence(token: string) {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [onlineUsers, setOnlineUsers] = useState<UserStatus[]>([]);
  const [userStatuses, setUserStatuses] = useState<Map<string, UserStatus>>(
    new Map(),
  );

  // CONNECT
  useEffect(() => {
    const newSocket = io('http://localhost:3000', {
      auth: { token },
    });

    setSocket(newSocket);

    return () => {
      newSocket.close();
    };
  }, [token]);

  // LISTEN FOR STATUS UPDATES
  useEffect(() => {
    if (!socket) return;

    // RECEIVE: Online users list
    socket.on('online-users', (users: UserStatus[]) => {
      setOnlineUsers(users);

      const statusMap = new Map<string, UserStatus>();
      users.forEach((user) => statusMap.set(user.userId, user));
      setUserStatuses(statusMap);
    });

    // RECEIVE: Status change (real-time)
    socket.on('user-status', (statusUpdate: UserStatus) => {
      setUserStatuses((prev) => {
        const newMap = new Map(prev);
        newMap.set(statusUpdate.userId, statusUpdate);
        return newMap;
      });

      // Update online users list
      if (statusUpdate.status === 'online' || statusUpdate.status === 'away') {
        setOnlineUsers((prev) => {
          const existing = prev.find((u) => u.userId === statusUpdate.userId);
          if (existing) {
            return prev.map((u) =>
              u.userId === statusUpdate.userId ? statusUpdate : u,
            );
          } else {
            return [...prev, statusUpdate];
          }
        });
      } else if (statusUpdate.status === 'offline') {
        setOnlineUsers((prev) =>
          prev.filter((u) => u.userId !== statusUpdate.userId),
        );
      }
    });

    return () => {
      socket.off('online-users');
      socket.off('user-status');
    };
  }, [socket]);

  // TRACK ACTIVITY (reset idle timer)
  useEffect(() => {
    if (!socket) return;

    const sendActivity = () => {
      socket.emit('activity');
    };

    // Send activity on user interaction
    window.addEventListener('mousemove', sendActivity);
    window.addEventListener('keydown', sendActivity);
    window.addEventListener('click', sendActivity);

    return () => {
      window.removeEventListener('mousemove', sendActivity);
      window.removeEventListener('keydown', sendActivity);
      window.removeEventListener('click', sendActivity);
    };
  }, [socket]);

  // GET STATUS OF SPECIFIC USER
  const getUserStatus = (userId: string): UserStatus | null => {
    return userStatuses.get(userId) || null;
  };

  // CHECK STATUS (query server)
  const checkUserStatus = (userId: string) => {
    if (!socket) return;
    socket.emit('check-user-status', userId);
  };

  return {
    onlineUsers,
    getUserStatus,
    checkUserStatus,
  };
}

// COMPONENT: Online users list
export function OnlineUsersList() {
  const token = 'user-token';
  const { onlineUsers } = usePresence(token);

  return (
    <div>
      <h3>Online Users ({onlineUsers.length})</h3>
      <ul>
        {onlineUsers.map((user) => (
          <li key={user.userId}>
            <span
              style={{
                display: 'inline-block',
                width: '10px',
                height: '10px',
                borderRadius: '50%',
                backgroundColor:
                  user.status === 'online'
                    ? 'green'
                    : user.status === 'away'
                      ? 'orange'
                      : 'gray',
                marginRight: '8px',
              }}
            />
            {user.username}
            {user.status === 'away' && ' (Away)'}
          </li>
        ))}
      </ul>
    </div>
  );
}

// COMPONENT: User status indicator
export function UserStatusIndicator({ userId }: { userId: string }) {
  const token = 'user-token';
  const { getUserStatus } = usePresence(token);

  const status = getUserStatus(userId);

  if (!status) return <span>Offline</span>;

  return (
    <span>
      <span
        style={{
          display: 'inline-block',
          width: '8px',
          height: '8px',
          borderRadius: '50%',
          backgroundColor:
            status.status === 'online'
              ? 'green'
              : status.status === 'away'
                ? 'orange'
                : 'gray',
          marginRight: '5px',
        }}
      />
      {status.status === 'online' && 'Online'}
      {status.status === 'away' && 'Away'}
      {status.status === 'offline' && (
        <span>
          Offline (last seen: {status.lastSeen ? new Date(status.lastSeen).toLocaleString() : 'never'})
        </span>
      )}
    </span>
  );
}
```

### **Production Enhancements**

```typescript
// Redis-backed presence (for multi-server)
import { Injectable } from '@nestjs/common';
import { Redis } from 'ioredis';

@Injectable()
export class RedisPresenceService {
  private redis: Redis;

  constructor() {
    this.redis = new Redis();
  }

  // Set user online
  async setOnline(userId: string): Promise<void> {
    // Add to online users set with TTL
    await this.redis.setex(`user:${userId}:online`, 60, '1');

    // Update last seen
    await this.redis.set(`user:${userId}:lastSeen`, new Date().toISOString());
  }

  // Check if user is online
  async isOnline(userId: string): Promise<boolean> {
    const result = await this.redis.get(`user:${userId}:online`);
    return result === '1';
  }

  // Get all online users
  async getOnlineUsers(): Promise<string[]> {
    const keys = await this.redis.keys('user:*:online');
    const userIds = keys.map((key) => key.split(':')[1]);

    // Filter only those that are still online
    const onlineStatuses = await Promise.all(
      userIds.map((id) => this.isOnline(id)),
    );

    return userIds.filter((_, i) => onlineStatuses[i]);
  }

  // Heartbeat (keep user online)
  async heartbeat(userId: string): Promise<void> {
    await this.redis.expire(`user:${userId}:online`, 60);
  }
}
```

**Interview Tip**: **Presence tracking** shows online/offline/away status in real-time. **Implementation**: **(1) Connection** - `handleConnection` adds user to in-memory Map `userSockets` (userId → Set<socketId> for multiple tabs), sets database `isOnline=true`, broadcasts 'user-status' event with `{ userId, status: 'online' }` to all clients; **(2) Disconnection** - `handleDisconnect` removes socketId, if no more sockets for user, sets `isOnline=false` with `lastSeen` timestamp, broadcasts offline status; **(3) Multiple connections** - user can have multiple tabs/devices, track all socketIds in Set, only mark offline when ALL connections close; **(4) In-memory tracking** - Map for fast lookup (`isUserOnline(userId)`), avoid DB query on every check; **(5) Idle detection** - track `lastActivity` Map, check periodically (setInterval every minute), if idle > 5 minutes set status to 'away', broadcast status change, reset on 'activity' event (mouse/keyboard); **(6) Online users list** - send on connect with `client.emit('online-users', list)`, client listens for real-time 'user-status' updates to add/remove/update; **(7) Heartbeat** - Socket.IO has built-in ping-pong (`pingTimeout: 60000`, `pingInterval: 25000`), auto-detects dead connections; **(8) Client** - sends 'activity' event on user interaction (mousemove, keydown, click) to reset idle, listens for 'user-status' events to update UI (green dot for online, orange for away, gray for offline + lastSeen); **(9) Database** - store `isOnline` boolean and `lastSeen` timestamp, query for initial state but use in-memory for real-time; **(10) Multi-server** - use **Redis** to share presence across servers (store `user:${userId}:online` with TTL, heartbeat to refresh). **Status types**: online (connected + active), away (connected + idle >5min), offline (disconnected). **Production**: Redis for scaling, custom status messages ("Working", "In a meeting"), mobile/desktop distinction, last activity details.

</details>
38. How do you implement typing indicators?

<details>
<summary><strong>Answer</strong></summary>

**Typing indicators** show when users are typing in real-time ("John is typing..."). Implementation: **(1) Client** sends **`typing` event** with `{ roomId, isTyping: true }` when user types, stops after timeout (2-3 seconds of inactivity); **(2) Server** receives typing event, broadcasts to **other users in room** (not sender) using `client.to(roomId).emit('user-typing', { userId, username, isTyping })` - note **`client.to()`** excludes sender; **(3) Client** listens for **`user-typing` events**, displays "X is typing..." in UI, removes after receiving `isTyping: false` or timeout; **(4) Debouncing** - client debounces typing events (send max once per 1-2 seconds) to reduce server load; **(5) Timeout** - client auto-sends `isTyping: false` after 2-3 seconds of no typing; **(6) Multiple users** - track Set of currently typing users, display "John, Jane, and 2 others are typing...".

### **Server-Side: Typing Handler**

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway({
  cors: { origin: '*' },
  namespace: '/chat',
})
export class TypingGateway {
  @WebSocketServer()
  server: Server;

  private logger = new Logger('TypingGateway');

  // Track typing users: roomId -> Set<userId>
  private typingUsers = new Map<string, Set<string>>();

  // TYPING INDICATOR
  @SubscribeMessage('typing')
  handleTyping(
    @MessageBody() data: { roomId: string; isTyping: boolean },
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;

    // Update tracking
    if (data.isTyping) {
      if (!this.typingUsers.has(data.roomId)) {
        this.typingUsers.set(data.roomId, new Set());
      }
      this.typingUsers.get(data.roomId)!.add(user.id);
    } else {
      const roomTypingUsers = this.typingUsers.get(data.roomId);
      if (roomTypingUsers) {
        roomTypingUsers.delete(user.id);
        if (roomTypingUsers.size === 0) {
          this.typingUsers.delete(data.roomId);
        }
      }
    }

    // Broadcast to OTHERS in room (not sender)
    client.to(data.roomId).emit('user-typing', {
      roomId: data.roomId,
      user: {
        id: user.id,
        username: user.username,
      },
      isTyping: data.isTyping,
    });

    this.logger.log(
      `User ${user.username} ${data.isTyping ? 'started' : 'stopped'} typing in ${data.roomId}`,
    );
  }

  // GET CURRENTLY TYPING USERS IN ROOM
  @SubscribeMessage('get-typing-users')
  handleGetTypingUsers(
    @MessageBody() roomId: string,
    @ConnectedSocket() client: Socket,
  ) {
    const typingUserIds = Array.from(
      this.typingUsers.get(roomId) || [],
    );

    return {
      event: 'typing-users',
      data: {
        roomId,
        userIds: typingUserIds,
      },
    };
  }

  // CLEANUP: Remove user from typing on disconnect
  handleDisconnect(client: Socket) {
    const user = client.data.user;

    // Remove from all rooms' typing lists
    this.typingUsers.forEach((typingSet, roomId) => {
      if (typingSet.has(user.id)) {
        typingSet.delete(user.id);

        // Notify others
        client.to(roomId).emit('user-typing', {
          roomId,
          user: { id: user.id, username: user.username },
          isTyping: false,
        });
      }
    });
  }
}
```

### **Client-Side: React Hook**

```typescript
import { useEffect, useState, useRef, useCallback } from 'react';
import { Socket } from 'socket.io-client';

interface TypingUser {
  userId: string;
  username: string;
}

export function useTypingIndicator(socket: Socket | null, roomId: string) {
  const [typingUsers, setTypingUsers] = useState<TypingUser[]>([]);
  const typingTimeoutRef = useRef<NodeJS.Timeout | null>(null);
  const isTypingRef = useRef(false);

  // LISTEN FOR TYPING EVENTS
  useEffect(() => {
    if (!socket) return;

    const handleUserTyping = (data: {
      roomId: string;
      user: TypingUser;
      isTyping: boolean;
    }) => {
      if (data.roomId !== roomId) return;

      if (data.isTyping) {
        // Add user to typing list
        setTypingUsers((prev) => {
          const exists = prev.some((u) => u.userId === data.user.userId);
          if (exists) return prev;
          return [...prev, data.user];
        });
      } else {
        // Remove user from typing list
        setTypingUsers((prev) =>
          prev.filter((u) => u.userId !== data.user.userId),
        );
      }
    };

    socket.on('user-typing', handleUserTyping);

    return () => {
      socket.off('user-typing', handleUserTyping);
    };
  }, [socket, roomId]);

  // SEND TYPING INDICATOR
  const setTyping = useCallback(
    (isTyping: boolean) => {
      if (!socket) return;

      // Don't send duplicate events
      if (isTypingRef.current === isTyping) return;

      isTypingRef.current = isTyping;
      socket.emit('typing', { roomId, isTyping });
    },
    [socket, roomId],
  );

  // START TYPING
  const startTyping = useCallback(() => {
    // Send "typing: true" immediately
    setTyping(true);

    // Clear existing timeout
    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
    }

    // Auto-stop after 3 seconds of inactivity
    typingTimeoutRef.current = setTimeout(() => {
      stopTyping();
    }, 3000);
  }, [setTyping]);

  // STOP TYPING
  const stopTyping = useCallback(() => {
    setTyping(false);

    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
      typingTimeoutRef.current = null;
    }
  }, [setTyping]);

  // CLEANUP
  useEffect(() => {
    return () => {
      if (typingTimeoutRef.current) {
        clearTimeout(typingTimeoutRef.current);
      }
      stopTyping();
    };
  }, [stopTyping]);

  return {
    typingUsers,
    startTyping,
    stopTyping,
  };
}
```

### **Client-Side: Chat Component**

```typescript
import React, { useState } from 'react';
import { io } from 'socket.io-client';
import { useTypingIndicator } from './useTypingIndicator';

export function ChatRoom({ roomId, token }: { roomId: string; token: string }) {
  const [socket] = useState(() =>
    io('http://localhost:3000/chat', { auth: { token } }),
  );
  const [message, setMessage] = useState('');
  const [messages, setMessages] = useState<any[]>([]);

  const { typingUsers, startTyping, stopTyping } = useTypingIndicator(
    socket,
    roomId,
  );

  // HANDLE INPUT CHANGE
  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value;
    setMessage(newValue);

    if (newValue.length > 0) {
      // Start typing indicator
      startTyping();
    } else {
      // Stop if input is empty
      stopTyping();
    }
  };

  // SEND MESSAGE
  const sendMessage = () => {
    if (!message.trim()) return;

    socket.emit('send-message', { roomId, message });
    setMessage('');

    // Stop typing indicator
    stopTyping();
  };

  // RENDER TYPING INDICATOR
  const renderTypingIndicator = () => {
    if (typingUsers.length === 0) return null;

    let text = '';
    if (typingUsers.length === 1) {
      text = `${typingUsers[0].username} is typing...`;
    } else if (typingUsers.length === 2) {
      text = `${typingUsers[0].username} and ${typingUsers[1].username} are typing...`;
    } else {
      const others = typingUsers.length - 2;
      text = `${typingUsers[0].username}, ${typingUsers[1].username}, and ${others} other${others > 1 ? 's' : ''} are typing...`;
    }

    return (
      <div
        style={{
          fontStyle: 'italic',
          color: '#666',
          padding: '10px',
          minHeight: '20px',
        }}
      >
        {text}
      </div>
    );
  };

  return (
    <div>
      {/* Messages */}
      <div style={{ height: '400px', overflowY: 'auto' }}>
        {messages.map((msg, i) => (
          <div key={i}>
            <strong>{msg.username}:</strong> {msg.text}
          </div>
        ))}
      </div>

      {/* Typing indicator */}
      {renderTypingIndicator()}

      {/* Input */}
      <div>
        <input
          type="text"
          value={message}
          onChange={handleInputChange}
          onKeyPress={(e) => {
            if (e.key === 'Enter') {
              sendMessage();
            }
          }}
          onBlur={stopTyping} // Stop when input loses focus
          placeholder="Type a message..."
        />
        <button onClick={sendMessage}>Send</button>
      </div>
    </div>
  );
}
```

### **Advanced: Debounced Typing**

```typescript
import { useEffect, useRef, useCallback } from 'react';
import { Socket } from 'socket.io-client';

export function useDebouncedTyping(
  socket: Socket | null,
  roomId: string,
  debounceMs: number = 1000, // Send max once per second
) {
  const isTypingRef = useRef(false);
  const lastEmitRef = useRef(0);
  const stopTimeoutRef = useRef<NodeJS.Timeout | null>(null);

  const emitTyping = useCallback(
    (isTyping: boolean) => {
      if (!socket) return;

      const now = Date.now();

      // Debounce: only send if enough time has passed
      if (isTyping && now - lastEmitRef.current < debounceMs) {
        return;
      }

      // Update state
      if (isTypingRef.current !== isTyping) {
        isTypingRef.current = isTyping;
        socket.emit('typing', { roomId, isTyping });
        lastEmitRef.current = now;
      }
    },
    [socket, roomId, debounceMs],
  );

  const startTyping = useCallback(() => {
    // Send "typing: true" (debounced)
    emitTyping(true);

    // Clear existing stop timeout
    if (stopTimeoutRef.current) {
      clearTimeout(stopTimeoutRef.current);
    }

    // Auto-stop after 3 seconds
    stopTimeoutRef.current = setTimeout(() => {
      emitTyping(false);
    }, 3000);
  }, [emitTyping]);

  const stopTyping = useCallback(() => {
    emitTyping(false);

    if (stopTimeoutRef.current) {
      clearTimeout(stopTimeoutRef.current);
      stopTimeoutRef.current = null;
    }
  }, [emitTyping]);

  // Cleanup
  useEffect(() => {
    return () => {
      if (stopTimeoutRef.current) {
        clearTimeout(stopTimeoutRef.current);
      }
      stopTyping();
    };
  }, [stopTyping]);

  return { startTyping, stopTyping };
}
```

### **Animated Typing Indicator (CSS)**

```typescript
export function TypingIndicator({ users }: { users: string[] }) {
  if (users.length === 0) return null;

  const text =
    users.length === 1
      ? `${users[0]} is typing`
      : users.length === 2
        ? `${users[0]} and ${users[1]} are typing`
        : `${users[0]}, ${users[1]}, and ${users.length - 2} others are typing`;

  return (
    <div style={{ display: 'flex', alignItems: 'center', padding: '10px' }}>
      <span style={{ fontStyle: 'italic', color: '#666', marginRight: '8px' }}>
        {text}
      </span>
      <div className="typing-dots">
        <span className="dot" />
        <span className="dot" />
        <span className="dot" />
      </div>
    </div>
  );
}

// CSS
const styles = `
.typing-dots {
  display: flex;
  gap: 4px;
}

.typing-dots .dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: #666;
  animation: typing-bounce 1.4s infinite ease-in-out;
}

.typing-dots .dot:nth-child(1) {
  animation-delay: 0s;
}

.typing-dots .dot:nth-child(2) {
  animation-delay: 0.2s;
}

.typing-dots .dot:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes typing-bounce {
  0%, 60%, 100% {
    transform: translateY(0);
  }
  30% {
    transform: translateY(-10px);
  }
}
`;
```

### **Production: Rate Limiting**

```typescript
@WebSocketGateway()
export class RateLimitedTypingGateway {
  @WebSocketServer()
  server: Server;

  // Rate limiting: userId -> last emit time
  private lastTypingEmit = new Map<string, number>();
  private readonly RATE_LIMIT_MS = 1000; // 1 second

  @SubscribeMessage('typing')
  handleTyping(
    @MessageBody() data: { roomId: string; isTyping: boolean },
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;
    const now = Date.now();

    // Rate limit: only process if enough time has passed
    const lastEmit = this.lastTypingEmit.get(user.id) || 0;
    if (data.isTyping && now - lastEmit < this.RATE_LIMIT_MS) {
      return; // Ignore this event
    }

    this.lastTypingEmit.set(user.id, now);

    // Broadcast to others
    client.to(data.roomId).emit('user-typing', {
      roomId: data.roomId,
      user: { id: user.id, username: user.username },
      isTyping: data.isTyping,
    });
  }
}
```

**Interview Tip**: **Typing indicators** show "X is typing..." in real-time. **Client**: on input change, send `socket.emit('typing', { roomId, isTyping: true })`, start timeout (3 seconds), on timeout or message send, emit `isTyping: false`. **Server**: `@SubscribeMessage('typing')` receives event, broadcasts to **others in room** (not sender) using **`client.to(roomId).emit()`** (excludes sender), payload `{ userId, username, isTyping }`. **Client receives**: listen for `socket.on('user-typing', data)`, add/remove user from typing list, display "John is typing..." or "John, Jane, and 2 others are typing...". **Key patterns**: (1) **Auto-stop** - client clears typing after 3 seconds of no input (setTimeout), prevents stuck "is typing" if user stops without sending; (2) **Debouncing** - send max once per 1-2 seconds to reduce server load, track `lastEmitTime`; (3) **Multiple users** - maintain Set of typing users, format display text (1 user: "X is", 2 users: "X and Y are", 3+: "X, Y, and N others are"); (4) **Exclude sender** - use `client.to(room)` not `server.to(room)` so sender doesn't see own typing indicator; (5) **Cleanup** - on disconnect, remove user from all typing lists, broadcast `isTyping: false`; (6) **Rate limiting** - server tracks last emit time per user, ignores events within 1 second; (7) **Stop conditions** - message sent (stopTyping on send), input cleared (check value.length === 0), focus lost (onBlur), timeout (3 seconds). **Production**: animated dots (CSS animation), server-side rate limiting (1 event/sec), per-room tracking (Map<roomId, Set<userId>>), persistence of typing state on reconnect (not common), accessibility (screen reader announcements).

</details>

## Performance & Scalability

39. How do you scale WebSocket connections across multiple servers?

<details>
<summary><strong>Answer</strong></summary>

Scaling WebSockets across multiple servers requires **Redis Adapter** to synchronize events between instances. Problem: with load balancer distributing connections across servers, **user A on server 1** can't directly receive messages from **user B on server 2** because Socket.IO servers don't communicate by default. Solution: **(1) Install Redis Adapter** (`@socket.io/redis-adapter`), **(2) Configure Redis** connection in gateway (`IoAdapter.create(redisClient)`), **(3) Redis pub/sub** broadcasts events across all servers - when server 1 emits, Redis publishes to all servers, server 2 receives and emits to its connected clients. **Setup**: `adapter.createAdapter()` in main.ts, configure load balancer with **sticky sessions** (same user always routes to same server) or **any-server routing** (Redis handles cross-server messaging).

### **Problem: Without Redis**

```
Load Balancer
     |
   /   \
Server1  Server2
  |        |
User A   User B

User A sends message to User B:
  1. User A → Server1 → emit('message', data)
  2. Server1 emits to its connected clients (only User A)
  3. User B on Server2 doesn't receive (servers don't communicate)
  4. ❌ Message not delivered to User B
```

### **Solution: With Redis Adapter**

```
Load Balancer
     |
   /   \
Server1  Server2
  |  \  /  |
  |   Redis  |
  |        |
User A   User B

User A sends message to User B:
  1. User A → Server1 → emit('message', data)
  2. Server1 → Redis pub/sub → publish event
  3. Redis → Server2 → receives published event
  4. Server2 → emits to User B
  5. ✅ Message delivered to User B
```

### **Installation**

```bash
npm install @socket.io/redis-adapter redis ioredis
```

### **Basic Redis Adapter Setup**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { IoAdapter } from '@nestjs/platform-socket.io';
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Create Redis clients (pub and sub)
  const pubClient = createClient({ url: 'redis://localhost:6379' });
  const subClient = pubClient.duplicate();

  await Promise.all([pubClient.connect(), subClient.connect()]);

  // Create Redis Adapter
  const redisAdapter = createAdapter(pubClient, subClient);

  // Use adapter for Socket.IO
  app.useWebSocketAdapter(
    new IoAdapter(app, { adapter: redisAdapter }),
  );

  await app.listen(3000);
  console.log('Application listening on port 3000');
}
bootstrap();
```

### **Custom Redis Adapter (NestJS)**

```typescript
// redis-io.adapter.ts
import { IoAdapter } from '@nestjs/platform-socket.io';
import { ServerOptions } from 'socket.io';
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';
import { INestApplicationContext } from '@nestjs/common';

export class RedisIoAdapter extends IoAdapter {
  private adapterConstructor: ReturnType<typeof createAdapter>;

  async connectToRedis(): Promise<void> {
    // Create pub and sub clients
    const pubClient = createClient({
      url: process.env.REDIS_URL || 'redis://localhost:6379',
    });
    const subClient = pubClient.duplicate();

    await Promise.all([pubClient.connect(), subClient.connect()]);

    // Create adapter
    this.adapterConstructor = createAdapter(pubClient, subClient);
  }

  createIOServer(port: number, options?: ServerOptions): any {
    const server = super.createIOServer(port, options);

    // Use Redis adapter
    server.adapter(this.adapterConstructor);

    return server;
  }
}

// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { RedisIoAdapter } from './redis-io.adapter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Create and connect adapter
  const redisIoAdapter = new RedisIoAdapter(app);
  await redisIoAdapter.connectToRedis();

  // Use adapter
  app.useWebSocketAdapter(redisIoAdapter);

  await app.listen(3000);
  console.log('Application listening on port 3000');
}
bootstrap();
```

### **With ioredis (Alternative)**

```typescript
// redis-io.adapter.ts
import { IoAdapter } from '@nestjs/platform-socket.io';
import { ServerOptions } from 'socket.io';
import { createAdapter } from '@socket.io/redis-adapter';
import Redis from 'ioredis';

export class RedisIoAdapter extends IoAdapter {
  private adapterConstructor: ReturnType<typeof createAdapter>;

  async connectToRedis(): Promise<void> {
    const pubClient = new Redis({
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT || '6379'),
      password: process.env.REDIS_PASSWORD,
    });

    const subClient = pubClient.duplicate();

    this.adapterConstructor = createAdapter(pubClient, subClient);
  }

  createIOServer(port: number, options?: ServerOptions): any {
    const server = super.createIOServer(port, options);
    server.adapter(this.adapterConstructor);
    return server;
  }
}
```

### **Load Balancer Configuration**

#### **Option 1: Sticky Sessions (Session Affinity)**

```nginx
# nginx.conf
upstream websocket_backend {
    ip_hash;  # Sticky sessions based on client IP
    server server1:3000;
    server server2:3000;
    server server3:3000;
}

server {
    listen 80;

    location / {
        proxy_pass http://websocket_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_read_timeout 86400;
    }
}
```

#### **Option 2: Round Robin (with Redis Adapter)**

```nginx
# nginx.conf
upstream websocket_backend {
    # Round robin (no ip_hash needed with Redis)
    server server1:3000;
    server server2:3000;
    server server3:3000;
}

server {
    listen 80;

    location / {
        proxy_pass http://websocket_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### **Testing Multi-Server Setup Locally**

```bash
# Terminal 1: Start Redis
docker run -d -p 6379:6379 redis:latest

# Terminal 2: Start Server 1
PORT=3001 npm run start

# Terminal 3: Start Server 2
PORT=3002 npm run start

# Terminal 4: Start Server 3
PORT=3003 npm run start

# Test: Connect clients to different servers
# Client 1 → localhost:3001
# Client 2 → localhost:3002
# Client 1 sends message → Client 2 receives (via Redis)
```

### **Docker Compose Setup**

```yaml
# docker-compose.yml
version: '3.8'

services:
  redis:
    image: redis:latest
    ports:
      - '6379:6379'

  app1:
    build: .
    ports:
      - '3001:3000'
    environment:
      - REDIS_URL=redis://redis:6379
    depends_on:
      - redis

  app2:
    build: .
    ports:
      - '3002:3000'
    environment:
      - REDIS_URL=redis://redis:6379
    depends_on:
      - redis

  app3:
    build: .
    ports:
      - '3003:3000'
    environment:
      - REDIS_URL=redis://redis:6379
    depends_on:
      - redis

  nginx:
    image: nginx:latest
    ports:
      - '80:80'
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app1
      - app2
      - app3
```

### **Monitoring Redis Adapter**

```typescript
// redis-io.adapter.ts
import { Logger } from '@nestjs/common';

export class RedisIoAdapter extends IoAdapter {
  private logger = new Logger('RedisIoAdapter');

  async connectToRedis(): Promise<void> {
    const pubClient = new Redis({
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT || '6379'),
    });

    const subClient = pubClient.duplicate();

    // Monitor connections
    pubClient.on('connect', () => {
      this.logger.log('Redis PUB client connected');
    });

    pubClient.on('error', (err) => {
      this.logger.error('Redis PUB client error:', err);
    });

    subClient.on('connect', () => {
      this.logger.log('Redis SUB client connected');
    });

    subClient.on('error', (err) => {
      this.logger.error('Redis SUB client error:', err);
    });

    this.adapterConstructor = createAdapter(pubClient, subClient);

    this.logger.log('Redis Adapter configured successfully');
  }
}
```

### **Kubernetes Deployment**

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: websocket-app
spec:
  replicas: 3  # Multiple instances
  selector:
    matchLabels:
      app: websocket-app
  template:
    metadata:
      labels:
        app: websocket-app
    spec:
      containers:
        - name: app
          image: your-app:latest
          env:
            - name: REDIS_URL
              value: redis://redis-service:6379
          ports:
            - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: websocket-service
spec:
  type: LoadBalancer
  selector:
    app: websocket-app
  ports:
    - port: 80
      targetPort: 3000

---
# Redis deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:latest
          ports:
            - containerPort: 6379

---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  selector:
    app: redis
  ports:
    - port: 6379
      targetPort: 6379
```

### **Performance Considerations**

```typescript
// Optimized Redis Adapter
export class OptimizedRedisIoAdapter extends IoAdapter {
  async connectToRedis(): Promise<void> {
    const pubClient = new Redis({
      host: process.env.REDIS_HOST,
      port: parseInt(process.env.REDIS_PORT),
      // Performance options
      maxRetriesPerRequest: 3,
      enableReadyCheck: true,
      enableOfflineQueue: false,
      // Connection pooling
      lazyConnect: false,
      // Timeouts
      connectTimeout: 10000,
      commandTimeout: 5000,
    });

    const subClient = pubClient.duplicate();

    this.adapterConstructor = createAdapter(pubClient, subClient, {
      // Adapter options
      key: 'socket.io', // Redis key prefix
      requestsTimeout: 5000,
    });
  }
}
```

**Interview Tip**: **Scaling WebSockets** across multiple servers requires **Redis Adapter** to synchronize events. **Problem**: load balancer distributes users to different servers (User A on Server1, User B on Server2), servers don't communicate by default, so Server1 can't send to Server2's clients. **Solution**: **(1) Install** `@socket.io/redis-adapter` and `redis`/`ioredis`; **(2) Create adapter** in main.ts with `createAdapter(pubClient, subClient)` (two Redis clients for pub/sub); **(3) Apply adapter** to Socket.IO server with `app.useWebSocketAdapter(new RedisIoAdapter(app))`; **(4) How it works** - when server emits event (`server.to(room).emit()`), Redis Adapter publishes to Redis pub/sub channel, all other servers subscribed to Redis receive and emit to their connected clients. **Load balancer**: can use **sticky sessions** (ip_hash in nginx, routes same IP to same server) or **any-server routing** (round robin, Redis handles cross-server). **Benefits**: horizontal scaling (add more servers), high availability (server crash doesn't lose all connections), better performance (distribute load). **Setup steps**: (1) create RedisIoAdapter extending IoAdapter, (2) connect to Redis in `connectToRedis()`, (3) override `createIOServer()` to apply adapter with `server.adapter()`, (4) call `await adapter.connectToRedis()` in main.ts before starting. **Redis clients**: need TWO clients (pub and sub) because Redis pub/sub blocks subscriber from other operations. **Testing**: run multiple instances on different ports (3001, 3002, 3003) with same Redis, connect clients to different ports, verify cross-server messaging. **Deployment**: Docker Compose (multiple app containers + Redis), Kubernetes (deployment with replicas=3, Redis service), nginx load balancer. **Trade-offs**: adds Redis dependency (single point of failure - use Redis Sentinel/Cluster for HA), slight latency increase (network hop to Redis), increased complexity.

</details>
40. What is a Redis Adapter for Socket.IO?

<details>
<summary><strong>Answer</strong></summary>

**Redis Adapter** is a Socket.IO adapter that uses **Redis pub/sub** to synchronize events between multiple Socket.IO servers. It replaces the default in-memory adapter with a **distributed adapter** that publishes all events (emits, broadcasts, room joins/leaves) to Redis, allowing all connected servers to receive and process them. This enables **horizontal scaling** - multiple servers can handle WebSocket connections while maintaining consistent state. Package: **`@socket.io/redis-adapter`**. Core functions: **(1) Publish** - when server emits, adapter publishes to Redis channel; **(2) Subscribe** - all servers subscribe to Redis channels, receive published events; **(3) Broadcast** - emit to all clients across all servers; **(4) Rooms** - room membership synchronized via Redis, `to(room)` works across servers.

### **How Redis Adapter Works**

```
SERVER 1                    REDIS                     SERVER 2
   |
   |  emit('msg', data)
   |─────────────────────────▶ PUBLISH channel
                              'socket.io#/#msg'
                                  data: {...}
                                     |
                                     |─────────────────▶ SUBSCRIBE channel
                                                        'socket.io#/#msg'
                                                             |
                                                             |  emit to clients
                                                             ▼
                                                          Client B
```

### **Basic Usage**

```typescript
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

// Create Redis clients
const pubClient = createClient({ url: 'redis://localhost:6379' });
const subClient = pubClient.duplicate();

await pubClient.connect();
await subClient.connect();

// Create adapter
const redisAdapter = createAdapter(pubClient, subClient);

// Apply to Socket.IO server
io.adapter(redisAdapter);
```

### **What Gets Synchronized**

1. **Broadcasts** - `io.emit()`, `socket.broadcast.emit()`
2. **Room broadcasts** - `io.to('room').emit()`
3. **Namespace broadcasts** - `io.of('/namespace').emit()`
4. **Room joins/leaves** - `socket.join('room')`, `socket.leave('room')`
5. **Socket disconnections** - cleanup across servers
6. **Fetch sockets** - `io.fetchSockets()` returns sockets from all servers

### **Complete Example**

```typescript
// redis-io.adapter.ts
import { IoAdapter } from '@nestjs/platform-socket.io';
import { ServerOptions } from 'socket.io';
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';
import { Logger } from '@nestjs/common';

export class RedisIoAdapter extends IoAdapter {
  private adapterConstructor: ReturnType<typeof createAdapter>;
  private logger = new Logger('RedisIoAdapter');

  async connectToRedis(): Promise<void> {
    const pubClient = createClient({
      url: process.env.REDIS_URL || 'redis://localhost:6379',
    });

    const subClient = pubClient.duplicate();

    await Promise.all([pubClient.connect(), subClient.connect()]);

    this.logger.log('Connected to Redis for Socket.IO adapter');

    // Create adapter with options
    this.adapterConstructor = createAdapter(pubClient, subClient, {
      // Key prefix for Redis keys (default: 'socket.io')
      key: 'socket.io',
      
      // Request timeout for cross-server operations (ms)
      requestsTimeout: 5000,
    });
  }

  createIOServer(port: number, options?: ServerOptions): any {
    const server = super.createIOServer(port, options);

    // Apply Redis adapter
    server.adapter(this.adapterConstructor);

    this.logger.log('Redis Adapter applied to Socket.IO server');

    return server;
  }
}

// main.ts
import { NestFactory } from '@nestjs/core';
import { RedisIoAdapter } from './redis-io.adapter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const redisIoAdapter = new RedisIoAdapter(app);
  await redisIoAdapter.connectToRedis();

  app.useWebSocketAdapter(redisIoAdapter);

  await app.listen(3000);
}
bootstrap();
```

### **Adapter Features**

```typescript
// 1. BROADCAST TO ALL SERVERS
io.emit('announcement', 'Hello everyone!');
// → Published to Redis
// → All servers receive and emit to their clients

// 2. BROADCAST TO ROOM (across servers)
io.to('room1').emit('message', 'Hello room!');
// → Published to Redis with room info
// → All servers emit to clients in 'room1'

// 3. FETCH SOCKETS (from all servers)
const sockets = await io.fetchSockets();
console.log(`Total connected sockets: ${sockets.length}`);
// → Redis Adapter queries all servers
// → Returns combined list

// 4. SERVER-TO-SERVER EVENTS
io.serverSideEmit('custom-event', { data: 'value' });
// → Sent to all servers (not clients)
// → Each server receives via io.on('custom-event', ...)

io.on('custom-event', (data) => {
  console.log('Received from another server:', data);
});

// 5. DISCONNECT ALL SOCKETS (across servers)
const sockets = await io.fetchSockets();
sockets.forEach((socket) => socket.disconnect());
```

### **Redis Pub/Sub Channels**

Redis Adapter creates these channels:

```
socket.io#/#           - Main pub/sub channel
socket.io#/#-request# - Request/response channel
socket.io#/#-response# - Response channel
```

```typescript
// Monitor Redis channels
import Redis from 'ioredis';

const monitorClient = new Redis();

monitorClient.psubscribe('socket.io*', (err, count) => {
  console.log(`Subscribed to ${count} channels`);
});

monitorClient.on('pmessage', (pattern, channel, message) => {
  console.log('Redis channel:', channel);
  console.log('Message:', message);
});
```

### **Room Synchronization**

```typescript
// SERVER 1
socket.join('room-123');
// → Redis Adapter publishes room join
// → All servers update their room membership

// SERVER 2
io.to('room-123').emit('update', { data: 'value' });
// → Redis Adapter checks room membership
// → Publishes to Redis
// → Server 1 receives and emits to socket in room-123
```

### **Namespace Support**

```typescript
// Each namespace has its own adapter instance
const chatNamespace = io.of('/chat');
chatNamespace.adapter(redisAdapter);

const gameNamespace = io.of('/game');
gameNamespace.adapter(redisAdapter);

// Broadcasts are namespace-isolated
chatNamespace.emit('chat-message', 'Hello');
// → Only published to /chat namespace
// → Game namespace clients don't receive
```

### **Adapter Options**

```typescript
import { createAdapter } from '@socket.io/redis-adapter';

const adapter = createAdapter(pubClient, subClient, {
  // Redis key prefix (default: 'socket.io')
  key: 'socket.io',

  // Timeout for requests between servers (ms)
  requestsTimeout: 5000,

  // Timeout for publish operations (ms)
  publishTimeout: 5000,

  // Parser for encoding/decoding messages
  parser: JSON, // Or custom parser
});
```

### **Custom Parser (MessagePack)**

```typescript
import { createAdapter } from '@socket.io/redis-adapter';
import * as msgpack from 'notepack.io';

// Use MessagePack for efficient binary encoding
const adapter = createAdapter(pubClient, subClient, {
  parser: {
    encode: (data) => msgpack.encode(data),
    decode: (data) => msgpack.decode(data),
  },
});
```

### **Monitoring Adapter**

```typescript
@WebSocketGateway()
export class MonitoringGateway implements OnGatewayInit {
  afterInit(server: Server) {
    // Get adapter instance
    const adapter = server.of('/').adapter;

    // Monitor adapter events
    adapter.on('create-room', (room) => {
      console.log(`Room created: ${room}`);
    });

    adapter.on('delete-room', (room) => {
      console.log(`Room deleted: ${room}`);
    });

    adapter.on('join-room', (room, socketId) => {
      console.log(`Socket ${socketId} joined room ${room}`);
    });

    adapter.on('leave-room', (room, socketId) => {
      console.log(`Socket ${socketId} left room ${room}`);
    });

    // Server-to-server communication
    server.on('connection', (socket) => {
      // Send ping to other servers
      server.serverSideEmit('ping', {
        serverId: process.env.SERVER_ID,
        timestamp: Date.now(),
      });
    });

    server.on('ping', (data) => {
      console.log('Received ping from server:', data.serverId);
    });
  }
}
```

### **Failover and Reconnection**

```typescript
export class RedisIoAdapter extends IoAdapter {
  async connectToRedis(): Promise<void> {
    const pubClient = createClient({
      url: process.env.REDIS_URL,
      // Reconnection options
      socket: {
        reconnectStrategy: (retries) => {
          if (retries > 10) {
            return new Error('Max reconnect attempts reached');
          }
          // Exponential backoff: 100ms, 200ms, 400ms, ...
          return Math.min(retries * 100, 3000);
        },
      },
    });

    const subClient = pubClient.duplicate();

    // Handle reconnection events
    pubClient.on('reconnecting', () => {
      this.logger.warn('Redis PUB client reconnecting...');
    });

    pubClient.on('ready', () => {
      this.logger.log('Redis PUB client ready');
    });

    subClient.on('reconnecting', () => {
      this.logger.warn('Redis SUB client reconnecting...');
    });

    await Promise.all([pubClient.connect(), subClient.connect()]);

    this.adapterConstructor = createAdapter(pubClient, subClient);
  }
}
```

### **Redis Cluster Support**

```typescript
import Redis from 'ioredis';

export class RedisIoAdapter extends IoAdapter {
  async connectToRedis(): Promise<void> {
    // Redis Cluster
    const pubClient = new Redis.Cluster([
      { host: 'redis-node1', port: 6379 },
      { host: 'redis-node2', port: 6379 },
      { host: 'redis-node3', port: 6379 },
    ]);

    const subClient = pubClient.duplicate();

    this.adapterConstructor = createAdapter(pubClient, subClient);
  }
}
```

### **Performance Metrics**

```typescript
@WebSocketGateway()
export class MetricsGateway implements OnGatewayInit {
  private metrics = {
    messagesPublished: 0,
    messagesReceived: 0,
    rooms: 0,
  };

  afterInit(server: Server) {
    const adapter = server.of('/').adapter;

    // Track published messages
    const originalEmit = server.emit.bind(server);
    server.emit = (...args) => {
      this.metrics.messagesPublished++;
      return originalEmit(...args);
    };

    // Track rooms
    adapter.on('create-room', () => {
      this.metrics.rooms++;
    });

    adapter.on('delete-room', () => {
      this.metrics.rooms--;
    });

    // Expose metrics endpoint
    setInterval(() => {
      console.log('Metrics:', this.metrics);
    }, 60000); // Every minute
  }
}
```

**Interview Tip**: **Redis Adapter** (`@socket.io/redis-adapter`) enables **Socket.IO horizontal scaling** by using **Redis pub/sub** to synchronize events across multiple servers. **Purpose**: default Socket.IO adapter is in-memory (single server), doesn't work with multiple instances - Redis Adapter makes all servers communicate via Redis. **How it works**: (1) **Publish** - when server emits event (`io.emit()`, `io.to(room).emit()`), adapter publishes to Redis channel `socket.io#/#`; (2) **Subscribe** - all servers subscribe to Redis channels, receive published events, emit to their connected clients; (3) **Rooms** - room membership stored in Redis, `socket.join(room)` published to all servers, `io.to(room).emit()` queries Redis for members across all servers. **Setup**: `createAdapter(pubClient, subClient)` - requires TWO Redis clients (pub and sub, because Redis pub/sub blocks subscriber), apply with `server.adapter(redisAdapter)`. **What syncs**: broadcasts (`io.emit()`), room broadcasts (`io.to()`), room joins/leaves, namespace events, `fetchSockets()` (queries all servers), `serverSideEmit()` (server-to-server). **Benefits**: horizontal scaling (add/remove servers dynamically), high availability (server crash doesn't affect others), load distribution. **Redis channels**: `socket.io#/#` (main), `socket.io#/#-request#` (requests), `socket.io#/#-response#` (responses). **Namespaces**: each namespace has separate adapter instance, events isolated. **Options**: `key` (Redis key prefix), `requestsTimeout` (cross-server timeout), `parser` (custom encoder like MessagePack). **Performance**: use Redis Cluster for HA, monitor pub/sub metrics, implement reconnection strategy. **vs sticky sessions**: Redis Adapter allows **any-server routing** (load balancer round robin), sticky sessions route same user to same server (simpler but less resilient). **Trade-offs**: adds Redis dependency (use Redis Sentinel/Cluster for HA), slight latency (network hop), more complex architecture.

</details>
41. Why do you need Redis for multi-server WebSocket deployments?

<details>
<summary><strong>Answer</strong></summary>

Redis is needed because **Socket.IO servers don't communicate by default** - each server only knows about its own connected clients. In multi-server deployments with load balancer, **User A on Server 1 can't send messages to User B on Server 2** without Redis. **Problem**: (1) servers have **isolated in-memory state** (rooms, connections), (2) broadcasts only reach clients on **same server**, (3) room membership **not shared** between servers. **Redis solution**: acts as **message broker** using **pub/sub** to relay events between all servers. When Server 1 emits, it publishes to Redis, Server 2 subscribes and receives, then emits to its clients. **Alternatives**: RabbitMQ, Kafka (more complex), Sticky Sessions (doesn't scale well, no HA).

### **Problem Illustration**

```typescript
// Scenario: 2 servers, load balancer, NO Redis

// SERVER 1
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('send-message')
  handleMessage(@MessageBody() data: any) {
    // Broadcasts to ALL clients
    this.server.emit('new-message', data);
    // ❌ PROBLEM: Only reaches clients connected to Server 1
    // User B on Server 2 doesn't receive this message
  }
}

// Load Balancer distributes:
// - User A → Server 1
// - User B → Server 2

// User A sends message:
//   User A → Server 1 → server.emit('new-message')
//   ✅ User A receives (on Server 1)
//   ❌ User B does NOT receive (on Server 2)
//   Result: Message not delivered to half the users!
```

### **Why Default Adapter Fails**

```typescript
// Default In-Memory Adapter

// SERVER 1 has its own state:
{
  rooms: {
    'room-1': Set(['socket-a1', 'socket-a2']),  // Only Server 1 clients
  },
  sockets: {
    'socket-a1': { userId: 'A', rooms: ['room-1'] },
    'socket-a2': { userId: 'C', rooms: ['room-1'] },
  }
}

// SERVER 2 has DIFFERENT state:
{
  rooms: {
    'room-1': Set(['socket-b1']),  // Only Server 2 clients
  },
  sockets: {
    'socket-b1': { userId: 'B', rooms: ['room-1'] },
  }
}

// When Server 1 emits to 'room-1':
server.to('room-1').emit('message', 'Hello');
// → Server 1 checks its local state: room-1 has [socket-a1, socket-a2]
// → Emits to socket-a1 and socket-a2
// → socket-b1 on Server 2 is NOT in Server 1's state
// → ❌ User B doesn't receive message
```

### **Redis Solution**

```typescript
// WITH Redis Adapter

// SERVER 1
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;  // Using Redis Adapter

  @SubscribeMessage('send-message')
  handleMessage(@MessageBody() data: any) {
    // Broadcasts to ALL clients across ALL servers
    this.server.emit('new-message', data);
    
    // Redis Adapter process:
    // 1. Server 1 publishes to Redis: { event: 'new-message', data }
    // 2. Redis broadcasts to ALL subscribed servers (Server 1, 2, 3...)
    // 3. Server 2 receives from Redis
    // 4. Server 2 emits to its connected clients
    // ✅ User A receives (Server 1)
    // ✅ User B receives (Server 2)
  }
}

// Flow:
User A (Server 1) sends message
  → Server 1: server.emit('new-message', data)
  → Redis Adapter: PUBLISH socket.io#/# {event, data}
  → Redis: broadcasts to all subscribers
  → Server 1: receives, emits to local clients (User A)
  → Server 2: receives, emits to local clients (User B)
  → Server 3: receives, emits to local clients (User C)
```

### **Why Not Alternatives?**

#### **1. Sticky Sessions (Session Affinity)**

```nginx
# Load balancer with ip_hash
upstream backend {
    ip_hash;  # Same IP always goes to same server
    server server1:3000;
    server server2:3000;
}
```

**Problems:**
- ❌ **Not true scaling**: User always routes to same server (no load distribution)
- ❌ **No HA**: If user's server crashes, they disconnect (can't failover to another server)
- ❌ **Unbalanced load**: Popular users overload one server
- ❌ **Mobile/dynamic IPs**: IP changes → routes to different server → loses session
- ✅ **Pro**: Simple, no Redis dependency

#### **2. Database Polling**

```typescript
// Check database every second for new messages
setInterval(async () => {
  const newMessages = await db.messages.findNew();
  newMessages.forEach(msg => {
    server.emit('new-message', msg);
  });
}, 1000);
```

**Problems:**
- ❌ **High latency**: 1+ second delay for messages
- ❌ **Database load**: Constant polling hammers database
- ❌ **Not real-time**: Defeats purpose of WebSockets
- ❌ **Inefficient**: Queries even when no new messages

#### **3. Direct Server-to-Server HTTP**

```typescript
// Server 1 sends HTTP request to Server 2, 3, ...
await axios.post('http://server2:3000/broadcast', { event, data });
await axios.post('http://server3:3000/broadcast', { event, data });
```

**Problems:**
- ❌ **Service discovery**: Need to know all server IPs (dynamic scaling breaks)
- ❌ **Network overhead**: N servers = N HTTP requests per emit
- ❌ **Complexity**: Manual retry, timeout, error handling
- ❌ **Latency**: Sequential HTTP calls add delay

### **Why Redis is Ideal**

```typescript
// Redis Pub/Sub Benefits:

// 1. DECOUPLED: Servers don't need to know about each other
//    - Add/remove servers dynamically
//    - No service discovery needed

// 2. EFFICIENT: Single publish reaches all servers
//    - O(1) operation regardless of server count
//    - Redis handles fan-out

// 3. LOW LATENCY: Sub-millisecond delivery
//    - Redis is in-memory
//    - Persistent TCP connections

// 4. RELIABLE: Built-in reconnection
//    - Redis client auto-reconnects
//    - Buffered messages during reconnect

// 5. SIMPLE: Socket.IO adapter handles complexity
//    - Transparent to application code
//    - Same API works with/without Redis
```

### **Redis Pub/Sub Mechanism**

```typescript
// How Redis Adapter uses pub/sub:

// 1. SETUP (each server)
const pubClient = createClient();  // Publish messages
const subClient = pubClient.duplicate();  // Subscribe to messages

// Why 2 clients? Redis pub/sub blocks subscriber from other ops

await pubClient.connect();
await subClient.connect();

const adapter = createAdapter(pubClient, subClient);
io.adapter(adapter);

// 2. SUBSCRIBE (each server)
subClient.subscribe('socket.io#/#', (message) => {
  // Received event from another server
  const { event, data, rooms, namespace } = parse(message);
  
  // Emit to local clients
  if (rooms) {
    io.to(rooms).emit(event, data);
  } else {
    io.emit(event, data);
  }
});

// 3. PUBLISH (when server emits)
io.emit('event', data);
// → Adapter intercepts
// → pubClient.publish('socket.io#/#', serialize({ event, data }))
// → Redis broadcasts to all subscribers
// → All servers receive and emit locally
```

### **Room Synchronization**

```typescript
// Problem: Rooms are local to each server

// WITHOUT Redis:
// Server 1: socket.join('room-1')
//   → Server 1 tracks: room-1 = [socket-a]
//   → Server 2 doesn't know about this

// Server 2: io.to('room-1').emit('message')
//   → Server 2 checks local state: room-1 is empty
//   → No one receives message (even though User A is in room-1 on Server 1)

// WITH Redis:
// Server 1: socket.join('room-1')
//   → Adapter publishes to Redis: { action: 'join', room: 'room-1', socketId }
//   → All servers update their tracking

// Server 2: io.to('room-1').emit('message')
//   → Adapter publishes to Redis: { action: 'emit', room: 'room-1', data }
//   → Server 1 receives, checks local clients in room-1, emits to them
```

### **Performance Comparison**

```
                  Single Server  Sticky Sessions  Redis Adapter
Horizontal Scale        ❌              ❌~              ✅
High Availability       ❌              ❌               ✅
Load Distribution       N/A            ❌               ✅
Real-time Sync          N/A            N/A            ✅
Cross-server Rooms      N/A            ❌               ✅
Complexity             Simple         Simple         Medium
Dependencies           None           None           Redis
Latency                Low            Low            Low+1ms
Failover               ❌              ❌               ✅
```

### **When You DON'T Need Redis**

1. **Single server deployment**
   - Small apps, prototypes, dev environment
   - No scaling requirements

2. **Sticky sessions acceptable**
   - Small user base
   - Server crashes rare/acceptable
   - Simple architecture priority

3. **Separate apps per region**
   - Users don't interact across regions
   - Each region has independent infrastructure

### **Redis High Availability**

```typescript
// Production: Use Redis Sentinel or Cluster

// Redis Sentinel (master-slave replication)
import Redis from 'ioredis';

const pubClient = new Redis({
  sentinels: [
    { host: 'sentinel1', port: 26379 },
    { host: 'sentinel2', port: 26379 },
    { host: 'sentinel3', port: 26379 },
  ],
  name: 'mymaster',
});

const subClient = pubClient.duplicate();

// Redis Cluster (sharded)
const pubClient = new Redis.Cluster([
  { host: 'cluster-node1', port: 6379 },
  { host: 'cluster-node2', port: 6379 },
  { host: 'cluster-node3', port: 6379 },
]);
```

**Interview Tip**: **Redis is required** for multi-server WebSockets because **servers don't communicate by default** - each has **isolated in-memory state** (rooms, connections). **Problem**: User A on Server1 sends message → `server.emit()` → only reaches Server1 clients → User B on Server2 doesn't receive. **Root cause**: default adapter stores rooms/sockets in **local memory**, `io.to(room).emit()` only checks **local state**, other servers not aware. **Redis solution**: acts as **message broker** with **pub/sub** - Server1 emits → publishes to Redis `socket.io#/#` channel → all servers subscribed to channel receive → Server2 emits to its local clients → User B receives. **Why Redis**: (1) **decoupled** - servers don't need to know each other (add/remove dynamically), (2) **efficient** - single publish reaches all (O(1)), (3) **low latency** - in-memory, sub-millisecond, (4) **reliable** - auto-reconnect, (5) **simple** - Socket.IO adapter handles complexity. **Alternatives**: (1) **Sticky sessions** (ip_hash) - same user always routes to same server (no true scaling, no HA, no cross-server rooms), (2) **Database polling** - check DB every second (high latency, DB load, not real-time), (3) **Server-to-server HTTP** - manual requests to all servers (service discovery, network overhead, complexity). **Redis Adapter** requires **2 clients** (pub and sub, because Redis pub/sub blocks subscriber). **HA**: use Redis Sentinel (master-slave) or Redis Cluster (sharded) in production. **When not needed**: single server, sticky sessions acceptable, separate regional apps. **Key insight**: WebSockets are **stateful** (long-lived connections), load balancers distribute clients to different servers, **state synchronization** required for cross-server messaging - Redis provides this via pub/sub.

</details>

## Testing

42. How do you test WebSocket Gateways?

<details>
<summary><strong>Answer</strong></summary>

Test WebSocket Gateways using: **(1) Unit tests** - test gateway methods in isolation with mocked `@WebSocketServer()` server and `Socket` clients using Jest; **(2) Integration tests** - spin up real Socket.IO server with `Test.createTestingModule()`, create real Socket.IO client (`io-client`), emit events and verify responses; **(3) E2E tests** - full app with `@nestjs/testing`, connect real clients, test complete flows (connect, authenticate, send messages, disconnect). **Tools**: Jest for unit/integration, `socket.io-client` for real client connections, `@nestjs/testing` for module creation, **mock** for Socket/Server objects with Jest spies.

### **Unit Testing (Isolated Gateway Methods)**

```typescript
// chat.gateway.ts
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  constructor(private messageService: MessageService) {}

  @SubscribeMessage('send-message')
  async handleMessage(
    @MessageBody() data: { roomId: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    const user = client.data.user;

    // Save message
    const message = await this.messageService.create({
      userId: user.id,
      roomId: data.roomId,
      content: data.message,
    });

    // Broadcast to room
    this.server.to(data.roomId).emit('new-message', {
      id: message.id,
      content: message.content,
      user: { id: user.id, username: user.username },
    });

    return {
      event: 'message-sent',
      data: { messageId: message.id },
    };
  }
}

// chat.gateway.spec.ts
import { Test } from '@nestjs/testing';
import { ChatGateway } from './chat.gateway';
import { MessageService } from './message.service';
import { Socket, Server } from 'socket.io';

describe('ChatGateway', () => {
  let gateway: ChatGateway;
  let messageService: MessageService;
  let mockServer: jest.Mocked<Server>;
  let mockClient: jest.Mocked<Socket>;

  beforeEach(async () => {
    // Mock MessageService
    const mockMessageService = {
      create: jest.fn(),
    };

    const module = await Test.createTestingModule({
      providers: [
        ChatGateway,
        {
          provide: MessageService,
          useValue: mockMessageService,
        },
      ],
    }).compile();

    gateway = module.get<ChatGateway>(ChatGateway);
    messageService = module.get<MessageService>(MessageService);

    // Mock Socket.IO Server
    mockServer = {
      to: jest.fn().mockReturnThis(),
      emit: jest.fn(),
    } as any;

    // Mock Socket.IO Client
    mockClient = {
      id: 'socket-123',
      data: {
        user: { id: 'user-1', username: 'john_doe' },
      },
    } as any;

    // Inject mock server
    gateway.server = mockServer;
  });

  describe('handleMessage', () => {
    it('should save message and broadcast to room', async () => {
      // Arrange
      const messageData = {
        roomId: 'room-123',
        message: 'Hello World',
      };

      const createdMessage = {
        id: 'msg-1',
        userId: 'user-1',
        roomId: 'room-123',
        content: 'Hello World',
      };

      jest.spyOn(messageService, 'create').mockResolvedValue(createdMessage);

      // Act
      const result = await gateway.handleMessage(messageData, mockClient);

      // Assert
      expect(messageService.create).toHaveBeenCalledWith({
        userId: 'user-1',
        roomId: 'room-123',
        content: 'Hello World',
      });

      expect(mockServer.to).toHaveBeenCalledWith('room-123');
      expect(mockServer.emit).toHaveBeenCalledWith('new-message', {
        id: 'msg-1',
        content: 'Hello World',
        user: { id: 'user-1', username: 'john_doe' },
      });

      expect(result).toEqual({
        event: 'message-sent',
        data: { messageId: 'msg-1' },
      });
    });

    it('should handle errors gracefully', async () => {
      // Arrange
      const messageData = { roomId: 'room-123', message: 'Hello' };
      jest.spyOn(messageService, 'create').mockRejectedValue(
        new Error('Database error'),
      );

      // Act & Assert
      await expect(
        gateway.handleMessage(messageData, mockClient),
      ).rejects.toThrow('Database error');
    });
  });
});
```

### **Integration Testing (Real Socket.IO Server)**

```typescript
// chat.gateway.integration.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';
import { MessageService } from './message.service';
import { io, Socket as ClientSocket } from 'socket.io-client';
import { Server } from 'socket.io';

describe('ChatGateway (Integration)', () => {
  let app: INestApplication;
  let gateway: ChatGateway;
  let clientSocket: ClientSocket;
  let port: number;

  beforeEach(async () => {
    // Create testing module
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        ChatGateway,
        {
          provide: MessageService,
          useValue: {
            create: jest.fn().mockResolvedValue({
              id: 'msg-1',
              content: 'Test message',
            }),
          },
        },
      ],
    }).compile();

    app = module.createNestApplication();
    await app.listen(0); // Random port

    // Get the port
    const address = app.getHttpServer().address();
    port = address.port;

    gateway = module.get<ChatGateway>(ChatGateway);
  });

  afterEach(async () => {
    if (clientSocket) {
      clientSocket.disconnect();
    }
    await app.close();
  });

  it('should connect and receive events', (done) => {
    // Connect client
    clientSocket = io(`http://localhost:${port}`, {
      transports: ['websocket'],
    });

    clientSocket.on('connect', () => {
      expect(clientSocket.connected).toBe(true);
      done();
    });
  });

  it('should send and receive messages', (done) => {
    clientSocket = io(`http://localhost:${port}`);

    // Listen for response
    clientSocket.on('message-sent', (data) => {
      expect(data).toEqual({
        messageId: 'msg-1',
      });
      done();
    });

    // Wait for connection
    clientSocket.on('connect', () => {
      // Emit message
      clientSocket.emit('send-message', {
        roomId: 'room-123',
        message: 'Hello',
      });
    });
  });

  it('should broadcast to room', (done) => {
    // Create two clients
    const client1 = io(`http://localhost:${port}`);
    const client2 = io(`http://localhost:${port}`);

    let connected = 0;
    const checkConnected = () => {
      connected++;
      if (connected === 2) {
        // Both connected, join same room
        client1.emit('join-room', 'room-123');
        client2.emit('join-room', 'room-123');

        // Client 1 sends message
        setTimeout(() => {
          client1.emit('send-message', {
            roomId: 'room-123',
            message: 'Hello room',
          });
        }, 100);
      }
    };

    client1.on('connect', checkConnected);
    client2.on('connect', checkConnected);

    // Client 2 should receive broadcast
    client2.on('new-message', (data) => {
      expect(data.content).toBe('Hello room');
      client1.disconnect();
      client2.disconnect();
      done();
    });
  });
});
```

### **E2E Testing**

```typescript
// test/websocket.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import { AppModule } from '../src/app.module';
import { io, Socket as ClientSocket } from 'socket.io-client';

describe('WebSocket E2E', () => {
  let app: INestApplication;
  let client1: ClientSocket;
  let client2: ClientSocket;
  let port: number;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
    await app.listen(0);

    const address = app.getHttpServer().address();
    port = address.port;
  });

  afterAll(async () => {
    await app.close();
  });

  afterEach(() => {
    if (client1) client1.disconnect();
    if (client2) client2.disconnect();
  });

  describe('Chat Flow', () => {
    it('should authenticate, join room, and exchange messages', (done) => {
      const token = 'valid-token';

      client1 = io(`http://localhost:${port}/chat`, {
        auth: { token },
      });

      client2 = io(`http://localhost:${port}/chat`, {
        auth: { token },
      });

      let messagesReceived = 0;

      // Client 1: Send message after joining
      client1.on('room-joined', (data) => {
        client1.emit('send-message', {
          roomId: 'test-room',
          message: 'Hello from client 1',
        });
      });

      // Client 2: Receive message
      client2.on('new-message', (data) => {
        expect(data.content).toBe('Hello from client 1');
        messagesReceived++;

        if (messagesReceived === 1) {
          done();
        }
      });

      // Wait for both to connect
      let connected = 0;
      const onConnect = () => {
        connected++;
        if (connected === 2) {
          // Both connected, join room
          client1.emit('join-room', 'test-room');
          client2.emit('join-room', 'test-room');
        }
      };

      client1.on('connect', onConnect);
      client2.on('connect', onConnect);
    }, 10000); // 10 second timeout
  });

  describe('Error Handling', () => {
    it('should reject connection without auth token', (done) => {
      client1 = io(`http://localhost:${port}/chat`, {
        auth: {}, // No token
      });

      client1.on('connect_error', (error) => {
        expect(error.message).toContain('Authentication required');
        done();
      });
    });

    it('should receive validation errors', (done) => {
      const token = 'valid-token';

      client1 = io(`http://localhost:${port}/chat`, {
        auth: { token },
      });

      client1.on('connect', () => {
        // Send invalid message (empty)
        client1.emit('send-message', {
          roomId: 'test-room',
          message: '', // Empty message
        });
      });

      client1.on('exception', (error) => {
        expect(error.error).toBe('VALIDATION_ERROR');
        expect(error.message).toContain('Message cannot be empty');
        done();
      });
    });
  });
});
```

### **Testing Lifecycle Hooks**

```typescript
// presence.gateway.spec.ts
describe('PresenceGateway', () => {
  let gateway: PresenceGateway;
  let mockServer: jest.Mocked<Server>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [PresenceGateway, UserService],
    }).compile();

    gateway = module.get<PresenceGateway>(PresenceGateway);

    mockServer = {
      use: jest.fn(),
      emit: jest.fn(),
    } as any;
  });

  describe('afterInit', () => {
    it('should register authentication middleware', () => {
      gateway.afterInit(mockServer);

      expect(mockServer.use).toHaveBeenCalled();
    });
  });

  describe('handleConnection', () => {
    it('should set user online and broadcast status', async () => {
      const mockClient = {
        id: 'socket-123',
        data: { user: { id: 'user-1', username: 'john' } },
        join: jest.fn(),
      } as any;

      gateway.server = mockServer;

      await gateway.handleConnection(mockClient);

      expect(mockServer.emit).toHaveBeenCalledWith('user-status', {
        userId: 'user-1',
        username: 'john',
        status: 'online',
        timestamp: expect.any(Date),
      });
    });
  });

  describe('handleDisconnect', () => {
    it('should set user offline and broadcast status', async () => {
      const mockClient = {
        id: 'socket-123',
        data: { user: { id: 'user-1', username: 'john' } },
      } as any;

      gateway.server = mockServer;

      await gateway.handleDisconnect(mockClient);

      expect(mockServer.emit).toHaveBeenCalledWith('user-status', {
        userId: 'user-1',
        username: 'john',
        status: 'offline',
        lastSeen: expect.any(Date),
        timestamp: expect.any(Date),
      });
    });
  });
});
```

### **Mock Helpers**

```typescript
// test/helpers/socket.mock.ts
import { Socket, Server } from 'socket.io';

export function createMockSocket(overrides?: Partial<Socket>): jest.Mocked<Socket> {
  return {
    id: 'socket-123',
    data: { user: { id: 'user-1', username: 'test' } },
    emit: jest.fn(),
    to: jest.fn().mockReturnThis(),
    join: jest.fn(),
    leave: jest.fn(),
    disconnect: jest.fn(),
    handshake: {
      auth: { token: 'test-token' },
      address: '127.0.0.1',
    },
    ...overrides,
  } as any;
}

export function createMockServer(): jest.Mocked<Server> {
  return {
    emit: jest.fn(),
    to: jest.fn().mockReturnThis(),
    of: jest.fn().mockReturnThis(),
    use: jest.fn(),
    on: jest.fn(),
    fetchSockets: jest.fn().mockResolvedValue([]),
  } as any;
}

// Usage
import { createMockSocket, createMockServer } from './test/helpers/socket.mock';

describe('ChatGateway', () => {
  let gateway: ChatGateway;
  let mockServer: jest.Mocked<Server>;
  let mockClient: jest.Mocked<Socket>;

  beforeEach(() => {
    mockServer = createMockServer();
    mockClient = createMockSocket();
    
    gateway.server = mockServer;
  });

  it('should work', async () => {
    await gateway.handleMessage({ message: 'test' }, mockClient);
    expect(mockServer.emit).toHaveBeenCalled();
  });
});
```

**Interview Tip**: **Test WebSocket Gateways** with **(1) Unit tests** - isolate gateway methods, mock `@WebSocketServer() server` and `Socket client` with Jest (`jest.fn().mockReturnThis()` for chainable methods like `server.to().emit()`), test handler logic without real connections, verify service calls and emit arguments; **(2) Integration tests** - create real NestJS app with `Test.createTestingModule()`, start on random port (`app.listen(0)`), connect real Socket.IO client (`io('http://localhost:${port}')`), emit events from client, listen for responses, verify client receives events; **(3) E2E tests** - import `AppModule`, test complete flows (connect → authenticate → join room → send message → receive broadcast), test multiple clients interaction, verify error handling (connect_error, exception events). **Mocking**: (1) `@WebSocketServer() server` - mock with `{ to: jest.fn().mockReturnThis(), emit: jest.fn() }` to test `server.to(room).emit()` chain, (2) `Socket client` - mock with `{ id, data: { user }, emit: jest.fn() }`, (3) Services - use `{ provide: Service, useValue: mockService }`. **Integration setup**: `app.listen(0)` for random port, get port with `app.getHttpServer().address().port`, create client with `io('http://localhost:${port}')`, use `done()` callback for async tests, disconnect clients in `afterEach`. **Testing patterns**: (1) **Connect** - `client.on('connect', () => { expect(client.connected).toBe(true); done(); })`, (2) **Emit & receive** - emit event, listen for response, verify data, (3) **Broadcast** - create 2 clients, both join room, client1 sends, client2 receives, (4) **Errors** - emit invalid data, listen for 'exception' event, verify error structure. **Lifecycle hooks**: test `afterInit` (middleware registration), `handleConnection` (user online, broadcast), `handleDisconnect` (user offline, cleanup). **Helpers**: create mock factories (`createMockSocket()`, `createMockServer()`) for reusable mocks. **Best practices**: use `done()` for async, set timeout (10s for E2E), disconnect clients in cleanup, test both success and error cases, verify exact emit arguments with `toHaveBeenCalledWith()`. **Tools**: Jest, `socket.io-client`, `@nestjs/testing`, `Test.createTestingModule()`.

</details>
43. How do you mock WebSocket clients in tests?

<details>
<summary><strong>Answer</strong></summary>

Mock WebSocket clients using **Jest mocks** with key Socket.IO client methods: **(1) Basic mock** - `{ id, emit: jest.fn(), on: jest.fn(), disconnect: jest.fn(), data: {} }` for unit tests; **(2) Chainable methods** - mock `.to().emit()` with `jest.fn().mockReturnThis()` for method chaining; **(3) Event simulation** - store event listeners in Map, call them manually to simulate server events; **(4) Mock factory** - create reusable `createMockSocket(overrides)` function for consistent mocks; **(5) Integration tests** - use real `socket.io-client` instead of mocks for true E2E behavior. Mock both **Socket** (client connection) and **Server** (Socket.IO server instance).

### **Basic Mock Socket**

```typescript
// Simple mock for unit tests
const mockSocket = {
  id: 'socket-123',
  data: {},  // For storing user data
  emit: jest.fn(),
  on: jest.fn(),
  off: jest.fn(),
  disconnect: jest.fn(),
  join: jest.fn(),
  leave: jest.fn(),
  to: jest.fn().mockReturnThis(),  // Chainable
  handshake: {
    auth: { token: 'test-token' },
    address: '127.0.0.1',
  },
} as any;

// Usage in test
const result = await gateway.handleMessage(
  { message: 'Hello' },
  mockSocket,
);

expect(mockSocket.emit).toHaveBeenCalledWith('response', { success: true });
```

### **Mock Factory Function**

```typescript
// test/mocks/socket.mock.ts
import { Socket } from 'socket.io';

export function createMockSocket(overrides?: Partial<Socket>): jest.Mocked<Socket> {
  const mockSocket = {
    id: 'socket-' + Math.random().toString(36).substr(2, 9),
    data: {},
    
    // Basic methods
    emit: jest.fn(),
    on: jest.fn(),
    once: jest.fn(),
    off: jest.fn(),
    removeAllListeners: jest.fn(),
    
    // Connection methods
    disconnect: jest.fn(),
    
    // Room methods
    join: jest.fn().mockResolvedValue(undefined),
    leave: jest.fn(),
    
    // Chainable methods
    to: jest.fn().mockReturnThis(),
    in: jest.fn().mockReturnThis(),
    except: jest.fn().mockReturnThis(),
    compress: jest.fn().mockReturnThis(),
    volatile: jest.fn().mockReturnThis(),
    
    // Broadcasting
    broadcast: {
      emit: jest.fn(),
      to: jest.fn().mockReturnThis(),
    },
    
    // Handshake
    handshake: {
      auth: {},
      headers: {},
      address: '127.0.0.1',
      secure: false,
      issued: Date.now(),
    },
    
    // Connection info
    connected: true,
    disconnected: false,
    
    // Rooms
    rooms: new Set(['socket-123']),
    
    // Apply overrides
    ...overrides,
  } as any;

  return mockSocket;
}

// Usage
import { createMockSocket } from './test/mocks/socket.mock';

const mockSocket = createMockSocket({
  id: 'custom-id',
  data: { user: { id: '123', username: 'test' } },
});
```

### **Mock Server**

```typescript
// test/mocks/server.mock.ts
import { Server } from 'socket.io';

export function createMockServer(): jest.Mocked<Server> {
  const mockServer = {
    // Emit methods
    emit: jest.fn(),
    emitWithAck: jest.fn().mockResolvedValue([]),
    
    // Targeting methods
    to: jest.fn().mockReturnThis(),
    in: jest.fn().mockReturnThis(),
    except: jest.fn().mockReturnThis(),
    
    // Namespace
    of: jest.fn().mockReturnThis(),
    
    // Middleware
    use: jest.fn(),
    
    // Events
    on: jest.fn(),
    once: jest.fn(),
    off: jest.fn(),
    
    // Server-side emit
    serverSideEmit: jest.fn(),
    
    // Socket management
    fetchSockets: jest.fn().mockResolvedValue([]),
    socketsJoin: jest.fn(),
    socketsLeave: jest.fn(),
    disconnectSockets: jest.fn(),
    
    // Adapter
    adapter: jest.fn(),
  } as any;

  return mockServer;
}
```

### **Event Simulation Mock**

```typescript
// Mock that simulates event listeners
export function createMockSocketWithEvents() {
  const eventListeners = new Map<string, Function[]>();

  const mockSocket = {
    id: 'socket-123',
    data: {},
    
    // Store event listeners
    on: jest.fn((event: string, handler: Function) => {
      if (!eventListeners.has(event)) {
        eventListeners.set(event, []);
      }
      eventListeners.get(event)!.push(handler);
    }),
    
    // Emit events
    emit: jest.fn((event: string, ...args: any[]) => {
      // Optionally trigger listeners
      const listeners = eventListeners.get(event) || [];
      listeners.forEach(listener => listener(...args));
    }),
    
    // Helper to trigger events from server
    triggerEvent(event: string, ...args: any[]) {
      const listeners = eventListeners.get(event) || [];
      listeners.forEach(listener => listener(...args));
    },
    
    // Rest of mock
    disconnect: jest.fn(),
    join: jest.fn(),
    leave: jest.fn(),
    to: jest.fn().mockReturnThis(),
  } as any;

  return mockSocket;
}

// Usage: Simulate server sending event to client
const mockSocket = createMockSocketWithEvents();

// Register listener
mockSocket.on('message', (data) => {
  console.log('Received:', data);
});

// Simulate server emitting
mockSocket.triggerEvent('message', { text: 'Hello' });
// Output: "Received: { text: 'Hello' }"
```

### **Testing with Mock Client**

```typescript
// chat.gateway.spec.ts
import { Test } from '@nestjs/testing';
import { ChatGateway } from './chat.gateway';
import { MessageService } from './message.service';
import { createMockSocket, createMockServer } from './test/mocks';

describe('ChatGateway', () => {
  let gateway: ChatGateway;
  let messageService: MessageService;
  let mockServer: jest.Mocked<Server>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        ChatGateway,
        {
          provide: MessageService,
          useValue: {
            create: jest.fn().mockResolvedValue({
              id: 'msg-1',
              content: 'Test',
            }),
          },
        },
      ],
    }).compile();

    gateway = module.get<ChatGateway>(ChatGateway);
    messageService = module.get<MessageService>(MessageService);

    mockServer = createMockServer();
    gateway.server = mockServer;
  });

  describe('handleMessage', () => {
    it('should emit message to room', async () => {
      const mockSocket = createMockSocket({
        data: { user: { id: 'user-1', username: 'john' } },
      });

      await gateway.handleMessage(
        { roomId: 'room-1', message: 'Hello' },
        mockSocket,
      );

      // Verify server.to().emit() was called correctly
      expect(mockServer.to).toHaveBeenCalledWith('room-1');
      expect(mockServer.emit).toHaveBeenCalledWith('new-message', {
        id: 'msg-1',
        content: 'Hello',
        user: { id: 'user-1', username: 'john' },
      });
    });
  });

  describe('handleConnection', () => {
    it('should join user to personal room', async () => {
      const mockSocket = createMockSocket({
        data: { user: { id: 'user-1' } },
      });

      await gateway.handleConnection(mockSocket);

      expect(mockSocket.join).toHaveBeenCalledWith('user:user-1');
      expect(mockServer.emit).toHaveBeenCalledWith('user-status', {
        userId: 'user-1',
        status: 'online',
      });
    });
  });
});
```

### **Mock Client-Server Interaction**

```typescript
// Simulate full client-server interaction
describe('Client-Server Interaction', () => {
  it('should handle request-response pattern', async () => {
    const mockSocket = createMockSocketWithEvents();
    mockSocket.data.user = { id: 'user-1', username: 'john' };

    // Client sends request
    const requestData = { query: 'test' };
    let responseData: any;

    // Client listens for response
    mockSocket.on('response', (data) => {
      responseData = data;
    });

    // Gateway handles request
    const result = await gateway.handleRequest(requestData, mockSocket);

    // Result includes event name for client
    expect(result).toEqual({
      event: 'response',
      data: { result: 'success' },
    });

    // Simulate server emitting response
    mockSocket.triggerEvent('response', result.data);

    expect(responseData).toEqual({ result: 'success' });
  });
});
```

### **Mock Multiple Clients**

```typescript
describe('Multiple Clients', () => {
  it('should broadcast to all clients in room', async () => {
    const client1 = createMockSocket({ id: 'socket-1' });
    const client2 = createMockSocket({ id: 'socket-2' });
    const client3 = createMockSocket({ id: 'socket-3' });

    // All join same room
    await gateway.handleJoinRoom('room-1', client1);
    await gateway.handleJoinRoom('room-1', client2);
    await gateway.handleJoinRoom('room-1', client3);

    // Client 1 sends message
    await gateway.handleMessage(
      { roomId: 'room-1', message: 'Hello' },
      client1,
    );

    // Verify broadcast
    expect(mockServer.to).toHaveBeenCalledWith('room-1');
    expect(mockServer.emit).toHaveBeenCalledWith('new-message', expect.any(Object));
  });
});
```

### **Mock with Spies**

```typescript
describe('Event Spies', () => {
  it('should track emit calls', () => {
    const mockSocket = createMockSocket();
    const emitSpy = jest.spyOn(mockSocket, 'emit');

    // Trigger some gateway action that emits
    gateway.sendNotification(mockSocket, 'Test notification');

    // Verify emit was called
    expect(emitSpy).toHaveBeenCalledTimes(1);
    expect(emitSpy).toHaveBeenCalledWith('notification', {
      message: 'Test notification',
    });

    emitSpy.mockRestore();
  });
});
```

### **Real Client for Integration Tests**

```typescript
// For integration tests, use REAL socket.io-client instead of mocks
import { io, Socket as ClientSocket } from 'socket.io-client';

describe('ChatGateway (Integration)', () => {
  let app: INestApplication;
  let clientSocket: ClientSocket;
  let port: number;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [ChatGateway, MessageService],
    }).compile();

    app = module.createNestApplication();
    await app.listen(0);

    port = app.getHttpServer().address().port;
  });

  afterEach(async () => {
    if (clientSocket) clientSocket.disconnect();
    await app.close();
  });

  it('should connect and receive events', (done) => {
    // Use REAL client
    clientSocket = io(`http://localhost:${port}`, {
      auth: { token: 'valid-token' },
    });

    clientSocket.on('connect', () => {
      expect(clientSocket.connected).toBe(true);
      done();
    });
  });

  it('should emit and receive messages', (done) => {
    clientSocket = io(`http://localhost:${port}`);

    clientSocket.on('message-sent', (data) => {
      expect(data.messageId).toBeDefined();
      done();
    });

    clientSocket.on('connect', () => {
      clientSocket.emit('send-message', {
        roomId: 'test-room',
        message: 'Hello',
      });
    });
  });
});
```

### **Assertion Helpers**

```typescript
// test/helpers/socket.assertions.ts

export function expectEmitToRoom(
  mockServer: jest.Mocked<Server>,
  room: string,
  event: string,
  data?: any,
) {
  expect(mockServer.to).toHaveBeenCalledWith(room);
  
  if (data) {
    expect(mockServer.emit).toHaveBeenCalledWith(event, data);
  } else {
    expect(mockServer.emit).toHaveBeenCalledWith(event, expect.any(Object));
  }
}

export function expectBroadcast(
  mockServer: jest.Mocked<Server>,
  event: string,
  data?: any,
) {
  if (data) {
    expect(mockServer.emit).toHaveBeenCalledWith(event, data);
  } else {
    expect(mockServer.emit).toHaveBeenCalledWith(event, expect.any(Object));
  }
}

// Usage
expectEmitToRoom(mockServer, 'room-1', 'new-message', {
  content: 'Hello',
});

expectBroadcast(mockServer, 'announcement');
```

**Interview Tip**: **Mock WebSocket clients** with **Jest mocks** for unit tests: create mock Socket with `{ id, data: {}, emit: jest.fn(), on: jest.fn(), disconnect: jest.fn(), join: jest.fn(), leave: jest.fn(), to: jest.fn().mockReturnThis() }` - note `.mockReturnThis()` for **chainable methods** like `socket.to(room).emit()`. **Mock factory**: create `createMockSocket(overrides)` function for reusable mocks with default values, override with custom `{ id, data: { user: {...} } }`. **Mock Server**: `{ emit: jest.fn(), to: jest.fn().mockReturnThis(), use: jest.fn(), on: jest.fn(), fetchSockets: jest.fn().mockResolvedValue([]) }`. **Testing patterns**: (1) **Handler methods** - pass mock socket to handler, verify `messageService` calls and `server.emit()` arguments with `toHaveBeenCalledWith()`; (2) **Lifecycle hooks** - test `handleConnection` (verify `join()`, `emit()` for status), `handleDisconnect` (verify cleanup); (3) **Chainable calls** - test `server.to('room').emit()` by verifying `to` called with room, then `emit` called with event/data. **Event simulation**: store listeners in Map (`on` saves callbacks), create `triggerEvent(event, data)` method to manually invoke listeners (simulate server sending to client). **Multiple clients**: create multiple mocks (`client1`, `client2`, `client3`), test broadcast reaches correct clients. **Spies**: `jest.spyOn(mockSocket, 'emit')` to track calls, `mockRestore()` to cleanup. **Integration vs Unit**: use **mocks** for unit tests (fast, isolated), use **real `socket.io-client`** for integration/E2E tests (true behavior, connect to real server). **Helpers**: create assertion functions (`expectEmitToRoom(server, room, event, data)`) for cleaner tests. **Key mocks**: Socket (client connection), Server (Socket.IO server), Services (message service, user service). **Best practices**: mock `.to().emit()` chain with `.mockReturnThis()`, test both success and error paths, verify exact emit arguments, use `done()` for async tests, disconnect/cleanup in `afterEach`.

</details>

## Best Practices

44. When should you use WebSockets vs Server-Sent Events (SSE)?

<details>
<summary><strong>Answer</strong></summary>

Use **WebSockets** for **bidirectional real-time** communication (chat, gaming, collaborative editing) where **both client and server send messages frequently**. Use **SSE (Server-Sent Events)** for **unidirectional server-to-client** updates (notifications, live feeds, stock tickers) where **client only receives** and occasionally sends HTTP requests. **Key difference**: WebSockets = **full-duplex** (both directions simultaneously), SSE = **half-duplex** (server-to-client push + client HTTP requests). **Trade-offs**: WebSockets (complex, requires Socket.IO library, stateful, scaling harder) vs SSE (simple, native browser API, HTTP-based, works with HTTP/2, easier scaling, automatic reconnection).

### **Comparison Table**

| Feature                  | WebSockets                        | Server-Sent Events (SSE)           |
|--------------------------|-----------------------------------|------------------------------------|
| **Communication**        | Bidirectional (full-duplex)      | Unidirectional (server-to-client) |
| **Protocol**             | ws:// or wss:// (custom)         | HTTP (same as REST)                |
| **Browser API**          | WebSocket (requires library)     | EventSource (native)               |
| **Client → Server**      | Real-time via socket             | Regular HTTP requests              |
| **Server → Client**      | Real-time push                   | Real-time push (SSE)               |
| **Connection**           | Persistent TCP                   | Persistent HTTP (chunked transfer) |
| **Overhead**             | Low after handshake              | Higher (HTTP headers per message)  |
| **Scaling**              | Harder (stateful, needs Redis)   | Easier (HTTP/2 multiplexing)       |
| **Reconnection**         | Manual (Socket.IO auto)          | Automatic (built-in)               |
| **Browser Support**      | All modern browsers              | All except IE (polyfill available) |
| **Firewalls/Proxies**    | Often blocked                    | Works (standard HTTP)              |
| **Text/Binary**          | Both                             | Text only                          |
| **Use Cases**            | Chat, gaming, collaboration      | Notifications, live feeds, updates |

### **Use WebSockets When...**

1. **Bidirectional communication required**
   - Chat applications (both users send/receive)
   - Real-time gaming (player actions + game state)
   - Collaborative editing (Google Docs-like)

2. **High frequency messages (both directions)**
   - Live multiplayer games
   - Real-time trading platforms
   - IoT device control

3. **Low latency critical**
   - Video conferencing signaling
   - Real-time collaboration
   - Live cursor positions

4. **Binary data needed**
   - Video/audio streaming metadata
   - File transfer progress

### **Use SSE When...**

1. **Only server needs to push**
   - Live notifications
   - Stock price updates
   - News feeds
   - Social media updates

2. **Client sends occasional requests**
   - Mark notification as read (HTTP POST)
   - User actions trigger updates

3. **Simple implementation preferred**
   - Simpler than WebSockets
   - No library needed (native browser API)

4. **HTTP infrastructure important**
   - Works with standard HTTP servers
   - Compatible with CDNs, proxies
   - HTTP/2 multiplexing benefits

### **WebSocket Example**

```typescript
// SERVER (NestJS WebSocket)
@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  // CLIENT → SERVER
  @SubscribeMessage('send-message')
  handleMessage(@MessageBody() data: { message: string }) {
    // SERVER → CLIENT (broadcast)
    this.server.emit('new-message', { message: data.message });
  }
}

// CLIENT
const socket = io('http://localhost:3000');

// CLIENT → SERVER
socket.emit('send-message', { message: 'Hello' });

// SERVER → CLIENT
socket.on('new-message', (data) => {
  console.log('New message:', data.message);
});
```

### **SSE Example**

```typescript
// SERVER (NestJS SSE)
import { Controller, Sse } from '@nestjs/common';
import { Observable, interval, map } from 'rxjs';

@Controller('events')
export class EventsController {
  // SERVER → CLIENT (SSE)
  @Sse('stream')
  streamEvents(): Observable<MessageEvent> {
    return interval(1000).pipe(
      map((num) => ({
        data: { message: `Update ${num}` },
      })),
    );
  }
}

// Or manual SSE with response
@Get('notifications')
async streamNotifications(@Res() res: Response) {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  // Send events
  setInterval(() => {
    res.write(`data: ${JSON.stringify({ message: 'Update' })}\n\n`);
  }, 1000);
}

// CLIENT (Native EventSource)
const eventSource = new EventSource('http://localhost:3000/events/stream');

// SERVER → CLIENT (receive)
eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received:', data.message);
};

// CLIENT → SERVER (regular HTTP)
fetch('http://localhost:3000/api/mark-read', {
  method: 'POST',
  body: JSON.stringify({ notificationId: '123' }),
});
```

### **Real-World Use Cases**

#### **Choose WebSockets:**

1. **Chat Application**
   ```typescript
   // Both users send/receive messages in real-time
   // Client A → Server → Client B
   // Client B → Server → Client A
   ```

2. **Multiplayer Game**
   ```typescript
   // Player actions (keyboard input) → Server
   // Game state updates ← Server
   // Both directions constantly
   ```

3. **Collaborative Editing (Google Docs)**
   ```typescript
   // User types character → Server
   // Server broadcasts change → All users
   // Real-time cursor positions
   ```

4. **Live Trading Platform**
   ```typescript
   // Place order → Server
   // Price updates ← Server
   // Order confirmations ← Server
   ```

#### **Choose SSE:**

1. **Notification System**
   ```typescript
   // Server → Client: "New message from John"
   // Client → Server (HTTP): Mark as read
   ```

2. **Stock Ticker**
   ```typescript
   // Server → Client: Price updates every second
   // Client rarely sends (subscribe/unsubscribe via HTTP)
   ```

3. **Live News Feed**
   ```typescript
   // Server → Client: New articles appear
   // Client → Server (HTTP): Load article details
   ```

4. **Server Monitoring Dashboard**
   ```typescript
   // Server → Client: CPU usage, memory, logs
   // Client → Server (HTTP): Change settings
   ```

5. **Social Media Updates**
   ```typescript
   // Server → Client: "John liked your post"
   // Client → Server (HTTP): Like/comment actions
   ```

### **Hybrid Approach**

```typescript
// Use both: SSE for notifications, HTTP for actions

// SERVER
@Controller('notifications')
export class NotificationsController {
  // SSE: Server pushes notifications
  @Sse('stream')
  streamNotifications(@Req() req): Observable<MessageEvent> {
    const userId = req.user.id;
    return this.notificationService.getStream(userId);
  }

  // HTTP: Client marks as read
  @Post('mark-read')
  markAsRead(@Body() { notificationId }: { notificationId: string }) {
    return this.notificationService.markAsRead(notificationId);
  }
}

// CLIENT
// Listen to notifications (SSE)
const eventSource = new EventSource('/notifications/stream');
eventSource.onmessage = (event) => {
  showNotification(JSON.parse(event.data));
};

// Mark as read (HTTP)
function markAsRead(id) {
  fetch('/notifications/mark-read', {
    method: 'POST',
    body: JSON.stringify({ notificationId: id }),
  });
}
```

### **Technical Differences**

```typescript
// WEBSOCKETS: Full-duplex
// Single connection, both directions
Client ↔↔↔ Server  (bidirectional on same connection)

// SSE: Half-duplex
// SSE for server-to-client, HTTP for client-to-server
Client ←←← Server  (SSE push)
Client →→→ Server  (HTTP request)
```

### **Performance Considerations**

```typescript
// WEBSOCKETS
// - Initial handshake: ~300 bytes
// - Per message: ~2-6 bytes overhead
// - Persistent connection (memory on server)

// SSE
// - Per message: HTTP headers (~200-500 bytes)
// - HTTP/2: Header compression (HPACK) reduces overhead
// - Persistent connection (lighter than WebSocket)
```

### **Scaling Comparison**

```typescript
// WEBSOCKETS
// - Stateful (client-server mapping)
// - Requires Redis Adapter for multi-server
// - Sticky sessions or Redis pub/sub

// SSE
// - Stateless (HTTP-based)
// - Works with standard load balancers
// - HTTP/2 multiplexing handles multiple streams
// - Easier horizontal scaling
```

### **Fallback Strategies**

```typescript
// WebSockets with SSE fallback
if (window.WebSocket) {
  // Use WebSocket
  const socket = new WebSocket('ws://localhost:3000');
} else if (window.EventSource) {
  // Fallback to SSE
  const eventSource = new EventSource('/events');
} else {
  // Fallback to polling
  setInterval(() => {
    fetch('/api/updates').then(res => res.json());
  }, 5000);
}

// Socket.IO automatically handles fallbacks:
// WebSocket → HTTP long-polling
```

**Interview Tip**: **WebSockets vs SSE**: use **WebSockets** for **bidirectional** real-time communication (chat, gaming, collaboration) where **both client and server send frequently** - full-duplex on single connection (`client ↔ server`). Use **SSE (Server-Sent Events)** for **unidirectional** server-to-client push (notifications, live feeds, stock tickers) where **client mostly receives**, sends occasional HTTP requests - half-duplex (`client ← server` for push, `client → server` for HTTP). **Key differences**: (1) **Direction** - WebSockets bidirectional, SSE server-to-client only; (2) **Protocol** - WebSockets custom `ws://`, SSE standard HTTP; (3) **API** - WebSockets requires library (Socket.IO), SSE native browser `EventSource`; (4) **Reconnection** - SSE automatic (built-in retry), WebSockets manual (Socket.IO handles); (5) **Scaling** - SSE easier (HTTP-based, stateless), WebSockets harder (stateful, needs Redis); (6) **Firewalls** - SSE works (HTTP), WebSockets often blocked. **Use WebSockets when**: (1) bidirectional needed (chat, gaming), (2) high frequency both ways, (3) low latency critical, (4) binary data needed. **Use SSE when**: (1) only server pushes (notifications), (2) client sends occasional requests, (3) simple implementation preferred, (4) HTTP infrastructure important (CDNs, proxies). **Trade-offs**: WebSockets (lower overhead per message, more complex, harder scaling) vs SSE (higher overhead, simpler, easier scaling, HTTP/2 benefits). **Real-world**: chat/gaming/collaboration → WebSockets, notifications/feeds/updates → SSE. **Hybrid**: SSE for server push + HTTP for client actions. **Fallback**: Socket.IO auto-falls back to long-polling, SSE falls back to polling.

</details>
45. How do you handle reconnection logic on the client side?

<details>
<summary><strong>Answer</strong></summary>

**Socket.IO handles reconnection automatically** by default with exponential backoff. Configure with **`reconnection: true`** (default), **`reconnectionAttempts: Infinity`**, **`reconnectionDelay: 1000`** (start delay), **`reconnectionDelayMax: 5000`** (max delay). **Manual handling**: listen for **`disconnect`** event (connection lost), **`reconnect_attempt`** (retry in progress), **`reconnect`** (successfully reconnected), **`reconnect_failed`** (all attempts failed). **Best practices**: (1) show UI feedback ("Connecting..."), (2) **resume state** after reconnect (re-join rooms, resync data), (3) **connection state recovery** - Socket.IO v4.6+ recovers missed messages during disconnect with `connectionStateRecovery: true`, (4) implement **exponential backoff** with max attempts, (5) **queue messages** during disconnect, send when reconnected.

### **Automatic Reconnection (Default)**

```typescript
import { io } from 'socket.io-client';

// Socket.IO reconnects automatically
const socket = io('http://localhost:3000', {
  reconnection: true,              // Enable auto-reconnect (default)
  reconnectionAttempts: Infinity,  // Try forever (default)
  reconnectionDelay: 1000,         // Initial delay: 1 second
  reconnectionDelayMax: 5000,      // Max delay: 5 seconds (exponential backoff)
  randomizationFactor: 0.5,        // Randomize delay (jitter)
});

// No manual reconnection needed - Socket.IO handles it
```

### **Reconnection Events**

```typescript
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000');

// CONNECTION LOST
socket.on('disconnect', (reason) => {
  console.log('Disconnected:', reason);
  
  if (reason === 'io server disconnect') {
    // Server disconnected us (kicked out)
    // Manual reconnect needed
    socket.connect();
  }
  // Other reasons auto-reconnect:
  // - 'transport close' (connection lost)
  // - 'transport error'
});

// RECONNECTION ATTEMPT
socket.on('reconnect_attempt', (attemptNumber) => {
  console.log(`Reconnecting... Attempt ${attemptNumber}`);
  // Show "Connecting..." UI
});

// RECONNECTION ERROR
socket.on('reconnect_error', (error) => {
  console.error('Reconnection failed:', error);
});

// RECONNECTION FAILED (all attempts exhausted)
socket.on('reconnect_failed', () => {
  console.error('Failed to reconnect after all attempts');
  // Show "Connection lost" message
  // Offer manual retry button
});

// SUCCESSFULLY RECONNECTED
socket.on('reconnect', (attemptNumber) => {
  console.log(`Reconnected after ${attemptNumber} attempts`);
  // Hide "Connecting..." UI
  // Resume state (re-join rooms, resync data)
});
```

### **UI Feedback (React Example)**

```typescript
import { useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';

type ConnectionState = 'connected' | 'connecting' | 'disconnected' | 'reconnecting';

export function useSocket(url: string) {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [connectionState, setConnectionState] = useState<ConnectionState>('disconnected');
  const [reconnectAttempts, setReconnectAttempts] = useState(0);

  useEffect(() => {
    const newSocket = io(url, {
      reconnection: true,
      reconnectionAttempts: 10,
      reconnectionDelay: 1000,
      reconnectionDelayMax: 5000,
    });

    // Connected
    newSocket.on('connect', () => {
      setConnectionState('connected');
      setReconnectAttempts(0);
    });

    // Disconnected
    newSocket.on('disconnect', (reason) => {
      setConnectionState('disconnected');
      console.log('Disconnected:', reason);
    });

    // Reconnecting
    newSocket.on('reconnect_attempt', (attemptNumber) => {
      setConnectionState('reconnecting');
      setReconnectAttempts(attemptNumber);
    });

    // Reconnected
    newSocket.on('reconnect', (attemptNumber) => {
      setConnectionState('connected');
      console.log(`Reconnected after ${attemptNumber} attempts`);
    });

    // Reconnection failed
    newSocket.on('reconnect_failed', () => {
      setConnectionState('disconnected');
      alert('Failed to reconnect. Please refresh the page.');
    });

    setSocket(newSocket);

    return () => {
      newSocket.close();
    };
  }, [url]);

  return { socket, connectionState, reconnectAttempts };
}

// Component usage
export function Chat() {
  const { socket, connectionState, reconnectAttempts } = useSocket('http://localhost:3000');

  return (
    <div>
      {/* Connection status banner */}
      {connectionState === 'disconnected' && (
        <div className="banner error">
          ❌ Connection lost. Attempting to reconnect...
        </div>
      )}
      
      {connectionState === 'reconnecting' && (
        <div className="banner warning">
          🔄 Reconnecting... (Attempt {reconnectAttempts})
        </div>
      )}
      
      {connectionState === 'connected' && (
        <div className="banner success">
          ✅ Connected
        </div>
      )}

      {/* Chat UI */}
      <ChatMessages socket={socket} />
    </div>
  );
}
```

### **Resume State After Reconnect**

```typescript
// After reconnect, re-join rooms and resync data

const socket = io('http://localhost:3000');
let currentRoomId: string | null = null;

// Join room initially
function joinRoom(roomId: string) {
  currentRoomId = roomId;
  socket.emit('join-room', roomId);
}

// On reconnect, re-join room
socket.on('reconnect', () => {
  console.log('Reconnected! Resuming state...');

  // Re-join room
  if (currentRoomId) {
    socket.emit('join-room', currentRoomId);
  }

  // Resync data
  socket.emit('get-latest-messages', { roomId: currentRoomId });
});

// Receive resynced data
socket.on('latest-messages', (messages) => {
  console.log('Received latest messages:', messages);
  // Update UI with latest messages
});
```

### **Connection State Recovery (Socket.IO v4.6+)**

```typescript
// SERVER: Enable connection state recovery
@WebSocketGateway({
  connectionStateRecovery: {
    // Max disconnection duration (ms)
    maxDisconnectionDuration: 2 * 60 * 1000, // 2 minutes
    // Skip middleware on recovery
    skipMiddlewares: true,
  },
})
export class ChatGateway {}

// CLIENT: Automatically recovers missed messages
const socket = io('http://localhost:3000', {
  // Auto-enabled if server supports it
});

// Detect recovery
socket.on('connect', () => {
  if (socket.recovered) {
    console.log('Connection recovered! Missed messages delivered.');
    // Missed events were automatically replayed
  } else {
    console.log('New connection.');
  }
});
```

### **Message Queue (Send When Reconnected)**

```typescript
import { io, Socket } from 'socket.io-client';

class SocketManager {
  private socket: Socket;
  private messageQueue: Array<{ event: string; data: any }> = [];
  private isConnected = false;

  constructor(url: string) {
    this.socket = io(url);

    this.socket.on('connect', () => {
      this.isConnected = true;
      this.flushQueue();
    });

    this.socket.on('disconnect', () => {
      this.isConnected = false;
    });
  }

  // Emit with queuing
  emit(event: string, data: any) {
    if (this.isConnected) {
      this.socket.emit(event, data);
    } else {
      // Queue message for later
      this.messageQueue.push({ event, data });
      console.log('Message queued (offline):', event);
    }
  }

  // Flush queued messages
  private flushQueue() {
    console.log(`Flushing ${this.messageQueue.length} queued messages...`);

    while (this.messageQueue.length > 0) {
      const { event, data } = this.messageQueue.shift()!;
      this.socket.emit(event, data);
    }
  }

  // Listen to events
  on(event: string, handler: (...args: any[]) => void) {
    this.socket.on(event, handler);
  }
}

// Usage
const socketManager = new SocketManager('http://localhost:3000');

// Messages are queued if offline
socketManager.emit('send-message', { message: 'Hello' });
// → If offline, queued
// → When reconnected, automatically sent
```

### **Exponential Backoff (Custom)**

```typescript
// Manual reconnection with exponential backoff

class ReconnectManager {
  private socket: Socket | null = null;
  private reconnectAttempts = 0;
  private maxAttempts = 10;
  private baseDelay = 1000; // 1 second
  private maxDelay = 30000; // 30 seconds

  connect(url: string) {
    this.socket = io(url, {
      reconnection: false, // Disable auto-reconnect
    });

    this.socket.on('disconnect', () => {
      this.scheduleReconnect();
    });

    this.socket.on('connect', () => {
      this.reconnectAttempts = 0; // Reset on success
    });
  }

  private scheduleReconnect() {
    if (this.reconnectAttempts >= this.maxAttempts) {
      console.error('Max reconnect attempts reached');
      return;
    }

    // Exponential backoff: 1s, 2s, 4s, 8s, 16s, ...
    const delay = Math.min(
      this.baseDelay * Math.pow(2, this.reconnectAttempts),
      this.maxDelay,
    );

    console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts + 1})`);

    setTimeout(() => {
      this.reconnectAttempts++;
      this.socket?.connect();
    }, delay);
  }
}
```

### **Network Detection**

```typescript
// Detect network changes and reconnect

const socket = io('http://localhost:3000');

// Browser online/offline events
window.addEventListener('online', () => {
  console.log('Network online - attempting to reconnect');
  if (!socket.connected) {
    socket.connect();
  }
});

window.addEventListener('offline', () => {
  console.log('Network offline');
  // Show offline banner
});

// Page visibility (tab active/inactive)
document.addEventListener('visibilitychange', () => {
  if (!document.hidden && !socket.connected) {
    console.log('Page visible - reconnecting');
    socket.connect();
  }
});
```

### **Heartbeat/Ping (Check Connection Health)**

```typescript
// Server-side heartbeat
@WebSocketGateway({
  pingTimeout: 60000,  // 60 seconds without ping = disconnect
  pingInterval: 25000, // Send ping every 25 seconds
})
export class ChatGateway {}

// Client-side: Socket.IO handles ping-pong automatically
// But you can monitor:

const socket = io('http://localhost:3000');

socket.on('ping', () => {
  console.log('Ping received from server');
});

socket.on('pong', (latency) => {
  console.log(`Pong! Latency: ${latency}ms`);
});
```

### **Production Reconnection Strategy**

```typescript
import { io, Socket } from 'socket.io-client';

export class ProductionSocket {
  private socket: Socket;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 10;
  private currentRoom: string | null = null;

  constructor(url: string, token: string) {
    this.socket = io(url, {
      auth: { token },
      
      // Reconnection config
      reconnection: true,
      reconnectionAttempts: this.maxReconnectAttempts,
      reconnectionDelay: 1000,
      reconnectionDelayMax: 5000,
      randomizationFactor: 0.5,
      
      // Timeout config
      timeout: 20000,
      
      // Transport config
      transports: ['websocket', 'polling'], // Fallback to polling
    });

    this.setupEventListeners();
  }

  private setupEventListeners() {
    // Connected
    this.socket.on('connect', () => {
      console.log('✅ Connected');
      this.reconnectAttempts = 0;
      this.resumeState();
    });

    // Disconnected
    this.socket.on('disconnect', (reason) => {
      console.log('❌ Disconnected:', reason);
      
      if (reason === 'io server disconnect') {
        // Server kicked us - don't auto-reconnect
        alert('Disconnected by server. Please log in again.');
      }
    });

    // Reconnecting
    this.socket.on('reconnect_attempt', (attemptNumber) => {
      this.reconnectAttempts = attemptNumber;
      console.log(`🔄 Reconnecting (${attemptNumber}/${this.maxReconnectAttempts})...`);
    });

    // Reconnected
    this.socket.on('reconnect', (attemptNumber) => {
      console.log(`✅ Reconnected after ${attemptNumber} attempts`);
    });

    // Failed
    this.socket.on('reconnect_failed', () => {
      console.error('❌ Reconnection failed');
      alert('Could not reconnect. Please refresh the page.');
    });

    // Connection error
    this.socket.on('connect_error', (error) => {
      console.error('Connection error:', error.message);
    });
  }

  // Resume state after reconnect
  private resumeState() {
    if (this.currentRoom) {
      this.socket.emit('join-room', this.currentRoom);
    }
  }

  // Join room
  joinRoom(roomId: string) {
    this.currentRoom = roomId;
    this.socket.emit('join-room', roomId);
  }

  // Emit with connection check
  emit(event: string, data: any) {
    if (this.socket.connected) {
      this.socket.emit(event, data);
    } else {
      console.warn('Not connected - message not sent');
    }
  }

  // Listen
  on(event: string, handler: (...args: any[]) => void) {
    this.socket.on(event, handler);
  }
}
```

**Interview Tip**: **Reconnection** handled **automatically** by Socket.IO with **exponential backoff**: config with `reconnection: true` (default), `reconnectionAttempts: Infinity` (tries forever), `reconnectionDelay: 1000` (start at 1s), `reconnectionDelayMax: 5000` (max 5s), `randomizationFactor: 0.5` (jitter to prevent thundering herd). **Events**: (1) `disconnect` (connection lost, auto-reconnects unless `reason === 'io server disconnect'`), (2) `reconnect_attempt` (retry in progress, show "Connecting..." UI), (3) `reconnect` (success, hide loading, **resume state**), (4) `reconnect_failed` (all attempts failed, show error, offer manual retry). **Resume state**: on `reconnect` event, **re-join rooms** (`socket.emit('join-room', currentRoom)`), **resync data** (`socket.emit('get-latest', {})`), restore UI state. **Connection State Recovery** (Socket.IO v4.6+): server config `connectionStateRecovery: { maxDisconnectionDuration: 120000 }` automatically replays missed events during disconnect (check `socket.recovered` on reconnect). **Message queue**: if disconnected, queue messages in array, flush when reconnected (send all queued). **UI feedback**: track connection state (`connected`, `connecting`, `disconnected`, `reconnecting`), show banner/spinner, display reconnect attempt number. **Production**: (1) set `reconnectionAttempts: 10` (not infinite), (2) `transports: ['websocket', 'polling']` for fallback, (3) monitor `connect_error` for auth failures, (4) use browser `online`/`offline` events to trigger reconnect, (5) **exponential backoff** with jitter prevents server overload. **Manual reconnection**: disable auto with `reconnection: false`, implement custom exponential backoff (delay = baseDelay * 2^attempts, cap at maxDelay). **Heartbeat**: Socket.IO built-in ping-pong (`pingTimeout: 60000`, `pingInterval: 25000`) detects dead connections. **Best practices**: show clear UI feedback, resume state on reconnect, limit max attempts, use jitter, handle server-initiated disconnect differently (don't auto-reconnect if kicked out).

</details>
