# Chat System Documentation for Flutter Frontend

## Table of Contents
1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Authentication](#authentication)
4. [REST API Endpoints](#rest-api-endpoints)
5. [Socket.IO Real-time Events](#socketio-real-time-events)
6. [Data Models](#data-models)
7. [Flutter Implementation Guide](#flutter-implementation-guide)
8. [User Presence System](#user-presence-system)
9. [Error Handling](#error-handling)
10. [Best Practices](#best-practices)

---

## Overview

This is a full-featured real-time chat system supporting:
- **Direct (1-on-1) conversations**
- **Group chats**
- **Real-time messaging** via Socket.IO
- **Message read receipts**
- **Typing indicators**
- **User presence** (online/offline/away status)
- **Message editing & deletion**
- **Reply to messages**
- **File/Image support** (via metadata)
- **Unread message counts**

---

## Architecture

The chat system uses a **hybrid approach**:
- **REST API**: For initial data loading, pagination, and non-time-critical operations
- **Socket.IO**: For real-time updates (new messages, typing, presence, etc.)

### Base URLs
```
REST API: https://your-domain.com/api
Socket.IO: wss://your-domain.com (path: /socket.io)
```

---

## Authentication

All API requests and Socket.IO connections require authentication using a **JWT Bearer Token**.

### Getting the Token
1. Login via REST API:
```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe"
    }
  }
}
```

### Using the Token

**REST API:**
```http
Authorization: Bearer YOUR_JWT_TOKEN
```

**Socket.IO:**
```dart
// Pass token in auth query or extraHeaders
IO.Socket socket = IO.io('https://your-domain.com', 
  IO.OptionBuilder()
    .setTransports(['websocket'])
    .setAuth({'token': 'YOUR_JWT_TOKEN'})
    .build()
);
```

---

## REST API Endpoints

### 1. Get User's Rooms (Conversations List)

**Endpoint:** `GET /api/messages/rooms` or `GET /api/rooms/rooms`

**Headers:**
```
Authorization: Bearer YOUR_JWT_TOKEN
```

**Response:**
```json
{
  "success": true,
  "data": {
    "rooms": [
      {
        "id": "room-uuid",
        "type": "direct",
        "name": null,
        "description": null,
        "avatar": null,
        "createdBy": "user-uuid",
        "lastMessageId": "message-uuid",
        "lastMessageAt": "2026-01-07T10:30:00Z",
        "createdAt": "2026-01-01T00:00:00Z",
        "updatedAt": "2026-01-07T10:30:00Z",
        "unreadCount": 5,
        "participants": [
          {
            "userId": "user-uuid",
            "firstName": "John",
            "lastName": "Doe",
            "avatar": "https://...",
            "role": "member"
          }
        ],
        "lastMessage": {
          "id": "message-uuid",
          "content": "Hello!",
          "type": "text",
          "senderId": "user-uuid",
          "createdAt": "2026-01-07T10:30:00Z"
        }
      }
    ]
  }
}
```

---

### 2. Get or Create Direct Room

**Endpoint:** `POST /api/messages/rooms/direct` or `POST /api/rooms/rooms/direct`

**Headers:**
```
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

**Body:**
```json
{
  "otherUserId": "target-user-uuid"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "room": {
      "id": "room-uuid",
      "type": "direct",
      "createdBy": "user-uuid",
      "createdAt": "2026-01-07T10:00:00Z"
    },
    "participants": [
      {
        "userId": "user-uuid-1",
        "firstName": "John",
        "lastName": "Doe"
      },
      {
        "userId": "user-uuid-2",
        "firstName": "Jane",
        "lastName": "Smith"
      }
    ],
    "unreadCount": 0
  }
}
```

---

### 3. Get Room Details

**Endpoint:** `GET /api/messages/rooms/:roomId` or `GET /api/rooms/rooms/:roomId`

**Headers:**
```
Authorization: Bearer YOUR_JWT_TOKEN
```

**Response:**
```json
{
  "success": true,
  "data": {
    "room": {
      "id": "room-uuid",
      "type": "group",
      "name": "Project Team",
      "description": "Team discussion",
      "avatar": "https://...",
      "createdBy": "user-uuid",
      "createdAt": "2026-01-01T00:00:00Z"
    },
    "participants": [
      {
        "userId": "user-uuid",
        "firstName": "John",
        "lastName": "Doe",
        "role": "admin"
      }
    ],
    "unreadCount": 3
  }
}
```

---

### 4. Get Room Messages (with Pagination)

**Endpoint:** `GET /api/messages/rooms/:roomId/messages` or `GET /api/rooms/rooms/:roomId/messages`

**Headers:**
```
Authorization: Bearer YOUR_JWT_TOKEN
```

**Query Parameters:**
- `page` (optional, default: 1)
- `pageSize` (optional, default: 50)

**Example:** `GET /api/messages/rooms/room-uuid/messages?page=1&pageSize=50`

**Response:**
```json
{
  "success": true,
  "data": {
    "messages": [
      {
        "id": "message-uuid",
        "roomId": "room-uuid",
        "senderId": "user-uuid",
        "type": "text",
        "content": "Hello everyone!",
        "metadata": null,
        "replyToId": null,
        "isEdited": false,
        "isDeleted": false,
        "deletedAt": null,
        "createdAt": "2026-01-07T10:30:00Z",
        "updatedAt": "2026-01-07T10:30:00Z",
        "sender": {
          "id": "user-uuid",
          "firstName": "John",
          "lastName": "Doe",
          "avatar": "https://..."
        }
      }
    ],
    "page": 1,
    "pageSize": 50,
    "roomId": "room-uuid"
  }
}
```

**Note:** Messages are returned in **reverse chronological order** (newest first).

---

### 5. Send Message (REST - Not Recommended)

**Endpoint:** `POST /api/messages/send` or `POST /api/rooms/send`

**Headers:**
```
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

**Body:**
```json
{
  "roomId": "room-uuid",
  "content": "Hello!",
  "type": "text",
  "metadata": null,
  "replyToId": null
}
```

**Note:** This endpoint exists for backward compatibility but **Socket.IO is recommended** for sending messages to ensure real-time delivery.

---

### 6. Get User Presence

**Endpoint:** `GET /api/presence/:userId`

**Headers:**
```
Authorization: Bearer YOUR_JWT_TOKEN
```

**Response:**
```json
{
  "success": true,
  "data": {
    "presence": {
      "userId": "user-uuid",
      "status": "online",
      "lastSeenAt": "2026-01-07T10:45:00Z",
      "updatedAt": "2026-01-07T10:45:00Z"
    }
  }
}
```

**Status values:** `"online"`, `"offline"`, `"away"`

---

## Socket.IO Real-time Events

### Connection Setup

```dart
import 'package:socket_io_client/socket_io_client.dart' as IO;

IO.Socket socket = IO.io('https://your-domain.com', 
  IO.OptionBuilder()
    .setTransports(['websocket'])
    .setAuth({'token': 'YOUR_JWT_TOKEN'})
    .enableAutoConnect()
    .enableReconnection()
    .build()
);

socket.connect();

// Listen for successful connection
socket.on('connect', (_) {
  print('Connected to socket server');
});

// Listen for authentication confirmation
socket.on('authenticated', (data) {
  print('Authenticated as user: ${data['userId']}');
});

socket.on('error', (error) {
  print('Socket error: $error');
});

socket.on('disconnect', (_) {
  print('Disconnected from socket server');
});
```

---

### Events to EMIT (Client → Server)

#### 1. Join a Room

**Event:** `room:join`

**Payload:**
```dart
socket.emit('room:join', {
  'roomId': 'room-uuid'
});
```

**Response Event:** `room:joined`
```dart
socket.on('room:joined', (data) {
  // data = { "room": { ... } }
  print('Joined room: ${data['room']['id']}');
});
```

---

#### 2. Leave a Room

**Event:** `room:leave`

**Payload:**
```dart
socket.emit('room:leave', {
  'roomId': 'room-uuid'
});
```

**Response Event:** `room:left`
```dart
socket.on('room:left', (data) {
  // data = { "roomId": "room-uuid" }
  print('Left room: ${data['roomId']}');
});
```

---

#### 3. Send a Message

**Event:** `message:send`

**Payload:**
```dart
socket.emit('message:send', {
  'roomId': 'room-uuid',
  'content': 'Hello World!',
  'type': 'text', // Optional: 'text', 'image', 'file', 'system'
  'metadata': null, // Optional: JSON object for file info, image dimensions, etc.
  'replyToId': null // Optional: UUID of message being replied to
});
```

**Broadcast Event:** `message:new` (received by all room participants)
```dart
socket.on('message:new', (data) {
  // data = { "message": { ... } }
  Message newMessage = Message.fromJson(data['message']);
  // Add message to chat UI
});
```

---

#### 4. Edit a Message

**Event:** `message:edit`

**Payload:**
```dart
socket.emit('message:edit', {
  'messageId': 'message-uuid',
  'content': 'Updated content'
});
```

**Broadcast Event:** `message:edited`
```dart
socket.on('message:edited', (data) {
  // data = { "message": { ... } }
  // Update the message in UI
});
```

---

#### 5. Delete a Message

**Event:** `message:delete`

**Payload:**
```dart
socket.emit('message:delete', {
  'messageId': 'message-uuid'
});
```

**Broadcast Event:** `message:deleted`
```dart
socket.on('message:deleted', (data) {
  // data = { "message": { "id": "...", "isDeleted": true, ... } }
  // Mark message as deleted in UI
});
```

---

#### 6. Mark Message(s) as Read

**Event:** `message:read`

**Payload (single message):**
```dart
socket.emit('message:read', {
  'messageId': 'message-uuid'
});
```

**Payload (all messages in room):**
```dart
socket.emit('message:read', {
  'roomId': 'room-uuid'
});
```

**Broadcast Event:** `message:read`
```dart
socket.on('message:read', (data) {
  // Single message: { "messageId": "...", "userId": "..." }
  // Entire room: { "roomId": "...", "userId": "..." }
  // Update read receipts in UI
});
```

---

#### 7. Typing Indicators

**Event:** `typing:start`

**Payload:**
```dart
socket.emit('typing:start', {
  'roomId': 'room-uuid'
});
```

**Event:** `typing:stop`

**Payload:**
```dart
socket.emit('typing:stop', {
  'roomId': 'room-uuid'
});
```

**Received Events:**
```dart
socket.on('typing:start', (data) {
  // data = { "roomId": "...", "userId": "..." }
  // Show "User is typing..." indicator
});

socket.on('typing:stop', (data) {
  // data = { "roomId": "...", "userId": "..." }
  // Hide typing indicator for this user
});
```

**Note:** Typing automatically stops after 3 seconds if no `typing:stop` is sent.

---

#### 8. Create a Room (Group Chat)

**Event:** `room:create`

**Payload:**
```dart
socket.emit('room:create', {
  'type': 'group', // 'direct' or 'group'
  'name': 'Project Team', // Required for group chats
  'description': 'Team discussion', // Optional
  'avatar': 'https://...', // Optional
  'participantIds': ['user-uuid-1', 'user-uuid-2'] // Optional
});
```

**Response Event:** `room:created`
```dart
socket.on('room:created', (data) {
  // data = { "room": { ... } }
  // Add new room to UI
});
```

---

#### 9. Update Room (Admin Only)

**Event:** `room:update`

**Payload:**
```dart
socket.emit('room:update', {
  'roomId': 'room-uuid',
  'name': 'Updated Name', // Optional
  'description': 'Updated description', // Optional
  'avatar': 'https://...' // Optional
});
```

**Broadcast Event:** `room:updated`
```dart
socket.on('room:updated', (data) {
  // data = { "room": { ... } }
  // Update room info in UI
});
```

---

#### 10. Add Participant (Admin Only)

**Event:** `room:add_participant`

**Payload:**
```dart
socket.emit('room:add_participant', {
  'roomId': 'room-uuid',
  'userId': 'new-user-uuid'
});
```

**Broadcast Event:** `room:participant_added`
```dart
socket.on('room:participant_added', (data) {
  // data = { "roomId": "...", "userId": "..." }
  // Update participants list
});
```

---

#### 11. Remove Participant (Admin Only or Self)

**Event:** `room:remove_participant`

**Payload:**
```dart
socket.emit('room:remove_participant', {
  'roomId': 'room-uuid',
  'userId': 'user-uuid-to-remove'
});
```

**Broadcast Event:** `room:participant_removed`
```dart
socket.on('room:participant_removed', (data) {
  // data = { "roomId": "...", "userId": "..." }
  // Update participants list
});
```

---

### Events to LISTEN (Server → Client)

#### Presence Updates

**Event:** `presence:update`

```dart
socket.on('presence:update', (data) {
  // data = {
  //   "userId": "user-uuid",
  //   "status": "online", // 'online', 'offline', 'away'
  //   "lastSeenAt": "2026-01-07T10:45:00Z"
  // }
  
  // Update user's online status in UI
});
```

**Note:** Presence updates are broadcast to:
- All rooms the user participates in
- The user's personal room (`user:userId`)

To receive presence updates for a room, join the room first using `room:join`.

---

## Data Models

### Room

```dart
class Room {
  final String id;
  final String type; // 'direct' or 'group'
  final String? name; // null for direct chats
  final String? description;
  final String? avatar;
  final String createdBy;
  final String? lastMessageId;
  final DateTime? lastMessageAt;
  final DateTime createdAt;
  final DateTime updatedAt;
  
  // Additional fields from API responses
  final int? unreadCount;
  final List<Participant>? participants;
  final Message? lastMessage;
}
```

### Message

```dart
class Message {
  final String id;
  final String roomId;
  final String senderId;
  final String type; // 'text', 'image', 'file', 'system'
  final String content;
  final Map<String, dynamic>? metadata; // For file info, dimensions, etc.
  final String? replyToId;
  final bool isEdited;
  final bool isDeleted;
  final DateTime? deletedAt;
  final DateTime createdAt;
  final DateTime updatedAt;
  
  // From API join
  final User? sender;
  final Message? replyTo;
}
```

### Participant

```dart
class Participant {
  final String id;
  final String roomId;
  final String userId;
  final String role; // 'admin' or 'member'
  final DateTime joinedAt;
  final DateTime? leftAt;
  final DateTime? lastReadAt;
  final bool isMuted;
  final bool isPinned;
  final DateTime createdAt;
  final DateTime updatedAt;
  
  // From API join
  final User? user;
}
```

### UserPresence

```dart
class UserPresence {
  final String userId;
  final String status; // 'online', 'offline', 'away'
  final DateTime? lastSeenAt;
  final DateTime? updatedAt;
}
```

### Message Metadata Examples

**For images:**
```json
{
  "url": "https://cdn.example.com/image.jpg",
  "width": 1920,
  "height": 1080,
  "size": 245678,
  "mimeType": "image/jpeg"
}
```

**For files:**
```json
{
  "url": "https://cdn.example.com/document.pdf",
  "fileName": "document.pdf",
  "size": 1024567,
  "mimeType": "application/pdf"
}
```

---

## Flutter Implementation Guide

### 1. Dependencies

Add to `pubspec.yaml`:
```yaml
dependencies:
  socket_io_client: ^2.0.3
  http: ^1.1.0
  provider: ^6.1.1 # Or your state management solution
```

---

### 2. Socket Service (Singleton)

```dart
import 'package:socket_io_client/socket_io_client.dart' as IO;

class SocketService {
  static final SocketService _instance = SocketService._internal();
  factory SocketService() => _instance;
  SocketService._internal();

  IO.Socket? _socket;
  String? _token;

  void initialize(String token) {
    _token = token;
    
    _socket = IO.io('https://your-domain.com', 
      IO.OptionBuilder()
        .setTransports(['websocket'])
        .setAuth({'token': token})
        .enableAutoConnect()
        .enableReconnection()
        .setReconnectionDelay(1000)
        .setReconnectionDelayMax(5000)
        .setReconnectionAttempts(5)
        .build()
    );

    _setupListeners();
    _socket!.connect();
  }

  void _setupListeners() {
    _socket!.on('connect', (_) {
      print('✅ Connected to socket server');
    });

    _socket!.on('authenticated', (data) {
      print('✅ Authenticated as: ${data['userId']}');
    });

    _socket!.on('disconnect', (_) {
      print('❌ Disconnected from socket server');
    });

    _socket!.on('error', (error) {
      print('❌ Socket error: $error');
    });
  }

  void joinRoom(String roomId) {
    _socket?.emit('room:join', {'roomId': roomId});
  }

  void leaveRoom(String roomId) {
    _socket?.emit('room:leave', {'roomId': roomId});
  }

  void sendMessage({
    required String roomId,
    required String content,
    String type = 'text',
    Map<String, dynamic>? metadata,
    String? replyToId,
  }) {
    _socket?.emit('message:send', {
      'roomId': roomId,
      'content': content,
      'type': type,
      'metadata': metadata,
      'replyToId': replyToId,
    });
  }

  void sendTyping(String roomId, bool isTyping) {
    _socket?.emit(isTyping ? 'typing:start' : 'typing:stop', {
      'roomId': roomId,
    });
  }

  void listenToMessages(Function(Map<String, dynamic>) callback) {
    _socket?.on('message:new', (data) {
      callback(data);
    });
  }

  void listenToTyping(
    Function(String roomId, String userId) onStart,
    Function(String roomId, String userId) onStop,
  ) {
    _socket?.on('typing:start', (data) {
      onStart(data['roomId'], data['userId']);
    });
    
    _socket?.on('typing:stop', (data) {
      onStop(data['roomId'], data['userId']);
    });
  }

  void listenToPresence(Function(Map<String, dynamic>) callback) {
    _socket?.on('presence:update', (data) {
      callback(data);
    });
  }

  void disconnect() {
    _socket?.disconnect();
    _socket?.dispose();
  }

  IO.Socket? get socket => _socket;
}
```

---

### 3. API Service

```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

class ChatApiService {
  final String baseUrl = 'https://your-domain.com/api';
  String? _token;

  void setToken(String token) {
    _token = token;
  }

  Map<String, String> get _headers => {
    'Authorization': 'Bearer $_token',
    'Content-Type': 'application/json',
  };

  Future<List<Room>> getUserRooms() async {
    final response = await http.get(
      Uri.parse('$baseUrl/messages/rooms'),
      headers: _headers,
    );

    if (response.statusCode == 200) {
      final data = json.decode(response.body);
      return (data['data']['rooms'] as List)
          .map((room) => Room.fromJson(room))
          .toList();
    }
    throw Exception('Failed to load rooms');
  }

  Future<Room> getOrCreateDirectRoom(String otherUserId) async {
    final response = await http.post(
      Uri.parse('$baseUrl/messages/rooms/direct'),
      headers: _headers,
      body: json.encode({'otherUserId': otherUserId}),
    );

    if (response.statusCode == 200) {
      final data = json.decode(response.body);
      return Room.fromJson(data['data']['room']);
    }
    throw Exception('Failed to create room');
  }

  Future<List<Message>> getRoomMessages({
    required String roomId,
    int page = 1,
    int pageSize = 50,
  }) async {
    final response = await http.get(
      Uri.parse('$baseUrl/messages/rooms/$roomId/messages?page=$page&pageSize=$pageSize'),
      headers: _headers,
    );

    if (response.statusCode == 200) {
      final data = json.decode(response.body);
      return (data['data']['messages'] as List)
          .map((msg) => Message.fromJson(msg))
          .toList();
    }
    throw Exception('Failed to load messages');
  }

  Future<UserPresence> getUserPresence(String userId) async {
    final response = await http.get(
      Uri.parse('$baseUrl/presence/$userId'),
      headers: _headers,
    );

    if (response.statusCode == 200) {
      final data = json.decode(response.body);
      return UserPresence.fromJson(data['data']['presence']);
    }
    throw Exception('Failed to load presence');
  }
}
```

---

### 4. Chat Screen Example

```dart
class ChatScreen extends StatefulWidget {
  final String roomId;
  
  const ChatScreen({required this.roomId});

  @override
  _ChatScreenState createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final TextEditingController _messageController = TextEditingController();
  final SocketService _socketService = SocketService();
  final ChatApiService _apiService = ChatApiService();
  
  List<Message> _messages = [];
  Set<String> _typingUsers = {};

  @override
  void initState() {
    super.initState();
    _initializeChat();
  }

  Future<void> _initializeChat() async {
    // Join the room
    _socketService.joinRoom(widget.roomId);

    // Load initial messages
    final messages = await _apiService.getRoomMessages(roomId: widget.roomId);
    setState(() {
      _messages = messages.reversed.toList(); // Reverse for chronological order
    });

    // Listen for new messages
    _socketService.listenToMessages((data) {
      final message = Message.fromJson(data['message']);
      if (message.roomId == widget.roomId) {
        setState(() {
          _messages.add(message);
        });
      }
    });

    // Listen for typing
    _socketService.listenToTyping(
      (roomId, userId) {
        if (roomId == widget.roomId) {
          setState(() => _typingUsers.add(userId));
        }
      },
      (roomId, userId) {
        if (roomId == widget.roomId) {
          setState(() => _typingUsers.remove(userId));
        }
      },
    );
  }

  void _sendMessage() {
    if (_messageController.text.trim().isEmpty) return;

    _socketService.sendMessage(
      roomId: widget.roomId,
      content: _messageController.text.trim(),
    );

    _messageController.clear();
  }

  void _onTextChanged(String text) {
    _socketService.sendTyping(widget.roomId, text.isNotEmpty);
  }

  @override
  void dispose() {
    _socketService.leaveRoom(widget.roomId);
    _messageController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Chat Room'),
      ),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: _messages.length,
              itemBuilder: (context, index) {
                final message = _messages[index];
                return MessageBubble(message: message);
              },
            ),
          ),
          if (_typingUsers.isNotEmpty)
            Padding(
              padding: EdgeInsets.all(8),
              child: Text('Someone is typing...'),
            ),
          _buildMessageInput(),
        ],
      ),
    );
  }

  Widget _buildMessageInput() {
    return Container(
      padding: EdgeInsets.all(8),
      child: Row(
        children: [
          Expanded(
            child: TextField(
              controller: _messageController,
              onChanged: _onTextChanged,
              decoration: InputDecoration(
                hintText: 'Type a message...',
                border: OutlineInputBorder(),
              ),
            ),
          ),
          SizedBox(width: 8),
          IconButton(
            icon: Icon(Icons.send),
            onPressed: _sendMessage,
          ),
        ],
      ),
    );
  }
}
```

---

## User Presence System

### How Presence Works

1. **Automatic Updates**: When a user connects/disconnects via Socket.IO, their presence is automatically updated to `online` or `offline`
2. **Real-time Broadcasts**: Presence changes are broadcast to all rooms the user participates in
3. **Database Storage**: Presence is persisted in the database with `lastSeenAt` timestamp

### Implementation in Flutter

```dart
class UserPresenceManager {
  final ChatApiService _apiService = ChatApiService();
  final SocketService _socketService = SocketService();
  
  final Map<String, UserPresence> _presenceCache = {};
  final StreamController<Map<String, UserPresence>> _presenceStream = 
      StreamController.broadcast();

  Stream<Map<String, UserPresence>> get presenceStream => _presenceStream.stream;

  void initialize() {
    // Listen for presence updates
    _socketService.listenToPresence((data) {
      final presence = UserPresence.fromJson(data);
      _presenceCache[presence.userId] = presence;
      _presenceStream.add(_presenceCache);
    });
  }

  Future<UserPresence> getPresence(String userId) async {
    // Check cache first
    if (_presenceCache.containsKey(userId)) {
      return _presenceCache[userId]!;
    }

    // Fetch from API
    final presence = await _apiService.getUserPresence(userId);
    _presenceCache[userId] = presence;
    return presence;
  }

  String getStatusText(UserPresence presence) {
    switch (presence.status) {
      case 'online':
        return 'Online';
      case 'away':
        return 'Away';
      case 'offline':
        if (presence.lastSeenAt != null) {
          return 'Last seen ${_formatLastSeen(presence.lastSeenAt!)}';
        }
        return 'Offline';
      default:
        return 'Unknown';
    }
  }

  String _formatLastSeen(DateTime lastSeen) {
    final now = DateTime.now();
    final difference = now.difference(lastSeen);

    if (difference.inMinutes < 1) {
      return 'just now';
    } else if (difference.inHours < 1) {
      return '${difference.inMinutes}m ago';
    } else if (difference.inDays < 1) {
      return '${difference.inHours}h ago';
    } else if (difference.inDays < 7) {
      return '${difference.inDays}d ago';
    } else {
      return 'on ${lastSeen.day}/${lastSeen.month}/${lastSeen.year}';
    }
  }

  void dispose() {
    _presenceStream.close();
  }
}
```

---

## Error Handling

### Socket.IO Errors

The server emits an `error` event when something goes wrong:

```dart
_socket.on('error', (data) {
  // data = { "message": "Error description" }
  showErrorDialog(data['message']);
});
```

Common errors:
- `"You are not a participant of this room"` - User tried to access a room they're not in
- `"You can only edit your own messages"` - User tried to edit someone else's message
- `"You can only delete your own messages"` - User tried to delete someone else's message
- `"Only admins can update rooms"` - Non-admin tried to update room settings
- `"Message not found"` - Referenced message doesn't exist

### REST API Errors

```dart
try {
  final rooms = await apiService.getUserRooms();
} catch (e) {
  if (e is http.ClientException) {
    // Network error
    showError('Network error. Please check your connection.');
  } else {
    showError('Failed to load rooms: ${e.toString()}');
  }
}
```

---

## Best Practices

### 1. Connection Management

```dart
// Connect when user logs in
void onLogin(String token) {
  SocketService().initialize(token);
  ChatApiService().setToken(token);
}

// Disconnect when user logs out
void onLogout() {
  SocketService().disconnect();
}

// Handle app lifecycle
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state == AppLifecycleState.paused) {
      // App is in background - optionally disconnect
      SocketService().disconnect();
    } else if (state == AppLifecycleState.resumed) {
      // App is back in foreground - reconnect
      final token = getStoredToken();
      if (token != null) {
        SocketService().initialize(token);
      }
    }
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }
}
```

---

### 2. Optimistic UI Updates

```dart
void sendMessage(String content) {
  // 1. Add message to UI immediately (optimistic)
  final optimisticMessage = Message(
    id: 'temp-${DateTime.now().millisecondsSinceEpoch}',
    content: content,
    senderId: currentUserId,
    roomId: roomId,
    type: 'text',
    isEdited: false,
    isDeleted: false,
    createdAt: DateTime.now(),
    updatedAt: DateTime.now(),
    status: 'sending', // Custom field
  );

  setState(() {
    _messages.add(optimisticMessage);
  });

  // 2. Send via socket
  _socketService.sendMessage(roomId: roomId, content: content);

  // 3. Replace with real message when received via message:new event
  _socketService.socket?.once('message:new', (data) {
    final realMessage = Message.fromJson(data['message']);
    setState(() {
      _messages.removeWhere((m) => m.id == optimisticMessage.id);
      _messages.add(realMessage);
    });
  });
}
```

---

### 3. Pagination

```dart
class ChatMessagesController {
  final String roomId;
  final ChatApiService _apiService = ChatApiService();
  
  List<Message> messages = [];
  int currentPage = 1;
  bool hasMore = true;
  bool isLoading = false;

  Future<void> loadMore() async {
    if (isLoading || !hasMore) return;

    isLoading = true;
    final newMessages = await _apiService.getRoomMessages(
      roomId: roomId,
      page: currentPage,
      pageSize: 50,
    );

    if (newMessages.isEmpty) {
      hasMore = false;
    } else {
      messages.insertAll(0, newMessages.reversed); // Add to beginning
      currentPage++;
    }

    isLoading = false;
  }
}
```

---

### 4. Typing Indicator Debouncing

```dart
import 'dart:async';

class TypingDebouncer {
  Timer? _timer;
  final Duration delay;
  final Function() onTypingStart;
  final Function() onTypingStop;

  bool _isTyping = false;

  TypingDebouncer({
    required this.delay,
    required this.onTypingStart,
    required this.onTypingStop,
  });

  void onTextChanged(String text) {
    if (text.isEmpty) {
      _stopTyping();
      return;
    }

    if (!_isTyping) {
      _isTyping = true;
      onTypingStart();
    }

    _timer?.cancel();
    _timer = Timer(delay, _stopTyping);
  }

  void _stopTyping() {
    if (_isTyping) {
      _isTyping = false;
      onTypingStop();
    }
  }

  void dispose() {
    _timer?.cancel();
  }
}

// Usage:
final _typingDebouncer = TypingDebouncer(
  delay: Duration(milliseconds: 500),
  onTypingStart: () => _socketService.sendTyping(roomId, true),
  onTypingStop: () => _socketService.sendTyping(roomId, false),
);

TextField(
  onChanged: _typingDebouncer.onTextChanged,
  // ...
)
```

---

### 5. Unread Count Management

```dart
class RoomListItem extends StatelessWidget {
  final Room room;

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text(room.name ?? 'Direct Chat'),
      subtitle: Text(room.lastMessage?.content ?? 'No messages yet'),
      trailing: room.unreadCount > 0
          ? Container(
              padding: EdgeInsets.all(6),
              decoration: BoxDecoration(
                color: Colors.red,
                shape: BoxShape.circle,
              ),
              child: Text(
                '${room.unreadCount}',
                style: TextStyle(color: Colors.white, fontSize: 12),
              ),
            )
          : null,
      onTap: () {
        // Navigate to chat screen
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (_) => ChatScreen(roomId: room.id),
          ),
        );
        
        // Mark as read when entering room
        SocketService().socket?.emit('message:read', {
          'roomId': room.id,
        });
      },
    );
  }
}
```

---

## Summary

### Quick Start Checklist

1. ✅ Add Socket.IO client to Flutter project
2. ✅ Implement authentication and get JWT token
3. ✅ Initialize Socket.IO connection with token
4. ✅ Fetch initial room list via REST API
5. ✅ Join rooms via Socket.IO
6. ✅ Load message history via REST API
7. ✅ Listen for real-time events (messages, typing, presence)
8. ✅ Send messages via Socket.IO
9. ✅ Implement typing indicators
10. ✅ Handle presence updates
11. ✅ Implement pagination for older messages
12. ✅ Handle errors and disconnections gracefully

### Key Takeaways

- **Use REST API for**: Initial data loading, pagination, historical data
- **Use Socket.IO for**: Real-time messaging, typing indicators, presence updates
- **Always authenticate**: Include JWT token in all requests
- **Join rooms**: Before sending/receiving messages in a room
- **Handle errors**: Both from REST API and Socket.IO events
- **Optimize**: Use optimistic UI updates and debouncing for better UX
- **Clean up**: Leave rooms and disconnect socket when appropriate

---

## Support

For issues or questions, please contact the backend team or open an issue in the project repository.

**Version:** 1.0  
**Last Updated:** January 7, 2026

