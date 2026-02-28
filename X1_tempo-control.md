# X1MK3 Tempo Control (Sync Hold + BPM Adjustment)

**Source:** X1MK3 Performance Mod v12 (Community Forum)  
**Commit:** https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_tempo-control.md  
**Traktor:** 4.4.1

---

## What It Does

Enhanced tempo control for precise BPM adjustment without accidentally toggling SYNC. Hold the SYNC button to unlock fine-grained tempo control via the Loop encoder (±0.01 BPM per step), with support for coarse adjustments (±1.00 BPM when Loop held), automatic clock master detection when global clock is tempo master, and screen feedback showing current BPM.

## Features

- **Sync [Tap]:** Toggles SYNC on/off (standard behavior)
- **Sync [Hold]:** Does NOT toggle SYNC; holds to check track BPM without altering it
- **Sync [Hold] + Loop [Turn]:** Fine tempo adjustment (±0.01 BPM per step)
- **Sync [Hold] + Loop [Hold] + Loop [Turn]:** Coarse tempo adjustment (±1.00 BPM per step)
- **Sync [Hold] + Loop [Push]:** Reset tempo to original track BPM
- **Clock Master Mode:** When global clock is tempo master, same controls adjust global BPM
- **Screen Feedback:** Current tempo displayed on deck screens

## Modified Files

- `CSI/X1MK3/X1MK3Deck.qml` - Tempo control wiring and encoder handlers
- `Screens/X1MK3/DeckView.qml` - BPM display feedback

**Note:** Without git history, exact line numbers cannot be provided. Refer to `X1MK3Deck.qml` for implementation location.

---

## Configuration

### MOD Settings

Add this to your main X1MK3.qml controller file:

```qml
// --- MOD SETTINGS: Tempo Control ---
MappingPropertyDescriptor {
    id: tempoControlEnabledProp
    path: "app.traktor.settings.tempo_control_enabled"
    initialValue: true  // Set to false to disable tempo hold/adjust
}
// --- END MOD SETTINGS ---
```

---

## See Also

- **Foundation:** [X1 Infrastructure](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_infrastructure.md) (SHIFT key handling, state management)
- **Related:** [Beatgrid Control](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_beatgrid-control.md) (beat position management)
- **Feedback:** [Transport LED Feedback](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_transport-led-feedback.md), [Screen Feedback](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_screen-feedback.md)

### Fine Tempo Adjustment (±0.01 BPM)

**Description:** Loop encoder turn adjusts tempo with fine granularity when SYNC is held.

**Implementation Pattern:**

```qml
// Fine BPM Control: Sync [Hold] + Loop [Turn]
Wire {
  enabled: shiftProp == false && syncHoldProp == true
  from: "surface.loop.is_turned"
  to: MappingPropertyDescriptor {
    path: "app.traktor.decks." + deckId + ".tempo"
    // Adjust by ±0.01 per step
  }
}
```

### Coarse Tempo Adjustment (±1.00 BPM)

**Description:** Loop encoder turn with Loop held adjusts tempo by ±1.00 BPM.

**Implementation Pattern:**

```qml
// Coarse BPM Control: Sync [Hold] + Loop [Hold] + Loop [Turn]
Wire {
  enabled: syncHoldProp == true && loopEncoderHeld == true
  from: "surface.loop.is_turned"
  to: MappingPropertyDescriptor {
    path: "app.traktor.decks." + deckId + ".tempo"
    // Adjust by ±1.00 per step
  }
}
```

### Tempo Reset

**Description:** Loop push resets tempo to original track BPM.

**Implementation Pattern:**

```qml
// Reset Tempo: Sync [Hold] + Loop [Push]
Wire {
  enabled: syncHoldProp == true
  from: "surface.loop.is_pressed"
  to: SetPropertyAdapter {
    path: "app.traktor.decks." + deckId + ".tempo"
    value: 0.0  // Reset to base BPM
  }
}
```

### Clock Master Mode Detection

**Description:** When global clock is master, same controls adjust global tempo instead of deck tempo.

**Implementation Pattern:**

```qml
// Clock Master Mode: Check if global clock is master
AppProperty {
  id: globalClockMaster
  path: "app.traktor.globals.clock_is_master_tempo"
}

Wire {
  enabled: syncHoldProp == true && globalClockMaster.value == true
  from: "surface.loop.is_turned"
  to: MappingPropertyDescriptor {
    path: "app.traktor.globals.master_bpm"  // Adjust global instead
  }
}
```

---

## Implementation Steps

1. **Identify SYNC button definition** in `X1MK3Deck.qml` (standard surface definition)
2. **Add syncHoldTimer** with 200ms interval to distinguish tap from hold
3. **Replace standard SYNC Wire** with ButtonScriptAdapter that:
   - Starts timer on press
   - Toggles SYNC only if timer is still running (< 200ms) on release
