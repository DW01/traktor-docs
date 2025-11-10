---
layout: default
permalink: /editor/asset-reference/
title: Asset Type Reference
parent: Editor
nav_order: 17
---

# Asset Type Reference

This page documents asset types available in Traktor, their properties, and how they're used. Assets are the bridge between external content (3D models, textures, sounds, videos) and the runtime engine.

**Note:** All properties shown here are verified from the current source code. Some assets have additional properties not listed. Use the editor to explore all available options for each asset type.

Most assets that reference external files inherit from the base `Asset` class, which provides:
- **fileName** (Path) - Relative path to the source file

Paths are relative to the **Asset path** configured in Editor Settings (Edit → Settings → General tab).

---

## Animation Assets

### traktor.animation.AnimationAsset

References animation data for skeletal animation.

**Properties:**

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `fileName` | Path | Path to source animation file (BVH, FBX, etc.) | - |
| `targetSkeleton` | Guid | Target skeleton for retargeting. If empty, uses source skeleton. | empty |
| `take` | String | Take/clip name to import from source file | "" |
| `scale` | Float | Scale factor applied to animation | 1.0 |
| `translate` | Vector4 | Translation offset applied to animation | (0,0,0) |
| `removeLocomotion` | Boolean | Remove root motion/locomotion from animation | true |

**Use case:** Character animations, skeletal motion, retargeted animation.

**Creating:** Right-click in Database → New instance → Select `traktor.animation.AnimationAsset`

---

### traktor.animation.SkeletonAsset

References skeletal rig data.

**Properties:**

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `fileName` | Path | Path to source file containing skeleton data | - |
| `offset` | Vector4 | Position offset for the skeleton | (0,0,0) |
| `scale` | Float | Scale factor for the skeleton | 1.0 |
| `radius` | Float | Radius for bone visualization | 0.25 |
| `invertX` | Boolean | Invert X-axis coordinates | false |
| `invertZ` | Boolean | Invert Z-axis coordinates | false |

**Use case:** Character rigs, skeletal hierarchies for animation.

**Creating:** Right-click in Database → New instance → Select `traktor.animation.SkeletonAsset`

---

### traktor.animation.ClothAsset

References cloth simulation data.

**Use case:** Cloth physics for clothing, capes, flags, banners.

**Creating:** Right-click in Database → New instance → Select `traktor.animation.ClothAsset`

---

## Mesh Assets

### traktor.mesh.MeshAsset

References 3D models for rendering.

**Properties:**

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `fileName` | Path | Path to source model (Collada, FBX, LWO, OBJ) | - |
| `importFilter` | String | Filter for importing specific parts of model | "" |
| `meshType` | Enum | Runtime mesh type (see below) | MtStatic |
| `materialShaders` | Map | Mapping from material names to shader Guids | empty |
| `materialTextures` | Map | Mapping from material names to texture Guids | empty |
| `scaleFactor` | Float | Scale factor applied to mesh | 1.0 |
| `offset` | Vector4 | Position offset applied to mesh | (0,0,0) |
| `renormalize` | Boolean | Recalculate normals during import | false |
| `center` | Boolean | Center mesh around origin | false |
| `grounded` | Boolean | Place mesh on ground plane (Y=0) | false |
| `decalResponse` | Boolean | Allow decals to project onto this mesh | true |
| `reduce` | Float | Geometry reduction factor (1.0 = no reduction) | 1.0 |
| `previewAngle` | Float | Rotation angle for preview in database browser | 0.0 |

**Mesh Types:**

| Type | Value | Description | Use Case |
|------|-------|-------------|----------|
| `MtInstance` | 0 | Hardware instanced mesh | Repeated objects: grass, rocks, debris |
| `MtSkinned` | 1 | Skeleton-deformed mesh | Animated characters, creatures |
| `MtStatic` | 2 | Static, non-instanced mesh | Unique props, buildings, environment |

**Note:** Earlier versions supported additional mesh types (MtBlend, MtPartition, MtIndoor, MtStream) which are no longer in the current codebase.

**Use case:** All visual geometry. Characters, props, buildings, vehicles.

**Creating:** Right-click in Database → **Create mesh asset...** → Select source model

---

## Physics Assets

### traktor.physics.MeshAsset

References collision meshes for physics simulation.

