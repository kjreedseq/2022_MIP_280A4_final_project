##  MIP 280A4 Final Project 2022

This report documents the final project I did for MIP 280A4, Microbial Sequence Analysis, in the fall of 2022.

It is written in [Markdown format](https://www.markdownguide.org/basic-syntax/).  

### Virus Hunting with Drosophila virilis

<img src="Drosophila_virilis.png"> *photographed by Darren J. Obbard*



For this project, I worked with a dataset derived from total RNA of pooled *Drosophila virilis* (males or females) from the Stenglein lab. Libary preparation and sequencing were performed by Tillie Dunham and Kai Chase. Wild-caught Drosophila virilis were pooled for sequencing. The run was performed using an Illumina NextSeq 500 with single-end 150 bp reads.
The goal of this project was to discover if there were virus reads present in the sequencing data. Briefly, sequencing reads were mapped to the *D. virilis* genome. Unmapped reads, where one might expect to find viral reads, were assembed using SPAdes, and the contigs from this assembly were analyzed using NCBI's BLASTn database and alogrithm. All work for this project is located on the thoth01.cvmbs.colostate.edu server. 


## Step 1: Create a clone of the repository on github
I created an account on github and then created this report to document the workflow and analysis for this project:

I cloned the repository using the following command:
```
kjreed@thoth01:~$ git clone https://github.com/kjreedseq/2022_MIP_280A4_final_project.git
```
While in the 2022_MIP_280A4_final_project repository on thoth01, I created a file to record my workflow and data, called nano_report.md and then cloned it on github:
```
kjreed@thoth01: ~/2022_MIP_280A4_final_project$ touch nano_report.md
```
Then the process of uploading it to the github repository was started, using the ```git add``` command:

```
kjreed@thoth01:~$ git add nano_report.md
```
The report was committed with the ```git commit``` command, with a -m argument so that the comment for the upload could be added simultaneously.
```
kjreed@thoth01:~$ git commit -m "adding nano_report.md"
```
While committing, the comment was added in the command line. 
The next step was to publish the file on github using ```git push origin main```
```
kjreed@thoth01:~$ git push origin main
```
 
## Step 2: **Download fasta file of Drosphila virilis**

1. Navigate to the 2022_MIP_280A4_final_project folder (/home/kjreed/2022_MIP_280A4_final_project on the thoth01.cvmbs.colostate.edu server.
2. Locate the reference genome on NCBI via a web browser.
3. Download directly to the repository on thoth01 using:
```
curl -OL https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/285/735/GCF_003285735.1_DvirRS2/GCF_003285735.1_DvirRS2_genomic.fna.gz
```
The file containing the compressed fasta file will directly upload to the directory.


## Step 3: FastQC

1. The data file for the *D. virilis* sequencing run was copied from the thoth01 server:
```
kjreed@thoth01: /home/data_for_classes/2022_MIP_280A4/final_project_datasets$ cp FoCo_virilis_R1.fastq  ~/2022_MIP_280A4_final_project
```
2. The conda environment was activated using:
```
kjreed@thoth01:~$ conda activate bio_tools
```

This puts the software tools in my PATH.

3. The FoCo_virilis_r1.fastq file was analyzed for quality using fastqc:
``` 
kjreed@thoth01:~/2022_MIP_280A4_final_project$ fastqc FoCo_virilis_R1.fastq 
```
<img src="FastQC_report_before_trimming.png">

<img src="FastQC_before_trimming_adapters.png">

The number of reads from the sequencing run was 1,616,192. The quality score for the reads was >30 for reads that were under 130bp. Above this, the quality score dropped to <30 and at 150 bp it was nearly Q=20. The average quality score was 34. There was a significant amount of PCR duplicates among the reads. Adapter sequences were present, so it was necessary to trim the ends of the reads to remove them. If not trimmed, reads with these adapter sequences won't map.

## Step 4: Trim Adapters

I used the following command to trim the adapters from the fastq file:
```
kjreed@thoth01:~/2022_MIP_280A4_final_project$ \
cutadapt \
> -a AGATCGGAAGAGC \
> -q 30,30 \
> --minimum-length 80 \
> -o FoCo_virilis_R1_trimmed_fastq \
> FoCo_virilis_R1_fastq \
> | tee cutadapt.log
```

## Step 5: Confirm QC
I ran FastQC again on the trimmed reads. The FastQC report was assessed on the web browser as before.

<img src="FastQC_base_qual_post_trim.png">

<img src="FastQC_adapter_post_trim.png">

After trimming, the dataset contained 1,428,938 reads. Adapter content was 0.

## Step 6: Create an index of the genome

Using bowtie2, I created an index of the genome using the following command:

```
kjreed@thoth01:~/2022_MIP_280A4_final_project$ bowtie2-build --threads 8 GCF_003285735.1_DvirRS2_genomic.fna  DvirRS2_genomic_index
```

This created a set of indexes that will increase the speed at which assembly can be processed by the computer.

## Step 7: Map reads to reference genome

Using bowtie2, reads were mapped to the reference sequence using the following command:

```
kjreed@thoth01:~/2022_MIP_280A4_final_project$ \
bowtie2 -x DvirRS2_genomic_index \
> -U FoCo_virilis_R1_trimmed_fastq \
> --no-unal \
> --threads 8 \
> -S FoCo_virilis_R1_mapped_to_DvirRS2.sam \
> --un FoCo_virilis_R1_not_mapped_fastq
```
Unmapped reads were put in a separate file, "FoCo_virilis_R1_not_mapped_fastq", and this file was used to perform an assembly of contigs that did not map to the Drosophila virilis genome. 

## Step 8: Assemble unmapped reads

Assembly was performed using SPAdes with the following command: 

```
spades.py -o FoCo_virilis_R1_spades_assembly \
>  -s FoCo_virilis_R1_not_mapped.fastq \
>  -m 24 -t 48

```
Note: I had to change the fastq file name to ".fastq" from "_fastq" to run this program.

## Step 9: Make a new file with largest 12 contigs

## Step 10: BLAST contigs and analyze hits

I took the first 24 lines (12 contigs) and created a new fasta file:

```
seqtk seq -A contigs.fasta | head -24 > first_12_unmapped_contigs.fasta
```

## Step 9: Remap 

First build a new index from the 12 contigs using the following command:

```
bowtie2-build \
> first_12_unmapped_contigs.fasta first_12_unmapped_contigs_index 
```
This created an index with which to remap
