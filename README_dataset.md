# PanPrimate-T2T dataset

## Species ID

| species_id | genus | species | common_name | accession_id | assembly_status |
| --- | --- | --- | --- | --- | --- |
| mandrillus_leucophaeus | Mandrillus | leucophaeus | Drill | PR00232 | released |
| papio_anubis | Papio | anubis | Olive Baboon | PR00036 | in_progress |
| eulemur_rubriventer | Eulemur | rubriventer | Red-bellied Lemur | PR00129 | planned |

`accession_id` is the identifier used throughout the S3 bucket path (`species_data/<accession_id>/...`) and in raw/version-specific filenames; `species_id` is a taxonomic identifier, not a path component. This table is illustrative — the authoritative, current record for every sample (not just species) is `manifests/sample_data_manifest.csv` (see below).

## Sample data manifest

`manifests/sample_data_manifest.csv` is the single metadata table for the whole dataset — one row per released assembly version, with every accession and every data file location as a column. A sample that has been re-assembled or re-curated has one row per version it has ever shipped (e.g., `PR00232_1.0` and `PR00232_1.1` are two separate rows), so no history is ever overwritten and every `genome_id` that was ever cited stays resolvable. Sample-identity fields (species, sex, accessions) are repeated identically across that sample's version rows. Raw data columns (`raw_ont`, `raw_pacbio_hifi`, `raw_chromatin_capture`, `raw_kinnex_rna`) hold a pipe-delimited (`|`) list of the run file(s) that fed that version's assembly — usually the same list across a sample's versions, but not always: raw sequencing is stored as one file per run (never merged), and if a new assembly version incorporates an additional run for a platform (e.g., a second ONT run added for deeper coverage), that row's column gains the new run's path while keeping the earlier one, and the earlier version's row is left exactly as it was. A published row is never rewritten after the fact, and no raw file is ever overwritten in place; new raw data always arrives as a new, separate run file (we use "append-only" to refer to this data method).

| Column | Description |
| --- | --- |
| `genome_id` | Primary key. `<accession_id>_<genome_version>`, e.g. `PR00232_1.1`. One row per `genome_id`. Cite this when referencing a specific assembly. Blank for samples with no released version yet. |
| `accession_id` | Coriell catalog ID for the sample. Repeats across all of that sample's version rows. |
| `genome_version` | Version of *this specific row's* assembly, e.g. `1.0`, `1.1` — matches the `v<major>.<minor>/` directory in S3. |
| `is_latest` | `TRUE` for the one row per sample that is the current/most-recent released version, `FALSE` for all superseded versions. Filter on this to get "current state of the dataset." |
| `species_id` | Taxonomic identifier, derived as lowercase `<genus>_<species>` — e.g. `genus=Mandrillus`, `species=leucophaeus` → `species_id=mandrillus_leucophaeus`. |
| `genus` | Genus, capitalized (e.g. `Mandrillus`) |
| `species` | Species epithet, lowercase (e.g. `leucophaeus`) |
| `common_name` | Common species name |
| `sex` | `male` / `female` / `unknown` |
| `biosample` | NCBI BioSample accession |
| `bioproject` | NCBI BioProject accession |
| `assembly_status` | `planned` / `in_progress` / `released` |
| `genome_hap1`, `genome_hap2` | S3 path to each phased haplotype assembly |
| `genome_pri`, `genome_alt` | S3 URIs for the primary and alternate assembly representations, derived from `genome_hap1`/`genome_hap2`: `genome_pri` takes the longer sequence at each homologous chromosome pair; `genome_alt` contains the shorter sequence from each pair. |
| `annotation_gff3` | S3 path to gene annotation |
| `variants_vcf` | S3 path to variant calls |
| `raw_ont`, `raw_pacbio_hifi`, `raw_chromatin_capture`, `raw_kinnex_rna` | Pipe-delimited (`\|`) list of S3 paths to the run file(s) used for this version, e.g. `..._run1.pod5\|..._run2.pod5` |
| `alignment_reads` | S3 path to HiFi reads aligned to the assembly |


Samples with no released version yet (`assembly_status` = `planned` or `in_progress`) have a single row with `genome_id`, `genome_version`, `is_latest`, and all data-path columns blank.

