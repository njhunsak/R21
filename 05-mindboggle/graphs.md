---
title: Graphs
toc: true
weight: 4
---

## Outlier Function

In R:

```r
outlier <- function(dt, var)
{
    var_name <- eval(substitute(dt[[var]]), eval(dt))
    tot <- sum(!is.na(var_name))
    na1 <- sum(is.na(var_name))
    m1 <- mean(var_name, na.rm = T)
    par(mfrow = c(2, 2), oma = c(0, 0, 3, 0))
    boxplot(var_name, main = "With outliers")
    hist(var_name, main = "With outliers", xlab = NA, ylab = NA)
    outlier <- boxplot.stats(var_name)$out
    mo <- mean(outlier)
    var_name <- ifelse(var_name %in% outlier, NA, var_name)
    boxplot(var_name, main = "Without outliers")
    hist(var_name, main = "Without outliers", xlab = NA, ylab = NA)
    title("Outlier Check", outer = TRUE)
    na2 <- sum(is.na(var_name))
    message("Outliers identified: ", na2 - na1, " from ", tot, " observations")
    message("Proportion (%) of outliers: ", (na2 - na1)/tot * 100)
    message("Mean of the outliers: ", mo)
    m2 <- mean(var_name, na.rm = T)
    message("Mean without removing outliers: ", m1)
    message("Mean if we remove outliers: ", m2)
    dt[as.character(var)] <- var_name
    assign(as.character(as.list(match.call())$dt), dt, envir = .GlobalEnv)
    return(invisible(dt))
    message("Outliers successfully removed", "\n")
}
```

## Cortical Thickness

In R:

``` r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
mydata = read.table("thickinthehead_per_freesurfer_cortex_label.csv", sep = ",", 
    header = T)
clist = c("ctx_lh_caudalanteriorcingulate", "ctx_lh_caudalmiddlefrontal", 
    "ctx_lh_cuneus", "ctx_lh_entorhinal", "ctx_lh_fusiform", "ctx_lh_inferiorparietal", 
    "ctx_lh_inferiortemporal", "ctx_lh_isthmuscingulate", "ctx_lh_lateraloccipital", 
    "ctx_lh_lateralorbitofrontal", "ctx_lh_lingual", "ctx_lh_medialorbitofrontal", 
    "ctx_lh_middletemporal", "ctx_lh_parahippocampal", "ctx_lh_paracentral", 
    "ctx_lh_parsopercularis", "ctx_lh_parsorbitalis", "ctx_lh_parstriangularis", 
    "ctx_lh_pericalcarine", "ctx_lh_postcentral", "ctx_lh_posteriorcingulate", 
    "ctx_lh_precentral", "ctx_lh_precuneus", "ctx_lh_rostralanteriorcingulate", 
    "ctx_lh_rostralmiddlefrontal", "ctx_lh_superiorfrontal", "ctx_lh_superiorparietal", 
    "ctx_lh_superiortemporal", "ctx_lh_supramarginal", "ctx_lh_transversetemporal", 
    "ctx_lh_insula", "ctx_rh_caudalanteriorcingulate", "ctx_rh_caudalmiddlefrontal", 
    "ctx_rh_cuneus", "ctx_rh_entorhinal", "ctx_rh_fusiform", "ctx_rh_inferiorparietal", 
    "ctx_rh_inferiortemporal", "ctx_rh_isthmuscingulate", "ctx_rh_lateraloccipital", 
    "ctx_rh_lateralorbitofrontal", "ctx_rh_lingual", "ctx_rh_medialorbitofrontal", 
    "ctx_rh_middletemporal", "ctx_rh_parahippocampal", "ctx_rh_paracentral", 
    "ctx_rh_parsopercularis", "ctx_rh_parsorbitalis", "ctx_rh_parstriangularis", 
    "ctx_rh_pericalcarine", "ctx_rh_postcentral", "ctx_rh_posteriorcingulate", 
    "ctx_rh_precentral", "ctx_rh_precuneus", "ctx_rh_rostralanteriorcingulate", 
    "ctx_rh_rostralmiddlefrontal", "ctx_rh_superiorfrontal", "ctx_rh_superiorparietal", 
    "ctx_rh_superiortemporal", "ctx_rh_supramarginal", "ctx_rh_transversetemporal", 
    "ctx_rh_insula")
for (i in clist)
{
    outlier(mydata, i)
}
demogr = read.table("/Volumes/wasatch/data/analyses/R21/demographics/R21_demographics.txt", 
    sep = ",", header = T)
mydata = merge(demogr, mydata, by = "studyid", all.y = T)
mydata$group = relevel(mydata$group, "oi")
for (i in 5:(length(mydata)))
{pdf(paste("thickinthehead_", colnames(mydata[i]), ".pdf", sep = ""))
    fit = lm(mydata[[i]] ~ group + age, data = mydata)
    sig = round(anova(fit)[[5]][1], 6)
    sig = p.adjust(sig, method = "fdr", n = 78)
    par(fig = c(0.65, 1, 0, 1))
    boxplot(mydata[[i]] ~ mydata$group, data = mydata, col = (c("black", 
        "red")), axes = FALSE)
    par(fig = c(0, 0.8, 0, 1), new = T)
    plot(mydata$age, mydata[[i]], data = mydata, pch = 19, col = c("black", 
        "red")[unclass(mydata$group)], main = bquote(atop(.(colnames(mydata[i])), 
        p(FDR) == .(sig))), ylab = "cortical thickness in mm", xlab = "age in years")
    abline(lm(subset(mydata, group == "oi")[[i]] ~ subset(mydata, group == 
        "oi")$age), col = "black", lwd = 3)
    abline(lm(subset(mydata, group == "tbi")[[i]] ~ subset(mydata, group == 
        "tbi")$age), col = "red", lwd = 3)
    legend("topleft", legend = c("OI", "TBI"), col = c("black", "red"), 
        lwd = 3, lty = 1, cex = 0.8, box.lty = 0)
    dev.off()
}
```

