---
layout: default
permalink: /engine/theater/
title: Theater
parent: Engine

nav_order: 18
---

# Theater - Scene Animation System

The Theater module provides a powerful scene animation system for creating cinematic sequences, cutscenes, camera movements, and choreographed entity animations. It uses a timeline-based approach with Acts and Tracks to animate entities along spline paths.

![TODO: Screenshot of Theater editor showing timeline with acts and tracks]

## Overview

Features:
- **Act-Based System:** Organize animations into named acts (sequences)
- **Multi-Track Animation:** Animate multiple entities simultaneously
- **Spline-Based Paths:** Smooth keyframed animation using TransformPath
- **Look-At Support:** Entities can automatically orient toward other entities
- **Timeline Control:** Play, stop, and check playback status
- **Editor Integration:** Visual timeline editor for creating animations
- **Scripting Access:** Control playback from Lua scripts

## Architecture

The Theater system consists of three main components:

### TheaterComponent

World component that manages all acts and handles playback:

```cpp
class TheaterComponent : public world::IWorldComponent
{
    bool play(const std::wstring& actName);
    void stop();
    bool isPlaying() const;
};
```

Located in: `code/Theater/TheaterComponent.h:31`

### Act

Named animation sequence containing multiple tracks:

```cpp
class Act : public Object
{
    const std::wstring& getName() const;
    float getStart() const;
    float getEnd() const;
};
```

Each act has:
- **Name:** Identifier for playing the act
- **Start/End Times:** Duration of the act in seconds
- **Tracks:** Collection of entity animation tracks

Located in: `code/Theater/Act.h:30`

### Track

Animation path for a single entity:

```cpp
class Track : public Object
{
    const Guid& getEntityId() const;
    const Guid& getLookAtEntityId() const;
    const TransformPath& getPath() const;
};
```

Each track contains:
- **Entity ID:** GUID of the entity to animate
- **Look-At Entity ID:** Optional GUID of entity to orient toward
- **Transform Path:** Keyframed spline path for position and orientation

Located in: `code/Theater/Track.h:29`

## TransformPath - Keyframed Animation

TransformPath provides spline-based interpolation between keyframes:

### Key Structure

```cpp
struct TransformPath::Key
{
    float T;                    // Time (seconds)
    Vector4 tcb;                // Tension/Continuity/Bias for TCB spline
    Vector4 position;           // Position in world space
    Vector4 orientation;        // Orientation (quaternion-like)
    float values[4];            // Custom values (not used by Theater)

    Transform transform() const;  // Convert to Transform
};
```

### Methods

```cpp
// Insert keyframe (automatically sorted by time)
size_t insert(const Key& key);

// Evaluate transform at specific time
Key evaluate(float at, bool closed) const;

// Get keyframe indices
int32_t getClosestKey(float at) const;
int32_t getClosestPreviousKey(float at) const;
int32_t getClosestNextKey(float at) const;

// Path measurements
float measureLength(bool closed) const;
float measureSegmentLength(float from, float to, bool closed, int32_t steps = 1000) const;
float estimateTimeFromDistance(bool closed, float distance, int32_t steps = 1000) const;

// Path operations
void split(float at, TransformPath& outPath1, TransformPath& outPath2) const;
TransformPath geometricNormalized(bool closed) const;

// Time bounds
float getStartTime() const;
float getEndTime() const;

// Keyframe access
size_t size() const;
const Key& get(size_t at) const;
void set(size_t at, const Key& k);
```

Located in: `code/Core/Math/TransformPath.h:33`

## Using Theater from Lua

### Accessing TheaterComponent

Get the TheaterComponent from the world:

```lua
import(traktor)

-- In your stage's create or update method:
local tc = self.world.world:getComponent(theater.TheaterComponent)
if tc ~= nil then
    -- Component is available
end
```

### Playing Acts

Start playing a named act:

```lua
local tc = self.world.world:getComponent(theater.TheaterComponent)
if tc ~= nil then
    -- Play act named "START"
    local success = tc:play("START")
    if success then
        print("Act started")
    else
        print("Act not found")
    end
end
```

