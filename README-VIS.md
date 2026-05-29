# Arcade-Robotron_MiSTer-VIS

vis_warp-enabled fork of [`MiSTer-devel/Arcade-Robotron_MiSTer`](https://github.com/MiSTer-devel/Arcade-Robotron_MiSTer).

**This is the first validated consumer core for vis_warp** — barrel-warp
working symmetrically on a real arcade game, on hardware. The framework
lives at the vis_warp repo (the Template_MiSTer-VIS fork); start there for
the architecture, the adoption pipeline (`ADOPTING-A-CORE.md`), and the
roadmap.

## What's different from upstream Robotron

- `sys/` carries the vis_warp framework files (`vis_warp.vhd`,
  `vis_warp_v2_wp.vhd`, `vis_warp_pkg_v2.vhd`, `vis_warp_luts_pkg.vhd`).
- `sys/sys_top.v` has the SITE C insertion under `` `ifdef MISTER_WARP ``:
  vis_warp sits before `VGA_scanlines`, warping the native source; the
  warped frame then flows through scanlines → ascal → HDMI.
- `Arcade-Robotron.qsf` defines `MISTER_WARP=1`.
- Dev-time warp defaults in `sys/vis_warp.vhd`: enabled, curvature k=2,
  bilinear on.

With `MISTER_WARP` unset, the build is bit-identical to upstream Robotron.

## Why Robotron works (where Galaga didn't)

Robotron is a **landscape** Williams game: `landscape=1` forces
`no_rotate=1`, so the `screen_rotate` DDR-framebuffer path stays dormant
and ascal reads its **live input** — exactly where SITE C vis_warp sits.
Vertical/rotated cores (Galaga) divert video through the framebuffer and
bypass the live input; Robotron doesn't. (Full rationale in the
framework's `design_vis_warp_constraints.md` / `ADOPTING-A-CORE.md`.)

The same core also runs Joust, Stargate, Bubbles, Splat, and Alien★ar —
all `landscape=1`, all warp-able from this one build. (Sinistar and
Playball are `landscape=0` = rotated = the bypassed class.)

## Status

| | |
|---|---|
| Compiles clean (MISTER_WARP=1) | ✅ |
| Symmetric barrel on hardware (native 4:3) | ✅ validated 2026-05-28 |
| Self-tuning sync-delay (no per-core constant) | ✅ |
| Pre-built `.rbf` released | ⏳ (build from source for now) |

**Known, NOT a vis_warp issue:**
- **Twin-stick controls** — mapping the right stick to fire is stock-core
  behaviour (vis_warp never touches input). Use the OSD **Control** option
  to route the second stick as fire, and map your right analog stick to
  the P2 directions in MiSTer's controller config. Same on vanilla
  Robotron.
- **HDMI**: leave the front-end **scandoubler OFF** — ascal scales the
  warped native frame. Scandoubler ON feeds vis_warp doubled lines it
  can't fully reach (resolution-gated bow). Native + ascal is the path.
- **Top-of-frame**: fully symmetric now (self-tuning sync-delay); no
  residual asymmetry on this core.

## How to use / build

1. Quartus 17.0.2 Lite → open `Arcade-Robotron.qpf` → compile.
2. Drop the `.rbf` from `output_files/` onto SD `_Arcade/cores/`.
3. Use the standard Robotron MRA; ROM from your own MAME set.
4. Load. Native warp shows symmetric. Dress via OSD: scaler filter →
   scanlines → shadowmask.

To disable the warp: remove `MISTER_WARP=1` from the `.qsf`, recompile →
bit-identical to upstream Robotron.

## Credits

Robotron / Williams core: original MiSTer-devel authors. vis_warp
framework: see the framework repo's credits. ROMs not distributed.

## License

GPL v2+ (matching upstream + the framework).
