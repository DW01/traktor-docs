---
layout: default
permalink: /editor/particle-editor/
title: Particle Effect Editor
parent: Editor
nav_order: 12
---

# Particle Effect Editor

![TODO: Screenshot of the Particle Effect Editor showing the layer panel with multiple effect layers and the viewport displaying a particle system in action]

The **Particle Effect Editor** is a specialized tool for designing layered particle effects. Visual effects like smoke, fire, sparks, magic spells, and environmental ambiance.

Particle effects in Traktor are built from **layers**. Each layer represents a distinct visual element (e.g., one layer for the core flame glow, another for rising smoke, another for sparks). By combining multiple layers with different emitters, textures, and behaviors, you create rich, complex effects from simple building blocks.

## Opening the Particle Editor

1. Create a new particle effect asset in the Database: Right-click → **New instance** → Select `traktor.spray.EffectData`
2. Double-click the effect asset to open the Particle Effect Editor

## Editor Layout

The Particle Effect Editor provides:

**Layer panel** - Lists all layers in the effect. Each layer is evaluated independently and rendered together to form the complete visual effect.

**Viewport** - Real-time preview of the particle effect with playback controls. See the effect play out as you adjust parameters.

**Properties panel** - Configure the selected layer's properties: emitter type, particle count, lifetime, velocity, textures, colors, and blend modes.

## Creating Particle Effects

**Add a layer:**
1. Right-click in the layer panel
2. Select **Add layer**
3. Configure the layer properties:
   - **Emitter type** - Point, line, sphere, mesh surface
   - **Emission rate** - Particles per second
   - **Particle lifetime** - How long particles live
   - **Velocity** - Initial particle speed and direction
   - **Texture** - Visual appearance of particles
   - **Color over time** - Gradient controlling particle color as it ages
   - **Size over time** - How particle size changes during its lifetime
   - **Blend mode** - Additive, alpha blend, etc.

**Layer order matters:** Layers are rendered in the order they appear in the panel. Use this to control visual compositing (e.g., glow layers behind smoke layers).

**Preview and iterate:** The viewport updates in real-time. Scrub the timeline, adjust parameters, and immediately see results. This tight feedback loop makes effect design fast and intuitive.

## Tips

- Start with a single layer and get it looking right before adding complexity
- Use additive blending for glowing effects (fire, magic, energy)
- Use alpha blending for smoke and clouds
- Combine multiple layers with different textures and timings for rich, varied effects
- Reference the texture from the Database by dragging it into the texture property field

## See Also

- [Effects](../engine/effects/) - The Spray module and effects system architecture
- [Database](database/) - Managing effect assets
- [Scene Editor](scene-editor/) - Placing effects in scenes
