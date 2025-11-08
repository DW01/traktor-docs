---
layout: default
permalink: /engine/physics/
title: Physics
parent: Engine

nav_order: 4
---

# Physics System - Bringing Your World to Life

Physics is what makes your game world feel real. When a player kicks a ball and it bounces off a wall, when a character jumps and gravity pulls them back down, when a car slides around a corner—that's all physics at work. Without it, your game would feel like objects floating in a void, passing through each other like ghosts.

Think of physics as the invisible set of natural laws that govern your virtual world. In the real world, we have gravity, friction, momentum, and collisions. Physics engines simulate these forces in your game, making everything behave the way players expect. Drop something? It falls. Push something? It moves. Crash into a wall? You stop (hopefully before taking damage).

Traktor gives you two powerful physics engines to choose from: **Jolt Physics** (modern and blazingly fast) and **Bullet Physics** (mature and battle-tested). Both provide everything you need to create convincing physical simulations—from simple falling objects to complex vehicle dynamics and character movement.

![TODO: Screenshot showing physics simulation with rigid bodies, constraints, and collision shapes visualized in the editor]

## Understanding the Physics Backend

Traktor doesn't lock you into one physics solution. Instead, it supports two backends, each with its own strengths:

**Jolt Physics** is the recommended choice for new projects. It's a modern engine designed for performance, making excellent use of multi-core processors. If you're building a game with lots of dynamic objects—think physics puzzles, destruction, or open worlds with interactive props—Jolt is your friend.

**Bullet Physics** has been around longer and powers many commercial games. It's feature-rich and extremely stable. If you need specific features that Bullet excels at, or if you're working with existing Bullet-based tools, it's a solid choice.

