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
