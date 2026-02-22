# T.Flight HOTAS One – Setup Model Reference

This document describes the physical layout and 3D model structure of the Thrustmaster T.Flight HOTAS One as implemented in this project. It is intended as a machine-readable reference for AI agents working on the codebase.

## Coordinate System

Three.js right-handed coordinate system:

- **+X** = right (toward joystick unit)
- **−X** = left (toward throttle unit)
- **+Y** = up
- **+Z** = toward the user / camera (front)
- **−Z** = away from the user (back)

The model is centered at world origin. A debug `AxesHelper` (0.1 length) is placed at `(0, 1, 0)` for orientation.

## Model Constants

All layout dimensions are derived from shared constants defined in the `Model Constants` region:

| Constant | Value | Description |
|---|---|---|
| `DOME_RADIUS` | 3.5 | Sphere radius – canonical size parameter for domes |
| `DOME_THETA` | π/5 (36°) | Polar angle of dome cap |
| `DOME_FOOT_R` | `DOME_RADIUS · sin(DOME_THETA)` ≈ 2.06 | Base ring radius of dome cap |
| `DOME_CAP_H` | `DOME_RADIUS · (1 − cos(DOME_THETA))` ≈ 0.67 | Cap height above base ring |
| `DOME_X` | = `DOME_FOOT_R` ≈ 2.06 | X offset so domes touch tangentially at center |
| `PLATE_THICKNESS` | 0.2 | Pedestal plate extrusion depth |
| `BTN_R` | 0.1 | Standard button radius |
| `BTN_R_SM` | 0.09 | Small button radius |
| `BTN_H` | 0.06 | Button cylinder height |
| `BTN_SEGS` | 10 | Button cylinder segments |
| `SHAFT_R_TOP` | 0.2 | Shaft top radius |
| `SHAFT_R_BOT` | 0.28 | Shaft bottom radius |
| `BELLOWS_R_TOP` | 0.4 | Bellows top radius |
| `BELLOWS_R_BOT` | 0.65 | Bellows bottom radius |

### Derived Layout Values

| Expression | ≈ Value | Description |
|---|---|---|
| `DOME_X + DOME_FOOT_R` | ≈ 4.12 | `plateHalfLen` – half-length of stadium pedestal |
| `DOME_FOOT_R` | ≈ 2.06 | `plateHalfZ` – half-width of stadium pedestal |
| `PLATE_THICKNESS + BTN_H / 2` | 0.23 | `centerBtnY` / `triBtnY` – button Y on pedestal surface |
| `DOME_FOOT_R - BTN_R` | ≈ 1.96 | `centerBtnZ` – btn10/11 Z, inside front edge |
| `DOME_RADIUS / 2 - BTN_R / 2` | ≈ 1.70 | `triBtnZ` – triangle buttons Z position |

## Physical Components

### Pedestal (shared base)

Stadium-shaped plate connecting both units (rectangle with semicircular ends).

- Geometry: `ExtrudeGeometry(stadiumShape, depth: PLATE_THICKNESS)`
- Shape: `plateHalfLen` × `plateHalfZ` with semicircle arcs at ±X ends
- The domes sit tangentially at center (touching at X = 0)
- Top surface at Y = `PLATE_THICKNESS`

### Dome Bases

Both the joystick and throttle unit sit on dome-shaped bases (sphere caps).

- Shape: `SphereGeometry(DOME_RADIUS, 24, 14, 0, 2π, 0, DOME_THETA)`
- Cap height: `DOME_RADIUS · (1 − cos(DOME_THETA))` ≈ 0.67
- Positioned so the base ring sits at Y = 0 of the group (dome shifted by `−DOME_RADIUS + DOME_CAP_H`)
- `makeDome(radius, parent, pos)` helper creates group + dome mesh

## Joystick Unit (right hand)

Group position on pedestal: `(DOME_X, PLATE_THICKNESS, 0)` ≈ `(2.06, 0.2, 0)`

### Scene Graph Hierarchy

