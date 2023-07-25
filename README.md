# Pipeline to analyze Genome-wide 3'RACE data (gw-3'RACE)
Script to analysis of gw-3'RACE data from fastq files.

## Sequencing Data Analysis Protocol
Sequencing data were filtered for read pairs where R2 contained a 3'-adapter sequence ligated at the first steps of the library preparation. Low-quality read pairs were removed using fastp [36].

Reads R1 and R2 were aligned separately to the genome using the STAR aligner. For R2, settings allowed soft-clipping and maintaining non-aligned reads in the final output (.sam file).

Following this, uniquely aligned reads were extracted from the R1 alignment file and matched to genomic features, forming a .bed type file, using samtools and bedtools utilities.

In the next step, R1 read names from the created .bed file were matched with their R2 read mates from the .sam file. A custom table was created containing in each row the R2 CIGAR string, read name, sequence, matched feature from the R1 mate, and R1 and R2 alignment coordinates.

These raw data tables served as a basis for tail analysis. Information about the tail existence, its length and type were extracted from the tables using a custom Python script.

The script identified tail sequences from the R2 read based on the CIGAR string and categorized tails as poly(A), poly(A)U, oligo(U), or other using text search (grep regular expressions). If R2 was not aligned and thus a CIGAR string was unavailable, the R2 sequence was scanned using text search to categorize the tail into one of the aforementioned four categories.

The script output was a .csv type table containing necessary information for downstream data analysis and visualization. The final table contained one row per uniquely aligned R1 read with information about tail type, 3'-end coordinate (where applicable), tail length, number of Us (where applicable), gene name, and distance of detected 3'-end from annotated TES.

## Stages of data analysis

### 0. Preparation of STAR index
The first stage of gw-3'RACE sequencing data analysis is the preparation of the index. In our script, we use the STAR aligner. Below is an example code for preparing a STAR reference for S. pombe yeast. Please pay attention to the appropriate selection of references, downloading them, and placing them in the appropriate directory to which we will refer later. I suggest creating a genome directory and directing the indexed references there.

```STAR --runMode genomeGenerate --genomeDir genome/ --genomeFastaFiles genome/Schizosaccharomyces_pombe.ASM294v2.dna.toplevel.fa```


### 1. test.sh
bash test_script.sh -i Wild_type_clone2_R1_001.fastq -I Wild_type_clone2_R2_001.fastq -o output_Wild_type_clone2_20220830


### 2. Joining 
  *run the script in the directory with the output)
bash ../joining_R1R2.sh -i R1_Aligned.sortedByCoord.out.bam -I R2_Aligned.out.bam -a ../genome/annotation_6k_clean.bed


### 3.bigwigs 
  * run the script in the directory with the output
  * R1 input: bed, R2 input bam!
bash ../R1_bed_bigWig.sh -i R1_Aligned.sortedByCoord.out.bam_sorted_unique.bed -g ../genome/Schizosaccharomyces_pombe.ASM294v2.dna.toplevel.fa.fai 

bash ../R2_bed_bigWig.sh -i R2_Aligned.out.bam  -g ../genome/Schizosaccharomyces_pombe.ASM294v2.dna.toplevel.fa.fai 

### 4. Analysis of tails using Python script
