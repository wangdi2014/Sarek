![](doc/images/CAW_logo.png)
# Cancer Analysis Workflow Results Delivery 
This README describes the delivery directory structure for files passed to users at [NGI][ngi-link]

There are four sections dedicated for different results: Annotation, Preprocessing, Reports and 
VariantCalling. All the four sections can have sub-directories containing results from different software.

## Annotation: 

This directory contains results from the final annotation steps: two software are used for annotation, [VEP][vep-link] and [snpEff][snpeff-link]. 
Only a subset of the VCF files are annotated, and only variants that have a PASS filter. FreeBayes results are not annotated in the moment yet as
we are lacking a decent somatic filter. For HaplotypeCaller the germline variations are annotated for both the tumour and the normal sample.

All the VCFs annotated have an `ann.vcf` extension, and a summary HTML file associated. 

### SnpEff

[SnpEff][snpeff-link] can add annotations for many sort of variants not only SNPs, and is using multiple databases for annotations. SnpEff prints out 
not only the annotated VCF files, but a summary HTML and CSV, also a list of affected genes with the actual changes and impact is included in a text file. 
The generated VCF header contains the software version and the used command line. 

Annotations added are in [cancer mode][snpeff-cancer-mode] are very rich, CAW is using the software in a single-sample mode. VCF files containing germline
calls are annotated in [regular mode][snpeff-regular-mode] of SnpEff.

### VEP

The [Variant Effect Predictor][vep-link] is based on Ensembl, and can determine the effects of all sorts of variants, including SNPs, indels, structural variants, 
CNVs. Some of the Manta VCF files are not always succeed in going through the VEP filtering though: there can be missing annotations for these variant calls. 

The HTML summary files show general statistics and quality-related measures. In the header of the annotated VCF files one can find the VEP/Ensembl version used 
for annotation, also the version numbers for additional databases like Clinvar or dbSNP used in the "VEP" line. The format of the [consequence annotations][VEP-predictions] is also 
in the VCF header describing the INFO field. In the moment it contains 

* Consequence: impact of the variation, if there is any
* Codons: the codon change, i.e. cGt/cAt
* Amino\_acids: change in amino acids, i.e. R/H if there is any
* Gene: ENSEMBL gene name 
* SYMBOL: gene symbol
* Feature: actual transcript name
* EXON: affected exon
* PolyPhen: prediction based on [PolyPhen][polyphen-link]
* SIFT: prediction by [SIFT][sift-link]
* Protein\_position: Relative position of amino acid in protein
* BIOTYPE: Biotype of transcript or regulatory feature

---
## Preprocessing:

The preprocessing is following the [GATK Best Practices][GATK-BP] to obtain aligned BAM files used for whole-genome germline analysis. 

### NonRealigned:

This directory is usually empty, as it is a placeholder for the original mapped, merged and duplicate marked BAM files. After these steps the BAM files are 
processed further, reads are realigned around known indels, and recalibrated.

### NonRecalibrated:

This is the place for the BAM file delivered to users: besides the realigned files the recalibration tables are also stored (`*.recal.table`), these can be
used to create base recalibrated files. The `.tsv` file is autogenerated also, these can be used by CAW for further processing and/or variant calling. 

The BAM file headers contain the details about the actual command-line arguments for mapping, merging, use `samtools view -H <filename>` to view the used 
reference, read groups etc.

### Recalibrated:

This directory is usually empty, it is the location for the final recalibrated files in the preprocessing pipeline: recalibrated BAMs are usually 2-3 times 
larger than the realigned files, and are needed only by MuTect1 and MuTect2 (considering GATK 3.8). To re-generate recalibrated BAMs you have to apply the 
recalibration table delivered to the `NonRecalibrated` directory either by calling CAW, or doing this [recalibration step][BQSR-link] yourself.

---
## Reports:

The `Reports` directory is the place for collecting outputs for different quality control (QC) software; going through these files can help us to decide 
whether the sequencing and the workflow was successful, or further steps are needed to get meaningful results. The main entry point it the [MultiQC][multiqc-link] 
directory: the HTML index file aggregates and visualizes all the software use for QC.
 
### MultiQC  
To assess the quality of the sequencing and workflow the best start is to view at the Reports\/MultiQC\/multiqc\_report.html file of the MultiQC directory, where the 
statistics and graphics of all the software below should be presented. The actual graphs and the tables are configurable, and generally much easier to view than the 
raw output of the individual software. The subsequent QC compartments are:

* bamQC: [Qualimap][qualimap-link] examines sequencing alignment data in SAM/BAM files according to the features of the mapped reads and provides an overall view 
	of the data provides quality control statistics about aligned BAM files