## Volumetric

In R:

``` r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
mydata = read.table("volume_per_freesurfer_label.csv", sep = ",", header = T)
clist = c("ctx_lh_caudalanteriorcingulate", "ctx_lh_caudalmiddlefrontal", 
    "ctx_lh_cuneus", "ctx_lh_entorhinal", "ctx_lh_fusiform", "ctx_lh_inferiorparietal", 
    "ctx_lh_inferiortemporal", "ctx_lh_isthmuscingulate", "ctx_lh_lateraloccipital", 
    "ctx_lh_lateralorbitofrontal", "ctx_lh_lingual", "ctx_lh_medialorbitofrontal", 
    "ctx_lh_middletemporal", "ctx_lh_parahippocampal", "ctx_lh_paracentral", 
    "ctx_lh_parsopercularis", "ctx_lh_parsorbitalis", "ctx_lh_parstriangularis", 
    "ctx_lh_pericalcarine", "ctx_lh_postcentral", "ctx_lh_posteriorcingulate", 
    "ctx_lh_precentral", "ctx_lh_precuneus", "ctx_lh_rostralanteriorcingulate", 
    "ctx_lh_rostralmiddlefrontal", "ctx_lh_superiorfrontal", "ctx_lh_superiorparietal", 
    "ctx_lh_superiortemporal", "ctx_lh_supramarginal", "ctx_lh_transversetemporal", 
    "ctx_lh_insula", "ctx_rh_caudalanteriorcingulate", "ctx_rh_caudalmiddlefrontal", 
    "ctx_rh_cuneus", "ctx_rh_entorhinal", "ctx_rh_fusiform", "ctx_rh_inferiorparietal", 
    "ctx_rh_inferiortemporal", "ctx_rh_isthmuscingulate", "ctx_rh_lateraloccipital", 
    "ctx_rh_lateralorbitofrontal", "ctx_rh_lingual", "ctx_rh_medialorbitofrontal", 
    "ctx_rh_middletemporal", "ctx_rh_parahippocampal", "ctx_rh_paracentral", 
    "ctx_rh_parsopercularis", "ctx_rh_parsorbitalis", "ctx_rh_parstriangularis", 
    "ctx_rh_pericalcarine", "ctx_rh_postcentral", "ctx_rh_posteriorcingulate", 
    "ctx_rh_precentral", "ctx_rh_precuneus", "ctx_rh_rostralanteriorcingulate", 
    "ctx_rh_rostralmiddlefrontal", "ctx_rh_superiorfrontal", "ctx_rh_superiorparietal", 
    "ctx_rh_superiortemporal", "ctx_rh_supramarginal", "ctx_rh_transversetemporal", 
    "ctx_rh_insula")
for (i in clist)
{
    outlier(mydata, i)
}
demogr = read.table("/Volumes/wasatch/data/analyses/R21/demographics/R21_demographics.txt",
    sep = ",", header = T)
mydata = merge(demogr, mydata, by = "studyid", all.y = T)
mydata$group = relevel(mydata$group, "oi")
for (i in 5:(length(mydata))) {
    pdf(paste("volume_", colnames(mydata[i]), ".pdf", sep = ""))
    fit = lm(mydata[[i]] ~ group + age, data = mydata)
    sig = round(anova(fit)[[5]][1], 6)
    sig = p.adjust(sig, method = "fdr", n = 78)
    par(fig = c(0.65, 1, 0, 1))
    boxplot(mydata[[i]] ~ mydata$group, data = mydata, col = (c("black",
        "red")), axes = FALSE)
    par(fig = c(0, 0.8, 0, 1), new = T)
    plot(mydata$age, mydata[[i]], data = mydata, pch = 19, col = c("black",
        "red")[unclass(mydata$group)], main = bquote(atop(.(colnames(mydata[i])),
        p(FDR) == .(sig))), ylab = bquote("volume in " ~ mm^3), xlab = "age in years")
    abline(lm(subset(mydata, group == "oi")[[i]] ~ subset(mydata, group ==
        "oi")$age), col = "black", lwd = 3)
    abline(lm(subset(mydata, group == "tbi")[[i]] ~ subset(mydata, group ==
        "tbi")$age), col = "red", lwd = 3)
    legend("topleft", legend = c("OI", "TBI"), col = c("black", "red"),
        lwd = 3, lty = 1, cex = 0.8, box.lty = 0)
    dev.off()
}
```

