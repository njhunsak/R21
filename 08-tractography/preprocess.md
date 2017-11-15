---
title: DWI Preprocessing Steps
toc: true
weight: 3
---

## DICOM to NIFTI

On my local computer:

```bash
## DICOM TO NIFTI
for i in $(find /Volumes/wasatch/data/images/R21/ -type d -name "*DTI_64_Directions"); do
    cd $(dirname $(dirname $i))/dti
    pwd
    mrconvert $i dwi.mif
done

## MASK
for i in $(find /Volumes/wasatch/data/images/R21/ -type f -name "dwi.nii.gz"); do
    cd $(dirname $i)
    pwd
    bet $i dwi -f 0.25 -g 0 -m -n
done

## VERIFY MASK
for i in $(find /Volumes/wasatch/data/images/R21/ -type f -name "dwi.mif"); do
    cd $(dirname $i)
    mrview dwi.mif -roi.load dwi_mask.nii.gz -roi.opacity 0.25
done
```

## Sync Data

```bash
rsync \
-rauv \
--exclude="dicom" \
--exclude=".*" \
/Volumes/wasatch/data/analayses/R21/mrtrix/ \
intj5@ssh.fsl.byu.edu:~/compute/analyses/R21/mrtrix/
```

## Job Script

Create script:

```bash
vi ~/scripts/R21/mrtrix_job.sh
```

Copy and paste code:

```bash
#!/bin/bash

#SBATCH --time=06:00:00   # walltime
#SBATCH --ntasks=1   # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=32768M  # memory per CPU core

# LOAD ENVIRONMENTAL VARIABLES
var=`id -un`
export PATH="/zhome/intj5/apps/mrtrix3/bin:$PATH"

# INSERT CODE, AND RUN YOUR PROGRAMS HERE
DATA_DIR=~/compute/analyses/R21/mrtrix/${1}/

## DENOISE
dwidenoise \
-mask ${DATA_DIR}/dwi_mask.nii.gz \
${DATA_DIR}/dwi.mif \
${DATA_DIR}/dwi_denoised.mif

## PREPROCESS
dwipreproc \
${DATA_DIR}/dwi_denoised.mif \
${DATA_DIR}/dwi_preproc.mif \
-rpe_none \
-pe_dir AP

## N4 BIAS FIELD CORRECTION
dwibiascorrect \
${DATA_DIR}/dwi_preproc.mif \
${DATA_DIR}/dwi_biascorrected.mif \
-mask ${DATA_DIR}/dwi_mask.nii.gz \
-fsl

## CREATE FA, RD, AND AD IMAGE
dwi2tensor \
${DATA_DIR}/dwi_biascorrected.mif \
${DATA_DIR}/dwi_tensor.mif \
-mask ${DATA_DIR}/dwi_mask.nii.gz

tensor2metric \
-adc ${DATA_DIR}/md.nii.gz \
${DATA_DIR}/dwi_tensor.mif \
-mask ${DATA_DIR}/dwi_mask.nii.gz

tensor2metric \
-fa ${DATA_DIR}/fa.nii.gz \
${DATA_DIR}/dwi_tensor.mif \
-mask ${DATA_DIR}/dwi_mask.nii.gz

tensor2metric \
-ad ${DATA_DIR}/ad.nii.gz \
${DATA_DIR}/dwi_tensor.mif \
-mask ${DATA_DIR}/dwi_mask.nii.gz

tensor2metric \
-rd ${DATA_DIR}/rd.nii.gz \
${DATA_DIR}/dwi_tensor.mif \
-mask ${DATA_DIR}/dwi_mask.nii.gz

## CHECK TENSORS
dwi2response \
tournier \
${DATA_DIR}/dwi_biascorrected.mif \
${DATA_DIR}/dwi_response.txt

dwi2fod \
csd \
${DATA_DIR}/dwi_biascorrected.mif \
${DATA_DIR}/dwi_response.txt \
${DATA_DIR}/dwi_fod.mif

## WARP FA TO MNI
antsRegistrationSyNQuick.sh \
-d 3 \
-m ${DATA_DIR}/fa.nii.gz \
-f ~/compute/templates/ICBM_FA.nii \
-o ${DATA_DIR}/fa_registered_ \
-t a
```

## Batch Script

Create script:

```bash
vi ~/scripts/R21/mrtrix_batch.sh
```

Copy and paste code:

```bash
#!/bin/bash
for subj in $(ls ~/compute/images/R21/); do
	sbatch \
	-o ~/logfiles/${1}/output_${subj}.txt \
	-e ~/logfiles/${1}/error_${subj}.txt \
	~/scripts/R21/mrtrix_job.sh \
	${subj}
	sleep 1
done
```

## Submit Jobs

```bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/$var
sh ~/scripts/R21/mrtrix_batch.sh $var
```

## Sync Data

```bash
rsync \
-rauv \
intj5@ssh.fsl.byu.edu:~/compute/analyses/R21/mrtrix/ \
/Volumes/wasatch/data/analayses/R21/mrtrix/
```

## Check Tensors

```bash
for i in $(find /Volumes/wasatch/data/images/R21/ -type f -name "dwi_fod.mif"); do
    cd $(dirname $i)
    mrview \
    dwi_biascorrected.mif \
    -odf.load_sh dwi_fod.mif \
    -fullscreen
done
```