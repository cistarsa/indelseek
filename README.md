INDELseek
=========
Detect complex indels from next-generation sequencing reads

[Download latest version in a ZIP package](https://github.com/tommyau/indelseek/zipball/master)

INDELseek algorithm (Au _et al._ _BMC Genomics_ 2017 18:16 [Figure 1](http://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-016-3449-9#Fig1))
![INDELseek algorithm](http://static-content.springer.com/image/art%3A10.1186%2Fs12864-016-3449-9/MediaObjects/12864_2016_3449_Fig1_HTML.gif)

### Dependencies, as tested on 64-bit CentOS 5.5
* [SAMtools](http://www.htslib.org/download/) version at least 1.3 ([to support depth >8000x](https://github.com/samtools/samtools/pull/322))
* [GNU Parallel](http://www.gnu.org/software/parallel/) _(optional but recommended)_ to analyze multiple BAM files and/or genomic regions

### Usage
`INDELseek.pl` analyzes SAM alignments in plain text to call complex indels in VCF format. Users are recommeneded to focus on a single BAM file and a small region at a time (e.g. a single PCR amplicon or single exon of protein-coding gene). Please refer to the following example to scale up the analysis to multiple BAM files and genomic regions in parallel.
>samtools view _bam_ _region_ | ./indelseek.pl [_options_]

- *bam*: indexed BAM alignment file
- *region*: 1-based position coordinates, e.g. amplicon chr4:55589744-55589911. For details please refer to [documentation of samtools view](http://www.htslib.org/doc/samtools.html).
- *standard output (STDOUT)*: variant calls in [VCF version 4.1](http://samtools.github.io/hts-specs/VCFv4.1.pdf)

*Options*
- **--refseq** _FILE_: indexed reference genome in FASTA format [ucsc.hg19.fasta]
- **--samtools** _FILE_: path to samtools executable [samtools]
- **--max_distance** _INT_: maximum distance of two variants to be considered together as a complex indel [5]
- **--quality_threshold** _INT_: complex indels with mean read base quality score below _INT_ are marked _LowQual_ [20]
- **--skip_lowqual**: skip _LowQual_ complex indel calls in output
- **--min_depth** _INT_: complex indels with depth (forward and reverse reads with the complex indel) below _INT_ are marked _LowDepth_ [50]
- **--skip_lowdepth**: skip _LowDepth_ complex indel calls in output
- **--min_af** _FLOAT_: complex indels with allele frequendcy below _FLOAT_ are marked _LowAF_ [0]
- **--skip_lowaf**: skip _LowAF_ complex indel calls in output
- **--depth_bam** _FILE_: indexed BAM alignment file is needed for allele frequency calculation
- **--phredoffset** _INT_: read base quality score encoding offset. Example: 33 for Illumina 1.8+ or Sanger, 64 for Illumina 1.3+ / 1.5+ or Solexa. (https://en.wikipedia.org/wiki/FASTQ_format#Encoding) [33]

Examples
```bash
# Any complex indel in KIT exon 8 with minimum allele frequency 2%?
# (Actual output of sample 10 described in manuscript)
samtools view sample.bam chr4:55589744-55589911 | ./indelseek.pl --skip_lowqual --skip_lowdepth --skip_lowaf --min_af 0.02 --depth_bam sample.bam | tee sample.complexindel.vcf
##fileformat=VCFv4.1
##source=INDELseek
##reference=file://ucsc.hg19.fasta
##INFO=<ID=DP2,Number=2,Type=Integer,Description="# alt-foward and alt-reverse reads">
##INFO=<ID=QS2,Number=2,Type=Float,Description="Mean quality scores of alt-foward and alt-reverse bases">
##INFO=<ID=AF,Number=1,Type=Float,Description="Allele Frequency">
##INFO=<ID=RDP,Number=1,Type=Float,Description="Mean read depth of REF positions">
##FILTER=<ID=LowAF,Description="AF below 0.02">
##FILTER=<ID=LowDepth,Description="ALT depth below 50">
##FILTER=<ID=LowQual,Description="Mean quality scores below 20 or ALT contains N">
#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO
chr4	55589768	.	CTTACGACA	AACCTC	35.63	PASS	DP2=857,752;QS2=37.44,33.56;AF=0.119;RDP=13525.3
chr4	55589767	.	ACTTACGACA	GGATGGAACT	36.33	PASS	DP2=251,203;QS2=37.37,35.04;AF=0.033;RDP=13651.1
chr4	55589766	.	GACTTACGA	TTTCCG	35.76	PASS	DP2=208,184;QS2=37.46,33.83;AF=0.029;RDP=13724.2
chr4	55589769	.	TTACGACA	CTCCT	35.68	PASS	DP2=151,125;QS2=37.61,33.35;AF=0.021;RDP=13376.0

# Speed up complex indel detection in list of BAM files and genomic regions by GNU Parallel
parallel --tag "samtools view {1} {2} | indelseek.pl --skip_lowqual --skip_lowdepth --skip_lowaf --min_af 0.02 --depth_bam {1}" ::: *.bam :::: region.list | grep -v "#" > complexindel.tab
```

Citations
--------
### INDELseek algorithm
* Au CH, Leung AYH, Kwong A, Chan TL and Ma ESK, 2017. [INDELseek: detection of complex insertions and deletions from next-generation sequencing data](http://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-016-3449-9). _BMC Genomics_ 18:16 (doi:10.1186/s12864-016-3449-9)

### Example studies that used INDELseek
* Cher CY, _et al._, 2016. [Next-generation sequencing with a myeloid gene panel in core-binding factor AML showed _KIT_ activation loop and _TET2_ mutations predictive of outcome](http://www.nature.com/bcj/journal/v6/n7/full/bcj201651a.html). _Blood Cancer Journal_ 6:e442 (doi:10.1038/bcj.2016.51)
* Lam SSY, _et al._, 2016. [Homoharringtonine (omacetaxine mepesuccinate) as an adjunct for _FLT3_-ITD acute myeloid leukemia](http://stm.sciencemag.org/content/8/359/359ra129). _Science Translational Medicine_ 8:359ra129 (doi:10.1126/scitranslmed.aaf3735)
