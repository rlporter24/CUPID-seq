# 16S-dual-index

[![Documentation](https://img.shields.io/badge/docs-online-5b9aa0)](https://rlporter24.github.io/CUPID-seq/)

The following document describes how to use the Docker and Singularity/Apptainer images to use our demultiplexing code. By using these containerization platforms, users can avoid the time consuming process of installing dependencies and configuring environments and conduct analyses in a highly reproducible manner. 

**Contents:**
* [Using Docker](https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/README.md#using-docker)
* [Using Singularity/Apptainer](https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/README.md#using-singularityapptainer)
* [Inputs](https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/README.md#inputs)
* [Building Images](https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/README.md#building-images)
* [Building with Docker](https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/README.md#building-with-docker)
* [Building with Singularity/Apptainer](https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/README.md#building-with-singularityapptainer)
* [Custom Primers](https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/README.md#custom-primers)

## Using Docker
To analyze our code to analyze dual-indexed sequencing data, first ensure that Docker is installed. Docker is a handy platform for sharing not only code but also coding environments, which enables easy code sharing without hours of installing dependencies. You can find instructions on Docker installation here {https://docs.docker.com/engine/install/}. Once Docker is installed, open the Docker desktop application or run ‘systemctl start docker’ to launch the Docker Daemon. 


1. **Installation and setting up the Docker image** \
  There are two ways to acquire the docker image needed for analysis, either by pulling directly from docker hub (the easiest approach) or using the dockerfile and other inputs provided in 'docker-demux.zip' (available on ZENODO LINK) to build the docker image.\
\
  **Pulling image:** \
  Launch Docker. Then in a terminal, run:\
 `docker pull rlporter24/dualindex-demux:1.0`.\
 This will make a local copy of the Docker image 'rlporter24/dualindex-demux', which contains the code and environment needed to process the data, as well as example input files for running a test.\
\
  **Building image:**\
  Instead of pulling the docker image from the Docker hub, the docker image can also be built from the Dockerfile. This process takes longer and is not recommended, but is described in the >Building images section below. 

3. **Run the test analysis**
  To check that the set up was successful, run a quick analysis using provided test data. Start by running\
  `docker run -it rlporter24/dualindex-demux:1.0`\
  to open a container from the image rlporter24/dualindex-demux in an interactive mode (specified by the flags -it). If you built your own image, replace 'rlporter24/dualindex-demux' with the  '{name}:{version}' you provided for the build. In this mode, we can enter a series of commands, step by step within this container. All of the necessary input files are already included within the container, so no files need to be imported. To run the test, run:\
  `snakemake --cores 1 -s test_Snakefile`\
 (or replace 1 with the desired number of cores for this run)\. This should take under 5 minutes, and will run a test analysis using ‘config/test_fastq.txt’, ‘config/test_samplesheet.txt’ and test files included in /fastq_data/test/. The output files will be generated in the ‘workflow/test_out/’ directory. If the run is successful, the following outputs should be generated in ‘workflow/test_out/trimmed’:\
 <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/testSuccessOutputs.png?raw=true" alt="Alt Text" width="400" height="1000">\
### ^ THIS WILL NEED TO BE UPDATED WITH NEW NAMES!\

  If these files are generated in ‘trimmed’, the run has been successful! To exit the container, simply type ‘exit’, or stay in the container to run the actual analysis.

5. **Run the actual analysis**\
   If you do not already have an open container, launch an interactive container by running:\
  `docker run -it rlporter24/dualindex-demux:1.0` .\
\
As before, if you built your own image, replace 'rlporter24/dualindex-demux' with the  '{name}:{version}' you provided for the build. Next, we can move in the input files (The essential inputs will be the raw fastq data, a fastq file list, and a samplesheet. Details on creating these inputs for your actual analysis are included below in >Inputs.). Transfering input files can be done from inside or outside of the container, as described here {https://docs.docker.com/reference/cli/docker/container/cp/}. From outside of the container (in a separate terminal), run:\
  `docker cp {local_path} {CONTAINER}:{container_path}`\
  The {local_path} can be to an individual file or a directory. The {CONTAINER} should be the container name, not the image name. The container name can be found by running 
  `docker container ls` to list all the current containers, or by looking at the containers in the docker decktop GUI. The {container_path} should be provided relative to the '16s-demux' directory, which is the home   directory within the container.\
\
In the ‘config’ directory, update ‘config.yaml’ so that `samplesheet:` and `fastqlist:` in lines 2 and 4 are followed by the paths to your input samplesheet and fastqlist, respectively (more details in the __Inputs__ section). If you are using custom primers or indexes, you may need to adjust the input file for ‘indicies:’ or the lengths of read1 and read2 indexes and primers (lines 3, 5, 6, 7, 8). For more details on custom primers, see the __Custom Primers__ section.\
\
Once the inputs and paths are updated, run:\
`snakemake --cores 1`\
within the 16S-demux directory (replacing 1 with the desired number of cores). The analysis time will vary with the number and size of input files as well as the machine used. If the run is successful, a message similar to that below should be output:\
<img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/snakemakeSuccessOutput.png?raw=true" alt="Alt Text" width="750" height="400">\
\
__Note:__ Before starting the actual run, you can use the command ‘snakemake -n’ to do a dry run. This is helpful for ensuring that names and file locations are correct before starting the full run.

7. **Transfer Output Files Out**\
Once the run is completed, output files must be transfered out of the container. To move files located in /16s-demux/workflow/out/ to a local destination, run the following:\
`docker cp {CONTAINER:/16s-demux/workflow/out/} {local_path}`.\
8. **Clean Up**\
After the run is completed and outputs have been moved out and saved, exit from the container. The remnants of the container created can be removed with the following code:\
`docker rm  {container name}`\
This will not remove the image - only the container that was just now built using this image. Be sure any important data or intermediates are transferred out of the container before removing it.\





## Using Singularity/Apptainer
Most HPC are not compatible with Docker use, but do support Singularity/Apptainer. To run using Apptainer, follow these steps:

1. **Setting up the Singularity/Apptainer image**\
   First, ensure that Singularity/Apptainer is installed. The pre-built Singularity image is not currently available for direct download, but the Singularity image can be built locally fairly quickly ( ~10 minutes) and only requires that some specific files be made available locally. Instructions are included in the ‘Building Images’ section below. Once the build is successfully completed, you should have a file named ‘demux-image.sif’ in the directory from which it was built.

### Running Interactively (Not Recommended):
Depending on the resources available locally, you can run the analysis in an interactive Singularity shell, rather than submitting a job to a resource manager. It is highly unlikely that a login node will provide sufficient resources, so check this before starting and use a compute node as needed. Running interactively is only recommended for testing inputs or troubleshooting. If you choose to run interactively follow these steps:

2. **Open a shell in the container**\
   To open a shell, run the following command from the directory containing the ‘demux-image.sif’:\
   `singularity shell demux-image.sif` \
   This will bring you to a shell within the container where you can interactively run processes.\


3. **Run the test analysis**\
   To check that the set up was successful, run a quick analysis using provided test data. To do this, run:\
  `snakemake --cores 1 -s test_Snakefile`
  within the ‘16S_demux_edits’ directory. This should take about 16 minutes depending on your system, and will run a test analysis using ‘config/test_fastq.txt’,   ‘config/test_samplesheet.txt’ and the test files included in /fastq_data/test/. The output files will be generated in the ‘workflow/test_out/’ directory. If the run is successful, the following outputs should be generated within the test_out/trimmed directory:\
     <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/testSuccessOutputs.png?raw=true" alt="Alt Text" width="400" height="1000">\
   ### ^ THIS WILL NEED TO BE UPDATED WITH NEW NAMES!\
  If the files are generated in the ‘trimmed’ directory are all present, the run has been successful.\
  To exit the container, simply type ‘exit’.

5. **Run the actual analysis**\
  To run the actual analysis, ensure your input files are accurate and located in the correct directory (see >Inputs for details), and ensure that the paths within ‘config/config.yaml’ are correct. 
\
  If you aren’t using a job manager like slurm, you can use the command:\
  `snakemake --cores 1`\
  in the 16S-demux directory to start the analysis.\
\
The analysis time will vary with the number and size of input files. If the run is successful, a message similar to that below should be output:\
  <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/snakemakeSuccessOutput.png?raw=true" alt="Alt Text" width="750" height="400">\
__Note:__ Before starting the actual run, you can use the command `snakemake -n` to do a dry run. This is helpful for ensuring that names and file locations are correct before starting the full run.

### Running with resource manager:
Most HPC clusters will use a resource manager, such as Slurm. To run the analysis through a manager, complete step 1 above, and then follow the instructions in this section. The code includes draft scripts for submitting through Slurm, but these can be adapted to other systems. 

2. **Run the test analysis**\
To check that the set up was successful, run a quick analysis using provided test data. To do this, edit the draft script ('submit_test_snakemake.sh') to submit to a job manager. The script should run the following line from the 16s-demux direcotry:\
  `singularity exec ../demux-image.sif snakemake --cores 1 -s test_Snakefile`. (Replace 1 with the desired number of cores)\
This should take around 10 minutes depending on your system, and will run a test analysis using ‘config/test_fastq.txt’, ‘config/test_samplesheet.txt’, and the test files included in /fastq_data/test/. The output files will be generated in the ‘workflow/test_out/’ directory. If the run is successful, the following outputs should be generated within the test_out/trimmed directory:\
     <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/testSuccessOutputs.png?raw=true" alt="Alt Text" width="400" height="1000">\
     ### ^ THIS WILL NEED TO BE UPDATED WITH NEW NAMES!\
   \
  If the files are generated in the ‘trimmed’ directory are all present, the run has been successful.\

4. **Run the actual analysis**\
To run the actual analysis, ensure your input files are accurate and located in the correct directory (see >Inputs for details), and ensure that the paths within ‘config/config.yaml’ are correct. Next edit and use a submission script as above to run the command:\
`singularity exec ../demux-image.sif snakemake --cores 1`. (Replace 1 with the desired number of cores)\
A dfradt script is provided as 'submit_snakemake.sh'.\
\
  __Note:__ To use this submission script, be sure to change the SBATCH parameters at the top as needed.

## Inputs
In addition to the sequencing data itself, there are two input files needed for demultiplexing: a fastq file list, and a sample sheet. Additionally, the included config.yaml file will need to be edited.\
\
The general file structure created by the Docker/Singularity images is shown below. Within the main folder (16s-demux), there are two important folders, config and workflow. The code for running the analysis is stored in workflow/rules/, and outputs are generated within workflow/out/. Test files and most input files are found within the config folder.\
Test files are included within config/ (‘test_config.yaml’, ‘test_fastq.txt’, ‘test_samplesheet.txt’), and the official config.yaml file, fastq file list, and samplesheets should also be included in the config folder. Below, the fastq file list and samplesheets are named ‘fastqlist.txt’ and ‘samplesheet.tsv’ respectively, but the names can vary. The indexfordemux.txt file, includes the unique in-line indices. The default indexfordemux.txt file features the 16S V4 indexes, so this file will need to be swapped with the corresponding indexfordemux.txt file for other regions or custom primers.\ 
\
The script for initiating the Snakemake run using Slurm (submitSnakemake.sh) and the Snakefile encoding the analysis are found in the top 16s-demux file.\
  <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/16s-demux_fileStructure.png?raw=true" alt="Alt Text" width="400" height="400">

**Note:** Most common issues in demultiplexing arise from errors in the input files. To help with troubleshooting, a quick check of the inputs will be conducted as the first step of the pipeline, and a summary will be output to '16s-demux/workflow/out/inputCheck_log.txt' (or '16s-demux/workflow/test_out/inputCheck_log.txt' for tests). If you encounter errors during the actual run but not in the test run, it may be helpful to check this log to ensure the inputs are properly formatted.  

1. **Fastq data:**\
  **Names:** Sample names should not include any spaces, underscores, or periods (hyphens are fine). If they do, they should be renamed before demultiplexing.\
   \
  **Format:** Files should be formatted as gzipped fastq files, or “.fastq.gz” files.\
   \
  **Location:** If the demultiplexing will be run locally or on a server, fastq files just need to be somewhere on that server. The paths to fastq files are defined by the entries included in 
the fastq file list and the config.yaml variable ‘fastqdir’. If the absolute path to the fastq files is provided in the fastq file list, then the fastqdir variable in config.yaml should be an empty string (“”). Otherwise, ensure that the ‘fastqdir’ path from config.yaml concatenated with the path in the fastq file list is correct.\
\
  For example, the file “/home/users/TEST/sequencing/exp01/fastqs/TEST_R1_001.fastq.gz” could be accurately described with the following combinations (and plenty others!):

| fastqdir:	(set in config.yaml) |	fastq file list: | 
| --- | --- |
| “”	|	“/home/users/TEST/sequencing/exp01/fastqs/TEST_R1_001.fastq.gz” |
| “/home/users/TEST/sequencing/exp01/fastqs/”	| “TEST_R1_001.fastq.gz” |

__Note:__ Periods in the samplenames are okay for this demultiplexing process, but are not for downstream analysis with some tools including DADA2. 

2. **Fastq file list:**\
  **Location:** The fastq file list should be located within the ‘config’ directory, and the file name should be updated in the fastqlist field of the ‘config.yaml’ file, (unless it is named ‘fastq.txt’, which is the default).\
   \
   **Contents:** The fastq list should be a tab-delimited text file, with the first column including the path to read 1, the second column including the path to read 2, and the final column including the shortened file name. This table should have headers of ‘read1’, ‘read2’, and ‘file’. For the file name, we recommend a format such as “{RunName}-{round2plate}-{well}”, where ‘RunName’ can be anything without underscores, spaces, or periods, and the next two terms specify the plate identifier for the round 2 barcodes and the well number respectively. It is essential that the ‘file’ field in the fastq file list matches the first part of the ‘filename’ field in the Samplesheet.\
   \
  The fastq file list for the test files is shown below:

| read1	|	read2	|	file |
| ------| ---- | ---- |
| ../fastq_data/test_inputs/KKRP-001_S441_R1_001.fastq.gz	| ../fastq_data/test_inputs/KKRP-001_S441_R2_001.fastq.gz | 15mc-003-P08B01-A01 |
| ../fastq_data/test_inputs/KKRP-002_S442_R1_001.fastq.gz	| ../fastq_data/test_inputs/KKRP-002_S442_R2_001.fastq.gz	| 15mc-003-P08B01-A02 |
| ../fastq_data/test_inputs/KKRP-003_S443_R1_001.fastq.gz	| ../fastq_data/test_inputs/KKRP-003_S443_R2_001.fastq.gz	| 15mc-003-P08B01-A03 |
| ../fastq_data/test_inputs/KKRP-004_S444_R1_001.fastq.gz	| ../fastq_data/test_inputs/KKRP-004_S444_R2_001.fastq.gz	| 15mc-003-P08B01-A04 |


3. **Samplesheet:**\
   **Location:** The samplesheet should be included in the ‘config’ directory, the ‘samplesheet’ path in the ‘config.yaml’ file should be updated to reflect the samplesheet’s name, unless it has the default name ‘samplesheet.txt’.\
   \
  **Contents:** The sample sheet will contain metadata for all the samples as a tab-delimited table (.tsv or .txt). This file should contain a header as the first row, and must include the following columns: ‘filename’, ‘sample’, and ‘group’. Additional columns can be included in the table. For example, the test samplesheet has columns 'No',	'RunName',	'plate',	'platename',	'well',	'R1index/phase',	'R2 plate',	'our file name',	'filename',	'sample',	and 'group'. Any extra column names can be included as desired, but ‘filename’, sample’ and ‘group’ are necessary.\
  The ‘sample’ column should contain the name that each individual sample will take after demultiplexing, and should not contain underscores or periods.\
  The ‘group’ column can contain any group identifier (without spaces or slashes). Samples in different groups will be output into different subdirectories within the ‘trimmed’ directory at the end of the run. If you don’t need reads separated, use one group specifier for all samples, or just leave the column blank (but do keep the ‘group’ header).\
  The ‘filename’ column should have the format:\
  `{RunName}-{round2plate}-{well}-L{round1index}`\
  with the ‘{RunName}-{round2plate}-{well}’ portion matching the corresponding ‘file’ entries in the fastq file list.\
  The 'round1index' should match the 'phase' entry in the indexfordemux.sh table.\
\
 Some of the columns from the test sample sheet are shown below:

| No	| RunName	| plate	| platename	| well	| R1index/phase	| R2 plate | our file name	| filename	| sample	| group |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 385	| 15mc-003	| P5	| plate5	| A01	| 4	| P08B01	| plateP-wellA1	| 15mc-003-P08B01-A01-L4	| P5-A01-plateP-wellA1	| group1 |
| 386	| 15mc-003	| P5	| plate5	| A02	| 4	| P08B01	| plateP-wellA2	| 15mc-003-P08B01-A02-L4	| P5-A02-plateP-wellA2	| group1 |
| 481	| 15mc-003	| P6	| plate6	| A01	| 5	| P08B01	| plateW-wellA5	| 15mc-003-P08B01-A01-L5	| P6-A01-plateW-wellA5	| group2 |
| 482	| 15mc-003	| P6	| plate6	| A02	| 5	| P08B01	| plateW-wellA6	| 15mc-003-P08B01-A02-L5	| P6-A02-plateW-wellA6	| group2 |



## Building Images
### Building with Docker:
To build a Docker image, ensure the necessary field are arranged properly, and build from the Dockerfile:
1. All necessary files are included in ‘docker-emux.zip’ in **ZENODO LOCATION**. Download and unzip the file. The following file structure should be created:\
\
<img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/docker_demux_fileStructure.png?raw=true" alt="Alt Text" width="400" height="400">\
\
‘Dockerfile’ and ‘requirements.txt’ are needed for the building process, and the fastq_data directory contains example data for the test. All of the code is contained within the ‘16s-demux’ directory, and outputs will be generated there as well.
4. Move to the directory containing the Dockerfile and run the following:\
‘docker build -t {name}:{version} .’\
\
The image will be created locally with the name and version provided following -t. The build should take a few minutes, and if it is completed successfully, the the final output will look something like this:\
\
 <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/docker_buildSuccessOutput.png?raw=true" alt="Alt Text" width="400" height="400">\
\
__Note:__ This step will likely require more resources than are available on a HPC login node, so be sure you are on a compute node or use a job manager to allocate resources.


6. Now that the image has been build, you can run the demultiplexing analysis within it using the command ‘docker run -it {name}:{version}’


### Building with Singularity/Apptainer:
To build a singularity/Apptainer image, ensure the necessary files are arranged properly and build from the '.def' file:

1. All necessary files are included in ‘demux.zip’ in **ZENODO LOCATION**. Download and unzip the file. The following file structure should be created:\
\
 <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/sing_demux_fileStructure.png?raw=true" alt="Alt Text" width="400" height="400">\
\
‘16s-demux.def’ and ‘requirements.txt’ are both needed for the building process, and the fastq_data contains example data for the test. All of the code is contained within the ‘16S-demux’ directory, and outputs will be generated there as well.


2. Move to the directory containing the def file (16s-demux.def) and run the following:\
`singularity build demux-image.sif 16s-demux.def`\
\
The image will be created locally and a file ‘demux-image.sif’ will be created in the current working directory. The build should take less than 10 minutes, and if it is completed successfully, the the final output will look something like this:\
<img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/buildSuccessOutput.png?raw=true" alt="Alt Text" width="750" height="400">\
\
Once completed, the file 'demux-image.sif' should be found in the current working directory.

__Note:__ This step will likely require more resources than are available on a HPC login node, so be sure you are on a compute node or use a job manager to allocate resources. A template slurm script is included in the demux.zip file, named 'slurmBuild.sh'. You may choose to change the node/memory/timing steps based on your system.

3. Now that the image has been built, you can run the demultiplexing analysis within it as described in ‘> Using Singularity/Apptainer’ using the image ‘demux-image.sif’. 



## Custom Primers
The instructions and code above all assume the primers and indexes used are the standard 16S V4 sets presented in the paper. If custom primers/indexes are used, several changes will need to be made.

1. **Update config/indexfordemux.txt**\
Make sure the indexfordemux.txt file (which contains the indicies used for demultiplexing R1 indicies) included in the config directory and specified in the 'config.yaml' file corresponds to the appropriate region. We provide primer sequences and corresponding 'indexfordemux.txt' files for the following 16S regions: V1 - V2, V1 - V3, V2 - V3, V3, V3 - V4, V4 - V5, V5, V5 - V7, V6, V6 - V7, V6 - V8, and V7 - V9. These are provided in the 'other indexfordemux' folder here, along with an Excel worksheet showing how these were derived, with a template for making additional primers ('other indexfordemux.xlsx').\
\
If you want to amplify another region, you can use this template to make a custom 'indexfordemux.sh' file. (To make a custom 'indexfordemux.sh' file in the 'other indexfordemux.xlsx' file, replace the entries in 'geneF' with the first part of the gene sequence (5' - 3', coding strand) and the entries in 'geneR' with the end of the gene sequence (5' - 3', template strand)).\
**(include image schematic of the reads with the index/spacer/genespecific region, etc?)**\
\
As a reminder, the final structure of reads after the library prep will look something like this (lengths not to scale):\
  <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/primerStructureImage.png?raw=true" alt="Alt Text" width="400" height="400">\
The round 1 indexes have variable lengths or ‘phases’ as shown in the table and image below:

| phase | variable FP | variable RP |
| --- | --- | --- |
| 0 |  |ATGGACT |
| 1 | T | GCTAGC |
| 2 | GG  | TGACT |
| 3 | ACT | CGGT |
| 4 | TAAC | GTA |
| 5 | CAGTC | AA |
| 6 | ATCGAT | C|
| 7 | GCAAGTC  | |

 <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/phasedPrimerImage.png?raw=true" alt="Alt Text" width="400" height="400">

However, during the demultiplexing, we treat the reads as if they have 7 base pair indexes on both ends. Any of the 7 base pairs that are not filled in with the index will be the spacer/gene-specific primer sequence. The table below shows the default indexes, with the actual index base pairs underlined, the spacer base pairs in lowercase, and the gene-specific primer regions uppercase and bolded. (The ‘bc’ or barcode column is simply the concatenated strings of read1index and read2index). The index, spacer, and gene specific regions will need to be edited according to the changes made.\

| phase | read1index | read2index | bc |
| --- | --- | --- | --- |
| 0 | cagt**AGA** | <ins>ATGGACT</ins> | CAGTAGAATGGACT | 
| 1 | <ins>T</ins>cagt**AG** | <ins>GCTAGC</ins>a | TCAGTAGGCTAGCA | 
| 2 | <ins>GG</ins>cagt**A** | <ins>TGACT</ins>at | GGCAGTATGACTAT | 
| 3 | <ins>ACT</ins>cagt | <ins>CGGT</ins>atc | ACTCAGTCGGTATC | 
| 4 | <ins>TAAC</ins>cag | <ins>GTA</ins>atcc | TAACCAGGTAATCC | 
| 5 | <ins>CAGTC</ins>ca | <ins>AA</ins>atcc**T** | CAGTCCAAAATCCT | 
| 6 | <ins>ATCGAT</ins>c | <ins>C</ins>atcc**TA** | ATCGATCCATCCTA | 
| 7 | <ins>GCAAGTC</ins> | atcc**TAC** | GCAAGTCATCCTAC |

If the same primer design and index scheme is used, and only the gene-specific region is changed, only the gene-specific regions within the indices will need to be updated. For the primers above, the gene of interest starts with 'AGA...' and ends with 'GTA' on the forward strand, hence the regions of homology include 'AGA' and 'TAC', both in the 5' to 3' direction.\
\
For a gene reading 'ATG ... CGT', the regions of homology within primers would become 'ATG' and 'ACG', both in the 5' to 3' direction. Thus the indexfordemux table should be edited to:

| phase | read1index | read2index | bc |
| --- | --- | --- | --- |
| 0 | cagt**ATG** | <ins>ATGGACT</ins> | CAGTAGAATGGACT | 
| 1 | <ins>T</ins>cagt**AT** | <ins>GCTAGC</ins>a | TCAGTAGGCTAGCA | 
| 2 | <ins>GG</ins>cagt**A** | <ins>TGACT</ins>at | GGCAGTATGACTAT | 
| 3 | <ins>ACT</ins>cagt | <ins>CGGT</ins>atc | ACTCAGTCGGTATC | 
| 4 | <ins>TAAC</ins>cag | <ins>GTA</ins>atcc | TAACCAGGTAATCC | 
| 5 | <ins>CAGTC</ins>ca | <ins>AA</ins>atcc**A** | CAGTCCAAAATCCT | 
| 6 | <ins>ATCGAT</ins>c | <ins>C</ins>atcc**AC** | ATCGATCCATCCTA | 
| 7 | <ins>GCAAGTC</ins> | atcc**ACG** | GCAAGTCATCCTAC |

Note that only the gene-specific regions have changed, and the index and spacer sequences are identical to the initial set.

__Note:__ We recommend avoiding any mixed base characters such as ‘W’ or ‘N’ in the first three positions of your gene specific primer. If such bases are included, you will need to include extra entries in the indexfordemux.txt table, one for each potential base (e.g., for a ‘W’, one version should have an ‘A’ and one should have a ‘T’). Each sample with a mixed base index should thus be included twice in the samplesheet, and the output files will then need to be merged downstream (either after trimming or after subsequent analyses).

2. **Update the index lengths in config/config.yaml**\ 
Within the ‘config/config.yaml’ file, the index/primer lengths may need to be updated. The “lenR1index” and “lenR2index” should be set to the longest version of these (e.g., in the provided primer/index set, the longest sequence of index bases is 7, so the value is set to 7). The “lenR1primer” and “lenR2primer” values should be set to equal the length of the gene-specific primer sequence plus the length of the spacer. For the 16S sets with provided indexfordemux.txt files (V1 - V2, V1 - V3, V2 - V3, V3, V3 - V4, V4 - V5, V5, V5 - V7, V6, V6 - V7, V6 - V8, and V7 - V9), the lengths do not need to be changed; the defaults below are correct.

| Region | length | 
| --- | --- | 
| lenR1primer |  23 | 
| lenR2primer | 24 | 
| lenR1index | 7 | 
| lenR2index | 7 |


## Troubleshooting
There are a couple common issues you may hit while demultiplexing described below. 

1. **Errors in inputs**\
   Getting the inputs perfect may be an iterative process! To help with this, the first step of the pipeline will check that the file paths are valid, there are no disallowed symbols in file names, and the file names in the samplesheet and fastqlist match. Any warning messages will be output in '16s-demux/workflow/out/inputCheck_log.txt' (or '16s-demux/workflow/test_out/inputCheck_log.txt' for tests). If the log only reads 'All done!' no issues were found, so common input errors shouldn't be causing issues.\
\
Another tool for validating inputs is the dry run in Snakemake. To conduct a dryrun, use the command:\
`snakemake -n`.\
\
This will determine the jobs to be run and inform you if there are missing inputs. To run this, you will need to be within a Docker container or a Singularity shell.

3. **Inadequate resources**\
   Another potential issue is inadequate allocation of resources. For building images and running the test analysis, 4GB of memory and one core are sufficient, but larger allocations may speed up these processes. If insufficient memory is allocated, the output errors may be hard to trace back to memory, but will reliably occur at the same stage.


## Test run full outputs
Within ‘workflow’, there should be a ‘test_out’ directory, containing ‘demux’ and ‘trimmed’ directories. Under ‘demux’, there should be four sets of 3 files; each should have 1 * .extract.log file, and 2 .fastq.gz files. There should also be two directories: ‘R1’ and ‘R2’. The contents of ‘R1’ and ‘R2’ should have the same filenames (but correspond to forward and reverse reads for the specified sample). For each sample in the above, there should be 8 files, ending with ‘-L * .fastq.gz’, where * is an integer from 0 to 7:

 <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/TestSuccessOutputs_1.png?raw=true" alt="Alt Text" height="600">


Within the ‘trimmed’ directory, there should be two subdirectories, ‘group1’ and 'group2', and within each of those, there should be directories ‘R1’, ‘R2’, ‘removed’, and two summary files: ‘lowReadsSummary.txt’, and ‘summary.txt’: 

 <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/testSuccessOutputs_2.png?raw=true" alt="Alt Text" height="600">

If these files are generated in ‘trimmed’, the run has been successful!

