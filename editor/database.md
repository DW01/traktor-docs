---
layout: default
permalink: /editor/database/
title: Database
parent: Editor
nav_order: 1
---

# Database - Your Project's Library

![TODO: Screenshot of the Database window showing the hierarchical tree structure with Source section expanded with example assets]

Think of the Database as your project's library—an organized, searchable catalog of everything in your game. Every texture, every script, every scene, every sound lives here. The Database isn't just a file browser; it's an intelligent asset management system that tracks dependencies, manages references, and keeps your project organized as it grows from a prototype to a shipping game.

The Database shows a hierarchical tree view of your assets, much like a file explorer, but with superpowers. It understands relationships between assets—when you reference a texture in a material, the Database knows. When you delete something, it warns you if other assets depend on it. When you rename an asset, all references update automatically.

## Understanding Database Files

Behind the scenes, the Database uses a few special file extensions. You don't usually interact with these directly—the editor handles them—but understanding what they mean helps when working with version control or troubleshooting:

**`.xdi` files** (Database Instance) contain your actual asset data—the scene layout, the material properties, the script code. This is the "real" asset.

**`.xdm` files** (Database Metadata) store information about the asset—its type, its unique ID, its dependencies. The editor uses this to track relationships and build the dependency graph.

**`.xil` files** (Instance Link) are references to other assets. Instead of duplicating an asset, you create a link that points to the original. Change the original, and all links reflect that change.

**`.xgl` files** (Group Link) are references to entire folders/groups. Useful for organizing large projects with shared asset libraries.

## Database Structure

The Database contains all your project's assets organized in a hierarchical structure. Everything you create and edit lives in the **Source** section—scenes, materials, textures, scripts, audio files, and more. Think of it as your workshop: messy, iterative, full of work-in-progress. You organize assets into groups (folders), create new assets, and manage references all within this tree view.

## Working with the Database

The Database workflow is intuitive—right-click to create, drag-and-drop to organize, double-click to edit.

**Creating folders (called Groups):** Right-click anywhere in the Database, select "New Group", and name it. Press F2 to rename later. Use groups to organize related assets—a "Characters" group for all character assets, an "Environments" group for levels and props.

**Creating assets:** Right-click on a group, select "New Instance", choose the asset type (Scene, Material, Script, etc.), and name it. The editor creates the asset in the Database. To edit the asset, double-click it to open the appropriate editor—for example, double-clicking a Material opens the Material Editor where you can assign textures and configure properties.

**Organizing:** Drag and drop assets between groups to reorganize. Create hierarchies that make sense for your project. Use instance links (`.xil` files) when you need to reference the same asset in multiple places—change the original, and the links automatically update.

## See Also

- [Pipeline](pipeline/) - How assets are built and optimized
- [Scene Editor](scene-editor/) - Editing 3D scenes
- [Shader Graph](shader-graph/) - Creating materials
