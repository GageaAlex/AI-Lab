# AI-Lab

**Phase 1  —  Report setup & introduction 
**
**1.  Write your introduction (~200 words) 
**
Cover background, overall aim, and specific objectives based on your class discussion. 

Background: what is scRNA-seq and why does it matter for cancer biology? 

Overall aim: build a classifier to predict hypoxia vs normoxia from single-cell gene expression. 

Specific objectives: EDA, dimensionality reduction, classification, cross-dataset testing. 

**2.  Write materials & methods (~300 words) 
**
Describe the 4 datasets (2 cell lines x 2 sequencing technologies), the file formats, and planned methods. 

4 experiments: MCF7 and HCC1806, each profiled with SmartSeq and another technology. 

Files per experiment: one metadata TSV + one count matrix TXT. 

Note the 3 preprocessing levels: unfiltered, filtered, normalised + top 3000 genes. 

Describe why each method is appropriate — justify your choices. 

**3.  Set up libraries & environment 
**
Install and import all required libraries. Document versions in a setup cell. 

Core libraries: numpy, pandas, matplotlib, seaborn, sklearn. 

Pin versions for reproducibility. 

Use %matplotlib inline and sns.set(color_codes=True) in your setup cell. 

 

**Phase 2  —  Exploratory data analysis (EDA) 
**
**4.  Load & inspect metadata 
**
Read each of the 4 metadata TSV files. Describe what the rows and columns represent. 

Use pd.read_csv(..., delimiter='\t', index_col=0). 

The BAM filename encodes: plate number, well position, and condition (Hypo/Norm). 

Metadata has 8 columns: Cell Line, PCR Plate, Pos, Condition, Hours, Cell name, etc. 

**5.  Load & inspect count matrices 
**
Load each unfiltered count matrix. Note the shape, data types, and what rows/columns represent. 

Use pd.read_csv(..., delimiter=' ', index_col=0) — space-separated with quoted names. 

Rows = genes (gene symbol), columns = cells (BAM filenames). 

Check df.shape, df.dtypes, df.head(). Discuss gene identifier types. 

Convert to numpy if needed: X = df.to_numpy() — may need transposing later. 

**6.  Check for missing values 
**
Use df.isnull().sum(). Discuss what missing data would mean in this context. 

Expected: no NaN values — zeros are used instead (gene not detected in that cell). 

Zeros are NOT missing data in scRNA-seq; they reflect biological dropouts. 

If NaN were present: options are removal (df.dropna()) or imputation. 

**7.  Compute descriptive statistics & visualise distributions 
**
Examine count distributions per cell and per gene. Plot histograms. Are the data normalised? 

Use df.describe() for summary statistics (mean, std, min, max per cell). 

Plot: histogram of total counts per cell; histogram of genes detected per cell. 

Repeat for unfiltered, filtered, and normalised files — compare and discuss. 

Key question: how does the distribution change after normalisation? 

**8.  Find & handle duplicate genes 
**
Use df.duplicated(keep=False) to identify genes with identical count profiles across all cells. 

Example: 56 duplicate gene rows were found in the MCF7 SmartSeq dataset. 

Inspect them; compute their correlation matrix (df.T.corr()) to confirm identity. 

Drop with df.drop_duplicates() and reassign: df = df_noDup. 

IMPORTANT: always log what was removed and why — dropping a gene affects biological interpretation. 

**9.  Explore data structure: correlation & heatmaps 
**
Compute cell-cell and gene-gene correlations. Visualise with heatmaps and histograms. 

c = df.corr(); sns.heatmap(c, cmap='coolwarm', center=0) 

Also plot a histogram of correlation values: sns.histplot(c, bins=100). 

Discuss: are some cells poorly correlated? Why might that be (low-quality cells)? 

Discuss: high gene-gene correlation can cause multicollinearity issues in ML models. 

**10.  Repeat EDA for all 4 datasets, then compare 
**
Run the same EDA steps for each experiment. Discuss similarities and differences. 

Compare MCF7 vs HCC1806: similar numbers of cells and genes? 

Compare SmartSeq vs the other technology: does data quality differ? 

Discuss how technology affects downstream analysis choices. 

**11.  Switch to pre-processed data for ML 
**
From here, use the ...3000...train.txt file (filtered, normalised, top 3000 most variable genes). 

