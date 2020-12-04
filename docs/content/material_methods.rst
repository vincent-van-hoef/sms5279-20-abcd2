.. _material_methods:

====================
Material and Methods
====================

This is a overview of the different steps taken to clean up the data and prepare the files for imputation.

Results can be found in the :ref:`results` section.

Quality Control
===============

Quality control is an important step prior to imputation. Here, we used the *Preimputation* module as implemented in the imputation pipeline `RICOPILI	<https://sites.google.com/a/broadinstitute.org/ricopili/preimputation-qc>`_. This module is basically a wrapper function around PLINK to filter out SNPs and samples of lower quality. The QC thresholds used are listed below and were executed in the following order:

#. SNPs: call rate < 0.95
#. IDs: call rate < 0.98
#. IDs: FHET outside +- 0.2
#. SNPs: call rate < 0.98
#. SNPs: with Minor Allele Frequency < 0.01
#. SNPs: Hardy-Weinberg equilibrium (HWE) < 1.0e-300  
#. SNPs: sHWE < 1.0e-6

.. code-block:: bash
    
    # RICOPILI has been installed on Bianca
    # Code to run the pre imputation QC; more info in link above
    # Make a QC folder and link or move the initial {bim,fam,bed} files into this folder before running following code
    preimp_dir --dis cog --pop mix --out abcd2 --maf 0.01 --hwe_th_ca 1.0e-300 --hwe_th_ca 1.0e-300

Special care was taken when it comes to detect SNP outside of HWE. HWE says that for a population meeting certain conditions, the genotype frequencies of a genetic locus can be expressed in terms of the allele frequencies. In studies such as genome-wide association studies (GWAS), HWE is treated as a baseline for quality control, where markers deviating strongly from HWE are filtered out as likely genotyping errors. Because of a strong population structure in the ABCD2 data, many SNP were found to break HWE when using standard cutoff thresholds. Therefore, a different approach was used to filter out SNPs breaking HWE (the 1.0e-300 used in the first QC run is basically equal to setting no filter). We opted for an approach called `sHWE <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6827367/>`_ or "structural HWE" as implemented in the R package `lfa	<https://www.bioconductor.org/packages/release/bioc/html/lfa.html>`_. SNP were tested using a model with 7 latent variables on a 4000 sample subsample of the QCed data. After this test, SNP not passing the p-value < 1.0e-6 threshold were removed as well.

.. code-block:: r

    # Demanding computation; run via SLURM or interactive session with 12 cores at least
    # Run in R to calculate sHWE p-values
    library("lfa")
    
    # read the binary files (basename is name without {bim,fam,bed} extension)
    abcd <- read.bed(basename)
    # Select random subset
    abcd_sample <- abcd[,sample(c(1:ncol(abcd)), 4000)]
    # Run sHWE
    fa 	<- lfa(abcd_sample, 7)
    abcd_shwe <- sHWE(abcd_sample, fa, 3)
    # Write output
    write.table(abcd_shwe, file = "Iteration_single7_4000.txt", quote=FALSE, sep="\t", col.names=FALSE)

.. note::
    The sHWE method is computationally very demanding, so the test was run on a reduced dataset of 4000 random chosen samples. 

Population Structure
====================

To check for outliers and population structure the `PCA module	<https://sites.google.com/a/broadinstitute.org/ricopili/pca>`_ as implemented in RICOPILI was used. Prior to calculating the principal components, some additional filtering was performed to reduce the influence of highly variable regions, related samples and SNP in strong linkage disequilibrium.

- IBD > 0.9 (remove duplicates/highly related samples)
- LD pruning: R2 < 0.2
- MAF < 0.05
- HWE < 1.0e-03
- SNP missing rate < 0.02
- no AT/GC SNPs
- no MHC region (Chr6:25-35Mb)
- no Chr8 inversion (Chr8:7-13Mb)

This resulted in a cleaned up dataset that could be used as input for the PCA analysis. Custom plotting was done using the R package ggplot2.

.. code-block:: bash

    # Run in folder with the QCed PLINK files; this performs the filtering and PCA
    pcaer --out cog_scr_time cog_abcd2_mix_vh-qc1.bim --rel 0.9

Imputation
==========

