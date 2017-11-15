---
title: Generate Output Files
toc: true
weight: 7
---

## Extract Values

This code shows you how to extract values on anisotropy and diffusivity measures for the white-matter pathways reconstructed by AFQ.

``` bash
mkdir -p ~/compute/analyses/R21/AFQ/orig
module load matlab
matlab
```
When MATLAB opens:

```matlab
var = getenv('HOME');
SPM8Path = [var, '/apps/matlab/spm8'];
addpath(genpath(SPM8Path));
vistaPath = [var, '/apps/matlab/vistasoft'];
addpath(genpath(vistaPath));
AFQPath = [var, '/apps/matlab/AFQ'];
addpath(genpath(AFQPath));
```

Use this code to save the data from the 20 fiber tracts:

```matlab
load ~/compute/analyses/R21/AFQ/afq_cc_2017_07_17_1603.mat
fgname = {'left_thalamic_radiation', 'right_thalamic_radiation', 'left_corticospinal', ...
    'right_corticospinal', 'left_cingulum_cingulate', 'right_cingulum_cingulate', ...
    'left_cingulum_hippocampus', 'right_cingulum_hippocampus', 'callosum_forceps_major', ...
    'callosum_forceps_minor', 'left_ifof', 'right_ifof', 'left_ilf', 'right_ilf', 'left_slf', ...
    'right_slf', 'left_uncinate', 'right_uncinate', 'left_arcuate', 'right_arcuate'};
outdir = fullfile('~/compute/analyses/R21/AFQ/orig/');
for n = 1:20
    csvwrite(fullfile(outdir, strcat('tbi_', fgname{n}, '_fa.csv')), afq.patient_data(n).FA)
    csvwrite(fullfile(outdir, strcat('tbi_', fgname{n}, '_rd.csv')), afq.patient_data(n).RD)
    csvwrite(fullfile(outdir, strcat('tbi_', fgname{n}, '_md.csv')), afq.patient_data(n).MD)
    csvwrite(fullfile(outdir, strcat('tbi_', fgname{n}, '_ad.csv')), afq.patient_data(n).AD)
    csvwrite(fullfile(outdir, strcat('oi_', fgname{n}, '_fa.csv')), afq.control_data(n).FA)
    csvwrite(fullfile(outdir, strcat('oi_', fgname{n}, '_rd.csv')), afq.control_data(n).RD)
    csvwrite(fullfile(outdir, strcat('oi_', fgname{n}, '_md.csv')), afq.control_data(n).MD)
    csvwrite(fullfile(outdir, strcat('oi_', fgname{n}, '_ad.csv')), afq.control_data(n).AD)
end
```

Use this code to save the data from the corpus callosum segmentation:

```matlab
load ~/compute/analyses/R21/AFQ/afq_cc_2017_07_17_1603.mat
fgname = {'cc_occipital', 'cc_post_parietal', 'cc_sup_parietal', 'cc_motor', 'cc_sup_frontal', ...
    'cc_ant_frontal', 'cc_orb_frontal', 'cc_temporal'};
outdir = fullfile('~/compute/analyses/R21/AFQ/orig/');
for n = 21:28
    csvwrite(fullfile(outdir, strcat('tbi_', fgname{n-20}, '_fa.csv')), afq.patient_data(n).FA)
    csvwrite(fullfile(outdir, strcat('tbi_', fgname{n-20}, '_rd.csv')), afq.patient_data(n).RD)
    csvwrite(fullfile(outdir, strcat('tbi_', fgname{n-20}, '_md.csv')), afq.patient_data(n).MD)
    csvwrite(fullfile(outdir, strcat('tbi_', fgname{n-20}, '_ad.csv')), afq.patient_data(n).AD)
    csvwrite(fullfile(outdir, strcat('oi_', fgname{n-20}, '_fa.csv')), afq.control_data(n).FA)
    csvwrite(fullfile(outdir, strcat('oi_', fgname{n-20}, '_rd.csv')), afq.control_data(n).RD)
    csvwrite(fullfile(outdir, strcat('oi_', fgname{n-20}, '_md.csv')), afq.control_data(n).MD)
    csvwrite(fullfile(outdir, strcat('oi_', fgname{n-20}, '_ad.csv')), afq.control_data(n).AD)
end
```

