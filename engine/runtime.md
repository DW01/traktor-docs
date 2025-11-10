---
layout: default
permalink: /engine/runtime/
title: Runtime
parent: Engine

nav_order: 2
---

# Runtime System - Your Game's Heartbeat

The Runtime System is your game's conductor, orchestrating all the moving pieces to create a cohesive experience. While the engine provides powerful systems for graphics, physics, and audio, the Runtime System is what ties them all together and keeps everything running in harmony.

Think of it like running a theater production: you have lighting technicians, sound engineers, actors, and stagehands. Each group does their job, but someone needs to coordinate them. Calling cues, managing scene changes, and ensuring everything happens in the right order. That's the Runtime System.

![TODO: Diagram showing the runtime architecture with Application at the top, Servers in the middle (Render, Physics, Audio, Script, etc.), and States/Stages at the bottom]

## How It All Fits Together

The Runtime System uses a **server-based architecture**. Instead of having one monolithic game loop that handles everything, different subsystems are implemented as "servers". Independent managers that each handle one domain. The **RenderServer** handles graphics, the **PhysicsServer** manages physics simulation, the **AudioServer** controls sound, and so on.

Your game's content is organized into **Stages**. High-level states like "Main Menu", "Level 1", or "Loading Screen." Each stage contains **Layers**, which are ordered groups of entities and logic. Think of layers like transparencies stacked on an overhead projector: the background layer, the gameplay layer, the UI layer, each rendered in order to create the final image.

For more complex scenarios where you need fine-grained control, you can use **States** instead. Lower-level constructs that give you complete control over the update cycle. But for most game development, Stages are the way to go: they're data-driven, editable in the editor, and much easier to iterate on.

## Application Architecture

### IApplication Interface

The `IApplication` interface is the foundation of any Traktor application:

```cpp
class IApplication : public Object
{
public:
    virtual bool initialize(
        const CommandLine& cmdLine
    ) = 0;

    virtual void destroy() = 0;
};
```

**Responsibilities:**

- Initialize servers and subsystems
- Create initial state/stage
- Handle application-level events
- Clean up on shutdown

### Server System

Servers are managers for major subsystems. Each server provides a specific set of functionality:

#### Common Servers

**IRenderServer** - Graphics rendering
```cpp
class IRenderServer
{
    virtual bool create(...) = 0;
    virtual void destroy() = 0;
    virtual render::IRenderView* createRenderView(...) = 0;
};
```

**IPhysicsServer** - Physics simulation
```cpp
class IPhysicsServer
{
    virtual bool create(...) = 0;
    virtual void destroy() = 0;
    virtual void update(float deltaTime) = 0;
};
```

**IAudioServer** - Audio playback
```cpp
class IAudioServer
{
    virtual bool create(...) = 0;
    virtual void destroy() = 0;
    virtual void update(float deltaTime) = 0;
};
```

**IScriptServer** - Script execution
```cpp
class IScriptServer
{
    virtual bool create(...) = 0;
    virtual void destroy() = 0;
    virtual void* getScriptContext() = 0;
};
```

**IWorldServer** - Entity/World management
```cpp
class IWorldServer
{
    virtual bool create(...) = 0;
    virtual void destroy() = 0;
    virtual void update(const UpdateInfo& info) = 0;
};
```

**IResourceServer** - Asset loading
```cpp
class IResourceServer
{
    virtual IResourceManager* getResourceManager() = 0;
};
```

**IInputServer** - Input handling
```cpp
class IInputServer
{
    virtual bool create(...) = 0;
    virtual void destroy() = 0;
    virtual void update(float deltaTime) = 0;
};
```

**IOnlineServer** - Online services
```cpp
class IOnlineServer
{
    virtual bool create(...) = 0;
    virtual void destroy() = 0;
};
```

### Environment

The `IEnvironment` interface provides access to all servers:

```cpp
Ref<IRenderServer> renderServer = environment->getRenderServer();
Ref<IPhysicsServer> physicsServer = environment->getPhysicsServer();
Ref<IScriptServer> scriptServer = environment->getScriptServer();
// ... etc
```

## State System

States represent different modes or screens of your application.

### IState Interface