## Area

In R:

``` r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
mydata = read.table("area_per_gyrus_label.csv", sep = ",", header = T)
clist = c("ctx_lh_caudalanteriorcingulate", "ctx_lh_caudalmiddlefrontal", 
    "ctx_lh_cuneus", "ctx_lh_entorhinal", "ctx_lh_fusiform", "ctx_lh_inferiorparietal", 
    "ctx_lh_inferiortemporal", "ctx_lh_isthmuscingulate", "ctx_lh_lateraloccipital", 
    "ctx_lh_lateralorbitofrontal", "ctx_lh_lingual", "ctx_lh_medialorbitofrontal", 
    "ctx_lh_middletemporal", "ctx_lh_parahippocampal", "ctx_lh_paracentral", 
    "ctx_lh_parsopercularis", "ctx_lh_parsorbitalis", "ctx_lh_parstriangularis", 
    "ctx_lh_pericalcarine", "ctx_lh_postcentral", "ctx_lh_posteriorcingulate", 
    "ctx_lh_precentral", "ctx_lh_precuneus", "ctx_lh_rostralanteriorcingulate", 
    "ctx_lh_rostralmiddlefrontal", "ctx_lh_superiorfrontal", "ctx_lh_superiorparietal", 
    "ctx_lh_superiortemporal", "ctx_lh_supramarginal", "ctx_lh_transversetemporal", 
    "ctx_lh_insula", "ctx_rh_caudalanteriorcingulate", "ctx_rh_caudalmiddlefrontal", 
    "ctx_rh_cuneus", "ctx_rh_entorhinal", "ctx_rh_fusiform", "ctx_rh_inferiorparietal", 
    "ctx_rh_inferiortemporal", "ctx_rh_isthmuscingulate", "ctx_rh_lateraloccipital", 
    "ctx_rh_lateralorbitofrontal", "ctx_rh_lingual", "ctx_rh_medialorbitofrontal", 
    "ctx_rh_middletemporal", "ctx_rh_parahippocampal", "ctx_rh_paracentral", 
    "ctx_rh_parsopercularis", "ctx_rh_parsorbitalis", "ctx_rh_parstriangularis", 
    "ctx_rh_pericalcarine", "ctx_rh_postcentral", "ctx_rh_posteriorcingulate", 
    "ctx_rh_precentral", "ctx_rh_precuneus", "ctx_rh_rostralanteriorcingulate", 
    "ctx_rh_rostralmiddlefrontal", "ctx_rh_superiorfrontal", "ctx_rh_superiorparietal", 
    "ctx_rh_superiortemporal", "ctx_rh_supramarginal", "ctx_rh_transversetemporal", 
    "ctx_rh_insula")
for (i in clist)
{
    outlier(mydata, i)
}
demogr = read.table("/Volumes/wasatch/data/analyses/R21/demographics/R21_demographics.txt",
    sep = ",", header = T)
mydata = merge(demogr, mydata, by = "studyid", all.y = T)
mydata$group = relevel(mydata$group, "oi")
for (i in 5:(length(mydata)))
{
    pdf(paste("area_", colnames(mydata[i]), ".pdf", sep = ""))
    fit = lm(mydata[[i]] ~ group + age, data = mydata)
    sig = round(anova(fit)[[5]][1], 6)
    sig = p.adjust(sig, method = "fdr", n = 78)
    par(fig = c(0.65, 1, 0, 1))
    boxplot(mydata[[i]] ~ mydata$group, data = mydata, col = (c("black",
        "red")), axes = FALSE)
    par(fig = c(0, 0.8, 0, 1), new = T)
    plot(mydata$age, mydata[[i]], data = mydata, pch = 19, col = c("black",
        "red")[unclass(mydata$group)], main = bquote(atop(.(colnames(mydata[i])),
        p(FDR) == .(sig))), ylab = bquote("area in " ~ mm^2), xlab = "age in years")
    abline(lm(subset(mydata, group == "oi")[[i]] ~ subset(mydata, group ==
        "oi")$age), col = "black", lwd = 3)
    abline(lm(subset(mydata, group == "tbi")[[i]] ~ subset(mydata, group ==
        "tbi")$age), col = "red", lwd = 3)
    legend("topleft", legend = c("OI", "TBI"), col = c("black", "red"),
        lwd = 3, lty = 1, cex = 0.8, box.lty = 0)
    dev.off()
}
```

