# X1MK3 Deck Features (Display Options, Gridlock, Vinyl Break)

**Source:** X1MK3 Performance Mod v12 (Community Forum)  
**Commit:** https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_deck-features.md  
**Traktor:** 4.4.1

---

## What It Does

Display customization for deck information (time, beats, cue countdown, remaining time), convenient grid lock toggling via SHIFT+Play, and realistic vinyl break simulation by holding the play button for 0.2+ seconds. Choose display modes, toggle grid lock, and simulate turntable slowdowns.

- **Display Options:** Cycle through Time → Beats → BeatsToCue → RemainingTime → Loop Size (customizable in Setup Page 2)
- **Beats Display:** Replace time with beat counter (bars.beats)
- **BeatsToCue Display:** Show bars and beats until next cue point
- **Gridlock Toggle:** Shift [Hold] + Play → Toggle grid lock on/off
- **Vinyl Break:** Hold Play/Pause for 0.2+ seconds during playback → Simulates vinyl turntable slowdown
- **Break Duration:** Longer hold = slower break; release to complete
- **Sync Behavior:** Sync disengages during break, re-engages on stop/restart
- **Screen Feedback:** All features reflected immediately on deck screens

## Modified Files

- `CSI/X1MK3/X1MK3Deck.qml` - Display cycling, gridlock toggle, vinyl break logic
- `Screens/X1MK3/DeckView.qml` - Display options screen feedback

**Note:** Without git history, exact line numbers cannot be provided. Refer to `X1MK3Deck.qml` for implementation location.

---

## Configuration

### MOD Settings

Add this to your main X1MK3.qml controller file:

```qml
// --- MOD SETTINGS: Deck Features ---
MappingPropertyDescriptor {
    id: deckFeaturesEnabledProp
    path: "app.traktor.settings.deck_features_enabled"
    initialValue: true  // Set to false to disable deck customization features
}
// --- END MOD SETTINGS ---
```

---

## See Also

- **Foundation:** [X1 Infrastructure](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_infrastructure.md)
- **Related:** [Tempo Control](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_tempo-control.md), [Beatgrid Control](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_beatgrid-control.md)
- **Feedback:** [Screen Feedback](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_screen-feedback.md)

---

## Changes in `X1MK3Deck.qml`

### Display Mode Cycling

**Description:** Cycle through different time displays via Setup configuration or manual toggle.

**Implementation Pattern:**

```qml
// Display Mode Cycling: Setup Page 2 (R1)
AppProperty {
  id: displayModeProp
  path: mapping.propertiesPath + ".display_mode"  // 0=Time, 1=Beats, 2=BeatsToCue, 3=RemainingTime, 4=LoopSize
}

Wire {
  from: "surface.cycleDisplayButton"  // Configurable via Setup
  to: SetPropertyAdapter {
    path: displayModeProp.path
    value: (displayModeProp.value + 1) % 5  // Cycle 0→1→2→3→4→0
  }
}
```

### Beats Display Mode

**Description:** Optional: Replace time display with beat counter (bars.beats).

**Implementation Pattern:**

```qml
// Beats Display: Setup Page 2 (R2)
AppProperty {
  id: beatDisplayToggle
  path: mapping.propertiesPath + ".beats_display_on"
}

Wire {
  enabled: beatDisplayToggle.value == true
  from: "surface.displayUpdateTrigger"
  to: "screenDisplayBeatCounter"
}

// Screen feedback shows bars.beats and beats_to_cue
```

### BeatsToCue Display

**Description:** Show bars and beats until next cue point (or end of phrase).

**Implementation Pattern:**

```qml
// BeatsToCue Display: Beats To Cue countdown
AppProperty {
  id: beatsToCueProp
  path: "app.traktor.decks." + deckId + ".cue.beats_to_cue"
}

AppProperty {
  id: beatPhraseLength
  path: mapping.propertiesPath + ".phrase_length"  // Adjustable in Setup
}

ScreenDisplay {
  id: beatsToCueDisplay
  text: beatsToCueProp.value >= 0 ?
    (Math.floor(beatsToCueProp.value / beatPhraseLength.value) + "." + (beatsToCueProp.value % beatPhraseLength.value)) :
    "--"
}
```

### Gridlock Toggle

**Description:** Shift [Hold] + Play toggles grid lock state.

**Implementation Pattern:**

```qml
// Gridlock Toggle: Shift [Hold] + Play
Wire {
  enabled: shiftProp == true && syncHoldProp == false
  from: "surface.play"
  to: ButtonScriptAdapter {
    onPress: {
      gridlockProp.value = !gridlockProp.value
      // Screen feedback shows "Grid Lock: ON" or "Grid Lock: OFF"
    }
  }
}

AppProperty {
  id: gridlockProp
  path: "app.traktor.decks." + deckId + ".gridlock_enabled"
}
```

### Vinyl Break Simulation

**Description:** Hold Play button for 0.2+ seconds to simulate vinyl turntable slowdown.

**Implementation Pattern:**

```qml
// Vinyl Break: Hold Play for 0.2+ seconds
Wire {
  from: "surface.play"
  to: ButtonScriptAdapter {
    onPress: {
      playHoldTimer.restart()
      vinylBreakActive = false
    }
    onRelease: {
      if (vinylBreakActive) {
        // Activate vinyl break effect
        activateVinylBreak(playHoldTimer.elapsed)  // Duration = hold time
        syncProp.value = false  // Disengage Sync temporarily
        playHoldTimer.stop()
      } else {
        // Standard play behavior
        playProp.toggle()
        playHoldTimer.stop()
      }
    }
  }
}

Timer {
  id: playHoldTimer
  interval: 200  // 0.2 second threshold
  onTriggered: { vinylBreakActive = true }
}

function activateVinylBreak(holdDuration) {
  // Non-linear slowdown: longer hold = slower break
  var breakSpeed = holdDuration / 1000  // Convert ms to seconds
  // Ramp down playback speed gradually over held duration
  deckPlaybackRate.rampTo(0.0, breakSpeed)
}
```