```cpp
class IState : public Object
{
    T_RTTI_CLASS;
public:
    enum UpdateResult
    {
        UrFailed = -1,  // Update failed, enter error recovery
        UrOk = 0,       // Update succeeded
        UrExit = 2      // Update succeeded but wants to terminate
    };

    enum BuildResult
    {
        BrFailed = -1,   // Build failed
        BrOk = 0,        // Build succeeded
        BrNothing = 1    // Nothing built, use default renderer
    };

    // Called when entering this state
    virtual void enter() = 0;

    // Called when leaving this state
    virtual void leave() = 0;

    // Update game logic
    virtual UpdateResult update(
        IStateManager* stateManager,
        const UpdateInfo& info
    ) = 0;

    // Post-physics update
    virtual UpdateResult postUpdate(
        IStateManager* stateManager,
        const UpdateInfo& info
    ) = 0;

    // Build rendering commands
    virtual BuildResult build(
        uint32_t frame,
        const UpdateInfo& info
    ) = 0;

    // Render the frame
    virtual bool render(
        uint32_t frame,
        const UpdateInfo& info
    ) = 0;

    // Handle events
    virtual bool take(const Object* event) = 0;
};
```

### State Lifecycle

![TODO: Diagram showing state lifecycle: Created → enter() → update loop (update → postUpdate → build → render) → leave() → Destroyed, with arrows showing transitions to other states]

1. **enter()** - Initialize state resources
2. **update()** - Update game logic (pre-physics)
3. **postUpdate()** - Update after physics
4. **build()** - Build rendering commands
5. **render()** - Submit rendering
6. **leave()** - Clean up state resources

### State Transitions

Use the `IStateManager` to change states:

```cpp
UpdateResult MyState::update(IStateManager* stateManager, const UpdateInfo& info)
{
    if (shouldLoadLevel)
    {
        // Transition to new state
        stateManager->push(new LevelState(...));
    }
    return UrOk;
}
```

**State Manager Methods:**

- **`push(IState* state)`** - Push a new state (keeps current state)
- **`pop()`** - Pop current state (returns to previous)
- **`replace(IState* state)`** - Replace current state
- **`clear()`** - Clear all states

## Stage System

Stages provide a higher-level, data-driven approach to organizing application content.

### What is a Stage?

A **Stage** represents the current high-level state of the application (menu, level, loading screen, etc.) and contains an ordered list of **Layers**.

### Layers

**Layers** are ordered groups within a stage that can contain:

- World entities
- UI elements
- Post-processing effects
- Custom logic

Layers are updated and rendered in order, allowing you to control execution and rendering sequence.

### Stage Structure

```
Stage (e.g., "MainMenu")
├── Layer: "Background"
│   └── Entities: Background visuals
├── Layer: "UI"
│   └── Entities: Menu buttons, text
└── Layer: "Effects"
    └── Entities: Particle effects
```

### Creating Stages

Stages are typically created in the editor:

1. In the Database, create a new **Stage** asset
2. Add **Layers** to the stage
3. Populate layers with entities
4. Configure transitions to other stages

### Stage Transitions

Stages can define named transitions to other stages:

```cpp
// In C++ (runtime)
Ref<Stage> nextStage = currentStage->loadStage(L"Level1", params);
currentStage->gotoStage(nextStage);

// Asynchronous loading
Ref<StageLoader> loader = currentStage->loadStageAsync(L"Level1", params);
// ... check loader->isReady() in update loop ...
if (loader->isReady())
{
    Ref<Stage> nextStage = loader->getStage();
    currentStage->gotoStage(nextStage);
}
```

```lua
-- In Lua (scripting)
stage:loadStage("Level1", params)
```

### Stage vs State

| Aspect | State | Stage |
|--------|-------|-------|
| **Abstraction Level** | Low-level, code-driven | High-level, data-driven |
| **Defined in** | C++ code | Editor (asset) |
| **Best for** | Custom application logic | Game levels, menus |
| **Flexibility** | Maximum control | Easier to iterate |
| **Layers** | Manual management | Built-in support |

**Recommendation:** Use **Stages** for most game development. Use **States** only when you need low-level control or custom application flow.

## Update Loop

The game loop follows this structure:

![TODO: Flow diagram showing: Input → Update → Physics → PostUpdate → Build Render Graph → Render → Present, with frame timing]

### Update Cycle

```
1. Input Processing
   - Poll input devices
   - Generate input events

2. Update (pre-physics)
   - Process game logic
   - Update animations
   - Handle AI

3. Physics Simulation
   - Step physics world
   - Resolve collisions

4. Post-Update (post-physics)
   - React to physics results
   - Finalize transforms

5. Build Render Graph
   - Culling
   - Sort render operations
   - Build GPU commands

6. Render
   - Execute render graph
   - GPU rendering

7. Present
   - Swap buffers
   - Display frame
```

