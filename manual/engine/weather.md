---
layout: default
permalink: /manual/engine/weather/
title: Weather
parent: Engine
grand_parent: Manual
nav_order: 16
---

# Weather System

The weather system provides dynamic environmental effects including rain, snow, clouds, and atmospheric conditions.

![TODO: Screenshot showing weather effects like rain, fog, and dynamic sky with weather editor panel]

## Overview

Features:
- **Dynamic Sky:** Time-of-day and atmospheric scattering
- **Precipitation:** Rain, snow
- **Clouds:** Volumetric and billboard clouds
- **Fog:** Distance and height fog
- **Lightning:** Storm effects

## Weather Component

```cpp
// C++ - Create weather
Ref<WeatherComponent> weather = new WeatherComponent();
weather->setPreset(weatherPreset);
entity->setComponent(weather);
```

```lua
import(traktor)

WeatherController = WeatherController or class("WeatherController", world.ScriptComponent)

function WeatherController:new()
    self._weather = self.owner:getComponent(weather.WeatherComponent)

    -- Change weather
    self._weather:setRainIntensity(0.8)    -- 0.0 to 1.0
    self._weather:setFogDensity(0.5)
    self._weather:setCloudCoverage(0.7)

    -- Time of day
    self._weather:setTimeOfDay(14.5)  -- 2:30 PM (0-24 hours)

    -- Lightning
    self._weather:triggerLightning()
end
```

## Weather Presets

Create presets in editor:
- **Clear Day:** Blue sky, no precipitation
- **Rainy:** Overcast, rain, fog
- **Stormy:** Dark clouds, heavy rain, lightning
- **Foggy:** Dense fog, limited visibility
- **Snowy:** Snow particles, cold atmosphere

## Dynamic Sky

Realistic sky simulation:
- **Sun Position:** Based on time of day
- **Atmospheric Scattering:** Rayleigh and Mie scattering
- **Stars:** Night sky
- **Moon:** Moon phases

## Precipitation

### Rain

```lua
-- Configure rain
weather:setRainIntensity(1.0)       -- Heavy rain
weather:setRainDirection(Vector4(0.2, -1, 0))  -- Wind effect
```

### Snow

```lua
-- Configure snow
weather:setSnowIntensity(0.5)
weather:setSnowParticleSize(0.01)
```

## Fog

```lua
-- Distance fog
weather:setFogDistance(50.0)       -- Fog starts at 50 units
weather:setFogDensity(0.3)

-- Height fog
weather:setHeightFogEnabled(true)
weather:setHeightFogDensity(0.5)
weather:setHeightFogAltitude(10.0)
```

## Clouds

```lua
-- Cloud configuration
weather:setCloudCoverage(0.6)      -- 60% cloud coverage
weather:setCloudSpeed(0.1)          -- Cloud movement speed
weather:setCloudAltitude(1000.0)
```

## Weather Transitions

Smoothly blend between weather states:

```lua
-- Transition from clear to rainy over 30 seconds
weather:transitionTo("Rainy", 30.0)

-- Instant change
weather:setPreset("Stormy")
```

## See Also

- [World System](world.md) - Weather components
- [Render System](render.md) - Atmospheric rendering

## References

- Source: `code/Weather/`
