# X1MK3 Stem/Remix Controls (Superknobs, Quantization, Beat Counter)

**Source:** X1MK3 Performance Mod v12 (Community Forum)  
**Note:** Feature distributed as QML code; no git history

---

## See Also

- **Foundation:** [X1 Infrastructure](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_infrastructure.md) (SHIFT key, overlay management)
- **Related:** [Mixer Overlay](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_mixer-overlay.md) (setup page 2, superknob wiring)
- **Feedback:** [Transport LED Feedback](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_transport-led-feedback.md), [Screen Feedback](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_screen-feedback.md)

Advanced stem and remix deck controls combining Superknob technology (dual-parameter knobs) with capture source management and quantization. FX knobs in Mixer Overlay control stem/remix channel volume and high-pass filter simultaneously. Shift [Hold] + Loop encoder manages capture source and quantization with screen feedback.

## Features

- **Superknob Controls:** FX knobs turn left for volume, right for high-pass filter simultaneously
- **Stem Superknob:** Turn left = Volume, Turn right = Filter HPF (4-knob system per deck)
- **Visual Feedback:** Screen displays current parameter being adjusted
- **Capture Source Selection:** Shift [Hold] + Loop [Turn] → Cycle through available capture sources
- **Quantization Toggle:** Shift [Hold] + Loop [Push] → Toggle quantization on/off
- **Quantization Adjustment:** Shift [Hold] + Loop [Press] + Loop [Turn] → Fine-tune quantization value
- **Screen Feedback:** Display shows current capture source and quantization setting
- **Remix Beat Counter:** Replaces time display with beat counter (bars.beats in remix deck)
- **Mute/FX Send Buttons:** Stem/remix buttons toggle mute (normal) or FX Send (shift, optional per Setup L2)
- **Layer-Aware Control:** All features respect shift-layer switching

## Modified Files

- `CSI/X1MK3/X1MK3FXSection.qml` - Superknob wiring, stem/remix controls
- `CSI/X1MK3/X1MK3Deck.qml` - Capture/quantization controls
- `Screens/X1MK3/DeckView.qml` - Beat counter, capture source feedback

**Note:** Without git history, exact line numbers cannot be provided.

---

## Changes in `X1MK3FXSection.qml`

### Superknob Implementation (Dual-Parameter Knobs)

**Description:** Single knob controls two parameters: turn left for volume, right for HPF filter.

**Implementation Pattern:**

```qml
// Superknob: FX Knob turns left for volume, right for HPF
Wire {
  from: "surface.fx.knobs.1"
  to: MappingPropertyAdapter {
    // Split control: left = volume (0.0-0.5), right = filter (0.5-1.0)
    onValueChanged: {
      var position = value  // 0.0-1.0
      if (position < 0.5) {
        // Left turn: adjust stem volume
        volumeProp.value = position * 2  // Scale 0.0-0.5 to 0.0-1.0
      } else {
        // Right turn: adjust HPF filter
        filterProp.value = (position - 0.5) * 2  // Scale 0.5-1.0 to 0.0-1.0
      }
    }
  }
}

AppProperty {
  id: volumeProp
  path: "app.traktor.decks." + deckId + ".stems." + stemSlot + ".volume"
}

AppProperty {
  id: filterProp
  path: "app.traktor.decks." + deckId + ".stems." + stemSlot + ".filter"
}
```

### Stem/Remix Mute & FX Send Buttons

**Description:** Buttons toggle mute by default; Shift [Hold] toggles FX Send instead (optional per Setup L2).

**Implementation Pattern:**

```qml
// Stem Mute / FX Send: Button behavior based on Setup L2
Wire {
  from: "surface.effect_buttons.1"  // One of 4 stem buttons
  to: ButtonScriptAdapter {
    onPress: {
      if (shiftProp == false || switchMuteFXSend == false) {
        // Normal: toggle mute
        muteProp.value = !muteProp.value
      } else {
        // Shift [Hold]: toggle FX send (if Setup L2 enabled)
        fxSendProp.value = !fxSendProp.value
      }
    }
  }
}

AppProperty {
  id: muteProp
  path: "app.traktor.decks." + deckId + ".stems." + stemSlot + ".muted"
}

AppProperty {
  id: fxSendProp
  path: "app.traktor.decks." + deckId + ".stems." + stemSlot + ".fx_send_on"
}
```

