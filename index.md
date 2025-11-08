---
layout: default
permalink: /
title: About Traktor
nav_order: 1
---

# About Traktor

Welcome to Traktor — a modular, C++ game engine built for high-performance real-time applications.

Traktor is an open-source 3D game engine written in C++. It has powered several successful commercial titles across various platforms including Steam, PSN, iOS, and the Mac Store.

## Why Traktor?

Traktor isn't just another game engine; it's a meticulously crafted toolkit designed for optimal performance and flexibility. Here's why it stands out:

- **Lean and Well-Designed:** Each module boasts clean responsibilities, ensuring an elegant and efficient codebase.
- **Minimal Footprint:** Traktor prioritizes low memory and storage usage, ensuring streamlined performance.
- **Optimized for Efficiency:** Utilizing modern rendering techniques and efficient algorithms, Traktor delivers exceptional performance.
- **Tailored Flexibility:** Unlike bloated, one-size-fits-all engines, Traktor is customizable to fit the unique needs of every project.

---

## Supported Platforms

Traktor currently offers full support for Windows and Linux platforms. While mobile versions for Android and iOS are available, they are not officially tested but are regularly updated to maintain compatibility.

**Editor:**
- Windows
- Linux

**Runtime:**
- **Primary Support:** Windows, Linux
- **Partially Working:** Android, iOS, macOS

---

## Key Features

### Editor
- **Feature-rich editor** with editor-first priority
- **One-click deploy** - Run and debug on any connected target
- **Hot reloading** - Active connection to running games with real-time asset updates
- **Asset Pipeline** - Highly parallelized build system with incremental builds
- **Multiple themes** - Light, dark, and customizable themes

### Rendering
- **Vulkan-based** - Modern frame graph renderer
- **Ray Tracing** - RTGI, RTAO, RT reflections, RT shadows
- **ReSTIR GI** - Advanced global illumination
- **Shader Graph** - Node-based visual shader editor with instant reload
- **Deferred & Forward+** - Flexible rendering paths
- **GPU Skin Cache** - Optimized character rendering

### Scripting
- **Lua** as primary scripting language
- **Integrated debugger** and profiler
- **Hot-reload** for rapid iteration

### Audio
- **HD Audio Pipeline** - Support for 2.0, 2.1, 5.1, 7.1, and custom configurations
- **Multiple backends** - XAudio2, DirectSound, OpenAL, ALSA, Pulse, etc.
- **Graph-based filters** for audio processing
- **Streaming** - MP3, FLAC, OGG support

### Physics
- **Jolt and Bullet** physics engines
- **Character controller** - Easy-to-use character physics
- **Vehicle simulation** - Advanced vehicle controller

---

## Documentation

### [Getting Started](getting-started/)
Learn how to build Traktor, create your first project, and start making games.

### [Engine Documentation](engine/)

Understand and use Traktor's core systems:

- **[Architecture](engine/architecture/)** - How the engine is designed and organized
- **[Scripting](engine/scripting/)** - Write gameplay code in Lua
- **[Rendering](engine/render/)** - Modern Vulkan-based graphics
- **[Physics](engine/physics/)** - Rigid bodies and character controllers
- **[World System](engine/world/)** - Entities and components
- **[And more...](engine/)** - Audio, animation, AI, networking, and all the systems

### [Editor Documentation](editor/)

Master the Traktor editor and your workflow:

- **Interface** - Understand the editor's powerful tools
- **Database** - Manage your project assets
- **Scene Editor** - Build your game worlds
- **Shader Graph** - Create materials visually
- **Pipeline** - Convert assets to optimized runtime formats
- **Deploy & Debug** - One-click deployment to any platform

---

## Community

- **GitHub:** [https://github.com/apistol78/traktor](https://github.com/apistol78/traktor)
- **Discord:** [https://discord.gg/fSMrww2B7C](https://discord.gg/fSMrww2B7C)
- **Developer:** [@apistol78](https://github.com/apistol78)

---

## License

Traktor is open source under the Mozilla Public License 2.0 (MPL 2.0).

---

## Known Issues & Experimental Features

- Forward pipeline lacks certain reflection features
- Some Vulkan SDK versions may show validation warnings
- UI scaling on high-DPI screens (work in progress)
- Mobile platforms (Android/iOS) are experimental
