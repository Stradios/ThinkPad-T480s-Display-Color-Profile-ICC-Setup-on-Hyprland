# ThinkPad T480s — Display Color Profile (ICC) Setup on Hyprland

## System info
- **Laptop**: Lenovo ThinkPad T480s (20L8002AMX)
- **Panel**: InfoVision Optoelectronics 0x057D (reported as `IVO057D` in neofetch, `0x057D` in `hyprctl monitors`)
- **Output name**: `eDP-1`
- **Native resolution**: 1920x1080 @ 60Hz
- **Correct scale**: `1.2`
- **Compositor**: Hyprland 0.56.2 (Wayland), CachyOS, Lua-based config (`hyprland.lua` + `config/*.lua` modules)

## Where the profile lives
Copy the ICM file(s) to a stable, permanent location — **do not** reference files in `~/Downloads`, since that folder gets cleaned out.

```bash
mkdir -p ~/.local/share/icc
cp TPLCD.ICM ~/.local/share/icc/TPLCD.ICM
```

Recommended: keep the **whole extracted bundle** here too, in case you want to try alternate profiles later:
```bash
mkdir -p ~/.local/share/icc/thinkpad-t480s-bundle
cp *.ICM *.icm ~/.local/share/icc/thinkpad-t480s-bundle/
```
(Original source: Lenovo's official "ThinkPad Monitor INF File" package for T480/T480s/X1 Carbon 6th Gen, extracted with `innoextract`. Note: this package does **not** include a profile made specifically for the IVO057D panel — `TPLCD.ICM` is Lenovo's generic ThinkPad LCD fallback profile, which happened to look correct/matching the old Windows install on this unit.)

## Where the config lives
File: `~/.config/hypr/config/monitors.lua`

**Important:** this CachyOS Hyprland setup uses the **Lua config API**, not classic hyprlang `monitor=...` syntax. The correct function is `hl.monitor({...})`, using `output` and `mode` as keys (not `name`/`resolution` — that structure silently fails with no error and falls back to auto-detected scale).

### Working config block
```lua
hl.monitor({
    output   = "eDP-1",
    mode     = "1920x1080@60",
    position = "0x0",
    scale    = "1.2",
    icc      = "/home/stradios/.local/share/icc/TPLCD.ICM",
})
```

Replace `/home/stradios/...` with your actual home path if this ever changes (e.g. reinstall with a different username).

## How to apply
```bash
hyprctl reload
hyprctl monitors
```

Check the output — confirm:
- `scale: 1.2`
- Color looks correct (matches the old Windows reference — not washed out/tinted)

## Troubleshooting notes (from setup)
- `hyprctl reload` returning `ok` does **not** guarantee the config was actually applied — a wrong table structure or key name can silently no-op instead of throwing a visible error.
- If scale reverts to something like `1.5` unexpectedly after editing `monitors.lua`, it usually means the whole `hl.monitor({...})` block failed to apply (wrong keys) and Hyprland fell back to auto-scale.
- Quick sanity check after any edit: always run `hyprctl monitors` and check `scale:` — don't rely on `hyprctl reload`'s "ok" alone.
- Keep a backup of the known-working file:
  ```bash
  cp ~/.config/hypr/config/monitors.lua ~/.config/hypr/config/monitors.lua.working-backup
  ```

## If you want to try other profile variants later
Other ICM files in the extracted bundle worth testing (swap the `icc` path and reload):
- `TPLCD95.icm`
- `TPLCD_40AE.icm`, `TPLCD_40AF.icm`, `TPLCD_40BE.icm`, `TPLCD_40BF.icm`
- `TPLCD_4124.icm`

None of these are calibrated specifically for the IVO057D panel (Lenovo's official package only maps `LEN4xxx`-branded hardware IDs, and IVO057D isn't among them) — `TPLCD.ICM` is simply the one that visually matched best in practice.

For a properly calibrated result (rather than "close enough"), the real fix is hardware calibration with a colorimeter (e.g. via **DisplayCAL** on Linux) — since no pre-made profile for this exact panel exists publicly.

## NAS backup
Store a copy of this file + the full ICM bundle on your NAS as the source of truth, in case `~/.local/share/icc` ever gets wiped on a reinstall.
