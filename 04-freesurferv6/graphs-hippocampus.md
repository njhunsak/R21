---
title: Graphs - Hippocampal Subfields
toc: true
weight: 5
---

## Generate Data Sheet

```bash
cd ~/compute/analyses/MIOS/
quantifyHippocampalSubfields.sh \
T1 \
~/compute/stats/MIOS/FreeSurferv6/HippocampalSubfields.txt \
~/compute/analyses/MIOS/FreeSurferv6/
```

## Sync

Sync files locally:

```bash
rsync -rauv \
intj5@ssh.fsl.byu.edu:~/compute/stats/MIOS/FreeSurferv6/ \
/Volumes/wasatch/data/stats/MIOS/FreeSurferv6/
```

## Graphs

Open R and generate graphs:

```r
setwd("/Volumes/wasatch/data/stats/R21/FreeSurferv6/")
mydata = read.table("HippocampalSubfields.txt", header = T, sep = "\t")
demo = read.table("/Volumes/wasatch/data/analyses/R21/demographics/R21_demographics.txt",
    header = T, sep = ",")
mydata = merge(demo, mydata, by = 1)
mydata = mydata[-which(mydata$studyid == 44), ]
pdf("HippocampalSubfields.pdf")
par(oma = c(2, 2, 2, 2))
for (i in 6:length(mydata))
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

## Participant 37

Open R and generate graphs:

```r
setwd("/Volumes/wasatch/data/stats/R21/FreeSurferv6/")
mydata = read.table("HippocampalSubfields.txt", header = T, sep = "\t")
demo = read.table("/Volumes/wasatch/data/analyses/R21/demographics/R21_demographics.txt",
    header = T, sep = ",")
mydata = merge(demo, mydata, by = 1)
mydata = mydata[-which(mydata$studyid == 37), ]
mydata = mydata[-which(mydata$studyid == 44), ]
pdf("HippocampalSubfields-37.pdf")
par(oma = c(2, 2, 2, 2))
for (i in 6:length(mydata))
{
    fit = lm(mydata[[i]] ~ group + age, data = mydata)
    sig = round(summary(fit)$coefficients[2, 4], 6)
    par(fig = c(0.65, 1, 0, 1))
    boxplot(mydata[[i]] ~ mydata$group, data = mydata, col = (c("cyan",
        "deeppink")), axes = FALSE, xlim = c(0.5, 3.5))
    points(3, outlier[[i]], type = "p", pch = 17, cex = 2)
    par(fig = c(0, 0.8, 0, 1), new = T)
    plot(mydata$age, mydata[[i]], data = mydata, pch = 19, col = c("cyan",
        "deeppink")[unclass(mydata$group)], main = bquote(atop(.(colnames(mydata[i])),
        p(group) == .(sig))), ylab = expression(paste("Volume in ", mm^3,
        sep = "")), xlab = "age in years")
    abline(lm(subset(mydata, group == "oi")[[i]] ~ subset(mydata, group ==
        "oi")$age), col = "cyan", lwd = 3)
    abline(lm(subset(mydata, group == "tbi")[[i]] ~ subset(mydata, group ==
        "tbi")$age), col = "deeppink", lwd = 3)
    points(outlier$age, outlier[[i]], type = "p", pch = 17, cex = 2)
    legend("topleft", legend = c("OI", "TBI", "#37"), col = c("cyan", "deeppink",
        "black"), lwd = 3, lty = 1, cex = 0.8, box.lty = 0)
}
dev.off()
```

## Biomarkers

Open R and generate graphs:

```r
setwd("/Volumes/wasatch/data/stats/R21/FreeSurferv6/")
mydata = read.table("HippocampalSubfields.txt", header = T, sep = "\t")
demo = read.table("/Volumes/wasatch/data/analyses/R21/demographics/R21_demographics.txt",
    header = T, sep = ",")
microRNA = read.csv("/Volumes/wasatch/data/analyses/R21/biomarkers/mircroRNA.csv",
    header = T, sep = ",", check.names = FALSE)
protein = read.csv("/Volumes/wasatch/data/analyses/R21/biomarkers/protein.csv",
    header = T, sep = ",", check.names = FALSE)[c(1, 2, 3, 5, 6, 7, 8,
    9, 10, 11, 13, 14, 15, 16, 18, 19)]
mydata = merge(demo, mydata, by = 1, all.y = T)[c(1:4, 6:17, 20:31)]
mydata = mydata[-which(mydata$studyid == 45), ]
dataset1 = merge(mydata, microRNA, by = 1, all.x = T)
dataset2 = merge(mydata, protein, by = 1, all.x = T)

# TABLE
tmp=describeBy(dataset2, group=dataset2$group)
tmp2=describeBy(dataset1, group=dataset1$group)
table=data.frame(cbind(rownames(tmp$oi),tmp$oi$n,paste(round(tmp$oi$mean,2), "±", round(tmp$oi$se,2), sep=" "),tmp$tbi$n,paste(round(tmp$tbi$mean,2), "±", round(tmp$tbi$se,2), sep=" ")))
table2=data.frame(cbind(rownames(tmp2$oi),tmp2$oi$n,paste(round(tmp2$oi$mean,2), "±", round(tmp2$oi$se,2), sep=" "),tmp2$tbi$n,paste(round(tmp2$tbi$mean,2), "±", round(tmp2$tbi$se,2), sep=" ")))
table=data.frame(rbind(table[3:28,],table2[29:51,],table[29:43,]))
write.table(table,"tmp.csv",sep=",",col.names=F,row.names=F)

# MICRORNA
results1 = rcorr(as.matrix(dataset1[c(5:51)]))

