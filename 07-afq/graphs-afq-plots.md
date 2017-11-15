---
title: Graphs - AFQ Plots
toc: true
weight: 11
---

## R Script

For group plot, create an R script:

```bash
vi /Volumes/wasatch/data/scripts/R21/AFQ/plot-afq-group.R
```

Copy and paste:

```r
metric <- readline(prompt = "What DTI metric do you want to analyze (fa, rd, md, ad): ")
projDir = "/Volumes/wasatch/data/stats/R21/AFQ/"
files = list.files(paste(projDir, "afq/", sep = ""), pattern = ".csv", 
    full.names = FALSE)
for (n in 1:length(files))
{
    mydata = read.table(paste(projDir, "afq/", files[n], sep = ""), 
        sep = ",", header = TRUE)
    pvalue = read.table(paste(projDir, "afq-pvalue/", metric, 
        "/", files[n], sep = ""), sep = ",", header = T, row.names = 1)
    melted = subset(melt(mydata, id = c("studyid", "group", "dtiscalar")), 
        dtiscalar == metric)
    melted = group.STDERR(value ~ variable * group, melted)
    names(melted) = c("node", "group", "lwr", "mean", "upr")
    p = ggplot() + geom_line(data = melted, aes(x = as.numeric(node), 
        y = mean, colour = group, linetype = group), size = 0.5) + 
        geom_ribbon(show.legend = F, data = melted, aes(x = as.numeric(node), 
            ymin = lwr, ymax = upr, fill = group), alpha = 0.25) + 
        scale_color_manual(values = c("black", "red")) + scale_fill_manual(values = c("black", 
        "red")) + scale_linetype_manual(values = c("solid", "solid")) + 
        theme_bw() + theme(legend.title = element_blank(), legend.position = "none", 
        plot.title = element_text(size = 10), text = element_text(size = 10), 
        panel.border = element_blank(), panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank(), axis.line = element_line(colour = "black")) + 
        ggtitle(toupper(str_replace_all(str_replace_all(files[n], 
            ".csv", ""), "_", " "))) + xlab("") + ylab("")
    theme()
    pvalue = round(pvalue, digits = 3)
    pvalue = subset(pvalue, pvalue$group <= 0.01)
    if (empty(pvalue) == FALSE)
    {
        for (z in 1:length(row.names(pvalue)))
        {
            min = as.numeric(row.names(pvalue))[z]
            max = min + 1
            lower = min(melted$lower)
            upper = max(melted$upper)
            p = p + annotate("rect", xmin = min, xmax = max, 
                ymin = lower, ymax = upper, alpha = 0.25)
        }
    }
    nam = paste("p.", n, sep = "")
    assign(nam, p)
    pdf(paste(projDir, "afq-plot/group/", str_replace_all(files[n], 
        ".csv", ""), "_", metric, ".pdf", sep = ""), width = 5, 
        height = 3)
    print(p + xlab("Location") + ylab(paste("Mean ", toupper(metric), 
        "")))
    dev.off()
}
p.1 = p.1 + ylab(paste("Mean ", toupper(metric), "")) + ggtitle("Callosum Forceps\nMajor")
p.2 = p.2 + ggtitle("Callosum Forceps\nMinor")
p.3 = p.3 + ggtitle("Left Arcuate\n")
p.12 = p.12 + ggtitle("Right Arcuate\n")
p.4 = p.4 + ylab(paste("Mean ", toupper(metric), "")) + ggtitle("Left Cingulum\nCingulate")
p.13 = p.13 + ggtitle("Right Cingulum\nCingulate")
p.5 = p.5 + ggtitle("Left Cingulum\nHippocampus")
p.14 = p.14 + ggtitle("Right Cingulum\nHippocampus")
p.6 = p.6 + ylab(paste("Mean ", toupper(metric), "")) + ggtitle("Left Corticospinal\n")
p.15 = p.15 + ggtitle("Right Corticospinal\n")
p.7 = p.7 + ggtitle("Left IFOF\n")
p.16 = p.16 + ggtitle("Right IFOF\n")
p.8 = p.8 + ylab(paste("Mean ", toupper(metric), "")) + ggtitle("Left ILF\n")
p.17 = p.17 + ggtitle("Right ILF\n")
p.9 = p.9 + ggtitle("Left SLF\n")
p.18 = p.18 + ggtitle("Right SLF\n")
p.10 = p.10 + ylab(paste("Mean ", toupper(metric), "")) + xlab("Location") + 
    ggtitle("Left Thalamic\nRadiation")
p.19 = p.19 + xlab("Location") + ggtitle("Right Thalamic\nRadiation")
p.11 = p.11 + xlab("Location") + ggtitle("Left Uncinate\n")
p.20 = p.20 + xlab("Location") + ggtitle("Right Uncinate\n")
p1 = plot_grid(p.1, p.2, p.3, p.12, p.4, p.13, p.5, p.14, p.6, 
    p.15, p.7, p.16, p.8, p.17, p.9, p.18, p.10, p.19, p.11, 
    p.20, align = "vh", hjust = -1, nrow = 5)
legend_b <- get_legend(p.1 + theme(legend.position = "bottom"))
p2 = plot_grid(p1, legend_b, ncol = 1, rel_heights = c(1, 0.1))
title = ggdraw() + draw_label("Automated Fiber Quantification (AFQ)", 
    fontface = "bold")
p3 = plot_grid(title, p2, ncol = 1, rel_heights = c(0.1, 1))
p4 = add_sub(p3, "Shaded sections equal group differences, p (unadj) < 0.01")
p5 = ggdraw(p4)
ggsave(paste(projDir, "afq-plot/group_", metric, ".pdf", sep = ""), 
    p5, device = "pdf", units = c("in"), width = 8, height = 10.5, 
    dpi = 300)
```

## Submit 

Run R script:

```
source("/Volumes/wasatch/data/scripts/R21/AFQ/plot-afq-group.R")
```