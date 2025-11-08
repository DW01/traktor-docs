---
layout: default
permalink: /editor/targets/
title: Targets
parent: Editor
nav_order: 8
---

# Targets - Managing Deployment

The **Targets** tab manages connected deployment targets—the devices and platforms where you can build and run your game. This is your command center for multi-platform development. Each target represents a destination where your game can run: a local Windows build, a connected Android device, a remote Linux server, or an iOS simulator.

In the Targets tab, you can:

**Connect to targets** - Add and configure deployment destinations. Connect a physical Android device over USB, set up a remote Linux machine over the network, or configure a local build target for your development platform.

**Build for targets** - Trigger asset builds specific to each target platform. The pipeline processes your assets using platform-specific settings, compressing textures appropriately, optimizing shaders, and packaging data for the target's architecture.

**Deploy and run** - With one click, package your game, deploy it to the connected target, and launch it with a live debugger connection. The editor handles all the complexity—copying files, starting the runtime, establishing communication.

**Monitor connections** - See which targets are currently connected, their status (building, running, idle), and any connection issues. If a target disconnects, the Targets tab shows the problem.

Once deployed and running, the editor maintains a **live connection** to your game on the target. Edit an asset, save it, and watch it hot-reload on the running device—no restart required. View real-time logs streaming back from the target. This active connection makes cross-platform development practical and efficient, whether you're testing on a phone, a console, or a remote server.

## See Also

- [Pipeline](pipeline/) - Platform-specific asset optimization
- [Logging](logging/) - Viewing logs from connected targets
