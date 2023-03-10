---
layout: page
title: Query with Wick
permalink: /data_analysis/wick/
parent: Data Analysis for CANDEL
nav_order: 4
---


## Accessing CANDEL from R using wick

`wick` is an R package that enables you to connect to CANDEL and issue queries against the database using Datalog. `wick` also has a selection of pre-built query functions.

To get started using `wick`, first install the package or ensure that it is installed with `packageVersion('wick')`. Once the package is installed, log in to CANDEL:

        wick::login(email = 'your-email@parkerici.org')

The command above will open a browser window and prompt you to log in with a Google-backed account. The login process works by issuing a token from Google, which is stored locally. Note that these tokens do expire after some time (weeks), so when you get a notice about an expired token, you will need to re-run login with the argument `force = TRUE`.

After you're logged in, set the database name to the database you wish to use. To query the CANDEL master database run:

        wick::set_dbname('candel-master')

If you have spun up a branch database with the `pret` command `request-db`, you can use the branch database name instead, ie `wick::set_dbname('candel-db-123')`.

Now that you are logged in and have your database name set, you can proceed to use any of the `wick` pre-built query functions or construct your own queries. 

## Pre-configured queries

`wick` contains several pre-configured queries that are described below. Please refer to the R documentation for the invidiual functions for details about parameters and return values.


### Getting information about reference entities

When preparing data for import you often need to get a list of all the reference entities in the database in order to make sure that your data does not include any invalid gene names, protein names etc. The following functions enable you to get this information


```
get_all_cell_types
get_all_cnv
get_all_ctcae_adverse_events
get_all_drugs
get_all_epitopes
get_all_gdc_anatomic_sites
get_all_gene_products
get_all_gene_symbols
get_all_genes
get_all_genes_with_gc
get_all_genomic_coordinates
get_all_meddra_diseases
get_all_proteins
get_all_variants
```


### Getting complete information for a dataset

In case you want to get complete information about subjects, timepoints, measurements etc. for a specific dataset, you can use the following functions:


```
get_all_cell_populations
get_all_clinical_observations
get_all_datasets
get_all_measurements
get_all_meas_set_samples
get_all_samples
get_all_subjects
get_all_timepoints
get_common_timepoints
get_dataset_summary
get_unified_timeline
```

### Helper functions

`wick` also has a few helper functions to perform additional queries and merge them into the results of other queries or perform other functions.

```
add_sample_context
add_variants_context
map_gene_symbols
group_by_assay_meas_set
```


## Datalog and creating your own queries

### Datalog

Datalog is the query language that is used to query CANDEL. Before you proceed you should familiarize yourself with how Datalog works. [This](http://www.learndatalogtoday.org/) is an excellent tutorial

### Custom CANDEL queries with wick

The `wick` package enables you to create arbitrary Datalog queries and issue them to the CANDEL database. Please refer to the [README](https://github.com/ParkerICI/wick) for more information on how to translate Datalog queries into R.

### Pull expressions 

Pull expressions are a mechanisms to specify which information you want returned about entities. They are described in detail in the Datomic documentation [here](https://docs.datomic.com/on-prem/pull.html). Please refer to the wick [README](https://github.com/ParkerICI/wick) for more information on how to convert pull expressions into R