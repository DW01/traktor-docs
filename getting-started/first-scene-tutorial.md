---
layout: default
permalink: /getting-started/first-scene-tutorial/
title: First Scene Tutorial
parent: Getting Started
nav_order: 4
---

# Tutorial: Create a Scene with a Lit Mesh

This tutorial walks you through creating your first complete scene with a 3D model and lighting. You'll learn the basic workflow for importing assets and building a scene.

## Prerequisites

- Traktor editor installed and running
- A 3D model file (Collada, FBX, LWO, or OBJ format)
- A workspace/project created

## Step 1: Configure Auto-Build

Before starting, enable automatic building to see changes instantly:

1. Open **Edit → Settings**
2. Go to the **General** tab
3. Check **"Build when source modified"**
4. Check **"Build when asset modified"**
5. Click OK

This ensures assets rebuild automatically when you save changes.

## Step 2: Import Your 3D Model

1. Right-click in the Database view (on a group or empty space)
2. Select **"Create mesh asset..."** from the wizard menu
3. Browse and select your 3D model file
4. Click OK

A new mesh asset instance appears in the database with the same name as your model. The pipeline automatically builds it in the background.

![TODO: Screenshot showing the Create mesh asset wizard dialog]

## Step 3: Create a Scene Asset

1. Right-click in the Database view
2. Select **"New instance..."**
3. In the type browser, navigate to `traktor/scene/SceneAsset`
4. Change the name from "Unnamed" to "MyScene"
5. Click OK

![TODO: Screenshot showing creating a new SceneAsset]

## Step 4: Open the Scene Editor

Double-click on "MyScene" in the Database. The Scene Editor opens showing an empty, dark viewport. This is normal. There's no lighting yet.

![TODO: Screenshot of empty Scene Editor with dark viewport]

## Step 5: Create a Root Entity

Scenes are organized hierarchically. Let's create a root entity to hold everything:

1. Right-click in the **Entities pane** (right side of Scene Editor)
2. Choose **"Add entity..."**
3. In the browser, select `traktor/world/GroupEntityData`
4. Click OK

A new entity appears in the Entities pane.

5. Select the new entity
6. In the **Properties view** (below Entities pane), click the name field
7. Enter "Root" and press Enter

## Step 6: Add a Directional Light

Now let's add lighting so we can see our mesh:

1. Right-click on the **"Root"** entity in the Entities pane
2. Choose **"Add entity..."**
3. Select `traktor/world/DirectionalLightEntityData`
4. Click OK

A directional light entity is added as a child of Root. This provides sunlight-style lighting from above.

## Step 7: Add Your Mesh Entity

1. Keep the **"Root"** entity selected
2. Right-click and choose **"Add entity..."** again
3. Select `traktor/mesh/MeshEntityData`
4. Click OK

A new mesh entity appears, but it's empty (no mesh assigned yet).

## Step 8: Assign the Mesh

1. Select the new mesh entity in the Entities pane
2. In the Properties view, find the **"Mesh"** property
3. Click the **"..."** button next to it
4. In the object browser, find your mesh asset (named after your source model)
5. Click OK

**The mesh now appears in the viewport, lit from above!**

![TODO: Screenshot showing the completed scene with lit mesh in viewport]

## Step 9: Navigate and Inspect

Use camera controls to inspect your scene:

- Hold **Ctrl + Left mouse button** - Move camera
- Hold **Ctrl + Right mouse button** - Rotate camera
- **Mouse wheel** - Zoom

Try selecting the mesh or light entities to see their properties.

## Step 10: Save and Test

1. Save the scene (**Ctrl+S** or **File → Save**)
2. The pipeline automatically builds the scene

Your first scene is complete! You can now:
- Add more meshes
- Adjust light properties (color, intensity)
- Add more lights (point lights, spot lights)
- Try different camera angles

## What's Next?

- Learn about [entity components](../engine/world.md) to add physics, scripts, and behavior
- Explore the [Scene Editor](../editor/scene-editor.md) features in depth
- Add [materials and shaders](../editor/shader-graph.md) to customize appearance
- Try [importing textures](../editor/workflows.md#creating-a-texture-asset)

## Troubleshooting

**Mesh doesn't appear:** Make sure the mesh asset built successfully. Check the Output tab for errors.

**Scene is still dark:** Verify the DirectionalLight entity was created and is a child of Root.

**Can't see the mesh:** It might be very small or very large. Try pressing **F** with the mesh selected to frame it in the viewport.
