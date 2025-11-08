---
layout: default
permalink: /
title: About Traktor
nav_order: 1
---

# About Traktor

Traktor is an open-source game engine written in C++ with a focus on modularity, performance, and developer workflow. It provides a complete development environment—a professional editor with hot-reloading, a modern Vulkan-based renderer with ray tracing, and a flexible architecture designed to be extended and customized.

The engine has shipped commercial games on Steam, PlayStation Network, iOS, and Mac, proving itself in real-world production. Traktor follows an "editor-first" philosophy: rather than treating the editor as an afterthought, the entire engine is built around a powerful editing workflow. Changes you make appear in your running game within milliseconds, making iteration fast and development efficient.

## What Makes Traktor Different

**Modular architecture:** Traktor is organized into clean, independent modules—Core provides fundamentals, specialized modules handle rendering, physics, audio, and scripting, and you can extend or replace systems as needed. This modularity means you're not locked into a monolithic engine where everything is tightly coupled.

**Low overhead:** The engine prioritizes minimal memory and storage footprint. Code is lean, systems are efficient, and you won't find unnecessary abstractions or bloat. Traktor aims to stay out of your way and let you build what you need.

**Modern rendering:** The Vulkan-based renderer includes advanced features—ray-traced global illumination, ambient occlusion, reflections, and shadows using ReSTIR GI for high-quality lighting. A visual shader graph makes material authoring approachable while still allowing custom GLSL when needed.

**Developer-friendly workflow:** Hot-reloading works across assets—textures, scripts, shaders, scenes—so you see changes instantly. The pipeline is highly parallelized, using all CPU cores for fast builds. One-click deployment lets you test on different platforms without manual setup.

---

## Supported Platforms

Traktor's editor runs on **Windows** and **Linux**. These are the primary development platforms with full support and regular testing.

The runtime (your actual game) has mature support for **Windows** and **Linux** as well. **Android**, **iOS**, and **macOS** builds are functional but considered experimental—they receive updates and should work, but aren't tested as rigorously as the desktop platforms. If you're targeting mobile or macOS, expect to do more testing and potentially encounter platform-specific issues.

---

## Core Systems

### Editor and Workflow

The Traktor editor is designed for productivity. It maintains an active connection to your running game, streaming asset changes in real-time—modify a texture, adjust a script, tweak a material, and watch it update within milliseconds without restarting. One-click deployment builds your assets, packages the game, deploys to any connected target (Windows, Linux, Android, iOS), and launches with a live debugger connection. The asset pipeline is highly parallelized, using all CPU cores for fast incremental builds—only modified assets rebuild, not the entire project. The editor supports light, dark, and custom themes to match your preferences and lighting environment.

### Rendering

Traktor uses a modern Vulkan-based frame graph renderer. Ray tracing support includes ray-traced global illumination (RTGI), ambient occlusion (RTAO), reflections, and shadows, with ReSTIR GI providing advanced global illumination quality. The shader graph is a node-based visual editor where you wire nodes together to create materials, with instant hot-reload when you make changes. If you need low-level control, you can drop into GLSL code within the graph. The renderer supports both deferred and Forward+ rendering paths, and includes a GPU skin cache for efficient character rendering.

### Scripting

**Lua** is the primary scripting language for gameplay logic. An integrated debugger lets you set breakpoints, step through code, and inspect variables. Hot-reload works for scripts—edit your Lua code, save, and the changes appear in your running game immediately. A built-in profiler helps identify performance bottlenecks in your script code.

### Audio

The HD Audio pipeline supports multichannel configurations: stereo (2.0), 2.1, 5.1, 7.1, and custom speaker arrangements. Multiple audio backends are available depending on your platform—XAudio2 and DirectSound on Windows, OpenAL, ALSA, and PulseAudio on Linux. Graph-based audio filters provide processing and effects. Streaming audio supports MP3, FLAC, and OGG formats for music and long sound files, keeping memory usage low.

### Physics

Traktor supports two physics engines: **Jolt** (modern, high-performance) and **Bullet** (mature, widely used). You can choose the backend that fits your project. A character controller provides easy-to-use character physics with ground detection, step climbing, and sliding. An advanced vehicle controller simulates realistic vehicle dynamics for racing games or driving mechanics.

---

## Documentation

**[Getting Started](getting-started/)** walks you through building Traktor from source, creating your first project, and understanding the basic workflow. If you're new to the engine, start here.

**[Engine Documentation](engine/)** covers the runtime systems—the code that powers your game. Learn about [architecture](engine/architecture/) and how the engine is organized, [scripting](engine/scripting/) with Lua for gameplay logic, [rendering](engine/render/) with the Vulkan-based graphics system, [physics](engine/physics/) for rigid bodies and character movement, and the [world system](engine/world/) that handles entities and components. Additional systems include [audio](engine/audio/), [animation](engine/animation/), [AI navigation](engine/ai/), [networking](engine/networking/), and more.

**[Editor Documentation](editor/)** teaches you the tools for building your game. Understand the editor interface and its dockable panels, manage assets in the Database, build 3D worlds in the Scene Editor, create materials with the visual Shader Graph, and use one-click deployment to test on any platform. The editor is where you'll spend most of your time, so mastering its workflow is essential.

---

## Community and Resources

Traktor is developed by [@apistol78](https://github.com/apistol78) and the open-source community. The source code is hosted on **[GitHub](https://github.com/apistol78/traktor)** where you can browse the codebase, report issues, and contribute improvements.

Join the **[Discord community](https://discord.gg/fSMrww2B7C)** to ask questions, share projects, get help with development, and connect with other Traktor developers. The Discord is the fastest way to get support and discuss the engine.

## License

Traktor is open source under the **Mozilla Public License 2.0 (MPL 2.0)**. This means you can use Traktor for commercial projects, modify the source code, and integrate it into your workflow. Any modifications to Traktor's source files must be made available under MPL 2.0, but your game code and assets remain yours.

## Known Limitations

Some areas are still being refined:

**Forward rendering pipeline** lacks certain reflection features available in the deferred path. If you need advanced reflections, use the deferred renderer.

**Vulkan SDK validation** may show warnings with certain SDK versions. These are typically non-critical but can be noisy during development.

**High-DPI scaling** in the editor UI is a work in progress. On high-resolution displays, interface elements may not scale perfectly.

**Mobile platforms** (Android and iOS) are functional but experimental. They receive updates but aren't tested as thoroughly as Windows and Linux, so expect platform-specific issues.
