---
title: Graphs - Brainstem
toc: true
weight: 4
---

## Generate Data Sheets

```bash
cd ~/compute/analyses/MIOS/
quantifyBrainstemStructures.sh \
~/compute/stats/MIOS/FreeSurferv6/Brainstem.txt \
~/compute/analyses/MIOS/FreeSurferv6/
```

## Sync

Sync files locally:

```bash
rsync -rauv \
intj5@ssh.fsl.byu.edu:~/compute/stats/MIOS/FreeSurferv6/ \
/Volumes/wasatch/data/stats/MIOS/FreeSurferv6/
```

## Graph

Open R and generate graphs:

```r
setwd("/Volumes/wasatch/data/stats/MIOS/FreeSurferv6/")
mydata = read.table("Brainstem.txt", header = T, sep = " ")
demo = read.table("/Volumes/wasatch/data/analyses/MIOS/demographics/MIOS_demographics.txt",
    header = T, sep = ",")
mydata = merge(demo, mydata, by = 1)
pdf("Brainstem.pdf")
par(oma = c(2, 2, 2, 2))
for (i in 9:length(mydata))
{
    fit = lm(mydata[[i]] ~ group + age, data = mydata)
    sig = round(summary(fit)$coefficients[2, 4], 6)
    par(fig = c(0.65, 1, 0, 1))
    boxplot(mydata[[i]] ~ mydata$group, data = mydata, col = (c("cyan",
        "deeppink")), axes = FALSE)
    par(fig = c(0, 0.8, 0, 1), new = T)
    plot(mydata$age, mydata[[i]], data = mydata, pch = 19, col = c("cyan",
        "deeppink")[unclass(mydata$group)], main = bquote(atop(.(colnames(mydata[i])),
        p(group) == .(sig))), ylab = expression(paste("Volume in ", mm^3,
        sep = "")), xlab = "age in years")
    abline(lm(subset(mydata, group == "oi")[[i]] ~ subset(mydata, group ==
        "oi")$age), col = "cyan", lwd = 3)
    abline(lm(subset(mydata, group == "tbi")[[i]] ~ subset(mydata, group ==
        "tbi")$age), col = "deeppink", lwd = 3)
    legend("topleft", legend = c("OI", "TBI"), col = c("cyan", "deeppink"),
        lwd = 3, lty = 1, cex = 0.8, box.lty = 0)
}
dev.off()
```