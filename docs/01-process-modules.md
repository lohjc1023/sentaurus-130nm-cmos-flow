# Process modules, step by step

The 46 grouped operations the walkthrough steps through, with their line ranges in
[`../sprocess/sprocess_fps.cmd`](../sprocess/sprocess_fps.cmd).

A step is a *group* of command lines, not a single statement — `photo` / `etch` / `strip
Photoresist` is one lithography step, and the five or six `temp_ramp` lines that define one thermal
cycle are one anneal. Grouping is what makes the file readable as a process.

Steps marked **(blocked)** execute, but the implant lands entirely in photoresist because the mask
is the complementary polarity for this domain. They are the PMOS flow, masked off — not dead code.

---

## 00 · Setup & substrate — lines 1–128

| Lines | Step | What happens |
|---|---|---|
| 11–37 | Load the mask layout and pick the domain | `icwb` reads `CMOS130_lyt.mac`, slices out one named device, applies the gate stretch mark, and computes `$Ymid` — the gate centreline and mirror plane |
| 44–78 | Design rules and process parameters | One `fset` block that feeds every etch rate, deposition thickness and refinement box below |
| 99–128 | Build the starting wafer | p− (100) silicon at 1×10¹⁵ cm⁻³, `AdvancedCalibration`, stress history on, reflecting implant boundaries, global mesh strategy |

The stretch mark is the mechanism that makes gate length sweepable: the layout is drawn with a
0.060 µm gate and `icwb stretch` inserts the remaining `Lgate − 0.060`, so `@Lgate@` changes without
redrawing masks.

## 01 · STI isolation — lines 130–170

| Lines | Step | What happens |
|---|---|---|
| 131–132 | Pad oxide and nitride hard mask | 10 nm oxide, then 100 nm nitride |
| 134 | Active-area lithography | `photo mask= Active_n` — resist covers the future active areas |
| 137–138 | Pattern the nitride, strip resist | Nitride cut open in the field |
| 141 | Clear the pad oxide in the field | Bare silicon now exposed |
| 144 | Etch the shallow trench | `type= trapezoidal`, 450 nm deep, 87° walls |
| 147–153 | Liner oxidation | Five-minute ramp to 800 °C, ten-minute hold, five-minute ramp down, `stress.relax` |
| 155–156 | Strip the hard mask | Nitride, pad oxide and liner all removed |
| 159–160 | Round the trench corners | 5 nm isotropic etch then 5 nm isotropic deposition — net thickness unchanged, corners rounded |
| 163 | Fill the trench with oxide | `type= fill coord= -0.002` — planar, with 2 nm over the active area |
| 166–170 | Refine the mesh over the active area | `refinebox` tied to the `Active_p` mask, then `grid remesh` |

The etch-then-deposit pair at 159–160 is worth recognising: it appears again on the gate at 245–246.
It is the standard idiom for rounding a corner without changing a layer thickness.

## 02 · Well & V_T — lines 172–217

| Lines | Step | What happens |
|---|---|---|
| 173 | Open the p-well | `photo mask= pWELL_n` — in the NMOS domain this opens the whole simulated area |
| 177–183 | p-well and V_T chain implant | Seven boron implants, 250 keV down to 10 keV, building a retrograde profile in one lithography step |
| 186–189 | Close the p-well mask, open the n-well | Same layer, opposite polarity |
| 192–199 | n-well implant **(blocked)** | Five phosphorus implants, 300 → 20 keV, absorbed entirely by 1 µm of resist |
| 201–211 | Well drive-in — spike anneal | 250 °C/s up to 1050 °C, 20 s hold, −90 °C/s down: activate and repair, diffuse as little as possible |
| 214–215 | Divot etch | Removes the 2 nm oxide over the active area and recesses the STI ~9 nm, leaving a divot at the silicon edge |

The seven-implant chain is the point of this module. A single deep implant gives a Gaussian with its
peak in the wrong place; stacking seven doses from 250 keV to 10 keV builds a **retrograde** well —
low at the surface for mobility and V_T control, high below for punch-through and latch-up immunity
— all under one mask.

## 03 · Gate oxide — lines 220–231

| Lines | Step | What happens |
|---|---|---|
| 221 | Grow the gate oxide | 1.5 nm — roughly five atomic layers |
| 224–230 | Densify the gate oxide | 24-minute ramp to 740 °C, 20-minute hold, fast ramp down |

Note that the oxide is placed with `deposit … type= isotropic rate= $gox` rather than a `diffuse`
in O₂. The flow specifies the final thickness directly instead of solving the oxidation kinetics —
faster, and exact on thickness, at the cost of not reproducing the real consumption of silicon.

## 04 · Poly gate — lines 232–248

| Lines | Step | What happens |
|---|---|---|
| 233 | Deposit the polysilicon gate | 160 nm, conformal over the topography |
| 236 | Gate lithography — the critical layer | `photo mask= POLY_n`, resist 130 nm wide |
| 239–243 | Etch the gate | Anisotropic, `rate= $Tpoly+$gox` to clear poly and stop on oxide; resist stripped |
| 245–246 | Smooth the poly profile | Etch-and-replace again, this time `selective.materials= {Poly}` |

## 05 · Recess & re-ox — lines 249–267

