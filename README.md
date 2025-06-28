# zmk-config
> zmk setup for my own keyboards

## Supported Keyboards
- [Fifi (own shield)](https://github.com/raychengy/fifi_split_keeb)
- [Corne Ish Zen](https://lowprokb.ca/products/corne-ish-zen)
- [Forager](https://github.com/carrefinho/forager)
- [Urchin](https://github.com/duckyb/urchin)
- ~[Dactyl Manuform](https://github.com/abstracthat/dactyl-manuform)~ - WIP

## General Principles
- usage of [**Homerow Mods**](#homerow-mods-(hrm))
- keep [thumb health](#thumb-health) in mind
- trying to work with 34 keys for less thumb responsibilities
- **combos**:
    - only use combos with multiple fingers (horizontal combos)
    - try to avoid pinkies
- support **left hand only** usage
    - combos on left hand side that work well with right-handed mouse usage (cut, copy, paste, esc)
    - common shortcuts and desktop operations
    - mostly covered by `NAV` layer
- Miryoku-style layers (mods on same side but with more functions on the same-side half)

### Thumb Health
- place only two to three keys in arc thumb naturally sweeps
- don't stretch to keys under palm/beyond index-column
    - **bending the thumb/stretching far shows up frequently in pain reports**
    - only use low-frequency functions or nothing at all
- put high-frequency key under thumb's resting position (e.g. spacebar)
- minimize sustained holds -> combination of layers and combos
- design/choose a thumb cluster for your own hand size
- try to put heavier loads on left thumb instead of right thumb (smartphone usage)
- sources can be found in [References](#references)

## Keymap
> keymap automatically drawn with [keymap-drawer](https://github.com/caksoylar/keymap-drawer)
![Keymap](./draw/forager.svg?raw=true "Keymap")

## Homerow Mods (HRM)
> Read [this](https://precondition.github.io/home-row-mods) post by precondition to understand
> further sources can be found in [References](#references)
> images are all taken from the ZMK documentation

### Problems with Naive HRM
- require some difficult timing to function properly
- hold longer than `tapping-term-ms` to produce a modifier key
- hold less than `tapping-term-ms` to produce a normal keypress
- for them to function properly consistent typing speeds are required 
- can lead to misfires on inconsistent typing speeds
- ![hold_tap](./assets/hold_tap.svg?raw=true "Hold Tap")

### Interrupt Flavors for Hold-Tap
> make sure to understand these cases for own your own trouble-shooting
- `hold-preferred`: triggers *hold* after `tapping-term-ms` or on another *keypress*
- `balanced`: triggers *hold* after `tapping-term-ms` or if another *keypress* is *pressed* and *released* while hold-tap is *held*
- `tap-preferred`: triggers *hold* after `tapping-term-ms` has expired 
- ![interrupt_flavors](./assets/interrupt_flavors.svg?raw=true "Interrupt Flavors")

### Variables for Hold-Tap explained
- `tapping-term-ms`: defines how long a key must be pressed to trigger a *hold*
- `quick-tap-ms`: *tap* into *tap*/*hold* again within `quick-tap-ms` always leads to *tap* (good for spamming keys)
- `require-prior-idle-ms`: 
    - hold-tap key pressed within `require-prior-idle-ms` of another non-modifier key always leads to *tap*
    - disables hold-tap when typing quickly and removes input delay
    - perfect solution for issues with rolling keys on homerow
- positional hold-tap:
    - `hold-trigger-key-positions`: enables the feature and defines keys that do not cancel *hold*
    - if you press any key *not listed in* `hold-trigger-key-positions` a *tap* will be produced (before `tapping-term-ms` expires)
    - useful for homerow mods as only cross-hand combinations can trigger *hold* (before `tapping-term-ms` expires)
    - `hold-trigger-on-release`: produce a *tap* when *released* not *pressed* -> set this with home row mods to combine same-hand modifiers

### Final Setup (based on [urob - zmk-config](https://github.com/urob/zmk-config))
- use `balanced` flavor to be independent of `tapping-term-ms`
- set `require-prior-idle-ms` to solve typing delay
- set `hold-trigger-key-positions` to solve rolling keys
- set `hold-trigger-on-release` to still allow combinations of modifiers on same hand
- don't set `tapping-term-ms` to infinity to still use modifier without another key or mod with alpha-key on same hand
- setup a *sticky shift* key - shifting may otherwise conflict with `require-prior-idle-ms` for fast typers

### Example
```C++
// adapt these to your keymap (0 is upper left -> columns -> rows)
#define KEYS_L 0 1 2 3 4 10 11 12 13 14 20 21 22 23 24 
#define KEYS_R 5 6 7 8 9 15 16 17 18 19 25 26 27 28 29
#define THUMBS 30 31 32 33 34 35

/ {
    behaviors {
        hml: home_row_mod_left {
            compatible = "zmk,behavior-hold-tap";
            #binding-cells = <2>;
            flavor = "balanced";
            tapping-term-ms = <280>;
            require-prior-idle-ms = <150>;
            bindings = <&kp>, <&kp>;
            hold-trigger-key-positions = <KEYS_R THUMBS>;
            hold-trigger-on-release;
        };
        hmr: home_row_mod_right {
            compatible = "zmk,behavior-hold-tap";
            #binding-cells = <2>;
            flavor = "balanced";
            tapping-term-ms = <280>;
            require-prior-idle-ms = <150>;
            bindings = <&kp>, <&kp>;
            hold-trigger-key-positions = <KEYS_L THUMBS>;
            hold-trigger-on-release;
        };
    };
}
```

### Troubleshooting
> in case something does not feel right
- noticeable delay when tapping HRMs: increase `require-prior-idle-ms` (rule of thumb: 10500/(relaxed wpm for english prose))
- false negatives (same-hand): reduce `tapping-term-ms` (or disable `hold-trigger-key-positions`)
- false negatives (cross-hand): reduce `require-prior-idle-ms`
- false positives (same-hand): increase tapping-term-ms
- false positives (cross-hand): increase require-prior-idle-ms (or set flavor to `tap-preferred`, which requires holding HRMs past tapping term to activate)
- deactivate `quick-tap-ms` if you have problems with your layers (e.g. layer on *space* hold after pressing *space)

## Generate locally for debugging
- `keymap -c draw/config.yaml parse -z config/<kb>.keymap > <kb>_keymap.yaml`
- `keymap -c draw/config.yaml draw -d boards/shields/<kb>/<kb>.dtsi <kb>_keymap.yaml > <kb>_keymap.svg`
    - `-d` is only required if it's a custom shield

## Custom shields
- make sure to add a [`keys` property to your physical layout for key positions](https://zmk.dev/docs/development/hardware-integration/physical-layouts#optional-keys-property)
- see [draw-keymaps.yml](./.github/workflows/draw-keymaps.yml) for workflow setup for custom shields

## Known Issues
- [Using combos on the homerow can interfere with trying to use multiple modifiers](https://github.com/zmkfirmware/zmk/issues/544)

## References
- zmk-config
    - [urob - zmk-config](https://github.com/urob/zmk-config)
    - [caksoylar - zmk-config](https://github.com/caksoylar/zmk-config)
- zmk
    - [ZMK - Own Keyboard Shield](https://zmk.dev/docs/development/hardware-integration/new-shield?keyboard-type=split)
    - [caksoylar - Tool for ZMK physical layout conversion](https://zmk-physical-layout-converter.streamlit.app/)
    - [caksoylar - Keymap Drawer](https://github.com/caksoylar/keymap-drawer/tree/main)
- Layout
    - [manna-harbour - Miryoku](https://github.com/manna-harbour/miryoku_zmk)
    - [Original Home Row Mods Post](https://precondition.github.io/home-row-mods)
    - [urob - Timeless Homerow Mods](https://github.com/urob/zmk-config?tab=readme-ov-file#timeless-homerow-mods)
    - [ZMK - Hold Tap](https://zmk.dev/docs/keymaps/behaviors/hold-tap)
- Thumb Health
    - [Getreuer - Blog Post on Thumb Health with Examples](https://getreuer.info/posts/keyboards/thumb-ergo/index.html#countermeasures)
    - [Study - Smartphone Use - Hand Pain](https://pubmed.ncbi.nlm.nih.gov/39044247/)
