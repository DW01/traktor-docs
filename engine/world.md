---
layout: default
permalink: /engine/world/
title: World
parent: Engine

nav_order: 3
---

# World System - Building Your Game Universe

Imagine your game as a stage where actors perform. The **World System** is that stage. It's where everything in your game exists and interacts. Whether it's the player character, enemies, power-ups, or environmental objects, they all live in the world.

Traktor uses an **Entity-Component** architecture, which is a fancy way of saying "build complex things from simple, reusable pieces." Think of it like building with Lego: instead of having one giant, custom-molded block for each type of object, you have small,standard pieces that click together in different ways.

![TODO: Diagram showing World containing multiple Entities, each Entity containing multiple Components]

## Understanding the Pattern

The World System has three key players:

**Worlds** are containers. They hold all the entities in a scene and coordinate their updates. When you load a level, you're loading a World. When the game ticks forward one frame, the World updates everything inside it.

**Entities** are game objects. The player, an enemy, a crate, a light source, whatever. But here's the twist: entities themselves are quite simple. They're basically just a position in space (a transform) and a collection of components. An entity without components is just an invisible point floating in your scene.

**Components** are where the magic happens. They're modular pieces of functionality that you attach to entities. Want an entity to be visible? Add a MeshComponent. Want it to be affected by physics? Add a RigidBodyComponent. Want it to think and make decisions? Add a ScriptComponent. Each component adds one capability, and by combining components, you create rich, complex game objects.

This approach. Composition over inheritance. Is incredibly powerful. Instead of writing a new class for every type of enemy or object, you just mix and match existing components in new ways.

## Core Concepts

### World

The `World` class is the container for all entities in your scene:

```cpp
class World : public Object
{
public:
    // Create world
    explicit World(
        resource::IResourceManager* resourceManager,
        render::IRenderSystem* renderSystem
    );

    // Entity management
    void addEntity(Entity* entity);
    void removeEntity(Entity* entity);

    // Entity queries
    Entity* getEntity(const Guid& id) const;
    Entity* getEntity(const std::wstring& name, int32_t index = 0) const;
    RefArray<Entity> getEntities(const std::wstring& name) const;
    RefArray<Entity> getEntitiesWithinRange(const Vector4& position, float range) const;

    // Update all entities
    void update(const UpdateParams& update);

    // World components
    void setComponent(IWorldComponent* component);
    template<typename T> T* getComponent() const;
};
```

**World Components** are global systems that operate on the entire world (e.g., physics world, lighting system).

### Entity

Entities are the game objects in your world:

```cpp
class Entity : public Object
{
public:
    // Create entity
    Entity(
        const Guid& id,
        const std::wstring_view& name,
        const Transform& transform,
        const EntityState& state = EntityState()
    );

    // Transform
    void setTransform(const Transform& transform);
    Transform getTransform() const;

    // State management
    void setVisible(bool visible);
    bool isVisible() const;
    void setDynamic(bool dynamic);
    bool isDynamic() const;

    // Component management
    void setComponent(IEntityComponent* component);
    template<typename T> T* getComponent() const;
    const RefArray<IEntityComponent>& getComponents() const;

    // Update
    void update(const UpdateParams& update);

    // Hierarchy
    const Guid& getId() const;
    const std::wstring& getName() const;
    World* getWorld() const;
};
```

**Entity State Flags:**
- **Visible:** Entity is rendered
- **Dynamic:** Entity can move/change
- **Locked:** Entity cannot be modified (editor)

### Components

Components add functionality to entities. All components derive from `IEntityComponent`:

```cpp
class IEntityComponent : public Object
{
    virtual void destroy() = 0;
    virtual void setOwner(Entity* owner) = 0;
    virtual void setTransform(const Transform& transform) = 0;
    virtual Aabb3 getBoundingBox() const = 0;
    virtual void update(const UpdateParams& update) = 0;
};
```

## Common Components

### Transform Component

Every entity has a transform (built-in):

```cpp
// Set position, rotation, scale
Transform transform(
    Vector4(x, y, z, 1.0f),     // Position
    Quaternion::identity(),      // Rotation
    Vector4(1.0f, 1.0f, 1.0f)   // Scale
);
entity->setTransform(transform);

// Get transform
Transform current = entity->getTransform();
```

### Mesh Component

Renders 3D geometry:

```cpp
Ref<MeshComponent> meshComp = new MeshComponent();
meshComp->setMesh(meshResource);
meshComp->setMaterial(materialResource);
entity->setComponent(meshComp);
```

### Script Component

Attaches Lua scripts for behavior:

```cpp
Ref<ScriptComponent> scriptComp = new ScriptComponent();
scriptComp->setScript(scriptResource);
entity->setComponent(scriptComp);
```

### Light Component

Provides illumination:

