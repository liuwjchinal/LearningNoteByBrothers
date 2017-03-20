BioSV
An accurate and efficient tool for structural variation calling and genotyping
###Authors: Li Zhang
Wubin Ding
Wenqiang Liu
Introduction
Whole genome sequencing (WGS) of parent-offspring trio samples has enabledthe identification of inherited DNA changes. Family-based sequencing studiesfor rare or complex inheritable diseases can facilitate the understanding ofpathogenic mechanism. However, most of the existing structural variation (SV)detection tools are designed to call somatic SVs, and rarely considergenotyping SVs, which hinders the identification of germline or de novopathogenic SVs. Here we present a novel tool, BioSV, to call and genotype SVsfrom WGS data. BioSV integrates read depth, split-reads and discordant readpairs to identify potential breakpoints, and build a binomial mixture model tocomprehensively genotype SVs. For both simulated and real WGS datasets,BioSV outperforms currently popular tools in SV calling and Delly in SVgenotyping. Moreover, BioSV successfully identifies two candidate pathogenicdeletions in the proband from WGS datasets of a family with autism spectrumdisorder. BioSV is freely available at http://www.megabionet.org/BioSV.
User manual of BioSV
1 Overview
Breakpoint-based identification of Structural Variants (BioSV), is an accurate and efficient SV caller, which not only uses split-reads and discordant read pairs for SV prediction, but also integrates discordant and concordant read pairs (fragments) to genotype SVs under a statistical framework. Specifically, BioSV also provides a family-based SV caller for the analyses of parent-child trio based WGS studies. Moreover, BioSV exhibits high performance on both simulated and real WGS data in SV calling and genotyping.
2 Requirements and installation
•	(1) Make sure awk, R programming tool and python2 or higher versions are available
•	(2) Download BioSV.tar.gz and decompress the packagetar zxvf BioSV.tar.gz
•	(3) Change to BioSV directory and install dependencies (bwa, bedtools, samtools, wgsim):cd BioSV sh Install.sh
•	(4) Install python packages:pandas (>=0.14.1)pip install pandasscipy (>=0.18.0)pip install scipypysam (>=0.8.4)pip install pysam
3 Documentation
3.1 Structural variation calling
python BioSV.py –h
•	Usage: python BioSV.py <-b|-1(-2)> -o out_dir -g genome [options]
•	BioVS: Breakpoint-based Identification Of Structure Variation
Options:
--version Show program's version number and exit
-1 , -2 or -b  Fastq or bam format files (Required)
-g /--genome= Reference genome sequence file (Required)
-t/--thread= Threads for samtools and bwa [1]
-o/--out_dir= Output directory, default= ./bioSV/--PE Flag on paired-end sequencing [default= False]
-l/--length= Minimum length of soft-clipped end [20]
-u/--uniqMapRate= Uniqueness score=(AS-XS)/AS [0], AS and XS are mapping tags by bwa mem.
-q/--mapq= Minimum mapping quality for the assessment of mapping uniqueness [0]
--minRC= Minimal read counts covering candidate SV breakpoints, including SV or reference-supported reads/RPs [5]
--maxRC= Maximal read counts covering candidate SV breakpoints, including SV or reference-supported reads/RPs [500]
--minMRC= Minimal mutant read counts [2]
-c CHROM Only detected SVs from chromosomes concerned File, One chromosome per line.
--MAF= Mutant allele frequency for duplication calling [0.05]-
-samtools= The executable file of samtools
--bwa= The executable file of bwa
--bedtools= The executable file of bedtools
-h/--help Show this help message and exit
####For example:
cd example
python ../BioSV.py -1 ./example_dir/example_N1.fq -2 ./example_dir/example_N2.fq -g ucsc.hg19.fasta -o ./example_dir/biosv_normal -t 4
####Output of BioSV:
•	(1) BioSV.bedpe: raw structural variation events detected by BioSV.
•	(2) BioSV.log: command lines of BioSV.
•	(3) bwaMap.bam: bam file mapped by bwa if -1 and -2 assigned.
•	(4) temp directory: directory containing temporary files
####Description of fields from BioSV.bedpe:
•	(1) Chromosome of the left part of SV event.
•	(2) Start position of the left breakpoint.
•	(3) End position of left breakpoint.
•	(4) Chromosome of the right part of SV event.
•	(5) Start position of the right breakpoint.
•	(6) End position of right breakpoint.
•	(7) Pattern of structural variation: TRA(translocation), DEL (deletion), INV (inversion), DUP (duplication)
•	(8) Strands: left strand + right strand and number of it.
•	(9) totalCount: the total read count of the SV event, including pair-end reads.
•	(10) Read_count: read count of SV event, not including pair-end reads.
•	(11) Mean_mapq: mean mapping quality of SV reads.
•	(12) covI: read coverage of the left part of SV event.
•	(13) covII: read coverage of the right part of SV event.
•	(14) P: Probability of genotyping.
•	(15) Genotype: genotype of the structural variation.
•	(16) uniqMapReadCountI: the uniquely mapped read count of left SV event. The uniquely mapped read is defined as UniScore >0, in which, UniScore=(AS-XS)/AS
•	(17) uniqMapReadCountII: the uniquely mapped read count of right SV event.
•	(18) SeqI: Sequence of left part of SV event.
•	(19) SeqII: Sequence of right part of SV event.
3.2 SV genotyping correction and high confidence SV filtering
Rscript GermlineSVs.R Bedpe-file Bam-file maxLen
Note: the bedpe file should be included in BioSV output directory.
R script ‘GermlineSVs.R’ aims to correct genotypes and filter false positives.
Bedpe-file Bedpe file generated by BioSV.py in Section 3.1
Bam-file Bam file for the tested sample
maxLen Maximum length of deletions for genotyping correction (default=1e6-bp)
####Output files:
•	(1) BioSV.hc.bedpe: BEDPE format of high confidence SVs
•	(2) BioSV.hc.rawformat.bedpe: Raw BEDPE format of high confidence SVs.
3.3 Trio-based SV calling
Rscript Trio.SV.calling.R fa-hc.raw.bedpe mo-hc.raw.bedpe offspring-hc.raw.bedpe minRD maxRC output-directory
R script ‘Trio.SV.calling.R’ aims to call denovo SVs and homozygous deletions. Prior to trio SV calling, users should perform SV genotyping correction for each sample of the family in Section 3.2.
fa-hc.raw.bedpe High confidence bedpe file for father
mo-hc.raw.bedpe High confidence bedpe file for mother
offspring-hc.raw.bedpe High confidence bedpe file for offspring
minRD Minimum read depth
maxRC Maximum read count supporting SVs for wild type calling
output-directory Output directory
####Output files:
•	trio_SVs.bedpe: BEDPE format file of trio-based SV calling, which includes three types of SVs, de novo, homozygous and father/mother transmitted SVs in the offspring.
•	De novo SVs (DENOVO): wild in the parents but mutated in the offspring;
•	Homozygous SVs (Homozygous): heterozygous in the parents but homozygous in the offspring;
•	SVs transmitted from father or mother (FA/MO): heterozygous in both the offspring and one of the parents. The three types of SVs were potentially pathogenic in recessive diseases.
3.4 Structural variation simulation from diploid genomes
python BioSV_simulator.py –h
•	Usage: python BioSV_simulator.py -G -O <out_dir> [options]
•	BioSV_simulator: Simulate structural variation from diploid genomes and output paired-end fastq files
Options:
--version Show program's version number and exit
-g/--Genome= Reference genome sequence file (Required)
-p/--prefix= Prefix of output files
-o/--out dir= Output directory-t Generate tumor-normal matched fastq files for somatic SV calling
-b/--bed= Using user-supplied instead of randomly generated SV events from bed file
-n/--num= Vector containing number of SV events, each element of which is seperated by commas and ranked in the order of ('DEL','INS','INV','TRA','DUP') default=[2500,0,2500,2500,2500]
-m/--mu= The parameter (mean) of Poisson distribution for DUP (copy number) [2]
-c/--chrom=CHROM Only simulate structural variation on chromosomes provided by the file (one chromosome per line)
--sv_len= Range of SV length [100,10000]
--wgsim= The executable file of wigsim
-e/--error rate= Base error rate [0.02]
-d/--distance= Minimal distance between the two ends from different SV events [350]
-s/--standard deviation= Standard deviation for insert size
-N= Number of read pairs [150000000]-l= Read length [100]
-r= Mutation/SNP rate [0.0010]
-R= Fraction of INDELs [0.15]
-X= Probability an INDEL is extended [0.30]
-S= Seed for random generator [-1]
-H Haplotype mode [False]
-h/--help Show this help message and exit
####For example:
cd example
python ../BioSV_simulator.py -g genome.fasta -o example_dir -p example -c chromosomes.txt -n 50,0,50,50,50 -N 2000000
####Output files of BioSV_simulator:
•	(1) example_N1.fq, example_N2.fq, paired-end fastq files of simulated normal tissue.
•	(2) example_T1.fq, example_T2.fq, paired-end fastq files of simulated tumor tissue.
•	(3) example_T.bed, example_N.bed, bed files for genomic coordinates of simulated structural variations.
•	(4) example_N.bedpe, example_T.bedpe, bedpe format files for structural variation events.
•	(5) example_Nsnp.txt, example_Tsnp.txt, the snps generated by wgsim.
####Description of fields from example_T.bed :
•	(1) Left chromosome of SV event to be simulating.
•	(2) Start position of left SV event.
•	(3) End position of left SV event.
•	(4) Right chromosome of SV event to be simulating.
•	(5) Start position of right SV event.
•	(6) End position of right SV event.
•	(7) Strand of the inserted part of genome.
•	(8) Genotype of the SV event.
•	(9) Somatic or germline for the SV event.
Description of fields from example_T.bedpe :
•	(1) Left chromosome of simulated SV event.
•	(2) Start position of the left breakpoint.
•	(3) Position of left breakpoint.
•	(4) Right chromosome of simulated SV event.
•	(5) Start position of the right breakpoint.
•	(6) Position of right breakpoint.
•	(7) Strand of left part of SV event.
•	(8) Strand of right part of SV event.
•	(9) Pattern of the simulated SV event.
•	(10) Genotype of simulated SV event.
•	(11) Simulated somatic or germline SV events.
4 IMPORTANT NOTICE
Users should NOT move the files in the output directory created by BioSV.py and scripts in “BioSV” and “BioSV/tools” to any other directories, otherwise, some errors may occur.
#Copyright Copyright (c) 2017, Wubin Ding, Li Zhang (East China Normal University).
If you would like to report any bugs when you running BioSV, don't hesitate to create an issue on github here, or email me:ding_wu_bin@163.com