## Travel Depths

In R:

``` r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
mydata = read.table("travel_depth_per_gyrus_label.csv", sep = ",", header = T)
clist = c("ctx_lh_caudalanteriorcingulate", "ctx_lh_caudalmiddlefrontal", 
    "ctx_lh_cuneus", "ctx_lh_entorhinal", "ctx_lh_fusiform", "ctx_lh_inferiorparietal", 
    "ctx_lh_inferiortemporal", "ctx_lh_isthmuscingulate", "ctx_lh_lateraloccipital", 
    "ctx_lh_lateralorbitofrontal", "ctx_lh_lingual", "ctx_lh_medialorbitofrontal", 
    "ctx_lh_middletemporal", "ctx_lh_parahippocampal", "ctx_lh_paracentral", 
    "ctx_lh_parsopercularis", "ctx_lh_parsorbitalis", "ctx_lh_parstriangularis", 
    "ctx_lh_pericalcarine", "ctx_lh_postcentral", "ctx_lh_posteriorcingulate", 
    "ctx_lh_precentral", "ctx_lh_precuneus", "ctx_lh_rostralanteriorcingulate", 
    "ctx_lh_rostralmiddlefrontal", "ctx_lh_superiorfrontal", "ctx_lh_superiorparietal", 
    "ctx_lh_superiortemporal", "ctx_lh_supramarginal", "ctx_lh_transversetemporal", 
    "ctx_lh_insula", "ctx_rh_caudalanteriorcingulate", "ctx_rh_caudalmiddlefrontal", 
    "ctx_rh_cuneus", "ctx_rh_entorhinal", "ctx_rh_fusiform", "ctx_rh_inferiorparietal", 
    "ctx_rh_inferiortemporal", "ctx_rh_isthmuscingulate", "ctx_rh_lateraloccipital", 
    "ctx_rh_lateralorbitofrontal", "ctx_rh_lingual", "ctx_rh_medialorbitofrontal", 
    "ctx_rh_middletemporal", "ctx_rh_parahippocampal", "ctx_rh_paracentral", 
    "ctx_rh_parsopercularis", "ctx_rh_parsorbitalis", "ctx_rh_parstriangularis", 
    "ctx_rh_pericalcarine", "ctx_rh_postcentral", "ctx_rh_posteriorcingulate", 
    "ctx_rh_precentral", "ctx_rh_precuneus", "ctx_rh_rostralanteriorcingulate", 
    "ctx_rh_rostralmiddlefrontal", "ctx_rh_superiorfrontal", "ctx_rh_superiorparietal", 
    "ctx_rh_superiortemporal", "ctx_rh_supramarginal", "ctx_rh_transversetemporal", 
    "ctx_rh_insula")
for (i in clist)
{
    outlier(mydata, i)
}
demogr = read.table("/Volumes/wasatch/data/analyses/R21/demographics/R21_demographics.txt",
    sep = ",", header = T)
mydata = merge(demogr, mydata, by = "studyid", all.y = T)
mydata$group = relevel(mydata$group, "oi")
for (i in 5:(length(mydata)))
{pdf(paste("travel_", colnames(mydata[i]), ".pdf", sep = ""))
    fit = lm(mydata[[i]] ~ group + age, data = mydata)
    sig = round(anova(fit)[[5]][1], 6)
    sig = p.adjust(sig, method = "fdr", n = 78)
    par(fig = c(0.65, 1, 0, 1))
    boxplot(mydata[[i]] ~ mydata$group, data = mydata, col = (c("black",
        "red")), axes = FALSE)
    par(fig = c(0, 0.8, 0, 1), new = T)
    plot(mydata$age, mydata[[i]], data = mydata, pch = 19, col = c("black",
        "red")[unclass(mydata$group)], main = bquote(atop(.(colnames(mydata[i])),
        p(FDR) == .(sig))), ylab = "travel depth", xlab = "age in years")
    abline(lm(subset(mydata, group == "oi")[[i]] ~ subset(mydata, group ==
        "oi")$age), col = "black", lwd = 3)
    abline(lm(subset(mydata, group == "tbi")[[i]] ~ subset(mydata, group ==
        "tbi")$age), col = "red", lwd = 3)
    legend("topleft", legend = c("OI", "TBI"), col = c("black", "red"),
        lwd = 3, lty = 1, cex = 0.8, box.lty = 0)
    dev.off()
}
```

