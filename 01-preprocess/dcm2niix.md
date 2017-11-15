---
title: DICOM to NIfTI
toc: true
weight: 2
---

## Sort DICOMs

In R:

```r
sortDicomDirectories(dicomDir, deleteOriginals = FALSE, sortOn = "series",
    seriesId = c("UID", "number", "time"), nested = TRUE)
```

## DICOM to NiFTI

Convert T1 image:

```bash
for i in $(find /Volumes/wasatch/data/images/R21/ -type d -name "*MPRAGE"); do
    cd $(dirname $(dirname $i))
    mkdir str
    ~/Applications/dcm2niix/bin/dcm2niix -o str/ -x y -f t1 -z n $i
done
```

Convert DW image:

```bash
for i in $(find /Volumes/wasatch/data/images/R21/ -type d -name "*DTI_64_Directions"); do
    cd $(dirname $(dirname $i))
    mkdir dti
    /Applications/MRIcron/dcm2nii64 -a y -g y -n y -x y -o dti/ $i/*
    cd dti/
    mv *.bval dwi.bval
    mv *.bvec dwi.bvec
    mv 2*.nii.gz dwi.nii.gz
done
```

## Sync Data

```bash
rsync -rauv \
--exclude=".*" \
--exclude="dicom" \
/Volumes/wasatch/data/images/R21/ \
intj5@ssh.fsl.byu.edu:~/compute/images/R21/
```