The good news? Both backends expose the same API in Traktor, so you can switch between them without rewriting your game code. They both provide rigid body dynamics (objects that move and collide realistically), collision detection (knowing when things hit each other), constraints and joints (connecting objects together), character controllers (pre-built movement for players), vehicle simulation (cars, bikes, anything with wheels), and raycasting (detecting what's in a straight line).

## Making Objects Physical

### Rigid Body Component

The heart of physics in Traktor is the **RigidBodyComponent**. Attach this to an entity, and suddenly it obeys the laws of physics. You define properties like mass, friction (how slippery it is), and restitution (how bouncy it is), and the physics engine takes care of the rest.

```cpp
// C++ - Create rigid body
Ref<RigidBodyComponent> body = new RigidBodyComponent();
body->setMass(10.0f);           // Mass in kg (0 = static)
body->setShape(shapeResource);  // Collision shape
body->setFriction(0.5f);
body->setRestitution(0.3f);     // Bounciness
entity->setComponent(body);
```

From Lua, you can apply forces to make objects move. Want to launch something into the air? Apply an upward impulse. Need to simulate wind? Apply a continuous force. It's intuitive and powerful:

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

Bodies come in three flavors: **Dynamic** bodies have mass and are fully simulated—they fall, bounce, and respond to forces. **Static** bodies (mass = 0) never move and have infinite mass—they're perfect for walls, floors, and buildings. **Kinematic** bodies move via animation rather than physics forces—useful for moving platforms or scripted sequences.

### Collision Shapes

Collision shapes define the physical boundaries of your objects. The physics engine uses these shapes to detect when things collide. The simpler the shape, the faster the simulation, so it's common to use simplified shapes rather than matching your visual mesh exactly.

**Box shapes** are perfect for crates, walls, and buildings:

```cpp
BoxShapeDesc boxDesc;
boxDesc.extent = Vector4(1.0f, 1.0f, 1.0f);  // Half-extents
```

**Sphere shapes** are ideal for balls, boulders, and anything roughly round:

```cpp
SphereShapeDesc sphereDesc;
sphereDesc.radius = 1.0f;
```

**Capsule shapes** (a cylinder with rounded ends) are the go-to choice for characters, as they slide smoothly over obstacles:

```cpp
CapsuleShapeDesc capsuleDesc;
capsuleDesc.radius = 0.5f;
capsuleDesc.length = 2.0f;
```

**Mesh shapes** let you use actual triangle meshes for collision. These are expensive to simulate, so they're typically used only for static geometry like terrain and level architecture:

```cpp
MeshShapeDesc meshDesc;
meshDesc.mesh = meshResource;  // Triangle mesh for static geometry
```

For complex objects, **compound shapes** let you combine multiple simple shapes. A car might be a box for the body plus four cylinders for the wheels:

```cpp
CompoundShapeDesc compoundDesc;
compoundDesc.shapes.push_back(shape1);
compoundDesc.shapes.push_back(shape2);
```

## Character Movement Made Easy

### Character Controller

Building responsive character movement from scratch is tricky—you need to handle slopes, stairs, sliding, and collisions just right. That's why Traktor provides a **CharacterComponent** that handles all the common cases for you.

```cpp
// C++ - Create character controller
Ref<CharacterComponent> character = new CharacterComponent();
character->setRadius(0.4f);
character->setHeight(1.8f);
character->setStepHeight(0.3f);
entity->setComponent(character);
```

In your scripts, controlling the character is straightforward. You tell it which direction to move, and it handles collision, sliding along walls, and climbing stairs automatically:

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

## Driving Vehicles

### Vehicle Controller

Want racing games? Off-road adventures? Chase sequences? The **VehicleComponent** simulates realistic vehicle physics, including suspension, tire friction, engine power, and steering. It's sophisticated but accessible:

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

Controlling a vehicle in Lua is as simple as setting throttle, steering, and braking values:

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

## Detecting What's There

### Raycasting

Raycasting is like shining an invisible laser through your world to see what it hits. It's incredibly useful: is there ground beneath the player? What's the player looking at? Is there a clear line of sight to the enemy?

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

Shape casting is like raycasting, but instead of a line, you sweep an entire shape through space. Need to know if there's room for the player to move forward? Sweep their capsule shape along the movement path:

```lua
-- Cast a shape through space
local shape = SphereShape(1.0)  -- Sphere with radius 1.0
local from = Transform(Vector4(0, 10, 0))
local to = Transform(Vector4(0, -10, 0))

local hit = context.physics:shapeCast(shape, from, to)
```

### Overlap Queries

Sometimes you need to know what's in an area right now, not what a ray would hit. Overlap queries find all physics bodies within a region:

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

## Controlling What Collides

### Collision Layers

Not everything should collide with everything else. Players shouldn't collide with triggers, bullets shouldn't collide with the character who fired them, and so on. **Collision layers** organize your physics objects into categories, and you control which categories interact.

```cpp
// Assign to layer
body->setCollisionLayer(CollisionLayer::Player);

// Set what this object collides with
body->setCollisionMask(
    CollisionLayer::World |
    CollisionLayer::Enemy
);
```

Common layers include **World** (static environment), **Player** (the player character), **Enemy** (enemy characters), **Projectile** (bullets and thrown objects), and **Trigger** (volumes that detect overlaps but don't physically block).

### Collision Groups

For even more fine-grained control, you can use collision groups. This lets you set up complex relationships—like "group 1 collides with groups 2 and 3 but not 4":

```cpp
// Set collision group
body->setCollisionGroup(1);  // Group ID

// Group 1 only collides with groups 2 and 3
```

## Connecting Objects Together

### Constraints and Joints

Joints connect physics bodies together, allowing relative motion while maintaining a relationship. Think of a door hinged to a frame, a rope bridge swaying in the wind, or a ragdoll's limbs connected at the joints.

**Hinge joints** rotate around an axis, perfect for doors, wheels, and pendulums:

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

**Ball joints** allow rotation in all directions from a point, like a shoulder or hip in a ragdoll:

```cpp
BallJointDesc ball;
ball.bodyA = upperArm;
ball.bodyB = lowerArm;
ball.anchor = Vector4(0, 0, 0);

Ref<Joint> joint = physicsWorld->createJoint(ball);
```

**Fixed joints** lock two bodies together rigidly, as if they were glued:

```cpp
FixedJointDesc fixed;
fixed.bodyA = bodyA;
fixed.bodyB = bodyB;

Ref<Joint> joint = physicsWorld->createJoint(fixed);
```

## Responding to Collisions

### Event Callbacks

Knowing when things collide is essential for gameplay. When the player touches an enemy, they should take damage. When a ball hits a target, you should play a sound. **Collision events** let you respond to these moments:

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

### Triggers

Triggers are special volumes that detect when objects enter or exit, but don't physically block movement. They're perfect for checkpoints, doors that open when approached, or danger zones that hurt the player:

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

## Tuning the Physics World

### World Configuration

The physics world itself has settings you can tune. Gravity is the most obvious—reduce it for a moon-like feel, or increase it for a heavier, grounded experience:

```cpp
// Set gravity
physicsWorld->setGravity(Vector4(0, -9.81f, 0));

// Simulation frequency
physicsWorld->setSimulationFrequency(60);  // 60 Hz

// Solver iterations (quality vs performance)
physicsWorld->setSolverIterations(10);
```

### Material Properties

Each body has material properties that control how it behaves when it collides. Friction determines how slippery it is—ice has low friction, rubber has high friction. Restitution controls bounciness—a basketball has high restitution, a beanbag has low restitution:

```cpp
// Per-body material
body->setFriction(0.5f);        // 0 = ice, 1 = rubber
body->setRestitution(0.3f);     // 0 = no bounce, 1 = perfect bounce
body->setLinearDamping(0.1f);   // Air resistance
body->setAngularDamping(0.1f);  // Rotational damping
```

## Keeping Performance High

Physics simulation can be expensive, especially with many dynamic objects. Here are the keys to keeping your game running smoothly:

**Use simple shapes whenever possible.** Boxes, spheres, and capsules are orders of magnitude faster than mesh colliders. A character can be a capsule even if they're visually detailed—players won't notice.

**Mark non-moving objects as static** by setting mass to zero. Static bodies are nearly free performance-wise because the engine doesn't simulate them.

**Let bodies sleep.** When objects come to rest, the physics engine automatically puts them to sleep, skipping simulation until they're disturbed again. Don't wake sleeping bodies unnecessarily.

**Use compound shapes** instead of multiple separate bodies when building complex objects. A single compound shape performs better than several individual rigid bodies.

**The broad phase** spatial partitioning system handles large numbers of objects efficiently by quickly eliminating pairs that couldn't possibly collide. You don't need to do anything special—it just works—but it's good to know it's there.

### Profiling

If you're hitting performance issues, check the physics stats to see what's consuming time:

```cpp
// Check physics performance
PhysicsStats stats = physicsWorld->getStats();
log::info << "Active bodies: " << stats.activeBodies << Endl;
log::info << "Collision pairs: " << stats.collisionPairs << Endl;
log::info << "Simulation time: " << stats.simulationTime << " ms" << Endl;
```

## Ragdolls and Advanced Features

For advanced use cases, Traktor supports ragdoll physics—turning a skeletal character into a floppy, physics-driven puppet. This is perfect for dramatic deaths, unconscious characters, or physics-based animation blending:

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

**Scale matters.** Physics engines work best when you use realistic scales—ideally, 1 unit = 1 meter. Going too small or too large can cause numerical instability and weird behavior.

**Avoid tiny objects.** Very small objects (like a 0.01-unit marble) are hard for physics engines to simulate accurately. If you need small visual objects, use larger collision shapes.

**Keep shapes simple.** Don't over-complicate collision shapes. A slightly approximate shape that's simple will both perform better and often behave better than a complex, exact shape.

**Use layers wisely.** Organize your collision filtering logically from the start. It's much easier to set up layers correctly than to debug weird collision issues later.

**Test thoroughly.** Physics can be unpredictable, especially in edge cases. Test scenarios like high speeds, stacked objects, and extreme forces to make sure everything behaves reasonably.

## Debugging

### Visual Debugging

The Traktor editor includes physics visualization. Enable it to see collision shapes as wireframes, contact points where objects touch, joint constraints, and velocity vectors. This makes it immediately obvious when shapes don't match your expectations or when objects are moving unexpectedly.

### Logging

When visual debugging isn't enough, log the physics state:

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
