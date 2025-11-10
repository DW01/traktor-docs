---
layout: default
permalink: /editor/settings/
title: Settings
parent: Editor
nav_order: 10
---

# Settings - Configuring the Editor

The Settings dialog (accessed via **Edit → Settings**) contains configuration options for various aspects of the editor. Settings are organized into tabs covering deployment, appearance, modules, shortcuts, and system-specific options.

## Deploy

The Deploy tab controls how the editor deploys and launches game projects.

**Remote database port** - Port used by running games to access content through the editor. This enables hot-reloading of assets while the game is running.

**Target manager port** - Port used by games to communicate runtime information back to the editor (frame rate, memory usage, object counts, etc.).

**Publish active editor** - When enabled, adds the ID of the currently opened object into the game's configuration file. Useful for debugging specific assets.

**Environment variables** - Deploy tools use environment variables to specify target hosts for different platforms (Android devices, remote Linux machines, iOS simulators, etc.).

## Colors

The Colors tab lets you customize preview colors used throughout the editor.

Double-click on any color row to open the color picker and modify the value. These colors affect how various elements are displayed in the 3D viewports and editor panels.

## General

The General tab contains essential editor settings:

**Source database** - Connection string to the source database where you edit assets.

**Output database** - Connection string to the output database where built/cooked assets are stored.

**Asset path** - Common path to source assets. This allows the overall project root to differ across machines or branches without modifying individual assets.

**Dictionary** - Language dictionary used for the editor interface. The editor supports multiple languages for localization.

**Auto-save opened instances before build** - Automatically saves all open assets before building content.

**Build when source modified** - Automatically builds content when a source object is modified and saved. Enables instant iteration.

**Build when asset modified** - Automatically builds content when a source asset is modified externally (e.g., textures edited in Photoshop).

## Modules

The Modules tab lists all modules currently loaded by the editor. Since the editor relies on modules to implement and extend functionality, this tab shows which systems are active. You can add or remove modules here to customize the editor's capabilities.

## Shortcuts

The Shortcuts tab allows you to modify keyboard shortcut bindings for almost all operations in the editor and its modules.

To change a shortcut:
1. Select the shortcut you wish to modify from the list
2. Click in the input field below the list (it turns green when focused)
3. Press the desired keyboard combination
4. The shortcut is now updated

## Renderer

This tab specifies which render system to use for previewing in the editor. Settings include:

**Renderer backend** - Choose the rendering backend for viewport previews

**Texture filtering** - Quality of texture filtering in previews

**MSAA** - Multi-sample anti-aliasing settings for smoother edges

## Pipeline

Settings for the caching mechanism used by the pipeline to reduce build times.

The pipeline can use **memcached** (a network-based protocol for storing data in fast volatile storage on a server) or a **file-based cache**. Configuration for both caching methods is found in this tab.

Caching significantly speeds up rebuilds by avoiding redundant processing of unchanged assets.

## Scene

A few settings specific to the Scene Editor are found on this tab, controlling default behavior, grid settings, and viewport preferences.

## Audio

The Audio tab contains settings for preview audio playback in the editor. This controls how sounds are played when you preview audio assets or test audio in the Scene Editor.

## See Also

- [Editor Overview](index.md) - Main editor documentation
- [Pipeline](pipeline.md) - Asset build system
- [Targets](targets.md) - Deployment configuration
