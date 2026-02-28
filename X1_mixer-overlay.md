# X1MK3 Mixer Overlay (Custom Knob & Button Assignments)

**Source:** X1MK3 Performance Mod v12 (Community Forum)  
**Note:** Feature distributed as QML code in ZIP archive; no git history

---

## See Also

- **Foundation:** [X1 Infrastructure](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_infrastructure.md) (overlay state management)
- **Related:** [Stem/Remix Controls](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_stem-remix-controls.md) (setup page 2), [FX Section](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_fx-section.md)
- **Advanced:** [Custom Overmapping Support](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_overmapping-support.md)
- **Feedback:** [Transport LED Feedback](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_transport-led-feedback.md), [Screen Feedback](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_screen-feedback.md)

Powerful mixer overlay customization system allowing full control over the 4 knobs and 8 buttons. Assign EQ (High/Mid/MidLow/Low), Volume, Gain, MixerFX (high-pass filter), Kill buttons, and Cue Monitor to any knob/button in any combination. Setup Page 1 provides intuitive color-coded interface for managing all assignments.

## Features

- **Fully Customizable Knobs:** Assign any parameter to any of 4 knobs
- **EQ Assignments:** High, Mid, MidLow, Low can be assigned independently or combined
- **Volume/Gain/MixerFX:** Assign to knobs with smart range adjustment when combined
- **Kill Buttons:** Assign kill functionality to any button
- **Cue Monitor:** Assign alongside Volume/Gain for deck monitoring
- **Multi-Parameter Knobs:** Single knob can control multiple parameters simultaneously
- **Smart Range Scaling:** Parameters adjust ranges when combined (e.g., EQ+Volume doesn't conflict)
- **Shift Layers:** Each parameter can have different behavior on shift layer (Shift [Hold])
- **Color Coding:** Visual feedback shows knob assignments (Blue/Yellow/Orange/Red)
- **Layer Indication:** Yellow (normal), Dark Orange (shift-layer), White (shared)
- **Setup Page 1:** 8 buttons (L1-R4) for all assignments with real-time color feedback
- **Block Mixer Option:** Can disable Mixer Overlay entirely (R4) to jump only between FX overlays

## Modified Files

- `CSI/X1MK3/X1MK3FXSection.qml` - Mixer knob/button assignments and wiring
- `CSI/X1MK3/X1MK3.qml` - Setup Page 1 state management

**Note:** Without git history, exact line numbers cannot be provided. Implementation in X1MK3FXSection.qml.

---

## Changes in `X1MK3FXSection.qml`

### Knob Assignment System

**Description:** Each of 4 knobs can be assigned to EQ, Volume, Gain, or MixerFX parameters.

**Implementation Pattern:**

```qml
// Knob 1 Assignment (Blue)
AppProperty {
  id: knob1Assignments
  path: mapping.propertiesPath + ".knob1_assignments"  // Bitmask: EQ_HIGH | EQ_MID | EQ_MIDLOW | EQ_LOW | VOLUME | GAIN | MIXER_FX
}

Wire {
  enabled: (knob1Assignments & VOLUME_FLAG) != 0
  from: "surface.fx.knobs.1"
  to: MappingPropertyDescriptor {
    path: "app.traktor.mixer.channels." + channelId + ".volume"
    min: 0.0
    max: 1.0
  }
}

Wire {
  enabled: (knob1Assignments & EQ_HIGH_FLAG) != 0
  from: "surface.fx.knobs.1"
  to: MappingPropertyDescriptor {
    path: "app.traktor.mixer.channels." + channelId + ".equalizer.high"
    min: 0.0
    max: 0.5  // Reduced range if combined with other params
  }
}
// Similar for Knob 2, 3, 4...
```

### Button Assignment System

**Description:** Each of 8 buttons can be assigned to Kill, Cue Monitor, or other functions.

**Implementation Pattern:**

```qml
// Button L1 Assignment
AppProperty {
  id: buttonL1Assignment
  path: mapping.propertiesPath + ".button_L1_assignment"  // KILL_HIGH | KILL_MID | etc.
}

Wire {
  enabled: (buttonL1Assignment & KILL_HIGH_FLAG) != 0
  from: "surface.effect_buttons.1"
  to: ButtonScriptAdapter {
    onPress: {
      eqHighKill.toggle()  // Toggle EQ High kill
    }
  }
}

Wire {
  enabled: (buttonL1Assignment & CUE_MONITOR_FLAG) != 0
  from: "surface.effect_buttons.1"
  to: ButtonScriptAdapter {
    onPress: {
      cueMonitor.toggle()  // Toggle cue monitoring
    }
  }
}
```

### Shift Layer Assignments

**Description:** Each parameter can have different shift-layer behavior.

**Implementation Pattern:**

```qml
// Parameter can have Shift-Layer assignment
AppProperty {
  id: knob1NormalLayer
  path: mapping.propertiesPath + ".knob1_normal_assignments"
}

AppProperty {
  id: knob1ShiftLayer
  path: mapping.propertiesPath + ".knob1_shift_assignments"
}

Wire {
  enabled: shiftProp == false && (knob1NormalLayer & VOLUME_FLAG) != 0
  from: "surface.fx.knobs.1"
  to: MappingPropertyDescriptor {
    path: "app.traktor.mixer.channels." + channelId + ".volume"
  }
}

Wire {
  enabled: shiftProp == true && (knob1ShiftLayer & MIXER_FX_FLAG) != 0
  from: "surface.fx.knobs.1"
  to: MappingPropertyDescriptor {
    path: "app.traktor.mixer.channels." + channelId + ".filter"  // Shift = filter
  }
}
```

### Smart Range Adjustment

**Description:** When multiple parameters assigned to same knob, ranges automatically scale.

**Implementation Pattern:**

```qml
function calculateKnobRange(parameter1, parameter2, parameter3, parameter4) {
  // Rules based on combination:
  // EQ + Volume: EQ at 0.0-0.5, Volume at 0.0-1.0, volume 100% at midpoint
  // EQ + MixerFX: EQ at 0.0-0.5, MixerFX at 0.0-1.0 (with cut)
  // Volume + Gain: Volume at 0.0-1.0, Gain full range, vol 100% at midpoint
  // etc.

  if (hasVolume && hasEQ) {
    return { volumeRange: [0, 1], eqRange: [0, 0.5], split: 0.5 }
  } else if (hasVolume && hasGain) {
    return { volumeRange: [0, 1], gainRange: [0, 1], split: 0.5 }
  } else {
    return { range: [0, 1] }  // Single parameter: full range
  }
}
```

---

## Implementation Steps

1. **Define parameter flags** (bitmask system for EQ_HIGH, EQ_MID, VOLUME, GAIN, MIXER_FX, etc.)
2. **Create knob assignment AppProperties** for Knob 1, 2, 3, 4 (normal and shift layers)
3. **Create button assignment AppProperties** for L1-L4, R1-R4 (normal and shift layers)
4. **Add conditional Wires** for each knob:
   - Each Wire checks if parameter assigned to this knob
   - Applies smart range scaling based on combination
5. **Add conditional Wires** for each button:
   - Kill button wiring when assigned
   - Cue Monitor wiring when assigned
6. **Add shift layer detection** to route parameters based on SHIFT state
7. **Implement smart range function** to calculate correct min/max for parameter combinations
8. **Add color coding** to Setup Page 1 buttons (Blue/Yellow/Orange/Red for knobs)
9. **Add layer feedback** on Setup display (White/Yellow/Dark Orange for layers)
10. **Test all combination scenarios** (EQ+Volume, EQ+Gain, Volume+Gain, EQ+MixerFX, etc.)

---

## Testing Checklist

- [ ] **Knob Assignments (Setup Page 1, L1-L4):** Each button toggles a knob assignment:
  - [ ] L1: EQ High
  - [ ] L2: EQ Mid
  - [ ] L3: EQ MidLow
  - [ ] L4: EQ Low
  - [ ] Verify color coding (all Blue for left side knob)

- [ ] **Button Assignments (Setup Page 1, R1-R4):** Right side buttons toggle button assignments:
  - [ ] R1: Volume (and Cue Monitor)
  - [ ] R2: Gain (and Cue Monitor)
  - [ ] R3: MixerFX (high-pass filter)
  - [ ] R4: Block Mixer Overlay (jumps only between FX overlays)

- [ ] **Single Parameter Assignment:** Assign Volume alone to Knob 1:
  - [ ] Knob 1 turn → Volume changes full range 0.0-1.0
  - [ ] Works on all four channels
  - [ ] 100% volume at maximum position

- [ ] **Dual Parameter Assignment (EQ + Volume):** Assign EQ High + Volume to Knob 1:
  - [ ] Left half (0.0-0.5) → EQ High (0.0-0.5 range)
  - [ ] Right half (0.5-1.0) → Volume (0.0-1.0, reaching 100% at center, full range to right)
  - [ ] No need to do both at center; split naturally

- [ ] **Dual Parameter (Volume + Gain):** Assign Volume + Gain to Knob 2:
  - [ ] Left half → Gain adjustment
  - [ ] Right half → Volume adjustment
  - [ ] Volume reaches 100% at midpoint

- [ ] **Triple Parameters:** Assign EQ High + EQ Mid + Volume:
  - [ ] All three parameters controllable from single knob
  - [ ] Ranges automatically scale to fit

- [ ] **Multiple EQs:** Assign all 4 EQs (High, Mid, MidLow, Low) to Knob 1:
  - [ ] All EQs controlled by single knob
  - [ ] Range is 0.0-1.0 (full boost) on all EQs

- [ ] **Kill Button Assignment:** Press button L1 (if assigned to Kill):
  - [ ] Toggles kill on assigned EQ band
  - [ ] LED feedback shows kill state (bright/dim)

- [ ] **Cue Monitor Assignment:** Press button R1 (if assigned to Cue):
  - [ ] Toggles cue monitoring for channel
  - [ ] LED shows monitor state

- [ ] **Shift Layer (Normal vs. Shift):** Assign Knob 1:
  - [ ] Normal: Volume control
  - [ ] Shift [Hold]: MixerFX (high-pass filter)
  - [ ] Verify both work; switching SHIFT switches function

- [ ] **Parameter Persistence:** Change assignments in Setup:
  - [ ] Assignments retained after Traktor restart
  - [ ] Assignments restored correctly

- [ ] **Block Mixer Overlay (R4):** Enable "Block Mixer" in Setup:
  - [ ] MODE single-tap no longer cycles to Mixer Overlay
  - [ ] Only cycles between FX1 → FX2 → FX1

- [ ] **No Regressions:** Standard FX Overlay still works:
  - [ ] Without Mixer Overlay active, FX knobs work normally
  - [ ] MODE cycling between FX1/FX2 unaffected

---

## Dependencies

**Requires:**

- [X1 Infrastructure](X1_infrastructure.md) - SHIFT state, Setup Page 1, overlay state
- Traktor 4.4.1 (exactly this version)
- X1MK3 hardware

**Optional:**

- Works alongside [X1 Stem/Remix Controls](X1_stem-remix-controls.md) (Superknob similar pattern)

---

## Notes

### Design Philosophy

- **Flexible Assignment System:** Bitmask allows unlimited combinations
- **Smart Range Scaling:** Parameters adjust automatically to avoid conflicts
- **Visual Feedback:** Color coding makes assignments obvious in Setup
- **Shift Awareness:** Different behaviors per layer for advanced users
- **Non-Destructive:** All assignments customizable without modifying code

### User Experience

- Setup Page 1 provides intuitive visual interface
- Color coding (Blue/Yellow/Orange/Red) helps identify knob assignments
- Smart range prevents clipped parameters when combined
- Can toggle assignments on/off without removing them permanently

### Traktor Behavior Notes

- MixerFX is high-pass filter (6 dB/octave slope)
- Kill buttons cut the signal completely (0 dB at minimum)
- Cue Monitor enables pre-fader listening for setup
- Single Cue Monitor mode (Setup Option L1) prevents multiple simultaneous monitors
- EQ ranges 0.0-1.0 (0.0=cut, 1.0=boost)

### Parameter Combination Rules

- **EQ + Volume:** Shows split at 0.5 point (left=EQ, right=Volume peaking at center)
- **EQ + Gain/MixerFX:** EQ limited to 0.0-0.5 (no boost); other param full range
- **Volume + Gain:** Split at 0.5; Volume 100% at center, Gain continues to right
- **All EQs:** Range extends to full 0.0-1.0 (all boost)

---

## Related Features

- [X1 Infrastructure](X1_infrastructure.md) - SHIFT state, Setup configuration
- [X1 Stem/Remix Controls](X1_stem-remix-controls.md) - Superknob (similar parameter control)
- [X1 FX Section](X1_fx-section.md) - Overlay integration
- [X1 Transport LED Feedback](X1_transport-led-feedback.md) - Button LED states

---

## Source & Resources

- **Forum:** [X1MK3 Community Performance MOD](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)
- **Creator:** [@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh)
- **Version:** 12 (Traktor 4.4.1)
