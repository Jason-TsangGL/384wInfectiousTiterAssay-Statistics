This repository provides Python tools and Jupyter notebooks for analyzing and visualizing replicate statistics from infectious titer assays performed in 384-well and 96-well plate formats. The code is designed to help researchers assess the variability and reliability of their measurements across different numbers of replicates and dilution factors.

---

## Features

- **Replicate-wise Bootstrapping:**  
  Visualize the distribution of sample means for any dilution factor (selected by column index) and replicate size using bootstrapped histograms.

- **Error Bar Visualization:**  
  Plot ±1 standard deviation error bars for bootstrapped means across replicate sizes, comparing 384-well and 96-well formats.

- **Flexible Titer Conversion:**  
  Optionally convert raw counts to infectious titer (IFU/mL) using user-specified media volumes.

- **Scientific Notation Support:**  
  Toggle scientific notation for clearer visualization of large or small values.

- **Console Summary Output:**  
  Print summary statistics (mean and SD) for each replicate size directly to the console.

Threshold Determination
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

## Getting Started

1. **Clone the Repository**
    ```sh
    git clone https://github.com/yourusername/384wInfectiousTiterAssay-Statistics.git
    cd 384wInfectiousTiterAssay-Statistics
    ```

2. **Install Requirements**
    ```sh
    pip install pandas numpy matplotlib openpyxl
    ```

3. **Prepare Your Data**
    - Place your Excel file (e.g., `96-384wReplicateStatistics.xlsx`) in the project directory.
    - Ensure each sheet contains dilution factors as column headers and replicate counts as rows.

4. **Run the Notebook**
    - Open `ReplicateStatisticsCode.ipynb` in VS Code or JupyterLab.
    - Follow the example cells to load your data and generate plots.

---

## Example Usage

```python
# Bootstrapped histogram for the third dilution column (index 2)
bootstrap_count_histograms(
    data,
    dilution_index=2,
    n_bootstrap=10000,
    replicates_range=range(2, 6),
    convert_to_titer=True,
    media_volume=48,
    sci_notation=True
)

# Error bar plot comparing 384-well and 96-well data
plot_sderrorbars(
    file_path="96-384wReplicateStatistics.xlsx",
    sheet1="0.5x384",
    sheet2="0.5x96",
    dilution_index=2,
    convert_to_titer=True
)
```

---

## Documentation

- Each function is documented in the notebook with parameter tables, usage examples, and output descriptions.
- See the markdown cells in `ReplicateStatisticsCode.ipynb` for detailed explanations.

---

## Notes

- **Dilution Selection:**  
  All functions now use **column index** (0-based) to select the dilution factor, making it easy to automate or iterate through dilutions.
- **Data Format:**  
  The first column of each sheet is assumed to be an index or label and is dropped automatically.
- **Excel File:**  
  Ensure your Excel file is **closed** before running the code to avoid read errors.

---

## License

MIT License

---

## Contact

For questions or suggestions, please open an issue or contact the repository maintainer.