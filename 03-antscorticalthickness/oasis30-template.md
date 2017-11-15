---
title: Use OASIS30 Template
toc: true
weight: 3
---

## Job Script

Create job script:

```bash
vi ~/scripts/R21/antsCT_OASIS30_job.sh
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
TEMPLATE_DIR=~/templates/OASIS-30_Atropos_template/
mkdir -p ~/compute/analyses/R21/antsCT_OASIS30/${1}/
~/apps/ants/bin/antsCorticalThickness.sh \
-d 3 \
-a ${DATA_DIR}/str/t1.nii.gz \
-e ${TEMPLATE_DIR}/T_template0.nii.gz \
-t ${TEMPLATE_DIR}/T_template0_BrainCerebellum.nii.gz \
-m ${TEMPLATE_DIR}/T_template0_BrainCerebellumProbabilityMask.nii.gz \
-f ${TEMPLATE_DIR}/T_template0_BrainCerebellumExtractionMask.nii.gz \
-p ${TEMPLATE_DIR}/Priors2/priors%d.nii.gz \
-q 1 \
-n 6 \
-x 6 \
-o ~/compute/analyses/R21/antsCT_OASIS30/${1}/
```

## Batch Script

Create  batch script:

```bash
vi ~/scripts/R21/antsCT_OASIS30_batch.sh
```

Copy and paste:

```bash
#!/bin/bash
for subj in $(ls ~/compute/images/R21/); do
  sbatch \
  -o ~/logfiles/${1}/output_${subj}.txt \
  -e ~/logfiles/${1}/error_${subj}.txt \
  ~/scripts/R21/antsCT_OASIS30_job.sh \
  ${subj}
  sleep 1
done
```

## Submit

Submit jobs:

```bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/$var
sh ~/scripts/R21/antsCT_OASIS30_batch.sh $var
```

## Sync

Sync data:

```bash
rsync -rauv \
intj5@ssh.fsl.byu.edu:/fslhome/intj5/compute/analyses/R21/antsCT_OASIS30 \
/Volumes/wasatch/data/analyses/R21/
```