* BCFToolsStats: [bcftools][bcftools] measuring non-reference allele frequency, depth distribution, stats by quality and per-sample counts, singleton stats, etc. of VCF files.
* [FastQC][fastqc]: provides statistics about the raw FASTQ files only. 
* MarkDuplicates: a [Picard][picard-md] tool to tag PCR/optical duplicates from aligned BAM data
* SamToolsStats: [samtools][samtools] collection of statistics from BAM files
---

## VariantCallings:

All the raw results regarding variant-calling are collected in this directory. Not all the software below are producing VCF files, also both somatic and germline 
variants are collected in this directory. 

* [Ascat][ascat]: is a method to derive copy number profiles of tumour cells, accounting for normal cell admixture and tumour aneuploidy. This direcory contains the 
graphical output of the software, CNV, ploidy and sample purity estimations.
* [FreeBayes][freebayes]: is for Bayesian haplotype-based genetic polymorphism discovery and genotyping. The single VCF file generated by FreeBayes
is huge, it is recommended to flatten and filter this VCF, i.e. using the provided [SpeedSeq][speedseq] filter
* [HaplotypeCaller][haplotypecaller] is the in-house germline caller of the Broad Institute, the non-recalibrated variant files are there to check the
germline variations and compare the two samples (tumour and normal) for possible mixup
* HaplotypeCallerGVCF: germline calls in [gVCF format][genomicvcf] even for the tumour sample: this format makes possible the joint analysis of a cohort
* [Manta][manta]: is a structural variant caller supported by Illumina. There are several output files, corresponding to germline (diploid) calls, candidate calls 
and somatic files. Manta provides a candidate list for small indels also that can be fed to Strelka, but this feature is not incorporated yet.
* [MuTect1][mutect1] is a now-defunct GATK-based somatic SNP-only caller - going to be left out for analysis in the future. It is sensitive, recommended to keep only 
lines with "PASS" filter. 
* [MuTect2][mutect2] is the current somatic caller of GATK for both SNPs and indels. Recommended to keep only lines with the "PASS" filter.
* [Strelka][strelka] is somatic SNP and indel caller supported by Illumina. Strelka gives filtered and unfiltered calls for SNPs and indels separately, together with germline calls.

[ascat]:https://www.crick.ac.uk/research/a-z-researchers/researchers-v-y/peter-van-loo/software/
[bcftools]: http://www.htslib.org/doc/bcftools.html
[BQSR-link]: https://gatkforums.broadinstitute.org/gatk/discussion/44/base-quality-score-recalibration-bqsr
[fastqc]: https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
[freebayes]: https://github.com/ekg/freebayes
[GATK-BP]: https://software.broadinstitute.org/gatk/best-practices/bp_3step.php?case=GermShortWGS
[haplotypecaller]: https://software.broadinstitute.org/gatk/documentation/tooldocs/current/org_broadinstitute_gatk_tools_walkers_haplotypecaller_HaplotypeCaller.php
[genomicvcf]: https://gatkforums.broadinstitute.org/gatk/discussion/4017/what-is-a-gvcf-and-how-is-it-different-from-a-regular-vcf
[manta]: https://github.com/Illumina/manta/blob/master/docs/userGuide/README.md#structural-variant-predictions
[multiqc-link]: http://multiqc.info/
[mutect1]: https://software.broadinstitute.org/gatk/download/mutect
[mutect2]: https://software.broadinstitute.org/gatk/documentation/tooldocs/current/org_broadinstitute_gatk_tools_walkers_cancer_m2_MuTect2.php
[ngi-link]: https://ngisweden.scilifelab.se/
[picard-md]: http://broadinstitute.github.io/picard/command-line-overview.html#MarkDuplicates
[polyphen-link]: http://genetics.bwh.harvard.edu/pph2/
[qualimap-link]: http://qualimap.bioinfo.cipf.es
[samtools]: http://www.htslib.org/
[sift-link]: http://sift.bii.a-star.edu.sg/
[snpeff-link]: http://snpeff.sourceforge.net/
[snpeff-cancer-mode]: http://snpeff.sourceforge.net/SnpEff_manual.html#cancer
[snpeff-regular-mode]: http://snpeff.sourceforge.net/SnpEff_manual.html#input
[speedseq]: https://github.com/SciLifeLab/CAW/blob/master/scripts/speedseq.filter.awk
[strelka]: https://github.com/Illumina/strelka
[vep-link]: http://www.ensembl.org/Tools/VEP
[VEP-predictions]: https://www.ensembl.org/info/genome/variation/predicted_data.html
[logo]: ttps://img.shields.io/github/release/SciLifeLab/CAW.svg
