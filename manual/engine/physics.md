---
layout: default
permalink: /manual/engine/physics/
title: Physics
parent: Engine
grand_parent: Manual
nav_order: 4
---

# Physics System

The Traktor physics system integrates both **Jolt** and **Bullet** physics engines, providing rigid body dynamics, collision detection, and character/vehicle controllers.

![TODO: Screenshot showing physics simulation with rigid bodies, constraints, and collision shapes visualized in the editor]

## Overview

Traktor supports two physics backends:

- **Jolt Physics:** Modern, high-performance physics engine (recommended)
- **Bullet Physics:** Mature, feature-rich physics engine

Both backends provide:
- Rigid body dynamics
- Collision detection
- Constraints and joints
- Character controllers
- Vehicle simulation
- Raycasting and shape casting

## Physics Components

### Rigid Body Component

Adds physics simulation to an entity:

```cpp
// C++ - Create rigid body
Ref<RigidBodyComponent> body = new RigidBodyComponent();
body->setMass(10.0f);           // Mass in kg (0 = static)
body->setShape(shapeResource);  // Collision shape
body->setFriction(0.5f);
body->setRestitution(0.3f);     // Bounciness
entity->setComponent(body);
```

```lua
-- Lua - Access rigid body
local body = self.owner:getComponent(physics.RigidBodyComponent)

-- Apply forces
body:applyForce(Vector4(0, 100, 0))          -- Continuous force
body:applyImpulse(Vector4(10, 0, 0))         -- Instant impulse
body:applyTorque(Vector4(0, 50, 0))          -- Rotational force

-- Set velocities
body:setLinearVelocity(Vector4(5, 0, 0))
body:setAngularVelocity(Vector4(0, 1, 0))

-- Get velocities
local vel = body:getLinearVelocity()
local angVel = body:getAngularVelocity()
```

**Body Types:**
- **Dynamic (mass > 0):** Fully simulated, affected by forces
- **Static (mass = 0):** Never moves, infinite mass
- **Kinematic:** Moves via animation, not forces

### Collision Shapes

Common collision shapes:

**Box Shape:**
```cpp
BoxShapeDesc boxDesc;
boxDesc.extent = Vector4(1.0f, 1.0f, 1.0f);  // Half-extents
```

**Sphere Shape:**
```cpp
SphereShapeDesc sphereDesc;
sphereDesc.radius = 1.0f;
```

**Capsule Shape:**
```cpp
CapsuleShapeDesc capsuleDesc;
capsuleDesc.radius = 0.5f;
capsuleDesc.length = 2.0f;
```

**Mesh Shape (Static):**
```cpp
MeshShapeDesc meshDesc;
meshDesc.mesh = meshResource;  // Triangle mesh for static geometry
```

**Compound Shape:**
Combine multiple shapes:
```cpp
CompoundShapeDesc compoundDesc;
compoundDesc.shapes.push_back(shape1);
compoundDesc.shapes.push_back(shape2);
```

## Character Controller

Pre-built character physics controller:

```cpp
// C++ - Create character controller
Ref<CharacterComponent> character = new CharacterComponent();
character->setRadius(0.4f);
character->setHeight(1.8f);
character->setStepHeight(0.3f);
entity->setComponent(character);
```

```lua
-- Lua - Control character (in update function)
local character = self.owner:getComponent(physics.CharacterComponent)

-- Movement
local moveDir = Vector4(1, 0, 0)  -- Move right
local speed = 5.0
character:move(moveDir * speed, false)  -- move(motion, vertical)

-- Jumping (assuming contextObject from update function)
if contextObject.input:isKeyDown("Space") and character:grounded() then
    character:jump()  -- Jump
end

-- Query state
local isGrounded = character:grounded()
local velocity = character:getVelocity()
```

## Vehicle Controller

Advanced vehicle simulation:

```cpp
// C++ - Create vehicle
Ref<VehicleComponent> vehicle = new VehicleComponent();
vehicle->setChassisMass(1500.0f);        // kg
vehicle->setMaxEngineForce(3000.0f);     // N
vehicle->setMaxBreakingForce(1000.0f);   // N

// Add wheels
WheelDesc wheel;
wheel.position = Vector4(1.0f, 0.0f, 1.5f);
wheel.radius = 0.4f;
wheel.suspensionLength = 0.3f;
vehicle->addWheel(wheel);
// ... add more wheels

entity->setComponent(vehicle);
```

