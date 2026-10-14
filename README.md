# Sentaurus 130 nm CMOS Process Flow

**An interactive, command-by-command walkthrough of a 130 nm bulk CMOS process in Synopsys Sentaurus Process (`sprocess`).**

The source material is a single Sentaurus Process command file — 490 lines of Tcl that take a bare
(100) silicon wafer through STI isolation, retrograde well formation, a 1.5 nm gate stack, halo and
extension implants, spacer formation, self-aligned CoSi₂, contact metallisation and finally a
device-ready mesh. The command file alone is hard to read as a *process*: the geometry is implicit,
the lithography steps refer to masks defined elsewhere, and roughly half the statements are
no-ops in any given domain because they belong to the complementary device.

This repository pairs that command file with an interactive page that steps through it one
operation at a time, drawing the wafer cross-section as it evolves beside the exact commands that
produced it.

> **Scope.** Personal study material for self-directed learning of the `sprocess` command language,
> built around the 130 nm reference flow used in NTU **EE4623 — CMOS Process & Device Simulation by
> CAD**. It is a reading aid for the command syntax and process sequence, not courseware, not a
> solution set, and not affiliated with Synopsys or NTU. See [Attribution](#attribution).

---

## The walkthrough

Open [`walkthrough/index.html`](walkthrough/index.html) in any browser. It is a single
self-contained file — no build step, no server, no dependencies to install.

**46 grouped operations across 11 process modules.** Each step shows:

- the wafer **cross-section after that operation**, drawn to scale in both directions from the
  etch, deposition and mask statements in the command file;
- the **verbatim command lines**, with a toggle that resolves `$variable` arithmetic to numbers
  (`rate= 1.5*$Lsp` → `0.0525 µm`);
- **what is physically happening and why** — why the STI etch is trapezoidal, why the halo implant
  needs four rotations, why the gate re-oxidation is a five-segment ramp;
- **implant overlays** with dose, energy, tilt and rotation, and **thermal sparklines** for every
  `temp_ramp` sequence;
- an optional **doping-region overlay** and **mesh refinement boxes**.

Arrow keys or the on-screen buttons step through; the left rail jumps between modules.

### Screenshots

<!-- Replace each placeholder below with a real capture. Drop the PNG into
     assets/screenshots/ using the filename already referenced here, and the
     image will appear with no further edits. Delete this comment afterwards.
     Capture guidance and naming rules: assets/screenshots/README.md -->

| | |
|---|---|
| ![Walkthrough — STI trench etch](assets/screenshots/walkthrough-sti-trench.png)<br>**STI trench etch** — trapezoidal profile, 87° sidewalls, nitride hard mask still in place | ![Walkthrough — gate stack after patterning](assets/screenshots/walkthrough-gate-stack.png)<br>**Gate patterning** — 130 nm poly gate on 1.5 nm oxide, resist still present |
| ![Walkthrough — halo implant overlay](assets/screenshots/walkthrough-halo-implant.png)<br>**Halo implant** — four rotations at 30° tilt, with the dose/energy overlay | ![Walkthrough — final mirrored device](assets/screenshots/walkthrough-device-final.png)<br>**Device structure** — mirrored about the gate centreline, contacts attached |

_Placeholders above; images land once captured._

### SDEVICE results

Electrical results from Sentaurus Device on the structure this flow produces. These are captured
separately and attached here as they become available.

<!-- Same mechanism: drop PNGs in assets/screenshots/ under these exact names. -->

| | |
|---|---|
| ![SDEVICE — Id–Vg transfer characteristic](assets/screenshots/sdevice-id-vg.png)<br>**Id–Vg** — subthreshold slope and V_T extraction | ![SDEVICE — Id–Vd output family](assets/screenshots/sdevice-id-vd.png)<br>**Id–Vd** — output characteristics and saturation behaviour |
| ![SDEVICE — electrostatic potential](assets/screenshots/sdevice-potential.png)<br>**Potential distribution** — channel and depletion region at bias | ![SDEVICE — electron current density](assets/screenshots/sdevice-current-density.png)<br>**Current density** — inversion channel under the gate |

_Not yet captured._

---

## Repository layout

```
.
├── walkthrough/
│   ├── index.html              # the interactive walkthrough — open this
│   └── README.md               # what it draws, and what it does not
├── sprocess/
│   ├── sprocess_fps.cmd        # the Sentaurus Process command file (507 lines)
│   └── README.md               # how to run it, and the parameters it expects
├── docs/
│   ├── 01-process-modules.md   # the 11 modules, with real line ranges
│   ├── 02-parameters.md        # every fset and Workbench parameter, decoded
│   └── 03-command-reference.md # the sprocess commands this flow uses
└── assets/
    └── screenshots/
        └── README.md           # capture guidance and naming convention
```

---

## The flow at a glance

| # | Module | Lines | What happens |
|---|---|---|---|
| 00 | Setup & substrate | 1–128 | Mask layout via `icwb`, design rules, parameter block, p− (100) wafer at 1×10¹⁵ cm⁻³, `AdvancedCalibration` |
| 01 | STI isolation | 130–170 | Pad oxide / nitride hard mask, active litho, 450 nm trapezoidal trench, liner oxidation, corner rounding, oxide fill |
| 02 | Well & V_T | 172–217 | Seven-step retrograde p-well chain, complementary n-well (blocked in NMOS), 1050 °C spike drive-in, STI divot etch |
| 03 | Gate oxide | 220–231 | 1.5 nm gate dielectric, then a 44-minute 740 °C densification |
| 04 | Poly gate | 232–248 | 160 nm poly deposition, the critical gate lithography, anisotropic etch, profile smoothing |
| 05 | Recess & re-ox | 249–267 | Gate oxide cleared from the S/D, 5 nm silicon recess at 45°, 5 nm poly re-oxidation |
| 06 | Halo & extension | 268–327 | Three nested refinement boxes, 4-rotation boron halo, 6 keV arsenic NLDD, complementary set, 1.5 s flash |
| 07 | Spacer & S/D | 328–372 | 35 nm nitride + 42 nm oxide spacer, anisotropic etch, 55 keV n⁺ HDD, activation anneal |
| 08 | Silicidation | 373–393 | Screen oxide strip, 8 nm Si consumption, self-aligned 20 nm CoSi₂, oxynitride liner, ILD fill |
| 09 | Contact & metal | 394–414 | Contact lithography, two-stage 82° tapered via etch, tungsten plug |
| 10 | Device structure | 415–490 | Truncate to 0.5 µm, strip back-end films, doping-adaptive re-mesh, electrodes, mirror to full MOSFET |

Line ranges refer to [`sprocess/sprocess_fps.cmd`](sprocess/sprocess_fps.cmd) as shipped here.
Full step-by-step breakdown: [`docs/01-process-modules.md`](docs/01-process-modules.md).

---

## Reading the command file

Three conventions make this flow much easier to follow once you know them.

**Only half the transistor is simulated.** The domain runs from the field/STI edge to `$Ymid`, the
gate centreline. Every profile is therefore a half-profile, and the very last operation mirrors it:

```tcl
exec tdx --mirr-tdr -Y -ren source=drain n@node@_half_fps n@node@_fps
```

The `-ren source=drain` renames the mirrored electrode, which is how one contact definition becomes
both source and drain.

**Half the statements are deliberately inert.** The flow is written once and run for either domain.
In the NMOS domain, `photo mask= pWELL_p` covers the entire simulated area, so the five phosphorus
n-well implants and the complementary PMOS halo/extension/HDD implants land entirely in resist. They
are not dead code — they are the PMOS flow, masked off. The walkthrough marks these steps explicitly
rather than letting them look like real doping.

**Two switches change what runs.** `fset FastMode 1` skips every `implant` and `diffuse`, giving
geometry in minutes with no doping — useful when debugging masks and etch sequences. `fset debug 1`
turns on the `WriteBND` boundary dumps after every operation, which is how you inspect the geometry
step by step inside Sentaurus itself.

See [`docs/02-parameters.md`](docs/02-parameters.md) for the full parameter table, and
[`docs/03-command-reference.md`](docs/03-command-reference.md) for the command vocabulary.

---

## Running it

The command file is designed to run as a Sentaurus Workbench tool node, not standalone — it
references Workbench parameters (`@Domain@`, `@Lgate@`, `@node@` and others) and a mask layout file
that is not included here. Setup details and the full list of what it expects:
[`sprocess/README.md`](sprocess/README.md).

---

## Attribution

The 130 nm reference flow is course material from NTU EE4623 (Lectures 3–4, Synopsys TCAD
Simulations). The command file is included for study of the `sprocess` command language. The
walkthrough page, the cross-section reconstruction and all documentation in `docs/` are my own work.

Synopsys, Sentaurus, Sentaurus Process and Sentaurus Workbench are trademarks of Synopsys, Inc. This
repository is not affiliated with, endorsed by, or supported by Synopsys or NTU. No Synopsys
software, licence material or documentation is redistributed here.

If you are taking this course: read the flow, don't copy the report.

## Licence

[MIT](LICENSE), covering the walkthrough page and the documentation. The command file is included
under the attribution above.
