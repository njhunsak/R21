---
title: DWI Preprocessing Steps
toc: true
weight: 3
---

## Fix MATLAB Code

{{% notice warning %}}
If you are using MATLAB r2010 or greater or running this pipeline on a computing cluster that does not allow for visualization, you will need to update the `dtiCheckMotion.m` file located at `vistasoft/mrDiffusion/preprocess/dtiCheckMotion.m`.
{{% /notice %}}

```matlab
function [fh, figurename] = dtiCheckMotion(ecXformFile, visibility)
% Plots rotations and translations from an eddy-current correction
% transform file.
%
%   [fh, figurename]  = dtiCheckMotion([ecXformFile=uigetfile],visibility)
%
% INPUTS:
%   ecXformFile - Eddy Current correction trasformation infromation. This
%                 file is generally generated and saved by dtiInit.m. See
%                 dtiInit.m and also dtiRawRohdeEstimateEddyMotion.m
%
%   visibility  - A figure with the estimates of rotation and translation
%                 will be either displayed (visibility='on') or not
%                 (visibility='off'). The figure will be always saved in
%                 the same directory of the ecXformFile.mat file.
%
% OUTPUTS:
%   fh - Handle to the figure of the motion estimae. Note, this figure
%        might be set to invisible using 'visibility'. To display the
%        figure invoke the following command: set(fh,'visibility','on').
%
%   figurename - Full path to the figure saved out to disk showing the
%                motion estimates.
%i
% Franco Pestilli & Bob Dougherty Stanford University

if notDefined('visibility'), visibility = 'off'; end
if (~exist('ecXformFile', 'var') || isempty(ecXformFile))
    [f, p] = uigetfile('*.mat', 'Select the ecXform file');
    if (isequal(f, 0)), disp('Canceled.'); retun; end
    ecXformFile = fullfile(p, f);
end

% Load the stored trasformation file.
ec = load(ecXformFile);
t = vertcat(ec.xform(:).ecParams);

% Generate dataset that contains the motion correct for each volume - NJH
% 2016-04-05
mc = t(:, 1:6);
mc(:, 4:6) = (mc(:, 4:6) / (2 * pi) * 360);
mc = dataset({mc, 'x', 'y', 'z', 'pitch', 'roll', 'yaw'});

% We make a plot of the motion correction during eddy current correction
% but we do not show the figure. We only save it to disk.
% fh = mrvNewGraphWin([], [], visibility);
% if isstruct(fh)
%     fh = fh.Number;
% end
% subplot(2, 1, 1);
% plot(t(:, 1:3));
% title('Translation');
% xlabel('Diffusion image number (diffusion direction)');
% ylabel('translation (voxels)');
% legend({'x', 'y', 'z'});

% subplot(2, 1, 2);
% plot(t(:, 4:6)/(2 * pi)*360);
% title('Rotation');
% xlabel('Diffusion image number (diffusion direction)');
% ylabel('rotation (degrees)');
% legend({'pitch', 'roll', 'yaw'});

% Save out a PNG figure with the same filename as the Eddy Currents correction xform.
[p, f, ~] = fileparts(ecXformFile);
% figurename = fullfile(p, [f, '.png']);

% Added export csv file of motion correction - NJH 2016-04-05
filename = fullfile(p, [f, '.csv']);
export(mc, 'File', filename, 'Delimiter', ',');

% Old Code
% printCommand = sprintf('print(%s, ''-painters'',''-dpng'', ''-noui'', ''%s'')', num2str(fh),figurename);
% New Code - NJH 2016-04-05
% printCommand = sprintf('saveas(fh,figurename)');

% fprintf('[%s] Saving Eddy Current correction figure: \n %s \n', ...
    mfilename, figurename);
% eval(printCommand);

% Delete output if it was not requested
% if (nargout < 1), close(fh); clear fh figurename; end

return;
```

## Job Script

Copy brain extracted image to directory:

``` bash
for subj in $(ls ~/compute/images/R21/); do
    FILE=~/compute/images/R21/${subj}/str/brain.nii.gz
    if [ ! -e $FILE ]; then
        cp -v \
        ~/compute/analyses/R21/antsCT_R21/${subj}/ExtractedBrain0N4.nii.gz \
        ~/compute/images/R21/${subj}/str/brain.nii.gz
    fi
done
```

Create script:

``` bash
vi ~/scripts/R21/dtiInit_job.sh
```

Copy and paste:

``` bash
#!/bin/bash

#SBATCH --time=04:00:00   # walltime
#SBATCH --ntasks=1  # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=24576M   # memory per CPU core

# LOAD MODULES, INSERT CODE, AND RUN YOUR PROGRAMS HERE
cd ~/scripts/R21/
module load matlab/r2013b
matlab -nodisplay -nojvm -nosplash -r "subjID('$1')"
```

## Batch Script

Create script:

``` bash
vi ~/scripts/R21/dtiInit_batch.sh
```

Copy and paste:

``` bash
#!/bin/bash
for subj in $(ls ~/compute/images/R21/); do
    sbatch \
    -o ~/logfiles/${1}/output_${subj}.txt \
    -e ~/logfiles/${1}/error_${subj}.txt \
    ~/scripts/R21/dtiInit_job.sh \
    ${subj}
    sleep 1
done
```

## MATLAB Function

The reason we are creating a function versus a script, is so we can pass a variable, namely the participant ID, into the script:

``` bash
vi ~/scripts/R21/subjID.m
```

Copy and paste:

``` matlab
%%%%%%%
function subjID(x)

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

% Set file names:
subjDir = [var, '/compute/images/R21/', x];
brainFile = [subjDir, '/str/brain.nii.gz'];
t1File = [subjDir, '/str/matlab.nii.gz'];
dtiFile = [subjDir, '/dti/dwi.nii.gz'];
cd(subjDir);

mrAnatAverageAcpcNifti(brainFile, t1File, ['/fslhome/intj5/templates/matlab.nii.gz']);

% Don't change the following code:
ni = readFileNifti(t1File);
ni = niftiSetQto(ni, ni.sto_xyz);
writeFileNifti(ni, t1File);

ni = readFileNifti(dtiFile);
ni = niftiSetQto(ni, ni.sto_xyz);
writeFileNifti(ni, dtiFile);

% Determine phase encode dir:
% > info=dicominfo([var,'/compute/images/R21/FRE_AD001/DICOM/diff/MR.22533.01274.dcm']);
% To get the manufacturer information:
% > info.(dicomlookup('0008','0070'))
% To get the axis of phase encoding with respect to the image:
% > info.(dicomlookup('0018','1312'))
% If phase encode dir is 'COL', then set 'phaseEncodeDir' to '2'
% If phase encode dir is 'ROW', then set 'phaseEncodeDir' to '1'
% For Siemens / Philips specific code we need to add 'rotateBvecsWithCanXform',
% AND ALWAYS DOUBLE CHECK phaseEncodeDir:
% > dwParams = dtiInitParams('rotateBvecsWithCanXform',1,'phaseEncodeDir',2,'clobber',1);
% For GE specific code,
% AND ALWAYS DOUBLE CHECK phaseEncodeDir:
% > dwParams = dtiInitParams('phaseEncodeDir',2,'clobber',1);
dwParams = dtiInitParams('rotateBvecsWithCanXform', 1, 'phaseEncodeDir', 2, 'clobber', 1);

% Here's the one line of code to do the DTI preprocessing:
dtiInit(dtiFile, t1File, dwParams);

% Clean up files and exit:
movefile('dwi_*', 'dti/');
movefile('dtiInitLog.mat', 'dti/');
movefile('ROIs', '*trilin');

exit;
```

Finally, submit the whole process by submitting the batch script:

```bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/${var}
sh ~/scripts/R21/dtiInit_batch.sh $var
```

## Sync Data

```bash
rsync -rauv intj5@ssh.fsl.byu.edu:~/compute/images/R21/ /Volumes/wasatch/data/images/R21/
```