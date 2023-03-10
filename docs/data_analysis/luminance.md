---
layout: page
title: Data analysis with Luminance
permalink: /data_analysis/luminance/
parent: Data Analysis for CANDEL
nav_order: 5
---




## General notes on luminance

In `luminance` each plotting function has a corresponding data gathering function that fetches the data from CANDEL. For instance there is a `plot_cell_pop_timeseries` function and a corresponding `get_cell_pop_timeseries` one. If you call the *plot* function without passing any data as input, the corresponding *get* function will be called automatically.

Alternatively you can call the *get* function yourself, and pass the resulting data to the *plot* function. This is useful in cases where you need to do some modifications to the data, that cannot be done from the plot function itself. For instance if you wanted to rename some timepoints, you can get the data, modify it manuall in R, and then pass the results to the plot function

## Plotting samples inventory


```R
wick::set_dbname("pici0002-merge-51")

luminance::plot_all_samples("pici0002")
```

![samples_inventory](/assets/samples_inventory.png)


## Plotting cell populations over time


### Setup

This code defines some variables that are going to be used throughout the following examples. For simplicity we will only plot a limited number of populations, timepoints and treatment arms. Note that all of these sections are completely optional


```R
library(luminance)

set_dbname("pici0002-merge-51") 

sel.timepoints <- c("C1D1", "C1D15", "C2D1", "C3D1", "C4D1", "C5D1", "C6D1")
sel.cell.pops <- c("Plasmablast", "Naive B cells", "Memory B cells")
sel.treatments <- c("B2", "C2")
```

All the plotting functions return a list with two elements:

- `tab`: the table of data that was used for plotting. This is the table that was used as input for ggplot, with all the necessary transformations applied, not simply the data as downloaded from CANDEL
- `plot`: the ggplot object with the actual plot

#### Plotting cell population abundances in individual subjects over time


```R
plot_cell_pop_timeseries(dataset.name = "pici0002", 
    measurement.type = "percent-of-leukocytes", 
    assay.name = "CyTOF",
    measurement.set.name = "Spitzer gated", 
    sel.treatments = sel.treatments, 
    sel.timepoints = sel.timepoints,
    sel.cell.pops = sel.cell.pops)
```

![subj_over_time](/assets/subj_over_time.png)

#### Normalizing by baseline

```R
plot_cell_pop_timeseries(dataset.name = "pici0002", 
    measurement.type = "percent-of-leukocytes", 
    assay.name = "CyTOF",
    measurement.set.name = "Spitzer gated",
    norm.timepoint = "C1D1", 
    norm.zero.value = 0.0001,
    sel.treatments = sel.treatments, 
    sel.timepoints = sel.timepoints,
    sel.cell.pops = sel.cell.pops)
```

![subj_over_time_normalized](/assets/subj_over_time_normalized.png)

#### Using a log scale for the y axis

```R
plot_cell_pop_timeseries(dataset.name = "pici0002", 
    measurement.type = "percent-of-leukocytes", 
    assay.name = "CyTOF",
    measurement.set.name = "Spitzer gated",
    norm.timepoint = "C1D1", 
    norm.zero.value = 0.0001,
    sel.treatments = sel.treatments, 
    sel.timepoints = sel.timepoints,
    sel.cell.pops = sel.cell.pops,
    horiz.unity.line = TRUE,
    y.scale = "log")
```

![subj_over_time_normalized_log](/assets/subj_over_time_normalized_log.png)

#### Color by individual subject

```R
plot_cell_pop_timeseries(dataset.name = "pici0002", 
    measurement.type = "percent-of-leukocytes", 
    assay.name = "CyTOF",
    measurement.set.name = "Spitzer gated",
    norm.timepoint = "C1D1", 
    norm.zero.value = 0.0001,
    color.by = "subject.id",
    sel.treatments = sel.treatments, 
    sel.timepoints = sel.timepoints,
    sel.cell.pops = sel.cell.pops)
```

![subj_over_time_normalized_bysubj](/assets/subj_over_time_normalized_bysubj.png)

#### Custom subject labels (e.g. Responder / Non Responder)

