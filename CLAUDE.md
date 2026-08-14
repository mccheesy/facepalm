# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Facepalm** — a personal ZMK keymap, forked from [minusfive/knucklehead](https://github.com/minusfive/knucklehead).

The repo is mid-refactor: knucklehead targeted 42-key corne / corneish_zen; Facepalm targets the **38-key Totem** first, then Felix, Lily58, Keezyboost40, X.Tips X7s.

`README.md` is still Knucklehead's, unedited. Its **design rationale** is still the intent (mnemonic affordances, static key placement, non-stacking upper layers, smart-word behaviors, timer-less home row mods). Its **facts are stale** — `./knucklehead/` paths, `L1_*.dtsi` filenames, 42-key claims, corne links. Do not cite it for current file layout.

## Architecture

`config/<board>.keymap` is a shim — `#include "../facepalm/base.dtsi"` plus a matrix transform. All three board keymaps are identical today. Real content lives in `facepalm/`.

`facepalm/base.dtsi` is both the include root and the configuration surface. Include order matters: behaviors → macros → combos → alpha layers → `L2` → `Fn`.

**Layer indices are computed, not literal.** `ALT`/`GAME` slots are conditionally compiled, and `L2`/`Fn` shift by however many are enabled. Adding or removing a layer means editing the arithmetic in `base.dtsi` *and* every `layers = <...>` list in `combos.dtsi`.

**Alpha layers are interchangeable slots.** `colemak-dh.dtsi`, `colemak.dtsi`, `dvorak.dtsi`, `qwerty.dtsi`, `qwerty_no_hrm.dtsi` each read `display-name = DISPLAY_NAME`, which the *includer* defines — so one file compiles into whichever slot includes it. An alpha layer file must never set its own display name. Select via `L1_FILE`/`ALT_FILE`/`GAME_FILE` in `base.dtsi`, not by editing the layer files.

**Key positions are the 38-key Totem grid**: `0–9` top, `10–19` home, `20–31` bottom (`20`/`31` are the outer keys), `32–37` thumbs. Every layer file carries the numbering as box-drawing comments — keep them accurate when editing bindings, because `combos.dtsi`, `KEYS_L`/`KEYS_R`/`THUMBS`, and `hold-trigger-key-positions` are all written against those numbers. Porting to a board with a different key count means re-deriving all of them.

**Autoshift is a three-layer stack**, one named group per shift-pair:

- `mm_XX` (behaviors.dtsi) — mod-morph, unshifted/shifted pair.
- `ht_XX` (behaviors.dtsi) — hold-tap: tap = the mod-morph, hold = the shifted key.
- `as_XX` (macros.dtsi) — the macro bound in keymaps.

Adding a pair means adding all three with matching suffixes. `&as <KEY>` is the parameterized generic version; `as_XX` are the fixed pairs.

Combos go through the `COMBO(NAME, BINDINGS, KEYPOS, LAYERS, TERM, QUICKTAP)` macro at the top of `combos.dtsi`.

## Commands

Firmware builds run in GitHub Actions (`build.yml` → `zmkfirmware/zmk` `build-user-config`). No local build is wired up — a local `west build` needs a west workspace initialized first (`zmk/`, `zephyr/`, `modules/`, `.west/` are gitignored).

`build.yaml` selects which board/shield combos build; only Totem is uncommented.

Keymap SVG:

```zsh
mise install          # installs keymap-drawer (pipx, caksoylar main)
./scripts/draw.zsh    # defaults to totem; arg is a config/<name>.keymap basename
```

Writes `img/<name>.svg` and `img/<name>.yaml`. Needs zsh. `keymap-drawer/config.yaml` holds `raw_binding_map` entries that give custom behaviors readable legends — new `as_*`/`csl`/`cmo`-style behaviors need an entry there or they draw as raw binding text. `keymap-drawer/combos.yaml` supplies combos the parser cannot infer.

## Conventions

- Indentation is 2 spaces, spaces only, everywhere — enforced by `.editorconfig` and `.prettierrc.json`. Prettier has no DeviceTree parser, so `.dtsi`/`.keymap` rely on `.editorconfig` alone; never introduce a tab into them.
- Binding rows in layer files are a fixed 16-column grid aligned to the key-position comments above them. Pad with spaces so a cell that outgrows 16 columns is the only one that shifts.
- Behaviors copied from upstream keep their attribution comment (`urob`, `caksoylar`, `knucklehead`). Preserve them.
- `config/west.yml`: the remote named `caksoylar` actually points at `github.com/dhruvinsh` and no project uses it. Check the URL, not the name.
