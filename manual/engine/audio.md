---
layout: default
permalink: /manual/engine/audio/
title: Audio
parent: Engine
grand_parent: Manual
nav_order: 12
---

# Audio System

The Traktor audio system provides high-definition audio playback with support for multiple speaker configurations, 3D spatial audio, and graph-based effects processing.

![TODO: Screenshot of audio graph editor showing nodes for sound sources, filters, and output]

## Overview

Traktor's audio system features:
- **HD Audio Pipeline:** Support for 2.0, 2.1, 5.1, 7.1, and custom channel configurations
- **Multiple Backends:** XAudio2, DirectSound, WinMM, OpenAL, OpenSL, ALSA, Pulse
- **Graph-based Filters:** User-definable audio processing
- **Sound Banks:** Easy sound effect customization
- **Streaming:** MP3, FLAC, OGG support

## Audio in Traktor

Traktor's audio system is primarily accessed through the `Sound` module's low-level API.

### Sound Component (Particle Effects)

The `SoundComponent` in the Spray module is specifically for particle effect audio:

```cpp
// C++ - Sound component for particle effects
Ref<SoundComponent> sound = new SoundComponent(soundPlayer, soundResource);
entity->setComponent(sound);

// Basic playback
sound->play();
sound->stop();
sound->setVolume(0.5f);
sound->setPitch(1.0f);
```

**Note:** This component is in `code/Spray/SoundComponent.h` and is designed for particle system audio.

### Direct Audio Playback

For general audio playback, use the Sound system directly:

```cpp
// Access sound player
Ref<SoundPlayer> soundPlayer = audioSystem->getSoundPlayer();

// Play sound
Ref<SoundHandle> handle = soundPlayer->play(soundResource, position, volume);

// Control playback
handle->setVolume(0.5f);
handle->setPitch(1.2f);
handle->stop();
```

## 3D Spatial Audio

Position audio in 3D space:

```lua
-- Enable 3D audio
sound:set3D(true)

-- Set position (automatically uses entity position)
local pos = self.owner.transform.translation
sound:setPosition(pos)

-- Set velocity (for doppler effect)
sound:setVelocity(velocity)

-- Attenuation settings
sound:setMinDistance(5.0)    -- Distance where volume starts decreasing
sound:setMaxDistance(50.0)   -- Distance where sound is inaudible
```

## Sound Banks

Organize and customize sound variations:

```xml
<!-- SoundBank.xdi -->
<SoundBank>
    <Category name="Footsteps">
        <Sound name="Grass" path="Audio/Footstep_Grass.wav" volume="0.8"/>
        <Sound name="Wood" path="Audio/Footstep_Wood.wav" volume="0.9"/>
        <Sound name="Metal" path="Audio/Footstep_Metal.wav" volume="1.0"/>
    </Category>
</SoundBank>
```

```lua
-- Play from sound bank
local soundBank = context.soundBank
local sound = soundBank:play("Footsteps/Grass")
```

## Audio Filters

Apply real-time audio effects using graph-based filters:

- **Reverb:** Simulate room acoustics
- **Echo:** Delay effects
- **EQ:** Equalization
- **Compression:** Dynamic range control
- **Distortion:** Overdrive/distortion effects

## Music Streaming

Stream music files to save memory:

```lua
-- Stream background music
local music = SoundComponent()
music:setSound(musicResource)
music:setStreaming(true)  -- Stream from disk
music:setLoop(true)
music:setVolume(0.7)
music:play()
```

## Audio Listener

Set up audio listener (usually camera or player):

```lua
-- Set listener position (usually in camera update)
local listenerPos = camera.transform.translation
local listenerDir = camera.transform.axisZ
local listenerUp = camera.transform.axisY

context.audio:setListenerPosition(listenerPos)
context.audio:setListenerOrientation(listenerDir, listenerUp)
```

## See Also

- [World System](world.md) - Audio components
- [Scripting](scripting.md) - Controlling audio from Lua

## References

- Source: `code/Sound/`
