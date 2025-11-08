---
layout: default
permalink: /editor/plugins/
title: Plugins
parent: Editor
nav_order: 7
---

# Plugins - Extending the Editor

The Traktor Editor isn't a monolithic application—it's a modular platform built on a clean plugin architecture. Most of the editor's functionality, including asset editors and tools, is implemented as plugins. This means extending the editor is natural rather than hacky.

Want to add support for a new asset type? Write a plugin. Need a custom import tool for your studio's file format? Plugin. Want to integrate external tools into the editor? Plugin. The architecture makes extension feel like a first-class feature, not an afterthought.

## What Plugins Can Do

**Editor Plugins** add new tools and windows—a custom batch processor, a level analyzer, a performance profiler. They integrate into the editor's menu and toolbar, looking like built-in features.

**Asset Editors** handle specific asset types. The Scene Editor, Shader Graph, Material Editor—all plugins. You can create custom editors for game-specific assets using the same patterns.

**Pipeline Processors** extend the build system. Add custom asset types with specialized cooking logic. The pipeline system is extensible by design, making it easy to add new converters.

**Import/Export Tools** bring data into and out of the editor. Import FBX models, export scene data to custom formats, integrate with external tools—all possible through plugins.

Plugins implement the `IEditorPlugin` interface, giving them access to the editor's core systems. They can register asset types, add UI elements, integrate with the pipeline, and add settings pages. The plugin system is designed for cleanliness and modularity, with hot-reload support during development.

## See Also

- [Architecture](../engine/architecture/) - Understanding the modular architecture
- [Pipeline](pipeline/) - Extending the asset pipeline
