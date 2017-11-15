---
title: Graphs - AFQ-CC Plots
toc: true
weight: 12
---

## R Script

For group plot, create an R script:

```bash
vi /Volumes/wasatch/data/scripts/R21/AFQ/plot-cc-group.R
```

Copy and paste:

```r
metric <- readline(prompt = "What DTI metric do you want to analyze (fa, rd, md, ad): ")
projDir = "/Volumes/wasatch/data/stats/R21/AFQ/"
files = list.files(paste(projDir, "cc/", sep = ""), pattern = ".csv", 
    full.names = FALSE)
for (n in 1:length(files))
{
    mydata = read.table(paste(projDir, "cc/", files[n], sep = ""), 
        sep = ",", header = TRUE)
    pvalue = read.table(paste(projDir, "cc-pvalue/", metric, 
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
    pdf(paste(projDir, "cc-plot/group/", str_replace_all(files[n], 
        ".csv", ""), "_", metric, ".pdf", sep = ""), width = 5, 
        height = 3)
    print(p + xlab("Location") + ylab(paste("Mean ", toupper(metric), 
        "")))
    dev.off()
}
p.4 = p.4 + ylab(paste("Mean ", toupper(metric), "")) + ggtitle("CC Orbital Frontal")
p.6 = p.6 + ggtitle("CC Superior Frontal")
p.1 = p.1 + ylab(paste("Mean ", toupper(metric), "")) + ggtitle("CC Anterior Frontal")
p.2 = p.2 + ggtitle("CC Motor")
p.7 = p.7 + ylab(paste("Mean ", toupper(metric), "")) + ggtitle("CC Superior Parietal")
p.5 = p.5 + ggtitle("CC Posterior Parietal")
p.3 = p.3 + xlab("Location") + ylab(paste("Mean ", toupper(metric), 
    "")) + ggtitle("CC Occipital")
p.8 = p.8 + xlab("Location") + ggtitle("CC Temporal")

p1 = plot_grid(p.4, p.6, p.1, p.2, p.7, p.5, p.3, p.8, align = "vh", 
    hjust = -1, nrow = 4)
legend_b <- get_legend(p.1 + theme(legend.position = "bottom"))
p2 = plot_grid(p1, legend_b, ncol = 1, rel_heights = c(1, 0.1))
title = ggdraw() + draw_label("Automated Fiber Quantification (AFQ)", 
    fontface = "bold")
p3 = plot_grid(title, p2, ncol = 1, rel_heights = c(0.1, 1))
p4 = add_sub(p3, "Shaded sections equal group differences, p (unadj) < 0.01")
p5 = ggdraw(p4)
ggsave(paste(projDir, "cc-plot/group_", metric, ".pdf", sep = ""), 
    p5, device = "pdf", units = c("in"), width = 8, height = 10.5, 
    dpi = 300)
```

## Submit

Run R script:

```
source("/Volumes/wasatch/data/scripts/R21/AFQ/plot-cc-group.R")
```