```lua
-- Lua - Control vehicle (in update function)
local vehicle = self.owner:getComponent(physics.VehicleComponent)

-- Acceleration/braking (assuming contextObject from update function)
local throttle = 0.0
if contextObject.input:isKeyDown("W") then throttle = 1.0 end
if contextObject.input:isKeyDown("S") then throttle = -1.0 end
vehicle.engineThrottle = throttle

-- Steering
local steerAngle = 0.0
if contextObject.input:isKeyDown("A") then steerAngle = -0.5 end  -- Radians
if contextObject.input:isKeyDown("D") then steerAngle = 0.5 end
vehicle.steerAngle = steerAngle

-- Braking
local brake = contextObject.input:isKeyDown("Space") and 1.0 or 0.0
vehicle.breaking = brake

-- Query state
local velocity = vehicle:getVelocity()
local torque = vehicle:getEngineTorque()
```

## Raycasting and Queries

### Raycasting

```lua
-- Ray cast from origin in direction
local origin = Vector4(0, 10, 0)
local direction = Vector4(0, -1, 0)
local maxDistance = 100.0

local hit = context.physics:rayCast(origin, direction, maxDistance)

if hit then
    log:info("Hit entity: " .. hit.entity:getName())
    log:info("Hit position: " .. tostring(hit.position))
    log:info("Hit normal: " .. tostring(hit.normal))
    log:info("Hit distance: " .. hit.distance)
end
```

### Shape Casting

```lua
-- Cast a shape through space
local shape = SphereShape(1.0)  -- Sphere with radius 1.0
local from = Transform(Vector4(0, 10, 0))
local to = Transform(Vector4(0, -10, 0))

local hit = context.physics:shapeCast(shape, from, to)
```

### Overlap Queries

```lua
-- Find all bodies overlapping a region
local center = Vector4(0, 0, 0)
local radius = 10.0

local bodies = context.physics:querySphere(center, radius)

for i, body in ipairs(bodies) do
    local entity = body:getEntity()
    log:info("Found: " .. entity:getName())
end
```

## Collision Filtering

### Layers

Organize physics objects into layers:

```cpp
// Assign to layer
body->setCollisionLayer(CollisionLayer::Player);

// Set what this object collides with
body->setCollisionMask(
    CollisionLayer::World |
    CollisionLayer::Enemy
);
```

**Common Layers:**
- `World` - Static environment
- `Player` - Player character
- `Enemy` - Enemy characters
- `Projectile` - Bullets, projectiles
- `Trigger` - Trigger volumes

### Collision Groups

Fine-grained control over collisions:

```cpp
// Set collision group
body->setCollisionGroup(1);  // Group ID

// Group 1 only collides with groups 2 and 3
```

## Constraints and Joints

### Hinge Joint

Rotational joint (door, wheel):

```cpp
HingeJointDesc hinge;
hinge.bodyA = door;
hinge.bodyB = frame;
hinge.anchor = Vector4(0, 0, 0);
hinge.axis = Vector4(0, 1, 0);  // Rotate around Y
hinge.minAngle = -90.0f;
hinge.maxAngle = 90.0f;

Ref<Joint> joint = physicsWorld->createJoint(hinge);
```

### Ball Joint

Point-to-point constraint (ragdoll):

```cpp
BallJointDesc ball;
ball.bodyA = upperArm;
ball.bodyB = lowerArm;
ball.anchor = Vector4(0, 0, 0);

Ref<Joint> joint = physicsWorld->createJoint(ball);
```

### Fixed Joint

Lock two bodies together:

```cpp
FixedJointDesc fixed;
fixed.bodyA = bodyA;
fixed.bodyB = bodyB;

Ref<Joint> joint = physicsWorld->createJoint(fixed);
```

## Collision Events

### Event Callbacks

Handle collision events:

