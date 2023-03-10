---
layout: page
title: CANDEL Schema
permalink: /candel_database/schema/
parent: CANDEL Database
nav_order: 3
---


# The CANDEL Schema
{: .no_toc }

In order to use CANDEL effectively you will have to familiarize yourself with the CANDEL schema. This [schema viewer](), provides a GUI to browse the structure of the schema and the documentation associated with each attribute. A [PDF diagram]() of the schema is also available

The rest of this page provides more details on how certainly critical aspects of the data are modeled. While this stuff is useful to understand the logic behind the schema, it is not necessary to go to this level of depth before starting to use CANDEL

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Entities and attributes

All data in CANDEL relates to the idea of a *dataset*. A dataset is a grouping of data that is logical to the person importing it -- this could possibly be a single clinical trial, a set of results from a published paper. When you import data, you will generally import an entire dataset. 

The first concept to understand in the schema is that all data is captured as *attributes* on *entities*. Entities are roughly "things that exist in the real world" - subjects, samples, genes, proteins, and more complex concepts like an assay run on a set of samples, or a dataset. Attributes are properties of these entities: the name of the gene, the ID of the subject, or the technology the assay used. We refer to attributes on entities with the notation `:entity/attribute`. Importantly, attributes can point to other entities. For example, the sample entity has an attribute `:sample/subject` which points to a subject entity.

It is important to understand that entities come in two varieties - one is *reference entities*, which are independent of any particular dataset and can be referenced by entities in many datasets. These are entities such as gene, protein, cell type, and variant. In the schema viewer linked above, these entities are shown in yellow. Some of these entities, such as genes and proteins, are already present in the database. Some, such as variants, are added to the database with each new import. If your dataset contains variants, you'll need to add your variants to the database before referencing them from your measurements. 

The other type of entities are *entities contained in a dataset*. These non-reference entities can only be referred to from within the dataset. This type of entity, such as subjects, samples, and measurements, will be the main data within your dataset import. In the schema viewer, these entities are shown in blue.


## Identity

First of all you need to understand what is the mechanism for naming *objects*, e.g. samples, subjects, genes and any other element that constitutes your data. Since you will be referring to these objects across multiple files (e.g. you may have a file with samples and another file that contains measurements that have been done on those samples), you need a mechanism to identify them consistently across your dataset. We will call the name of an object its `id`. There are three types of ids that you need to be aware of:

- **globally unique id**: An id that is unique within the entire database, e.g. the HUGO symbol for a gene
- **dataset unique id**: An id that is unique within a dataset but not within the entire database, e.g. a sample id. You can use that id to uniquely identify a sample within your dataset, but you cannot guarantee that that id is unique to your dataset, i.e. there may be another dataset in the datbase that uses the same id to identify a different sample
- **contextual id**: An id that is unique only within the context of its parent containers (and therefore by extension only within your dataset). For instance, if A is a parent of B (A → B), and the id of B is unique only within the children of A.

As detailed in the CANDEL schema documentation, different entity types (e.g. subjects, samples etc.) require different types of id for identification, and it is your responsibilty to provide ids that meet the requirements

## Representing molecular and clinical data

CANDEL uses a similar approach to represent both molecular measurements (e.g. gene expression) and clinical observations (e.g. progression free survival). 

If you think of a spreeadsheet with row names and column names indicating things such as samples and genes respectively, then each cell in the spreadsheet is represented as an individual entity with attributes indicating:

1. its value
2. the connection to the entity representing the corresponding row in the spreadsheet
3. the connection to the entity representing the corresponding column in the spreadsheet

Molecular measurements live in the `measurement` namespace and link together samples (**where** the measurement was made) with biological concepts such as gene proteins etc (**what** was measured).

Clinical measurements live in the `clinical-observation` namespaced and link together subjects (**where** the measurement was made) and timepoints (**when** the measurement was made)

For both measurements and clinical observations, each unit of measure (e.g. gene expression, protein abundance, progression free survival, BMI etc.) is indicated by a separate attribute which indicates the **kind** of measurement (e.g. `:measurement/tpm` or `:clinical-observation/bmi`). Each measurement or clinical observation will have one and only one of these attributes defining its kind.

## Representing antibody based measurements

CANDEL does not contain explicit description of things such as different protein isoforms or post-translational modifications. These concepts are instead incapsulated in an `epitope` entity. An epitope represents the target of an antibody-based measurement (e.g. from flow-cytometry or immuno histo-chemistry). Epitope ids are a concatenation of the Uniprot name of the protein they refer to and, when necessary, either the post-translational modification, isoform name, or any other identifier that distinguishes the specific target of the antibody.

Any protein based measurement will be connected to an epitope entity. For each protein, CANDEL contains at least one epitope entry, possibly many more if there are known post-translational modifications, isoforms etc. 

## Representing gene expression measurements

