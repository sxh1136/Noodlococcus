# Long Reads

## Long Read QC
```
filtlong --min_length 1000 --keep_percent 95 noodloccous_long-reads.fastq > noodloccous_long-reads_filtered.fastq
```
## Long read assembly with Trycycler
Subset reads into 12 different samples
```
trycycler subsample --reads noodloccous_long-reads_filtered.fastq --out_dir noodloccous_long-reads_subsets
```
Assemble with 3 different assemblers (4 samples each; Flye, Miniasm+Minipolish, Raven)
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
All assemblies fit into one cluster so can proceed

### Reconciling contigs
```
trycycler reconcile --reads noodloccous_long-reads_filtered.fastq --cluster_dir trycycler/cluster_001
```
Removed "K_utg000001l.fasta" as its ends could not be found in the other assemblies
