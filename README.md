# Batch Statistical Process Control with PCA

**PCA-based batch monitoring for baker's yeast production using historical reference batches and current running batches.**

## Project Overview

This project builds a Batch Statistical Process Control (BSPC) workflow for baker's yeast production. The notebook uses historical reference batch data to train a PCA monitoring model, then projects today's running batches into the same PCA space to compare current behavior against the reference operating region.

The objective is not to forecast a future target variable. Instead, the goal is to summarize multivariate process behavior, visualize batch trajectories, and identify current batches that may deviate from historical reference behavior.

## Dataset

The notebook uses two Excel files:

| Dataset | File used in notebook | Purpose | Rows | Columns | Batches |
|---|---:|---|---:|---:|---:|
| Reference batch data | `bakers_yeast_reference_batches.xlsx` | Build the PCA monitoring model | 1,328 | 10 | 16 |
| Current batch data | `todays_batches.xlsx` | Project current batches into the reference PCA space | 166 | 10 | 2 |

The reference dataset contains the following columns:

| Column | Type shown in notebook | Role |
|---|---|---|
| `Primary ID` | integer | Identifier |
| `BatchID` | object | Batch identifier |
| `Time` | float | Time index / process variable |
| `Ethanol` | float | Process variable |
| `Temperature` | float | Process variable |
| `Molasses flow` | float | Process variable |
| `NH3 flow` | float | Process variable |
| `Air flow` | float | Process variable |
| `Level` | float | Process variable |
| `pH` | float | Process variable |

Each reference batch has the same duration and number of observations.

| Batch summary metric | Value |
|---|---:|
| Number of reference batches | 16 |
| Observations per reference batch | 83 |
| Average observations per batch | 83.0 |
| Batch duration per reference batch | 13.6667 |
| Average batch duration | 13.67 |

## Methodology

### 1. Data loading and validation

The notebook loads the reference and current batch datasets from Excel files. It removes index-like columns if present and validates that required identifier, time, and process variable columns exist before modeling.

This step makes the workflow more reliable because PCA depends on consistent feature columns across the reference and current datasets.

### 2. Exploratory data analysis

The process variables are plotted over time with one line per batch. This helps show whether the reference batches follow similar operating patterns and where batch-to-batch variation is most visible.

Notebook observations:

| Process variable | Notebook observation |
|---|---|
| Ethanol | Shows visible batch-to-batch differences |
| Molasses flow | Shows visible batch-to-batch differences |
| NH3 flow | Shows visible batch-to-batch differences |
| Temperature | Shows visible batch-to-batch differences |
| pH | Shows visible batch-to-batch differences |
| Air flow | Follows a repeatable ramp-and-plateau pattern for most batches, with some deviations |
| Level | Follows a consistent increasing trend, with final-level differences across batches |

These observations support PCA because the process contains multiple correlated variables whose combined behavior is easier to monitor in a lower-dimensional space.

### 3. Standardization before PCA

The notebook standardizes the selected process variables before fitting PCA. This is necessary because the variables are measured on different numerical scales; without scaling, variables with larger numeric ranges could dominate the PCA components.

The PCA feature set contains 8 variables:

| PCA feature columns |
|---|
| Time |
| Ethanol |
| Temperature |
| Molasses flow |
| NH3 flow |
| Air flow |
| Level |
| pH |

### 4. PCA model fitting

A 5-component PCA model is fit using the standardized reference batch data. The model is used to produce PCA scores, loadings, and explained variance values.

| Principal Component | Explained Variance Ratio | Explained Variance (%) | Cumulative Variance (%) |
|---|---:|---:|---:|
| PC1 | 0.528601 | 52.860131 | 52.860131 |
| PC2 | 0.256072 | 25.607245 | 78.467377 |
| PC3 | 0.103760 | 10.375975 | 88.843351 |
| PC4 | 0.064286 | 6.428613 | 95.271965 |
| PC5 | 0.023534 | 2.353388 | 97.625353 |

PC1 and PC2 together explain 78.467377% of the standardized process variation, making them useful for two-dimensional visualization. The first 4 components exceed 95% cumulative explained variance, while the 5-component model explains 97.625353% cumulatively.

### 5. Loadings interpretation

PCA loadings are extracted to understand which original process variables contribute to each principal component. The notebook uses these loadings in a PC1-PC2 biplot.

| Variable | PC1 | PC2 | PC3 | PC4 | PC5 |
|---|---:|---:|---:|---:|---:|
| Time | 0.471425 | 0.115005 | 0.073487 | 0.109928 | -0.166210 |
| Ethanol | -0.315056 | 0.164245 | 0.630171 | 0.587107 | -0.255200 |
| Temperature | 0.399243 | -0.203158 | -0.051128 | 0.540846 | 0.647797 |
| Molasses flow | 0.127401 | 0.608623 | -0.366748 | 0.045936 | -0.239382 |
| NH3 flow | -0.254209 | 0.542698 | 0.115348 | -0.249949 | 0.618487 |
| Air flow | 0.318021 | 0.481637 | 0.256120 | 0.079962 | 0.044048 |
| Level | 0.472402 | 0.064508 | -0.060503 | 0.001973 | -0.210803 |
| pH | 0.337563 | -0.132078 | 0.614657 | -0.528890 | 0.037395 |

