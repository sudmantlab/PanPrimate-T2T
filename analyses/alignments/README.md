# Whole-Genome Alignment & Synteny

Multi-species whole-genome alignments using **Progressive Cactus** and **HAL**, investigating large-scale chromosomal rearrangements, synteny conservation, and structural evolution spanning ~70 million years of primate lineage diversification.

**Inputs:** phased haplotype assemblies (`genome_hap1`/`genome_hap2` in `../../manifests/sample_data_manifest.csv`) for all released species.

**Outputs:** pairwise (MAF) and multi-species (HAL) alignments, published under the shared top-level `comparative/` prefix — see [`../../comparative/README.md`](../../comparative/README.md) for the full layout, naming convention, and `comparative_manifest.csv` (which genome_ids went into each HAL build), since these outputs span all species and aren't tied to any single sample's assembly version.

```text
alignments/
├── pipeline.nf | Snakefile        # Progressive Cactus / HAL workflow definition
├── environment/                   # Docker/Singularity container specs
├── config/                        # Species-set and parameter configs
└── docs.md                        # Step-by-step instructions to re-run against the S3 dataset
```

> Placeholder — pipeline manifest, container spec, and docs to be added.
