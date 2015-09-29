A Walkthrough of Co-occurrence Analyses
=============

These are R scripts used to perform co-occurrence analysis following the paper,
 [Demonstrating microbial co-occurrence pattern analyses within and between ecosystems](http://journal.frontiersin.org/Journal/10.3389/fmicb.2014.00358/full).

Pulling data from MGRAST
===========

First, we pulled data from [MGRAST](http://metagenomics.anl.gov/) using the script
 [pulling_data_from_MGRAST_with_matR.R](https://raw.githubusercontent.com/ryanjw/co-occurrence/master/pulling_data_from_MGRAST_with_matR.R).  
This script uses APIs to pull 16S rRNA amplicon datasets from the MGRAST database. See the paper link above for a full description of data used here.  

When pulling data from MGRAST, you are required to have an authentication key that can be received after registering with the database.  The authentication key then goes between the `""` in the `msession$setAuth("")` command within the script.    

If problems occur when trying to pull data from MGRAST, or if you wish to skip this step, .csv's of the data can be downloaded here:  
[Data summarized by bacterial order](https://github.com/ryanjw/co-occurrence/blob/master/data/total_order_info.csv)  
[Data summarized by bacterial family](https://github.com/ryanjw/co-occurrence/blob/master/data/total_family_info.csv)

The data tables are organized as the following:

|reads |MGRASTid |trt |rep |order 1 |order 2 |order 3 |... |
|:-----|:--------|:---|:---|:-------|:-------|:-------|:---|
|1000 |mgm1234567.89  |soil |forest |3|0|50|... |
|1500 |mgm1234567.10  |soil |desert |10|1|0|... |
  

Once the data has been pulled into R using `read.csv()`, the co-occurrence analysis can begin.

Performing the Analysis
=======================

The script, [co_occurrence_pairwise_routine.R](https://raw.githubusercontent.com/ryanjw/co-occurrence/master/co_occurrence_pairwise_routine.R),
is used to perform co-occurrence analysis based on the organization of the data listed above.  It can be customized to work for any dataset where samples are rows and colummns are sample information (which needs to be considered when creating iterators for `for` loops) along with abundance of each OTU.  

In short, this script uses nested for loops to perform pairwise correlations between each column in the dataset.  Here, a Spearman's correlation is used to avoid data transformation issues that can occur with these types of data.  The product of the script is a data frame called 'results' that lists the trt (i.e. the environment that the samples originate from), the pair of OTUs, correlation coefficient, P value for statistical test, and the abundances of the OTUs.

MGRAST does also include some Eukaryotic taxa that are not part of the analysis here.  They are removed in the script as well.    

A quicker way to perform this analysis uses the function in the script, [edgelist_creation.R](https://raw.githubusercontent.com/ryanjw/co-occurrence/master/edgelist_creation.R).  This script uses a function called `co_occur_pairs()` that can be used on the datasets.  This function works returns a data frame with no correlation values but is grouped by the correlation cut-off that has been predetermined in the script.

Testing for Differences in Network Topology
==========================================

One analysis that can be performed is a test to see if networks have different structures (i.e. topology).  The rationale behind this analysis is the following; If you assume that two networks have equivalent sets of nodes (microbial taxa) you can test to see if the edges between nodes (correlation from co-occurrence analysis) from the two networks are different in a multivariate framework.  Using a permutational multivariate analysis of variance (PERMANOVA), we can use permutations of edges from one network are different from another.  The beginning portion of the script [co-occurrence_permanova_sim.R](https://raw.githubusercontent.com/ryanjw/co-occurrence/master/co-occurrence_permanova_sim.R).

The script for actually performing the PERMANOVA on the data presented here is located in [permanova_script.R](https://raw.githubusercontent.com/ryanjw/co-occurrence/master/permanova_script.R).  Note that this test can take a while to run, and should be run with caution (or on a faster computer).  

Community Detection
===================

Another analysis that can be done among these co-occurrence relationships is community detection.  In short, this analysis looks for sub-networks within the larger co-occurrence network.  In other words, it finds small groups of highly-connected nodes that are sparsely connected with the rest of the network.  The script, [comm_stat_function.R](https://raw.githubusercontent.com/ryanjw/co-occurrence/master/comm_stat_function.R), contains a function that can return the communities within a network.  This script works with the object, `results`, produced by the [co_occurrence_pairwise_routine.R](https://raw.githubusercontent.com/ryanjw/co-occurrence/master/co_occurrence_pairwise_routine.R) script.  This script also considers networks constrained by different correlation strengths along with both positive co-occurrence and negative co-occurrence networks.  If different cut-offs would like to be used, edit the `rhos<-c(-.75,-.5,.5,.75)` statement at the top of the script.

It should be noted that often, in the literature, cut-offs based on P-values are often used after a correction through some sort of methodology.  The dominant method of P-value correction used is based on a q-value based on false discovery rate (FDR).  Within the manuscript we discuss our choice of a cut-off based on correlation strength rather than P-value.  

Other Network Statistics
========================

Other network statistics can be generated from the script, [network_statistics.R](https://raw.githubusercontent.com/ryanjw/co-occurrence/master/network_statistics.R). Here, the function `network_stats()` produces the clustering coefficient for the network, the clustering coefficient for a random network of the same size, and the ratio between these two values.  These values allow you to determine whether clustering is occurring at a level that is more or less than random.  These loops can also be easily modified to find the degree distributions with `degree()`, betweenness with `betweenness.centrality()`, or any other statistics that are needed.       