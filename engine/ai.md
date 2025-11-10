---
layout: default
permalink: /engine/ai/
title: AI
parent: Engine

nav_order: 15
---

# AI System - Teaching Characters to Navigate

Game AI creates the illusion that non-player characters think, plan, and react to the world around them. An enemy that patrols a hallway, chases you when spotted, and takes cover when under fire feels intelligent. Even though it's following simple rules. The foundation of most game AI is **navigation**: characters need to know where they can walk, how to get from point A to point B, and how to move around obstacles without getting stuck.

Traktor's AI system is built on **Recast/Detour**, a battle-tested library used in countless commercial games. It generates **navigation meshes** (navmeshes). Simplified representations of walkable surfaces in your level. And provides **pathfinding** to calculate routes from any start point to any destination. The system handles all the complexity: finding valid paths, steering around obstacles, handling different agent sizes, and doing it all asynchronously so pathfinding doesn't freeze your game.

Think of a navigation mesh like a simplified map of your level that AI characters can read. Instead of the full geometric complexity, the navmesh shows only the walkable floor areas, taking into account agent size (can they fit through doorways?), slope (can they climb stairs?), and obstacles (walls, furniture, cliffs). Characters query the navmesh to find paths, pick random destinations for wandering, or find the closest valid position if they're spawned inside a wall.

![TODO: Screenshot showing navigation mesh visualization with AI agents pathfinding around obstacles]

## Understanding Navigation Meshes

A **navigation mesh** (navmesh) is a 3D mesh that represents walkable surfaces in your game world. Unlike your visual level geometry, which might have millions of triangles, a navmesh has far fewer polygons. Just enough to represent where characters can walk.

When you generate a navmesh, you configure **agent parameters** that define the size and capabilities of the characters who will use it:

**Agent radius** determines how wide your character is. A radius of 0.5 meters means the character needs at least 1 meter of clearance to pass through a gap. This prevents characters from clipping through walls or getting stuck in tight spaces.

**Agent height** defines how tall your character is. Characters won't path under low ceilings or through doorways too short for them.

**Max climb** sets how high a step the character can climb. Stairs are fine, but a tall ledge might be impassable.

**Max slope** limits how steep a surface can be. Gentle ramps are walkable, but cliffs are not.

The navmesh generator uses these parameters to "rasterize" your level geometry, figuring out what's walkable and what's not. The result is a simplified mesh that's perfect for pathfinding. Small enough to be fast, detailed enough to be accurate.

### Creating a Navigation Mesh

In the Traktor editor:

1. Select your level geometry (the static environment)
2. Create a **NavMesh asset**
3. Configure agent parameters to match your characters
4. Click **Build navigation mesh**

