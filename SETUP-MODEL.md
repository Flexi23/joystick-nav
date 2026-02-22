# T.Flight HOTAS One – Setup Model Reference

This document describes the physical layout and 3D model structure of the Thrustmaster T.Flight HOTAS One as implemented in this project. It is intended as a machine-readable reference for AI agents working on the codebase.

## Coordinate System

Three.js right-handed coordinate system:

- **+X** = right (toward joystick unit)
- **−X** = left (toward throttle unit)
- **+Y** = up
- **+Z** = toward the user / camera (front)
- **−Z** = away from the user (back)

The model is centered at world origin.

## Unit Convention

All geometry dimensions in the code are specified in **centimeters** and converted to meters via:

```js
const S = 0.01; // cm → m
```

This document lists raw cm values as they appear in the code. Multiply by `S` to get meter-space values used by Three.js.

## Model Constants

| Constant | Value (cm) | Description |
|---|---|---|
| `S` | 0.01 | cm → m conversion factor |
| `DOME_RADIUS` | 15 | Sphere radius for dome caps |
| `DOME_THETA` | π/5 (36°) | Polar angle of dome cap |
| `DOME_FOOT_R` | `15 · sin(π/5)` ≈ 8.82 | Base ring radius of dome cap |
| `DOME_CAP_H` | `15 · (1 − cos(π/5))` ≈ 2.87 | Cap height above base ring |
| `PLATE_THICKNESS` | 0.5 | Pedestal plate extrusion depth |
| `SHAFT_R_TOP` | 0.85 | Shaft top radius |
| `SHAFT_R_BOT` | 1.20 | Shaft bottom radius |
| `BELLOWS_R_TOP` | 1.60 | Bellows top radius |
| `BELLOWS_R_BOT` | 2.70 | Bellows bottom radius |
| `THROTTLE_W` | 8.5 | Throttle handle width |
| `THROTTLE_H` | 5.5 | Throttle handle height |
| `THROTTLE_D` | 2.4 | Throttle handle depth |
| `BTN_R` | 0.5 | Button radius (all buttons) |
| `BTN_H` | 0.25 | Button cylinder height |
| `BTN_SEGS` | 10 | Button cylinder segments |

### Animation Constants

| Constant | Value | Description |
|---|---|---|
| `STICK_MAX_ANGLE` | π/6 (30°) | Max pitch/roll tilt |
| `TWIST_MAX_ANGLE` | π/4 (45°) | Max yaw twist |
| `THROTTLE_TILT_ANGLE` | π/6 (30°) | Max throttle lever tilt |
| `ROCKER_MAX_ANGLE` | π/24 (7.5°) | Max rocker tilt |
| `DEADZONE` | 0.05 | Axis deadzone threshold |
| `SMOOTH` | 0.12 | Lerp factor for smooth interpolation |

### Derived Layout Values

| Expression | ≈ Value (cm) | Description |
|---|---|---|
| `2 · DOME_FOOT_R` | ≈ 17.64 | `plateHalfLen` – half-length of stadium pedestal |
| `DOME_FOOT_R` | ≈ 8.82 | `plateHalfZ` – half-width of stadium pedestal |
| `PLATE_THICKNESS + DOME_CAP_H · 0.4` | ≈ 1.65 | `centerBtnY` – button pair Y on dome slope |
| `DOME_FOOT_R · 0.9 − BTN_R` | ≈ 7.44 | `centerBtnZ` – btn10/11 Z position |
| `DOME_RADIUS / 2 − BTN_R / 2` | ≈ 7.25 | `triBtnZ` – triangle buttons Z position |
| `THROTTLE_W / 2` | 4.25 | `sideX` – thumb button column X |
| `−THROTTLE_D / 2` | −1.2 | `backZ` – back-face button Z |

## Physical Components

### Pedestal (shared base)

Stadium-shaped plate connecting both units (rectangle with semicircular ends).

- Geometry: `ExtrudeGeometry(stadiumShape, extrudePath)` along Y axis
- Shape: `plateHalfLen` × `plateHalfZ` with semicircle arcs at ±X ends
- The domes sit tangentially at center (touching at X = 0)
- Top surface at Y = `PLATE_THICKNESS` (0.5 cm)

