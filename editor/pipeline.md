---
layout: default
permalink: /editor/pipeline/
title: Assets & Pipeline
parent: Editor
nav_order: 2
---

# Assets & Pipeline - From Source to Runtime

You edit assets in a friendly format. Text for scripts, visual graphs for shaders, WYSIWYG for scenes. Your game doesn't run these directly. Instead, a sophisticated **pipeline system** transforms your editable assets into optimized runtime formats. This is like cooking: you work with raw ingredients (source assets), and the pipeline cooks them into a finished dish (runtime data) that the game engine can consume efficiently.

The pipeline runs automatically in the background. Save a texture? The pipeline converts it to an optimized GPU format. Modify a script? The pipeline validates it and prepares it for hot-reload. Edit a scene? The pipeline updates dependent materials, meshes, and entities. You rarely think about this. It just works. But understanding it helps when things don't.

## Why the Pipeline Matters

**Speed:** The pipeline is **highly parallelized**, using all your CPU cores to build assets simultaneously. What might take minutes on a single core takes seconds across eight. **Incremental builds** mean only modified assets rebuild. Change one texture, rebuild one texture, not the whole project.

**Dependencies:** The pipeline tracks the **dependency graph** - which assets reference which. Modify a texture used by ten materials? The pipeline automatically rebuilds those ten materials. Delete a mesh? The editor warns you if any scenes reference it. This dependency awareness prevents broken references and ensures consistency.

**Hot Reloading:** The pipeline's real magic is instant iteration. Save an asset, and within milliseconds, it rebuilds and streams to your running game. No restarting, no waiting. Just immediate feedback. This tight loop between editing and testing is what makes development feel fluid rather than frustrating.

**Platform-Specific Optimization:** The pipeline generates different outputs for different platforms. A texture might be ASTC-compressed for mobile, BC7-compressed for PC, and DXT-compressed for older hardware. The pipeline handles this automatically based on your target configuration.

## How the Pipeline Works

The flow is straightforward:

1. **Source Asset** - You create and edit this in the editor (a scene, a material, a script)
2. **Pipeline Processor** - A specialized converter that transforms the source into runtime format
3. **Output Asset** - The optimized, ready-to-run data stored in the Output section
4. **Dependency Tracking** - The system that monitors relationships and triggers rebuilds

You can extend the pipeline with **custom processors** for new asset types or specialized build logic. The pipeline architecture is clean and modular, making it easy to add new converters.

## Asset Types

The editor supports many asset types out of the box: **SceneAsset** for 3D levels, **MeshAsset** for geometry, **MaterialAsset** for surface properties, **ShaderAsset** for visual graphs, **TextureAsset** for images, **ScriptAsset** for Lua code, **SoundAsset** for audio, and **AnimationAsset** for movement. Each type has its own specialized editor and pipeline processor.

## See Also

- [Database](database/) - Managing source assets
- [Output](output/) - Monitoring pipeline builds
- [Plugins](plugins/) - Extending the pipeline with custom processors
