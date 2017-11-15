---
title: Template
toc: true
weight: 3
---

## Job Script

Create a job script:

``` bash
vi /fslhome/intj5/scripts/R21/template.sh
```

Copy and paste code:

``` bash
#!/bin/bash

#SBATCH --time=05:00:00   # walltime
#SBATCH --ntasks=1   # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=32768M  # memory per CPU core

# LOAD ENVIRONMENTAL VARIABLES
var=`id -un`
export ANTSPATH=/fslhome/${var}/apps/ants/bin/
PATH=${ANTSPATH}:${PATH}

# INSERT CODE, AND RUN YOUR PROGRAMS HERE
cd ~/templates/R21
~/apps/ants/bin/buildtemplateparallel.sh \
-d 3 \
-z ~/templates/R21/pt1template.nii.gz \
-o pt2 \
-c 5 \
img*.nii.gz
```

## Submit

Submit job script:

``` bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/${var}
sbatch \
-o ~/logfiles/${var}/output-template.txt \
-e ~/logfiles/${var}/error-template.txt \
/fslhome/intj5/scripts/R21/template.sh
```

## Clean Up Directory

``` bash
cd ~/templates/R21
mkdir data
mv img*.nii.gz data/
mv pt2template.nii.gz template.nii.gz
find . \( ! -name "data" ! -name "img*.nii.gz" ! -name "template.nii.gz" \) \
-exec rm -f {} \;
```

