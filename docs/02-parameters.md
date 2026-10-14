# Parameters

Everything in the flow is written in terms of the `fset` block at lines 44–93, which is why the
command file reads as a recipe rather than a pile of magic numbers. Change a value here and every
etch rate, deposition thickness and refinement box downstream follows.

## Design rules — lines 44–46

| Variable | Value | Meaning |
|---|---|---|
| `PolyPitch` | 0.32 µm | Poly-to-poly pitch; also the denominator for lateral mesh refinement |
| `Tpoly` | 0.16 µm | Polysilicon gate thickness |
| `IsoPitch` | 0.345 µm | Active-area / isolation pitch — sets where the STI edge falls |

## Geometry and process — lines 52–83

| Variable | Value | Meaning |
|---|---|---|
| `Lg` | `@Lgate@` | Gate length, swept from Workbench. 0.13 µm nominal |
| `Lg2` | `@<Lgate/2.0>@` | Half gate length, used to place the `WritePlx` cutlines |
| `gox` | 0.0015 µm | Gate oxide — 1.5 nm |
| `PolyReox` | 0.005 µm | Poly re-oxidation thickness after gate etch |
| `SiRecess` | 0.005 µm | Source/drain silicon recess depth |
| `Lsp` | 0.035 µm | Spacer nitride thickness. The oxide is `1.2*Lsp`, the etch `1.5*Lsp` |
| `Tsub` | 10.0 µm | Substrate depth during processing — cut to 0.5 µm before device simulation |
| `Nat_oxide` | 0.002 µm | Native / screen oxide, deposited at 500 °C before the HDD implant |
| `Metal1` | 0.15 µm | ILD thickness; the fill goes to `-Metal1-0.1` = −0.25 µm |
| `Divot` | 0.01 µm | Target STI divot depth at the silicon edge |
| `STI_depth` | 0.45 µm | Shallow trench depth |
| `STI_tilt` | 87.0° | Trench sidewall angle from horizontal — near-vertical, slightly tapered |
| `VIA_tilt` | 82.0° | Contact via sidewall angle — more tapered, for fill |
| `CoSi_thick` | 0.020 µm | Cobalt silicide thickness |

## Simulation control — lines 88–93

| Variable | Value | Meaning |
|---|---|---|
| `fp` | 1.0 | **Global mesh multiplier.** Every refinement spacing is divided by it |
| `FastMode` | 0 | `1` skips every `implant` and `diffuse` — geometry only |
| `debug` | 0 | `1` enables the `WriteBND` boundary dump after every operation |
| `count` | 1 | Counter used by `WriteBND` to number its output files |
| `MinNormalSize` | 0.005 | Minimum element size normal to an interface |
| `NormalGrowthRatio` | 2.0 | How fast elements may grow away from an interface |

These three are the ones to reach for:

**`fp` — mesh density.** `fp = 2` halves element size everywhere at once; runtime climbs steeply,
roughly with the square of it in 2D. Raise it when doping profiles look under-resolved in the
extension or at the gate-oxide interface. Note that the device re-mesh at line 434 uses a *separate*
multiplier, `fd`, so process and device mesh density are tuned independently.

**`FastMode` — geometry-only runs.** With `FastMode 1` every implant and anneal is skipped, so the
run finishes in minutes with no doping at all. This is the right setting while debugging masks, etch
sequences and spacer profiles; switch it back to 0 once the geometry is right and you need physics.

**`debug` — step-by-step boundary dumps.** With `debug 1` the `WriteBND` procedure at lines 95–105
writes `n<node>_01.bnd`, `n<node>_02.bnd` … after each operation, which you can then open in
sequence. That is the Sentaurus-native equivalent of what the walkthrough page does, and it is worth
running once if you have a licence — the real level-set geometry differs from the reconstruction at
corners, at the divot and in the exact spacer profile.

## Workbench parameters

The file is a Workbench tool node, not a standalone script. These `@…@` tokens are substituted by
Sentaurus Workbench before `sprocess` ever sees the file:

| Token | Purpose |
|---|---|
| `@Domain@` | `NMOS` or `PMOS` — selects which device is sliced out of the layout and which masks open |
| `@Lgate@` | Gate length. Also appears as `@<(Lgate-0.060)>@` and `@<Lgate/2.0>@`, which are Workbench-evaluated expressions, not Tcl |
| `@node@` | The Workbench node number, used to name every output file |
| `@Poly@` `@Ext@` `@SD@` `@Silicide@` `@Contact@` `@DeviceMesh@` | `#split` points — where Workbench may branch the run tree so downstream variations reuse upstream results |

The `#split` markers are the reason the flow is ordered the way it is. A split at `@Ext@` means
every extension-implant variation reuses one shared STI/well/gate result instead of re-running it,
so a sweep over extension conditions costs a fraction of a full re-run per branch.

## Derived quantities

Values the flow computes rather than states, worth having to hand when reading the commands:

| Expression | Resolves to | Where |
|---|---|---|
| `$Ymid` | `($Ymax-$Ymin)/2.0` — the gate centreline | Line 34; the mirror plane for the whole flow |
| `$Tpoly+$gox` | 0.1615 µm | Gate etch depth, line 239 |
| `1.2*$Lsp` | 0.042 µm | Spacer oxide thickness, line 334 |
| `1.5*$Lsp` | 0.0525 µm | Spacer etch depth, line 337 |
| `(0.001+$Divot)*0.8` | 0.0088 µm | Anisotropic part of the divot etch, line 214 |
| `(0.001+$Divot)*0.2` | 0.0022 µm | Isotropic part of the divot etch, line 215 |
| `1.65*$Metal1` | 0.2475 µm | Via etch depth, lines 399 and 401 |
| `-$Metal1-0.1` | −0.25 µm | ILD top surface, line 391 |
| `$Xbot` | 0.5 µm | Depth the structure is truncated to, line 435 |

The walkthrough's **resolved values** toggle shows these inline beside each command, which is the
quickest way to read an unfamiliar statement.
