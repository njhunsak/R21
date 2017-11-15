---
title: Generate Datasets
toc: true
weight: 3
---

The data need to be combined across participants.

## Cortical Thickness

```r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
files = list.files(path = "/Volumes/wasatch/data/analyses/R21/mindboggled/", pattern = "thickinthehead_per_freesurfer_cortex_label.csv",
    full.names = T, recursive = T, include.dirs = T)
mydata = data.frame()
for (i in 1:(length(files) - 1))
{
    temp = read.table(files[i], sep = ",", header = F, skip = 1)[c(1, 3)]
    temp = setNames(data.frame(t(temp[, -1])), temp[, 1])
    tempvar = strsplit(files[i], "/")
    studyid = sapply(tempvar, "[[", (length(tempvar[[1]]) - 2))
    temp = cbind(studyid, temp)
    mydata = rbind(mydata, temp)
}
names(mydata) = gsub("-", "_", names(mydata))
write.table(mydata, "thickinthehead_per_freesurfer_cortex_label.csv", row.names = F,
    sep = ",")
```

## Volume

```r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
files = list.files(path = "/Volumes/wasatch/data/analyses/R21/mindboggled/", pattern = "volume_per_freesurfer_label.csv",
    full.names = T, recursive = T, include.dirs = T)
mydata = data.frame()
for (i in 1:(length(files) - 1))
{
    temp = read.table(files[i], sep = ",", header = F, skip = 1)[c(1, 3)]
    temp = setNames(data.frame(t(temp[, -1])), temp[, 1])
    tempvar = strsplit(files[i], "/")
    studyid = sapply(tempvar, "[[", (length(tempvar[[1]]) - 2))
    temp = cbind(studyid, temp)
    if (i == 1)
    {
        mydata = temp
    } else
    {
        mydata = rbind.fill(mydata, temp)
    }
}
names(mydata) = gsub("-", "_", names(mydata))
write.table(mydata, "volume_per_freesurfer_label.csv", row.names = F, sep = ",")
```

## Area

```r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
files = list.files(path = "/Volumes/wasatch/data/analyses/R21/mindboggled/", pattern = "label_shapes.csv",
    full.names = T, recursive = T, include.dirs = T)
matches = grep(pattern = "^\\/Volumes/wasatch/data/analyses/R21/mindboggled//.*/left_cortical_surface/",
    files, value = T)
lh = data.frame()
for (i in 1:(length(matches) - 1))
{
    temp = read.table(matches[i], sep = ",", header = F, skip = 1)[c(1,
        2, 3)]
    temp = subset(temp, temp[2] < 2000)[c(1, 3)]
    temp = setNames(data.frame(t(temp[, -1])), temp[, 1])
    tempvar = strsplit(matches[i], "/")
    studyid = sapply(tempvar, "[[", (length(tempvar[[1]]) - 3))
    temp = cbind(studyid, temp)
    if (i == 1)
    {
        lh = temp
    } else
    {
        lh = rbind.fill(lh, temp)
    }
}
matches = grep(pattern = "^\\/Volumes/wasatch/data/analyses/R21/mindboggled//.*/right_cortical_surface/",
    files, value = T)
rh = data.frame()
for (i in 1:(length(matches) - 1))
{
    temp = read.table(matches[i], sep = ",", header = F, skip = 1)[c(1,
        2, 3)]
    temp = subset(temp, temp[2] > 1999)[c(1, 3)]
    temp = setNames(data.frame(t(temp[, -1])), temp[, 1])
    tempvar = strsplit(matches[i], "/")
    studyid = sapply(tempvar, "[[", (length(tempvar[[1]]) - 3))
    temp = cbind(studyid, temp)
    if (i == 1)
    {
        rh = temp
    } else
    {
        rh = rbind.fill(rh, temp)
    }
}
mydata = merge(lh, rh, by = 1)
for (i in 2:(length(mydata)))
{
    mydata[i] = round(mydata[i], 6)
    mydata[i][mydata[i] <= 0] <- NA
}
names(mydata) = gsub("-", "_", names(mydata))
write.table(mydata, "area_per_gyrus_label.csv", row.names = F, sep = ",")
```

## Travel Depth

```r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
files = list.files(path = "/Volumes/wasatch/data/analyses/R21/mindboggled/", pattern = "label_shapes.csv",
    full.names = T, recursive = T, include.dirs = T)
matches = grep(pattern = "^\\/Volumes/wasatch/data/analyses/R21/mindboggled//.*/left_cortical_surface/",
    files, value = T)
lh = data.frame()
for (i in 1:(length(matches) - 1))
{
    temp = read.table(matches[i], sep = ",", header = F, skip = 1)[c(1,
        2, 6)]
    temp = subset(temp, temp[2] < 2000)[c(1, 3)]
    temp = setNames(data.frame(t(temp[, -1])), temp[, 1])
    tempvar = strsplit(matches[i], "/")
    studyid = sapply(tempvar, "[[", (length(tempvar[[1]]) - 3))
    temp = cbind(studyid, temp)
    if (i == 1)
    {
        lh = temp
    } else
    {
        lh = rbind.fill(lh, temp)
    }
}
matches = grep(pattern = "^\\/Volumes/wasatch/data/analyses/R21/mindboggled//.*/right_cortical_surface/",
    files, value = T)
rh = data.frame()
for (i in 1:(length(matches) - 1))
{
    temp = read.table(matches[i], sep = ",", header = F, skip = 1)[c(1,
        2, 6)]
    temp = subset(temp, temp[2] > 1999)[c(1, 3)]
    temp = setNames(data.frame(t(temp[, -1])), temp[, 1])
    tempvar = strsplit(matches[i], "/")
    studyid = sapply(tempvar, "[[", (length(tempvar[[1]]) - 3))
    temp = cbind(studyid, temp)
    if (i == 1)
    {
        rh = temp
    } else
    {
        rh = rbind.fill(rh, temp)
    }
}
mydata = merge(lh, rh, by = 1)
for (i in 2:(length(mydata)))
{
    mydata[i] = round(mydata[i], 6)
    mydata[i][mydata[i] <= 0] <- NA
}
names(mydata) = gsub("-", "_", names(mydata))
write.table(mydata, "travel_depth_per_gyrus_label.csv", row.names = F,
    sep = ",")
```