In the biplot, longer arrows indicate stronger contribution within the PC1-PC2 plane. The notebook also notes that PC1-PC2 loadings do not represent the full PCA model because additional variance is captured by PC3, PC4, and PC5.

### 6. Batch score trajectories

The notebook creates PCA score trajectories by pivoting PC scores over time for each batch. This transforms each batch into a trajectory through PCA space.

The interpretation is visual: batches following similar PC1-PC2 paths are behaving similarly in the reduced PCA space, while separated trajectories may indicate process behavior that differs from the reference region.

### 7. Monitoring today's batches

Today's batches are transformed using the already fitted reference scaler and PCA model. The notebook explicitly uses the trained reference transformation instead of refitting PCA on today's data. This matters because refitting would create a different PCA coordinate system and would make comparison against the reference batches invalid.

Current batch summary:

| Current batch dataset metric | Value |
|---|---:|
| Rows | 166 |
| Columns | 10 |
| Number of current batches | 2 |
| Current batch IDs shown in notebook | Ya, Za |

First five transformed rows for batch `Ya`:

| Row | PC1 | PC2 | PC3 | PC4 | PC5 | BatchID | Time |
|---:|---:|---:|---:|---:|---:|---|---:|
| 0 | -3.096781 | -3.798958 | -1.166746 | -0.602916 | -0.937231 | Ya | 0.000000 |
| 1 | -3.715861 | -3.216763 | -2.529801 | 1.183846 | -0.839431 | Ya | 0.166667 |
| 2 | -3.928722 | -2.972255 | -1.411527 | 2.217525 | -1.037556 | Ya | 0.333333 |
| 3 | -4.362027 | -1.145315 | 0.230988 | 2.807162 | -1.130690 | Ya | 0.500000 |
| 4 | -4.476710 | -1.268811 | 2.409623 | 3.259502 | -1.559791 | Ya | 0.666667 |

### 8. Reference vs current batch interpretation

The final monitoring plot compares current batch trajectories with the historical reference trajectories. The notebook concludes that batch `Za` appears more consistent with reference behavior, while batch `Ya` shows a stronger deviation from the reference operating region.

| Batch | Notebook interpretation |
|---|---|
| Za | More consistent with reference batch behavior; still shows some deviation near parts of the trajectory |
| Ya | Stronger deviation; trajectory moves farther left of the reference region and remains separated for a noticeable portion of the run |

### 9. Cross-validation and forecasting decisions

Cross-validation, chronological train/test splitting, differencing, and model-family comparison are not used in this notebook. The notebook's model is unsupervised PCA for process monitoring, not a supervised forecasting model. Model adequacy is therefore assessed using explained variance, loadings, score trajectories, and visual comparison of current batches against the historical reference operating region.



## Dependencies

The notebook imports and uses:

| Dependency | Use in project |
|---|---|
| `pathlib` | Image folder and file path handling |
| `numpy` | Numerical calculations and cumulative variance |
| `pandas` | Excel loading, cleaning, grouping, pivoting, and tables |
| `matplotlib` | Process profile plots, PCA trajectory plots, and biplots |
| `IPython.display` | Notebook table display |
| `scikit-learn` | `StandardScaler` and `PCA` |


## Key Lessons / Takeaways

- PCA is useful for batch process monitoring because it compresses several process variables into a smaller number of components while preserving most of the standardized variation.
- Scaling is essential before PCA when process variables use different units and magnitudes.
- Current batches must be transformed using the reference scaler and PCA model; refitting PCA on current data would invalidate comparison with historical reference batches.
- PC1-PC2 visualizations are useful for interpretation, but they do not contain all variation. In this notebook, PC1 and PC2 explain 78.467377%, while 4 components are needed to exceed 95% cumulative variance.
- Visual PCA monitoring can highlight potentially abnormal batch behavior, but formal control limits such as Hotelling's T² and SPE/Q residuals would strengthen the monitoring system.

## References

- Uploaded notebook: `Batch_Lad_Kush_comments_replaced.ipynb` / exported HTML notebook.
- Dataset files used in the notebook: `bakers_yeast_reference_batches.xlsx` and `todays_batches.xlsx`.
- Python libraries used in the notebook: `pandas`, `numpy`, `matplotlib`, and `scikit-learn`.
- Method used in the notebook: Principal Component Analysis for Batch Statistical Process Control.

