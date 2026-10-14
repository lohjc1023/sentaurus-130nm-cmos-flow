# Screenshots

Drop PNGs in this folder using the filenames below and they appear in the main
[README](../../README.md) automatically — the image tags are already in place.

## Expected filenames

### From the walkthrough page

| Filename | What to capture |
|---|---|
| `walkthrough-sti-trench.png` | Module 01, step "Etch the shallow trench" — trapezoidal profile with the nitride hard mask still on |
| `walkthrough-gate-stack.png` | Module 04, step "Etch the gate" — 130 nm poly on 1.5 nm oxide |
| `walkthrough-halo-implant.png` | Module 06, step "Halo (pocket) implant" — with the implant overlay showing dose, energy, tilt and rotations |
| `walkthrough-device-final.png` | Module 10, final step — the mirrored full transistor |

### From Sentaurus Device

| Filename | What to capture |
|---|---|
| `sdevice-id-vg.png` | Transfer characteristic, log scale, with the V_T extraction visible |
| `sdevice-id-vd.png` | Output characteristic family across gate biases |
| `sdevice-potential.png` | Electrostatic potential at bias, channel and depletion region |
| `sdevice-current-density.png` | Electron current density showing the inversion channel |

## Adding more

Any file beyond this list needs its own image tag in the README. Keep the naming pattern —
`<source>-<subject>.png`, lowercase, hyphenated — so the folder stays sortable as it grows.

## Capture notes

- **Full-window captures of the walkthrough**, not crops of the cross-section alone. The value is in
  seeing the drawing, the command panel and the explanation together.
- **Light theme** reads better on GitHub, which renders README images against a white or near-white
  background in either colour mode. The page follows your OS theme; override it in devtools with
  `data-theme="light"` on `<html>` if you need to force it.
- **Keep them under ~400 KB each.** These live in git history forever. PNG is right for UI and
  line-art plots; avoid JPEG for anything with text in it.
- **Widths around 1400–1600 px** look sharp on high-DPI screens without bloating the repo.
