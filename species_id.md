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

BLAST showed ~100% matches (the first 10):
                                                                  Scientific      Common                     Max       Total     Query   E   Per.   Acc.                        
Alchemilla exigua chloroplast, complete genome	Alchemilla exigua	NA	478305	129100	376000	100%	0	100	151947	=HYPERLINK("https://www.ncbi.nlm.nih.gov/nucleotide/PP316508.1?report=genbank&log$=nucltop&blast_rank=1&RID=2JFB0W95016","PP316508.1")
Alchemilla acutiloba plastid	Alchemilla acutiloba	NA	1572672	155400	281000	100%	0	99.97	126286	=HYPERLINK("https://www.ncbi.nlm.nih.gov/nucleotide/KY420009.1?report=genbank&log$=nucltop&blast_rank=2&RID=2JFB0W95016","KY420009.1")
Alchemilla monticola chloroplast, complete genome	Alchemilla monticola	NA	1532256	104800	372700	100%	0	99.68	151881	=HYPERLINK("https://www.ncbi.nlm.nih.gov/nucleotide/PP316510.1?report=genbank&log$=nucltop&blast_rank=3&RID=2JFB0W95016","PP316510.1")
Alchemilla faeroensis genome assembly, chromosome: 8	Alchemilla faeroensis	NA	478306	92464	433800	85%	0	99.6	35937374	=HYPERLINK("https://www.ncbi.nlm.nih.gov/nucleotide/OZ360586.1?report=genbank&log$=nucltop&blast_rank=4&RID=2JFB0W95016","OZ360586.1")
Alchemilla faeroensis genome assembly, chromosome: 32	Alchemilla faeroensis	NA	478306	92459	402400	85%	0	99.6	30672008	=HYPERLINK("https://www.ncbi.nlm.nih.gov/nucleotide/OZ360610.1?report=genbank&log$=nucltop&blast_rank=5&RID=2JFB0W95016","OZ360610.1")
Alchemilla faeroensis genome assembly, organelle: plastid:chloroplast	Alchemilla faeroensis	NA	478306	104600	371600	100%	0	99.59	152288	=HYPERLINK("https://www.ncbi.nlm.nih.gov/nucleotide/OZ360792.1?report=genbank&log$=nucltop&blast_rank=6&RID=2JFB0W95016","OZ360792.1")
Alchemilla transiens chloroplast, complete genome	Alchemilla transiens	NA	478348	104500	370400	100%	0	99.57	152275	=HYPERLINK("https://www.ncbi.nlm.nih.gov/nucleotide/PP316511.1?report=genbank&log$=nucltop&blast_rank=7&RID=2JFB0W95016","PP316511.1")
Alchemilla faeroensis genome assembly, chromosome: 9	Alchemilla faeroensis	NA	478306	33179	109600	35%	0	99.54	35832031	=HYPERLINK("https://www.ncbi.nlm.nih.gov/nucleotide/OZ360587.1?report=genbank&log$=nucltop&blast_rank=8&RID=2JFB0W95016","OZ360587.1")
Alchemilla microcarpa voucher Brent W. Steury 080520.1 chloroplast, complete genome	Alchemilla microcarpa	NA	1053328	38703	349500	99%	0	99.44	150586	=HYPERLINK("https://www.ncbi.nlm.nih.gov/nucleotide/NC_087047.1?report=genbank&log$=nucltop&blast_rank=9&RID=2JFB0W95016","NC_087047.1")
Alchemilla glaucescens chloroplast, complete genome	Alchemilla glaucescens	NA	667203	91633	372900	100%	0	99.33	151906	=HYPERLINK("https://www.ncbi.nlm.nih.gov/nucleotide/PP316509.1?report=genbank&log$=nucltop&blast_rank=10&RID=2JFB0W95016","PP316509.1")
<img width="2221" height="171" alt="image" src="https://github.com/user-attachments/assets/56b11a69-6a10-45c1-b63c-d6f42e3d68aa" />
 

My sequence is 152024 nucleotides long

```bash
get_organelle_from_reads.py -1 ../Chloroplast-179_S31_L001_R1_001.fastq.gz -2 ../Chloroplast-179_S31_L001_R2_001.fastq.gz -o 179_S31_plastome -R 15 -k 21,45,65,85,105 -F embplant_pt -t 20
get_organelle_from_reads.py -1 ../Chloroplast-180_S33_L001_R1_001.fastq.gz -2 ../Chloroplast-180_S33_L001_R2_001.fastq.gz -o 180_S33_plastome -R 15 -k 21,45,65,85,105 -F embplant_pt -t 20
get_organelle_from_reads.py -1 ../Chloroplast-KAB-1_S32_L001_R1_001.fastq.gz -2 ../Chloroplast-KAB-1_S32_L001_R1_001.fastq.gz -o KAB-1_S32_plastome -R 15 -k 21,45,65,85,105 -F embplant_pt -t 20
get_organelle_from_reads.py -1 ../Chloroplast-KAB-5_S3_L001_R1_001.fastq.gz -2 ../Chloroplast-KAB-5_S3_L001_R1_001.fastq.gz -o KAB-5_S3_plastome -R 15 -k 21,45,65,85,105 -F embplant_pt -t 20
get_organelle_from_reads.py -1 ../Chloroplast-KAB-6_S31_L001_R1_001.fastq.gz -2 ../Chloroplast-KAB-6_S31_L001_R1_001.fastq.gz -o KAB-6_S31_plastome -R 15 -k 21,45,65,85,105 -F embplant_pt -t 20
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
