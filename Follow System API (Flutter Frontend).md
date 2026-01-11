# Follow System API (Flutter Frontend)

This document describes the **Follow system** endpoints to be used by the Flutter frontend.

## Base

- **Base URL**: `http(s)://<YOUR_DOMAIN>/api`
- **Auth**: required for all endpoints in this doc
- **Auth header**:

```http
Authorization: Bearer <token>
```

## Response Wrapper (All Endpoints)

Every endpoint responds using:

```json
{ "success": true/false, "data": ..., "message": "..." }
```

## Endpoints

### 1) Follow a user

- **POST** `/follows`
- **Auth**: required
- **Body**:

```json
{ "followingId": "USER_UUID" }
```

- **Success (201)** (`data` is the created follow row):

```json
{
  "success": true,
  "data": {
    "id": "FOLLOW_UUID",
    "followerId": "MY_UUID",
    "followingId": "USER_UUID",
    "createdAt": "2026-01-07T10:00:00.000Z",
    "updatedAt": "2026-01-07T10:00:00.000Z"
  },
  "message": null
}
```

### 2) Unfollow a user

- **DELETE** `/follows`
- **Auth**: required
- **Body**:

```json
{ "followingId": "USER_UUID" }
```

- **Success (200)** (`data` is the deleted follow row):

```json
{
  "success": true,
  "data": {
    "id": "FOLLOW_UUID",
    "followerId": "MY_UUID",
    "followingId": "USER_UUID",
    "createdAt": "2026-01-07T10:00:00.000Z",
    "updatedAt": "2026-01-07T10:00:00.000Z"
  },
  "message": null
}
```

### 3) Get my followers

- **GET** `/follows/followers/me`
- **Auth**: required
- **Success (200)** (`data` is a list of follow rows including `follower` user details):

```json
{
  "success": true,
  "data": [
    {
      "id": "FOLLOW_UUID",
      "followerId": "SOMEONE_UUID",
      "followingId": "MY_UUID",
      "createdAt": "2026-01-07T10:00:00.000Z",
      "updatedAt": "2026-01-07T10:00:00.000Z",
      "follower": {
        "id": "SOMEONE_UUID",
        "firstName": "A",
        "lastName": "B",
        "email": "a@b.com",
        "avatar": null,
        "isSeller": false,
        "createdAt": "2026-01-07T10:00:00.000Z"
      }
    }
  ],
  "message": null
}
```

### 4) Get who I’m following

- **GET** `/follows/following/me`
- **Auth**: required
- **Success (200)** (`data` is a list of follow rows including `following` user details):

```json
{
  "success": true,
  "data": [
    {
      "id": "FOLLOW_UUID",
      "followerId": "MY_UUID",
      "followingId": "OTHER_UUID",
      "createdAt": "2026-01-07T10:00:00.000Z",
      "updatedAt": "2026-01-07T10:00:00.000Z",
      "following": {
        "id": "OTHER_UUID",
        "firstName": "C",
        "lastName": "D",
        "email": "c@d.com",
        "avatar": null,
        "isSeller": true,
        "createdAt": "2026-01-07T10:00:00.000Z"
      }
    }
  ],
  "message": null
}
```

### 5) Get a user’s followers

- **GET** `/follows/followers/:userId`
- **Auth**: required
- **Success (200)** same response shape as **Get my followers**.

### 6) Get a user’s following

- **GET** `/follows/following/:userId`
- **Auth**: required
- **Success (200)** same response shape as **Get who I’m following**.

### 7) Check if I follow a user

- **GET** `/follows/status/:userId`
- **Auth**: required
- **Success (200)**:

```json
{
  "success": true,
  "data": { "isFollowing": true, "followId": "FOLLOW_UUID" },
  "message": null
}
```

### 8) Follow stats (counts)

- **GET** `/follows/stats/me` (my counts)
- **GET** `/follows/stats/:userId` (any user counts)
- **Auth**: required
- **Success (200)**:

```json
{
  "success": true,
  "data": { "followers": 10, "following": 5 },
  "message": null
}
```

### 9) Get user profile + followers + following + stats (single call)

- **GET** `/follows/user/:userId`
- **Auth**: required
- **Success (200)**:

```json
{
  "success": true,
  "data": {
    "id": "USER_UUID",
    "firstName": "X",
    "lastName": "Y",
    "email": "x@y.com",
    "phoneNumber": "....",
    "avatar": null,
    "isNotify": false,
    "isActive": true,
    "isApproved": true,
    "isSeller": false,
    "createdAt": "2026-01-07T10:00:00.000Z",
    "updatedAt": "2026-01-07T10:00:00.000Z",
    "followers": [
      /* same items as GET /follows/followers/:userId */
    ],
    "following": [
      /* same items as GET /follows/following/:userId */
    ],
    "stats": { "followers": 10, "following": 5 }
  },
  "message": null
}
```

## Flutter Example (Dio)

```dart
import 'package:dio/dio.dart';

class Api {
  final Dio dio;

  Api(String baseUrl, {required String token})
      : dio = Dio(BaseOptions(
          baseUrl: '$baseUrl/api',
          headers: {'Authorization': 'Bearer $token'},
        ));

  Future<Map<String, dynamic>> follow(String userId) async {
    final r = await dio.post('/follows', data: {'followingId': userId});
    return r.data as Map<String, dynamic>;
  }

  Future<Map<String, dynamic>> unfollow(String userId) async {
    final r = await dio.delete('/follows', data: {'followingId': userId});
    return r.data as Map<String, dynamic>;
  }

  Future<Map<String, dynamic>> followStatus(String userId) async {
    final r = await dio.get('/follows/status/$userId');
    return r.data as Map<String, dynamic>;
  }

  Future<Map<String, dynamic>> userWithFollows(String userId) async {
    final r = await dio.get('/follows/user/$userId');
    return r.data as Map<String, dynamic>;
  }

  Future<List<dynamic>> myFollowers() async {
    final r = await dio.get('/follows/followers/me');
    return (r.data as Map<String, dynamic>)['data'] as List<dynamic>;
  }

  Future<List<dynamic>> myFollowing() async {
    final r = await dio.get('/follows/following/me');
    return (r.data as Map<String, dynamic>)['data'] as List<dynamic>;
  }
}
```


