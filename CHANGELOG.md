# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.0] - 2026-02-22

### Added
- **LICENSE** file (CC BY-SA 4.0 full legal text)
- **OutlineEffect** for toon-style rendering with black outlines
- **Floating 3D labels** on stick (Roll/Pitch/Yaw) and throttle (Throttle/Rocker) with live value updates
- **HUD overlay** showing raw axis values with bar graphs and button state indicators
- **Gamepad API** integration with HOTAS-priority detection (prefers T.Flight HOTAS One when multiple controllers are connected)
- **Keyboard fallback** (WASD + QE + RF) for testing without hardware
- **SETUP-MODEL.md** tracked in git (was previously untracked)
- Axis constants (`AXIS_ROLL`, `AXIS_PITCH`, `AXIS_YAW`, `AXIS_THROTTLE`, `AXIS_ROCKER`, `AXIS_HAT`) and animation constants (`STICK_MAX_ANGLE`, `TWIST_MAX_ANGLE`, etc.)
- Smoothed animation with `lerp` interpolation for all axes
- `applyDeadzone` utility function
- Button highlighting (`matBtnOn`) for pressed state feedback
- Hat switch knob displacement animation (8-way POV from axis 9)
- Trigger pull animation

### Changed
- **Unit system** converted from centimeters to meters (1 unit = 1 m) with `S = 0.01` conversion factor
- **Ground grid** extended to horizon (200 m radius, 2000 divisions, semi-transparent fade)
- Ground plane enlarged to 400 × 400 m
- Throttle handle raised 1 cm (`handleY` 6.25 → 7.25 cm)
- Throttle shaft shortened to reach only from bellows top to handle bottom surface
- Throttle shaft made symmetric with joystick shaft (same bellows + shaft geometry)
- Renderer uses `THREE.NoToneMapping` for consistent OutlineEffect appearance
- `pollGamepad()` now prioritizes controllers with "hotas" in their ID string
- **README.md** fully rewritten: project framed as reusable 5-DoF controller reference, feature table, adaptation guide, CC BY-SA 4.0 license section, badges

### Removed
- Debug wireframe cube (replaced by full model)
- Idle breathing animation (superseded by gamepad-driven animation)

## [0.3.0] - 2026-02-18

### Added
- Stadium-shaped pedestal (rectangle + semicircular ends) replacing the old box
- Dome bases (sphere caps, `SphereGeometry` with θ = π/5, 36°) replacing cylinder mounts
- Throttle shaft + bellows connecting dome to throttle handle
- Debug axes helper at `(0, 1, 0)` with X/Y/Z labels for orientation
- `Model Constants` region with all shared dimension constants
- Derived layout values (`centerBtnY`, `triBtnY`, `centerBtnZ`, `triBtnZ`) computed from constants

### Changed
- Renamed `DOME_R` → `DOME_RADIUS` (clarifies it's the sphere radius, not foot radius)
- Renamed `PLATE_H` → `PLATE_THICKNESS` (clarifies it's a thickness, not a height)
- Repositioned btn10 + btn11 to pedestal surface (`Y = PLATE_THICKNESS + BTN_H / 2`) at front edge (`Z = DOME_FOOT_R − BTN_R`)
- Triangle buttons (btn12–14) now sit on pedestal surface with computed Y and Z positions
- LED toggle (index 16) removed from scene – firmware-only, kept as null reference
- SETUP-MODEL.md fully rewritten with constants table, derived values, updated scene graphs
- README.md updated: stadium pedestal, dome caps, corrected rocker face description

### Removed
- Hardcoded position values for base buttons (replaced by constant expressions)
- LED toggle mesh from scene (was never readable via Gamepad API)

## [0.2.1] - 2026-02-17

### Fixed
- Inverted yaw axis (twist) to match physical stick direction
- Inverted throttle axis to match lever direction
- Hat switch now reads 8-way POV from axis 9 (was incorrectly using buttons 12-15)
- Rocker switch now uses axis 7 (was incorrectly on axis 9)

### Changed
- Added Button 1 (prominent central button on grip top, below hat switch)
- Replaced LB/RB/Start/Back/Guide throttle buttons with correct thumb buttons 4-7
- Throttle buttons 4-5-6 arranged top-to-bottom, button 7 below with extra spacing
- Throttle buttons and rocker now move with the throttle lever
- Added ROCKER (Axis 7) label on throttle unit
- Removed Xbox-style button naming in favor of hardware-accurate indices
- Version bumped to v0.2.1

## [0.2.0] - 2026-02-17

### Added
- Accurate T.Flight HOTAS One model based on official Thrustmaster specs
- D-pad / thumb cross on grip (left side, angled for thumb reach)
- Thumb buttons on grip (right side, 3 buttons)
- Trigger guard geometry
- Finger groove texture strips on grip
- Rubber bellows boot at stick base
- Ergonomic hand rest with curved extruded shape
- Rubber feet on both base units
- Rocker switch on throttle unit side
- Base buttons on throttle: LB, RB, Start, Back, Guide (Xbox layout)
- Throttle handle with rounded top cap
- Hat switch direction indicators (N/E/S/W dots)
- Hat switch knob tilt animation from d-pad buttons
- ACES filmic tone mapping for better visual quality
- Fill light for more dimension
- Higher shadow map resolution (2048)

### Changed
- Joystick model completely rebuilt with proper proportions
- Throttle unit is now a separate scene group (reflecting detachable design)
- Labels now show axis indices (e.g. "THROTTLE (Axis 2)")
- Version bumped to v0.2.0

### Removed
- Skeleton figure (removed entirely)
- Unused bone/joint materials

## [0.1.0] - 2026-02-17

### Added
- Procedural 3D joystick model (stick with gimbal, twist, throttle, trigger, hat switch, buttons)
- Humanoid skeleton figure with bone segments and joints
- Real-time axis mapping: pitch, roll, yaw, throttle → skeleton body movements
- Gamepad API integration for T.Flight HOTAS One
- HUD overlay showing raw axis values and button states
- Keyboard fallback (WASD + QE + RF) for testing without hardware
- Idle breathing animation when no input is active
- OrbitControls for camera navigation
- Shadow mapping and multi-light setup
- README with axis mapping table and usage instructions
