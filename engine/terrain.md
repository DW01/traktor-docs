---
layout: default
permalink: /engine/terrain/
title: Terrain
parent: Engine

nav_order: 14
---

# Terrain System - Building Vast Outdoor Worlds

Open-world games need something traditional 3D modeling struggles with: enormous, natural-looking outdoor environments. Modeling every hill, valley, and mountain by hand would be impossibly time-consuming, and the resulting geometry would be so dense it would bring even powerful GPUs to their knees. That's where **terrain systems** come in. Specialized tools for creating and rendering large-scale outdoor landscapes efficiently.

Traktor's terrain system uses **heightfield-based terrain**: instead of modeling terrain as arbitrary 3D meshes, you define the ground as a grid where each point has a height value. Think of it like a topographic map. Looking down from above, you see a grid, and each grid cell knows how high the ground is at that point. This simple representation is incredibly efficient: you can represent vast landscapes with manageable memory, render them quickly with LOD (level of detail) systems, and edit them intuitively with sculpting brushes.

But terrain isn't just height data. The system also handles **multi-layer texturing** (blending grass, rock, dirt, and snow based on height, slope, or manual painting), **collision** (so characters and physics objects interact correctly), and **detail objects** (grass, rocks, trees scattered across the surface). The result is rich, believable outdoor environments that run smoothly even when spanning kilometers.

![TODO: Screenshot of terrain editor showing heightfield editing with brushes, layers panel, and painted terrain]

## Understanding Heightfield Terrain

A **heightfield** is a 2D grid where each cell stores a single height value. Imagine looking at terrain from directly above. You see an X-Z plane, and for every (x, z) position, there's a height (y) value. This height determines how high the ground is at that location.

**Resolution** determines detail. A 512x512 heightfield has 262,144 height samples. A 1024x1024 heightfield has over a million samples, providing finer detail but using more memory. The size in world units is separate. A 512x512 heightfield might span 1 kilometer, giving you roughly 2-meter resolution.

**Height precision** is typically stored as 16-bit or 32-bit values. Higher precision allows for subtle elevation changes, while lower precision is sufficient for most terrain.

The beauty of heightfields is their simplicity: **there can only be one height at any (x, z) position**. This means no overhangs, caves, or arches. But for outdoor landscapes (hills, mountains, valleys), it's perfect. The restriction enables massive performance optimizations.

## Creating and Editing Terrain

Terrain is created and sculpted in the Traktor editor:

### Initial Setup

1. **Create a Terrain asset** in the database
2. **Set dimensions** - Choose heightfield resolution (512, 1024, 2048) and world size (100m, 500m, 1km+)
3. **Initialize heightfield** - Start flat or import from a height map image
4. **Assign materials** - Set up base textures and blending

### Sculpting with Brushes

The terrain editor provides brushes for sculpting height:

**Raise/Lower brush** lifts or lowers terrain. Use it for creating hills, digging valleys, or shaping mountain ridges.

**Smooth brush** softens harsh edges and creates gentle slopes. Essential for natural-looking terrain.

**Flatten brush** levels terrain to a specific height. Perfect for creating plateaus, roads, or building sites.

**Noise brush** adds procedural noise for realistic roughness. Terrain that's too smooth looks artificial. Noise adds natural variation.

**Brush parameters:**
- **Size** - How large an area the brush affects
- **Strength** - How much the brush changes terrain per stroke
- **Falloff** - How the brush strength decreases toward edges (sharp or soft)

### Texture Painting

Terrain uses **texture splatting** to blend multiple materials across the surface:

**Base layer** covers the entire terrain. Typically grass or dirt.

**Additional layers** (rock, gravel, sand, snow) are painted on top, blending smoothly with the base.

**Blend modes** determine how layers mix:
- **Height-based** - Rock appears on steep slopes, snow on peaks
- **Slope-based** - Different textures on flat vs steep surfaces
- **Manual painting** - Paint textures exactly where you want them
- **Procedural blending** - Combine rules for natural-looking results

**Material properties** for each layer include:
- Albedo texture (base color)
- Normal map (surface detail)
- Roughness/metallic maps (PBR properties)
- Tiling scale (how many times the texture repeats per meter)

## Terrain Layers: Building Up Complexity