---

## Changes in `X1MK3Deck.qml`

### Capture Source Selection

**Description:** Shift [Hold] + Loop [Turn] cycles through available capture sources for remix decks.

**Implementation Pattern:**

```qml
// Capture Source: Shift [Hold] + Loop [Turn]
Wire {
  enabled: shiftProp == true && isDeckRemix == true
  from: "surface.loop.is_turned"
  to: ButtonScriptAdapter {
    onPress: {
      captureSrcIndex = (captureSrcIndex + 1) % numCaptureSources
      captureSourceProp.value = captureSrcIndex
      screenFeedback.show("Capture: " + captureSrcNames[captureSrcIndex])
    }
  }
}

AppProperty {
  id: captureSourceProp
  path: "app.traktor.decks." + deckId + ".remix.capture_source"
}
```

### Quantization Control

**Description:** Shift [Hold] + Loop [Push] toggles; Shift [Hold] + Loop [Hold] + Loop [Turn] adjusts.

**Implementation Pattern:**

```qml
// Quantization Toggle: Shift [Hold] + Loop [Push]
Wire {
  enabled: shiftProp == true
  from: "surface.loop.is_pressed"
  to: ButtonScriptAdapter {
    onPress: {
      quantizationOn.value = !quantizationOn.value
      screenFeedback.show("Quantize: " + (quantizationOn.value ? "ON" : "OFF"))
    }
  }
}

// Quantization Adjust: Shift [Hold] + Loop [Hold] + Loop [Turn]
Wire {
  enabled: shiftProp == true && loopHeld == true
  from: "surface.loop.is_turned"
  to: MappingPropertyDescriptor {
    path: "app.traktor.decks." + deckId + ".remix.quantization_value"
    step: 0.1  // Fine adjustment
  }
}

AppProperty {
  id: quantizationOn
  path: "app.traktor.decks." + deckId + ".remix.quantization_on"
}
```

### Remix Beat Counter

**Description:** Display beat counter instead of time in remix deck (bars.beats format).

**Implementation Pattern:**

```qml
// Remix Beat Counter
AppProperty {
  id: beatCounterProp
  path: "app.traktor.decks." + deckId + ".remix.beat_counter"
}

ScreenDisplay {
  id: remixBeatDisplay
  text: isDeckRemix == true ?
    Math.floor(beatCounterProp.value / phraseLengthProp.value) + "." +
    (beatCounterProp.value % phraseLengthProp.value) :
    "-- "
}
```

---

## Implementation Steps

1. **Add Superknob AppProperties** for volume and HPF filter for each stem slot
2. **Create Superknob Wire** on FX knob that:
   - Splits 0.0-1.0 range at 0.5 midpoint
   - Left half (0.0-0.5) controls volume
   - Right half (0.5-1.0) controls HPF filter
3. **Add stem button detection** to identify stem deck vs. remix deck
4. **Add Mute/FX Send Wires** on buttons with shift-layer detection
5. **Add switchMuteFXSend detection** from Setup Page 3 (L2)
6. **Add capture source cycling** on Shift [Hold] + Loop [Turn]
7. **Add quantization toggle** on Shift [Hold] + Loop [Push]
8. **Add quantization adjustment** on Shift [Hold] + Loop [Hold] + Loop [Turn]
9. **Add beat counter AppProperty** for remix decks
10. **Add screen feedback** for capture source, quantization, beat counter
11. **Test all stem slots** (Stem 1, 2, 3, 4) on remix decks

---

## Testing Checklist

- [ ] **Superknob - Volume:** FX Knob 1 turn left:
  - [ ] Volume increases smoothly from 0% to 100%
  - [ ] Works on all 4 stem channels
  - [ ] Screen shows "Volume: XX%"

- [ ] **Superknob - Filter:** FX Knob 1 turn right:
  - [ ] High-pass filter frequency increases
  - [ ] Works on all 4 stem channels
  - [ ] Screen shows "Filter: XX Hz"

