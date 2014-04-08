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
When investigating the relationship between a genomic locus and an mRNA transcript's abundance, having a robust estimate is important.  When investigating these relationships genome-wide, robustness is crucial.  BeQTL robustly identifies loci correlating with mRNA transcript levels using linear regression and a statistical bootstrap. 

Method
-----------------
<h3>Materials<h3>

Input genotype and expression data consisted of TCGA data obtained from the GDAC portal maintained by the Broad Institute.  For genotype data, BeQTL used the PANCAN 12 genomewide SNP 6.0 level 2 birdseed genotype calls from.  For expression  the PANCAN 12 RNAseqv2 Illumina HiSeq level 3 RSEM data was used.   SNPs with a low sample minor allele frequency (MAF<15%) and transcripts not expressed in greater than 85% of cases for a particular cancer type were excluded from the analysis. BeQTL was performed on 9 cancer types (outlined in Table 1).
<h3>Math</h3>

in BeQTL, gene expression is modeled as function of allele count, thus $g$ expression level of a particular gene is estimated by linear regression:

$g=x \times s + c$ where $s$ is the number of copies of a particular allele at a particular locus and $c$ is a constant. In BeQTL this regression is performed hundreds of times for each transcript-SNP pair during the bootstrap phase and a distribution for the correlation coefficient is estimated.  During the bootstrap, cases are sampled with replacement $n\times \log (n)$ times where $n$ is the number of cases for that cancer type.




<h3>Software </h3>

BeQTL exists as a core C++ application with some supporting R scripts.  BeQTL utilizes the Intel Math Kernel Library (MKL) for the matrix multiplication and quantile estimation, the HDF5 library to store and structure the input data, and the R package sqldf to speed up the initial reading in of the data.  The process of running BeQTL is fairly straightforward:

1. Structure your genotype and expression data according to the [guidlines described here.](http://Beqtl.org/format.html "Format") and ensure you have annotation files for both your genotype and expression files
2. Run the CleanDatasets.R script.  This script ensures that your expression and genotype files have the same number of columns, and creates copies of the data that will be used to create the HDF5 file used by BeQTL.  

<h2>Robustness</h2>
To assess how the effect of the bootstrap