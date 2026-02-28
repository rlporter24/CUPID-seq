# CUPID-seq

**CUPID-seq** (Containerized Dual-Index Demultiplexing) is a containerized pipeline for demultiplexing dual-indexed 16S amplicon sequencing data. By packaging the analysis in Docker and Singularity/Apptainer images, CUPID-seq eliminates dependency installation headaches and ensures highly reproducible results across computing environments.

## Key Features

- **Containerized** — runs in Docker or Singularity/Apptainer with zero dependency setup
- **Reproducible** — identical environments across local machines and HPC clusters
- **Flexible** — supports 13 pre-built 16S primer/index sets plus custom primers
- **DADA2 Integration** — optional downstream species inference with DADA2

## Pipeline Overview

The CUPID-seq pipeline processes paired-end amplicon reads through the following stages:

1. **Input validation** — checks file paths, naming conventions, and samplesheet/fastqlist consistency
2. **Demultiplexing** — separates reads by dual inline indexes (round 1 + round 2 barcodes)
3. **Trimming** — removes index, spacer, and primer sequences from demultiplexed reads
4. **Grouped output** — organizes trimmed reads into user-defined group subdirectories

### Read Structure

The dual-index primer design uses variable-length phased indexes on both read 1 and read 2:

![Primer structure overview](images/primerStructureImage.png)

![Phased primer design](images/phasedPrimerImage.png)

## Supported 16S Regions

Pre-built index files are provided for the following regions. The default is **V4**.

| Region | Index File |
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

These files are located in the `other_index_for_demux/` directory of the repository.

## Next Steps

- [**Getting Started**](getting-started.md) — install a container image and run your first analysis
- [**Configuration**](configuration.md) — detailed reference for config.yaml, input files, and custom primers
- [**Troubleshooting**](troubleshooting.md) — common issues and solutions