- [ ] **Stem Mute (Normal):** Press stem button (e.g., button upon L1):
  - [ ] Toggles stem mute on/off
  - [ ] Button LED shows mute state (bright/dim)
  - [ ] Audio from stem muted/unmuted

- [ ] **Stem FX Send (Shift [Hold]):** Shift [Hold] + press button:
  - [ ] If Setup L2 enabled: toggles FX Send
  - [ ] If Setup L2 disabled: still toggles mute (no shift behavior)
  - [ ] Works on all 4 stem channels

- [ ] **Capture Source Selection:** Shift [Hold] + Loop [Turn]:
  - [ ] Cycles through available sources (Deck A, Deck B, Deck C, Deck D, or similar)
  - [ ] Only works on remix deck
  - [ ] Screen shows "Capture: [Source Name]"

- [ ] **Quantization Toggle:** Shift [Hold] + Loop [Push]:
  - [ ] Toggles quantization on/off
  - [ ] Screen shows "Quantize: ON" or "Quantize: OFF"
  - [ ] On = quantize cells; Off = free-form

- [ ] **Quantization Adjust:** Shift [Hold] + Loop [Hold 1+ sec] + Loop [Turn]:
  - [ ] Quantization value adjusts ±0.1 per click
  - [ ] Works while capturing
  - [ ] Screen shows current quantization value

- [ ] **Remix Beat Counter:** Load remix deck + set display to beats:
  - [ ] Time display replaced with beat counter
  - [ ] Format: Bars.Beats (e.g., "3.2" = bar 3, beat 2)
  - [ ] Counts up continuously during playback

- [ ] **Remix Beat Counter - Phrase Length:** Adjust phrase length in Setup:
  - [ ] Phrase length = 1: beats displayed individually
  - [ ] Phrase length = 4: bars display as groups of 4
  - [ ] Phrase length affects display format only

- [ ] **Multi-Stem Setup:** Load track with stems on Deck A:
  - [ ] All 4 Superknobs work independently
  - [ ] Changing stem 1 volume doesn't affect stem 2-4
  - [ ] Filter adjustments isolated per stem

- [ ] **No Regressions:** Normal (non-remix) deck behavior:
  - [ ] Shift [Hold] + Loop controls standard features (gridlock, etc.)
  - [ ] FX knobs work normally when not in Mixer Overlay

---

## Dependencies

**Requires:**

- [X1 Infrastructure](X1_infrastructure.md) - SHIFT state, Setup configuration
- [X1 Mixer Overlay](X1_mixer-overlay.md) - FX knobs context
- Traktor 4.4.1 with stem/remix support
- X1MK3 hardware

**Optional:**

- Works with [X1 Stem-specific Features](X1_deck-features.md) (beat counter display)

---

## Notes

### Design Philosophy

- **Superknob Efficiency:** Single knob controls two related parameters (volume + filter)
- **Shift-Layer Awareness:** Different behaviors for normal vs. shift
- **Screen Feedback:** Real-time parameter display guides user adjustments
- **Capture Source Cycling:** Quick selection without opening remix panel

### User Experience

- Superknob reduces button presses for common stem adjustments
- Can adjust volume while monitoring filter response
- Quantization control enables precise timing of sample captures
- Beat counter provides visual reference for remix timing

### Traktor Behavior Notes

- Stem volume range 0.0-1.0 (0% to 100%)
- HPF filter range typically 20 Hz to 20,000 Hz
- Quantization values: 0.25, 0.5, 1.0, 2.0, 4.0 bars (configurable)
- Capture source available only on remix decks
- Beat counter updates at 60 FPS for smooth real-time display

---

## Related Features

- [X1 Mixer Overlay](X1_mixer-overlay.md) - Superknob knob integration
- [X1 Deck Features](X1_deck-features.md) - Beat counter display
- [X1 Infrastructure](X1_infrastructure.md) - SHIFT state, Setup Page 3 (L2)

---

## Source & Resources

- **Forum:** [X1MK3 Community Performance MOD](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)
- **Creator:** [@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh)
- **Version:** 12 (Traktor 4.4.1)
