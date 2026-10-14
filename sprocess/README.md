# The command file

[`sprocess_fps.cmd`](sprocess_fps.cmd) — 490 lines of Tcl for Synopsys Sentaurus Process.

## What it needs to run

This is a **Sentaurus Workbench tool node**, not a standalone script. Handing it directly to
`sprocess` will fail, because two things are missing before substitution:

**1. Workbench parameters.** The `@…@` tokens are replaced by Workbench before `sprocess` sees the
file. Required:

| Parameter | Example | Notes |
|---|---|---|
| `Domain` | `NMOS` | Also accepts `PMOS`; selects masks and the device sliced from the layout |
| `Lgate` | `0.13` | In µm. Must be ≥ 0.060, since the stretch mark adds `Lgate − 0.060` |
| `node` | supplied by Workbench | Used to name every output file |

`@Poly@`, `@Ext@`, `@SD@`, `@Silicide@`, `@Contact@` and `@DeviceMesh@` are `#split` markers rather
than values — Workbench uses them as branch points in the run tree.

**2. The mask layout.** Line 11 reads `CMOS130_lyt.mac`, which is **not included here** — it is
course-supplied material. Every `photo mask=` and `refinebox mask=` statement resolves against it.
The layers the flow expects:

`Active_n`, `Active_p`, `pWELL_n`, `pWELL_p`, `POLY_n`, `VIA1_p`, `Mesh_EXT_p`, `GATE`, `SOURCE`

Without the layout file the flow cannot run at all. The walkthrough reconstructs the geometry from
the design rules in the parameter block (`IsoPitch 0.345`, `PolyPitch 0.32`) precisely because the
layout was unavailable.

## Outputs

| File | Written at | Contents |
|---|---|---|
| `n<node>_well.plx` | Line 206 | 1D boron/phosphorus cut at the gate centreline after well drive-in |
| `n<node>_asImplant_fps.tdr` | Line 320 | Full structure as-implanted, before the extension anneal |
| `n<node>_extimp.plx` | Line 318 | Implanted boron/arsenic 10 nm inside the gate edge |
| `n<node>_asImpHDD_fps.tdr` | Line 364 | Full structure after the HDD implant |
| `n<node>_anneal.plx` | Line 367 | Active boron/arsenic/phosphorus after activation |
| `n<node>_half_fps.tdr` | Line 485 | Device-ready half structure with electrodes |
| `n<node>_fps.tdr` | Line 488 | **The full mirrored MOSFET** — this is what SDevice consumes |
| `n<node>_<nn>.bnd` | On `debug 1` | Boundary dump after every operation |

## Two switches worth knowing

```tcl
fset FastMode 0    ;# 1 → skip every implant and diffuse; geometry in minutes, no doping
fset debug    0    ;# 1 → WriteBND boundary dump after every single operation
```

`FastMode 1` is the setting to use while debugging masks and etch sequences. `debug 1` gives you the
step-by-step geometry natively — the Sentaurus-side equivalent of the walkthrough page, and worth
running once if you have a licence, since the real level-set result differs from the reconstruction
at corners, at the STI divot and in the exact spacer profile.

`fset fp 1.0` is the global mesh multiplier; `fset fd 1.0` at line 434 is the separate one for the
device mesh. Full parameter table: [`../docs/02-parameters.md`](../docs/02-parameters.md).

## Reading it

Three things to know before the file makes sense:

1. **It is half a transistor.** The domain runs from the field edge to `$Ymid`, the gate centreline.
   The last command mirrors it.
2. **About half the implants are inert** in any given domain — they are the complementary device's,
   blocked by resist. See the mask polarity idiom in
   [`../docs/03-command-reference.md`](../docs/03-command-reference.md).
3. **`refinebox` is declarative.** Nothing meshes until `grid remesh`.

Command-by-command walkthrough: [`../walkthrough/index.html`](../walkthrough/index.html).
