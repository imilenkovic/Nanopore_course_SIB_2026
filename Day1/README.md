
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
 pod5 inspect reads subset_yeast.pod5 | head
```

Count the total number of groups/datasets:

```bash
pod5 inspect summary subset_yeast.pod5
```

Now, do the same with the mouse data!


##  Hands-on I - basecalling with dorado and mapping with minimap2 in the MOP pipeline (yeast rRNA)

Navigate to the pre-processing directory:

```bash
cd ~/master_of_pores/mop_preprocess
```

Edit the  `params.yaml` file

```bash
pod5: /home/[your_username]/data/yeast/pod5/yeast_subset.pod5
basecalling: "dorado"
demultiplexing: "seqtagger"
reference: /home/[your_username]/references/yeast_rRNA_ref.fa
ref_type : transcriptome
output: /home/[your_username]/output_mop/yeast_dorado_fast
dorado: "fast"
```


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

##  Hands-on II - Indirect RNA modification mapping in rRNA

### Run Epinano on the MoP output

```
epinano_variants -b BAM -r REFERENCE -c 1 -o output_folder_name
```

### Compare two epinano output files

```
Rscript scatterplot_script.R file_1.csv file_2.csv output.pdf
```


### Compare fast vs hac

Now do this on the bam file made with the hac dorado model, and compare bc_19 (fast) vs bc_19 (hac)


##  Hands-on III - Basecall your mouse pod5 files with Dorado m6A DRACH

Navigate to the pre-processing directory:

```bash
cd ~/master_of_pores/mop_preprocess
```

Edit the  `params.yaml` file

```bash
pod5: /home/[your_username]/data/mouse/CTR/CTR.pod5
basecalling: "dorado-mod"
demultiplexing: "NO"
reference: /home/[your_username]/references/chr19.fa
annotation: /home/[your_username]/references/chr19_annotation.gtf
ref_type : genome
counting: htseq
output: /home/[your_username]/output_mop/mRNA_CTR_m6A
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

Now, do the same with the KO file :) 

## Extra: Some useful commands to explore the bam files with samtools 

```bash
samtools view file.bam | wc –l 
samtools view –F 4 file.bam | wc –l 
samtools view –s 0.1 file.bam | wc –l

samtools view –H file.bam | less
samtools view –h file.bam > file.sam

samtools view –Sb file.sam > new_file.bam
```

##  Documentation

Full documentation of Master of Pores:  
[(https://biocorecrg.github.io/master_of_pores/MOP4-dev/)]
