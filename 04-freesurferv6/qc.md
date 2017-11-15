---
title: Quality Control
toc: true
weight: 3
---

[https://brainder.org/2011/09/10/quickly-inspect-freesurfer-cortical-surfaces/](https://brainder.org/2011/09/10/quickly-inspect-freesurfer-cortical-surfaces/)

Quickly inspect FreeSurfer aseg, white matter, and cortical surfaces:

```bash
export SUBJECTS_DIR=/Volumes/wasatch/data/analyses/R21/FreeSurferv6

~/Applications/QA/snap_tksurfer.sh \
-l /Volumes/wasatch/data/analyses/R21/FreeSurferv6/subjects.txt \
-m pial -m inflated -m white -p aparc

~/Applications/QA/snap_tkmedit.sh \
-l /Volumes/wasatch/data/analyses/R21/FreeSurferv6/subjects.txt

~/Applications/QA/generate_html.sh \
-l /Volumes/wasatch/data/analyses/R21/FreeSurferv6/subjects.txt \
-m pial -m inflated -m white -p aparc \
-d /Users/njhunsak/Google/Projects/R21/FreeSurfer6/
```

Quickly inspect FreeSurfer hippocampal subregional ROIs. First, convert mgz to NIfTI:

``` bash
for i in $(ls /Volumes/wasatch/data/analyses/R21/FreeSurferv6/); do
  subjDir=/Volumes/wasatch/data/analyses/R21/FreeSurferv6/$i/mri/
  mri_convert --out_orientation RAS \
  ${subjDir}/brainmask.mgz ${subjDir}/brainmask.nii.gz
  mri_convert --out_orientation RAS -rt nearest \
  ${subjDir}/lh.hippoSfLabels-T1.v10.FSvoxelSpace.mgz \
  ${subjDir}/lh.hippoSfLabels-T1.v10.FSvoxelSpace.nii.gz
  mri_convert --out_orientation RAS -rt nearest \
  ${subjDir}/rh.hippoSfLabels-T1.v10.FSvoxelSpace.mgz \
  ${subjDir}/rh.hippoSfLabels-T1.v10.FSvoxelSpace.nii.gz
done
```

In R, run the following code to generate montage image:

```r
setwd("/Volumes/wasatch/data/stats/R21/FreeSurferv6/hippocampus/")
projDir = "/Volumes/wasatch/data/analyses/R21/antsCT_R21/"
dirs = list.dirs(projDir, full.names = FALSE, recursive = FALSE)
for (n in 1:length(dirs))
{
    fname = paste(projDir, dirs[n], "/CorticalThickness.nii.gz", sep = "")
    roi1name = paste(projDir, dirs[n], "/mri/lh.hippoSfLabels-T1.v10.FSvoxelSpace.nii.gz", 
        sep = "")
    roi2name = paste(projDir, dirs[n], "/mri/rh.hippoSfLabels-T1.v10.FSvoxelSpace.nii.gz", 
        sep = "")
    img = readNIfTI(fname, reorient = FALSE)
    roi1 = readNIfTI(roi1name, reorient = FALSE)
    roi2 = readNIfTI(roi2name, reorient = FALSE)
    roi1[roi1 == 0] = NA
    roi2[roi2 == 0] = NA
    dd = dropEmptyImageDimensions(img, value = 0, threshold = 0, other.imgs = list(roi1, 
        roi2), keep_ind = TRUE, reorient = FALSE)
    img_cropped = dd$outimg
    roi1_cropped = dd$other.imgs[[1]]
    roi2_cropped = dd$other.imgs[[2]]
    slices1 = getEmptyImageDimensions(roi1_cropped, value = 0)
    slices2 = getEmptyImageDimensions(roi2_cropped, value = 0)
    for (x in 1:length(slices1[[2]]))
    {
        png(paste("/Volumes/wasatch/data/stats/R21/FreeSurferv6/hippocampus/", 
            dirs[n], "-left-", sprintf("%02d", x), ".png", sep = ""), width = 1800, 
            height = 1800, units = "px")
        overlay(img_cropped, y = roi1_cropped, col.y = rainbow(40, alpha = 0.5), 
            z = slices1[[2]][x], plot.type = "single", plane = "coronal")
        dev.off()
    }
    OUTPUT = paste("/Volumes/wasatch/data/stats/R21/FreeSurferv6/hippocampus/", 
        dirs[n], "-left.gif", sep = "")
    system(paste("convert -delay 50 *.png", OUTPUT, sep = " "))
    file.remove(list.files(pattern = ".png"))
    
    # ORIGINAL CODE FOR MONTAGE
    # png(paste('/Volumes/wasatch/data/stats/R21/FreeSurferv6/hippocampus/',
    # dirs[n], '-left.png', sep = ''), width =
    # ceiling(length(slices1[[2]])/6) * 300, height = 1800, units = 'px')
    # overlay(img_cropped, y = roi1_cropped, col.y = rainbow(100, 0.3), z =
    # slices1[[2]], plot.type = 'single', plane = 'coronal', mfrow = c(6,
    # ceiling(length(slices1[[2]])/6))) dev.off()
    
    for (x in 1:length(slices2[[2]]))
    {
        png(paste("/Volumes/wasatch/data/stats/R21/FreeSurferv6/hippocampus/", 
            dirs[n], "-right-", sprintf("%02d", x), ".png", sep = ""), 
            width = 1800, height = 1800, units = "px")
        overlay(img_cropped, y = roi2_cropped, col.y = rainbow(40, alpha = 0.5), 
            z = slices2[[2]][x], plot.type = "single", plane = "coronal")
        dev.off()
    }
    OUTPUT = paste("/Volumes/wasatch/data/stats/R21/FreeSurferv6/hippocampus/", 
        dirs[n], "-right.gif", sep = "")
    system(paste("convert -delay 50 *.png", OUTPUT, sep = " "))
    file.remove(list.files(pattern = ".png"))
    
    # ORIGINAL CODE FOR MONTAGE
    # png(paste('/Volumes/wasatch/data/stats/R21/FreeSurferv6/hippocampus/',
    # dirs[n], '-right.png', sep = ''), width =
    # ceiling(length(slices2[[2]])/6) * 300, height = 1800, units = 'px')
    # overlay(img_cropped, y = roi2_cropped, col.y = alpha('green', 0.3), z
    # = slices2[[2]], plot.type = 'single', plane = 'coronal', mfrow = c(6,
    # ceiling(length(slices2[[2]])/6))) dev.off()
}
```