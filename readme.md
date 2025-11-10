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

---

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


## 🚢 Production Deployment

<div align="center">

### 📦 Deploy to AWS EC2 with nginx & SSL

*Complete blueprint for production-ready Socket.IO deployment*

</div>

### 🖥️ EC2 Instance Setup

**1. Launch & Configure EC2 Instance**
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Node.js (using NodeSource repository)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install nginx
sudo apt install -y nginx

# Install PM2 for process management
sudo npm install -g pm2
```

**2. Clone & Setup Application**
```bash
# Clone your repository
git clone <your-repo-url>
cd <your-project-folder>

# Install dependencies
npm install

# Start with PM2
pm2 start server.js --name flood-it-backend
pm2 startup systemd  # Enable auto-restart on reboot
pm2 save
```

### 🔒 SSL Certificate Setup

**Configure Let's Encrypt for HTTPS**
```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Obtain SSL certificate (replace your-domain.com)
sudo certbot --nginx -d your-domain.com

# Auto-renewal is configured by default
# Test renewal with:
sudo certbot renew --dry-run
```

### ⚙️ nginx Configuration

**Create nginx config file**: `/etc/nginx/sites-available/flood-it-backend`

```nginx
# WebSocket upgrade handling
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

# HTTP → HTTPS redirect
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$host$request_uri;
}

# HTTPS server with Socket.IO support
server {
    listen 443 ssl http2;
    server_name your-domain.com;

    # SSL certificate paths (auto-configured by certbot)
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    
    # SSL security settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # Socket.IO specific proxy
    location /socket.io/ {
        proxy_pass http://localhost:4000/socket.io/;
        proxy_http_version 1.1;
        
        # WebSocket headers
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Extended timeout for long-lived connections
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
        
        # Disable buffering for real-time data
        proxy_buffering off;
    }

    # Regular HTTP endpoints
    location / {
        proxy_pass http://localhost:4000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Enable the configuration**
```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/flood-it-backend /etc/nginx/sites-enabled/

# Test nginx configuration
sudo nginx -t

# Restart nginx
sudo systemctl restart nginx
```

### 🔍 Verification & Testing

**Check service status**
```bash
# Verify PM2 is running
pm2 status

# Check nginx status
sudo systemctl status nginx

# View application logs
pm2 logs flood-it-backend

# View nginx error logs
sudo tail -f /var/log/nginx/error.log
```

**Test WebSocket connection**
```bash
# Test from local machine
curl https://your-domain.com

# Test Socket.IO endpoint
curl https://your-domain.com/socket.io/?EIO=4&transport=polling
```

### 🔥 Firewall Configuration

```bash
# Allow HTTP, HTTPS, and SSH
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

### 📝 Important Notes

> **🎯 Key Configuration Points:**
> - Server must run on `localhost:4000` (not `0.0.0.0`)
> - nginx handles SSL termination and proxies to Node.js
> - WebSocket upgrade headers are **critical** for Socket.IO
> - Extended timeouts prevent WebSocket disconnections
> - PM2 ensures the app restarts on crashes/reboots

> **⚠️ Troubleshooting:**
> - **502 Bad Gateway**: Check if Node.js server is running (`pm2 status`)
> - **WebSocket fails**: Verify nginx `Upgrade` and `Connection` headers
> - **CORS errors**: Update Socket.IO CORS config to include your domain
> - **SSL issues**: Ensure certbot renewal is working (`sudo certbot renew --dry-run`)

---

## 🤝 Contributing

This is a backend server for the Flood It game. When contributing, please ensure:
- All socket events are properly handled
- Database operations include error handling
- Room cleanup logic prevents memory leaks
- Scoring algorithm remains consistent

---

*Built with ❤️*
