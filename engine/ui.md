---
layout: default
permalink: /engine/ui/
title: UI
parent: Engine

nav_order: 13
---

# UiKit - In-Game UI System

UiKit is a Lua-only UI framework for building in-game menus, HUDs, and interactive interfaces. It is based on the Spark module (`code/Spark`), which provides Flash-like rendering capabilities.

![TODO: Screenshot of in-game menu using UiKit with buttons and layouts]

**Note:** The `code/Ui` folder is for editor tooling UI, NOT in-game UI. For in-game interfaces, use UiKit.

## Overview

Features:
- **Lua-Only Framework:** All UI code written in Lua scripts
- **Spark-Based Rendering:** Uses Flash-like movie clips for rendering
- **Widget Hierarchy:** Composable UI elements (Container, Button, Image, etc.)
- **Layout System:** Automatic sizing and positioning (TableLayout, FloodLayout, StackLayout)
- **Event Handling:** Mouse, keyboard, and custom events
- **Flipboard Pattern:** Page-based navigation with transitions
- **Virtual Resolution:** Resolution-independent UI scaling
- **Styles:** Customizable appearance via Style class

Runtime code location: `data/Source/System/UiKit/Scripts`

## Initialization

Initialize UiKit in your game stage by loading resource kits (Spark movies containing UI assets):

```lua
import(traktor)

-- In your stage's create method:
Widget.initialize(
    environment.resource.resourceManager,
    {
        "\{43489F5B-5785-1249-A7BA-CB5259C3F064}",  -- UI resource kit GUIDs
        "\{27DDC0FD-39DE-0946-BB1B-7D84A001C522}",
        "\{5ABEAE05-EB26-934F-932F-7F3C7C582383}"
    }
)

-- Create frame with virtual resolution (1280x720)
self._frame = Frame(
    self.spark.root,
    1280, 720,
    FloodLayout(0, 0)
)
self._frame:update()
```

**Note:** UiKit scripts must be referenced using `#using {GUID}` directives at the top of your scripts.

## Widget Base Class

All UiKit widgets inherit from the `Widget` base class, which provides:

### Alignment Constants

```lua
Widget.ALIGN_LEFT      -- Horizontal left alignment
Widget.ALIGN_CENTER    -- Horizontal/vertical center
Widget.ALIGN_RIGHT     -- Horizontal right alignment
Widget.ALIGN_TOP       -- Vertical top alignment
Widget.ALIGN_BOTTOM    -- Vertical bottom alignment
```

### Common Widget Methods

```lua
widget:setVisible(visible)           -- Show/hide widget
widget:isVisible(recursive)          -- Check visibility
widget:setEnable(enable)             -- Enable/disable interaction
widget:isEnabled()                   -- Check enabled state
widget:setHorizontalAlign(align)     -- Set horizontal alignment
widget:setVerticalAlign(align)       -- Set vertical alignment
widget:setAlpha(alpha)               -- Set opacity (0-1)
widget:setStyle(style, value)        -- Apply style
widget:remove()                      -- Remove widget from hierarchy
widget:removeAllChildren()           -- Remove all child widgets
widget:setFocus()                    -- Set keyboard focus
widget:killFocus()                   -- Remove keyboard focus
widget:setModal()                    -- Make widget modal
widget:releaseModal()                -- Release modal state
widget:addEventListener(eventType, target, fn) -- Add event listener
```

## Container Widget

Container is the base widget for holding child widgets with a layout:

```lua
#using \{7947759C-88DB-794E-8D09-7F30A40B6669}  -- Container
#using \{26FCC8EA-F349-5545-93B5-6ABDAE065E6F}  -- TableLayout

MyView = MyView or class("MyView", Container)

function MyView:new(parent)
    -- Create container with TableLayout
    Container.new(self, parent, TableLayout({100}, {0, 0}, 0, 0, 0, 0))

    -- Add child widgets
    local button = Button(self, "ButtonUp", "ButtonDown", "ButtonHover")
        :setHorizontalAlign(Widget.ALIGN_CENTER)
        :setOnClick(function() print("Clicked!") end)
end
```

## Common Widgets

### Button

Interactive button with up/down/hover states:

```lua
#using \{40191BBE-DDD0-0E47-92A9-66AF2CEC0F6F}  -- Button

local button = Button(parent, "MC_ButtonUp", "MC_ButtonDown", "MC_ButtonHover")
    :setHorizontalAlign(Widget.ALIGN_CENTER)
    :setOnClick(function(btn)
        print("Button clicked!")
    end)
```

### Image

Display a Spark movie clip or bitmap:

