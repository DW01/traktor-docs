---
layout: default
permalink: /engine/networking/
title: Networking
parent: Engine

nav_order: 17
---

# Networking - Connecting Players Together

Multiplayer games create moments that single-player games simply can't match. The thrill of competing against real people, the joy of cooperating with friends, the emergent gameplay that happens when unpredictable human players interact. But networking is also one of the most challenging aspects of game development. Players are separated by hundreds or thousands of miles, connected by unreliable internet connections with varying latency. Data arrives late, out of order, or not at all. And you need to make it all feel responsive and fair.

Traktor's networking system provides three layers of functionality: **low-level networking** (TCP/UDP sockets, HTTP for direct communication), **online services** (session management, matchmaking with lobbies, leaderboards, achievements, cloud saves), and **state replication** (the Jungle module for synchronizing game state across peers). Together, these tools let you build anything from simple leaderboards to fully synchronized multiplayer experiences.

Think of networking as building a bridge between separate game instances running on different machines. You need to decide what information crosses that bridge (positions? Health? Input?), how it's transmitted (reliably or fast?), and how to handle the inevitable problems (lag, packet loss, cheating). Traktor gives you the low-level tools for direct control (Net module), high-level online services (Online module for lobbies and matchmaking), and state replication systems (Jungle module) for synchronizing game state.

**Note:** This documentation describes the networking architecture and common patterns. Specific API details may vary. Refer to `code/Net/` and `code/Online/` for actual interfaces.

## The Challenge of Networking

Before diving into code, it's important to understand what makes networking hard:

**Latency (ping)** is the time it takes for data to travel from one machine to another. Even in ideal conditions, light-speed limits mean players on opposite sides of the world will have 100+ms delays. Every action takes time to propagate.

**Packet loss** happens when data fails to arrive. UDP packets can be lost, requiring retransmission or acceptance of missing data.

**Bandwidth** limits how much data you can send per second. Send too much, and you'll saturate connections, causing lag and stuttering.

**Cheating** is easier in multiplayer. Client-side validation isn't enough. The server must verify everything, or players will manipulate local data to cheat.

**Synchronization** is complex. Every client sees a slightly different version of the world due to latency. You need techniques like client-side prediction and server reconciliation to make it feel smooth.

Understanding these challenges is the first step to building robust networked games.

## Low-Level Networking (`Net` Module)

The `Net` module provides foundational networking primitives:

### TCP Sockets: Reliable Streaming

**TCP** guarantees that data arrives in order and without loss. It's perfect for critical data (player inventory, login credentials, chat messages) where reliability matters more than speed.

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

**Use TCP for:**
- Authentication and login
- Chat messages
- Important state changes (inventory, equipment)
- Anything where missing or out-of-order data would break the game

