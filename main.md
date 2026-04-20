Following https://github.com/mrmckain/Fast-Plast#fast-plast-rapid-de-novo-assembly-and-finishing-for-whole-chloroplast-genomes
 
I didn't find any paprs that used it but I will give it a try
 
git clone https://github.com/mrmckain/Fast-Plast.git

I will make a docker container with the installation:

create my own container
```bash
mkdir my-docker-project & cd my-docker-project
touch Dockerfile
nano Dockerfile
```

```Dockerfile
FROM ubuntu:24.04 AS builder
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y python3
```

```bash
docker build -t my-docker-image .
```

```bash
docker run -it -v /mnt/harddisk/biostar/NCB/chloroplasts:/home my-docker-image bash
```

Inside shell
```bash
cd home/
apt-get update && apt-get install -y perl
apt install python3-pip
apt install python3-h5py
cd Fast-Plast/
perl INSTALL.pl
```

For questions state No (meaning you don't have any dependencies installed), then All (It will install all dependencies)

Outside of that shell (in normal command line)
```bash
docker commit d625476cce65 fast-plast:v1.0
```

perl fast-plast.pl -1 /home/Chloroplast-176_S23_L001_R1_001.fastq.gz -2 /home/Chloroplast-176_S23_L001_R2_001.fastq.gz --name sample_176_s23 --bowtie_index All --coverage_analysis --clean light

Gives no trimmed reads so i will do fastqc manually to see if anything is wrong; i do it outside of container in chloroplasts folder
```bash
conda activate fastqc
mkdir fastqc
fastqc *fastq.gz -o fastqc_output/
multiqc fastqc_output/
```
