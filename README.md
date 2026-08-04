# Pan-Primate Reference Genome Project (PanPrimate-T2T)

Of the approximately 500 species of Primates, the vast majority still lack high-quality reference genomes-limiting our ability to study primate biology, conservation, evolution, and human health. The Pan-Primate Reference Genome Project (PanPrimate-T2T) addresses this gap by generating chromosome-scale, phased *de novo* genome assemblies and annotations for ~50 primate species spanning roughly 70 million years of evolutionary diversification across all major lineages.

All samples are derived from established cell lines maintained by the Coriell Institute for Medical Research, ensuring renewable biological resources and long-term reproducibility.

---

## Assemblies & Data Scope

Assemblies are generated using a standardized multi-platform sequencing pipeline:

- **Sequencing:** PacBio HiFi long reads, Oxford Nanopore (ONT) ultra-long reads, chromatin conformation capture (Omni-C or Pore-C), and PacBio Kinnex full-length transcriptomics.
- **Assembly & Curation:** Assembled with [Verkko](https://github.com/marbl/verkko) and [hifiasm](https://github.com/chhylp123/hifiasm), scaffolded with [YaHS](https://github.com/c-zhou/yahs), and manually curated.

This uniform approach resolves complex genomic regions-including centromeres, segmental duplications, immune loci, and sex chromosomes. Released datasets include haplotype-resolved assemblies, gene annotations, structural and small variant call sets, read alignments, and quality metrics.

The complete collection - spanning raw sequencing signals through whole-genome alignments - is expected to total **180–220 TB**.

- **Analysis Pipelines:** See [`analyses/README.md`](analyses/README.md) for workflows covering assembly, annotation, multi-sequence alignment, and anaysies performed as part of our publications.

---

## Species List & Data Access

- **Background & Biological Context:** See the [Coriell Pan-Primate Genome Project Page](https://www.coriell.org/1/Pan-Primate-Genome-Project) for species listings, taxonomy, and cell line details.
- **Dataset Manifests:** See [`README_dataset.md`](README_dataset.md) and [`manifests/sample_data_manifest.csv`](manifests/sample_data_manifest.csv) for per-sample metadata, accessions, and file locations. For HAL and other multi-species comparative data, see [`comparative/README.md`](comparative/README.md).

### Cloud & Public Repository Access

- **AWS Open Data:** Hosted for cloud-scale access without downloading multi-terabyte files.
- **NCBI SRA / BioProject:** Archived under BioProject/BioSample accessions listed in `manifests/sample_data_manifest.csv`. <!-- TODO: add SRA/BioProject link once accessions are assigned -->

Inspect the S3 bucket hierarchy via AWS CLI:

```bash
aws s3 ls s3://primate-t2t-genomics-open/species_data/ --no-sign-request
```

Full bucket layout, species IDs, and per-sample metadata: [`README_dataset.md`](README_dataset.md).

---

## Data Reuse and License
All data is released to the public under **Creative Commons Attribution 4.0 International (CC BY 4.0)**.

https://creativecommons.org/licenses/by/4.0/

You are free to share and adapt this material for any purpose, including commercially, provided appropriate credit is given.

---

## Relevant Citations

A DOI and associated publications will be added upon release.

---

## Contact & Team

For general questions regarding the PanPrimate-T2T Project, please contact [psudmant@berkeley.edu](mailto:psudmant@berkeley.edu).

### Project Leadership

* **Peter Sudmant** - [psudmant@berkeley.edu](mailto:psudmant@berkeley.edu)
* **Matthew Mitchell** - [mmitchell@coriell.org](mailto:mmitchell@coriell.org)
* **Erik Garrison** - [egarris5@uthsc.edu](mailto:egarris5@uthsc.edu)
* **Glennis Logsdon** - [glogsdon@pennmedicine.upenn.edu](mailto:glogsdon@pennmedicine.upenn.edu)

#### The Team

- **University of California, Berkeley**
    - Peter Sudmant
    - Scott Ferguson

- **Coriell Institute for Medical Research**
    - Matthew Mitchell
    - Harshleen Chawla

- **University of Tennessee Health Science Center**
    - Erik Garrison
    - Shuo Cao
    - Andrea Guarracino

- **University of Pennsylvania**
    - Glennis Logsdon
    - Shu-Cheng Chuang
    - Keisuke (Keith) Oshima
    - Shenghan Gao
    
- **University of California, Santa Cruz**
    - Prajna Hebbar

---

## History

```
* [date TBD]. v1.0 release - initial release of 10 genomes and associated sequencing, annotation data, and analysis scripts.
```