Traktor's terrain supports multiple texture layers blended together:

### Layer 0: Base Layer

The foundation that covers the entire terrain. This should be your most common surface type (usually grass or dirt).

### Layers 1-N: Detail Layers

Additional materials painted or procedurally applied:

**Layer 1: Rock** - Painted on cliff faces and steep slopes
**Layer 2: Path/Dirt** - Manually painted trails
**Layer 3: Snow** - Appears automatically above certain elevation
**Layer 4: Sand** - Near water or in desert regions

### Blending

The shader blends layers based on:
- **Painted masks** - Manual control via painting
- **Height** - Different materials at different elevations
- **Slope** - Steep areas get rock, flat areas get grass
- **Procedural noise** - Adds natural variation to blending

This creates organic transitions. Grass gradually giving way to rock on slopes, snow appearing on peaks, dirt paths winding through grass.

## Level of Detail (LOD)

Terrain can have millions of triangles if rendered at full resolution. **LOD systems** reduce detail at distance, keeping performance smooth:

**Near terrain** (close to camera) renders at full resolution with all detail.

**Medium distance** reduces polygon density. You won't notice the difference from far away.

**Far distance** uses very low-poly representation, maybe just a handful of triangles per terrain tile.

**LOD transitions** are smooth and gradual. The system avoids popping by blending between LOD levels as the camera moves.

**Quadtree structure** organizes terrain into a hierarchy of tiles. The engine only renders visible tiles and automatically picks appropriate LOD for each tile based on distance.

## Physics Integration

Terrain needs to collide correctly with characters, vehicles, and physics objects:

**Collision mesh** is generated from the heightfield, typically at lower resolution than the visual mesh. Physics doesn't need the full detail. A coarser mesh performs better while still providing accurate collision.

**Surface materials** can define physics properties. Grass has different friction than ice, mud slows movement.

```cpp
// C++ - Create terrain with physics
Ref<TerrainComponent> terrain = new TerrainComponent();
terrain->setHeightfield(heightfieldResource);
terrain->setMaterial(materialResource);
terrain->setEnableCollision(true);
entity->setComponent(terrain);
```

From Lua, you can access terrain properties:

```lua
-- Access terrain component
local terrainEntity = world:getEntity("Terrain")
local terrainComp = terrainEntity:getComponent(terrain.TerrainComponent)

-- Get terrain resource (read-only)
local terrainResource = terrainComp.terrain

-- Access terrain properties
local patchCount = terrainComp.patchCount

-- Get heightfield data (read-only)
local heightfield = terrainResource.heightfield
local heightfieldSize = heightfield.size
local worldExtent = heightfield.worldExtent
```

**Note:** The Lua API provides read-only access to terrain data. Height querying and terrain modification are typically done through physics raycasting (for placing objects on terrain) or in the editor. There is no direct `getHeight(x, z)` method exposed to Lua.

## Detail Objects: Grass, Rocks, and Foliage

Large expanses of bare terrain look barren. **Detail objects** add life:

**Grass and small plants** - Scattered densely across terrain using instancing (rendering thousands of grass patches efficiently).

**Rocks and debris** - Procedurally placed based on slope, height, and manual painting.

**Trees and vegetation** - Larger objects placed manually or with distribution tools.

**LOD for details** - Grass fades out at distance, distant rocks use simpler models. This keeps performance high while maintaining visual richness up close.

## Performance Optimization

Terrain can be expensive if not optimized. Here's how Traktor keeps it fast:

**LOD reduces polygon count** drastically at distance. Terrain that would be millions of triangles at full resolution might be only tens of thousands with LOD.

**Frustum culling** only renders terrain tiles visible from the camera. Off-screen terrain is skipped entirely.

**Texture streaming** loads high-res textures only for nearby terrain. Distant terrain uses lower-resolution textures.

**Batch rendering** combines terrain tiles into fewer draw calls, reducing CPU overhead.

**Collision mesh simplification** uses a coarser mesh for physics than for rendering. Visual detail doesn't need to match collision detail.

## Best Practices

**Choose resolution based on size.** For a 1km x 1km terrain, 1024x1024 provides good detail (about 1 meter per sample). For smaller, detailed areas, use higher resolution. For enormous landscapes, use lower resolution with detail maps.

