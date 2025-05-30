# gadi_cheatsheet
- user guides https://opus.nci.org.au/
- username and login set up, and project information on https://opus.nci.org.au/
	- Invited to two projects from Jo, dz70 and tr07

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

example PBS header 1
```bash
#!/bin/bash

#PBS -l ncpus=1
#PBS -l mem=16GB
#PBS -l jobfs=2GB
#PBS -q copyq
#PBS -P tr07
#PBS -l walltime=24:00:00
#PBS -l storage
```

example PBS header 2

```bash
#PBS -l mem=16GB
#PBS -l jobfs=2GB
#PBS -q normal
#PBS -P dz70
#PBS -l walltime=10:00:00
#PBS -l storage=scratch/tr07+gdata/dz70
#PBS -l wd


```
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
some examples

```bash
mdss -P tr07 ls -lh Nanopore_Level_1
```

running mdss get as a PBS job on copyq
```bash
#!/bin/bash

#PBS -l ncpus=1
#PBS -l mem=16GB
#PBS -l jobfs=2GB
#PBS -q copyq
#PBS -P tr07
#PBS -l walltime=10:00:00
#PBS -l storage=massdata/tr07+scratch/dz70
#PBS -l wd

cd /scratch/dz70/dt9853/from_mdss/Nanopore_Level_1

mdss -P tr07 get -r Nanopore_Level_1/raw_data

echo "finished 1"

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
- addition config required for running on gadi
- 

- the institutional config set most of the environment, but extra parameters need to be set with this
- Documentation : https://github.com/nf-core/configs/blob/master/docs/nci_gadi.md
```
process {
  executor = 'pbspro'
  queue = 'normal'
  project = 'dz70'
  storage = 'scratch/dz70+gdata/dz70'
       
  withName: 'task1' {
    cpus = 2
    time = '1d'
    memory = '8GB'
  }
}
```



# Setting default project
modify ~/.config/gadi-login.conf
```
PROJECT tr07
SHELL /bin/bash
```
- this is read in nextflow processes

## copying between projects
- pbs header for moving data between projects
- submit with qsub move.sh

```bash
#PBS -l mem=16GB
#PBS -l jobfs=2GB
#PBS -q normal
#PBS -P dz70
#PBS -l walltime=10:00:00
#PBS -l storage=scratch/tr07+gdata/dz70
#PBS -l wd
mv -v /scratch/tr07/ /g/data/dz70/
```

# persistent sessions
- a substitute for screen on gadi, for long running jobs with little memory
- help page https://opus.nci.org.au/spaces/Help/pages/241927941/Persistent+Sessions...

```bash
# Starting Sessions
persistent-sessions start [ -p <project> ] <name>

# Connecting to Sessions
ssh <name>.<user>.<project>.ps.gadi.nci.org.au

# Listing Running Sessions
persistent-sessions list [ -p <project> ] [ <uuid> ]

# Terminating Sessions
persistent-sessions kill <uuid>

# finding session names
ls -l  ~/.persistent-sessions/

```

# setting up rclone
- followed these instructions
- https://rclone.org/sftp/

installed rclone on my mac
```bash
cd ~/Downloads
sudo -v ; curl https://rclone.org/install.sh | sudo bash
# ran config setup, followed prompts
rclone config
```
-this was the summary messeage ofter setting up config
```
Configuration complete.
Options:
- type: sftp
- host: gadi.nci.org.au
- user: dt9853
- pass: *** ENCRYPTED ***
- ask_password: true
- shell_type: unix
- known_hosts_file: ~/.ssh/known_hosts
```
pw: same as gadi

- example commands after intallation and config setup
```bash
rclone lsd gadi:/
rclone mkdir gadi:/home/566/dt9853/rclonetest
rclone ls gadi:/scratch/dz70/dt9853/projects/20250408_P53sra_AlanaWelm_RNAseq/nf_RNAseq/outs/star_salmon/bams
```

- made mountpoint on mac with rclone
- just mounting /scratch
```bash
mkdir -p ~/gadi_rclone_mount/scratch_dz70_projects
cd  ~/gadi_rclone_mount/scratch_dz70_projects