### Checking Playback Status

Check if an act is currently playing:

```lua
local tc = self.world.world:getComponent(theater.TheaterComponent)
if tc ~= nil and tc.playing then
    print("Animation is playing")
else
    print("No animation playing")
end
```

### Stopping Playback

Stop the currently playing act:

```lua
local tc = self.world.world:getComponent(theater.TheaterComponent)
if tc ~= nil then
    tc:stop()
end
```

### Example: Cutscene State Machine

Example from kartong showing how to use Theater for intro cutscenes:

```lua
import(traktor)

Game = Game or class("Game", world.IWorldEntityRendererCallback)

function Game:create(environmentClass, environmentObject, worldResourceName)
    -- ... initialization ...

    -- Set initial state to play cutscene
    local tc = self.world.world:getComponent(theater.TheaterComponent)
    if tc ~= nil then
        tc:play("START")
        self._updateFn = Game._updateTheater
    else
        self._updateFn = Game._updateStart
    end
end

function Game:update(info)
    -- Delegate to current update function
    self:_updateFn(info)
end

function Game:_updateTheater(info)
    local tc = self.world.world:getComponent(theater.TheaterComponent)
    if not tc.playing then
        -- Cutscene finished, transition to gameplay
        self._followCamera0:reset()
        self._followCamera1:reset()
        self._updateFn = Game._updateStart
    end
end

function Game:postUpdate(info)
    -- Only update gameplay camera when cutscene is not playing
    local tc = self.world.world:getComponent(theater.TheaterComponent)
    if tc == nil or not tc.playing then
        local dT = info.simulationDeltaTime
        self._followCamera0:update(dT)
        self._followCamera1:update(dT)
    end
end
```

This pattern allows smooth transitions between cutscenes and gameplay.

## Creating Animations in the Editor

### Adding TheaterComponent to a Scene

1. Open your scene in the Scene Editor
2. Select the World node in the hierarchy
3. Add a **Theater Component** to World Components
4. The component will be added to the scene data

### Creating Acts

1. Select the TheaterComponent in the scene
2. In the properties panel, expand the **Acts** section
3. Click **Add** to create a new act
4. Set the act properties:
   - **Name:** Identifier for the act (e.g., "START", "INTRO", "ENDING")
   - **Duration:** Length of the act in seconds (e.g., 5.0)

### Creating Tracks

1. Expand an act in the properties panel
2. In the **Tracks** section, click **Add** to create a new track
3. Set the track properties:
   - **Entity ID:** GUID of the entity to animate (select from scene)
   - **Look-At Entity ID:** Optional GUID of entity to orient toward
   - **Path:** TransformPath defining the animation

### Editing Transform Paths

The path editor allows you to create keyframed animations:

1. Select a track in the properties panel
2. Click **Edit Path** to open the path editor
3. Add keyframes by clicking in the timeline
4. For each keyframe, set:
   - **Time:** When the keyframe occurs (seconds)
   - **Position:** World space position (X, Y, Z)
   - **Orientation:** Rotation (as quaternion or Euler angles)
   - **TCB:** Tension, Continuity, Bias for spline interpolation
5. The path is automatically interpolated between keyframes using TCB spline

### Look-At Feature

When a track has a **Look-At Entity ID** set:
- The animated entity's position follows the path
- The entity's orientation is overridden to face the look-at target
- Useful for cameras tracking moving objects or characters facing each other

Example use cases:
- Camera following a moving character
- Character maintaining eye contact during dialogue
- Spotlight tracking a moving target

## Data Structure Example

Example from kartong scene showing the XML structure:

