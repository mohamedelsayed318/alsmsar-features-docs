# Notifications API Documentation

This document describes how to integrate with the notification system in the frontend application.

## Table of Contents
- [Overview](#overview)
- [Socket.io Integration (Web Apps)](#socketio-integration-web-apps)
- [FCM Integration (Mobile Apps)](#fcm-integration-mobile-apps)
- [REST API Endpoints](#rest-api-endpoints)
  - [Get User Notifications](#1-get-user-notifications)
  - [Mark Notification as Read](#2-mark-notification-as-read)
  - [Get Unread Notifications Count](#3-get-unread-notifications-count)
  - [Mark All Notifications as Read](#4-mark-all-notifications-as-read)
- [Read/Unread Status](#readunread-status)
- [Notification Data Structure](#notification-data-structure)
- [Examples](#examples)

---

## Overview

The notification system supports two delivery methods:
- **Socket.io** - Real-time notifications for web applications
- **FCM (Firebase Cloud Messaging)** - Push notifications for mobile applications

All notifications are also stored in the database and can be fetched via REST API.

---

## Socket.io Integration (Web Apps)

### Connection Setup

Users automatically join their personal notification room when they connect to Socket.io. The room name format is: `user:${userId}`

### Listening for Notifications

Listen to the `"notification:new"` event to receive real-time notifications:

```javascript
import { io } from 'socket.io-client';

// Connect to Socket.io server
const socket = io('YOUR_SERVER_URL', {
  path: '/socket.io',
  auth: {
    token: 'YOUR_JWT_TOKEN' // Include JWT token for authentication
  }
});

// Listen for new notifications
socket.on('notification:new', (data) => {
  const notification = data.notification;
  
  console.log('New notification received:', notification);
  
  // Handle the notification
  // - Show toast notification
  // - Update notification badge
  // - Add to notification list
  // - Navigate to relevant page if needed
});
```

### Notification Event Data Structure

```typescript
interface NotificationEvent {
  notification: {
    id: string;
    title: string;
    message: string;
    type: 'info' | 'success' | 'warning' | 'error';
    category: 'general' | 'order' | 'payment' | 'system' | 'promotion' | 'message' | 'follow' | 'ad';
    isGlobal: boolean;
    visibleAfter: string; // ISO date string
    createdAt: string; // ISO date string
  };
}
```

### Example: React Hook for Notifications

```typescript
import { useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';

interface Notification {
  id: string;
  title: string;
  message: string;
  type: 'info' | 'success' | 'warning' | 'error';
  category: string;
  isGlobal: boolean;
  visibleAfter: string;
  createdAt: string;
}

export const useNotifications = (token: string) => {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [notifications, setNotifications] = useState<Notification[]>([]);

  useEffect(() => {
    if (!token) return;

    // Connect to Socket.io
    const newSocket = io('YOUR_SERVER_URL', {
      path: '/socket.io',
      auth: { token },
    });

    // Listen for notifications
    newSocket.on('notification:new', (data: { notification: Notification }) => {
      const notification = data.notification;
      
      // Add to notifications list
      setNotifications((prev) => [notification, ...prev]);
      
      // Show toast notification
      showToast(notification);
      
      // Update badge count
      updateBadgeCount();
    });

    // Listen for authentication confirmation
    newSocket.on('authenticated', (data: { userId: string }) => {
      console.log('Socket authenticated:', data.userId);
    });

    setSocket(newSocket);

    // Cleanup on unmount
    return () => {
      newSocket.close();
    };
  }, [token]);

  return { socket, notifications };
};
```

---

## FCM Integration (Mobile Apps)

### Setup

1. Install Firebase SDK in your mobile app
2. Get FCM token from the device
3. Send FCM token to the backend API

### Registering FCM Token

```typescript
// Send FCM token to backend
const registerFCMToken = async (fcmToken: string, authToken: string) => {
  const response = await fetch('YOUR_API_URL/api/fcm/token', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${authToken}`,
    },
    body: JSON.stringify({ fcmToken }),
  });
  
  return response.json();
};
```

### Handling FCM Notifications

#### React Native (Expo)

```typescript
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';

// Configure notification handler
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});

// Listen for notifications
Notifications.addNotificationReceivedListener((notification) => {
  const data = notification.request.content.data;
  
  // Handle notification based on type
  if (data.type === 'notification') {
    // Regular notification
    handleNotification(data);
  } else if (data.type === 'chat_message') {
    // Chat message notification
    handleChatMessage(data);
  } else if (data.type === 'new_ad') {
    // New ad notification
    handleNewAd(data);
  }
});

// Handle notification tap
Notifications.addNotificationResponseReceivedListener((response) => {
  const data = response.notification.request.content.data;
  
  // Navigate based on notification type
  if (data.type === 'new_ad' && data.adId) {
    navigation.navigate('AdDetails', { adId: data.adId });
  } else if (data.type === 'chat_message' && data.roomId) {
    navigation.navigate('Chat', { roomId: data.roomId });
  }
});
```

## REST API Endpoints

### Base URL
```
YOUR_API_URL/api/notifications
```

### Authentication
All endpoints require JWT authentication. Include the token in the Authorization header:
```
Authorization: Bearer YOUR_JWT_TOKEN
```

### Endpoints

#### 1. Get User Notifications

Get paginated list of notifications for the authenticated user.

**Endpoint:** `GET /api/notifications/user-notifications`

**Query Parameters:**
- `page` (optional, default: 1) - Page number
- `pageSize` (optional, default: 10) - Items per page

**Response:**
```json
{
  "success": true,
  "data": {
      "notifications": [
        {
          "id": "uuid",
          "title": "New Follower",
          "message": "John Doe started following you",
          "type": "info",
          "category": "follow",
          "isGlobal": false,
          "visibleAfter": "2024-01-01T00:00:00.000Z",
          "createdAt": "2024-01-01T00:00:00.000Z",
          "readAt": "2024-01-01T01:00:00.000Z" | null,
          "isRead": true | false
        }
      ],
      "unreadCount": 3,
    "currentPage": 1,
    "pageSize": 10,
    "totalNotifications": 25,
    "totalPages": 3,
    "hasNextPage": true,
    "hasPreviousPage": false,
    "count": 10
  }
}
```

**Example:**
```typescript
const fetchNotifications = async (page = 1, pageSize = 10) => {
  const response = await fetch(
    `YOUR_API_URL/api/notifications/user-notifications?page=${page}&pageSize=${pageSize}`,
    {
      headers: {
        'Authorization': `Bearer ${authToken}`,
      },
    }
  );
  
  return response.json();
};
```

#### 2. Mark Notification as Read

Mark a specific notification as read.

**Endpoint:** `PATCH /api/notifications/read/:id`

**Path Parameters:**
- `id` - Notification ID

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "userId": "uuid",
    "notificationId": "uuid",
    "readAt": "2024-01-01T01:00:00.000Z",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T01:00:00.000Z"
  }
}
```

**Example:**
```typescript
const markAsRead = async (notificationId: string) => {
  const response = await fetch(
    `YOUR_API_URL/api/notifications/read/${notificationId}`,
    {
      method: 'PATCH',
      headers: {
        'Authorization': `Bearer ${authToken}`,
      },
    }
  );
  
  return response.json();
};
```

#### 3. Get Unread Notifications Count

Get the total count of unread notifications for the authenticated user.

**Endpoint:** `GET /api/notifications/unread-count`

**Response:**
```json
{
  "success": true,
  "data": {
    "unreadCount": 5
  }
}
```

**Example:**
```typescript
const getUnreadCount = async () => {
  const response = await fetch(
    `YOUR_API_URL/api/notifications/unread-count`,
    {
      headers: {
        'Authorization': `Bearer ${authToken}`,
      },
    }
  );
  
  return response.json();
};
```

#### 4. Mark All Notifications as Read

Mark all notifications as read for the authenticated user.

**Endpoint:** `PATCH /api/notifications/read-all`

**Response:**
```json
{
  "success": true,
  "data": {
    "updated": 25
  }
}
```

**Example:**
```typescript
const markAllAsRead = async () => {
  const response = await fetch(
    `YOUR_API_URL/api/notifications/read-all`,
    {
      method: 'PATCH',
      headers: {
        'Authorization': `Bearer ${authToken}`,
      },
    }
  );
  
  return response.json();
};
```

---

## Read/Unread Status

The notification system tracks whether each notification has been read by the user.

### How It Works

- **Unread**: A notification is unread when `readAt` is `null` and `isRead` is `false`
- **Read**: A notification is read when `readAt` has a timestamp and `isRead` is `true`
- Notifications are automatically marked as unread when created
- Users can mark notifications as read individually or all at once

### Checking Read Status

Each notification object includes two fields for read status:

```typescript
{
  readAt: string | null,  // ISO date string or null
  isRead: boolean         // true if read, false if unread
}
```

**Recommended**: Use the `isRead` boolean for easier checks in your frontend code.

### Marking Notifications as Read

#### Single Notification
```typescript
// Mark one notification as read
await markAsRead(notificationId);
```

#### All Notifications
```typescript
// Mark all notifications as read
await markAllAsRead();
```

### Getting Unread Count

```typescript
// Get total unread count
const { unreadCount } = await getUnreadCount();
// Use for badge display: {unreadCount > 0 && <Badge>{unreadCount}</Badge>}
```

### Best Practices

1. **Auto-mark as read**: Mark notification as read when user clicks/interacts with it
2. **Badge display**: Use `unreadCount` to show notification badge
3. **Visual indicators**: Use `isRead` to style unread notifications differently
4. **Bulk actions**: Provide "Mark all as read" button for better UX
5. **Real-time updates**: Update unread count when new notifications arrive via Socket.io

---

## Notification Data Structure

### Notification Object

```typescript
interface Notification {
  id: string;                    // UUID
  title: string;                  // Notification title
  message: string;                // Notification message/content
  type: NotificationType;         // 'info' | 'success' | 'warning' | 'error'
  category: NotificationCategory; // See categories below
  isGlobal: boolean;              // Whether notification is global
  visibleAfter: string;           // ISO date - when notification becomes visible
  createdAt: string;              // ISO date - when notification was created
  readAt: string | null;          // ISO date - when notification was read (null if unread)
  isRead: boolean;                // Boolean flag - true if notification is read, false if unread
}
```

### Notification Types

```typescript
type NotificationType = 'info' | 'success' | 'warning' | 'error';
```

### Notification Categories

```typescript
type NotificationCategory = 
  | 'general'    // General notifications
  | 'order'      // Order-related notifications
  | 'payment'    // Payment-related notifications
  | 'system'     // System notifications
  | 'promotion'  // Promotional notifications
  | 'message'    // Chat message notifications
  | 'follow'     // Follow/follower notifications
  | 'ad';        // New ad notifications
```

---

## Examples

### Complete React Component Example

```typescript
import React, { useEffect, useState } from 'react';
import { io } from 'socket.io-client';
import { useAuth } from './auth';

interface Notification {
  id: string;
  title: string;
  message: string;
  type: string;
  category: string;
  isGlobal: boolean;
  visibleAfter: string;
  createdAt: string;
  readAt: string | null;
}

const NotificationCenter: React.FC = () => {
  const { token, user } = useAuth();
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);

  // Fetch notifications from API
  const fetchNotifications = async () => {
    setLoading(true);
    try {
      const response = await fetch(
        `YOUR_API_URL/api/notifications/user-notifications?page=${page}&pageSize=20`,
        {
          headers: {
            'Authorization': `Bearer ${token}`,
          },
        }
      );
      const data = await response.json();
      if (data.success) {
        setNotifications(data.data.notifications);
        setUnreadCount(
          data.data.notifications.filter((n: Notification) => !n.readAt).length
        );
      }
    } catch (error) {
      console.error('Error fetching notifications:', error);
    } finally {
      setLoading(false);
    }
  };

  // Mark notification as read
  const handleMarkAsRead = async (notificationId: string) => {
    try {
      const response = await fetch(
        `YOUR_API_URL/api/notifications/read/${notificationId}`,
        {
          method: 'PATCH',
          headers: {
            'Authorization': `Bearer ${token}`,
          },
        }
      );
      
      if (response.ok) {
        // Update local state
        setNotifications((prev) =>
          prev.map((n) =>
            n.id === notificationId
              ? { 
                  ...n, 
                  readAt: new Date().toISOString(),
                  isRead: true 
                }
              : n
          )
        );
        setUnreadCount((prev) => Math.max(0, prev - 1));
      }
    } catch (error) {
      console.error('Error marking notification as read:', error);
    }
  };

  // Socket.io connection
  useEffect(() => {
    if (!token || !user) return;

    const socket = io('YOUR_SERVER_URL', {
      path: '/socket.io',
      auth: { token },
    });

    socket.on('notification:new', (data: { notification: Notification }) => {
      const notification = data.notification;
      
      // Add to notifications list
      setNotifications((prev) => [notification, ...prev]);
      
      // Increment unread count (new notifications are unread by default)
      setUnreadCount((prev) => prev + 1);
      
      // Show toast notification
      showToast({
        title: notification.title,
        message: notification.message,
        type: notification.type,
      });
    });
    
    // Fetch unread count on mount
    const fetchUnreadCount = async () => {
      try {
        const response = await fetch(
          `YOUR_API_URL/api/notifications/unread-count`,
          {
            headers: {
              'Authorization': `Bearer ${token}`,
            },
          }
        );
        const data = await response.json();
        if (data.success) {
          setUnreadCount(data.data.unreadCount);
        }
      } catch (error) {
        console.error('Error fetching unread count:', error);
      }
    };
    
    fetchUnreadCount();

    socket.on('authenticated', () => {
      console.log('Socket authenticated');
    });

    return () => {
      socket.close();
    };
  }, [token, user]);

  // Fetch notifications on mount
  useEffect(() => {
    fetchNotifications();
  }, [page]);

  return (
    <div className="notification-center">
      <div className="notification-header">
        <h2>Notifications</h2>
        {unreadCount > 0 && (
          <span className="badge">{unreadCount}</span>
        )}
      </div>
      
      {loading ? (
        <div>Loading...</div>
      ) : (
        <div className="notification-list">
          {notifications.map((notification) => (
            <div
              key={notification.id}
              className={`notification-item ${
                !notification.isRead ? 'unread' : ''
              }`}
              onClick={() => handleMarkAsRead(notification.id)}
            >
              <div className="notification-icon">
                {getIconForType(notification.type)}
              </div>
              <div className="notification-content">
                <h3>{notification.title}</h3>
                <p>{notification.message}</p>
                <span className="notification-time">
                  {formatDate(notification.createdAt)}
                </span>
              </div>
            </div>
          ))}
        </div>
      )}
    </div>
  );
};

export default NotificationCenter;
```

### Handling Different Notification Categories

```typescript
const handleNotificationClick = (notification: Notification) => {
  // Mark as read if not already read
  if (!notification.isRead) {
    markAsRead(notification.id);
  }
  
  // Navigate based on category
  switch (notification.category) {
    case 'follow':
      // Navigate to user profile
      navigate(`/users/${getUserIdFromNotification(notification)}`);
      break;
      
    case 'ad':
      // Navigate to ad details
      const adId = getAdIdFromNotification(notification);
      navigate(`/ads/${adId}`);
      break;
      
    case 'message':
      // Navigate to chat room
      const roomId = getRoomIdFromNotification(notification);
      navigate(`/chat/${roomId}`);
      break;
      
    default:
      // Navigate to notifications page
      navigate('/notifications');
  }
};
```

### Mark All as Read

```typescript
const handleMarkAllAsRead = async () => {
  try {
    const response = await fetch(
      `YOUR_API_URL/api/notifications/read-all`,
      {
        method: 'PATCH',
        headers: {
          'Authorization': `Bearer ${authToken}`,
        },
      }
    );
    
    if (response.ok) {
      // Update all notifications to read
      setNotifications((prev) =>
        prev.map((n) => ({
          ...n,
          readAt: new Date().toISOString(),
          isRead: true,
        }))
      );
      setUnreadCount(0);
    }
  } catch (error) {
    console.error('Error marking all as read:', error);
  }
};
```

---

## Best Practices

1. **Always authenticate Socket.io connections** with JWT token
2. **Handle connection errors** and implement reconnection logic
3. **Show visual feedback** when notifications arrive (toast, badge, etc.)
4. **Mark notifications as read** when user interacts with them
5. **Filter notifications** by category if needed
6. **Implement pagination** for notification lists
7. **Cache notifications** locally to reduce API calls
8. **Handle offline scenarios** gracefully

---

## Error Handling

### Socket.io Connection Errors

```typescript
socket.on('connect_error', (error) => {
  console.error('Socket connection error:', error);
  // Implement retry logic
});

socket.on('disconnect', (reason) => {
  console.log('Socket disconnected:', reason);
  // Implement reconnection logic
});
```

### API Error Handling

```typescript
const fetchNotifications = async () => {
  try {
    const response = await fetch(/* ... */);
    
    if (!response.ok) {
      if (response.status === 401) {
        // Handle unauthorized - redirect to login
        redirectToLogin();
      } else {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
    }
    
    return await response.json();
  } catch (error) {
    console.error('Error:', error);
    // Show error message to user
  }
};
```

---

## Support

For issues or questions, please contact the backend team or refer to the API documentation.

