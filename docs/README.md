## **Getting onto the server**
Each of the users has been assigned login details which includes a username and password. To log onto the uvri server, we use ssh (secure shell) as shown in the command below.

```
ssh username@galaxy.uvri.go.ug
```
Notes:
* `username`: Assigned user name for example `student1`
* `galaxy.uvri.go.ug`: name of the server that we are using for this training.

On pressing `enter`, you will be prompted to enter your password and once that is done, you will be logged onto the server and ready to start working. On successful login, your screen will have something that this ;

```
[username@h3abionet ~]$
```

## **Introduction to Linux**
This section includes the usage of some common linux commands that are useful in bioinformatics. 

* `pwd`: print working directory command writes the full pathname of the current working directory. Typing `pwd` followed by `enter` will print the following `/home/username`. This command is important in identifying current location on the file structure. 

* `touch`: This command is used for creating empty files. For example, the following command creates an empty file `test`.
`touch test`

* `ls`: This command is used to list contents of a given directory. 

* `cd`: change directory command is used for moving from one working directory to another. 

* `man`: This is the manual command, it is used to check usage of a given command, for example, `man ls` gives details of how `ls` is used.

* `mkdir`: make directory command is used for creating a directory, for example `mkdir UVRI` creates a directory named UVRI inside the current working directory. 

* `rmdir`: This command is used to delete/remove an **empty** directory

* `rm `: This command removes files and directories; 

* `ln`: The link command creates shortcuts to desired files and directories 

* `less`: used to navigate/look through files.  

Each users' home  includes a directory called `Data` with three sub-directories; `Ebola`, `Dengue` and `HCV`. Use the `cd` and `ls` commands to navigate into each of these and list the contents of each of them. As an example, we illustrate how to do this with `HCV`. Assuming that you are at the home directory, `cd Data/HCV` will take you to `HCV` directory. There are a couple of files in this directory, as of now we are most interested in the raw reads (identified with a file extension of `.fq` or `.fastq`).

## **Quality control**
To check the quality of the raw reads, we use `fastqc` program. In this case, we use the graphical interface to look at these data.
```
fastqc &
```

After inspecting raw data quality, proceed to trim off poor quality reads using `trim_galore`.

```
trim_galore -q 25 --length 75 --paired hcv-bad1.fq hcv-bad2.fq
```
## **Reference mapping**

```
tanoti -p 1 -r H77.fa -i hcv-bad1_val_1.fq hcv-bad2_val_2.fq -o hcv.sam
```

## **De novo assembly**

```
spades.py -1 hcv-gap-1.fq -2 hcv-gap-2.fq -o spades_out2 -k 31 --only-assembler
```

## **Contig consolidation**

```
 contigMerger ../../H77.fa contigs.fasta
```

## **Kraken classification of NGS reads**

```
kraken2 --db /home/mars/Training/DBs --report kraken.out hcv-bad1_val_1.fq hcv-bad2_val_2.fq > res
cut -f2,3 res > res.tmp
ktImportTaxonomy res.tmp -o res.html
firefox res.html
```
