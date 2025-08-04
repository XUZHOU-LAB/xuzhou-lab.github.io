---
layout: posts
permalink: /getting_started_old/
title: "Getting Started"
author_profile: true
last_modified_at: 2022-05-27T11:59:26-04:00
toc: true
math: true
---

Use [our online tool](linsig_online.md) or [install it](installation.md).


#### Overview

LinSig is a web-based tool designed to help researchers identify and interpret how two biological signals interact at the transcriptomic level. By uploading RNA-seq data from a carefully designed experiment, users can explore how combinations of signals affect gene expression - beyond what would be expected from each signal alone.

*Note: A future version will support analysis of interactions between three signals.*

#### Experimental Design Requirements

To use LinSig, your experiment must be designed to evaluate the interaction between two distinct signals (for example, cytokines, drugs, or environmental cues). Specifically, your RNA-seq dataset should include **four conditions**, each with at least **two replicates**:
•	Control (no signal)
•	Condition A (signal A only)
•	Condition B (signal B only)
•	Condition A + B (both signals applied together)

#### Input File Format

LinSig accepts data in CSV format (comma-separated values). Each row should represent one gene, and each column should represent a sample (i.e., a replicate from one of the four conditions).
•	The first column must contain the gene names or IDs (e.g., GeneSymbol or Ensembl ID). These must be unique—do not include duplicate gene entries.
•	The remaining columns must contain RNA expression values (e.g., raw counts or TPMs) for each replicate.
•	All conditions must have at least two replicates.
•	Do not include any missing values or non-numeric entries.
•	Ensure all row (gene) names are unique.

Example column format:

rownames | Ctrl_1 | Ctrl_2 | CondA_1 | CondA_2 | CondB_1 | CondB_2 | CondAB_1 | CondAB_2

Where:
- Ctrl_* = replicates for the control condition
- CondA_* = replicates for Condition A
- CondB_* = replicates for Condition B
- CondAB_* = replicates for Condition A + B combined


#### Workflow

