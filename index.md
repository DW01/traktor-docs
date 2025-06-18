---
title: Getting Started
nav_order: 1
---


# Getting Started with Traktor

Welcome to Traktor — a modular, data-driven C++ game engine built for high-performance real-time applications.

Traktor includes its own custom editor, rendering pipelines, animation and audio systems, UI toolkits, and a full asset build pipeline. It is designed with flexibility and extensibility in mind, supporting both 2D and 3D development across multiple platforms.

---

## 🌟 Key Features

- 🔧 **Powerful Editor** – Built-in editor with live asset previews, in-game UI editing, and scene management.
- 🎮 **Runtime Engine** – High-performance ECS-based engine runtime.
- 🎨 **Flexible Rendering** – Forward, deferred, and simple rendering pipelines with Vulkan backend.
- 📦 **Asset Pipeline** – Data-driven workflow with content processing and asset caching via Avalanche.
- 🧩 **Modular Design** – Each system (UI, sound, animation, etc.) is implemented as an independent module.
- 🎭 **In-Game UI** – Scalable vector UI via `UiKit`, inspired by Flash but tailored for games.

> Traktor is developed and maintained by [@apistol78](https://github.com/apistol78) and is open source under the MPL license.

---

## 🧭 Editor and Runtime

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

## 📥 Getting Traktor

### 1. Clone the Repository
```bash
git clone --recursive https://github.com/apistol78/traktor.git
```

### 2. Generate Project Files
Launch the generator:

```
Tools/ProjectGen/Traktor.ProjectGen.App.exe
```

### 3. Open in Visual Studio
- Set startup project: `Traktor.Editor.App`
- Build: `DebugShared` or `ReleaseShared`

### 4. Run the Editor
```bash
bin/latest/win64/DebugShared/Traktor.Editor.App.exe
```

---

## 📦 Example Projects

- `Kartong` – First-person cardboard-themed shooter
- `Kobolt` – Experimental mini project

---

## 🧪 Experimental / Known Issues

- Forward pipeline lacks reflections
- MSAA causes artifacts in deferred mode
- Shader validation errors with some Vulkan SDKs
- UI scaling issues on high-DPI screens
- Some features (e.g. Theater) lack UI support

---

Next: [Engine Architecture](architecture.md)