### UpdateInfo

The `UpdateInfo` structure provides timing and frame information:

```cpp
struct UpdateInfo
{
    double totalTime;       // Total elapsed time
    float deltaTime;        // Frame delta time
    uint32_t frame;         // Frame number
    // ... additional fields
};
```

### Frame Rate Control

```cpp
class UpdateControl
{
    void setMaxFrameRate(float fps);
    float getFrameRate() const;
    float getDeltaTime() const;
};
```

## Runtime Configuration

### Application Configuration

Applications can be configured through:

1. **Command Line:** Pass parameters via command line
2. **Settings Files:** Load configuration from files
3. **Editor:** Configure via editor tools

### Example Application

```cpp
class MyGame : public runtime::IApplication
{
public:
    virtual bool initialize(const CommandLine& cmdLine) override
    {
        // Initialize servers
        m_renderServer = getRenderServer();
        m_worldServer = getWorldServer();

        // Create initial state/stage
        Ref<Stage> initialStage = loadStage(L"MainMenu");
        getStateManager()->push(initialStage);

        return true;
    }

    virtual void destroy() override
    {
        // Cleanup
    }

private:
    Ref<IRenderServer> m_renderServer;
    Ref<IWorldServer> m_worldServer;
};
```

## Plugins

The Runtime system supports plugins to extend functionality:

```cpp
class IRuntimePlugin
{
    virtual void setup(IEnvironment* environment) = 0;
};
```

Plugins can:

- Register custom components
- Add custom servers
- Extend resource factories

## Best Practices

### 1. Use Stages for Game Content

Prefer stages over states for levels, menus, and gameplay screens:

```
Good: Create stages in editor → transition between them
Avoid: Writing custom state classes for every screen
```

### 2. Organize Layers Logically

Order layers by:
- Update dependency
- Rendering order
- Logical grouping

### 3. Handle Transitions Gracefully

```cpp
// Load next stage asynchronously
Ref<StageLoader> loader = stage->loadStageAsync(L"NextLevel", nullptr);

// Show loading screen while loading
while (!loader->isReady())
{
    // Update loading screen
}

// Transition when ready
stage->gotoStage(loader->getStage());
```

### 4. Clean State Management

```cpp
void MyState::enter()
{
    // Acquire resources
}

void MyState::leave()
{
    // Release resources - always cleanup!
}
```

### 5. Use UpdateInfo Properly

```cpp
void MyState::update(IStateManager* sm, const UpdateInfo& info)
{
    // Use deltaTime for frame-independent updates
    position += velocity * info.deltaTime;

    // Use totalTime for timers
    if (info.totalTime > m_startTime + 5.0)
    {
        // 5 seconds elapsed
    }
}
```

## Common Patterns

### Loading Screen

```cpp
class LoadingState : public IState
{
    virtual void enter() override
    {
        // Start async load
        m_loader = m_stage->loadStageAsync(L"Level1", nullptr);
    }

    virtual UpdateResult update(...) override
    {
        if (m_loader->isReady())
        {
            stateManager->replace(m_loader->getStage());
        }
        return UrOk;
    }

    virtual BuildResult build(...) override
    {
        // Render loading screen
        return BrOk;
    }
};
```

### Main Menu

Create a Stage asset in the editor with:
- Background layer (visuals)
- UI layer (buttons)
- Script component for button handling

### Gameplay Level

Create a Stage asset with:
- Environment layer (lighting, sky)
- Terrain layer
- Objects layer (gameplay entities)
- UI layer (HUD)
- Effects layer (particles)

## Debugging

### Logging

```cpp
log::info << "State entered: " << getName() << Endl;
log::warning << "Frame rate drop: " << info.getDeltaTime() << Endl;
```

### Profiling

Use the integrated profiler to measure:
- Update time
- Render time
- Physics time
- Memory usage

## See Also

- [Architecture Overview](architecture.md) - Engine design
- [World System](world.md) - Entity and component system
- [Scripting](scripting.md) - Lua integration
- [Resources](resources.md) - Asset management

## References

- Source: `code/Runtime/IApplication.h`
- Source: `code/Runtime/IState.h`
- Source: `code/Runtime/Engine/Stage.h`
- Source: `code/Runtime/Engine/Layer.h`
