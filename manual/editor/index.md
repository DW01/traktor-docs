---
layout: default
title: Editor
parent: Manual
nav_order: 1
---

# Editor

# Traktor Editor

The **Traktor Editor** is a standalone application that allows users to author, organize, and preview game content. It acts as the central hub for working on Traktor-based game projects.

---

## Purpose

The editor exists to streamline the game development workflow by providing tools to:

- Import and manage assets
- Author and preview scenes
- Inspect running games
- Build and export projects
- Animate and configure entities

---

## Core Features

### Asset Database
- All imported assets (models, textures, audio, fonts, etc.) are tracked in the editor’s **asset database**.
- Assets are stored in editor-friendly formats.
- Assets can be **built** into engine-friendly formats used at runtime.

### Asset Building
- The editor performs **conversion and optimization** of assets:
  - Images → compressed GPU textures
  - Models → runtime meshes
  - Audio → streamable or memory-loaded formats
- The build process supports batching and handles dependencies automatically.

### Scene Authoring
- Scenes are composed visually using:
  - Entities
  - Components (e.g., Sprite, Camera, Light)
  - Hierarchical transforms
- Scenes can be **previewed live** in the editor.
- Supports drag-and-drop authoring and editing via the scene graph.

### Game Preview & Inspection
- Users can launch a **preview of the running game** directly within the editor.
- Live inspection of entities and components is supported.
- Useful for debugging and iteration.

### Animation Tools
- TODO

### Project Export
- Final builds can be **exported for deployment**.
- The output contains only runtime-ready assets and binaries.