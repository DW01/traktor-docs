---
layout: default
permalink: /editor/workflows/
title: Workflows
parent: Editor
nav_order: 11
---

# Editor Workflows

This page covers common workflows and techniques for working efficiently in the Traktor editor.

## Importing Assets

### Creating a Mesh Asset

1. Select a group node in the Database view and right-click
2. Select **"Create mesh asset..."** from the wizard menu
3. Browse and select your source model (Collada, FBX, LWO, or OBJ file)
4. A new mesh asset instance is created in the database with the same name as the source model
5. Double-click the instance to open the Mesh Asset editor
6. Configure runtime specifications:
   - **Mesh type** - Static, instanced, skinned, etc.
   - **Material shaders** - Mapping between source materials and runtime shaders
   - By default, automatic shaders are generated from the model's materials

### Creating a Texture Asset

1. Select a group node in the Database and right-click
2. Select **"Import texture batch..."** from the wizard menu
3. The Import Texture Batch dialog appears
4. Click the '+' button in the toolbar to add source textures
5. For each texture, configure usage properties:
   - Compression settings
   - Mipmap generation
   - Normal map conversion
   - Scaling options
6. Click OK - texture asset instances are created for each imported texture
7. Double-click any texture asset to modify its properties later

### Manually Creating Assets

For asset types without custom wizards (sounds, videos, etc.):

1. Right-click in the Database and select **"New instance..."**
2. Browse and select the asset type (e.g., `traktor.sound.SoundAsset`)
3. Name the asset and click OK
4. Double-click the new asset to open its editor
5. Browse for the source file and configure build properties

## Scene Editor Navigation

### Camera Controls

**Navigate the viewport:**
- Hold **Ctrl** + **Left mouse button** - Translate camera position
- Hold **Ctrl** + **Right mouse button** - Rotate camera
- **Mouse wheel** - Zoom in/out

**Focus on selection:**
- Press **F** while an entity is selected to frame it in the viewport

### Transform Controls

**Select the transform mode** from the toolbar:
- **Translation** - Move entities
- **Rotation** - Rotate entities
- **Scale** - Resize entities

**Apply transformations:**
- Hold **Left mouse button** and drag to apply the current modifier
- **X, Y, Z buttons** on toolbar - Constrain to specific axes

**Snapping:**
- **Magnetic snapping** - Snap entities to each other's bounding boxes when close enough
- **Grid snapping** - Snap to grid intervals (select distance from toolbar dropdown)

### Selection Techniques

**Select single entity:**
- Click on entity in viewport OR
- Click on entity in Entities panel

**Select multiple entities:**
- Hold **Shift** and click to add to selection
- **Click and drag** to create selection box

### Toolbar Overview

- **Selection tool** - Enable selection through 3D views
- **Translation/Rotation/Scale** - Transform modifiers
- **X/Y/Z axis toggles** - Constrain transformations
- **Magnetic/Grid snapping** - Snapping modes
- **Grid distance** - Snap interval selector
- **Playback controls** - For animation preview
- **Viewport configuration** - 1/2/4 view layout

## Working with Prefabs (External Entities)

Prefabricated entities are ready-made entities stored as separate objects in the database that can be reused across multiple scenes.

**Creating a prefab:**
1. Build an entity (or entity hierarchy) in a scene
2. Right-click the entity in the Entities panel
3. Select **"Extract as external entity..."**
4. Choose a location in the Database and name it
5. The entity in your scene is now a reference to the external entity

**Using prefabs:**
1. Drag an external entity from the Database into the Entities panel
2. The external entity appears in your scene
3. Changes to the source external entity automatically update all instances

**Benefits:**
- Reuse complex entities across scenes
- Update all instances by editing the source
- External entities can reference other external entities
- No constraints on complexity

## Standard Dialogs

### Type Browser

Used when selecting a concrete C++ type (e.g., component types, entity types).

- **Tree view** - Shows namespace organization
- **Type list** - Shows available types in the selected namespace
- **Filter** - Search for specific types

### Object Browser

Used to browse for objects in the database, often filtered by specific type.

**Features:**
- Automatic thumbnail preview generation
- Shader graph previews
- Texture previews
- Filtered by type for easier selection

### Color Dialog

Color picker dialog similar to Adobe Photoshop's color picker, used whenever editing colors in properties or materials.

## Dependencies Panel

The Dependencies panel shows what assets the selected entity uses:

- Double-click any dependency to open its editor
- Quickly identify which meshes, textures, materials an entity references
- Useful for optimization and troubleshooting

## See Also

- [Scene Editor](scene-editor.md) - Building 3D worlds
- [Database](database.md) - Managing assets
- [Pipeline](pipeline.md) - Asset build system
- [Settings](settings.md) - Editor configuration
