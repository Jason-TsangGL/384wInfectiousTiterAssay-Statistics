# 384wInfectiousTiterAssay-Statistics

## Overview
This function analyzes fluorescence response (FR) count data across various dilution factors from an Excel spreadsheet. It computes statistical summaries (mean, standard deviation, coefficient of variation), determines usable upper and lower FR count thresholds based on precision and linearity metrics, and produces a series of visual plots for interpretation.

Each block:
- Contains 16 replicate well counts  
- Is derived from dilution factors applied across multiple folds (e.g., √10× in this example)  
- Contributes to curve-based detection of usable FR count range  

---

## Parameters

| Parameter    | Type    | Default    | Description                                              |
|--------------|---------|------------|----------------------------------------------------------|
| `file_path`  | `str`   | _Required_ | Path to the Excel file containing FR count data.         |
| `sheet_name` | `str`   | _Required_ | Name of the Excel sheet to load data from.               |

---

## Functionality

### Data Handling
- Loads data from the Excel file with `header=None`, converting all entries to numeric.  
- Organizes the data into 16-column replicate blocks per fold.  
- Suppresses non-critical warnings.  

### Statistical Computation
- For each block (excluding the positive control at index 0):  
  - **Mean FR count**  
  - **Standard deviation**  
  - **Coefficient of variation (CV)** = std / mean  
- Constructs a dilution factor (DF) series as  
- DF = base_start × (fold ** block_index)


### CV Normalization
- Scales all CV values into the [0, 1] range for visual and comparative analysis:
cv_norm = (cv – cv.min()) / (cv.max() – cv.min())

### Threshold Determination
- **Upper Limit:**  
Fit successive linear regressions (mean vs DF) until the R² drops below 0.99 (default).
  Method:
  - Incrementally fits a linear regression model to increasing subsets of the dilution series.
  - Stops once R² drops below 0.99.
  - Returns the last point where the model was still a good fit.
  Why:
  - A high R² indicates linear response; deviation marks the upper bound of reliable quantification. 
- **Lower Limit:**  
  Locate the DF where normalized CV is closest to 0.1 (default).
  - Method:
    - Finds the point where normalized CV is closest to 0.1.
    - Extracts corresponding mean and dilution factor.
  - Why:
    - High variability (CV > 0.1) implies data unreliability. 
    - Thus, CV = 0.1 is used as a lower limit of acceptable noise.
### Visualization
Produces three plots:
1. **Dilution vs Mean FR** (log-log scale, with upper limit line)  
2. **Dilution vs Normalized CV** (log-log scale, with CV = 0.1 line)  
3. **Normalized CV vs Mean FR** (log-y scale, with lower limit lines)  

All plots include axis labels, legends, and tight layout.

---

## Notes
- Assumes exactly 16 replicates per block.  
- You can adjust `r2_thresh` or `target_cv` by modifying the function defaults.  
- In this template, only one fold (`√10×`) is shown — extend `folds` list for more.

---

## Example

```python
file_path  = r"C:/Users/your_name/your_data.xlsx"
sheet_name = "Sheet2"

analyze_fr_data(file_path, sheet_name)
```

## `bootstrap_count_histograms` — Replicate-wise Bootstrap Histogram Visualizer

### Overview
This function performs bootstrapped sampling for a specific dilution factor **by column index** and visualizes the distribution of sample means for varying replicate sizes. Optional support is provided for:
- Converting raw counts to Infectious Titer (IFU/mL),
- Formatting results using scientific notation.

---

### Parameters

| Parameter             | Type               | Description |
|-----------------------|--------------------|-------------|
| `data`                | `pandas.DataFrame` | Cleaned input data with numeric dilution factors as columns |
| `dilution_index`      | `int`              | **Index** (0-based) of the dilution column to analyze (e.g., `0` for first dilution, `1` for second, etc.) |
| `n_bootstrap`         | `int`              | Number of bootstrap iterations |
| `replicates_range`    | `iterable`         | Range of replicate counts (e.g., `range(2, 17)`) |
| `convert_to_titer`    | `bool`             | If `True`, converts values using `(count × dilution × (1000 / media_volume))` |
| `media_volume`        | `float`            | Volume in µL used for titer conversion (default: 48 for 96-well) |
| `sci_notation`        | `bool`             | Whether to display mean values and labels in scientific notation |

---

### Functionality

1. **Data Validation**  
   Verifies the specified `dilution_index` is within the DataFrame's columns.

