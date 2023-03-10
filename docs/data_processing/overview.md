---
layout: page
title: Overview
permalink: /data_processing/overview/
parent: Data Processing
nav_order: 2
---

The CANDEL database is designed to store processed values instead of raw data, and so various workflows have been developed to assist in the transfer of the raw data into the processed form. This can include identification of cells and marker intensity of IHC images, population gating and summary statistic exports from flow cytometry data, or alignment of sequncing data, identificaiton of mutations, or gene expression quantification. The tool workerbee is designed to help bridge the final step, and munge processed data into a format for easy import into the database.

Some of these processes can be accomplished with tools in the CANDEL platform, and many of these are built around external tools as well. 

Bulk processing of NGS or high dimensional imaging data is handeled by workflows we have developed for Terra, written in WDL and available on dockstore.

Our cytometry tools are built around the assumption of using cellengine.

