---
layout: posts
permalink: /getting_started/
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

##### Step 1 – Run the deconvolution
1. Upload your counts file.  
2. Set **Replicates** to match your dataset (≥ 2 per condition).  
3. Click **Run Deconvolution**.  
   A progress popup appears; 10 k genes typically finish in ≈30 s.  
   The preview table shows the first ten rows of deconvoluted statistics.

##### Step 2 – Filter & inspect
1. Go to **Filtering and Visualisation**.  
2. Decide whether to begin with  
   * **Default** thresholds (R² ≥ 0.8, |log₂FC| ≥ 0.585, P ≤ 0.05)  
   * **No filter** (inspect the raw model)  
   * or adjust the sliders / numeric boxes manually.  
3. Press **Apply thresholds**.  
4. Work through the **three plot blocks**:

   *Raise or lower sliders until the pictures look sensible, then regenerate the static heat-map preview (sampled to ≤ 2000 genes for speed).*


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

### Sliders & Numeric Inputs

#### Deconvolution

* **Replicates** – integer box; adjust to the number of replicates in the experiment (minimum 2).
* **Add Pseudo-count** – Since we are working with count data and need to compute ratios, it can occur that the counts for the conditions control and A are 0 and 1000. This workflow uses the Log2 ratios between conditions as input for the linear model. Since the ratio between 0 and any non-zero number is infinite, we need to deal with this by adding a "pseudo" count. More info on this can be read in "Naught all zeros in sequence count data are the same" by J. Silverman et al, 2020. The default here is 1.
* **Count Threshold** – RNA transcript count (or TPM) threshold. A count threshold of 10 is often used.
* **LOWESS Normalisation** – optional mean-variance smoothing (unchecked by default).
* **Run Deconvolution** – green button; runs the linear model with the settings above. A progress should pop up in the bottom right corner demonstrating the progress. It usually takes 30 seconds to run the model.

#### Filtering (clustering step)

Each threshold has a **slider and a numeric box** that stay synchronised. Editing either control switches the sidebar mode to **Custom**; click **Apply thresholds** to refresh all plots.

##### Choosing filtering thresholds for clustering

* **R2 Threshold**: The LinSig model deconvolutes signals using a linear model. The adjusted R-squared value of this model is used to filter out gens with hgh unexplained variance. The default value is 0.8.
* **LFC threshold**: Here it's possible to filter for genes that are not interesting. The standard setting is 0.585 (1.5 fold change), and this keeps only genes where the fold change between at least two conditions is 1.5 or higher.
* **p-value threshold**: Here it's possible to filter for genes that are not interesting. The standard setting is 0.585 (1.5 fold change), and this keeps only genes where the fold change between at least two conditions is 1.5 or higher.

Sidebar radio buttons offer two shortcuts:

* **Default** – resets to 0.8 / 0.585 / 0.05
* **No filter** – resets to 0 / 0 / 1 and shows every modelled gene

#### Empirical FDR

Inside the **Empirical FDR** tab you’ll find:

* **Compute empirical FDR** – amber button; simulates a null dataset the same size as the input and re-fits the model once (≈ 1 min for 40 k genes).
  The resulting panel plots null tails, real tails and their ratio (empirical FDR) with a dotted 5 % line. Adopt the p-value where the FDR curve meets 0.05 if you want gene-level 5 % control.

