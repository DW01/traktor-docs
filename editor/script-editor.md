---
layout: default
permalink: /editor/script-editor/
title: Script Editor
parent: Editor
nav_order: 14
---

# Script Editor

![TODO: Screenshot of the Script Editor showing a Lua script file open with syntax highlighting, line numbers, and the script references panel on the left listing included dependencies]

The **Script Editor** is a specialized tool for editing gameplay scripts written in Lua or JavaScript. It provides syntax highlighting, dependency management, and integration with the Database for managing script assets.

Scripts are the primary way to define gameplay behavior in Traktor. Components written in script can respond to events, manipulate entities, handle input, trigger effects, and orchestrate game logic. The Script Editor makes this development workflow smooth and integrated with the rest of the editor.

## Opening the Script Editor

1. Create a new script asset in the Database: Right-click → **New instance** → Select `traktor.script.ScriptAsset`
2. Double-click the script asset to open the Script Editor
3. Alternatively, for existing scripts: Double-click any `.lua` or `.js` script reference in the Database

## Editor Features

**Syntax highlighting** - Color-coded syntax for both Lua and JavaScript. Keywords, strings, comments, and functions are visually distinguished, making code more readable and reducing errors.

**Line numbers** - Each line is numbered for easy reference and debugging. Error messages from the runtime reference line numbers, making it simple to locate issues.

**Script references panel (left)** - Lists all dependencies. Other scripts that this script includes or references. These referenced scripts are automatically included in the "global scope" of the current script when it runs.

## Script Dependencies

Scripts can reference other scripts to share code, define libraries, or organize functionality. This is handled through the **script references panel**.

**Adding a dependency:**
1. In the Script Editor, locate the script references panel on the left
2. Click the **Add** button (or right-click and select **Add reference**)
3. Browse the Database for the script you want to include
4. Click OK

The referenced script is now automatically loaded before your script executes. All functions, variables, and classes defined in the referenced script are available in your script's global scope.

**Use case example:** Create a `Utilities.lua` script with common helper functions. Reference it from all your gameplay scripts. Now every script can call those utility functions without duplicating code.

## Workflow Tips

**Organize scripts in folders:** Use the Database to create groups (folders) for organizing scripts by purpose. For example: `Scripts/Player/`, `Scripts/Enemies/`, `Scripts/UI/`.

**Use references for shared code:** Don't copy-paste common functions across scripts. Extract them to a shared script and reference it.

**Test in the Scene Editor:** Scripts attached to entity components can be tested directly in the Scene Editor using preview mode. Make changes in the Script Editor, save, and the hot-reload system updates the running scene.

**Check the Output tab for errors:** Syntax errors and runtime errors appear in the Output tab with line numbers. Double-click an error to jump to the Script Editor at that line (if supported by the editor).

**Comment your code:** Use `--` for single-line comments in Lua or `//` in JavaScript. Document complex logic, parameters, and intentions. Future you (and other developers) will thank you.

## Example Script Structure

A typical gameplay script in Traktor Lua:

```lua
-- PlayerController.lua
-- Handles player movement and input

import(traktor.script)
import(traktor.input)
import(traktor.world)

PlayerController = class("PlayerController", Component)

function PlayerController:create()
    -- Called when component is created
    self.speed = 5.0
end

function PlayerController:update(context, deltaTime)
    -- Called every frame
    local inputState = context:getInputState()

    if inputState:isKeyDown(input.KeyCodes.W) then
        self:move(Vector4(0, 0, 1) * self.speed * deltaTime)
    end
end

function PlayerController:move(offset)
    local transform = self.owner.transform
    transform:setTranslation(transform:getTranslation() + offset)
end

return PlayerController
```

## See Also

- [Scripting](../engine/scripting/) - Complete scripting guide with API reference
- [Database](database/) - Managing script assets
- [Scene Editor](scene-editor/) - Attaching scripts to entities
- [Output](output/) - Viewing script errors and logs
