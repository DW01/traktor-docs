---
layout: default
permalink: /editor/scene-editor/
title: Scene Editor
parent: Editor
nav_order: 5
---

# Scene Editor - Building Your World

![TODO: Screenshot of the Scene Editor showing a 3D viewport with a scene containing terrain, objects, and lighting. Show the Entities panel with layer hierarchy on the left and Properties panel on the right]

The Scene Editor is where imagination becomes playable space. This is where you place characters, arrange environments, position lights, and compose the 3D worlds your players will explore. Think of it as a stage where you're the director—positioning actors (entities), setting up lighting, and choreographing the scene.

The Scene Editor combines a 3D viewport for visual editing with panels for organization and properties. You can fly through your scene in the viewport, click to select objects, and drag them around. Every change is immediate—move a light, and shadows update in real-time. Adjust a material property, and the object reflects it instantly.

## Interface Panels

**Entities Panel** shows your scene's structure as a tree. Every object in your scene—characters, lights, cameras, effects—appears here as an entity. You can organize entities into layers (like "Environment", "Gameplay", "UI"), nest them in parent-child hierarchies, and enable/disable them for testing. Select an entity here or click it in the viewport—both work.

**Properties Panel** displays everything about the selected entity. Position, rotation, scale? Here. Mesh, material, physics settings? All here. Edit properties in real-time and see changes immediately in the viewport. The panel adapts based on what's selected—a light shows intensity and color, a mesh shows geometry and materials.

**Dependencies Panel** reveals what assets your scene uses. Wondering which textures are loaded? Check here. Trying to optimize a scene? See which meshes and materials are referenced. This transparency helps manage complexity and track down performance issues.

**Guides and Visualization** provide control over what you see in the viewport. Enable a grid for snapping objects to exact positions. Add reference guides for alignment. Measure distances between objects. Beyond basic guides, you can toggle visualization for technical elements: navigation meshes, skeletal rigs, entity bounding boxes, light volumes and ranges, physics shapes and joints, cloth simulation, animation controller state, terrain heightfields, and more. These visualization options let you see the invisible systems at work—turn on physics shapes to debug collisions, show navigation meshes to verify AI pathfinding, or display skeleton bind poses when rigging characters.

## Working with Entities

Building a scene is simple: create entities, add components, and configure properties.

**Creating entities:** Right-click in the Entities panel (or directly in a layer), select "Add Entity", and name it. The entity appears in the tree, ready for components.

**Adding components:** Entities are containers; components define what they do. Every entity has a **transform property** by default (position, rotation, scale), so you can immediately place and orient it. Select an entity, click "Add Component" in the Properties panel, and choose a type. Add a **Mesh** component to render geometry. Add a **Light** component to illuminate the scene. Add a **Script** component to bring it to life with behavior. Add **Physics** components for collisions and movement. Add an **Audio** component for sounds.

**Common components you'll use:**
- **Mesh** - Renders 3D geometry with a material
- **Light** - Directional, point, or spot light sources
- **Script** - Lua scripts for behavior and logic
- **Physics** - Rigid bodies, colliders, character controllers
- **Audio** - 3D sound emitters

## Layers for Organization

As scenes grow, they get messy. **Layers** bring order. Group related entities into layers—"Environment" for props and terrain, "Gameplay" for interactive objects, "Lighting" for lights. Toggle a layer's visibility to focus on specific parts of your scene. Disable a layer to test without certain elements. Layers are organizational tools that scale with your project.

## CSG - Building Geometry from Primitives

Sometimes you need simple geometry—platforms, walls, trigger volumes—without modeling in an external tool. **CSG (Constructive Solid Geometry)** lets you create meshes from basic shapes using boolean operations.

Create an entity, add a **Group Component** and **Solid Component**, then add child entities with **Primitive Components** (boxes, spheres, cylinders). Configure operations—**Union** combines shapes, **Subtract** cuts holes, **Intersect** keeps only overlapping areas. The result is a compound mesh generated procedurally, perfect for blockouts and prototypes.

## See Also

- [World System](../engine/world/) - Entities and components in the engine
- [Scripting](../engine/scripting/) - Adding behavior to entities
- [Database](database/) - Managing scene assets
