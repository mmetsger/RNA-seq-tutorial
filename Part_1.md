### Getting the RNA-seq data

For the tutorial, we will use the bulk RNA-seq data from ageing mouse tissues (Tabula Muris project). The data is published here https://www.nature.com/articles/s41586-020-2499-y.
From all the tissues, we select the Bone Marrow and Spleen data for 3 ages (1,12,24 m), both male and female. SRR files list [here](https://github.com/mmetsger/RNA-seq-tutorial/blob/master/SRR_acc_list.txt)

Geeting SRA splitted in 2 reads :

 ```bash
 cat SRR_acc_list.txt | parallel --jobs 10 "~/software/sratoolkit.2.9.1-1-ubuntu64/bin/fastq-dump --origfmt --gzip --split-files {}" :::
 ```