```R
subj.labels <- list('Response' = c(
    '840-100100-003' = 'R',
    '840-100200-002' = 'R',
    '840-100100-002' = 'NR',
    '840-100100-006' = 'NR',
    '840-100400-003' = 'R',
    '840-100100-008' = 'R',
    '840-100200-004' = 'R',
    '840-100300-003' = 'R')
 )

plot_cell_pop_timeseries(dataset.name = "pici0002", 
    measurement.type = "percent-of-leukocytes", 
    assay.name = "CyTOF",
    measurement.set.name = "Spitzer gated",
    norm.timepoint = "C1D1", 
    norm.zero.value = 0.0001,
    color.by = "custom",
    custom.subj.label = subj.labels,
    sel.treatments = sel.treatments, 
    sel.timepoints = sel.timepoints,
    sel.cell.pops = sel.cell.pops)
```

![subj_over_time_normalized_custom](/assets/subj_over_time_normalized_custom.png)


#### Average by custom subject labels (or any other variable observations have been grouped on)

```R
subj.labels <- list('Response' = c(
    '840-100100-003' = 'R',
    '840-100200-002' = 'R',
    '840-100100-002' = 'NR',
    '840-100100-006' = 'NR',
    '840-100400-003' = 'R',
    '840-100100-008' = 'R',
    '840-100200-004' = 'R',
    '840-100300-003' = 'R')
 )

plot_cell_pop_timeseries(dataset.name = "pici0002", 
    measurement.type = "percent-of-leukocytes", 
    assay.name = "CyTOF",
    measurement.set.name = "Spitzer gated",
    norm.timepoint = "C1D1", 
    norm.zero.value = 0.0001,
    color.by = "custom",
    group.by.color.method = "median",
    custom.subj.label = subj.labels,
    sel.treatments = sel.treatments, 
    sel.timepoints = sel.timepoints,
    sel.cell.pops = sel.cell.pops)
```

![subj_over_time_normalized_custom_averaged](/assets/subj_over_time_normalized_custom_averaged.png)

#### Display confidence intervals using ribbons

```R
subj.labels <- list('Response' = c(
    '840-100100-003' = 'R',
    '840-100200-002' = 'R',
    '840-100100-002' = 'NR',
    '840-100100-006' = 'NR',
    '840-100400-003' = 'R',
    '840-100100-008' = 'R',
    '840-100200-004' = 'R',
    '840-100300-003' = 'R')
 )

plot_cell_pop_timeseries(dataset.name = "pici0002", 
    measurement.type = "percent-of-leukocytes", 
    assay.name = "CyTOF",
    measurement.set.name = "Spitzer gated",
    norm.timepoint = "C1D1", 
    norm.zero.value = 0.0001,
    color.by = "custom",
    group.by.color.method = "median",
    confidence.interval.style = "ribbon",
    custom.subj.label = subj.labels,
    sel.treatments = sel.treatments, 
    sel.timepoints = sel.timepoints,
    sel.cell.pops = sel.cell.pops)
```

![subj_over_time_normalized_custom_averaged_ribbon](/assets/subj_over_time_normalized_custom_averaged_ribbon.png)

#### Boxplots

```R

subj.labels <- list('Response' = c(
    '840-100100-003' = 'R',
    '840-100200-002' = 'R',
    '840-100100-002' = 'NR',
    '840-100100-006' = 'NR',
    '840-100400-003' = 'R',
    '840-100100-008' = 'R',
    '840-100200-004' = 'R',
    '840-100300-003' = 'R')
 )

plot_cell_pop_timeseries(dataset.name = "pici0002", 
    measurement.type = "percent-of-leukocytes", 
    assay.name = "CyTOF",
    measurement.set.name = "Spitzer gated",
    norm.timepoint = "C1D1", 
    norm.zero.value = 0.0001,
    color.by = "custom",
    plot.type = "boxplot",
    custom.subj.label = subj.labels,
    sel.treatments = sel.treatments, 
    sel.timepoints = sel.timepoints,
    sel.cell.pops = sel.cell.pops)

```

