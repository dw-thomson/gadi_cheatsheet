# gadi_cheatsheet
- user guides https://opus.nci.org.au/

to log in
```bash
ssh dt9853@gadi.nci.org.au
```

check disk space allocation
```bash
nci_account
```
The home directory does not have much disk space
```
/home/566/dt9853
```
Jobs submitted from the /scratch directory
files are deleted every 3 months
```
/scratch/tr07/dt9853/
```
project specific drive, for data storage
```
/g/data/tr07
```
the project code, is how billing is determined, I've been given access to two groups
```bash
id
uid=23676(dt9853) gid=9174(tr07) groups=9174(tr07),566(U.Adel),9906(dz70)
```
# PBS command cheat sheet
- gadi uses the PBS job scheduler

List nodes and their features

```bash
pbsnodes
```
Run an interactive job on a node with 4 RTX 2080 GPUs, 8 CPUs, 90GB of RAM, for 2 hours

```bash
qsub -l ngpus=4 -l ncpus=8 -l mem=90G -l gputype=RTX2080 -l walltime=02:00:00   -I
```

Alternatively this can be compressed into

```bash
qsub -l ngpus=4:ncpus=8:mem=90G:gputype=RTX2080:walltime=02:00:00 -I
```

The -I flag indicates the interactive aspect of the job.

Cancel a job

```bash
qdel -p <job-id>
```
List information on queues

```bash
qstat
```
List current jobs owned by $USER

```bash
 qstat -u $USER
```
If you want to export variables from the current environment to the job, pass the -V flag which passes all current environment variables to the job's environment.

If you want to run one job after another, you can use the depend additional attribute, e.g.
```bash
qsub -W depend=afterok:<first-job-id> -l ncpus=8:ngpus=4:mem=90G:walltime=02:00:00 job-script.sh
```

# PBS job submission

You can also request space on multiple compute nodes by adding the directive -l jobfs to your job script, for example,
```
#PBS -l jobfs=100GB

```
The limit on $PBS_JOBFS is 400 GiB.The limit on $PBS_JOBFS is 400 GiB.


# mass data
Tape Filesystem - massdata

some simple commands that can help while navigating massdata. These commands can be run from the login nodes and begin with the prefix 'mdss', for example
```
man mdss
mdss put
mdss ls
mdss get
mdss mkdir
mdss rmdir
```

# check storage

```
nci_account -P tr07
```

# nf-core

# nf-core_setup_gadi
https://github.com/nf-core/configs/blob/master/docs/nci_gadi.md

nci_account -P tr07


running nf-core

- needs to run on /scratch , not home or gdata
- softlinks to data (e.g. fastqs, or references) not on scratch fail
- screen doesn't seems to work properly between different login nodes.
- I've been kicking of nextflow screipts via a pbs script submitted to the copyq queeue, since only the login/head nodes have internet access to pull down
- singularity cache dir, was set up on ~/ but this filled up very quick, 
- .nextflow directory with nf-core assets is still ~/.nextflow
- module load nextflow singularity

- example kickoff script
qsub run.pbs.sh

```
#!/bin/bash

#PBS -l ncpus=1
#PBS -l mem=16GB
#PBS -l jobfs=2GB
#PBS -q copyq
#PBS -P tr07
#PBS -l walltime=10:00:00
#PBS -l wd

module purge
module load nextflow singularity
BaseDir='/scratch/tr07/dt9853/projects/20250303_CDK46_RNAseq'
JobName='nf_RNAseq'
RunDir=${BaseDir}/${JobName}
OutDir=${RunDir}/outs_${JobName}

cd ${RunDir}
mkdir -p ${OutDir}

nextflow run nf-core/rnaseq \
        -profile singularity,nci_gadi \
        -r 3.14.0 \
        --input ${RunDir}/nfSampleSheet.csv \
        --outdir ${OutDir} \
        --fasta /scratch/tr07/dt9853/references/GRCh38_Ensembl/Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa.gz \
        --gtf /scratch/tr07/dt9853/references/GRCh38_Ensembl/Homo_sapiens.GRCh38.113.gtf.gz \
        --skip_dupradar \
        --skip_qualimap \
        --save_reference \
        -resume \
> std.out_${JobName}.txt

```