**Use LOD aggressively.** Players don't notice lower detail at distance, but they absolutely notice poor frame rates. Let distant terrain be simple.

**Limit texture layers.** Each additional layer adds shader complexity. 4-6 layers is typically enough. More starts impacting performance with diminishing visual returns.

**Paint blending naturally.** Abrupt transitions look artificial. Use large, soft brushes for gradual blending between materials.

**Test at scale.** Terrain that looks good in the editor might perform poorly in-game with full view distances. Profile on target hardware early.

**Add variation with noise.** Perfectly smooth terrain looks artificial. Add subtle noise for natural roughness.

**Match collision to gameplay needs.** Racing games might use very coarse collision for performance. Slow-paced exploration games might need finer collision for precise foot placement.

## Common Patterns

### Accessing Terrain Data (Read-Only)

```lua
-- Read terrain properties from Lua
import(traktor)

TerrainInfo = TerrainInfo or class("TerrainInfo", world.ScriptComponent)

function TerrainInfo:setup()
    local terrainComp = self.owner:getComponent(terrain.TerrainComponent)
    local terrainResource = terrainComp.terrain

    -- Get heightfield information
    local heightfield = terrainResource.heightfield
    log:info("Heightfield size: " .. tostring(heightfield.size))
    log:info("World extent: " .. tostring(heightfield.worldExtent))

    -- Get terrain configuration
    log:info("Patch count: " .. tostring(terrainComp.patchCount))
    log:info("Detail skip: " .. tostring(terrainResource.detailSkip))
    log:info("Patch dim: " .. tostring(terrainResource.patchDim))
end
```

**Note:** Procedural terrain generation is done in the editor or through custom C++ components, not via Lua scripting.

### Snapping Objects to Terrain Using Physics Raycasting

```lua
-- Place objects on terrain surface using physics raycast
import(traktor)

TerrainPlacer = TerrainPlacer or class("TerrainPlacer", world.ScriptComponent)

function TerrainPlacer:placeObject(x, z)
    -- Get physics manager
    local physicsManager = self.owner.world:getComponent(physics.PhysicsManager)

    -- Raycast from above downward
    local rayStart = Vector4(x, 1000, z)  -- Start high above
    local rayDirection = Vector4(0, -1, 0)  -- Downward
    local maxLength = 2000  -- 2000 units downward

    -- Create query filter (include all groups)
    local queryFilter = physics.QueryFilter()

    -- Perform raycast
    local result = physicsManager:queryRay(rayStart, rayDirection, maxLength, queryFilter, false)

    if result then
        -- Hit terrain, place object at hit position
        local terrainHeight = result.position
        log:info("Terrain height at (" .. x .. ", " .. z .. "): " .. tostring(terrainHeight.y))
        -- ... create entity at terrainHeight
    else
        log:warning("No terrain hit at position")
    end
end
```

**Note:** This uses physics raycasting to find terrain height, which requires the terrain to have collision enabled.

**Note on Terrain Modification:**

Dynamic terrain deformation (creating craters, raising/lowering terrain at runtime) is not supported through the Lua API. Terrain height data is read-only from Lua. Terrain modifications must be done in the editor or through custom C++ components.

## Debugging Terrain

When terrain doesn't look or perform as expected:

**Wireframe mode** shows polygon density. If distant terrain is still dense, LOD might not be working.

**Collision visualization** displays the physics mesh. If physics is wrong, check collision generation settings.

**Texture layer visualization** shows which layers are active where. Useful for debugging blending issues.

**Performance profiling** identifies bottlenecks. If terrain is slow, check polygon count, draw calls, and texture resolution.

Common issues:

**Terrain pops or stutters during movement:** LOD transitions might be too aggressive. Increase blend distances.

**Physics objects fall through terrain:** Collision mesh resolution might be too low, or collision isn't enabled.

**Textures look blurry or sharp:** Check texture tiling scale and mipmap settings.

**Frame rate drops near terrain:** Too much geometry or too many draw calls. Increase LOD aggressiveness or reduce heightfield resolution.

## See Also

- [World System](world.md) - Terrain entities and components
- [Render System](render.md) - Terrain rendering pipeline
- [Physics System](physics.md) - Terrain collision

## References

- Source: `code/Terrain/`
