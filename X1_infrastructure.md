# X1MK3 Infrastructure & Core Controls

**Source:** X1MK3 Performance Mod v12 (Community Forum)  
**Commit:** https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_infrastructure.md  
**Traktor:** 4.4.1

---

## What It Does

This is the **foundational infrastructure** on which all other X1MK3 Performance Mod features depend. It establishes setup system (3 customizable pages), MODE button control for overlay switching, SHIFT key handling for modifier states, screen state management for tracking overlays and modes, device setup on startup, and LED/visual feedback foundation. **All other features depend on this infrastructure.**

---

## Features

### Setup System

- **3 Customizable Pages** - Accessible via MODE button (hold 1 second)
- **Visual Feedback** - FX screens display current custom toggle status
- **Persistent State** - Setup selections maintained across sessions
- **Per-Overlay Configuration** - Different settings for each overlay context

### MODE Button States

- **Single Tap** - Cycle through FX Overlay 1 → FX Overlay 2 → Mixer Overlay
- **Single Tap w/ Setup L4** - Switch deck pairs (AB ↔ CD)
- **Double Tap** - Cycle through all deck combinations
- **Hold 1+ second** - Enter Setup mode (tap to cycle pages, hold buttons to change)

### SHIFT Key Handling

- **Tap** - Toggle Browser Mode (when enabled)
- **Hold** - Unlock special functions on other buttons
- **Hold + Button Combos** - Access shift-layer parameters
- **Visual Indicator** - SHIFT button LED blinks when Browser Mode active

### Overlay State Tracking

- FX Overlay 1 (primary effects control)
- FX Overlay 2 (secondary effects control)
- Mixer Overlay (custom mixer + stem/remix controls)

### Deck State Management

- Single deck selection (A, B, C, or D)
- Deck pair selection (AB, AC, AD, BC, BD, CD)
- Focus tracking for multi-deck operations

---

## Modified Files

**File:** `CSI/X1MK3/X1MK3.qml` (Main entry point)

This is the core QML file that:

- Defines all property states (shift, overlay, deck focus, etc.)
- Sets up timers for MODE button timing (single vs. double tap)
- Manages DeviceSetupState transitions
- Coordinates all overlay and setup page logic
- Routes SHIFT and MODE button events

**Note:** Without git history, we cannot provide exact line numbers. Refer to the actual file for implementation details.

---

## Configuration

### MOD Settings

Add this to your main X1MK3.qml controller file:

```qml
// --- MOD SETTINGS: X1MK3 Infrastructure ---
MappingPropertyDescriptor {
    id: infrastructureEnabledProp
    path: "app.traktor.settings.x1mk3_infrastructure_enabled"
    initialValue: true  // Set to false to disable all overlay/mode features
}
// --- END MOD SETTINGS ---
```

---

## See Also

- All other features depend on this infrastructure
- Related: [Tempo Control](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_tempo-control.md), [Beatgrid Control](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_beatgrid-control.md), [Browser Mode](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_browser-mode.md)
- Advanced: [Custom Overmapping Support](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_overmapping-support.md)

---

## Core Algorithms

### MODE Button Logic

```
Press MODE:
  if (singlePressed and not yet double-pressed):
    start doubleClickTimer
    Wait 300ms for second press

  if (secondPress arrives within 300ms):
    Execute DOUBLE-TAP action (cycle decks)
    Cancel singleClickTimer

  else (timeout):
    Execute SINGLE-TAP action (cycle overlay)

Hold MODE (1+ second):
  Enter Setup Mode
  On each TAP: advance setupPage (1→2→3→1)
  On each HOLD of button: toggle that button's value
```

### SHIFT Modifier System

```
SHIFT Press:
  Set shiftProp = true
  Start holdShift_countdown (200ms)

SHIFT Hold (< 200ms):
  Access shift-layer functions
  (Tempo adjust, grid controls, etc.)

SHIFT Tap (release < 200ms):
  Toggle Browser Mode (if enabled)
  Close pre-existing browser view
```

### Overlay Cycling