Here's where we add the missing subject IDs to the output files:

```bash
cd ~/compute/analyses/R21/AFQ/
for i in $(find ~/compute/analyses/R21/AFQ/orig -type f -name "oi*.csv"); do
	paste -d, <(cut -d, -f 1 oi.csv) $i \
	> ~/compute/stats/R21/AFQ/$(basename $i);
done
for i in $(find ~/compute/analyses/R21/AFQ/orig -type f -name "tbi*.csv"); do
	paste -d, <(cut -d, -f 1 tbi.csv) $i \
	> ~/compute/stats/R21/AFQ/$(basename $i);
done
```

## Merge Tables

For the original 20 fiber tracts open R:

```bash
module load r
R
```

Run the following code:

```r
projDir = "~/compute/stats/R21/"
demogr = read.table("~/compute/analyses/R21/demographics/R21_demographics.txt",
    sep = ",", header = TRUE)
rawdata = c("callosum_forceps_major", "callosum_forceps_minor", "left_arcuate",
    "left_cingulum_cingulate", "left_cingulum_hippocampus", "left_corticospinal",
    "left_ifof", "left_ilf", "left_slf", "left_thalamic_radiation", "left_uncinate",
    "right_arcuate", "right_cingulum_cingulate", "right_cingulum_hippocampus",
    "right_corticospinal", "right_ifof", "right_ilf", "right_slf", "right_thalamic_radiation",
    "right_uncinate")
for (n in 1:length(rawdata))
{
    files = list.files(paste(projDir, "AFQ/", sep = ""), pattern = rawdata[n],
        full.names = TRUE)
    matches = grep(pattern = "^\\/fslhome/intj5/compute/stats/R21/AFQ//.*i_.*.csv",
        files, value = T)
    mydata = data.frame()
    for (i in 1:length(matches))
    {
        temp = read.table(matches[i], sep = ",", header = FALSE)
        temp$variable = matches[i]
        mydata = rbind(mydata, temp)
    }
    tempvar = strsplit(as.character(mydata$variable), "/")
    mydata$variable = sapply(tempvar, "[[", length(tempvar[[1]]))
    tempvar = strsplit(as.character(mydata$variable), "_")
    mydata$dtiscalar = sapply(tempvar, "[[", length(tempvar[[1]]))
    mydata$dtiscalar = substr(mydata$dtiscalar, 1, 2)
    tempid = strsplit(as.character(mydata$V1), "/")
    mydata$studyid = sapply(tempid, "[[", 1)
    mydata$studyid = substr(mydata$studyid, 1, 4)
    mydata = mydata[c(104, 103, 2:101)]
    names(mydata) = c("studyid", "dtiscalar", "node1", "node2", "node3",
        "node4", "node5", "node6", "node7", "node8", "node9", "node10",
        "node11", "node12", "node13", "node14", "node15", "node16", "node17",
        "node18", "node19", "node20", "node21", "node22", "node23", "node24",
        "node25", "node26", "node27", "node28", "node29", "node30", "node31",
        "node32", "node33", "node34", "node35", "node36", "node37", "node38",
        "node39", "node40", "node41", "node42", "node43", "node44", "node45",
        "node46", "node47", "node48", "node49", "node50", "node51", "node52",
        "node53", "node54", "node55", "node56", "node57", "node58", "node59",
        "node60", "node61", "node62", "node63", "node64", "node65", "node66",
        "node67", "node68", "node69", "node70", "node71", "node72", "node73",
        "node74", "node75", "node76", "node77", "node78", "node79", "node80",
        "node81", "node82", "node83", "node84", "node85", "node86", "node87",
        "node88", "node89", "node90", "node91", "node92", "node93", "node94",
        "node95", "node96", "node97", "node98", "node99", "node100")
    mydata = merge(mydata, demogr, by = "studyid", all.x = TRUE)[c(1, 103,
        2:102)]
    mydata = na.omit(mydata)
    temp = data.frame(rowMeans(mydata[4:103], na.rm = TRUE))
    temp = cbind(mydata[1:3], temp)
    names(temp) = c("studyid", "group", "dtiscalar", rawdata[n])
    assign(paste("avg.", n, sep = ""), temp)
    write.table(mydata, paste(projDir, "AFQ/afq/", rawdata[n], ".csv",
        sep = ""), row.names = F, sep = ",")
}
```

