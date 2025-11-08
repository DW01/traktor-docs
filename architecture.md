---
layout: default
permalink: /architecture/
title: Engine Architecture
nav_order: 3
---

# Engine Architecture

Traktor is a modular, cross-platform game engine designed with a layered architecture. It separates editor and runtime systems while promoting code reuse through general-purpose libraries. The structure of the engine enables clean separation of concerns, extensibility, and adaptability across platforms.

---

## Layered Architecture

Traktor is structured into several architectural layers:

### 1. **Core Layer**

At the base is the **Core library**, which provides fundamental building blocks used by all other modules.

**Core includes:**
- Custom container types
- Platform abstractions (Android, Linux, Windows, macOS)
- Date/time utilities
- File I/O utilities
- Debugging and logging
- Math utilities
- Memory management
- RTTI and serialization
- Threading primitives

The core library is platform-aware and provides consistent behavior across supported targets.

---

### 2. **General Modules**

Above the core layer are a collection of **general-purpose libraries**. These contain reusable systems that can be consumed by both the **runtime** and the **editor** layers.

Examples include:
- **AI** – Navmesh, pathfinding
- **Animation** – Skeletal animation, inverse kinematics (IK), ragdoll, cloth simulation
- **Heightfield**
- **Input**
- **Mesh** / **Model**
- **Physics**
- **Render**
- **Resource**
- **Scene**
- **Script** – Based on RTTI (currently Lua backend only)
- **Sound**
- **Spark** – In-game UI system
- **Spray** – Particle system
- **Terrain**
- **Theater** – Scene animation
- **Video**
- **Weather** – Sky systems, precipitation
- **World** – Scene graph, entity and component management

Each general module may have one or both of the following specialized variants:

---

### 3. **Runtime Counterparts**

The **runtime** layer is what runs your game or application.

It is built using:
- The **core** layer
- **General modules**
- **Runtime-specific extensions** of those modules

The runtime defines a top-level `Application` class, which orchestrates the engine via a series of **servers**.

**Core Runtime Servers:**
- **AudioServer**
- **InputServer**
- **OnlineServer**
- **PhysicsServer**
- **RenderServer**
- **ResourceServer**
- **ScriptServer**
- **WorldServer**

The runtime supports a **plugin system**, allowing developers to extend engine functionality with custom logic.

It also introduces the concepts of:
- **Stages** – A stage in Traktor corresponds to a high-level state of the game.
- **Layers** – Compose different features or systems active in that state.

---

### 4. **Editor Counterparts**

The **editor** layer is a distinct application built on:
- The **core**
- **General modules**
- **Editor-specific extensions**

The editor reuses the same core systems but integrates editor-only tools such as:
- Scene editing
- Asset import and conversion
- Project export
- Live preview and inspection

Like the runtime, the editor supports **plugins**, allowing custom tools or panels to be integrated into the workflow.

---

## Data Flow

### Asset Flow:

Raw Assets → Editor Asset Database → Built Runtime Assets → Packaged Game

---

## 🧪 Summary

- **Core** = low-level platform and utility layer
- **General** = reusable systems for both runtime/editor
- **Runtime** = actual game engine; has servers, plugins, world logic
- **Editor** = authoring tool; built using editor-specific module variants
- **Plugins** = available for both runtime and editor

This modular, layered architecture makes Traktor flexible, extensible, and easy to port or scale across projects and platforms.

---

## Learn More

For a comprehensive deep dive into Traktor's architecture, including:
- Object system and reference counting
- Module dependencies
- Memory management
- Multithreading and job system
- Plugin architecture
- Coding conventions

See the [Engine Architecture Manual](manual/engine/architecture/).
