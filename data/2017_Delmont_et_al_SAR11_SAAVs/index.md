---
layout: page
title: Single amino acid variants in SAR11
modified: 2017-04-20
excerpt: "A complete workflow behind the manuscript 'The large-scale biogeography of amino acid variants within a single SAR11 population is governed by natural selection' by Delmont et al"
comments: true

---

{% include _toc.html %}

{% capture images %}{{site.url}}/data/2017_Delmont_et_al_SAR11_SAAVs/images{% endcapture %}

This document describes the reproducible bioinformatics workflow for our study titled "*The large scale biogeography of amino acid variants within a single SAR11 population is governed by natural selection*". Here you will find program names and exact parameters we used throughout every step of the analysis of SAR11 genomes and metagenomes from the TARA Oceans and Ocean Sampling Day projects, which relied predominantly on the open-source analysis platform [anvi’o](http://merenlab.org/software/anvio) (Eren et al., 2015).

{:.notice}
All anvi'o analyses in this document are performed using the anvi'o version `v2.4.0`. Please see [the installation notes]({% post_url anvio/2016-06-26-installation-v2 %}) to download the appropriate version through PyPI, Docker, or GitHub.

{:.notice}
The URL [http://www.biorxiv.org/content/early/2017/07/31/170639](http://www.biorxiv.org/content/early/2017/07/31/170639){:target="_blank"} serves the pre-print of the study described in this document.

<!--

{:.notice}
The URL [http://merenlab.org/data/#XXX](http://merenlab.org/data/#XXX){:target="_blank"} serves all public data items used and produced by this study.

-->

{:.notice}
The URL [http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/](http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/){:target="_blank"} serves the most up-to-date version of this document.


<div class="extra-info" markdown="1">

<span class="extra-info-header">Summary</span>

In this study,

* We characterized the metapangenome of 21 SAR11 isolate genomes using metagenomes from the TARA Oceans and Ocean Sampling Day projects,

* Identified single-nucleotide variants (SNVs) and single-amino acid variants (SAAVs) within a single remarkably abundant and widespread SAR11 population,

* Estimated distances between metagenomes based on SAAVs using Deep Learning,

* Linked SAAVs to predicted tertiary structures of S-LLPA proteins.

Sections in this document will detail all the steps of downloading and processing SAR11 genomes and metagenomes, mapping metagenomic reads onto the SAR11 genomes, identifying and processing genomic variability to explore the amino acid diversification traits of a single population.

</div>


Please feel free to leave a comment, or send an e-mail to [us]({{ site.url }}/people/) if you have any questions.


## Setting the stage

This section explains how to download a set of 21 SAR11 genomes and download and quality filter short metagenomic reads from the TARA Oceans project (Sunagawa et al., 2015) and Ocean Sampling Day project (Kopf et al., 2015).

{:.notice}
The TARA Oceans metagenomes we analyzed are publicly available through the European Bioinformatics Institute (EBI) repository and NCBI under project IDs `ERP001736` and `PRJEB1787`, respectively.

### Downloading the 21 SAR11 cultivar genomes

You can get a copy of the FASTA file containing all 21 SAR11 cultivar genomes into your work directory using this command line: 

``` bash
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/SAR11-isolates.fa.gz
gzip -d SAR11-isolates.fa.gz
``` 

### Downloading the TARA Oceans and Ocean Sampling Day metagenomes

[This file](http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/ftp-links-for-raw-data-files.txt){:target="_blank"} contains URLs for FTP access for each raw data file for 103 samples, and you can get a copy of it into your work directory, 

``` bash
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/ftp-links-for-raw-data-files.txt
``` 

and download each of the raw sequencing data file from the EMBL servers this way:

``` bash
for url in `cat ftp-links-for-raw-data-files.txt`
do
    wget $url
done
```

This may take quite a while.


### Defining metagenomic sets, setting sample names, and linking those with the raw data

We defined 12 'metagenomic sets' for geographically bound locations TARA Oceans samples originated from, consistent with our previous [work flow to reconstruct ~1,000 population genomes](http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/). We defined an additional metagenomic set for the 10 Ocean Sampling Day metagenomes that cover high-latitudes of the north hemisphere. 

We tailored our sample naming schema for convenience and reproducibility. [This file](http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/sets.txt){:target="_blank"}{:target="_blank"} contains the three-letter identifiers for each of the 13 metagenomic sets, which will become prefixes for each sample name for direct access to all samples from a given metagenomic set. This file will be referred to as `sets.txt` throughout the document, and you can get a copy of this file into your work directory:

``` bash
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/sets.txt
``` 

We used these three-letter prefixes to name each of the 103 samples, and to associate them with metagenomic sets with which they were affiliated. [This file](http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/samples.txt){:target="_blank"} contains sample names, and explains which raw data files are associated with each sample. It will be referred to as `samples.txt` throughout the document, and you can get a copy of this file into your work directory:

``` bash
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/samples.txt
```

TARA samples file should look like this:

``` bash
$ wc -l samples.txt
      103 samples.txt
$ head samples.txt
sample     r1                            r2
MED_18_05M ERR599140_1.gz,ERR598993_1.gz ERR599140_2.gz,ERR598993_2.gz
MED_23_05M ERR315858_1.gz,ERR315861_1.gz ERR315858_2.gz,ERR315861_2.gz
MED_25_05M ERR599043_1.gz,ERR598951_1.gz ERR599043_2.gz,ERR598951_2.gz
MED_30_05M ERR315862_1.gz,ERR315863_1.gz ERR315862_2.gz,ERR315863_2.gz
RED_31_05M ERR599106_1.gz,ERR598969_1.gz ERR599106_2.gz,ERR598969_2.gz
RED_32_05M ERR599041_1.gz,ERR599116_1.gz ERR599041_2.gz,ERR599116_2.gz
RED_33_05M ERR599049_1.gz,ERR599134_1.gz ERR599049_2.gz,ERR599134_2.gz
RED_34_05M ERR598959_1.gz,ERR598991_1.gz ERR598959_2.gz,ERR598991_2.gz
ION_36_05M ERR599143_1.gz,ERR598966_1.gz ERR599143_2.gz,ERR598966_2.gz
```

This file above also represents the standard input for `illumina-utils` library for processing the raw metagenomic data for each sample. 


### Quality-filtering of raw reads 

We used the [illumina-utils library](https://github.com/meren/illumina-utils){:target="_blank"} `v1.4.1` (Eren et al., 2013) to remove noise from raw reads prior to mapping.

To produce the configuration files [illumina utils requires](https://github.com/merenlab/illumina-utils#config-file-format){:target="_blank"}, we first run the program `iu-gen-configs` on the file `samples.txt`:

``` bash
iu-gen-configs samples.txt
```

This step should generate 103 config files with the extension of `.ini` in the working directory. 

To perform the quality filtering on samples config files describe, we used the program `iu-filter-quality-minoche` with default parameters, which implements the approach detailed in Minoche et al. (Minoche et al., 2011):

``` bash
for sample in `awk '{print $1}' samples.txt`
do
    if [ "$sample" == "sample" ]; then continue; fi
    iu-filter-quality-minoche $sample.ini --ignore-deflines
done
```

This step generated files that summarize the quality filtering results for each sample. Here is an example for one:

``` bash
$ cat ASW_82_40M-STATS.txt
number of pairs analyzed      : 222110286
total pairs passed            : 204045433 (%91.87 of all pairs)
total pairs failed            : 18064853 (%8.13 of all pairs)
  pairs failed due to pair_1  : 3237955 (%17.92 of all failed pairs)
  pairs failed due to pair_2  : 8714230 (%48.24 of all failed pairs)
  pairs failed due to both    : 6112668 (%33.84 of all failed pairs)
  FAILED_REASON_N             : 302725 (%1.68 of all failed pairs)
  FAILED_REASON_C33           : 17762128 (%98.32 of all failed pairs)
```

This step also generated FASTQ files for each sample that contain quality-filtered reads for downstream analyses. The resulting `*-QUALITY_PASSED_R*.fastq.gz` files in the work directory represent the main inputs for mapping.


## Mapping metagenomic reads to SAR11 genomes

This section explains various steps to characterize the occurrence of each SAR11 isolate genome in metagenomes.

### Generating an anvi'o CONTIGS database

We used the program `anvi-gen-contigs-database` with default parameters to profile all contigs for SAR11 isolate genomes, and generate an anvi’o contigs database that stores for each contig the DNA sequence, GC-content, tetranucleotide frequency, and open reading frames as Prodigal `v2.6.3` (Hyatt et al., 2010) identifies them:

``` bash
anvi-gen-contigs-database -f SAR11-isolates.fa \
                          -o SAR11-CONTIGS.db
```

{:.notice}
[doi:10.5281/zenodo.835218](https://doi.org/10.5281/zenodo.835218){:target="_blank"} serves the resulting contigs database.


### Identifying single-copy core genes

We used the program `anvi-run-hmms` to identify archaeal and bacterial single-copy core genes (SCGs) in SAR11 isolate contigs:

``` bash
anvi-run-hmms -c SAR11-CONTIGS.db \
              --num-threads 20
```

This step uses HMMER `v3.1b2` (Eddy, 2011) to match genes to [previously determined HMM profiles](https://github.com/merenlab/anvio/tree/v2.4.0/anvio/data/hmm){:target="_blank"} for bacterial and archaeal SCGs by [Campbell et al](http://www.pnas.org/content/110/14/5540.short){:target="_blank"} and [Rinke et al](http://www.nature.com/nature/journal/v499/n7459/full/nature12352.html){:target="_blank"}.


### Inferring functions with COGs

We used the program `anvi-run-ncbi-cogs` to search the NCBI's Clusters of Orthologous Groups (COGs) to assign functions to genes in our CONTIGS database for SAR11 isolates:

``` bash
anvi-run-ncbi-cogs -c SAR11-CONTIGS.db \
                   --num-threads 20
```

Results of this step is used to display protein clusters with known functions in the interactive interface for our downstream pangenomic analysis.


### Recruitment of metagenomic reads

The recruitment of metagenomic reads is commonly used to assess the coverage of scaffolds, which provides the essential information that is employed by anvi'o to assess the relative distribution of genomes across metagenomes and characterize single-nucleotide (SNVs) and single-amino acid variants (SAAVs).

We mapped short reads from the 103 TARA and OSD metagenomes onto the scaffolds contained in `SAR11-isolates.fa` using Bowtie2 `v2.0.5` (Langmead and Salzberg, 2012). We stored the recruited reads as BAM files using samtools (Li et al., 2009).

We first built a Bowtie2 database for `SAR11-isolates.fa `:

``` bash
bowtie2-build SAR11-isolates.fa SAR11-isolates
```

We then mapped each metagenome against the scaffolds contained in `SAR11-isolates.fa`:

``` bash
for sample in `awk '{print $1}' samples.txt`
do
    if [ "$sample" == "sample" ]; then continue; fi    
    # do the bowtie mapping to get the SAM file:
    bowtie2 --threads 20 \
            -x SAR11-isolates \
            -1 $sample-QUALITY_PASSED_R1.fastq.gz \
            -2 $sample-QUALITY_PASSED_R2.fastq.gz \
            --no-unal \
            -S $sample.sam
    
    # covert the resulting SAM file to a BAM file:
    samtools view -F 4 -bS $sample.sam > $sample-RAW.bam

    # sort and index the BAM file:
    samtools sort $sample-RAW.bam -o $sample.bam
    samtools index $sample.bam

    # remove temporary files:
    rm $sample.sam $sample-RAW.bam
done
```

This process has resulted in 103 sorted and indexed BAM files that describe the mapping of more than 30 billion short reads to scaffolds contained in the FASTA file `SAR11-isolates.fa`.


### Profiling the mapping results with anvi'o 

After recruiting metagenomic short reads using scaffolds stored in the anvi'o CONTIGS database for SAR11 isolates, we used the program `anvi-profile` to process the BAM files and to generate anvi'o PROFILE databases that contain the coverage and detection statistics of each SAR11 scaffold in a given metagenome:

``` bash
for sample in `awk '{print $1}' samples.txt`
do
    if [ "$sample" == "sample" ]; then continue; fi
    
    anvi-profile -c SAR11-CONTIGS.db \
                 -i $sample.bam \
                 --profile-AA-frequencies \
                 --num-threads 16 \
                 -o $sample
done
```

{:.notice}
The profiling of single-nucleotide variants (SNVs) and single-amino acid variants (SAAVs) is performed during this step.

{:.notice}
Please read the following article for parallelization of anvi'o profiling (details of which can be important to consider especially if you are planning to send it to a cluster): [The new anvi'o BAM profiler]({% post_url anvio/2017-03-07-the-new-anvio-profiler %}).

### Generating a merged anvi'o profile

Once the individual PROFILE databases were generated, we used the program `anvi-merge` to generate a merged anvi'o profile database:


``` bash
    anvi-merge */PROFILE.db -o SAR11-MERGED -c SAR11-CONTIGS.db 
done
```

The resulting profile database describes the coverage and detection statistics, as well as SNVs and SAAVs for each scaffold across all 103 metagenomes.

{:.notice}
[doi:10.5281/zenodo.835218](https://doi.org/10.5281/zenodo.835218){:target="_blank"} serves the merged anvi'o profile.


### Generating a genomic collection

At this point anvi'o still doesn't know how to link scaffolds to each isolate genome. In anvi'o, this kind of knowledge is maintained through 'collections'. In order to link scaffolds to genomes of origin, we used the program `anvi-import-collection` to create an anvi'o collection in our merged profile database.

We first generated a 2-columns, TAB-delimited collections file:

``` bash
for split_name in `sqlite3 SAR11-CONTIGS.db 'select split from splits_basic_info;'`
do
    # in this loop $split_name goes through names like this: HIMB058_Contig_0001_split_00001,
    # HIMB058_Contig_0001_split_00002, HIMB058_Contig_0001_split_00003, HIMB058_Contig_0001_split_00003...; so we can extract
    # the genome name it belongs to:
    MAG=`echo $split_name | awk 'BEGIN{FS="_"}{print $1}'`
    
    # print it out with a TAB character
    echo -e "$split_name\t$MAG"
done > SAR11-GENOME-COLLECTION.txt
```

The resulting file `SAR11-GENOME-COLLECTION.txt` looked like this, and listed each scaffold name in the first column, and which genome they belonged in the second:

``` bash
$ head SAR11-GENOME-COLLECTION.txt
HIMB058_Contig_0001_split_00001 HIMB058
HIMB058_Contig_0001_split_00002 HIMB058
HIMB058_Contig_0001_split_00003 HIMB058
HIMB058_Contig_0001_split_00004 HIMB058
HIMB058_Contig_0002_split_00001 HIMB058
HIMB058_Contig_0002_split_00002 HIMB058
HIMB058_Contig_0002_split_00003 HIMB058
HIMB058_Contig_0003_split_00001 HIMB058
HIMB058_Contig_0003_split_00002 HIMB058
HIMB058_Contig_0003_split_00003 HIMB058
(...)
```

We then used the program `anvi-import-collection` to import this collection into the anvi'o profile database by naming this collection `Genomes`:

``` bash
anvi-import-collection SAR11-GENOME-COLLECTION.txt \
                       -c SAR11-CONTIGS.db \
                       -p SAR11-MERGED/PROFILE.db \
                       -C Genomes
```

### Summarizing the collection 'Genomes'

We used `anvi-summarize` to create a static HTML summary of the profiling results:

``` bash
anvi-summarize -c SAR11-CONTIGS.db \
               -p SAR11-MERGED/PROFILE.db \
               -C Genomes \
               --init-gene-coverages \
               -o SAR11-SUMMARY
```

This summary contains all key information regarding the occurrence of isolate genomes in metagenomes (i.e., their coverage at the gene, scaffold, and genome-level, functional annotations per gene, etc.)

{:.notice}
[doi:10.6084/m9.figshare.5248453](https://doi.org/10.6084/m9.figshare.5248453){:target="_blank"} serves the summary output. This is a static HTML web page and you can display the output without an anvi'o installation.

## SAR11 metapangenome

We used the anvi'o programs `anvi-gen-genomes-storage`, `anvi-pan-genome`, and `anvi-display-pan` to create and visualize the SAR11 metapangenome.

### Computing the pangenome

We first created the file `internal-genomes.txt` that connects genome IDs to the CONTIGS and PROFILE databases ([details the anvi'o pangenomic workflow]({% post_url anvio/2016-11-08-pangenomics-v2 %})). This file can be downloaded using this command:

``` bash
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/internal-genomes.txt
```

And here is how it looks like:

``` bash
$ head internal-genomes.txt
name	bin_id	collection_id	profile_db_path	contigs_db_path
HIMB058	HIMB058	Genomes	SAR11-MERGED/PROFILE.db	SAR11-CONTIGS.db
HIMB083	HIMB083	Genomes	SAR11-MERGED/PROFILE.db	SAR11-CONTIGS.db
HIMB114	HIMB114	Genomes	SAR11-MERGED/PROFILE.db	SAR11-CONTIGS.db
HIMB122	HIMB122	Genomes	SAR11-MERGED/PROFILE.db	SAR11-CONTIGS.db
HIMB1321	HIMB1321	Genomes	SAR11-MERGED/PROFILE.db	SAR11-CONTIGS.db
HIMB140	HIMB140	Genomes	SAR11-MERGED/PROFILE.db	SAR11-CONTIGS.db
HIMB4	HIMB4	Genomes	SAR11-MERGED/PROFILE.db	SAR11-CONTIGS.db
HIMB5	HIMB5	Genomes	SAR11-MERGED/PROFILE.db	SAR11-CONTIGS.db
```

We generated a genomes storage, 

``` bash
anvi-gen-genomes-storage -i internal-genomes.txt \
                         -o SAR11-PAN-GENOMES.h5
```

Computed the pangenome,

``` bash
anvi-pan-genome -g SAR11-PAN-GENOMES.h5 \
                --use-ncbi-blast \
                --minbit 0.5 \
                --mcl-inflation 2 \
                -J SAR11-METAPANGENOME \
                -o SAR11-METAPANGENOME -T 20
```

and visualized it:

``` bash
anvi-display-pan -p SAR11-PAN-PAN.db \
                 -s SAR11-PAN-SAMPLES.db \
                 -g SAR11-GENOMES.h5
```

So far so good? Good.

From the interface, we selected our bins of interest (protein clusters detected in all 21 genomes ("core-pangenome") an those characteristic to HIMB83 ("HIMB83")), saved them as a collection named `default`. We then summarized the SAR11 pangenome using the program `anvi-summarize`:

``` bash
anvi-summarize -p SAR11-PAN-PAN.db \
               -g SAR11-GENOMES.h5 \
               -C default \
               -o SAR11-PAN-SUMMARY
```

The resulting summary folder contains a file that links each gene to protein clusters, genomes, functions, and bins selected from the interface.

### Linking the pangenome to the environment

From the anvi'o metagenomic summary output described in the previous section, we determined the relative distribution of each genome across the 103 metagenomes. We created a text file, `SAR11-PAN-samples-information.txt` (avilable [here]({{ site.url }}/data/2017_Delmont_et_al_SAR11_SAAVs/files/SAR11-PAN-samples-information.txt)) to link the pangenome to the environment using genomic coverage values across samples, as well as to display other information such as gneme lengths and clade information. From this file we generated an anvi'o [samples database](http://merenlab.org/2015/11/10/samples-db/){:target="_blank"}, `SAR11-PAN-SAMPLES.db`. In addition, we used the summary output of the SAR11 pangenome to identify protein clusters containing a list of HIMB83 genes of interest (the 799 core S-LLPA genes), and created the file `S-LLPA-CORE-GENES.txt` (avilable [here]({{ site.url }}/data/2017_Delmont_et_al_SAR11_SAAVs/files/S-LLPA-CORE-GENES.txt)), the contents of which looked like this:

``` bash
$ head S-LLPA-CORE-GENES.txt
samples	S-LLPA_core_genes
PC_00000070	Yes
PC_00000046	Yes
PC_00000072	Yes
PC_00000079	No
PC_00000252	Yes
PC_00000176	Yes
PC_00000087	Yes
PC_00000043	Yes
PC_00000112	Yes
```

Finally, with all these additions, we visualized the SAR11 metapangenome:

``` bash
anvi-display-pan -p SAR11-PAN-PAN.db \
                 -s SAR11-PAN-SAMPLES.db \
                 -g SAR11-GENOMES.h5 \
                 -A S-LLPA-CORE-GENES.txt
```

This resulting display is publicly available on the anvi-server just so you can play with it interactively in case you read it all the way here:

[https://anvi-server.org/merenlab/SAR11_metapangenome](https://anvi-server.org/merenlab/SAR11_metapangenome){:target="_blank"}

{:.notice}
[doi:10.6084/m9.figshare.5248459](https://doi.org/10.6084/m9.figshare.5248459){:target="_blank"} serves the anvi'o files for the SAR11 metapangenome.


## Creating a self-contained profile for HIMB83

The 21 SAR11 genomes recruited more than one billion metagenomics reads, however we determined that one of these isolate genomes, HIMB83, was remarkably abundant and widespread across metagenomes and was of particular interest to study the genomic variability of a single population across geographies. To isolate HIMB83 from the rest of the data we used `anvi-split`, and created a self-contained anvi'o profile database for this genome.

Briefly, the program `anvi-split` creates a PROFILE subset of the merged PROFILE database so genomes of interest can be downloaded independently, and visualized interactively:

``` bash
anvi-split -c SAR11-CONTIGS.db \
           -p SAR11-MERGED/PROFILE.db \
           -C Genomes \
           -b HIMB083 \
           -o SAR11-SPLIT-GENOMES
```

The resulting directory `SAR11-SPLIT-GENOMES` contains a subdirectory for HIMB83:

``` bash
$ ls SAR11-SPLIT-GENOMES | head
HIMB083
$ ls NON-REDUNDANT-MAGs-SPLIT/HIMB083
AUXILIARY-DATA.h5.gz  CONTIGS.db  CONTIGS.h5  PROFILE.db
```

It is then possible to invoke the interactive interface and inspect reads this genome recruited across metagenomes using this command line:

``` bash
anvi-interactive -c NON-REDUNDANT-MAGs-SPLIT/HIMB083/CONTIGS.db \
                 -p NON-REDUNDANT-MAGs-SPLIT/HIMB083/PROFILE.db
```

We finally run the summary of this bin;

``` bash
anvi-summarize -p NON-REDUNDANT-MAGs-SPLIT/HIMB083/PROFILE.db \
               -c NON-REDUNDANT-MAGs-SPLIT/HIMB083/CONTIGS.db \
               -C DEFAULT \
               --init-gene-coverages \
               -o NON-REDUNDANT-MAGs-SPLIT/HIMB083/SUMMARY
```

{:.notice}
[doi:10.6084/m9.figshare.5248435](https://doi.org/10.6084/m9.figshare.5248435){:target="_blank"} serves the self-contained anvi'o profile for HIMB83 across all metagenomes.


## Generating genomic variation data for HIMB83

To explore the genomic variability of S-LLPA, the SAR11 population we could access through HIMB83, we characterized SNVs and SAAVs for a set of HIMB83 genes across across metagenomes. Both for SNVs and SAAVs, we only considered positions of nucletides or codons that met a minimum coverage expectation. Controlling the minimum coverage of nucleotide or codon positions across metagenomes improves the confidence in variability analyses. 

`anvi-summary` allowed us to identity a list of 74 metagenomes where the mean coverage of HIMB83 was `>50x`. We then manually identified 799 HIMB83 genes that systematically occurred in all 74 metagenomes (i.e. genes that were detected when HIMB83 was detected). Please see the methods section in [our study](http://www.biorxiv.org/content/early/2017/07/31/170639){:target="_blank"} for a more detailed description of these steps. But just to give a visual idea here, this shows the HIMB83 genes that systemmatically detected across metagenomes: 

[![SAR11]({{images}}/s-llpa-core.png)]({{images}}/s-llpa-core.png){:.center-img .width-70}

The files `metagenomes-of-interest.txt` and `core-S-LLPA-genes.txt` contain the names of of 74 metagenomes and gene caller id's for 799 core genes, respectively. You can download these files into your work directory the following way:

``` bash
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/metagenomes-of-interest.txt
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/core-S-LLPA-genes.txt
```

We generated three files to report the variability.

First we generated a report of SNVs in 799 genes across 74 metagenomes with a minimum departure from consensus value of 1%, and minimum nucleotide coverage across metagenomes value of 20x:

``` bash
anvi-gen-variability-profile -c SAR11-CONTIGS.db \
                             -p SAR11-MERGED/PROFILE.db \
                             -C Genomes \
                             -b HIMB083 \
                             --samples-of-interest metagenomes-of-interest.txt \
                             --genes-of-interest core-S-LLPA-genes.txt \
                             --min-coverage-in-each-sample 20 \
                             --engine NT \
                             --quince-mode \
                             --min-departure-from-consensus 0.01 \
                             -o S-LLPA_SNVs_20x_1percent_departure.txt
```

Then we generated another report of SNVs in 799 genes across 74 metagenomes with a minimum departure from consensus value of 10%, and minimum nucleotide coverage across metagenomes value of 20x:

``` bash
anvi-gen-variability-profile -c SAR11-CONTIGS.db \
                             -p SAR11-MERGED/PROFILE.db \
                             -C Genomes -b HIMB083 \
                             --samples-of-interest metagenomes-of-interest.txt \
                             --genes-of-interest core-S-LLPA-genes.txt \
                             --min-coverage-in-each-sample 20 \
                             --engine NT \
                             --quince-mode \
                             --min-departure-from-consensus 0.1 \
                             -o S-LLPA_SNVs_20x_10percent_departure.txt
```

And finally we generated a report of SAAVs for 799 genes across 74 metagenomes with a minimum departure from consensus value of 10%, and a minimum codon coverage across metagenomes of 20x:

``` bash
anvi-gen-variability-profile -c SAR11-CONTIGS.db \
                             -p SAR11-MERGED/PROFILE.db \
                             -C Genomes \
                             -b HIMB083 \
                             --samples-of-interest metagenomes-of-interest.txt \
                             --genes-of-interest core-S-LLPA-genes.txt \
                             --min-coverage-in-each-sample 20 \
                             --engine AA \
                             --quince-mode \
                             --min-departure-from-consensus 0.1 \
                             -o S-LLPA_SAAVs_20x_10percent_departure.txt
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

These reports were the key data to compare the differential occurrence of variants across metagenomes to study the evolutionary forces operating on this population at large scale.

{:.notice}
[doi:10.6084/m9.figshare.5248447](https://doi.org/10.6084/m9.figshare.5248447){:target="_blank"} serves the anvi'o variability tables described in this section.

## Application of Deep Learning to SAAVs

This section describes our use of deep learning to estimate distances between TARA metagnomes based on SAAVs identified in core S-LLPA genes. We used the resulting distance matrix to identify the main groups, and proteotypes displayed in Figure 3 in our study.

### Setting the stage

Although it uses the SAAVs reported in the previous section, this part of our analysis is outside of anvi'o. To set the stage for this analysis, we first will create a Python 2.7 virtual environment and do some installation and configuration work. You can follow these steps to get everything ready:

``` bash
# create a new python 2.7 virtual environment
virtualenv -p python2.7 ~/virtual-envs/LALNets

# activate it
source ~/virtual-envs/LALNets/bin/activate

# install tensorflow and Ozsel's LALNets library (while
# you're on it install ipython, too)
pip install tensorflow
pip install git+https://github.com/ozcell/LALNets.git
pip install ipython

# setup keras with theano backend
mkdir -p ~/.keras
cat << EOF >> ~/.keras/keras.json
{
    "image_dim_ordering": "tf",
    "epsilon": 1e-07,
    "floatx": "float32",
    "backend": "theano"
}
EOF
```

The rest of this section does two things: explaining the approach, and linking those explanations to Python code snippets, which you can run in a Python terminal (type `ipython` to start one). Alternatively, all steps are summarized in a single Python program at the end of this section, which you can run after making sure the path to the SAAVs table is correct. We are planning to implement this approach in anvi'o to streamline the application of this algorithm, but until then, you can use the following recipe.

These methods are adopted by our collaborators [Ozsel Kilinc](https://www.linkedin.com/in/%C3%B6zsel-k%C4%B1l%C4%B1n%C3%A7-44b01092/){:target="_blank"} and [Ismail Uysal](https://www.researchgate.net/profile/Ismail_Uysal3){:target="_blank"}. We thank them for their help with these analyses and for their patience :)

### The approach

This approach builds upon a previous study on learning of latent annotations on neural networks using [auto-clustering output layer](https://arxiv.org/abs/1702.08648){:target="_blank"} (ACOL) when a coarse level of supervision is available for all observations (i.e. parent-class labels), but the model has to learn a deeper level of latent annotations (i.e. sub-classes, under each one of parents).

ACOL is a novel output layer modification for deep neural networks to allow simultaneous supervised classification (per provided parent-classes) and unsupervised clustering (within each parent) where clustering is performed with a recently proposed [Graph-based Activity Regularization](https://arxiv.org/abs/1705.07219){:target="_blank"} (GAR) technique. More specifically, as ACOL duplicates the softmax nodes at the output layer for each class, GAR allows for competitive learning between these duplicates on a traditional error-correction learning framework. 

To study SAAVs, we modified ACOL to learn latent annotations in a fully unsupervised setup by substituting the real, yet unavailable, parent-class information with a pseudo one (i.e. randomly generated pseudo parent-classes). To generate examples for a pseudo parent-class, we choose a domain specific transformation to be applied to every sample in the dataset. The transformed dataset constitutes the examples of that pseudo parent-class and every new transformation (i.e. the random sampling of codon positions) generates a new pseudo parent-class. Naturally, the main classification task performed over these pseudo parent-classes does not represent any meaningful knowledge about the data by itself. However, frequent and random selection of these pseudo parent-classes allow the ACOL neural network to learn sub-classes of these pseudo parents without bias. While each sub-class corresponds to a latent annotation which may or may not be meaningful, the combination of these annotations learned through abundant and concurrent clusterings reveals an unbiased and robust similarity metric between different metagenomes.

Following sections describe the pseudo parent-class generation and similarity metric calculation along with the details of model training.

### Loading the dataset

First we run all the import statements and read the anvi'o output for SAAVs into a Pandas dataframe:

``` python
from __future__ import print_function

import numpy as np
import pandas as pd

from keras.optimizers import SGD

from lalnets.commons.datasets import load_sar11
from lalnets.commons.utils import get_similarity_matrix
from lalnets.acol.models import define_mlp
from lalnets.acol.trainings import train_with_parents, get_model_truncated
from lalnets.metagenome.preprocessing import get_pseudo_labels_mini

np.random.seed(1337)

# path to the dataset:
loc = 'S-LLPA_SAAVs_20x_10percent_departure.txt'

#load the dataset
df, sample_id_map = load_sar11(loc)
```

### Pseudo parent-class generation

Consider an $$n_s \times n_p \times n_f$$ metagenomics dataset represented by the 3-D tensor $$\mathbf{D}$$, where $$n_s$$ is the number of metagenomes to be clustered, $$n_p$$ is the number of codon positions per metagenome and $$n_f$$ is the number of features representing each codon position. Specifically, our dataset can be specified by a $$74 \times 37,416 \times 2$$ tensor, as each codon position is represented by the two most frequent amino acids found in this position.

To generate the examples of $$i^{th}$$ pseudo parent-class, we randomly sample $$n_{\acute{p}}$$ positions out of $$n_p$$ with replacement. Resulting $$n_s \times n_{\acute{p}} \times n_f$$ subsets which correspond to $$d=n_{\acute{p}}n_f$$ dimensional $$n_s$$ examples of pseudo parent-class $$i$$. This procedure is repeated for $$n_\psi$$ times, and ultimately we obtain an $$m \times d$$ input matrix $$\boldsymbol{X} = [\boldsymbol{x}_1,...,\boldsymbol{x}_{m}]^T$$ and corresponding pseudo parent-class labels $$\boldsymbol{t}=[t_1,...,t_{m}]^T$$ to train a neural network, where $$n_\psi$$ is the number of the pseudo parents and $$m$$ is the total number of the examples generated in this procedure, such that $$m=n_\psi n_s$$.
 
We select $$n_{\acute{p}}=2000$$ and $$n_\psi=1000$$ to produce the results in this article, therefore $$\boldsymbol{X}$$ is  $$74000 \times 4000$$ matrix where each one of 1000 pseudo parents is equally represented by 74 examples sampled from different metagenomes. We also keep track of the metagenome labels $$\boldsymbol{q}=[q_1,...,q_{m}]^T$$, indicating the source metagenome of every example. This information will be used later to produce the similarity matrix. 

Setting dataset paramters to set the stage for pseudo parent-class generation procedure:

``` python
# number of metagenomes
n_s = 74

# number of positions to be sampled
n_p_acute = 2000

# number of features per position
n_f = 2

# number of dimensions of the input
d = n_p_acute*n_f

# number of pseudo-classes 
n_psi = 1000

# running the 'Algorithm 1: Pseudo-class generation':
X, t, q = get_pseudo_labels_mini(df, n_s, n_p_acute, n_psi)
```

The following describes the algorithm for sampling and pseudo parent-class generation procedure:

[![SAR11]({{images}}/algorithm1.png)]({{images}}/algorithm1.png){:.center-img .width-100}

### Learning Latent Annotations

Neural networks define a family of functions parameterized by weights and biases which define the relation between inputs and outputs. In multi-class categorization tasks, outputs correspond to class labels, hence in a typical output layer structure there exists an individual output node for each class. An activation function, such as softmax is then used to calculate normalized exponentials to convert the previous hidden layer's activations, i.e. scores, into probabilities.

Unlike traditional output layer structure, ACOL defines more than one softmax node ($$k$$ duplicates) per class (in this particular case, we prefer to use "pseudo parent-class" term to emphasize that these classes are not expert-defined but automatically generated depending on random sampling). Outputs of $$k$$ duplicated softmax nodes that belong to the same pseudo parent are then combined in a subsequent pooling layer for the final prediction. Training is performed in the configuration shown in figure below. This might look like a classifier with redundant softmax nodes. However, duplicated softmax nodes of each pseudo parent are specialized due to [dropout](http://jmlr.org/papers/v15/srivastava14a.html){:target="_blank"} and the unsupervised regularization applied throughout the training in a way that each one of $$n=n_\psi k$$ softmax nodes represents an individual sub-class of a pseudo parent, i.e. latent annotation.

[![SAR11]({{images}}/schema_acol.png)]({{images}}/schema_acol.png){:.center-img .width-70}

Consider a neural network with ACOL consisting of $$L-1$$ hidden layers where $$l$$ denotes the individual index for each layer such that $$l \in \{0,...,L\}$$. Let $$\boldsymbol{Y} ^{(l)}$$ denote the output of the nodes at layer $$l$$. $$\boldsymbol{Y} ^{(0)}=\boldsymbol{X}$$ is the input and $$f(\boldsymbol{X})=f^{(L)}(\boldsymbol{X})=\boldsymbol{Y}^{(L)}=\boldsymbol{Y}$$ is the output of the entire network, and $$\boldsymbol{W} ^{(l)}$$ and $$\boldsymbol{b}^{(l)}$$ are the weights and biases of layer $$l$$, respectively. Then, the feedforward operation of the neural network can be written as

$$
f^{(l)}\big(\boldsymbol{X}\big) = 
\boldsymbol{Y}^{(l)} = 
h^{(l)}\big(\boldsymbol{Y}^{(l-1)}\boldsymbol{W}^{(l)} + \boldsymbol{b}^{(l)}\big)
$$

where $$h^{(l)}$$(.) is the activation function applied at layer $$l$$. For the sake of clarity, let's specify the activities going into the augmented softmax layer of this network such that

$$
\boldsymbol{Z} := \boldsymbol{Y}^{(L-2)}\boldsymbol{W}^{(L-1)} + \boldsymbol{b}^{(L-1)}
$$

and its positive part as $$\boldsymbol{B}$$ such that

$$
g\big(\boldsymbol{X}\big) = \boldsymbol{B} := \max{\big(\boldsymbol{0}, \boldsymbol{Z}\big)}
$$

both of which correspond to $$m \times n$$ matrices, where $$n=n_\psi k$$, and $$k$$ is the clustering coefficient of ACOL (which we chose as 4 for our SAAVs data). The output of a neural network with ACOL can then be written in terms of $$\boldsymbol{Z}$$ as 

$$
f\big(\boldsymbol{X}\big) = 
\boldsymbol{Y} =  
h^{(L)}\bigg(h^{(L-1)}\big(\boldsymbol{Z}\big)\boldsymbol{W}^{(L)} + \boldsymbol{b}^{(L)}\bigg)
$$

where $$\boldsymbol{Y}$$ is an $$m \times n_\psi$$ matrix in which $$Y_{ij}$$ is the probability of the $$i^{th}$$ example belonging to the $$j^{th}$$ pseudo parent. Since $$h^{(L-1)}(.)$$ and $$h^{(L)}(.)$$ respectively correspond to softmax and linear activation functions and $$\boldsymbol{b}^{(L)} = \boldsymbol{0}$$, then this expression further simplifies into 

$$
f\big(\boldsymbol{X}\big) = 
\boldsymbol{Y} =  
softmax\big(\boldsymbol{Z}\big)\boldsymbol{W}^{(L)}
$$

where $$\boldsymbol{W}^{(L)}$$ is an $$n \times n_\psi$$ matrix representing the constant weights between augmented softmax layer and pooling layer.

$$
\boldsymbol{W}^{(L)}=
\begin{bmatrix}
\boldsymbol{I}_{n_\psi} \\
\boldsymbol{I}_{n_\psi} \\
\vdots \\
\boldsymbol{I}_{n_\psi}
\end{bmatrix}
$$

and simply sums up the output probabilities of the softmax nodes belonging to the same pseudo-class. Since the output of the augmented softmax layer is already normalized, no additional averaging is needed at the pooling layer and summation alone is sufficient to calculate final pseudo-class probabilities. 
Also, unsupervised regularization $$\mathcal{U}(.)$$ is applied to $$\boldsymbol{B}$$ to penalize the range of its distribution.
The overall objective function of the training then becomes

$$
\mathcal{L}\big(\boldsymbol{Y}, \boldsymbol{t}\big) +
\mathcal{U}\big(\boldsymbol{B}\big) = 
\mathcal{L}\big(softmax(\boldsymbol{Z})\boldsymbol{W}^{(L)}, \boldsymbol{t}\big) +
c_R\sum\limits_{j=1}^{n}\bigg(\max\big(\boldsymbol{B}_{:,j}\big)-\min\big(\boldsymbol{B}_{:,j}\big)\bigg)
$$

where $$\mathcal{L}(.)$$ is the supervised log loss function, $$\boldsymbol{t}=[t_1,...,t_{m}]^T$$ is the vector of the provided pseudo parent labels such that $$t_i \in \{1,...,n_\psi\}$$, $$\boldsymbol{B}_{:,j}$$ corresponds to $$j^{th}$$ column vector of matrix $$\boldsymbol{B}$$ and $$c_R$$ is the weighting coefficient. 

We employ a neural network with 3 hidden layers of 2048 nodes,


``` python
# number of hidden layers, number of nodes per layer 
mlp_params =(3, 2048)
```

and choose ACOL hyperparameters as follows:

```python
# clustering coeffient of ACOL
k = 4
 
# dropout ratio to be applied to augmented softmax layer
p = 0.5

# weighting coeffient of the unsupervised regularization
c_R = 0.00001
```

Training of the newwork is performed according to simultaneous supervised and unsupervised updates resulting from the objective function given above. We adopt stochastic gradient descent in the mini-batch mode for optimization:


```python
# define optimizer
sgd = SGD(lr=0.01, decay=1e-6, momentum=0.95, nesterov=True)   
```

Algorithm 2 below describes the entire training procedure. Along with unsupervised regularization, it also applies [dropout method](http://jmlr.org/papers/v15/srivastava14a.html){:target="_blank"} to the augmented softmax layer to distribute examples across the duplicated softmax nodes. For each mini-batch, an $$1 \times n$$ row vector $$\boldsymbol{r}$$ of independent Bernoulli random variables is sampled and multiplied element-wise with the output of the augmented softmax layer. This operation corresponds to dropping out each one of the $$n$$ softmax nodes with the probability of $$p$$ for that batch.  

[![SAR11]({{images}}/algorithm2.png)]({{images}}/algorithm2.png){:.center-img .width-100}


```python
#Algorithm 2: Model training for 100 epochs
metrics, acti, model = train_with_parents(nb_parents = n_psi, 
                                          nb_clusters_per_parent = k,
                                          define_model = define_mlp, 
                                          model_params = model_params, 
                                          optimizer = sgd,
                                          X_train = X, 
                                          y_train = None, 
                                          y_train_parent = t, 
                                          X_test = X, 
                                          y_test = None, 
                                          y_test_parent = t,
                                          nb_reruns = 1, 
                                          nb_epoch = 100, 
                                          nb_dpoints = 10, 
                                          batch_size = 128,
                                          test_on_test_set = False, 
                                          update_c3 = None,
                                          return_model = True,
                                          save_after_each_rerun=None,
                                          model_in=None
                                          )
```

After training phase is completed, network is simply truncated by completely disconnecting the pooling layer as shown below.

[![SAR11]({{images}}/schema_red.png)]({{images}}/schema_red.png){:.center-img .width-70}

The rest of the network with trained weights is used to assign the annotations to each example. This assignment can be described as

$$
y_i :=  argmax_{1\le j \le n}Z_{ij}
$$

where $$y_i$$ is the annotation assigned to $$i^{th}$$ example. 


```python
#get assigned annotations
model_truncated = model.get_model_truncated(define_mlp, model_params, n_psi)
y = model_truncated.predict_classes(X, verbose=0)
```

Using the assigned annotations $$\boldsymbol{y}$$ and the metagenome labels $$\boldsymbol{q}$$, the similarity matrix $$\boldsymbol{S}$$ used to obtain the dendrogram provided in this article is constructed as shown in Algorithm 3.

[![SAR11]({{images}}/algorithm3.png)]({{images}}/algorithm3.png){:.center-img .width-100}


```python
# Algorithm 3: Obtaining similarity matrix
_, S = get_similarity_matrix(q, y, n_psi, k, True)
```

Since applying dropout to distribute the examples across the duplicated softmax nodes also introduces a variance to the assigned annotations, we take the average of the similarity matrices obtained during the last 20 epochs of the training. Moreover, to reduce the variance due to the random generation of the pseudo parent-classes, we repeat the entire procedure 100 times with a new selection of pseudo parents for each repetition. 


```python
# Averaging hyperparameters for variance reduction 
# number of epochs whose results to be averaged
n_avg = 20

# number of retrainings
n_exp = 100      
```

### The entire flow in Python

Assume the following code is stored in a file called `ACOL_Pseudo_SAR11.py`. We used the following code to obtain a matrix that describes pairwise distances between our metagenomes.

```python
from __future__ import print_function

import numpy as np
import pandas as pd

from keras.optimizers import SGD

from lalnets.commons.datasets import load_sar11
from lalnets.commons.utils import get_similarity_matrix
from lalnets.acol.models import define_mlp
from lalnets.acol.trainings import train_with_parents, get_model_truncated
from lalnets.metagenome.preprocessing import get_pseudo_labels_mini

# set seed for reproducibility
np.random.seed(1337)

# path of the dataset
loc = 'S-LLPA_SAAVs_20x_10percent_departure.txt'

# load the dataset
df, sample_id_map = load_sar11(loc)

# Dataset hyperparameters ######################################
n_s = 74 # num metagenomes
n_p_acute = 2000 # num positions to be sampled
n_f = 2 # num features per position
n_psi = 1000 # num pseudo-classes 
d = n_p_acute*n_f # num dimensions of the input
#################################################################

# NN hyperparameters ############################################
mlp_params =(3, 2048) # num hidden layers & num nodes per layer 
#################################################################

# ACOL hyperparameters ##########################################
k = 4 # clustering coeffient of ACOL
p = 0.5 # dropout ratio to be applied to augmented softmax layer
c_R = 0.00001 # weighting coeff. of the unsuperv. regularization
#################################################################

# define model
acol_params = (k, p, c_R, None, None, None, 0, 'average', False)
model_params = ((d,), n_psi, mlp_params, (0., 0., 2.), True, acol_params)

# define optimizer
sgd = SGD(lr=0.01, decay=1e-6, momentum=0.95, nesterov=True)   

# number of epochs whose results to be averaged
n_avg = 20

# number of retrainings
n_exp = 100      

# initialize similarity matrices
S_avg = np.zeros((n_s, n_s, n_avg))
S_all = np.zeros((n_s, n_s, n_exp))

for exp in range(n_exp):
    # Algorithm 1: Pseudo-class generation
    X, t, q = get_pseudo_labels_mini(df, n_s, n_p_acute, n_psi)

    # Algorithm 2: Model training for 100 epochs
    metrics, acti, model = train_with_parents(nb_parents = n_psi, 
                                              nb_clusters_per_parent = k,
                                              define_model = define_mlp, 
                                              model_params = model_params, 
                                              optimizer = sgd,
                                              X_train = X, 
                                              y_train = None, 
                                              y_train_parent = t, 
                                              X_test = X, 
                                              y_test = None, 
                                              y_test_parent = t,
                                              nb_reruns = 1, 
                                              nb_epoch = 100, 
                                              nb_dpoints = 10, 
                                              batch_size = 128,
                                              test_on_test_set = False, 
                                              update_c3 = None,
                                              return_model = True,
                                              save_after_each_rerun=None,
                                              model_in=None
                                              )

    # train the model for n_avg more epochs
    for i in range(n_avg):
        # Algorithm 2: Model training for 1 epoch
        metrics, acti, model = train_with_parents(nb_parents = n_psi, 
                                                  nb_clusters_per_parent = k,
                                                  define_model = define_mlp, 
                                                  model_params = model_params, 
                                                  optimizer = sgd,
                                                  X_train = X, 
                                                  y_train = None, 
                                                  y_train_parent = t, 
                                                  X_test = X, 
                                                  y_test = None, 
                                                  y_test_parent = t,
                                                  nb_reruns = 1, 
                                                  nb_epoch = 1, 
                                                  nb_dpoints = 1, 
                                                  batch_size = 128,
                                                  test_on_test_set = True, 
                                                  update_c3 = None,
                                                  return_model = True,
                                                  save_after_each_rerun=None,
                                                  model_in=model
                                                  )

        # get assigned annotations
        model_truncated = model.get_model_truncated(define_mlp, model_params, n_psi)
        y = model_truncated.predict_classes(X, verbose=0)
        
        # Algorithm 3: Obtaining similarity matrix
        _, S = get_similarity_matrix(q, y, n_psi, k, True)
        S_avg[:,:,i] = S
        
    # take the average of the similarity matrices obtained during last n_avg epochs
    S_all[:,:,exp] = S_avg.mean(axis=-1)
    
# take the average of all similarity matrices obtained in n_exp retrainings 
S = S_all.mean(axis=-1)
df_S = pd.DataFrame(S, index=sample_id_map)

# report the distance matrix
df_S.to_csv('S-LLPA-DEEP-LEARNING-DIST-MAT.csv', header=True, index=True, sep='\t')
```

This will run orders of magnitude faster When a cuda enabled GPU is available on the machine. In which case you can run it this way: 

``` bash
export THEANO_FLAGS=mode=FAST_RUN,device=gpu,lib.cnmem=0.95,floatX=float32
python ACOL_Pseudo_SAR11.py
```

otherwise you can run it this way:

```python
export THEANO_FLAGS=mode=FAST_RUN,device=cpu,lib.cnmem=0.95,floatX=float32
python ACOL_Pseudo_SAR11.py
```

The output file `S-LLPA-DEEP-LEARNING-DIST-MAT.csv` will contain deep learning estimated distances. 

The output file the code above generated when we run it on xx is [available here]({{ site.url }}/data/2017_Delmont_et_al_SAR11_SAAVs/files/S-LLPA-DEEP-LEARNING-DIST-MAT.csv)){:target="_blank"}.

### Identifying proteotypes from Deep Learning-estimated distances

The distance matrix `S-LLPA-DEEP-LEARNING-DIST-MAT.csv` contained distances across metagenoems, from which we could obtain a hierarchical clustering of our metagenomes. Yet, the identification of proteotypes required us to cut this dendrogram into a reasonable number of sub-clusters. To determine the appropriate number, we relied on the elbow of the intra-cluster sum-of-squares curve of k-means by running the following R code on the `S-LLPA-DEEP-LEARNING-DIST-MAT.csv`,

``` R
#!/usr/bin/env Rscript
# Visualize the squared mean error per number of k-means clusters given a distance matrix

library(ggplot2)

deep_learning_dists <- read.table("S-LLPA-DEEP-LEARNING-DIST-MAT.csv", sep=",", header=TRUE, row.names=1)

df <- data.frame(k=numeric(), error=numeric())

for (i in 1:250){
  for (j in 2:15){
    df <- rbind(df, c(j, sum(kmeans(deep_learning_dists, centers=j)$withinss)))
  }
}

names(df) <- c('k', 'error')

pdf('./num_clusters.pdf')
ggplot(df, aes(x=k, y=error, group=k)) + geom_jitter(alpha=0.1, size=0.5) + geom_violin()
dev.off()
```

which suggested that 'six' was an appropriate number to divide our dendrogram:

[![SAR11]({{images}}/kmeans.png)]({{images}}/kmeans.png){:.center-img .width-70}


## SAAVs on protein structures

The starting point for this section of the workflow is the SAAVs table for the core S-LLPA genes, which was generated in the "**Generating genomic variation data for HIMB83**" section as `S-LLPA_SAAVs_20x_10percent_departure.txt`.

### Finding core S-LLPA genes with PDB matches

First, we BLAST searched 799 core S-LLPA genes against the Protein Data Bank (PDB) to determine which of them had sufficiently similar homologues with solved tertiary structure.

To do so we first created a FASTA file with amono acid sequences of the 799 genes,

``` bash
# get all AA sequences in the contigs db
anvi-get-aa-sequences-for-gene-calls -c SAR11-CONTIGS.db -o SAR11-GENE-AA-SEQUENCES.fa

# subsample the core genes from it:
anvi-script-reformat-fasta SAR11-GENE-AA-SEQUENCES.fa
                           -I core-S-LLPA-genes.txt
                           -o CORE-S-LLPA-GENE-AA-SEQUENCES.fa
```

downloaded the PDB database as a single FASTA file,


``` bash
wget ftp://ftp.wwpdb.org/pub/pdb/derived_data/pdb_seqres.txt.gz
gzip -d pdb_seqres.txt.gz
```

{:.notice}
At the time of download for our study, the `pdb_seqres.txt` file was `125,199,982` bytes of data, although as of August 17th we note this has increased to `128,517,512` bytes. Since there is no version control with PDB data, new additions may yield slightly different results if you run our entire workflow, although we belive it is unlikely that the newly added protein sequences may influence the BLAST results.

created a BLAST database for PDB seqeunces,

``` bash
makeblastdb -in pdb_seqres.fasta
            -dbtype prot
            -out pdb_seqres.fasta
```

and ran BLAST on the 799 genes:

``` bash
blastp -query CORE-S-LLPA-GENE-AA-SEQUENCES.fa \
       -db pdb_seqres.fasta \
       -outfmt "6 qseqid sseqid pident qlen length mismatch gapopen qstart qend sstart send evalue bitscore" \
       -out CORE-S-LLPA-GENE-AA-SEQUENCES-vs-PDB-BLAST.fa
```

Next, we filtered out sequences with insufficient matches to the PDB. We accomplished this by considering only the PDB hit with the highest bitscore for each core S-LLPA gene, and by calculating `proper_pident`, which is the percentage of the query amino acids that were identical to an entry in the database given the entire lenght of the query sequence (so `proper_pident` helps us avoid the inflation of identity scores due to partial good matches). We filtered out genes in `CORE-S-LLPA-GENE-AA-SEQUENCES.fa` that had less than 30% `proper_pident` using the program `anvi-script-filter-fasta-by-blast`:

``` bash
wget https://raw.githubusercontent.com/merenlab/anvio/master/sandbox/anvi-script-filter-fasta-by-blast -O anvi-script-filter-fasta-by-blast
chmod +x anvi-script-filter-fasta-by-blast
./anvi-script-filter-fasta-by-blast CORE-S-LLPA-GENE-AA-SEQUENCES.fa \
                                  --blast CORE-S-LLPA-GENE-AA-SEQUENCES-vs-PDB-BLAST.fa \
                                  --threshold 30 \
                                  --output-fasta CORE-S-LLPA-GENE-AA-SEQUENCES-W-PDB-MATCHES.fa \
                                  --outfmt "qseqid sseqid pident qlen length mismatch gapopen qstart qend sstart send evalue bitscore"
```

436 genes matched to these criteria are stored in `CORE-S-LLPA-GENE-AA-SEQUENCES-W-PDB-MATCHES.fa` for downstream analyses.

### Protein structure prediction 

We computationally predicted the protein structures of these 436 genes using a template-based tertiary structure prediction algorithm called [RaptorX Structure Prediction](http://raptorx.uchicago.edu/). In short, RaptorX aligns query sequences to a database of solved protein structures, e.g. the PDB, and then uses the solved structures as templates to determine the conformation of the query sequence.

Using the submission form [http://raptorx.uchicago.edu/StructurePrediction/predict/](http://raptorx.uchicago.edu/StructurePrediction/predict/), we submitted protein structure prediction jobs in batches of 20 sequences (the limit imposed by RaptorX) by subdividing `CORE-S-LLPA-GENE-AA-SEQUENCES-W-PDB-MATCHES.fa` into 22 FASTA files.

RaptorX Structure Prediction outputs a zipped folder for each protein prediction named `<sequence_id>.all_in_one.zip`, where `<sequence_id>` is a unique tag generated by RaptorX. We created a new directory `RaptorXProperty`, manually moved all `<sequence_id>.all_in_one.zip` files into it, and unzipped them all. To make things more identifiable, we renamed the `<sequence_id>.all_in_one` folders to `<corresponding_gene_call>.all_in_one`, where `<corresponding_gene_call>` is the gene caller id defined by the SAAV table. To do this we created a python script called `rename_all_in_ones.py`, and executed it the following way:

``` bash
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/rename_all_in_ones.py
cd RaptorXProperty
python rename_all_in_ones.py
cd -
```

Before mapping SAAVs onto the predicted protein structure, we first did some maintenance on the SAAV table with the Python script `curate_SAAV_table.py`, which reads the original SAAV table `S-LLPA_SAAVs_20x_10percent_departure.txt`, and the `ALL_SPLITS-gene_non_outlier_coverages.txt` file that is generated when we summarized `HIMB83` in the section "**Creating a self-contained profile for HIMB83**".


``` bash
# downlod the script
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/curate_SAAV_table.py

# copy HIMB83 gene coverages from the summary dir (if you don't have this directory, the
# URL https://doi.org/10.6084/m9.figshare.5248435 serves HIMB83 profile with the SUMMARY
# output directory 
cp NON-REDUNDANT-MAGs-SPLIT/HIMB083/SUMMARY/bin_by_bin/ALL_SPLITS/ALL_SPLITS-gene_non_outlier_coverages.txt . 

# make sure the file S-LLPA_SAAVs_20x_10percent_departure.txt is also in this directory.
# if you don't have it, you may download the archive file that contains this and other files
# from the URL https://doi.org/10.6084/m9.figshare.5248447

# run the script
python curate_SAAV_table.py
```

Running this script will update the SAAV table with additional information, and create a new file called `S-LLPA_SAAVs_20x_10percent_departure_curated.txt` in the work directory.

### Structural mapping of SAAVs

To map SAAVs onto protein structures, we developed a tool called SAAV-structural-mapping ([https://github.com/merenlab/SAAV-structural-mapping](https://github.com/merenlab/SAAV-structural-mapping)) that visualizes SAAVs as spheres with different colors, sizes, and opacities. Each user-defined "perspective" defines how these three variables are controlled. Please visit the github page for instructions on how to install.

{:.notice}
Unfortunately due to the limitations of PyMOL, SAAV-structural-mapping requires Python version 2, thus is not in the anvi'o codebase. We hope things will get better, and this step will be a part of the anvi'o codebase.

{:.notice}
SAAV-structural-mapping depends on [PyMOL](https://pymol.org/), a molecular visualization package written in Python. For full functionality, a licensed version is required, however partial functionality is possible either with the free open-source version or the educational version (best alternative).

There are five inputs for this to work: 

* A gene list specifying the genes we are interested in visualizing. In the past we have visualized all of the 436 genes in the FASTA file (`CORE-S-LLPA-GENE-AA-SEQUENCES-W-PDB-MATCHES.fa`), however the images require a lot of storage space and the script takes a very long time to run. To create the interface available at [http://anvio.org/data/S-LLPA-SAAVs/](http://anvio.org/data/S-LLPA-SAAVs/), we subselected genes that looked visually interesting to us. If you have your own list of genes you're interested in looking at, we encourage you to create your own gene list and let us know what you find! You can get a copy of ours this way:


* A TAB-delimited file with the first column specifying the samples we are interested in visualizing. We chose to only consider surface samples at a depth of 5m. The additional columns define how samples group together for the creation of multi-sample images. We grouped our samples based on region, main groups, and proteotypes. You can get a copy of our sample groups the following way:

* A SAAV table. We used `S-LLPA_SAAVs_20x_10percent_departure_curated.txt`.

* A collection of structure predictions for all the genes in `genes-of-interest-for-PyMOL.txt`. We already created this: its the `RaptorXProperty` directory.

* A configuration file specifying how to visualize the SAAVs. Below is the configuration file we used (`structural-mapping-of-SAAVs-config.ini`). For a complete description of how you can define your own config file please visit the help documentation of the `anvi-map-saavs-to-structure` script in SAAV-structural-mapping. 

You can get copies of missing files for a full analysis (gene list, samples mapping, and the configuration file) the following way:

``` bash
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/genes-of-interest-for-PyMOL.txt
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/genes-of-interest-for-PyMOL.txt
wget http://merenlab.org/data/2017_Delmont_et_al_SAR11_SAAVs/files/samples-of-interest-for-PyMOL.txt
```

For your information, our configuration file looked like this, and it is highly flexible for advanced users:

``` ini
[BLOSUM90]
color_var = BLOSUM90
color_scheme = darkred_to_darkblue
merged_sphere_size_var = prevalence

[BLOSUM90_weighted]
color_var = BLOSUM90_weighted
color_scheme = darkred_to_darkblue
merged_sphere_size_var = prevalence

[solvent_accessibility]
color_var = solvent_acc
color_scheme = solvent_acc_cmap
merged_sphere_size_var = prevalence

[secondary_structure]
color_var = ss3
color_scheme = ss3_cmap
merged_sphere_size_var = prevalence

[AASTs]
color_var = competing_aas
color_scheme = competing_aas
merged_sphere_size_var = prevalence

[coverage_deviation]
color_var = rel_diff_from_mean_gene_cov
color_scheme = (#b6c7ee,#e8edf9,#ffffff,#fcf5f4,#7c0b03);(-0.94,-0.45,0.04,0.54,3.00)
merged_sphere_size_var = prevalence
```

With all of these pieces together, and assuming you did configure `anvi-map-saavs-to-structure` to be in your PATH, you can run the following command to generate output images:

``` bash
pymol -r anvi-map-saavs-to-structure -- \
      --gene-list genes-of-interest-for-PyMOL.txt \
      --sample-groups samples-of-interest-for-PyMOL.txt \
      --saav-table S-LLPA_SAAVs_20x_10percent_departure_curated.txt \
      --input-dir RaptorXProperty \
      --pymol-config structural-mapping-of-SAAVs-config.ini \
      --ray 1 \
      --res 1200 \
      --output-dir SAR11-S-LLPA-SAAVs-ON-STRUCTURE
```

{:.notice}
This will take some time. If you would like to speed it up, you can cut down on the number of perspectives in `structural-mapping-of-SAAVs-config.ini` and/or the number of genes in `genes-of-interest-for-PyMOL.txt`, and/or set `--ray 0` and `--res 600` for lower quality images.

{:.notice}
The URL [doi:10.5281/zenodo.835218](https://doi.org/10.5281/zenodo.835218){:target="_blank"} serves the output diretory, `SAR11-S-LLPA-SAAVs-ON-STRUCTURE`.


The output `SAR11-S-LLPA-SAAVs-ON-STRUCTURE` contains both PNG images for all the SAAV structure visualizations, as well as PyMOL files that can be opened to manipulate the structures.

### Generating an interactive output

The resulting images with SAAVs mapped onto predicted tertiary structures of proteins can be neatly organized with functional annotation in an interactive interface with the following anvi'o command:

``` bash
anvi-saavs-and-protein-structures-summary -i SAR11-S-LLPA-SAAVs-ON-STRUCTURE \
                                          -c SAR11-CONTIGS.db \
                                          -o S-LLPA-SAAVs \
                                          --soft-link-images \
                                          --perspectives 'AASTs,BLOSUM90,solvent_accessibility'
```

[http://anvio.org/data/S-LLPA-SAAVs/](http://anvio.org/data/S-LLPA-SAAVs/){:target="_blank"} serves our `S-LLPA-SAAVs` output directory, which contains an interactive web page to explore 58 proteins across 49 samples with 3 perspectives in our config file.

The same output is also archived at [doi:10.6084/m9.figshare.5248432](https://doi.org/10.6084/m9.figshare.5248432){:target="_blank"}.


## References

* Eddy SR. (2011). **Accelerated Profile HMM Searches**. PLoS Comput Biol 7: e1002195.

* Eren AM, Esen ÖC, Quince C, Vineis JH, Morrison HG, Sogin ML, et al. (2015). **Anvi’o: an advanced analysis and visualization platform for ‘omics data**. PeerJ 3: e1319.

* Eren AM, Vineis JH, Morrison HG, Sogin ML. (2013). **A Filtering Method to Generate High Quality Short Reads Using Illumina Paired-End Technology**. PLoS One 8: e66643.

* Hyatt D, Chen G-L, Locascio PF, Land ML, Larimer FW, Hauser LJ. (2010). **Prodigal: prokaryotic gene recognition and translation initiation site identification**. BMC Bioinformatics 11: 119.

* Langmead B, Salzberg SL. (2012). **Fast gapped-read alignment with Bowtie 2**. Nat Methods 9: 357–359.

* Li H, Handsaker B, Wysoker A, Fennell T, Ruan J, Homer N, et al. (2009). **The Sequence Alignment/Map format and SAMtools.** Bioinformatics 25: 2078–2079.

* Minoche AE, Dohm JC, Himmelbauer H. (2011). **Evaluation of genomic high-throughput sequencing data generated on Illumina HiSeq and genome analyzer systems**. Genome Biol 12: R112.

* Sunagawa S, Coelho LP, Chaffron S, Kultima JR, Labadie K, Salazar G, et al. (2015). **Ocean plankton. Structure and function of the global ocean microbiome**. Science 348: 1261359.

* Kopf, A., M. Bicak, and R. Kottmann. **The ocean sampling day consortium**. GigaScience. 2015; 4: 27.

<div style="display: block; height: 200px;">&nbsp;</div>
