TL;DR
This repo provides a reproducible notebook to simulate GRBs with the DC3 tools and a ready-made Tier-1 dataset you can use immediately for slides and quick analyses (spectra, CDS rings, sky maps). It’s designed to (1) visualize the CDS ring signature of GRBs vs background and (2) create initial training data for ML.

What’s in this repository

Grb Tier-1 Simulation Notebook.ipynb
End-to-end pipeline to:

fetch the imaging response and orientation (if missing),

define a small grid of GRB cases (short/long spectra × 3 fluxes × 2 sky positions),

run SourceInjector → produce COSI-like event files (.h5),

build HEALPix sky maps (*_sky_map.fits),

generate plots (spectra, CDS 2D, sky views, flux overlays),

write a manifest (CSV/JSON).

figs.zip
All ready-made PNG figures for slides:

*_spectrum.png — counts vs energy (log–log),

*_cds2d.png — CDS 2D ring maps,

*_sky_moll.png and *_sky_gnom_*.png — all-sky + zooms,

overlay_*.png and cds_COMPARE_*.png — flux overlays & 3-panel comparisons.

tier1_data.zip
The dataset produced by the notebook:

GRB_*.h5 — injected event lists,

GRB_*_sky_map.fits — HEALPix maps,

tier1_manifest.csv and tier1_manifest.json — the index of all cases.

Tip: keep figs.zip for quick visual review; use tier1_data.zip if you want to re-plot or run analyses.

Why we did this

Understand CDS patterns: GRBs form a ring in Compton Data Space; background does not.

Stress basic knobs: short vs long spectra, low/nominal/high flux, plane vs high-latitude.

Prepare ML data: generate consistent, labeled examples for an initial GRB vs background classifier.

What the notebook does (overview)

Setup: output folders, paths.

Fetch inputs if missing:

DC3 imaging response (.h5 inside an auto-unpacked zip),

DC3 orientation (.ori, March 2028 in this setup).

Define Band spectra presets:

S1, S2 ≈ long/softer; S4, S5 ≈ short/harder (α, β, xp set in a small dict).

Choose sky positions:

Galactic plane: (l,b)=(0°, 0°) and high-lat: (l,b)=(45°, +50°).

Build the grid of GRB cases:

durations: SHORT0p8, LONG60,

spectra: S1, S2, S4, S5,

flux: 0.1×, 1×, 10×,

sky: plane & high-lat.

Inject events with SourceInjector → one .h5 per case.

Convert to HEALPix → write *_sky_map.fits.

Plot spectra, CDS 2D, sky maps; make overlays & comparisons.

Write manifest (CSV/JSON) with all parameters and filenames.

What’s inside the dataset

After unzipping tier1_data.zip you’ll see:

tier1_data/
├─ GRB_*_sky_map.fits            # HEALPix maps
├─ GRB_*.h5                      # injected COSI-like event files
├─ tier1_manifest.csv
└─ tier1_manifest.json


Naming convention

GRB_<DURATION>_<SPEC>_<FLUX>_L<LLL>_B±<BB>.<ext>

DURATION → SHORT0p8 or LONG60

SPEC → S1, S2 (long/soft), S4, S5 (short/hard)

FLUX → F0p1 (0.1×), F1 (1×), F10 (10×), Fmin (≈background)

L<LLL> → Galactic longitude (deg, zero-padded)

B±<BB> → Galactic latitude (deg with sign)

Example:
GRB_LONG60_S2_F1_L000_B+00.h5 → long S2, nominal flux, at (l=0°, b=0°).

Quickstart (use the data immediately)

Open a sky map

import healpy as hp
import matplotlib.pyplot as plt

m = hp.read_map("tier1_data/GRB_LONG60_S2_F1_L000_B+00_sky_map.fits", verbose=False)
hp.mollview(m, coord="G", title="All-sky (Galactic)"); hp.graticule(); plt.show()


Plot a spectrum from an HDF5 case

from histpy import Histogram
import matplotlib.pyplot as plt

h = Histogram.open("tier1_data/GRB_LONG60_S2_F1_L000_B+00.h5")
spec = h.project("Em")  # the notebook includes robust helpers if names differ
spec.draw(); plt.xscale("log"); plt.yscale("log"); plt.show()

Reproduce the dataset (run the notebook)

Conda (recommended)

conda create -n cosi-grb python=3.10 -y
conda activate cosi-grb
pip install numpy matplotlib astropy h5py healpy histpy threeML cosipy

jupyter lab  # open: Grb Tier-1 Simulation Notebook.ipynb


The notebook auto-downloads the DC3 response/orientation if they’re not present.

What to look for in the figures

Flux scaling (overlay_*.png): counts rise from 0.1× → 1× → 10×.

CDS 2D ring (*_cds2d.png): clear ring for GRB; no ring for background → strong ML signal.

Short vs long: S4/S5 (short/hard) push more counts to higher energies; S1/S2 (long/soft) are softer.

Sky checks: brightest pixel near injected coords; plane vs high-lat behave as expected.

Suggested slides (from figs.zip)

overlay_LONG_S2_G1.png — flux overlay (long GRB on plane)

overlay_SHORT_S4_G5.png — flux overlay (short GRB, high-lat)

cds_COMPARE_LONG_S2_G1.png — CDS 2D low/nominal/high with shared color scale

One or two per-case images:

GRB_*_spectrum.png (spectra)

GRB_*_cds2d.png (CDS ring)

GRB_*_sky_moll.png / GRB_*_sky_gnom_*.png (sky views)

Next steps (roadmap)

Compute simple metrics per case: total counts, background, SNR, ring contrast.

Expand Ep/xp and add more sky positions (random all-sky).

Train a baseline GRB vs background classifier on CDS 2D images.

Standardize CDS cuts/binning and time profiles (short vs long light curves).

Notes

Please do not commit the raw DC3 response/orientation into the repo; the notebook fetches them on demand.

If you want collaborators to download the large dataset easily, attach tier1_data.zip to a GitHub Release.

All code and data here are for internal COSI/DC3 simulation/testing purposes.
