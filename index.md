---
layout: default
permalink: /
title: About Traktor
---


# About Traktor

Welcome to Traktor — a modular, C++ game engine built for high-performance real-time applications.

Traktor isn't just another game engine; it's a meticulously crafted toolkit designed for optimal performance and flexibility.
Here's why it stands out:
- **Lean and Well-Designed:** - Each module boasts clean responsibilities, ensuring an elegant and efficient codebase.
- **Minimal Footprint: Traktor** - prioritizes low memory and storage usage, ensuring streamlined performance.
- **Optimized for Efficiency:** - Utilizing modern rendering techniques and efficient algorithms, Traktor delivers exceptional performance.
- **Tailored Flexibility:** - Unlike bloated, one-size-fits-all engines, Traktor is customizable to fit the unique needs of every project.

---

## Supported Platforms

Traktor currently offers full support for Windows and Linux platforms. While mobile versions for Android and iOS are available, they are not officially tested but are regularly updated to maintain compatibility.

## Key Features

- **Powerful Editor** – Built-in editor with live asset previews, in-game UI editing, and scene management.
- **Runtime Engine** – High-performance modular engine runtime.
- **Flexible Rendering** – Forward, deferred, and simple rendering pipelines with Vulkan backend.
- **Asset Pipeline** – Data-driven workflow with content processing and asset caching via Avalanche.
- **Modular Design** – Each system (UI, sound, animation, etc.) is implemented as an independent module.
- **In-Game UI** – Scalable vector UI via `UiKit`, inspired by Flash but tailored for games.

Traktor is developed and maintained by [@apistol78](https://github.com/apistol78) and is open source under the MPL license.

---

## Editor and Runtime

### Traktor Editor
The editor is a standalone application that allows you to:
- Browse and manage assets
- Edit scenes visually
- Preview UI and gameplay scripts
- Configure rendering settings, pipelines, and stages

### Traktor Runtime
The runtime is a lightweight standalone application that loads scenes, stages, and modules defined by your project. You can:
- Launch the game from within the editor or build separately
- Extend functionality with native code or scripts

---

## 🧪 Experimental / Known Issues

- Forward pipeline lacks reflections
- MSAA causes artifacts in deferred mode
- Shader validation errors with some Vulkan SDKs
- UI scaling issues on high-DPI screens
- Some features (e.g. Theater) lack UI support

