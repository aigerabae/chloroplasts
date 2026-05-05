## SHORT-READ ONLY ASSEMBLY

### 1\_pre\_assembly
### QualityControl

Run fastqc to check the quality
```
./1_pre_assembly/1_qualityControl/shortRread/1_qualityCheck/run_fastqc.sh
```
Trim adaptor and low quality region (<30). Read length <50 bp will be removed. 
```
./1_pre_assembly/1_qualityControl/shortRead/2_adapterTrim/run_bbduk.sh 
```
Rerun fastqc to check the data again
```
./1_pre_assembly/1_qualityControl/shortRead/3_qualityCheck/run_fastqc.sh 
```

### cp\_DNA\_extraction

Mapping all trimmed reads to refs (31 known Eucalyptus chloroplast genomes, all double-up, in case read maps to the 'cut-point') to get the chloroplast reads, using bowtie2.
```
./1_pre_assembly/2_cpDNAExtraction/shortRead/1_run_bowtie2.sh 
```
Get chloroplast read from the Bowtie2 output
```
./1_pre_assembly/2_cpDNAExtraction/shortRead/2_getCPRead.py 
```

### 2\_assembly
Short-reads were randomly selected (5x, 8x, 10x, 20x, 40x, 60x, 80x, 100x, 200x, 300x, 400x and 500x, assuming the genome size is 160kb). 100x coverage of short-read was seprated first as the validation data which did not use in assembly.
```
./2_assembly/randomSelection/split_pair_read.sh
```
Unicycler is used to do the short-read only assembly with different coverage. In the default setting, the read is corrected by SPAdes. In this study, we have tried three different read correction: SPAdes, Karect and SPAdes+Karect. However, the different error-correction pipelinne performed very similar.

get Karect-correct read
```
./2_assembly/run_Karect_correction.sh
```
For SPAdes-correct read assembly, using normal read as input:
```
./2_assembly/shortReadOnly/run_shortRead_unicycler.sh 
```
For Karect-correct read assembly, using the Karect-correct read as input:
```
./2_assembly/shortReadOnly/run_shortRead_unicycler_noSPAdes.sh 
```
For Karect-SPAdes-correct read assembly, using the Karect-correct read as input:
```
./2_assembly/shortReadOnly/run_shortRead_unicycler.sh 
```
### 3\_post\_assembly
### mummer
as described above in long-read only assembly part.

Run direction.py
```
./3_post_assembly/1_same_structure/direction.py 
```
Run mummer\_plot.sh
```
./3_post_assembly/1_same_structure/mummer_direction.sh 
```
### polish
we used Pilon to polish the assembly, run until result unchanged.
```
./3_post_assembly/2_polish/run_pilon.sh 
```
### assembly\_quality\_control
As described above in the long-read only assembly part，we used the 100x short-read (randomly selected first, not used in assembly, method see below) to remap to the assembly to assess its quality. Qualimap was used to grep the mapping information.
```
./3_assembly_quality_control/1_run_bowtie2.sh 
./3_assembly_quality_control/2_run_qualimap.sh 
```
