---
title: T1 Preprocessing Steps
toc: true
weight: 3
---

## Job Script

Create script:

```bash
vi ~/scripts/R21/preprocess_job.sh
```

Copy and paste code:

```bash
#!/bin/bash

#SBATCH --time=00:15:00   # walltime
#SBATCH --ntasks=1   # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=32768M  # memory per CPU core

# LOAD ENVIRONMENTAL VARIABLES
var=`id -un`
export ANTSPATH=/fslhome/${var}/apps/ants/bin/
PATH=${ANTSPATH}:${PATH}

# INSERT CODE, AND RUN YOUR PROGRAMS HERE
DATA_DIR=~/compute/images/R21/${1}/

## ACPC ALIGN
~/apps/art/acpcdetect \
-M \
-o ${DATA_DIR}/str/acpc.nii \
-i ${DATA_DIR}/str/t1.nii

## CROP
~/apps/c3d/bin/c3d \
${DATA_DIR}/str/acpc.nii \
-trim 20vox \
-o ${DATA_DIR}/str/cropped.nii.gz

## N4 BIAS FIELD CORRECTION
~/apps/ants/bin/N4BiasFieldCorrection \
-v -d 3 \
-i  ${DATA_DIR}/str/cropped.nii.gz \
-o ${DATA_DIR}/str/n4.nii.gz \
-s 4 -b [200] -c [50x50x50x50,0.000001]

## RESAMPLE
~/apps/c3d/bin/c3d \
${DATA_DIR}/str/n4.nii.gz \
-resample-mm 1x1x1mm \
-o ${DATA_DIR}/str/resampled.nii.gz
```

## Batch Script

Create script:

```bash
vi ~/scripts/R21/preprocess_batch.sh
```

Copy and paste code:

```bash
#!/bin/bash
for subj in $(ls ~/compute/images/R21/); do
    sbatch \
    -o ~/logfiles/${1}/output_${subj}.txt \
    -e ~/logfiles/${1}/error_${subj}.txt \
    ~/scripts/R21/preprocess_job.sh \
    ${subj}
    sleep 1
done
```

## Submit Jobs

```bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/$var
sh ~/scripts/R21/preprocess_batch.sh $var
```

## Sync Data

```bash
rsync \
-rauv \
intj5@ssh.fsl.byu.edu:~/compute/images/R21/ \
/Volumes/wasatch/data/images/R21/
```

## Clean Up Directory

Once you have confirmed your resampled.nii.gz came out correctly:

```bash
find ~/compute/images/R21/ -type f -name "*.ppm" -exec rm {} \;
find ~/compute/images/R21/ -type f -name "t1_Crop_1.nii" -exec rm {} \;
find ~/compute/images/R21/ -type f -name "n4.nii.gz" -exec rm {} \;
find ~/compute/images/R21/ -type f -name "acpc.nii" -exec rm {} \;
find ~/compute/images/R21/ -type f -name "t1_ACPC.txt" -exec rm {} \;
find ~/compute/images/R21/ -type f -name "cropped.nii.gz" -exec rm {} \;
for i in $(find ~/compute/images/R21/ -type f -name "resampled.nii.gz"); do
    cd $(dirname $i)
    mv $(dirname $i)/t1.nii $(dirname $i)/orig.nii
    mv $i $(dirname $i)/t1.nii.gz
done
```

## Final Sync

```bash
rsync \
-rauv \
--exclude="dicom" \
intj5@ssh.fsl.byu.edu:~/compute/images/R21/ \
/Volumes/wasatch/data/images/R21/ \
--delete
```
