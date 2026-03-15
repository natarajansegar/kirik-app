# Kirik REST API Documentation

Base URL: `https://your-server.onrender.com`

All authenticated endpoints require:
```
Authorization: Bearer <firebase-id-token>
```

---

## Health Check

### `GET /health`
Returns server status.

**Response:**
```json
{ "status": "ok", "timestamp": "2024-01-15T12:00:00.000Z" }
```

---

## Authentication

### `POST /api/auth/register`
Create a user profile in Firestore after Firebase Auth signup.

**Auth:** Required

**Body:**
```json
{ "username": "Alice" }
```

**Response:**
```json
{
  "success": true,
  "profile": {
    "userId": "firebase-uid",
    "username": "Alice",
    "avatar": "default",
    "level": 1,
    "totalScore": 0
  }
}
```

---

### `POST /api/auth/guest`
Get a guest session ID.

**Rate limit:** 10 requests / 15 minutes

**Body:**
```json
{ "username": "GuestPlayer" }
```

**Response:**
```json
{ "guestId": "uuid", "username": "GuestPlayer" }
```

---

## Rooms

### `GET /api/rooms`
List all public waiting rooms.

**Response:**
```json
{
  "rooms": [
    {
      "id": "uuid",
      "code": "ABC123",
      "ownerName": "Alice",
      "type": "public",
      "maxPlayers": 8,
      "playerCount": 3,
      "totalRounds": 5,
      "roundDuration": 80,
      "language": "en",
      "status": "waiting"
    }
  ]
}
```

---

### `GET /api/rooms/:code`
Get a room by its 6-character code.

**Response:**
```json
{ "room": { ...roomObject } }
```

**Error:**
```json
{ "error": "Room not found" }  // 404
```

---

## Leaderboard

### `GET /api/leaderboard/global?limit=50`
Get the global all-time leaderboard.

**Response:**
```json
{
  "leaderboard": [
    { "rank": 1, "userId": "uid1", "username": "Alice", "totalScore": 12500, "totalGames": 48 },
    { "rank": 2, "userId": "uid2", "username": "Bob", "totalScore": 9800, "totalGames": 37 }
  ]
}
```

---

### `GET /api/leaderboard/weekly?limit=50`
Get this week's leaderboard (resets every Monday).

---

### `GET /api/leaderboard/friends`
Get friends leaderboard for the authenticated user.

**Auth:** Required

---

### `GET /api/leaderboard/rank/:userId`
Get a specific player's global rank.

**Response:**
```json
{ "rank": { "rank": 42, "score": 5600 } }
```

---

## Profile

### `GET /api/profile/:userId`
Get a user's public profile.

**Response:**
```json
{
  "profile": {
    "userId": "uid",
    "username": "Alice",
    "avatar": "default",
    "level": 5,
    "totalScore": 12500,
    "totalGames": 48,
    "wins": 12
  }
}
```

---

### `POST /api/profile/friend/:friendId`
Add a player to your friends list.

**Auth:** Required

**Response:**
```json
{ "success": true }
```

---

## Error Responses

All errors follow this format:
```json
{ "error": "Human-readable error message" }
```

| Status | Meaning |
|--------|---------|
| 400 | Bad request (validation error) |
| 401 | Unauthorized (missing/invalid token) |
| 404 | Resource not found |
| 429 | Too many requests (rate limited) |
| 500 | Internal server error |
