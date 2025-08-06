# 🌊 Flood It - Game Backend Server

A real-time multiplayer game backend server built with Node.js, Socket.IO, and AWS DynamoDB. This server powers the Flood It puzzle game, enabling both multiplayer room-based gameplay and single-player score tracking with persistent leaderboards.

## ✨ Features

### 🎮 Multiplayer Game Rooms
- **Room Creation**: Players can create custom game rooms with configurable settings
- **Room Joining**: Join existing rooms with unique room keys
- **Real-time Updates**: Live player list updates and room status changes
- **Host Controls**: Room creators can start games for all participants

### ⚙️ Game Configuration
- **Grid Size**: Customizable game board dimensions
- **Color Count**: Adjustable number of colors (affects difficulty)
- **Rounds**: Configurable number of game rounds per session

### 🏆 Scoring & Leaderboards
- **Advanced Scoring**: Sophisticated scoring algorithm considering:
  - Average moves per round
  - Average time per round
  - Color difficulty multiplier
- **Global Leaderboards**: Persistent leaderboards stored in AWS DynamoDB
- **Board Size Categories**: Separate leaderboards for different grid sizes
- **Single Player Support**: Direct score submission for solo gameplay

### 🔄 Real-time Communication
- **WebSocket Support**: Full Socket.IO implementation with fallback to polling
- **Connection Management**: Automatic cleanup of disconnected players
- **Live Game Updates**: Real-time game state synchronization

## 🚀 Getting Started

### Prerequisites
- Node.js (v14 or higher)
- AWS Account with DynamoDB access
- AWS credentials configured

### Installation

1. **Clone and install dependencies:**
```bash
npm install
```

2. **Configure AWS DynamoDB:**
   - Ensure you have a DynamoDB table named `Flood-It-Leaderboard`
   - Table should have a Global Secondary Index named `ScoreIndex`
   - Configure AWS credentials for the `eu-north-1` region

3. **Start the server:**
```bash
npm run server
```

The server will start on port 4000.

## 📡 API Reference

### Socket Events

#### Client → Server

| Event | Payload | Description |
|-------|---------|-------------|
| `create-room` | `{ name, gridSize, colors, rounds }` | Create a new game room |
| `join-room` | `{ roomKey, name }` | Join an existing room |
| `start-game` | `roomKey` | Start the game (host only) |
| `player-finished` | `{ roomKey, playerId, name, moves, time }` | Submit player completion data |
| `get-leaderboard` | `{ boardSize, limit }` | Fetch leaderboard for board size |
| `leave-room` | `{ roomKey }` | Leave the current room |

#### Server → Client

| Event | Payload | Description |
|-------|---------|-------------|
| `room-updated` | `Room` | Room state has changed |
| `game-started` | `settings` | Game has begun with settings |
| `leaderboard-update` | `stats[]` | End-of-game leaderboard |

### REST Endpoints

#### POST `/insertSinglePlayer`
Submit single player game results.

**Request Body:**
```json
{
  "playerId": "string",
  "player": "string",
  "gridSize": "number",
  "rounds": "number", 
  "colors": "number",
  "moves": "number",
  "endTime": "number"
}
```

**Responses:**
- `200`: Success - Score inserted into leaderboard
- `400`: Bad Request - Missing or invalid fields
- `500`: Server Error - Database insertion failed

## 🏗️ Architecture

### Room Management
```typescript
type Room = {
  host: string;           // Socket ID of room creator
  players: Player[];      // List of connected players
  settings: {             // Game configuration
    gridSize: number;
    colors: number;
    rounds: number;
  };
  started: boolean;       // Game state
};
```

### Scoring Algorithm
The scoring system uses a sophisticated algorithm that accounts for:
- **Moves**: Average moves per round (lower is better)
- **Time**: Average time per round in milliseconds
- **Difficulty**: Color count multiplier (15% increase per color above 5)

```
score = (avgMoves × 1,000,000 + avgTime) ÷ colorFactor
```

## 🛠️ Technologies

- **Node.js** - Runtime environment
- **Express.js** - Web application framework
- **Socket.IO** - Real-time bidirectional communication
- **AWS DynamoDB** - NoSQL database for leaderboards
- **nanoid** - Unique room key generation
- **CORS** - Cross-origin resource sharing

## 🔧 Configuration

### Environment Setup
- **AWS Region**: `eu-north-1`
- **Server Port**: `4000`
- **DynamoDB Table**: `Flood-It-Leaderboard`
- **GSI**: `ScoreIndex`

### Socket.IO Configuration
- **CORS**: Enabled for all origins
- **Transports**: WebSocket with polling fallback
- **Compatibility**: Supports Engine.IO v3 clients

## 📊 Database Schema

### DynamoDB Table Structure
```
Flood-It-Leaderboard
├── boardSize (S) - Partition key (e.g., "5x5")
├── score (N) - Sort key for ScoreIndex GSI
├── username (S) - Player name
├── playerId (S) - Unique player identifier
├── moves (N) - Total moves taken
├── time (N) - Total time in milliseconds
├── timestamp (N) - Unix timestamp
└── colors (N) - Number of colors used
```

## 🤝 Contributing

This is a backend server for the Flood It game. When contributing, please ensure:
- All socket events are properly handled
- Database operations include error handling
- Room cleanup logic prevents memory leaks
- Scoring algorithm remains consistent

---

*Built with ❤️*
