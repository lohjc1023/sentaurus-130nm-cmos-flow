# Command reference

The `sprocess` vocabulary this flow actually uses, in the order you meet it. Not a substitute for
the Sentaurus Process user guide — this is the subset that appears in
[`../sprocess/sprocess_fps.cmd`](../sprocess/sprocess_fps.cmd), with the idioms that make it
readable.

---

## Layout and domain

### `icwb`

Reads a mask layout and slices one device out of it.

```tcl
icwb filename= "CMOS130_lyt.mac" scale= 1e-3
icwb domain= "@Domain@"
icwb stretch name= "${Type}_L" value= @<(Lgate-0.060)>@
fset Ymin [icwb bbox left]
```

`scale= 1e-3` converts layout nanometres to simulation micrometres. The `bbox` queries return the
domain extents, which is how `$Ymid` gets computed. `icwb.create.all.masks` then materialises every
layer so later `photo mask=` and `refinebox mask=` statements can refer to them by name.

A **stretch mark** named in the layout lets a dimension be parameterised without redrawing masks —
here the gate is drawn at 0.060 µm and stretched by `Lgate − 0.060`.

### `icwb.contact.mask`

Attaches an electrode using a layout layer rather than hand-placed coordinates.

```tcl
icwb.contact.mask layer.name= "GATE" name= gate box PolySilicon xlo= ... xhi= ...
icwb.contact.mask layer.name= "SOURCE" name= source point CobaltSilicide x= $SilicideX replace
contact bottom name= substrate Silicon
```

`box` selects everything in a region; `point` places a single contact point. `contact bottom` is the
plain form, used for the substrate where no mask is needed.

---

## Structure and mesh

### `line` / `region` / `init`

Builds the starting wafer.

```tcl
line x location= 0.0 tag= top
line y location= $Ymid spacing= 100.0 tag= right
region Silicon substrate xlo= top xhi= bottom ylo= left yhi= right
init Silicon field= Boron concentration= 1.0e15 wafer.orient= {0 0 1} \
     flat.orient= {1 1 0} slice.angle= 45 !DelayFullD
```

`tag=` names a coordinate so `region` can refer to it symbolically. `wafer.orient` and `flat.orient`
matter for implant channelling and for stress; `slice.angle` sets the cross-section orientation
relative to the wafer flat.

### `refinebox`

Declares where the mesh should be fine. Declarative, not immediate — nothing happens until
`grid remesh`.

```tcl
refinebox name= HALO mask= Active_p \
  extend= 0.01 extrusion.min= -0.05 extrusion.max= 0.15 \
  xrefine= "0.004/$fp" yrefine= "$PolyPitch/(100.0*$fp)" zrefine= "..."
```

- `mask=` ties the box to a layout layer, so it follows the device rather than sitting at fixed
  coordinates — essential when gate length is being swept.
- `extrusion.min/max` are the depth limits; `extend` pads the lateral extent.
- `interface.materials= {Silicon Oxide}` with `min.normal.size` refines *normal to an interface*
  instead of filling a box — that is how the flow gets 0.2 nm resolution at the gate oxide without
  meshing the whole channel that finely.
- `refine.fields= {NetActive} … adaptive` makes refinement follow the doping gradient. Used only in
  the device re-mesh.

`refinebox clear` discards every box. The flow does this at line 446 before building the device mesh.

### `grid` / `transform`

```tcl
grid remesh
grid set.min.normal.size= $MinNormalSize/$fp set.normal.growth.ratio.2d= 2.0
transform cut min= {-10 -10 -10} max= {$Xbot 10 10}
```

`grid remesh` applies the accumulated `refinebox` declarations. `transform cut` truncates the
structure — the 10 µm process substrate becomes 0.5 µm for the device solver.

---

## Patterning

### `deposit`

```tcl
deposit material= {Nitride} type= anisotropic rate= {1.0} time= 0.1
deposit material= {Poly}    type= isotropic   thickness= $Tpoly
deposit material= {Oxide}   type= fill        coord= -0.002
deposit material= {CobaltSilicide} type= isotropic rate= $CoSi_thick time= 1.0 \
        temperature= 500 selective.materials= {Silicon}
```

Thickness is given either directly (`thickness=`) or as `rate × time`. The three types:

| Type | Behaviour | Used for |
|---|---|---|
| `anisotropic` | Adds only to horizontal surfaces | Pad oxide, hard mask |
| `isotropic` | Conformal — follows the topography | Poly, spacer films, liners |
| `fill` | Fills up to the given `coord` | STI oxide, ILD, tungsten plug |

`selective.materials=` restricts deposition to where it touches those materials. This is what makes
the silicide self-aligned.