```cpp
Ref<LightComponent> lightComp = new LightComponent();
lightComp->setType(LightType::Directional);
lightComp->setColor(Color4f(1.0f, 1.0f, 1.0f));
lightComp->setIntensity(1.0f);
entity->setComponent(lightComp);
```

**Light Types:**
- **Directional:** Sun-like lighting (e.g., for outdoor scenes)
- **Point:** Omnidirectional light source (e.g., light bulb)
- **Spot:** Cone-shaped light (e.g., flashlight)

### Rigid Body Component

Adds physics simulation:

```cpp
Ref<RigidBodyComponent> bodyComp = new RigidBodyComponent();
bodyComp->setMass(10.0f);
bodyComp->setShape(shapeResource);
entity->setComponent(bodyComp);
```

### Audio Component

Plays sound effects and music:

```cpp
Ref<AudioComponent> audioComp = new AudioComponent();
audioComp->setSound(soundResource);
audioComp->setLoop(true);
audioComp->play();
entity->setComponent(audioComp);
```

## Working with Entities

### Creating Entities in Code

```cpp
// Create entity
Ref<Entity> entity = new Entity(
    Guid::create(),              // Unique ID
    L"MyEntity",                 // Name
    Transform::identity(),       // Transform
    EntityState::All             // State (visible, dynamic)
);

// Add components
entity->setComponent(new MeshComponent(...));
entity->setComponent(new ScriptComponent(...));

// Add to world
world->addEntity(entity);
```

### Finding Entities

```cpp
// By ID
Ref<Entity> entity = world->getEntity(guid);

// By name
Ref<Entity> entity = world->getEntity(L"Player");

// Multiple entities with same name
RefArray<Entity> enemies = world->getEntities(L"Enemy");

// Within range
RefArray<Entity> nearby = world->getEntitiesWithinRange(
    playerPosition,
    100.0f  // Radius
);
```

### Accessing Components

```cpp
// Get specific component
MeshComponent* mesh = entity->getComponent<MeshComponent>();
if (mesh)
{
    // Use mesh component
}

// Iterate all components
const RefArray<IEntityComponent>& components = entity->getComponents();
for (auto* component : components)
{
    // Process component
}
```

### Modifying Entities

```cpp
// Change transform
Transform t = entity->getTransform();
t.setTranslation(Vector4(10, 0, 0, 1));
entity->setTransform(t);

// Change visibility
entity->setVisible(false);

// Add new component
entity->setComponent(new AudioComponent(...));
```

### Removing Entities

```cpp
world->removeEntity(entity);
```

## Entity Lifecycle

![TODO: Flowchart showing: Create → Add to World → Update Loop (update components) → Remove from World → Destroy]

1. **Create** - Entity is instantiated
2. **Add to World** - Entity becomes active
3. **Update** - Components are updated each frame
4. **Remove from World** - Entity becomes inactive
5. **Destroy** - Entity resources are cleaned up

### Update Cycle

```cpp
void World::update(const UpdateParams& update)
{
    // Update all entities
    for (Entity* entity : m_entities)
    {
        entity->update(update);
    }
}

void Entity::update(const UpdateParams& update)
{
    // Update all components
    for (IEntityComponent* component : m_components)
    {
        component->update(update);
    }
}
```

**UpdateParams** contains:
```cpp
struct UpdateParams
{
    float totalTime;        // Total elapsed time
    float deltaTime;        // Frame delta time
    float simulationDelta;  // Physics simulation delta
    // ... additional fields
};
```

## Entity Hierarchy

While Traktor entities don't have built-in parent-child relationships in the traditional scene graph sense, you can implement hierarchies using:

1. **Component-based relationships:** Use components to track parent/child
2. **Transform parenting:** Manually propagate transforms
3. **Scripting:** Implement custom hierarchy logic in Lua

## Creating Custom Components

### Component Data

First, create a data class for the editor (derives from `IEntityComponentData`):

```cpp
class MyComponentData : public IEntityComponentData
{
    T_RTTI_CLASS;
public:
    // Properties (serialized)
    float m_speed = 1.0f;
    int32_t m_health = 100;

    virtual int32_t getOrdinal() const override;
    virtual void setTransform(const EntityData* owner, const Transform& transform) override;
    virtual void serialize(ISerializer& s) override;
};
```

### Component Runtime

Then create the runtime component (derives from `IEntityComponent`):

```cpp
class MyComponent : public IEntityComponent
{
    T_RTTI_CLASS;
public:
    MyComponent(float speed, int32_t health)
    :   m_speed(speed)
    ,   m_health(health)
    {
    }

    virtual void destroy() override
    {
        // Cleanup
    }

    virtual void setOwner(Entity* owner) override
    {
        m_owner = owner;
    }

    virtual void update(const UpdateParams& update) override
    {
        // Update logic here
        Vector4 pos = m_owner->getTransform().translation();
        pos += Vector4(m_speed * update.deltaTime, 0, 0);

        Transform t = m_owner->getTransform();
        t.setTranslation(pos);
        m_owner->setTransform(t);
    }

    virtual Aabb3 getBoundingBox() const override
    {
        return Aabb3();
    }

    virtual void setTransform(const Transform& transform) override
    {
        // Handle transform changes
    }

private:
    Entity* m_owner = nullptr;
    float m_speed;
    int32_t m_health;
};
```

