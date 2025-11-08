---
layout: default
permalink: /editor/
title: Editor

nav_order: 4
has_children: false
---

# Traktor Editor - Your Creative Workshop

The Traktor Editor is where you build, configure, and test your game. It provides a complete development environment with tools for scene editing, asset management, shader creation, and multi-platform deployment. The editor is tightly integrated with the engine runtime, enabling hot-reloading of assets—when you save changes, they appear in your running game within milliseconds without requiring a restart.

The Traktor Editor follows an "editor-first" philosophy: instead of bolting editing tools onto an existing runtime, the entire engine is built around a powerful, professional editor. This means deep integration, hot-reloading that actually works, and tools that feel cohesive rather than tacked on. You edit, save, and instantly see your changes in the running game—no lengthy rebuilds, no context switching.

## Table of Contents

- [Interface Overview](#interface-overview)
- [Database](#database)
- [Assets & Pipeline](#assets--pipeline)
- [Output](#output)
- [Audio](#audio)
- [Scene Editor](#scene-editor)
- [Shader Graph](#shader-graph)
- [Plugins](#plugins)
- [Targets](#targets)
- [Logging](#logging)

## Interface Overview

![TODO: Screenshot of Traktor Editor main interface showing the layout with Database panel on left, Scene Editor in center, Properties panel on right, and toolbar at top]

When you first open the Traktor Editor, you'll see a clean, professional workspace. The interface uses a **modular, dockable window system**—every panel can be moved, resized, and arranged to match your workflow. Working on a single monitor? Maximize the scene view. Have multiple displays? Spread panels across screens for maximum efficiency.

The editor offers **light and dark themes** for different lighting conditions and preferences. Working late at night? Switch to dark mode. Prefer traditional interfaces? Light theme has you covered. You can even create custom themes to match your exact preferences.

### What Makes the Editor Special

**Instant Feedback:** Change a texture, tweak a script, adjust a material—save it and watch it update in your running game within milliseconds. No waiting for compiles, no restarting, just immediate iteration. This hot-reloading system maintains an active connection to your game, streaming changes as you work.

**Full Undo/Redo:** Made a mistake? Press Ctrl+Z. Every editor, every panel, every action supports undo and redo. You can experiment fearlessly knowing you can always step back.

**One-Click Deploy:** When you're ready to test on a different platform, click one button. The editor builds your assets, packages everything, deploys to the target device, and launches the game—all automatically. Test on Windows, Linux, Android, or iOS without leaving the editor.

**Localization Built In:** The editor itself supports multiple languages, and your game can too. Internationalization (i18n) is a first-class feature, not an afterthought.

## Database - Your Project's Library

![TODO: Screenshot of the Database window showing the hierarchical tree structure with Source, Output, Audio, and Targets sections expanded with example assets]

Think of the Database as your project's library—a organized, searchable catalog of everything in your game. Every texture, every script, every scene, every sound lives here. The Database isn't just a file browser; it's an intelligent asset management system that tracks dependencies, manages references, and keeps your project organized as it grows from a prototype to a shipping game.

The Database shows a hierarchical tree view of your assets, much like a file explorer, but with superpowers. It understands relationships between assets—when you reference a texture in a material, the Database knows. When you delete something, it warns you if other assets depend on it. When you rename an asset, all references update automatically.

### Understanding Database Files

Behind the scenes, the Database uses a few special file extensions. You don't usually interact with these directly—the editor handles them—but understanding what they mean helps when working with version control or troubleshooting:

**`.xdi` files** (Database Instance) contain your actual asset data—the scene layout, the material properties, the script code. This is the "real" asset.

**`.xdm` files** (Database Metadata) store information about the asset—its type, its unique ID, its dependencies. The editor uses this to track relationships and build the dependency graph.

**`.xil` files** (Instance Link) are references to other assets. Instead of duplicating an asset, you create a link that points to the original. Change the original, and all links reflect that change.

**`.xgl` files** (Group Link) are references to entire folders/groups. Useful for organizing large projects with shared asset libraries.

### Database Structure

The Database contains all your project's assets organized in a hierarchical structure. Everything you create and edit lives in the **Source** section—scenes, materials, textures, scripts, audio files, and more. Think of it as your workshop: messy, iterative, full of work-in-progress. You organize assets into groups (folders), create new assets, and manage references all within this tree view.

### Working with the Database

The Database workflow is intuitive—right-click to create, drag-and-drop to organize, double-click to edit.

**Creating folders (called Groups):** Right-click anywhere in the Database, select "New Group", and name it. Press F2 to rename later. Use groups to organize related assets—a "Characters" group for all character assets, an "Environments" group for levels and props.

**Creating assets:** Right-click on a group, select "New Instance", choose the asset type (Scene, Material, Script, etc.), and name it. The editor creates the asset in the Database. To edit the asset, double-click it to open the appropriate editor—for example, double-clicking a Material opens the Material Editor where you can assign textures and configure properties.

**Organizing:** Drag and drop assets between groups to reorganize. Create hierarchies that make sense for your project. Use instance links (`.xil` files) when you need to reference the same asset in multiple places—change the original, and the links automatically update.

## Assets & Pipeline - From Source to Runtime

You edit assets in a friendly format—text for scripts, visual graphs for shaders, WYSIWYG for scenes. Your game doesn't run these directly. Instead, a sophisticated **pipeline system** transforms your editable assets into optimized runtime formats. This is like cooking: you work with raw ingredients (source assets), and the pipeline cooks them into a finished dish (runtime data) that the game engine can consume efficiently.

The pipeline runs automatically in the background. Save a texture? The pipeline converts it to an optimized GPU format. Modify a script? The pipeline validates it and prepares it for hot-reload. Edit a scene? The pipeline updates dependent materials, meshes, and entities. You rarely think about this—it just works—but understanding it helps when things don't.

### Why the Pipeline Matters

**Speed:** The pipeline is **highly parallelized**, using all your CPU cores to build assets simultaneously. What might take minutes on a single core takes seconds across eight. **Incremental builds** mean only modified assets rebuild—change one texture, rebuild one texture, not the whole project.

**Dependencies:** The pipeline tracks the **dependency graph**—which assets reference which. Modify a texture used by ten materials? The pipeline automatically rebuilds those ten materials. Delete a mesh? The editor warns you if any scenes reference it. This dependency awareness prevents broken references and ensures consistency.

**Hot Reloading:** The pipeline's real magic is instant iteration. Save an asset, and within milliseconds, it rebuilds and streams to your running game. No restarting, no waiting—just immediate feedback. This tight loop between editing and testing is what makes development feel fluid rather than frustrating.

**Platform-Specific Optimization:** The pipeline generates different outputs for different platforms. A texture might be ASTC-compressed for mobile, BC7-compressed for PC, and DXT-compressed for older hardware. The pipeline handles this automatically based on your target configuration.

### How the Pipeline Works

The flow is straightforward:

1. **Source Asset** - You create and edit this in the editor (a scene, a material, a script)
2. **Pipeline Processor** - A specialized converter that transforms the source into runtime format
3. **Output Asset** - The optimized, ready-to-run data stored in the Output section
4. **Dependency Tracking** - The system that monitors relationships and triggers rebuilds

You can extend the pipeline with **custom processors** for new asset types or specialized build logic. The pipeline architecture is clean and modular, making it easy to add new converters.

### Asset Types

The editor supports many asset types out of the box: **SceneAsset** for 3D levels, **MeshAsset** for geometry, **MaterialAsset** for surface properties, **ShaderAsset** for visual graphs, **TextureAsset** for images, **ScriptAsset** for Lua code, **SoundAsset** for audio, and **AnimationAsset** for movement. Each type has its own specialized editor and pipeline processor.

## Output - Pipeline Monitoring

The **Output** tab in the editor shows build logs and pipeline activity in real-time. When you save an asset and the pipeline processes it, all the details appear here—which assets are being built, whether builds succeeded or failed, warnings about potential issues, and timing information for performance analysis.

This is where you go to understand what the pipeline is doing. If an asset fails to build, the Output tab shows the error messages and stack traces. If builds are slow, you can see which processors are taking time. The Output tab gives you visibility into the automated build process that transforms your source assets into runtime-ready data.

You can filter messages, search for specific assets or errors, and export logs for troubleshooting. The Output tab is essential for debugging pipeline issues and understanding the asset build process.

## Audio - Editor Volume Control

The **Audio** tab controls audio playback volume within the editor. When you're working on your game and testing audio—previewing sound effects, listening to music tracks, or testing 3D spatial audio—this tab lets you adjust the overall volume without affecting your system settings.

This is a simple but essential tool for audio work. Lower the volume when you need to concentrate on visual editing, raise it when tuning audio balance, or mute it entirely when collaborating in a noisy environment. The Audio tab ensures you have quick control over editor audio output.

## Scene Editor - Building Your World

![TODO: Screenshot of the Scene Editor showing a 3D viewport with a scene containing terrain, objects, and lighting. Show the Entities panel with layer hierarchy on the left and Properties panel on the right]

The Scene Editor is where imagination becomes playable space. This is where you place characters, arrange environments, position lights, and compose the 3D worlds your players will explore. Think of it as a stage where you're the director—positioning actors (entities), setting up lighting, and choreographing the scene.

The Scene Editor combines a 3D viewport for visual editing with panels for organization and properties. You can fly through your scene in the viewport, click to select objects, and drag them around. Every change is immediate—move a light, and shadows update in real-time. Adjust a material property, and the object reflects it instantly.

### Interface Panels

**Entities Panel** shows your scene's structure as a tree. Every object in your scene—characters, lights, cameras, effects—appears here as an entity. You can organize entities into layers (like "Environment", "Gameplay", "UI"), nest them in parent-child hierarchies, and enable/disable them for testing. Select an entity here or click it in the viewport—both work.

**Properties Panel** displays everything about the selected entity. Position, rotation, scale? Here. Mesh, material, physics settings? All here. Edit properties in real-time and see changes immediately in the viewport. The panel adapts based on what's selected—a light shows intensity and color, a mesh shows geometry and materials.

**Dependencies Panel** reveals what assets your scene uses. Wondering which textures are loaded? Check here. Trying to optimize a scene? See which meshes and materials are referenced. This transparency helps manage complexity and track down performance issues.

**Guides and Visualization** provide control over what you see in the viewport. Enable a grid for snapping objects to exact positions. Add reference guides for alignment. Measure distances between objects. Beyond basic guides, you can toggle visualization for technical elements: navigation meshes, skeletal rigs, entity bounding boxes, light volumes and ranges, physics shapes and joints, cloth simulation, animation controller state, terrain heightfields, and more. These visualization options let you see the invisible systems at work—turn on physics shapes to debug collisions, show navigation meshes to verify AI pathfinding, or display skeleton bind poses when rigging characters.

### Working with Entities

Building a scene is simple: create entities, add components, and configure properties.

**Creating entities:** Right-click in the Entities panel (or directly in a layer), select "Add Entity", and name it. The entity appears in the tree, ready for components.

**Adding components:** Entities are containers; components define what they do. Every entity has a **transform property** by default (position, rotation, scale), so you can immediately place and orient it. Select an entity, click "Add Component" in the Properties panel, and choose a type. Add a **Mesh** component to render geometry. Add a **Light** component to illuminate the scene. Add a **Script** component to bring it to life with behavior. Add **Physics** components for collisions and movement. Add an **Audio** component for sounds.

**Common components you'll use:**
- **Mesh** - Renders 3D geometry with a material
- **Light** - Directional, point, or spot light sources
- **Script** - Lua scripts for behavior and logic
- **Physics** - Rigid bodies, colliders, character controllers
- **Audio** - 3D sound emitters

### Layers for Organization

As scenes grow, they get messy. **Layers** bring order. Group related entities into layers—"Environment" for props and terrain, "Gameplay" for interactive objects, "Lighting" for lights. Toggle a layer's visibility to focus on specific parts of your scene. Disable a layer to test without certain elements. Layers are organizational tools that scale with your project.

### CSG - Building Geometry from Primitives

Sometimes you need simple geometry—platforms, walls, trigger volumes—without modeling in an external tool. **CSG (Constructive Solid Geometry)** lets you create meshes from basic shapes using boolean operations.

Create an entity, add a **Group Component** and **Solid Component**, then add child entities with **Primitive Components** (boxes, spheres, cylinders). Configure operations—**Union** combines shapes, **Subtract** cuts holes, **Intersect** keeps only overlapping areas. The result is a compound mesh generated procedurally, perfect for blockouts and prototypes.

## Shader Graph - Visual Shader Creation

![TODO: Screenshot of the Shader Graph editor showing a node graph with various shader nodes connected together (texture sampling, math operations, vertex/fragment output). Include the node palette on the side]

Shaders define how surfaces look—their colors, reflections, textures, and lighting response. Traditionally, shaders are written in code (GLSL or HLSL), which works but requires programming expertise and makes iteration slow. The **Shader Graph** takes a different approach: visual, node-based shader creation where you connect boxes to build complex materials.

Think of it like wiring together audio equipment—each node is a device (texture sampler, math operation, color mixer), and you connect them with cables (node connections) to create the final output. No code required, though you can add inline GLSL/HLSL when you need low-level control.

### Why It's Powerful

**Visual and Intuitive:** Instead of typing "texture2D(sampler, uv) * color", you connect a Texture node to a Multiply node to a Color node. You see the flow of data visually, making shaders easier to understand and debug.

**Instant Feedback:** Change a node parameter, and the material updates in real-time. Move a slider for metallic, watch the reflections adjust instantly. This tight feedback loop makes shader development feel creative rather than tedious.

**All Shader Stages:** The graph supports **vertex shaders** (manipulate geometry), **fragment shaders** (compute pixel colors), and **compute shaders** (general GPU computing). You're not limited to basic materials—you can create advanced effects like tessellation, procedural geometry, or custom lighting models.

**Platform Agnostic:** The graph compiles to Vulkan SPIR-V, meaning your shaders work across Windows, Linux, Android, and any platform Vulkan supports. Write once, run everywhere.

**Custom Code When Needed:** Need something the built-in nodes don't provide? Add an **inline code node** with raw GLSL/HLSL. The shader graph doesn't lock you into visual-only—it gives you escape hatches when necessary.

### Creating Shaders

Create a new **ShaderAsset** in the Database and double-click it. The Shader Graph editor opens, showing an empty canvas and a node palette. Drag nodes from the palette onto the canvas—texture samplers, math operations, vertex position modifiers, fragment color outputs. Connect nodes by dragging from one output port to another input port. Configure each node's properties. Preview the result on a sphere or plane in the editor. Save, and the pipeline compiles it to optimized SPIR-V.

## Plugins - Extending the Editor

The Traktor Editor isn't a monolithic application—it's a modular platform built on a clean plugin architecture. Most of the editor's functionality, including asset editors and tools, is implemented as plugins. This means extending the editor is natural rather than hacky.

Want to add support for a new asset type? Write a plugin. Need a custom import tool for your studio's file format? Plugin. Want to integrate external tools into the editor? Plugin. The architecture makes extension feel like a first-class feature, not an afterthought.

### What Plugins Can Do

**Editor Plugins** add new tools and windows—a custom batch processor, a level analyzer, a performance profiler. They integrate into the editor's menu and toolbar, looking like built-in features.

**Asset Editors** handle specific asset types. The Scene Editor, Shader Graph, Material Editor—all plugins. You can create custom editors for game-specific assets using the same patterns.

**Pipeline Processors** extend the build system. Add custom asset types with specialized cooking logic. The pipeline system is extensible by design, making it easy to add new converters.

**Import/Export Tools** bring data into and out of the editor. Import FBX models, export scene data to custom formats, integrate with external tools—all possible through plugins.

Plugins implement the `IEditorPlugin` interface, giving them access to the editor's core systems. They can register asset types, add UI elements, integrate with the pipeline, and add settings pages. The plugin system is designed for cleanliness and modularity, with hot-reload support during development.

## Targets - Managing Deployment

The **Targets** tab manages connected deployment targets—the devices and platforms where you can build and run your game. This is your command center for multi-platform development. Each target represents a destination where your game can run: a local Windows build, a connected Android device, a remote Linux server, or an iOS simulator.

In the Targets tab, you can:

**Connect to targets** - Add and configure deployment destinations. Connect a physical Android device over USB, set up a remote Linux machine over the network, or configure a local build target for your development platform.

**Build for targets** - Trigger asset builds specific to each target platform. The pipeline processes your assets using platform-specific settings, compressing textures appropriately, optimizing shaders, and packaging data for the target's architecture.

**Deploy and run** - With one click, package your game, deploy it to the connected target, and launch it with a live debugger connection. The editor handles all the complexity—copying files, starting the runtime, establishing communication.

**Monitor connections** - See which targets are currently connected, their status (building, running, idle), and any connection issues. If a target disconnects, the Targets tab shows the problem.

Once deployed and running, the editor maintains a **live connection** to your game on the target. Edit an asset, save it, and watch it hot-reload on the running device—no restart required. View real-time logs streaming back from the target. This active connection makes cross-platform development practical and efficient, whether you're testing on a phone, a console, or a remote server.

## Logging - Real-Time Debugging

The **LogView** is your window into what's happening in your game. Every log message from the engine, your scripts, or the editor itself appears here in real-time. Debugging becomes easier when you can see exactly what's executing, what's failing, and what's being loaded.

Logs are categorized by severity: **Info** for general messages, **Warning** for potential issues, **Error** for problems that need fixing, and **Debug** for detailed diagnostic information. The LogView color-codes messages by level, making errors instantly visible in a sea of info messages.

You can **filter by level** to see only errors, **search** for specific messages, **export logs** for bug reports, and **view logs from connected targets**—when your game runs on a device, its logs stream back to the editor in real-time. This remote logging makes debugging on physical devices practical.

From your code, logging is simple:

```cpp
// C++
log::info << "Initialization complete" << Endl;
log::error << "Failed to load asset: " << assetName << Endl;
```

```lua
-- Lua
log:info("Player spawned at position: " .. tostring(position))
log:error("Invalid input detected")
```

Logs aren't just for debugging—they're for understanding. Add logs to track game state, asset loading, and performance. When something goes wrong, well-placed logs make the difference between hours of guesswork and minutes of diagnosis.

## Making the Editor Yours

The editor is flexible. **Themes** let you customize appearance—switch between light and dark, or create custom themes. **Workspace layouts** adapt to your workflow—save panel arrangements, use multiple monitors, customize keyboard shortcuts. The editor remembers your preferences, making it feel personal rather than generic.

Performance is a priority. The editor handles **large projects** with thousands of assets without slowing down. Builds use all available CPU cores for speed. The UI stays responsive even during intensive operations. Memory usage is kept low, leaving resources for your game and other tools.

## Next Steps

- Explore [Engine Documentation](../engine/) to understand the runtime systems
- Learn [Scripting](../engine/scripting/) to add gameplay logic
- Study the [Rendering System](../engine/render/) for advanced rendering

## Additional Resources

- Check the editor source code in `code/Editor/`
- Join the [Discord Community](https://discord.gg/fSMrww2B7C) for help
- Look for plugin examples in the codebase