rclone mount gadi:/scratch/dz70/dt9853/projects/ ~/gadi_rclone_mount/scratch_dz70_projects

```
- got this error message from FUSE
"Please open the Privacy & Security System Settings and allow loading system software from developer "Benjamin Fleischer". "
- had to change systems settings, following prompts, this also required a reboot of the mac in recovery mode/security utility to allow apps from developers (this was tedious)


# Rstudio with gadi
- adapted from Elyssa's notes

# Setting up my R 4.2 to try out some multi-omics clustering analysis options on Gadi

```bash
# needed to do the following to overcome a compiling related error:
#mkdir /scratch/[PROJECT]/[USER]/.R
mkdir /scratch/dz70/dt9853/.R
nano /scratch/dz70/dt9853/.R/Makevars
## add the following line and save file;
CXXFLAGS += -wd308

# Set up a user specific R_Lib
# something like this:
#mkdir /g/data/[PROJECT]/[USER]/R_Libs
#mkdir /g/data/[PROJECT]/[USER]/R_Libs/[R.VERSION]
mkdir /g/data/dz70/dt9853/Compute_Assets/Software/R_Libs/
mkdir  /g/data/dz70/dt9853/Compute_Assets/Software/R_Libs/4.2.1

# set user specific dir for RStudio server session temp data
# this prevents the default home directory from getting full and running out of storage as it only has 10Gb allocated.
#mkdir /scratch/[PROJECT]/[USER]/xdg_data_home
mkdir /scratch/dz70/dt9853/xdg_data_home

## need to load specific version of the intel compiler to be compatible with your R version
module load R/4.2.1
module load intel-compiler/2021.6.0

#Start R 
R

#setup your .libPath() so R can see both the shared library and your personal user R library
## see default libPath;
.libPaths()
## Add your user specific libPath;
.libPaths(c("/g/data/dz70/dt9853/Compute_Assets/Software/R_Libs/4.2.1", "/apps/R/4.2.1/lib64/R/library"))
R_MAKEVARS_USER <- "/scratch/dz70/dt9853/.R/Makevars"

## Install packages
## installing requirements for shinyngs
install.packages("remotes")
install.packages("markdown")
remotes::install_github("pinin4fjords/shinyngs")

## after sucessful install, quit and save your workspace
quit()

## respond "y" when promped to save.

# head over to the ARE dashboard and navigate to the RStudio server app
# https://are.nci.org.au/pun/sys/dashboard > "interactive apps" > "RStudio"
#in "Advanced Options"; under the "Modules" heading you can add a space-separated list of modules to load, you'll need to load the same version of R you installed your packages to;
R/4.2.1 intel-compiler/2021.6.0

# Also under the "Environment variables" heading, you can add a space-separated list of environment variables to define (via pbs 'qsub -v') e.g. NAME="VALUE".
# you will need to specify ther Dirs for your auto generated RStudio session temp data to be written and your R_LIBS_USER for your user specific packages;
XDG_DATA_HOME="/scratch/dz70/dt9853/xdg_data_home",R_LIBS_USER="/g/data/dz70/dt9853/Compute_Assets/Software/R_Libs/4.2.1"

# make sure project is what you want, e.g. dz70
# scratch/dz70+gdata/dz70
# also just incase put your "Jobfs size" as 100GB

# on https://are.nci.org.au, after job is requested, a 'connect to RStudio Server' button appears

# should be able to see the job submitted by PBS
qstat
```

# Data Mover Node
- copying data using the gadi data mover node example

```bash
scp -v dt9853@gadi-dm.nci.org.au:/scratch/dz70/dt9853/for_filesender/Hi-C_Level_1.tar ./
```
