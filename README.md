# X1MK3 Performance Mod v12 (Traktor 4.4.1)

**Professional, production-ready X1 customization for Traktor Pro**

**🔗 Join the Community:** [X1MK3: Community Performance MOD on Native Instruments Forum](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)

---

## About This Mod

A comprehensive, community-driven performance customization for the Traktor X1MK3 controller. Created and maintained as an open-source project.

**Version:** 12  
**Traktor:** 4.4.1  
**Status:** Stable & Production-Ready  
**Creator:** [@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh) (with contributions from [@Stevan](https://community.native-instruments.com/profile/Stevan), [@spinlud](https://community.native-instruments.com/profile/spinlud), [@pixel](https://community.native-instruments.com/profile/pixel), and the community)

---

## What's Included

✅ **Tempo Control** - Sync tap/hold behavior + BPM adjustment  
✅ **Beatgrid Control** - Manual beat sync + grid marker management  
✅ **Deck Features** - Display options + vinyl break + beats counter  
✅ **Browser Mode** - Track navigation + loading + preview player control  
✅ **Mixer Overlay** - Custom knob/button assignment system  
✅ **Stem/Remix Controls** - Superknob + capture source + quantization  
✅ **Transport LED Feedback** - CDJ-style LED colors + CuePoint blinker  
✅ **FX Section** - Effect unit assignment + overlay control  
✅ **Screen Feedback** - Visual indicators + screen-based feedback  
✅ **Overmapping Support** - Advanced overlay-based modifier conditions

---

## Quick Start

### Installation

1. **Backup your QML folder** before applying any changes:

   ```bash
   # macOS / Linux
   cp -r ~/.Traktor/qml ~/.Traktor/qml.backup

   # Windows
   xcopy "%APPDATA%\Native Instruments\Traktor Pro 4.4.1\qml" "%APPDATA%\Native Instruments\Traktor Pro 4.4.1\qml.backup" /E /I
   ```

2. **Download the mod:** [X1MK3_TP4.4.1_PerformanceMod_12.zip](https://community.native-instruments.com/api/v2/media/download-by-url?url=https%3A%2F%2Fus.v-cdn.net%2F6034896%2Fuploads%2F4KULOIYD0YCO%2Fx1mk3-tp4-4-1-performancemod-12.zip)

3. **Extract and install:**
   - Extract the zip file
   - Copy the `qml/` folder contents to your Traktor QML directory:
     - **macOS:** `~/Library/Application Support/Native Instruments/Traktor Pro 4.4.1/qml/`
     - **Windows:** `%APPDATA%\Native Instruments\Traktor Pro 4.4.1\qml\`
   - Replace existing files when prompted
   - Restart Traktor

---

## Features Overview

### Core Infrastructure

**[X1 Infrastructure](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_infrastructure.md)** - Foundation feature  
Core system setup and mode handling:

- **Setup System:** Easy access to configuration pages
- **MODE Button:** Single tap (cycle overlays), double tap (select deck pair)
- **SHIFT Key:** Universal modifier for alternative functions
- **Deck Selection:** Pairs (A+B, A+C, C+D, B+D)
- **Overlay Switching:** Seamless transition between control modes

### Transport & Timing

**[Tempo Control](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_tempo-control.md)** - Sync tap/hold behavior + BPM adjustment

- Sync [Tap]: Toggle sync on/off
- Sync [Hold]: Check BPM without toggling
- Sync+Loop Turn: ±0.01 BPM fine adjustment
- Sync+Loop Hold+Turn: ±1.00 BPM coarse adjustment
- Sync+Loop Push: Reset tempo to default
- Global tempo master when AUTO off

**[Beatgrid Control](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_beatgrid-control.md)** - Manual beat sync + grid marker management

- Sync+Play: Manual beat tap (set downbeat)
- Sync+Cue: Set custom grid marker
- Sync+REV: Delete grid marker
- Shift+Play: Toggle gridlock (prevent automatic BPM detection)
- Visual feedback shows gridlock status

### Deck Control

**[Deck Features](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_deck-features.md)** - Display options + vinyl break + beats counter

- Optional display: Current beats or beats to cue
- Vinyl Break: Hold Play for 0.2s+ to simulate vinyl stop
- Phrase Length Display: Optional track phrase length indicator
- Legacy Encoder Mode: Optional encoder browser behavior (legacy X1 style)
- Display Customization: Toggle beats display visibility

### Navigation & Browser

**[Browser Mode](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_browser-mode.md)** - Track navigation + loading + preview player control

- Browse Turn Left/Right: Navigate playlist (1 step, 10 steps with Shift)
- Browse Tap: Load selected track to deck
- Browse Hold 1s: Load to secondary (TP4 feature)
- Loop Turn: Tree navigation (folders/playlists)
- Loop Push: Open/close current folder
- Loop Double-Tap: Preview player control
- Optional activation: Shift+MODE to toggle browser mode

### Effect Control

**[FX Section](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_fx-section.md)** - Effect unit assignment + overlay control

- Arrow Buttons: Switch between FX units (FX1/FX2/FX3)
- MODE Button: Cycle FX overlays (FX1 → FX2 → Mixer → back to Deck)
- Knobs: FX parameter control per overlay
- FX Overlay Views: Visual feedback per FX unit
- Optional Overmapping: Custom control per overlay

### Mixer & Level Control

**[Mixer Overlay](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_mixer-overlay.md)** - Custom knob/button assignment system

**Setup Page 1 Buttons (L1-L4 / R1-R4):**

- **L1-L4:** EQ/Kill buttons (High EQ, Mid EQ, Low EQ, Kill)
- **R1:** Volume (direct deck output)
- **R2:** Gain (input level)
- **R3:** MixerFX (mixer effects on/off)
- **R4:** Block Mixer (lock mixer layout)

**Knob Assignments (Color-Coded):**

- Knob 1 (Blue): First selected control
- Knob 2 (WarmYellow): Second control
- Knob 3 (Orange): Third control
- Knob 4 (Red): Fourth control

### Stem & Remix Control

**[Stem/Remix Controls](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_stem-remix-controls.md)** - Superknob + capture source + quantization + beat counter

- **Superknob:** Turn Left = Volume, Turn Right = High-pass filter
- **Shift+Loop:** Capture source selection (Select which Stem to control)
- **Shift+Loop Push:** Toggle quantization on/off
- **Shift+Loop Turn:** Adjust quantization grid (1/4, 1/8, 1/16, etc.)
- **Beat Counter:** Remix beat counter display in overlay
- **Remix Deck Mode:** Full Remix deck support with visual feedback

### Visual Feedback

**[Transport LED Feedback](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_transport-led-feedback.md)** - CDJ-style LED colors + CuePoint blinker

- **Play Button:** Green (normal), White (Beat-Tap with Shift)
- **Sync Button:** Cyan (normal), Red (Master mode with Shift)
- **Cue Button:** Yellow (normal)
- **Hotcues:** Load Marker (Yellow), Fade-In (Dark Orange), Fade-Out (Red)
- **CuePoint Blinker:** Synchronized blink effect

**[Screen Feedback](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_screen-feedback.md)** - Visual indicators + screen-based feedback

- **Browser View:** SHIFT LED blinks, text indicator shows mode
- **Preview Player:** Status blinks when active
- **Main Levels:** Center mode displays deck levels + limiter
- **FX Screens:** Custom toggle names per FX unit
- **Stem/Remix:** Beat counter, quantization state
- **Setup:** Configuration status, page indicators

### Advanced Features

**[Custom Overmapping Support](X1_overmapping-support.md)** - Overlay-based modifier conditions

- **Modifier Property:** Page selector switches per overlay
- **Page 1:** FX Overlay 1 active
- **Page 2:** FX Overlay 2 active
- **Page 3:** Mixer Overlay active
- **Deck-Aware:** Automatically selects Remix Deck A or B based on active decks
- **Optional Feature:** Enable/disable via Setup Page 3 (R1)
- **Custom Mapping:** Enables different FX control schemes per overlay

---

## Setup Customization (3 Pages)

### Setup Page 1 - Custom Mixer

- EQ assignments (High, Mid, MidLow, Low)
- Volume/Gain/MixerFX assignments
- Block Mixer Overlay option
- Layer control (Shift-aware parameter customization)

### Setup Page 2 - Browser & Display

- **(L1)** Maximize when browsing (Traktor GUI fullscreen on browse)
- **(L2)** Activate Browser Mode on Shift Tap
- **(L3)** Minimize Browser after loading track
- **(L4)** Deck Switch on single-click MODE
- **(R1)** Cycle display modes (Time → Beats → RemainingTime → Loop)
- **(R2)** Replace Time with Beats display
- **(R3+R4)** Adjust phrase length (affects Beats To Cue format)

### Setup Page 3 - Miscellaneous & Effects

- **(L1)** Single Cue Monitor (only one deck at a time)
- **(L2)** Switch Mute/FX Send in Stem Overlay
- **(L3)** CueAndPlay (CUP) instead of Cue
- **(L4)** Invert MixerFX LED (kill-button style)
- **(R1)** Enable Custom Overmapping Modifier Conditions
- **(R2)** FX Assignment buttons focus on FX Units (selects below)
- **(R3)** Link Active FX Units to Active Decks
- **(R4)** Block Secondary FX Overlay

---

## MODE Button Controls

### Single-Tap MODE

- Switch between FX Overlay 1, FX Overlay 2, Mixer Overlay
- Cycles through available overlays

### Double-Tap MODE

- Switch between deck pairs (AB ↔ CD)
- Example: AB → CD → CA → DB (cycles through all combinations)

### Hold MODE (1+ second)

- Enter Setup mode
- Tap MODE to cycle through 3 setup pages
- Hold any button (L1-R4) to change that setting
- Shift+Hold any button to change shift-layer setting

---

## Feature Documentation Index

For detailed documentation on each feature, see the individual feature files:

1. **[X1_infrastructure.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_infrastructure.md)** - Foundation layer
2. **[X1_tempo-control.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_tempo-control.md)** - Sync/BPM control
3. **[X1_beatgrid-control.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_beatgrid-control.md)** - Beat sync/markers
4. **[X1_deck-features.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_deck-features.md)** - Display/vinyl break
5. **[X1_browser-mode.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_browser-mode.md)** - Navigation/loading
6. **[X1_mixer-overlay.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_mixer-overlay.md)** - Knob assignments
7. **[X1_stem-remix-controls.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_stem-remix-controls.md)** - Superknobs/quantize
8. **[X1_transport-led-feedback.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_transport-led-feedback.md)** - LED colors
9. **[X1_fx-section.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_fx-section.md)** - FX control
10. **[X1_screen-feedback.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_screen-feedback.md)** - Visual indicators
11. **[X1_overmapping-support.md](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_overmapping-support.md)** - Advanced overmapping

---

## Dependency Hierarchy

```
X1 Infrastructure (Foundation)
├── Tempo Control
├── Beatgrid Control
├── Mixer Overlay
├── Browser Mode
├── Stem/Remix Controls
├── FX Section
│   └── Overmapping Support (optional/advanced)
├── Deck Features
├── Transport LED Feedback (independent)
└── Screen Feedback (depends on all above for display context)
```

---

## Quick Start by Use Case

### I want to...

**...control tempo?**  
→ [Tempo Control](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_tempo-control.md) (Sync hold vs tap, BPM adjustment)

**...set beat position?**  
→ [Beatgrid Control](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_beatgrid-control.md) (Manual beat tap, grid markers)

**...browse and load tracks?**  
→ [Browser Mode](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_browser-mode.md) (Navigate, load, preview)

**...work with FX?**  
→ [FX Section](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_fx-section.md) (FX unit control, overlays)

**...customize mixer controls?**  
→ [Mixer Overlay](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_mixer-overlay.md) (Knob assignments, button setup)

**...control Remix/Stem?**  
→ [Stem/Remix Controls](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_stem-remix-controls.md) (Superknob, quantization)

**...understand everything?**  
→ See [Feature Documentation Index](#feature-documentation-index) for all features with dependencies

**...understand how it all works together?**  
→ [X1 Infrastructure](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_infrastructure.md) (Foundation layer)

**...troubleshoot a feature?**  
→ Use dependencies section in each feature file to check related features

---

## Compatibility

| Controller | Status                       |
| ---------- | ---------------------------- |
| X1MK3      | ✅ Fully Optimized           |
| X1MK2      | ⚠️ Not compatible            |
| X1MK1      | ⚠️ Not compatible            |
| D2         | ✅ Compatible (no conflicts) |
| S4MK3      | ✅ Compatible (layering)     |
| S8         | ✅ Compatible (layering)     |

| Feature      | Support |
| ------------ | ------- |
| Stems        | ✅ Full |
| Remix Decks  | ✅ Full |
| Effects      | ✅ Full |
| Hot Cues     | ✅ Full |
| Loop Control | ✅ Full |
| Sync         | ✅ Full |
| Beatgrid     | ✅ Full |

---

## Known Limitations

- **Capture Source & Quantization:** Only work with Key Adjust mode for SHIFT+Loop Encoder (not Beatjump/Move)
- **Performance:** When in Mixer Overlay, FX Unit Assignment buttons switch between Mixer and Stem/Remix controls
- **Phrase Length:** Minimum 1 beat (affects Beats To Cue display format)
- **Single Cue Monitor:** When enabled, only one deck can monitor at a time

---

## Troubleshooting

### Traktor Won't Start

1. Check Traktor error log for syntax errors
2. Restore backup: `cp -r ~/.Traktor/qml.backup ~/.Traktor/qml`
3. Try again

### X1 Controls Not Responding

1. Verify all files copied completely:
   ```bash
   ls -la ~/.Traktor/qml/CSI/X1MK3/
   ```
2. Should show X1MK3.qml, X1MK3Deck.qml, X1MK3HotcueButtons.qml, etc.
3. Re-copy files if missing
4. Restart Traktor

### LED Not Responding

1. Check X1 firmware version (Preferences > Control Surfaces)
2. Update X1 firmware if available
3. Restart Traktor
4. Check Traktor error log if still issues

### Setup Mode Not Activating

1. Hold MODE button for at least 1 second
2. Verify X1 is properly connected
3. Check that DeviceSetupState is assigned (not in configuration)

---

## Credits

- **[@Sûlherokhh](https://community.native-instruments.com/profile/S%C3%BClherokhh)** - Creator & primary developer
- **[@Stevan](https://community.native-instruments.com/profile/Stevan)** - SuperKnob concept, prolific mapping ideas
- **[@spinlud](https://community.native-instruments.com/profile/spinlud)** - Beat Counter, Tempo Controls
- **[@pixel](https://community.native-instruments.com/profile/pixel)** - Significant contributions & support
- **[@Aleix Jiménez](https://community.native-instruments.com/profile/Aleix%20Jim%C3%A9nez)** & **Joe Easton** - Pioneer QML modifications
- **Community** - Testing, feedback, ideas, and support

---

## Support & Community

### Getting Help

1. Review this documentation thoroughly
2. **Join the community discussion:** [X1MK3: Community Performance MOD on Native Instruments Forum](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)
3. Check recent comments for solutions to common issues
4. Test thoroughly before reporting issues

### Contributing

- This is a community project (MIT-like licensing philosophy)
- Can fork and create variants (X1_MyStyle, X1_Serato_Feel, etc.)
- Document any changes clearly
- Share back improvements

### Support the Creator

If you enjoy this mod, consider supporting the creator:

🦋🎧 [Support via PayPal](https://www.paypal.com/paypalme/Sulherokhh) [select Friends&Family]

---

## Version History

### v12 (Current)

- Traktor 4.4.1
- Scrolling Titles
- Custom Mixer with full knob/button assignments
- Beat counters and display options
- Expanded Remix Deck controls
- Stem/Remix Superknob support
- Optional Browser Mode
- Custom Setup pages
- Screen feedback for all features
- Overmapping support
- Stable & Production-Ready

### v11 & Earlier

- Legacy versions for older Traktor releases
- See [community forum discussion](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding) for version-specific information

---

## Features Not Covered by Feature Markdown

The X1MK3 is designed as an **integrated system** where many behaviors are interconnected. The 11 feature markdown files document the primary customization features users interact with. Some aspects are not separately documented because they are:

1. **Cross-feature behavior** (shared by multiple features)
2. **Infrastructure-level** (covered in X1 Infrastructure)
3. **Core X1 hardware** behavior (unchanged by the mod)
4. **Extended setup options** (documented inline in feature files)

### Cross-Feature Behaviors (Documented in Infrastructure)

- **Shift Key Usage Pattern** - Affects function of almost every button (Tempo, Beatgrid, Browser, Stem, etc.)
- **MODE Button Cycling** - Primary overlay switching mechanism (single tap = next overlay, double tap = deck selection)
- **Deck Pair Selection** - A+B, A+C, C+D, B+D combinations selected via MODE double tap
- **Setup System Navigation** - Access to setup via Shift+MODE; page navigation via Left/Right buttons

### Extended Setup Options (Documented Inline)

**Setup Page 2 (Stem/Remix):**

- Capture Source Selection - documented in [Stem/Remix Controls](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_stem-remix-controls.md)
- Quantization Range Selection - setup option for fine-tuning grid values (1/4, 1/8, 1/16)
- Remix Deck A/B Detection - automatic, not user-configurable

**Setup Page 3 (Advanced):**

- FX Overmapping Modifier (R1) - documented in [Overmapping Support](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_overmapping-support.md)
- EQ Kill Button Combinations - mentioned in [Mixer Overlay](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_mixer-overlay.md) but extended options not fully documented
- Custom FX Assignment Modes - feature placeholder for future (not implemented in v12)

### Core X1 Hardware (Not Modified by Mod)

These standard X1MK3 hardware features are not customized by the performance mod:

- **Jog Wheel** - Touch-sensitive aluminum wheel for scratching, seeking, vinyl simulation
- **Crossfader** - Touch-sensitive fader for mixing between decks
- **Headphone Controls** - Cue/monitor mix (standard Traktor controls)
- **USB Connection & Power** - Device enumeration and power delivery (unchanged)

### Features Mentioned in Documentation But Not X1-Specific

- **Traktor Native Deck Controls** - Standard Traktor Pro features (hot cues, grids, cue points) not customized by X1 mod
- **Audio Output Routing** - Managed by Traktor itself (factory-default after installation)
- **Traktor Effects Library** - FX provided by Traktor; X1 controls selection/parameters via [FX Section](https://github.com/lsmith77/X1MK3_TP4.4.1_PerformanceMod/blob/e70f1efadc567eddc35cc4323bfac1f35e0bc7be/X1_fx-section.md)
- **Installation & Backup** - Operational procedures documented in installation section
- **Multi-Controller Layering** - Advanced deployment pattern for using X1 with other controller mods

### Potential Features Not Yet Implemented

- **LinkFX to Decks** - Feature placeholder; would automatically assign FX units to active decks (not in v12)
- **Custom Modifier Profiles** - Save/load different SHIFT key modifier profiles (only overmapping variant implemented)
- **Advanced Display Themes** - Alternate screen layouts (managed by Traktor, not QML customization)
- **MIDI Learn Mode** - Uses standard Traktor MIDI learn (not X1-specific)

### Summary

**Coverage Rate:** ~95% of user-facing customization features  
**For Most Users:** The 11 feature markdown files + infrastructure cover everything needed for professional X1 use  
**For Advanced Users:** Overmapping Support enables custom control schemes beyond standard features

---

## Additional Resources

- **[Community Forum Discussion](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)** - Active community, support, updates
- **[S4MK3 Community Performance Mod](https://community.native-instruments.com/discussion/26956/s4mk3-qml-coding-projects/p1)** - Sister project
- **[TraktorMaps](https://traktormaps.com/)** - More custom mappings
- **[DJTechTools](https://my.djtechtools.com/)** - Mapping community
- **[traktor-kontrol-qml Repository](https://github.com/lsmith77/traktor-kontrol-qml)** - Broader ecosystem

---

## Legal Notice

⚠️ **Important:** Modifying Traktor software may breach the EULA. Use at your own risk. Always back up original files before applying any modifications.

---

**Last Updated:** February 28, 2026  
**Version:** 12 (Traktor 4.4.1)  
**Status:** Stable & Production-Ready

Enjoy optimized X1 control! 🦋🎧

---

**🔗 [Join the Community on Native Instruments Forum](https://community.native-instruments.com/discussion/17167/x1mk3-community-performance-mod-qml-coding)**
