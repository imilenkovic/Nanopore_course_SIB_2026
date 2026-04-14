
#  Master of Pores – Day 1 Demultiplexing, Basecalling and Mapping

This guide walks through the first steps of exploring **Master of Pores** and inspecting example nanopore input data.



Connect to the cluster by running:
```
ssh -i your_key.pem your_username@3.126.104.186
```




First - check if you have nextflow installed.

```
nextflow -version
```
If not, install it this way:
```
cd ~
wget https://github.com/nextflow-io/nextflow/releases/download/v25.04.0/nextflow
chmod +x nextflow
mkdir -p ~/bin
mv nextflow ~/bin/
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source .bashrc
```
---


##  Explore Input Data (pod5)

Navigate to the input directory:

```bash
cd ~/data/yeast/pod5

```

List contents of a `.pod5` file:

```bash
 pod5 inspect reads | head
```

Count the total number of groups/datasets:

```bash
pod5 inspect summary subset_yeast.pod5
```


##  Hands-on I - basecalling with dorado and mapping with minimap2 in the MOP pipeline (yeast rRNA)

Navigate to the pre-processing directory:

```bash
cd ~/master_of_pores/mop_preprocess
```

Edit the  `params.yaml` file

Run the pipeline! (make sure you are in ~/master_of_pores/mop_preprocess)

```bash
nextflow run mop_preprocess.nf -params-file params.yaml -with-singularity -profile local -bg > output.log
```


If it fails? Resume!

```bash
nextflow run mop_preprocess.nf -params-file params.yaml -with-singularity -profile local -bg -resume > output2.log
```

If you need to kill it? (don’t restart if you haven’t killed the previous processes!)

```bash
cat .nextflow.pid

> 1234

kill 1234
```

##  Hands-on II - running epinano on the MOP output

```
Epinano_Variants.py [-h] -b BAM -r REFERENCE -c 1 -o output_file_name
```


##  Hands-on III - Basecall your mouse pod5 files with Dorado m6A DRACH

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
Output: ../../output_mop/mRNA_CTR_m6A
dorado-mod: "hac,m6A_DRACH"
```
Run the pipeline!

```bash
nextflow run mop_preprocess.nf -params-file params.yaml -with-singularity -profile local -bg > log_mRNA_CTR_m6A.txt
```

Now, do the same with the KO file :) 

##  Hands-on IV - Extract m6A information from the bam files with modkit

Navigate to the alignment directory:

```bash
cd ~/output_mop/mRNA_CTR_m6A/alignment
```

Extract modification frequency per position with Modkit

```bash
modkit pileup CTR_s.bam ../modkit/CTR_m6A_pileup.bed --log-filepath ../modkit/CTR_m6A_pileup.log
```

Extract modification probability per position per read with Modkit

```bash
modkit extract full CTR_s.bam --num-reads 1000 ../modkit/CTR_modkit_full.txt
``` 

Now, do the same with the KO file :) 

##  Basecall your yeast pod5 files with Dorado 

Navigate to the pre-processing directory:

```bash
cd ~/MOP4/mop_preprocess
```

Edit the  `params.yaml` file

Dataset 2: 
pod5 files - rRNA from S. cerevisiae

Share/data/yeast/pod5   -> choose one pod5 file from this folder

Reference:
S. cerevisiae rRNA sequence (you will find it in the Share/references folder)

Ref_type : transcriptome
Counting: nanocount


seqtagger: "-k b96_RNA004"

bc19 and bc20 - WT yeast (replicates)


Run the pipeline!

```bash
nextflow run mop_preprocess.nf -params-file params.yaml -with-singularity --nv -profile local -bg > demultiplexing.log
```

##  Run the pipeline starting from fastq files

Navigate to the pre-processing directory:

```bash
cd ~/MOP4/mop_preprocess
```

Edit the  `params.yaml` file

- Remove the path to pod5 files, and only put a path to fastq files (you can use the fastq files generated in the output folder of one of your previous folders)

- Explore the output (it should contain a folder called alignment in which you will find the bam files, and a folder called counts in which you will find the counts)

## Explore the bam files with samtools 

Some useful commands: 

```bash
samtools view file.bam | wc –l 
samtools view –F 4 file.bam | wc –l 
samtools view –s 0.1 file.bam | wc –l

samtools view –H file.bam | less
samtools view –h file.bam > file.sam

samtools view –Sb file.sam > new_file.bam
```

##  Documentation

Full documentation:  
[[https://biocorecrg.github.io/master_of_pores/](https://biocorecrg.github.io/master_of_pores/MOP4-dev/index.html)]
