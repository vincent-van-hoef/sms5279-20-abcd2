===================
Project Description
===================

Support request
===============

Children ages 8-12 years spend an average of 4-6 hours a day watching or using screens. However, especially in children and adolescents, consistent evidence point to the negative effects of increased screen time on several aspects such as the development of physical abilities, socio-emotional development, sleep, obesity, depression, anxiety, stress regulation, and mental imagery. 

Based on empirical findings, Lillard et al.describe different theories that account for long-term media influences on executive functions. One points to the fact that media time is time away from other activities that may foster executive functions. Another highlights the interferences of attentional processes due to rapid scene changes and high levels of sensory stimulation in media.

However, previous evidence most frequently based on correlational approaches or retrospective comparisons, which are biased by numerous confounding variables and errors in estimating screen time. Therefore, causal relations between screen time and executive functions are limited.

To solve this issue, here in this project we will combine the approaches of Mendelian Randomization and Polygenic Scores based on the genotype data and screen time use of a sample of children. The sample comes from the `ABCD study <http://abcdstudy.org>`_ and consists of N=11,875 individuals aged 9/10 years old at baseline. A wide range of measurements was collected for each individual. In addition to demographic and cognitive variables, we also obtained the whole-genome genotyping data.

Genotyping was performed using the Smokescreen array35, consisting of 646,247 genetic variants. Before variant imputation, quality controls (QC) on the genotyping were performed to ensure each genetic variant has been successfully called in more than 95 percent of the sample and that missingness for each individual was not higher than 20%. After this QC 517,724 SNPs and 10,659 individuals remained.

Now, we need support to perform the SNP imputation of this QCed genotype data using the appropriate reference panel. This will allow us to later calculate polygenic scores in our sample based on the effect sizes of a large GWAS and use these for Mendelian Randomization analyses.

Our main request for NGI is to perform the imputations of the genotype data for us and then perform (if needed) a post imputation QC (to guarantee parameters such as minor allele frequency above 5% and Hardy-Weinberg threshold of 10-6). Additionally, if there is anyone with the expertise, we could also use technical help later with the Mendelian Randomization analyses.

Requested analyses
==================

The following analyses were agreed upon:

* Preprocessing of data supplied by user

    * Additional QC for HWE, Heterozygosity rate, MAF
    * PCA to check for outliers
    * Splitting into separate chromosomes
    * Pre-phasing using Shapeit

* Imputation (XY not included)

    * Imputation using IMPUTE2 (or similar)
    * Merging of data
    * Post-imputation QC with info score
    * Converting to selected output format

* Results

    * Report including methods and parameters used, code and files