### Component Factory

Register your component so it can be instantiated:

```cpp
class MyComponentFactory : public IEntityComponentFactory
{
    T_RTTI_CLASS;
public:
    virtual Ref<IEntityComponent> createComponent(
        const IEntityComponentData* componentData,
        ...
    ) const override
    {
        const MyComponentData* data = checked_type_cast<const MyComponentData*>(componentData);
        return new MyComponent(data->m_speed, data->m_health);
    }
};
```

## World Components

World components are global systems that operate on the entire world:

```cpp
class IWorldComponent : public Object
{
    virtual void destroy() = 0;
    virtual void update(World* world, const UpdateParams& update) = 0;
};
```

**Example: Physics World Component**

```cpp
class PhysicsWorldComponent : public IWorldComponent
{
public:
    virtual void update(World* world, const UpdateParams& update) override
    {
        // Step physics simulation
        m_physicsWorld->simulate(update.simulationDelta);

        // Sync entity transforms from physics
        for (Entity* entity : world->getEntities())
        {
            if (RigidBodyComponent* body = entity->getComponent<RigidBodyComponent>())
            {
                Transform t = body->getPhysicsTransform();
                entity->setTransform(t);
            }
        }
    }

private:
    PhysicsWorld* m_physicsWorld;
};
```

## Best Practices

### 1. Favor Composition

Instead of:
```cpp
class PlayerEntity : public Enemy, public Character, public Interactive { };
```

Use:
```cpp
Ref<Entity> player = new Entity(...);
player->setComponent(new CharacterController());
player->setComponent(new HealthComponent());
player->setComponent(new InventoryComponent());
```

### 2. Single Responsibility Components

Keep components focused:
- ✓ `HealthComponent` - Manages health
- ✓ `DamageComponent` - Handles damage
- ✗ `PlayerComponent` - Does everything

### 3. Use Entity Queries Wisely

```cpp
// Cache results when possible
RefArray<Entity> enemies = world->getEntities(L"Enemy");

// Avoid expensive queries every frame
for (int i = 0; i < 1000; i++)
{
    world->getEntitiesWithinRange(...);  // Slow!
}
```

### 4. Clean Up Properly

```cpp
void MyComponent::destroy()
{
    // Release resources
    m_resource = nullptr;

    // Call base
    IEntityComponent::destroy();
}
```

### 5. Handle Null Components

```cpp
// Always check for null
if (MeshComponent* mesh = entity->getComponent<MeshComponent>())
{
    // Safe to use mesh
}
```

## Performance Tips

### Spatial Queries

Use spatial data structures for efficient entity queries:

```cpp
// Efficient range query
RefArray<Entity> nearby = world->getEntitiesWithinRange(position, radius);
```

### Component Updates

Only update components that need it:

```cpp
virtual void update(const UpdateParams& update) override
{
    if (!m_active)
        return;  // Skip inactive components

    // Update logic
}
```

### Batch Operations

Process similar entities together:

```cpp
// Get all entities with physics
RefArray<Entity> dynamicEntities;
for (Entity* entity : world->getEntities())
{
    if (entity->isDynamic())
        dynamicEntities.push_back(entity);
}

// Process in batch
for (Entity* entity : dynamicEntities)
{
    // Physics update
}
```

## Integration with Other Systems

### With Scripting

```lua
-- From Lua script
local entity = world:getEntity("Player")
local health = entity:getComponent(game.HealthComponent)
health:damage(10)
```

### With Physics

```cpp
// Physics component automatically syncs with entity transform
RigidBodyComponent* body = entity->getComponent<RigidBodyComponent>();
body->applyForce(Vector4(0, 100, 0));  // Entity will move
```

### With Rendering

```cpp
// Mesh component automatically renders at entity position
MeshComponent* mesh = entity->getComponent<MeshComponent>();
mesh->setMaterial(newMaterial);  // Material changes
```

## Debugging

### Entity Inspector

In the editor, select an entity to see:
- Transform
- State flags
- All components
- Component properties

### Logging

```cpp
log::info << "Entity count: " << world->getEntities().size() << Endl;
log::info << "Entity '" << entity->getName() << "' at "
          << entity->getTransform().translation() << Endl;
```

## See Also

- [Runtime System](runtime.md) - Application and stages
- [Physics System](physics.md) - Physics components
- [Scripting](scripting.md) - Lua integration with entities
- [Render System](render.md) - Rendering components

## References

- Source: `code/World/World.h`
- Source: `code/World/Entity.h`
- Source: `code/World/IEntityComponent.h`
- Source: `code/World/IWorldComponent.h`
