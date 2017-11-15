---
title: Process Individual Participants
toc: true
weight: 4
---

After you’ve preprocessed your diffusion weighted data using dtiInit, you are ready to run the AFQ pipeline.

## Parameters

AFQ pipeline needs several MATLAB vectors in order to run. First, you will need to set the path to each participant’s dt6.mat file under the variable *sub_dirs*, then you will need to set the group information (0 for the control group and 1 for study group) under the variable *sub_group*. In order to keep a record of the process, let’s make this into a script. Create a script that will generate the variables, *sub_dirs* and *sub_group*. The csv output files from AFQ contain no participant ID information, so let's create two addition files that contain the oi subject IDs and tbi subject IDs:

```bash
workdir=~/compute/analyses/R21/AFQ
rm ${workdir}/sub_dirs.txt
rm ${workdir}/sub_group.txt
rm ${workdir}/oi.csv
rm ${workdir}/tbi.csv
for i in $(find ~/compute/images/R21/ -type d -name "*trilin"); do
    echo "$i" >> ${workdir}/sub_dirs.txt
    subjid=$(basename $(dirname $i))
    group=`grep $subjid ~/compute/analyses/R21/demographics/R21_demographics.txt | cut -d',' -f2`
    if [ $group = "tbi" ]; then
        echo -n "1, " >> ${workdir}/sub_group.txt
        echo "$subjid, 1" >> ${workdir}/tbi.csv
    else
        echo -n "0, " >> ${workdir}/sub_group.txt
        echo "$subjid, 0" >> ${workdir}/oi.csv
    fi
done
```

## Batch Script

Create batch script:

``` bash
vi ~/scripts/R21/afqIndividual_batch.sh
```

Copy and paste:

```bash
#!/bin/bash
total=`find ~/compute/images/R21/ -type d -name "*trilin" | wc -l`
for subj in `seq 1 $total`; do
    sbatch \
    -o ~/logfiles/${1}/output_${subj}.txt \
    -e ~/logfiles/${1}/error_${subj}.txt \
    ~/scripts/R21/afqIndividual_job.sh \
    ${subj}
    sleep 1
done
```

## Job Script

Create job script:

``` bash
vi ~/scripts/R21/afqIndividual_job.sh
```

Copy and paste:

```bash
#!/bin/bash

#SBATCH --time=02:00:00   # walltime
#SBATCH --ntasks=1  # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=24576M   # memory per CPU core

# LOAD MODULES, INSERT CODE, AND RUN YOUR PROGRAMS HERE
cd ~/scripts/R21/
module load matlab/r2013b
matlab -nodisplay -nojvm -nosplash -r "afqIndividual($1)"
```

## MATLAB Script

Create MATLAB script:

``` bash
vi ~/scripts/R21/afqIndividual.m
```

Copy and paste:

``` matlab
%%%%%%%
function afqIndividual(x)

% Display participant ID:
display(x);

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
afq = AFQ_Create('sub_dirs', sub_dirs, 'sub_group', sub_group, 'showfigs', false);
afq = AFQ_set(afq, 'runsubjects', x);
[afq, patient_data, control_data, norms, abn, abnTracts] = AFQ_run(sub_dirs, sub_group, afq);
exit;
```

## Submit

Submit batch script:

```bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/${var}
sh ~/scripts/R21/afqIndividual_batch.sh $var
```