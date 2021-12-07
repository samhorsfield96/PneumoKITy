# README for PneumoKITy

## Table of content

* Introduction
* [Dependencies](https://github.com/CarmenSheppard/PneumoKITy#dependencies)
* [Running PneumoKITy](https://github.com/CarmenSheppard/PneumoKITy#running-PneumoKITy)
* [User customisable options](https://github.com/CarmenSheppard/PneumoKITy#User-customisable-options)
* [Example command lines](https://github.com/CarmenSheppard/PneumoKITy#Example-command-lines)
* [How PneumoKITy works](https://github.com/CarmenSheppard/PneumoKITy#How-PneumoKITy-works)
* [Quality checks](https://github.com/CarmenSheppard/PneumoKITy#Quality-checks)
* [PneumoKITy Output](https://github.com/CarmenSheppard/PneumoKITy#Output-files)
* [Interpretation of results](https://github.com/CarmenSheppard/PneumoKITy#Interpretation-of-results)

PneumoKITy (**Pneumo**coccal **K**mer **I**ntegrated **Ty**ping) is a 
lite version of the in-development PneumoCaT2. It is a from the ground up, redevelopment of the original [PneumoCaT](https://github.com/phe-bioinformatics/PneumoCaT)
capsular typing tool, written for **Python 3.7+**, using different methods. Stage 1 uses the excellent tool [MASH](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-016-0997-x) for 
kmer based analysis.  As PneumoCaT2 is not ready yet, we decided to create a lite version (PneumoKITy) basis,
as this lite version could still be very useful for fast serotype assessment and detection of mixed serotypes in fastQ data 
even though it is not capable of fully serotyping all serotypes.
PneumoKITy can serotype about 58% of the serotypes defined by the SSI Diagnostica typing sera, and provides some useful
information regarding subtypes and genetic types. 

PneumoKITy, like the original PneumoCaT tool assigns capsular types to
*S.pneumoniae* genomic data  using a using a two step approach, however PneumoKITy, is limited only to second stage determinations that can be assessed using presence or absence or gene allele variants.

Serotypes that require determination using alternative variants such as SNPs cannot be distinquished using PneumoKITy.
PneumoKITy has the advantage that it can be used on assembly files OR illumina fastq read files and it is incredibly fast.
   
PneumoKITy uses very different methods to previous PneumoCaT versions and requires a new running environment.

 The **C**apsular **T**ype **V**ariant database (CTVdb) used in PneumoKITy is now a real database, running in SQLite3,
 allowing  for much easier updating of information and better scope for the database to grow in the future as more variants and serotype 
 determinants are added. The use of this format will also allow us to store  extra related information about the serotypes, such as subtype information 
 (subtyping etc is planned in a future update). 

For PneumoKITy specifically any snp or gene_function variants have been removed from the CTVdb as it is not possible to perform
these determinations usign kmer screening alone, so the database is very small. 
The database is populated from the included excel template in database_tools. 

At present the database import script can only import new data and has not yet been programmed with any functions to update existing records (future update)

## Dependencies and getting up and running 

PneumoKITy is written for Python 3.7+ and is **NOT** compatible with earlier versions of Python. In particular the
method of running the mash subprocess requires python 3.7+ (not 3.6)


PneumoKITy requires the following packages installed before running:
* Mash version 2.3 (or 2.0+) [https://github.com/marbl/Mash](https://github.com/marbl/Mash)
* numpy [http://www.scipy.org/scipylib/download.html](http://www.scipy.org/scipylib/download.html)
* pandas [https://pandas.pydata.org/](https://pandas.pydata.org/)
* SQLite3 [https://www.sqlite.org/index.html](https://www.sqlite.org/index)
* SQLalchemy [https://www.sqlalchemy.org/](https://www.sqlalchemy.org/)

Due to the dependencies PneumoKITy can only be run on Linux based operating systems,
however the software can be run on Windows 10 using the Windows Subsystem for Linux 
[WSL](https://docs.microsoft.com/en-us/windows/wsl/). 
Please note if using conda environments the version of mash installed from Conda (1.X) is NOT Compatible with PneumoKITy. Please use 2.3 (or 2.0+, - 2.3 recommended). 

An easy way to install the dependencies is to use a Python 3 conda or venv environment.  

Install numpy, pandas and SQLalchemy in the environment (SQLite3 is likely to be bundled anyway).

Download Mash 2.3 as a tar file from [here](https://github.com/marbl/Mash/releases/download/v2.3/mash-Linux64-v2.3.tar) for linux or [here](https://github.com/marbl/Mash/releases/download/v2.3/mash-Linux64-v2.3.tar) for OSX. 

Move the mash tar file into a relevant place on your system and untar e.g. `tar -xvf foo.tar`

Now you should be able to run mash, check that it gives the command line help by simply using the mash command. You  will need specify the full path to the mash file - eg. `/home/software/mash-Linux64-v2.2/mash` unless you have fully installed the software or added it to your PATH variable.
If successful you will see the Mash software command line help options. 

For convenience, add the mash folder to your path variable, if successful, then PneumoKITy can be run without the need to specify the MASH location each time. 

Eg: `export PATH="/home/username/mash-Linux64-v2.3:$PATH"`


The above can be added to ~/.profile to preserve the path variable for future sessions. 

Once this is working you should be able to run PneumoKITy as detailed below.

## Running PneumoKITy
#### Mandatory commandline inputs

PneumoKITy accepts 3 input options, two for read input and one for assembly
 input. It mandatory to give at least one of these options. 
 
**Option 1 -i**: a folder containing two fastq files (argument: `-i 
folder_path`)
 
**Option 2 -f**: Specified paths to forward and reverse read files with input 
paths for two fastq files (argument: `-f read1_path read2_path`)

**Option 3 -a**: A specified assembly file (argument: `-a path_to_assembly`)


### Customisable settings (with default behaviour)
 
**-o** (output directory): An output directory can be specified using `-o 
output_dir`. PneumoKITy will place a subfolder `pneumo_capsular_typing` 
within any specified output directory.  If no output dir is specified 
PneumoKITy defaults to using the input directory for either read1 if using 
fastq input or the assembly in this case ensure the input directory is writable or an
error will occur.
 
 
**-t** (threads) Number of threads to use for subprocesses (default = 1) eg: `-t 8`

**-m** (mash): path to mash software. By default PneumoKITy will use the 
command  `mash` which will only work if the mash software is included in the
 `PATH` variable. Otherwise the path to the mash software file location
  **must** be provided eg: `-m path_to_mash/mash`.

**-p** (minpercent): Alternative filter cut-off value for kmer percentage, ie 
percentage of kmer hits to reference. eg:  `-p 80` (default = 90) NB: The software will automatically 
incrementally drop the kmer percentage if no hits are initially obtained - and report serotype with an amber 
RAG status if determined with the dropped back cut off (auto drop min= 70%). 

**-n** (minmulti): minimum, median-multiplicity value cut off (relevant for fastq 
read input only). eg: `-n 4` (default = 10) Used to minimise the kmers present due to 
sequencing errors only.  Default set quite stringently to avoid over sensitive detection
 of mixed samples. Reduce to increase serotype mixture sensitivity. Recommended min 4. 

**-s** (sampleid): Specify a sample ID for output files. If not specified 
PneumoKITy will default to the assembly file name or the fastq file name 
(split on character as specified in -S or on "." default). eg: `-s sample-name`

**-S** (split): Specify a split character to split filename on for sample ID for output files. If not specified 
PneumoKITy will default to using ".". Eg if using `-S _`, filename `sampleid_1.fastq.gz` becomes sampleID `sampleid`
(split on first "_").

**-c** (collate): Specify a folder for PneumoKITy to collate results from the run into a file called "Collated_result_data.csv"
This is useful when running multiple PneumoKITy jobs for a particular project, for example via a queue submission system or Bash loop command. The basic result data will be appended to this file until either the flag is not specified, a different folder is specified or the resulting file is moved or renamed. In rare instances multiple processing MAY result in this file not being writable, and a result beng missed from the collation. The original data files from the run will be saved in their output location.

**-d** (database): path to capsular type variant database (ctvdb). By default this 
will be the ctvdb that is contained within the PneumoKITy program. 


An alternative ctvdb folder can be specified (ADVANCED USERS ONLY). If you wish to use an 
 alternative ctvdb we recommend that this is stored in a separate folder location
 and called using the -d option. To avoid confusion when troubleshooting,
  please do **NOT**  overwrite the included ctvdb and run without specifying the option -d.
 eg: `-d path_to_alternativectvdb

## Example command lines

1. Input folder containing read 1 and read 2, 8 threads, path to mash, custom sample name.

`python pneumokity.py -i my_input_folder -t 8 -m path_to_mash/mash -s my_name`

2. Input read files, path to mash, custom output_dir, collate file folder location

`python pneumokity.py -f path_to_fastq/fastq1 path_to_fastq/fastq2 -o my_output_dir
 -c path_to_collate_folder `

3. Input fastq file paths, custom output directory, path to mash, custom kmer percentage cut off

`python pneumokity.py -f path_to_fastq/fastq1 path_to_fastq/fastq2 -o my_output_dir
 -m path_to_mash/mash -k 75`

4. Input assembly, path to mash, custom initial kmer percentage cut off

`python pneumokity.py -a path_to_assembly/assembly  -m path_to_mash/mash -k 75`

5. Input assembly, path to mash ,collate file folder location

`python pneumokity.py -a path_to_assembly/assembly  -m path_to_mash/mash -c path_to_collate_folder`

6. Input assembly, path to mash ,specified split character for filename

`python pneumokity.py -a path_to_assembly/assembly  -m path_to_mash/mash -S _`


## How PneumoKITy works  

PneumoKITy  uses a kmer based approach to 
screen the read files or input assembly file against the capsular operon 
references (references.fasta) or gene presence absence references is stored as a mash sketch file (references.msh).

In the first stage, the input sequences are screened against a file of the
 capsular operon references (references.msh). The resulting data is analysed for hit
  percentage (kmers in sample / kmers in reference x 100) and then filtered according to
  the cut off values specified.

-p kmer percentage - Percentage of hit kmers against total reference kmers for a given
serotype reference sequence, calculated by PneumoKITy.

-n median multiplicity -the median number of multiples of a given kmer in the dataset,
given in mash screen output. Low kmer multiplicity could be caused by 
sequencing errors or mixed samples - only applicable for input read files if
 assembly files are input this value is automatically set to 1. The default for fastq is 4.


This results in several stage 1 outcome categories and also come with a RAG 
status (see Quality and Error checking):

- `type` - A serotype that can be determined in stage 1 only. Resulted as final result 
when run in normal mode 
- `subtype` - a serotype that determined in stage 1 only but also has subtypes which 
can be determined to add further information (*future update*)
- `variants`  - a serotype that cannot determined in stage 1 only - needs further 
determination against capsular type variant database.
- `mix` - Mixed serotypes are present
- `acapsular` - the kmer percentages against all capsular operon references were 20% and 
suggests that the sample does NOT have a capsular operon present and may be an acapsular 
organism. Check phenotype and species ID.
- `No hits` - No hits were determined but the kmer percentage to the reference operons
 was higher than that that might suggest an acapsular organism. Could be due to serotype
variant or due to poor sequencing quality. 

If the `variants` category is returned in stage 1 and the serotype has presence absence or allele variants
 then PneumoKITy proceeds to stage 2.

**Stage 2** -  only two categories of variants are available in stage 2, these are
gene presence/absence and allele variants. Both of these methods are implemented using MASH in
a similar procedure to described above. Only genogroups 15F_15A and 19A_19AF(variant) are able to fully
determine in stage 2. However some genogroups contain types that can be separated using the kmer screening determinations
eg 12F from 12A_12B_44_46 and 38 from 25F_25A. So depending on the serotype with in a genogroup you
may get either a type specific or genogroup specific result.

Additional reference files have been added to the CTVdb for serotypes 24F and 33F due to the extra specificity of the 
kmer screening method causing many test samples to give low top hits and AMBER calls, due to divergence between the old reference strains
used for the original PneumoCaT reference.fasta and isolates now circulating (in the UK). Once more data has been collected, potentially 
it may be possible to call other serotypes from groups to serotype level due to the greater specificity of the kmer method and increased
match with the new references (eg 24F from B), however additional testing is needed to validate this.

All other serotypes requiring variant analysis will return a genogroup as the methods for determining SNPS and gene function/non-function
variants are not available. A partial gene profile will be present for those where some of the determinations are allele or presence/absence based.


## Quality checks

PneumoKITy is written for use in an accredited laboratory and to aid this,
stores various metrics which are reported in the final 
results report file created at the end of the run.  This includes the 
versions of software used (PneumoKITy itself, Mash etc), paths to input 
files,  information about the reference database used and the cut offs 
specified. 

Outputs from PneumoKITy are automatically assigned a RAG status:

![#4CAF50](https://via.placeholder.com/15/4CAF50/000000?text=+) GREEN: Analysis passed within expected cut-off

![#F39C12](https://via.placeholder.com/15/F39C12/000000?text=+) AMBER: Result obtained but caution advised, check top hit percentages possible variant or low sequence quality.

![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) RED: Analysis failed.

Amber result status is assigned when no-hits occur in stage 1 analysis, the 
kmer percentage cut off is automatically dropped by 10% from the initial cut 
off (so usually reset to 81% unless the user has input a custom kmer percent
 cut off) and analysis is run again. This was implemented to help avoid 
 those annoying situations when a sample would miss just below the cut off as sometimes happened with PneumoCaT1 while also warning the user that something may be wrong with the result. 
If an AMBER result is obtained it could be due to either poor sequence quality, or a variant of the sequence which does not match very well with the reference sequences available in the CTVdb - please check the results. 

Amber result status also occurs in stage 2 for unrecognised variant profiles.

   
RED rag status alerts the user to failure of the serotyping. This could be 
due to an unexpected pattern of results, mixed serotypes or no-hits in the 
analysis.

**IMPORTANT:** Once software is released publically - each time an update is added to the included ctvdb we increment the 
version of the overall PneumoKITy software (eg. from version 2.0 to 2.0.1). 
However if an alternative version of the ctvdb is used it is up to the user to record 
which version is used for their analysis.

## Output Files

PneumoKIty produces several output files. 

`SAMPLEID_serotyping_results.txt` *(All serotypes)*

A text file contaning human-readable formatted information about the run metrics and sample results

`SAMPLEID_quality_system_data.csv` *(All serotypes)*

A csv file containing information about the run metrics (folder and file locations, software versions, cut offs etc)

`SAMPLEID_result_data.csv` *(All serotypes)*

csv file containing final result data from run

`SAMPLEID_stage1_screen.csv` *(All serotypes)*

csv file containing all of the stage1 screen MASH run hits.


`SAMPLEID_GENE_screen.csv` *(Serotypes resulting at stage 2 only)*

csv files containing MASH screen run hits information for the relevant variant genes.


## Interpretation of results

**`predicted serotype`** -  This is the predicted PHENOTYPICAL type of the organism if characterised using the commercially available [SSI Diagnostica serotyping sera](https://www.ssidiagnostica.com/antisera/pneumococcus-antisera/). However, the organism could have a specific underlying genetic type as in the case of 23B1 and 19A/F for example. 

Some previously described important genetic subtypes **are** represented in the ctvdb and can be determined by looking at the stage 1 hits. Eg. in the case of the variant 19A/F isolates that have the [genetic background of a 19A but produce a 19F capsular polysaccharide](https://pubmed.ncbi.nlm.nih.gov/19439547/) , the predicted serotype result would be 19F, but the stage 1 result is recorded as 19AF and the result is determined from 19A in stage 2 via wzy analysis, whereas a standard 19F isolate would be determine in stage 1 analysis alone by hit to the 19F reference. For 23B1 the top hit in stage 1 will be the 23B1 reference. At present there is no specific "genetic subtype" result field implemented (this is planned in a future version), so the user needs to look at the stage 1 hits to determine these.

New genetic types are being discovered all the time and it is not possible for us to keep up with them, however we hope that the outputs obtained from PneumoKITy will help the user to determine if their isolate may be a novel genetic serotype for which a reference is not available in the ctvdb, and can then be further investigated.

if the sample has failed to hit a serotype a description of the result is output in this field.

**`top hits`** - this is a list of the top 5 hits in the stage 1 analysis, with their MASH hit kmer percentage score in order highest to lowest. For serogroups 6 and 19, subtype references from [Elberse et al](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0025018_) are included in the stage 1 references (eg 19F-III, 19A-IV).  Subtypes are not yet interpreted in the program but the closest hit subtype can be determined by reference to the top hit.

**`max percent`** - the kmer percentage of the maximum hit in the MASH screen stage 1 analysis.

**`folder`** - the folder location within the \ctvdb folder used for the stage 2 analysis/

**`stage 1 result`** - the outcome of stage 1 analysis.

**`mix_mm`** - the median multiplicity value for the individual hits (with percent >cut off) in a 
serotype mix. This value reflects an estimation of the relative abundance of the kmers in the mix. 
- Only applicable if input is fastq and mixed serotypes are found

**`stage 2 varids`** - the variant ID (keys) in the ctvdb sql database of the variants used for determination in stage2.

**`stage 2 hits`** - the variant genes and results of stage 2 analysis, eg for those variants determined using MASH screen (gene presence/absence and allele) the result will be a hit % of kmers from sample vs kmers in the gene. 

**`stage 2 result`** - interpreted version f stage 1, eg hit variant determined (eg detected, not detected)

**`rag status`** - the overall quality status (traffic light system) of the run as described above.







