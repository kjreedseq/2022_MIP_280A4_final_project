##  MIP 280A4 Final Project 2022

This report documents the final project I did for MIP 280A4, Microbial Sequence Analysis, in the fall of 2022.

It is written in [Markdown format](https://www.markdownguide.org/basic-syntax/).  

<img src="Drosophila_virilis.png"> *photographed by Darren J. Obbard*

For this project, I worked with a dataset derived from total RNA of pooled *Drosophila virilis* (males or females) from the Stenglein lab. Libary preparation and sequencing were performed by Tillie Dunham and Kai Chase. Wild-caught Drosophila virilis were pooled for sequencing. An Illumina NextSeq 500 with single-end 150 bp reads.
The goal of this project was to discover if there were virus reads present in the sequencing data. Briefly, sequencing reads were mapped to the *D. virilis* genome. Unmapped reads, were assembed using SPAdes and the contigs from this assembly were analyzed using NCBI's BLASTn program. All work for this project is located at kjreed@thoth01.cvmbs.colostate.edu. 


## Step 1: Create a clone of the repository on github
I created an account on github and created this report.
I cloned the repository using the following command:
```
git clone https://github.com/kjreedseq/2022_MIP_280A4_final_project.git
```
`I then created a report to record my workflow and data, called nano_report.md in the repository on thoth01:
```
thoth01.cvmbs.colostate.edu/home/kjreed/2022_MIP_280A4_final_project
```

```
git add nano_report.md
```
The report was committed with the following command:
```
git commit -m "adding nano_report.md"
```
While committing, the comment was added in the command line. 
The next step was to publish the file on github using the command:
```
git push origin main
```
 
## Step 2: **Download fasta file of Drosphila virilis**

1. Navigate to the 2022_MIP_280A4_final_project folder (/home/kjreed/2022_MIP_280A4_final_project on the thoth01.cvmbs.colostate.edu server.
2. Locate the reference genome on NCBI via a web browser.
3. [Copy link here](curl -OL https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/285/735/GCF_003285735.1_DvirRS2/GCF_003285735.1_DvirRS2_genomic.fna.gz) 

The file containing the compressed fasta file will directly upload to the directory.


## Step 3: FastQC

1. The data file for the *D. virilis* sequencing run was copied from the thoth01 server:
```
/home/data_for_classes/2022_MIP_280A4/final_project_datasets$ cp FoCo_virilis_R1.fastq  ~/2022_MIP_280A4_final_project
```
2. The conda environment was activated using:
```conda activate bio_tools
```

This puts the software tools in the PATH.

3. The FoCo_virilis_r1.fastq file was analyzed for quality using fastqc: 
fastqc FoCo_virilis_R1.fastq
```
<img src="FastQC_report_before_trimming.png">

<img src="FastQC_before_trimming_adapters.png">

## Step 4: Trim Adapters

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
## Step 5: Confirm QC
I ran fastqc again on the trimmed reads. The FastQC report was assessed on the web browser as before.

<img src="FastQC_base_qual_post_trim.png">

<img src="FastQC_adapter_post_trim.png">

## Step 6: Create an index of the genome

Using bowtie2, create an index of the genome using the following command:

```
bowtie2-build --threads 8 GCF_003285735.1_DvirRS2_genomic.fna  DvirRS2_genomic_index
```

## Step 7: Map reads to reference genome

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
