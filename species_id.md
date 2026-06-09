I made a directory ~/biostar/NCB/chloroplasts/species_id

```bash
conda create --name chloroplasts
conda activate chloroplasts
conda install bioconda::seqtk
seqtk sample -s 42 ../Chloroplast-176_S23_L001_R1_001.fastq.gz 500 | seqtk seq -a - > subsample.fasta
```

A simple cat species_blast1.txt | grep ">" | sort | uniq -c | head
 on blast results over subsample doesn't give it because they are all named after chromosomes or assemblies; need to strip that first