Similar to proteins, CANDEL does not contain an explicit representation of transcripts, isoforms etc. These concepts are encapsulated in a `gene-product` entity. Gene expression measurements will thus refer to `gene-product` entities

Both `gene-product` and `epitope` provide a level of indirection that enable representing complicated scenario in details when needed (e.g. post-translational modifications or specific isoforms), while keeping things simple in the common case.


## Representing single-cell data

Representing single-cell data (e.g. from flow or imaging) requires you to first define the cell populations that are the targets of the measurements. Cell popuplations can be derived either from clustering or from manual gating. 

### Gated populations

Cell populations derived from gating must be associated with a `cell-type`, as defined by the Cell Ontology. This is necessary to be able to have some standardization of cell populations across different datasets. This process essentially involves finding out the Cell Ontology term that most closely correspond to your gated population. Some common populations, and their respective standardized Cell Ontology names, are listed in the table below

| population | Cell Ontology name|
|------------|-------------------|
| NK cells | natural killer cell |
| CD4 T cells | CD4-positive, alpha-beta T cell |
| CD8 T cells | CD8-positive, alpha-beta T cell |
| CD4 T cells, central memory (this and the other categories below are normally differentiated on a CD45RA vs CCR7 biaxial plot) | central memory CD4-positive, alpha-beta T cell |
| CD4 T Cells, naive | naive thymus-derived CD4-positive, alpha-beta T cell |
| CD4 T cells, effector memory | effector memory CD4-positive, alpha-beta T cell |
| CD8 T cells, central memory | central memory CD8-positive, alpha-beta T cell |
| CD8 T cells, effector memory | effector memory CD8-positive, alpha-beta T cell |
| CD8 T Cells, naive | naive thymus-derived CD8-positive, alpha-beta T cell |
| TEMRA | effector memory CD8-positive, alpha-beta T cell, terminally differentiated |
| T regs | CD4-positive, CD25-positive, alpha-beta regulatory T cell |
| classical monocytes (as defined on a CD14 vs CD16 biaxial) | CD14-positive, CD16-negative classical monocyte |
| non-classical monocytes | CD14-low, CD16-positive monocyte | 
| intermediate monocytes | CD14-positive, CD16-low monocyte |
| myeloid (or conventional) dendritic cells (mDC or cDC) | myeloid dendritic cell |
| plasmacytoid dendritic cells (pDC) |	plasmacytoid dendritic cell |
| plasmablast | plasmablast |
| unswitched memory B cells | unswitched memory B cells |
| class switched memory B cell | class switched memory B cell |
| naive B cells (IgD+ CD27-) | naive B cell |

In addition cell populations derived by gating can be further characterized by using the attributes `:cell-population/positive-markers` and `:cell-population/negative-markers`. These are meant to represent things such as Ki67+ CD8 T cells, where the markers are used to specify some functional attributes. These attributes are also useful because the Cell Ontology does not represent every arbitrary population that one could identify by gating. Note that these are not meant to represent all the markers that a given cell population is positive (or negative for), only the ones that are used  to further characterize it beyond what is already implied by its identity. For instance a Ki67+ positive CD8 T cell is also positive for CD3, CD45, CD8 etc., but you would only include Ki67 as positive marker, as the other ones are already implied by its identity

### Measurements on cell populations

Two types of measurement can be associated with a cell poulation:
1. Measurements representing the expression of a specific marker
2. Measurements representing the abundance of a cell population in a sample

Expression measurements contain multiple attributes as they link together a sample, a cell population, an epitope (i.e. the marker that was measured), and the measurement value itself, represented using the `:measurement/median-channel-value` attribute.

Abundance measurement can be expressed as either an absolute or relative number (e.g. using `:measurement/percent-of-nuclei` or `:measurement/percent-of-lymphocytes`). Please make sure to always include absolute measurements when you import a dataset. Note that when importing absolute measurements you also need to include a measurement representing the total number of cells present in the sample (using attributes such as `:measurement/live-count`, `:measurement/nuclei-count` etc.). Such measurements do not have a cell population as target, because they refer to the sample as a whole. The `wick` package contains convenience functions to generate the appropriate measurement data from a table of cell counts.

### Cell populations derived from clustering

For cell populations derived by clustering you do not have to associate them with a cell type from the Cell Ontology, and you also should not use the positive and negative markers attributes. However you need to include expression measurements in order to define the characteristics of the cluster.



## Representing time

Deciding how to model time is one of the trickiest decision when importing (or even simply analyzing) a dataset 

Each patient has his/her own timeline, but for the purpose of analysis these timelines need to be adjusted to a common reference frame that groups together events that are conceptually related even though they may not have happened simultaneously from a chrnological perspective, or they may not have the same relationship to other events that occured before or after in each patient timeline (for instance surgery may have happened two weeks after the last treatment for patient A, and 4 weeks after for patient B)

Also to consider is the fact that detailed chronological information is often missing