### Dome Bases

Both the joystick and throttle unit sit on dome-shaped bases (sphere caps).

- Shape: `SphereGeometry(DOME_RADIUS, 24, 14, 0, 2π, 0, DOME_THETA)`
- Cap height: ≈ 2.87 cm
- Positioned so the base ring sits at Y = 0 of the group (dome shifted by `−DOME_RADIUS + DOME_CAP_H`)
- `makeDome(radius, pos)` helper creates group + dome mesh + adds to scene

## Joystick Unit (right hand)

Group variable: `stickDome`
Position on pedestal: `(DOME_FOOT_R, PLATE_THICKNESS, 0)` ≈ `(8.82, 0.5, 0)` cm

### Scene Graph Hierarchy

All Y values are in cm (code multiplies by `S`).

```
stickDome (dome group @ DOME_FOOT_R, PLATE_THICKNESS, 0)
 ├─ dome (sphere cap, r=15)
 └─ gimbalPivot (Group, y=DOME_CAP_H−1.28 ≈ 1.59) ← animated: rotation.x (pitch), rotation.z (roll)
     └─ twistPivot (Group) ← animated: rotation.y (yaw)
         ├─ bellows (rubber cylinder, 1.60→2.70, h=2.77, y=0.51)
         ├─ shaft (tapered cylinder, 0.85→1.20, h=11.92, y=7.45)
         ├─ lblStick (floating label, y=24.5)
         └─ gripGroup (Group, y=12.77)
             ├─ gripLower (cylinder, r=1.79→1.49, h=3.83)
             ├─ gripUpper (cylinder, r=1.49→1.79, h=4.26, y=4.04, top face sloped 45°)
             ├─ capTilt (Group, y=6.17, rot.x=π/4) ─ tilted 45° away from camera
             │   ├─ gripHemi (hemisphere, r=1.79, rot.x=π → dome faces down)
             │   ├─ capDisc (circle, r=1.79, flat top surface, y=0.002)
             │   ├─ hatGroup (Group, x=−1.19, y=0.5)
             │   │   ├─ hat base (cylinder, r=0.77, h=0.34)
             │   │   └─ hatKnob (sphere, r=0.55, y=0.43) ← animated: position.x/z from axis 9
             │   ├─ btn1 (cylinder, y=0.5, center) ← Button 1
             │   └─ btn3 (cylinder, x=+1.19, y=0.5) ← Button 3
             ├─ btn2 (cylinder, red emissive, x=1.0, y=5.2, z=−1.5, rot YXZ) ← Button 2, grip side near trigger
             └─ triggerPivot (Group, y=5.53, z=−1.49) ← animated: rotation.x on button 0
                 ├─ triggerBody (box, 0.85×2.34×0.77, red) ← Button 0
                 └─ triggerGuard (torus arc, r=0.94, tube=0.13)
```

### Stick Controls

| Control | Type | Axis/Button | Animation |
|---|---|---|---|
| Roll | Analog axis | Axis 0 | `gimbalPivot.rotation.z = -smooth.roll * STICK_MAX_ANGLE` |
| Pitch | Analog axis | Axis 1 | `gimbalPivot.rotation.x = smooth.pitch * STICK_MAX_ANGLE` |
| Yaw (Twist) | Analog axis | Axis 5 | `twistPivot.rotation.y = smooth.yaw * TWIST_MAX_ANGLE` (axis inverted) |
| Trigger | Button | Button 0 | `triggerPivot.rotation.x = −0.35` |
| Center top | Button | Button 1 | Material swap → `matBtnOn` (green glow) |
| Side grip | Button | Button 2 | Material swap (red emissive → `matBtnOn`) |
| Right top | Button | Button 3 | Material swap → `matBtnOn` (green glow) |
| Hat switch | 8-way POV | Axis 9 | `hatKnob` position offset ±0.21 cm |

> Button 2 is a red emissive button on the grip side (child of `gripGroup`, not `capTilt`), positioned near the trigger.

## Throttle Unit (left hand)