The imputation of the filtered and cleaned data was done using a different pipeline: `Odyssey <https://github.com/Orion1618/Odyssey>`_. This pipeline uses SHAPEIT2 to phase the input data and then imputes using IMPUTE4. This combination of tools is computationally efficient but not available in RICOPILI, hence the switch to this pipeline. It has excellent documentation so I refer you to the :download:`tutorial <../_static/Odyssey_Tutorial.docx>` for a detailed walkthrough of the pipeline. Everything is available on Bianca for it to run again, if necessary. As a reference population, the 1,000 Genomes haplotypes -- Phase 3 integrated variant set release in NCBI build 37 (hg19) coordinates were used. This is a mixed population dataset consisting of 2504 samples and 5008 haplotypes. More info and the reference itself can be found on this `website <https://mathgen.stats.ox.ac.uk/impute/1000GP_Phase3.html>`_.

Phasing was carried out per chromosome, using 35 MCMC iterations and a 100 states per window (windows of 2Mb). Imputation was carried out in chunks of 5Mb with a 250kb buffer region and an effective population size of 20000. Imputed variants in each non-overlapping part of each chunk were concatenated into per-chromosome files.

.. note::
    One important question is how to choose a reference panel that will produce high imputation accuracy in a population of interest. The answer is seldom obvious because human populations have experienced complex demographic histories with many migration and mixture events. Consequently, it can be hard to decide which reference haplotypes should be used in a particular study. The makers of IMPUTE2/4 have proposed a simple and universal solution to this problem: provide all available reference haplotypes to IMPUTE2/4, then let the software choose a "custom" reference panel for each individual to be imputed. There are several advantages to this approach:

    - Investigators do not need to waste time deciding which haplotypes to include in the reference panel. 
    - This strategy works in a variety of human populations. Researchers have used this approach to successfully impute populations ranging from homogeneous isolates to recent and complex admixtures.
    - IMPUTE2/4 is often more accurate with an ancestrally inclusive reference panel than with a smaller panel chosen by intuition. This is because individuals from "diverged" populations may still share genomic segments of recent common ancestry, and IMPUTE2/4 can use this haplotype sharing to improve accuracy. At the same time, the software can ignore haplotypes that are not helpful. 

    The benefits of using inclusive reference panels are greatest at low-frequency variants (MAF < 5%), since these variants may be poorly represented in a reference panel from the population of interest (due to sampling effects) but well-represented in panel from a different population (e.g., due to genetic drift).

IMPUTE4 is considerably faster than IMPUTE2 but leaves out some additional quality checks. One of those is the concordance rate. This metric is calculated by masking one input variant at a time and imputing it by using the surrounding variants. The imputed genotypes can then be compared to the actual genotype seen on the array to give the concordance. This is done for each input variant resulting in an overall concordance rate. Good quality imputation should result in concordance rates around 95%. Imputation is done in chunks of 5Mb at a time (so 588 chunks in total). To reduce computation time a random selection of 25 chunks was used to calculate concordance rates using IMPUTE2. Custom plotting was done in ggplot2. 

QCTOOL was used to calculate the minor allele frequency (MAF) and imputation information score of each imputed variant. The imputation information is a metric between 0 and 1. A value of 1 indicates that there is no uncertainty in the imputed genotypes whereas a value of 0 means that there is complete uncertainty about the genotypes. A value of α in a sample of N individuals indicates that the amount of data at the imputed SNP is approximately equivalent to a set of perfectly observed genotype data in a sample size of α N. Many GWAS carried out to date have used filters on MAF and information score by applying a threshold on these metrics. There is no single correct threshold to use. However, as MAF decreases it is generally the case that imputation quality decreases. Previous studies have tended to use a filter on information between 0.3-0.5. An information measure of 0.3 was applied here. In ~10,000 samples this roughly corresponds to an effective sample size of ~3,000, which would be expected to yield good power to detect association. Some variants are imputed as monomorphic, or close to monomorphic i.e. no or almost no variation in the genotypes. Such sites were removed using QCTOOL using a filter on MAF of 0.0001. 

Manuscript
==========

Here an example of a Material and Methods section for a potential manuscript:

    Before imputation, SNPs were excluded if they had high levels of missing data (SNP call rate < 98%), departed from Hardy-Weinberg equilibrium as calculated in the lfa R package (sHWE) (P<1 x 10−6), or had minor allele frequencies (MAF) <1%. Moreover, individuals with an absolute autosomal heterozygosity > 0.2 or more than 2% missing genotyoes were excluded. These filtering steps resulted in a cleaned dataset of 10,069 individuals and 430,622 variants. Subsequently, haplotypes were pre-phased with SHAPEIT2. Genetic markers were imputed using the IMPUTE4 software and the 1000 Genomes References Panel (phase 3, build 37). After imputation, genotypes with an INFO score  < 0.3 or a MAF < 0.001% were excluded. The final number of SNPs after imputation was 40,637,119 in a total of 10,069 individuals. 

