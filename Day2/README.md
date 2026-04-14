
#  Master of Pores – Day 2 Principles of single molecule resolution: polyA tail, isoform usage, per read modification



## Predict isoform usage with Isoquant 

For each bam file we can assign reads to knwon isoforms:

```bash
isoquant --reference ~/references/chr19.fa --genedb ~/references/chr19_annotation.db --complete_genedb --bam ~/output_mop/mRNA/alignment/pod5_s.bam --data_type nanopore -o ~/output_mop/mRNA/isoquant
```

## Discover isoforms with Isoquant

```bash 
isoquant --reference ~/references/chr19.fa --fastq ~/output_mop/mRNA/fastq/pod5.fq.gz --data_type nanopore -o isoquant_discovery_test
```




## basecall modifications with estimating the poly(A)-tail

Navigate to the pre-processing directory:

```bash
cd ~/MOP4_copied/mop_preprocess
```

Edit the  `params.yaml` file:

```
# Basecalling can be either NO, dorado, dorado-mod or dorado-duplex
basecalling: "dorado-mod"
```

```
dorado-mod: "hac,pseU --estimate-poly-a"
```

```
nextflow run mop_preprocess.nf -params-file params.yaml -with-singularity -profile local -bg > log_file.log
```

Inspect the bam file:

```
samtools view pod5_s.bam | awk '/pt:i:/ { for (i=1; i<=NF; i++) if ($i ~ /pt:i:/) matched=$i; print $1, $2, $3, $4, matched; }' | less
```


## analyze modifications through basecalling errors 

Navigate to the pre-processing directory:

```bash
cd ~/MOP4_copied/mop_mod
```

Edit the  `params.yaml` file

```
input_path: "path_to_your_mop_preprocess_output"
input_pod5: ""
comparison: "${projectDir}/comparison.tsv"

reference: "/home/ubuntu/Share/references/yeast_rRNA_ref.fa"
output: "${projectDir}/name_of_your_output_folder"
pars_tools: "${projectDir}/tools_opt.tsv"

# flows for nanoconsensus
epinano: "YES"
# nanoRMA needs mop_preprocess to be run with dorado-mod mode
nanoRMS: "NO"
baseQ: "NO"
f5c: "NO"

# Other flows
m6anet: "NO"
modkit: "NO"

#f5c kmer  model
f5c_kmer_model: "${projectDir}/models/rna004.nucleotide.5mer.model"

# epinano plots
epinano_plots: "YES"
email: ""

# Program params (workflows / tools)
progPars:
  epinano:
    epinano: ""
  f5c:
    f5c: "--rna"
  baseQ:
    baseQ: ""
  nanoRMS:
    nanoRMS: ""
  m6Anet:
    f5c: "--rna"
    inference: "--pretrained_model HEK293T_RNA004"
  modkit:
    modkit: "--mod-threshold 21891:0.90 --edge-filter 500,500"
    cov_filtering: ""
    strand: ""
    field_sel: "11"
```
Run the pipeline!

```
nextflow run mop_mod.nf -params-file params.yaml -with-singularity -profile local -bg > log_file.log
```



Optional:

Rerun mop_preprocess with the hac model, and then run mop_mod on that output. Let's 1) inspect and compare the bams, and 2) see how the epinano output is different


```
nextflow run mop_preprocess.nf -params-file params.yaml -with-singularity -profile local -bg > log_file.log
```


## To compare two epinano output files

```
Rscript scatterplot_script.R file_1.csv file_2.csv output.pdf
```




## basecall modifications with modification-aware basecalling models 

Navigate to the pre-processing directory:

```bash
cd ~/MOP4_copied/mop_preprocess
```

Edit the  `params.yaml` file

```
Basecalling can be either NO, dorado, dorado-mod or dorado-duplex
basecalling: "dorado-mod"
```

```
dorado-mod: "hac,pseU"
```


Run the pipeline!

```bash
 nextflow run mop_preprocess.nf -params-file params.yaml -with-singularity -profile local -bg > log_file.log
```









## Extract modification frequency per position with Modkit

```bash
modkit pileup Share/data/mouse/output/dorado_m6A_drach/mouse_drach_trial/alignment/pod5---bc_1_s.bam modkit/CTR_m6A_pileup.bed --log-filepath modkit/CTR_m6A_pileup.log
```


## Extract modification frequency per read with Modkit

```bash
modkit extract full Share/data/mouse/output/dorado_m6A_drach/mouse_drach_trial/alignment/pod5---bc_1_s.bam --num-reads 1000 test_modkit_full.txt
``` 



## Assignment 1

In ```/home/ubuntu/Share/data/bacteria/pod5``` you will find raw sequencing data done on bacterial rRNA samples (E. coli).

The run contains 12 barcodes.

bc1 and bc2 are WT strains.

bc3, bc4, bc6, and bc9 are KO strains lacking some rRNA modifications.

Find which modifications are missing in each barcode.

## Assignment 2

In ```/home/ubuntu/Share/data/yeast/pod5``` you will find raw sequencing data done on yeast rRNA samples (S. cerevisiae).

Use a modification-aware basecaller to predict pseU sites in 18S and 25S rRNA. Investigate how well the predictions match the ground truth (known modifications).

## Assignment 3

Go back to ```/home/ubuntu/Share/data/bacteria/pod5```.

Use a modification-aware basecaller to predict m5C sites. Investigate how well the predictions match the ground truth (known modifications).

