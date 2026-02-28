# Configuration

This page covers the container file structure, config.yaml settings, input file formats, and custom primer/index setup.

## Container File Structure

The Docker and Singularity images create the following directory layout:

![Container file structure](images/16s-demux_fileStructure.png)

Within the main `16s-demux` directory:

- **`config/`** — configuration files, fastq file list, samplesheet, and index files
- **`workflow/rules/`** — analysis code
- **`workflow/out/`** — output directory for real analyses
- **`workflow/test_out/`** — output directory for test runs
- **`fastq_data/`** — test sequencing data

The Snakefile and Slurm submission scripts are in the top-level `16s-demux` directory.

---

## config.yaml Reference

The `config/config.yaml` file controls all pipeline settings. The key fields are:

| Field | Description | Default |
| --- | --- | --- |
| `samplesheet` | Path to the samplesheet file | `samplesheet.txt` |
| `indices` | Path to the index file for demultiplexing | `indexfordemux.txt` |
| `fastqlist` | Path to the fastq file list | `fastq.txt` |
| `lenR1index` | Length of the read 1 index (longest phase) | `7` |
| `lenR2index` | Length of the read 2 index (longest phase) | `7` |
| `lenR1primer` | Length of read 1 primer (gene-specific + spacer) | `23` |
| `lenR2primer` | Length of read 2 primer (gene-specific + spacer) | `24` |

All paths are relative to the `config/` directory unless absolute paths are provided.

---

## Input Files

Three inputs are required beyond the sequencing data itself: a **fastq file list**, a **samplesheet**, and the sequencing **fastq data**.

!!! note
    Most common pipeline errors arise from input file formatting issues. The pipeline validates inputs automatically — check `workflow/out/inputCheck_log.txt` for any warnings.

### Fastq Data

**Naming rules:**

- Sample names must not contain spaces, underscores, or periods (hyphens are fine)
- Rename files before demultiplexing if they violate these rules

**Format:**

- Gzipped fastq files (`.fastq.gz`)

**Location:**

Fastq files can be located anywhere accessible to the container. Their paths are defined by combining the `fastqdir` variable in `config.yaml` with the paths in the fastq file list.

| `fastqdir` (config.yaml) | Fastq file list entry |
| --- | --- |
| `""` | `/home/users/TEST/sequencing/exp01/fastqs/TEST_R1_001.fastq.gz` |
| `/home/users/TEST/sequencing/exp01/fastqs/` | `TEST_R1_001.fastq.gz` |

!!! tip
    Periods in sample names are accepted for demultiplexing but will cause issues with downstream tools like DADA2. Avoid them if you plan to use DADA2.

### Fastq File List

**Location:** Place in the `config/` directory and update the `fastqlist` field in `config.yaml` (default name: `fastq.txt`).

**Format:** Tab-delimited text file with three columns:

| Column | Description |
| --- | --- |
| `read1` | Path to read 1 fastq.gz file |
| `read2` | Path to read 2 fastq.gz file |
| `file` | Shortened file identifier |

The `file` column should use the format `{RunName}-{round2plate}-{well}`, where:

- `RunName` — any identifier without underscores, spaces, or periods
- `round2plate` — plate identifier for round 2 barcodes
- `well` — well number

**Example:**

| read1 | read2 | file |
| --- | --- | --- |
| `../fastq_data/test_inputs/KKRP-001_S441_R1_001.fastq.gz` | `../fastq_data/test_inputs/KKRP-001_S441_R2_001.fastq.gz` | `15mc-003-P08B01-A01` |
| `../fastq_data/test_inputs/KKRP-002_S442_R1_001.fastq.gz` | `../fastq_data/test_inputs/KKRP-002_S442_R2_001.fastq.gz` | `15mc-003-P08B01-A02` |

!!! warning "Important"
    The `file` field in the fastq file list must match the first part of the `filename` field in the samplesheet.

### Samplesheet

