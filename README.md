# SeroBA
This is a fork of Wellcome Sanger Institute Pathogen Informatics' SeroBA. As the original SeroBA is no longer maintained, this fork aims to integrate bug fixes to provide a stable Docker enviornment for the [GPS Unified Pipeline](https://github.com/HarryHung/gps-unified-pipeline) (a Nextflow Pipeline for processing Streptococcus pneumoniae sequencing raw reads). 

## Warning
We do not provide any support for the use or interpretation of this software, and it is provided on an "as-is" basis.

## About 
SeroBA is a k-mer based Pipeline to identify the Serotype from Illumina NGS reads for given references. You can use SeroBA to download references from [PneumoCaT](https://github.com/phe-bioinformatics/PneumoCaT) to do identify the capsular type of Streptococcus pneumoniae.

## Contents
- [SeroBA](#seroba)
  - [Warning](#warning)
  - [About](#about)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Docker Image](#docker-image)
  - [Usage](#usage)
    - [Running the tests](#running-the-tests)
    - [Setting up the database](#setting-up-the-database)
    - [Creates a Database for kmc and ariba](#creates-a-database-for-kmc-and-ariba)
    - [Identify serotype of your input data](#identify-serotype-of-your-input-data)
    - [Summaries the output in one tsv file](#summaries-the-output-in-one-tsv-file)
  - [Output](#output)
  - [Troubleshooting](#troubleshooting)
  - [License](#license)
  - [Citation](#citation)

## Introduction
SeroBA can predict serotypes, by identifying the cps locus, directly from raw whole genome sequencing read data with 98% concordance using a k-mer based method, can process 10,000 samples in just over 1 day using a standard server and can call serotypes at a coverage as low as 10x. SeroBA is implemented in Python3 and is freely available under an open source GPLv3

## Docker Image
A pre-built Docker Image is available on [harryhungch/seroba on Docker Hub](https://hub.docker.com/repository/docker/harryhungch/seroba/)


## Usage
All the following instructions are assuming you are working within a Docker container

### Running the tests
The test can be run from the top level directory:  

```
python3 setup.py test
```

### Setting up the database
You can use the CTV of PneumoCaT by using seroba  getPneumocat. It is also possible to add new serotypes by adding the references sequence to the "references.fasta" file in the database folder. Out of the information provided by this database a TSV file is created while using seroba createDBs. You can easily put in additional genetic information for any of these serotypes in the given format.

Since SeroBA v0.1.3 an updated variant of the CTV from PneumoCaT is provided in the SeroBA package. This includes the serotypes 6E, 6F, 11E, 10X, 39X and two NT references. It is not necessary to use SeroBA getPneumocat.
For SeroBA version 0.1.3 and greater, download the database provided within this git repository:

__For git users__  
Clone the git repository:
```
git clone https://github.com/sanger-pathogens/seroba.git
```

Copy the database to a directory:
```
cp -r seroba/database my_directory
```

Delete the git repository to clean up your system:
```
rm -r seroba
```

__For svn users__  
Install svn. Checkout the database directory:
```
svn checkout "https://github.com/sanger-pathogens/seroba/trunk/database"
```
Continue with Step 2.

__For SeroBA version 0.1.2 and smaller:__
```
usage: seroba  getPneumocat <database dir>

Downloads PneumoCat and build an tsv formatted meta data file out of it

positional arguments:
  database dir      directory to store the PneumoCats capsular type variant (CTV) database
```

### Creates a Database for kmc and ariba
```
usage: seroba createDBs  <database dir> <kmer size>

positional arguments:
    database dir     output directory for kmc and ariba Database
    kmer size   kmer_size you want to use for kmc , recommended = 71

Example : 
seroba createDBs my_database/ 71
```
### Identify serotype of your input data
```
usage: seroba runSerotyping [options]  <databases directory> <read1> <read2> <prefix>

    positional arguments:
      database dir         path to database directory
      read1              forward read file
      read2              reverse read file
      prefix             unique prefix

    optional arguments:
      -h, --help         show this help message and exit

    Other options:
      --noclean NOCLEAN  Do not clean up intermediate files (assemblies, ariba
                         report)
      --coverage COVERAGE  threshold for k-mer coverage of the reference sequence (default = 20)                         
```

### Summaries the output in one tsv file
```
usage: seroba summary  <output folder>

positional arguments:
  output folder   directory where the output directories from seroba runSerotyping are stored
```   

## Output
In the folder 'prefix' you will find a pred.tsv including your predicted serotype as well as a file called detailed_serogroup_info.txt including information about SNP, genes, and alleles that are found in your reads. After the use of "seroba summary" a tsv file called summary.tsv is created that consists of three columns (sample Id , serotype, comments). Serotypes that do not match any reference are marked as "untypable"(v0.1.3).

__detailed_serogroup_info example:__
```
Predicted Serotype:       23F
Serotype predicted by ariba:    23F
assembly from ariba has an identity of:   99.77    with this serotype

Serotype       Genetic Variant
23F            allele  wchA
```
In the detailed information you can see the finally predicted serotype as well as the serotypes that had the closest reference in that specific serogroup according to ARIBA. Furthermore you can see the sequence identity between the sequence assembly and the reference sequence.  

## Troubleshooting
* Case 1:
	* SeroBA predicts 'untypable'. An 'untypable' prediction can either be a
real 'untypable' strain or can be caused by different problems. Possible problems are:
bad quality of your input data, submission of a wrong species or to low coverage
of your sequenced reads. Please check your data again and run a quality control.

* Case 2:
	* 	Low alignment identity in the 'detailed_serogroup_info' file. This can
be a hint for a mosaic serotpye.
	* Possible solution: perform a blast search on the whole genome assembly

* Case 3:
	* The third column in the summary.tsv indicates "contamination". This means that
    at least one heterozygous SNP was detected in the read data with at least
    10% of the mapped reads at the specific position supporting the SNP.
	* Possible solution: please check the quality of your data and have a look
    for contamination within your reads

## License
SeroBA is free software, licensed under [GPLv3](https://github.com/sanger-pathogens/seroba/blob/master/LICENSE)

## Citation
__SeroBA: rapid high-throughput serotyping of Streptococcus pneumoniae from whole genome sequence data__  
Epping L, van Tonder, AJ, Gladstone RA, GPS Consortium, Bentley SD, Page AJ, Keane JA, Microbial Genomics 2018, doi: [10.1099/mgen.0.000186](http://mgen.microbiologyresearch.org/content/journal/mgen/10.1099/mgen.0.000186)