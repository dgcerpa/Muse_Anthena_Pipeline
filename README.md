# Muse Anthena S - Processing Pipeline

Preprocessing and spectral-analysis pipeline for 4-channel EEG recorded with the MUSE Athena S headband, built on EEGLAB (MATLAB) and R.

## Context

This code supports the EEG arm of the SAPIENS Project (Sedentary Activities and
Passive-to-Intense Effects on Neurocognitive States), which compares the acute
neurophysiological responses elicited by mentally active versus passive sedentary
tasks. Trial registration: ClinicalTrials.gov NCT07087236. A study-protocol
manuscript is in preparation.

The pipeline covers one subset of the study: 8 participants (IDs 01, 02, 03, 05,
06, 07, 08, 09) recorded across 9 conditions — Basal (resting baseline),
Documental, Leer (reading), Música, Podcast, Reality, Tetris, TikTok, and Maximal
(an N-back working-memory task). The MUSE Athena S records four channels mapped to
10-20 positions: TP9 (left temporal), AF7 (left frontal), AF8 (right frontal), and
TP10 (right temporal).

## What's in this repo

```
Muse_EEG/
├── Script_csv.m          # Primary preprocessing, from Muse Monitor .csv exports
├── Script_edf.m          # Fallback preprocessing, from .edf (used only for truncated .csv)
├── Spectra.m             # Power-spectrum extraction from an EEGLAB STUDY
├── Datos Filtrados/
│   └── All/              # Preprocessed .set datasets + EEGLAB STUDY (study_file.study)
├── Spectral Data/
│   ├── analisis ratio.R  # Theta/Alpha and Alpha/Theta band-ratio analysis
│   └── *.csv             # Per-participant and group spectral tables (pipeline output)
└── docs/
    └── pipeline.md       # Detailed per-step documentation for every script
```

Raw device recordings are **not** included in this repository — see [Data](#data).

## How to reproduce

The MATLAB scripts run interactively inside EEGLAB and must be edited to point at
your own input/output paths before running; they are not batch scripts.

1. **Preprocess each recording.** Open EEGLAB in MATLAB, then run `Script_csv.m`
   file by file (edit the `pop_musemonitor` input path and both `pop_saveset`
   output paths for each participant/task). Use `Script_edf.m` only when a `.csv`
   export is truncated. Output: one `.set` per recording in `Datos Filtrados/All/`.

2. **Build the STUDY and precompute spectra.** In EEGLAB: `Study > Create a STUDY
   set` over the `.set` files, then `Study > Precompute channel measures > Power
   Spectrum`. This produces `study_file.study` and the `.datspec` files.

3. **Export spectral tables.** Run the relevant cell of `Spectra.m` (edit the
   `pop_loadstudy` path). Sections export group- and participant-level power tables
   as CSV into `Spectral Data/`.

4. **Compute band ratios.** With the working directory set to `Spectral Data/`:

   ```r
   install.packages("dplyr")   # if not already installed
   source("analisis ratio.R")  # note the space in the filename
   ```

   Output: `mean_band_participants.csv` and `ThetaAlpha_ratios.csv`.

Step-by-step details for each script (filter settings, ASR parameters, event
handling, the per-section export logic in `Spectra.m`) are in
[docs/pipeline.md](docs/pipeline.md).

### Software versions

| Component | Version |
|-|-|
| MATLAB | R2024a |
| EEGLAB | 2025.1.0 |
| ERPLAB plugin (`pop_basicfilter`) | 12.01 — required only by `Script_edf.m` |
| Muse Monitor Import (`pop_musemonitor`) | bundled with EEGLAB |
| BIOSIG (`pop_biosig`) | bundled with EEGLAB |
| clean_rawdata (`pop_clean_rawdata`) | bundled with EEGLAB |
| R | with the `dplyr` package (record `sessionInfo()` for exact versions) |

## Data

Raw EEG recordings are from identifiable human participants and are **not**
distributed in this repository for privacy and research-ethics reasons. The scripts
expect Muse Monitor `.csv` exports (256 Hz) or `.edf` files as input. Preprocessed,
de-identified `.set` datasets and the derived spectral CSV tables are retained under
`Datos Filtrados/` and `Spectral Data/`. Access to raw data is governed by the
SAPIENS study protocol; contact the author for data-sharing requests.

## License

Recommended: MIT for the analysis code — see the LICENSE file (create from GitHub's
official template). Derived data tables, if released, are better licensed CC-BY-4.0.

## Author

Diego Garrido Cerpa — Viña del Mar, Chile.