![subj_over_time_normalized_boxplot](/assets/subj_over_time_normalized_boxplot.png)

#### Plotting cell populations that map to a specific Cell Ontology cell type

```R
plot_cell_pop_timeseries(dataset.name = "pici0002", 
    measurement.type = "percent-of-leukocytes", 
    assay.name = "CyTOF",
    measurement.set.name = "Spitzer gated", 
    sel.treatments = sel.treatments, 
    sel.timepoints = sel.timepoints,
    sel.cell.pops = get_cell_populations_by_co(
        dataset.name = "pici0002",
        assay.name = "CyTOF",
        measurement.set.name = "Spitzer gated",
        co.cell.types = "central memory CD8-positive, alpha-beta T cell"
    ))
```

![subj_over_time_cell_ontology](/assets/subj_over_time_cell_ontology.png)

#### Plotting marker expression over time in each cell population for individual subjects

```R

luminance::plot_cell_pop_timeseries(
    dataset.name = "pici0002", 
    measurement.type = "median-channel-value", 
    assay.name = "CyTOF",
    measurement.set.name = "Bendall gated",
    timeout = 120000,
    sel.cell.pops = "Plasmablast",
    sel.epitopes =  c("CD20", "CD40L", "CR2", "SDC1"),
    color.by = "subject",
    value.fun = luminance::transform_cytof # Use transform_flow for flow data
)

```
![markers_over_time_by_subj](/assets/markers_over_time_by_subj.png)


## Plotting individual timepoints

### Plot cell populations for an individual timepoint

By default this will color by `subject.id`. 
Optional arguments included here are `sel.treatments` and `sel.cell.pops` to filter the 
plots for simplicity of the example.

```R
wick::set_dbname("pici0002-ph2-5") 
sel.cell.pops <- c("Plasmablast", "Naive B cells", "Memory B cells")
sel.treatments <- c("B2", "C2")
luminance::plot_cell_pop_timepoint(dataset.name = "pici0002", 
                                   measurement.type = "percent-of-leukocytes", 
                                   assay.name = "CyTOF",
                                   measurement.set.name = "Spitzer gated",
                                   sel.timepoint = "C1D1", 
                                   sel.treatments = sel.treatments, 
                                   sel.cell.pops = sel.cell.pops)
```

![cell_pop_timepoint](/assets/cell_pop_timepoint.png)

### Plot timepoint grouped by color

Add `color.by` to change the default colors from `subject.id` to desired attribute, and 
set `group.by.color` to `TRUE` to separate the colors into groups.

```R
wick::set_dbname("pici0002-ph2-5") 
sel.cell.pops <- c("Plasmablast", "Naive B cells", "Memory B cells")
sel.treatments <- c("B2", "C2")
luminance::plot_cell_pop_timepoint(dataset.name = "pici0002", 
                                   measurement.type = "percent-of-leukocytes", 
                                   assay.name = "CyTOF",
                                   measurement.set.name = "Spitzer gated",
                                   sel.timepoint = "C1D1",
                                   sel.treatments = sel.treatments, 
                                   sel.cell.pops = sel.cell.pops,
                                   color.by = "subject.sex",
                                   group.by.color = TRUE)
```

![cell_pop_timepoint_bysubj](/assets/cell_pop_timepoint_bysex_grouped.png)

### Plot timepoint with multiple measurement sets

```R
luminance::plot_cell_pop_timepoint(dataset.name = "pici0002", 
                                   measurement.type = "percent-of-leukocytes", 
                                   assay.name = "CyTOF",
                                   measurement.set.name = c("Spitzer gated","phase2 gated"),
                                   sel.timepoint = "C1D1", 
                                   sel.treatments = sel.treatments, 
                                   sel.cell.pops = sel.cell.pops,
                                   color.by = "measurement.set.name",
                                   group.by.color = TRUE)
```

![cell_pop_timepoint_bysubj](/assets/cell_pop_timepoint_bymeas_grouped.png)

## Modifying plot theme attributes

Most `luminance` functions return `ggplot2` objects, which can be further modified using standard ggplot2 functions as in the following example

