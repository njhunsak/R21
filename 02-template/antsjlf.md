---
title: ANTs Joint Label Fusion
toc: true
weight: 5
---

## Job Script

Create a job script:

``` bash
vi ~/scripts/R21/template-JLF.sh
```

Copy and paste:

``` bash
#!/bin/bash

#SBATCH --time=50:00:00   # walltime
#SBATCH --ntasks=2   # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=32768M  # memory per CPU core

# LOAD ENVIRONMENTAL VARIABLES
var=`id -un`
export ANTSPATH=/fslhome/$var/apps/ants/bin/
PATH=${ANTSPATH}:${PATH}

# INSERT CODE, AND RUN YOUR PROGRAMS HERE
DATA_DIR=~/templates/R21
TEMPLATE_DIR=~/templates/OASIS-TRT-20_volumes
LABEL_DIR=~/templates/OASIS-TRT-20_DKT31_CMA_labels_v2
cd $DATA_DIR
mkdir labels
${ANTSPATH}/antsJointLabelFusion.sh \
-d 3 \
-c 5 -j 2 -y s -k 1 \
-o ${DATA_DIR}/labels/ \
-t ${DATA_DIR}/template.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-1/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-1_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-2/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-2_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-3/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-3_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-4/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-4_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-5/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-5_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-6/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-6_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-7/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-7_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-8/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-8_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-9/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-9_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-10/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-10_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-11/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-11_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-12/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-12_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-13/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-13_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-14/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-14_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-15/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-15_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-16/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-16_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-17/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-17_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-18/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-18_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-19/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-19_DKT31_CMA_labels.nii.gz \
-g ${TEMPLATE_DIR}/OASIS-TRT-20-20/t1weighted.nii.gz -l ${LABEL_DIR}/OASIS-TRT-20-20_DKT31_CMA_labels.nii.gz
```

## Submit

Submit job script:

``` bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/$var
sbatch \
-o ~/logfiles/$var/output.txt \
-e ~/logfiles/$var/error.txt \
~/scripts/R21/template-JLF.sh
```

## Cook Tissue Priors

Clean up tissue segmentation:

```bash
cd ~/templates/R21/
C3DPATH=~/apps/c3d/bin

# Split label file
${C3DPATH}/c3d template_6labels.nii.gz -threshold 1 1 1 0 -o label1.nii.gz
${C3DPATH}/c3d template_6labels.nii.gz -threshold 2 2 2 0 -o label2.nii.gz
${C3DPATH}/c3d template_6labels.nii.gz -threshold 3 3 3 0 -o label3.nii.gz
${C3DPATH}/c3d template_6labels.nii.gz -threshold 4 4 4 0 -o label4.nii.gz
${C3DPATH}/c3d template_6labels.nii.gz -threshold 5 5 5 0 -o label5.nii.gz
${C3DPATH}/c3d template_6labels.nii.gz -threshold 6 6 6 0 -o label6.nii.gz

# Create Label 4 file from JLF
${C3DPATH}/c3d labels/Labels.nii.gz -threshold 51 52 4 0 -o temp4.nii.gz
${C3DPATH}/c3d labels/Labels.nii.gz -threshold 12 13 4 0 temp4.nii.gz -add -o temp4.nii.gz
${C3DPATH}/c3d labels/Labels.nii.gz -threshold 50 50 4 0 temp4.nii.gz -add -o temp4.nii.gz
${C3DPATH}/c3d labels/Labels.nii.gz -threshold 11 11 4 0 temp4.nii.gz -add -o temp4.nii.gz
${C3DPATH}/c3d labels/Labels.nii.gz -threshold 60 60 4 0 temp4.nii.gz -add -o temp4.nii.gz
${C3DPATH}/c3d labels/Labels.nii.gz -threshold 28 28 4 0 temp4.nii.gz -add -o temp4.nii.gz
${C3DPATH}/c3d labels/Labels.nii.gz -threshold 58 58 4 0 temp4.nii.gz -add -o temp4.nii.gz
${C3DPATH}/c3d labels/Labels.nii.gz -threshold 26 26 4 0 temp4.nii.gz -add -o temp4.nii.gz
${C3DPATH}/c3d labels/Labels.nii.gz -threshold 49 49 4 0 temp4.nii.gz -add -o temp4.nii.gz
${C3DPATH}/c3d labels/Labels.nii.gz -threshold 10 10 4 0 temp4.nii.gz -add -o temp4.nii.gz
${C3DPATH}/c3d temp4.nii.gz label4.nii.gz -add -threshold 1 inf 4 0 -o label4.nii.gz
${C3DPATH}/c3d label4.nii.gz -holefill 4 1 -o label4.nii.gz
${C3DPATH}/c3d label4.nii.gz -smooth 0.5x0.5x0.5vox -o label4.nii.gz
${C3DPATH}/c3d label4.nii.gz -threshold 1 inf 4 0 -o label4.nii.gz

# Create Label 5 file from JLF
${C3DPATH}/c3d labels/Labels.nii.gz -threshold 16 16 5 0 -o temp5.nii.gz
${C3DPATH}/c3d temp5.nii.gz label5.nii.gz -add -threshold 1 inf 5 0 -o label5.nii.gz
${C3DPATH}/c3d label5.nii.gz -holefill 5 1 -o label5.nii.gz
${C3DPATH}/c3d label5.nii.gz -smooth 0.5x0.5x0.5vox -o label5.nii.gz
${C3DPATH}/c3d label5.nii.gz -threshold 1 inf 5 0 -o label5.nii.gz

# Combine Labels
${C3DPATH}/c3d label1.nii.gz label2.nii.gz -add -clip 0 2 -o temp.nii.gz
${C3DPATH}/c3d temp.nii.gz label3.nii.gz -add -clip 0 3 -o temp.nii.gz
${C3DPATH}/c3d temp.nii.gz label4.nii.gz -add -clip 0 4 -o temp.nii.gz
${C3DPATH}/c3d temp.nii.gz label5.nii.gz -add -clip 0 5 -o temp.nii.gz
${C3DPATH}/c3d temp.nii.gz label6.nii.gz -add -clip 0 6 -o temp.nii.gz
```

Once you are sure everything worked:

```bash
cd ~/templates/R21/
mv temp.nii.gz template_6labels.nii.gz
rm label*.nii.gz
rm temp4.nii.gz
rm temp5.nii.gz
```

Finish by recreating your tissue priors:

```bash
cd ~/templates/R21/
for(( j=1; j<=6; j++ )); do
	prior=priors/priors${j}.nii.gz
	~/apps/ants/bin/ThresholdImage 3 template_6labels.nii.gz $prior $j $j 1 0
	~/apps/ants/bin/SmoothImage 3 $prior 1 $prior
done
```