```
Single Tap MODE:
  overlayProp = (overlayProp + 1) mod 3

  if (overlayProp == 3):
    Show Mixer Overlay
  else:
    Show FX Overlay (1 or 2)
    Update FX screens with custom names
```

---

## Dependencies

**This is the FOUNDATION feature.** All other X1MK3 features depend on:

- ✅ Setup state management (pages, custom toggles)
- ✅ MODE button behavior (overlay/deck switching)
- ✅ SHIFT key state tracking
- ✅ Device setup state (hardware ready)
- ✅ Screen feedback mechanism

**Features that depend on this:**

- X1_tempo-control.md (needs SHIFT state)
- X1_beatgrid-control.md (needs SHIFT state)
- X1_mixer-overlay.md (needs overlay state)
- X1_stem-remix-controls.md (needs setup + overlay state)
- X1_browser-mode.md (needs SHIFT + setup state)
- X1_transport-led-feedback.md (needs device state)

---

## Implementation Notes

### Why Infrastructure First?

1. **Single Source of Truth** - All overlays reference shared state variables
2. **Consistent Behavior** - MODE/SHIFT handling identical across all features
3. **Setup System** - Must be initialized before any feature uses custom toggles
4. **Screen Feedback** - All features route through same display mechanism

### Color & Layer Coding (Setup)

**Knob Colors (for visual identification during setup):**

- Blue = Knob 1
- Warm Yellow = Knob 2
- Light Orange = Knob 3
- Red = Knob 4

