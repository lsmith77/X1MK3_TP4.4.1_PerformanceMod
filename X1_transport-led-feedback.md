# X1MK3 Transport LED Feedback (CDJ-Style Colors & Blinkers)

**Source:** X1MK3 Performance Mod v12 (Community Forum)  
**Note:** Feature distributed as QML code; no git history

---

## See Also

- **Foundation:** [X1 Infrastructure](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_infrastructure.md) (LED system foundation)
- **Related:** [Tempo Control](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_tempo-control.md) (sync LED states), [Deck Features](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_deck-features.md)
- **Complementary:** [Screen Feedback](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_screen-feedback.md) (visual feedback)

Enhanced LED feedback for Play, Cue, Sync, and Hotcue buttons using CDJ-style colors and blinking cue point indicators. Each button reflects deck state with context-aware colors, making it easy to see deck status at a glance in dark clubs.

## Features

- **Play Button:** Green (default) with CDJ-style CuePoint blinker
- **Play (Beat Tap):** White LED when beat-tap active
- **Cue Button:** Yellow (default) with CDJ-style CuePoint blinker
- **Sync Button:** Cyan (default) with bright indication
- **Sync (Master Mode):** Red LED when global clock is master
- **Hotcue (Loaded):** Yellow LED showing cue point available
- **Hotcue (Fade-In):** Dark Orange/Amber LED for fade-in cue type
- **Hotcue (Fade-Out):** Red LED for fade-out cue type
- **CuePoint Blinker:** Synchronized blinking when at cue point (Pioneer CDJ style)
- **Multi-Color Feedback:** Different colors for different states/modes
- **Brightness Control:** On/dim states based on current condition
- **Sync Master Indication:** Red LED clearly shows when global clock is master

## Modified Files

- `CSI/X1MK3/X1MK3TransportButtons.qml` - Transport button LED wiring
- `CSI/X1MK3/X1MK3HotcueButtons.qml` - Hotcue button LED colors

**Note:** Without git history, exact line numbers cannot be provided.

---

## Changes in `X1MK3TransportButtons.qml`

### Play Button LED (Green + Blinker)

**Description:** Green baseline LED; blinks when at cue point (CDJ-style).

**Implementation Pattern:**

```qml
// Play Button LED: Green with CuePoint blinker
Wire {
  from: "surface.play"
  to: LED {
    id: playLED
    color: Color.Green
    brightness: {
      if (isPlayingProp.value) {
        // Blinking at cue point (CDJ style)
        return cuePointBlinkerProp.value ? 0.5 : 1.0  // Alternate bright/dim
      } else {
        return 0.3  // Dim when stopped
      }
    }
  }
}

AppProperty {
  id: isPlayingProp
  path: "app.traktor.decks." + deckId + ".play"
}

AppProperty {
  id: cuePointBlinkerProp
  path: "app.traktor.decks." + deckId + ".cue_point_blinker"
}

Timer {
  id: cuePointBlinker
  interval: 300  // 300ms blink rate (Pioneer CDJ timing)
  running: isAtCuePoint == true
}
```

### Cue Button LED (Yellow + Blinker)

**Description:** Yellow LED; blinks when at cue point.

**Implementation Pattern:**

```qml
// Cue Button LED: Yellow with CuePoint blinker
Wire {
  from: "surface.cue"
  to: LED {
    id: cueLED
    color: Color.Yellow
    brightness: {
      if (isAtCuePointProp.value) {
        // Blinking at cue point
        return cuePointBlinkerProp.value ? 0.5 : 1.0
      } else {
        return 0.3  // Dim when not at cue
      }
    }
  }
}

AppProperty {
  id: isAtCuePointProp
  path: "app.traktor.decks." + deckId + ".is_at_cue_point"
}
```

### Sync Button LED (Cyan / Red Master)

**Description:** Cyan LED normally; turns Red when global clock is master tempo.

**Implementation Pattern:**

```qml
// Sync Button LED: Cyan (default) / Red (Master)
Wire {
  from: "surface.sync"
  to: LED {
    id: syncLED
    color: {
      if (isGlobalClockMasterProp.value) {
        return Color.Red  // Red when global clock is master
      } else if (isSyncedProp.value) {
        return Color.Cyan  // Cyan when synced to slave
      } else {
        return Color.Cyan  // Cyan baseline
      }
    }
    brightness: {
      if (isSyncedProp.value) {
        return 1.0  // Full brightness when synced
      } else {
        return 0.3  // Dim when not synced
      }
    }
  }
}

AppProperty {
  id: isSyncedProp
  path: "app.traktor.decks." + deckId + ".sync"
}

AppProperty {
  id: isGlobalClockMasterProp
  path: "app.traktor.globals.clock_is_master_tempo"
}
```

---

## Changes in `X1MK3HotcueButtons.qml`

### Hotcue LED Colors (Loaded / Fade-In / Fade-Out)

**Description:** Different colors for cue type; color indicates function.

**Implementation Pattern:**

```qml
// Hotcue Button LEDs: Color indicates cue type
Wire {
  from: "surface.hotcue1"  // Hotcue slot 1
  to: LED {
    id: hotcue1LED
    color: {
      var cueType = hotcue1TypeProp.value
      if (cueType == "fadeIn") {
        return Color.DarkOrange  // Fade-in: orange/amber
      } else if (cueType == "fadeOut") {
        return Color.Red  // Fade-out: red
      } else {
        return Color.Yellow  // Loaded/default: yellow
      }
    }
    brightness: {
      if (hotcue1LoadedProp.value) {
        return 1.0  // Full brightness if loaded
      } else {
        return 0.15  // Dim if not loaded
      }
    }
  }
}

AppProperty {
  id: hotcue1LoadedProp
  path: "app.traktor.decks." + deckId + ".cue_hotcues.1.is_loaded"
}

AppProperty {
  id: hotcue1TypeProp
  path: "app.traktor.decks." + deckId + ".cue_hotcues.1.type"  // "default", "fadeIn", "fadeOut"
}
// Repeat for hotcue2-8...
```

