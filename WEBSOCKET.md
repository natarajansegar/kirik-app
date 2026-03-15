# Kirik WebSocket Events Documentation

All real-time communication uses **Socket.IO** over WebSocket (with polling fallback).

---

## Authentication

Connect with a Firebase ID token or guest credentials:

```javascript
// Authenticated user
const socket = io(SERVER_URL, {
  auth: { token: "firebase-id-token" }
});

// Guest user
const socket = io(SERVER_URL, {
  auth: { guestId: "uuid", guestName: "PlayerName" }
});
```

---

## Client → Server Events

### Room Management

| Event | Payload | Description |
|-------|---------|-------------|
| `create_room` | `{ type, maxPlayers, rounds, roundDuration, language, customWords }` | Create a new game room |
| `join_room` | `{ code: string }` | Join a room by 6-char code |
| `leave_room` | `{}` | Leave current room |
| `get_public_rooms` | `{}` | Request list of public rooms |
| `kick_player` | `{ targetSocketId: string }` | (Owner only) Kick a player |
| `report_player` | `{ targetUserId: string, reason: string }` | Report a player |

### Game Actions

| Event | Payload | Description |
|-------|---------|-------------|
| `start_game` | `{ roomId: string }` | (Owner only) Start the game |
| `guess` | `{ message: string }` | Submit a word guess |
| `chat_message` | `{ message: string }` | Send a chat message |
| `reaction` | `{ emoji: string }` | Send an emoji reaction |
| `mute_player` | `{ targetSocketId: string }` | (Owner only) Mute/unmute a player |
| `request_canvas_replay` | `{}` | Request full canvas history (for late joiners) |

### Drawing (drawer only)

| Event | Payload | Description |
|-------|---------|-------------|
| `draw` | `{ type, x, y, color, size }` | Send a drawing stroke |
| `clear_canvas` | `{}` | Clear the entire canvas |
| `undo` | `{}` | Undo the last stroke |

**Draw event types:**
- `start` — pen down, begins a new stroke
- `move` — pen moving, continues stroke
- `end` — pen up, finishes stroke

### Voice Chat (WebRTC Signaling)

| Event | Payload | Description |
|-------|---------|-------------|
| `request_peers` | `{}` | Get list of peer socket IDs in room |
| `webrtc_offer` | `{ targetSocketId, offer }` | Send WebRTC offer to a peer |
| `webrtc_answer` | `{ targetSocketId, answer }` | Send WebRTC answer |
| `webrtc_ice_candidate` | `{ targetSocketId, candidate }` | ICE candidate exchange |
| `voice_state` | `{ speaking: boolean }` | Broadcast push-to-talk state |

---

## Server → Client Events

### Room Events

| Event | Payload | Description |
|-------|---------|-------------|
| `room_created` | `{ room, code, player }` | Room was created successfully |
| `room_joined` | `{ room, players, spectator, currentRound, hint }` | Successfully joined room |
| `player_joined` | `{ player, playerCount }` | Another player joined |
| `player_left` | `{ socketId, username, reason, playerCount }` | A player left/disconnected |
| `new_owner` | `{ socketId, username }` | Room ownership transferred |
| `kicked` | `{ reason: string }` | You were kicked from the room |
| `public_rooms` | `{ rooms: Room[] }` | List of public rooms |
| `room_reset` | `{ room }` | Room reset to waiting state after game |

### Game Flow Events

| Event | Payload | Description |
|-------|---------|-------------|
| `game_started` | `{ totalRounds }` | Game has started |
| `round_start` | `{ round, totalRounds, drawerId, drawerName, wordLength, hint, duration, players }` | New round (for guessers) |
| `round_start_drawer` | `{ round, totalRounds, word, wordChoices, duration, players }` | New round (for drawer only) |
| `round_end` | `{ word, reason, playerScores, players }` | Round ended |
| `game_end` | `{ leaderboard }` | Game over with final rankings |
| `timer_tick` | `{ secondsLeft: number }` | Countdown tick every second |
| `hint_reveal` | `{ hint: string, hintLevel: number }` | A letter hint was revealed |
| `correct_guess` | `{ socketId, username, points, playerScores }` | Someone guessed correctly |

**Round end reasons:**
- `"time_up"` — timer ran out
- `"all_guessed"` — all non-drawers guessed correctly
- `"drawer_left"` — the drawer disconnected

### Chat Events

| Event | Payload | Description |
|-------|---------|-------------|
| `chat_message` | `{ socketId, username, message, type, timestamp }` | A chat message |
| `player_reaction` | `{ socketId, username, emoji }` | Emoji reaction |
| `player_muted` | `{ socketId, username, muted }` | Player muted/unmuted |

**Message types:** `chat`, `guess`, `close_guess`, `system`

### Canvas Events

| Event | Payload | Description |
|-------|---------|-------------|
| `draw_data` | `{ type, x, y, color, size }` | Incoming drawing stroke |
| `canvas_clear` | `{}` | Canvas was cleared |
| `canvas_replay` | `{ history: DrawEvent[] }` | Full canvas history replay |

### Voice Events

| Event | Payload | Description |
|-------|---------|-------------|
| `room_peers` | `{ peers: string[] }` | List of socket IDs for WebRTC mesh |
| `webrtc_offer` | `{ fromSocketId, offer }` | Incoming WebRTC offer |
| `webrtc_answer` | `{ fromSocketId, answer }` | Incoming WebRTC answer |
| `webrtc_ice_candidate` | `{ fromSocketId, candidate }` | ICE candidate |
| `player_voice_state` | `{ socketId, username, speaking }` | Peer speaking state |

### Error Events

| Event | Payload | Description |
|-------|---------|-------------|
| `error` | `{ message: string }` | Server-side error |

---

## Data Structures

### Room Object
```json
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
```

### Player Object
```json
{
  "socketId": "xxx",
  "userId": "firebase-uid",
  "username": "Alice",
  "avatar": "default",
  "score": 350,
  "isDrawing": false,
  "hasGuessedCorrectly": true,
  "isConnected": true
}
```

### DrawEvent Object
```json
{
  "type": "move",
  "x": 124.5,
  "y": 88.3,
  "color": "#FF4444",
  "size": 6
}
```

---

## Error Codes

| Scenario | Error Message |
|----------|---------------|
| Room not found | `"Room not found"` |
| Room is full | `"Room is full"` |
| Game already started | `"Game already started"` |
| Not room owner | `"Only room owner can start"` |
| Not enough players | `"Not enough players (need at least 2)"` |
| Rate limit exceeded | `"Rate limit exceeded. Slow down!"` |
| Auth failed | Connection rejected with `Error("Invalid authentication token")` |
