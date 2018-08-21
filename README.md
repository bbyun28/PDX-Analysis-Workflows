#PDX Pipelines

##Overview
These pipelines were developed using the Civet framework and, as such, are designed to be command line analysis pipelines which execute through a batch system commonly used by High Performance Computing (HPC) systems. For more information on Civet, visit https://github.com/TheJacksonLaboratory/civet.

The pipeline is defined by an XML file which describes the file inputs (hardcoded reference files, files input by the user, or files generated by another step in the pipeline) and the steps that utilize these files.  Each tool described in the pipeline that can be invoked by the pipeline is encoded in a separate XML file; these tools can be shared amongst pipelines.  Additionally, the pipelines use python (v. 2.7.3).

###Disclaimer
Note that the pipeline and tool files provided here are merely examples of how to construct pipelines and create separate tool xml files; additional information is necessary to explicitly run the pipelines.  Python, R, and Perl scripts are mostly functional with some scripts require bit tweaking to fit individual user needs.  Although the CTP and RNA pipelines provided here are for paired end samples, the pipelines can be modified for single end samples.

###License
This software is licensed for non-commercial purposes only. See the [LICENSE.md](https://github.com/TheJacksonLaboratory/PDX-Analysis-Workflows/blob/master/LICENSE.md) file in this repository for the terms and conditions of this license.

##CTP
This pipeline is designed for processing tumor samples.  The pipeline takes as input a single sample for which paired-end sequencing has been performed, and outputs annotated variants and insertions and deletions (indels).

The pipeline encompasses several different steps.  The first step is a quality control step in which low-quality bases are trimmed, and low-quality reads are discarded.  The trimmed fastq files are then processed, and reads are classified as human or mouse; human reads are retained, and other reads are discarded.  The trimmed reads are then aligned against the human genome (hg38) using the BWA-MEM algorithm (bwakit v. 0.7.15) and a SAM file generated using samtools (v. 0.1.18).  There are then many processing steps that involve the removal of duplicates using picard (v. 2.8.1) followed by re-alignment around indels and base quality recalibration (GATK v. 3.4.0 and java v. 1.7.0); a BAM file is generated.  Target coverage is then calculated using picard hybrid selection metrics.  Initial variants are then called and filtered using GATK unifiedGenotyper and variantFiltration.  SnpEff (v. 4.2) and SnpSift, in addition to a python script, are employed to filter variants with coverage less than 140 and call variant effects; variants are then further filtered using allele frequency cut-offs.  Micro indels are then called and filtered using pindel (v. 0.2.5a3).  SnpEff and SnpSift are utilized to annotate the filtered variants and micro indels.  Alignment, quality control, duplication metrics, and coverage statistics are then compiled into a single file.  Scripts, in addition to GATK, are employed to calculate coverage.  Finally, intermediate files are removed from the run directory.

The provided config file can be used to specify specific files or set various options; for example, specifying sample ploidy.
 
##RNA
This pipeline is designed for processing RNA-seq samples.  The pipeline takes as input a single sample for which paired-end sequencing has been performed, and outputs read counts.

The first step takes as input raw fastq files and outputs fastq files which contain trimmed reads.  The trimmed fastq files are then processed, and reads are classified as human or mouse; human reads are retained, and other reads are discarded.  Reads are then aligned against the human genome (hg38) using the bowtie2 option for rsem (v. 1.2.19) and a BAM file is output.  Gene names are then added and read counts normalized.  Read group information is then added to the BAM file followed by picard (v. 2.8.1) reordering and sorting of the BAM file.  Alignment metrics are then generated using picard followed by the compilation of all metrics (e.g., statistics from filtering, rsem, and picard).  Finally, a classifier is run, and exon level coverage statistics are determined from the alignment file; this requires samtools (v. 0.1.18) and bedtools (v. 2.25.0).


##CNV
This pipeline is designed for processing Affymetrix SNP 6.0 array samples.  The pipeline takes as input a single tumor sample (CEL file) and outputs copy numbers for genes and segments.

The pipeline encompasses four separate steps: 1) allele-specific signal from CEL files and normalization with HapMap CEL files; 2) output of Log R ratios (LRR) and B-Allele frequencies (BAF); 3) output of aberrant tumor cell fraction, ploidy, and segments containing gains or losses; 4) annotation of ASCAT segments with loss of heterozygosity (LOH), chromosome arm fraction, and ploidy and annotation of genes with copy number.  LRR and BAF are calculated using PennCNV-Affy and Affymetrix Power Tools (v. 1.15.0).  The GC correction, heterozygous SNP estimation, and the computation of aberrant cell fraction, ploidy, and allele-specific copy numbers are performed following the implementation of the ASCAT R package (v. 2.4.3) in the included R script.  Annotation is performed using the included Perl scripts.
