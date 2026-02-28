# X1MK3 Custom Overmapping Support (Modifier Conditions)

**Source:** X1MK3 Performance Mod v12 (Community Forum)  
**Note:** Feature distributed as QML code; no git history

---

## See Also

- **Foundation:** [X1 Infrastructure](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_infrastructure.md)
- **Related:** [FX Section](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_fx-section.md), [Mixer Overlay](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_mixer-overlay.md)
- **Reference:** Full feature index in [README](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/README.md)

Advanced feature enabling custom overmapping modifier conditions per overlay. Switching overlays (FX1, FX2, Mixer) automatically changes a modifier property that can be used in external overmapping assignments. This enables different FX control mappings for each overlay without code modifications.

## Features

- **Overlay-Based Modifier:** Switching overlays sets a modifier property
- **Sample Page Selector:** Uses Remix Deck A or B sample page selector as modifier
- **Page 1 = FX Overlay 1:** When in FX1 overlay, page = 1
- **Page 2 = FX Overlay 2:** When in FX2 overlay, page = 2
- **Page 3 = Mixer Overlay:** When in Mixer overlay, page = 3
- **Deck Combo Awareness:** Modifier affects which Remix Deck page is set (A or B)
- **Optional Feature:** Can be enabled/disabled in Setup Page 3 (R1)
- **No Audio Impact:** Visual/control mapping only, doesn't affect audio routing
- **Powerful Compatibility:** Enables users to define custom FX control schemes per overlay

---

## Supported Modifier Conditions

### Page Values (Set by Overlay)

- **Page 1:** FX Overlay 1 active
- **Page 2:** FX Overlay 2 active
- **Page 3:** Mixer Overlay active

### Deck Combo Routing

**Remix Deck A (Decks A+B, A+C, C+A):**

```
Deck A + B → Remix Deck A
Deck A + C → Remix Deck A
Deck C + A → Remix Deck A (same as A+C)
```

**Remix Deck B (Decks C+D, B+D):**

```
Deck C + D → Remix Deck B
Deck B + D → Remix Deck B
```

When in these combinations, switching overlays updates the correct Remix Deck page selector.

---

## Implementation Steps

1. **Add Setup Page 3 (R1) option** to enable/disable overmapping support
2. **Detect current deck combination** (AB, AC, AD, BC, BD, CD, CA, CB, DA, DB, DC)
3. **Map deck combos to Remix Decks:**
   - If Deck A involved → use Remix Deck A
   - If Deck D involved (and A not involved) → use Remix Deck B
4. **Create PAGE property** that tracks current overlay (1=FX1, 2=FX2, 3=Mixer)
5. **Add Wire** that sets sample page selector based on overlay + deck combo:
   ```qml
   if (deckComboIsRemixDeckA) {
     remixDeckA.samplePageSelector = currentOverlay  // 1, 2, or 3
   } else if (deckComboIsRemixDeckB) {
     remixDeckB.samplePageSelector = currentOverlay
   }
   ```
6. **Test all deck combinations** with overmapping
7. **Verify page selector updates** on overlay switch
8. **Create documentation** for users on how to use modifier in overmapping

---

## Testing Checklist

- [ ] **Feature Disabled (Default):** Start X1 with overmapping disabled (Setup R1):
  - [ ] Overlay switching works normally
  - [ ] No modifier property set
  - [ ] Remix sample page selector unchanged

- [ ] **Feature Enabled (Setup R1):** Enable overmapping support:
  - [ ] Setting persists across restart
  - [ ] Modifier property now available for overmapping

- [ ] **Page 1 = FX Overlay 1:** Load FX Overlay 1:
  - [ ] Sample page selector = 1 (for appropriate Remix Deck)
  - [ ] Remains 1 while in FX1 overlay
  - [ ] Visible in overmapping interface

- [ ] **Page 2 = FX Overlay 2:** Switch to FX Overlay 2:
  - [ ] Sample page selector = 2
  - [ ] Remains 2 while in FX2 overlay

- [ ] **Page 3 = Mixer Overlay:** Switch to Mixer Overlay:
  - [ ] Sample page selector = 3
  - [ ] Remains 3 in Mixer overlay

