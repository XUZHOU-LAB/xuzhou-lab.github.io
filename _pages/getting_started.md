---
layout: posts
permalink: /getting_started/
title: "Getting Started"
author_profile: true
last_modified_at: 2022-05-27T11:59:26-04:00
toc: true
---

<div style="font-size: 0.9rem; line-height: 1.55;" markdown="1">

Use [our online tool](linsig_online.md) or [install it](installation.md).
### LinSig Instructions

This webapp is designed to deconvolute the interaction between 2 signals.
By providing a RNAseq dataset of an experiment determined to study the interaction of two signals, the user can easily analyze and interact with the data.

In order to do this, the experiment needs to consist of 4 RNAseq analyses, each having 2 replicates or more. The following analyses are required in the input dataset for this webapp.

* Control (> 2 replicates)
* Condition A (> 2 replicates)
* Condition B (> 2 replicates)
* Condition A + B (> 2 replicates)

The input dataset needs to be a CSV file, with the columns structured in the following way:

rownames | Ctrl_A | Ctrl_B | Cond1_A | Cond1_B | Cond2_A | Cond2_B | Cond12_A | Cond12_B

Each row should contain the RNA count data (or TPM) for a gene.
All rownames should be unique, duplicate genes are not allowed.

#### Workflow

An example dataset can be downloaded on the top of this page with the **Download Example Dataset** button [1].

1. Upload your dataset by clicking on the **Upload CSV** button

2. Adjust the number of **replicates** to the number of replicates in your dataset

3. Press **Deconvolute** to run the model with the default settings.

4. Press **Compute FDR** to compute the FDR-statistic (q-value)


----

#### Brief Model Overview

The linear deconvolution of the signal interaction can be described mathematically as follows:

Y ~ $\beta$ * X + $\epsilon$




$$\begin{bmatrix}
ctrl \; vs \; condA\\ 
condB \;vs \; condAB\\ 
ctrl \; vs \; condB\\ 
condA \; vs \;condAB\\ 
ctrl \; vs \; condAB
\end{bmatrix}
=
\begin{bmatrix}
0 & 1 & 0\\ 
0 & 1 & 1\\ 
1 & 0 & 0\\ 
1 & 0 & 1\\ 
1 & 1 & 1
\end{bmatrix}
*
\begin{bmatrix}
\beta_{condB} \\ 
\beta_{condA} \\ 
\beta_{condAB}
\end{bmatrix}
+ 
\begin{bmatrix}
\epsilon_{ctrl/condA} \\ 
\epsilon_{condB/condAB}\\ 
\epsilon_{ctrl/condB} \\ 
\epsilon_{condA/condAB} \\ 
\epsilon_{ctrl/condAB}
\end{bmatrix}$$


Where `Y` is a vector of the measured log fold change responses measured in the RNAseq experiment. We use 5 comparisons: 

Control/ConditionA, 
Control/ConditionB, 
Control/ConditionAB, 
ConditionAB/ConditionA, 
ConditionAB/Condition

More information on this method can be found in the 2011 paper by Zhou & O'Shea [2].

The covariance matrix of the linear model is used to calculate the P-value of the fit for each gene. 
Each gene fit also comes with an R2 value, which can be used as a filter to discard model fits with high unexplained variance.

----

#### Explanation of the sliders and numeric inputs

- **Replicates**: Adjust to the number of replicates in the experiment

- **Add Pseudo Count**: Since we are working with count data and need to compute ratios, it can occur that the counts for the conditions control and A are 0 and 1000. This workflow uses the Log2 ratios between conditions as input for the linear model. Since the ratio between 0 and any non-zero number is infinite, we need to deal with this by adding a "pseudo" count. More info on this can be read in "Naught all zeros in sequence count data are the same" by J. Silverman et al, 2020.

- **Threshold**: RNA transcript count (or TPM) threshold. A count threshold of 10 is often used.

- **LFC threshold**: Here it's possible to filter for genes that are not interesting. The standard setting is 0.585 (1.5 fold change), and this keeps only genes where the fold change between at least two conditions is 1.5 or higher.

- **Null-Hypothesis Test FC Threshold**: With the variance/covariance matrix derived from the linear models for each gene, we can compute if a gene is significantly upregulated. The standard value for this is 1.5, meaning that for each gene we will test if the fold change is siginificantly more than 1.5. This value is closely related to the **LFC threshold** filter.

- **R2 Threshold**: The LinSig model deconvolutes signals using a linear model. The adjusted R-squared value of this model is used to filter out gens with hgh unexplained variance. The default value is 0.8.

- **Deconvolute Signals**: The button to run the model. After pressing, a progress box should pop up in the bottom right corner demonstrating the progress. It usually takes 30 seconds to run the model.

- **FDR genes**: Indicates the size of the random sampling to be used. A higher number is more accurate but takes a lot longer to compute. The default value of 40000 samples takes around 1 minute.

- **Compute FDR**: Compute the false discover statistics using random sampling non-parametric approach.

#### Explanation of the output results

- **Home page**: Here you will find a datatable with all the significant genes in the dataset. It's possible to download this dataset at the bottom left of the page. There's also an informative Venn diagram that demonstrates how many genes are regulated by signal A/B/AB or a combination thereof.


- **R2~Beta Plots**: Here you can find 3 figures for signal A, B and A+B. The R2 of the linear model is plotted against the $\beta_{condA}$, $\beta_{condB}$, $\beta_{condA+B}$ component for each gene.

- **FDR**: This page shows some figures about the False Discovery Rate calculation based on the random sampling of the input dataset. The first plot shows the mean counts plotted against the variance (mean-variance plot). The second plot is demonstrating the chance of a false discovery (a gene that looks like it's regulated but its not actually regulated) over a varying fold change. The chance of a false discovery is the lowest when the fold change is high, because its very unlikely for a gene to be appear upregulated when there's such a big difference between e.g. the control condition and condition A.

- **Volcano Plots**: This is a common plot with differential expression analysis. The y-axis shows log p-value and x-axis the fold change. The user can drag and drop a selection square in the figure in order to look at a certain group of interest. 

- **Heatmap**: This plot shows a heatmap of genes clustered by regulation type. Only clusters with more than 20 genes are included.

- **Enrichment Analysis**: Here the user can select clusters and perform Gene Enrichment Analysis using the clusterProfiler package in R. This process can take up to 5 minutes, depending on the computer used.




For questions, issues or suggestions:

sybren.bouwman@gmail.com

http://www.xuzhoulab.com


[1]. This dataset is reduced RNAseq dataset derived from the following paper: *Predicting gene level sensitivity to JAK-STAT signaling perturbation using a mechanistic-to-machine learning framework.
Neha Cheemalavagu, Karsen E. Shoger, Yuqi M. Cao, Brandon A. Michalides, Samuel A. Botta, James R. Faeder, Rachel A. Gottschalk
bioRxiv 2023.05.19.541151; doi: https://doi.org/10.1101/2023.05.19.541151*

[2]. *Zhou X, O'Shea EK. Integrated approaches reveal determinants of genome-wide binding and function of the transcription factor Pho4. 
Mol Cell. 2011 Jun 24;42(6):826-36. doi: 10.1016/j.molcel.2011.05.025. PMID: 21700227; PMCID: PMC3127084.*

</div>