pdf("/Volumes/wasatch/data/stats/R21/FreeSurferv6/HippocampalSubfields-microRNA.pdf",
    width = 8, height = 8)
par(xpd = TRUE)
corrplot(results1$r[1:24, 25:47], method = "ellipse", type = "full", tl.cex = 1,
    tl.col = "black", is.corr = FALSE, outline = FALSE, tl.srt = 90, mar = c(0,
        0, 3, 0), cl.pos = "b")
title("Correlation Matrix")
dev.off()

pdf("/Volumes/wasatch/data/stats/R21/FreeSurferv6/HippocampalSubfields-microRNA-sig.pdf",
    width = 8, height = 8)
par(xpd = TRUE)
corrplot(results1$r[1:24, 25:47], method = "square", type = "full", tl.cex = 1,
    tl.col = "black", is.corr = FALSE, p.mat = results1$P[1:24, 25:47],
    sig.level = 0.019, insig = "blank", outline = FALSE, tl.srt = 90, mar = c(0,
        0, 3, 0), cl.pos = "b")
title("p < 0.01")
dev.off()

for (ii in 29:length(dataset1))
{
    pdf(paste("/Volumes/wasatch/data/stats/R21/FreeSurferv6/microRNA/", colnames(dataset1[ii]),
        ".pdf", sep = ""))
    for (i in 5:28)
    {
        par(oma = c(1, 1, 1, 1))
        par(fig = c(0.65, 1, 0, 0.8))
        boxplot(dataset1[[ii]] ~ dataset1$group, col = (c("cyan", "deeppink")),
            axes = FALSE)
        mtext("OI", line = 1, side = 1, at = 1)
        mtext("TBI", line = 1, side = 1, at = 2)
        par(fig = c(0, 0.8, 0.55, 1), new = T)
        boxplot(dataset1[[i]] ~ dataset1$group, col = (c("cyan", "deeppink")),
            axes = FALSE, horizontal = TRUE)
        par(fig = c(0, 0.8, 0, 0.8), new = T)
        plot(dataset1[[i]], dataset1[[ii]], pch = 19, col = c("cyan", "deeppink")[unclass(dataset1$group)],
            xlab = colnames(dataset1[i]), ylab = colnames(dataset1[ii]))
        abline(lm(dataset1[[ii]] ~ dataset1[[i]]), col = "black", lwd = 3)
        r.text = bquote(r == .(round(results1$r[i - 4, ii - 4], 2)))
        p.text = bquote(p(unadj) == .(round(results1$P[i - 4, ii - 4],
            2)))
        mtext(paste(c(r.text, p.text), collapse = ", "), line = -3, side = 3,
            outer = TRUE)
    }
    dev.off()
}

## PROTEIN
results2 = rcorr(as.matrix(dataset2[c(5:length(dataset2))]))

pdf("/Volumes/wasatch/data/stats/R21/FreeSurferv6/HippocampalSubfields-protein.pdf",
    width = 8, height = 8)
par(xpd = TRUE)
corrplot(results2$r[1:24, 25:39], method = "ellipse", type = "full", tl.cex = 1,
    tl.col = "black", is.corr = FALSE, outline = FALSE, tl.srt = 90, mar = c(0,
        0, 2, 0), title = "Correlation Matrix", cl.pos = "b")
dev.off()

pdf("/Volumes/wasatch/data/stats/R21/FreeSurferv6/HippocampalSubfields-protein-sig.pdf",
    width = 8, height = 8)
par(xpd = TRUE)
corrplot(results2$r[1:24, 25:39], method = "square", type = "full", tl.cex = 1,
    tl.col = "black", is.corr = FALSE, p.mat = results2$P[1:24, 25:39],
    sig.level = 0.019, insig = "blank", outline = FALSE, tl.srt = 90, mar = c(0,
        0, 2, 0), title = "p < 0.01", cl.pos = "b")
dev.off()

for (ii in 29:length(dataset2))
{
    pdf(paste("/Volumes/wasatch/data/stats/R21/FreeSurferv6/protein/", colnames(dataset2[ii]),
        ".pdf", sep = ""))
    for (i in 5:28)
    {
        if (colSums(is.na(dataset2[ii])) != nrow(dataset2[ii]))
        {
            par(oma = c(1, 1, 1, 1))
            par(fig = c(0.65, 1, 0, 0.8))
            boxplot(dataset2[[ii]] ~ dataset2$group, col = (c("cyan", "deeppink")),
                axes = FALSE)
            mtext("OI", line = 1, side = 1, at = 1)
            mtext("TBI", line = 1, side = 1, at = 2)
            par(fig = c(0, 0.8, 0.55, 1), new = T)
            boxplot(dataset2[[i]] ~ dataset2$group, col = (c("cyan", "deeppink")),
                axes = FALSE, horizontal = TRUE)
            par(fig = c(0, 0.8, 0, 0.8), new = T)
            plot(dataset2[[i]], dataset2[[ii]], pch = 19, col = c("cyan",
                "deeppink")[unclass(dataset2$group)], xlab = colnames(dataset2[i]),
                ylab = colnames(dataset2[ii]))
            try(abline(lm(dataset2[[ii]] ~ dataset2[[i]]), col = "black",
                lwd = 3))
            r.text = bquote(r == .(round(results2$r[i - 4, ii - 4], 2)))
            p.text = bquote(p(unadj) == .(round(results2$P[i - 4, ii -
                4], 2)))
            mtext(paste(c(r.text, p.text), collapse = ", "), line = -3,
                side = 3, outer = TRUE)
        }
    }
    dev.off()
}
```