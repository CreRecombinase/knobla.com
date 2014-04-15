---
layout: post
title:  "Introduction to BeQTL"
date:   2014-04-08 15:31:53
categories: First-post BeQTL 
---


BeQTL Method
========================================================

Introduction
------------------

Expression quantitative trait loci (eQTL) mapping is widely performed to study the genetics of gene regulation.  While the genome wide association study (GWAS) can yield the alleles correlated with a disease or phenotype of interest, on it’s own it rarely gives insight as to the functional significance of these alleles. Finding the transcripts whose expression levels correlate with an allele known to predict disease risk can be an important step in elucidating the functional consequences of the variant.  Reciprocally, countless molecular experiments have been undertaken to understand the molecular mechanisms of disease, and have described genes and gene networks known to influence disease phenotype.  Finding genetic variants that correlate with the expression levels of these genes can bridge the divide between genotype and phenotype.
 
	The development of massively parallel methods for DNA and RNA sequencing have enabled the generation of extremely large data-sets containing samples that have undergone both genotyping and expression profiling genomewide. 
As the cost of both genotyping and gene expression profiling continue to decline, the size and number of these datasets is likely to increase in the coming years. A major bottleneck to performing large-scale integrative analyses on these is the computational resources necessary to comprehensively analyze all potential interactions in a data-set. 
 This problem becomes even more intractable, when using resampling-based statistical methods (including the bootstrap), which have demonstrated wide utility across a range of domains for identifying robust associations from  large data sets. Assessing these relationships in a robust manner using standard approaches is not feasible, due to the computational expense required. For example, a researcher can now generate full genotype and RNA expression data on a cohort of hundreds of patients in days-to-weeks. 
Performing comprehensive assessment of all locus-transcript interactions using the bootstrap, would take XXX years on commodity hardware (e.g. 8 GB RAM).  Thus, there is a major need for the development of new computational methods to enable the identification of robust statistical relationships in an efficient manner.
 One recently developed method, Matrix eQTL, utilizes matrix multiplication to efficiently estimate the association between all SNP-transcript pairs in a dataset, but only provides a point estimate of associations and does not enable large-scale parrallelization of analyses across multiple CPUs in an HPC environment. 
Point estimates may be sensitive to the influence of outliers and rare events (as are frequently encountered in both genetic and gene expression data), and statistical-resampling based methods enable estimates of an entire distribution of values that more fully captures relationships between variables, and are less susceptible to spurious associations due to rare events and outliers. Here we present Bootstrap eQTL (BeQTL). 
BeQTL uses matrix multiplication to comprehensively estimate all SNP-Transcript associations in a dataset, followed by statistical bootstrap to identify robust eQTLs. The BeQTL implementation enables parallelization across hundreds of CPUs in an HPC environment to generate robust statistics in a fraction of the time required using standard approaches.   
The Cancer Genome Atlas project is a large-scale effort to perform comprehensive molecular profiling across hundreds-to-thousands of patient samples spanning all major human cancer types. While most data in TCGA is focused on characterization of somatic alterations in cancer, the project performed genomewide array-based genotyping assays on blood samples from study participants. In addition, TCGA performed expression profiling by RNAseq on study participants  on cancer samples. Thus, this presents an ideal data-set for assessing the performance of BeQTL. Further, the identification of robust eQTL in cancer will be useful to our understanding of the molecular mechanisms of carcinogenesis.

Method
-----------------
<h3>Materials</h3>
<h4> Cancer Type </h4>
BeQTL was run on 9 different cancer types in TCGA.  Invasive breast cancer was run as two cancer types, ER positive and ER negative.  The cases were assigned to ER positive or ER negative based on the available clinical annotation.  Cases were included only if both blood genome-wide SNP and tumor RNA-Seq expression data were available.  Input genotype and expression data consisted of TCGA data obtained from the GDAC portal maintained by the Broad Institute.  
<h4> SNP data </h4>
Germline genotype data consisted of level 2 genome wide SNP data from blood.  SNPs were excluded from analysis on a cancer type if the observed minor allele frequency (MAF) was less than fifteen percent.  The resulting matrix $S$ had columns representing the cases in the cancer type and rows representing SNPs.  For $j \in \left{ {0,1,...,n} \right}$ where $n$ is the number of cases inthe cancer type and $i \in \left{ {0,1,...,s} \right}$ where $s$ is the number of SNPs tested in the cancer type, we have the element of matrix $S$: \$S\sub{i,j} \in \left\{ {0,1,2} \right\}$, representing the copies of the variant allele $i$ in case $j$.

<h4> Gene expression </h4>
Tumor gene expression data consisted of level 3 RNAseqv2 Illumina Hiseq from primary tumors.  Gene expression level was estimated using the RSEM method.  transcripts were excluded from analysis on a cancer type if there was no measured expression in more than fifteen percent of cases.  The resulting matrix $G$ had columns representing the cases in the cancer type and rows representing transcripts.  For $j \in \left{ {0,1,...,n} \right}$ where $n$ is the number of cases inthe cancer type and $i \in \left{ {0,1,...,g} \right}$ where $g$ is the number of transcripts tested in the cancer type, we have the element of matrix $G$: \$G\sub{i,j} \in \mathbb{R}$, representing the RSEM estimate of expression level for transcript $i$ in case $j$.
<h3>Method</h3>
<h4>Estimation of SNP-Transcript association</h4>
For assessing whether a particular SNP-transcript pair constitute a robust eQTL, BeQTL uses bootstrapped linear regression. In this regression transcript expression is modeled as function of allele count;  $g$, the expression level of a particular transcript as estimated by RSEM is estimated by linear regression:
$g=x \times s + c$ where $s$ is the number of copies of a particular allele at a particular locus as assessed by birdseed, and $c$ is a constant. In BeQTL this regression is performed on a resampled dataset $n\times \log(n)$ times, where $n$ is the number of cases in the cancer type, for each transcript-SNP pair, generating a distribution of correlation coefficients.  These correlation coefficients are then converted to t-statistics using the formula $t=\sqrt{n-2}\times\frac{r}{\sqrt{1-r^2}}$, where $r$ is the correlation coefficient.  The median and 95% confidence interval are computed for each distribution, and a p-value is computed from the median t-statistic.  

BeQTL is also able to discrimiate between local and distant eQTL.  If a SNP-transcript pair are located on the same chromosome, the minimum of the distance from the SNP to the transcription start and end site was recorded.

<h2>Results<h2>
Across the 10 cancer types analyzed, 60,182,277 eQTL were detected with a p-value threshold of 2.4e-4.  

<h3>Clustering<h3>





<h3>Software </h3>

BeQTL exists as a core C++ application with some supporting R scripts.  BeQTL utilizes the Intel Math Kernel Library (MKL) for the matrix multiplication and quantile estimation, the HDF5 library to store and structure the input data, and the R package sqldf to speed up the initial reading in of the data.  The process of running BeQTL is fairly straightforward:

1. Structure your genotype and expression data according to the [guidlines described here.](http://Beqtl.org/format.html "Format") and ensure you have annotation files for both your genotype and expression files
2. Run the CleanDatasets.R script.  This script ensures that your expression and genotype files have the same number of columns, and creates copies of the data that will be used to create the HDF5 file used by BeQTL.  

<h2>Robustness</h2>
To assess how the effect of the bootstrap