To merge the corpus callosum fibers:

```r
projDir = "~/compute/stats/R21/"
demogr = read.table("~/compute/analyses/R21/demographics/R21_demographics.txt",
    sep = ",", header = TRUE)
rawdata = c("cc_occipital", "cc_post_parietal", "cc_sup_parietal", "cc_motor",
    "cc_sup_frontal", "cc_ant_frontal", "cc_orb_frontal", "cc_temporal")
for (n in 1:length(rawdata))
{
    files = list.files(paste(projDir, "AFQ/", sep = ""), pattern = rawdata[n],
        full.names = TRUE)
    matches = grep(pattern = "^\\/fslhome/intj5/compute/stats/R21/AFQ//.*i_.*.csv",
        files, value = T)
    mydata = data.frame()
    for (i in 1:length(matches))
    {
        temp = read.table(matches[i], sep = ",", header = FALSE)
        temp$variable = matches[i]
        mydata = rbind(mydata, temp)
    }
    tempvar = strsplit(as.character(mydata$variable), "/")
    mydata$variable = sapply(tempvar, "[[", length(tempvar[[1]]))
    tempvar = strsplit(as.character(mydata$variable), "_")
    mydata$dtiscalar = sapply(tempvar, "[[", length(tempvar[[1]]))
    mydata$dtiscalar = substr(mydata$dtiscalar, 1, 2)
    tempid = strsplit(as.character(mydata$V1), "/")
    mydata$studyid = sapply(tempid, "[[", 1)
    mydata$studyid = substr(mydata$studyid, 1, 4)
    mydata = mydata[c(104, 103, 2:101)]
    names(mydata) = c("studyid", "dtiscalar", "node1", "node2", "node3",
        "node4", "node5", "node6", "node7", "node8", "node9", "node10",
        "node11", "node12", "node13", "node14", "node15", "node16", "node17",
        "node18", "node19", "node20", "node21", "node22", "node23", "node24",
        "node25", "node26", "node27", "node28", "node29", "node30", "node31",
        "node32", "node33", "node34", "node35", "node36", "node37", "node38",
        "node39", "node40", "node41", "node42", "node43", "node44", "node45",
        "node46", "node47", "node48", "node49", "node50", "node51", "node52",
        "node53", "node54", "node55", "node56", "node57", "node58", "node59",
        "node60", "node61", "node62", "node63", "node64", "node65", "node66",
        "node67", "node68", "node69", "node70", "node71", "node72", "node73",
        "node74", "node75", "node76", "node77", "node78", "node79", "node80",
        "node81", "node82", "node83", "node84", "node85", "node86", "node87",
        "node88", "node89", "node90", "node91", "node92", "node93", "node94",
        "node95", "node96", "node97", "node98", "node99", "node100")
    mydata = merge(mydata, demogr, by = "studyid", all.x = TRUE)[c(1, 103,
        2:102)]
    mydata = na.omit(mydata)
    temp = data.frame(rowMeans(mydata[4:103], na.rm = TRUE))
    temp = cbind(mydata[1:3], temp)
    names(temp) = c("studyid", "group", "dtiscalar", rawdata[n])
    assign(paste("avg.", n, sep = ""), temp)
    write.table(mydata, paste(projDir, "AFQ/cc/", rawdata[n], ".csv", sep = ""),
        row.names = F, sep = ",")
}
q()
```

Clean up directory:

```bash
cd ~/compute/stats/R21/AFQ
rm tbi*
rm oi*
```

## Sync Data

```bash
rsync -rauv \
intj5@ssh.fsl.byu.edu:~/compute/stats/R21/AFQ/ \
/Volumes/wasatch/data/stats/R21/AFQ/

rsync -rauv \
intj5@ssh.fsl.byu.edu:~/compute/analyses/R21/AFQ/ \
/Volumes/wasatch/data/analyses/R21/AFQ/
```