---
layout: default
permalink: /manual/engine/architecture/
title: Architecture
parent: Engine
grand_parent: Manual
nav_order: 1
---

# Traktor Engine Architecture

This document describes the architectural design and core systems of the Traktor engine.

![TODO: Diagram showing the modular architecture with Core at the base, Runtime/World/Resource layer in the middle, and specialized systems (Render, Physics, Audio, Script) at the top]

## Overview

Traktor follows a modular, layered architecture where each module has well-defined responsibilities and dependencies. The architecture is designed for:

- **Modularity:** Clean separation between systems
- **Performance:** Minimal overhead and efficient resource usage
- **Flexibility:** Easy to extend and customize
- **Maintainability:** Clear code structure and conventions

## Core Design Principles

### 1. Object-Oriented with Reference Counting

All major engine objects inherit from the `Object` base class, which provides:

- **Reference Counting:** Automatic memory management using `Ref<T>` smart pointers
- **RTTI (Runtime Type Information):** Dynamic type checking and casting
- **Serialization:** Built-in support for saving/loading objects

```cpp
// Example: Using reference-counted objects
Ref<Scene> scene = new Scene();
// No need to manually delete - reference counting handles cleanup
```

### 2. Data-Driven Design

The engine emphasizes data-driven development:

- Configuration and content are externalized from code
- Assets are defined using editor tools
- Behavior is scripted in Lua
- Pipeline system converts source assets to runtime data

### 3. Separation of Editor and Runtime

There's a clear distinction between editor code and runtime code:

- **Editor Code:** Tools, asset editors, pipeline processors
- **Runtime Code:** The actual game engine that runs on target platforms

This separation keeps the runtime lean and ensures editor-specific code doesn't bloat deployed games.

## Architectural Layers

### Layer 1: Core

The **Core** module provides fundamental functionality used throughout the engine:

#### Object System

- **`Object`:** Base class for all engine objects
- **`Ref<T>`:** Smart pointer for reference counting
- **`IRefCount`:** Interface for reference-counted objects

```cpp
class MyComponent : public Object
{
    T_RTTI_CLASS; // Macro for RTTI support
public:
    // Your code here
};
```

#### Containers

Custom container classes optimized for game development:

- **`AlignedVector<T>`:** Vector with aligned storage
- **`SmallMap<K, V>`:** Efficient small maps
- **`RefArray<T>`:** Array of reference-counted objects
- **`RefSet<T>`:** Set of reference-counted objects

#### Threading

Platform-agnostic threading primitives:

- **`Thread`:** Thread creation and management
- **`Semaphore`:** Thread synchronization
- **`CriticalSection`:** Mutual exclusion
- **`JobQueue`:** Job system for parallel work
- **`ThreadPool`:** Managed thread pool

#### Math

Comprehensive math library:

- **`Vector4`:** 4D vectors (used for 3D positions with w=1)
- **`Matrix44`:** 4x4 matrices
- **`Quaternion`:** Rotation representation
- **`Aabb3`:** Axis-aligned bounding boxes
- **`Frustum`:** View frustum for culling

#### I/O

File and stream I/O:

- **`IStream`:** Stream interface
- **`FileSystem`:** File system abstraction
- **`Path`:** Path manipulation
- **`Reader`/`Writer`:** Text I/O

#### Serialization

Binary and XML serialization:

- **`ISerializer`:** Serialization interface
- **`ISerializable`:** Interface for serializable objects
- **`BinarySerializer`:** Binary format
- **`XmlSerializer`:** XML format

#### Logging

Comprehensive logging system:

```cpp
log::info << "Engine initialized" << Endl;
log::warning << "Missing optional resource" << Endl;
log::error << "Failed to load: " << path << Endl;
```

### Layer 2: Resource Management

The **Resource** module handles asset loading and memory management:

- **`IResourceManager`:** Resource loading interface
- **`IResourceFactory`:** Creates runtime resources from data
- **Streaming:** Asynchronous asset loading
- **Caching:** Manages resource lifetime and memory

### Layer 3: Runtime Framework

The **Runtime** module provides the application framework:

#### Application Model

The runtime uses a **Server-based architecture** where different subsystems are implemented as "servers":

```cpp
class IApplication
{
    virtual bool initialize(const ...);
    virtual void destroy();
};
```

**Server Types:**

- **`IRenderServer`:** Graphics rendering
- **`IPhysicsServer`:** Physics simulation
- **`IAudioServer`:** Audio playback
- **`IScriptServer`:** Script execution
- **`IWorldServer`:** Entity management
- **`IInputServer`:** Input handling
- **`IResourceServer`:** Resource management
- **`IOnlineServer`:** Online services

#### State Management

Applications are organized into **States**:

```cpp
class IState
{
    virtual bool create();
    virtual void destroy();
    virtual void update(UpdateInfo& info);
};
```

States represent different modes of the application (menu, gameplay, loading, etc.).

### Layer 4: World System

The **World** module implements the entity-component system:

#### Entity-Component Architecture

- **`World`:** Container for all entities
- **`Entity`:** Game objects in the world
- **`IEntityComponent`:** Components attached to entities

```cpp
Ref<Entity> entity = new Entity();
entity->setComponent(new MeshComponent(...));
entity->setComponent(new ScriptComponent(...));
world->addEntity(entity);
```

