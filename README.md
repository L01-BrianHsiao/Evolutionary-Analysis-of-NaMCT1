# Methods
Analysis of NaMCT1 Protein

Lab 5
```bash
ncbi-acc-download -F fasta -m protein XP_001619353.2 
#Download protein sequences using succession in NCBI

blastp -db allprotein.fas -query XP_001619353.2 -outfmt 0 -max_hsps 1 -out NaMCT1.blastp.typical.out 

less NaMCT1.blastp.typical.out 

blastp -db allprotein.fas -query XP_032239066.1.fa -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident stitle"  -max_hsps 1 -out maguk.blastp.detail.out
#Find homologs of NaMCT1 protein sequence

awk '{if ($6<0.00000000000001)print $1 }' NaMCT1.blastp.detail.out > NaMCT1.blastp.detail.filtered.out
#Remove homologs with evalue greater than 10e-14, giving us the putative homologs of NaMCT1

seqkit grep --pattern-file NaMCT1.blastp.detail.filtered.out allprotein.fas > NaMCT1.blastp.detail.filtered.fas
#Change file format from .out to .fas

muscle -in NaMCT1.blastp.detail.filtered.fas -out NaMCT1.blastp.detail.filtered.aligned.fas
#Align the sequence of our protein

t_coffee -other_pg seq_reformat -in NaMCT1.blastp.detail.filtered.aligned.fas -output sim
#Provide statistics of our aligne sequences including percent identity.

t_coffee -other_pg seq_reformat -in NaMCT1.blastp.detail.filtered.aligned.fas -action +rm_gap 50 -out NaMCT1.blastp.detail.filtered.aligned.r50.fas
#Removes columns containing greater than 50% gapped residues

```

Lab 6
```bash

sed "s/ /_/g" NaMCT1.blastp.detail.filtered.aligned.fas > NaMCT1.blastp.detail.filtered.aligned_.fas
#Replaces spaces in our NaMCT1 fasta file with underscores so the spaces dont get deleted.

iqtree -s NaMCT1.blastp.detail.filtered.aligned_.fas -nt 2
#Gives us maximum likelihood tree estimate, a tree file, summary of tree search, and the substitution model we use for our tree file and tree estimate.

gotree reroot midpoint -i NaMCT1.blastp.detail.filtered.aligned_.fas.treefile -o NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile
#Reroots the tree at the midpoint, such that the root of the tree is half the length of the longest lineage of the tree.

nw_display -s  NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile -w 1000 -b 'opacity:0' > NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile.svg
#View the midpoint rooted tree
```

Lab 7
```bash
java -jar ~/tools/Notung-3.0-beta/Notung-3.0-beta.jar -s speciesTreeBilateriaCnidaria.tre -g NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile --reconcile --speciestag prefix --savepng --events
#Reconcile our species tree with our rerooted mipoint tree in lab6 using notung.

python2.7 ~/tools/recPhyloXML/python/NOTUNGtoRecPhyloXML.py -g NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile.reconciled --include.species
#Generate the reconciled tree as a xml file

thirdkind -f NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile.reconciled.xml -o  NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile.genes.tre.reconciled.svg
#Reformat the renconciled tree from xml to svg

java -jar ~/tools/Notung-3.0-beta/Notung-3.0-beta.jar -s speciesTreeBilateriaCnidaria.tre -g NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile --root --speciestag prefix --savepng --events
#Minimizes the gene duplications and losses present in our reconciled tree and reroots it based on those changes.

iqtree -redo -s NaMCT1.blastp.detail.filtered.aligned_.fas -bb 1000 -nt 2 -m LG+F+I+G4
 -t NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile -pre NaMCT1.genes.ufboot
#reruns our iqtree support then adds bootstrap support and calculates a score for all possible trees, ultimately giving us one optimal tree with the highest score.

gotree reroot midpoint -i NaMCT1.blastp.detail.filtered.aligned_.fas.treefile -o NaMCT1.genes.midpoint.ufboot
#Reroots our tree given the bootstrap data we had just calculated for. This tree factors in bootstrap support.

java -jar ~/tools/Notung-3.0-beta/Notung-3.0-beta.jar -s speciesTreeBilateriaCnidaria.tre -g NaMCT1.genes.midpoint.ufboot --root --speciestag prefix --savepng --events
#Minimizes the gene duplications and losses present in our newly produced reconciled tree and reroots it.
```

Lab 8
```bash

iprscan5   --email brian.hsiao@stonybrook.edu  --multifasta --useSeqId --sequence   NaMCT1.blastp.detail.filtered.fas
#Sends Sends our protein sequence to the inteproscan servers for analysis

cat ~/labs/lab8-L01-BrianHsiao/myfamily/*.tsv.tsv > ~/labs/lab8-L01-BrianHsiao/NaMCT1.domains.all.tsv
#Concatenates all the .tsv files for all the species produced from inteproscan analysis into one .tsv file

grep Pfam ~/labs/lab8-L01-BrianHsiao/NaMCT1.domains.all.tsv >  ~/labs/lab8-L01-BrianHsiao/NaMCT1.domains.pfam.tsv
#grep filters out domains that aren't defined on the Pfam database. Grep will only include domains that have a definition on the Pfam database in our tsv file.

awk 'BEGIN{FS="\t"} {print $1"\t"$3"\t"$7"@"$8"@"$5}' ~/labs/lab8-L01-BrianHsiao/NaMCT1.domains.pfam.tsv | datamash -sW --group=1,2 collapse 3 | sed 's/,/\t/g' | sed 's/@/,/g' > ~/labs/lab8-L01-BrianHsiao/NaMCT1.domains.pfam.evol.tsv

```