## Geodesic Depth

In R:

``` r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
mydata = read.table("geodesic_depth_per_gyrus_label.csv", sep = ",", header = T)
clist = c("ctx_lh_caudalanteriorcingulate", "ctx_lh_caudalmiddlefrontal", 
    "ctx_lh_cuneus", "ctx_lh_entorhinal", "ctx_lh_fusiform", "ctx_lh_inferiorparietal", 
    "ctx_lh_inferiortemporal", "ctx_lh_isthmuscingulate", "ctx_lh_lateraloccipital", 
    "ctx_lh_lateralorbitofrontal", "ctx_lh_lingual", "ctx_lh_medialorbitofrontal", 
    "ctx_lh_middletemporal", "ctx_lh_parahippocampal", "ctx_lh_paracentral", 
    "ctx_lh_parsopercularis", "ctx_lh_parsorbitalis", "ctx_lh_parstriangularis", 
    "ctx_lh_pericalcarine", "ctx_lh_postcentral", "ctx_lh_posteriorcingulate", 
    "ctx_lh_precentral", "ctx_lh_precuneus", "ctx_lh_rostralanteriorcingulate", 
    "ctx_lh_rostralmiddlefrontal", "ctx_lh_superiorfrontal", "ctx_lh_superiorparietal", 
    "ctx_lh_superiortemporal", "ctx_lh_supramarginal", "ctx_lh_transversetemporal", 
    "ctx_lh_insula", "ctx_rh_caudalanteriorcingulate", "ctx_rh_caudalmiddlefrontal", 
    "ctx_rh_cuneus", "ctx_rh_entorhinal", "ctx_rh_fusiform", "ctx_rh_inferiorparietal", 
    "ctx_rh_inferiortemporal", "ctx_rh_isthmuscingulate", "ctx_rh_lateraloccipital", 
    "ctx_rh_lateralorbitofrontal", "ctx_rh_lingual", "ctx_rh_medialorbitofrontal", 
    "ctx_rh_middletemporal", "ctx_rh_parahippocampal", "ctx_rh_paracentral", 
    "ctx_rh_parsopercularis", "ctx_rh_parsorbitalis", "ctx_rh_parstriangularis", 
    "ctx_rh_pericalcarine", "ctx_rh_postcentral", "ctx_rh_posteriorcingulate", 
    "ctx_rh_precentral", "ctx_rh_precuneus", "ctx_rh_rostralanteriorcingulate", 
    "ctx_rh_rostralmiddlefrontal", "ctx_rh_superiorfrontal", "ctx_rh_superiorparietal", 
    "ctx_rh_superiortemporal", "ctx_rh_supramarginal", "ctx_rh_transversetemporal", 
    "ctx_rh_insula")
for (i in clist)
{
    outlier(mydata, i)
}
demogr = read.table("/Volumes/wasatch/data/analyses/R21/demographics/R21_demographics.txt",
    sep = ",", header = T)
mydata = merge(demogr, mydata, by = "studyid", all.y = T)
mydata$group = relevel(mydata$group, "oi")
for (i in 5:(length(mydata)))
{pdf(paste("geodesic_", colnames(mydata[i]), ".pdf", sep = ""))
    fit = lm(mydata[[i]] ~ group + age, data = mydata)
    sig = round(anova(fit)[[5]][1], 6)
    sig = p.adjust(sig, method = "fdr", n = 78)
    par(fig = c(0.65, 1, 0, 1))
    boxplot(mydata[[i]] ~ mydata$group, data = mydata, col = (c("black",
        "red")), axes = FALSE)
    par(fig = c(0, 0.8, 0, 1), new = T)
    plot(mydata$age, mydata[[i]], data = mydata, pch = 19, col = c("black",
        "red")[unclass(mydata$group)], main = bquote(atop(.(colnames(mydata[i])),
        p(FDR) == .(sig))), ylab = "geodesic depth", xlab = "age in years")
    abline(lm(subset(mydata, group == "oi")[[i]] ~ subset(mydata, group ==
        "oi")$age), col = "black", lwd = 3)
    abline(lm(subset(mydata, group == "tbi")[[i]] ~ subset(mydata, group ==
        "tbi")$age), col = "red", lwd = 3)
    legend("topleft", legend = c("OI", "TBI"), col = c("black", "red"),
        lwd = 3, lty = 1, cex = 0.8, box.lty = 0)
    dev.off()
}
```

