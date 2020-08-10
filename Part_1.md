### Getting the RNA-seq data

For the tutorial, we will use the bulk RNA-seq data from ageing mouse tissues (Tabula Muris project). The data is published here https://www.nature.com/articles/s41586-020-2499-y.
From all the tissues, we select the Bone Marrow and Spleen data for 3 ages (1,12,24 m), both male and female. 

To download Sequence Read Archive (SRA) data, firstly get sratoolkit [here](https://github.com/ncbi/sra-tools/wiki/01.-Downloading-SRA-Toolkit) and install it on the server.
Get the SRR files list [here](https://github.com/mmetsger/RNA-seq-tutorial/blob/master/SRR_acc_list.txt)

Next, get SRA RNA-seq data splitted in 2 reads (forward and reverse):

 ```bash
 cat SRR_acc_list.txt | parallel --jobs 10 "~/software/sratoolkit.2.9.1-1-ubuntu64/bin/fastq-dump --origfmt --gzip --split-files {}" :::
 gunzip *.fastq.gz
 
 ```
### FASTQ quality control
 In fastq folder
 
 ```bash
 fastqc -t 10 *.fastq.gz
 multiqc./
 ```

 
### Getting Mouse reference and transcriptome gtf annotation.

```{bash}

mkdir Mouse_Ref_GRCm38
cd Mouse_Ref_GRCm38 

wget ftp://ftp.ensembl.org/pub/release-100/fasta/mus_musculus/dna/Mus_musculus.GRCm38.dna.primary_assembly.fa.gz

wget ftp://ftp.ensembl.org/pub/release-100/gtf/mus_musculus/Mus_musculus.GRCm38.100.gtf.gz

```
### Get and install RSEM on the server

https://github.com/deweylab/RSEM
https://github.com/bli25broad/RSEM_tutorial

```{bash}

mkdir rsem
cd rsem
wget https://github.com/deweylab/RSEM/archive/v1.3.3.tar.gz
tar -xzf v1.3.3.tar.gz
cd RSEM-1.3.3
make
make ebseq
make install

```

### Prepare transcript indexed references for RSEM (bowtie2 -based)

```{bash}

cd Mouse_Ref_GRCm38 

/home/*USERNAME*/*path_to_rsem*/rsem-prepare-reference --gtf Mus_musculus.GRCm38.100.gtf \
                            --bowtie2 \
                             Mus_musculus.GRCm38.dna.primary_assembly.fa \
                            ./Mus_Musculus_ref_RSEM_2020
      
```


### FASTQ files trimming (removing low quality bases and adapters

Get and install Trimmomatic software [here](http://www.usadellab.org/cms/index.php?page=trimmomatic)

```{bash}
ls -1 *_1.fastq | sed 's:_1.fastq::' | parallel --line-buffer -j 20 'java -jar /home/mmetsger/software/Trimmomatic-0.36/trimmomatic-0.36.jar PE {}_1.fastq {}_2.fastq \
{}_1_paired.fastq {}_1_unpaired.fastq {}_2_paired.fastq {}_2_unpaired.fastq \
ILLUMINACLIP:NexteraPE-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36'

```
### RSEM pseudoalignment and quantififcation


```{bash}

ls -1 *_1_paired.fastq | sed 's:_1_paired.fastq::' | parallel --line-buffer -j 5 'rsem-calculate-expression -p 4 
-paired-end -bowtie2 --estimate-rspd {}_1_paired.fastq {}_2_paired.fastq /home/mmetsger/resources/Mus_Musculus_GRCm38/Mus_Musculus_ref_RSEM {}.rsem > {}.log 2> {}.err'

```