**Avoid TCP for:**
- Real-time player movement (latency adds up)
- Fast-paced action (TCP's reliability comes with delay)

### UDP Sockets: Fast and Unreliable

**UDP** sends data without guarantees. Packets can arrive out of order, be lost entirely, or be duplicated. But it's fast. No handshaking, no retransmission delays.

```cpp
// C++ - UDP
Ref<UdpSocket> socket = new UdpSocket();
socket->bind(SocketAddressIPv4(8080));

// Send
socket->sendTo(data, size, remoteAddress);

// Receive
socket->recvFrom(buffer, bufferSize, senderAddress);
```

**Use UDP for:**
- Player positions and movement
- Fast-paced combat
- Physics simulation
- Anything where the latest data matters more than every update

**Handle packet loss** by sending frequent updates. If one packet is lost, the next one arrives shortly after. For player position, you'd rather have the latest position than wait for retransmission of an old one.

### HTTP Requests: Web Integration

**HTTP** lets you communicate with web services—REST APIs, web servers, cloud services:

```lua
-- Lua - HTTP GET request
local httpClient = net.HttpClient()
local url = net.Url("https://api.example.com/data")
local result = httpClient:get(url)

-- result is HttpClientResult
local response = result.response
if response.statusCode == 200 then
    -- Read response body from stream
    local stream = result.stream
    -- Process stream data...
    log:info("Request successful, status: " .. tostring(response.statusCode))
end
```

The HttpClient also supports POST and PUT with optional content:

```lua
-- HTTP POST with body
local content = net.HttpRequestContent("{ \"key\": \"value\" }")
local result = httpClient:post(url, content)
```

**Use HTTP for:**
- Fetching data from web APIs
- Submitting scores to online leaderboards
- Downloading updates or user-generated content
- Communicating with custom game servers

**Note:** The response body is provided as an IStream. Use stream reading methods to extract the data. HTTP operations are synchronous in the current API.

## Online Services (`Online` Module)

The `Online` module handles common online features:

### Session Management and User Identity

The `ISessionManager` interface manages the online session and provides access to online services:

```lua
-- Get the session manager (typically provided by the runtime)
local sessionManager = context.sessionManager

-- Check if connected to online services
if sessionManager.connected then
    log:info("Connected to online services")

    -- Get current user
    local user = sessionManager.user
    local userName = user.name
    log:info("Logged in as: " .. userName)

    -- Access online services
    local matchMaking = sessionManager.matchMaking
    local leaderboards = sessionManager.leaderboards
    local achievements = sessionManager.achievements
    local saveData = sessionManager.saveData
end
```

**Note:** Authentication and login are typically handled by the platform (Steam, Epic, console services) before your game starts. The SessionManager provides access to the already-authenticated user.

### Leaderboards: Competition and Comparison

Leaderboards let players compare scores and compete for rankings:

```lua
import(traktor)

LeaderboardManager = LeaderboardManager or class("LeaderboardManager", world.ScriptComponent)

function LeaderboardManager:submitScore(leaderboardId, score)
    local sessionManager = self.context.sessionManager
    local leaderboards = sessionManager.leaderboards

    if leaderboards and leaderboards.ready then
        leaderboards:setScore(leaderboardId, score)
        log:info("Submitted score: " .. tostring(score))
    end
end

function LeaderboardManager:getTopScores(leaderboardId)
    local sessionManager = self.context.sessionManager
    local leaderboards = sessionManager.leaderboards

    if leaderboards and leaderboards.ready then
        -- Get global top scores (async)
        local result = leaderboards:getGlobalScores(leaderboardId, 0, 10)  -- Top 10

        -- Check result when ready
        if result:succeed() then
            local scores = result.scores
            for i, score in ipairs(scores) do
                local user = score.user
                log:info(score.rank .. ". " .. user.name .. ": " .. score.score)
            end
        end
    end
end
```

**Multiple leaderboards** track different metrics. Create separate leaderboard IDs for each:

```lua
leaderboards:setScore("FastestRun", timeInSeconds)
leaderboards:setScore("MostKills", killCount)
```

### Achievements: Goals and Progression

Achievements reward players for completing specific challenges:

```lua
import(traktor)

AchievementManager = AchievementManager or class("AchievementManager", world.ScriptComponent)

function AchievementManager:unlockAchievement(achievementId)
    local sessionManager = self.context.sessionManager
    local achievements = sessionManager.achievements

    if achievements and achievements.ready then
        -- Check if already unlocked
        if not achievements:have(achievementId) then
            -- Unlock the achievement
            achievements:set(achievementId, true)
            log:info("Achievement unlocked: " .. achievementId)
        end
    end
end

function AchievementManager:listAchievements()
    local sessionManager = self.context.sessionManager
    local achievements = sessionManager.achievements

    if achievements and achievements.ready then
        -- Get all achievement IDs
        local achievementIds = achievements:enumerate()

        for _, id in ipairs(achievementIds) do
            local unlocked = achievements:have(id)
            log:info(id .. ": " .. tostring(unlocked))
        end
    end
end
```

**Note:** The achievements API uses `set(id, true)` to unlock achievements. Progress tracking for incremental achievements would need to be implemented in your game logic using statistics or custom data.

### Cloud Saves: Play Anywhere

Cloud saves let players continue their progress on different devices:

```lua
import(traktor)

CloudSaveManager = CloudSaveManager or class("CloudSaveManager", world.ScriptComponent)

function CloudSaveManager:saveToCloud(saveSlotId, gameState)
    local sessionManager = self.context.sessionManager
    local saveData = sessionManager.saveData

    if saveData and saveData.ready then
        -- Save data (async operation)
        local result = saveData:set(
            saveSlotId,
            "My Save",  -- Title
            "Auto-save",  -- Description
            gameState,  -- ISerializable object
            true  -- Replace existing
        )
        -- Check result later with result:succeed()
    end
end

function CloudSaveManager:loadFromCloud(saveSlotId)
    local sessionManager = self.context.sessionManager
    local saveData = sessionManager.saveData

    if saveData and saveData.ready then
        -- Load data (async operation)
        local result = saveData:get(saveSlotId)

        -- Check result when ready
        if result:succeed() then
            local loadedData = result.attachment
            -- Deserialize loadedData
            return loadedData
        end
    end
    return nil
end
```

**Note:** Save data must be ISerializable objects. Cloud save operations are asynchronous. Check results with the `:succeed()` method.

## Multiplayer: Matchmaking and State Replication

Multiplayer in Traktor uses two systems working together:
- **Online Module** (IMatchMaking and ILobby) - for finding players and managing lobbies
- **Jungle Module** (Replicator) - for synchronizing game state across peers

### Matchmaking and Lobbies

**Lobbies** bring players together before starting a match:

```lua
import(traktor)

MatchmakingManager = MatchmakingManager or class("MatchmakingManager", world.ScriptComponent)

function MatchmakingManager:createLobby()
    local sessionManager = self.context.sessionManager
    local matchMaking = sessionManager.matchMaking

    if matchMaking and matchMaking.ready then
        -- Create a lobby (async)
        local result = matchMaking:createLobby(
            4,  -- Max players
            "public"  -- Access: "public", "private", or "friends"
        )

        -- Check result when ready
        if result:succeed() then
            self._lobby = result.lobby
            log:info("Lobby created")
            log:info("Max players: " .. tostring(self._lobby.maxParticipantCount))
        end
    end
end

function MatchmakingManager:findLobbies()
    local sessionManager = self.context.sessionManager
    local matchMaking = sessionManager.matchMaking

    if matchMaking and matchMaking.ready then
        -- Create search filter
        local filter = online.LobbyFilter()
        filter.slots = 1  -- Need at least 1 open slot
        filter.count = 10  -- Return up to 10 results

        -- Find matching lobbies (async)
        local result = matchMaking:findMatchingLobbies(filter)

        -- Check result when ready
        if result:succeed() then
            local lobbies = result.lobbies
            log:info("Found " .. tostring(#lobbies) .. " lobbies")
        end
    end
end

function MatchmakingManager:joinLobby(lobby)
    -- Join the lobby
    lobby:join()
    self._lobby = lobby
end

function MatchmakingManager:leaveLobby()
    if self._lobby then
        self._lobby:leave()
        self._lobby = nil
    end
end
```

### State Synchronization with Jungle Replicator

The **Jungle module** provides state replication across peers. It uses a template system to define what data gets synchronized:

```lua
import(traktor)

ReplicationManager = ReplicationManager or class("ReplicationManager", world.ScriptComponent)

function ReplicationManager:new()
    self._replicator = nil
    self._stateTemplate = nil
end

function ReplicationManager:setup()
    -- Create state template defining what to replicate
    self._stateTemplate = jungle.StateTemplate()

    -- Declare state variables
    -- Parameters: name, latency tolerance, priority?
    self._stateTemplate:declare(jungle.TransformTemplate("playerTransform"))
    self._stateTemplate:declare(jungle.FloatTemplate("health"))
    self._stateTemplate:declare(jungle.VectorTemplate("velocity"))

    -- Create P2P network topology
    local sessionManager = self.context.sessionManager
    local p2pProvider = jungle.OnlinePeer2PeerProvider(
        sessionManager,
        self._lobby,  -- ILobby from matchmaking
        true,  -- Enable P2P
        false  -- Not relayed
    )

    local topology = jungle.Peer2PeerTopology(p2pProvider)

    -- Create replicator
    self._replicator = jungle.Replicator()
    self._replicator:create(topology, nil)  -- nil uses default config
    self._replicator:setStateTemplate(self._stateTemplate)
end

function ReplicationManager:update(deltaTime)
    if self._replicator then
        -- Update replicator each frame
        self._replicator:update(deltaTime)

        -- Check if time is synchronized with other peers
        if self._replicator.timeSynchronized then
            -- Send state
            self:sendState()

            -- Receive state from other peers
            self:receiveStates()
        end
    end
end

function ReplicationManager:sendState()
    -- Pack current state
    local state = jungle.State()
    state:packBegin()
    state:pack(jungle.TransformValue(self.owner.transform))
    state:pack(jungle.FloatValue(self._health))
    state:pack(jungle.VectorValue(self._velocity))

    -- Send to all peers
    self._replicator:setSendState(state)
end

function ReplicationManager:receiveStates()
    -- Iterate through all peer proxies
    for i = 0, self._replicator.proxyCount - 1 do
        local proxy = self._replicator:getProxy(i)

        if proxy.connected then
            local state = proxy:getState()
            if state then
                -- Unpack remote peer's state
                state:unpackBegin()
                local transform = state:unpack():get()  -- TransformValue
                local health = state:unpack():get()  -- FloatValue
                local velocity = state:unpack():get()  -- VectorValue

                -- Apply to remote player entity
                -- (You would update the corresponding remote player here)
            end
        end
    end
end
```

**Key concepts:**
- **StateTemplate** defines what variables are replicated
- **State** contains packed values to send/receive
- **Replicator** manages network topology and synchronization
- **ReplicatorProxy** represents each remote peer

### Events: Remote Notifications

The Jungle Replicator provides an event system for sending notifications between peers. Events are ISerializable objects broadcast to all connected peers:

```lua
import(traktor)

-- Define an event class (must be ISerializable)
PlayerJoinedEvent = PlayerJoinedEvent or class("PlayerJoinedEvent", serialization.Serializable)

function PlayerJoinedEvent:new(playerId, playerName)
    self.playerId = playerId
    self.playerName = playerName
end

-- Register event type with replicator
replicator:addEventType(typeof("PlayerJoinedEvent"))

-- Add event listener
local listener = replicator:addEventListener(
    typeof("PlayerJoinedEvent"),
    function(replicator, eventTime, fromProxy, eventObject)
        -- Handle event
        local event = eventObject
        log:info("Player joined: " .. event.playerName)
        -- Spawn player avatar...
    end
)

-- Broadcast event to all peers
local event = PlayerJoinedEvent("player123", "John")
replicator:broadcastEvent(event, true)  -- true = in order

-- Or send event to primary peer only
replicator:sendEventToPrimary(event, false)  -- false = not guaranteed order
```

**Key concepts:**
- Event objects must be ISerializable (inherit from serialization.Serializable)
- Register event types with `addEventType()` before using
- `broadcastEvent()` sends to all peers
- `sendEventToPrimary()` sends only to the primary/host peer
- The `inOrder` parameter controls whether events arrive in order (reliable) or not (faster)

Events are great for notifications. Player joined, item picked up, ability used. Where you need to notify other machines of something that happened.

## Network Serialization: Efficient Data

Sending data over the network must be efficient. Every byte counts, especially on slower connections:

```cpp
// Serialize
BinaryWriter writer;
writer << position;      // 12 bytes (3 floats)
writer << velocity;      // 12 bytes
writer << health;        // 4 bytes (1 float)
// Total: 28 bytes

// Deserialize
BinaryReader reader(data);
reader >> position;
reader >> velocity;
reader >> health;
```

**Optimization techniques:**

**Quantization** - Store floats as smaller integers. Position might only need centimeter precision, not floating-point accuracy. Store as `int16` instead of `float`, saving 50% bandwidth.

**Delta compression** - Only send what changed. If position changed but health didn't, only send position.

**Prediction** - Don't send every value. Send velocity, let clients predict position.

## Advanced Multiplayer Techniques

### Client-Side Prediction

Without prediction, player input feels sluggish. You press W, wait for the server, then see movement. **Client-side prediction** makes local input feel instant:

1. Client predicts result of input immediately (moves locally)
2. Client sends input to server
3. Server simulates and sends authoritative position
4. Client reconciles (if prediction was wrong, correct it smoothly)

This makes movement feel responsive even with latency.

### Server Reconciliation

When server authority contradicts client prediction, you need reconciliation:

```lua
-- Client predicted: position = (10, 0, 5)
-- Server says: position = (9.8, 0, 5.1)
-- Reconcile: Smoothly blend to server position over 100ms
```

Instant snapping looks janky. Smooth interpolation looks natural.

### Lag Compensation

Players don't want to lead their shots to account for lag. **Lag compensation** (also called "rewinding") runs server simulation in the past:

When you shoot, the server rewinds other players to where they were from your perspective (accounting for your ping), checks hit, then returns to present. Your bullets hit where you aimed, despite lag.

### Interpolation

Remote players' positions arrive in discrete updates (maybe 20 times per second). **Interpolation** smooths between updates so movement looks continuous, not jerky:

```lua
-- Interpolate between known positions
function interpolate(prevPos, nextPos, t)
    return lerp(prevPos, nextPos, t)
end
```

## Best Practices

**Choose the right protocol.** UDP for real-time gameplay, TCP for reliable data. Don't use TCP for player movement. Latency stacks up.

**Minimize bandwidth.** Send only what changed. Quantize values. Compress data. Bandwidth is precious.

**Server authority prevents cheating.** Never trust clients. Validate everything on the server.

**Client prediction makes input responsive.** Predict locally, reconcile with server. Players notice input lag instantly.

**Handle disconnects gracefully.** Networks fail. Detect disconnects, try to reconnect, allow players to rejoin.

**Test with real network conditions.** Add artificial latency and packet loss during development to catch issues early.

**Monitor network stats.** Track ping, packet loss, bandwidth. Log and display for debugging.

## Common Networking Architectures

### Client-Server (Recommended)

The server is authoritative. Clients are dumb terminals that send input and render results:

```
Server (Authoritative)
  ├─ Receives input from all clients
  ├─ Simulates game world
  ├─ Validates actions (anti-cheat)
  └─ Broadcasts state to clients

Clients
  ├─ Send input to server
  ├─ Predict local player movement
  ├─ Render game state
  └─ Interpolate remote entities
```

**Pros:** Hard to cheat, consistent simulation, handles disconnects well

**Cons:** Requires dedicated server, clients feel latency

### Peer-to-Peer

All clients share authority. No dedicated server needed:

```
All Peers
  ├─ Share equal authority
  ├─ Broadcast actions to all peers
  ├─ Simulate independently
  └─ Resolve conflicts via consensus
```

**Pros:** No server costs, low latency between peers

**Cons:** Easy to cheat, hard to keep in sync, one slow peer affects everyone

### Hybrid

One client acts as "host" with authority, but other clients connect directly to each other for fast updates:

**Pros:** Balance of server benefits and peer performance

**Cons:** Complex to implement, host migration issues

## Debugging Network Issues

### Network Statistics

Monitor performance in real-time using the Jungle Replicator's latency properties:

```lua
-- Get network stats from replicator
if replicator then
    log:info("Average Latency: " .. tostring(replicator.averageLatency) .. " s")
    log:info("Best Latency: " .. tostring(replicator.bestLatency) .. " s")
    log:info("Worst Latency: " .. tostring(replicator.worstLatency) .. " s")
    log:info("Average Reverse Latency: " .. tostring(replicator.averageReverseLatency) .. " s")
    log:info("Time Synchronized: " .. tostring(replicator.timeSynchronized))
    log:info("Time Variance: " .. tostring(replicator.timeVariance) .. " s")
end
```

**Latency** - One-way time from local to remote peer. Values are in seconds (multiply by 1000 for milliseconds).

**Reverse latency** - One-way time from remote peer back to local.

**Best/Worst latency** - Minimum and maximum observed latencies. Track these to see connection stability.

**Time synchronized** - Boolean indicating if peers have synchronized clocks (required for accurate replication).

**Time variance** - Difference in synchronized time between peers.

### Packet Logging

Log all packets for debugging:

```cpp
// Enable packet logging
netConfig.logPackets = true;
```

This shows exactly what data is being sent and received, invaluable for tracking down sync issues.

### Simulate Network Conditions

Test with artificial lag and packet loss:

```cpp
// Simulate 100ms latency and 5% packet loss
netConfig.simulatedLatency = 100;  // ms
netConfig.simulatedPacketLoss = 5;  // percent
```

If your game works well under these conditions, it'll handle real-world networks.

## Security Considerations

**Never trust the client.** Clients can be modified. Always validate on the server.

**Encrypt sensitive data.** Passwords, payment info, personal data. Encrypt in transit.

**Rate limiting** prevents spam and DoS attacks. Limit how many messages a client can send per second.

**Authentication** ensures players are who they claim to be. Use secure tokens, not just usernames.

**Server-side validation** checks all actions. Client says "I picked up item"? Server checks if the item exists and is nearby.

## See Also

- [Scripting](scripting.md) - Networking code in Lua
- [World System](world.md) - Replicating entities

## References

- Source: `code/Net/` - Low-level networking (TCP, UDP, HTTP)
- Source: `code/Online/` - Online services (matchmaking, leaderboards, achievements, cloud saves)
- Source: `code/Jungle/` - State replication and P2P networking
