---
layout: default
permalink: /manual/engine/input/
title: Input
parent: Engine
grand_parent: Manual
nav_order: 6
---

# Input System

The Input system provides unified input handling across different devices and platforms including keyboard, mouse, gamepads, and touch screens.

## Overview

Traktor's input system supports:
- **Keyboard:** Key presses and text input
- **Mouse:** Position, buttons, and scroll wheel
- **Gamepad:** Multiple controllers with buttons and analog sticks
- **Touch:** Touch screen input (mobile)
- **Input Mapping:** Remap controls without code changes

## Accessing Input

### From Lua Scripts

```lua
import(traktor)

InputExample = InputExample or class("InputExample", world.ScriptComponent)

function InputExample:update(contextObject, totalTime, deltaTime)
    local input = contextObject.input

    -- Keyboard
    if input:isKeyDown("W") then
        -- Move forward
    end

    -- Mouse
    local mousePos = input:getMousePosition()
    if input:isMouseButtonDown(0) then  -- Left button
        -- Fire weapon
    end

    -- Gamepad
    if input:isButtonDown(0, "A") then  -- Controller 0, A button
        -- Jump
    end
end
```

### From C++

```cpp
IInputServer* inputServer = environment->getInputServer();
InputState state = inputServer->getState();

// Check keyboard
if (state.isKeyDown(KeyCode::W))
{
    // Move forward
}

// Check mouse
Vector2 mousePos = state.getMousePosition();
if (state.isMouseButtonDown(0))
{
    // Fire weapon
}
```

## Keyboard Input

```lua
-- Check if key is currently pressed
if input:isKeyDown("W") then end
if input:isKeyDown("Escape") then end

-- Check if key was just pressed this frame
if input:wasKeyPressed("Space") then
    -- Jump (only once per press)
end

-- Check if key was just released
if input:wasKeyReleased("Ctrl") then end

-- Special keys
input:isKeyDown("Shift")
input:isKeyDown("Ctrl")
input:isKeyDown("Alt")
input:isKeyDown("Enter")
input:isKeyDown("Escape")
```

## Mouse Input

```lua
-- Mouse position (screen coordinates)
local pos = input:getMousePosition()  -- Vector2
local x = pos.x
local y = pos.y

-- Mouse movement delta
local delta = input:getMouseDelta()

-- Mouse buttons (0 = left, 1 = right, 2 = middle)
if input:isMouseButtonDown(0) then end       -- Currently pressed
if input:wasMouseButtonPressed(1) then end   -- Just pressed
if input:wasMouseButtonReleased(2) then end  -- Just released

-- Mouse wheel
local scroll = input:getMouseWheel()  -- Positive = up, Negative = down
```

## Gamepad Input

```lua
-- Check gamepad connection
if input:isGamepadConnected(0) then  -- Controller 0

    -- Buttons (Xbox layout)
    if input:isButtonDown(0, "A") then end
    if input:isButtonDown(0, "B") then end
    if input:isButtonDown(0, "X") then end
    if input:isButtonDown(0, "Y") then end

    -- Triggers
    if input:isButtonDown(0, "LeftTrigger") then end
    if input:isButtonDown(0, "RightTrigger") then end

    -- Bumpers
    if input:isButtonDown(0, "LeftBumper") then end
    if input:isButtonDown(0, "RightBumper") then end

    -- D-Pad
    if input:isButtonDown(0, "DPadUp") then end
    if input:isButtonDown(0, "DPadDown") then end

    -- Analog sticks (-1.0 to 1.0)
    local leftX = input:getAxisValue(0, "LeftStickX")
    local leftY = input:getAxisValue(0, "LeftStickY")
    local rightX = input:getAxisValue(0, "RightStickX")
    local rightY = input:getAxisValue(0, "RightStickY")

    -- Triggers as analog (0.0 to 1.0)
    local leftTrigger = input:getAxisValue(0, "LeftTrigger")
    local rightTrigger = input:getAxisValue(0, "RightTrigger")

    -- Vibration
    input:setVibration(0, 0.5, 0.5)  -- Controller 0, left motor, right motor
end
```