This plot is demonstrating the chance of a false discovery (a gene that looks like it's regulated but its not actually regulated) over a varying fold change. The chance of a false discovery is the lowest when the fold change is high, because its very unlikely for a gene to be appear upregulated when there's such a big difference between e.g. the control condition and condition A.


---

### Output Tabs

**R² histogram**
This panel shows how well the linear model explains each gene. The tall bars to the right of the red reference line are genes whose behaviour is mostly captured by the model; bars to the left are genes with a large unexplained component. If the histogram is heavily weighted to the left, lower the R² threshold so you do not discard most of the data. When the bulk of the distribution sits to the right of the line you can increase the threshold with little loss of information.

**MA plots (A, B and AB)**
Each scatter plots the log-averaged read count on the x-axis against the log₂ fold-change for that term on the y-axis. Genes that change strongly at very low counts often represent noise, because dividing by numbers near zero exaggerates fold-change. If you see tall spikes hugging the left axis, raise the |log₂FC| cut-off until those low-count outliers disappear while the mid-count cloud remains. When the points form a symmetric funnel centred on zero, the threshold is in a reasonable place.

**Volcano plots (A, B and AB)**
Here every dot’s horizontal position is its log₂ fold-change and its vertical position is the –log₁₀ p-value. A tidy volcano has a wide base of low-significance points, a clean lower half with few dots above the P-value cut, and two wings of highly significant, large-effect genes. If the cloud is diffuse, lower the P-value threshold or raise |log₂FC|. If nothing reaches the top corners, relax the cut-offs or verify that the experimental design has genuine signal.

**Empirical FDR curves**
The three line plots compare null and real tail probabilities across a grid of p-value thresholds; the ratio is the empirical false-discovery rate. The horizontal dotted line marks 5 %. Slide along the curve for each term until it crosses 0.05 and use that p-value as a principled cut-off. If a curve never drops below 0.05 you will not reach gene-level 5 % FDR without raising the fold-change requirement.

**Cluster-size bar plot**
Each bar counts how many genes fall into a mechanistic cluster after filtering. Very tall bars indicate broad, non-specific responses; vanishing bars signal that a cluster has been filtered away. Aim for bars between roughly ten and a few hundred genes so that downstream enrichment has enough statistical power without being overwhelmed.

**Venn diagram**
The coloured circles show how many genes are unique to A, unique to B, unique to AB, or shared. Large overlaps between A and AB or B and AB often suggest that the interaction term is driven by one main effect rather than genuine synergy. If the triple intersection dominates, tighten thresholds to separate the terms; if every intersection is empty, thresholds are probably too strict.

**UpSet plot**
Where the Venn gives a quick qualitative view, the UpSet plot quantifies every intersection. Tall bars for the single-term columns mean mostly independent responses, whereas tall bars for double and triple intersections mean shared regulation. Use it to confirm what you saw in the Venn and decide whether to focus on term-specific clusters or on genes regulated by multiple signals.

**Heat-map preview**
This static map displays a subsample of genes clustered by similarity across conditions. Block-diagonal colour patterns indicate coherent co-regulation within clusters. If the map is mostly noise or blank space, revisit the thresholds: too lenient produces noisy vertical stripes, too strict leaves only a few isolated rows. Once the blocks are crisp and sizable, the data are ready for functional enrichment.


A **Download filtered table (.csv)** button in the sidebar always exports the exact gene list shown in the current plots.





For questions, issues or suggestions, contact:

[sybren.bouwman@gmail.com](mailto:sybren.bouwman@gmail.com) and/or [diana.leung@childrens.harvard.edu](mailto:diana.leung@childrens.harvard.edu)

Lab homepage: [http://www.xuzhoulab.com](http://www.xuzhoulab.com)


[1]. This dataset is reduced RNAseq dataset derived from the following paper: *Predicting gene level sensitivity to JAK-STAT signaling perturbation using a mechanistic-to-machine learning framework. Neha Cheemalavagu, Karsen E. Shoger, Yuqi M. Cao, Brandon A. Michalides, Samuel A. Botta, James R. Faeder, Rachel A. Gottschalk bioRxiv 2023.05.19.541151; doi: https://doi.org/10.1101/2023.05.19.541151*

[2]. *Zhou X, O'Shea EK. Integrated approaches reveal determinants of genome-wide binding and function of the transcription factor Pho4.
Mol Cell. 2011 Jun 24;42(6):826-36. doi: 10.1016/j.molcel.2011.05.025. PMID: 21700227; PMCID: PMC3127084.**

[3]. *Wu Z, Pope SD, Ahmed NS, Leung DL, Hajjar S, Yue Q, Anand DM, Kopp EB, Okin D, Ma W, Kagan JC, Hargreaves DC, Medzhitov R, Zhou X. Control of Inflammatory Response by Tissue Microenvironment. bioRxiv [Preprint]. 2025 May 30:2024.05.10.592432. doi: 10.1101/2024.05.10.592432. PMID: 38798655; PMCID: PMC11118380.*