```
stickUnit (dome group @ +DOME_X, PLATE_THICKNESS, 0)
 ├─ dome (sphere cap, r=DOME_RADIUS)
 ├─ gimbalPivot (Group, y=1.05) ← animated: rotation.x (pitch), rotation.z (roll)
 │   └─ twistPivot (Group) ← animated: rotation.y (yaw)
 │       ├─ bellows (rubber cylinder, BELLOWS_R_TOP→BELLOWS_R_BOT)
 │       ├─ shaft (tapered cylinder, SHAFT_R_TOP→SHAFT_R_BOT)
 │       └─ gripGroup (Group, y=12.77*S)
 │           ├─ gripLower (cylinder, palm)
 │           ├─ gripUpper (cylinder, top sloped 45° to match hemisphere)
 │           ├─ capTilt (Group, y=6.17*S, rot.x=π/4) ─ tilted 45° away from camera
 │           │   ├─ gripHemi (hemisphere, r=1.79*S, dome down)
 │           │   ├─ capDisc (circle, flat top surface)
 │           │   ├─ hatGroup (Group, x=−1.19*S, y=0.5*S)
 │           │   │   ├─ hat base (cylinder)
 │           │   │   └─ hatKnob (sphere) ← animated: position.x/z from axis 9
 │           │   ├─ btn1 (cylinder, y=0.5*S, center) ← Button 1
 │           │   └─ btn3 (cylinder, x=+1.19*S, y=0.5*S) ← Button 3
 │           ├─ btn2 (cylinder, red, x=1.0*S, y=5.2*S, z=−1.5*S) ← Button 2, near trigger
 │           ├─ triggerPivot (Group, y=5.53*S, z=−1.49*S) ← animated: rotation.x on button 0
 │           │   ├─ triggerBody (box, red) ← Button 0
 │           │   └─ triggerGuard (torus arc)
```

### Stick Controls

| Control | Type | Axis/Button | Animation |
|---|---|---|---|
| Roll | Analog axis | Axis 0 | gimbalPivot.rotation.z, max ±π/6 |
| Pitch | Analog axis | Axis 1 | gimbalPivot.rotation.x, max ±π/6 |
| Yaw (Twist) | Analog axis | Axis 5 | twistPivot.rotation.y, max ±π/4 (inverted) |
| Trigger | Button | Button 0 | triggerPivot.rotation.x = −0.35 |
| Center top | Button | Button 1 | Material swap (green glow) |
| Side near trigger | Button | Button 2 | Material swap (red → green glow) |
| Right top | Button | Button 3 | Material swap (green glow) |
| Hat switch | 8-way POV | Axis 9 | hatKnob position offset ±0.05 |

> Button 2 is a red button on the grip side, next to the trigger, on the hemisphere surface.

## Throttle Unit (left hand)

Group position on pedestal: `(−DOME_X, PLATE_THICKNESS, 0)` ≈ `(−2.06, 0.2, 0)`

### Scene Graph Hierarchy

```
throttleUnit (dome group @ −DOME_X, PLATE_THICKNESS, 0)
 ├─ dome (sphere cap, r=DOME_RADIUS)
 ├─ throttleBellows (rubber cylinder, BELLOWS_R_TOP→BELLOWS_R_BOT, y=0.12)
 ├─ throttleShaft (tapered cylinder, SHAFT_R_TOP→SHAFT_R_BOT, y=0.55)
 └─ throttlePivot (Group, y=0.95) ← animated: rotation.x (throttle tilt ±π/6)
     ├─ throttleHandle (box, 2.2 × 1.2 × 0.4, y=0.9)
     ├─ tBtn4 (cylinder, +X side, y=1.30) ← Button 4, rot.z=π/2
     ├─ tBtn5 (cylinder, +X side, y=1.05) ← Button 5, rot.z=π/2
     ├─ tBtn6 (cylinder, +X side, y=0.80) ← Button 6, rot.z=π/2
     ├─ tBtn7 (cylinder, +X side, y=0.40) ← Button 7 (extra gap), rot.z=π/2
     ├─ tBtn8 (cylinder, −Z face, x=0.7, y=1.30) ← Button 8, rot.x=π/2
     ├─ tBtn9 (cylinder, −Z face, x=0.7, y=1.05) ← Button 9, rot.x=π/2
     └─ rockerPivot (Group, x=−0.25, y=1.3, z=−0.24) ← animated: rotation.y from axis 7
         └─ rockerSwitch (box, 1.2 × 0.2 × 0.1)
```

