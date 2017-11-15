---
title: Use Population Template
toc: true
weight: 2
---

## Job Script

Create job script:

```bash
vi ~/scripts/R21/antsCT_R21_job.sh
```

Copy and paste:

```bash
#!/bin/bash

#SBATCH --time=10:00:00   # walltime
#SBATCH --ntasks=1   # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=16384M  # memory per CPU core

# LOAD ENVIRONMENTAL VARIABLES
var=`id -un`
export ANTSPATH=/fslhome/$var/apps/ants/bin/
PATH=${ANTSPATH}:${PATH}

# INSERT CODE, AND RUN YOUR PROGRAMS HERE
DATA_DIR=~/compute/images/R21/${1}/
TEMPLATE_DIR=~/templates/R21/
mkdir -p ~/compute/analyses/R21/antsCT_R21/${1}/
~/apps/ants/bin/antsCorticalThickness.sh \
-d 3 \
-a ${DATA_DIR}/str/t1.nii.gz \
-e ${TEMPLATE_DIR}/template.nii.gz \
-t ${TEMPLATE_DIR}/template_BrainCerebellum.nii.gz \
-m ${TEMPLATE_DIR}/template_BrainCerebellumProbabilityMask.nii.gz \
-f ${TEMPLATE_DIR}/template_BrainCerebellumExtractionMask.nii.gz \
-p ${TEMPLATE_DIR}/priors/priors%d.nii.gz \
-q 1 \
-n 6 \
-x 6 \
-o ~/compute/analyses/R21/antsCT_R21/${1}/
```

## Batch Script

Create  batch script:

```bash
vi ~/scripts/R21/antsCT_R21_batch.sh
```

Copy and paste:

```bash
#!/bin/bash
for subj in $(ls ~/compute/images/R21/); do
    sbatch \
    -o ~/logfiles/${1}/output_${subj}.txt \
    -e ~/logfiles/${1}/error_${subj}.txt \
    ~/scripts/R21/antsCT_R21_job.sh \
    ${subj}
    sleep 1
done
```

## Submit

Submit jobs:

```bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/$var
sh ~/scripts/R21/antsCT_R21_batch.sh $var
```

## Sync

Sync data:

```bash
rsync -rauv \
intj5@ssh.fsl.byu.edu:/fslhome/intj5/compute/analyses/R21/antsCT_R21 \
/Volumes/wasatch/data/analyses/R21/
```