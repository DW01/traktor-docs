---
layout: default
permalink: /engine/audio/
title: Audio
parent: Engine

nav_order: 12
---

# Audio System - Sound that Brings Worlds to Life

Close your eyes and think about your favorite game moment. Chances are, sound played a huge role in making it memorable. The crack of a gunshot, the atmospheric hum of a mysterious location, the triumphant swell of music when you defeat a boss—audio isn't just decoration, it's half the experience. Without it, even the most visually stunning game feels hollow and disconnected.

Traktor's audio system gives you the tools to create rich, immersive soundscapes. Whether you're placing footsteps in 3D space so they echo correctly off walls, streaming hours of music without consuming memory, or processing audio through custom filters for that perfect underwater effect, the audio system has you covered. It's designed to be both powerful and flexible, supporting everything from simple 2D sound effects to complex surround-sound mixes.

Think of the audio system as your game's sound studio. It handles playback on everything from simple stereo to full 7.1 surround, positions sounds in 3D space with realistic distance attenuation and doppler effects, processes audio through graph-based filters, organizes sounds into banks for easy variation, and streams large music files efficiently. All of this happens with minimal latency, so sounds play exactly when they should—no awkward delays that break immersion.

![TODO: Screenshot of audio graph editor showing nodes for sound sources, filters, and output]

## The Foundation: HD Audio Pipeline

Traktor's audio system is built around a **high-definition audio pipeline** that supports a wide range of speaker configurations: simple stereo (2.0), stereo with subwoofer (2.1), surround sound (5.1 and 7.1), and even custom channel configurations for unusual setups. The system automatically adapts to the player's hardware, ensuring your carefully crafted audio works whether they're using headphones, desktop speakers, or a full home theater system.

The audio backend is also flexible. Depending on your target platform, Traktor can use **XAudio2** (modern Windows), **DirectSound** (legacy Windows), **WinMM** (fallback Windows), **OpenAL** (cross-platform), **OpenSL** (mobile), **ALSA** (Linux), or **PulseAudio** (Linux desktop). You don't need to worry about these details during development—Traktor abstracts the platform differences and provides a consistent API.

## Playing Sounds

For game audio—footsteps, gunshots, dialogue, UI sounds—you use the **Sound system directly**. This gives you full control over playback:

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

The sound player returns a handle that lets you control the sound after it starts—adjust volume, change pitch, or stop it early. This is essential for dynamic audio that responds to gameplay.

## 3D Spatial Audio: Placing Sound in Space

One of the most powerful features is **3D spatial audio**. Instead of sounds playing at a fixed volume in both ears, they're positioned in 3D space relative to the listener. A gunshot to your left sounds louder in your left ear. As you walk past a waterfall, the sound shifts from one ear to the other. Move away from a sound source, and it naturally fades.

Setting up 3D audio is straightforward:

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

**Attenuation** controls how sound volume decreases with distance. The **minDistance** is where the sound is at full volume—get closer and it doesn't get louder. The **maxDistance** is where the sound becomes completely inaudible. Between these two distances, the volume smoothly fades based on realistic distance falloff.

The **velocity** parameter enables the **doppler effect**—that pitch shift you hear when a fast-moving object passes by. Set a sound's velocity, and as it approaches the listener, the pitch rises; as it moves away, the pitch falls. Perfect for racing games, flyby effects, or any fast-moving sound source.

## Sound Banks: Organizing Your Audio

Games often need variations of the same sound—different footstep sounds for grass, wood, and metal; multiple gunshot variations to avoid repetition; various grunt sounds for character hits. **Sound banks** organize these variations:

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

From your scripts, playing a sound from a bank is simple:

```lua
-- Play from sound bank
local soundBank = context.soundBank
local sound = soundBank:play("Footsteps/Grass")
```

Sound banks can also randomly select from multiple variations, apply volume adjustments, and handle other organizational tasks. They're a clean way to manage large audio libraries.

## Audio Filters: Processing Sound in Real-Time

Sometimes you need to process audio—add reverb when entering a cave, muffle sounds when underwater, or apply distortion to a broken radio. Traktor's **graph-based audio filters** let you build custom processing chains:

**Reverb** simulates room acoustics. A small, tiled bathroom sounds very different from a vast cathedral. Reverb adds those reflections and echoes that make spaces feel real.

**Echo** creates delay effects—classic echoes, slapback delays, or rhythmic repeats.

**EQ (Equalization)** adjusts frequency balance. Cut the bass for tinny radio sounds, boost the lows for rumbling explosions, or sculpt sounds to fit the mix.

**Compression** controls dynamic range, making quiet sounds louder and loud sounds quieter. This ensures dialogue is always audible over background noise.

**Distortion** adds grit and aggression—overdrive effects, bit crushing, or lo-fi degradation.

These filters are combined in a visual graph editor, making it easy to experiment and iterate on audio processing.

## Music Streaming: Big Audio Without Big Memory

A single music track can be tens of megabytes. Load several tracks into memory at once, and you've consumed a significant chunk of your memory budget. **Streaming** solves this by loading and playing music directly from disk:

```lua
-- Stream background music
local music = SoundComponent()
music:setSound(musicResource)
music:setStreaming(true)  -- Stream from disk
music:setLoop(true)
music:setVolume(0.7)
music:play()
```

