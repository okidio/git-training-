---
title: "Advanced visualizations"
author: "Meeta Mistry, Mary Piper, Radhika Khetani"
date: "October 19th, 2017"
---

Approximate time: 75 minutes

## Learning Objectives 

* Exploring expression data using data visualization
* Using volcano plots to evaluate relationships between DEG statistics
* Plotting expression of significant genes using heatmaps

## Visualizing the results

When we are working with large amounts of data it can be useful to display that information graphically to gain more insight. During this lesson, we will get you started with some basic and more advanced plots commonly used when exploring differential gene expression data, however, many of these plots can be helpful in visualizing other types of data as well.

Let's start by loading a few libraries (if not already loaded):

```r
# load libraries
library(reshape)
library(ggplot2)
library(ggrepel)
library(DEGreport)
library(RColorBrewer)
library(DESeq2)
library(pheatmap)
```
We will be working with three different files generally created or obtained during a differential expression analysis:

- Metadata for our samples: `mov10_meta`
- Expression data for every gene in each of our samples: `normalized_counts`
- Differential expression results: `res_tableOE`

Let's take a look at each of these files before we start plotting.

```r
# Create tibbles including row names
mov10_meta <- mov10_meta %>% 
        rownames_to_column() %>% 
        as_tibble() %>%
        rename(samplename = rowname))
        
normalized_counts <- normalized_counts %>%
        rownames_to_column() %>% 
        as_tibble() %>%
        rename(gene = rowname)

res_tableOE <- res_tableOE %>% 
        rownames_to_column() %>% 
        as_tibble() %>%
        rename(gene = rowname)
```

### Plotting signicant DE genes

One way to visualize results would be to simply plot the expression data for a handful of genes. We could do that by picking out specific genes of interest or selecting a range of genes:

#### Using `ggplot2` to plot one or more genes (e.g. top 20)

Often it is helpful to check the expression of multiple genes of interest at the same time. This often first requires some data wrangling.

We are going to plot the normalized count values for the **top 20 differentially expressed genes (by padj values)**. 

To do this, we first need to determine the gene names of our top 20 genes by ordering our results and extracting the top 20 genes (by padj values):

```r
## Order results by padj values
top20_sigOE_genes <- res_tableOE %>% 
        arrange(padj) %>% #Arrange rows by padj values
        pull(gene) %>% #Extract character vector of ordered genes
        .[1:20] #Extract the first 20 genes

```

Then, we can extract the normalized count values for these top 20 genes:

```r
## normalized counts for top 20 significant genes
top20_sigOE_norm <- normalized_counts %>%
        filter(gene %in% top20_sigOE_genes)
```

Now that we have the normalized counts for each of the top 20 genes for all 8 samples, to plot using `ggplot()`, we need to gather the counts for all samples into a single column to allow us to give ggplot the one column with the values we want it to plot.

The `gather()` function in the **tidyr** R package will perform this operation and will output the normalized counts for all genes for *Mov10_oe_1* listed in the first 20 rows, followed by the normalized counts for *Mov10_oe_2* in the next 20 rows, so on and so forth.

<img src="../img/melt_wide_to_long_format.png" width="800">

```r
# Gathering the columns to have normalized counts to a single column
gathered_top20_sigOE <- normalized_counts %>%
        gather(colnames(normalized_counts)[2:9],
               key =  "samplename",
               value = "normalized_counts")

## check the column header in the "gathered" data frame
View(gathered_top20_sigOE)
```

Now, if we want our counts colored by sample group, then we need to combine the metadata information with the melted normalized counts data into the same data frame for input to `ggplot()`:

```r
gathered_top20_sigOE <- inner_join(mov10_meta, gathered_top20_sigOE)
```

The `inner_join()` will merge 2 data frames with respect to the "samplename" column, i.e. a column with the same column name in both data frames.

Now that we have a data frame in a format that can be utilised by ggplot easily, let's plot! 

```r
## plot using ggplot2
ggplot(gathered_top20_sigOE) +
        geom_point(aes(x = gene, y = normalized_counts, color = sampletype), position=position_jitter(w=0.1,h=0)) +
        scale_y_log10() +
        xlab("Genes") +
        ylab("Normalized Counts") +
        ggtitle("Top 20 Significant DE Genes") +
        theme_bw() +
	theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
	theme(plot.title=element_text(hjust=0.5))
```

<img src="../img/sig_genes_melt.png" width="600">

