# X1MK3 Browser Mode (Navigation & Track Loading)

**Source:** X1MK3 Performance Mod v12 (Community Forum)  
**Note:** Feature distributed as QML code in ZIP archive; no git history

---

## See Also

- **Foundation:** [X1 Infrastructure](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_infrastructure.md) (SHIFT key, MODE button)
- **Feedback:** [Screen Feedback](https://github.com/lsmith77/X1MK3_PerformanceMod/blob/20cf2811fe29b2a559068fb4c6c6b8fd4d1994af/X1_screen-feedback.md)

Dedicated browser mode transforms the X1's encoders into a powerful track navigation and preview system. Toggle BrowserMode via Shift [Hold] + Shift [Tap] to navigate playlists, trees, load tracks, preview music, and manage favorites—without touching the Traktor GUI.

## Features

- **Browser Mode Toggle:** Shift [Hold] activates modifier, Shift [Tap] toggles Browser Mode (Setup L2)
- **Browser View Integration:** Opening browser automatically opens Traktor's browser view
- **Playlist Navigation:** Left/Right Browse [Turn] → navigate playlist (1 step per click)
- **Fast Navigation:** Shift [Hold] + Browse [Turn] → navigate playlist (10 steps per click)
- **Track Loading:** Left/Right Browse [Tap] → Load Primary deck track (closes Preview)
- **Secondary Loading (TP4):** Left/Right Browse [Hold 1 sec] → Load Secondary deck track only on Traktor 4
- **Tree Navigation:** Left Loop [Turn] → Navigate folder tree (1 step per click)
- **Fast Tree:** Shift [Hold] + Left Loop [Turn] → Navigate tree (10 steps per click)
- **Open/Close Node:** Left Loop [Push] → Open/close current folder
- **Preview Player:** Right Loop [Push] → Load track preview; Right Loop [Turn] → Seek preview
- **Favorites:** Shift [Hold] + Right Loop [Turn] → Cycle through favorites
- **Preparation List:** Shift [Hold] + Right Loop [Push] → Jump to preparation list
- **Automatic Minimize:** Browser view closes automatically when track loaded (optional in Setup L3)
- **Screen Feedback:** Deck screens show current browser navigation mode and preview status

## Modified Files

- `CSI/X1MK3/X1MK3.qml` - Browser mode state management
- `CSI/X1MK3/X1MK3Deck.qml` - Encoder function switching in Browser Mode
- `Screens/X1MK3/DeckView.qml` - Browser mode indicator, playlist/tree feedback

**Note:** Without git history, exact line numbers cannot be provided. Refer to files for implementation.

---

## Changes in `X1MK3.qml`

### Browser Mode Toggle

**Description:** Shift [Tap] toggles Browser Mode on/off when setup is enabled (Setup Page 2, L2).

**Implementation Pattern:**

```qml
// Browser Mode Toggle: Shift [Tap] (when enabled in Setup L2)
AppProperty {
  id: fullscreenBrowseProp
  path: "app.traktor.browser.full_screen"
}

AppProperty {
  id: browserModeProp
  path: mapping.propertiesPath + ".browser_mode"  // 0=off, 1=on
}

Timer {
  id: holdShift_countdown
  interval: 200  // Distinguish tap from hold
}

Wire {
  from: "surface.shift"
  to: ButtonScriptAdapter {
    onPress: {
      shiftProp.value = true
      holdShift_countdown.restart()
    }
    onRelease: {
      shiftProp.value = false
      if (holdShift_countdown.running && setupL2.value == true) {
        // Browser Mode Tap: toggle mode
        browserModeProp.value = !browserModeProp.value
        // Also toggle Traktor browser view
        fullscreenBrowseProp.value = !fullscreenBrowseProp.value
        holdShift_countdown.stop()
      }
    }
  }
}
```

---

## Changes in `X1MK3Deck.qml`

### Playlist Navigation (Browse Encoders)

**Description:** Left/Right Browse encoders navigate playlist when Browser Mode active.

**Implementation Pattern:**

```qml
// Playlist Navigation: Browse [Turn] when Browser Mode ON
Wire {
  enabled: browserModeProp.value == true
  from: "surface.left.browse.is_turned"
  to: MappingPropertyDescriptor {
    path: "app.traktor.browser.selected_item_index"
    step: shiftProp == true ? 10 : 1  // Shift = 10-step jump
  }
}

Wire {
  enabled: browserModeProp.value == true
  from: "surface.right.browse.is_turned"
  to: MappingPropertyDescriptor {
    path: "app.traktor.browser.selected_item_index"
    step: shiftProp == true ? 10 : 1
  }
}
```

### Load Track (Browse [Tap])

**Description:** Browse [Tap] loads selected track to this deck.

**Implementation Pattern:**

```qml
// Load Track: Browse [Tap]
Wire {
  enabled: browserModeProp.value == true
  from: "surface.left.browse.is_pressed"
  to: SetPropertyAdapter {
    path: "app.traktor.decks." + deckId + ".load_track"
    // Also close Preview Player on load
    onValueChanged: {
      previewPlayerActive.value = false
      fullscreenBrowseProp.value = false  // Close browser (if Setup L3)
    }
  }
}
```

### Tree Navigation (Loop Encoders)

**Description:** Left Loop encoder navigates folder tree when Browser Mode active.

**Implementation Pattern:**

```qml
// Tree Navigation: Left Loop [Turn]
Wire {
  enabled: browserModeProp.value == true
  from: "surface.left.loop.is_turned"
  to: MappingPropertyDescriptor {
    path: "app.traktor.browser.selected_category_index"
    step: shiftProp == true ? 10 : 1  // Shift = 10-step jump
  }
}

// Open/Close Node: Left Loop [Push]
Wire {
  enabled: browserModeProp.value == true
  from: "surface.left.loop.is_pressed"
  to: SetPropertyAdapter {
    path: "app.traktor.browser.open_category"
    value: true
  }
}
```

### Preview Player (Right Loop Encoder)

**Description:** Right Loop controls preview player: Push to load, Turn to seek.

**Implementation Pattern:**

```qml
// Load Preview: Right Loop [Push]
Wire {
  enabled: browserModeProp.value == true
  from: "surface.right.loop.is_pressed"
  to: SetPropertyAdapter {
    path: "app.traktor.browser.load_preview"
    value: true
  }
}

// Seek Preview: Right Loop [Turn]
Wire {
  enabled: browserModeProp.value == true
  from: "surface.right.loop.is_turned"
  to: MappingPropertyDescriptor {
    path: "app.traktor.browser.preview_seek"
  }
}

// Favorites: Shift [Hold] + Right Loop [Turn]
Wire {
  enabled: browserModeProp.value == true && shiftProp == true
  from: "surface.right.loop.is_turned"
  to: SetPropertyAdapter {
    path: "app.traktor.browser.cycle_favorites"
    value: true
  }
}

// Preparation List: Shift [Hold] + Right Loop [Push]
Wire {
  enabled: browserModeProp.value == true && shiftProp == true
  from: "surface.right.loop.is_pressed"
  to: SetPropertyAdapter {
    path: "app.traktor.browser.goto_preparation_list"
    value: true
  }
}
```

---

## Implementation Steps

1. **Add browserModeProp** to infrastructure state tracking (on/off)
2. **Add Shift [Tap] detection** with 200ms timer to distinguish tap from hold
3. **Add Browser Mode toggle Wire** that also opens/closes Traktor browser view
4. **Add auto-minimize Browser Logic** when track loaded (checks Setup L3)
5. **Modify Browse encoder Wires** to route differently when Browser Mode ON:
   - Normal: Disabled or unused
   - Browser Mode: Navigate playlist + tree
6. **Modify Loop encoder Wires** to switch functions in Browser Mode:
   - Left: Tree navigation + open/close
   - Right: Preview player + favorites/prep list
7. **Add screen feedback** showing "Browser Mode: ON/OFF" and playlist/tree position
8. **Add Preview Player indicator** on screen ("Preview Playing: OFF/ON")
9. **Test Browse encoder navigation** at different playlist depths
10. **Test all four decks** independently manage browser context

---

## Testing Checklist

- [ ] **Browser Mode Toggle:** Shift [Tap] when Setup L2 enabled:
  - [ ] Browser Mode toggles on/off
  - [ ] Traktor browser view opens/closes automatically
  - [ ] Screen shows "Browser Mode: ON" or "Browser Mode: OFF"
- [ ] **Playlist Navigation (normal):** Browser Mode ON + Browse [Turn]:
  - [ ] Playlist scrolls 1 item per click
  - [ ] Works on left and right browse encoders
  - [ ] Wraps correctly at playlist boundaries
- [ ] **Playlist Navigation (fast):** Browser Mode ON + Shift [Hold] + Browse [Turn]:
  - [ ] Playlist scrolls 10 items per click
  - [ ] No lag at rapid scrolling speed
- [ ] **Track Loading:** Browser Mode ON + Browse [Tap]:
  - [ ] Selected track loads to deck
  - [ ] Preview player closes automatically
  - [ ] Browser view closes if Setup L3 enabled
- [ ] **Secondary Loading (TP4 only):** Browse [Hold 1 sec]:
  - [ ] Hold longer than 1 second → loads as Secondary deck
  - [ ] Tap loads as Primary (default)
- [ ] **Tree Navigation:** Browser Mode ON + Left Loop [Turn]:
  - [ ] Folder tree navigation works
  - [ ] Shows current folder depth on screen
  - [ ] Shift [Hold] + Left Loop [Turn] → 10-step jumps
- [ ] **Open/Close Folder:** Browse Mode ON + Left Loop [Push]:
  - [ ] Opens selected folder
  - [ ] Closes folder if already open
  - [ ] Works at different tree depths
- [ ] **Preview Player:** Browser Mode ON + Right Loop [Push]:
  - [ ] Loads selected track preview
  - [ ] Screen shows "Preview: Playing"
  - [ ] Preview plays audio (verify with headphones)
- [ ] **Preview Seek:** Right Loop [Turn]:
  - [ ] Seeks through preview playback
  - [ ] Position updates on screen
- [ ] **Favorites:** Shift [Hold] + Right Loop [Turn]:
  - [ ] Cycles through saved favorite playlists
  - [ ] Screen feedback shows current favorite
- [ ] **Preparation List:** Shift [Hold] + Right Loop [Push]:
  - [ ] Jumps to preparation list
  - [ ] Browser resets to prep list root
- [ ] **Auto-Close Browser (Setup L3):** Load track while Browser Mode ON:
  - [ ] Track loads and browser view closes (if L3 enabled)
  - [ ] Browser stays open if L3 disabled
- [ ] **Browser Mode OFF:** Disable Browser Mode:
  - [ ] Browse/Loop encoders revert to normal functions
  - [ ] Tree navigation disabled
  - [ ] Preview player controls disabled

---

## Dependencies

**Requires:**

- [X1 Infrastructure](X1_infrastructure.md) - SHIFT state, Setup configuration (Page 2: L2, L3)
- Traktor 4.4.1 (exactly this version)
- X1MK3 hardware

**Optional:**

- Works alongside all other features (no conflicts)
- Configuration via Setup Page 2 (L1, L2, L3)

---

## Notes

### Design Philosophy

- **Encoder Function Switching:** Browse/Loop encoders behave differently based on Browser Mode state
- **Shift as Modifier:** Shift [Hold] provides fast navigation (1 step → 10 steps)
- **Traktor Integration:** Browser view opens/closes automatically with mode toggle
- **Non-Destructive:** Browsing doesn't modify tracks or playlists

### User Experience

- Removes need to switch to Traktor GUI for track selection
- Powerful tree navigation from controller encoders
- Preview tracks before loading (essential for mixing)
- Fast playlist scrolling for large libraries (10-step jumps)

### Traktor Behavior Notes

- `selected_item_index` must be updated programmatically for smooth navigation
- `load_preview` property closes existing preview before loading new one
- Preview player separate from deck playback (safe to preview while playing)
- Tree navigation (`selected_category_index`) applies only in tree view, not flat playlists
- Favorites cycling requires playlists marked as favorites in Traktor

### Known Limitations

- Tree navigation depends on playlist structure (flat playlists have no tree)
- Preview seek position resets on new preview load
- Cannot move between decks while in Browser Mode (deck focus stays)
- Shift [Tap] has 200ms window; faster taps may not register

---

## Related Features

- [X1 Infrastructure](X1_infrastructure.md) - SHIFT state, Setup Page 2 configuration
- [X1 Deck Features](X1_deck-features.md) - Screen feedback integration
- [X1 Transport LED Feedback](X1_transport-led-feedback.md) - SHIFT LED blinks in Browser Mode

---

## Source & Resources

- **Forum:** [X1MK3 Community Performance MOD](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)
- **Creator:** [@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh)
- **Version:** 12 (Traktor 4.4.1)