Group variable: `throttleDome`
Position on pedestal: `(−DOME_FOOT_R, PLATE_THICKNESS, 0)` ≈ `(−8.82, 0.5, 0)` cm

### Scene Graph Hierarchy

```
throttleDome (dome group @ −DOME_FOOT_R, PLATE_THICKNESS, 0)
 ├─ dome (sphere cap, r=15)
 └─ throttlePivot (Group, y=DOME_CAP_H−1.28 ≈ 1.59) ← animated: rotation.x (throttle tilt ±π/6)
     ├─ throttleBellows (rubber cylinder, 1.60→2.70, h=2.77, y=0.51)
     ├─ throttleShaft (tapered cylinder, 0.85→1.20, h=2.6, y=3.2)
     ├─ throttleHandle (RoundedBoxGeometry 8.5×5.5×2.4, corner=0.5, y=7.25)
     ├─ tBtn4 (cylinder, x=4.25, y=8.75, rot.z=π/2) ← Button 4
     ├─ tBtn5 (cylinder, x=4.25, y=7.55, rot.z=π/2) ← Button 5
     ├─ tBtn6 (cylinder, x=4.25, y=6.35, rot.z=π/2) ← Button 6
     ├─ tBtn7 (cylinder, x=4.25, y=5.35, rot.z=π/2) ← Button 7 (extra gap)
     ├─ tBtn8 (cylinder, x=2.125, y=8.75, z=−1.2, rot.x=π/2) ← Button 8
     ├─ tBtn9 (cylinder, x=2.125, y=7.55, z=−1.2, rot.x=π/2) ← Button 9
     └─ rockerPivot (Group, x=−1.0625, y=8.75, z=−1.2) ← animated: rotation.y from axis 7
         ├─ rockerSwitch (box, 4.675×0.85×0.43)
         └─ lblThrottle (floating label)
```

### Throttle Controls

| Control | Type | Axis/Button | Animation |
|---|---|---|---|
| Throttle lever | Analog axis | Axis 2 | `throttlePivot.rotation.x = smooth.throttle * THROTTLE_TILT_ANGLE` (axis inverted) |
| Rocker switch | Analog axis | Axis 7 | `rockerPivot.rotation.y = -smooth.rocker * ROCKER_MAX_ANGLE` |
| Thumb top | Button | Button 4 | Material swap |
| Thumb mid | Button | Button 5 | Material swap |
| Thumb bottom | Button | Button 6 | Material swap |
| Thumb lower | Button | Button 7 | Material swap |
| Back-face top | Button | Button 8 | Material swap |
| Back-face bottom | Button | Button 9 | Material swap |

### Throttle Handle Faces

```
         +Y (top)
          │
  −Z ─────┼───── +Z
 (back)   │    (front / user-facing)
          │
   +X side = joystick-facing (right) → Buttons 4–7 (thumb column)
   −X side = outer (left)
   −Z face = back → Rocker (left-center) + Buttons 8–9 (right of rocker)
```

## Base Buttons (on pedestal)

All base buttons use `btnSmGeo` (same `BTN_R` / `BTN_H` as `btnGeo`).

### Button Pair: btn10 + btn11

In front of the right joystick dome, on the dome slope.

- Y = `PLATE_THICKNESS + DOME_CAP_H · 0.4` ≈ 1.65 cm
- X = `DOME_FOOT_R ± 0.64` cm (centered on joystick dome)
- Z = `DOME_FOOT_R · 0.9 − BTN_R` ≈ 7.44 cm
- Tilted: `rotation.x = DOME_THETA / 2` to follow dome curvature

| Button | Index | X Position (cm) |
|---|---|---|
| btn10 | 10 | DOME_FOOT_R − 0.64 ≈ 8.18 |
| btn11 | 11 | DOME_FOOT_R + 0.64 ≈ 9.46 |

### LED Toggle (index 16)

Firmware-handled only. Pressing it toggles the on-board LED. It does not generate a Gamepad API event. In `allButtons`, index 16 is `null` (not added to the scene).

### Button Group: Triangle

Between the two stick units, centered at X = 0, on the pedestal surface.