## Curvature

In R:

``` r
setwd("/Volumes/wasatch/data/stats/R21/mindboggle/")
mydata = read.table("curvature_per_gyrus_label.csv", sep = ",", header = T)
clist = c("ctx_lh_caudalanteriorcingulate", "ctx_lh_caudalmiddlefrontal", 
    "ctx_lh_cuneus", "ctx_lh_entorhinal", "ctx_lh_fusiform", "ctx_lh_inferiorparietal", 
    "ctx_lh_inferiortemporal", "ctx_lh_isthmuscingulate", "ctx_lh_lateraloccipital", 
    "ctx_lh_lateralorbitofrontal", "ctx_lh_lingual", "ctx_lh_medialorbitofrontal", 
    "ctx_lh_middletemporal", "ctx_lh_parahippocampal", "ctx_lh_paracentral", 
    "ctx_lh_parsopercularis", "ctx_lh_parsorbitalis", "ctx_lh_parstriangularis", 
    "ctx_lh_pericalcarine", "ctx_lh_postcentral", "ctx_lh_posteriorcingulate", 
    "ctx_lh_precentral", "ctx_lh_precuneus", "ctx_lh_rostralanteriorcingulate", 
    "ctx_lh_rostralmiddlefrontal", "ctx_lh_superiorfrontal", "ctx_lh_superiorparietal", 
    "ctx_lh_superiortemporal", "ctx_lh_supramarginal", "ctx_lh_transversetemporal", 
    "ctx_lh_insula", "ctx_rh_caudalanteriorcingulate", "ctx_rh_caudalmiddlefrontal", 
    "ctx_rh_cuneus", "ctx_rh_entorhinal", "ctx_rh_fusiform", "ctx_rh_inferiorparietal", 
    "ctx_rh_inferiortemporal", "ctx_rh_isthmuscingulate", "ctx_rh_lateraloccipital", 
    "ctx_rh_lateralorbitofrontal", "ctx_rh_lingual", "ctx_rh_medialorbitofrontal", 
    "ctx_rh_middletemporal", "ctx_rh_parahippocampal", "ctx_rh_paracentral", 
    "ctx_rh_parsopercularis", "ctx_rh_parsorbitalis", "ctx_rh_parstriangularis", 
    "ctx_rh_pericalcarine", "ctx_rh_postcentral", "ctx_rh_posteriorcingulate", 
    "ctx_rh_precentral", "ctx_rh_precuneus", "ctx_rh_rostralanteriorcingulate", 
    "ctx_rh_rostralmiddlefrontal", "ctx_rh_superiorfrontal", "ctx_rh_superiorparietal", 
    "ctx_rh_superiortemporal", "ctx_rh_supramarginal", "ctx_rh_transversetemporal", 
    "ctx_rh_insula")
for (i in clist)
{
    outlier(mydata, i)
}
demogr = read.table("/Volumes/wasatch/data/analyses/R21/demographics/R21_demographics.txt",
    sep = ",", header = T)
mydata = merge(demogr, mydata, by = "studyid", all.y = T)
mydata$group = relevel(mydata$group, "oi")
pdf("curvature_per_gyrus_label.pdf")
for (i in 5:(length(mydata)))
{pdf(paste("curvature_", colnames(mydata[i]), ".pdf", sep = ""))
    fit = lm(mydata[[i]] ~ group + age, data = mydata)
    sig = round(anova(fit)[[5]][1], 6)
    sig = p.adjust(sig, method = "fdr", n = 78)
    par(fig = c(0.65, 1, 0, 1))
    boxplot(mydata[[i]] ~ mydata$group, data = mydata, col = (c("black",
        "red")), axes = FALSE)
    par(fig = c(0, 0.8, 0, 1), new = T)
    plot(mydata$age, mydata[[i]], data = mydata, pch = 19, col = c("black",
        "red")[unclass(mydata$group)], main = bquote(atop(.(colnames(mydata[i])),
        p(FDR) == .(sig))), ylab = "curvature", xlab = "age in years")
    abline(lm(subset(mydata, group == "oi")[[i]] ~ subset(mydata, group ==
        "oi")$age), col = "black", lwd = 3)
    abline(lm(subset(mydata, group == "tbi")[[i]] ~ subset(mydata, group ==
        "tbi")$age), col = "red", lwd = 3)
    legend("topleft", legend = c("OI", "TBI"), col = c("black", "red"),
        lwd = 3, lty = 1, cex = 0.8, box.lty = 0)
    dev.off()
}
```