## Geodesic Depth

```r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
files = list.files(path = "/Volumes/wasatch/data/analyses/R21/mindboggled/", pattern = "label_shapes.csv",
    full.names = T, recursive = T, include.dirs = T)
matches = grep(pattern = "^\\/Volumes/wasatch/data/analyses/R21/mindboggled//.*/left_cortical_surface/",
    files, value = T)
lh = data.frame()
for (i in 1:(length(matches) - 1))
{
    temp = read.table(matches[i], sep = ",", header = F, skip = 1)[c(1,
        2, 14)]
    temp = subset(temp, temp[2] < 2000)[c(1, 3)]
    temp = setNames(data.frame(t(temp[, -1])), temp[, 1])
    tempvar = strsplit(matches[i], "/")
    studyid = sapply(tempvar, "[[", (length(tempvar[[1]]) - 3))
    temp = cbind(studyid, temp)
    if (i == 1)
    {
        lh = temp
    } else
    {
        lh = rbind.fill(lh, temp)
    }
}
matches = grep(pattern = "^\\/Volumes/wasatch/data/analyses/R21/mindboggled//.*/right_cortical_surface/",
    files, value = T)
rh = data.frame()
for (i in 1:(length(matches) - 1))
{
    temp = read.table(matches[i], sep = ",", header = F, skip = 1)[c(1,
        2, 14)]
    temp = subset(temp, temp[2] > 1999)[c(1, 3)]
    temp = setNames(data.frame(t(temp[, -1])), temp[, 1])
    tempvar = strsplit(matches[i], "/")
    studyid = sapply(tempvar, "[[", (length(tempvar[[1]]) - 3))
    temp = cbind(studyid, temp)
    if (i == 1)
    {
        rh = temp
    } else
    {
        rh = rbind.fill(rh, temp)
    }
}
mydata = merge(lh, rh, by = 1)
for (i in 2:(length(mydata)))
{
    mydata[i] = round(mydata[i], 6)
    mydata[i][mydata[i] <= 0] <- NA
}
names(mydata) = gsub("-", "_", names(mydata))
write.table(mydata, "geodesic_depth_per_gyrus_label.csv", row.names = F,
    sep = ",")
```

## Curvature

```r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
files = list.files(path = "/Volumes/wasatch/data/analyses/R21/mindboggled/", pattern = "label_shapes.csv",
    full.names = T, recursive = T, include.dirs = T)
matches = grep(pattern = "^\\/Volumes/wasatch/data/analyses/R21/mindboggled//.*/left_cortical_surface/",
    files, value = T)
lh = data.frame()
for (i in 1:(length(matches) - 1))
{
    temp = read.table(matches[i], sep = ",", header = F, skip = 1)[c(1,
        2, 22)]
    temp = subset(temp, temp[2] < 2000)[c(1, 3)]
    temp = setNames(data.frame(t(temp[, -1])), temp[, 1])
    tempvar = strsplit(matches[i], "/")
    studyid = sapply(tempvar, "[[", (length(tempvar[[1]]) - 3))
    temp = cbind(studyid, temp)
    if (i == 1)
    {
        lh = temp
    } else
    {
        lh = rbind.fill(lh, temp)
    }
}
matches = grep(pattern = "^\\/Volumes/wasatch/data/analyses/R21/mindboggled//.*/right_cortical_surface/",
    files, value = T)
rh = data.frame()
for (i in 1:(length(matches) - 1))
{
    temp = read.table(matches[i], sep = ",", header = F, skip = 1)[c(1,
        2, 22)]
    temp = subset(temp, temp[2] > 1999)[c(1, 3)]
    temp = setNames(data.frame(t(temp[, -1])), temp[, 1])
    tempvar = strsplit(matches[i], "/")
    studyid = sapply(tempvar, "[[", (length(tempvar[[1]]) - 3))
    temp = cbind(studyid, temp)
    if (i == 1)
    {
        rh = temp
    } else
    {
        rh = rbind.fill(rh, temp)
    }
}
mydata = merge(lh, rh, by = 1)
for (i in 2:(length(mydata)))
{
    mydata[i] = round(mydata[i], 6)
}
names(mydata) = gsub("-", "_", names(mydata))
write.table(mydata, "curvature_per_gyrus_label.csv", row.names = F, sep = ",")
```