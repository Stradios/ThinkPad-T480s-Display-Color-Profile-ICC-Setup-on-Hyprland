# ThinkPad T480s — Hyprland Display Color Profile (ICC) Setup

Fixes washed-out / incorrect display colors on the ThinkPad T480s's InfoVision `IVO057D` panel under Hyprland (Wayland), by loading a Lenovo-provided ICC profile via Hyprland's native per-monitor `icc` support (landed in Hyprland 0.55+).

> No pre-made ICC profile exists publicly for the exact `IVO057D` panel. This setup uses Lenovo's generic ThinkPad LCD fallback profile (`TPLCD.ICM`), which visually matches the color rendering of the factory Windows install on this unit. It is **not** a hardware-calibrated profile — see [Caveats](#caveats) below.

---

## System

| | |
|---|---|
| **Model** | Lenovo ThinkPad T480s (20L8002AMX) |
| **Panel** | InfoVision Optoelectronics `0x057D` (`IVO057D`) |
| **Output name** | `eDP-1` |
| **Native resolution** | 1920×1080 @ 60Hz |
| **Scale** | `1.2` |
| **Compositor** | [Hyprland](https://hypr.land) `0.56.2` (Wayland) |
| **Distro** | CachyOS, Lua-based Hyprland config (`hyprland.lua` + `config/*.lua`) |

---

## Setup

### 1. Get the ICC profile

The extracted profile bundle is included in this repo — no need to re-extract it yourself:

**[⬇️ Download `ThinkPad_ICM_profiles.zip`](https://github.com/Stradios/ThinkPad-T480s-Display-Color-Profile-ICC-Setup-on-Hyprland/blob/main/ThinkPad_ICM_profiles.zip)**

```bash
wget https://github.com/Stradios/ThinkPad-T480s-Display-Color-Profile-ICC-Setup-on-Hyprland/raw/main/ThinkPad_ICM_profiles.zip
unzip ThinkPad_ICM_profiles.zip
```

<details>
<summary>Where this came from (if you want to re-extract it yourself instead)</summary>

Lenovo ships this bundle of ICM profiles for various T480/T480s/X1 Carbon 6th Gen panel variants in their **ThinkPad Monitor INF File** package.

1. Download it from [Lenovo Support](https://support.lenovo.com) — search "ThinkPad Monitor INF File" for your model.
2. It's an Inno Setup installer (`.exe`). Extract it on Linux with [`innoextract`](https://constexpr.org/innoextract/):
   ```bash
   sudo pacman -S innoextract   # Arch/CachyOS
   innoextract ThinkPadMonitorFile.exe
   ```
3. You'll get a folder of `.ICM`/`.icm` files plus `TPLCD.inf` (the hardware-ID → profile mapping table).

</details>

> ⚠️ **Note:** Lenovo's `TPLCD.inf` only maps `LEN4xxx`-style hardware IDs. IVO-branded panels (`IVO057D`) are **not** listed. Windows wouldn't auto-match this panel either — the generic `TPLCD.ICM` fallback is what's used here.

### 2. Store the profile somewhere permanent

Don't reference files from `~/Downloads` — put them in a stable location:

```bash
mkdir -p ~/.local/share/icc
cp ThinkPad_ICM_profiles/TPLCD.ICM ~/.local/share/icc/TPLCD.ICM
```

Optionally keep the full extracted bundle too, in case you want to try alternate variants later:

```bash
cp -r ThinkPad_ICM_profiles ~/.local/share/icc/thinkpad-t480s-bundle
```

### 3. Configure Hyprland

This setup uses Hyprland's **Lua config API** (`hl.monitor({...})`), not the classic hyprlang `monitor=...` line.

File: `~/.config/hypr/config/monitors.lua`

```lua
hl.monitor({
    output   = "eDP-1",
    mode     = "1920x1080@60",
    position = "0x0",
    scale    = "1.2",
    icc      = "/home/YOUR_USERNAME/.local/share/icc/TPLCD.ICM",
})
```

Replace `YOUR_USERNAME` with your actual home directory owner.

### 4. Apply and verify

```bash
hyprctl reload
hyprctl monitors
```

Check the output for `eDP-1`:
- `scale: 1.2` ✅
- Colors look correct — no gray tint / wash-out

---

## ⚠️ Gotcha: `hyprctl reload` returning `ok` doesn't mean it worked

If you use the **wrong table structure** (e.g. `hl.config({ monitor = {{ name = ..., resolution = ... }} })` instead of `hl.monitor({ output = ..., mode = ... })`), Hyprland accepts the reload with no visible error — but silently no-ops the block and falls back to auto-detected scale.

**Always verify with `hyprctl monitors` after every edit**, don't trust `reload`'s "ok" alone.

Keep a backup of your known-working config:
```bash
cp ~/.config/hypr/config/monitors.lua ~/.config/hypr/config/monitors.lua.working-backup
```

---

## Alternate profiles to try

If `TPLCD.ICM` doesn't look right on your unit, other profiles in the same Lenovo bundle worth testing (swap the `icc` path, reload, compare):

- `TPLCD95.icm`
- `TPLCD_40AE.icm`
- `TPLCD_40AF.icm`
- `TPLCD_40BE.icm`
- `TPLCD_40BF.icm`
- `TPLCD_4124.icm`

## Caveats

- None of these profiles are calibrated specifically for the `IVO057D` panel — they're the closest available generic fallbacks from Lenovo.
- For a properly calibrated profile, use a hardware colorimeter with [DisplayCAL](https://displaycal.net/) (works on Linux) to build a profile matched to your exact panel.

## License

Profile files (`.ICM`) are © Lenovo Group Limited, redistributed here for personal backup/reference only — not for redistribution as a package.