**Location:** Place in the `config/` directory and update the `samplesheet` field in `config.yaml` (default name: `samplesheet.txt`).

**Format:** Tab-delimited file (`.tsv` or `.txt`) with a header row. Three columns are **required**:

| Column | Description |
| --- | --- |
| `filename` | Format: `{RunName}-{round2plate}-{well}-L{round1index}` |
| `sample` | Final sample name after demultiplexing (no underscores or periods) |
| `group` | Group identifier for output organization (no spaces or slashes) |

Additional columns can be included freely — only `filename`, `sample`, and `group` are used by the pipeline.

The `{RunName}-{round2plate}-{well}` portion of `filename` must match the corresponding `file` entries in the fastq file list. The `round1index` value should match the `phase` entry in the `indexfordemux.txt` table.

**Group behavior:** Samples in different groups are output into separate subdirectories within `trimmed/`. If you don't need grouping, use a single group name for all samples or leave the column blank (keep the `group` header).

**Example** (key columns shown):

| filename | sample | group |
| --- | --- | --- |
| `15mc-003-P08B01-A01-L4` | `P5-A01-plateP-wellA1` | `group1` |
| `15mc-003-P08B01-A02-L4` | `P5-A02-plateP-wellA2` | `group1` |
| `15mc-003-P08B01-A01-L5` | `P6-A01-plateW-wellA5` | `group2` |
| `15mc-003-P08B01-A02-L5` | `P6-A02-plateW-wellA6` | `group2` |

---

## Index Files

The `indexfordemux.txt` file contains the inline indexes used for demultiplexing. The **default** file uses the 16S **V4** index set.

### Pre-built Index Sets

Index files for 13 regions are provided in the `other_index_for_demux/` directory:

| Region | File |
| --- | --- |
| V1 - V2 | `V1-V2_index_for_demux.txt` |
| V1 - V3 | `V1-V3_index_for_demux.txt` |
| V2 - V3 | `V2-V3_index_for_demux.txt` |
| V3 | `V3_index_for_demux.txt` |
| V3 - V4 | `V3-V4_index_for_demux.txt` |
| V4 | `V4_index_for_demux.txt` (default) |
| V4 - V5 | `V4-V5_index_for_demux.txt` |
| V5 | `V5_index_for_demux.txt` |
| V5 - V7 | `V5-V7_index_for_demux.txt` |
| V6 | `V6_index_for_demux.txt` |
| V6 - V7 | `V6-V7_index_for_demux.txt` |
| V6 - V8 | `V6-V8_index_for_demux.txt` |
| V7 - V9 | `V7-V9_index_for_demux.txt` |

To use a different region, copy the appropriate file into `config/` and update the `indices` field in `config.yaml`.

!!! tip
    For all pre-built index sets, the default `config.yaml` length parameters are correct — no changes needed for `lenR1index`, `lenR2index`, `lenR1primer`, or `lenR2primer`.

### Custom Index Files

If you are using custom primers, you need to create a custom `indexfordemux.txt` file and may need to update the length parameters in `config.yaml`.

An Excel template (`other_index_for_demux.xlsx`) is provided in the `other_index_for_demux/` directory to help derive custom index files.

#### Understanding the Read Structure

The final read structure after library prep looks like this (lengths not to scale):

![Primer structure](images/primerStructureImage.png)

The round 1 indexes use variable-length "phases" (0–7). Each phase has a different number of index bases, with the remaining positions filled by spacer and gene-specific primer sequence:

| Phase | Variable FP | Variable RP |
| --- | --- | --- |
| 0 | | `ATGGACT` |
| 1 | `T` | `GCTAGC` |
| 2 | `GG` | `TGACT` |
| 3 | `ACT` | `CGGT` |
| 4 | `TAAC` | `GTA` |
| 5 | `CAGTC` | `AA` |
| 6 | `ATCGAT` | `C` |
| 7 | `GCAAGTC` | |

![Phased primer design](images/phasedPrimerImage.png)

