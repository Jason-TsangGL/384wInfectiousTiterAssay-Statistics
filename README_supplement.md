# Supplemental README: Infectious Titer Assay Statistics

This repository accompanies the manuscript and provides Python tools to analyze replicate statistics from infectious titer assays. The code supports both 384‑well and 96‑well microplate formats.

## Repository Contents

- `ReplicateStatisticsCode.ipynb` – Jupyter notebook with all analysis and plotting functions.
- `Documentation.md` – standalone documentation describing function behavior and parameters.
- `96-384wReplicateStatistics.xlsx` – example workbook with two sheets (`0.5x384` and `0.5x96`).
- `README.md` – general usage instructions.

## Core Functions

1. **`load_and_prepare_data(file_path, sheet_name)`**
   - Reads a worksheet from the Excel file.
   - Drops the first column (assumed to be labels) and returns a tidy `DataFrame`.

2. **`get_titer_values(data, col_index, convert_to_titer=True, media_volume=18)`**
   - Extracts one dilution column by index.
   - Optionally converts raw counts to infectious titer (IFU/mL).

3. **`bootstrap_count_histograms(data, dilution_index, ...)`**
   - Performs bootstrap sampling for a chosen dilution column.
   - Produces histograms showing how the mean varies with replicate count.

4. **`plot_sderrorbars(file_path, sheet1, sheet2, ...)`**
   - Compares variability between 384‑well and 96‑well plates.
   - Bootstraps the mean and visualizes ±1 SD error bars.

## Typical Workflow

1. Prepare your spreadsheet so that dilution factors are columns and replicates are rows.
2. Load a sheet using `load_and_prepare_data`.
3. Use `bootstrap_count_histograms` to examine how mean counts change as replicates increase.
4. Use `plot_sderrorbars` to contrast 384‑well and 96‑well variability.

## Important Notes

- Dilution columns are referenced by **0‑based index**.
- Default media volumes are hardcoded: `16 µL` for 384‑well and `48 µL` for 96‑well data.
- The example in `plot_sderrorbars` uses three replicates for the 96‑well plate despite the documentation mentioning eight replicates.

## Further Learning

- Familiarity with Pandas and Matplotlib will aid in adapting the notebook.
- Review the statistical bootstrap approach to understand uncertainty estimation.
- Inspect how infectious titer values are calculated in `get_titer_values`.

