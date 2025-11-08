---
layout: default
permalink: /manual/engine/particles/
title: Particles
parent: Engine
grand_parent: Manual
nav_order: 11
---

# Particle System

The particle system (Spray) provides GPU-accelerated particle effects for fire, smoke, explosions, and other visual effects.

![TODO: Screenshot showing various particle effects (fire, smoke, explosion, sparks) with particle editor visible]

## Overview

Features:
- **GPU Acceleration:** Thousands of particles
- **Emitter Types:** Point, box, sphere, mesh surface
- **Forces:** Gravity, wind, vortex, turbulence
- **Collision:** Particle-world collision
- **Trails:** Motion trails
- **Sorting:** Proper alpha blending

## Particle Component

```cpp
// C++ - Create particle effect
Ref<ParticleComponent> particles = new ParticleComponent();
particles->setEffect(effectResource);
entity->setComponent(particles);
```

```lua
-- Lua - Control particles
local particles = self.owner:getComponent(spray.ParticleComponent)

-- Emit particles
particles:emit(10)  -- Emit 10 particles

-- Control
particles:play()
particles:stop()
particles:setRate(100)  -- Particles per second
```

## Particle Effects

Create effects in the editor:
1. Create Particle Effect asset
2. Configure emitter properties
3. Add forces and modifiers
4. Assign material/texture
5. Preview and adjust

## Common Effects

### Explosion
- High emission rate, short duration
- Expanding sphere emitter
- Radial velocity
- Size over lifetime
- Alpha fade out

### Fire
- Continuous emission
- Upward velocity
- Turbulence force
- Color gradient (red → orange → yellow)
- Additive blending

### Smoke
- Medium emission rate
- Upward drift
- Size increases over lifetime
- Alpha fades slowly
- Soft particles

## See Also

- [Render System](render.md) - Particle rendering

## References

- Source: `code/Spray/`
