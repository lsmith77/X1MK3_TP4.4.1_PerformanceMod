# X1MK3 FX Section (Effects Control & Assignment)

**Source:** X1MK3 Performance Mod v12 (Community Forum)  
**Note:** Feature distributed as QML code; no git history

---

## See Also

- **Foundation:** [X1 Infrastructure](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_infrastructure.md) (MODE button, overlay switching)
- **Related:** [Mixer Overlay](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_mixer-overlay.md) (knob/button integration)
- **Advanced:** [Custom Overmapping Support](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_overmapping-support.md) (optional overlay-specific mapping)
- **Feedback:** [Transport LED Feedback](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_transport-led-feedback.md), [Screen Feedback](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_screen-feedback.md)

Enhanced FX section control for managing effects units and assignments. FX knobs and buttons provide intuitive effect parameter control. FX assignment arrows (→/←) intelligently manage FX unit assignment with optional overlay modifier support.

## Features

- **FX Knobs (Primary):** Control effect parameters (Filter, Delay time, Reverb decay, etc.)
- **FX Buttons (Toggles):** Enable/disable effects with visual LED feedback
- **FX Assignment Arrows (→/←):** Assign FX units to decks or focus on FX unit below (Setup R2)
- **Deck-Based Assignment:** Left arrow assigns to Deck A, Right arrow to Deck B (normal)
- **FX Focus Mode (Setup R2):** Arrows focus on FX unit (→ selects FX1, ← selects FX2, etc.)
- **Color Coding:** Light Orange (default FX) with state-based brightness
- **LED Feedback:** Button brightness shows FX on/off state
- **Overlay Integration:** FX keeps consistency across FX1, FX2, and Mixer overlays
- **Custom Overmapping:** Optional integration with overlay-specific control mapping

## Modified Files

- `CSI/X1MK3/X1MK3FXSection.qml` - FX knob/button wiring, assignments
- `CSI/X1MK3/X1MK3.qml` - FX overlay state management

**Note:** Without git history, exact line numbers cannot be provided.

---

## Changes in `X1MK3FXSection.qml`

### FX Knob Control

**Description:** FX knobs adjust effect parameters (knob 1-4 map to FX 1-4).

**Implementation Pattern:**

```qml
// FX Knob 1: Control FX Unit 1 parameters
Wire {
  from: "surface.fx.knobs.1"
  to: MappingPropertyDescriptor {
    path: "app.traktor.mixer.fx_units.1.parameter_value"
    min: 0.0
    max: 1.0
  }
}

AppProperty {
  id: fxParam1Prop
  path: "app.traktor.mixer.fx_units.1.parameter_value"
}

ScreenDisplay {
  text: "FX1: " + Math.round(fxParam1Prop.value * 100) + "%"
}
// Repeat for FX Knob 2, 3, 4...
```

### FX Enable/Disable Buttons

**Description:** FX buttons toggle on/off state with brightness feedback.

**Implementation Pattern:**

```qml
// FX Button 1: Toggle FX Unit 1 on/off
Wire {
  from: "surface.effect_buttons.1"
  to: ButtonScriptAdapter {
    onPress: {
      fxOnOffProp1.toggle()
    }
  }
}

Wire {
  from: "surface.effect_buttons.1"
  to: LED {
    id: fxButton1LED
    color: Color.LightOrange
    brightness: fxOnOffProp1.value ? 1.0 : 0.3  // Bright on, dim off
  }
}

AppProperty {
  id: fxOnOffProp1
  path: "app.traktor.mixer.fx_units.1.on"
}
// Repeat for FX Button 2, 3, 4...
```

### FX Assignment Arrows

**Description:** Assignment arrows (→/←) manage FX unit assignments with two modes.

**Implementation Pattern:**

```qml
// FX Assignment Mode 1: Assign to Deck
Wire {
  enabled: fxAssignmentMode == MODE_DECK  // Setup R2 disabled
  from: "surface.fx_arrow_right"
  to: ButtonScriptAdapter {
    onPress: {
      // Right arrow: assign FX to Deck B
      fxAssignProp.value = "deck_B"
      screenFeedback.show("FX: Deck B")
    }
  }
}

// FX Assignment Mode 2: Focus on FX Unit (Setup R2)
Wire {
  enabled: fxAssignmentMode == MODE_FX_FOCUS  // Setup R2 enabled
  from: "surface.fx_arrow_right"
  to: ButtonScriptAdapter {
    onPress: {
      // Right arrow: focus on FX Unit 1
      selectedFXUnit.value = 1
      screenFeedback.show("FX Unit: 1")
    }
  }
}

AppProperty {
  id: fxAssignProp
  path: "app.traktor.mixer.current_fx_assignment"
}

AppProperty {
  id: selectedFXUnit
  path: mapping.propertiesPath + ".selected_fx_unit"
}
```

