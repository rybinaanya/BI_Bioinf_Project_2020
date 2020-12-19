## Bacterial genome assembly and decontamination

*Note*: This repository describes my project performed at Bioinformatics Institute during the fall term - 2020. **something about files located here**
**Results/** - contains results of FastQC analysis, assembly statistics (QUAST output),  completeness and contamination assessment (CheckM report), 
### Background 
Maintaining balanced gut microbiota was shown to be crucial for human health. Probiotics such as *Escgerichia coli* Nissle 1917 could recover beneficial functions in gut microbial communities and prevent it from being populated with pathogenic bacteria. **LINK!!** 

*Escherichia coli* str. Nissle 1917 is one of common probiotics used for maintaining balanced gut microbiota. **LINK!!** Sample of this probiotic obtained by our laboratory at drugstore occured to be contaminated. Sample inoculation led to producing two morphologically distinct types of colonies: of small and big size. We assumed that small colonies (referred further as "Nissle Small", or simply "NS") were formed only by *E. coli* str. Nissle 1917, while big ones ("Nissle Big", or "NB") consisted of both *E. coli* str. Nissle 1917 and some other bacterial contaminants. Both colonies were subjected to NextSeq sequencing in paired-end mode with the read length of 75 nt, generating two sequencing samples: NB and NS. 

**about NCBI - there are 2 complete assembly but they differ**
In this work, we aimed to perform decontamination and *de novo* assembly of two sequencing samples obtained from the pharmaceutical *E. coli* str. Nissle 1917. 

To ,,, we objectives:
.... 
### Required programs used in the study and - check prokka and others - про комп
In this study, the following programs were used:
* FastQC v0.11.9
* SPAdes v3.13.1 
* QUAST v5.1.0rc1
* CONCOCT v1.1.0
* CheckM 
* Bowtie2 v2.2.1
* Samtools



* Kraken
* jellyfish (for running Kraken)

### Workflow
#### 1. Quality assessment of raw sequencing data (NS & NB samples)

Quality of reads was analyzed using FastQC v0.11.9 with default parameters. Variable ${working_dir} specifies the directory where all the output of data analysis would be saved (e.g. /home/rybina/Nissle_project). Variable ${path_to_reads} is the directory where raw reads in fastq format are located (e.g. /home/rybina/Nissle_project/Reads_data).  
```{bash}
fastqc -o ${working_dir}/fastqc_output ${path_to_reads}/*.fastq.gz
```
FastQC report can be found  here **Results/FastQC/**. There was no need to subject reads to trimming.

#### 2. *De novo* assembly (NS & NB)
Bacterial genome assembly was performed via SPAdes v3.13.1 (with option --careful):

```{bash}
# NB sample
spades.py --careful -o ${working_dir}/NB_spades -1 ${path_to_reads}/ARyb_NB3_S54_R1_001.fastq.gz -2 ${path_to_reads}/ARyb_NB3_S54_R2_001.fastq.gz
```
```{bash}
# NS sample
spades.py --careful -o ${working_dir}/NS_spades -1 ${path_to_reads}/ARyb_NS2_S53_R1_001.fastq.gz -2 ${path_to_reads}/ARyb_NS2_S53_R2_001.fastq.gz 
```

#### 3. Assembly statistics (NS & NB samples)
Statistics on resulting contigs and scaffolds for both samples was obtained using QUAST v5.1.0rc1. Each assembly was compared to complete genome of the *E. coli* Nissle 1917 available under GenBank assembly accession either GCA_003546975.1 (dated on 2018) or GCA_000714595.1 (dated on 2014).  Both QUAST output are located at **Results/QUAST/**

```{bash}
# Download gff files to the working directory ${working_dir}:
wget -P ${working_dir}/ https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/546/975/GCA_003546975.1_ASM354697v1/GCA_003546975.1_ASM354697v1_genomic.gff.gz 
wget -P ${working_dir}/ https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/714/595/GCA_000714595.1_ASM71459v1/GCA_000714595.1_ASM71459v1_genomic.gff.gz

# Download genome fasta files to the working directory ${working_dir}:
wget -P ${working_dir}/ https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/546/975/GCA_003546975.1_ASM354697v1/GCA_003546975.1_ASM354697v1_genomic.fna.gz 
wget -P ${working_dir}/ https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/714/595/GCA_000714595.1_ASM71459v1/GCA_000714595.1_ASM71459v1_genomic.fna.gz

# Decompressing:
gunzip ${working_dir}/GCA_000714595.1_ASM71459v1_genomic.gff.gz
gunzip ${working_dir}/GCA_003546975.1_ASM354697v1_genomic.gff.gz 
gunzip ${working_dir}/GCA_003546975.1_ASM354697v1_genomic.fna.gz
gunzip ${working_dir}/GCA_000714595.1_ASM71459v1_genomic.fna.gz

# Running QUAST: NS & NB assemblies vs GCA_003546975.1 (dated on 2018):
quast.py ${working_dir}/NS_spades/contigs.fasta ${working_dir}/NS_spades/scaffolds.fasta ${working_dir}/NB_spades/contigs.fasta ${working_dir}/NB_spades/scaffolds.fasta -r ${working_dir}/GCA_003546975.1_ASM354697v1_genomic.fna -g ${working_dir}/GCA_003546975.1_ASM354697v1_genomic.gff -o ${working_dir}/quast_output_NS_NB_Nissle2018

# Running QUAST: NS & NB assemblies vs  GCA_000714595.1 (dated on 2014):
quast.py ${working_dir}/NS_spades/contigs.fasta ${working_dir}/NS_spades/scaffolds.fasta ${working_dir}/NB_spades/contigs.fasta ${working_dir}/NB_spades/scaffolds.fasta -r ${working_dir}/GCA_000714595.1_ASM71459v1_genomic.fna -g ${working_dir}/GCA_000714595.1_ASM71459v1_genomic.gff -o ${working_dir}/quast_output_NS_NB_Nissle2014
```

#### 4. Contamination and completeness assessment (NB & NS samples)
##### 4.1. Binning (NB sample)
Binning was done using CONCOCT v1.1.0. For running CONCOCT, alignment of reads to the contigs should be provided. First, NB sample reads were mapped to the SPAdes-derived contigs via bowtie2 v2.2.1. Resulting alignment was modified using samtools 1.11. 
```{bash} 
# Indexing contigs
bowtie2-build ${working_dir}/NB_spades/contigs.fasta ${working_dir}/NBspades_contigs

# Mapping
bowtie2 -x ${working_dir}/NBspades_contigs -1 ${path_to_reads}/ARyb_NB3_S54_R1_001.fastq.gz -2 ${path_to_reads}/ARyb_NB3_S54_R2_001.fastq.gz -S ${working_dir}/NB3_reads_contigs.sam

# Sam to bam conversion
samtools view -S ${working_dir}/NB3_reads_contigs.sam -b -o ${working_dir}/NB3_reads_contigs.bam

# Sorting
samtools sort ${working_dir}/NB3_reads_contigs.bam -o ${working_dir}/NB3_reads_contigs.sorted.bam

# Indexing
samtools index ${working_dir}/NB3_reads_contigs.sorted.bam
```

Binning:
```{bash}
# Rename a copy of indexed bam file for bining
cp ${working_dir}/NB3_reads_contigs.sorted.bai ${working_dir}/NB3_reads_contigs.sorted.index.bam

# Create a folder for concoct output
mkdir ${working_dir}/concoct_output

# Cut up contigs fasta file in non-overlapping (-o 0) parts of length 10K (chunk size is 10K: -c 10000), the last contig part would be between 10K and 20K long (--merge_last); contig parts would be specified in the BED file (-b contigs_10K.bed):
cut_up_fasta.py ${working_dir}/NB_spades/contigs.fasta -c 10000 -o 0 --merge_last -b ${working_dir}/concoct_output/contigs_10K.bed > ${working_dir}/concoct_output/contigs_10K.fa

# Create coverage table based on  data on contigs parts in BED format and sorted indexed alignment file of reads to contigs:
concoct_coverage_table.py ${working_dir}/concoct_output/contigs_10K.bed ${working_dir}/NB3_reads_contigs.sorted.index.bam > ${working_dir}/concoct_output/coverage_table.tsv 

# Perform unsupervised binning of contigs
concoct --composition_file ${working_dir}/concoct_output/contigs_10K.fa --coverage_file ${working_dir}/concoct_output/coverage_table.tsv -b ${working_dir}/concoct_output/

# Merge subcontig clustering (clustering_gt1000.csv obtained by running concoct command) into original contig clustering (clustering_merged.csv):
merge_cutup_clustering.py ${working_dir}/concoct_output/clustering_gt1000.csv > ${working_dir}/concoct_output/clustering_merged.csv

# Create a folder for bins:
mkdir ${working_dir}/concoct_output/NB_bins

# Extract a fasta file for each cluster specified by concoct
extract_fasta_bins.py ${working_dir}/NB_spades/contigs.fasta ${working_dir}/concoct_output/clustering_merged.csv --output_path ${working_dir}/concoct_output/NB_bins
```
##### 4.2. Bins evaluation using single copy genes (NB & NS samples)
NB sample bins were assessed for completeness and contamination using CheckM v1.1.3. 

```{bash}
# Create a directory for CheckM output:
mkdir ${working_dir}/CheckM_output_NB

# Create a directory for CheckM database:
mkdir ${working_dir}/CheckM_database

# Download CheckM database files
wget -P ${working_dir}/CheckM_database https://data.ace.uq.edu.au/public/CheckM_databases/checkm_data_2015_01_16.tar.gz

# Decompressing
tar -xzvf ${working_dir}/CheckM_database/checkm_data_2015_01_16.tar.gz

# Set the directory where CheckM database would be located
checkm data setRoot ${working_dir}/CheckM_database

# Run workflow using lineage-specific marker sets (see https://github.com/Ecogenomics/CheckM/wiki/Workflows#lineage-specific-workflow)
checkm lineage_wf -x fa ${working_dir}/concoct_output/NB_bins ${working_dir}/CheckM_output_NB
```

To evaluate completeness and contamination of NS sample, the following commands were used (memory consuming process):
```{bash}
# Create a directory for CheckM output:
mkdir ${working_dir}/CheckM_output_NS

# Prepare a bin folder:
mkdir ${working_dir}/CheckM_output_NS/NS_bins
cp ${working_dir}/NS_spades/contigs.fasta ${working_dir}/CheckM_output_NS/NS_bins

# Run workflow using lineage-specific marker sets
checkm lineage_wf ${working_dir}/CheckM_output_NS/NS_bins ${working_dir}/CheckM_output_NS
```
Summary on bins evaluation might be found at **Results/CheckM/** for both samples. 
#### 5. Annotation (NS sample)
##### 5.1. Prokka

First, prepare output directory and additional input file:
```{bash}
# Create a directory for prokka output:
mkdir ${working_dir}/prokka_output_NS

# Download and decompress last modified curated annotation of *E. coli* Nissle 1917
wget -P ${working_dir}/ https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/546/975/GCF_003546975.1_ASM354697v1/GCF_003546975.1_ASM354697v1_genomic.gbff.gz
gunzip ${working_dir}/GCF_003546975.1_ASM354697v1_genomic.gbff.gz
```

For running prokka, several options were specified to force overwriting existing output folder (--force), specify genus and species, use filename output prefix "NS" (--prefix NS), add 'gene' features for each 'CDS' feature (--addgenes), use "Nissle" as locus tag prefix (--locustag Nissle), use genus-specific BLAST databases (--usegenus), search for ncRNAs with Infernal+Rfam (--rfam), specify Gram negative (--gram neg),and use RefSeq gbff file to first annotate from (--proteins):
```{bash}
prokka \ 
--outdir ${working_dir}/prokka_output_NS \ 
--force --genus Escherichia \ 
--species coli \ 
--strain Nissle \
--prefix NS \ 
--addgenes \ 
--locustag Nissle \ 
--kingdom Bacteria \ 
--gcode 11 \ 
--usegenus \ 
--rfam \ 
--gram neg \ 
--proteins ${working_dir}/GCF_003546975.1_ASM354697v1_genomic.gbff.gz ${working_dir}/NS_spades/scaffolds.fasta
```
**?? Можно ли выкладывать аннотацию**

##### 5.2. PGAP
Draft assembly was also annotated using PGAP. First, we need to prepare three input files: 
* input.yaml
```
fasta: 
    class: File
    location: scaffolds.fasta
submol:
  class: File
  location: submol.yaml
```

* submol.yaml
```
organism:
    genus_species: 'Escherichia coli'
    strain: 'Nissle 1917'
```

* scaffolds.fasta

All three files should be put into the folder located in pgap directory:
```{bash}
# Enter the pgap folder git-cloned from the official repository (https://github.com/ncbi/pgap):
cd ${working_dir}/PGAP/pgap

# Create here a folder where three input files would be placed:
mkdir ${working_dir}/PGAP/pgap/Nissle 

# Create input.yaml (see above):
nano ${working_dir}/PGAP/pgap/Nissle/input.yaml

# Create submol.yaml
nano  ${working_dir}/PGAP/pgap/Nissle/submol.yaml

# Copy draft assmebly to ${working_dir}/PGAP/pgap/Nissle/ directory:
cp ${working_dir}/NS_spades/scaffolds.fasta ${working_dir}/PGAP/pgap/Nissle/
```

After that, if PGAP was installed correctrly, we could run the annotation pipeline. We would save output in ${working_dir}/PGAP/Nissle_output directory, skip errors from control analysis to obtain draft annotation (--ignore-all-errors), without reporting to NCBI (-n) and self-updating (--no-self-update):

```{bash}
${working_dir}/PGAP/pgap/scripts/pgap.py --ignore-all-errors -n --no-self-update -o ${working_dir}/PGAP/Nissle_output Nissle/input.yaml
```

#### 6. 16S rRNA gene homology search (NB sample)

Comparing *E. coli* str. Nissle 1917 (GenBank accession number GCA_003546975.1) with NS sample assembly via QUAST (see above) showed an average number of mismatches per 100 kbp aligned bases in assembly equal to 0.78 while assembly alignment to another *E. coli* str. Nissle 1917 genome (GenBank accession number GCA_000714595.1) yielded value of 2.06. Thus, *E. coli* str. Nissle 1917 genome GCA_003546975.1 was chosen as a reference genome for a subsequent analysis (See **Results/QUAST**).

We assumed that NB sample reads which failed to be mapped against the reference genome might belong to bacterial contaminant(s). We obtained draft assembly out of NB reads unmapped to the reference genome and extracted 16S rRNA gene. Its homologs were searched and classified in the SILVA database (https://www.arb-silva.de/aligner/ ) (**!!! LINK**). Minimum identity with query sequence was sset to 85, other parameters were default. The best hit with 100 % identity belonged to *Bacillus cereus* group.

```{bash}
# Indexing the reference genome (specifying Nissle2018 as prefix):
bowtie2-build ${working_dir}/GCA_003546975.1_ASM354697v1_genomic.fna ${working_dir}/Nissle2018

# Mapping:
bowtie2 -x ${working_dir}/Nissle2018 -1 ${path_to_reads}/ARyb_NB3_S54_R1_001.fastq.gz -2 ${path_to_reads}/ARyb_NB3_S54_R2_001.fastq.gz -S ${working_dir}/NB_Nissle2018.sam

# Sam to bam conversion:
samtools view -b ${working_dir}/NB_Nissle2018.sam  > ${working_dir}/NB_Nissle2018.bam

# Sorting:
samtools sort ${working_dir}/NB_Nissle2018.bam -o ${working_dir}/NB_Nissle2018.sorted.bam

# Indexing:
samtools index ${working_dir}/NB_Nissle2018.sorted.bam 

# Extracting unmapped reads:
samtools view -b -F 2 ${working_dir}/NB_Nissle2018.sorted.bam > ${working_dir}/NB_Nissle2018.unmapped.bam

# Sorting unmapped reads by name (required for bamToFastq v2.27.0):
samtools sort -n ${working_dir}/NB_Nissle2018.unmapped.bam -o ${working_dir}/NB_Nissle2018.unmapped.sortedName.bam

# Bam to fastq conversion:
bamToFastq -i ${working_dir}/NB_Nissle2018.unmapped.sortedName.bam -fq ${working_dir}/NB_Nissle2018.unmappedR1.fastq -fq2 ${working_dir}/NB_Nissle2018.unmappedR2.fastq

# Assembling unmapped reads:
spades.py --careful -o ${working_dir}/NBunmapped_spades -1 ${working_dir}/NB_Nissle2018.unmappedR1.fastq -2 ${working_dir}/NB_Nissle2018.unmappedR2.fastq

# Predicting rRNA genes:
barrnap -o ${working_dir}/rrna_NBunmappedNissle2018.fa ${working_dir}/NBunmapped_spades/scaffolds.fasta

# Extracting sequence(s) of 16S rRNA gene (print line matching the pattern "16S" and the next line):
awk '/16S/{print;getline;print;}' ${working_dir}/rrna_NBunmappedNissle2018.fa > ${working_dir}/16Srrna_NBunmappedNissle2018.fa
```
#### 7. Taxonomy classification using Kraken (NB)
Kraken v1.1.1 was used to identify taxonomy of the NB sample sequences (either initial raw reads or contigs obtained in previous step **6**  (```${working_dir}/NBunmapped_spades```)). HPC cluster should be probably used, as a kraken database preparation (```kraken-build --build ``` part) is a computationally intensive process.

##### 7.1. Build a Kraken customized database 
Before runing kraken, we should create custum database which would include only RefSeq complete bacterial genomes

Create a folder with the name of a custom database that would be built (e.g., BacDB):
```{bash}
mkdir ${working_dir}/BacDB
```
Download NCBI taxonomy files (the sequence ID to taxon map, the taxonomic names and tree information). The command would create a folder ```taxonomy/``` within a custom database directory (e.g., ```${working_dir}/BacDB```): 
```{bash}
kraken-build --download-taxonomy --db ${working_dir}/BacDB
```
*Note:  If the error "rsync_from_ncbi.pl: unexpected FTP path (new server?) for na" error was thrown, the commands below might help to solve the problem (from https://github.com/DerrickWood/kraken2/issues/226):*
```{bash}
awk -v FS='\t' '$20 != "na" {print $0}' ${working_dir}/BacDB/library/bacteria/assembly_summary.txt > ${working_dir}/BacDB/library/bacteria/new_assembly_summary.txt
cp ${working_dir}/BacDB/library/bacteria/new_assembly_summary.txt ${working_dir}/BacDB/library/bacteria/assembly_summary.txt
```
*After that, comment the line ```rm -f assembly_summary.txt```  within the scipt  ```download_genomic_library.sh```,  located at ```$(which download_genomic_library.sh)``` directory.*

Finally, install the library: download all the RefSeq bacterial genomes (about 86Gb size) to a folder ```library/bacteria/``` within your custom database directory. This task is the most computationally expensive. For instance, the following command was run on the computational node with 1Tb RAM during about 2 days:
```{bash}
kraken-build --build  --threads 24 --jellyfish-hash-size 4000000 --db ${working_dir}/BacDB
```

##### 7.2. Taxonomy assignation using customized database
After constructing a custom database, the following commands for taxonomic assignation of DNA reads against the custome database could be run:
```{bash}
# If as an input we would provide scaffolds of NB reads that could not map against the reference genome (see step 6 above):
kraken --db ${working_dir}/BacDB --threads 10 --unclassified-out ${working_dir}/unclassified_NBunmappedScaffolds  --classified-out ${path_out}/classified_NBunmappedScaffolds --output ${working_dir}/kraken_NBunmappedScaffolds_kraken.output --fasta-input ${working_dir}/NBunmapped_spades/scaffolds.fasta

# If as input files we would specify raw reads of the NB sample:
kraken --db ${path_db} --threads 24 --unclassified-out ${working_dir}/unclassified_NBreads --classified-out ${working_dir}/classified_NBreads --output ${working_dir}/kraken_NBreads.output  --paired  --fastq-input --gzip-compressed ${path_to_reads}/ARyb_NB3_S54_R1_001.fastq.gz ${path_to_reads}/ARyb_NB3_S54_R2_001.fastq.gz
```

mkm | mmmkm |
    |        |
4632 sequences (7.20 Mbp) processed in 2825.719s (0.1 Kseq/m, 0.15 Mbp/m).
  4625 sequences classified (99.85%)
  7 sequences unclassified (0.15%)

6791092 sequences (1039.04 Mbp) processed in 7064.555s (57.7 Kseq/m, 8.82 Mbp/m).
  6743684 sequences classified (99.30%)
  47408 sequences unclassified (0.70%)





