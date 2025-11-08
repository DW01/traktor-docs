---
layout: default
permalink: /manual/engine/rendering/
title: Rendering
parent: Engine
grand_parent: Manual
nav_order: 8
---

# Render System

The Traktor render system is a modern, high-performance Vulkan-based renderer with advanced features including ray tracing, frame graphs, and a powerful shader graph editor.

![TODO: Screenshot of a rendered scene showing PBR materials, dynamic lighting, shadows, and reflections]

**Note:** This documentation describes features based on the engine's capabilities. Specific API calls shown are illustrative - refer to actual source code in `code/Render/` for exact interfaces.

## Overview

Key features:
- **Vulkan API:** Modern low-level graphics API
- **Frame Graph:** Declarative rendering pipeline
- **Shader Graph:** Node-based visual shader editor
- **Ray Tracing:** RTGI, RTAO, RT reflections, RT shadows
- **ReSTIR GI:** Advanced global illumination
- **PBR:** Physically-based rendering
- **Deferred & Forward+:** Flexible rendering paths
- **GPU Skin Cache:** Optimized character rendering
- **Occlusion Culling:** GPU-based visibility

## Architecture

### Render Graph

The frame graph system allows you to declaratively define your rendering pipeline:

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

**Benefits:**
- Automatic barrier insertion
- Resource aliasing
- Optimal scheduling
- Clear data flow

### Rendering Paths

#### Deferred Rendering

```
1. Geometry Pass → G-Buffer (albedo, normal, metallic/roughness)
2. Lighting Pass → Accumulate lights
3. Post Processing → Final image
```

**Advantages:**
- Many lights
- Consistent material quality
- Efficient lighting

#### Forward+ Rendering

```
1. Depth Pre-Pass → Z-buffer
2. Light Culling → Per-tile light lists
3. Forward Pass → Shading with visible lights
```

**Advantages:**
- Transparency support
- MSAA antialiasing
- Lower memory usage

## Materials and Shaders

### Shader Graph

![TODO: Screenshot of shader graph editor showing nodes for textures, math operations, PBR output]

Create shaders visually:

1. Open Shader Graph editor
2. Add input nodes (textures, vertex data)
3. Add processing nodes (math, color ops)
4. Connect to output nodes
5. Compile and preview

**Node Types:**
- **Input:** Texture samples, vertex attributes, uniforms
- **Math:** Add, multiply, dot, cross, etc.
- **Color:** RGB/HSV conversion, color grading
- **Utility:** If/else, switch, functions
- **Output:** Fragment color, normal, metallic/roughness

### PBR Materials

Physically-based rendering using:
- **Albedo:** Base color
- **Metallic:** Metal vs dielectric
- **Roughness:** Surface smoothness
- **Normal:** Surface detail
- **AO:** Ambient occlusion
- **Emissive:** Self-illumination

```cpp
// Material properties
material->setTexture("Albedo", albedoTexture);
material->setTexture("Normal", normalTexture);
material->setTexture("Metallic", metallicTexture);
material->setTexture("Roughness", roughnessTexture);
```

### Custom Shaders

Inline GLSL/HLSL code when needed:

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

## Lighting

### Light Types

**Directional Light (Sun):**
```cpp
Ref<LightComponent> sun = new LightComponent();
sun->setType(LightType::Directional);
sun->setColor(Color4f(1.0f, 0.95f, 0.8f));
sun->setIntensity(5.0f);
sun->setCastShadow(true);
```

**Point Light:**
```cpp
Ref<LightComponent> light = new LightComponent();
light->setType(LightType::Point);
light->setRadius(10.0f);
light->setIntensity(10.0f);
```

**Spot Light:**
```cpp
Ref<LightComponent> spotlight = new LightComponent();
spotlight->setType(LightType::Spot);
spotlight->setInnerAngle(30.0f);
spotlight->setOuterAngle(45.0f);
spotlight->setRange(20.0f);
```

### Shadows

**Shadow Maps:**
- Cascaded shadow maps (CSM) for directional lights
- Cube shadow maps for point lights
- Perspective shadow maps for spot lights

**Ray-Traced Shadows:**
```cpp
light->setRayTracedShadows(true);  // Use RT shadows
```

### Global Illumination

**ReSTIR GI:**
- Real-time global illumination
- Reservoir-based spatiotemporal importance resampling
- Dynamic lighting

**Ray-Traced GI:**
```cpp
renderSettings->setRTGI(true);
renderSettings->setRTGIBounces(2);  // Number of light bounces
```

## Ray Tracing

### Hardware Ray Tracing

Enable RT features:

```cpp
// Ray-traced ambient occlusion
renderSettings->setRTAO(true);
renderSettings->setRTAORadius(1.0f);

// Ray-traced reflections
renderSettings->setRTReflections(true);

// Ray-traced shadows
renderSettings->setRTShadows(true);
```

**Requirements:**
- Vulkan 1.3
- RTX or equivalent GPU
- Ray tracing enabled in project settings

### Acceleration Structures

Scene geometry is automatically built into BVH (Bounding Volume Hierarchy) for ray tracing.

## Post-Processing

### Common Effects

**Bloom:**
```cpp
Ref<BloomPass> bloom = new BloomPass();
bloom->setThreshold(1.0f);
bloom->setIntensity(0.5f);
```

**Tone Mapping:**
```cpp
renderSettings->setToneMapping(ToneMapping::ACES);
renderSettings->setExposure(1.0f);
```

**Depth of Field:**
```cpp
Ref<DOFPass> dof = new DOFPass();
dof->setFocusDistance(10.0f);
dof->setAperture(2.8f);
```

**Anti-Aliasing:**
- **TAA:** Temporal anti-aliasing (recommended)
- **FXAA:** Fast approximate anti-aliasing
- **MSAA:** Multi-sample anti-aliasing (forward only)

### Image Processing

Create custom post effects using the graph-based image processing system:

1. Create Image Process Graph
2. Add texture samples
3. Apply filters and operations
4. Output to screen

## Performance

### GPU Culling

Automatic GPU-based occlusion culling:
- Objects are tested for visibility on GPU
- Only visible objects are rendered
- Significant performance gain in complex scenes

### LOD (Level of Detail)

Mesh LOD system:
```cpp
mesh->addLOD(highDetailMesh, 0.0f);    // 0-50 units
mesh->addLOD(mediumDetailMesh, 50.0f); // 50-100 units
mesh->addLOD(lowDetailMesh, 100.0f);   // 100+ units
```

### Batching

Automatic batching of similar objects:
- Instanced rendering for identical objects
- Reduces draw calls
- Improves performance

### Profiling

```cpp
// Get rendering stats
RenderStats stats = renderSystem->getStats();
log::info << "Draw calls: " << stats.drawCalls << Endl;
log::info << "Triangles: " << stats.triangles << Endl;
log::info << "GPU time: " << stats.gpuTime << " ms" << Endl;
```

## Camera

### Camera Setup

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

### Camera Effects

```cpp
// Motion blur
camera->setMotionBlur(true);

// HDR
camera->setHDR(true);
camera->setExposure(1.0f);
```

## Optimization Tips

1. **Use LOD:** Reduce geometry at distance
2. **Cull Aggressively:** Frustum and occlusion culling
3. **Batch:** Group similar objects
4. **Optimize Shaders:** Profile GPU performance
5. **Use Texture Compression:** BC7 for color, BC5 for normals
6. **Limit Overdraw:** Avoid overlapping transparent objects
7. **Profile:** Use built-in profiler to identify bottlenecks

## Debugging

### Visual Debugging

In editor, enable visualization:
- **Wireframe:** See geometry
- **Normals:** Visualize normals
- **Texture UVs:** Check UV mapping
- **Lightmaps:** View baked lighting
- **Shadow Maps:** Inspect shadow quality
- **G-Buffer:** View deferred buffers

### Shader Debugging

```cpp
// Shader hot-reload
// Edit shader in editor → automatically recompiles and reloads
```

### Render Doc Integration

Use RenderDoc for advanced debugging:
1. Launch with RenderDoc
2. Capture frame
3. Inspect draw calls, resources, shaders
4. Analyze performance

## Advanced Features

### Texture Streaming

Automatic texture streaming based on visibility and distance:
```cpp
textureSettings->setStreaming(true);
textureSettings->setStreamingPoolSize(512);  // MB
```

### Virtual Textures

Support for mega-textures (very large textures):
```cpp
texture->setVirtual(true);
```

### Compute Shaders

Use compute shaders for custom GPU work:
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

1. **Profile First:** Measure before optimizing
2. **Use PBR:** Physically-based materials look consistent
3. **Bake When Possible:** Static lighting should be baked
4. **Minimize State Changes:** Group by material/shader
5. **Test on Target Hardware:** Profile on actual target devices
6. **Use Shader Graph:** Easier iteration than code
7. **Enable Ray Tracing Selectively:** RT is expensive, use wisely

## See Also

- [World System](world.md) - Mesh and light components
- [Editor](../editor.md) - Shader graph editor
- [Architecture](architecture.md) - Render server

## References

- Source: `code/Render/`
- Vulkan specification: https://www.vulkan.org/
- PBR guide: https://learnopengl.com/PBR/Theory