---

## Implementation Steps

1. **Add display mode property** to track current display (0-4: Time, Beats, BeatsToCue, RemainingTime, LoopSize)
2. **Add display cycling Wire** that increments mode on button press/Setup trigger
3. **Add Beats Display Wire** conditional on Setup Page 2 (R2) toggle
4. **Add BeatsToCue calculation** showing bars.beats remaining until next cue
5. **Add phrase length property** customizable in Setup (affects BeatsToCue format)
6. **Add gridlock detection AppProperty** for each deck
7. **Add Gridlock Toggle Wire** on Shift [Hold] + Play button
8. **Add vinyl break timer** (200ms threshold for tap vs. hold detection)
9. **Add vinyl break function** that ramps down playback rate based on hold duration
10. **Add Sync disengagement** during vinyl break, re-engagement on playback stop
11. **Add screen feedback** for all display modes and gridlock state
12. **Test on all four decks** independently

---

## Testing Checklist

- [ ] **Display Cycling:** Load track → press display cycle button (Setup configurable):
  - [ ] Cycles through Time → Beats → BeatsToCue → RemainingTime → Loop → Time
  - [ ] Screen updates immediately on each cycle
  - [ ] Works on all four decks independently
- [ ] **Beats Display (Setup R2):** Enable beats display in Setup Page 2:
  - [ ] Time display is replaced with beat counter (e.g., "2.4" = bar 2, beat 4)
  - [ ] Beat counter updates in real-time during playback
  - [ ] Disable beats display → reverts to time
- [ ] **BeatsToCue Display:** Play track with loaded cue → watch BeatsToCue countdown:
  - [ ] Shows bars.beats remaining until cue point
  - [ ] Format: "3.2" = 3 bars, 2 beats until cue
  - [ ] Counts down as playback advances
  - [ ] Resets on cue point hit
- [ ] **Phrase Length Adjustment (Setup R3+R4):**
  - [ ] Set phrase length = 1 → BeatsToCue shows single beat format
  - [ ] Set phrase length = 4 → BeatsToCue shows 4-beat phrase format
  - [ ] Change affects BeatsToCue display format immediately
- [ ] **Gridlock Toggle:** Shift [Hold] + Play → toggle grid lock:
  - [ ] Grid lock toggles on/off
  - [ ] Screen shows "Grid Lock: ON" or "Grid Lock: OFF"
  - [ ] Gridlock state persists across track changes
  - [ ] When grid is locked, beatgrid controls disabled (infrastructure check)
- [ ] **Vinyl Break (< 0.2 seconds):** Tap Play button quickly:
  - [ ] Standard play button behavior (toggle playback)
  - [ ] No vinyl break effect
- [ ] **Vinyl Break (0.2+ seconds hold):** Hold Play button during playback:
  - [ ] Playback starts slowing (ramping down)
  - [ ] Slower hold = slower ramp; longer hold = faster ramp
  - [ ] Release button to complete break at current speed
  - [ ] Sync disengages during break
- [ ] **Vinyl Break Completion:** Hold Play until playback stops:
  - [ ] Playback reaches full stop smoothly
  - [ ] Release button to restart playback
  - [ ] Sync re-engages after break completes
- [ ] **No Regressions:** All original playback controls still work:
  - [ ] Cue button works normally
  - [ ] Reverse button works normally
  - [ ] Jog wheel control unaffected

---

## Dependencies

**Requires:**

- [X1 Infrastructure](X1_infrastructure.md) - SHIFT state, screen feedback
- [X1 Beatgrid Control](X1_beatgrid-control.md) - Grid lock affects beatgrid operations
- Traktor 4.4.1 (exactly this version)
- X1MK3 hardware

**Optional:**

- [X1 Mixer Overlay](X1_mixer-overlay.md) - Setup Page 2 shares customization

---

## Notes

### Design Philosophy

- **Hold vs. Tap:** 200ms timer distinguishes Vinyl Break hold from standard Play tap
- **Non-Destructive Display:** Cycling through displays doesn't change underlying track position
- **Sync-Aware Vinyl Break:** Automatically disengages/re-engages Sync during break
- **Gridlock Protection:** Prevents accidental grid modifications when locked

### User Experience

- DJs can quickly switch between different time reference (absolute time, beat count, remaining time)
- Vinyl Break simulates turntable feel; useful for dramatic DJ transitions
- Gridlock toggle prevents accidental grid modifications without entering setup
- All features integrated into playback workflow

### Traktor Behavior Notes

- Display mode is X1-specific; Traktor deck display unchanged
- Vinyl break is simulated via playback rate ramping (not physical jog)
- BeatsToCue counter is absolute (based on cue point location, not phrase length)
- Gridlock state synchronized with Traktor (visible in Traktor GUI)
- Phrase length affects display format only; doesn't change cue detection

---

## Related Features

- [X1 Infrastructure](X1_infrastructure.md) - SHIFT state, Setup configuration
- [X1 Beatgrid Control](X1_beatgrid-control.md) - Grid lock interaction
- [X1 Tempo Control](X1_tempo-control.md) - Tempo + gridlock relationship
- [X1 Transport LED Feedback](X1_transport-led-feedback.md) - Play button LED states

---

## Source & Resources

- **Forum:** [X1MK3 Community Performance MOD](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)
- **Creator:** [@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh)
- **Version:** 12 (Traktor 4.4.1)
