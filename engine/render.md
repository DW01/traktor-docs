---
layout: default
permalink: /engine/rendering/
title: Rendering
parent: Engine

nav_order: 8
---

# Render System - Painting Your Virtual World

Rendering is what turns all your game data—3D models, textures, lights, and animations—into the beautiful images players see on screen. It's the bridge between the abstract mathematical representation of your world and the vibrant, immersive visuals that make your game come alive.

Think of the render system as a sophisticated camera and darkroom combined. Each frame, it captures your scene from the camera's perspective, processes it through various stages (like developing film), applies effects and filters, and finally presents the polished result. All of this happens 60+ times per second, creating the illusion of smooth, real-time motion.

Traktor's render system is built on **Vulkan**, a modern graphics API that gives you direct control over the GPU. It's powerful and flexible, supporting cutting-edge features like **ray tracing** (simulating actual light rays like in the real world), **physically-based rendering** (materials that look realistic under any lighting), and a **shader graph editor** (visually designing how surfaces look without writing code). Whether you're building stylized cartoon worlds or photorealistic environments, Traktor's renderer has you covered.

![TODO: Screenshot of a rendered scene showing PBR materials, dynamic lighting, shadows, and reflections]

**Note:** This documentation describes features based on the engine's capabilities. Specific API calls shown are illustrative - refer to actual source code in `code/Render/` for exact interfaces.

## The Big Picture: Frame Graphs

Modern rendering isn't a simple linear process. You don't just "draw everything and you're done." Instead, rendering involves multiple passes—shadow maps, geometry, lighting, reflections, post-processing—each building on the previous results. Managing this complexity manually is error-prone and tedious.

That's where the **Frame Graph** comes in. Think of it as a blueprint or recipe for your rendering pipeline. You declare what passes you need and what data flows between them, and the system figures out the optimal execution order, manages GPU resources, and handles synchronization automatically.

```cpp
// Build render graph
void buildRenderGraph(RenderGraph& rg)
{
    // Define passes
    auto shadowPass = rg.addPass("Shadow", ...);
    auto geometryPass = rg.addPass("Geometry", ...);
    auto lightingPass = rg.addPass("Lighting", ...);
    auto postPass = rg.addPass("Post", ...);

    // Define resources and dependencies
    // Graph automatically handles synchronization and layout transitions
}
```

