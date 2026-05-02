<div align="center">

# 🧬 Epigenetic Aging & Methylome Analysis
**Benchmarking aging clocks and processing WGBS data via Galaxy and Bio-learn**

[![Galaxy](https://img.shields.io/badge/Platform-Galaxy-blue)](https://usegalaxy.org/)
[![Bio-learn](https://img.shields.io/badge/Library-Bio--learn-green)](https://bio-learn.github.io/)
[![Python](https://img.shields.io/badge/Language-Python-yellow)](https://www.python.org/)

[Overview](#overview) · [Epigenetics](#what-is-epigenetics) · [WGBS Results](#wgbs) · [EPIC Array Results](#epic-array-bio-learn-1) · [Glossary](#glossary)

</div>

## Overview

In this repo, we explore epigenetics via a hands-on [Galaxy tutorial](https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/methylation-seq/tutorial.html) and then use [EPIC Array (bio-learn)](https://bio-learn.github.io/) to benchmark 8 epigenetic aging clocks across two EPIC array blood methylation datasets. Our analysis includes correlation matrices, age deviation heatmaps, predicted versus chronological age plots, MAE (Mean Absolute Error) comparisons, and predicted age distribution analyses.

## What is Epigenetics?

Nearly all cells in your body have the same DNA, yet a brain cell and a skin cell look and act completely different. That’s because they don’t use the same parts of the DNA.

Epigenetics is basically the system that controls which genes are turned on or off in different cells, without changing the DNA itself. It helps explain how one genome can produce many different cell types during development and how cells stay specialized over time.

It is also influenced by things like diet, stress, and toxins, which can subtly change how genes are used without rewriting the genetic code.

> Think how identical DNA twins can still develop noticeable biological differences over time because their epigenetic patterns drift with environment and lifestyle.

DNA methylation, histone modifications, and chromatin remodeling are all mechanisms of epigenetic gene regulation.

## Results

### WGBS
The Whole Genome Bisulfite Sequencing pipeline was applied to breast cancer and normal breast tissue samples from Lin et al. (2015), processed through the Galaxy Training Network methylation-seq tutorial. 

#### QC Result: The "Expected Fail"
During the Quality Control phase using FastQC/Falco, the **Per base sequence content** report typically returns a "Fail" status. This is expected and actually indicates a successful experiment. 

In a standard DNA sample, the distribution of A, C, T, and G is relatively even. However, in bisulfite-treated data, almost all unmethylated Cytosines (C) are converted to Thymines (T). This causes the "T" line (red) to spike to the top and the "C" line (blue) to drop to nearly zero. This imbalance triggers the "Fail" warning, but it confirms the chemical conversion was successful.

### EPIC Array (bio-learn)
Using the `bio-learn` library, we analyzed the following:

*   **Datasets:** **GSE40279** (Human Aging Rates Study) and **GSE41169** (Dutch Schizophrenia/Control Cohort).
*   **Clocks Benchmarked:** Horvathv1, Horvathv2 (Skin & Blood), Hannum, PhenoAge, GrimAgeV1, GrimAgeV2, DunedinPACE, and Zhang.

| Analysis Type | Description |
| --- | --- |
| **Correlation Matrix** | Shows how strongly different aging clocks agree with one another on biological age. |
| **Deviation Heatmap** | Visualizes the "Epigenetic Noise" or the difference between Predicted Age and Chronological Age. |
| **Prediction Plots** | Scatter plots comparing Chronological Age (X-axis) vs. Predicted Age (Y-axis) to evaluate clock accuracy. |

## Glossary

### What are Methylomes?
DNA is a giant sequence of letters (bases): **A, C, G, and T**. Sometimes, a tiny chemical "tag" called a **methyl group** sticks to the letter **C**.

- **Methylation:** The act of sticking that tag on the DNA.
    - Methylated genes are **MOSTLY** switched **OFF**.
    - Conversely, unmethylated genes are **MOSTLY** switched **ON**.
    - This is not a universal rule; it’s context-dependent.
- **Methylome:** The complete "map" of every single *tag* in a cell's DNA.
    - **1** = 100% methylation at this spot.
    - **0** = No methylation at this spot.
    - Methylation is often fractional, not binary.

**Visual Representation of a Methylome:**
```text
    DNA STRAND:      A --- C --- G --- T --- C --- A --- G
                           |                 |
    METHYL TAGS:        [Methyl]          [Methyl]
                           |                 |
    METHYLOME MAP:   (Pos 2: YES)      (Pos 5: YES)
```

**Example:** In a normal cell, only specific cytosines are methylated. In a **cancer cell**, the methylation pattern can change in two main ways:
- **Hypermethylation:** (Too many methyl tags) Important genes (like tumor suppressors) get shut OFF.
- **Hypomethylation:** (Too few methyl tags) Genes that should stay OFF become active.

### Differentially Methylated Regions (DMRs)
Differentially methylated regions are segments of DNA that show consistent and statistically significant differences in DNA methylation patterns between distinct biological samples, such as different tissues or disease states.

### CpG Sites
A CpG site is a specific location in DNA where a cytosine (C) is directly followed by a guanine (G), linked through a phosphate backbone (that’s what the “p” stands for).

> These sites are important because they are the primary places where DNA methylation occurs.

At a CpG site, the cytosine can receive a methyl group, which can influence whether nearby genes are active or silent. In this way, CpG sites function like regulatory checkpoints in the genome, helping control gene expression. When scientists study a methylome, they are essentially scanning across all CpG sites for hyper- or hypomethylation.

### "Hierarchical Clustering of Breast Cancer Methylomes"?
This means grouping breast cancer patients by how similar their DNA methylation patterns are.
- **Breast cancer methylomes:** DNA “tag maps” (methylation patterns) from different patients.
- **Hierarchical clustering:** A method that **builds a tree** by grouping the most similar patients first.

### EPIC Array Data Structure
This uses processed data where the hard work of finding tags is already done.
- **Input Type:** **Beta Values** (A table of numbers between 0 and 1).
- **0.0** = No methylation; **1.0** = 100% methylation.

| Sample ID | Age | CpG_Site_1 | CpG_Site_2 | CpG_Site_3 |
| --- | --- | --- | --- | --- |
| Patient_01 | 45 | 0.82 | 0.10 | 0.55 |
| Patient_02 | 62 | 0.79 | 0.12 | 0.60 |

### QC Results Explanation
DNA sequencing machines are "blind" to methylation. If a machine reads a DNA strand, it just sees **A, C, G, and T**. It cannot "see" the tiny chemical tags on the **C** letters.

> To fix this, we use a chemical trick called **Bisulfite Treatment** before we put the DNA into the machine.

#### Bisulfite Treatment Trick
Imagine you have two strands of DNA that look identical to a computer:
1. **Strand A (Normal):** `G - A - C - T`
2. **Strand B (Methylated):** `G - A - C* - T` (The `*` is the invisible methyl tag)

If we sequence them normally, the computer says they are both just `GACT`. **We lose the information.**

#### How Bisulfite Treatment Works:
We soak the DNA in Sodium Bisulfite. This chemical is "aggressive" toward **unprotected** C's.
- **Rule 1:** If a **C** is "naked" (unmethylated), the chemical attacks it and turns it into a **U** (which the sequencer reads as a **T**).
- **Rule 2:** If a **C** has a "shield" (a methyl tag), the chemical can't hurt it. It stays a **C**.

---

#### The Result
Look at what happens to those two strands after the treatment:
1. **Imagine this is the strand:** `G - A - C - T`
2. **Strand A (Normal):** The `C` had no shield. It becomes `T`.
    - **New Sequence:** `G - A - T - T`
3. **Strand B (Methylated):** The `C` had a shield. It stays `C`.
    - **New Sequence:** `G - A - C - T`

By comparing the treated sequence to the original, we can finally "see" the methylation. Since most C's in the body are unmethylated, the high concentration of T's in our results proves the treatment worked.

## Resources

*   [Galaxy Training Network: DNA Methylation data analysis](https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/methylation-seq/tutorial.html)
*   [Galaxy Training Network: Introduction to DNA Methylation](https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/introduction-dna-methylation/slides-plain.html)
*   [Biolearn Documentation](https://bio-learn.github.io/)
*   [Biolearn GEO Data Sources](https://bio-learn.github.io/data.html)
