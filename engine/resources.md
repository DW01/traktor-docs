---
layout: default
permalink: /engine/resources/
title: Resources
parent: Engine

nav_order: 7
---

# Resource Management - Your Game's Asset Library

Imagine you're building a massive game world with thousands of 3D models, textures, sounds, and scripts. You can't load everything into memory at once—that would consume gigabytes and make startup painfully slow. But you also can't manually track what's loaded, when to load it, and when to unload it. That's a recipe for crashes, hitches, and headaches.

This is where Traktor's **Resource system** shines. Think of it as a smart librarian for your game's assets. You ask for a resource by name, and the system handles all the details: finding it on disk, loading it into memory, caching it so you don't load it twice, and even automatically reloading it when you make changes during development. The system is lazy (resources only load when actually needed), efficient (shared resources are loaded once), and automatic (reference counting handles cleanup).

The resource system manages everything your game needs: meshes, textures, materials, sounds, scripts, animations, and more. It's one of those invisible systems that, when done right, you never think about—it just works.

## The Proxy Pattern: Claim Tickets for Resources

At the heart of the resource system is the **Proxy** pattern. Think of a `Proxy<T>` as a claim ticket or voucher. When you request a resource, you get a proxy immediately—it's lightweight and fast. The actual resource might not be loaded yet, but that's okay. When you try to use the resource, the proxy automatically loads it if needed.

```cpp
// Declare resource proxy
resource::Proxy<render::Mesh> m_meshResource;

// Load resource (returns immediately, actual load happens lazily)
m_meshResource = resourceManager->load<render::Mesh>("Meshes/Character");

// Access resource (triggers load if not yet loaded)
render::Mesh* mesh = m_meshResource;

// Check if loaded
if (m_meshResource)
{
    // Use mesh
}
```

