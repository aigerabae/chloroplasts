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

Can only be executed in Fast-plast folder:
```bash
perl fast-plast.pl -1 /home/Chloroplast-176_S23_L001_R1_001.fastq.gz -2 /home/Chloroplast-176_S23_L001_R2_001.fastq.gz --name sample_176_s23 --bowtie_index All --coverage_analysis --clean light
```

Gives no trimmed reads so i will do fastqc manually to see if anything is wrong; i do it outside of container in chloroplasts folder
```bash
conda activate fastqc
mkdir fastqc
fastqc *fastq.gz -o fastqc_output/
# multiqc fastqc_output/
```

Strange. the quality is satisfactory and the command seems correct but it says there are no trimmed reads in the error log

Let's try again with a different sample:
docker run -it -v /mnt/harddisk/biostar/NCB/chloroplasts:/home fast-plast:v1.0 bash
cd home/Fast-Plast/
perl fast-plast.pl -1 /../Chloroplast-179_S31_L001_R1_001.fastq.gz -2 ../Chloroplast-179_S31_L001_R2_001.fastq.gz --name sample_179_s31 --bowtie_index all --coverage_analysis --threads 10

Could it be that after trimming it doesn't see any reads? In the perl file fast-plast.pl | grep Trimmomatic I get this output:
print $LOGFILE "$current_runtime\tStarting read trimming with Trimmomatic.\n\t\t\t\tUsing $TRIMMOMATIC.\n";
		my $trim_exec = "java -classpath " . $TRIMMOMATIC . " org.usadellab.trimmomatic.TrimmomaticPE -threads " . $threads . " " . $p1_array[$i] . " " . $p2_array[$i] . " " . $name."_".$i.".trimmed_P1.fq " . $name."_".$i.".trimmed_U1.fq " . $name."_".$i.".trimmed_P2.fq " . $name."_".$i.".trimmed_U2.fq " . "ILLUMINACLIP:".$adapters.":1:30:10 SLIDINGWINDOW:10:20 MINLEN:" . $min_length_trim;
		my $trim_exec = "java -classpath " . $TRIMMOMATIC . " org.usadellab.trimmomatic.TrimmomaticSE -threads " . $threads . " " . $s_array[$i] . " " . $name."_".$i.".trimmed_SE.fq " . "ILLUMINACLIP:".$adapters.":1:30:10 SLIDINGWINDOW:10:20 MINLEN:" . $min_length_trim;
Fast-Plast requires Trimmomatic, bowtie2, SPAdes, and BLAST+.
Fast-Plast is coded to use 4 threads during the Trimmomatic, bowtie2, SPAdes, and afin steps. This can simply be changed by the user if this number is not available.

Let's change values for trimming:
perl fast-plast.pl -1 /home/Chloroplast-176_S23_L001_R1_001.fastq.gz -2 /home/Chloroplast-176_S23_L001_R2_001.fastq.gz --name sample_176_s23 --bowtie_index all --coverage_analysis --threads 10 --min_length_trim 50 --adapters TruSeq

Didn't work. Looking at the data, the issue might be in trail of Ns in the beginning of the sequence. I adjusted the fast-plast.pl code and saved original as fast-plast-original.pl
perl fast-plast.pl -1 /home/Chloroplast-176_S23_L001_R1_001.fastq.gz -2 /home/Chloroplast-176_S23_L001_R2_001.fastq.gz --name sample_176_s23 --bowtie_index all --coverage_analysis --threads 10 --min_length_trim 50 --adapters TruSeq

Doesn't work. Let's skip trimming:
perl fast-plast.pl -1 /home/Chloroplast-176_S23_L001_R1_001.fastq.gz -2 /home/Chloroplast-176_S23_L001_R2_001.fastq.gz --name sample_176_s23 --bowtie_index all --coverage_analysis --threads 10 --skip trim