---

## Implementation Steps

1. **Map FX knobs** to FX unit parameter values
2. **Add FX button Wires** for enable/disable on all 4 buttons
3. **Add LED feedback** for FX buttons showing on/off state (bright/dim)
4. **Add FX assignment arrows** with dual-mode support:
   - Mode 1 (default): Assign to deck pairs (→=B, ←=A, Shift+→=D, Shift+←=C)
   - Mode 2 (Setup R2): Focus on FX units (→=FX1, ←=FX2, Shift+→=FX3, Shift+←=FX4)
5. **Add screen feedback** showing current FX assignments
6. **Add FX parameter display** on screens (percentage/value)
7. **Test FX assignment** across all deck combinations
8. **Verify no conflicts** with Mixer Overlay FX parameter control

---

## Testing Checklist

- [ ] **FX Knob 1 Control:** Turn FX Knob 1:
  - [ ] FX Unit 1 parameter adjusts (filter, delay time, etc.)
  - [ ] Parameter value displayed on screen (0-100%)
  - [ ] Works smoothly across full range

- [ ] **FX Button 1 Toggle:** Press FX Button 1:
  - [ ] FX Unit 1 toggles on/off
  - [ ] LED bright when on, dim when off
  - [ ] Audio affected (FX applied/removed)

- [ ] **FX Assignment Arrows (Deck Mode):** Press Right arrow:
  - [ ] FX assignment to Deck B confirmed on screen
  - [ ] Left arrow → Deck A
  - [ ] Shift [Hold] + Right → Deck D
  - [ ] Shift [Hold] + Left → Deck C

- [ ] **FX Assignment Arrows (Focus Mode - Setup R2):** Enable R2 option:
  - [ ] Right arrow → FX Unit 1
  - [ ] Left arrow → FX Unit 2
  - [ ] Shift [Hold] + Right → FX Unit 3
  - [ ] Shift [Hold] + Left → FX Unit 4

- [ ] **LED Color Consistency:** Check all FX buttons:
  - [ ] All LEDs Light Orange color
  - [ ] Brightness varies with on/off state
  - [ ] No color conflicts with transport buttons

- [ ] **Overlay Switching:** Cycle through FX1 → FX2 → Mixer overlays:
  - [ ] FX knobs/buttons maintain consistency
  - [ ] Assignment arrows work in all overlays
  - [ ] Parameter values persist across overlays

- [ ] **No Regressions:** Standard FX control still works:
  - [ ] FX enable/disable functions
  - [ ] FX parameter adjustment smooth
  - [ ] Deck to FX assignment logical

---

## Dependencies

**Requires:**

- [X1 Infrastructure](X1_infrastructure.md) - Overlay state, SHIFT state
- [X1 Mixer Overlay](X1_mixer-overlay.md) - Overlay integration
- Traktor 4.4.1 (exactly this version)
- X1MK3 hardware

**Optional:**

- [X1 Stem/Remix Controls](X1_stem-remix-controls.md) - FX relevant to stem processing
- [X1 Overmapping Support](X1_overmapping-support.md) - Custom modifier conditions

---

## Notes

### Design Philosophy

- **Knob-to-FX Mapping:** Knob 1 = FX1, Knob 2 = FX2, etc. (intuitive mapping)
- **Assignment Modes:** Two-mode system (Deck or FX Focus) serves different workflow styles
- **Visual Clarity:** Light Orange color distinct from Transport LEDs
- **Non-Destructive:** AssignmentsChangeable without losing FX state

### User Experience

- FX knobs provide direct parameter adjustment
- Assignment arrows handle unit selection without menu diving
- Two modes (Deck vs. Unit focus) accommodate different DJ styles
- LED feedback provides immediate on/off confirmation

### Traktor Behavior Notes

- FX parameter range is always 0.0-1.0 (normalized)
- FX enable/disable state independent per unit
- FX assignment affects where the FX unit applies (deck routing)
- FX Unit 1-4 available (can focus on any)

---

## Related Features

- [X1 Infrastructure](X1_infrastructure.md) - Overlay state management
- [X1 Mixer Overlay](X1_mixer-overlay.md) - Overlay integration
- [X1 Transport LED Feedback](X1_transport-led-feedback.md) - LED color scheme
- [X1 Overmapping Support](X1_overmapping-support.md) - Custom effects control

---

## Source & Resources

- **Forum:** [X1MK3 Community Performance MOD](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)
- **Creator:** [@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh)
- **Version:** 12 (Traktor 4.4.1)
