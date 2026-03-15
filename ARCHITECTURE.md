# Kirik Architecture

## System Architecture Diagram

```mermaid
graph TB
    subgraph Mobile["📱 Flutter Mobile App (iOS + Android)"]
        UI[Game UI / Screens]
        RP[Riverpod State]
        SS[Socket Service]
        AS[Auth Service]
    end

    subgraph Backend["⚙️ Node.js Backend Server"]
        EX[Express REST API]
        SIO[Socket.IO Server]
        GM[Game Controller]
        RM[Room Controller]
        CM[Chat Controller]
        VM[Voice Controller]
        MW[Auth + Rate Limit Middleware]
    end

    subgraph DataLayer["💾 Data Layer"]
        RD[(Redis\nRoom State\nLeaderboard Cache\nSessions)]
        FS[(Firebase Firestore\nUser Profiles\nGame History\nReports)]
        FA[Firebase Auth\nEmail / Google / Apple]
    end

    subgraph Scaling["📈 Horizontal Scaling"]
        RA[Redis Adapter\nSocket.IO Rooms\nAcross Instances]
        LB[Load Balancer\nNginx / Cloud]
    end

    UI --> RP
    RP --> SS
    RP --> AS
    AS --> FA
    SS -- "WebSocket\nSocket.IO" --> SIO
    SS -- "HTTPS REST" --> EX

    SIO --> MW
    EX --> MW
    MW --> GM
    MW --> RM
    MW --> CM
    MW --> VM

    GM --> RD
    RM --> RD
    GM --> FS
    EX --> FS
    EX --> RD

    SIO --> RA
    RA --> RD
    LB --> Backend
```

## Data Flow: Game Round

```mermaid
sequenceDiagram
    participant Owner as Room Owner
    participant Server as Backend Server
    participant Guesser as Other Players
    participant Redis as Redis Cache

    Owner->>Server: start_game
    Server->>Redis: Set room.status = 'playing'
    Server->>Owner: round_start_drawer (with word)
    Server->>Guesser: round_start (word length + hint)

    loop Draw Loop
        Owner->>Server: draw {x,y,color,size}
        Server->>Guesser: draw_data (broadcast)
    end

    Note over Server: After 25 seconds
    Server->>Guesser: hint_reveal (1 letter)

    Guesser->>Server: guess {message: "cat"}
    Server->>Server: Check: "cat" == "cat" ✓
    Server->>Owner: correct_guess (+50 pts)
    Server->>Guesser: correct_guess (+350 pts, scores)

    Note over Server: All guessed or timer ends
    Server->>Owner: round_end {word, scores}
    Server->>Guesser: round_end {word, scores}

    Note over Server: 4 second pause
    Server->>Owner: round_start_drawer (next word)
    Server->>Guesser: round_start (next round)
```

## WebRTC Voice Chat Flow

```mermaid
sequenceDiagram
    participant P1 as Player 1
    participant Server as Signaling Server
    participant P2 as Player 2

    P1->>Server: request_peers
    Server->>P1: room_peers [P2_socketId]

    P1->>Server: webrtc_offer {targetSocketId: P2, offer}
    Server->>P2: webrtc_offer {fromSocketId: P1, offer}

    P2->>Server: webrtc_answer {targetSocketId: P1, answer}
    Server->>P1: webrtc_answer {fromSocketId: P2, answer}

    P1->>Server: webrtc_ice_candidate {targetSocketId: P2, candidate}
    Server->>P2: webrtc_ice_candidate {fromSocketId: P1, candidate}

    Note over P1,P2: Direct P2P voice connection established
    P1-->>P2: Audio stream (direct WebRTC, no server relay)

    P1->>Server: voice_state {speaking: true}
    Server->>P2: player_voice_state {socketId: P1, speaking: true}
```

## Database Schema

### Firestore Collections

```
users/
  {userId}/
    username: string
    email: string
    avatar: string
    level: number
    totalScore: number
    totalGames: number
    wins: number
    friends: string[]   // array of userIds
    createdAt: timestamp
    lastPlayedAt: timestamp

game_history/
  {gameId}/
    roomId: string
    language: string
    totalRounds: number
    playerCount: number
    leaderboard: [{userId, username, score}]
    endedAt: timestamp

reports/
  {reportId}/
    reporterId: string
    reporterName: string
    targetUserId: string
    reason: string
    roomId: string
    status: 'pending' | 'reviewed' | 'actioned'
    createdAt: timestamp

weekly_scores/
  {docId}/
    userId: string
    username: string
    score: number
    weekStart: timestamp
```

### Redis Key Schema

```
room:{roomId}               → JSON (Room state, TTL: 2h)
room_code:{CODE}            → roomId string (TTL: 2h)
public_rooms                → Set of roomIds
leaderboard:global          → Sorted set (userId → totalScore)
leaderboard:weekly          → Sorted set (userId → weekScore, TTL: end of week)
```
