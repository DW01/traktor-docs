---
layout: default
permalink: /getting-started/
title: Getting Started
nav_order: 2
has_children: true
---

# Getting Started with Traktor

## What is Traktor?

Traktor is an open-source 3D game engine written in C++. It has powered several successful commercial titles across various platforms including Steam, PSN, iOS, and the Mac Store.

## Requirements

- **Graphics:** Vulkan 1.3 capable graphics card
- **Operating System:** Windows or Linux
- **Build Tools:** Visual Studio 2019+ (Windows) or GCC/Clang (Linux)

---

## Installation

### Option 1: Download Binary

Download the latest binary version from [GitHub Releases](https://github.com/apistol78/traktor/releases).

**Note:** Releases may not be the latest version. Building from source is recommended for the latest bug fixes and features.

### Option 2: Build from Source

1. **Clone the repository:**
   ```bash
   git clone --recursive https://github.com/apistol78/traktor.git
   cd traktor
   ```

2. **Follow platform-specific build instructions:**
   - [Build on Linux](./build-linux/)
   - [Build on Windows](./build-windows/)

---

## Create Your First Project

### Creating a New Workspace

1. Launch the Traktor editor
2. Go to **File → New Workspace**
3. Select **Desktop** template
4. Enter your project name
5. Choose a project path/location
6. Click **OK**

### Project Structure

After creation, your project will have this structure:

```
ProjectName/
├── ProjectName.workspace
├── data/
│   ├── Output/          # Built/cooked assets
│   ├── Source/          # Source assets
│   │   ├── Input/
│   │   ├── Scripts/
│   │   ├── Stages/
│   │   └── ...
│   └── Temp/            # Temporary build files
└── info/
```

**File Extension Reference:**
- `.xdi` - Database Instance
- `.xdm` - Database Metadata
- `.xil` - Instance Link
- `.xgl` - Group Link

---

## Creating Your First Scene

### 1. Create a Scene Asset

1. Locate the **Database** tab in the editor
2. Right-click on **Source** and select **New Group**
3. Rename to **Scenes** (press F2)
4. Right-click **Scenes** and select **New Instance**
5. Select **Scene** category and **SceneAsset** type
6. Name it "MyScene" and click **OK**

### 2. Edit the Scene

1. Double-click **MyScene** in the Database
2. Wait for editor assets to build (first time only)
3. The scene will open (initially empty and dark)

### 3. Add Lighting

Entities are organized in **layers**. Create an **Environment** layer:

1. Create a layer named **Environment**
2. Add entities:
   - **Sun** - Directional lighting
   - **Sky** - Skybox/atmosphere
   - **Probe** - Environment reflection

### 4. Add Geometry

Create an **Objects** layer and add geometry:

**Note:** Traktor doesn't have built-in primitives. Use CSG (Constructive Solid Geometry):

1. Create an entity named "MyCube"
2. Add **Group Component**
3. Add **Solid Component**
4. Create child entity "Box Shape"
5. Add **Primitive Component** to child
6. Set operation to **Union** and shape to **BoxShape**

---

## Next Steps

- Read the [Architecture](../engine/architecture/) guide to understand how Traktor is designed
- Explore the [Editor Documentation](../editor/) to master your workflow
- Learn about the [Engine Systems](../engine/) for scripting, physics, and rendering
- Join the [Discord community](https://discord.gg/fSMrww2B7C) for help and discussions

---

## Example Projects

Check the Traktor repository for example projects and tutorials that demonstrate various features.
