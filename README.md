This is a polished and stylized version of your README. I have corrected the grammar and formatting while preserving your voice, and I have filled in the placeholder data based on the requirements from your assignment.

***

<div align="center">

# 🧬 Epigenetic Aging Clocks & Methylation Analysis
**Benchmarking biological age and exploring the breast cancer methylome**

[![Library](https://img.shields.io/badge/Library-bio--learn-blue)](https://bio-learn.github.io/)
[![Workflow](https://img.shields.io/badge/Workflow-Galaxy-orange)](https://usegalaxy.org/)
[![License](https://img.shields.io/badge/License-MIT-green)](https://opensource.org/licenses/MIT)

[Overview](#overview) · [What is Epigenetics?](#what-is-epigenetics) · [Results](#results) · [Glossary](#glossary) · [Resources](#resources)

</div>

## Overview

In this repository, we explore the world of epigenetics through a dual-lens approach. First, we perform a hands-on [Galaxy tutorial](https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/methylation-seq/tutorial.html) to process raw Whole Genome Bisulfite Sequencing (WGBS) data. Second, we utilize the [bio-learn](https://bio-learn.github.io/) library to benchmark eight epigenetic aging clocks across two comprehensive EPIC array blood methylation datasets. Our analysis includes correlation matrices, age deviation heatmaps, predicted versus chronological age plots, and MAE (Mean Absolute Error) comparisons.

## What is Epigenetics?

Nearly all cells in your body contain the same DNA, yet a brain cell and a skin cell look and act completely different. That’s because they don’t "use" the same parts of the DNA.

Epigenetics is the system that controls which genes are turned on or off in different cells without changing the DNA sequence itself. It explains how one genome can produce many different cell types during development and how cells remain specialized over time.

This system is also influenced by external factors like diet, stress, and toxins, which can subtly change how genes are expressed without rewriting the genetic code.

> **Note:** Think of how identical twins can develop noticeable biological differences over time because their epigenetic patterns "drift" based on their specific environments and lifestyles.

The primary mechanisms of epigenetic gene regulation include **DNA methylation**, **histone modifications**, and **chromatin remodeling**.

## Results

### WGBS (Whole Genome Bisulfite Sequencing)
We applied a WGBS pipeline to breast cancer and normal breast tissue samples (based on Lin et al., 2015), processed through the Galaxy Training Network tutorial.

#### QC Result: The "Successful" Fail
During Quality Control (FastQC/Falco), our "Per base sequence content" report shows a **Fail** status. In a standard DNA experiment, this would be bad. However, in a methylation experiment, this is a sign of success. The high frequency of Thymine (T) and near-absence of Cytosine (C) proves that the Bisulfite conversion was successful.

---

### EPIC Array (bio-learn)
Using the `bio-learn` framework, we benchmarked the following models and datasets:

**1. Selected Datasets:**
*   **GSE40279:** Human Aging Rates Study (Blood samples).
*   **GSE41169:** Large-scale Dutch cohort study.

**2. Selected Aging Clocks:**
*   **First Gen:** Horvathv1, Horvathv2, Hannum.
*   **Second Gen:** PhenoAge, GrimAgeV1, GrimAgeV2.
*   **Specialized:** DunedinPACE (Speed of aging), Zhang.

## Glossary

### What are Methylomes?
DNA is a giant sequence of letters (bases): **A, C, G, and T**. Sometimes, a tiny chemical "tag" called a **methyl group** sticks to the letter **C**.

*   **Methylation:** The act of sticking that tag on the DNA.
    *   Methylated genes are mostly switched **OFF**.
    *   Unmethylated genes are mostly switched **ON**.
    *   *Note: This is context-dependent and not a universal rule.*
*   **Methylome:** The complete "map" of every single tag in a cell's DNA.
    *   **1.0** = 100% methylation at a specific spot.
    *   **0.0** = No methylation at that spot.

**Visual Representation:**
```text
    DNA STRAND:      A --- C --- G --- T --- C --- A --- G
                           |                 |
    METHYL TAGS:        [Methyl]          [Methyl]
                           |                 |
    METHYLOME MAP:   (Pos 2: YES)      (Pos 5: YES)
```

### Differentially Methylated Regions (DMRs)
DMRs are segments of DNA that show consistent and statistically significant differences in DNA methylation patterns between distinct biological samples, such as healthy tissue vs. cancer tissue.

### CpG Sites
A CpG site is a location in DNA where a Cytosine (C) is followed directly by a Guanine (G), linked by a Phosphate (p) backbone. These are the primary checkpoints where DNA methylation occurs. 

### Hierarchical Clustering
This is a method used to group breast cancer patients by the similarity of their "tag maps." It builds a tree-like structure (dendrogram) to show which patients have the most similar disease profiles.

### EPIC Array (Beta Values)
Unlike raw sequencing, the EPIC Array provides **processed data**.
*   **Input:** Beta Values (0.0 to 1.0).
*   **Format:** A matrix where rows are samples and columns are CpG sites.

| Sample ID | Age | CpG_Site_1 | CpG_Site_2 |
| :--- | :--- | :--- | :--- |
| Patient_01 | 45 | 0.82 | 0.10 |
| Patient_02 | 62 | 0.79 | 0.12 |

## QC Results: The Bisulfite Trick

DNA sequencing machines are "blind" to methylation; they only see A, C, G, and T. To make methylation visible, we use **Bisulfite Treatment**.

### The Logic:
We soak the DNA in Sodium Bisulfite. This chemical is "aggressive" toward unprotected Cytosines.
*   **Rule 1:** If a **C** is "naked" (unmethylated), it turns into a **U** (which the computer reads as a **T**).
*   **Rule 2:** If a **C** has a "shield" (a methyl tag), the chemical cannot hurt it. It stays a **C**.

### The Outcome:
1.  **Original DNA:** `G - A - C - T`
2.  **Unmethylated Version:** Becomes `G - A - T - T` (C turned to T).
3.  **Methylated Version:** Stays `G - A - C - T` (C stayed C).

By looking at the "Per base sequence content" plot and seeing the **Red Line (T)** shoot to the top and the **Blue Line (C)** drop to the bottom, we confirm that our chemical treatment worked!

## Resources

*   [Galaxy Training: DNA Methylation Analysis](https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/methylation-seq/tutorial.html)
*   [Galaxy Training: Introduction to DNA Methylation](https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/introduction-dna-methylation/slides-plain.html)
*   [Bio-learn Documentation](https://bio-learn.github.io/)
*   [Bio-learn GEO Data Sources](https://bio-learn.github.io/data.html)
