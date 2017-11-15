---
title: Corpus Callosum Segmentation
toc: true
weight: 6
---

## Job Script

Create job script:

```bash
vi ~/scripts/R21/afqCC_job.sh
```

Copy and paste:

```bash
#!/bin/bash

#SBATCH --time=60:00:00   # walltime
#SBATCH --ntasks=1  # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=24576M   # memory per CPU core

# LOAD MODULES, INSERT CODE, AND RUN YOUR PROGRAMS HERE
cd ~/scripts/R21/
module load matlab/r2013b
matlab -nodisplay -nojvm -nosplash -r afqCC
```

## MATLAB Script

Create MATLAB script:

```bash
vi ~/scripts/R21/afqCC.m
```

Copy and paste:

```matlab
% Get home directory:
var = getenv('HOME');

% Add modules to MATLAB. Do not change the order of these programs:
SPM8Path = [var, '/apps/matlab/spm8'];
addpath(genpath(SPM8Path));
vistaPath = [var, '/apps/matlab/vistasoft'];
addpath(genpath(vistaPath));
AFQPath = [var, '/apps/matlab/AFQ'];
addpath(genpath(AFQPath));

load ~/compute/analyses/R21/AFQ/afq_2017_07_17_0835.mat
outdir = fullfile([var, '/compute/analyses/R21/AFQ/']);
outname = fullfile(outdir, ['afq_cc_', datestr(now, 'yyyy_mm_dd_HHMM')]);
afq = AFQ_SegmentCallosum(afq, [false, false])
save(outname, 'afq');
```

Submit job script:

```bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/${var}
sbatch \
-o ~/logfiles/${var}/ouput.txt \
-e ~/logfiles/${var}/error.txt \
~/scripts/R21/afqCC_job.sh
```