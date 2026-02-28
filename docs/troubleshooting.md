# Troubleshooting

This page covers common issues encountered when running the CUPID-seq pipeline.

## Input Validation

The pipeline automatically checks inputs as its first step. Review the validation log for any warnings:

- **Real runs:** `16s-demux/workflow/out/inputCheck_log.txt`
- **Test runs:** `16s-demux/workflow/test_out/inputCheck_log.txt`

!!! success
    If the log only reads `All done!`, no input issues were detected.

You can also run a **dry run** before starting the full analysis to catch missing files and configuration errors:

```bash
snakemake -n
```

This determines all jobs that would be run and reports any missing inputs, without executing anything. You must run this inside a Docker container or Singularity shell.

---

## Common Input Issues

### Sample Naming Errors

**Problem:** Sample names contain spaces, underscores, or periods.

**Solution:** Rename fastq files and update the fastq file list and samplesheet so that sample names use only alphanumeric characters and hyphens.

!!! tip
    Periods are accepted by the demultiplexing pipeline but will cause errors with downstream tools like DADA2. Remove them if you plan to use DADA2.

### Fastqlist / Samplesheet Mismatch

**Problem:** The `file` column in the fastq file list doesn't match the `filename` column in the samplesheet.

**Solution:** The `filename` field in the samplesheet should follow the format `{RunName}-{round2plate}-{well}-L{round1index}`, where the `{RunName}-{round2plate}-{well}` portion exactly matches the `file` entries in the fastq file list. Check for typos, extra whitespace, or inconsistent naming.

### Wrong File Paths

**Problem:** The pipeline can't find fastq files.

**Solution:** Verify that the `fastqdir` variable in `config.yaml` combined with the paths in your fastq file list produces valid absolute paths. Check that files exist at those locations within the container.

### Wrong Index File

**Problem:** Using the default V4 index file when targeting a different 16S region.

**Solution:** Copy the correct index file from `other_index_for_demux/` into `config/` and update the `indices` field in `config.yaml`. See the [Configuration](configuration.md#pre-built-index-sets) page for the full list of available index files.

---

## Resource Issues

### Insufficient Memory

**Problem:** The pipeline fails at a consistent stage with hard-to-interpret errors.

**Solution:** Allocate at least 4 GB of memory. For building images and running the test, 4 GB and one core are sufficient, but larger allocations will speed up processing. If errors reliably occur at the same stage, insufficient memory is a likely cause.

### Build Failures on Login Nodes

**Problem:** Image building fails or is killed on an HPC login node.

**Solution:** Login nodes typically have strict resource limits. Build images on a compute node or submit the build as a job:

=== "Docker"

    Use a compute node with Docker access:

    ```bash
    docker build -t {name}:{version} .
    ```

=== "Singularity/Apptainer"

    Use the included Slurm template:

    ```bash
    # Edit slurmBuild.sh with appropriate SBATCH parameters, then:
    sbatch slurmBuild.sh
    ```

---

## Docker-Specific Issues

### Finding the Container Name

**Problem:** You need the container name for `docker cp` commands but don't know it.

**Solution:** List running containers:

```bash
docker container ls
```

Or check the Docker Desktop GUI under the Containers tab.

### Files Lost After Exiting

**Problem:** Output files are gone after exiting the container.

**Solution:** Always transfer output files **before** exiting:

```bash
docker cp {CONTAINER}:/16s-demux/workflow/out/ {local_path}
```

Once you exit and remove a container (`docker rm`), all data inside is permanently deleted. The image itself is preserved and can be used to start new containers.

---

## Singularity-Specific Issues

### Interactive vs. Job Manager

**Problem:** Analysis is too slow or gets killed when running interactively.

**Solution:** Interactive mode (`singularity shell`) is only recommended for testing and troubleshooting. For real analyses, submit jobs through your cluster's job manager:

```bash
singularity exec ../demux-image.sif snakemake --cores 1
```

A draft Slurm submission script (`submit_snakemake.sh`) is included. Edit the SBATCH parameters at the top for your system.

---

## DADA2-Specific Issues

### Silva Species-Level Error

**Problem:** The DADA2 pipeline raises an error when using the Silva database at the species level.

**Solution:** The Silva database (`silva_nr_v132_train_set.fa.gz`) does not support species-level (L7) resolution. If using Silva, edit `workflow/scripts/summary_taxa.R`:

1. Update line 14 to point to the Silva database:
    ```r
    ref_fasta <- "/databases/silva_nr_v132_train_set.fa.gz"
    ```
2. Comment out lines 93–106 of `summary_taxa.R` to skip species-level inference.

The default GreenGenes database (`gg_13_8_train_set_97.fa.gz`) supports all taxonomic levels including species.

---

## Getting Help

If your issue isn't covered here, please [open a GitHub Issue](https://github.com/rlporter24/Amplicon-dual-index-demux/issues) with:

- A description of the error
- The contents of `inputCheck_log.txt` (if applicable)
- The command you ran and any error output
- Your container type (Docker or Singularity) and version
