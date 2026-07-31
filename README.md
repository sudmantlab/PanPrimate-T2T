# Pan-Primate Reference Genome Project (PanPrimate-T2T)

## Dataset Structure & Technical Documentation

This repository provides technical documentation for the **Pan-Primate Reference Genome Project (PanPrimate-T2T)** dataset, including dataset organization, file formats, metadata standards, and cloud-based access workflows.

The PanPrimate-T2T project generates chromosome-scale, phased reference genomes and associated genomic resources for approximately 50 primate species spanning all major evolutionary lineages. The resulting resource supports:

- Comparative genomics
- Human disease and functional variant interpretation
- Conservation biology
- Genome assembly and annotation benchmarking
- Machine learning and genomic AI development

PanPrimate-T2T resources will be distributed through complementary public repositories, including **AWS Open Data** for cloud-scale analysis and **NCBI** for long-term archival access to sequencing data, genome assemblies, and biological sample metadata.

---

# Project Overview

Of the approximately 500 recognized primate species, the majority still lack high-quality reference genomes. This gap limits studies in:

- Primate evolution and genome diversity
- Human-specific genetic changes and disease mechanisms
- Conservation genetics of threatened species
- Functional genome biology across primate lineages

The PanPrimate-T2T project addresses this gap by generating chromosome-scale, phased reference genomes using a standardized multi-platform sequencing strategy across diverse primate species.

Genome resources include:

- PacBio HiFi long reads
- Oxford Nanopore ultra-long reads
- ONT raw signal data (POD5)
- Omni-C chromatin capture
- PacBio Kinnex full-length transcriptomics

Assemblies are generated using modern long-read assembly approaches, including Verkko and hifiasm, scaffolded with YaHS, and refined through manual curation. Resulting resources include phased genome references, annotations, variants, and comparative genome resources.

---

# Biological Resource Information

For project background, biological samples, species lists, and project updates:

**Coriell Pan-Primate Genome Project**  
https://www.coriell.org/1/Pan-Primate-Genome-Project

All genomes are generated from established Coriell cell line resources to support renewable sample access and long-term reproducibility.

---

# Public Data Resources and Access

The PanPrimate-T2T project will provide data through multiple complementary public resources.

## AWS Open Data

The planned AWS Open Data release is designed for cloud-based genomic analysis, enabling researchers to analyze large-scale datasets without downloading multi-terabyte files.

Planned S3 Bucket Location:

```text
s3://primate-t2t-genomics-open/
