# Multiple Biomarkers of Aging — EPIC Array (Biolearn)

## Overview

DNA methylation data from two public blood cohorts. Run 8 epigenetic aging clocks. Compare predictions across clocks and datasets.

Paper: Ying et al. (2023). *Biolearn, an open-source library for biomarkers of aging.* https://doi.org/10.1101/2023.12.02.569722

---

## Setup

Install the Biolearn library before running anything.

```python
!pip install biolearn -q
```

Import all dependencies.

```python
import matplotlib.pyplot as plt
import warnings
import os
import pandas as pd
import numpy as np

from biolearn.data_library import DataLibrary
from biolearn.model_gallery import ModelGallery
from biolearn.visualize import (
    plot_clock_correlation_matrix,   # Task 4
    plot_clock_deviation_heatmap,    # Task 5
    plot_age_prediction              # Task 6
)

warnings.filterwarnings('ignore')
os.makedirs('plots', exist_ok=True)
```

---

## Datasets

Two blood DNA methylation datasets from GEO, both with chronological age metadata.

| Dataset | Platform | Samples | Age Range | Sex Present |
|---------|----------|---------|-----------|-------------|
| GSE120307 | Illumina 450k | 34 | 19–54 years | Yes |
| GSE41169 | Illumina 450k | 95 | 18–65 years | Yes |

**GSE120307** is the dataset used in the official Biolearn documentation examples. Well validated, broad age range, used as the primary benchmark dataset.

**GSE41169** is a Dutch population cohort. Larger sample count. Both age and sex metadata present, making it compatible with all 8 selected clocks.

Load both datasets directly from GEO via Biolearn. No manual downloading needed.

```python
# Downloads and caches dataset from GEO automatically
print("Loading Dataset 1: GSE120307 ...")
data1 = DataLibrary().get("GSE120307").load()
print(f"Samples: {data1.metadata.shape[0]}, CpG sites: {data1.dnam.shape[0]}")
print(f"Age range: {data1.metadata['age'].min():.1f} to {data1.metadata['age'].max():.1f} years")

print("Loading Dataset 2: GSE41169 ...")
data2 = DataLibrary().get("GSE41169").load()
print(f"Samples: {data2.metadata.shape[0]}, CpG sites: {data2.dnam.shape[0]}")
print(f"Age range: {data2.metadata['age'].min():.1f} to {data2.metadata['age'].max():.1f} years")
```

---

## Aging Clocks

Eight clocks spanning three generations of epigenetic clock development.

| Clock | Year | Generation | Predicts | CpGs Used |
|-------|------|------------|----------|-----------|
| Horvathv1 | 2013 | 1st | Chronological age | 353 |
| Hannum | 2013 | 1st | Chronological age (blood) | 71 |
| Lin | 2016 | 1st | Chronological age (blood) | 99 |
| PhenoAge | 2018 | 2nd | Biological age via clinical biomarkers | 513 |
| Zhang_10 | 2019 | 2nd | Mortality risk | 10 |
| YingCausAge | 2022 | 2nd | Age via causally linked CpGs | — |
| YingDamAge | 2022 | 2nd | Biological damage accumulation | — |
| DunedinPACE | 2022 | 3rd | Pace of aging (years/year, not years) | — |

> **Note:** DunedinPACE outputs a rate, not an age in years. It is included in the correlation matrix but excluded from age prediction plots.

Load all 8 clocks from the Biolearn model gallery.

```python
clock_names = [
    "Horvathv1",    # Original pan-tissue clock
    "Hannum",       # Blood-specific 1st gen clock
    "PhenoAge",     # Trained on clinical biomarkers
    "DunedinPACE",  # Pace of aging — outputs rate, not years
    "Lin",          # Blood-specific, 99 CpGs
    "Zhang_10",     # Mortality clock, only 10 CpGs
    "YingCausAge",  # Causality-enriched clock
    "YingDamAge",   # Damage-accumulation clock
]

gallery = ModelGallery()
models = [gallery.get(name) for name in clock_names]  # Load all at once for visualize functions
print(f"Loaded {len(models)} aging clocks successfully.")
```

---

## Steps

### Task 4 — Correlation Matrix

Shows how strongly each pair of clocks agree across all samples. High correlation = clocks rank samples similarly. Low correlation = different biological signals captured.

```python
for i, data in enumerate([data1, data2], 1):
    ds_name = "GSE120307" if i == 1 else "GSE41169"

    # Compute pairwise Pearson correlations between all clock predictions
    plot_clock_correlation_matrix(models=models, data=data)
    plt.suptitle(f"Clock Correlation Matrix — {ds_name}", fontsize=13, y=1.02)
    plt.tight_layout()
    plt.savefig(f'plots/correlation_matrix_{ds_name}.png', bbox_inches='tight')
    plt.show()
```