An example dataset can be downloaded on the top of the [LinSig Online](https://xuzhou-lab.github.io/linsig_online/) page with the **Download Example Dataset** button [1].

1. Upload your dataset by clicking on the **Upload CSV** button

2. Adjust the number of **replicates** to the number of replicates in your dataset

3. Press **Deconvolute** to run the model with the default settings.

4. Press **Compute FDR** to compute the FDR-statistic (q-value)


----

#### Brief Model Overview


The LinSig algorithm uses a linear modeling approach to deconvolute the contributions of two individual signals (A and B) and their interaction to gene expression changes. Specifically, the model estimates how much of the observed gene expression change can be attributed to Signal A, Signal B, or their combined interaction.

The linear model is expressed as: **Y = Xβ + ε**.

Where:
- Y is a vector of observed log2 fold changes derived from RNA-seq comparisons.
- X is the design matrix that maps each comparison to the modeled effects.
- $$\beta$$ represents the estimated contributions of Signal A, Signal B, and their interaction (A×B).
- $$\varepsilon$$ is the error term, capturing the variance not explained by the model.


$$ Y \sim \beta * X + \epsilon $$


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


Here **Y** is the vector of measured log fold changes. The five comparisons are Control vs ConditionA, Control vs ConditionB, Control vs ConditionAB, ConditionAB vs ConditionA, and ConditionAB vs ConditionB.


#### Model Evaluation and Output

For each gene, LinSig fits this model and estimates the contribution of each component. The model fit is evaluated using:
•	P-value: derived from the covariance matrix of the linear regression; reflects statistical confidence in the model fit.
•	R² (coefficient of determination): quantifies how much of the variation in the observed data is explained by the model. Genes with low R² values may be filtered out due to poor model fit.

More detailed description of this modeling framework can be found in 
-	[Wu et. al, Cell, 2025.](https://pmc.ncbi.nlm.nih.gov/articles/PMC11118380/)
-	[Zhou & O’Shea, Molecular Cell, 2011](https://pmc.ncbi.nlm.nih.gov/articles/PMC3127084/).

----

#### Explanation of the sliders and numeric inputs

##### Deconvolution

- **Replicates**: Adjust to the number of replicates in the experiment

- **Add Pseudo Count**: Since we are working with count data and need to compute ratios, it can occur that the counts for the conditions control and A are 0 and 1000. This workflow uses the Log2 ratios between conditions as input for the linear model. Since the ratio between 0 and any non-zero number is infinite, we need to deal with this by adding a "pseudo" count. More info on this can be read in "Naught all zeros in sequence count data are the same" by J. Silverman et al, 2020.

- **Threshold**: RNA transcript count (or TPM) threshold. A count threshold of 10 is often used.

- **Deconvolute Signals**: The button to run the model. After pressing, a progress box should pop up in the bottom right corner demonstrating the progress. It usually takes 30 seconds to run the model.

##### Choosing filtering thresholds for clustering

- **R2 Threshold**: The LinSig model deconvolutes signals using a linear model. The adjusted R-squared value of this model is used to filter out gens with hgh unexplained variance. The default value is 0.8.

- **LFC threshold**: Here it's possible to filter for genes that are not interesting. The standard setting is 0.585 (1.5 fold change), and this keeps only genes where the fold change between at least two conditions is 1.5 or higher.

- **Null-Hypothesis Test FC Threshold**: With the variance/covariance matrix derived from the linear models for each gene, we can compute if a gene is significantly upregulated. The standard value for this is 1.5, meaning that for each gene we will test if the fold change is siginificantly more than 1.5. This value is closely related to the **LFC threshold** filter.


- **FDR genes**: Indicates the size of the random sampling to be used. A higher number is more accurate but takes a lot longer to compute. The default value of 40000 samples takes around 1 minute.

- **Compute FDR**: Compute the false discover statistics using random sampling non-parametric approach.


#### Explanation of the output results

Once your dataset is uploaded and analyzed, LinSig provides several interactive visualizations and result tables to help interpret signal-specific and combinatorial gene regulation. Below is a guide to each tab in the results interface:

- **Home page**: Here you will find a datatable with all the significant genes in the dataset. It's possible to download this dataset at the bottom left of the page. There's also an informative Venn diagram that demonstrates how many genes are regulated by signal A/B/AB or a combination thereof.


- **R2~Beta Plots**: Here you can find 3 figures for signal A, B and A+B. The R2 of the linear model is plotted against the $\beta_{condA}$, $\beta_{condB}$, $\beta_{condA+B}$ component for each gene.

- **FDR**: This page shows some figures about the False Discovery Rate calculation based on the random sampling of the input dataset. The first plot shows the mean counts plotted against the variance (mean-variance plot). The second plot is demonstrating the chance of a false discovery (a gene that looks like it's regulated but its not actually regulated) over a varying fold change. The chance of a false discovery is the lowest when the fold change is high, because its very unlikely for a gene to be appear upregulated when there's such a big difference between e.g. the control condition and condition A.

- **Volcano Plots**: This is a common plot with differential expression analysis. The y-axis shows log p-value and x-axis the fold change. The user can drag and drop a selection square in the figure in order to look at a certain group of interest. 

- **Heatmap**: This plot shows a heatmap of genes clustered by regulation type. Only clusters with more than 20 genes are included.

- **Enrichment Analysis**: Here the user can select clusters and perform Gene Enrichment Analysis using the clusterProfiler package in R. This process can take up to 5 minutes, depending on the computer used.




For questions, issues or suggestions, contact:

[sybren.bouwman@gmail.com](mailto:sybren.bouwman@gmail.com) and/or [diana.leung@childrens.harvard.edu](mailto:diana.leung@childrens.harvard.edu)

[http://www.xuzhoulab.com](http://www.xuzhoulab.com)


[1]. This dataset is reduced RNAseq dataset derived from the following paper: *Predicting gene level sensitivity to JAK-STAT signaling perturbation using a mechanistic-to-machine learning framework. Neha Cheemalavagu, Karsen E. Shoger, Yuqi M. Cao, Brandon A. Michalides, Samuel A. Botta, James R. Faeder, Rachel A. Gottschalk bioRxiv 2023.05.19.541151; doi: https://doi.org/10.1101/2023.05.19.541151*

[2]. *Zhou X, O'Shea EK. Integrated approaches reveal determinants of genome-wide binding and function of the transcription factor Pho4.
Mol Cell. 2011 Jun 24;42(6):826-36. doi: 10.1016/j.molcel.2011.05.025. PMID: 21700227; PMCID: PMC3127084.**

[3]. *Wu Z, Pope SD, Ahmed NS, Leung DL, Hajjar S, Yue Q, Anand DM, Kopp EB, Okin D, Ma W, Kagan JC, Hargreaves DC, Medzhitov R, Zhou X. Control of Inflammatory Response by Tissue Microenvironment. bioRxiv [Preprint]. 2025 May 30:2024.05.10.592432. doi: 10.1101/2024.05.10.592432. PMID: 38798655; PMCID: PMC11118380.*