The frame graph automatically inserts GPU barriers (ensuring one pass finishes before another starts), reuses memory where possible (aliasing resources that don't overlap), schedules passes optimally, and makes the data flow crystal clear. You describe *what* you want to render, and the system handles *how* to make it efficient.

## Rendering Strategies

Traktor supports two main rendering approaches, each with different strengths:

**Deferred Rendering** works in stages: first, render all geometry into a **G-Buffer** (geometry buffer) containing albedo, normals, metallic/roughness, and depth. Then, in a separate pass, calculate lighting by reading from the G-Buffer. Finally, apply post-processing. This approach excels when you have many lights—hundreds or even thousands—because lighting is calculated per-pixel rather than per-object. It's also great for consistent material quality and efficient lighting calculations.

**Forward+ Rendering** takes a different approach: it first renders a depth-only pre-pass to establish the Z-buffer, then performs per-tile light culling (figuring out which lights affect which screen regions), and finally renders in a forward pass, shading only with the lights visible in each tile. Forward+ shines when you need transparency (which is tricky in deferred), MSAA anti-aliasing, or lower memory usage.

Both approaches have their place, and Traktor lets you choose based on your project's needs.

## Creating Beautiful Materials

### Shader Graph

The **Shader Graph** is a visual, node-based editor for creating materials. Instead of writing shader code by hand (though you can if you want), you connect nodes together: sample a texture here, multiply by a color there, add some math, and wire it all up to the output. It's intuitive, fast to iterate on, and powerful.

![TODO: Screenshot of shader graph editor showing nodes for textures, math operations, PBR output]

To create a shader:
1. Open the Shader Graph editor in Traktor
2. Add input nodes—texture samples, vertex attributes, uniforms
3. Add processing nodes—math operations, color conversions, utility functions
4. Connect everything to output nodes (fragment color, normals, metallic/roughness)
5. Compile and preview in real-time

The node library includes inputs (textures, vertex data, constants), math operations (add, multiply, dot product, cross product), color utilities (RGB/HSV conversion, color grading), conditional logic (if/else, switches), and specialized outputs for PBR and custom rendering.

### Physically-Based Rendering (PBR)

PBR is a rendering approach based on real-world physics. Instead of faking how materials look with arbitrary parameters, PBR models how light actually interacts with surfaces. The result? Materials that look consistent and realistic under any lighting condition.

A PBR material uses these core properties:

**Albedo** is the base color—what color the material is when lit by pure white light. **Metallic** defines whether the surface is metal (1.0) or dielectric like plastic or wood (0.0). Metals reflect light very differently than non-metals. **Roughness** controls surface smoothness—a smooth surface (low roughness) produces sharp, mirror-like reflections, while a rough surface (high roughness) scatters light and looks matte. **Normal** maps add fine surface detail without additional geometry. **Ambient Occlusion (AO)** darkens crevices where indirect light doesn't reach. **Emissive** makes surfaces glow, emitting their own light.

```cpp
// Material properties
material->setTexture("Albedo", albedoTexture);
material->setTexture("Normal", normalTexture);
material->setTexture("Metallic", metallicTexture);
material->setTexture("Roughness", roughnessTexture);
```

### Custom Shaders

While the Shader Graph handles most cases, sometimes you need to write shader code directly. Traktor supports inline GLSL/HLSL when you need that level of control:

```glsl
// Vertex shader
#version 450

layout(location = 0) in vec3 inPosition;
layout(location = 1) in vec3 inNormal;
layout(location = 2) in vec2 inTexCoord;

layout(location = 0) out vec2 outTexCoord;

void main()
{
    outTexCoord = inTexCoord;
    gl_Position = ...;
}
```

## Lighting Your World

Good lighting makes or breaks a scene. Traktor provides several light types, each suited to different purposes:

**Directional lights** simulate distant light sources like the sun. They illuminate everything from the same direction, perfect for outdoor scenes:

```cpp
Ref<LightComponent> sun = new LightComponent();
sun->setType(LightType::Directional);
sun->setColor(Color4f(1.0f, 0.95f, 0.8f));
sun->setIntensity(5.0f);
sun->setCastShadow(true);
```

**Point lights** emit light in all directions from a point, like a light bulb or torch:

```cpp
Ref<LightComponent> light = new LightComponent();
light->setType(LightType::Point);
light->setRadius(10.0f);
light->setIntensity(10.0f);
```

**Spot lights** emit a cone of light, perfect for flashlights, car headlights, or stage lighting:

```cpp
Ref<LightComponent> spotlight = new LightComponent();
spotlight->setType(LightType::Spot);
spotlight->setInnerAngle(30.0f);
spotlight->setOuterAngle(45.0f);
spotlight->setRange(20.0f);
```

### Shadows

Shadows ground objects in the world and add depth. Traktor supports traditional **shadow maps**—rendering the scene from the light's perspective to determine what's in shadow—using cascaded shadow maps for directional lights (multiple resolution levels for near and far), cube shadow maps for point lights (six-sided maps covering all directions), and perspective shadow maps for spot lights.

For even higher quality, you can enable **ray-traced shadows** that trace actual light rays to determine if a point is in shadow:

```cpp
light->setRayTracedShadows(true);  // Use RT shadows
```

### Global Illumination

Real-world lighting doesn't come from just direct light sources. Light bounces off surfaces, illuminating other surfaces indirectly. This is **global illumination (GI)**, and it's what makes scenes feel truly realistic.

Traktor supports **ReSTIR GI** (Reservoir-based Spatiotemporal Importance Resampling), a cutting-edge real-time GI technique. It handles dynamic lighting—lights can move, and indirect lighting updates in real-time. For even more accuracy, you can enable **ray-traced GI** which simulates light bounces:

```cpp
renderSettings->setRTGI(true);
renderSettings->setRTGIBounces(2);  // Number of light bounces
```

## Ray Tracing: Simulating Reality

Traditional rendering uses tricks and approximations to make things look good. Ray tracing, on the other hand, actually simulates how light works in the real world—tracing rays from the camera through the scene, bouncing off surfaces, and gathering light. The result is stunningly realistic reflections, shadows, and lighting.

Traktor supports hardware-accelerated ray tracing for several effects:

**Ray-traced Ambient Occlusion (RTAO)** calculates how ambient light is blocked by nearby geometry, creating realistic contact shadows and depth:

```cpp
renderSettings->setRTAO(true);
renderSettings->setRTAORadius(1.0f);
```

**Ray-traced Reflections** produce accurate, dynamic reflections on any surface—no pre-baked cube maps needed:

```cpp
renderSettings->setRTReflections(true);
```

**Ray-traced Shadows** trace rays to light sources for pixel-perfect, soft shadows:

```cpp
renderSettings->setRTShadows(true);
```

**Requirements:** Ray tracing needs Vulkan 1.3, an RTX or equivalent GPU, and ray tracing enabled in your project settings. The engine automatically builds acceleration structures (Bounding Volume Hierarchies) from your scene geometry to make ray tracing fast.

## Post-Processing: The Final Polish

Post-processing effects run after the main scene is rendered, adding that final layer of polish. These effects can dramatically change the mood and feel of your game.

**Bloom** makes bright areas glow, simulating how real cameras respond to intense light:

```cpp
Ref<BloomPass> bloom = new BloomPass();
bloom->setThreshold(1.0f);
bloom->setIntensity(0.5f);
```

**Tone Mapping** converts the high dynamic range (HDR) rendered image to the limited range of your display, preserving detail in both bright and dark areas:

```cpp
renderSettings->setToneMapping(ToneMapping::ACES);
renderSettings->setExposure(1.0f);
```

**Depth of Field** mimics a camera lens, blurring objects outside the focal distance for a cinematic look:

```cpp
Ref<DOFPass> dof = new DOFPass();
dof->setFocusDistance(10.0f);
dof->setAperture(2.8f);
```

**Anti-Aliasing** smooths jagged edges. Traktor supports **TAA** (Temporal Anti-Aliasing, recommended for most cases), **FXAA** (Fast Approximate Anti-Aliasing, cheapest but lower quality), and **MSAA** (Multi-Sample Anti-Aliasing, only in forward rendering).

### Custom Post Effects

The graph-based image processing system lets you create custom post effects. Build a graph of texture samples, filters, and operations, then output to the screen. It's flexible and powerful for unique visual styles.

## Performance Matters

Great graphics mean nothing if your game runs at 10 FPS. Traktor includes several features to keep rendering fast:

**GPU Culling** automatically tests object visibility on the GPU. Only objects visible from the camera are rendered, dramatically improving performance in complex scenes—and it's all automatic.

**Level of Detail (LOD)** systems render simpler versions of meshes when they're far away. Up close, you see all the detail. At a distance, a simplified model that looks identical but renders much faster:

```cpp
mesh->addLOD(highDetailMesh, 0.0f);    // 0-50 units
mesh->addLOD(mediumDetailMesh, 50.0f); // 50-100 units
mesh->addLOD(lowDetailMesh, 100.0f);   // 100+ units
```

**Automatic batching** groups similar objects together, reducing draw calls. Instanced rendering draws many copies of the same object in a single call—perfect for foliage, rocks, or crowds.

Check rendering stats to see what's consuming time:

```cpp
// Get rendering stats
RenderStats stats = renderSystem->getStats();
log::info << "Draw calls: " << stats.drawCalls << Endl;
log::info << "Triangles: " << stats.triangles << Endl;
log::info << "GPU time: " << stats.gpuTime << " ms" << Endl;
```

## Cameras: Your Window to the World

The camera defines what the player sees. Setting it up is straightforward:

```cpp
Ref<CameraComponent> camera = new CameraComponent();
camera->setProjection(
    60.0f,    // FOV
    16.0f/9.0f,  // Aspect ratio
    0.1f,     // Near plane
    1000.0f   // Far plane
);
camera->setPerspective(true);
entity->setComponent(camera);
```

Cameras can also have special effects. **Motion blur** simulates camera movement for a more cinematic feel. **HDR** (High Dynamic Range) captures a wider range of brightness, from deep shadows to blinding highlights:

```cpp
// Motion blur
camera->setMotionBlur(true);

// HDR
camera->setHDR(true);
camera->setExposure(1.0f);
```

## Optimization Tips

Rendering performance can make or break your game's experience. Here are the keys to keeping frame rates high:

**Use LOD systems.** Reduce geometry detail at a distance. Players won't notice simplified models when they're far away, but your GPU will thank you.

**Cull aggressively.** Enable frustum culling (don't render what's outside the camera view) and occlusion culling (don't render what's behind other objects). Both are automatic in Traktor.

**Batch draw calls.** Group similar objects together. Every draw call has overhead, so fewer is better.

**Optimize shaders.** Profile your GPU to find expensive shaders, then simplify or optimize them.

**Use texture compression.** BC7 for color textures, BC5 for normal maps. Compressed textures are smaller in memory and faster to sample.

**Limit overdraw.** Avoid layering lots of transparent objects. Each pixel rendered multiple times wastes GPU power.

**Profile first, optimize second.** Measure what's actually slow before spending time optimizing. Traktor's built-in profiler shows exactly where time is spent.

## Debugging Your Visuals

When something doesn't look right, visual debugging tools help you see what's happening:

In the editor, enable visualization modes: **Wireframe** shows the underlying geometry, **Normals** displays surface orientation, **Texture UVs** reveals how textures map to geometry, **Lightmaps** shows baked lighting, **Shadow Maps** lets you inspect shadow quality, and **G-Buffer** views let you see individual deferred rendering buffers.

**Shader hot-reload** is incredibly useful during development. Edit a shader, save, and it automatically recompiles and reloads—instant feedback.

For deeper analysis, use **RenderDoc integration**. Launch your game with RenderDoc, capture a frame, and inspect every draw call, resource, and shader in detail. It's invaluable for tracking down performance issues and rendering bugs.

## Advanced Features

### Texture Streaming

Loading every texture at full resolution would consume enormous amounts of memory. **Texture streaming** automatically loads textures at appropriate resolutions based on visibility and distance. Close objects get high-res textures, distant objects get lower-res versions:

```cpp
textureSettings->setStreaming(true);
textureSettings->setStreamingPoolSize(512);  // MB
```

### Virtual Textures

For truly massive textures (think entire terrains with unique detail everywhere), **virtual textures** (mega-textures) let you use textures far larger than GPU memory by streaming only visible portions:

```cpp
texture->setVirtual(true);
```

### Compute Shaders

Compute shaders let you use the GPU for arbitrary parallel computation, not just rendering. Perfect for procedural generation, physics simulation, or custom effects:

```glsl
#version 450

layout (local_size_x = 16, local_size_y = 16) in;

layout (binding = 0, rgba8) uniform image2D outputImage;

void main()
{
    ivec2 pos = ivec2(gl_GlobalInvocationID.xy);
    // Compute and write to image
    imageStore(outputImage, pos, vec4(1.0, 0.0, 0.0, 1.0));
}
```

## Best Practices

**Profile before optimizing.** Measure what's actually slow. Guessing leads to wasted effort optimizing things that don't matter.

**Use PBR materials.** Physically-based materials look consistent across all lighting conditions. It's much easier to get good results than with ad-hoc material models.

**Bake static lighting.** If something never moves, bake its lighting. Real-time lighting is expensive—baked lighting is essentially free at runtime.

**Minimize state changes.** Group rendering by material and shader. Every state change (switching shaders, changing textures) has overhead.

**Test on target hardware.** A high-end development machine isn't representative of player hardware. Profile on actual target devices.

**Use Shader Graph when possible.** Visual shader editing is faster to iterate on than writing code, compiling, and restarting.

**Enable ray tracing selectively.** RT is expensive. Use it where it makes a big visual difference, not everywhere indiscriminately.

## See Also

- [World System](world.md) - Mesh and light components
- [Editor](../editor.md) - Shader graph editor
- [Architecture](architecture.md) - Render server

## References

- Source: `code/Render/`
- Vulkan specification: https://www.vulkan.org/
- PBR guide: https://learnopengl.com/PBR/Theory
