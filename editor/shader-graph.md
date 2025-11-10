---
layout: default
permalink: /editor/shader-graph/
title: Shader Graph
parent: Editor
nav_order: 6
---

# Shader Graph - Visual Shader Creation

![TODO: Screenshot of the Shader Graph editor showing a node graph with various shader nodes connected together (texture sampling, math operations, vertex/fragment output). Include the node palette on the side]

Shaders define how surfaces look. Their colors, reflections, textures, and lighting response. Traditionally, shaders are written in code (GLSL or HLSL), which works but requires programming expertise and makes iteration slow. The **Shader Graph** takes a different approach: visual, node-based shader creation where you connect boxes to build complex materials.

Think of it like wiring together audio equipment. Each node is a device (texture sampler, math operation, color mixer), and you connect them with cables (node connections) to create the final output. No code required, though you can add inline GLSL/HLSL when you need low-level control.

## Why It's Powerful

**Visual and Intuitive:** Instead of typing "texture2D(sampler, uv) * color", you connect a Texture node to a Multiply node to a Color node. You see the flow of data visually, making shaders easier to understand and debug.

**Instant Feedback:** Change a node parameter, and the material updates in real-time. Move a slider for metallic, watch the reflections adjust instantly. This tight feedback loop makes shader development feel creative rather than tedious.

**All Shader Stages:** The graph supports **vertex shaders** (manipulate geometry), **fragment shaders** (compute pixel colors), and **compute shaders** (general GPU computing). You're not limited to basic materials. You can create advanced effects like tessellation, procedural geometry, or custom lighting models.

**Platform Agnostic:** The graph compiles to Vulkan SPIR-V, meaning your shaders work across Windows, Linux, Android, and any platform Vulkan supports. Write once, run everywhere.

**Custom Code When Needed:** Need something the built-in nodes don't provide? Add an **inline code node** with raw GLSL/HLSL. The shader graph doesn't lock you into visual-only. It gives you escape hatches when necessary.

## Creating Shaders

Create a new **ShaderAsset** in the Database and double-click it. The Shader Graph editor opens, showing an empty canvas and a node palette. Drag nodes from the palette onto the canvas. Texture samplers, math operations, vertex position modifiers, fragment color outputs. Connect nodes by dragging from one output port to another input port. Configure each node's properties. Preview the result on a sphere or plane in the editor. Save, and the pipeline compiles it to optimized SPIR-V.

## See Also

- [Rendering](../engine/render/) - The rendering system that uses shaders
- [Pipeline](pipeline/) - How shaders are compiled
- [Database](database/) - Managing shader assets
