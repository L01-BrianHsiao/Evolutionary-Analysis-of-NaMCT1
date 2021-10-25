# Methods
Analysis of NaMCT1 Protein

Lab 5
```bash
ncbi-acc-download -F fasta -m protein XP_001619353.2 

blastp -db allprotein.fas -query XP_001619353.2 -outfmt 0 -max_hsps 1 -out NaMCT1.blastp.typical.out 

less NaMCT1.blastp.typical.out 

blastp -db allprotein.fas -query XP_032239066.1.fa -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident stitle"  -max_hsps 1 -out maguk.blastp.detail.out

awk '{if ($6<0.00000000000001)print $1 }' NaMCT1.blastp.detail.out > NaMCT1.blastp.detail.filtered.out

wc -l NaMCT1.blastp.detail.filtered.out

seqkit grep --pattern-file NaMCT1.blastp.detail.filtered.out allprotein.fas > NaMCT1.blastp.detail.filtered.fas

muscle -in NaMCT1.blastp.detail.filtered.fas -out NaMCT1.blastp.detail.filtered.aligned.fas

t_coffee -other_pg seq_reformat -in NaMCT1.blastp.detail.filtered.aligned.fas -output sim

alv -k NaMCT1.blastp.detail.filtered.aligned.fas | less -r 

alv -kli --majority NaMCT1.blastp.detail.filtered.aligned.fas | less -RS

t_coffee -other_pg seq_reformat -in NaMCT1.blastp.detail.filtered.aligned.fas -action +rm_gap 50 -out NaMCT1.blastp.detail.filtered.aligned.r50.fas

alv -kli --majority NaMCT1.blastp.detail.filtered.aligned.r50.fas | less -RS
```

Lab 6
```bash

cp ../lab5-L01-BrianHsiao/NaMCT1.blastp.detail.filtered.aligned.fas .

sed "s/ /_/g" NaMCT1.blastp.detail.filtered.aligned.fas > NaMCT1.blastp.detail.filtered.aligned_.fas

iqtree -s NaMCT1.blastp.detail.filtered.aligned_.fas -nt 2

gotree reroot midpoint -i NaMCT1.blastp.detail.filtered.aligned_.fas.treefile -o NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile
    
nw_display -s  NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile -w 1000 -b 'opacity:0' > NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile.svg
```

Lab 7
```bash
java -jar ~/tools/Notung-3.0-beta/Notung-3.0-beta.jar -s speciesTreeBilateriaCnidaria.tre -g NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile --reconcile --speciestag prefix --savepng --events

python2.7 ~/tools/recPhyloXML/python/NOTUNGtoRecPhyloXML.py -g NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile.reconciled --include.species

thirdkind -f NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile.reconciled.xml -o  NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile.genes.tre.reconciled.svg

java -jar ~/tools/Notung-3.0-beta/Notung-3.0-beta.jar -s speciesTreeBilateriaCnidaria.tre -g NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile --root --speciestag prefix --savepng --events

iqtree -redo -s NaMCT1.blastp.detail.filtered.aligned_.fas -bb 1000 -nt 2 -m LG+F+I+G4
 -t NaMCT1.blastp.detail.filtered.aligned_.fas.midpoint.treefile -pre NaMCT1.genes.ufboot

gotree reroot midpoint -i [name your output gene tree].ufboot -o [name your output gene tree].midpoint.ufboot

gotree reroot midpoint -i NaMCT1.blastp.detail.filtered.aligned_.fas.treefile -o NaMCT1.genes.midpoint.ufboot

java -jar ~/tools/Notung-3.0-beta/Notung-3.0-beta.jar -s speciesTreeBilateriaCnidaria.tre -g NaMCT1.genes.midpoint.ufboot --root --speciestag prefix --savepng --events
```

Lab 8
```bash
mkdir ./myfamily

cp ~/labs/lab5-L01-BrianHsiao/NaMCT1.blastp.detail.filtered.fas ~/labs/lab8-L01-BrianHsiao/myfamily

iprscan5   --email brian.hsiao@stonybrook.edu  --multifasta --useSeqId --sequence   NaMCT1.blastp.detail.filtered.fas

cat ~/labs/lab8-L01-BrianHsiao/myfamily/*.tsv.tsv > ~/labs/lab8-L01-BrianHsiao/NaMCT1.domains.all.tsv

grep Pfam ~/labs/lab8-L01-BrianHsiao/NaMCT1.domains.all.tsv >  ~/labs/lab8-L01-BrianHsiao/NaMCT1.domains.pfam.tsv

awk 'BEGIN{FS="\t"} {print $1"\t"$3"\t"$7"@"$8"@"$5}' ~/labs/lab8-L01-BrianHsiao/NaMCT1.domains.pfam.tsv | datamash -sW --group=1,2 collapse 3 | sed 's/,/\t/g' | sed 's/@/,/g' > ~/labs/lab8-L01-BrianHsiao/NaMCT1.domains.pfam.evol.tsv
```