### Task 5 — Age Deviation Heatmap

Deviation = predicted age minus chronological age. Red = clock thinks sample is older than it is. Blue = clock thinks sample is younger.

```python
for i, data in enumerate([data1, data2], 1):
    ds_name = "GSE120307" if i == 1 else "GSE41169"

    # Each cell = how many years off that clock is for that sample
    plot_clock_deviation_heatmap(models=models, data=data)
    plt.suptitle(f"Age Deviation Heatmap — {ds_name}", fontsize=13, y=1.02)
    plt.tight_layout()
    plt.savefig(f'plots/deviation_heatmap_{ds_name}.png', bbox_inches='tight')
    plt.show()
```

### Task 6 — Predictions vs Chronological Age

DunedinPACE excluded here — its output is a rate (years/year), not comparable to age in years on the same axis.

```python
# Separate model list without DunedinPACE for age prediction plots
age_clock_names = ["Horvathv1", "Hannum", "PhenoAge", "Lin", "Zhang_10", "YingCausAge", "YingDamAge"]
age_models = [gallery.get(name) for name in age_clock_names]

for i, data in enumerate([data1, data2], 1):
    ds_name = "GSE120307" if i == 1 else "GSE41169"

    # Each subplot = one clock; diagonal line = perfect prediction
    plot_age_prediction(models=age_models, data=data)
    plt.suptitle(f"Age Prediction vs Chronological Age — {ds_name}", fontsize=13, y=1.02)
    plt.tight_layout()
    plt.savefig(f'plots/age_prediction_{ds_name}.png', bbox_inches='tight')
    plt.show()
```

### Additional — MAE Comparison

Mean Absolute Error measures how many years off each clock is on average. Lower = better.

```python
mae_results = {}
for name, model in zip(age_clock_names, age_models):
    try:
        pred1 = model.predict(data1).iloc[:, 0]
        pred2 = model.predict(data2).iloc[:, 0]

        # Align predictions to metadata index before subtracting
        mae1 = np.mean(np.abs(pred1 - data1.metadata['age']))
        mae2 = np.mean(np.abs(pred2 - data2.metadata['age']))

        mae_results[name] = {'GSE120307': round(mae1, 2), 'GSE41169': round(mae2, 2)}
    except Exception as e:
        print(f"Error calculating MAE for {name}: {e}")

mae_df = pd.DataFrame(mae_results).T
mae_df.plot(kind='bar', figsize=(10, 5), colormap='Set2')
plt.title("Mean Absolute Error (MAE) Comparison")
plt.ylabel("Years")
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.savefig('plots/mae_comparison.png', bbox_inches='tight')
plt.show()
```

### Additional — Predicted Age Distribution

Shows spread of each clock's predictions. Red dashed line = mean actual chronological age of cohort.

```python
fig, axes = plt.subplots(1, 2, figsize=(14, 6))
for ax, data, title in zip(axes, [data1, data2], ["GSE120307", "GSE41169"]):
    preds = {name: model.predict(data).iloc[:, 0].values
             for name, model in zip(age_clock_names, age_models)}
    pd.DataFrame(preds).boxplot(ax=ax, rot=45)

    # Red line marks where clocks should center if unbiased
    ax.axhline(y=data.metadata['age'].mean(), color='red', linestyle='--', label='Mean Actual Age')
    ax.set_title(f"Predicted Age Distribution — {title}")
    ax.set_ylabel("Years")
    ax.legend()
plt.tight_layout()
plt.savefig('plots/predicted_age_distributions.png', bbox_inches='tight')
plt.show()
```

---

## Output

Running this notebook produces 6 plots saved to the `plots/` folder.

| File | What it Shows |
|------|---------------|
| `correlation_matrix_GSE120307.png` | Inter-clock agreement on dataset 1 |
| `correlation_matrix_GSE41169.png` | Inter-clock agreement on dataset 2 |
| `deviation_heatmap_GSE120307.png` | Per-sample age over/under-estimation on dataset 1 |
| `deviation_heatmap_GSE41169.png` | Per-sample age over/under-estimation on dataset 2 |
| `age_prediction_GSE120307.png` | Predicted vs actual age scatter plots on dataset 1 |
| `age_prediction_GSE41169.png` | Predicted vs actual age scatter plots on dataset 2 |
| `mae_comparison.png` | MAE bar chart across both datasets |
| `predicted_age_distributions.png` | Boxplots of clock predictions vs mean actual age |

---

## Reference

Ying et al. (2023). Biolearn, an open-source library for biomarkers of aging. *bioRxiv*.
https://doi.org/10.1101/2023.12.02.569722

Official Biolearn docs and tutorial: https://bio-learn.github.io/
