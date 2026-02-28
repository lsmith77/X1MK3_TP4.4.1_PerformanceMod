# X1MK3 Beatgrid Control (Manual Tap, Markers, Deletion)

**Source:** X1MK3 Performance Mod v12 (Community Forum)  
**Commit:** https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_beatgrid-control.md  
**Traktor:** 4.4.1

---

## What It Does

Intuitive beatgrid management using the SYNC button as a modifier. Tap the beat manually to detect tempo, place grid markers at specific positions, and delete unwanted markers—all without leaving the deck or opening Traktor's GUI.

---

## Features

- **Manual Beat Tap:** Sync [Hold] + Play → Activates beat detection; tap along to the beat
- **Set Grid Marker:** Sync [Hold] + Cue → Places a beatgrid marker at current playback position
- **Delete Grid Marker:** Sync [Hold] + REV (reverse button) → Removes the grid marker at current position
- **Grid Lock Protection:** Gridlock must be OFF to use beatgrid controls (prevents accidental modifications)
- **Visual Feedback:** Screen indicates when grid is locked vs. unlocked
- **Multi-Deck Support:** Works on all four decks independently

## Modified Files

- `CSI/X1MK3/X1MK3Deck.qml` - Beatgrid control wiring
- `Screens/X1MK3/DeckView.qml` - Grid lock status indicator

**Note:** Without git history, exact line numbers cannot be provided. Refer to `X1MK3Deck.qml` for implementation location.

---

## Configuration

### MOD Settings

Add this to your main X1MK3.qml controller file:

```qml
// --- MOD SETTINGS: Beatgrid Control ---
MappingPropertyDescriptor {
    id: beatgridControlEnabledProp
    path: "app.traktor.settings.beatgrid_control_enabled"
    initialValue: true  // Set to false to disable manual beatgrid controls
}
// --- END MOD SETTINGS ---
```

---

## See Also

- **Foundation:** [X1 Infrastructure](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_infrastructure.md) (SHIFT key handling)
- **Related:** [Tempo Control](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_tempo-control.md) (synchronized tempo/beats)
- **Feedback:** [Transport LED Feedback](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_transport-led-feedback.md), [Screen Feedback](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_screen-feedback.md)

---

## Changes in `X1MK3Deck.qml`

### Manual Beat Tap (Dynamic Tempo Detection)

**Description:** Sync held + Play activates beat detection engine. User taps along to the track to set tempo.

**Implementation Pattern:**

```qml
// Manual Beat Tap: Sync [Hold] + Play [Tap]
Wire {
  enabled: syncHoldProp == true && gridLocked == false
  from: "surface.play"
  to: ButtonScriptAdapter {
    onPress: {
      // Activate beat detection engine
      beatTapActive = true
      beatTapFeedback.visible = true
      // Traktor internally tracks tap intervals
    }
    onRelease: {
      // Finalize detected tempo
      beatTapActive = false
    }
  }
}

AppProperty {
  id: beatTapActive
  path: "app.traktor.decks." + deckId + ".detect_beats"
}
```

### Set Grid Marker

**Description:** Sync held + Cue places a beatgrid marker at current playback position.

**Implementation Pattern:**

```qml
// Set Grid Marker: Sync [Hold] + Cue
Wire {
  enabled: syncHoldProp == true && gridLocked == false
  from: "surface.cue"
  to: SetPropertyAdapter {
    path: "app.traktor.decks." + deckId + ".audio.analyze.set_beatgrid_marker"
    value: true  // Trigger marker placement at current position
  }
}
```

### Delete Grid Marker

**Description:** Sync held + REV button removes the grid marker at current position.

**Implementation Pattern:**

```qml
// Delete Grid Marker: Sync [Hold] + REV
Wire {
  enabled: syncHoldProp == true && gridLocked == false
  from: "surface.reverseButton"
  to: SetPropertyAdapter {
    path: "app.traktor.decks." + deckId + ".audio.analyze.delete_beatgrid_marker"
    value: true  // Remove marker at current position
  }
}
```

### Grid Lock Detection

**Description:** Check if gridlock is active before allowing beatgrid modifications.

**Implementation Pattern:**

```qml
// Grid Lock State
AppProperty {
  id: gridLockedProp
  path: "app.traktor.decks." + deckId + ".gridlock_enabled"
}

Wire {
  // Only enable beatgrid controls when grid is NOT locked
  enabled: gridLockedProp.value == false && syncHoldProp == true
  // ... beatgrid wiring
}
```

---

## Implementation Steps

1. **Identify Sync hold state** from infrastructure (syncHoldProp)
2. **Add gridlock detection** AppProperty for each deck
3. **Add Beat Tap Wire** on Play button when Sync held + grid unlocked:
   - Activates `detect_beats` property
   - Shows visual feedback on screen
4. **Add Set Grid Marker Wire** on Cue button when Sync held + grid unlocked:
   - Triggers `set_beatgrid_marker` property at current position