### Throttle Controls

| Control | Type | Axis/Button | Animation |
|---|---|---|---|
| Throttle lever | Analog axis | Axis 2 | throttlePivot.rotation.x, range ±π/6 |
| Rocker switch | Analog axis | Axis 7 | rockerPivot.rotation.y, max ±π/24 (inverted) |
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

All base buttons sit on the pedestal surface at `Y = PLATE_THICKNESS + BTN_H / 2` ≈ 0.23.

### Button Pair: btn10 + btn11

In front of the right joystick dome, just inside the pedestal front edge.

- Y = `PLATE_THICKNESS + BTN_H / 2` ≈ 0.23
- X = `DOME_X ± 0.15` (centered on joystick dome)
- Z = `DOME_FOOT_R − BTN_R` ≈ 1.96

| Button | Index | Position offset from center |
|---|---|---|
| btn10 | 10 | X − 0.15 |
| btn11 | 11 | X + 0.15 |

### LED Toggle (index 16)

Firmware-handled only. Pressing it toggles the on-board LED. It does not generate a Gamepad API event and is **not added to the scene** (`btnXbox = null`).

### Button Group: Triangle

Between the two stick units, centered at X = 0, on the pedestal surface.

- Y = `PLATE_THICKNESS + BTN_H / 2` ≈ 0.23
- Z = `DOME_RADIUS / 2 − BTN_R / 2` ≈ 1.70

| Button | Index | Position |
|---|---|---|
| btn14 | 14 | X = 0 (center) |
| btn12 | 12 | X = −0.4 (left) |
| btn13 | 13 | X = +0.4 (right) |

## Complete Axis Index Table

| Index | Input | Value Range | Notes |
|---|---|---|---|
| 0 | Stick X (Roll) | −1 … +1 | Deadzone applied |
| 1 | Stick Y (Pitch) | −1 … +1 | Deadzone applied |
| 2 | Throttle | −1 … +1 | Deadzone applied |
| 5 | Twist (Yaw) | −1 … +1 | Inverted in code |
| 7 | Rocker | −1 … +1 | Deadzone applied, inverted |
| 9 | Hat (POV) | −1 … +1 | Encoded: `round((v+1)·3.5)` → 0–7 = directions, 8 = neutral |

## Complete Button Index Table

| Index | Variable | Location | Description |
|---|---|---|---|
| 0 | triggerBody | Stick | Trigger (back, index finger) |
| 1 | btn1 | Stick | Center top button |
| 2 | btn2 | Stick grip | Side button near trigger (red, on hemisphere) |
| 3 | btn3 | Stick | Right top button |
| 4 | tBtn4 | Throttle (+X) | Thumb column, top |
| 5 | tBtn5 | Throttle (+X) | Thumb column, middle |
| 6 | tBtn6 | Throttle (+X) | Thumb column, bottom |
| 7 | tBtn7 | Throttle (+X) | Thumb column, lower (extra gap) |
| 8 | tBtn8 | Throttle (−Z) | Back face, right of rocker, top |
| 9 | tBtn9 | Throttle (−Z) | Back face, right of rocker, bottom |
| 10 | btn10 | Base | Pair left, front of joystick dome |
| 11 | btn11 | Base | Pair right, front of joystick dome |
| 12 | btn12 | Base | Triangle left |
| 13 | btn13 | Base | Triangle right |
| 14 | btn14 | Base | Triangle center |
| 15 | — | — | Unused |
| 16 | btnXbox | — | LED toggle (firmware-only, not in scene) |

## Materials

| Variable | Hex | Usage |
|---|---|---|
| matBase | `#3a3a4a` | Pedestal plate, dome bases |
| matHousing | `#4a4a5a` | Grip hemisphere, cap disc, structural housings |
| matShaft | `#555566` | Shaft, trigger guard |
| matGrip | `#3a3a4a` | Grip cylinder surfaces |
| matRubber | `#2a2a3a` | Bellows |
| matTrigger | `#ff4422` | Trigger body, hat knob (red) |
| matBtn | `#6666aa` | All buttons (off state) |
| matBtnOn | `#44ff66` | Buttons (pressed, green glow, emissive) |
| matHat | `#7777aa` | Hat switch base |
