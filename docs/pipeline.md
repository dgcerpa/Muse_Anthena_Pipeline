# Pipeline details

Detailed documentation for each script in the Muse_EEG pipeline. See the
[README](../README.md) for the high-level overview and reproduction steps.

All MATLAB scripts run interactively inside EEGLAB and require manual editing of
input/output paths for each participant and task. Run them file by file.

## 1. `Script_csv.m` — primary preprocessing

Processes Muse Monitor `.csv` exports. This is the main workflow.

Steps:

1. **Import** — `pop_musemonitor`, sampling rate 256 Hz, `importall` on.
2. **Remove epoch baseline** — `pop_rmbase`.
3. **Extract events** — `pop_chanevent` reads events from the event channel
   (leading edge).
4. **Select channels** — keep the four EEG channels `eeg_1`–`eeg_4`.
5. **Filter** — 1–35 Hz FIR bandpass via `pop_eegfiltnew`.
6. **Trim (optional)** — a commented-out block trims the recording to a `MM:SS`
   window. Uncomment and set `tiempo_inicio` / `tiempo_fin` to use it.
7. **Artifact cleaning** — Artifact Subspace Reconstruction via
   `pop_clean_rawdata` (`BurstCriterion` = 20, `WindowCriterion` = 0.25, burst
   rejection on; flatline/channel/line-noise/highpass criteria off).
8. **Save** — writes the `.set` twice: to a per-task folder and to the combined
   `All/` folder used for the STUDY.

Before running, edit: the `pop_musemonitor` input path, and the two `pop_saveset`
`filename`/`filepath` arguments.

## 2. `Script_edf.m` — fallback preprocessing

Used **only** when a `.csv` export is incomplete (in this dataset, this occurred
for some TikTok recordings, where the `.csv` was truncated relative to the `.edf`).

Differences from `Script_csv.m`:

- Imports `.edf` with `pop_biosig` instead of `.csv`.
- **Resamples to 256 Hz** with `pop_resample`, since `.edf` sampling rate may
  differ.
- Renames channels to `eeg_1`–`eeg_4` manually.
- Applies **average reference** across channels (`pop_reref`).
- Uses a 2nd-order Butterworth 1–35 Hz bandpass via ERPLAB (`pop_basicfilter`)
  instead of `pop_eegfiltnew`.

All other steps (event extraction, ASR cleaning, saving) match `Script_csv.m`.

## 3. `Spectra.m` — spectral extraction

Operates on an EEGLAB STUDY. Prerequisites:

1. All `.set` files in a common folder (`Datos Filtrados/All/`).
2. A STUDY created via `Study > Create a STUDY set`, defining participants,
   conditions/tasks, and groups.
3. Channel power spectra precomputed via
   `Study > Precompute channel measures > Power Spectrum`.

The script extracts numeric values directly from the line objects produced by
`std_specplot`, because EEGLAB has no built-in export for these values. Sections
(separated by `%%`) export different tables:

| Section | Output | Description |
|-|-|-|
| 1 | `spectral_data_9_tasks.csv` | All available frequencies × 9 tasks (group level) |
| 2 | `spectral_data_1to20Hz.csv` | 1–20 Hz, original values (group level) |
| 3 | `spectral_data_1to20Hz_interpolated.csv` | Integer 1–20 Hz, linearly interpolated (group level) |
| 4 | `spectral_data_XX_1to20Hz.csv` | Per-participant, integer 1–20 Hz, interpolated |

Before running: set the `pop_loadstudy` filename/filepath. For section 4, confirm
`participant_ids` matches the IDs defined in the STUDY
(`{'01','02','03','05','06','07','08','09'}`).

## 4. `analisis ratio.R` — band ratios

Computes Theta/Alpha and Alpha/Theta power ratios from the per-participant CSVs
produced by `Spectra.m` section 4.

Frequency bands (by CSV row / integer frequency):

- **Theta:** 4–8 Hz (rows 4–8)
- **Alpha:** 8–12 Hz (rows 8–12)

Outputs:

- `mean_band_participants.csv` — mean Theta and Alpha power per participant and task.
- `ThetaAlpha_ratios.csv` — T/A and A/T ratios per participant and task.

Before running: place the `spectral_data_XX_1to20Hz.csv` files in the working
directory and confirm the `participantes` vector matches the subject IDs
(`c(1, 2, 3, 5, 6, 7, 8, 9)`). Note the filename contains a space (`analisis ratio.R`).
