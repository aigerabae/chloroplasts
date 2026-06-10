I made a directory ~/biostar/NCB/chloroplasts/species_id

```bash
conda create --name chloroplasts
conda activate chloroplasts
conda install bioconda::seqtk
seqtk sample -s 42 ../Chloroplast-176_S23_L001_R1_001.fastq.gz 500 | seqtk seq -a - > subsample.fasta
```

A simple cat species_blast1.txt | grep ">" | sort | uniq -c | head
 on blast results over subsample doesn't give it because they are all named after chromosomes or assemblies; need to strip that first

cat species_blast1.txt | grep ">" | grep "chloroplast" | sed -E 's/ (chromosome|genome|complete|chloroplast|isolate|voucher|var.|strain|subsp.|sp.|cultivar|plastid).*//' | sort | uniq -c | sort -n
This results in 719 different species with Dasiphora fruticosa with 793 matches being the most common entry; this should show specifically matches on chloroplasts (tho it might miss those that just say genome assembly)

cat species_blast1.txt | grep ">"  | sed -E 's/ (chromosome|genome|complete|chloroplast|isolate|voucher|var.|strain|subsp.|sp.|cultivar|plastid).*//' | sort | uniq -c | sort -n | wc -l
This results in 2787 entries and the mostcommon entry is Dasiphora fruticosa with 849 entries

cat species_blast1.txt | grep ">" | grep "Dasiphora fruticosa" | wc -l
Overall Dasiphora fruticosa is matched 854 times out of 19896 matches made

I get recommended to use GetOrganelle for assembly. They recommend using adapter-trimmed raw reads without quality control for input. Based on my fastqc output there should be no adapters already
```bash
conda activate chloroplasts
conda install -c bioconda getorganelle
get_organelle_config.py -a embplant_pt,embplant_mt
get_organelle_from_reads.py -1 ../Chloroplast-176_S23_L001_R1_001.fastq.gz -2 ../Chloroplast-176_S23_L001_R2_001.fastq.gz -o 176_S23_plastome -R 15 -k 21,45,65,85,105 -F embplant_pt -t 20
# i first used it without multithreading so i had to also add --continue for the second run
```

BLAST showed that 176_S23_L001_R1_001 has ~100% matches with several species Alchemilla exigua chloroplast:


My sequence is 152024 nucleotides long

```bash
get_organelle_from_reads.py -1 ../Chloroplast-179_S31_L001_R1_001.fastq.gz -2 ../Chloroplast-179_S31_L001_R2_001.fastq.gz -o 179_S31_plastome -R 15 -k 21,45,65,85,105 -F embplant_pt -t 20
get_organelle_from_reads.py -1 ../Chloroplast-180_S33_L001_R1_001.fastq.gz -2 ../Chloroplast-180_S33_L001_R2_001.fastq.gz -o 180_S33_plastome -R 15 -k 21,45,65,85,105 -F embplant_pt -t 20
get_organelle_from_reads.py -1 ../Chloroplast-KAB-1_S32_L001_R1_001.fastq.gz -2 ../Chloroplast-KAB-1_S32_L001_R2_001.fastq.gz -o KAB-1_S32_plastome -R 15 -k 21,45,65,85,105 -F embplant_pt -t 20
get_organelle_from_reads.py -1 ../Chloroplast-KAB-5_S3_L001_R1_001.fastq.gz -2 ../Chloroplast-KAB-5_S3_L001_R2_001.fastq.gz -o KAB-5_S3_plastome -R 15 -k 21,45,65,85,105 -F embplant_pt -t 20
get_organelle_from_reads.py -1 ../Chloroplast-KAB-6_S31_L001_R1_001.fastq.gz -2 ../Chloroplast-KAB-6_S31_L001_R2_001.fastq.gz -o KAB-6_S31_plastome -R 15 -k 21,45,65,85,105 -F embplant_pt -t 20
ls
```

I will have them running one after another; 1 sample took 395.24 s, so total should be done in 2000 seconds = 33 mins

I will be annotating the first sample with Prokka in the meantime:
```bash
conda create -n prokka bioconda::prokka
conda activate prokka
prokka --version
```

Nevermind. Prokka is not well suited for chloroplast annotation

BLAST results:

1 - 176_S23: Alchemilla exigua chloroplast (2nd result by p-value but has higher percent identity and identical length ~150k)  
2 - 179_S31: Alfredia cernua chloroplast, complete genome  
3 - 180_S33: Echinops exaltatus chloroplast, complete genome  
4 - KAB-1_S32:	Tulipa alberti chloroplast, complete genome  
5 - KAB-5_S3: Tulipa butkovii plastid, complete genome  
6 - KAB-6_S31: Tulipa greigii chloroplast, complete genome (didn't assemble well; has 5 scaffolds and 4/5 match with Tulipa greigii)

Annotation with GeSeq:
Tillich M, Lehwark P, Pellizzer T, Ulbricht-Jones ES, Fischer A, Bock R and Greiner S (2017) GeSeq – versatile and accurate annotation of organelle genomes. Nucleic Acids Research 45: W6-W11
8223 = 176_S23 (5)
121749 = 179_S31 (3)
3763 = 180_S33 (1)
63081 = KAB-1_S32 (2)
2775+ = KAB-5_S3 (4)
KAB-6_S31 = no single circular plastid
