# RNA-seq for Ewing Sarcoma cells. cDNA sequencing and direct RNA sequencing.
# 26, Aug., 2025
# Masahiko Ajiro

# environment
Python 3.12.0
dorado 0.7.3
minimap2 2.26-r1175
samtools 1.19.2
igv 2.19.1
modkit 0.6.1

####################
ONT cDNA-sequencing

# basecalling
# MinKNOW 24.11.10
# flowcell FLO-PRO114M
# library SQK-PCS114
    
# filtering:
    Q >= 9
    length >= 200

# aligment
minimap2 -t <Int> \
-ax splice \
--eqx \
--secondar=yes \
--MD \
--cs=long \
--splice-flank=no \
<GRCh38.fa> <input.fa> \
> <output.sam>

samtools sort -@ <Int> \
-o <output.bam> \
-O BAM \
-T <prefix> \
<output.sam>

samtools index -@ <Int> \
-b \
<output.bam> \
<output.bam.bai>

# visualization
igv 2.19.1

####################
ONT direct RNA-sequencing

# pod5 data acquisition
# MinKNOW 24.11.10
# flowcell FLO-PRO04RA
# library SQK-RNA004
    
# basecalling
dorado basecaller hac,m6A pod5_dir/ \
--verbose \
--mm2-preset splice \
-k 14 \
> <output.bam> \
2> log.txt

# alignment
dorado aligner <GRCh38.fa> \
<output.bam> \
> <aligned.bam>

samtools sort \
-@ <Int> \
-o <sorted_aligned.bam> \
<aligned.bam>

samtools index \
-@ <Int> \
-b \
<sorted_aligned.bam> \
<sorted_aligned.bam.bai>

# m6A modification quantification

modkit pileup <sorted_aligned.bam> <sorted_aligned.bed> \
--modified-bases m6A \
-t <Int> \
--reference GRCh38.fa \
--mod-threshold a:0.3

###########################
short-read RNA-seq analysis

# emvironment
Python 3.12.0
samtools 1.19.2
STAR 2.7.10b
stringtie 2.2.1

# index preparation
STAR --runThreadN <Int> \
--runMode genomeGenerate \
--genomeDir <index_name> \
--genomeFastaFiles GRCh38.fa \
--sjdbGTFfile <GRCh38.gtf> \
--sjdbOverhang 99

# alignment
STAR --runThreadN <Int> \
--twopassMode Basic \
--limitBAMsortRAM <Int> \
--outSAMtype BAM SortedByCoordinate (optional; --readFilesCommand gunzip -c) \
--genomeDir <index> \
--outFileNamePrefix <prefix> \
--readFilesIn <fastq_files>

# indexing
samtools index -@ <Int> <output_bam>

# assembly
stringtie -p <Int> -e -G <reference> \
-o <output_gtf> <output_bam>
prepDE.py3 -i <gtf_list> -l 100
