---
layout: default
permalink: /editor/
title: Editor
nav_order: 4
has_children: true
---

# Traktor Editor

The Traktor Editor is where you build, configure, and test your game. It provides a complete development environment with tools for scene editing, asset management, shader creation, and multi-platform deployment. The editor is tightly integrated with the engine runtime, enabling hot-reloading of assets—when you save changes, they appear in your running game within milliseconds without requiring a restart.

![TODO: Screenshot of Traktor Editor main interface showing the layout with Database panel on left, Scene Editor in center, Properties panel on right, and toolbar at top]

The Traktor Editor follows an "editor-first" philosophy: instead of bolting editing tools onto an existing runtime, the entire engine is built around a powerful, professional editor. This means deep integration, hot-reloading that actually works, and tools that feel cohesive rather than tacked on. You edit, save, and instantly see your changes in the running game—no lengthy rebuilds, no context switching.

## Core Tools

- [Database](database/) - Manage project assets and dependencies
- [Pipeline](pipeline/) - Asset build system and optimization
- [Scene Editor](scene-editor/) - Build 3D worlds with entities and components
- [Shader Graph](shader-graph/) - Visual shader creation

## Workflow

- [Output](output/) - Monitor pipeline builds and logs
- [Audio](audio/) - Control editor audio volume
- [Targets](targets/) - Deploy to multiple platforms
- [Logging](logging/) - Real-time debugging and logs

## Extensibility

- [Plugins](plugins/) - Extend the editor with custom tools

## Key Features

**Instant Feedback:** Change a texture, tweak a script, adjust a material—save it and watch it update in your running game within milliseconds. No waiting for compiles, no restarting, just immediate iteration. This hot-reloading system maintains an active connection to your game, streaming changes as you work.

**Full Undo/Redo:** Made a mistake? Press Ctrl+Z. Every editor, every panel, every action supports undo and redo. You can experiment fearlessly knowing you can always step back.

**One-Click Deploy:** When you're ready to test on a different platform, click one button. The editor builds your assets, packages everything, deploys to the target device, and launches the game—all automatically. Test on Windows, Linux, Android, or iOS without leaving the editor.

**Customizable Interface:** The interface uses a modular, dockable window system—every panel can be moved, resized, and arranged to match your workflow. The editor offers light and dark themes for different lighting conditions and preferences.

**Localization Built In:** The editor itself supports multiple languages, and your game can too. Internationalization (i18n) is a first-class feature, not an afterthought.
