.. _results:

=======
Results
=======

More info on the methods is found in the :ref:`material_methods` section.

Quality Control
===============

The initial dataset consists of 10627 samples and 517724 variants. Prior to imputation, additional filtering was performed. At :numref:`FilterTable`, you can see an overview of the filtering steps and how many variants and samples were removed.

.. list-table:: Overview of Filtering steps and removed SNP and Samples. 
    :name: FilterTable
    :widths: 75 25
    :header-rows: 1
    :align: center

    * - Filter
      - N 
    * - SNP call rate < 0.95
      - 140
    * - ID call rate < 0.98
      - 523
    * - ID FHET outside +- 0.2
      - 36
    * - SNP call rate < 0.98
      - 21981
    * - SNP MAF < 0.01
      - 47945
    * - SNP HWE < 1.0e-300
      - 111
    * - SNP sHWE < 1.0e-6
      - 11361

After removing unidentified, duplicated SNP as well, a dataset consisting of 10069 samples and 430622 variants remains as input for the imputation step. 

.. note:: 
    There is a small discrepancy of around 5000 SNP between the number of filtered SNP and the remaining SNP for imputation. This is a consequence of the --exclude function in PLINK which removes listed variants AND all of their duplicates. There are quite some duplicate IDs in the original dataset called "Physical.Position_X_X" which are counted once for filtering while all of the duplicates are removed during the subsetting resulting in a slightly lower number of remaining SNP.

Population structure
====================

To check for population structure a `PCA <https://builtin.com/data-science/step-step-explanation-principal-component-analysis>`_ was performed (see :numref:`pcaplot`). This shows that - as expected - samples group more or less according to their ancestry. This will influence the HWE p-values strongly and highlights the need for the alternative HWE calculation used in the project.

.. figure:: /_static/PCA_plot.png
    :name: pcaplot
    
    PCA plot of the QCed data coloured according to self-declared ethnic group.


Imputation
==========

Imputation resulted in 81.699.844 SNP in total prior to any quality control. Overall concordance rate were calculated for a random sample of 25 "chunks" using IMPUTE2. This is the result of an internal cross-validation that the program performs automatically. For this analysis, IMPUTE2 masks the genotypes of one variant at a time in the study data, then imputes the masked genotypes with information from the reference data and nearby study variants. The imputed genotypes are then compared with the original genotypes to evaluate the quality of the imputation. This number should typically be around 95%; it may be lower in certain populations or regions of the genome, but if it is much lower then you may need to double-check the analysis. Overall concordances for the 25 random chunks are shown in :numref:`concordance`. For most chunks, the overall concordance rate is around 95% or higher.

.. figure:: /_static/concordance.png
    :name: concordance

    Overall concordance of 25 random chunks. Indicated is the 95% marker, indicating high quality imputation.

The INFO score is an important metric indicating the uncertainty in the imputed genotype. SNP not reaching a 0.3 threshold value were removed. A representative distribution of the INFO scores is shown in :numref:`infoplot`. Similar plots for the other chromosomes can be downloaded :download:`here	<../_static/infoplots.zip>`.

.. figure:: /_static/chr1_info_hist.png
    :name: infoplot

    Histogram of the INFO scores of Chromosome 1. Threshold value indicated in red.

After additional filtering of SNP with MAF < 0.001% and identical duplicates, a total of **40.637.119 variants in 10.069 individuals remained**.

The data is stored on Bianca in two folders: one containing the data split up per chromosome ane contains the whole dataset in a single file.


::    

    proj/
    |-- sens2020568
        |-- nobackup
            |-- results
                |-- combined_data
                |   |-- ABCD2_imputed_03_noDup.{bim,fam,bed}
                |-- per_chromosome
                    |-- Chr1_imputed_03_0001.{bim,fam,bed}
                    |-- Chr2_imputed_03_0001.{bim,fam,bed}
                    |-- ... 
                    |-- Chr22_imputed_03_0001.{bim,fam,bed}
