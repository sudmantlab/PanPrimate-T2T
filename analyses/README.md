# Analysis Pipelines & Key Research Outputs

The primary publications accompanying the PanPrimate-T2T project focus on core analytical themes across primate evolution, comparative genomics, and functional biology. Analysis workflows, configuration files, and custom scripts for these studies are housed in dedicated subdirectories within this repository:

* **Whole-Genome Alignment & Synteny (`/analyses/alignments/`):**
  Multi-species whole-genome alignments using **Progressive Cactus** and **HAL** to investigate large-scale chromosomal rearrangements, synteny conservation, and structural evolution spanning ~70 million years of primate lineage diversification.

* **Structural Variant & Karyotype Evolution (`/analyses/structural_variants/`):**
  Comprehensive characterization of inversions, translocations, segmental duplications, and complex structural variation using haplotype-resolved T2T assemblies, identifying species-specific vs. evolutionarily conserved structural changes.

* **Complex Loci & Centromeric Regions (`/analyses/complex_loci/`):**
  Targeted resolution and comparative analysis of previously inaccessible genomic features, including centromeric satellite arrays, major histocompatibility complex (MHC) immune loci, and sex chromosomes across diverse primate clades.

* **Functional Annotation & Evolutionary Constraint (`/analyses/annotation/`):**
  Integration of PacBio Kinnex full-length transcriptomics and comparative pipelines (Ensembl VEP, OrthoFinder) to map gene family evolution, functional variant constraint, and regulatory sequence conservation.

> **Note on Reproducibility:** Each analysis subdirectory contains Nextflow/Snakemake pipeline manifests, containerized environment specifications (Docker/Singularity), and step-by-step documentation detailing how to re-run analyses against the S3 dataset.

Each subdirectory's inputs are the released data products described in [`../README_dataset.md`](../README_dataset.md) — assemblies, annotations, variant calls, and comparative alignments, referenced by S3 path or `genome_id` from [`../manifests/sample_data_manifest.csv`](../manifests/sample_data_manifest.csv).
