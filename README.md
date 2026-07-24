
Synthesized bedtools methodology for generating GC content files
You can generate the hg38_50kb_gc.bed file yourself with bedtools — it's not something you download pre-made, since it depends on your exact bin size and genome build. Here's the standard workflow:

1. Get the hg38 reference and chromosome sizes
bash
# Reference genome
wget https://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.fa.gz

gunzip hg38.fa.gz

# Chromosome sizes
wget https://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.chrom.sizes



If you only want main chromosomes (no alt/random/chrUn contigs), filter first:

bash
grep -E '^chr([1-9]|1[0-9]|2[0-2]|X|Y)\s' hg38.chrom.sizes > hg38.main.chrom.sizes



2. Make 50kb genome-wide windows

bash
bedtools makewindows -g hg38.main.chrom.sizes -w 50000 > hg38_50kb_windows.bed



3. Compute GC content per window

bash
bedtools nuc -fi hg38.fa -bed hg38_50kb_windows.bed > hg38_50kb_nuc.txt
This adds columns including pct_gc (fraction GC), pct_at, N count, etc.

4. Extract just chrom/start/end/GC into a clean bed

bash
awk 'BEGIN{OFS="\t"} NR>1{print $1,$2,$3,$5}' hg38_50kb_nuc.txt > hg38_50kb_gc.bed
(Check the column index for pct_gc with head -1 hg38_50kb_nuc.txt since it can shift depending on your bedtools version — usually it's the 5th column.)

That gives you a 4-column bed: chrom  start  end  gc_fraction, which is what control_compartment_analysis.py --gc-bed expects (assuming it's used the same way most A/B compartment sign-correction scripts use it — as a GC reference to orient eigenvector sign, since compartment A is typically GC-richer).
