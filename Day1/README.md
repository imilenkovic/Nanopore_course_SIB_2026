
#  Master of Pores – Day 1 Setup & Exploration

This guide walks through the first steps of exploring **Master of Pores** and inspecting example nanopore input data.


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
cd ~/Share/data/mouse/pod5

```

List contents of a `.pod5` file:

```bash
 pod5 inspect reads | head
```

Count the total number of groups/datasets:

```bash
pod5 inspect summary mouse_mRNA.pod5
```


##  Demultiplex your pod5 files with Seqtagger

Navigate to the pre-processing directory:

```bash
cd ~/MOP4/mop_preprocess
```

Edit the  `params.yaml` file

Run the pipeline!

```bash
nextflow run mop_preprocess.nf -params-file params.yaml -with-singularity -profile local -bg > demultiplexing.log
```


If it fails? Resume!

```bash
nextflow run mop_preprocess.nf -params-file params.yaml -with-singularity -profile local -bg -resume > demultiplexing2.log
```

If you need to kill it? (don’t restart if you haven’t killed the previous processes!)

```bash
cat .nextflow.pid

> 1234

kill 1234
```

##  Documentation

Full documentation:  
[[https://biocorecrg.github.io/master_of_pores/](https://biocorecrg.github.io/master_of_pores/MOP4-dev/index.html)]
