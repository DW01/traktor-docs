---
layout: default
permalink: /editor/sound-editor/
title: Sound Bank Editor
parent: Editor
nav_order: 13
---

# Sound Bank Editor

![TODO: Screenshot of the Sound Bank Editor showing the grain tree view on the left with nested grains (Play, Random, Sequence) and the properties panel on the right displaying grain-specific settings]

The **Sound Bank Editor** is a specialized tool for creating complex, dynamic sound systems. Rather than simply playing individual sound files, Traktor's sound banks use a compositional approach: you build sound behavior from **grains**, which are modular building blocks that can be layered, randomized, sequenced, and nested.

This system gives you powerful control over audio variation and behavior without requiring code. A single "footstep" sound bank, for example, can randomize between multiple footstep samples, add slight pitch variation to each playback, and ensure the same sample doesn't repeat consecutively. All defined visually in the editor.

## What are Sound Banks?

A **sound bank** is a collection of sounds, groups, and behaviors combined into what the game perceives as a single playable sound. When your game script calls `playSound("Footstep")`, it's actually triggering an entire tree of grains that determine which sound plays, how it plays, and with what variations.

Sound banks are composed of **grains**. Each grain represents a sound operation: playing a file, randomizing between options, sequencing sounds in order, or introducing silence.

## Opening the Sound Bank Editor

1. Create a new sound bank asset in the Database: Right-click → **New instance** → Select `traktor.sound.BankAsset`
2. Double-click the bank asset to open the Sound Bank Editor

## Editor Layout

The Sound Bank Editor is split into two main areas:

**Grain tree view (left)** - Hierarchical view of all grains in the bank. Parent grains can contain child grains, allowing complex nesting (e.g., a Random grain that picks between multiple Sequence grains).

**Properties panel (right)** - Displays properties for the selected grain. Each grain type has specific properties that control its behavior.

**Toolbar** - Includes a **Play** button to preview the selected grain. Essential for testing randomization and sequencing without running the game.

## Available Grain Types

The sound engine supports the following grain types:

### Mute

Introduces silence for a specified duration.

**Properties:**
- **Duration** - Number of milliseconds of silence

**Use case:** Adding pauses in sequences, simulating delays in machinery sounds, or creating rhythmic gaps in music.

### Play

Plays a sound. Either a sound asset or another sound bank (yes, banks can reference other banks for modular composition).

**Properties:**
- **Sound** - Reference to a `SoundAsset` or another `BankAsset`
- **Gain** - Volume adjustment (positive or negative dB)
- **Pitch** - Pitch shift (useful for creating variation)

**Use case:** The fundamental grain for actually playing audio. Adjust gain and pitch to create subtle variations that make repeated sounds feel less robotic.

### Random

Randomly selects one child grain to play. This is the workhorse for audio variation.

**Properties:**
- **Humanize** - Prevents the same grain from playing multiple times consecutively. If enabled, the random selector remembers the last choice and avoids repeating it. This makes randomization feel more natural.

**Use case:** Footsteps, gunshots, UI clicks, impacts. Anything that repeats frequently benefits from randomization. Add 3-5 Play grains as children with slightly different samples, and the sound bank will pick randomly between them.

### Repeat

Repeats a child grain a specified number of times.

**Properties:**
- **Count** - Number of repetitions

**Use case:** Machine gun bursts, engine loops, alarm sounds. Combine with Random children to create varied repeated patterns.

### Sequence

Plays child grains in sequential order, one after another.

**Properties:** None (behavior determined by child order)

**Use case:** Complex sound events composed of multiple phases. An explosion might be a sequence of: impact → rumble → debris settling. Or a character voice line might be: breath in → speech → breath out.

## Building a Sound Bank

**1. Create the root grain:**

Right-click in the grain tree view and select **Add grain**. This is typically a Random or Sequence grain depending on your use case.

**2. Add child grains:**

Select the parent grain, right-click, and add children. For a randomized footstep bank:
- Root: Random (with humanize enabled)
  - Child 1: Play (footstep1.wav)
  - Child 2: Play (footstep2.wav, slight pitch variation)
  - Child 3: Play (footstep3.wav, slight gain variation)

**3. Nest grains for complexity:**

Grains can be nested arbitrarily. For a gun reload sound:
- Root: Sequence
  - Child 1: Play (mag_release.wav)
  - Child 2: Mute (200ms)
  - Child 3: Random (humanize enabled)
    - Grandchild 1: Play (mag_insert1.wav)
    - Grandchild 2: Play (mag_insert2.wav)
  - Child 4: Play (chamber_rack.wav)

**4. Preview:**

Select any grain and press **Play** in the toolbar. The selected grain (and all its children) will play. This lets you test individual branches or the entire bank.

## Tips

- **Use humanize on Random grains:** It dramatically improves perceived quality by avoiding robotic repetition
- **Slight pitch/gain variation:** Even small adjustments (±5% pitch, ±1-2 dB gain) make repeated sounds feel alive
- **Modular banks:** Create small, reusable banks and reference them from larger banks. For example, a "MetalImpact" bank used by multiple weapon sound banks
- **Sequence for narrative sounds:** Dialog, cutscene audio, or any sound with a clear beginning/middle/end structure
- **Test frequently:** Use the Play button often. Audio design is iterative.

## See Also

- [Audio](../engine/audio/) - The audio system architecture
- [Database](database/) - Managing sound assets and banks
- [Workflows](workflows/) - Importing sound assets