- Y = `PLATE_THICKNESS` = 0.5 cm
- Z = `DOME_RADIUS / 2 − BTN_R / 2` ≈ 7.25 cm

| Button | Index | X Position (cm) |
|---|---|---|
| btn14 | 14 | 0 (center) |
| btn12 | 12 | −1.70 (left) |
| btn13 | 13 | +1.70 (right) |

## Complete Axis Index Table

| Index | Input | Value Range | Notes |
|---|---|---|---|
| 0 | Stick X (Roll) | −1 … +1 | Deadzone applied |
| 1 | Stick Y (Pitch) | −1 … +1 | Deadzone applied |
| 2 | Throttle | −1 … +1 | Deadzone applied, inverted in code |
| 5 | Twist (Yaw) | −1 … +1 | Deadzone applied, inverted in code |
| 7 | Rocker | −1 … +1 | Deadzone applied |
| 9 | Hat (POV) | −1 … +1 | Encoded: `round((v+1)·3.5)` → 0–7 = directions, 8 = neutral |

## Complete Button Index Table

| Index | Variable | Location | Description |
|---|---|---|---|
| 0 | `triggerBody` | Stick | Trigger (back, index finger, red) |
| 1 | `btn1` | Stick cap | Center top button (on tilted hemisphere) |
| 2 | `btn2` | Stick grip | Side button near trigger (red emissive, on gripGroup) |
| 3 | `btn3` | Stick cap | Right top button (on tilted hemisphere) |
| 4 | `tBtn4` | Throttle (+X) | Thumb column, top |
| 5 | `tBtn5` | Throttle (+X) | Thumb column, middle |
| 6 | `tBtn6` | Throttle (+X) | Thumb column, bottom |
| 7 | `tBtn7` | Throttle (+X) | Thumb column, lower (extra gap) |
| 8 | `tBtn8` | Throttle (−Z) | Back face, right of rocker, top |
| 9 | `tBtn9` | Throttle (−Z) | Back face, right of rocker, bottom |
| 10 | `btn10` | Base | Pair left, front of joystick dome |
| 11 | `btn11` | Base | Pair right, front of joystick dome |
| 12 | `btn12` | Base | Triangle left |
| 13 | `btn13` | Base | Triangle right |
| 14 | `btn14` | Base | Triangle center |
| 15 | — | — | Unused (`null` in `allButtons`) |
| 16 | — | — | LED toggle (firmware-only, `null` in `allButtons`) |

## Materials

| Variable | Hex | Usage |
|---|---|---|
| `matBase` | `#3a3a4a` | Pedestal plate (declared but plate uses inline material) |
| `matDome` | `#3a3a4a` | Dome caps |
| `matHousing` | `#4a4a5a` | Grip hemisphere (`gripHemi`), cap disc (`capDisc`) |
| `matShaft` | `#555566` | Shaft, trigger guard |
| `matGrip` | `#3a3a4a` | Grip cylinders (`gripLower`, `gripUpper`), throttle handle |
| `matRubber` | `#2a2a3a` | Bellows (both sticks) |
| `matTrigger` | `#ff4422` | Trigger body, hat knob (red accent) |
| `matBtn` | `#6666aa` | All buttons (off state), rocker switch |
| `matBtnOn` | `#44ff66` | Buttons (pressed state, green glow, emissive: 0.5) |
| `matHat` | `#7777aa` | Hat switch base |
| `btn2` default | `#ff2200` | Button 2 custom material (red, emissive: 0.3) |

## Keyboard Fallback

Keyboard input drives the same state values, both when no gamepad is connected and alongside a connected gamepad:

| Key | State property | Behavior |
|---|---|---|
| W / S | `state.pitch` | Lerp toward −1 / +1 (rate 0.15) |
| A / D | `state.roll` | Lerp toward −1 / +1 (rate 0.15) |
| Q / E | `state.yaw` | Lerp toward −1 / +1 (rate 0.15) |
| R / F | `state.throttle` | Increment / decrement by 0.04 per frame |

When keys are released without a gamepad connected, values decay (multiply by 0.92 each frame).
