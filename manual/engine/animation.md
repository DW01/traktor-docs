---
layout: default
permalink: /manual/engine/animation/
title: Animation
parent: Engine
grand_parent: Manual
nav_order: 9
---

# Skeletal Animation

The skeletal animation system provides character animation with skeletal meshes, animation state graphs, and inverse kinematics.

![TODO: Screenshot showing a character with skeleton visible, animation timeline, and state graph editor]

## Overview

Features:
- **Skeletal Animation:** Bone-based character animation
- **Animation State Graphs:** State machine-based animation control
- **Pose Controllers:** Various controllers for animation blending and retargeting
- **IK (Inverse Kinematics):** Procedural animation
- **Ragdoll:** Physics-based character animation
- **Cloth Simulation:** Cloth physics
- **Path Following:** Animate entities along paths

## Animated Mesh Component

The `AnimatedMeshComponent` displays and animates skeletal meshes:

```cpp
// C++ - Create animated mesh
Ref<AnimatedMeshComponent> animMesh = new AnimatedMeshComponent();
animMesh->setMesh(skeletalMeshResource);
entity->setComponent(animMesh);
```

## Skeleton Component

The `SkeletonComponent` manages the skeletal hierarchy and pose:

```cpp
// C++ - Create skeleton
Ref<SkeletonComponent> skeleton = new SkeletonComponent();
skeleton->setSkeleton(skeletonResource);
skeleton->setPoseController(poseController);
entity->setComponent(skeleton);
```

## Pose Controllers

Pose controllers determine how animations are played and blended.

### SimpleAnimationController

Plays a single animation:

```cpp
Ref<SimpleAnimationController> controller = new SimpleAnimationController();
controller->setAnimation(animationResource);
controller->setLoop(true);
controller->setPlaybackSpeed(1.0f);
```

### AnimationGraphPoseController

Uses animation state graphs for complex animation logic:

```cpp
Ref<AnimationGraphPoseController> controller = new AnimationGraphPoseController();
controller->setStateGraph(stateGraphResource);
```

### RetargetPoseController

Retargets animation from one skeleton to another:

```cpp
Ref<RetargetPoseController> controller = new RetargetPoseController();
controller->setSourceSkeleton(sourceSkeleton);
controller->setTargetSkeleton(targetSkeleton);
```

## Animation State Graphs

State graphs (not blend trees) provide state machine-based animation control.

### Creating State Graphs

In the editor:
1. Create StateGraph asset
2. Add animation states
3. Define transitions between states
4. Set transition conditions
5. Assign to AnimationGraphPoseController

### State Graph Structure

```
State Graph
├── State: "Idle"
│   ├── Animation: Idle.anim
│   └── Transitions → Walk (when speed > 0.1)
├── State: "Walk"
│   ├── Animation: Walk.anim
│   ├── Transitions → Idle (when speed <= 0.1)
│   └── Transitions → Run (when speed > 5.0)
└── State: "Run"
    ├── Animation: Run.anim
    └── Transitions → Walk (when speed <= 5.0)
```

### Controlling from Scripts

```lua
import(traktor)

AnimationController = AnimationController or class("AnimationController", world.ScriptComponent)

function AnimationController:update(contextObject, totalTime, deltaTime)
    local skeleton = self.owner:getComponent(animation.SkeletonComponent)
    local controller = skeleton:getPoseController()

    -- Set state graph parameters
    local velocity = self:getVelocity()
    local speed = velocity:length()

    controller:setParameter("Speed", speed)
    controller:setParameter("Grounded", self._isGrounded)

    -- Trigger state changes
    if contextObject.input:wasKeyPressed("Space") then
        controller:trigger("Jump")
    end
end
```

## Inverse Kinematics (IK)

The `IKComponent` provides procedural animation using inverse kinematics:

```cpp
// C++ - Create IK component
Ref<IKComponent> ik = new IKComponent();
ik->setTargetJoint("LeftHand");
ik->setTarget(targetPosition);
entity->setComponent(ik);
```

Common IK uses:
- **Foot Placement:** Keep feet on uneven ground
- **Hand Targets:** Reach for objects
- **Look At:** Head/eye tracking

```lua
-- Lua - IK example for looking at target (in update function)
local ik = self.owner:getComponent(animation.IKComponent)
local target = self.owner.world:getEntity("Player").transform.translation
ik:setTarget(target)
```

## Ragdoll Physics

Ragdoll provides physics-based character animation:

```cpp
// C++ - Enable ragdoll
Ref<RagdollComponent> ragdoll = new RagdollComponent();
ragdoll->setSkeleton(skeleton);
ragdoll->setRagdollData(ragdollData);
entity->setComponent(ragdoll);

// Activate ragdoll
ragdoll->setEnabled(true);
```

Use cases:
- Character death
- Impacts and collisions
- Procedural reactions

## Additional Animation Components

### Cloth Component

Simulates cloth physics:

```cpp
Ref<ClothComponent> cloth = new ClothComponent();
cloth->setMesh(clothMesh);
entity->setComponent(cloth);
```

### Path Component

Animates entities along spline paths:

```cpp
Ref<PathComponent> path = new PathComponent();
path->setPath(pathResource);
path->setDuration(5.0f);  // Time to traverse path
entity->setComponent(path);
```

### Rotator Components

Simple procedural rotation animations:

- **RotatorComponent:** Continuous rotation
- **PendulumComponent:** Pendulum motion
- **WobbleComponent:** Wobble/shake effect
- **OrientateComponent:** Orient toward target

```cpp
Ref<RotatorComponent> rotator = new RotatorComponent();
rotator->setAxis(Vector4(0, 1, 0));  // Rotate around Y axis
rotator->setSpeed(1.0f);  // Radians per second
entity->setComponent(rotator);
```

### Boids Component

Flock/swarm simulation:

```cpp
Ref<BoidsComponent> boids = new BoidsComponent();
entity->setComponent(boids);
```

## Animation Data

### Animation Class

The `Animation` class stores keyframed poses:

```cpp
// Animation structure
class Animation
{
    struct KeyPose
    {
        float at;      // Time
        Pose pose;     // Skeletal pose
    };

    // Get pose at specific time
    bool getPose(float time, Pose& outPose) const;
};
```

### Skeleton Class

Defines the bone hierarchy:

```cpp
class Skeleton
{
    // Get joint by name
    const Joint* findJoint(const std::wstring& name) const;

    // Get number of joints
    uint32_t getJointCount() const;
};
```

## Best Practices

1. **Use State Graphs:** For complex animation logic, use animation state graphs
2. **Optimize Bones:** Minimize bone count for performance
3. **LOD Animations:** Use simpler animations for distant characters
4. **Cache Controllers:** Don't create new controllers every frame
5. **IK Sparingly:** IK is more expensive than keyframed animation

## See Also

- [World System](world.md) - Animation components
- [Scene Animation](scene-animation.md) - Object/camera animation
- [Physics System](physics.md) - Ragdoll physics

## References

- Source: `code/Animation/`
