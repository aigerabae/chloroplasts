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