2. **Optional Titer Conversion**  
   Applies the titer formula if enabled:  
   `titer = count × dilution × (1000 / media_volume)`

3. **Bootstrapping & Plotting**  
   - For each replicate count `n`, samples the data `n_bootstrap` times with replacement.
   - Computes and plots the distribution (histogram) of the bootstrapped means.
   - Annotates the mean and ±1 SD with vertical lines.
   - Customizes plot layout and format based on `sci_notation`.

---

### Output

- Matplotlib grid of histograms across all specified replicate sizes.
- Visual inspection of variability in sample means.
- Optionally supports scientific notation and IFU/mL outputs.

---

### Example

```python
bootstrap_count_histograms(
    data,
    dilution_index=2,      # Use the third dilution column (0-based index)
    n_bootstrap=10000,
    replicates_range=range(2, 6),
    convert_to_titer=True,
    media_volume=48,
    sci_notation=True
)
```

## 📊 `plot_sderrorbars` — Visualize Bootstrap ±1 SD Error Bars of Mean Distribution based on number of replicates

### Overview
This function performs statistical bootstrapping on experimental data from two microplate formats (384-well and 96-well), and visualizes the variability as ±1 standard deviation error bars across replicate sizes. It supports conversion to infectious titers, scientific notation formatting, and prints a summary table to the console as a CSV-formatted table.

---

### Parameters

| Parameter             | Type           | Description |
|------------------------|----------------|-------------|
| `file_path`           | `str`          | Path to the Excel file containing the dataset. Ensure the file is **closed** before execution. |
| `sheet1`              | `str`          | Sheet name for the **384-well** microplate (e.g., `"3xA"`). |
| `sheet2`              | `str`          | Sheet name for the **96-well** microplate (e.g., `"3x96"`). |
| `col`                 | `int or float` | Dilution factor (used as a column header in the Excel sheets). |
| `n_bootstrap`         | `int`          | Number of bootstrap iterations to perform (default: `10000`). |
| `convert_to_titer`    | `bool`         | Whether to convert raw data to **infectious titer** using the formula: <br> `(FR Count × Dilution Factor × (1000 / media volume))` |
| `sci_notation`        | `bool`         | Toggle for scientific notation in axis formatting. |
| `csv_output_path`     | `str`          | **[Currently Ignored]** Was used to save output CSV file — now results are only printed to console. |

---

### Supporting Functions

#### `load_and_prepare_data(file_path, sheet_name)`
- Loads a sheet from the Excel file.
- Drops the first column (assumed to be an index or non-relevant label).
- Returns a cleaned `pandas.DataFrame`.

#### `get_titer_values(data, col, convert_to_titer=True, media_volume=16)`
- Extracts a specific dilution column and optionally converts values to infectious titers.
- Raises `KeyError` if the dilution factor is missing.
- Uses `media_volume=16` for 384-well data and `media_volume=48` for 96-well data.

---

### Function Logic

1. **Data Loading**  
   Loads and cleans data from both 384-well and 96-well sheets, extracting the selected dilution factor.

2. **Optional Titer Conversion**  
   Converts to infectious titer if `convert_to_titer=True`.

3. **Bootstrapping (384-well)**  
   - Iterates from 2 to 16 replicates.
   - For each `n`, samples `n` values with replacement and computes the mean for each of `n_bootstrap` iterations.
   - Calculates the mean and standard deviation of these bootstrap samples.
   - Plots error bars and stores results.

4. **Bootstrapping (96-well)**  
   - Fixed at `n=8` replicates.
   - Same bootstrap approach, plotted as ±1 SD reference lines.

5. **CSV Summary Output**  
   - Summary statistics (mean and SD per replicate) are collected in a table.
   - Instead of writing to disk, the results are printed as a DataFrame to the console.

6. **Plotting**  
   - Displays a line chart with error bars and optional scientific notation.
   - Includes labels, legend, and grid formatting.

---

### Output

- 📊 **Plot**: Matplotlib chart with error bars across replicate sizes for 384-well data and horizontal SD references for 96-well data.
- 🖨️ **Console Table**: Printed summary of means and standard deviations for each plate type.
- ❌ **No File Saved**: CSV output is printed, not saved to disk.

---

### Notes

- Ensure that the dilution factor column (`col`) exists and is numeric in both sheets.
- File must not be open in Excel when running the function.
- Hardcoded media volumes: `16 µL` for 384-well and `48 µL` for 96-well formats.