- [ ] **Deck Combo A+B → Remix Deck A:** Select Deck A+B pair:
  - [ ] Remix Deck A sample page selector used
  - [ ] Overlay switch updates Remix Deck A page

- [ ] **Deck Combo C+D → Remix Deck B:** Select Deck C+D pair:
  - [ ] Remix Deck B sample page selector used
  - [ ] Overlay switch updates Remix Deck B page

- [ ] **Deck Combo A+C → Remix Deck A:** Select Deck A+C pair:
  - [ ] Remix Deck A sample page selector used (A takes priority)
  - [ ] Works correctly even though not adjacent decks

- [ ] **Page Mapping in Overmapping:** Create test overmapping with modifier condition:
  - [ ] Condition "Sample Page = 1" when overlay is FX1
  - [ ] Condition "Sample Page = 2" when overlay is FX2
  - [ ] Condition "Sample Page = 3" when overlay is Mixer
  - [ ] Overmapping rules trigger/untrigger correctly per overlay

- [ ] **No Audio Impact:** Verify overmapping doesn't affect:
  - [ ] Audio routing
  - [ ] Deck playback
  - [ ] FX signal flow
  - [ ] Remix sample playback (unless explicitly mapped)

---

## Dependencies

**Requires:**

- [X1 Infrastructure](X1_infrastructure.md) - Overlay state, deck selection
- Traktor 4.4.1 with overmapping support (Traktor 4+ feature)
- X1MK3 hardware
- Remix deck available (for sample page selector to work)

**Optional:**

- [X1 FX Section](X1_fx-section.md) - FX overlay integration

---

## Notes

### Design Philosophy

- **Non-Destructive Mapping:** Overmapping is purely control-layer, doesn't affect audio
- **Deck-Aware Modifier:** Automatically selects correct Remix Deck based on active decks
- **Optional Feature:** Can be disabled for simpler workflows
- **Professional Integration:** Enables advanced users to create highly customized FX control schemes

### User Experience

- Advanced feature for users creating complex custom mappings
- Allows different FX parameter assignments per overlay
- No need to manually manage Remix Deck selection
- Clear modifier values (1, 2, 3) easy to remember and track

### Traktor Behavior Notes

- Remix sample page selector available on all Remix Decks
- Sample page selector is primarily used for overmapping conditions
- Page values 1-3 match overlay numbering (mnemonic)
- Overmapping system built-in to Traktor 4+

### Known Limitations

- Remix Deck must be available/loaded for page selector to work
- Overmapping feature requires Traktor 4 (not available in TP3.x)
- Cannot customize page mapping (fixed 1=FX1, 2=FX2, 3=Mixer)
- Requires user to understand overmapping (not beginner-friendly)

---

## Example Overmapping Use Case

```
Scenario: Different FX knobs per overlay

Setup:
1. Enable overmapping (Setup Page 3, R1)
2. Create 3 overmapping rules:

Rule 1: Sample Page = 1 (FX Overlay 1)
  FX Knob 1 → FX Unit 1 Parameter
  FX Knob 2 → FX Unit 2 Parameter
  [Standard FX control]

Rule 2: Sample Page = 2 (FX Overlay 2)
  FX Knob 1 → FX Unit 3 Parameter
  FX Knob 2 → FX Unit 4 Parameter
  [Different control scheme]

Rule 3: Sample Page = 3 (Mixer Overlay)
  FX Knobs disabled (Superknob/Mixer active instead)
  [No FX parameter control]

Result: Seamlessly switch FX control schemes per overlay!
```

---

## Related Features

- [X1 Infrastructure](X1_infrastructure.md) - Overlay state management
- [X1 FX Section](X1_fx-section.md) - FX control integration
- [X1 Mixer Overlay](X1_mixer-overlay.md) - Overlay switching coordination

---

## Source & Resources

- **Forum:** [X1MK3 Community Performance MOD](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)
- **Creator:** [@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh)
- **Version:** 12 (Traktor 4.4.1)
- **Traktor Overmapping Docs:** Available in Traktor Pro 4 documentation
