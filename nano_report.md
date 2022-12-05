# MIP 280A4 Final Project 2022

This report documents the final project I did for MIP 280A4, Microbial Sequence Analysis, in the fall of 2022.

It is written in [Markdown format](https://www.markdownguide.org/basic-syntax/).  

## Step 1: **Download fasta file of Drosphila virilis**

1. Create a directory for the download.
2. [Locate reference genome from NCBI and copy link here](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/285/735/GCF_003285735.1_DvirRS2/GCF_003285735.1_DvirRS2_genomic.fna.gz) 
3. Use this command to download from NCBI directly to the project folder on the server:

**Be sure that you are in the directory that you want the file to download to.**


## Step 2: FastQC
1. Use FastQC to create html file using this command:
```
fastqc FoCo_virilis_R1.fastq
```
<img src="FastQC_report_before_trimming.png">

<img src="FastQC_before_trimming_adapters.png">

## Step 3: Trim Adapters

I used the following command to trim the adapters from the fastq file:

```
cutadapt \
> -a AGATCGGAAGAGC \
> -q 30,30 \
> --minimum-length 80 \
> -o FoCo_virilis_R1_trimmed_fastq \
> FoCo_virilis_R1_fastq \
> | tee cutadapt.log

```
## Step 4: Confirm QC
I ran fastqc again on the trimmed reads. The FastQC report was assessed on the web browser as before.

<img src="FastQC_base_qual_post_trim.png">

<img src="FastQC_adapter_post_trim.png">

## Step 5: Create an index of the genome

Using bowtie2, create an index of the genome using the following command:

```
bowtie2-build --threads 8 GCF_003285735.1_DvirRS2_genomic.fna  DvirRS2_genomic_index
```

## Step 6: Map reads to reference genome

Using bowtie2, reads were mapped to the reference sequence using the following command:

```
bowtie2 -x DvirRS2_genomic_index \
> -U FoCo_virilis_R1_trimmed_fastq \
> --no-unal \
> --threads 8 \
> -S FoCo_virilis_R1_mapped_to_DvirRS2.sam \
> --un FoCo_virilis_R1_not_mapped_fastq
```
Unmapped reads were put in a separate file, "FoCo_virilis_R1_not_mapped_fastq",  and this file was used to perform an assembly of contigs that did not map to the Drosophila virilis genome. 

## Step 7: Assemble unmapped reads

Assembly was performed using SPAdes with the following command: 

```
spades.py -o FoCo_virilis_R1_spades_assembly \
>  -s FoCo_virilis_R1_not_mapped.fastq \
>  -m 24 -t 48

```
Note: I had to change the fastq file name to ".fastq" from "_fastq" to run this program.

## Step 8: Make a new file with largest 12 contigs

## Step 9: BLAST contigs and analyze hits

I took the first 24 lines (12 contigs) and created a new fasta file:

```
seqtk seq -A contigs.fasta | head -24 > first_12_unmapped_contigs.fasta
```
I then created a new index from the newly created file:

```
bowtie2-build first_12_unmapped_contigs.fasta first_12_unmapped_contigs_index

```
## Step 9: Remap 

First build a new index from the 12 contigs using the following command:

```
bowtie2-build \
> first_12_unmapped_contigs.fasta first_12_unmapped_contigs_index
 
```
This created an index with which to remap