```lua
import(traktor)

CollisionScript = CollisionScript or class("CollisionScript", world.ScriptComponent)

function CollisionScript:new()
    -- Register for collision events
    self.owner:addEventListener("Collision", self.onCollision)
end

function CollisionScript:onCollision(event)
    -- Get other entity
    local other = event.other

    log:info("Collided with: " .. other:getName())

    -- Access collision details
    local impulse = event.impulse
    local position = event.position
    local normal = event.normal

    -- Respond to collision
    if other:getName() == "Enemy" then
        self:takeDamage(10)
    end
end

function CollisionScript:takeDamage(amount)
    -- Damage logic
end
```

## Triggers

Trigger volumes detect overlaps without physical collision:

```cpp
// Create trigger
Ref<RigidBodyComponent> trigger = new RigidBodyComponent();
trigger->setMass(0);              // Static
trigger->setTrigger(true);        // Mark as trigger
trigger->setShape(boxShape);
```

```lua
import(traktor)

TriggerScript = TriggerScript or class("TriggerScript", world.ScriptComponent)

function TriggerScript:new()
    self.owner:addEventListener("TriggerEnter", self.onEnter)
    self.owner:addEventListener("TriggerExit", self.onExit)
end

function TriggerScript:onEnter(event)
    log:info(event.other:getName() .. " entered trigger")
end

function TriggerScript:onExit(event)
    log:info(event.other:getName() .. " exited trigger")
end
```

## Physics Settings

### World Configuration

```cpp
// Set gravity
physicsWorld->setGravity(Vector4(0, -9.81f, 0));

// Simulation frequency
physicsWorld->setSimulationFrequency(60);  // 60 Hz

// Solver iterations (quality vs performance)
physicsWorld->setSolverIterations(10);
```

### Material Properties

```cpp
// Per-body material
body->setFriction(0.5f);        // 0 = ice, 1 = rubber
body->setRestitution(0.3f);     // 0 = no bounce, 1 = perfect bounce
body->setLinearDamping(0.1f);   // Air resistance
body->setAngularDamping(0.1f);  // Rotational damping
```

## Performance

### Optimization Tips

1. **Use Simple Shapes:** Prefer boxes, spheres, capsules over mesh colliders
2. **Static Geometry:** Mark non-moving objects as static (mass = 0)
3. **Sleeping:** Bodies automatically sleep when at rest
4. **Broad Phase:** Spatial partitioning handles many objects efficiently
5. **Compound Shapes:** Better than individual bodies for complex objects

### Profiling

```cpp
// Check physics performance
PhysicsStats stats = physicsWorld->getStats();
log::info << "Active bodies: " << stats.activeBodies << Endl;
log::info << "Collision pairs: " << stats.collisionPairs << Endl;
log::info << "Simulation time: " << stats.simulationTime << " ms" << Endl;
```

## Ragdolls

Create ragdoll physics from skeletal meshes:

```cpp
// Create ragdoll from skeleton
RagdollDesc ragdollDesc;
ragdollDesc.skeleton = skeletonResource;

// Configure bones as rigid bodies with joints
Ref<Ragdoll> ragdoll = physicsWorld->createRagdoll(ragdollDesc);

// Apply to character
character->enableRagdoll(ragdoll);
```

## Best Practices

1. **Scale Appropriately:** Use realistic scales (1 unit = 1 meter)
2. **Avoid Small Objects:** Physics struggles with very small or very large objects
3. **Limit Complexity:** Don't over-complicate collision shapes
4. **Use Layers:** Organize collision filtering logically
5. **Test Thoroughly:** Physics can be unpredictable, test edge cases

## Debugging

### Visual Debugging

Enable physics visualization in editor:
- Collision shapes (wireframe)
- Contact points
- Joint constraints
- Velocity vectors

### Logging

```lua
-- Debug physics
local body = self.owner:getComponent(physics.RigidBodyComponent)
log:info("Position: " .. tostring(body:getPosition()))
log:info("Velocity: " .. tostring(body:getLinearVelocity()))
log:info("Is sleeping: " .. tostring(body:isSleeping()))
```

## See Also

- [World System](world.md) - Entity and component system
- [Scripting](scripting.md) - Controlling physics from Lua
- [Animation](animation.md) - Combining physics with animation

## References

- Source: `code/Physics/`
- Jolt Physics: https://github.com/jrouwe/JoltPhysics
- Bullet Physics: https://pybullet.org/