Streaming music uses only a small buffer in memory—usually a few seconds of audio—while the rest stays on disk. The system reads ahead smoothly, so there's no stuttering or interruption. You can have hours of music without memory concerns.

Traktor supports **MP3**, **FLAC**, and **OGG** formats for streaming. MP3 is widely compatible and reasonably compressed. FLAC is lossless for pristine quality. OGG offers excellent compression with good quality.

## The Audio Listener: The Player's Ears

For 3D audio to work, the system needs to know where the "listener" is—typically the player's camera or character. The listener's position and orientation determine how 3D sounds are positioned:

```lua
-- Set listener position (usually in camera update)
local listenerPos = camera.transform.translation
local listenerDir = camera.transform.axisZ
local listenerUp = camera.transform.axisY

context.audio:setListenerPosition(listenerPos)
context.audio:setListenerOrientation(listenerDir, listenerUp)
```

Update the listener every frame to match the camera, and 3D audio automatically follows the player's movement and view direction. Sounds in front sound like they're in front, sounds behind sound like they're behind, and everything shifts naturally as the player turns.

The listener's velocity can also be set for doppler effects when the listener moves at high speed—perfect for racing games or flight simulators.

## Best Practices

**Use 3D audio for diegetic sounds** (sounds that exist in the game world). Footsteps, gunshots, ambient loops, NPC dialogue—these should all be positioned in 3D space. It dramatically improves immersion.

**Reserve 2D audio for UI and music.** Menu sounds, notifications, and background music don't need spatial positioning. Play them in 2D at a fixed volume.

**Manage sound instances.** Playing hundreds of sounds simultaneously can overload the audio system. Limit concurrent sounds, prioritize important sounds, and cull distant or quiet sounds.

**Use streaming for music, not for short effects.** Streaming has overhead. Sound effects should be loaded into memory for instant playback without disk access.

**Organize with sound banks.** Don't manage dozens of individual sound files in code. Use sound banks to group related sounds and handle variations.

**Test with different speaker configurations.** Your game should sound good on stereo headphones and surround speakers. Test both to ensure proper balance and positioning.

**Avoid audio clipping.** Keep your master volume under 1.0 to prevent distortion when multiple sounds play at once. Use compression or ducking (automatically lowering background music when dialogue plays) to manage dynamic range.

## Common Patterns

### Footstep System

```lua
import(traktor)

FootstepController = FootstepController or class("FootstepController", world.ScriptComponent)

function FootstepController:new()
    self._stepTimer = 0
    self._soundBank = context.soundBank
end

function FootstepController:update(contextObject, totalTime, deltaTime)
    local velocity = self:getVelocity()
    local speed = velocity:length()

    if speed > 0.1 then
        self._stepTimer = self._stepTimer + deltaTime
        local stepInterval = 0.5 / (speed / 5.0)  -- Faster steps at higher speed

        if self._stepTimer >= stepInterval then
            self._stepTimer = 0

            -- Determine surface type and play appropriate sound
            local surface = self:getSurfaceType()
            local sound = self._soundBank:play("Footsteps/" .. surface)
            sound:setPosition(self.owner.transform.translation)
        end
    else
        self._stepTimer = 0
    end
end

function FootstepController:getSurfaceType()
    -- Raycast down to detect surface material
    -- Return "Grass", "Wood", "Metal", etc.
    return "Grass"
end
```

### Ambient Audio Zones

Place invisible trigger volumes that change ambient audio when the player enters:

```lua
AmbientZone = AmbientZone or class("AmbientZone", world.ScriptComponent)

function AmbientZone:new()
    self._ambientSound = nil
end

function AmbientZone:onTriggerEnter(event)
    if event.other:getName() == "Player" then
        -- Fade in ambient sound
        self._ambientSound = SoundComponent()
        self._ambientSound:setSound(self._ambientResource)
        self._ambientSound:setLoop(true)
        self._ambientSound:setStreaming(true)
        self._ambientSound:play()
        self._ambientSound:fadeIn(2.0)  -- 2 second fade
    end
end

function AmbientZone:onTriggerExit(event)
    if event.other:getName() == "Player" then
        -- Fade out and stop
        if self._ambientSound then
            self._ambientSound:fadeOut(2.0)
        end
    end
end
```

## Debugging Audio

When audio doesn't work as expected, check these common issues:

**No sound plays:** Verify the sound resource loaded correctly, check that volume isn't zero, ensure the audio system initialized successfully.

**3D sound doesn't pan correctly:** Make sure you're updating the listener position every frame, verify the sound is set to 3D mode, check min/max distance values.

**Performance issues:** Too many concurrent sounds can overload the mixer. Limit active sounds and use distance-based culling.

**Streaming music stutters:** Disk I/O might be saturated. Reduce other disk activity or use a larger streaming buffer.

Use logging to track sound playback:

```lua
log:info("Playing sound at position: " .. tostring(position))
log:info("Active sounds: " .. context.audio:getActiveSoundCount())
```

## See Also

- [World System](world.md) - Audio components
- [Scripting](scripting.md) - Controlling audio from Lua

## References

- Source: `code/Sound/`