The fundamental entity used to represent time is a `timepoint`. In importing a dataset you need to make sure to create all the timepoint entities that are necessary to represent the totality of the timepoints that appear in your data, irrespective of how many observations are connected to each timepoint. For instance, assuming that the following table represents samples that have been collected for subjects on a study

|Subject |Timepoint 1 |Timepoint 2 |Timepoint 3 |
|--------|:----------:|:----------:|:----------:|
|101     |X           |            |X           |
|102     |X           |            |X           |
|103     |X           |X           |X           |

You would have to create entities for all three timepoints, even though timepoint 2 is only relevant for subject `103`

The timepoint namespace contains the following attributes

| Attribute | Description |
|-----------|-------------|
| `:timepoint/type` | This attribute is a reference to entities in the timepoint namespace and is used to indicate what is the type of this timepoint. The type of a timepoint represent what kind of event this timepoint represents, and refers to concepts such as `baseline`, `surgery`, `treatment` and so on |
| `:timepoint/offset` | This is an optional attribute to indicate a point in time that occurred with a positive or negative offset (in days) with respect to the logical timepoint that this entity is meant to represent. This can be useful in cases where there is a well-determined schedule of events (e.g. in a clinical trial), but some events occurred before or after the time they were supposed to. For instance if this timepoint represented `treatment2` but collection for some patients happened 5 days before or after this, you would indicate this with two separate entities having both `treatment2` as type and offests -5 and +5 respectively |
| `:timepoint/relative-order` | This attribute is mandatory for timepoints that are part of a treatment-regimen (see below) and indicates the relative order of this timepoint **within** the treatment regimen it is part of (i.e. not among all timepoints). If a timepoint is not part of a treatment-regimen it should **not** include this attribute |
| `:timepoint/treatment-regimen` | This attribute points to the treatment regimen this timepoint is part of, if any |
| `:timepoint/id` | This attribute is a string that uniquely identifies the timepoint within the dataset. The id is constructed by joining using `/` the following fields (when present, note that only the type is mandatory): 1) the treatment regimen name, 2) the timepoint type, 3) the timepoint offset |

Best practices:

1. Certain timepoints may exist outside of any treatment regimen. This is intended for timepoints that represent concepts such as `baseline` or `end of study`
2. Unless more detailed information is available, study-level endpoints such as progression-free survival, overall survival etc., should be connected to the end of study timepoint `:timepoint/eos`
3. Unless more detailed information is available, subject `age` should be connected to the baseline timepoint (`timepoint/baseline`) 

### Ordering of treatment regimens

Since the ordering of timepoints is relative to a treatment regimen, it is necessary to specify the ordering of treatment regimens themselves in order to reconstruct the sequence of events as it happened for a particular subject.

The way to do that is to use entities of type `therapy` which are connected to each subject through the `:subject/therapies` attribute

Note that it is mandatory to explicitly specify therapies even for datasets that contain a single treatment regimen that was the same for all subjects.

Therapy entities simply specify the relative order of treatment regimens for a particular subject (refer to the schema documentation for more details). Note that timepoints that do not refer to any treatment regimen represents concepts (such as `baseline`) and therefore have no real chronological placement

## Representing data for survival analysis (e.g. PFS, OS etc.)

Data for survival analysis (e.g. PFS, OS etc.) is represented using a combination of three attributes:

- The value for the variable (e.g. the number of days of Progression Free Survival)
- A boolean `event` flag that indicates whether an event has occurred (see below) that triggered a change in state, or whether the observation is censored. The convention in this type of data is to indicate censoring with `0` (i.e. `false` in CANDEL) and the occurrence of an event with `1` (i.e. `true` in CANDEL)
- A `reason` attribute that points to enums in the `:clinical-observation.event-reason` namespace indicating the nature of the event or the reason for censoring. 

These three attributes are repeated for each type of data, i.e. there are `pfs`, `pfs-event` and `pfs-reason` as well as `os`, `os-event` and `os-reason` etc.
Here are a few examples of how these attributes are combined in common scenarios:

- A subject went 245 until clinical progression happened:

```
{:clinical-observation/pfs         245
 :clinical-observation/event       true
 :clinical-observation/pfs-reason  :clinical-observation.event-reason/progressed}
```

- A subject has a PFS of 245 but the observation is censored because the subject was lost to follow-up

```
{:clinical-observation/pfs         245
 :clinical-observation/event       false
 :clinical-observation/pfs-reason  :clinical-observation.event-reason/censored-lost-to-follow-up}
```

Note that OS datasets often contain a variable indicating survival status at 1 year (or any other relevant milestone), which is not explicitly represented in CANDEL as it can be trivially derived from the combination of the above variables as follows


| OS (days) | Event | Survival status at 1 Year|
|-----------|-------|--------------------------|
| < 365     | false | Unknown                  |
| < 365     | true  | Dead                     |
| > 365     | false | Alive                    |
| > 365     | true  | Alive                    |