4. **Add fine adjustment Wire** linking Loop encoder turn to tempo property when SYNC held
5. **Add coarse adjustment Wire** linking Loop encoder turn to tempo with multiplier when both SYNC and Loop are held
6. **Add tempo reset Wire** on Loop push when SYNC held
7. **Add clock master detection** AppProperty and conditional wiring
8. **Add screen feedback** showing current tempo on deck view
9. **Test across all four decks** (A, B, C, D)
10. **Verify** no conflicts with other SYNC-based features (Sync LED feedback, Sync tap behavior)

---

## Testing Checklist

- [ ] **Sync Tap:** Press and quickly release SYNC → SYNC toggles on/off
- [ ] **Sync Hold:** Hold SYNC for 1+ second → SYNC does NOT toggle
- [ ] **Check BPM While Held:** Hold SYNC and look at deck display → see track BPM without toggling
- [ ] **Fine Adjustment (±0.01):** Sync held + turn Loop encoder → BPM changes ±0.01 per click
  - [ ] Verify on Deck A
  - [ ] Verify on Deck B
  - [ ] Verify on Deck C
  - [ ] Verify on Deck D
- [ ] **Coarse Adjustment (±1.00):** Sync held + Loop held + turn Loop encoder → BPM changes ±1.00 per click
  - [ ] Verify works with ±1.00 increment
  - [ ] Verify survives rapid tempo changes
- [ ] **Tempo Reset:** Sync held + push Loop encoder → tempo resets to song original BPM
  - [ ] Verify reset value is actually the original BPM
- [ ] **Deck vs. Clock Master:** When global clock is master:
  - [ ] Sync [Hold] + Loop adjusts global clock BPM (not deck)
  - [ ] Sync [Hold] + Loop [Push] resets global clock to current track BPM
  - [ ] Regular deck BPM adjustments still work when clock is NOT master
- [ ] **Sync LED Feedback:** Check that Sync button LED color changes reflect state
- [ ] **Screen Display:** Deck screens show current BPM value during adjustment
- [ ] **No Regressions:** Verify other Sync features still work (Sync LED, Sync LED colors for master mode)

---

## Compatibility

### Requires (Prerequisites)

- **X1 Infrastructure** ([X1_infrastructure.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_infrastructure.md))
  - SHIFT key handling, state management
  - Must be installed first

- **Traktor Pro 4.4.1+** — Core API required
- **X1 MK3 Hardware** — SYNC and Loop controls required

### Works Alongside

- **Beatgrid Control** ([X1_beatgrid-control.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_beatgrid-control.md))
  - Complementary tempo/grid features
  - Both use SYNC modifier, no conflict

- **Transport LED Feedback** ([X1_transport-led-feedback.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_transport-led-feedback.md))
  - Shows SYNC button state colors
  - No conflict

- **Screen Feedback** ([X1_screen-feedback.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_screen-feedback.md))
  - Displays BPM during adjustment
  - Complementary

### Conflicts With

- **Standard SYNC Toggle** — This feature redefines SYNC behavior (hold vs. tap distinction)

### Tested On

- X1 MK3 with Traktor Pro 4.4.1 (60–180 BPM range)
- X1 MK3 with Traktor Pro 4.5+

### NOT Tested On

- X1 MK2, other controllers (X1-specific)
- Clock master scenarios with multiple decks

---

## Notes

### Design Philosophy

- **Tap vs. Hold Distinction:** 200ms timer prevents accidental SYNC toggles when trying to hold button
- **Encoder Multiplier Pattern:** Loop held doubles tempo adjustment granularity (0.01 → 1.00)
- **Clock Master Awareness:** Automatically adjusts global vs. deck tempo based on Traktor state
- **Non-Destructive Hold:** Holding SYNC allows "peeking" at BPM without consequences

### User Experience

- Professional DJs familiar with Pioneer equipment will recognize "hold to adjust without toggling" pattern
- Fine adjustment (±0.01) allows micro-tuning during beatmatching
- Coarse adjustment (±1.00) allows quick macro adjustments (e.g., 95 BPM → 96 BPM instantly)
- Clock master support essential for live sets with multiple deck/master scenarios

### Traktor Behavior Notes

- "Reset tempo" sets tempo to 0.0, which Traktor interprets as "original track BPM"
- Clock master state can change mid-performance; wiring automatically routes to correct target
- Screen feedback updates in real-time as BPM adjusted (tested at 60 BPM and 180 BPM)

---

## Related Features

- [X1 Beatgrid Control](X1_beatgrid-control.md) - Complementary grid/beat features
- [X1 Deck Features](X1_deck-features.md) - Display options, gridlock, vinyl break
- [X1 Infrastructure](X1_infrastructure.md) - Foundational state management
- [X1 Transport LED Feedback](X1_transport-led-feedback.md) - Sync button LED colors/states

---

## Source & Resources

- **Forum:** [X1MK3 Community Performance MOD](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)
- **Creator:** [@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh)
- **Version:** 12 (Traktor 4.4.1)