#### Component Types

Common components include:

- **Transform:** Position, rotation, scale
- **Mesh:** 3D geometry rendering
- **Light:** Light sources
- **Script:** Lua script behavior
- **RigidBody:** Physics simulation
- **Sound:** Audio emitters

### Layer 5: Specialized Systems

Higher-level systems built on the foundation:

- **Render:** Vulkan-based rendering
- **Physics:** Jolt/Bullet integration
- **Sound:** Multi-channel audio
- **Script:** Lua integration
- **Animation:** Skeletal and scene animation
- **Terrain:** Heightfield terrain
- **Spray:** Particle effects
- **UI:** User interface
- **Ai:** AI and pathfinding

## Module Dependencies

![TODO: Diagram showing module dependency graph with Core at the bottom, and arrows pointing upward showing how higher-level modules depend on lower-level ones]

The dependency hierarchy ensures:

- **Core** has no dependencies (except C++ standard library)
- **All modules** can depend on Core
- **Circular dependencies** are avoided
- **Clean builds** are possible by building in dependency order

### Example Dependency Chain

```
UI System → World System → Resource System → Core
```

## Memory Management

### Reference Counting

The engine uses intrusive reference counting:

```cpp
Ref<Mesh> mesh = new Mesh(); // refCount = 1
Ref<Mesh> mesh2 = mesh;      // refCount = 2
mesh2 = nullptr;             // refCount = 1
// When mesh goes out of scope, object is deleted
```

**Advantages:**

- Automatic cleanup
- No garbage collection overhead
- Predictable destruction timing

**Considerations:**

- Be aware of circular references
- Use weak references when needed

### Custom Allocators

Some systems use custom allocators for optimization:

- **Pool Allocators:** For frequently created/destroyed objects
- **Stack Allocators:** For temporary allocations
- **Aligned Allocators:** For SIMD data

## Multithreading

### Job System

The engine uses a job-based parallelism model:

```cpp
JobQueue& queue = JobQueue::getInstance();
queue.add([](Thread*) {
    // Parallel work here
});
queue.wait(); // Wait for completion
```

### Thread Safety

- **Immutable Data:** Prefer immutable data structures
- **Synchronization:** Use mutexes/semaphores when needed
- **Thread-Local Storage:** For per-thread data

## Plugin Architecture

The engine supports plugins for both editor and runtime:

### Editor Plugins

```cpp
class IEditorPlugin
{
    virtual bool create(...);
    virtual void destroy();
};
```

Editor plugins can add:

- Custom asset types
- New editors
- Pipeline processors
- Tools and utilities

### Runtime Plugins

```cpp
class IRuntimePlugin
{
    virtual void setup(...);
};
```

Runtime plugins can extend:

- Component types
- Resource factories
- Server implementations

## Coding Conventions

### Naming

- **Classes:** PascalCase (e.g., `MeshComponent`)
- **Methods:** camelCase (e.g., `updateTransform()`)
- **Members:** m_camelCase (e.g., `m_position`)
- **Constants:** c_camelCase (e.g., `c_maxEntities`)

### Macros

Common macros:

- **`T_RTTI_CLASS`:** Declare RTTI for a class
- **`T_IMPLEMENT_RTTI_CLASS`:** Implement RTTI
- **`T_SAFE_ADDREF`:** Safely add reference
- **`T_SAFE_RELEASE`:** Safely release reference

### File Organization

```cpp
// Header file (*.h)
#pragma once
#include "Dependencies.h"

namespace traktor::module
{

class MyClass : public Object
{
    T_RTTI_CLASS;
public:
    // Public interface
private:
    // Private members
};

}

// Implementation file (*.cpp)
#include "MyClass.h"

namespace traktor::module
{

T_IMPLEMENT_RTTI_CLASS(L"module.MyClass", MyClass, Object)

// Implementation

}
```

## Performance Considerations

### Design for Cache Friendliness

- Group frequently accessed data together
- Use struct-of-arrays (SoA) when beneficial
- Minimize indirection

### Minimize Allocations

- Pre-allocate when possible
- Use object pools
- Reuse containers

### Leverage SIMD

The math library supports SIMD operations:

```cpp
Vector4 result = v1 + v2; // Uses SIMD when available
```

## Extension Points

The architecture provides many extension points:

1. **Custom Components:** Extend entity behavior
2. **Custom Resources:** Add new asset types
3. **Custom Shaders:** Create materials via Shader Graph
4. **Custom Pipelines:** Process assets in custom ways
5. **Plugins:** Add editor and runtime functionality

## Best Practices

1. **Derive from Object:** For automatic memory management
2. **Use Ref<T>:** Instead of raw pointers
3. **Follow RTTI Patterns:** Use macros correctly
4. **Leverage Data-Driven Design:** Externalize configuration
5. **Profile First:** Optimize based on measurements
6. **Read the Code:** The codebase is well-structured

## Next Steps

- Learn about [Runtime System](runtime/) for application structure
- Explore [World System](world/) for entity-component patterns
- Study [Resource Management](resources/) for asset handling

## References

- Source code: `code/Core/Object.h`
- Source code: `code/Core/Ref.h`
- Source code: `code/Runtime/IApplication.h`
- Source code: `code/World/World.h`