During demultiplexing, reads are treated as having 7 base pair indexes on both ends. Any positions not filled by the actual index contain spacer or gene-specific primer sequence. In the default V4 index set:

- **Underlined** = actual index bases
- **lowercase** = spacer bases
- **BOLD UPPERCASE** = gene-specific primer region

| Phase | read1index | read2index | bc |
| --- | --- | --- | --- |
| 0 | cagt**AGA** | <ins>ATGGACT</ins> | CAGTAGAATGGACT |
| 1 | <ins>T</ins>cagt**AG** | <ins>GCTAGC</ins>a | TCAGTAGGCTAGCA |
| 2 | <ins>GG</ins>cagt**A** | <ins>TGACT</ins>at | GGCAGTATGACTAT |
| 3 | <ins>ACT</ins>cagt | <ins>CGGT</ins>atc | ACTCAGTCGGTATC |
| 4 | <ins>TAAC</ins>cag | <ins>GTA</ins>atcc | TAACCAGGTAATCC |
| 5 | <ins>CAGTC</ins>ca | <ins>AA</ins>atcc**T** | CAGTCCAAAATCCT |
| 6 | <ins>ATCGAT</ins>c | <ins>C</ins>atcc**TA** | ATCGATCCATCCTA |
| 7 | <ins>GCAAGTC</ins> | atcc**TAC** | GCAAGTCATCCTAC |

#### Changing Only the Gene-Specific Region

If you use the same primer design and index scheme but target a different gene, only the gene-specific regions within the indexes need to change.

**Example:** For the default V4 primers, the gene starts with `AGA...` and ends with `GTA` on the forward strand, giving regions of homology `AGA` and `TAC` (both 5' to 3').

For a gene reading `ATG ... CGT`, the regions of homology become `ATG` and `ACG` (both 5' to 3'). The updated index table would be:

| Phase | read1index | read2index | bc |
| --- | --- | --- | --- |
| 0 | cagt**ATG** | <ins>ATGGACT</ins> | CAGTAGAATGGACT |
| 1 | <ins>T</ins>cagt**AT** | <ins>GCTAGC</ins>a | TCAGTAGGCTAGCA |
| 2 | <ins>GG</ins>cagt**A** | <ins>TGACT</ins>at | GGCAGTATGACTAT |
| 3 | <ins>ACT</ins>cagt | <ins>CGGT</ins>atc | ACTCAGTCGGTATC |
| 4 | <ins>TAAC</ins>cag | <ins>GTA</ins>atcc | TAACCAGGTAATCC |
| 5 | <ins>CAGTC</ins>ca | <ins>AA</ins>atcc**A** | CAGTCCAAAATCCT |
| 6 | <ins>ATCGAT</ins>c | <ins>C</ins>atcc**AC** | ATCGATCCATCCTA |
| 7 | <ins>GCAAGTC</ins> | atcc**ACG** | GCAAGTCATCCTAC |

Only the gene-specific regions (bold) have changed — index and spacer sequences remain identical.

!!! danger "Mixed Base Characters"
    Avoid mixed base characters such as `W` or `N` in the first three positions of your gene-specific primer. If such bases are present, you must include extra entries in the `indexfordemux.txt` table — one for each possible base (e.g., for `W`, one entry with `A` and one with `T`). Each affected sample should appear twice in the samplesheet, and the output files will need to be merged downstream.

#### Updating Length Parameters

If your custom primers change the index or primer lengths, update `config.yaml`:

| Parameter | Description | Default |
| --- | --- | --- |
| `lenR1index` | Longest read 1 index length | `7` |
| `lenR2index` | Longest read 2 index length | `7` |
| `lenR1primer` | Gene-specific primer + spacer length (read 1) | `23` |
| `lenR2primer` | Gene-specific primer + spacer length (read 2) | `24` |

!!! note
    For all pre-built 16S index sets (V1–V2 through V7–V9), the default length values are correct and do not need to be changed.
