---
title: Probabilistic Tractography of Fornix
toc: true
weight: 4
---

## ROIs

Place ROIs on MNI registered FA image, then warp the ROIs back to the participant domain:

``` bash
for i in $(ls /Volumes/wasatch/data/analyses/R21/mrtrix/); do
    cd /Volumes/wasatch/data/analyses/R21/mrtrix/$i
    
    # c3d fornix_seed_mni.nii.gz -dilate 1 0x1x0vox -o fornix_seed_mni.nii.gz
    WarpImageMultiTransform \
    3 \
    fornix_seed_mni.nii.gz \
    fornix_seed.nii.gz \
    -R fa.nii.gz \
    -i fa_registered_0GenericAffine.mat \
    --use-BSpline
    
    # c3d fornix_right_mni.nii.gz -dilate 1 0x1x0vox -o fornix_right_mni.nii.gz
    WarpImageMultiTransform \
    3 \
    fornix_right_mni.nii.gz \
    fornix_right.nii.gz \
    -R fa.nii.gz \
    -i fa_registered_0GenericAffine.mat \
    --use-BSpline
    
    # c3d fornix_left_mni.nii.gz -dilate 1 0x1x0vox -o fornix_left_mni.nii.gz
    WarpImageMultiTransform \
    3 \
    fornix_left_mni.nii.gz \
    fornix_left.nii.gz \
    -R fa.nii.gz \
    -i fa_registered_0GenericAffine.mat \
    --use-BSpline
done
```

{{% notice note %}}
Create the exclude ROI as well, but in native space.
{{% /notice %}}

## Run Tractography

Generate the track:

```
for i in $(ls /Volumes/wasatch/data/analyses/R21/mrtrix/); do
    cd /Volumes/wasatch/data/analyses/R21/mrtrix/$i
    pwd
    tckgen \
    -algorithm Tensor_Prob \
    -select 1000 \
    -seeds 0 \
    -stop \
    -seed_image fornix_seed.nii.gz \
    -seed_unidirectional \
    -include fornix_right.nii.gz \
    -exclude fornix_exclude.nii.gz \
    dwi_biascorrected.mif \
    fornix_right.tck -force
    
    tckgen \
    -algorithm Tensor_Prob \
    -select 1000 \
    -seeds 0 \
    -stop \
    -seed_image fornix_seed.nii.gz \
    -seed_unidirectional \
    -include fornix_left.nii.gz \
    -exclude fornix_exclude.nii.gz \
    dwi_biascorrected.mif \
    fornix_left.tck -force
done
```

## Generate Tables

Sample values of an associated image along tracks:

```
for i in $(ls /Volumes/wasatch/data/analyses/R21/mrtrix/); do
    cd /Volumes/wasatch/data/analyses/R21/mrtrix/$i
    pwd
    tckresample -num_points 100 fornix_right.tck fornix_right_resampled.tck -force
    tckresample -num_points 100 fornix_left.tck fornix_left_resampled.tck -force
    tcksample fornix_right_resampled.tck fa.nii.gz fornix_right_fa.txt
    tcksample fornix_left_resampled.tck fa.nii.gz fornix_left_fa.txt
    tcksample fornix_right_resampled.tck md.nii.gz fornix_right_md.txt
    tcksample fornix_left_resampled.tck md.nii.gz fornix_left_md.txt
    tcksample fornix_right_resampled.tck rd.nii.gz fornix_right_rd.txt
    tcksample fornix_left_resampled.tck rd.nii.gz fornix_left_rd.txt
    tcksample fornix_right_resampled.tck ad.nii.gz fornix_right_ad.txt
    tcksample fornix_left_resampled.tck ad.nii.gz fornix_left_ad.txt
done
```

## View Results

```bash
for i in $(ls /Volumes/wasatch/data/analyses/R21/mrtrix/); do
    cd /Volumes/wasatch/data/analyses/R21/mrtrix/$i
    pwd
    mrview \
    fa.nii.gz \
    -tractography.load fornix_right_resampled.tck \
    -tractography.load fornix_left_resampled.tck \
    -plane 0 \
    -size 1400,1400 \
    -position 100,0 \
    -noannotations
done
```

## Summarize Data

Run in R:

```r
projDir = "/Volumes/wasatch/data/stats/R21/fornix/"
files = list.files("/Volumes/wasatch/data/analyses/R21/mrtrix/", pattern = "fornix_.*.txt", 
    full.names = FALSE, recursive = TRUE)
mydata = data.frame()
for (n in 1:length(files)) {
    temp = read.table(paste("/Volumes/wasatch/data/analyses/R21/mrtrix/", files[n], 
        sep = ""), sep = "", header = FALSE)
    temp = data.frame(t(colMeans(temp)))
    tempvar = strsplit(as.character(files[n]), "/")
    studyid = sapply(tempvar, "[[", 1)
    tempvar = strsplit(as.character(tempvar[[1]][2]), "_")
    hemi = sapply(tempvar, "[[", 2)
    tempvar = strsplit(as.character(tempvar[[1]][3]), ".", fixed = TRUE)
    dtiscalar = sapply(tempvar, "[[", 1)
    temp = cbind(studyid, hemi, dtiscalar, temp)
    mydata = rbind(mydata, temp)
}
names(mydata) = c("studyid", "hemisphere", "dtiscalar", "node1", "node2", "node3", "node4", 
    "node5", "node6", "node7", "node8", "node9", "node10", "node11", "node12", 
    "node13", "node14", "node15", "node16", "node17", "node18", "node19", 
    "node20", "node21", "node22", "node23", "node24", "node25", "node26", 
    "node27", "node28", "node29", "node30", "node31", "node32", "node33", 
    "node34", "node35", "node36", "node37", "node38", "node39", "node40", 
    "node41", "node42", "node43", "node44", "node45", "node46", "node47", 
    "node48", "node49", "node50", "node51", "node52", "node53", "node54", 
    "node55", "node56", "node57", "node58", "node59", "node60", "node61", 
    "node62", "node63", "node64", "node65", "node66", "node67", "node68", 
    "node69", "node70", "node71", "node72", "node73", "node74", "node75", 
    "node76", "node77", "node78", "node79", "node80", "node81", "node82", 
    "node83", "node84", "node85", "node86", "node87", "node88", "node89", 
    "node90", "node91", "node92", "node93", "node94", "node95", "node96", 
    "node97", "node98", "node99", "node100")
write.csv(mydata, paste(projDir, "data.csv", sep = ""), row.names = F)
```

## Calculate P-Values

Run in R:

```r
projDir = "/Volumes/wasatch/data/stats/R21/fornix/"
motion = read.table("/Volumes/wasatch/data/stats/R21/motion.csv", sep = ",", header = TRUE)
mydata = read.table(paste(projDir, "data.csv", sep = ""), sep = ",", header = TRUE)
demogr = read.table("/Volumes/wasatch/data/analyses/R21/demographics/R21_demographics.txt", 
    sep = ",", header = TRUE)
mydata = merge(demogr, mydata, by = 1, all.y = T)
mydata = merge(mydata, motion[c(1, 8, 9)], by = 1, all.x = T)
pvalue = data.frame()
for (x in 7:106)
{
    left_fa = subset(mydata, hemisphere == "left" & dtiscalar == "fa")
    right_fa = subset(mydata, hemisphere == "right" & dtiscalar == "fa")
    left_md = subset(mydata, hemisphere == "left" & dtiscalar == "md")
    right_md = subset(mydata, hemisphere == "right" & dtiscalar == "md")
    left_rd = subset(mydata, hemisphere == "left" & dtiscalar == "rd")
    right_rd = subset(mydata, hemisphere == "right" & dtiscalar == "rd")
    left_ad = subset(mydata, hemisphere == "left" & dtiscalar == "ad")
    right_ad = subset(mydata, hemisphere == "right" & dtiscalar == "ad")
    left_fa = anova(lm(left_fa[[x]] ~ group + avgDisp + avgRot, data = left_fa))[5][1, ]
    right_fa = anova(lm(right_fa[[x]] ~ group + avgDisp + avgRot, data = right_fa))[5][1, ]
    left_md = anova(lm(left_md[[x]] ~ group + avgDisp + avgRot, data = left_md))[5][1, ]
    right_md = anova(lm(right_md[[x]] ~ group + avgDisp + avgRot, data = right_md))[5][1, ]
    left_rd = anova(lm(left_rd[[x]] ~ group + avgDisp + avgRot, data = left_rd))[5][1, ]
    right_rd = anova(lm(right_rd[[x]] ~ group + avgDisp + avgRot, data = right_rd))[5][1, ]
    left_ad = anova(lm(left_ad[[x]] ~ group + avgDisp + avgRot, data = left_ad))[5][1, ]
    right_ad = anova(lm(right_ad[[x]] ~ group + avgDisp + avgRot, data = right_ad))[5][1, ]
    left_fa = data.frame(left_fa)
    right_fa = data.frame(right_fa)
    left_md = data.frame(left_md)
    right_md = data.frame(right_md)
    left_rd = data.frame(left_rd)
    right_rd = data.frame(right_rd)
    left_ad = data.frame(left_ad)
    right_ad = data.frame(right_ad)
    temp = cbind(left_fa, right_fa, left_md, right_md, left_rd, right_rd, left_ad, right_ad)
    pvalue = rbind(pvalue, temp)
}
write.csv(pvalue, paste(projDir, "pvalue.csv", sep = ""), row.names = T)
```