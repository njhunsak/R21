---
title: Generate AFQ Structure
toc: true
weight: 5
---

## Job Script

Create job script

``` bash
vi ~/scripts/R21/afqGroup_job.sh
```

Copy and paste:

``` bash
#!/bin/bash

#SBATCH --time=24:00:00   # walltime
#SBATCH --ntasks=1  # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=24576M   # memory per CPU core

# LOAD MODULES, INSERT CODE, AND RUN YOUR PROGRAMS HERE
cd ~/scripts/R21/
module load matlab/r2013b
matlab -nodisplay -nojvm -nosplash -r afqGroup
```

## MATLAB Script

Create MATLAB script:

```bash
vi ~/scripts/R21/afqGroup.m
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

sub_dirs = importdata('~/compute/analyses/R21/AFQ/sub_dirs.txt');
load ~/compute/analyses/R21/AFQ/sub_group.txt
outdir = fullfile([var, '/compute/analyses/R21/AFQ/']);
outname = fullfile(outdir, ['afq_'datestr(now, 'yyyy_mm_dd_HHMM')]);
afq = AFQ_Create('sub_dirs', sub_dirs, 'sub_group', sub_group, 'showfigs', false);
[afq, patient_data, control_data, norms, abn, abnTracts] = AFQ_run(sub_dirs, sub_group, afq);
save(outname, 'afq');
```

## Submit

Submit job:

```bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/${var}
sbatch \
-o ~/logfiles/${var}/output.txt \
-e ~/logfiles/${var}/error.txt \
~/scripts/R21/afqGroup_job.sh
```