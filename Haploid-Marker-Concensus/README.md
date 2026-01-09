# Haploid Marker Alignment Pipeline (Snakemake)

This repository contains a **standalone Snakemake pipeline** for extracting, aligning, and generating consensus sequences for **haploid genetic markers** from paired-end FASTQ files.

The pipeline was originally developed for mitochondrial and sex-linked markers (e.g. **12S**, **mtDNA**, **SRY**) and is intentionally restricted to **haploid data**. Diploid loci requiring phasing are **out of scope** and should be handled with a separate workflow.

---

## Overview

**Input**
- Paired-end FASTQ files
- A reference FASTA for a single haploid marker
- A sample sheet mapping sample IDs to FASTQs

**Processing steps**
1. Targeted read extraction using **BBduk** (k-mer baiting)
2. Alignment to the marker reference using **BWA**
3. BAM sorting and indexing
4. Mean depth calculation across the marker
5. Zero-coverage masking
6. Variant calling assuming **haploid ploidy**
7. Generation of a per-sample consensus FASTA
8. Merging of all consensus sequences with the reference

**Output**
- Per-sample BAM/BAI files
- Per-sample VCF + consensus FASTA
- Mean depth and QC summaries
- A merged FASTA suitable for downstream analyses (e.g. BLAST, alignment, phylogeny)

---

## 🧬 Intended Use Cases

This pipeline is appropriate for:

- Mitochondrial markers (e.g. 12S, cyt b, control region)
- Y-linked markers (e.g. SRY)
- Any **haploid locus** where a single consensus sequence per sample is appropriate

This pipeline is **not** intended for:

- Diploid nuclear genes
- Loci requiring genotype phasing
- Haplotype reconstruction from diploid data

---

## 📁 Repository Contents
```text
.
├── Snakefile
├── config/
│   └── config.yml
├── envs/
│   ├── align.yml
│   ├── bcftools.yml
│   └── bbduk.yml
└── README.md
```
---

## ⚙️ Configuration

All runtime settings are controlled via a YAML configuration file.

### Example `config.yml`

```yaml
out_dir: "/path/to/output_directory"
sample_sheet: "/path/to/sample_sheet.tsv"

marker:
  name: "12S"
  fasta: "/path/to/marker_reference.fa"

  bbduk:
    k: 23
    hdist: 2
    minlen: 30
    threads: 8

  bwa_threads: 8

  # Optional advanced settings
  map:
    bwa_mem_k: 11

  mpileup:
    max_depth: 100000
    minMQ: 0
    minBQ: 0
```
**Marker block**
* name: Short name used in output filenames
* fasta: Reference FASTA for the marker (single sequence recommended)

**Sample Sheet Format**
The sample sheet must be a tab-delimited file with the following columns:
```
sample	r1	r2	sex (optional)
SAMPLE_001	/path/sample1_R1.fastq.gz	/path/sample1_R2.fastq.gz	F
SAMPLE_002	/path/sample2_R1.fastq.gz	/path/sample2_R2.fastq.gz	M
```
**Running the Pipeline**
```
snakemake -s Snakefile --profile slurm 
```
**Output Structure**
```
output_dir/
├── 01_bait/
│   ├── sample.marker.bbduk.stats.txt
│   └── sample.marker.R1/R2.fq.gz
├── 02_map/
│   ├── sample.marker.bam
│   └── sample.marker.bam.bai
├── 03_consensus/
│   ├── sample.marker.vcf.gz
│   ├── sample.marker.consensus.fa
│   └── marker.merged_with_reference.fa
└── 05_qc/
    ├── sample.marker.mean_depth.txt
    ├── sample.marker.zero_cov.bed
    └── marker.depth_summary.tsv
```
**Notes on Consensus Generation**
* Variants are called assuming haploid ploidy (--ploidy 1)
* Zero-coverage positions are masked to N
**Quality Control**
The pipeline reports:
* Mean depth across the marker (can be useful for determing accuracy of sex-specific markers)
* BBduk read retention summaries
* Zero-coverage regions (e.g. female samples + SRY marker)
Low depth or extensive zero-coverage may indicate:
* Poor baiting efficiency
* Off-target amplification
* An inappropriate marker reference
**Important Limitations**
This pipeline assumes haploid data
Heterozygous sites should not occur under correct usage
Diploid loci require a separate workflow with explicit phasing

📬 Added by SPQ on 01/08/2026


