# DNA Methylation Pipeline: WGBS Data

## Overview
This pipeline identifies DNA methylation patterns from raw Whole Genome Bisulfite Sequencing (WGBS) reads using Galaxy. 
Data Source: [Zenodo Repository](https://zenodo.org/record/557099)

## Setup (Code)
Import raw sequencing data and reference files into a new Galaxy history.

```bash
# Upload these raw FASTQ files
https://zenodo.org/record/557099/files/subset_1.fastq
https://zenodo.org/record/557099/files/subset_2.fastq

# Upload reference CpG Islands
https://zenodo.org/record/557099/files/CpGIslands.bed
```

## Steps (Narrative + Commented Tool Parameters)

Check raw read quality to confirm bisulfite conversion worked.
```python
# Tool: Falco (v1.2.4)
# Input: subset_1.fastq, subset_2.fastq
# Note: 'Per base sequence content' SHOULD fail. 
# High T and low C levels indicate successful chemical conversion.
```

Align the converted reads to the human reference genome.
```python
# Tool: bwameth (v0.2.7)
# Index: Human (hg38full)
# Library: Paired-end
# Read 1: subset_1.fastq
# Read 2: subset_2.fastq
# Output: aligned_subset.bam # Mapping identifies if T was originally a C
```

Determine if methylation levels are biased at the ends of DNA fragments.
```python
# Tool: MethylDackel mbias (v0.5.2)
# Reference: Human (hg38)
# Input: aligned_subset.bam
# Options: Keep singletons=Yes, Keep discordant=Yes
# Result: SVG images # Check if start/end positions need trimming
```

Extract the final methylation metrics for every CpG site.
```python
# Tool: MethylDackel extract (v0.5.2)
# Reference: Human (hg38)
# Input: aligned_subset.bam
# Option: Merge per-Cytosine metrics=Yes
# Option: CpG methylation fractions (--fraction)
# Output: CpG_fractions.bedGraph
```

Prepare the data for visual profiling by converting formats.
```python
# Tool: Wig/BedGraph-to-bigWig
# Input: CpG_fractions.bedGraph
# Output: CpG_fractions.bigWig # Required for computeMatrix
```

Generate a profile plot of methylation levels across CpG Islands.
```python
# Tool: computeMatrix (v3.5.4)
# Regions: CpGIslands.bed
# Score file: CpG_fractions.bigWig
# Mode: reference-point

# Tool: plotProfile (v3.5.4)
# Input: Matrix from computeMatrix
# Output: Profile Plot # Shows average methylation dip at Islands
```

Compare different samples to find Differentially Methylated Regions (DMRs).
```python
# Tool: Metilene (v0.2.6.1)
# Group 1: NB1_CpG.meth.bedGraph, NB2_CpG.meth.bedGraph
# Group 2: BT198_CpG.meth.bedGraph
# Regions: CpGIslands.bed
# Output: DMR_Report.pdf # Identifies epigenetic state differences
```

## Results (What to see)

| Step | Expected Result |
| :--- | :--- |
| **Falco** | Failed sequence content; Red line (T) is high, Blue line (C) is near zero. |
| **MethylDackel** | M-bias plots showing stable methylation across the read length. |
| **plotProfile** | A "valley" shape; methylation usually drops at CpG islands/promoters. |
| **Metilene** | Statistics on DMR length, CpG count, and q-values for differences. |

## Reference
Official Tutorial: [Galaxy Training - DNA Methylation Sequencing](https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/methylation-seq/tutorial.html)