If we only wanted to look at a single gene, we could extract that gene for plotting with ggplot. **Within `ggplot()` we can use the `geom_text_repel()` from the 'ggrepel' R package to label our individual points on the plot.** The vignette for 'ggrepel' package is quite nice and details the different options available in the ggrepel package is [available](https://cran.r-project.org/web/packages/ggrepel/vignettes/ggrepel.html).

```r
## plot using ggplot2 for a single gene
mov10 <- filter(gathered_top20_sigOE, gene == "MOV10")

ggplot(mov10, aes(x = sampletype, y=normalized_counts,  color = sampletype)) +
        geom_point(position=position_jitter(w=0.1,h=0)) +
	geom_text_repel(aes(label = samplename)) +
        scale_y_log10() +
        xlab("Samples") +
        ylab("Normalized Counts") +
        ggtitle("MOV10") +
        theme_bw() +
        theme(plot.title=element_text(hjust=0.5))
```

<img src="../img/plotCounts_ggrepel.png" width="600">

### Volcano plot

The above plot would be great to look at the expression levels of a good number of genes, but for more of a global view there are other plots we can draw. A commonly used one is a volcano plot; in which you have the log transformed adjusted p-values plotted on the y-axis and log2 fold change values on the x-axis. 

To generate a volcano plot, we first need to have a column in our results data indicating whether or not the gene is considered differentially expressed based on p-adjusted values.

```r
## Obtain logical vector regarding whether padj values are less than 0.05
res_tableOE <- res_tableOE %>% mutate(threshold_OE = padj < 0.05)
```

Now we can start plotting. The `geom_point` object is most applicable, as this is essentially a scatter plot:

```r
## Volcano plot
ggplot(res_tableOE) +
        geom_point(aes(x=log2FoldChange, y=-log10(padj), colour=threshold_OE)) +
        ggtitle("Mov10 overexpression") +
        xlab("log2 fold change") + 
        ylab("-log10 adjusted p-value") +
        #scale_y_continuous(limits = c(0,50)) +
        theme(legend.position = "none",
              plot.title = element_text(size = rel(1.5), hjust = 0.5),
              axis.title = element_text(size = rel(1.25)))  
```

<img src="../img/volcanoplot-1.png" width=500> 

This is a great way to get an overall picture of what is going on, but what if we also wanted to know where the top 10 genes (lowest padj) in our DE list are located on this plot? We could label those dots with the gene name on the Volcano plot using `geom_text_repel()`.

To make this work we have to take the following 3 steps:
(Step 1) Create a new data frame sorted or ordered by padj
(Step 2) Indicate in the data frame which genes we want to label by adding a logical vector to it, wherein "TRUE" = genes we want to label.
 
```r
## Create a column to indicate which genes to label
res_tableOE <- res_tableOE %>% arrange(padj) %>% mutate(genelabels = "")

res_tableOE$genelabels[1:10] <- res_tableOE$gene[1:10]

View(res_tableOE)
```

(Step 3) Finally, we need to add the `geom_text_repel()` layer to the ggplot code we used before, and let it know which genes we want labelled. 

```r
ggplot(res_tableOE) +
        geom_point(aes(x = log2FoldChange, 
                       y = -log10(padj),
                       colour = threshold_OE)) +
        geom_text_repel(aes(x = log2FoldChange, 
                            y = -log10(padj), 
                            label = genelabels)) +
        ggtitle("Mov10 overexpression") +
        xlab("log2 fold change") + 
        ylab("-log10 adjusted p-value") +
        theme(legend.position = "none",
              plot.title = element_text(size = rel(1.5), hjust = 0.5),
              axis.title = element_text(size = rel(1.25))) 
```

<img src="../img/volcanoplot-2.png" width=500> 


### Heatmap

Alternatively, we could extract only the genes that are identified as significant and the plot the expression of those genes using a heatmap:

```r
## Extract significant genes
sigOE <- res_tableOE %>% filter(padj < 0.05) %>% pull(gene)

### Extract normalized expression for significant genes and set the gene column to row names
norm_OEsig <- normalized_counts %>% filter(gene %in% sigOE) %>% column_to_rownames(var = "gene")
```

Now let's draw the heatmap using `pheatmap`:

```r
### Annotate our heatmap (optional)
annotation <- mov10_meta %>% select(samplename, sampletype) %>% column_to_rownames(var = "samplename")

### Set a color palette
heat_colors <- brewer.pal(6, "YlOrRd")

### Run pheatmap
pheatmap(as.data.frame(norm_OEsig), 
         color = heat_colors, 
         cluster_rows = T, 
         show_rownames=F,
         annotation= as.data.frame(annotation), 
         border_color=NA, 
         fontsize = 10, 
         scale="row", 
         fontsize_row = 10, 
         height=20)
```
         
![sigOE_heatmap](../img/sigOE_heatmap.png)       

> *NOTE:* There are several additional arguments we have included in the function for aesthetics. One important one is `scale="row"`, in which Z-scores are plotted, rather than the actual normalized count value. 
>
> Z-scores are computed on a gene-by-gene basis by subtracting the mean and then dividing by the standard deviation. The Z-scores are computed **after the clustering**, so that it only affects the graphical aesthetics and the color visualization is improved.

### MA Plot

Another plot often useful to exploring our results is the MA plot. The MA plot shows the mean of the normalized counts versus the log2 foldchanges for all genes tested. The genes that are significantly DE are colored to be easily identified. The DESeq2 package also offers a simple function to generate this plot:

```r
ma <- res_tableOE %>% select(c("baseMean", "log2FoldChange", "threshold_OE"))

plotMA(ma, ylim=c(-2,2))
```
<img src="../img/MA_plot.png" width="600">

We would expect to see significant genes across the range of expression levels.

***

***NOTE:** If using the DESeq2 tool for differential expression analysis, the package 'DEGreport' can use the DESeq2 results output to make the top20 genes and the volcano plots generated above by writing a few lines of simple code. While you can customize the plots above, you may be interested in using the easier code. Below are examples of the code to create these plots:*

>```r
>DEGreport::degPlot(dds = dds, res = res, n=20, xs="type", group = "condition") # dds object is output from DESeq2
>
>DEGreport::degVolcano(
>    as.data.frame(res[,c("log2FoldChange","padj")]), # table - 2 columns
>    plot_text=as.data.frame(res[1:10,c("log2FoldChange","padj","id")])) # table to add names
>    
># Available in the newer version for R 3.4
>DEGreport::degPlotWide(dds = dds, genes = row.names(res)[1:5], group = "condition")
>```
***

*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*

* *Materials and hands-on activities were adapted from [RNA-seq workflow](http://www.bioconductor.org/help/workflows/rnaseqGene/#de) on the Bioconductor website*