```lua
#using \{8B7D68BD-9B9C-454D-AB2A-8FB6A0F79E15}  -- Image

local logo = Image(parent, "MC_Logo")
    :setHorizontalAlign(Widget.ALIGN_CENTER)
    :setScale(0.9, 0.9)
```

### Static

Display static text:

```lua
#using \{65079C2E-2443-F248-BCF1-F48F8A2EE1F5}  -- Static

local label = Static(parent, "Score: 0")
    :setTextSize(48)
    :setTextColor(Color4f(1, 1, 1, 1))
```

### Edit

Text input field:

```lua
#using \{816FB49B-11B4-7A40-8F72-5F76DC858D5F}  -- Edit

local edit = Edit(parent)
    :setOnChange(function(text)
        print("Text changed: " .. text)
    end)
```

### CheckBox

Checkbox widget:

```lua
#using \{7122E0C9-1BEE-634A-8BF7-49087F52A189}  -- CheckBox

local checkbox = CheckBox(parent, "Enable sound")
    :setChecked(true)
    :setOnChange(function(checked)
        print("Checked: " .. tostring(checked))
    end)
```

### Slider

Slider for value adjustment:

```lua
#using \{69E4AD87-C702-DA45-9AB7-22048C1A954F}  -- Slider

local slider = Slider(parent, 0, 100, 50)
    :setOnChange(function(value)
        print("Value: " .. value)
    end)
```

### ProgressBar

Display progress:

```lua
#using \{E571F548-103B-DB47-BD21-0E6D3662681C}  -- ProgressBar

local progressBar = ProgressBar(parent)
    :setProgress(0.5)  -- 0.0 to 1.0
```

## Layout System

Layouts control how child widgets are positioned within containers.

### TableLayout

Grid-based layout with columns and rows:

```lua
#using \{26FCC8EA-F349-5545-93B5-6ABDAE065E6F}  -- TableLayout

-- TableLayout(columns, rows, hmargin, vmargin, hpad, vpad)
-- Column/row values: 0 = fit to content, >0 = percentage of space

-- 2 columns (50% each), 2 rows (fit content)
local layout = TableLayout({50, 50}, {0, 0}, 10, 10, 5, 5)

Container.new(self, parent, layout)
```

### FloodLayout

Fill entire parent area:

```lua
#using \{BA802A78-F9E0-7843-B731-198BEB633236}  -- FloodLayout

-- FloodLayout(hmargin, vmargin)
local layout = FloodLayout(16, 16)  -- 16 pixel margins

Container.new(self, parent, layout)
```

### StackLayout

Stack widgets vertically or horizontally:

```lua
#using \{E1AC8A37-C364-5E47-BB5F-41B22A0CDC5E}  -- StackLayout

-- StackLayout(vertical, spacing)
local layout = StackLayout(true, 10)  -- Vertical with 10px spacing

Container.new(self, parent, layout)
```

## Flipboard - Page Navigation

Flipboard manages page-based navigation with animated transitions:

```lua
#using \{BA802A78-F9E0-7843-B731-198BEB633236}  -- Flipboard

-- Create flipboard
self._flipBoard = Flipboard(self._frame)

-- Create pages as views
local mainView = MainView(self._flipBoard)
local optionsView = OptionsView(self._flipBoard)

-- Show page by flipboardId
self._flipBoard:showPage("MAIN")
```

Views define their page ID using `__flipboardId`:

```lua
MainView = MainView or class("MainView", Container)

MainView.__flipboardId = "MAIN"  -- Page identifier

function MainView:new(parent)
    Container.new(self, parent, FloodLayout())
    -- Add widgets...
end

-- Optional enter/leave callbacks
function MainView:enter(arg)
    print("Page entered with arg: " .. tostring(arg))
end

function MainView:leave()
    print("Page left")
end
```

## Event Handling

UiKit provides various event types for interaction:

```lua
#using \{7F5ADE59-4125-3342-A3F1-AEA3F584834E}  -- Events

-- Mouse events
widget:addEventListener(MousePressEvent, self, function(event)
    print("Mouse pressed")
    return true  -- Event handled
end)

widget:addEventListener(MouseReleaseEvent, self, function(event)
    print("Mouse released, inside: " .. tostring(event.inside))
    return true
end)

widget:addEventListener(MouseEnterEvent, self, function(event)
    print("Mouse entered")
    return true
end)

widget:addEventListener(MouseLeaveEvent, self, function(event)
    print("Mouse left")
    return true
end)

-- Keyboard events
widget:addEventListener(KeyDownEvent, self, function(event)
    if event.code == spark.Key.AkEnter then
        print("Enter key pressed")
    end
    return true
end)

widget:addEventListener(KeyUpEvent, self, function(event)
    print("Key released: " .. event.code)
    return true
end)

-- Custom callbacks (simpler alternative)
button:setOnClick(function(btn)
    print("Button clicked!")
end)
```