**Properties:**

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `fileName` | Path | Path to source geometry file | - |
| `importFilter` | String | Filter for importing specific parts | "" |
| `calculateConvexHull` | Boolean | Calculate convex hull. Required for dynamic objects. | true |
| `margin` | Float | Collision margin around mesh | 0.04 |
| `scaleFactor` | Float | Scale factor applied to collision mesh | 1.0 |
| `reduce` | Float | Geometry reduction factor | 1.0 |
| `center` | Boolean | Center mesh around origin | false |
| `grounded` | Boolean | Place on ground plane | false |
| `materials` | Map | Mapping from mesh materials to physics materials | empty |

**Use case:** Collision geometry for physics bodies.

**Creating:** Right-click in Database → New instance → Select `traktor.physics.MeshAsset`

---

## Render Assets

### traktor.render.TextureAsset

References image data for textures.

**Properties:** (via TextureOutput)

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `fileName` | Path | Path to source image (TGA, PNG, JPEG, GIF) | - |
| `textureFormat` | Enum | Target texture format | TfInvalid |
| `normalMap` | Boolean | Convert height to normal map | false |
| `scaleDepth` | Float | Depth scale for normal map conversion | 0.0 |
| `generateMips` | Boolean | Generate mipmap chain | true |
| `keepZeroAlpha` | Boolean | Preserve zero alpha in mipmaps | false |
| `textureType` | Enum | 2D, 3D, or Cube | Tt2D |
| `hasAlpha` | Boolean | Force assume alpha channel exists | false |
| `generateAlpha` | Boolean | Generate alpha channel | false |
| `ignoreAlpha` | Boolean | Ignore source alpha channel | false |
| `invertAlpha` | Boolean | Invert alpha values | false |
| `premultiplyAlpha` | Boolean | Premultiply alpha | false |
| `dilateImage` | Boolean | Dilate image edges | false |
| `scaleImage` | Boolean | Resize image | false |
| `scaleWidth` | Integer | Target width (if scaleImage enabled) | 0 |
| `scaleHeight` | Integer | Target height (if scaleImage enabled) | 0 |
| `flipX` | Boolean | Flip image horizontally | false |
| `flipY` | Boolean | Flip image vertically | false |
| `enableCompression` | Boolean | Enable texture compression | true |
| `encodeAsRGBM` | Boolean | Encode as RGBM format | false |
| `inverseNormalMapX` | Boolean | Invert normal map X component | false |
| `inverseNormalMapY` | Boolean | Invert normal map Y component | false |
| `scaleNormalMap` | Float | Scale normal map intensity | 0.0 |
| `assumeLinearGamma` | Boolean | Source is in linear gamma space | false |
| `generateSphereMap` | Boolean | Generate sphere map from cube | false |
| `preserveAlphaCoverage` | Boolean | Preserve alpha coverage in mipmaps | false |
| `alphaCoverageReference` | Float | Reference value for alpha coverage | 0.5 |
| `sharpenRadius` | Integer | Sharpening filter radius | 0 |
| `sharpenStrength` | Float | Sharpening strength | 0.0 |
| `noiseStrength` | Float | Add noise strength | 0.0 |
| `systemTexture` | Boolean | Mark as system texture | false |

**Use case:** All 2D images. Diffuse maps, normal maps, UI textures, etc.

**Creating:** Right-click in Database → **Import texture batch...** → Add textures

---

### traktor.render.ImageGraphAsset

Procedural texture generation using node graphs.

**Use case:** Procedural textures, runtime-generated images.

**Creating:** Right-click in Database → New instance → Select `traktor.render.ImageGraphAsset`

---

### traktor.render.EnvironmentTextureAsset

Environment textures for image-based lighting.

**Use case:** Environment maps, reflection probes, skyboxes.

**Creating:** Right-click in Database → New instance → Select `traktor.render.EnvironmentTextureAsset`

---

## Scene Assets

### traktor.scene.SceneAsset

Complete 3D scene with entities, lighting, and environment.

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `worldRenderSettings` | Object | Rendering configuration for the scene |
| `worldComponents` | Array | World-level components (navigation, environment, etc.) |
| `layers` | Array | Entity hierarchy organized into layers |
| `operationData` | Array | Editor operation history/data |

**Note:** SceneAsset doesn't inherit from Asset. It's edited through the Scene Editor, not as a simple property list.

**Use case:** Game levels, test scenes, cutscene environments.

**Creating:** Right-click in Database → New instance → Select `traktor.scene.SceneAsset`

---

## Sound Assets

### traktor.sound.SoundAsset

References individual sound files.

