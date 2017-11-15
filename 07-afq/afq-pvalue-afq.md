---
title: Calculate p-value AFQ
toc: true
weight: 9
---

## R Script

For AFQ, create an R script:

```bash
vi /Volumes/wasatch/data/scripts/R21/AFQ/pvalues-afq.R
```

Copy and paste:

```r
metric <- readline(prompt = "What DTI metric do you want to analyze (fa, rd, md, ad): ")
projDir = "/Volumes/wasatch/data/stats/R21/AFQ/"
motion = read.table(paste(projDir, "motion.csv", sep = ""), sep = ",",
    header = TRUE)
files = list.files(paste(projDir, "afq/", sep = ""), pattern = ".csv",
    full.names = FALSE)
for (n in 1:length(files))
{
    mydata = read.table(paste(projDir, "afq/", files[n], sep = ""), sep = ",",
        header = TRUE)
    mydata = merge(mydata, motion[c(1,8,9)], by = 1, all.x = T)
    mydata = subset(mydata, dtiscalar == metric)
    pvalue = data.frame()
    for (x in 4:103)
    {
        group = anova(lm(mydata[[x]] ~ group + avgDisp +
            avgRot, data = mydata))[5][1, ]
        translation = anova(lm(mydata[[x]] ~ group + avgDisp +
            avgRot, data = mydata))[5][2, ]
        rotation = anova(lm(mydata[[x]] ~ group + avgDisp +
            avgRot, data = mydata))[5][3, ]
        group = data.frame(group)
        translation = data.frame(translation)
        rotation = data.frame(rotation)
        tmp = cbind(group, translation, rotation)
        pvalue = rbind(pvalue, tmp)
    }
    pvalue$group.pFDR = p.adjust(pvalue$group, method = c("fdr"))
    pvalue$translation.pFDR = p.adjust(pvalue$translation, method = c("fdr"))
    pvalue$rotation.pFDR = p.adjust(pvalue$rotation, method = c("fdr"))
    write.csv(pvalue, paste(projDir, "afq-pvalue/", metric, "/", files[n],
        sep = ""), row.names = T)
}
```

## Submit

Run R script:

```
source("/Volumes/wasatch/data/scripts/R21/AFQ/pvalues-afq.R")
```

Your options for DTI metrics are: ***fa***, ***rd***, ***ad***, and ***md***.