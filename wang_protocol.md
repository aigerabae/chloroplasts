Following protocol for short read assembly from https://pmc.ncbi.nlm.nih.gov/articles/PMC6311037/

```bash
git clone https://github.com/asdcid/Chloroplast-genome-assembly
```

In the same docker container as fast-plast:
```bash
apt install curl
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

In another window:
```bash
docker commit db5c6a8acba7 fast-plast:v2.0
```

```bash
docker run -it -v /mnt/harddisk/biostar/NCB/chloroplasts:/home fast-plast:v2.0
conda create --name wang
conda activate wang
conda install bioconda::bbmap
```

Script for bbduk trimming:
```bash
mkdir /home/wang_protocol/trimmed
export PATH="/home/wang_protocol/":$PATH
inputf='/home/'
outputtrimreads='/home/wang_protocol/trimmed/'
minlen=50
adaptors='/root/miniconda3/envs/wang/share/bbmap/resources/adapters.fa'
trimq=30
threads=40
for in1 in $(find $inputf -name "*R1_001.fastq.gz"); do
    in2=${in1%%R1_001.fastq.gz}"R2_001.fastq.gz"
    echo "running bbduk on"
    echo $in1
    echo $in2

    f1=$(basename ${in1%%R1.fastq.gz}"R1.trim.fastq.gz")
    f2=$(basename ${in1%%R1.fastq.gz}"R2.trim.fastq.gz")

    out1=$outputtrimreads/$f1
    out2=$outputtrimreads/$f2
    sampleid=$outputtrimreads${f1%%R1.trim.fastq.gz}

    bbduk.sh in1=$in1 in2=$in2 out1=$out1 out2=$out2 minlen=$minlen k=25 mink=8 ktrim=r ref=$adaptors hdist=1 overwrite=f qtrim=rl trimq=$trimq t=$threads bhist=$sampleid"bhist.txt" qhist=$sampleid"qhist.txt" gchist=$sampleid"gchist.txt" aqhist=$sampleid"aqhist.txt" lhist=$sampleid"lhist.txt" > $sampleid"bbduk_log.txt"

done
```

fastqc again:
```bash
conda install bioconda::fastqc
mkdir /home/wang_protocol/trimmed/fastqc_result
fastqc /home/wang_protocol/trimmed/*fastq.gz -o /home/wang_protocol/trimmed/fastqc_result
```

I will use chlorplast genome from https://www.ncbi.nlm.nih.gov/datasets/organelle/?taxon=3750 as reference genome
I saved it into wang_protocol/ref
```bash
sed -n '/^>NC_061549.1/,/^>/p' wang_protocol/ref/genome.fna | head -n -1 > wang_protocol/ref/single_ref.fna
# because it has both ">NC_061549.1 Malus domestica cultivar Red Delicious-ww chloroplast" and ">OK458681.1 Malus domestica cultivar Red Delicious-ww chloroplast"

Doubling up genome
# 1. Get the header
grep ">" wang_protocol/ref/single_ref.fna > wang_protocol/ref/doubled_genome.fna

# 2. Get the sequence, remove line breaks, repeat it, and save it
grep -v ">" wang_protocol/ref/single_ref.fna | tr -d '\n' > wang_protocol/ref/sequence_only.txt
cat wang_protocol/ref/sequence_only.txt wang_protocol/ref/sequence_only.txt >> wang_protocol/ref/doubled_genome.fna

Assembly:
conda install bioconda::karect

I manually copied trimmed reads and saved them into inoutkarect folder. i also renamd them to remove R1_001.fastq.gz in the name and gunzipped them
```bash
gunzip input_karect/*fastq.gz 
```

Made result_assembly folder  
```bash
mkdir -p wang_protocol/result_assembly/
mkdir -p temp
```

Trying for one sample
```bash
threads=40
outputDir='wang_protocol/result_assembly/'
temp='temp'

mkdir $temp

karect \
    -correct \
     -inputfile='input_karect/Chloroplast-176_S23_L001_R1.trim.fastq' \
     -inputfile='input_karect/Chloroplast-176_S23_L001_R2.trim.fastq' \
     -resultdir=$outputDir \
    -tempdir=$temp \
    -threads=$threads \
    -celltype='haploid' \
    -matchtype='hamming'
```

Others:
```bash
karect \
    -correct \
     -inputfile='input_karect/Chloroplast-179_S31_L001_R1.trim.fastq' \
     -inputfile='input_karect/Chloroplast-179_S31_L001_R2.trim.fastq' \
     -resultdir=$outputDir \
    -tempdir=$temp \
    -threads=$threads \
    -celltype='haploid' \
    -matchtype='hamming'

karect \
    -correct \
     -inputfile='input_karect/Chloroplast-180_S33_L001_R1.trim.fastq' \
     -inputfile='input_karect/Chloroplast-180_S33_L001_R2.trim.fastq' \
     -resultdir=$outputDir \
    -tempdir=$temp \
    -threads=$threads \
    -celltype='haploid' \
    -matchtype='hamming'

karect \
    -correct \
     -inputfile='input_karect/Chloroplast-KAB-1_S32_L001_R1.trim.fastq' \
     -inputfile='input_karect/Chloroplast-KAB-1_S32_L001_R2.trim.fastq' \
     -resultdir=$outputDir \
    -tempdir=$temp \
    -threads=$threads \
    -celltype='haploid' \
    -matchtype='hamming'

karect \
    -correct \
     -inputfile='input_karect/Chloroplast-KAB-5_S3_L001_R1.trim.fastq' \
     -inputfile='input_karect/Chloroplast-KAB-5_S3_L001_R1.trim.fastq' \
     -resultdir=$outputDir \
    -tempdir=$temp \
    -threads=$threads \
    -celltype='haploid' \
    -matchtype='hamming'

karect \
    -correct \
     -inputfile='input_karect/Chloroplast-KAB-6_S31_L001_R1.trim.fastq' \
     -inputfile='input_karect/Chloroplast-KAB-6_S31_L001_R1.trim.fastq' \
     -resultdir=$outputDir \
    -tempdir=$temp \
    -threads=$threads \
    -celltype='haploid' \
    -matchtype='hamming'

ls 
```

Next step - assembly with unicycler (from hybrid section)

In the end will annotate with Prokka
