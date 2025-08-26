# 🚀 COSI GRB — Tier-1 Simulations
> A compact, reproducible set of **COSI** gamma-ray burst (GRB) simulations to study the **CDS ring** signature and prepare data for first-pass **ML experiments**.
---

## ✨ What you get

| File                                       | What it is                                                                                                                                                                                                         |                               |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------- |
| **`Grb Tier-1 Simulation Notebook.ipynb`** | End-to-end pipeline: fetch DC3 response/ORI (if missing), simulate GRBs with `SourceInjector`, produce **event files** (`.h5`), **sky maps** (`.fits`), **plots** (spectra, CDS 2D, overlays), and a **manifest**. |                               |
| **`figs.zip`**                             | Ready-to-use **PNG figures** for slides: spectra, **CDS 2D** ring maps, sky (Mollweide + zooms), plus **overlays/3-panel comparisons**.                                                                            |                               |
| **`tier1_data.zip`**                       | The complete **Tier-1 dataset**: `GRB_*.h5` (COSI-like events), `GRB_*_sky_map.fits` (HEALPix), and \`tier1\_manifest.(csv                                                                                         | json)\` (index of all cases). |

> **Why this exists:** GRBs produce a **ring** in Compton Data Space (CDS). Background does not. This repo gives you clean, labeled examples to **see it**, **explain it**, and **train a baseline classifier**.

---

## 🧪 What we simulate (Tier-1 grid)

* **Spectral families (Band):**

  * **S1 / S2** → *long/softer*
  * **S4 / S5** → *short/harder*
* **Durations:** `SHORT0p8` (≈0.8 s), `LONG60` (≈60 s)
* **Flux scales:** `F0p1` (0.1×), `F1` (1×), `F10` (10×)
* **Sky positions (Galactic):**

  * **Plane**: `l=0°, b=0°`
  * **High-lat**: `l=45°, b=+50°`

**File naming:**
`GRB_<DURATION>_<SPEC>_<FLUX>_L<LLL>_B±<BB>.<ext>`
Example → `GRB_LONG60_S2_F1_L000_B+00.h5` (long, S2, nominal flux, on plane)

---

## 📦 What’s inside the dataset

Unzip **`tier1_data.zip`** and you’ll see:

```
tier1_data/
├─ GRB_*.h5                      # injected COSI-like event files
├─ GRB_*_sky_map.fits            # HEALPix sky maps
├─ tier1_manifest.csv
└─ tier1_manifest.json
```

Unzip **`figs.zip`** for slide-ready images, e.g.:

* `overlay_LONG_S2_G1.png` — spectra overlay (long on plane)
* `overlay_SHORT_S4_G5.png` — spectra overlay (short, high-lat)
* `cds_COMPARE_LONG_S2_G1.png` — CDS 2D low/nominal/high (shared color scale)
* Per-case: `*_spectrum.png`, `*_cds2d.png`, `*_sky_moll.png`, `*_sky_gnom_*.png`

---

## 🔍 What to look for (key observations)

* **Flux scaling:** counts rise cleanly from **0.1× → 1× → 10×** (see overlay PNGs).
* **CDS ring vs background:** GRBs show a **clear ring** in `*_cds2d.png`; background (≈Fmin) does **not**.
* **Short vs long:** S4/S5 (short/hard) push more counts to **higher energies** than S1/S2 (long/soft).
* **Sky sanity-checks:** brightest pixel near injected coordinates; plane vs high-lat behave as expected.

---

## 🧰 Use the data immediately

> You don’t need to run the notebook to explore — the ZIPs are ready.

**Sky map (FITS) preview**

```python
import healpy as hp, matplotlib.pyplot as plt
m = hp.read_map("tier1_data/GRB_LONG60_S2_F1_L000_B+00_sky_map.fits", verbose=False)
hp.mollview(m, coord="G", title="All-sky (Galactic)"); hp.graticule(); plt.show()
```

**Spectrum from an HDF5 case**

```python
from histpy import Histogram
import matplotlib.pyplot as plt
h = Histogram.open("tier1_data/GRB_LONG60_S2_F1_L000_B+00.h5")
spec = h.project("Em")          # the notebook includes robust helpers if names differ
spec.draw(); plt.xscale("log"); plt.yscale("log"); plt.show()
```

---

## ▶️ Reproduce (run the notebook)

> The notebook auto-downloads DC3 **response** and **orientation** if they’re not present.

**Conda (recommended)**

```bash
conda create -n cosi-grb python=3.10 -y
conda activate cosi-grb
pip install numpy matplotlib astropy h5py healpy histpy threeML cosipy

# then open the notebook
jupyter lab  # or: jupyter notebook
```

<details>
<summary>What the notebook does (expanded)</summary>

1. Sets up I/O paths.
2. **Fetches** the imaging response & orientation (once).
3. Defines **Band** spectra presets (S1/S2/S4/S5).
4. Chooses two **sky** positions (plane & high-lat).
5. Builds the **Tier-1 grid** (durations × spectra × flux × sky).
6. Runs **SourceInjector** → writes one `.h5` per case.
7. Converts to **HEALPix** → writes `*_sky_map.fits`.
8. Produces **plots** (spectra, CDS 2D, sky, overlays/3-panel).
9. Saves a **manifest** (`tier1_manifest.csv|json`) with all parameters + filenames.

</details>

---

## 🧭 Why this repo is useful

* A compact, **explainable** GRB set for CDS pattern intuition.
* Ready-to-share **figures** for meetings.
* Clean, labeled examples to bootstrap **GRB vs background** ML.

---

## 📌 Notes

* Please **don’t commit** DC3 response/orientation into the repo (the notebook fetches them on demand).
* For large artifacts, keep them zipped (like here) or attach them to a **GitHub Release**.

---

## 🙏 Acknowledgements

COSI collaboration and tooling; libraries: `cosipy`, `threeML`, `histpy`, `healpy`, `astropy`, `matplotlib`, `numpy`.

---

### ✅ Suggested slide set (grab from `figs.zip`)

1. `overlay_LONG_S2_G1.png` — flux overlay (long, plane)
2. `overlay_SHORT_S4_G5.png` — flux overlay (short, high-lat)
3. `cds_COMPARE_LONG_S2_G1.png` — CDS 2D (low/nom/high)
4. One **per-case** CDS ring (`*_cds2d.png`)
5. One **sky** panel (`*_sky_moll.png` + a gnomonic zoom)

---

*Maintainer: Ghulam Mustafa*

