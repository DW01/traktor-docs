---
layout: default
title: Getting Started
nav_order: 2
---

## Getting Traktor

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

## Example Projects

- [Kartong](https://github.com/apistol78/kartong) – Simple split screen Mario Kart inspired game.
- [Kobolt](https://github.com/apistol78/kobolt) – Simple multiplayer shooter

---

Next: [Engine Architecture](architecture.md)