Discuss why 3000 was chosen as the feature threshold. 

Discuss alternatives: PCA-based reduction, mutual information, Lasso-based selection. 

Transpose matrix if needed so rows = cells and columns = genes (required by sklearn). 

 

**Phase 3  —  Unsupervised learning 
**
**12.  PCA — dimensionality reduction 
**
Apply PCA to the 3000-gene matrix. Plot explained variance and PCs coloured by condition. 

from sklearn.decomposition import PCA; X_pca = PCA(n_components=50).fit_transform(X) 

Plot a scree plot (explained variance per PC). 

Plot PC1 vs PC2, colouring points by Hypoxia/Normoxia label. 

Key question: does PCA visually separate the two conditions? 

**13.  Clustering 
**
Try k-means and/or hierarchical clustering. Compare cluster assignments to true labels. 

Try on both raw 3000-gene features and PCA-reduced features — compare results. 

Visualise with heatmaps and/or dendrograms. 

Quantify agreement with true labels using Adjusted Rand Index (ARI) or NMI. 

Discuss: do clusters match biological conditions, or batch/plate effects? 

 

**Phase 4  —  Supervised learning: classification 
**
**14.  Prepare labels & train/validation split 
**
Extract Hypoxia/Normoxia labels from metadata. Split into training and validation sets. 

Labels come from the metadata 'Condition' column (Hypo/Norm). 

Use train_test_split with stratify=y to preserve class balance. 

A separate held-out test set will be given at the end — do NOT use it during development. 

**15.  Train classifiers per cell line 
**
Try multiple ML methods. Compare accuracy, precision, recall, and F1 on the validation set. 

Try at minimum: Logistic Regression, Random Forest, and SVM. 

Use cross-validation (StratifiedKFold) rather than a single train/test split. 

Report confusion matrices for each classifier and discuss differences. 

Start from the 3000-gene data; optionally also try PCA-reduced features. 

Justify every methodological choice in writing. 

**16.  Feature selection 
**
Identify which genes are most informative. Apply at least one feature selection method. 

Options: Random Forest feature importances, SelectKBest, Lasso coefficients, mutual information. 

Compare: are the top genes the same across MCF7 and HCC1806? 

Discuss biological relevance: are top genes known hypoxia markers (e.g. HIF pathway targets)? 

**17.  Cross-cell-line & cross-technology testing 
**
Test classifiers trained on one cell line/technology on unseen cell lines/technologies. 

Train on MCF7 SmartSeq → test on HCC1806 SmartSeq (same tech, different cell line). 

Train on MCF7 SmartSeq → test on MCF7 other technology (same cell line, different tech). 

Key questions: does performance hold? Why might it drop? Does normalisation help? 

Are some ML methods more robust to cross-dataset transfer than others? 

**18.  Build a general classifier (all data combined) 
**
Train one classifier on all cell lines and technologies pooled. Evaluate generalisability. 

Pool all training data across the 4 experiments. 

Compare performance to the cell-line-specific classifiers. 

Discuss trade-offs: general classifier may be less accurate but more practical. 

This is likely the classifier you will use to predict the final held-out test set. 

 

**Phase 5  —  Final prediction & report 
**
**19.  Predict on the held-out test set 
**
Apply your best classifier to the test set provided by the instructor. Save predictions as a TSV. 

Output format: tab-delimited file with cell name and predicted label (Hypoxia/Normoxia). 

Double-check column names match what the instructor expects. 

Do NOT retrain or adjust your model after seeing the test set. 

**20.  Finalise notebook & submit 
**
Complete the notebook with all results, visualisations, and discussion. Submit notebook + predictions TSV. 

Every section needs: code, output/plots, and written interpretation. 

Capture the class discussion and justify every methodological choice. 

Connect findings back to general ML theory — avoid being too domain-specific. 

Send report to the instructor; it will be shared with other course instructors for evaluation. 

 

Tips from the instructor notebook 

Use both text and visualisations in every section — don't just show code output. 

Capture the discussion you had in class and with your group — this is part of the grade. 

Focus on general ML concepts, not just domain-specific biology knowledge. 

The final report should reflect YOUR group's own results, ideas, and discussion. 

Always log any data cleaning decisions (e.g. dropped genes) so results are reproducible. 

~200 words for the introduction, ~300 words for materials & methods. 
