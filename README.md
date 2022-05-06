# Noodlococcus Hybrid Assembly 

## Long Read QC
```
filtlong --min_length 1000 --keep_percent 95 noodloccous_long-reads.fastq > noodloccous_long-reads_filtered.fastq
```
## Long read assembly with Trycycler
Subset reads into 12 different samples.
```
trycycler subsample --reads noodloccous_long-reads_filtered.fastq --out_dir noodloccous_long-reads_subsets
```
Assemble with 3 different assemblers (4 samples each; Flye, Miniasm+Minipolish, Raven).
```
threads=24  # change as appropriate for your system
mkdir assemblies

flye --nano-raw noodloccous_long-reads_subsets/sample_01.fastq --threads "$threads" --out-dir assembly_01 && cp assembly_01/assembly.fasta assemblies/assembly_01.fasta && rm -r assembly_01
miniasm_and_minipolish.sh noodloccous_long-reads_subsets/sample_02.fastq "$threads" > assembly_02.gfa && any2fasta assembly_02.gfa > assemblies/assembly_02.fasta && rm assembly_02.gfa
raven --threads "$threads" noodloccous_long-reads_subsets/sample_03.fastq > assemblies/assembly_03.fasta && rm raven.cereal

flye --nano-raw noodloccous_long-reads_subsets/sample_04.fastq --threads "$threads" --out-dir assembly_04 && cp assembly_04/assembly.fasta assemblies/assembly_04.fasta && rm -r assembly_04
miniasm_and_minipolish.sh noodloccous_long-reads_subsets/sample_05.fastq "$threads" > assembly_05.gfa && any2fasta assembly_05.gfa > assemblies/assembly_05.fasta && rm assembly_05.gfa
raven --threads "$threads" noodloccous_long-reads_subsets/sample_06.fastq > assemblies/assembly_06.fasta && rm raven.cereal

flye --nano-raw noodloccous_long-reads_subsets/sample_07.fastq --threads "$threads" --out-dir assembly_07 && cp assembly_07/assembly.fasta assemblies/assembly_07.fasta && rm -r assembly_07
miniasm_and_minipolish.sh noodloccous_long-reads_subsets/sample_08.fastq "$threads" > assembly_08.gfa && any2fasta assembly_08.gfa > assemblies/assembly_08.fasta && rm assembly_08.gfa
raven --threads "$threads" noodloccous_long-reads_subsets/sample_09.fastq > assemblies/assembly_09.fasta && rm raven.cereal

flye --nano-raw noodloccous_long-reads_subsets/sample_10.fastq --threads "$threads" --out-dir assembly_10 && cp assembly_10/assembly.fasta assemblies/assembly_10.fasta && rm -r assembly_10
miniasm_and_minipolish.sh noodloccous_long-reads_subsets/sample_11.fastq "$threads" > assembly_11.gfa && any2fasta assembly_11.gfa > assemblies/assembly_11.fasta && rm assembly_11.gfa
raven --threads "$threads" noodloccous_long-reads_subsets/sample_12.fastq > assemblies/assembly_12.fasta && rm raven.cereal
```
### Clustering contigs
```
trycycler cluster --assemblies assemblies/*fasta --reads noodloccous_long-reads_filtered.fastq --out_dir trycycler
```
All assemblies fit into one cluster so can proceed.

### Reconciling contigs
```
trycycler reconcile --reads noodloccous_long-reads_filtered.fastq --cluster_dir trycycler/cluster_001
```
Removed "K_utg000001l.fasta" as its ends could not be found in the other assemblies - this is normal.

### Trycycler MSA 
```
trycycler msa --cluster_dir trycycler/cluster_001
```

### Partitioning reads
```
trycycler partition --reads noodloccous_long-reads_filtered.fastq --cluster_dirs trycycler/cluster_*
```

### Generating consensus
```
trycycler consensus --cluster_dir trycycler/cluster_001
```

## Medaka consensus
```
medaka_consensus -i ../../noodloccous_long-reads_filtered.fastq -d 7_final_consensus.fasta -o medaka -m r941_min_high_g360
```

## Short Read QC
```
fastp --in1 28645_Noodlococcus_1_trimmed.fastq.gz --in2 28645_Noodlococcus_2_trimmed.fastq.gz --out1 28645_Noodlococcus_1_trimmed_fastp_fastq.gz --out2 28645_Noodlococcus_2_trimmed_fastp_fastq.gz --unpaired1 28645_Noodlococcus_unpaired_fastp.fastq.gz --unpaired2 28645_Noodlococcus_unpaired_fastp.fastq.gz

Read1 before filtering:
total reads: 584307
total bases: 98535746
Q20 bases: 96786578(98.2248%)
Q30 bases: 94782807(96.1913%)

Read2 before filtering:
total reads: 584307
total bases: 92250576
Q20 bases: 89203214(96.6966%)
Q30 bases: 86014157(93.2397%)

Read1 after filtering:
total reads: 584296
total bases: 98493152
Q20 bases: 96745337(98.2254%)
Q30 bases: 94743248(96.1927%)

Read2 after filtering:
total reads: 584296
total bases: 92067549
Q20 bases: 89055897(96.7289%)
Q30 bases: 85900893(93.302%)

Filtering result:
reads passed filter: 1168592
reads failed due to low quality: 2
reads failed due to too many N: 20
reads failed due to too short: 0
reads with adapter trimmed: 28280
bases trimmed due to adapters: 221731

Duplication rate: 12.3209%

Insert size peak (evaluated by paired-end reads): 40
```
## PolyPolish
```
bwa index consensus.fasta
bwa mem -t 16 -a consensus.fasta ../../noodlococcus_short/28645_Noodlococcus_1_trimmed_fastp_fastq.gz > alignments_1.sam
bwa mem -t 16 -a consensus.fasta ../../noodlococcus_short/28645_Noodlococcus_2_trimmed_fastp_fastq.gz > alignments_2.sam
polypolish consensus.fasta alignments_1.sam alignments_2.sam > noodlococcus_polypolish.fasta
```
## POLCA
```
polca.sh -a noodlococcus_polypolish.fasta -r "../../../noodlococcus_short/28645_Noodlococcus_1_trimmed_fastp_fastq.gz ../../../noodlococcus_short/28645_Noodlococcus_2_trimmed_fastp_fastq.gz" -t 16 -m 1G
mv noodlococcus_polypolish.fasta.PolcaCorrected.fa noodlococcus_final_assembly.fa
```

# Phylogenetic analysis
Downloaded all Kocuria genomes from RefSeq (Genbank genomes were all duplicated in RefSeq and a couple were weirdly missing from GenBank) - 
https://www.ncbi.nlm.nih.gov/datasets/genomes/?taxon=57493