| Lines | Step | What happens |
|---|---|---|
| 250–251 | Open the source/drain and recess the silicon | Gate oxide cleared, then 5 nm of silicon removed with a 45° trapezoidal ramp |
| 255–266 | Poly re-oxidation | 5 nm oxide, then a five-segment ramp peaking at 900 °C for 5 s |

The 45° ramp matters: a vertical recess step would put a field concentration right at the gate edge.
The re-oxidation then rounds the poly foot and repairs etch damage before the extension implant.

## 06 · Halo & extension — lines 268–327

| Lines | Step | What happens |
|---|---|---|
| 271–284 | Refine the mesh for the extension | Three nested `refinebox` statements: 1 nm laterally in the channel, 0.2 nm normal to the gate-oxide interface |
| 286–294 | Halo (pocket) implant — four rotations | Boron, 8×10¹² cm⁻² at 25 keV, tilt 30°, rotations 0/90/180/−90° |
| 297–301 | n-type S/D extension (NLDD) | Arsenic, 2.65×10¹⁵ cm⁻² at only 6 keV |
| 303–317 | PMOS halo and extension **(blocked)** | 4-rotation arsenic halo and 0.7 keV boron extension, absorbed by resist |
| 320–326 | Save the as-implanted structure, then flash anneal | `struct tdr=` dump, then 1.5 s at 1000 °C |

**Why four rotations.** At 30° tilt the beam is shadowed by the gate on one side. Firing from four
rotations 90° apart gives a symmetric pocket on both source and drain sides, and — because the
simulated domain is only half the device — keeps the half-profile correct under mirroring. Dropping
to a single rotation produces an asymmetric transistor.

**Why 6 keV.** The extension has to be shallow enough to limit short-channel effects but heavily
doped enough to keep series resistance down. 2.65×10¹⁵ cm⁻² at 6 keV is that compromise at 130 nm.

## 07 · Spacer & S/D — lines 328–372

| Lines | Step | What happens |
|---|---|---|
| 331–335 | Deposit the spacer films | 35 nm nitride, then 42 nm oxide (`1.2*$Lsp`), both conformal |
| 337–341 | Anisotropic spacer etch | `rate= 1.5*$Lsp`, `force.full.levelset` — clears horizontals, survives on the sidewalls |
| 344–352 | n⁺ deep source/drain implant | Arsenic, 5.5×10¹⁵ cm⁻² at 55 keV, now masked by the spacer as well as the gate |
| 354–370 | p⁺ S/D **(blocked)**, then activation | Boron absorbed by resist; one 1.5 s 1000 °C flash activates everything |

The spacer is what makes the LDD structure work: the extension was implanted with only the gate as a
mask, the deep S/D with gate *plus* spacer, so the lightly doped extension survives as a buffer
between channel and n⁺ region.

`force.full.levelset` is needed because the etch has to resolve the corner where the conformal film
meets the gate sidewall — the simplified solver gets the spacer foot wrong.

## 08 · Silicidation — lines 373–393

| Lines | Step | What happens |
|---|---|---|
| 375–379 | Pre-silicide clean | Screen oxide off, 8 nm of silicon consumed, exposing clean crystal; ramp from room temperature to 500 °C |
| 382–385 | Form the cobalt silicide | 20 nm CoSi₂, `selective.materials= {Silicon}` — self-aligned, no mask |
| 387–392 | Oxynitride liner and ILD fill | 50 nm liner in six steps, then oxide `type= fill` to −0.25 µm |

`selective.materials= {Silicon}` is the whole idea of a salicide: the cobalt reacts only where it
touches single-crystal silicon, so source and drain get contacted and the nitride spacer and STI
oxide stay bare. No lithography involved.

## 09 · Contact & metal — lines 394–414

| Lines | Step | What happens |
|---|---|---|
| 396 | Contact lithography | `photo mask= VIA1_p` |
| 399–405 | Etch the via — oxide, then the liner | Two trapezoidal etches at 82°, stopping on the silicide |
| 407–413 | Tungsten plug | `type= fill`, planarised flush with the ILD, then a `refinebox` across the plug interface |

The via is etched in two statements because the oxynitride liner is a different material: the first
etch clears the ILD and stops on the liner, the second removes the liner itself. One etch would
either stop short or punch through into the silicide.

## 10 · Device structure — lines 415–490

| Lines | Step | What happens |
|---|---|---|
| 418–426 | Truncate and strip for device simulation | `transform cut` to 0.5 µm, then `strip` of oxide, tungsten, aluminium and oxynitride |
| 430–462 | Rebuild the mesh for the device solver | `refinebox clear`, `line clear`, then a doping-adaptive mesh driven by `NetActive` |
| 465–482 | Dope the gate and define the contacts | Uniform 1.5×10²⁰ cm⁻³ n⁺ poly, then gate / source / substrate electrodes via `icwb.contact.mask` |
| 485–491 | Save and mirror into the full transistor | `struct tdr=`, then `tdx --mirr-tdr -Y -ren source=drain` |

The mesh swap is the step most people miss. A process mesh is refined where the *physics of
fabrication* needed resolution — implant tails, oxidation fronts, stress at corners. A device mesh
needs resolution where *carriers* move: the inversion layer, the junction depletion regions, the
drain field peak. So the process mesh is thrown away entirely and rebuilt adaptively on the
`NetActive` field.

`-ren source=drain` is the small detail that makes one electrode definition serve both terminals:
the half-structure has a single `source` contact, and the mirrored copy becomes `drain`.
