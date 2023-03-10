---
layout: page
title: Imaging Workflow
permalink: /data_processing/imaging/
parent: Data Processing
nav_order: 3
---



This page describes the use of our [Terra]({% link docs/terra/index.markdown %}) pipeline for image segmentation, using the [Mantis](https://www.biorxiv.org/content/10.1101/2021.03.01.431313v2) tool from the Vanvalen lab.

## Put image files in RawSugar

Follow the instructions from the [protocol for raw data for PICI trials](https://dataplatform.parkerici.org/rawdata/raw-data-pici-trials/) to get your data into Rawsugar. 

##### RawSugar data setup
- Use the op `New sheet for files of type` to ensure only .tiff or .tif files are in your sheet
- If multiple images are associated with a single sample, use the op `Separate files into individual rows` to get each image file on its own row. This will create a new ROI column, numbered 0,1,2....
- Use the op `Concatenate two columns' values` to join sample id and ROI, to create a new column with unique sample IDS for each file. This will be required for Terra.

## Transfer data to Terra

- Create a workspace on [Terra](https://app.terra.bio/#)
	- From the Workspaces tab on Terra, click on the "+" to add a new workspace, and name it after your project. 
	- For billing project select "pici-firecloud". 
	- **Do not** select an Authorization Domain.
	- Enter a brief description of what this workspace will be used for.
- Share the workspace with **GROUP_pici-internal@firecloud.org** and give "Writer" access. 
	- From the Dashboard tab of your new workspace page, click on the three-dot button on the top right of the page and select "share".
- Use the rawsugar op `sheet-files->terra` to transfer files to Terra. 
	- If you have panel info, it is easier to use the **panel** as **participant-id column** to maintain that information in Terra.
	- For **sample-id column** choose a column with unique sample ids, usually the result of `concatenate two columns' values`. 
	- For **terra-column name** use `multi-tiff`. This will create a new column in your workspace called multi-tiff containing your files from rawsugar.


## Run Segmentation

#### Set permissions

This step only needs to be done once per user, and can often be easier to ask [rkageyama@parkerici.org](mailto:rkageyama@parkerici.org) or [mthompson@parkerici.org](mailto:mthompson@parkerici.org) for assistance.  If you would like to do it yourself, the steps are listed below.

To use any data from GCP on Terra you must first give your proxy account object storage reader permissions in GCP. This should only need to be done once for each google bucket.

- In [https://app.terra.bio/#profile](https://app.terra.bio/#profile) you will find your Proxy Group id. Copy it.
- Navigate Google Storage Console for the bucket [pici-data-warehouse](https://console.cloud.google.com/storage/browser/pici-data-warehouse?forceOnBucketsSortingFiltering=false&project=pici-internal) and select the Permissions tab
	- Select "Add members"
	- Paste your proxy id
	- Select "Storage Object Viewer"

This should be done for the rawsugar bucket [pici-data-warehouse](https://console.cloud.google.com/storage/browser/pici-data-warehouse?forceOnBucketsSortingFiltering=false&project=pici-internal)


#### Determine channel IDs

The Mesmer pipeline can run two separate types of segmentation: nuclear and whole-cell

To run the process, you must first identify the channel IDs of your nuclear marker(s) and your cytoplasmic/membrane marker(s) in your multi-channel tiff.

The `List Channels` workflow exports a list of channel IDs with their tags/channel names to help with this process. If all samples in your experiment use the same marker panel, you likely only need to run `List Channels` on a single sample and can use the same channel ID lists for all images.

- Import the [list channels](https://dockstore.org/workflows/github.com/ParkerICI/mesmer-wdl-workflow/List-Channels) workflow to your workspace (by clicking the Terra button in the Launch With menu).
- Select your workspace from the dropdown menu and click `Import`
- You should be redirected to the workflow configuration page, which you can also reach from your workspace, navigating to Workflows and selecting `List-Channels`
- Select Data
	- Set the Root Entity Type to Sample 
	- Click the Select Data button and choose one (or multiple) sample rows
- Run on your samples:
	- Root Entity Type: `sample`
	- Inputs:
		- multi_tiff: `this.multi-tiff` (assuming this is what you named your tiff column from the RawSugar import)
		- docker_image: _leave blank for default_
		- mem_gb: _leave blank for default_
		- rename_to_sampleid: can set to "true" if you want the output file renamed to your sample id
		- sampleid: Required if you set rename to true. Set to `this.sampleid` assuming your sample ID column is called sampleid
	- Outputs:
		-  channel_list: `this.channel_list`
- Output will be saved to the DATA tab under sample, and the column `channel_list`

#### Choose Channels

Examine the file(s) saved in the `channel_list` column for the sample(s) you ran. The list provides a channel ID and the tags/channel names from the multi-layer tiff.

For running nuclear segmentation only, you will need to choose at least one channel that has your nuclear/DNA signal. You can choose multiple channels and the next step will flattena and sum them into a single combined channel. Note the channel ID(s) of the channel(s) you wish to use for the nuclear signal.

You may also choose to run whole-cell segmentation. Similar to nuclear, you will need to choose one or more channels in the multi-tiff that represent your cytoplasmic/membrane signal. Note the IDs you wish to use for the whole-cell segmentation.

#### Run Mesmer Segmentation

- Import the [mesmer-segmentation-full](https://dockstore.org/workflows/github.com/ParkerICI/mesmer-wdl-workflow/mesmer-segmentation-full) workflow to your workspace (by clicking the Terra button in the Launch With menu).
- Select your workspace from the dropdown menu and click `Import`
- You should be redirected to the workflow configuration page, which you can also reach from your workspace, navigating to Workflows and selecting `mesmer-segmentation-full`
- Select Data
	- Set the Root Entity Type to Sample 
	- Click the Select Data button and choose the sample rows you wish to segment
- Run on your samples:
	- Root Entity Type: `sample`
	- Inputs:
		- multi_tiff: `this.multi-tiff` (assuming this is what you named your tiff column from the RawSugar import)
		- nuc_channel_ids: *REQUIRED* JSON string array of the channel IDs you wish to use for nuclear segmentation. Example: `"[1, 2]"` would use channels 1 and 2; `"[1]"` would use channel 1 alone
		- mem_gb: _leave blank for default_
		- mesmer_docker_image: _leave blank for default_	
		- rename_to_sampleid: can set to "true" if you want the output masks renamed to your sample id (recommended)
		- run_wc: boolean indicating whether you wish to run whole-cell segmentation (in addition to nuclear). Set to "true" to run both
		- sampleid: Required if you set rename_to_sampleid to true. Set to `this.sampleid` assuming your sample ID column is called sampleid
		- tiff_tools_docker_image: _leave blank for default_
		- wc_channel_ids: JSON string array of the channel IDs you wish to use for whole-cell segmentation. Example: `"[1, 2]"` would use channels 1 and 2; `"[1]"` would use channel 1 alone
	- Outputs:
		-  nuc_mask: `this.nuc_mask`
		-  wc_mask: `this.wc_mask`
- Output segmentation masks will be saved to the DATA tab under sample, and the columns `nuc_mask` and `wc_mask` (if you ran both).