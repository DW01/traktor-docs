---
layout: default
permalink: /editor/logging/
title: Logging
parent: Editor
nav_order: 9
---

# Logging - Real-Time Debugging

The **LogView** is your window into what's happening in your game. Every log message from the engine, your scripts, or the editor itself appears here in real-time. Debugging becomes easier when you can see exactly what's executing, what's failing, and what's being loaded.

Logs are categorized by severity: **Info** for general messages, **Warning** for potential issues, **Error** for problems that need fixing, and **Debug** for detailed diagnostic information. The LogView color-codes messages by level, making errors instantly visible in a sea of info messages.

You can **filter by level** to see only errors, **search** for specific messages, **export logs** for bug reports, and **view logs from connected targets**—when your game runs on a device, its logs stream back to the editor in real-time. This remote logging makes debugging on physical devices practical.

## Writing Log Messages

From your code, logging is simple:

```cpp
// C++
log::info << "Initialization complete" << Endl;
log::error << "Failed to load asset: " << assetName << Endl;
```

```lua
-- Lua
log:info("Player spawned at position: " .. tostring(position))
log:error("Invalid input detected")
```

Logs aren't just for debugging—they're for understanding. Add logs to track game state, asset loading, and performance. When something goes wrong, well-placed logs make the difference between hours of guesswork and minutes of diagnosis.

## See Also

- [Scripting](../engine/scripting/) - Logging from Lua scripts
- [Output](output/) - Pipeline build logs
- [Targets](targets/) - Logs from connected deployment targets
