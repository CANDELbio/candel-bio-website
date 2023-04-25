---
layout: page
title: Workerbee
permalink: /data_processing/workerbee/
parent: Data Processing
nav_order: 7
---



# R pre-processing
{: .no_toc }

This page contains information about pre-processing different types of data for usage with pret by leveraging the `workerbee` R package

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Available generators

The currently available generators are the following (please refer to the R documentation for detailed information)


|Function                               |Purpose   |
|---------------------------------------|-----------|
|`generate_regimens_therapies`          |generate treatment regimens and therapies starting from a table of subjects information|
|`generate_timepoints`                  |combine timepoint and treatment regimens information to generate all the individual timepoint / treatment regimen combinations|
|`generate_cell_population_measurements`|generate cell population measurements starting from a table of event counts downloaded from CellEngine|
|`generate_cell_cluster_measurements`   |generate cluster measurements starting from the output of the `grappolo` package|
|`generate_clinical_observations`|generate clinical observations by combining clinical data with timepoint information|
|`generate_personalis_data`|generate multiple types of data from Personalis output|



## Timepoints and treatment regimens

This example assumes a simple scenario where there a number of treatment regimens, all with the same schedule of events

Assuming you have a table of subjects indicating the arm (i.e. treatment regimen) for each one (the `TRTACD` column in the example below)

|USUBJID|	SEX|	TRTACD|
|-------|------|----------|
|840-100100-001|	M|	B1|
|840-100100-002|	F|	B2|
|840-100100-003|	F|	B2|
|840-100100-004|	M|	C1|

you can create treatment regimens and therapies as follows

```R
subjects <- read.table("original/subjects.txt", header = T, stringsAsFactors = F)

reg.therapies <- workerbee::generate_regimens_therapies(subjects,
    subject.var = "USUBJID",
    treatment.var = "TRTACD"
)
```

The `reg.therapies` object is a list with two elements
- `therapies` contains the therapy information for each subject (i.e. the assignment of each subj to a treatment arm)
- `regimens` contains the regimen ids

Now create a schedule of events file similar to the following

|CYCLE|	DAY|	NOMINAL|	ORDER|	TYPE|
|-----|----|-----------|---------|------|
|1|	1|	C1D1|	1|	:timepoint.type/on-treatment|
|1|	3|	C1D3|	2|	:timepoint.type/on-treatment|
|1|	4|	C1D4|	3|	:timepoint.type/on-treatment|
|1|	8|	C1D8|	4|	:timepoint.type/on-treatment|

You can then combine the schedule of events with the regimens information as follows

```R

schedule.of.events <- read.table("original/schedule_of_events.txt", header = T, stringsAsFactors = F) 
timepoints <- workerbee::generate_timepoints(schedule.of.events,
    regimens = reg.therapies$regimens$id,
    timepoint.name.var = "NOMINAL"    
)
```

This will generate timepoints by duplicating the schedule of events for each treatment regimens as follows

|regimen| CYCLE| DAY| ORDER|TYPE|id|
|-------|------|----|------|----|--|
|B1|     1|   1|     1| :timepoint.type/on-treatment|    B1/C1D1|
|B2|     1|   1|     1| :timepoint.type/on-treatment|    B2/C1D1|
|C1|     1|   1|     1| :timepoint.type/on-treatment|    C1/C1D1|
|B1|     1|   3|     2| :timepoint.type/on-treatment|    B1/C1D3|
|B2|     1|   3|     2| :timepoint.type/on-treatment|    B2/C1D3|
|C1|     1|   3|     2| :timepoint.type/on-treatment|    C1/C1D3|

Now save everything as follows

```R
write.table(timepoints, "processed/timepoints.txt", sep = "\t", col.names = T, row.names = F, quote = F)
write.table(reg.therapies$therapies, "processed/therapies.txt", sep = "\t", col.names = T, row.names = F, quote = F)
write.table(reg.therapies$regimens, "processed/treatment-regimens.txt", sep = "\t", col.names = T, row.names = F, quote = F)
```

## Sequencing data from Personalis

The `generate_personalis_data` is a high-level entry point that will generate multiple types of data by calling a number of downstream generators. You can control which generators are called with the `generators` argument. Different types of generators assume the presence of files with different names in the `input.dir` folder. You should grab files matching the following regular expressions from the Personalis output folder and copy them to `input.dir`

|generator|required files|
|---------|--------------|
|`gene_expression`|`*tumor_rna_expression_report*`|
|`cna`|`*somatic_dna_gene_cna_report*`|
|`tmb`|`*dna_statistics*`|
|`variants`|`*somatic_dna_small_variant_report_preferred*`|
|`tcr_alpha`|`*rna_tcr_alpha_clone_report*`|
|`tcr_beta`|`*rna_tcr_beta_clone_report*`|

The Personalis data files contain the sample name in the file name. Most likely you will have to supply a mapping between the file names and the actual sample barcodes. 

Assuming you have a table that contains mapping between filenames and barcodes, such as a table downloaded from RawSugar after the matching process, you can create this mapping as follows


```R
name.to.sample <- read.table("filename_to_sample.txt", header = T, stringsAsFactors = F)

samples.map <- as.vector(name.to.sample$sample.barcode)
names(samples.map) <- name.to.sample$filename
```

Alternatively, instead of the full filename you can also specify a sample ID that will be extracted from the filename.

You can then use the `generate_personalis_data` function as follows