```xml
<worldComponents>
    <item type="traktor.theater.TheaterComponentData">
        <acts>
            <item type="traktor.theater.ActData">
                <name>START</name>
                <duration>5</duration>
                <tracks>
                    <item type="traktor.theater.TrackData">
                        <entityId>{485FEF1B-7170-F04A-915A-312559ED69CF}</entityId>
                        <lookAtEntityId>{00000000-0000-0000-0000-000000000000}</lookAtEntityId>
                        <path>
                            <keys>
                                <item>
                                    <T>0</T>
                                    <tcb>0 0 0 0</tcb>
                                    <position>-50 10 0 1</position>
                                    <orientation>0 0 0 1</orientation>
                                </item>
                                <item>
                                    <T>5</T>
                                    <tcb>0 0 0 0</tcb>
                                    <position>0 5 -20 1</position>
                                    <orientation>0 0.3 0 1</orientation>
                                </item>
                            </keys>
                        </path>
                    </item>
                </tracks>
            </item>
        </acts>
    </item>
</worldComponents>
```

## Common Use Cases

### 1. Intro Cutscene

Animate camera from initial position to gameplay view:

```lua
function Game:create(...)
    local tc = self.world.world:getComponent(theater.TheaterComponent)
    if tc ~= nil then
        tc:play("INTRO")
    end
end

function Game:update(info)
    local tc = self.world.world:getComponent(theater.TheaterComponent)
    if tc ~= nil and tc.playing then
        -- Wait for cutscene to finish
        return
    end
    -- Normal gameplay update
end
```

### 2. Multiple Cutscenes

Use different acts for different sequences:

```lua
function Game:playCredits()
    local tc = self.world.world:getComponent(theater.TheaterComponent)
    if tc ~= nil then
        tc:play("CREDITS")
    end
end

function Game:playVictory()
    local tc = self.world.world:getComponent(theater.TheaterComponent)
    if tc ~= nil then
        tc:play("VICTORY")
    end
end
```

### 3. Synchronized Entity Animation

Create tracks for multiple entities in the same act:

- Track 1: Camera movement
- Track 2: Character A walking
- Track 3: Character B running
- Track 4: Door opening

All tracks play simultaneously, perfectly synchronized.

### 4. Camera Following Character

Use look-at to keep camera focused on character:

1. Create track for camera with movement path
2. Set **Look-At Entity ID** to character entity
3. Camera will follow path while always facing character

## Best Practices

1. **Plan Your Timeline:** Sketch out the sequence before creating keyframes
2. **Use Descriptive Act Names:** "INTRO", "BOSS_ENTER", "VICTORY" are clear
3. **Keep Acts Focused:** One act per sequence makes them easier to manage
4. **Test Frequently:** Play back animations often during creation
5. **Use Look-At Sparingly:** Manual orientation gives more control for complex movements
6. **TCB Values:** Start with (0,0,0) and adjust if interpolation looks wrong
7. **Keyframe Spacing:** More keyframes = more control, fewer keyframes = smoother curves
8. **Consider Performance:** Many simultaneous tracks can be CPU-intensive

## Performance Notes

- Spline evaluation is fast but not free
- Each track updates one entity's transform per frame
- Look-at calculations add minimal overhead
- Consider limiting number of simultaneous tracks in performance-critical scenes

## Lua API Reference

### TheaterComponent

```lua
-- Access from world
local tc = world:getComponent(theater.TheaterComponent)

-- Methods
tc:play(actName)     -- Play named act, returns boolean success
tc:stop()            -- Stop current act
tc.playing           -- Property: true if act is playing
```

### Example Scene Integration

From kartong's FrontEnd stage:

```lua
import(traktor)

FrontEnd = FrontEnd or class("FrontEnd", world.Stage)

function FrontEnd:create(environmentClass, environmentObject, worldResourceName)
    world.Stage.create(self, environmentClass, environmentObject, worldResourceName)

    -- Play intro animation
    local tc = self.world.world:getComponent(theater.TheaterComponent)
    tc:play("START")
end
```

## See Also

- [World System](world.md) - Entity and component system
- [Scripting](scripting.md) - Lua scripting guide
- [Animation](animation.md) - Skeletal animation system
- Source: `code/Theater/`

## References

- Source: `code/Theater/TheaterComponent.h:31`
- TransformPath: `code/Core/Math/TransformPath.h:33`
- Act: `code/Theater/Act.h:30`
- Track: `code/Theater/Track.h:29`
- Sample: `build/kartong` (uses Theater for intro cutscenes)