### `etch`

```tcl
etch material= {Silicon} type= trapezoidal rate= $STI_depth time= 1.0 angle= $STI_tilt
etch material= {Oxide Nitride} type= anisotropic rate= {1.5*$Lsp 1.5*$Lsp} time= 2.0 \
     force.full.levelset
```

Same three types plus `trapezoidal`, which takes an `angle=` and produces a sloped sidewall — used
for both the STI trench (87°) and the contact via (82°). With multiple materials, `rate=` takes one
entry per material in the same order.

`force.full.levelset` switches to the full level-set solver for that etch. Slower, but needed where
the surface has a real corner to resolve — the spacer etch at line 337.

### `photo` / `strip`

```tcl
photo mask= Active_n thickness= 0.2
strip Photoresist
strip nitride
```

`photo` deposits patterned resist in one statement; the mask name comes from the `icwb` layout.
`strip` removes a material everywhere it is exposed.

**The mask polarity idiom.** `pWELL_n` and `pWELL_p` are the same layout layer at opposite polarity.
In the NMOS domain `pWELL_n` opens the simulated area and `pWELL_p` covers it completely — which is
how one file runs both devices, with the complementary implants landing harmlessly in resist.

### The etch-and-replace idiom

```tcl
etch    material= {Silicon} type= isotropic rate= {0.005} time= 1.0
deposit material= {Silicon} type= isotropic rate= {0.005} time= 1.0
```

Net thickness unchanged, corners rounded. Appears twice — on the trench corners (159–160) and on the
gate profile (245–246). Worth recognising, because it looks like a no-op until you think about what
isotropic removal and addition do to a sharp corner.

---

## Doping and thermal

### `implant`

```tcl
implant Boron dose= 8.0e12 energy= 25.0 tilt= 30 rotation= 0
implant Arsenic dose= 2.65e15 energy= 6.0 tilt= 0.0
```

Dose in cm⁻², energy in keV, angles in degrees. `tilt` is from the surface normal, `rotation` about
it. A tilted implant is shadowed by the gate, so a symmetric pocket needs four rotations 90° apart.

`pdbSet ImplantData ResistSkip 1` at line 134 lets the implant model skip the resist layer in its
range calculation; the `…Boundary Reflect` settings make the domain edges reflecting, which is
required for a half-device domain — without it, dose is lost at the boundary.

### `diffuse` / `temp_ramp`

```tcl
temp_ramp name= well time= 0.03  temp= 600.0  ramprate= 250.0
temp_ramp name= well time= 0.333 temp= 1050.0 ramprate= 0.0
temp_ramp name= well time= 0.083 temp= 1050.0 ramprate= -90.0
diffuse stress.relax temp_ramp= well
```

A named ramp is built from segments — `time` in minutes, `temp` the *starting* temperature of the
segment, `ramprate` in °C/s. Then one `diffuse` executes the whole named cycle.

Short cycles are written directly instead:

```tcl
diffuse time= 1.5<s> temp= 1000<C>
```

`stress.relax` lets the mechanics solver relieve accumulated stress during the thermal step. The
flow enables `pdbSet Mechanics StressHistory 1` at the top so stress is tracked across the entire
process, which is why `stress.relax` appears on nearly every `diffuse`.

### `select`

```tcl
select z= "1.5e20" name= NetActive store PolySilicon
```

Writes a field value into a material — here a uniform n⁺ gate doping, replacing the implanted poly
profile with a constant. `select z=` also defines expressions for extraction; `select z=1` followed
by `layers` is the standard idiom for reading out layer thicknesses.

---

## Output

```tcl
struct tdr= n@node@_asImplant
SetPlxList { BTotal AsTotal PTotal }
WritePlx n@node@_anneal.plx y= [expr $Ymid-($Lg2-0.01)]
LogFile "message"
```

`struct tdr=` dumps the full structure. `WritePlx` writes a 1D cut — `SetPlxList` chooses which
fields, and the `y=` expression here cuts 10 nm inside the gate edge, which is where the extension
profile matters. `LogFile` writes to the node log, and is the simplest route for getting an extracted
scalar out of a process run.

### The mirror

```tcl
if { [catch { exec tdx --mirr-tdr -Y -ren source=drain \
      n@node@_half_fps n@node@_fps } Err] != 0 } {
    LogFile $Err
}
```

`tdx` is an external utility, so the call is wrapped in `catch` — a failure is logged rather than
silently producing no output. `--mirr-tdr -Y` mirrors about the Y plane and `-ren source=drain`
renames the mirrored electrode, turning the simulated half into a complete two-terminal MOSFET.