---

## Implementation Steps

1. **Identify transport button locations** (Play, Cue, Sync, Hotcue 1-8)
2. **Add color constant definitions** (Green, Yellow, Cyan, Red, DarkOrange, White)
3. **Add Play Button Wire** with:
   - Green color
   - Brightness toggle (full when playing, dim when stopped)
   - CuePoint blinker on/off (300ms interval)
4. **Add Cue Button Wire** with:
   - Yellow color
   - Brightness based on at-cue-point state
   - CuePoint blinker
5. **Add Sync Button Wire** with:
   - Cyan color (default)
   - Red color (when global clock is master)
   - Brightness (full when synced, dim when not)
6. **Add Hotcue Button Wires** for each of 8 hotcue slots with:
   - Color based on cue type (default=Yellow, fadeIn=Orange, fadeOut=Red)
   - Brightness (full if loaded, dim if empty)
7. **Add CuePointBlinker timer** (300ms interval, synchronized across all buttons)
8. **Test LED feedback** on all buttons in various states
9. **Verify colors** visible in dark/bright environments
10. **Test across all 4 decks**

---

## Testing Checklist

- [ ] **Play Button Color:** Check Play button LED:
  - [ ] Green when playing
  - [ ] Dim green when stopped
  - [ ] Green blinking at cue point (CDJ-style)

- [ ] **Play (Beat Tap):** Activate beat tap (Sync [Hold] + Play):
  - [ ] Play LED turns White while beat-tap active
  - [ ] Returns to Green when beat-tap done

- [ ] **Cue Button Color:** Check Cue button LED:
  - [ ] Yellow when not at cue point
  - [ ] Gets brighter when approaching cue
  - [ ] Blinks yellow when at exact cue point

- [ ] **Sync Button Color (Normal):** Load track without syncing:
  - [ ] Sync LED shows Cyan
  - [ ] Dim when not synced

- [ ] **Sync Button Color (Synced):** Click Sync to enable:
  - [ ] Sync LED bright Cyan
  - [ ] Indicates sync is active

- [ ] **Sync Button Color (Master Mode):** Enable global clock master (Setup):
  - [ ] Sync LED turns Red
  - [ ] Bright red when clock is master
  - [ ] Indicates global clock is driving tempo

- [ ] **Hotcue - Default (Yellow):** Load hotcue with default type:
  - [ ] Button LED bright Yellow
  - [ ] Indicates cue is loaded

- [ ] **Hotcue - Fade-In (Orange):** Load hotcue with fade-in type:
  - [ ] Button LED Dark Orange/Amber
  - [ ] Different color clearly shows custom cue type

- [ ] **Hotcue - Fade-Out (Red):** Load hotcue with fade-out type:
  - [ ] Button LED Red
  - [ ] Visually distinct from fade-in

- [ ] **Hotcue - Empty (Dim):** If hotcue slot is empty:
  - [ ] Button LED very dim
  - [ ] Clear indication no cue loaded

- [ ] **CuePoint Blinker Sync:** Check Play and Cue buttons:
  - [ ] Both blink in sync at 300ms interval
  - [ ] Blinking is smooth and synchronized

- [ ] **Multi-Deck LED:** Check all 4 decks:
  - [ ] Each deck shows independent LED states
  - [ ] No LED conflicts between decks
  - [ ] Colors visible from distance/dark club

---

## Dependencies

**Requires:**

- [X1 Infrastructure](X1_infrastructure.md) - Device state, LED capability
- [X1 Tempo Control](X1_tempo-control.md) - SYNC behavior
- [X1 Beatgrid Control](X1_beatgrid-control.md) - Beat tap integration
- Traktor 4.4.1 (exactly this version)
- X1MK3 hardware with RGB LED support

---

## Notes

### Design Philosophy

- **CDJ-Style Blinker:** Familiar to Pioneer equipment users
- **Context-Aware Colors:** Colors indicate state (Red=master, Yellow=cue loaded, etc.)
- **Brightness Variation:** Dim LEDs when inactive, bright when active
- **Sync Master Clarity:** Red is universally understood as "master" in DJ equipment

### User Experience

- LEDs provide at-a-glance deck status without looking at software
- Familiar CDJ blinker pattern for experienced DJs
- Color coding removes ambiguity (red always = master/critical state)
- Essential in dark club environments where screen visibility is poor

### Traktor Behavior Notes

- CuePoint blinker property available per deck
- Cue type affects hotcue appearance (default/fadeIn/fadeOut)
- Global clock master state overrides deck sync state for Sync LED
- LED colors don't affect audio; purely visual feedback

### Known Limitations

- LED colors depend on X1 RGB support (older hardware may have limited colors)
- Blinker timing (300ms) is fixed (not configurable)
- Hotcue8 button LED same as hotcue1-7 (cue type + loaded state)

---

## Related Features

- [X1 Infrastructure](X1_infrastructure.md) - Device setup, LED capability
- [X1 Tempo Control](X1_tempo-control.md) - Sync button behavior
- [X1 Beatgrid Control](X1_beatgrid-control.md) - Beat tap LED
- [X1 Deck Features](X1_deck-features.md) - Gridlock LED indicator

---

## Source & Resources

- **Forum:** [X1MK3 Community Performance MOD](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)
- **Creator:** [@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh)
- **Version:** 12 (Traktor 4.4.1)
