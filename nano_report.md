# MIP 280A4 Final Project 2022

This report documents the final project I did for MIP 280A4, Microbial Sequence Analysis, in the fall of 2022.

It is written in [Markdown format](https://www.markdownguide.org/basic-syntax/).  

## Step 1: **Download fasta file of Drosphila virilis**

1. Create a directory for the download.
2. [Locate reference genome from NCBI and copy link here](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/285/735/GCF_003285735.1_DvirRS2/GCF_003285735.1_DvirRS2_genomic.fna.gz) 
3. Use this command to download from NCBI directly to the project folder on the server:
```
curl -OL https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/285/735/GCF_003285735.1_DvirRS2/GCF_003285735.1_DvirRS2_genomic.fna.gz
```
**Be sure that you are in the directory that you want the file to download to.**

## Step 2: FastQC
1. Use Fastqc to create html file using this command:
```
fastqc FoCo_virilis_R1.fastq
```
<img src="FastQC_report_before_trimming.png">

## Step 3: Trim Adapters

I used the following command to trim the adapters from the fastq file:

```
curl -OL cutadapt \
> -a AGATCGGAAGAGC \
> -q 30,30 \
> --minimum-length 80 \
> -o FoCo_virilis_R1_trimmed_fastq \
> FoCo_virilis_R1_fastq \
> | tee cutadapt.log

```
## Step 4: Confirm QC
I ran fastqc again on the trimmed reads. 
```

```
##Step 5: Map reads to reference genome

Using bowtie2, reads were mapped to the reference sequence using the following command:

```
curl -OL bowtie2 -x DvirRS2_genomic_index -U FoCo_virilis_R1_trimmed_fastq --no-unal --threads 8 -S FoCo_virilis_R1_mapped_to_DvirRS2.sam
```
