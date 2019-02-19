
![alt text](assets/Capture.PNG?raw=true "header")


## **Introduction**

UVRI and MRC-University of Glasgow Center for Virus Research have organised a one day joint bioinformatics training happening on 26th Feb 2019. The training will focus on genome assemblies and taxonomic profiling from NGS data. A quick intro to linux will be offered as well.
The training is aimed at graduate students and other researchers. You don't need to have any previous knowledge of the tools that will be presented at the training.
UVRI E-Library, Plot 51-59, Nakiwogo Road Entebbe on Feb 26, 2019. Participants owning laptops can bring them along.
They do not need to have specific software packages installed since we sahll be using the UVRI server.

Contact: Please email <a href="jkayondo@uvri.go.ug">jkayondo@uvri.go.ug<a/> or <a href="assekagiri@gmail.com">assekagiri@gmail.com<a/> for more information.

Collaborative Notes will be posted here using <a href="http://pad.software-carpentry.org/2019-02-19-uvri">Etherpad</a>


## **Outline**

* Linux basics
* Tools required
* Tools for analysing metagenomic data
* Metagenome assembly
* Taxonomic identification
* Visualisation

### **Getting started**
Create a directory named "MNGS2" in your home directory. This is where the analysis will be carried out. As we go on, we shall be creating sub-directories accordingly to ensure that intermediate outputs can be refered to easily whenever needed for downstream analyses. 
```
mkdir MNGS2
```
Make a sub-directory `data` in MNGS2 and download the data. 
```
mkdir MNGS2/data
cd data
wget https://mgexamples.s3.climb.ac.uk/HMP_GUT_SRS052697.25M.1.fastq.gz
wget https://mgexamples.s3.climb.ac.uk/HMP_GUT_SRS052697.25M.2.fastq.gz
```
To confirm that the links have been successfully created, list the contents of the folder using the `ls` command. If the data is compressed (i.e .gz files), uncompress them using `gunzip *` Check to see if the data has been downloaded by using `ls -lh`. What is the size of these files?

### **Quality control check**
Here we use `fastqc` , it is necessary that we store quality control files for easy reference. In MNGS2, create a sub-directory qcresults, this is where the fastqc results will be stored.
After that, do the quality checks;
```
mkdir $HOME/MNGS2/qcresults
fastqc $HOME/MNGS2/data/*.fastq -o $HOME/MNGS2/qcresults
```
Once this is done, navigate to `qcresults` and view the ".html" files. This report can be used to assess quality distribution, length distribution, GC-content, nucleotide distribution. This informs downstream quality and adapter trimming.

### **Data quality trimming**
We use `trim_galore` for quality and adapter trimming. Depending on the qc results, it would be necessary to change some parameters used here accordingly. This script clips off the first 16 bases of the reads from the 5' end. In addition, it removes bases with phred quality less than 25 on the 3' end of the reads. We need to store quality trimmed reads, as such, in WGS directory, make a sub directory `trimmed`.
```
mkdir $HOME/WGS/trimmed
trim_galore -q 25 -l 75 --dont_gzip --clip_R1 16 --clip_R2 16 --paired ../data/HMP_GUT_SRS052697.25M.1.fastq ../data/HMP_GUT_SRS052697.25M.2.fastq -o .
```

If all goes well, trimmed reads will be available in trimmed. You may consider looking at the trimmed reads using `fastqc` to check the improvement made by `trim_galore`.

#### **Remove host sequences**
There is need to get rid of host DNA sequences that could have contaminated the data earlier in the sampling and extraction stage. This is done by mapping the reads to host reference genome and picking the unmapped sequences. 

There is a copy of the human genome on this system, so we shall share that into every one's working directory.
Create a directory `hostgenome` which will harbour the host genome.
Note: Change the genome name accordingly.

```
mkdir hostgenome
bwa index Gh37.fa
```

At this point, we create a directory to store host-free sequence data. `Samtools` and `bedtools` are the key tools used for this purpose.
```
mkdir hostfree
cd hostfree
bwa aln ../hostgenome/Gh37.fa ../data/HMP_GUT_SRS052697.25M.1_trim.fastq > HMP_GUT_SRS052697.25M.1.sai
bwa aln ../hostgenome/Gh37.fa ../data/HMP_GUT_SRS052697.25M.2_trim.fastq > HMP_GUT_SRS052697.25M.2.sai
bwa sampe ../hostgenome/Gh37.fa HMP_GUT_SRS052697.25M.1.sai HMP_GUT_SRS052697.25M.2.sai ../trimmed/HMP_GUT_SRS052697.25M.1_trim.fastq ../trimmed/HMP_GUT_SRS052697.25M.2_trim.fastq > reads12_alignment.sam
```

Extract the unmapped reads, sort them and create a corresponding bam file
```
samtools view -bS reads_mapped_and_unmapped.sam | samtools view -b -f 12 -F 256 | samtools sort -n > reads_unmapped.bam
```
Convert bam files to fastq
```
bedtools bamtofastq -i reads_unmapped.bam -fq read_1.fastq -fq2 read_2.fastq
```
Now at this point we very much hope that we dont have any host sequences sequences.

####  **De novo assembly**
Here we use `IDBA-UD` to assemble host free reads to obtain consesus contig.This program takes on a single input. As such we need to merge the forward and backward reads before asssemly.

```
mkdir $HOME/MNGS2/denovo
cd denovo
fq2fa --merge  ../hostfree/read_1.fastq ../hostfree/read_2.fastq  ../hostfree/reads_12.fa
idba_ud -r ../hostfree/reads_12.fa --num_threads 20 -o .
```

To evaluate and assess the assembly, we use `quast`. This will provide a summary of the metagenome assembly, including but not limited to N50, N75, L50, L75, GC percentage, number of contigs with size greater than 500bp (Only assesses the consensus, similar procedure can be used to assess other outputs).
```
quast.py -t 4 -f --meta contigs.fa -o .
```

### **Taxonomic classification**