**Parameter Layers (what you're controlling):**

- White = No shift layer (normal)
- Yellow = Non-shift layer (normal operation)
- Dark Orange = Shift-layer (when holding SHIFT)

### State Management Pattern

Each feature that adds new controls typically:

1. Checks infrastructure state (shift, overlay, deck focus)
2. Enables/disables wiring based on current state
3. Updates screen feedback via shared mechanism
4. Respects setup configuration

---

## Testing Checklist

- [ ] MODE button single-tap cycles through overlays correctly (FX1 → FX2 → Mixer → FX1)
- [ ] MODE button double-tap switches deck pairs (AB ↔ CD ↔ etc.)
- [ ] MODE button hold (1+ sec) enters Setup mode
- [ ] In Setup mode: tap MODE cycles through pages (1→2→3→1)
- [ ] In Setup mode: holding L1-R4 buttons toggles their values
- [ ] In Setup mode: Shift+holding buttons toggles shift-layer values
- [ ] SHIFT button tap toggles Browser Mode (if enabled in Setup)
- [ ] SHIFT button hold enables shift-layer functions
- [ ] SHIFT LED blinks when Browser Mode is active
- [ ] Overlay state is reflected on FX screens
- [ ] Current deck is tracked and used by other features
- [ ] Device setup state is properly initialized
- [ ] Setup custom toggle changes are retained after restart
- [ ] No state conflicts between overlays

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                  X1MK3 Infrastructure                   │
│  (Setup System, MODE, SHIFT, State Management, Screen)  │
└────────────┬──────────────────────────────┬─────────────┘
             │                              │
    ┌────────▼────────┐         ┌──────────▼──────────┐
    │  MODE Button    │         │  SHIFT Key          │
    │  • Single Tap   │         │  • Tap (toggle)     │
    │  • Double Tap   │         │  • Hold (layer)     │
    │  • Hold (setup) │         └─────────┬───────────┘
    └────────┬────────┘                   │
             │                            │
    ┌────────▼─────────────────────────────▼──────┐
    │         Shared State Variables               │
    │  • overlayProp (current overlay)            │
    │  • deckFocusProp / deckPairProp            │
    │  • shiftProp (SHIFT pressed)               │
    │  • setupPage / custom toggles              │
    │  • browserModeProp                         │
    │  • deviceStandardMode                      │
    └────────┬─────────────────────────────────────┘
             │
    ┌────────▼──────────────────────────────────────┐
    │    Enabled by Other Features:                │
    │  ✓ Tempo Control (uses SHIFT + timers)      │
    │  ✓ Beatgrid Control (uses SHIFT)            │
    │  ✓ Browser Mode (uses SHIFT + setup)        │
    │  ✓ Mixer Overlay (uses overlay state)       │
    │  ✓ Stem/Remix (uses setup + overlay)        │
    │  ✓ Transport LED (uses device state)        │
    └────────────────────────────────────────────┘
```

---

## Customization Points

### Available via Setup

**Setup Page 1 (Mixer):**

- L1-L4: Custom EQ/Kill assignments
- R1-R3: Volume/Gain/MixerFX assignments

**Setup Page 2 (Browser & Display):**

- L1: Fullscreen browser on browse
- L2: Browser Mode on Shift Tap
- L3: Minimize browser after load
- L4: Deck Switch on single-click MODE

**Setup Page 3 (Misc & Effects):**

- L1: Single Cue Monitor
- L2: Switch Mute/FX Send
- L3: CueAndPlay mode
- L4: Invert MixerFX LED
- R1: Custom Overmapping support
- R2: FX button behavior
- R3: Link FX Units to Decks
- R4: Block Secondary FX Overlay

### Not Customizable (Fixed by Design)

- MODE/SHIFT button behavior
- Overlay cycling order (FX1 → FX2 → Mixer)
- Deck pair cycling algorithm
- Setup page count/order

---

## Compatibility

### Requires (Prerequisites)

- **X1 MK3 Hardware** — Controller must be hardware X1 MK3
- **Traktor Pro 4.4.1+** — Core API required
- **QML Support** — All features in CSI/X1MK3/X1MK3.qml

### Works Alongside

- All X1MK3 features ([Tempo Control](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_tempo-control.md), [Beatgrid](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_beatgrid-control.md), [Mixer](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_mixer-overlay.md), etc.)
  - Infrastructure provides shared state for all features
  - No conflicts when all features in same mod

### Conflicts With

- **Custom X1 Overlays** — Non-standard overlay definitions
- **Different X1 Mods** — Cannot load multiple X1 performance mods simultaneously

### Tested On

- X1 MK3 with Traktor Pro 4.4.1
- X1 MK3 with Traktor Pro 4.5+

### NOT Tested On

- X1 MK2, S1 MK3, other controllers (X1-specific)
- Multi-controller setups (untested but likely compatible with D2, S4, etc.)

---

## Notes

### Design Decisions

1. **Centralized State** - All features read from shared state variables (prevents conflicts)
2. **Timers for Tap vs. Hold** - Distinguishes single-tap (overlay switch) from hold (setup)
3. **3-Page Setup System** - Balances customization depth with UI clarity
4. **Setup Mode Isolation** - Controls must be disabled during setup to avoid accidents
5. **Screen Feedback Layer** - Unified mechanism for all overlays to display status

### Why No Git History?

This mod was distributed as a ZIP archive from the community forum, not as a git repository. The source code cannot be committed to git with history preserved, so documentation is based on:

- Forum post feature descriptions
- Actual behavior testing
- QML file structure analysis

### Integration with Other Mods

When layering X1 with other mods (S4, S8, D2, etc.):

- X1 infrastructure properties should not conflict with other controllers
- Setup system is X1-specific (safe for multi-controller setups)
- SHIFT/MODE button behavior isolated to X1MK3 surface definition
- Overlay state tracked separately per device

---

## Related Features

**Next Steps:** With infrastructure in place, the following features can be documented:

1. **X1_tempo-control.md** - Sync button tap/hold + encoder modifiers
2. **X1_beatgrid-control.md** - Grid marker / beat tap
3. **X1_deck-features.md** - Display options, gridlock, vinyl break
4. **X1_browser-mode.md** - Browser encoder functions (depends on setup L2)
5. **X1_mixer-overlay.md** - Custom knob assignments (depends on setup page 1)
6. **X1_stem-remix-controls.md** - Superknob, quantization (depends on setup)
7. **X1_transport-led.md** - CDJ-style LED feedback
8. **X1_fx-section.md** - FX control enhancements

---

## Support & References

- **Source:** [X1MK3 Community Performance MOD (qml coding)](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)
- **Creator:** [@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh)
- **Related:** X1MK3_PerformanceMod_12 (ZIP archive)