## Input Mapping

Configure input bindings in editor (`.xdi` files):

```xml
<!-- InputMapping.xdi -->
<InputMapping>
    <Action name="Forward" keys="W,Up" gamepadButton="LeftStickUp"/>
    <Action name="Back" keys="S,Down" gamepadButton="LeftStickDown"/>
    <Action name="Jump" keys="Space" gamepadButton="A"/>
    <Action name="Fire" mouse="0" gamepadButton="RightTrigger"/>
</InputMapping>
```

```lua
-- Use mapped actions
if input:isActionActive("Forward") then
    -- Move forward (works with W, Up arrow, or left stick)
end

if input:wasActionTriggered("Jump") then
    -- Jump (works with Space or A button)
end
```

## Touch Input (Mobile)

```lua
-- Get touch count
local touchCount = input:getTouchCount()

for i = 0, touchCount - 1 do
    local touch = input:getTouch(i)

    -- Touch ID (persistent across frames)
    local id = touch.id

    -- Touch position
    local pos = touch.position  -- Vector2

    -- Touch state
    if touch.state == "Began" then
        -- Touch started
    elseif touch.state == "Moved" then
        -- Touch moved
    elseif touch.state == "Ended" then
        -- Touch ended
    end
end
```

## Common Patterns

### Character Movement

```lua
import(traktor)

CharacterMovement = CharacterMovement or class("CharacterMovement", world.ScriptComponent)

function CharacterMovement:new()
    self._speed = 5.0
end

function CharacterMovement:update(contextObject, totalTime, deltaTime)
    local input = contextObject.input
    local moveDir = Vector4(0, 0, 0)

    -- Keyboard
    if input:isKeyDown("W") then moveDir.z = moveDir.z - 1 end
    if input:isKeyDown("S") then moveDir.z = moveDir.z + 1 end
    if input:isKeyDown("A") then moveDir.x = moveDir.x - 1 end
    if input:isKeyDown("D") then moveDir.x = moveDir.x + 1 end

    -- Gamepad (overrides keyboard)
    if input:isGamepadConnected(0) then
        local x = input:getAxisValue(0, "LeftStickX")
        local y = input:getAxisValue(0, "LeftStickY")
        if math.abs(x) > 0.1 or math.abs(y) > 0.1 then
            moveDir = Vector4(x, 0, -y)
        end
    end

    -- Normalize and apply
    if moveDir:length() > 0 then
        moveDir = moveDir:normalized()
        local character = self.owner:getComponent(physics.CharacterComponent)
        character:move(moveDir * self._speed, false)
    end
end
```

### Camera Look

```lua
import(traktor)

CameraLook = CameraLook or class("CameraLook", world.ScriptComponent)

function CameraLook:new()
    self._yaw = 0
    self._pitch = 0
    self._sensitivity = 0.1
end

function CameraLook:update(contextObject, totalTime, deltaTime)
    local input = contextObject.input

    -- Mouse look
    local mouseDelta = input:getMouseDelta()

    self._yaw = self._yaw + mouseDelta.x * self._sensitivity
    self._pitch = self._pitch - mouseDelta.y * self._sensitivity
    self._pitch = math.max(-89, math.min(89, self._pitch))  -- Clamp

    -- Apply rotation
    local rotation = Quaternion.fromEulerAngles(
        math.rad(self._pitch),
        math.rad(self._yaw),
        0
    )
    -- ... apply to camera
end
```

## Best Practices

1. **Use Input Mapping:** Don't hardcode keys, use actions
2. **Support Multiple Devices:** Test with keyboard, mouse, and gamepad
3. **Provide Sensitivity Options:** Let players adjust mouse/stick sensitivity
4. **Handle Disconnects:** Check gamepad connection before use
5. **Dead Zones:** Apply dead zones to analog sticks to avoid drift

## See Also

- [Scripting](scripting.md) - Using input from Lua
- [Runtime System](runtime.md) - Input server setup

## References

- Source: `code/Input/`
