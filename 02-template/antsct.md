---
title: ANTs Cortical Thickness
toc: true
weight: 4
---

## Job Script

Make an output directory for antsCorticalThickness.sh output files:

``` bash
mkdir -p ~/templates/R21/antsCT/
```

Create a job script:

``` bash
vi ~/scripts/R21/template-antsCT.sh
```

Copy and paste:

``` bash
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
DATA_DIR=~/templates/R21/
TEMPLATE_DIR=~/templates/OASIS-30_Atropos_template/
~/apps/ants/bin/antsCorticalThickness.sh \
-d 3 \
-a ${DATA_DIR}/template.nii.gz \
-e ${TEMPLATE_DIR}/T_template0.nii.gz \
-t ${TEMPLATE_DIR}/T_template0_BrainCerebellum.nii.gz \
-m ${TEMPLATE_DIR}/T_template0_BrainCerebellumProbabilityMask.nii.gz \
-f ${TEMPLATE_DIR}/T_template0_BrainCerebellumExtractionMask.nii.gz \
-p ${TEMPLATE_DIR}/Priors2/priors%d.nii.gz \
-q 1 \
-n 6 \
-x 6 \
-o ${DATA_DIR}/antsCT/

# COPY MASK
cp \
${DATA_DIR}/antsCT/BrainExtractionMask.nii.gz \
${DATA_DIR}/template_BrainCerebellumMask.nii.gz

# EXTRACT BRAIN IMAGE
${ANTSPATH}/ImageMath \
3 \
${DATA_DIR}/template_BrainCerebellum.nii.gz \
m \
${DATA_DIR}/template_BrainCerebellumMask.nii.gz \
${DATA_DIR}/template.nii.gz

# CONVERT MASK ROI TO PROBABILITY MASK
${ANTSPATH}/SmoothImage \
3 \
${DATA_DIR}/template_BrainCerebellumMask.nii.gz \
1 \
${DATA_DIR}/template_BrainCerebellumProbabilityMask.nii.gz

# DILATE MASK IMAGE TO GENERATE EXTRACTION MASK
~/apps/c3d/bin/c3d \
${DATA_DIR}/template_BrainCerebellumMask.nii.gz \
-dilate 1 28x28x28vox \
-o ${DATA_DIR}/template_BrainCerebellumExtractionMask.nii.gz

# DILATE MASK IMAGE TO GENERATE REGISTRATION MASK
~/apps/c3d/bin/c3d \
${DATA_DIR}/template_BrainCerebellumMask.nii.gz \
-dilate 1 18x18x18vox \
-o ${DATA_DIR}/template_BrainCerebellumRegistrationMask.nii.gz

# COPY TISSUE SEGMENTATION
cp \
${DATA_DIR}/antsCT/BrainSegmentation.nii.gz \
${DATA_DIR}/template_6labels.nii.gz

# COPY TISSUE PRIORS
mkdir ${DATA_DIR}/priors/
cp ${DATA_DIR}/antsCT/BrainSegmentationPosteriors1.nii.gz ${DATA_DIR}/priors/priors1.nii.gz
cp ${DATA_DIR}/antsCT/BrainSegmentationPosteriors2.nii.gz ${DATA_DIR}/priors/priors2.nii.gz
cp ${DATA_DIR}/antsCT/BrainSegmentationPosteriors3.nii.gz ${DATA_DIR}/priors/priors3.nii.gz
cp ${DATA_DIR}/antsCT/BrainSegmentationPosteriors4.nii.gz ${DATA_DIR}/priors/priors4.nii.gz
cp ${DATA_DIR}/antsCT/BrainSegmentationPosteriors5.nii.gz ${DATA_DIR}/priors/priors5.nii.gz
cp ${DATA_DIR}/antsCT/BrainSegmentationPosteriors6.nii.gz ${DATA_DIR}/priors/priors6.nii.gz
```

## Submit

Submit job script:

``` bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/$var
sbatch \
-o ~/logfiles/$var/output.txt \
-e ~/logfiles/$var/error.txt \
~/scripts/R21/template-antsCT.sh
```

## Sync

Sync template:

``` bash
rsync \
-rauv \
intj5@ssh.fsl.byu.edu:~/templates/R21/ \
/Volumes/data/templates/R21/
```