## Creating Custom Views

Example from kartong showing a main menu view:

```lua
#using \{FC4400A2-BDB6-BA45-9A22-12B9676E71FA}  -- Container
#using \{FC45EF09-AC3E-7F41-94C7-5437459D9517}  -- Custom widgets

MainView = MainView or class("MainView", Container)

MainView.__flipboardId = "MAIN"

function MainView:new(parent)
    Container.new(self, parent, TableLayout({0}, {0, 0}, 0, 0, 0, 0))

    self._logo = Image(self, "MC_Logo")
        :setHorizontalAlign(Widget.ALIGN_CENTER)
        :setScale(0.9, 0.9)

    Selector(self)
        :setHorizontalAlign(Widget.ALIGN_CENTER)
        :add("PLAY SINGLE")
        :add("PLAY MULTI")
        :add("OPTIONS")
        :add("EXIT")
        :setOnActivate(function(index)
            if index == 0 then self._fnPlay("SINGLE") end
            if index == 1 then self._fnPlay("MULTI") end
            if index == 2 then self._fnOptions() end
            if index == 3 then self._fnExit() end
        end)

    -- Default callbacks
    self._fnPlay = function(mode) end
    self._fnOptions = function() end
    self._fnExit = function() end
end

function MainView:setOnPlay(fn)
    self._fnPlay = fn
    return self
end

function MainView:setOnOptions(fn)
    self._fnOptions = fn
    return self
end

function MainView:setOnExit(fn)
    self._fnExit = fn
    return self
end
```

## HUD Example

Example HUD view with nested containers:

```lua
#using \{FC4400A2-BDB6-BA45-9A22-12B9676E71FA}  -- Container
#using \{6EB38A62-F0A4-3C44-BBEE-7B98717C3536}  -- HudPlayerView
#using \{9A73FE32-33E8-F949-9079-C0157707A292}  -- RacePosition

HudView = HudView or class("HudView", Container)

HudView.__flipboardId = "HUD"

function HudView:new(parent, nkarts)
    Container.new(self, parent, FloodLayout())

    -- Player stats in table layout
    local ct = Container(self, TableLayout({50, 50}, {100}, 0, 0, 0, 0))
    self._playerView0 = HudPlayerView(ct)
    self._playerView1 = HudPlayerView(ct)

    -- Race position widget at bottom center
    local ct = Container(self, FloodLayout(16, 16))
    self._racePosition = RacePosition(ct, nkarts)
        :setHorizontalAlign(Widget.ALIGN_CENTER)
        :setVerticalAlign(Widget.ALIGN_BOTTOM)
end

function HudView:setPosition(index, position, lanePosition)
    self._racePosition:setPosition(index, position, lanePosition)
    self._racePosition:update()
end
```

## Referencing UiKit Scripts

Use `#using {GUID}` directives to reference UiKit scripts and widgets:

```lua
#using \{7947759C-88DB-794E-8D09-7F30A40B6669}  -- Container
#using \{40191BBE-DDD0-0E47-92A9-66AF2CEC0F6F}  -- Button
#using \{8B7D68BD-9B9C-454D-AB2A-8FB6A0F79E15}  -- Image
#using \{26FCC8EA-F349-5545-93B5-6ABDAE065E6F}  -- TableLayout
#using \{BA802A78-F9E0-7843-B731-198BEB633236}  -- FloodLayout
```

The GUIDs correspond to `.xdi` script assets in `data/Source/System/UiKit/Scripts`.

## Best Practices

1. **Use Virtual Resolution:** Create Frame with fixed virtual size (e.g., 1280x720) for consistent UI across resolutions
2. **Leverage Layouts:** Use TableLayout for grids, FloodLayout for full-screen, StackLayout for lists
3. **Method Chaining:** Most widget methods return `self`, enabling fluent API style
4. **Flipboard for Pages:** Organize UI into pages with Flipboard for clean navigation
5. **Event Handling:** Return `true` from event listeners to mark events as handled
6. **Resource Kits:** Load all required Spark movie resources via `Widget.initialize()`
7. **Alignment:** Use `setHorizontalAlign()` and `setVerticalAlign()` instead of manual positioning

## See Also

- [Scripting](scripting.md) - Lua scripting fundamentals
- Source: `data/Source/System/UiKit/Scripts`
- Spark module: `code/Spark`
- Editor module: `code/UiKit/Editor`

## References

- Runtime: `data/Source/System/UiKit/Scripts`
- Sample: `build/kartong` (kartong sample project)
- Spark: `code/Spark` (Flash-like rendering)
