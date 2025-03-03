


1. Circular Consensus generating, primer removal and demultiplexing were performed by the sequencing company using the following options for `ccs (4.0.0)` and `lima (1.10.0)`:
```bash
ccs filtered.chunk0.subreadset.xml out.consensusreadset.xml --log-level INFO --noPolish --minLength 50 --maxLength 15000 --minPasses 1 --minSnr 2.5 --maxPoaCoverage 10 --minPredictedAccuracy 0.8 --draft-mode winpoa --disable-heuristics --task-report task-report.json -j 15
lima -j 15 --isoseq --peek-guess gathered.consensusreadset.xml smrtlink_barcodes.barcodeset.xml fl.datastore.json
```
2. Afterwards, reads were refined with `refine (1.14.99)` from the ISOSeq pipeline and mapped against the European robin reference genome with `pbmm2 (1.19.2)`, sorted and concatenated with `samtools (1.20)`:
```bash
refine LID50556/fl.datastore.bc1001_5p--bc1001_3p.bam isoseq_primers.fasta datastore.bc1001_5p--bc1001_3p_1.flnc.bam --require-polya
refine LID50556/fl.datastore.bc1002_5p--bc1002_3p.bam isoseq_primers.fasta datastore.bc1002_5p--bc1002_3p_2.flnc.bam --require-polya
refine LID50557/fl.datastore.bc1003_5p--bc1003_3p.bam isoseq_primers.fasta datastore.bc1003_5p--bc1003_3p_3.flnc.bam --require-polya
refine LID50557/fl.datastore.bc1004_5p--bc1004_3p.bam isoseq_primers.fasta datastore.bc1004_5p--bc1004_3p_4.flnc.bam --require-polya
refine LID50564/fl.datastore.bc1001_5p--bc1001_3p.bam isoseq_primers.fasta datastore.bc1001_5p--bc1001_3p_5.flnc.bam --require-polya
refine LID50564/fl.datastore.bc1002_5p--bc1002_3p.bam isoseq_primers.fasta datastore.bc1002_5p--bc1002_3p_6.flnc.bam --require-polya
refine LID50565/fl.datastore.bc1003_5p--bc1003_3p.bam isoseq_primers.fasta datastore.bc1003_5p--bc1003_3p_7.flnc.bam --require-polya
refine LID50565/fl.datastore.bc1004_5p--bc1004_3p.bam isoseq_primers.fasta datastore.bc1004_5p--bc1004_3p_8.flnc.bam --require-polya
for i in 1 2 3 4; do
    pbmm2 align bEriRub.mmi datastore.bc100${i}_5p--bc100${i}_3p_${i}.flnc.bam -j 8 --preset ISOSEQ | samtools sort -@2 > bEriRub_${i}_1.sorted.bam
    pbmm2 align bEriRub.mmi datastore.bc100${i}_5p--bc100${i}_3p_$(($i + 4)).flnc.bam -j 8 --preset ISOSEQ | samtools sort -@2 > bEriRub_${i}_2.sorted.bam
done

samtools cat bEriRub_1_1.sorted.bam bEriRub_1_2.sorted.bam bEriRub_2_1.sorted.bam bEriRub_2_2.sorted.bam bEriRub_3_1.sorted.bam bEriRub_3_2.sorted.bam bEriRub_4_1.sorted.bam bEriRub_4_2.sorted.bam
samtools sort -o bEriRub_joined.sorted.bam bEriRub_joined.bam
```
3. Then, the Cry4 sequence was blasted against the reference genome to get a rough genomic position and from that the `Cry4.bed` file was created (also provided in the repository). With that, we extract the ISOSeq reads and their barcodes from that region.
```bash
samtools view --regions-file Cry4.bed bEriRub_joined.sorted.bam | cut -f 1,13 > Cry4.barcodes.txt
```
4. This region was then manually reviewed in [IGV](https://igv.org/) and reads were manually assigned to their ISO form (in `Cry4.categories.txt`)
5. Also, a gff file was manually created for the different ISOforms found (in `Cry4.gff`), from which the exons where extracted using `bedtools`:
```bash
bedtools getfasta -fi ~/work/genomes/bEriRub/bEriRub.fasta -fo Cry4.fasta -bed Cry4.gff -split -name
```
6. The exons where then manually assembled into the different transcript sequences for `Cry4_all.fasta`.
