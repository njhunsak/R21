---
title: Motion
toc: true
weight: 8
---

## R Script

Run in R:

```r
projDir = "/Volumes/data/stats/R21/AFQ/"
files = list.files("/Volumes/data/images/R21/", pattern = "dwi_aligned_trilin_ecXform.csv",
    full.names = FALSE, recursive = TRUE)
motion = data.frame()
for (n in 1:length(files))
{
    mydata = read.table(paste("/Volumes/data/images/R21/", files[n], sep = ""), sep = ",",
        header = TRUE)
    temp = data.frame(t(colMeans(mydata)))
		tempvar = strsplit(as.character(files[n]), "/")
		studyid = sapply(tempvar, "[[", 1)
		temp = cbind(studyid, temp)
		motion = rbind(motion, temp)
}
motion$avgDisp = rowMeans(motion[c(2,3,4)])
motion$avgRot = rowMeans(motion[c(5,6,7)])
write.csv(motion, paste(projDir, "motion.csv", sep = ""), row.names = F)
```

## Graphs

Run in R:

```r
projDir = "/Volumes/data/stats/R21/AFQ/"
motion = read.table(paste(projDir, "motion.csv", sep = ""), sep = ",",
        header = TRUE)
pdf(paste(projDir, "displacement.pdf", sep = ""))
beeswarm(motion$avgDisp, data = motion, pch = 20, cex = 2, col = rainbow(8), main = 'Motion', ylab = 'Displacement in mm', xlab = '', method = c("square"))
text(motion$avgDisp[motion$studyid =="37"], labels = motion$studyid[motion$studyid =="37"], cex=0.9, font=2, pos = 4)
dev.off()
pdf(paste(projDir, "rotation.pdf", sep = ""))
beeswarm(motion$avgRot, data = motion, pch = 20, cex = 2, col = rainbow(8), main = 'Motion', ylab = 'Rotation in Degrees', xlab = '', method = c("square"))
text(motion$avgRot[motion$studyid =="37"], labels = motion$studyid[motion$studyid =="37"], cex=0.9, font=2, pos = 4)
dev.off()
q()
```