The editor generates the navmesh and displays it as an overlay on your level. You can visualize it to ensure it covers the areas you expect and doesn't include areas where characters shouldn't go (like onto tables or off cliffs, unless that's intentional).

### Adding NavMesh to Your World

Once generated, the navmesh is added to your world as a **world-level component**, not an entity component. You can add it in the editor or at runtime via code:

```cpp
// C++ - Add navigation mesh to world at runtime
world->setComponent(new ai::NavigationMeshComponent(navMeshResource));
```

In the editor, you typically add the NavigationMesh component to the world through the World properties panel. The navmesh exists at the world level and is accessible to all AI characters without needing to reference a specific entity.

## Pathfinding: Finding the Way

Once you have a navmesh, characters can request paths from one point to another. Pathfinding is the process of calculating a valid route that avoids obstacles and stays on walkable surfaces.

Traktor's pathfinding is **asynchronous**. Calculating paths can be expensive, especially over long distances, so the system returns a query result immediately and calculates the path in the background. When the path is ready, you retrieve it and start following it.

### The MoveQuery System

The **MoveQuery** class handles navigation from a start position to an end position. Here's the flow:

```cpp
// C++ - Create move query
Ref<NavMesh> navMesh = ...;
Vector4 start = ...;
Vector4 end = ...;

// Create async pathfinding query
Ref<MoveQueryResult> queryResult = navMesh->createMoveQuery(start, end);

// Check each frame if query is complete
if (queryResult->isReady())
{
    Ref<MoveQuery> moveQuery = queryResult->getMoveQuery();

    // Update to get next waypoint
    Vector4 currentPos = ...;
    Vector4 moveToPos;
    if (moveQuery->update(currentPos, moveToPos, 0.5f))
    {
        // Move character to moveToPos
    }
}
```

The `MoveQueryResult` becomes ready once the path is calculated. Then you repeatedly call `moveQuery->update()` each frame, passing in your current position. It returns the next waypoint to move toward. When you reach the destination, `update()` returns false.

The third parameter (0.5f in the example) is the "path following distance". How far ahead along the path to look for the next waypoint. Larger values make the character cut corners more aggressively; smaller values make them follow the path more precisely.

### NavMesh Queries

Beyond pathfinding, the navmesh supports several useful queries:

**Find closest point on navmesh:** If you spawn a character slightly inside a wall or above the floor, this finds the nearest valid position on the navmesh:

```cpp
Vector4 searchFrom = ...;
Vector4 closestPoint;
if (navMesh->findClosestPoint(searchFrom, closestPoint))
{
    // Snap character to closestPoint
}
```

**Find random point on navmesh:** Perfect for wandering behavior. Pick a random destination and walk to it:

```cpp
Vector4 randomPoint;
if (navMesh->findRandomPoint(randomPoint))
{
    // Use randomPoint for wandering
}
```

**Find random point within radius:** For local wandering around a home position:

```cpp
Vector4 center = ...;
float radius = 10.0f;
Vector4 randomPointInRadius;
if (navMesh->findRandomPoint(center, radius, randomPointInRadius))
{
    // Wander within radius of center
}
```

## Building AI Behaviors with Pathfinding

Let's look at common AI patterns using the navigation system:

### Chase Behavior

An enemy that chases the player:

```lua
import(traktor)

ChaseAI = ChaseAI or class("ChaseAI", world.ScriptComponent)

function ChaseAI:new()
    self._moveQuery = nil
    self._queryResult = nil
    self._speed = 5.0
    self._updatePathTimer = 0
end

function ChaseAI:setup()
    -- Get navmesh from world component
    local navMeshComp = self.owner.world:getComponent(ai.NavigationMeshComponent)
    self._navMesh = navMeshComp:getNavMesh()
end

function ChaseAI:update(contextObject, totalTime, deltaTime)
    -- Update path every 0.5 seconds (player might have moved)
    self._updatePathTimer = self._updatePathTimer + deltaTime
    if self._updatePathTimer > 0.5 then
        self._updatePathTimer = 0
        self:updatePath()
    end

    -- Follow current path
    if self._moveQuery then
        local currentPos = self.owner.transform.translation
        local moveToPos = Vector4()

        if self._moveQuery:update(currentPos, moveToPos, 0.5) then
            -- Move toward waypoint
            local dir = (moveToPos - currentPos):normalized()
            local character = self.owner:getComponent(physics.CharacterComponent)
            if character then
                character:move(dir * self._speed, false)
            end
        end
    end
end

function ChaseAI:updatePath()
    -- Check if pathfinding query completed
    if self._queryResult and self._queryResult:isReady() then
        self._moveQuery = self._queryResult:getMoveQuery()
        self._queryResult = nil
    end

    -- Start new path to player
    if not self._queryResult then
        local player = self.owner.world:getEntity("Player")
        if player then
            local currentPos = self.owner.transform.translation
            local targetPos = player.transform.translation
            self._queryResult = self._navMesh:createMoveQuery(currentPos, targetPos)
        end
    end
end
```

This AI recalculates the path every 0.5 seconds to track the moving player, but doesn't do it every frame (too expensive).

### Patrol Behavior

An AI that patrols between waypoints:

```lua
import(traktor)

PatrolAI = PatrolAI or class("PatrolAI", world.ScriptComponent)

function PatrolAI:new()
    self._waypoints = {
        Vector4(0, 0, 0),
        Vector4(10, 0, 0),
        Vector4(10, 0, 10),
        Vector4(0, 0, 10)
    }
    self._currentWaypoint = 1
    self._moveQuery = nil
    self._queryResult = nil
    self._speed = 3.0
end

function PatrolAI:setup()
    -- Get navmesh from world component
    local navMeshComp = self.owner.world:getComponent(ai.NavigationMeshComponent)
    self._navMesh = navMeshComp:getNavMesh()

    -- Start patrolling to first waypoint
    self:moveToNextWaypoint()
end

function PatrolAI:update(contextObject, totalTime, deltaTime)
    -- Check if pathfinding query completed
    if self._queryResult and not self._moveQuery then
        if self._queryResult:isReady() then
            self._moveQuery = self._queryResult:getMoveQuery()
            self._queryResult = nil
        else
            return -- Still calculating
        end
    end

    if self._moveQuery then
        local currentPos = self.owner.transform.translation
        local moveToPos = Vector4()

        if self._moveQuery:update(currentPos, moveToPos, 0.5) then
            -- Move toward waypoint
            local dir = (moveToPos - currentPos):normalized()
            local character = self.owner:getComponent(physics.CharacterComponent)
            if character then
                character:move(dir * self._speed, false)
            end
        else
            -- Reached waypoint, move to next
            self:moveToNextWaypoint()
        end
    end
end

function PatrolAI:moveToNextWaypoint()
    self._currentWaypoint = self._currentWaypoint + 1
    if self._currentWaypoint > #self._waypoints then
        self._currentWaypoint = 1
    end

    local currentPos = self.owner.transform.translation
    local targetPos = self._waypoints[self._currentWaypoint]
    self._queryResult = self._navMesh:createMoveQuery(currentPos, targetPos)
    self._moveQuery = nil
end
```

### Wander Behavior

An AI that wanders randomly within an area:

```lua
import(traktor)

WanderAI = WanderAI or class("WanderAI", world.ScriptComponent)

function WanderAI:new()
    self._moveQuery = nil
    self._queryResult = nil
    self._speed = 2.0
    self._wanderRadius = 15.0
    self._homePosition = nil
end

function WanderAI:setup()
    -- Get navmesh from world component
    local navMeshComp = self.owner.world:getComponent(ai.NavigationMeshComponent)
    self._navMesh = navMeshComp:getNavMesh()

    self._homePosition = self.owner.transform.translation
    self:pickNewDestination()
end

function WanderAI:update(contextObject, totalTime, deltaTime)
    -- Check if pathfinding query completed
    if self._queryResult and not self._moveQuery then
        if self._queryResult:isReady() then
            self._moveQuery = self._queryResult:getMoveQuery()
            self._queryResult = nil
        else
            return
        end
    end

    if self._moveQuery then
        local currentPos = self.owner.transform.translation
        local moveToPos = Vector4()

        if self._moveQuery:update(currentPos, moveToPos, 0.5) then
            -- Move toward destination
            local dir = (moveToPos - currentPos):normalized()
            local character = self.owner:getComponent(physics.CharacterComponent)
            if character then
                character:move(dir * self._speed, false)
            end
        else
            -- Reached destination, pick new one
            self:pickNewDestination()
        end
    end
end

function WanderAI:pickNewDestination()
    local randomPoint = Vector4()
    if self._navMesh:findRandomPoint(self._homePosition, self._wanderRadius, randomPoint) then
        local currentPos = self.owner.transform.translation
        self._queryResult = self._navMesh:createMoveQuery(currentPos, randomPoint)
        self._moveQuery = nil
    end
end
```

## Best Practices

**Use asynchronous pathfinding.** Always use `createMoveQuery()` which returns a result asynchronously. Synchronous pathfinding can freeze your game for frames at a time, especially over long distances.

**Don't recalculate paths every frame.** Pathfinding is expensive. Recalculate when the target moves significantly (more than a few meters), or on a timer (every 0.5 to 1 second), not every frame.

**Cache query results.** Store the `MoveQuery` object and reuse it until you need a new path. Creating new queries constantly wastes performance.

**Validate positions.** Before requesting a path, use `findClosestPoint()` to ensure both start and end positions are on the navmesh. Paths from or to invalid positions will fail.

**Tune agent parameters carefully.** Match the radius and height to your actual character size. Too small and characters clip through walls; too large and they can't path through doorways.

**Use different navmeshes for different agent sizes.** A human-sized navmesh won't work for a large vehicle or a tiny critter. Generate separate navmeshes with different agent parameters for different character types.

**Handle path failures gracefully.** If pathfinding fails (no valid path exists), have a fallback. Maybe the AI stands still, wanders randomly, or tries a different target.

**Visualize during development.** Enable navmesh visualization in the editor to see exactly where AI can walk. If characters get stuck, the navmesh might be missing connections or have incorrect parameters.

## Debugging AI Navigation

When AI doesn't behave as expected, check these common issues:

**AI gets stuck:** The navmesh might have gaps or disconnected regions. Visualize the navmesh and look for problems. Adjust agent parameters if needed.

**AI paths through walls:** The agent radius is too small. Increase it so the navmesh properly excludes thin walls and obstacles.

**AI can't fit through doorways:** The agent radius is too large. Decrease it or widen your doorways in the level geometry.

**Pathfinding never completes:** The start or end position might be off the navmesh. Use `findClosestPoint()` to snap positions to valid navmesh points before pathfinding.

**AI takes strange routes:** The navmesh might include unwanted areas (like tabletops) or exclude wanted areas (like narrow passages). Adjust generation parameters or manually edit the navmesh.

Log pathfinding operations for debugging:

```lua
log:info("Pathfinding from " .. tostring(startPos) .. " to " .. tostring(endPos))
if queryResult:isReady() then
    log:info("Path found with " .. moveQuery:getWaypointCount() .. " waypoints")
else
    log:info("Pathfinding in progress...")
end
```

## Advanced Techniques

### Combining Behaviors with State Machines

Real AI characters often have multiple behaviors. Use a state machine to switch between them:

```lua
AIController = AIController or class("AIController", world.ScriptComponent)

function AIController:new()
    self._state = "patrol"  -- patrol, chase, flee
    self._patrolAI = PatrolAI:new()
    self._chaseAI = ChaseAI:new()
end

function AIController:update(contextObject, totalTime, deltaTime)
    -- Check conditions and switch states
    local player = self:getPlayer()
    local distToPlayer = (player.transform.translation - self.owner.transform.translation):length()

    if distToPlayer < 10.0 then
        self._state = "chase"
    elseif distToPlayer > 20.0 then
        self._state = "patrol"
    end

    -- Run current state behavior
    if self._state == "patrol" then
        self._patrolAI:update(contextObject, totalTime, deltaTime)
    elseif self._state == "chase" then
        self._chaseAI:update(contextObject, totalTime, deltaTime)
    end
end
```

### Dynamic Obstacles

For dynamic obstacles (moving platforms, destructible walls), you might need to regenerate the navmesh or use local avoidance. This is advanced and typically requires custom solutions.

### Off-Mesh Links

For special connections (jump down from a ledge, climb a ladder), Recast/Detour supports "off-mesh links". Manual connections between navmesh areas. This is configured during navmesh generation.

## See Also

- [World System](world.md) - World-level components and entity management
- [Scripting](scripting.md) - AI logic in Lua
- [Physics System](physics.md) - Character movement with CharacterComponent

## References

- Source: `code/Ai/`
- Recast/Detour: https://github.com/recastnavigation/recastnavigation
