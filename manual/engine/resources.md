---
layout: default
permalink: /manual/engine/resources/
title: Resources
parent: Engine
grand_parent: Manual
nav_order: 7
---

# Resource Management

The Resource system handles asset loading, caching, and memory management for all game content including meshes, textures, sounds, and scripts.

## Overview

Traktor uses a **resource proxy** system that:
- **Lazy loads** assets when first accessed
- **Caches** loaded resources to avoid duplicates
- **Streams** large assets asynchronously
- **Manages lifetime** automatically via reference counting
- **Hot-reloads** assets when modified in editor

## Resource Proxy

Resources are accessed through `Proxy<T>` smart pointers:

```cpp
// Declare resource proxy
resource::Proxy<render::Mesh> m_meshResource;

// Load resource
m_meshResource = resourceManager->load<render::Mesh>("Meshes/Character");

// Access resource (may trigger load)
render::Mesh* mesh = m_meshResource;

// Check if loaded
if (m_meshResource)
{
    // Use mesh
}
```

## Resource Manager

The `IResourceManager` interface provides resource loading:

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

### Accessing Resource Manager

```cpp
// From C++
IResourceServer* resourceServer = environment->getResourceServer();
IResourceManager* resourceManager = resourceServer->getResourceManager();

Proxy<Mesh> mesh = resourceManager->load<Mesh>("Meshes/Character");
```

```lua
-- From Lua (usually handled automatically)
local mesh = context.resourceManager:load("Meshes/Character")
```

## Common Resource Types

### Meshes

```cpp
resource::Proxy<render::Mesh> meshResource;
meshResource = resourceManager->load<render::Mesh>("Meshes/Character");

render::Mesh* mesh = meshResource;
if (mesh)
{
    // Use mesh for rendering
}
```

### Textures

```cpp
resource::Proxy<render::Texture> textureResource;
textureResource = resourceManager->load<render::Texture>("Textures/Albedo");

render::Texture* texture = textureResource;
```

### Materials

```cpp
resource::Proxy<render::Shader> shaderResource;
shaderResource = resourceManager->load<render::Shader>("Materials/PBR");

render::Shader* shader = shaderResource;
```

### Sounds

```cpp
resource::Proxy<sound::Sound> soundResource;
soundResource = resourceManager->load<sound::Sound>("Audio/Explosion");

sound::Sound* sound = soundResource;
```

### Scripts

```cpp
resource::Proxy<script::Script> scriptResource;
scriptResource = resourceManager->load<script::Script>("Scripts/Player");

script::Script* script = scriptResource;
```

## Asynchronous Loading

For large resources or loading screens:

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

## Resource Lifetime

Resources use reference counting:

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

**Automatic Cleanup:** Resources are automatically unloaded when no longer referenced.

## Hot Reloading

During development, resources automatically reload when modified:

1. Edit asset in external tool (e.g., texture in Photoshop)
2. Save file
3. Traktor editor detects change
4. Asset is rebuilt by pipeline
5. Running game automatically reloads resource

**Manual Reload:**
```cpp
// Force reload specific resource
resourceManager->reload(resourceGuid);

// Reload all resources
resourceManager->reloadAll();
```

## Resource Factories

Custom resource types need factories:

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

## Memory Management

### Resource Budgets

Control memory usage:

```cpp
// Set maximum cache size (in MB)
resourceManager->setMaxCacheSize(512);

// Clear cache to free memory
resourceManager->releaseUnused();
```

### Preloading

Preload resources to avoid hitches:

```cpp
// Preload level resources
resourceManager->preload("Levels/Level1");

// Wait for preload to complete
while (!resourceManager->isPreloadComplete())
{
    // Show loading screen
}
```

## Best Practices

1. **Use Proxies:** Always use `Proxy<T>`, never raw pointers
2. **Preload Critical Assets:** Load important resources during loading screens
3. **Release Unused:** Periodically release unused resources to free memory
4. **Check Validity:** Always check if proxy is valid before use
5. **Avoid Sync Load:** Use async loading for large assets

## Common Patterns

### Component with Resource

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

### Loading Screen

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

## Debugging

### Resource Statistics

```cpp
// Get resource stats
ResourceStats stats = resourceManager->getStats();

log::info << "Loaded resources: " << stats.loadedCount << Endl;
log::info << "Cache size: " << stats.cacheSize << " MB" << Endl;
log::info << "Peak usage: " << stats.peakUsage << " MB" << Endl;
```

### Resource List

```cpp
// List all loaded resources
RefArray<ISerializable> resources = resourceManager->getAllResources();
for (auto resource : resources)
{
    log::info << "Resource: " << type_name(resource) << Endl;
}
```

## See Also

- [Architecture](architecture.md) - Resource system design
- [Editor](../editor.md) - Asset pipeline and building

## References

- Source: `code/Resource/`