This approach has huge benefits: you can set up references to resources during initialization without stalling while they load, resources load on-demand rather than all at once, and multiple objects can share the same resource (it's only loaded once in memory).

## The Resource Manager: Your Asset Librarian

The **IResourceManager** is the central hub for loading resources. It tracks what's loaded, manages the cache, and handles hot-reloading during development:

```cpp
class IResourceManager
{
    // Load resource by path
    template<typename T>
    Proxy<T> load(const std::string& path);

    // Reload resource (hot-reload)
    void reload(const Guid& resourceId);

    // Release all cached resources
    void releaseAll();
};
```

### Accessing the Resource Manager

From C++, you get the resource manager through the environment:

```cpp
// From C++
IResourceServer* resourceServer = environment->getResourceServer();
IResourceManager* resourceManager = resourceServer->getResourceManager();

Proxy<Mesh> mesh = resourceManager->load<Mesh>("Meshes/Character");
```

From Lua, the context object usually provides access (though most of the time, resources are loaded automatically):

```lua
-- From Lua (usually handled automatically)
local mesh = context.resourceManager:load("Meshes/Character")
```

## Loading Different Resource Types

The beauty of the resource system is that it works the same way regardless of what you're loading:

**Meshes** (3D models):

```cpp
resource::Proxy<render::Mesh> meshResource;
meshResource = resourceManager->load<render::Mesh>("Meshes/Character");

render::Mesh* mesh = meshResource;
if (mesh)
{
    // Use mesh for rendering
}
```

**Textures** (images):

```cpp
resource::Proxy<render::Texture> textureResource;
textureResource = resourceManager->load<render::Texture>("Textures/Albedo");

render::Texture* texture = textureResource;
```

**Materials** (shaders and rendering properties):

```cpp
resource::Proxy<render::Shader> shaderResource;
shaderResource = resourceManager->load<render::Shader>("Materials/PBR");

render::Shader* shader = shaderResource;
```

**Sounds** (audio files):

```cpp
resource::Proxy<sound::Sound> soundResource;
soundResource = resourceManager->load<sound::Sound>("Audio/Explosion");

sound::Sound* sound = soundResource;
```

**Scripts** (Lua code):

```cpp
resource::Proxy<script::Script> scriptResource;
scriptResource = resourceManager->load<script::Script>("Scripts/Player");

script::Script* script = scriptResource;
```

The pattern is always the same: declare a proxy of the appropriate type, load it by path, and use it when ready.

## Asynchronous Loading: Avoiding Hitches

Synchronous loading (waiting for a resource to load before continuing) can freeze your game for seconds when loading large assets. That's unacceptable for smooth gameplay. **Asynchronous loading** lets you start a load in the background and check back later when it's ready:

```cpp
// Start async load
Ref<ResourceLoader> loader = resourceManager->loadAsync<Mesh>("Meshes/LargeLevel");

// Check if loaded (in update loop)
if (loader->isReady())
{
    Proxy<Mesh> mesh = loader->getResource();
    // Use mesh
}
else
{
    // Still loading, show progress
    float progress = loader->getProgress();  // 0.0 to 1.0
}
```

This is perfect for loading screens. Start the async load, display a progress bar that updates each frame, and transition to gameplay when everything's ready.

## Automatic Lifetime Management

Resources use **reference counting** to manage their lifetime. When you copy a proxy, the reference count increases. When a proxy goes out of scope or is set to null, the reference count decreases. When the count hits zero, the resource can be unloaded:

```cpp
// Resource loaded (refCount = 1)
Proxy<Mesh> mesh1 = resourceManager->load<Mesh>("Meshes/Character");

// Share resource (refCount = 2)
Proxy<Mesh> mesh2 = mesh1;

// Release reference (refCount = 1)
mesh2 = nullptr;

// Release last reference (refCount = 0, resource may be unloaded)
mesh1 = nullptr;
```

This is automatic and safe. You never need to manually unload resources—when they're no longer referenced, the system takes care of cleanup.

## Hot Reloading: Instant Iteration

One of the most developer-friendly features of Traktor is **hot reloading**. During development, when you modify an asset, the engine automatically reloads it without restarting the game:

Here's how it works:
1. You edit an asset in an external tool (e.g., a texture in Photoshop, a script in your code editor)
2. You save the file
3. The Traktor editor detects the change
4. The asset pipeline rebuilds the asset
5. The running game automatically reloads the resource

No restart, no manual reload—just instant feedback. This dramatically speeds up iteration.

You can also manually trigger reloads:

```cpp
// Force reload specific resource
resourceManager->reload(resourceGuid);

// Reload all resources
resourceManager->reloadAll();
```

## Extending the Resource System

If you create custom resource types, you need to register a **resource factory**:

```cpp
class MyResourceFactory : public resource::IResourceFactory
{
    T_RTTI_CLASS;
public:
    virtual bool create(
        resource::IResourceManager* resourceManager,
        const db::Database* database,
        const db::Instance* instance,
        Ref<ISerializable>& outResource
    ) const override
    {
        // Load and create resource
        Ref<MyResource> resource = new MyResource();
        // ... initialize resource ...
        outResource = resource;
        return true;
    }
};
```

The factory tells the resource system how to create instances of your custom type from the database.

## Managing Memory

The resource system is smart about memory, but you can tune it for your specific needs.

### Resource Budgets

Control how much memory the cache uses:

```cpp
// Set maximum cache size (in MB)
resourceManager->setMaxCacheSize(512);

// Clear cache to free memory
resourceManager->releaseUnused();
```

This is useful on memory-constrained platforms or when transitioning between levels—clear unused resources to make room for the next batch.

### Preloading Resources

To avoid hitches during gameplay, preload critical resources during loading screens:

```cpp
// Preload level resources
resourceManager->preload("Levels/Level1");

// Wait for preload to complete
while (!resourceManager->isPreloadComplete())
{
    // Show loading screen
}
```

Preloading loads everything up front so there are no surprises later.

## Best Practices

**Always use Proxies.** Never store raw pointers to resources. Proxies handle reference counting and lazy loading automatically.

**Preload critical assets.** Load important resources during loading screens so they're ready when needed.

**Release unused resources.** Periodically call `releaseUnused()` to free memory from resources no longer referenced.

**Check validity before use.** Always check if a proxy is valid (non-null) before accessing the resource—it might still be loading.

**Use async loading for large assets.** Synchronous loading can freeze your game. Async loading keeps things smooth.

## Common Patterns

### Component Holding a Resource

Components often hold references to resources:

```cpp
class MeshComponent : public IEntityComponent
{
private:
    resource::Proxy<render::Mesh> m_mesh;

public:
    void setMesh(resource::Proxy<render::Mesh> mesh)
    {
        m_mesh = mesh;
    }

    virtual void update(const UpdateParams& update) override
    {
        if (m_mesh)  // Check if resource is loaded
        {
            // Use mesh
        }
    }
};
```

### Loading Screen with Progress

A common pattern is a loading screen that shows progress while assets load:

```lua
import(traktor)

LoadingScreen = LoadingScreen or class("LoadingScreen", world.ScriptComponent)

function LoadingScreen:new()
    -- Start loading next level
    self._loader = context.resourceManager:loadAsync("Levels/NextLevel")
end

function LoadingScreen:update(contextObject, totalTime, deltaTime)
    if self._loader:isReady() then
        -- Loading complete, transition to level
        local level = self._loader:getResource()
        contextObject.stage:gotoStage(level)
    else
        -- Update loading bar
        local progress = self._loader:getProgress()
        self:updateLoadingBar(progress)
    end
end

function LoadingScreen:updateLoadingBar(progress)
    -- Update UI with progress
end
```

This gives players visual feedback and keeps the game responsive during long loads.

## Debugging Resources

### Resource Statistics

Check memory usage and loaded resource counts:

```cpp
// Get resource stats
ResourceStats stats = resourceManager->getStats();

log::info << "Loaded resources: " << stats.loadedCount << Endl;
log::info << "Cache size: " << stats.cacheSize << " MB" << Endl;
log::info << "Peak usage: " << stats.peakUsage << " MB" << Endl;
```

This helps you track down memory leaks or understand what's consuming memory.

### Listing Loaded Resources

Get a list of all currently loaded resources:

```cpp
// List all loaded resources
RefArray<ISerializable> resources = resourceManager->getAllResources();
for (auto resource : resources)
{
    log::info << "Resource: " << type_name(resource) << Endl;
}
```

Useful for debugging why resources aren't unloading or finding unexpected resource usage.

## See Also

- [Architecture](architecture.md) - Resource system design
- [Editor](../editor.md) - Asset pipeline and building

## References

- Source: `code/Resource/`
