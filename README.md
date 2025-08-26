# 🚀 COSI GRB — Tier-1 Simulations

> Compact, reproducible **COSI** GRB simulations to visualize the **CDS ring** signature and bootstrap first-pass **ML experiments**.

**Quick links:**
[▶️ Run the notebook](./Grb%20Tier-1%20Simulation%20Notebook.ipynb) ·
[⬇️ Download figures (PNG)](https://github.com/Mustafa-hub-maker/cosi-grb-tier1/raw/main/figs.zip) ·
[⬇️ Download dataset (HDF5/FITS + manifest)](https://github.com/Mustafa-hub-maker/cosi-grb-tier1/raw/main/tier1_data.zip)

---

## ✨ What’s in this repo

| File                                       | Purpose                                                                                                                                                                                                                     |                              |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| **`Grb Tier-1 Simulation Notebook.ipynb`** | End-to-end pipeline: fetch DC3 response/ORI (if missing), simulate GRBs with `SourceInjector`, generate **event files** (`.h5`), **HEALPix sky maps** (`.fits`), **plots** (spectra, CDS 2D, overlays), and a **manifest**. |                              |
| **`figs.zip`**                             | Slide-ready **PNG** images (spectra, **CDS 2D rings**, sky Mollweide & gnomonic zooms, overlay & 3-panel comparisons).                                                                                                      |                              |
| **`tier1_data.zip`**                       | The **Tier-1 dataset**: `GRB_*.h5` (COSI-like events), `GRB_*_sky_map.fits` (HEALPix), \`tier1\_manifest.csv                                                                                                                | json\` (index of all cases). |

> **Why this exists:** GRBs produce a **ring** in Compton Data Space (CDS). Background does **not**. This repo gives clean, labeled examples to **see it**, **explain it**, and **train** a baseline classifier.

---

## 🧪 What we simulate (Tier-1 grid)

* **Spectral families (Band):**

  * **S1 / S2** → *long/softer*
  * **S4 / S5** → *short/harder*
* **Durations:** `SHORT0p8` (≈0.8 s), `LONG60` (≈60 s)
* **Flux scales:** `F0p1` (0.1×), `F1` (1×), `F10` (10×)
* **Sky positions (Galactic):** **plane** (`l=0°, b=0°`) and **high-lat** (`l=45°, b=+50°`)

**File naming:** `GRB_<DURATION>_<SPEC>_<FLUX>_L<LLL>_B±<BB>.<ext>`
Example → `GRB_LONG60_S2_F1_L000_B+00.h5` (long, S2, nominal flux, on plane)

---

## 📦 What’s inside the ZIPs

### `tier1_data.zip`

```
tier1_data/
├─ GRB_*.h5                      # injected COSI-like event files
├─ GRB_*_sky_map.fits            # HEALPix sky maps
├─ tier1_manifest.csv
└─ tier1_manifest.json
```

### `figs.zip`

* `overlay_LONG_S2_G1.png` — spectra overlay (long GRB on plane)
* `overlay_SHORT_S4_G5.png` — spectra overlay (short GRB, high-lat)
* `cds_COMPARE_LONG_S2_G1.png` — CDS 2D low/nominal/high (shared color scale)
* Per-case images:

  * `GRB_*_spectrum.png` — counts vs energy (log–log)
  * `GRB_*_cds2d.png` — CDS 2D ring maps
  * `GRB_*_sky_moll.png`, `GRB_*_sky_gnom_*.png` — sky views (Galactic + Equatorial)

---

## 🔍 What to look for

* **Flux scaling:** counts rise cleanly **0.1× → 1× → 10×** (overlay PNGs).
* **GRB vs background:** GRBs show a **clear CDS ring**; background (≈Fmin) does **not**.
* **Short vs long:** S4/S5 (short/hard) push more counts to **higher energies** than S1/S2 (long/soft).
* **Sky sanity checks:** brightest pixel near injected coords; plane vs high-lat behave as expected.

---

## 🧰 Use the data immediately (no setup)

**View a sky map**

```python
import healpy as hp, matplotlib.pyplot as plt
m = hp.read_map("tier1_data/GRB_LONG60_S2_F1_L000_B+00_sky_map.fits", verbose=False)
hp.mollview(m, coord="G", title="All-sky (Galactic)"); hp.graticule(); plt.show()
```

**Plot a spectrum from an HDF5 case**

```python
from histpy import Histogram
import matplotlib.pyplot as plt
h = Histogram.open("tier1_data/GRB_LONG60_S2_F1_L000_B+00.h5")
spec = h.project("Em")          # the notebook includes robust helpers if names differ
spec.draw(); plt.xscale("log"); plt.yscale("log"); plt.show()
```

---

## ▶️ Reproduce the dataset (run the notebook)

> The notebook auto-downloads DC3 **response** and **orientation** if they’re not present.

**Conda (recommended)**

```bash
conda create -n cosi-grb python=3.10 -y
conda activate cosi-grb
pip install numpy matplotlib astropy h5py healpy histpy threeML cosipy

jupyter lab  # open: Grb Tier-1 Simulation Notebook.ipynb
```

<details>
<summary><b>Notebook flow (expanded)</b></summary>

1. Sets up I/O paths
2. **Fetches** imaging response & orientation (once)
3. Defines **Band** presets (S1/S2/S4/S5)
4. Chooses two **sky** positions (plane & high-lat)
5. Builds the **Tier-1 grid** (durations × spectra × flux × sky)
6. Runs **SourceInjector** → `.h5` per case
7. Converts to **HEALPix** → `*_sky_map.fits`
8. Produces **plots** (spectra, CDS 2D, sky, overlays/3-panel)
9. Saves **manifest** (`tier1_manifest.csv|json`)

</details>

---

## 🧭 Good slide set (grab from `figs.zip`)

1. `overlay_LONG_S2_G1.png` — flux overlay (long, plane)
2. `overlay_SHORT_S4_G5.png` — flux overlay (short, high-lat)
3. `cds_COMPARE_LONG_S2_G1.png` — CDS 2D (low/nom/high)
4. One **per-case** CDS ring (`*_cds2d.png`)
5. One **sky** panel (`*_sky_moll.png` + a gnomonic zoom)

---

## 📌 Notes

* Please don’t commit DC3 response/orientation to the repo; the notebook fetches them on demand.
* For large artifacts, keep them zipped (as here) or attach to a **GitHub Release**.

---

**Maintainer:** Ghulam Mustafa
**Thanks:** COSI collaboration; libraries — `cosipy`, `threeML`, `histpy`, `healpy`, `astropy`, `matplotlib`, `numpy`.
