---
layout: page
title: Data Exploration with Enflame
permalink: /data_analysis/enflame/
parent: Data Analysis for CANDEL
nav_order: 3
---



# Table of Contents

1.  [Introduction](#org81de198)
2.  [UI Overview](#org54a848b)
    1.  [Block palette](#org2e3044a)
    2.  [Block canvas](#orgb7bf573)
    3.  [Results Browser](#org6f31807)
    4.  [Entity Browser](#org93b6ff4)
    5.  [Other right hand panes](#org1cce01f)
        1.  [DB selector](#orgb202297)
        2.  [Query](#org4575696)
        3.  [Wick](#org27ac78e)
        4.  [Library](#org9dffd41)
3.  [Blocks and their semantics](#orgcd474de)
    1.  [Block types](#org9f71d42)
    2.  [Entity blocks](#org0835a0e)
    3.  [Attribute blocks](#orgefad95d)
        1.  [Entity inputs](#orgf605744)
        2.  [Complex relationships](#org2940ce8)
    4.  [Relation to Schema](#org5b941db)


<a id="org81de198"></a>

# Introduction

[Enflame](http://enflame.parkerici.org/index.html) is a visual query builder for CANDEL. You can construct a query by combining blocks. You can then run the query against a selected CANDEL database, or you can take the generated text representation of it (in both Clojure and R (Wick) formats) and include them in your own code.  

CANDEL is a graph database containing roughly 30 different *types* of *entity* (or object). Eg types include `gene`, `subject`, and `sample` .  [The complete schema]() is available. Queries specify a type and some constraints on that type, which might involve relationships with other entities.  

For instance, this query will return all the variants of the gene `BRCA1`:  

![img](Screen_Shot_2019-09-20_at_12.31.37_PM.png)  

The results look like this, with each row representing a variant and its associated attributes.  

![img](Screen_Shot_2019-09-23_at_5.32.15_PM.png)  

Another example: This query produces a list of diseases along with the count of subjects who have that disease:  

![img](Screen_Shot_2019-09-20_at_12.36.07_PM.png)  

![img](Screen_Shot_2019-09-23_at_5.34.10_PM.png)  

Enflame makes use of the [Blockly library](https://developers.google.com/blockly/) from Google which is inspired by [Scratch](https://scratch.mit.edu/).  


<a id="org54a848b"></a>

# UI Overview

These are various components of the Enflame user interface, described below:  

![img](layout.png)  


<a id="org2e3044a"></a>

## Block palette

This is the source of all blocks, organized by semantic type. Each type has a unique color that is reflected in the blocks and in the browsers.  


<a id="orgb7bf573"></a>

## Block canvas

This is where you compose blocks into a query.  

Note: one non-obvious feature – you can put arbitray blocks into the canvas, including scrap parts. The block structure closest to the upper-left hand corner will be the one that actually is turned into a query, and stray blocks will be ignored.  


<a id="org6f31807"></a>

## Results Browser

This is where the results of queries are displayed. Columns are colored by type. The columns with names like 𝞙|dataset indicate the actual queried entity itself – other columns are (typically) attributes of that entity. For instance, this table is the result of a query on `subject` s. The first column represents the subject entity itself, the other columns (of the same color) are individual attributes of that entity.  

![img](Screen_Shot_2019-09-23_at_5.35.35_PM.png)  

Values that are entities themselves are displayed with their unique-id if available, and as links. The links will open up that entity in the entity browser.  


<a id="org93b6ff4"></a>

## Entity Browser

The entity browser shows a single entity and its values, and lets you navigate through the graph structure of the CANDEL.  

TODO: Back button, history, translate to block.  


<a id="org1cce01f"></a>

## Other right hand panes

The right-hand column of the display has a bunch of separate panes for particular purposes.  


<a id="orgb202297"></a>

### DB selector

This pane lets you choose the server and database to query against.  


<a id="org4575696"></a>

### Query

This pane contains the generated query (in Clojure format) and a **Go** button to start a query.  


<a id="org27ac78e"></a>

### Wick

This contains the generated query in Wick (R) format.  

You can run these queries by passing the text as an argument to the function:  

{% highlight nil %}
wick::do_query(“....”)
{% endhighlight %}


<a id="org9dffd41"></a>

### Library

The library pane allows you to save a query. Queries are saved with the following information:  

-   a picture of the block structure
-   an automatically generated text version of the query (eg `[measurements where [nanostring-count > 0]]`)
-   an optional user-supplied description.


<a id="orgcd474de"></a>

# Blocks and their semantics


<a id="org9f71d42"></a>

## Block types

Blocks are organized by semantic type (ie `subject` or `drug`), each of which has a separate color.  

![img](Screen_Shot_2019-09-20_at_4.35.29_PM.png)  

The types and colors are displayed on the left-hand side of the Enflame screen:  

![img](Screen_Shot_2019-09-20_at_4.36.23_PM.png)  

For each type, there are a few different kinds of block availble, described below. You get blocks by clicking on a type name, which will expose a palette of available blocks  

![img](Screen_Shot_2019-09-20_at_4.37.43_PM.png)  


<a id="org0835a0e"></a>

## Entity blocks

**Entity blocks** have a nub on their left hand side. You can think of them as generating a single entity (eg a `gene`) or a set of entities.  

There are two kinds of entity blocks. A **named entity block** returns a single entity, For instance this produces the single subject with the given id:  

![img](Screen_Shot_2019-09-20_at_12.45.50_PM.png)  

A **query entity block** produces a set of entities of a given type. For instance, this produces the set of all subjects:  

![img](Screen_Shot_2019-09-20_at_12.46.18_PM.png)  

Query entity blocks have an additional selector that lets you specify the output type. The options are:  

-   `include`: (default) include the entity itself and its label (unique-id) if available
-   `pull`: include the entity and all of its attributes
-   `count`: don՚t return the entity itself, but instead the count of its unique values based on the rest of the query
-   `omit`: don՚t return anything for this entity


<a id="orgefad95d"></a>

## Attribute blocks

**Attribute blocks** specify a *constraint* on query entity block. They fit into the right-hand side of  a query entity block of the same color (type). Eg, here՚s how you could use three attribute blocks to get a list of all the subjects who are dead white males:  

![img](Screen_Shot_2019-09-20_at_12.50.54_PM.png)  

Note that the constraints are ANDed together. If you want to specify an OR, there is a special block for that. This query specifies subjects that are alive **and** are either Asian **or** Pacific Islander:  


<a id="orgf605744"></a>

### Entity inputs

![img](Screen_Shot_2019-09-20_at_12.52.50_PM.png)  

Some attribute blocks require an entity block as an input. The query below returns diseases of white male subjects. The attribute block is in the middle (“subject is”) and connects two query entity blocks.  

![img](Screen_Shot_2019-09-20_at_12.58.30_PM.png)  


<a id="org2940ce8"></a>

### Complex relationships

Most attribute blocks represent either a primitive-valued property of an entity type, or a semantic relation that corresponds to a single edge on the schema graph. That is, they are a 1:1 mapping of the underlying CANDEL schema. However, there are some blocks that represent more complex relationships. For instance, the age block below is a complex relationship (age is actually a direct property of clinical-observation, not subject). The marker ⨷ is used to distinguish complex blocks.  

![img](Screen_Shot_2019-10-08_at_2.45.20_PM.png)  


<a id="org5b941db"></a>

## Relation to Schema

The CANDEL schema can be visualized as a graph where nodes are entity types and edges are the defined attributes that connect them:  


Any block query can be mapped to a subset of the schema diagram. For instance, this query (samples of all subjects with disease prostate cancer)  

![img](Screen_Shot_2019-10-08_at_2.02.20_PM.png)  

Is specifying entities and attributes from the section of the graph highlighted in green:  

![img](alzabo-highlighted.png)  