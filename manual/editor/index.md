---
layout: default
permalink: /manual/editor/
title: Editor
parent: Manual
nav_order: 2
has_children: false
---

# Traktor Editor Guide

The Traktor Editor is a powerful, feature-rich development environment for creating games and interactive applications. The editor follows an "editor-first" philosophy, providing professional-grade tools for asset management, scene editing, and deployment.

## Table of Contents

- [Interface Overview](#interface-overview)
- [Database](#database)
- [Assets & Pipeline](#assets--pipeline)
- [Scene Editor](#scene-editor)
- [Shader Graph](#shader-graph)
- [Plugins](#plugins)
- [Targets](#targets)
- [Logging](#logging)

## Interface Overview

![TODO: Screenshot of Traktor Editor main interface showing the layout with Database panel on left, Scene Editor in center, Properties panel on right, and toolbar at top]

The Traktor Editor features a modular, dockable window system with multiple themes:

- **Light Theme:** Traditional light interface
- **Dark Theme:** Modern dark interface
- **Custom Themes:** Easily customizable appearance

### Key Features

- **Undo/Redo:** Full undo/redo support across all editors
- **Localization:** Built-in i18n support
- **Hot Reloading:** Active connection to running games with real-time asset updates
- **One-Click Deploy:** Deploy, run, and debug on any connected target platform
- **Multi-Platform:** Edit on Windows or Linux, deploy to multiple platforms

## Database

![TODO: Screenshot of the Database window showing the hierarchical tree structure with Source, Output, Audio, and Targets sections expanded with example assets]

The Database is the central hub for all project assets. It provides a hierarchical view of your project's content and manages asset organization.

### File Extensions

Traktor uses several file extensions to manage assets:

- **`.xdi`** - Database Instance (asset data)
- **`.xdm`** - Database Metadata (asset metadata)
- **`.xil`** - Instance Link (reference to another asset)
- **`.xgl`** - Group Link (reference to a group/folder)

### Database Sections

#### Source
Contains all editable source assets. This is where you create and edit:
- Scenes
- Materials
- Textures
- Scripts
- Audio files
- And all other game content

#### Output
Contains built/cooked runtime-optimized data. These files are generated automatically by the pipeline system and shouldn't be edited manually.

#### Audio
Dedicated section for audio asset management:
- Sound banks
- Audio groups
- Music tracks
- Sound effects

#### Targets
Platform-specific build configurations and target settings.

### Working with the Database

**Creating Groups (Folders):**
1. Right-click in the Database view
2. Select "New Group"
3. Name your group (F2 to rename)

**Creating Assets:**
1. Right-click on a group
2. Select "New Instance"
3. Choose asset category and type
4. Name your asset

**Organizing Assets:**
- Drag and drop to reorganize
- Use groups to create logical hierarchies
- Use instance links to reference assets across different locations

## Assets & Pipeline

The Traktor Editor uses a sophisticated pipeline system to convert source assets into optimized runtime data.

### Asset Pipeline Features

- **Parallel Building:** Highly parallelized build system for fast iteration
- **Incremental Builds:** Only rebuilds modified assets
- **Dependency Tracking:** Automatically rebuilds dependent assets
- **Hot Reloading:** Instantly reloads changed assets in running games
- **Platform-Specific:** Different build outputs for different target platforms

### Pipeline System

The pipeline system is formalized and extensible:

1. **Source Asset:** Editable asset in the Database
2. **Pipeline Processor:** Converts source to runtime format
3. **Output Asset:** Optimized runtime data
4. **Dependency Graph:** Tracks relationships between assets

**Custom Pipelines:**
You can create custom pipeline processors to handle new asset types or implement specialized build logic.

### Asset Types

Common asset types include:

- **SceneAsset:** 3D scenes and levels
- **MeshAsset:** 3D geometry
- **MaterialAsset:** Material definitions
- **ShaderAsset:** Shader graphs
- **TextureAsset:** Images and textures
- **ScriptAsset:** Lua scripts
- **SoundAsset:** Audio files
- **AnimationAsset:** Animation data

## Scene Editor

![TODO: Screenshot of the Scene Editor showing a 3D viewport with a scene containing terrain, objects, and lighting. Show the Entities panel with layer hierarchy on the left and Properties panel on the right]

The Scene Editor is where you build and edit your game worlds.

### Interface Components

#### Entities Panel
Displays the hierarchical structure of entities in your scene. Entities can be:
- Organized in layers
- Nested as parent/child relationships
- Enabled/disabled for testing

#### Properties Panel
Displays and edits properties of the selected entity or component. Features:
- Real-time editing
- Type-safe property editors
- Undo/redo support

#### Dependencies Panel
Shows asset dependencies for the current scene, helping you understand what resources are used.

#### Guides Panel
Configure visual guides and grid settings:
- Grid size and spacing
- Snap settings
- Reference guides

#### Measurements Panel
Tools for measuring distances and angles in the scene.

#### Resources Panel
View and manage scene resources like textures, materials, and meshes used in the scene.

### Working with Entities

**Creating Entities:**
1. Right-click in the Entities panel or in a layer
2. Select "Add Entity"
3. Name your entity

**Adding Components:**
1. Select an entity
2. In the Properties panel, click "Add Component"
3. Choose component type
4. Configure component properties

**Common Components:**
- **Transform:** Position, rotation, scale
- **Mesh:** 3D geometry rendering
- **Light:** Light sources (directional, point, spot)
- **Script:** Attach Lua scripts for behavior
- **Physics:** Rigid bodies, colliders
- **Audio:** Sound emitters

### Layers

Layers help organize entities:
- Group related entities together
- Toggle visibility for editing
- Enable/disable groups of entities
- Organize scene structure logically

### CSG (Constructive Solid Geometry)

Create compound meshes from primitive shapes:

1. Create an entity with **Group Component** and **Solid Component**
2. Add child entities with **Primitive Components**
3. Configure operations (Union, Subtract, Intersect)
4. Choose primitive shapes (Box, Sphere, Cylinder, etc.)

## Shader Graph

![TODO: Screenshot of the Shader Graph editor showing a node graph with various shader nodes connected together (texture sampling, math operations, vertex/fragment output). Include the node palette on the side]

The Shader Graph is a powerful node-based shader editor that supports all shader stages.

### Features

- **Visual Node Editor:** Create shaders visually
- **All Shader Stages:** Vertex, fragment, and compute shader support
- **Inline Code:** Insert custom shader code when needed
- **Live Preview:** See changes in real-time
- **Instant Reload:** Side-by-side shader recompile and reload for rapid iteration
- **Platform Agnostic:** Compiles to Vulkan SPIR-V

### Shader Stages

- **Vertex Shader:** Process vertex data
- **Fragment Shader:** Process pixel/fragment data
- **Compute Shader:** General-purpose GPU computing

### Working with Shader Graph

1. Create a new ShaderAsset in the Database
2. Double-click to open the Shader Graph editor
3. Add nodes from the node palette
4. Connect nodes to build shader logic
5. Add inline code nodes for custom GLSL/HLSL
6. Preview material in the editor

## Plugins

The Traktor Editor has a clean plugin architecture that makes it easy to extend functionality.

### Plugin Types

- **Editor Plugins:** Add new tools and windows
- **Asset Editors:** Custom editors for specific asset types
- **Pipeline Processors:** Custom asset build pipelines
- **Import/Export:** Data import and export tools

### Creating Plugins

Plugins implement the `IEditorPlugin` interface and can:
- Register new asset types
- Add menu items and toolbar buttons
- Create custom windows and panels
- Integrate with the build pipeline
- Add settings pages

### Plugin System Benefits

- Clean separation of editor code
- Easy to add new editors and tools
- Modular architecture
- Hot-reload support during development

## Targets

The Targets system manages deployment to different platforms.

### Supported Targets

- **Windows (Desktop)**
- **Linux (Desktop)**
- **Android** (experimental)
- **iOS** (experimental)
- **macOS** (experimental)

### Target Configuration

For each target, you can configure:
- Platform-specific build settings
- Quality/performance profiles
- Asset cooking parameters
- Deployment options

### One-Click Deploy

1. Select your target platform
2. Click the "Deploy and Run" button
3. The editor will:
   - Build modified assets
   - Package for target platform
   - Deploy to device/platform
   - Launch and connect debugger

### Active Connection

Once deployed, the editor maintains an active connection to running games:
- Hot-reload modified assets
- View real-time logs
- Profile performance
- Debug scripts

## Logging

The LogView provides real-time logging and debugging information.

### Log Levels

- **Info:** General information
- **Warning:** Warnings that don't stop execution
- **Error:** Errors that need attention
- **Debug:** Detailed debug information

### Features

- Filter by log level
- Search log messages
- Export logs
- View logs from connected targets
- Color-coded messages

### Using Logs

In your code, you can output to the log:

```cpp
// C++
log::info << "Initialization complete" << Endl;
log::error << "Failed to load asset: " << assetName << Endl;
```

```lua
-- Lua
log.info("Player spawned at position: " .. tostring(position))
log.error("Invalid input detected")
```

## Advanced Features

### Themes

Customize the editor appearance:
1. Go to Settings
2. Select Theme
3. Choose from built-in themes or create custom

### Workspace Management

- Save workspace layouts
- Multiple monitor support
- Customizable keyboard shortcuts

### Performance

The editor is optimized for:
- Large projects with thousands of assets
- Quick iteration times
- Low memory footprint
- Responsive UI even during builds

## Next Steps

- Explore [Engine Documentation](../engine/) to understand the runtime systems
- Learn [Scripting](../engine/scripting/) to add gameplay logic
- Study the [Rendering System](../engine/render/) for advanced rendering

## Additional Resources

- Check the editor source code in `code/Editor/`
- Join the [Discord Community](https://discord.gg/fSMrww2B7C) for help
- Look for plugin examples in the codebase