---
layout: default
permalink: /engine/input/
title: Input
parent: Engine

nav_order: 6
---

# Input System - The Bridge Between Player and Game

Input is the conversation between player and game. Every jump, every turn, every action starts with the player pressing a button or moving a stick. Without responsive, reliable input handling, even the most beautiful game feels sluggish and frustrating. Players notice input lag immediately. It breaks the illusion that they're actually controlling the character.

Traktor's input system provides a unified way to handle input from any device: keyboard and mouse for PC players, gamepads for console-style play, and touch screens for mobile devices. More importantly, it handles all the details that make input feel good. Dead zones on analog sticks so slight drift doesn't move your character, delta tracking for smooth mouse look, and frame-by-frame state so you can distinguish between "held down" and "just pressed."

Think of the input system as the translator that turns physical button presses into meaningful game events. Whether a player presses W on the keyboard, pushes up on an analog stick, or taps an on-screen button, you can map all of those to a single "move forward" action. This means players can use whatever control scheme feels natural to them, and you don't have to write separate code for each input device.

## Accessing Input in Your Scripts

From Lua scripts, input is accessed through the `contextObject` passed to your update function. This gives you a snapshot of input state for the current frame:

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

From C++, you access the input server directly:

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

## Keyboard Input: Pressed, Held, or Released?

Keyboard input has three states that matter: **currently pressed** (held down), **just pressed** (this frame only), and **just released** (this frame only). Understanding the difference is critical for responsive controls.

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

**isKeyDown** is for continuous actions like movement. While W is held, keep moving forward. **wasKeyPressed** is for discrete actions like jumping. You want to jump once per press, not every frame while the button is held. **wasKeyReleased** is for actions that happen when you let go, like releasing a charged shot.

## Mouse Input: Position, Movement, and Clicks

Mouse input combines position tracking, button states, and scroll wheel input. For camera control, you typically use **mouse delta** (how much the mouse moved this frame) rather than absolute position:

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

**Mouse delta** is perfect for first-person camera controls. It tells you how many pixels the mouse moved since last frame, which you can use to rotate the camera smoothly. Using absolute position would feel jumpy and unnatural.

The **mouse wheel** value resets each frame, so you read it as "scrolled up this frame" or "scrolled down this frame," perfect for weapon switching or zooming.

## Gamepad Input: Buttons and Analog Sticks

Gamepads provide buttons, analog sticks, and triggers. The system follows Xbox controller layout by default, which maps well to most modern controllers:

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

Always check if a gamepad is connected before reading its input. Players can connect and disconnect controllers at any time, and reading from a disconnected controller gives you zeroes.

**Analog sticks** return values from -1.0 to 1.0 on each axis. You should apply a **dead zone** (ignoring small values near zero) to prevent stick drift from causing unwanted movement. A typical dead zone is around 0.1 to 0.2.

**Vibration** (rumble) provides force feedback. Use it sparingly. Subtle rumble on impacts and explosions adds immersion, but constant rumble is annoying and drains battery.

## Input Mapping: Flexibility for Players

Hardcoding keys in your game scripts is a mistake. Different players prefer different control schemes—WASD vs arrow keys, different button layouts, remapped controls for accessibility. **Input mapping** lets you define logical actions ("Jump", "Fire", "Forward") and bind them to physical inputs in configuration files:

```xml
<!-- InputMapping.xdi -->
<InputMapping>
    <Action name="Forward" keys="W,Up" gamepadButton="LeftStickUp"/>
    <Action name="Back" keys="S,Down" gamepadButton="LeftStickDown"/>
    <Action name="Jump" keys="Space" gamepadButton="A"/>
    <Action name="Fire" mouse="0" gamepadButton="RightTrigger"/>
</InputMapping>
```

In your scripts, check for actions instead of specific keys:

```lua
-- Use mapped actions
if input:isActionActive("Forward") then
    -- Move forward (works with W, Up arrow, or left stick)
end

if input:wasActionTriggered("Jump") then
    -- Jump (works with Space or A button)
end
```

Now players can remap controls without you changing any code. You can even provide multiple default mappings for different play styles, and let players customize them in an options menu.

## Touch Input for Mobile

On touch-enabled devices, input comes from touches on the screen. Each touch has a unique ID that persists across frames, so you can track individual fingers:

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

Touch input is great for mobile games, but you typically combine it with virtual on-screen buttons or joysticks for intuitive controls.

## Common Patterns

### Character Movement

A typical character movement system combines keyboard and gamepad input, with gamepad overriding keyboard when available:

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
        if math.abs(x) > 0.1 or math.abs(y) > 0.1 then  -- Dead zone
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

Notice the **dead zone check** (0.1) on the analog stick. Without it, even slight stick drift causes unwanted movement.

### Camera Look with Mouse

For first-person camera controls, use mouse delta and track yaw/pitch:

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

**Clamping pitch** prevents the camera from flipping upside down when looking straight up or down. Most games clamp pitch to around -89 to +89 degrees.

## Best Practices

**Use input mapping, not hardcoded keys.** Define actions and map them to inputs. This gives players flexibility and makes your code cleaner.

**Support multiple devices.** Test with keyboard/mouse and gamepad. Many PC players prefer controllers for certain genres. Make sure both work well.

**Provide sensitivity options.** Different players have different preferences. Let them adjust mouse sensitivity and stick sensitivity in options.

**Handle gamepad disconnects gracefully.** Always check if a gamepad is connected before reading input. Display a "reconnect controller" message if a player's controller disconnects mid-game.

**Apply dead zones to analog sticks.** Most controllers have some stick drift. A dead zone around 0.1 to 0.2 prevents this from causing unwanted input.

**Use "just pressed" for discrete actions.** Jumping, shooting, opening menus. These should trigger once per button press, not every frame while held.

**Normalize diagonal movement.** When a player presses W+D simultaneously, the movement vector's length is √2. Normalize it so diagonal movement isn't faster than cardinal movement.

**Respond immediately.** Input should feel instant. Avoid multi-frame delays between pressing a button and seeing the result.

## Debugging Input

When input doesn't work as expected, log the input state:

```lua
-- Debug input
log:info("Key W down: " .. tostring(input:isKeyDown("W")))
log:info("Mouse position: " .. tostring(input:getMousePosition()))
log:info("Gamepad 0 connected: " .. tostring(input:isGamepadConnected(0)))
log:info("Left stick X: " .. input:getAxisValue(0, "LeftStickX"))
```

Common issues:

**Input doesn't register:** Make sure you're checking input in the update loop, not just once. Verify the correct key names and button names.

**Gamepad input is jittery:** Apply a dead zone to analog sticks. Even brand-new controllers have slight drift.

**Mouse look is too sensitive or too slow:** Provide a sensitivity slider. Different players have vastly different preferences.

**Jump triggers multiple times:** Use `wasKeyPressed` instead of `isKeyDown` for discrete actions.

## See Also

- [Scripting](scripting.md) - Using input from Lua
- [Runtime System](runtime.md) - Input server setup

## References

- Source: `code/Input/`
