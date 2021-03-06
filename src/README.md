# Instructions for enloc

```enloc``` implements the analysis pipeline for enrichment estimation aided colocalization analysis.

## Prerequisite and installation

### Installation

The single execution point to carry out the full integrative analysis is the perl script ```enloc```, which assumes a few scripts and binary executables pre-installed in the running machine.

* ```torus```: binary executable to carry out the EM algorithm for enrichment estimation. ([repo](https://github.com/xqwen/dap/tree/master/torus_src))
* ```dap1```: binary executable to perform fine-mapping analysis using DAP-1 algorithm ([repo](https://github.com/xqwen/dap/tree/master/dap1_src))
*  ```openmp_wrapper```: binary executable to carry out multi-thread batch processing ([repo](https://github.com/xqwen/openmp_wrapper))
* ```mi_eqtl```: perl script to sample multiple imputation QTL data sets (available in this directory)
* ```get_pip```: perl script to collect QTL posterior inclusion probabilities (PIPs) from QTL analysis results
* ```compute_rcp```: perl script to compute regional colocalization probabilities (RCPs)

None of the above needs to be explicitly invoked by users, but they (or their symbolic links) should be placed in a *single* directory that enloc can access.

### Ideal running environment

This implementation of analysis pipeline is designed and tested in a multi-core single machine environment. The typical Unix/Linux utilities are also assumed to be available. 


## Data preparation for enloc analysis


### Molecular QTL Data

```enloc``` requires summary information from the molecular QTL analysis to impute binary annotations (i.e. the latent true association status of each SNP with respect to the molecular phenotype) in enrichment analysis. These summary information details all possible association models (i.e., combination of associated SNPs) and their assessed probabilities for each molecular trait.  This information can be obtained from Bayesian multi-SNP QTL analysis implemented in the software package ```dap```.

A sample set of eQTL annotation data from the cis-eQTL analysis of GTEx whole blood samples (version 6 release) can be downloaded [here](http://www-personal.umich.edu/~xwen/download/gtex_whole_blood.v6.tgz).
The data set contains fine-mapping result of cis-eQTLs from 22,749 genes analyzed by the greedy version of the DAP algorithm.
We also provide a wiki page on the complete flow of Bayesian molecular QTL analysis.

**IMPORTANT**: The name of each molecular QTL analysis result file should start with an identifier of the corresponding molecular trait. This information will be used by ```enloc```. E.g. for eQTL analysis, the result should be named as "gene_name.suffix". ***In particular, the identifier should only include alphanumerics (i.e., the set of letters of the alphabet and numeric characters)***.

### GWAS Data

The default option for GWAS data is to utilize only the single-SNP testing summary statistics, namely the z-scores for each candidate SNPs. All the input data for a given GWAS trait should be organized into a single gzipped text file with the following simple format

```
   chr1:998395   Loc1  -0.178471
   chr1:1000156  Loc1  -0.169669
   chr1:1001177  Loc1  -0.247359
   chr1:1002932  Loc1  -0.240580
   chr1:1003629  Loc1  -0.169000
   chr1:1004957  Loc1  -1.145393
   chr1:1004980  Loc1  -1.145393
   chr1:1006223  Loc1  -1.174756
```
The first and second columns represent the ID the LD block of the corresponding SNP, respectively. The last column indicates the z-scores. The LD blocks are defined based on the results of [Berisa and Pickrell, 2015](http://bioinformatics.oxfordjournals.org/content/32/2/283). The segmentation of the LD blocks are population-specific, the detailed information can be found [here](https://bitbucket.org/nygcresearch/ldetect-data).

For reference, we provide a complete sample data from the GWAS of high density cholesterol (HDL): [HDL.z_score.gz](http://www-personal.umich.edu/~xwen/download/gwas_hdl/HDL.z_score.gz). The data set is orignially from [Teslovich (2010)](https://www.ncbi.nlm.nih.gov/pubmed/20686565) with additional 1000Genome SNPs imputed by [Pickrell (2014)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3980523/).


## Parameter file for enloc

Parameters required by the integrative analysis are supplied through a single text file. An example of the parameter file is provided [here](../examples/HDL_blood.enloc.params). The informative line of the parameter file has the following format
```
KEYWORD   VALUE
```
The keywords are case-insensitive. Comments line are allowed in the parameter file by annotating a ```#``` at the beginning of the comment.

### Required parameters

* **BIN_DIR**:  ```enloc``` expects all the executable utilities, namely ```mi_eqtl```, ```get_pip```, ```compute_rcp```, ```dap1```, ```openmp_wrapper``` and ```torus``` can be accessed from a single directory, and BIN_DIR specifies the location of this directory containing all the utilities. Users can make symbolic links to  BIN_DIR instead of copying the executables. Relative path is allowed.

* **GWAS_DATA**: location of the gzipped GWAS z-score file. Relative path is allowed.
* **QTL_FM_DIR**: location of the directory containing fine-mapping results of molecular QTL analysis. Relative path is allowed.
* **OUT_DIR**: output (and working) directory name. enloc will create the directory if it does not already exist. Relative path is allowed.
* **TRAIT_NAME**: the name of the GWAS trait



## Running enloc

Perform the integrative analysis by issuing the following command 
``` enloc parameter_file```
in the command line.


## Results

Upon finishing the analysis, enloc will output the final results (along with intermediate result/script files) into OUT_DIR direcotry. The two most important result files summarize the results from enrichment and colocalization analyses.

### Enrichment analysis summary

This file is named ```TRAIT_NAME.enrich.est```. It summarizes the enrichment estimation from multiple imputations. An example output is given below
```
 -10.380    -10.435    -10.324
  4.759      3.098      6.421
```
The first line indicate the estimate of the intercept term (first column) and its 95\% confidence interval (second and third column). The second line represents the enrichment estimate in log-odds ratio (first column) and its corresponding 95\% confidence interval (second and third column).

### Colocalization result

The results of colocalization analysis are summarized in the file ```TRAIT_NAME.enloc.rst```. The columns of this file are
```
gwas_locus   molecular_qtl_trait    locus_gwas_pip    locus_rcp   lead_coloc_SNP  lead_snp_scp
```
Specifically,  ```gwas_locus``` indicates the locus name in GWAS analysis. The entry  ```molecular_qtl_trait``` represents the specific molecular trait examined. In the case of eQTL analysis, this entry would indicate the gene name.  The entry  ```locus_gwas_pip``` represents the cumulative posterior inclusion probability (PIP) of the locus containing a causal GWAS hit. The entry ```locus_rcp``` represents the regional colocalization probability of the overlapping regions (between GWAS and molecular QTL) containing a co-localized signal. The entry ```lead_coloc_SNP``` provides the lead SNP with highest SNP-level colocalization probability whose exact value is shown in the entry ```lead_snp_scp```.

An example output is shown below
```
 Loc1615   ENSG00000170866      1.000      0.988          chr19:54797848       0.974
 Loc1129   ENSG00000134825      1.000      0.897          chr11:61557803       0.885
 Loc949    ENSG00000155158      1.000      0.867           chr9:15304782       0.693
 Loc1615   ENSG00000170889      1.000      0.851          chr19:54800500       0.487
```
The entries are sorted according to the descending order of thier locus_rcp values.

Besides the summary information, the detailed SNP-level information on colocalization analysis is saved in the directory OUT_DIR/rcp_out.