```R
library(ggplot2)

x <- luminance::plot_cell_pop_timeseries(
    # Details omitted
)
p <- x$plot +  theme(strip.text.x = element_text(size = 12))

# Use print to display the plot
print(p)
```




## Plotting gene expression results


Raw counts from RNA-seq or NanoString experiments can be retrieved using
`get_raw_gene_expression()`.  Use `get_dgelist()` to convert these counts
into a `DGEList` object, which is an object type used by the package `edgeR`
(see [https://doi.org/doi:10.18129/B9.bioc.edgeR](https://doi.org/doi:10.18129/B9.bioc.edgeR)).


```R
library(wick)
library(luminance)
set_dbname("test-db-pici-prod")

x <- get_raw_gene_expression("template-dataset", assay.name = "nanostring",
                             measurement.set.name = "baseline",
                             measurement.type = "nanostring-count")

dge <- get_dgelist(x)
```


When using `get_dgelist()`, a grouping variable can be specified, which will
become the "group" column in the sample annotations of the `DGEList` object.
This is the default grouping variable used when filtering genes for minimum
expression ([https://rdrr.io/bioc/edgeR/man/filterByExpr.html](https://rdrr.io/bioc/edgeR/man/filterByExpr.html))
or when performing differential expression comparisons
([https://rdrr.io/bioc/edgeR/man/estimateDisp.html](https://rdrr.io/bioc/edgeR/man/estimateDisp.html)).

```R
# Create fake treatment and response annotations for this example dataset
dge$samples$treatment.name <- factor(c(rep("A", 27), rep("B", 27)))
dge$samples$response <- factor(c(rep("CR", 27), rep("PD", 27)), levels = c("PD", "SD", "PR", "CR"))
dge$samples$response[c(2, 15, 20, 38, 42, 50)] <- "SD"
dge$samples$response[c(18, 25, 46:49)] <- "PR"
dge <- get_dgelist(x, sample.annotations = dplyr::select(dge$samples, sample.id, treatment.name, response), grouping.variables = "treatment.name")
```


Select top differentially expressed genes between treatment groups and plot a
heatmap using those genes:

```R
testresults <- differential_expression_test(dge, ~treatment.name)
edgeR::topTags(testresults)
geneset <- rownames(edgeR::topTags(testresults))

plot_expression_heatmap(dge, geneset, annotation.column.names = c("treatment.name", "response"),
                        annotation_colors = list(treatment.name = c(A = "lightblue2", B = "lightgoldenrod2")))
```

![expression_heatmap](/assets/expression_heatmap.png)



Calculate summary scores for pre-defined gene signatures and plot a heatmap of
the signature scores:

```R
plot_expression_signature_heatmap(dge,
                                  geneset.names = c("msigdb-hallmark-ifna-sig",
                                                    "msigdb-hallmark-ifng-sig",
                                                    "msigdb-hallmark-tgfb-sig",
                                                    "msigdb-nk-cells",
                                                    "msigdb-wnt-sig"),
                                  sel.subjects = NULL,
                                  sel.timepoints = NULL,
                                  score.method = "count.geo.mean",
                                  scale = TRUE,
                                  annotation.column.names = c("treatment.name", "response"),
                                  annotation_colors = list(treatment.name = c(A = "lightblue2", B = "lightgoldenrod2")))
```

![expression_signature_heatmap](/assets/expression_signature_heatmap.png)

Principal components plot using gene expression:
```R
plot_pca_biplot(dge,
                colby = "treatment.name",
                lab = dge$samples$subject.id)
```

![expression_pca](/assets/expression_pca.png)

Volcano plot for differential expression test results:

```R
p <- plot_volcano(testresults)
p$plot
```

![expression_volcano](/assets/expression_volcano.png)

Gene set enrichment plots:

```R
gsea.results <- gsea_table(testresults)
plot_pathway_enrichment_scores(gsea.results)
plot_enrichment("HALLMARK_APOPTOSIS", testresults)
```

![expression_enrichment_scores](/assets/expression_enrichment_scores.png)

![expression_enrichment](/assets/expression_enrichment.png)
