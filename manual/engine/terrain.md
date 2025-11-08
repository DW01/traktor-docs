---
layout: default
permalink: /manual/engine/terrain/
title: Terrain
parent: Engine
grand_parent: Manual
nav_order: 14
---

# Terrain System

The terrain system provides large-scale outdoor environments with heightfields, texturing, and vegetation.

![TODO: Screenshot of terrain editor showing heightfield editing with brushes, layers panel, and painted terrain]

## Overview

Features:
- **Heightfield Terrain:** Large outdoor areas
- **Multi-layer Texturing:** Blend multiple materials
- **LOD:** Level-of-detail for performance
- **Painting:** Texture splatting
- **Collision:** Physics integration

## Terrain Component

```cpp
// C++ - Create terrain
Ref<TerrainComponent> terrain = new TerrainComponent();
terrain->setHeightfield(heightfieldResource);
terrain->setMaterial(materialResource);
entity->setComponent(terrain);
```

## Heightfield Editing

Edit terrain in editor:
1. Create Terrain asset
2. Sculpt height with brushes
3. Paint textures
4. Add details (grass, rocks)
5. Optimize LOD settings

## Terrain Layers

Layer multiple materials:
- Base layer (grass, dirt)
- Detail layers (rocks, paths)
- Blend based on height, slope, or painted

## Performance

- **LOD:** Reduces geometry at distance
- **Culling:** Only renders visible sections
- **Texture Streaming:** Loads textures on demand

## See Also

- [World System](world.md) - Terrain entities
- [Render System](render.md) - Terrain rendering

## References

- Source: `code/Terrain/`
