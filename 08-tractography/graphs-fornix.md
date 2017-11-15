---
title: Graphs - Fornix
toc: true
weight: 5
---

## Along the Track

Copy and paste in R:

```r
mydata = read.table("/Volumes/1/data/stats/R21/fornix/data.csv", 
    sep = ",", header = TRUE)
demogr = read.table("/Volumes/1/data/analyses/R21/demographics/R21_demographics.txt", 
    sep = ",", header = TRUE)
mydata = merge(demogr, mydata, by = 1, all.y = T)
pvalue = read.table("/Volumes/1/data/stats/R21/fornix/pvalue.csv", 
    sep = ",", header = TRUE)
for (side in c("left", "right"))
{
    for (scalar in c("fa", "rd", "ad", "md"))
    {
        temp = subset(mydata, mydata$hemisphere == side & mydata$dtiscalar == 
            scalar)
        melted = melt(temp, id = c("studyid", "group", "age", 
            "gender", "hemisphere", "dtiscalar"))
        melted = group.STDERR(value ~ variable * group, melted)
        names(melted) = c("node", "group", "lwr", "mean", "upr")
        p = ggplot() + geom_line(data = melted, aes(x = as.numeric(node), 
            y = mean, colour = group, linetype = group), size = 0.5) + 
            geom_ribbon(show.legend = F, data = melted, aes(x = as.numeric(node), 
                ymin = lwr, ymax = upr, fill = group), alpha = 0.25) + 
            scale_color_manual(values = c("black", "red")) + 
            scale_fill_manual(values = c("black", "red")) + scale_linetype_manual(values = c("solid", 
            "solid")) + theme_bw() + theme(legend.title = element_blank(), 
            plot.title = element_text(size = 10), text = element_text(size = 10), 
            panel.border = element_blank(), panel.grid.major = element_blank(), 
            panel.grid.minor = element_blank(), axis.line = element_line(colour = "black")) + 
            ggtitle("") + xlab("% Distance Along Fiber Tract") + 
            ylab("")
        theme()
        pvalue = round(pvalue, digits = 3)
        sig = subset(pvalue, pvalue[c(paste(side, scalar, sep = "_"))] <= 
            0.01)
        if (empty(sig) == FALSE)
        {
            for (z in 1:length(row.names(sig)))
            {
                min = as.numeric(row.names(sig))[z]
                max = min + 1
                lower = min(melted$lower)
                upper = max(melted$upper)
                p = p + annotate("rect", xmin = min, xmax = max, 
                  ymin = lower, ymax = upper, alpha = 0.25)
            }
        }
        nam = paste(side, scalar, sep = "_")
        assign(nam, p)
    }
}
p1 = plot_grid(left_fa + theme(legend.position = "none"), right_fa + 
    theme(legend.position = "none"), left_rd + theme(legend.position = "none"), 
    right_rd + theme(legend.position = "none"), left_ad + theme(legend.position = "none"), 
    right_ad + theme(legend.position = "none"), left_md + theme(legend.position = "none"), 
    right_md + theme(legend.position = "none"), align = "vh", 
    labels = c("Left FA", "Right FA", "Left RD", "Right RD", 
        "Left AD", "Right AD", "Left MD", "Right MD"), hjust = -1, 
    nrow = 4)
legend_b <- get_legend(left_fa + theme(legend.position = "bottom"))
p2 <- plot_grid(p1, legend_b, ncol = 1, rel_heights = c(1, 0.1))
title <- ggdraw() + draw_label("Probabilistic Tractography of the Body of the Fornix", 
    fontface = "bold")
p3 = plot_grid(title, p2, ncol = 1, rel_heights = c(0.1, 1))
p4 = add_sub(p3, "Shaded sections equal group differences, p (unadj) < 0.01")
p5 = ggdraw(p4)
ggsave("/Volumes/1/data/stats/R21/fornix/group.pdf", p5, device = "pdf", 
    units = c("in"), width = 8, height = 10.5, dpi = 300)
```