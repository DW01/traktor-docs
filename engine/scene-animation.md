---
layout: default
permalink: /engine/scene-animation/
title: Scene Animation
parent: Engine

nav_order: 10
---

# Scene Animation

**Note:** Scene animation in Traktor is provided by the [Theater](theater/) module. This page provides an overview and redirects to the detailed Theater documentation.

## Overview

The Theater module provides scene animation capabilities for creating:
- **Cutscenes:** Cinematic sequences with multiple animated entities
- **Camera Animation:** Smooth camera paths and movements
- **Entity Animation:** Animated positions and orientations of scene objects
- **Synchronized Sequences:** Multiple entities animated in perfect sync

## Quick Start

### 1. Add TheaterComponent to Scene

1. Open your scene in the Scene Editor
2. Select the World node
3. Add **Theater Component** to World Components

### 2. Create an Act

1. In TheaterComponent properties, add a new Act
2. Set **Name** (e.g., "INTRO") and **Duration** (e.g., 5.0 seconds)

### 3. Add Tracks

1. Add tracks to your act (one per animated entity)
2. Set **Entity ID** to the entity you want to animate
3. Define the **Transform Path** with keyframes

### 4. Play from Script

```lua
import(traktor)

local tc = self.world.world:getComponent(theater.TheaterComponent)
if tc ~= nil then
    tc:play("INTRO")
end
```

## Key Concepts

### Acts

Named animation sequences with defined start and end times. Each act can contain multiple tracks.

Example acts:
- "INTRO" - Opening cutscene
- "VICTORY" - Victory sequence
- "BOSS_ENTER" - Boss entrance animation

### Tracks

Individual entity animation paths. Each track animates one entity along a spline path with keyframed positions and orientations.

### Transform Paths

Spline-based keyframe animation using TCB (Tension-Continuity-Bias) interpolation for smooth motion between keyframes.

### Look-At

Entities can automatically orient toward other entities while following their animation path, perfect for cameras tracking characters.

## Animation Types

Theater supports:
- **Position Animation:** Move entities along paths
- **Orientation Animation:** Rotate entities over time
- **Look-At Constraints:** Automatically orient toward targets
- **Multi-Entity Sync:** Coordinate multiple entities in one timeline

## Common Use Cases

### Intro Cutscene

```lua
function Game:create(...)
    local tc = self.world.world:getComponent(theater.TheaterComponent)
    if tc ~= nil then
        tc:play("INTRO")
        self._inCutscene = true
    end
end

function Game:update(info)
    local tc = self.world.world:getComponent(theater.TheaterComponent)
    if tc ~= nil and not tc.playing then
        self._inCutscene = false
    end
end
```

### Camera Path

Create a track for the camera entity with a smooth path through your scene, perfect for establishing shots and reveals.

### Synchronized Animation

Add multiple tracks to one act:
- Track 1: Camera movement
- Track 2: Character A walking
- Track 3: Door opening
- Track 4: Light fading

All play simultaneously, perfectly synchronized.

## Editor Integration

The Theater system includes a visual timeline editor (accessed through the Scene Editor) for:
- Creating and editing acts
- Adding and removing tracks
- Setting up transform paths with keyframes
- Previewing animations
- Adjusting spline interpolation

## For More Information

See the comprehensive [Theater](theater/) documentation for:
- Complete API reference
- TransformPath details
- Advanced scripting examples
- Editor workflow
- Performance considerations
- Best practices

## See Also

- **[Theater](theater/)** - Complete Theater documentation
- [Animation](animation.md) - Skeletal character animation
- [Scripting](scripting.md) - Lua scripting guide
- [World](world.md) - Entity-component system

## References

- Theater Module: `code/Theater/`
- Sample: `build/kartong` (uses Theater for intro cutscenes)
