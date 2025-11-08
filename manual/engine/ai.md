---
layout: default
permalink: /manual/engine/ai/
title: AI
parent: Engine
grand_parent: Manual
nav_order: 15
---

# AI System

The AI system provides navigation mesh support and pathfinding for game AI using the Recast/Detour library.

![TODO: Screenshot showing navigation mesh visualization with AI agents pathfinding around obstacles]

## Overview

Features:
- **Navigation Meshes:** Walkable surface generation using Recast/Detour
- **Pathfinding:** Async pathfinding queries
- **NavMesh Queries:** Find closest points, random points on navmesh

## Navigation Mesh

The NavMesh represents walkable surfaces in your level. It's automatically generated from level geometry.

### NavMeshComponent

Add navigation mesh to an entity:

```cpp
// C++ - Create nav mesh component
Ref<NavMeshComponent> navMesh = new NavMeshComponent();
navMesh->setNavMesh(navMeshResource);
entity->setComponent(navMesh);
```

### Generating Navigation Mesh

In the editor:
1. Select level geometry
2. Create NavMesh asset
3. Configure agent parameters:
   - Agent radius
   - Agent height
   - Max climb
   - Max slope
4. Build navigation mesh

## Pathfinding

### MoveQuery

The `MoveQuery` class handles pathfinding from start to end position:

```cpp
// C++ - Create move query
Ref<NavMesh> navMesh = ...;
Vector4 start = ...;
Vector4 end = ...;

// Create async pathfinding query
Ref<MoveQueryResult> queryResult = navMesh->createMoveQuery(start, end);

// Wait for query to complete
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

**Note:** Pathfinding queries are asynchronous and expensive, so results are returned via `MoveQueryResult` which becomes ready when the path is calculated.

### NavMesh Queries

```cpp
// Find closest point on navmesh
Vector4 searchFrom = ...;
Vector4 closestPoint;
if (navMesh->findClosestPoint(searchFrom, closestPoint))
{
    // Use closestPoint
}

// Find random point on navmesh
Vector4 randomPoint;
if (navMesh->findRandomPoint(randomPoint))
{
    // Use randomPoint for wandering, etc.
}

// Find random point within radius
Vector4 center = ...;
float radius = 10.0f;
Vector4 randomPointInRadius;
if (navMesh->findRandomPoint(center, radius, randomPointInRadius))
{
    // Use randomPointInRadius
}
```

## Lua Scripting Example

```lua
import(traktor)

AIMovement = AIMovement or class("AIMovement", world.ScriptComponent)

function AIMovement:new()
    self._moveQuery = nil
    self._queryResult = nil
    self._target = nil
    self._speed = 5.0
end

function AIMovement:setup()
    -- Get navmesh from world (called after entity is added to world)
    local navMeshEntity = self.owner.world:getEntity("NavMesh")
    local navMeshComp = navMeshEntity:getComponent(ai.NavMeshComponent)
    self._navMesh = navMeshComp:getNavMesh()
end

function AIMovement:setTarget(targetPos)
    local currentPos = self.owner.transform.translation

    -- Start async pathfinding
    self._queryResult = self._navMesh:createMoveQuery(currentPos, targetPos)
    self._target = targetPos
end

function AIMovement:update(contextObject, totalTime, deltaTime)
    if not self._target then
        return
    end

    -- Check if pathfinding is complete
    if self._queryResult and not self._moveQuery then
        if self._queryResult:isReady() then
            self._moveQuery = self._queryResult:getMoveQuery()
            self._queryResult = nil
        else
            return -- Still calculating path
        end
    end

    if self._moveQuery then
        local currentPos = self.owner.transform.translation
        local moveToPos = Vector4()

        -- Get next waypoint
        if self._moveQuery:update(currentPos, moveToPos, 0.5) then
            -- Move toward waypoint
            local dir = (moveToPos - currentPos):normalized()

            local character = self.owner:getComponent(physics.CharacterComponent)
            if character then
                character:move(dir * self._speed, false)
            end
        else
            -- Reached target
            self._moveQuery = nil
            self._target = nil
        end
    end
end
```

## Best Practices

1. **Async Pathfinding:** Always use async queries (`createMoveQuery`) as pathfinding is expensive
2. **Cache Queries:** Don't create new queries every frame
3. **Reuse Paths:** Recalculate paths only when target moves significantly
4. **Check NavMesh Validity:** Ensure points are on navmesh using `findClosestPoint`
5. **Tune Parameters:** Adjust agent radius/height to match your characters

## See Also

- [World System](world.md) - NavMesh components
- [Scripting](scripting.md) - AI logic in Lua
- [Physics System](physics.md) - Character movement

## References

- Source: `code/Ai/`
- Recast/Detour: https://github.com/recastnavigation/recastnavigation
