### Maker (version 2.31.8)
#### Required files:
1. Assembly file called ```g.morbida.scaffolds.fasta```.
2. ***Fusarium solani*** (version v2.0.26) cdna and proteins files.
3. CEGMA generated ```gff``` output. This file is provided in the github repo. 
4. SNAP trained ```hmm``` output.   
5. Provided ```makerwrap.sh```.

#### A. Train SNAP with CEGMA generated ```gff``` output

```
mkdir snap_1
cd snap_1
/maker/bin/cegma2zff g.morbida.output.cegma.gff g.morbida.scaffolds.fasta
/snap/fathom -categorize 1000 genome.ann genome.dna
/snap/fathom -export 1000 -plus uni.ann uni.dna
/snap/forge export.ann export.dna
/snap/hmm-assembler.pl g.morbida . > ../g.morbida_1.hmm
cd ..
```
#### B. Generate control files
```
maker -CTL
```
#### Download 
#### MIGHT NOT NEED the makerPrep.py script!
#### Fix headers in ALLPaths-LG generated assembly file

``` 
python3 makerPrep.py --fasta g.morbida.scaffolds.fasta \
--output g.morbida.scaffolds.fixed.fasta
```
#### C. Prepare ```maker_opts.ctl``` by changing the following lines in the file

```
#-----Genome (these are always required)
genome=~/g.morbida.scaffolds.fasta #genome sequence (fasta file or fasta embeded in GFF3 file)
.
.
.
#-----EST Evidence (for best results provide a file for at least one)
est= #set of ESTs or assembled mRNA-seq in fasta format
altest=~/Fusarium_solani.v2.0.26.cdna.all.fa #EST/cDNA sequence file in fasta format from an alternate organism
est_gff= #aligned ESTs or mRNA-seq from an external GFF3 file
altest_gff= #aligned ESTs from a closly relate species in GFF3 format

#-----Protein Homology Evidence (for best results provide a file for at least one)
protein=~/Fusarium_solani.v2.0.26.pep.all.fa #protein sequence file in fasta format (i.e. from mutiple oransisms)
protein_gff=  #aligned protein homology evidence from an external GFF3 file
.
.
.
#-----Gene Prediction
snaphmm=~/g.morbida_1.hmm #SNAP HMM file
gmhmm= #GeneMark HMM file
augustus_species=fusarium #Augustus gene prediction species model
fgenesh_par_file= #FGENESH parameter file
pred_gff= #ab-initio predictions from an external GFF3 file
model_gff= #annotated gene models from an external GFF3 file (annotation pass-through)
est2genome=1 #infer gene predictions directly from ESTs, 1 = yes, 0 = no
protein2genome=1 #infer predictions from protein homology, 1 = yes, 0 = no
trna=0 #find tRNAs with tRNAscan, 1 = yes, 0 = no
snoscan_rrna= #rRNA file to have Snoscan find snoRNAs
unmask=0 #also run ab-initio prediction programs on unmasked sequence, 1 = yes, 0 = no
.
.
.
#-----MAKER Behavior Options
max_dna_len=3000000 #length for dividing up contigs into chunks (increases/decreases memory usage)
min_contig=10000 #skip genome contigs below this length (under 10kb are often useless)
.
.
.
```

#### D. First Maker run (run1)
```
mkdir g.morbida.run1.maker.output
cd g.morbida.run1.maker.output
./makerwrap.sh 16 g.morbida.run1
```

#### E. Train SNAP using Maker generated ```gff``` output 
```
maker/bin/fasta_merge -d g.morbida.run1.maker.output/g.morbida.run1_master_datastore_index.log
maker/bin/gff3_merge -d g.morbida.run1.maker.output/g.morbida.run1_master_datastore_index.log
cd ../
mkdir snap_2
cd snap_2
maker2zff ~/g.morbida.run1.maker.output/g.morbida.run1.all.gff
/opt/snap/fathom -categorize 1000 genome.ann genome.dna
/opt/snap/fathom -export 1000 -plus uni.ann uni.dna
/opt/snap/forge export.ann export.dna
/opt/snap/hmm-assembler.pl g.morbida_run1 . > ../g.morbida_2.hmm
cd ../
```
#### F. Copy the ```maker_opts.ctl``` to a different file
```
scp maker_opts.ctl maker_opts_run1.ctl
```
#### G. Prepare ```maker_opts.ctl``` again by changing the following lines in the file. Note only one line is edited this time. 

```
.
.
.
#-----Gene Prediction
snaphmm=~/g.morbida_2.hmm #SNAP HMM file
gmhmm= #GeneMark HMM file
augustus_species=fusarium #Augustus gene prediction species model
fgenesh_par_file= #FGENESH parameter file
pred_gff= #ab-initio predictions from an external GFF3 file
model_gff= #annotated gene models from an external GFF3 file (annotation pass-through)
est2genome=1 #infer gene predictions directly from ESTs, 1 = yes, 0 = no
protein2genome=1 #infer predictions from protein homology, 1 = yes, 0 = no
trna=0 #find tRNAs with tRNAscan, 1 = yes, 0 = no
snoscan_rrna= #rRNA file to have Snoscan find snoRNAs
unmask=0 #also run ab-initio prediction programs on unmasked sequence, 1 = yes, 0 = no
.
.
.
```


#### H. Run Maker (run2)
```
mkdir g.morbida.run2.maker.output
cd g.morbida.run2.maker.output
makerwrap.sh 16 g.morbida.run2
```
#### I. Repeat F-H for a third Maker run and merge all the fastas and gffs one last time.

#### J. Convert structural fastas and gffs into functional files using a database of choice.

```
/maker/bin/maker_functional_fasta uniprot.sprot.fasta output.txt g.morbida.run3.all.maker.transcripts.fasta > g.morbida.transcripts.fasta
```

```
/maker/bin/maker_functional_fasta uniprot.sprot.fasta output.txt g.morbida.run3.all.maker.proteins.fasta > g.morbida.proteins.fasta
```
```
/maker/bin/maker_functional_gff uniprot.sprot.fasta output.txt g.morbida.run3.all.gff > g.morbida.gff
```