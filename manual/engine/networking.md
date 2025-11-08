---
layout: default
permalink: /manual/engine/networking/
title: Networking
parent: Engine
grand_parent: Manual
nav_order: 17
---

# Networking

The networking system provides low-level networking, online services, and multiplayer capabilities.

**Note:** This documentation describes the networking architecture. Specific API examples are illustrative - refer to `code/Net/` and `code/Online/` for actual interfaces.

## Overview

Traktor networking includes three layers:

### Low-Level Networking (`Net` module)
- **Sockets:** TCP and UDP communication
- **HTTP Client:** Web requests
- **WebSockets:** Real-time web communication
- **Serialization:** Network message serialization

### Online Services (`Online` module)
- **Authentication:** User login/registration
- **Leaderboards:** Score tracking
- **Achievements:** Achievement system
- **Cloud Save:** Save game to cloud

### Multiplayer (`Runtime/Multiplayer`)
- **Session Management:** Create/join game sessions
- **State Synchronization:** Replicate game state
- **RPC:** Remote procedure calls

## Low-Level Networking

### TCP Sockets

```cpp
// C++ - TCP client
Ref<TcpSocket> socket = new TcpSocket();
if (socket->connect(SocketAddressIPv4("127.0.0.1", 8080)))
{
    // Send data
    const char* message = "Hello Server";
    socket->send(message, strlen(message));

    // Receive data
    char buffer[1024];
    int received = socket->recv(buffer, sizeof(buffer));
}
```

### UDP Sockets

```cpp
// C++ - UDP
Ref<UdpSocket> socket = new UdpSocket();
socket->bind(SocketAddressIPv4(8080));

// Send
socket->sendTo(data, size, remoteAddress);

// Receive
socket->recvFrom(buffer, bufferSize, senderAddress);
```

### HTTP Requests

```lua
-- Lua - HTTP GET
local http = context.net:createHttpClient()
local response = http:get("https://api.example.com/data")

if response.status == 200 then
    log:info("Response: " .. response.body)
end
```

## Online Services

### Authentication

```lua
-- Login
local online = context.online
local result = online:login(username, password)

if result.success then
    log:info("Logged in as: " .. result.userId)
end
```

### Leaderboards

```lua
-- Submit score
online:submitScore("HighScores", 12345)

-- Get leaderboard
local scores = online:getLeaderboard("HighScores", 10)  -- Top 10

for i, entry in ipairs(scores) do
    log:info(entry.rank .. ". " .. entry.username .. ": " .. entry.score)
end
```

### Achievements

```lua
-- Unlock achievement
online:unlockAchievement("FirstKill")

-- Get achievements
local achievements = online:getAchievements()
```

## Multiplayer

### Session Management

```lua
-- Create session
local session = multiplayer:createSession("MyGame", 4)  -- Max 4 players

-- Join session
local session = multiplayer:joinSession(sessionId)

-- Leave session
multiplayer:leaveSession()
```

### State Synchronization

```cpp
// C++ - Mark object for replication
class MyReplicatedObject : public IReplicatedObject
{
    virtual void serialize(ISerializer& s) override
    {
        s >> m_position;
        s >> m_health;
    }
};
```

### RPC (Remote Procedure Calls)

```lua
-- Call function on remote client
multiplayer:rpc("onPlayerJoined", {
    playerId = playerId,
    playerName = playerName
})

-- Handle RPC
function Script:onPlayerJoined(self, params)
    log:info("Player joined: " .. params.playerName)
end
```

## Network Serialization

Efficient binary serialization:

```cpp
// Serialize
BinaryWriter writer;
writer << position;
writer << velocity;
writer << health;

// Deserialize
BinaryReader reader(data);
reader >> position;
reader >> velocity;
reader >> health;
```

## Best Practices

1. **Use UDP for Real-Time:** Fast-paced games need UDP
2. **TCP for Reliability:** Use TCP for important state (inventory, etc.)
3. **Minimize Bandwidth:** Only send what changed
4. **Client Prediction:** Predict local player movement
5. **Server Authority:** Server validates all actions
6. **Handle Disconnects:** Gracefully handle network errors

## Common Patterns

### Client-Server Architecture

```
Server (Authoritative)
  ├─ Validates all actions
  ├─ Simulates game world
  └─ Broadcasts state to clients

Clients
  ├─ Send input to server
  ├─ Predict local movement
  └─ Interpolate remote entities
```

### Peer-to-Peer

```
All Peers
  ├─ Share equal authority
  ├─ Synchronize state
  └─ Resolve conflicts
```

## Debugging

### Network Statistics

```lua
-- Get network stats
local stats = multiplayer:getStats()
log:info("Ping: " .. stats.ping .. " ms")
log:info("Packet loss: " .. stats.packetLoss .. "%")
log:info("Bandwidth: " .. stats.bandwidth .. " KB/s")
```

### Packet Logging

```cpp
// Enable packet logging
netConfig.logPackets = true;
```

## See Also

- [Scripting](scripting.md) - Network code in Lua

## References

- Source: `code/Net/` - Low-level networking
- Source: `code/Online/` - Online services
