# 🕹️ joystick-nav

**A reusable, procedural 3D model of a 5-DoF HOTAS controller with live Gamepad API bindings.**

Built as a reference implementation for the [Thrustmaster T.Flight HOTAS One](https://www.thrustmaster.com/products/t-flight-hotas-one/), this project provides an interactive, browser-based 3D visualization that maps all 5 degrees of freedom and 17 buttons in real time. It is designed to be easily adapted for generating controller bindings for different applications — with or without AI assistance.

> Created with the help of **Claude Opus 4.6**. The model description in [`SETUP-MODEL.md`](SETUP-MODEL.md) is structured so that an AI coding agent can understand the controller layout and generate input bindings for arbitrary applications.

![Three.js](https://img.shields.io/badge/Three.js-r170-blue)
![License](https://img.shields.io/badge/License-CC--BY--SA%204.0-orange)
![No Build](https://img.shields.io/badge/Build-None-green)

---

## Features

| | |
|---|---|
| 🎮 **5 Degrees of Freedom** | Pitch, Roll, Yaw (twist), Throttle, Rocker |
| 🧊 **Procedural 3D Model** | No external assets — geometry built entirely in code |
| ⚡ **Real-time Animation** | All axes and buttons animate the model live via Gamepad API |
| 📊 **HUD Overlay** | Raw axis values, bar graphs, and button state indicators |
| ⌨️ **Keyboard Fallback** | WASD + QE + RF — works standalone or alongside gamepad |
| 🏷️ **Floating Labels** | Live value readouts on stick and throttle |
| 🔍 **Orbit Camera** | Inspect the model from any angle |
| 🎨 **Outline Effect** | Toon-style outlines via Three.js `OutlineEffect` |

## Axis Mapping

| Axis | Index | Range | Animation |
|---|---|---|---|
| **Roll** (Stick X) | 0 | −1 … +1 | Stick tilts left / right |
| **Pitch** (Stick Y) | 1 | −1 … +1 | Stick tilts forward / backward |
| **Throttle** | 2 | −1 … +1 | Throttle lever tilts along rail |
| **Yaw** (Twist) | 5 | −1 … +1 | Stick rotates around vertical axis |
| **Rocker** | 7 | −1 … +1 | Rocker switch tilts on throttle back |
| **Hat** (8-way POV) | 9 | discrete | Hat knob displacement |

> **Note:** Axis indices may vary by OS and driver. Use the HUD to read raw values and adjust the `AXIS_*` constants if needed.

## Button Mapping

| Index | Variable | Location | Description |
|---|---|---|---|
| 0 | `triggerBody` | Stick | Trigger (index finger, red) |
| 1 | `btn1` | Stick cap | Center top button (on tilted hemisphere) |
| 2 | `btn2` | Stick grip | Side button near trigger (red) |
| 3 | `btn3` | Stick cap | Right top button (on tilted hemisphere) |
| 4 | `tBtn4` | Throttle +X | Thumb column, top |
| 5 | `tBtn5` | Throttle +X | Thumb column, middle |
| 6 | `tBtn6` | Throttle +X | Thumb column, bottom |
| 7 | `tBtn7` | Throttle +X | Thumb column, lower (extra gap) |
| 8 | `tBtn8` | Throttle −Z | Back face, right of rocker, top |
| 9 | `tBtn9` | Throttle −Z | Back face, right of rocker, bottom |
| 10 | `btn10` | Base | Pair left, front of joystick dome |
| 11 | `btn11` | Base | Pair right, front of joystick dome |
| 12 | `btn12` | Base | Triangle left |
| 13 | `btn13` | Base | Triangle right |
| 14 | `btn14` | Base | Triangle center |
| 15 | — | — | (unused) |
| 16 | — | Base | LED toggle (firmware-only, no API event) |

## Controls

### 🎮 With Joystick

Plug in the T.Flight HOTAS One and press any button — the browser detects it automatically. If multiple controllers are connected, the HOTAS is prioritized by name.

### ⌨️ Keyboard

Keyboard input works both standalone and alongside a connected gamepad.

| Key | Function |
|---|---|
| `W` / `S` | Pitch |
| `A` / `D` | Roll |
| `Q` / `E` | Yaw |
| `R` / `F` | Throttle up / down |

Click to toggle auto-rotate. Mouse drag to orbit the camera.

## Getting Started

```bash
# No dependencies, no build step.
# Just open in a browser:
open index.html

# Or use any local server, e.g.:
npx serve .
```

## Project Structure

```
joystick-nav/
├── index.html        # Main visualizer (single-file, self-contained)
├── SETUP-MODEL.md    # Machine-readable model & layout reference (for AI agents)
├── CHANGELOG.md      # Version history (Keep a Changelog)
├── LICENSE           # CC BY-SA 4.0 full legal text
└── README.md         # You are here
```

## Tech

- **[Three.js](https://threejs.org/) r170** — ES Modules via CDN (`importmap`)
- **[Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API)** — `navigator.getGamepads()`
- **Procedural geometry** — no glTF, no OBJ, no external models
- **Smooth interpolation** — all inputs are lerp'd for fluid animation
- **OutlineEffect** — toon-style rendering

## Adapting for Other Controllers

The model is parameterized via constants at the top of the script. To adapt:

1. **Read your controller's axes** using the HUD — note which index maps to which physical axis
2. **Update `AXIS_*` constants** (`AXIS_ROLL`, `AXIS_PITCH`, `AXIS_YAW`, `AXIS_THROTTLE`, `AXIS_ROCKER`, `AXIS_HAT`)
3. **Adjust axis transforms** if needed (inversion, deadzone, scaling)
4. **For AI-assisted binding generation**: point your agent at [`SETUP-MODEL.md`](SETUP-MODEL.md) for the full scene graph and constant definitions

## License

This work is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/).

You are free to share and adapt this work, even commercially, as long as you give appropriate credit and distribute derivatives under the same license.

**© 2026 Felix Woitzel / [cake23.de](https://cake23.de)**
