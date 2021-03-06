# GRiD
Growth Rate Index (GRiD) measures bacterial growth rate from reference genomes (including draft quality genomes) and metagenomic bins at ultra-low sequencing coverage (> 0.2x). 

GRiD algorithm consists of two modules;

1. < single > - which is applicable for growth analysis involving a single reference genome
2. < multiplex > - for the high-throughput growth analysis of all identified bacteria in a sample. Prior knowledge of microbial composition is not required. To use this module, download the GRiD database, consisting of 32,819 representative bacteria genomes, from **ftp://ftp.jax.org/ohlab/Index/**   

# INSTALLATION
The easiest way to install GRiD is through miniconda which resolves all required dependencies. 

1.    If you do not have anaconda or miniconda already installed, download miniconda https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh and run the install script. Reload .bashrc environment `source ~/.bashrc`
    
    -- Set up channels --
    
    conda config --add channels r
    conda config --add channels defaults
    conda config --add channels conda-forge
    conda config --add channels bioconda
          
2.    Install GRiD

`conda install grid=1.0.6`

**NOTE: Ensure you include the GRiD version as above (grid=1.0.6) during conda installation**

**It is highly recommended to run the example test to ensure proper installation before running GRiD on your dataset. You do not need to have downloaded the GRiD database to run the test (see "Example test" below)**.


# USAGE

    grid -h                     Display this help message
    grid single <options>       GRiD using a single genome
    grid multiplex <options>    GRiD high throughput

    grid single <options>
    <options>
    -r      Reads directory (single end reads)
    -o      Output directory
    -g      Reference genome (fasta)
    -l      Path to file listing a subset of reads
            for analysis [default = analyze all samples in reads directory]
    -n INT  Number of threads for bowtie mapping (default 1)
    -h      Display this message

    grid multiplex <options>
    <options>
    -r         Reads directory (single end reads)
    -o         Output directory
    -d         GRiD database directory
    -c  FLOAT  Coverage cutoff (>= 0.2) [default 1]
    -p         Enable reassignment of ambiguous reads using Pathoscope2
    -t  INT    Theta prior for reads reassignment [default 0]. Requires the -p flag
    -l         Path to file listing a subset of reads
               for analysis [default = analyze all samples in reads directory]
    -m         merge output tables into a single matrix file
    -n  INT    Number of threads for bowtie mapping (default 1)
    -h         Display this message


**NOTE: Sample reads must be in single-end format**. If reads are only available in paired-end format, use either of the mate pairs, or concatenate both pairs into a single fastq file. In addition, reads must have the .fastq extension (and not .fq). Using either modules, the default is to analyze all samples present in the reads directory. However, analysis can be restricted to a subset of samples by using the -l flag and specifying a file that lists the subset of samples.    

For the 'multiplex' module, reads mapping to multiple genomes are reassigned using Pathoscope 2 when the -p flag is set. The degree to which reads are reassigned is set by the -t (theta prior) flag. The theta prior value represents the number of non-unique reads that are not subject to reassignment. Finally, when the coverage cutoff (-c flag) is set below 1, only genomes with fragmentation levels below 90 fragments/Mbp are analyzed (see xxx et al. for more details). **Note that to use the 'multiplex' module, you must have downloaded the GRiD database from ftp://ftp.jax.org/ohlab/Index/. However, you do not need the database to run the example test**.

# OUTPUT
`single module` - two output files are generated
- A plot (.pdf) showing coverage information across the genome 
- A table of results (.txt) displaying growth rate (GRiD), 95% confidence interval, unrefined GRiD value, species heterogeneity, genome coverage, *dnaA/ori* ratio, and *ter/dif* ratio. Species heterogeneity is a metric estimating the degree to which closely related strains/species contribute to variance in growth predictions (range between 0 - 1 where 0 indicate no heterogeneity). In most bacteria genomes, *dnaA* is located in close proximity to the *ori* whereas replication typically terminates at/near *dif* sequence. Thus, the closer *dnaA/ori* and *ter/dif* ratios are to one, the more likely the accuracy of GRiD scores.  

`multiplex module` - two output files are generated per sample
- A heatmap (.pdf), displaying growth rate (GRiD) from genomes above the coverage cutoff with hierachical clustering. 
- A table of results (.txt) displaying growth rate (GRiD) of genomes above the coverage cutoff, unrefined GRiD value, species heterogeneity, and genome coverage. If -m flag is set, all tables will be merged into a single matrix file called "merged_table.txt".


# Example test
The test sample contain reads from *Staphylococcus epidermids*, *Lactobacillus gasseri*, and *Campylobacter upsaliensis*, each with coverage of ~ 0.5. Download the GRiD folder and run the test as shown below. 

**NOTE: Ensure GRiD-1.0.6 was installed via conda. Please see installation instructions**  

`wget https://github.com/ohlab/GRiD/archive/1.0.6.tar.gz`

`tar xvf 1.0.6.tar.gz`

`cd GRiD-1.0.6/test`

`chmod +x ../grid`

`../grid single -r . -g S_epidermidis.LRKNS118.fna`

`../grid multiplex -r . -d . -p -c 0.2`

For each module, output files (a pdf and a text file) are generated in the test folder.


# Updating GRiD database 
The database can be updated with metagenomic bins or newly sequenced bacterial genomes by running the update_database script (requires bowtie2).
 

    update_database <options>
    <options>
    -d      GRiD database directory
    -g      Bacterial genomes directory
    -n      Name for new database
    -l      Path to file listing specific genomes
            for inclusion in database [default = include all genomes in directory]
    -h      Display this message

NOTE: bacteria genomes must be in fasta format and must have either .fasta, .fa or .fna extensions.


# DEPENDENCIES
- R (tested using v 3.2.3) 
    - Required R libraries - 
    dplyr,
    getopt,
    ggplot2,
    gsubfn,
    gplots
    
- bowtie2 (tested using v 2.3.1)
- seqtk (tested using v 1.0-r31)
- samtools (tested using v 1.5)
- bedtools (tested using v 2.26.0)
- bamtools (tested using v 1.0.2)
- blast (tested using v 2.6.0)
- Pathoscope2    
