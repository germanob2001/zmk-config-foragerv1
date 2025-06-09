# zmk-config
> zmk setup for my own keyboards

## Supported Keyboards
- [Fifi (own shield)](https://github.com/raychengy/fifi_split_keeb)
- [Corne Ish Zen](https://lowprokb.ca/products/corne-ish-zen)
- ~[Dactyl Manuform](https://github.com/abstracthat/dactyl-manuform)~ - WIP

## Homerow Mods (HRM)
> see an awesome explaination at [urob - zmk-config](https://github.com/urob/zmk-config?tab=readme-ov-file#timeless-homerow-mods)
- can lead to misfires on inconsistent typing speeds
- zmk's balanced flavor: produces `hold` if another key is pressed & released within `tapping-term` -> don't need to wait for `tapping-term`
- `require-prior-idle-ms`: solves typing delay by resolve HRM as `tap` when tapped shortly after another key has been tapped
- positional hold-tap: HRM always resolves as `tap` if next key is on same side of keyboard -> solves rolling keys
- `hold-trigger-on-release`: allow combination of modifiers on same hand with positional hold-tap
- `tapping-term`: don't set to infinity to be able to use modifier without another key or mod with alpha-key on same hand
- **shift**: shifting alphas may conflict with `require-prior-idle-ms` for fast typers -> works best with dedicated shift key (e.g. sticky shift on home thumb)

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
            quick-tap-ms = <175>;
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
            quick-tap-ms = <175>;
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

## Using combos instead of layers
> might be more comfortable instead of doing frequent layer switches (less thumb movements)
> currently working on integrating those if useful
- `require-prior-idle-ms` solves the issue of combo misfires, even on the home row
- principles:
    - should be easy to access and easy to remember
    - combos on left hand side that work well with right-handed mouse usage (cut, copy, paste, esc)
    - only use combos with multiple fingers (horizontal combos)

## Keymap
> keymap automatically drawn with [keymap-drawer](https://github.com/caksoylar/keymap-drawer)
![Keymap](./draw/fifi.svg?raw=true "Keymap")

### Generate locally for debugging
- `keymap -c draw/config.yaml parse -z config/<kb>.keymap > <kb>_keymap.yaml`
- `keymap -c draw/config.yaml draw -d boards/shields/<kb>/<kb>.dtsi <kb>_keymap.yaml > <kb>_keymap.svg`
    - `-d` is only required if it's a custom shield

### Custom shields
- make sure to add a [`keys` property to your physical layout for key positions](https://zmk.dev/docs/development/hardware-integration/physical-layouts#optional-keys-property)
- see [draw-keymaps.yml](./.github/workflows/draw-keymaps.yml) for workflow setup for custom shields

## References
- [urob - zmk-config](https://github.com/urob/zmk-config)
- [caksoylar - zmk-config](https://github.com/caksoylar/zmk-config)
- [ZMK - Own Keyboard Shield](https://zmk.dev/docs/development/hardware-integration/new-shield?keyboard-type=split)
- [manna-harbour - Miryoku](https://github.com/manna-harbour/miryoku_zmk)
- [Original Home Row Mods Post](https://precondition.github.io/home-row-mods)
- [caksoylar - Tool for ZMK physical layout conversion](https://zmk-physical-layout-converter.streamlit.app/)
- [caksoylar - Keymap Drawer](https://github.com/caksoylar/keymap-drawer/tree/main)
