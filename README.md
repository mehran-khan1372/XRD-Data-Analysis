# XRD Residual and Noise-Pattern Analysis of PGO Membranes

This repository contains Jupyter notebooks for comparing the X-ray diffraction (XRD) datasets of neat PVDF, PGO-0.3%, and PGO-0.5%, with a focus on the local point-to-point residual patterns in the PGO-0.3% and PGO-0.5% datasets.

The workflow separates each measured intensity profile into a smooth component and a residual component, compares the residual patterns numerically, and creates a **synthetic diagnostic dataset** for visualization and methodological exploration.

> **Important:** `PGO-0.3_synthetic_noise_pattern_from_0.5.txt` is a computational reconstruction, **not an experimental measurement** and **not the original PGO-0.3% dataset**. It must be clearly identified as synthetic wherever it is used.

## Repository contents

- `PGO-0.3_synthetic_noise_pattern_from_0.5.ipynb`  
  Reads the numerical datasets, aligns the PGO-0.5% intensity values to the PGO-0.3% 2θ grid, estimates smooth signal components, calculates residuals and their correlation, and creates a synthetic diagnostic dataset.

- `Plot_PVDF, 0.3, 0.3 synthetic, 0.5.ipynb`  
  Creates comparison plots for the original PVDF/PGO datasets and the synthetic diagnostic dataset.

- `PGO-0.3_synthetic_noise_pattern_from_0.5.txt`  
  Generated output from the first notebook. This file is created when that notebook is run; it is not an experimental dataset.

## Required input data

The notebooks expect the following numerical text files to be available in the same working directory as the notebooks:

- `Pure Polymer.txt`
- `PGO-0.3%.txt`
- `PGO-0.5%.txt`

Each file is expected to contain two columns:

1. `2θ` (diffraction angle)
2. `Intensity`

The notebooks use UTF-8 encoding for `Pure Polymer.txt` and `PGO-0.3%.txt`, and UTF-16 encoding for `PGO-0.5%.txt`, matching the current code. If your files use different encodings or filenames, update the corresponding `read_data(...)` calls.

**The input datasets are not included in this repository by default.** Add them only if you have permission to share them and can document their provenance. Do not describe exported numerical text files as original instrument files unless that is verified.

## Method overview

### 1. Align the angular grids

The PGO-0.3% and PGO-0.5% datasets use slightly different 2θ sampling grids. For comparison, the PGO-0.3% grid is used as the reference grid, and the PGO-0.5% intensities are interpolated onto it:

```python
y05_aligned = np.interp(x03, x05, y05)
```

This interpolation is performed for comparison and does not overwrite the input data.

### 2. Estimate the smooth component

Each intensity profile is represented conceptually as:

\[
Y(2\theta) = S(2\theta) + N(2\theta)
\]

where `Y` is the supplied intensity profile, `S` is an estimated smooth component, and `N` is the residual after subtracting that estimate.

A Savitzky–Golay filter is used with:

- `window_length = 21`
- `polyorder = 3`

```python
signal03 = savgol_filter(y03, window_length=21, polyorder=3)
signal05 = savgol_filter(y05_aligned, window_length=21, polyorder=3)

noise03 = y03 - signal03
noise05 = y05_aligned - signal05
```

The residuals are **operationally defined high-frequency components** under this selected filtering procedure. They should not automatically be interpreted as pure instrumental noise; they can also contain real fine-scale signal features and processing effects.

### 3. Compare residual magnitude and correlation

The notebook calculates the standard deviation of each residual series and the Pearson correlation between the aligned residual patterns.

For the analysis recorded with this workflow, the approximate values were:

| Metric | Approximate value |
|---|---:|
| PGO-0.3% residual standard deviation | 60.79 |
| PGO-0.5% residual standard deviation | 48.60 |
| Pearson correlation of residual patterns | 0.085 |
| Residual scaling factor, σ(0.3%)/σ(0.5%) | 1.251 |

These are results from a particular run and input files. Re-running the notebooks with different input data, software versions, or parameter choices may produce different values.

A correlation of approximately `0.085` indicates very weak linear, point-by-point correlation between the residual series under this specific alignment and filtering procedure. This result alone does **not** establish the authenticity, origin, or history of either dataset.

### 4. Create a synthetic diagnostic reconstruction

The first notebook scales the PGO-0.5% residual component to match the standard deviation of the PGO-0.3% residual component, then combines it with the estimated PGO-0.3% smooth component:

```python
noise_scale = std_noise03 / std_noise05
transferred_noise = noise_scale * noise05
y03_synthetic = signal03 + transferred_noise
```

The resulting synthetic file is saved as:

`PGO-0.3_synthetic_noise_pattern_from_0.5.txt`

This is a **computational diagnostic simulation** designed to explore the effect of combining an estimated smooth component with a rescaled residual pattern. It is not recovered experimental data, does not prove how any historical figure was produced, and must never be presented as the original PGO-0.3% measurement.

## How to run

1. Install Python 3 and Jupyter Notebook or JupyterLab.
2. Install the required packages:

   ```bash
   pip install numpy pandas matplotlib scipy jupyter
   ```

3. Place the three required input text files in the same directory as the notebooks.
4. Open and run `PGO-0.3_synthetic_noise_pattern_from_0.5.ipynb` from top to bottom. It creates the synthetic text file and the residual-analysis plots.
5. Run `Plot_PVDF, 0.3, 0.3 synthetic, 0.5.ipynb` to create the comparison plots. This notebook requires the synthetic output from step 4.

## Scope and limitations

- The analysis compares the supplied numerical datasets; it does not independently verify their provenance or whether they are the original instrument output.
- Interpolation is used to place the two datasets on a common angular grid. Interpolation can affect residual values, especially where sampling differs or extrapolation would be required.
- The smooth/residual decomposition depends on the Savitzky–Golay parameters. Other reasonable parameters may yield different residuals and correlations.
- A low residual correlation is evidence only about the residual patterns under the chosen procedure. It is not definitive proof of data authenticity or proof that copying, manipulation, or other issues did or did not occur.
- The synthetic reconstruction is for diagnostic and visualization purposes only.

## Reproducibility and responsible use

When using or adapting this workflow, please:

- retain the original input files unchanged;
- document the source and encoding of each dataset;
- report the filtering parameters and alignment method;
- label all synthetic data and plots explicitly;
- avoid presenting synthetic or processed data as experimental measurements; and
- report limitations alongside numerical results.

## Software

- Python
- NumPy
- pandas
- Matplotlib
- SciPy (`scipy.signal.savgol_filter`)
- Jupyter Notebook
