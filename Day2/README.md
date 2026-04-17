
#  Day 2 - Principles of single molecule resolution: polyA tail, isoform usage, per read modification

## Hands-on V: Isoform analysis with Isoquant 

Navigate to the alignment directory:

```bash
cd ~/output_mop/mRNA_CTR_m6A/alignment
```

For each bam file we can assign reads to knwon isoforms:

```bash
isoquant --reference ~/references/chr19.fa --genedb ~/references/chr19_annotation.gtf --complete_genedb --bam CTR_s.bam --data_type nanopore -o ../isoquant_CTR
```

or discover isoforms:

```bash 
isoquant --reference ~/references/chr19.fa --bam CTR_s.bam --data_type nanopore -o ../isoquant_discovery_CTR
```

Explore the output: 

```bash 
ls -l ~/output_mop/mRNA_CTR_m6A/isoquant_CTR/OUT
less ~/output_mop/mRNA_CTR_m6A/isoquant_CTR/OUT/OUT.read_assignments.tsv.gz

ls -l ~/output_mop/mRNA_CTR_m6A/isoquant_discovery_CTR/OUT
less ~/output_mop/mRNA_CTR_m6A/isoquant_discovery_CTR/OUT/OUT.transcript_models.gtf
```

You can download the new annotation file (.gtf) and load it in IGV.

Now, do the same with the KO file :) 

## Hands-on VI: poly(A)-tail estimation with Dorado


Navigate to the pre-processing directory:

```bash
cd ~/master_of_pores/mop_preprocess
```

Edit the  `params.yaml` file

```bash
Data: ../../mouse/CTR/CTR.pod5
Reference: ../../references/chr19.fa
Annotation: ../../references/chr19_annotation.gtf
Ref_type : genome
Counting: htseq
Output: ../../output_mop/mRNA_CTR_polyA
dorado-mod: "hac,m6A_DRACH --estimate-poly-a"
```
Run the pipeline!

```bash
nextflow run mop_preprocess.nf -params-file params.yaml -with-singularity -profile local -bg > log_mRNA_CTR_polyA.txt
```

Navigate to the alignment directory:

```bash
cd ~/output_mop/mRNA_CTR_polyA/alignment
```

Inspect the bam file:

```
samtools view CTR_s.bam | less 
samtools view CTR_s.bam | awk '/pt:i:/ { for (i=1; i<=NF; i++) if ($i ~ /pt:i:/) matched=$i; print $1, $2, $3, $4, matched; }' | less
```

## Hands-on VII: per read modification information

Navigate to the alignment directory:

```bash
cd ~/output_mop/mRNA_CTR_m6A/alignment
```

Extract modification probability per position per read with Modkit

```bash
modkit extract full CTR_s.bam --num-reads 1000 ../modkit/CTR_modkit_full.txt
```
The ```mod_qual``` column will contain the modification probability for each position in each read.

Now, do the same with the KO file :) 


## (Advanced) Hands-on VIII: per isoform analysis

Use the isoquant read assignments to split the bam by different isoforms of Tmem134 (exon 6 skipped or included). You can use IGV for better visualization.

For example, let's focus on Rhod gene. We find the ensembl gene ID

```
cd ~/output_mop/mRNA_CTR_m6A/isoquant_CTR/OUT
grep Rhod OUT.extended_annotation.gtf
```

extract the read ids and the isoform ids 

```
zcat OUT.read_assignments.tsv.gz | grep ENSMUSG00000041845.11 | cut -f 1,4 > reads_isoforms.txt
```

how many isoforms are there? which ones? use grep to split the reads by isoform id. 

```
zcat OUT.read_assignments.tsv.gz | grep ENSMUSG00000041845.11 | cut -f 4 | sort -u
zcat OUT.read_assignments.tsv.gz | grep ENSMUSG00000041845.11 |  grep ENSMUST00000048197.9 | cut -f 1 > isoform1_readids.txt
```

do the same for the other isoforms.

Now, use these files to extract the reads from the bam file and make a minibam for each isoform. 

```
cd ~/output_mop/mRNA_CTR_m6A/alignment/
samtools view -H CTR_s.bam > isoform1.sam
samtools view CTR_s.bam | grep -f ../isoquant_CTR/OUT/isoform1_readids.txt >> isoform1.sam
samtools sort isoform1.sam  | samtools view -Sb -o isoform1.bam
samtools index isoform1.bam
```

Repeat for the other isoforms. Now you can visualize the split bams in IGV, run modkit separately on each bam to get per isoform modification frequency, study per isoform polyA tail length... etc


# Group assignments

## Assignment 1

In ```/home/[your_username]/data/bacteria/pod5``` you will find raw sequencing data from bacterial rRNA samples (E. coli).

The run contains 12 barcodes.

bc1 and bc2 are WT strains.

bc3, bc7, and bc9 are KO strains lacking some rRNA modifications.

Find which modifications are missing in each of these KO strains.

## Assignment 2

In ```/home/[your_username]/data/yeast/pod5``` you will find raw sequencing data from yeast rRNA samples (S. cerevisiae).

Use a modification-aware basecaller to predict pseU sites in 18S and 25S rRNA. Investigate how well the predictions match the ground truth (known modifications). Use bc19 for the analysis.

## Assignment 3

In ```/home/[your_username]/data/yeast/pod5``` you will find raw sequencing data from yeast rRNA samples (S. cerevisiae).

Bc19 is the WT sample. Find which modifications are missing in barcodes 13, 14 and 15.

## Assignment 4

In ```/home/[your_username]/data/yeast/pod5``` you will find raw sequencing data from yeast rRNA samples (S. cerevisiae).

Bc19 is the WT sample. Find which modifications are missing in barcodes 17, 21, 22 and 23.

## Assignment 5

Go back to ```/home/[your_username]/data/bacteria/pod5```.

Use a modification-aware basecaller to predict m5C sites. Investigate how well the predictions match the ground truth (known modifications). Use bc1 for the analysis.