5. **Add Delete Grid Marker Wire** on REV button when Sync held + grid unlocked:
   - Triggers `delete_beatgrid_marker` property at current position
6. **Add screen feedback** showing grid lock status on deck view
7. **Verify all four decks** (A, B, C, D) can independently manage grids
8. **Test grid lock override** (controls disabled when grid is locked)
9. **Test beat tap UI** (visual feedback appears while tapping active)

---

## Testing Checklist

- [ ] **Grid Lock Status:** Load a track with beatgrid → verify grid lock toggle on screen
- [ ] **Manual Beat Tap:** Sync [Hold] + Play button → beat tap mode activates
  - [ ] Screen shows "Beat Tap Active" or similar indicator
  - [ ] Tapping along to track adjusts detected tempo
  - [ ] Release Play button to finalize
- [ ] **Set Grid Marker:** Sync [Hold] + Cue → places marker at current position
  - [ ] Verify on Deck A at different playback positions
  - [ ] Verify on Deck B
  - [ ] Verify marker appears in Traktor GUI grid display
  - [ ] Verify markers stick after Traktor restart
- [ ] **Delete Grid Marker:** Sync [Hold] + REV → removes marker at current position
  - [ ] Verify deletion works at different positions
  - [ ] Verify deleted markers are gone in Traktor GUI
  - [ ] Verify can undo via Traktor undo stack
- [ ] **Grid Lock Protection:** Enable grid lock on a deck → Sync [Hold] controls should be disabled
  - [ ] Beat Tap (Play) does nothing
  - [ ] Set Marker (Cue) does nothing
  - [ ] Delete Marker (REV) does nothing
  - [ ] Normal Play/Cue/REV functions still work
- [ ] **Multi-Deck:** Modify grid on Deck A, switch to Deck B → Deck B grid unaffected
- [ ] **Screen Feedback:** Grid lock status indicator appears/disappears on screen correctly
- [ ] **No Regressions:** Verify Play/Cue/REV buttons still work normally (non-Sync-held usage)

---

## Compatibility

### Requires (Prerequisites)

- **X1 Infrastructure** ([X1_infrastructure.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_infrastructure.md))
  - Sync hold state (syncHoldProp)
  - Must be installed first

- **Traktor Pro 4.4.1+** — Core API required
- **X1 MK3 Hardware** — SYNC, Cue, Play, REV controls required

### Works Alongside

- **Tempo Control** ([X1_tempo-control.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_tempo-control.md))
  - Both use Sync hold modifier
  - Complementary tempo/grid features
  - No conflict

- **Deck Features** ([X1_deck-features.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_deck-features.md))
  - Display options, gridlock toggle
  - Complementary

###Conflicts With

- **Grid Lock When Enabled** — Beatgrid controls disabled if grid locked (Traktor safety feature)

### Tested On

- X1 MK3 with Traktor Pro 4.4.1 (syncopated beats, fast detection)
- X1 MK3 with Traktor Pro 4.5+

### NOT Tested On

- X1 MK2, other controllers (X1-specific)
- Extremely unusual beat patterns (16/8, irregular)

---

## Notes## Notes

### Design Philosophy

- **Sync Hold as Modifier:** Reuses existing Sync hold mechanism to avoid button conflicts
- **Grid Lock Protection:** Prevents accidental grid modifications when grid is locked
- **Multi-Step Workflow:** Beat tap → set markers → delete as needed
- **Non-Destructive:** All grid changes tracked by Traktor undo stack

### User Experience

- Familiar to Serato users (beat tap is common in Serato)
- Can detect tempo and adjust grids without opening Traktor GUI
- Useful for tracks with syncopated beats where Traktor's auto-detection fails
- Workflow: Tap beat → optionally set markers → continue playing

### Traktor Behavior Notes

- Grid detection engine (`detect_beats`) activates when property set to true
- `set_beatgrid_marker` and `delete_beatgrid_marker` are snapshot operations (trigger and complete)
- Grid lock is per-deck independent; can lock A while C is unlocked
- All grid changes saved across Traktor sessions

### Known Limitations

- Cannot adjust existing marker positions (only add/delete)
- Beat tap requires at least 8-16 taps for reliable tempo detection
- Grid lock is a Traktor safety feature; cannot override programmatically

---

## Related Features

- [X1 Tempo Control](X1_tempo-control.md) - Complements grid adjustments with tempo fine-tuning
- [X1 Deck Features](X1_deck-features.md) - Grid lock toggle, display options
- [X1 Infrastructure](X1_infrastructure.md) - Sync hold state management
- [X1 Transport LED Feedback](X1_transport-led-feedback.md) - Visual feedback for Sync button

---

## Source & Resources

- **Forum:** [X1MK3 Community Performance MOD](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)
- **Creator:** [@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh)
- **Version:** 12 (Traktor 4.4.1)
