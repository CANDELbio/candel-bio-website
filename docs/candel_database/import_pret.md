---
layout: page
title: Import with Pret
permalink: /candel_database/import_pret/
parent: CANDEL Database
nav_order: 5
---


# pret
{: .no_toc }

**pret**, which stands for "Programmable ETL" (and also means *ready* in French), is the tool that enables you to map arbitrary datasets onto the CANDEL [schema](), and allows you to request a new database and load data into it

Please use the sidebar to navigate to instructions about using the pret CLI as well as detailed documentation on the directives that specify how data is processed by pret.

If you prefer to learn by looking at real-world practical examples, please go to the [Importing data]()


## Environment setup and installation

### Prerequisites

`pret` requires the following installed:
1. Java version 11. Also likely compatible with Java 1.8 or 1.9, but this is not guaranteed. See [OpenJDK](https://openjdk.java.net/install/) for installation instructions.
2. [The Google Cloud SDK](https://cloud.google.com/sdk/docs/)

Configure your access credentials with the Google Cloud CLI by running:

`gcloud auth application-default login`

at a terminal and following the login procedure.

### Installing pret

You can retrieve the latest version of `pret` by running the following command at the terminal:

`gsutil cp gs://pret-releases/pret-$VERSION.zip .`

Where `$VERSION` is the version of `pret` you want. To see all releases use `gsutil ls gs://pret-releases`. Normally you will want the latest version. Note that version numbers do not have leading zeroes, so `pret-1.0.9` may appear after `pret-1.0.35`, but in that case `pret-1.0.35` is the more recent version.

After you've downloaded the latest version of pret, unzip the archive:

`unzip pret-$VERSION.zip`

And then change into the directory you just created. 

`cd pret-$VERSION`

## Running

Invoke `pret` from inside the unzipped directory with:

`./pret` in linux or macOS

`pretw.bat` in Windows

This will echo the command line usage options (For details, see below). 
For instance, to execute the `prepare` task, you would run:

```./pret prepare --import-config /path/to/config.edn --working-directory /path/to/working-dir/```

### pret CLI usage and options

The `pret` command line utility (CLI) is run as follows (Using mac/linux script. For Windows usage see above.):

```./pret <task> <options>```

Run `pret` with no arguments, ie `./pret`, for a full list of commands and arguments.

`pret` can be run to execute the following five primary tasks:

| Task | Description |
|-----------|-------------|
| `login` | Confirms identity via Google and issues an authentication token. Requires `--email` arg. Required before execution of `request-db`, `transact`, and `validate` tasks. |
| `request-db` | Creates a new branch database that is a copy of the master CANDEL database. User must be logged in. Use this task when you begin an import, or after a failed partial transaction, to start with a clean database. Returns the name of the branch database. This request takes a few minutes to fulfill and will notify you by email when complete.|
|`request-empty-db`| Creates a new database with bootstrapped reference data only (does not inclue other datasets). This can be helpful for getting an initial import working, but provides no information about whether or not e.g. any reference data in the data imported into it will conflict or overlap with data in the existing master database.|
| `prepare` | Takes your import config and input data files and prepares them for transacting into the database. Requires the import config and working directory options (see below). This task also performs numerous validations on your config and data files and will emit an error if your files don't pass these validations. Note that this task can be run repeatedly to iterate on correct formatting of config and import data - just be sure to empty out your working directory if you have some processed files in it. `pret` will emit an error if you don't do that.|
| `transact` | Take your prepared data and transact it into a database. Requires `--database` argument with name previously used for provisioning a branch or empty db, as well as the `--working directory` argument. You'll want to run this task against the database you spun up using the `request-db` task. You can also transact data created with `diff` by using the `update` arg. |
|`diff`| The diff workflow can be used to stage a subset of prepared data for transaction. See the documentation for the `diff` workflow below. |
| `validate` | After transacting your data successfully, this task will perform additional validations on the data which will determine whether e.g. measurements have all necessary attributes, or that references in the dataset refer to other entities that have been successfully transacted. |

The options for these tasks are as follows:

| Option | Description |
|-----------|-------------|
| `--email USER-EMAIL` | User email. Identifier for google backed domain (e.g. gmail, parkerici.org, or other allowed google org), used for login.|
| `--import-config IMPORT-CONFIG`| Import Config edn file (required for prepare)|
| `--working-directory IMPORT-WORKING-DIRECTORY`| Directory where prepared data goes, transact uses data prepare puts here (required for prepare and transact)|
|  `--tx-batch-size TX-BLOCK-SIZE`| Datomic transaction batch size (defaults to 50), most of the time you will not need to set this. |
| `-h, --help` | Help function.|

### Example usage

The following example would log in and then request a branch database, with the name `candel-db-123` specified. Next the commands prepare data as specified in `~/repos/pret-datasets/tcga/config.edn` to the working directory `~/data/tcga-import/tmp-working`, and then transact the data into the database created in the `request-db` task, and validate the transacted data.

```./pret login --email bestuser@parkerici.org```

```./pret request-db --database candel-db-123```

OUTPUT: `Request successful, created database:  candel-db-123`

```./pret prepare --import-config ~/repos/pret-datasets/tcga/config.edn --working-directory ~/data/tcga-import/tmp-working```

```./pret transact --database candel-db-123 --working-directory ~/data/tcga-import/tmp-working```

```./pret validate --database candel-db-123 --working-directory ~/data/tcga-import/tmp-working```

### Diff merge documentation

See the full diff workflow documentation [here]()