## Montage

To create montage of all the graphs and clean up the directory. In the Terminal window:

```bash
vi /Volumes/wasatch/data/stats/R21/mindboggle/montage.sh
```
Copy and paste:

```bash
cd /Volumes/wasatch/data/stats/R21/mindboggle/
montage *ctx_lh_caudalanteriorcingulate.pdf -geometry +0+0 ctx_lh_caudalanteriorcingulate.png
montage *ctx_lh_caudalmiddlefrontal.pdf -geometry +0+0 ctx_lh_caudalmiddlefrontal.png
montage *ctx_lh_cuneus.pdf -geometry +0+0 ctx_lh_cuneus.png
montage *ctx_lh_entorhinal.pdf -geometry +0+0 ctx_lh_entorhinal.png
montage *ctx_lh_fusiform.pdf -geometry +0+0 ctx_lh_fusiform.png
montage *ctx_lh_inferiorparietal.pdf -geometry +0+0 ctx_lh_inferiorparietal.png
montage *ctx_lh_inferiortemporal.pdf -geometry +0+0 ctx_lh_inferiortemporal.png
montage *ctx_lh_insula.pdf -geometry +0+0 ctx_lh_insula.png
montage *ctx_lh_isthmuscingulate.pdf -geometry +0+0 ctx_lh_isthmuscingulate.png
montage *ctx_lh_lateraloccipital.pdf -geometry +0+0 ctx_lh_lateraloccipital.png
montage *ctx_lh_lateralorbitofrontal.pdf -geometry +0+0 ctx_lh_lateralorbitofrontal.png
montage *ctx_lh_lingual.pdf -geometry +0+0 ctx_lh_lingual.png
montage *ctx_lh_medialorbitofrontal.pdf -geometry +0+0 ctx_lh_medialorbitofrontal.png
montage *ctx_lh_middletemporal.pdf -geometry +0+0 ctx_lh_middletemporal.png
montage *ctx_lh_paracentral.pdf -geometry +0+0 ctx_lh_paracentral.png
montage *ctx_lh_parahippocampal.pdf -geometry +0+0 ctx_lh_parahippocampal.png
montage *ctx_lh_parsopercularis.pdf -geometry +0+0 ctx_lh_parsopercularis.png
montage *ctx_lh_parsorbitalis.pdf -geometry +0+0 ctx_lh_parsorbitalis.png
montage *ctx_lh_parstriangularis.pdf -geometry +0+0 ctx_lh_parstriangularis.png
montage *ctx_lh_pericalcarine.pdf -geometry +0+0 ctx_lh_pericalcarine.png
montage *ctx_lh_postcentral.pdf -geometry +0+0 ctx_lh_postcentral.png
montage *ctx_lh_posteriorcingulate.pdf -geometry +0+0 ctx_lh_posteriorcingulate.png
montage *ctx_lh_precentral.pdf -geometry +0+0 ctx_lh_precentral.png
montage *ctx_lh_precuneus.pdf -geometry +0+0 ctx_lh_precuneus.png
montage *ctx_lh_rostralanteriorcingulate.pdf -geometry +0+0 ctx_lh_rostralanteriorcingulate.png
montage *ctx_lh_rostralmiddlefrontal.pdf -geometry +0+0 ctx_lh_rostralmiddlefrontal.png
montage *ctx_lh_superiorfrontal.pdf -geometry +0+0 ctx_lh_superiorfrontal.png
montage *ctx_lh_superiorparietal.pdf -geometry +0+0 ctx_lh_superiorparietal.png
montage *ctx_lh_superiortemporal.pdf -geometry +0+0 ctx_lh_superiortemporal.png
montage *ctx_lh_supramarginal.pdf -geometry +0+0 ctx_lh_supramarginal.png
montage *ctx_lh_transversetemporal.pdf -geometry +0+0 ctx_lh_transversetemporal.png
montage *ctx_rh_caudalanteriorcingulate.pdf -geometry +0+0 ctx_rh_caudalanteriorcingulate.png
montage *ctx_rh_caudalmiddlefrontal.pdf -geometry +0+0 ctx_rh_caudalmiddlefrontal.png
montage *ctx_rh_cuneus.pdf -geometry +0+0 ctx_rh_cuneus.png
montage *ctx_rh_entorhinal.pdf -geometry +0+0 ctx_rh_entorhinal.png
montage *ctx_rh_fusiform.pdf -geometry +0+0 ctx_rh_fusiform.png
montage *ctx_rh_inferiorparietal.pdf -geometry +0+0 ctx_rh_inferiorparietal.png
montage *ctx_rh_inferiortemporal.pdf -geometry +0+0 ctx_rh_inferiortemporal.png
montage *ctx_rh_insula.pdf -geometry +0+0 ctx_rh_insula.png
montage *ctx_rh_isthmuscingulate.pdf -geometry +0+0 ctx_rh_isthmuscingulate.png
montage *ctx_rh_lateraloccipital.pdf -geometry +0+0 ctx_rh_lateraloccipital.png
montage *ctx_rh_lateralorbitofrontal.pdf -geometry +0+0 ctx_rh_lateralorbitofrontal.png
montage *ctx_rh_lingual.pdf -geometry +0+0 ctx_rh_lingual.png
montage *ctx_rh_medialorbitofrontal.pdf -geometry +0+0 ctx_rh_medialorbitofrontal.png
montage *ctx_rh_middletemporal.pdf -geometry +0+0 ctx_rh_middletemporal.png
montage *ctx_rh_paracentral.pdf -geometry +0+0 ctx_rh_paracentral.png
montage *ctx_rh_parahippocampal.pdf -geometry +0+0 ctx_rh_parahippocampal.png
montage *ctx_rh_parsopercularis.pdf -geometry +0+0 ctx_rh_parsopercularis.png
montage *ctx_rh_parsorbitalis.pdf -geometry +0+0 ctx_rh_parsorbitalis.png
montage *ctx_rh_parstriangularis.pdf -geometry +0+0 ctx_rh_parstriangularis.png
montage *ctx_rh_pericalcarine.pdf -geometry +0+0 ctx_rh_pericalcarine.png
montage *ctx_rh_postcentral.pdf -geometry +0+0 ctx_rh_postcentral.png
montage *ctx_rh_posteriorcingulate.pdf -geometry +0+0 ctx_rh_posteriorcingulate.png
montage *ctx_rh_precentral.pdf -geometry +0+0 ctx_rh_precentral.png
montage *ctx_rh_precuneus.pdf -geometry +0+0 ctx_rh_precuneus.png
montage *ctx_rh_rostralanteriorcingulate.pdf -geometry +0+0 ctx_rh_rostralanteriorcingulate.png
montage *ctx_rh_rostralmiddlefrontal.pdf -geometry +0+0 ctx_rh_rostralmiddlefrontal.png
montage *ctx_rh_superiorfrontal.pdf -geometry +0+0 ctx_rh_superiorfrontal.png
montage *ctx_rh_superiorparietal.pdf -geometry +0+0 ctx_rh_superiorparietal.png
montage *ctx_rh_superiortemporal.pdf -geometry +0+0 ctx_rh_superiortemporal.png
montage *ctx_rh_supramarginal.pdf -geometry +0+0 ctx_rh_supramarginal.png
montage *ctx_rh_transversetemporal.pdf -geometry +0+0 ctx_rh_transversetemporal.png
```

Run in the Terminal:

```bash
sh /Volumes/wasatch/data/stats/R21/mindboggle/montage.sh
```