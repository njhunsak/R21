---
title: recon-all
toc: true
weight: 2
---

## Job Script

Create job script:

```bash
vi ~/scripts/R21/freesurfer_job.sh
```

Copy and paste:

```bash
#!/bin/bash

#SBATCH --time=15:00:00   # walltime
#SBATCH --ntasks=1   # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=16384M  # memory per CPU core

# LOAD ENVIRONMENTAL VARIABLES
var=`id -un`
export FREESURFER_HOME=/fslhome/${var}/apps/freesurfer
source $FREESURFER_HOME/SetUpFreeSurfer.sh

# INSERT CODE, AND RUN YOUR PROGRAMS HERE
~/apps/freesurfer/bin/recon-all \
-subjid ${1} \
-i /fslhome/${var}/compute/images/R21/${1}/str/t1.nii.gz \
-wsatlas \
-all \
-hippocampal-subfields-T1 \
-brainstem-structures \
-sd /fslhome/${var}/compute/analyses/R21/FreeSurferv6/
```

## Batch Script

Create  batch script:

```bash
vi ~/scripts/R21/freesurfer_batch.sh
```

Copy and paste:

```bash
#!/bin/bash

for subj in $(ls ~/compute/images/R21/); do
	sbatch \
	-o ~/logfiles/${1}/output_${subj}.txt \
	-e ~/logfiles/${1}/error_${subj}.txt \
	~/scripts/R21/freesurfer_job.sh \
	${subj}
	sleep 1
done
```

## Submit Jobs

```bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/$var
sh ~/scripts/R21/freesurfer_batch.sh $var
```

## Sync Data

```bash
rsync -rauv \
intj5@ssh.fsl.byu.edu:/fslhome/intj5/compute/analyses/R21/FreeSurferv6 \
/Volumes/wasatch/data/analyses/R21/
```