```R
wick::set_dbname("your-db-name")
all.genes <- wick::get_all_genes()

workerbee::generate_personalis_data(
    input.dir = "original/personalis",
    output.dir = "processed/personalis"
    assembly = "GRCh38",
    all.genes = all.genes,
    samples.map = samples.map,
    generators = c("cna", "variants", "gene_expression", "tmb", "tcr_alpha", "tcr_beta")
)
```

This function will process your data, save it in the folder specified in the `output.dir` argument, and also output snippets that you can include in your `config.edn` file.

## Flow and CyTOF data

For flow and CyTOF data the default values for the column names arguments in the generators functions described below will work for data that has been downloaded from CellEngine or clustered using `grappolo`, otherwise you will have to provide these arguments yourself (refer to the R documentation)

### Gated data

For gated data you will have to create a table with cell population names, and how they map to standardized cell ontology names, similar to the following (see the [Schema](../candel_database/schema) section for more details on representing cell populations in CANDEL).

|name|	cell.type|	positive.epitopes|
|----|-----------|-------------------|
|Plasmablast|	plasmablast|	NA|
|NK cells|	natural killer cell|	NA|
|CD4 T Cells|	CD4-positive, alpha-beta T cell|	NA|
|CD4 T Cells > CD25+|	CD4-positive, alpha-beta T cell|	IL2RA|

Remember that when specifying cell populations in your `config.edn` file you will also need to indicate that they do not come from clustering, like in the following example:

```clojure
:cell-populations [{:pret/input-file "processed/cell_populations.txt"
                    :pret/na "NA"
                    :name "name"
                    :positive-markers "positive.epitopes"
                    :cell-type "cell.type"
                    :pret/constants {:from-clustering false}}]
```

#### Cell populations abundance from gated data

Assuming you have a table of event counts for each sample and population you can use the `generate_cell_population_measurements` function. You will have to specify which population you want to use for normalization of event counts(e.g. `CD45+` in the following example). Note that the function will also output a snippet that you can include in your `config.edn` file

```R
tab <- read.table("original/event_count.txt", header = T, sep = "\t", check.names = F)
# This is optional, to limit the results to a pre-specified set of cell populations
cell.populations.mapping <- read.table("original/cell_populations_mapping.txt", header = T, 
                                        sep = "\t", check.names = F)

workerbee::generate_cell_population_abundance_measurements(
    tab,
    norm.population = "CD45+",
    out.file = "processed/event_count.txt",
    all.cell.populations = cell.populations.mapping$name
)
```

#### Cell populations abundance as percent of parent

If you want to import percent of parent data you can do so by starting from a table that contains percent of parent data already (as opposed to event counts), and skipping any normalization, as follows (refer to the documentation for more details)

```R
tab <- read.table("original/percent_parent.txt", header = T, sep = "\t", check.names = F)
# This is optional, to limit the results to a pre-specified set of cell populations
cell.populations.mapping <- read.table("original/cell_populations_mapping.txt", header = T, 
                                        sep = "\t", check.names = F)

workerbee::generate_cell_population_abundance_measurements(
    tab,
    out.file = "processed/percent_parent.txt",
    measurement.type = "percent-of-parent",
    divide.by.100 = TRUE, # Percentages in CANDEL are represented in [0, 1]
    all.cell.populations = cell.populations.mapping$name
)
```

#### Marker expression measurements from gated data

You will have to provide a table mapping reagent names in the data to epitopes in CANDEL. The easiest way to do this is by creating a TSV file with this information, similar to the following

|reagent|	epitope|
|-------|----------|
|113In_CD40|	TNR5|
|115In_CD20|	CD20|
|141Pr_CD3|	CD3E|
|142Nd_CD19|	CD19|
|143Nd_CD117__c-kit_|	KIT|

and pass it as input to the generator as exemplified below

```R
tab <- read.table("original/marker_expression.txt", header = T, sep = "\t", check.names = F)
reagents.mapping <- read.table("original/reagents_mapping.txt", header = T, sep = "\t", check.names = F)

# This is optional, to limit the results to a pre-specified set of cell populations
cell.populations.mapping <- read.table("original/cell_populations_mapping.txt", header = T,
                                        sep = "\t", check.names = F)

# This is also optional, to limit the results to epitopes that are present in the CANDEL database
wick::set_dbname("YOUR-DATABASE")
all.epitopes <- wick::get_all_epitopes()

generate_cell_population_epitope_measurements(
    tab, 
    reagents.mapping = reagents.mapping, 
    all.epitopes = all.epitopes, 
    all.cell.populations = cell.populations.mapping$name,
    out.file = "processed/marker_expression.txt"
)
```

### Clustered data

In the case of clustered data both cell abundance and marker expression measurements will be generated with a single function call. Similarly to the above, it is recommended to provide in input a table specifying mapping between reagents and epitopes in the database

```R
tab <- read.table("original/data.clustered.txt", header = T, sep = "\t", check.names = F)
reagents.mapping <- read.table("original/cytof_reagents_mapping.txt", header = T, sep = "\t")

# This is optional, to limit the results to epitopes that are present in the CANDEL database
wick::set_dbname("YOUR-DATABASE")
all.epitopes <- wick::get_all_epitopes()

generate_cell_cluster_measurements(
    tab, 
    reagents.mapping = reagents.mapping, 
    all.epitopes = all.epitopes,
    out.file = "processed/clusters_data.txt"
)
```


## Mapping gene symbols

If you your data contains gene symbols, wick contains a function to map strings to HGNC symbols. You can perform the mapping as follows:

```R
library(wick)

set_dbname("YOUR-DATBASE")

all.genes <- get_all_genes()

mapping.res <- map_gene_symbols(all.genes, vector.of.symbols.to.map)
```

Note that the function will return `NA` for symbols that cannot be mapped