**Versioning is per-sample, per-assembly, and lives directly in the S3 path** — there is no single bucket-wide release version. `raw_sequencing/` is append-only: new files can be added when more sequencing is generated for a sample, but existing files are never modified or removed, so any file a past version's manifest row points to remains exactly as it was. Each re-assembly or re-curation gets its own `v<major>.<minor>/` directory containing that version's `assembly/`, `annotation/`, `variants/`, and `alignments/`. See [Bucket layout](#bucket-layout-and-download) below. `genome_version` in the manifest always matches the version directory that row's paths point into, and `genome_id` (`<accession_id>_<genome_version>`) is what you cite for a specific assembly.

**`genome_version` follows `<major>.<minor>`, and the two numbers mean different things:**

| Bump | Meaning | Example |
| --- | --- | --- |
| **Minor** (`1.0` → `1.1`) | Error fix or re-scaffolding on the *same* underlying raw data — no new sequencing incorporated | Fixing a misjoin, improving scaffolding with the existing Omni-C/Pore-C data |
| **Major** (`1.1` → `2.0`) | Full re-assembly, which *may* include newly incorporated raw data | Adding a second ONT run for deeper coverage and re-assembling from scratch |

A minor bump's `raw_*` columns are always identical to the version it patches. A major bump's `raw_*` columns may point at newer, additional raw files if new data was incorporated (see below) — or may stay the same if the re-assembly used identical inputs with a different pipeline/parameters.

Every version a sample has ever had ships as its own row — nothing is overwritten when a new version is released, so a previously-cited `genome_id` always stays resolvable in the manifest. Filter on `is_latest == TRUE` to get one row per sample reflecting only the current state of the dataset.

```bash
# Pull the manifest and list every currently-latest released genome's ID and path
aws s3 cp --no-sign-request \
  s3://primate-t2t-genomics-open/manifests/sample_data_manifest.csv - \
  | awk -F, '$12 == "released" && $4 == "TRUE" { print $1, $13 }'

# List every version that has ever existed for one sample
aws s3 cp --no-sign-request \
  s3://primate-t2t-genomics-open/manifests/sample_data_manifest.csv - \
  | awk -F, '$2 == "PR00232" { print $1, $3, $4 }'
```

## AWS Access & Compute

All indexed files (BAM `.bai`, bgzip VCF/GFF3/BED `.tbi`, bgzip FASTA `.fai`) support HTTP byte-range requests directly from S3 — you can pull a single genomic interval without downloading the file, using standard tools:

```bash
# A region of aligned reads
samtools view \
  https://primate-t2t-genomics-open.s3.amazonaws.com/species_data/PR00232/v1.0/alignments/PR00232_1.0.hifi.sorted.bam \
  chr7:1000000-1050000

# The same region's variant calls
bcftools view \
  https://primate-t2t-genomics-open.s3.amazonaws.com/species_data/PR00232/v1.0/variants/PR00232_1.0.vcf.gz \
  chr7:1000000-1050000

# The same region's gene annotation
tabix \
  https://primate-t2t-genomics-open.s3.amazonaws.com/species_data/PR00232/v1.0/annotation/PR00232_1.0.gff3.gz \
  chr7:1000000-1050000
```

IGV and JBrowse can also load these S3 URLs directly as remote tracks, without downloading anything locally.

**Running compute directly against S3**, rather than copying multi-terabyte files to local or attached storage:

| Service | Use |
| --- | --- |
| **Amazon EC2** | Mount via `mountpoint-s3`/`s3fs`, or point `samtools`/`bcftools` directly at S3 URLs |
| **AWS Batch** | Containerized jobs reading/writing `s3://primate-t2t-genomics-open/...` directly, e.g. array jobs iterating over `sample_data_manifest.csv` |
| **Amazon SageMaker** | Training/processing jobs with the bucket as an S3 input channel — e.g. combining raw ONT/HiFi signal with assemblies for genomic foundation model training |
| **AWS HealthOmics** | Whole-genome alignment, variant discovery, and comparative genomics workflows using the released BAM/VCF/assembly files as inputs |

## Bucket layout and download

Data lives at:

```text
s3://primate-t2t-genomics-open/
```

```bash
# Everything for one sample (all versions + raw data)
aws s3 cp --no-sign-request --recursive \
  s3://primate-t2t-genomics-open/species_data/PR00232/ .

# Just the current released assembly (v1.0)
aws s3 cp --no-sign-request --recursive \
  s3://primate-t2t-genomics-open/species_data/PR00232/v1.0/ .
```

Layout:

```text
s3://primate-t2t-genomics-open/
├── manifests/
│   └── sample_data_manifest.csv        # one row per sample — accessions + current-version paths
│
└── species_data/
    └── <accession_id>/
        ├── raw_sequencing/              # APPEND-ONLY — one file per sequencing run, never merged or overwritten
        │   ├── ont/                     # POD5 raw signal + FASTQ.GZ per run
        │   ├── pacbio_hifi/             # Unaligned HiFi BAM + FASTQ.GZ per run
        │   ├── chromatin_capture/       # BAM + FASTQ.GZ per run — Omni-C, Pore-C, or future methods
        │   └── kinnex_rna/              # Isoform RNA BAM + FASTQ.GZ per run
        │
        ├── metadata/                    # APPEND-ONLY — sample-level, not tied to any assembly version; new runs get appended, nothing is edited/removed
        │   └── <accession_id>_sample_metadata.json
        │
        ├── v1.0/                        # Assembly release 1.0
        │   ├── assembly/                # Phased FASTA files + QC
        │   ├── annotation/              # GFF3 gene models + repeat BED
        │   ├── variants/                # VCF files
        │   └── alignments/              # Reads aligned to the v1.0 assembly
        │
        └── v1.1/                        # Assembly release 1.1 (patched/re-curated)
            ├── assembly/
            ├── annotation/
            ├── variants/
            └── alignments/              # Reads aligned to the v1.1 assembly
```

Raw sequencing is generated once per sample and never re-derived per assembly version, so it lives outside the `v*/` directories. Everything version-specific — the assembly itself and everything built from it (annotation, variant calls, read alignments) — is fully contained within its own `v<major>.<minor>/` directory, so two versions never overwrite or depend on each other.

**Version-specific files are named with the full `genome_id`, not just `accession_id`:**

| File | Pattern | Example |
| --- | --- | --- |
| Haplotype 1 assembly | `<genome_id>.hap1.fa.gz` | `PR00232_1.1.hap1.fa.gz` |
| Haplotype 2 assembly | `<genome_id>.hap2.fa.gz` | `PR00232_1.1.hap2.fa.gz` |
| Gene annotation | `<genome_id>.gff3.gz` | `PR00232_1.1.gff3.gz` |
| Variant calls | `<genome_id>.vcf.gz` | `PR00232_1.1.vcf.gz` |
| Read alignment | `<genome_id>.hifi.sorted.bam` | `PR00232_1.1.hifi.sorted.bam` |

Embedding `genome_id` directly in the filename makes every file **self-describing outside of directory context** — if it's downloaded, cited, or shared standalone, the filename alone still tells you exactly which assembly version it belongs to, without needing the surrounding `v1.1/` path for that information.

**Raw file storage:** each sequencing run is stored as its own file, named `<accession_id>_<platform>_run<N>.<ext>` (e.g. `PR00232_hifi_run1.bam`) inside that platform's directory — runs are **never merged** into a single combined file. When new sequencing arrives for a platform, it's added as a new `run<N>` file alongside the existing ones; nothing is ever overwritten or replaced. A sample can have multiple run files per platform sitting in the same directory. The manifest's `raw_*` columns record exactly which run file(s) fed a given assembly version as a pipe-delimited list (see above) — this is how "which combination of files" is answered without needing a separate lookup.

**Every run also ships a bgzip-compressed FASTQ alongside its native file** — `<accession_id>_<platform>_run<N>.fastq.gz` sits next to `..._run<N>.bam` (or `.pod5`) in the same directory, sharing the same `run_id`. This isn't a separate manifest entry: since it's always the same basename with a different extension, it's fully implied by the `raw_*` path already in the manifest — swap the extension to get it. FASTQ files are provided, especially for ONT, for tools that can only make use of this data type or to allow direct streaming of sequence data.

**Why not merge runs into one file per platform?** Merging (e.g. `pod5 merge`) would mean re-storing every prior run's data again each time new data arrives — at this dataset's scale, that duplicates multi-TB files repeatedly for no benefit. Assembly and basecalling tools (hifiasm, verkko, dorado) all accept multiple input files natively, so there's no processing reason to pre-merge; a user who wants one combined file can merge on demand from the exact run files listed in the manifest.

**How this interacts with region-based streaming:** the byte-range/streaming examples in [AWS Access & Compute](#aws-access--compute) apply to **indexed, per-version outputs** (`alignments/`, `variants/`, `annotation/`) — files with a `.bai`/`.tbi`/`.fai` index built for genomic-coordinate lookup. Raw sequencing was never part of that pattern: it isn't aligned to anything yet, so there's no coordinate to seek to. Consuming raw data means fetching the specific run file(s) you need (from the manifest's pipe-delimited list) and feeding them to a processing tool as a file list — not a byte-range query. These are two different access patterns for two different kinds of file, not a limitation introduced by storing multiple runs per platform.

## PacBio HiFi

`raw_sequencing/pacbio_hifi/<accession_id>_hifi_run<N>.bam` (+ `..._run<N>.fastq.gz`) — unaligned HiFi reads, one file per run (never merged). The primary assembly input for Verkko/hifiasm, which accept multiple run files directly. Coordinate-sorted alignment against a specific assembly version is at `species_data/<accession_id>/<version>/alignments/<genome_id>.hifi.sorted.bam` (+ `.bai`).

## ONT

`raw_sequencing/ont/<accession_id>_ont_run<N>.pod5` (+ `..._run<N>.fastq.gz` **derived** basecalled) — raw signal per run, not just basecalled reads, so the data supports basecaller/methylation-model benchmarking directly. Basecalling tools (e.g. `dorado`) accept a directory or list of POD5 files natively; the FASTQ is provided as a convenience for tools that need basecalled reads directly rather than raw signal. Basecalling model and version used for the shipped calls, if any accompany the POD5, are noted per-run in `metadata/<accession_id>_sample_metadata.json` (`sequencing.ont[N].chemistry`, matched by `run_id`).

## Chromatin Capture (Omni-C / Pore-C)

`raw_sequencing/chromatin_capture/<accession_id>_chromatin_capture_<method>_run<N>.bam` (+ `..._run<N>.fastq.gz`) — chromatin conformation capture reads used for YaHS scaffolding to chromosome scale. The directory and manifest column (`raw_chromatin_capture`) are method-agnostic since the project currently uses **Omni-C** but may use **Pore-C** or other chromatin capture methods for future species; the actual method and kit/protocol version are recorded per-run in metadata (`sequencing.chromatin_capture[N].method`, `sequencing.chromatin_capture[N].kit_version`), and the filename itself embeds the method (`..._omnic_run1.bam`, `..._porec_run1.bam`) so it's identifiable without opening the JSON.

## Kinnex RNA

`raw_sequencing/kinnex_rna/<accession_id>_kinnex_run<N>.bam` (+ `..._run<N>.fastq.gz`) — PacBio Kinnex full-length transcriptome reads, one file per run, used to annotate genomes for a given assembly version.

## Sample metadata

Every sample has a full metadata record at `metadata/<accession_id>_sample_metadata.json`. Real-format example: [`examples/mandrillus_leucophaeus_sample_metadata.json`](examples/mandrillus_leucophaeus_sample_metadata.json).

```json
{
  "species_id": "mandrillus_leucophaeus",
  "accession_id": "PR00232",
  "sex": "male",
  "sequencing": {
    "ont": [
      { "run_id": "run1", "instrument": "PromethION 24", "chemistry": "R10.4.1", "run_accession": "SRR00000010" },
      { "run_id": "run2", "instrument": "PromethION 2 Solo", "chemistry": "R10.4.1", "run_accession": "SRR00000014" }
    ],
    "pacbio_hifi": [
      { "run_id": "run1", "instrument": "Revio", "chemistry": "SMRTbell prep kit 3.0", "run_accession": "SRR00000011" }
    ],
    "chromatin_capture": [
      { "run_id": "run1", "method": "omni_c", "kit_version": "Dovetail Omni-C v2", "run_accession": "SRR00000012" }
    ],
    "kinnex_rna": [
      { "run_id": "run1", "instrument": "Revio", "run_accession": "SRR00000013" }
    ]
  },
  "ncbi_biosample": "SAMN00000001"
}
```

Each `run_id` (`run1`, `run2`, ...) matches the `_run<N>` suffix in that platform's raw filenames, so a manifest path like `..._ont_run2.pod5` can be looked up directly against `sequencing.ont[1]` in this file for its instrument/chemistry/accession. New runs are appended to the relevant platform's list as they're generated — existing entries are never edited or removed.

The per-sample JSON carries richer detail (instrument, chemistry) than fits in a flat CSV column; `sample_data_manifest.csv` is the fast index, this JSON is the full record — use the manifest to find the sample, then fetch its JSON for full detail if needed.

## NCBI cross-reference

Every sample's NCBI accessions are columns in `sample_data_manifest.csv` (`bioproject`, `biosample`). Raw reads and finished assemblies are additionally archived at NCBI; AWS hosts the full cloud-optimized lineage (raw signal through comparative alignments) for interactive, region-based analysis.

## Comparative and multi-species data

HAL alignments and other data products that span multiple species (rather than belonging to one sample) don't fit the per-accession model above. See [`comparative/README.md`](comparative/README.md) for how that data is organized, versioned, and named.
