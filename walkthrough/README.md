# The walkthrough page

Open [`index.html`](index.html) in any browser. Single self-contained file — no build, no server, no
dependencies. Web fonts load from Google Fonts when online and fall back to system fonts when not.

## Controls

| | |
|---|---|
| `←` `→` | Previous / next step |
| `Home` `End` | First / last step |
| Left rail | Jump between the 11 modules; the bar shows progress within each |
| **resolved values** | Toggles `rate= 1.5*$Lsp` ⇄ `0.0525 µm` |
| **doping** | Overlays the implanted regions, revealed cumulatively |
| **mesh boxes** | Shows the active `refinebox` extents |

Your position and toggle states persist in `localStorage`, so closing and reopening resumes where
you left off.

## What it draws

- Cross-section **to scale in both directions**, about 1.2 × 0.85 µm of the device.
- Films thinner than ~6 nm — the 1.5 nm gate oxide, the 2 nm screen oxide, the STI liner — are drawn
  thicker so they stay visible. Real values always appear in the command panel.
- Implant overlays carry dose, energy, tilt and rotation; `temp_ramp` sequences get a thermal
  sparkline.
- The domain shown is the **full mirrored device**, even though the command file simulates only the
  half from the field edge to the gate centreline.

## What it is not

The geometry is **reconstructed** from the etch, deposition and mask statements in the command file —
it is not read back from a TDR file. A real level-set solution differs in detail, most visibly:

- at the trench and gate corners, where the rounding is idealised;
- at the STI divot, which is drawn from the etch arithmetic rather than solved;
- in the exact spacer foot profile, which depends on the full level-set solver.

Doping regions are **schematic** — drawn to show where each implant lands, not to represent a
computed concentration. For real profiles, read the `.plx` cuts the flow writes, or run with
`debug 1` and open the boundary dumps.

## Assumptions baked in

Since the layout file `CMOS130_lyt.mac` was unavailable, the page assumes:

- `@Domain@ = NMOS`, `@Lgate@ = 0.13 µm`
- `fp = 1.0`, `FastMode = 0`, `debug = 0` — so every implant and anneal runs, and the `WriteBND`
  dumps are skipped
- Active-area width and contact placement taken from the design rules in the parameter block
  (`IsoPitch 0.345`, `PolyPitch 0.32`)

These are stated on the page itself, in the header chip and the footer.

## Provenance

Built against [`../sprocess/sprocess_fps.cmd`](../sprocess/sprocess_fps.cmd) as shipped in this
repository. The line-number badges on each step refer to that file. If you edit the command file,
the badges will drift — they are not generated at load time.
