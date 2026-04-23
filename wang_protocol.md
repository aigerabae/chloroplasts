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
