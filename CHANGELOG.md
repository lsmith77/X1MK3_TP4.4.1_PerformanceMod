# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Fixed - QML Issues

#### FX Section Mode Switching

- **Issue**: References to undefined `fx_section` variable caused FX mode switching to fail
- **Impact**: When toggling between FX and Mixer modes on the device, or when users changed FX assignment settings, the mode wouldn't properly switch. Device could lock up in the wrong mode or become unresponsive to mode change attempts.
- **Solution**: Corrected camelCase identifier from `fx_section` to `fxSection` throughout X1MK3.qml (4 occurrences)
- **Files**:
  - `qml/CSI/X1MK3/X1MK3.qml`

#### Fader Soft Takeover Indicator

- **Issue**: Missing `faderSoftTakeoverDirection` property definition in FXScreen.qml
- **Impact**: When adjusting faders with soft takeover enabled, the visual feedback animation (up/down arrows indicating takeover direction) wouldn't display. Users had no visual indication of whether the fader was in soft takeover mode or which direction it was taking over.
- **Solution**: Added missing MappingProperty definition for fader-specific soft takeover direction tracking
- **Files**:
  - `qml/Screens/X1MK3/FXScreen.qml`

#### FX Unit Assignment Display

- **Issue**: Image paths concatenated undefined `leftFxIdx` and `rightFxIdx` values on initialization, resulting in `undefined_A.png` file not found errors
- **Impact**: The small display on the X1MK3 showed blank/broken image areas instead of FX unit assignments (1_A.png, 2_A.png, etc.). Users couldn't see at a glance which FX units were assigned to the left and right sides of the controller.
- **Solution**: Added null-safety checks to image source bindings using ternary operators
- **Files**:
  - `qml/Screens/X1MK3/ModeScreen.qml`

**Result**: Restored proper FX section functionality, fader takeover visual feedback, and FX unit indicator display on the X1MK3 controller.
