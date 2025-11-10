---
layout: default
permalink: /editor/video-editor/
title: Video Editor
parent: Editor
nav_order: 16
---

# Video Editor

![TODO: Screenshot of the Video Editor showing a video file being previewed with playback controls (play, pause, stop, seek bar) and video properties panel]

The **Video Editor** is a preview tool for video assets. It displays video files through Traktor's video player, allowing you to verify that video content imports correctly and plays as expected before integrating it into your game.

Video assets are typically used for cutscenes, intro sequences, background animations, or in-game screens (monitors, billboards, etc.). The Video Editor doesn't provide editing capabilities (trimming, effects, encoding). Instead, it's a **verification and preview tool** for video files that will be used at runtime.

## Opening the Video Editor

1. Create a new video asset in the Database: Right-click → **New instance** → Select `traktor.video.VideoAsset`
2. In the video asset properties, set the **file path** to your source video file
3. Double-click the video asset to open the Video Editor

## Editor Features

**Video viewport** - Displays the video content at its native resolution or scaled to fit the editor window.

**Playback controls** - Standard controls for video preview:
- **Play** - Start playback from current position
- **Pause** - Pause playback
- **Stop** - Stop playback and return to beginning
- **Seek bar** - Scrub through the video timeline

**Fit options** - Control how the video is displayed:
- **Fit to window** - Scale the video to fill the editor viewport
- **Native size** - Display at the video's original resolution

## Supported Formats

Traktor's video system supports common video formats. Check the `VideoAsset` documentation for the complete list of supported codecs and containers. Common formats include:
- MP4 (H.264)
- WebM (VP8/VP9)
- OGV (Theora)

The video player uses platform-appropriate decoders at runtime for optimal performance.

## Using Video Assets in Games

Once verified in the Video Editor, video assets can be:
- **Played in UI elements** - Video backgrounds, animated logos, tutorial screens
- **Displayed on 3D surfaces** - In-game monitors, billboards, projected textures
- **Triggered by events** - Cutscenes activated by gameplay events

The video playback API in scripts provides control over playback: start, stop, pause, seek, loop, and volume.

## Tips

**Optimize video files before import:** Use external video editing tools to encode videos at appropriate resolutions and bitrates for your target platforms. Mobile platforms may require lower resolutions than desktop.

**Test on target hardware:** Video decoding performance varies significantly between platforms. A 4K video that plays smoothly on desktop may stutter on mobile. Preview in the Video Editor, but verify on actual devices.

**Use appropriate codecs:** H.264 is widely supported and hardware-accelerated on most platforms. Consider platform-specific codec support when choosing formats.

**Keep file sizes reasonable:** Video files are often the largest assets in a game. Balance quality with file size, especially for downloadable games.

## See Also

- [Database](database/) - Managing video assets
- [UI](../engine/ui/) - Displaying video in user interfaces
- [Runtime](../engine/runtime/) - Video playback API
