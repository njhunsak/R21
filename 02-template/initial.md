---
title: Initial Template
toc: true
weight: 2
---

Copy preprocessed images for template construction:

``` bash
mkdir -p ~/templates/R21
for i in $(ls ~/compute/images/R21/); do
    cp -v ~/compute/images/R21/$i/str/t1.nii.gz ~/templates/R21/img_${i}.nii.gz;
done
```

## Job Script

Create a job script:

``` bash
vi /fslhome/intj5/scripts/R21/template-initial.sh
```

Copy and paste code:

``` bash
#!/bin/bash

#SBATCH --time=02:00:00   # walltime
#SBATCH --ntasks=1   # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=16384M  # memory per CPU core

# LOAD ENVIRONMENTAL VARIABLES
var=`id -un`
export ANTSPATH=/fslhome/${var}/apps/ants/bin/
PATH=${ANTSPATH}:${PATH}

# INSERT CODE, AND RUN YOUR PROGRAMS HERE
cd ~/templates/R21/
~/apps/ants/bin/buildtemplateparallel.sh \
-d 3 \
-m 1x0x0 \
-o pt1 \
-c 5 \
img*.nii.gz
```

## Submit 

Submit job script:

``` bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/${var}
sbatch \
-o ~/logfiles/${var}/output-initial.txt \
-e ~/logfiles/${var}/error-initial.txt \
/fslhome/intj5/scripts/R21/template-initial.sh
```