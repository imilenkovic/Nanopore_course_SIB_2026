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
samtools view pod5---bc_19_s.bam |  head -n 20 | cut -f 1,3,4,25,26
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