**Properties:**

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `fileName` | Path | Path to sound file (WAV, FLAC, MP3, OGG) | - |
| `category` | Guid | Sound category for mixing | empty |
| `stream` | Boolean | Stream from disk vs. load into memory | false |
| `preload` | Boolean | Preload sound data | false |
| `compressed` | Boolean | Use compressed format at runtime | true |
| `gain` | Float | Volume adjustment (dB) | 0.0 |

**Use case:** All audio. Music, sound effects, voice, ambient.

**Creating:** Right-click in Database → New instance → Select `traktor.sound.SoundAsset`

---

### traktor.sound.BankAsset

Sound bank composed of grains for dynamic audio.

Edited through the [Sound Bank Editor](sound-editor/).

**Use case:** Complex, randomized audio. Footsteps, impacts, UI sounds.

**Creating:** Right-click in Database → New instance → Select `traktor.sound.BankAsset`

---

### traktor.sound.GraphAsset

Audio processing graphs for effects and filters.

**Use case:** Audio DSP, reverb, filters, real-time effects.

**Creating:** Right-click in Database → New instance → Select `traktor.sound.GraphAsset`

---

## Effects Assets

### traktor.spray.PointSetAsset

Point set data for particle emission.

**Use case:** Point-based particle emission, procedural placement.

**Creating:** Right-click in Database → New instance → Select `traktor.spray.PointSetAsset`

---

## Terrain Assets

### traktor.terrain.TerrainAsset

Large-scale terrain rendering.

**Properties:**

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `detailSkip` | Integer | LOD detail skip factor | 1 |
| `patchDim` | Integer | Patch dimension (must be power-of-2 + 1) | 129 |
| `heightfield` | Resource ID | Reference to heightfield resource | empty |
| `surfaceShader` | Resource ID | Shader for terrain surface | empty |

**Use case:** Large outdoor environments, landscapes.

**Creating:** Right-click in Database → New instance → Select `traktor.terrain.TerrainAsset`

---

## Heightfield Assets

### traktor.heightfield.HeightfieldAsset

Heightfield data stored as database instance data channels.

**Properties:**

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `worldExtent` | Vector4 | Size of heightfield in world space (XZ plane) | (0,0,0) |
| `erosionEnable` | Boolean | Enable erosion simulation | false |
| `erodeIterations` | Integer | Number of erosion iterations | 100000 |

**Note:** Actual heightfield data is stored as database instance data channels, not as an external file reference.

**Use case:** Terrain heightfields, displacement maps.

**Creating:** Right-click in Database → New instance → Select `traktor.heightfield.HeightfieldAsset`

---

## Video Assets

### traktor.video.VideoAsset

References video files for playback.

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `fileName` | Path | Path to video file (MP4, WebM, OGV, etc.) |

**Use case:** Cutscenes, intro videos, in-game monitors.

**Creating:** Right-click in Database → New instance → Select `traktor.video.VideoAsset`

---

## UI Assets (Spark)

### traktor.spark.MovieAsset

References SWF (Flash) files for vector-based UI.

**Properties:**

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `fileName` | Path | Path to SWF file | - |
| `staticMovie` | Boolean | Movie contains no dynamic content | false |
| `fonts` | Array | Font files to embed | empty |

**Use case:** Vector UI, Flash animations, scalable interfaces.

**Creating:** Right-click in Database → New instance → Select `traktor.spark.MovieAsset`

---

## AI Assets

### traktor.ai.NavMeshAsset

Navigation mesh data for AI pathfinding.

**Use case:** AI navigation, pathfinding for NPCs.

**Creating:** Right-click in Database → New instance → Select `traktor.ai.NavMeshAsset`

---

## Localization Assets

### traktor.i18n.DictionaryAsset

String tables for internationalization.

Edited through the [Localization Editor](localization-editor/).

**Use case:** Multi-language text, UI strings, dialog.

**Creating:** Right-click in Database → New instance → Select `traktor.i18n.DictionaryAsset`

---

## World Assets

### traktor.world.IrradianceGridAsset

Irradiance grid data for global illumination.

**Use case:** Baked global illumination, light probes.

**Creating:** Right-click in Database → New instance → Select `traktor.world.IrradianceGridAsset`

---

## Input Assets

### traktor.input.InputMappingAsset

Input configuration for controller/keyboard bindings.

**Use case:** Remappable controls, input profiles.

**Creating:** Right-click in Database → New instance → Select `traktor.input.InputMappingAsset`

---

## See Also

- [Database](database/) - Managing assets
- [Pipeline](pipeline/) - How assets are built
- [Workflows](workflows/) - Asset import workflows
- [Settings](settings/) - Configuring the asset path
