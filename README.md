# Example of Converting Seurat Objects to AnnData for Cellxgene Repository
This repository demonstrates the process of converting Seurat objects into AnnData format, suitable for uploading to the public single-cell RNA-seq (scRNA-seq) repository Cellxgene.

## Project Overview
The project involves the conversion of two Seurat objects derived from the study:

**Smoking Modulates Different Secretory Subpopulations Expressing SARS-CoV-2 Entry Genes in the Nasal and Bronchial Airways** (Xu et al., 2022).

The goal is to prepare these objects for publication in Cellxgene while ensuring compliance with repository requirements. The Seurat objects contain scRNA-seq data from nasal and bronchial brush samples collected from participants undergoing lung cancer screening.

## Background
The study investigates how smoking influences the expression of SARS-CoV-2 entry genes (ACE2, TMPRSS2, and CTSL) in different secretory cell populations of the nasal and bronchial airways. scRNA-seq data was generated from nasal and bronchial samples, revealing distinct cell populations and differential gene expression patterns linked to smoking status.

The Seurat objects used in this project represent these datasets:
- Nasal Brushings: Samples from nasal swabs.
- Bronchial Brushings: Samples from the bronchial airways.

### Study Highlights
- Smoking is associated with increased ACE2 and TMPRSS2 expression in bronchial goblet cells.
- ACE2 expression in nasal secretory cells is minimally affected by smoking.
- The findings provide insights into how smoking may modulate susceptibility to SARS-CoV-2 in distinct airway compartments.


## Workflow
### 1) Converting Seurat to Anndata
There are many ways to convert a Seurat object into an Anndata object. However, the most convenient way I found to retain most of the information in the original object is first to convert the Seurat object into an h5Seurat object, then convert the corresponding h5Seurat object into an h5ad file. Here is an example:
```
# Load necessary libraries
library(Seurat)
library(SeuratDisk)

# Load the Seurat object
example <- readRDS("example_object.rds")
DefaultAssay(example) <- "RNA"

# Convert Seurat to h5Seurat
SaveH5Seurat(example, filename = "example_object.h5Seurat")

# Convert h5Seurat to AnnData
Convert("example_object.h5Seurat", dest = "h5ad")
```

### 2) Adding Required Components for Cellxgene
Certain components, such as PCA and UMAP embeddings, may not transfer during conversion. These components must be re-added manually. For example:
```
# Save PCA matrix from Seurat object
pca_matrix <- example[["pca"]]@cell.embeddings
write.csv(pca_matrix, "example_pca_matrix.csv")

# Load and add PCA matrix to AnnData object in Python
import pandas as pd
import anndata as ad

adata = ad.read_h5ad("example_object.h5ad")
pca_matrix = pd.read_csv("example_pca_matrix.csv", index_col=0)
adata.obsm['X_pca'] = pca_matrix.values
adata.write("example_object_updated.h5ad")
```

Other than that, there are many requirements laid out by cellxgene. [Here](https://cellxgene.cziscience.com/docs/032__Contribute%20and%20Publish%20Data) is the website for the requirements.
To see how you can meet all these requirements, take a look at the uploaded Jupyter notebooks that contain the detailed steps of how I fulfilled the cellxgene requirements for both of the Anndata objects in this project. Note that the steps I took do not necessarily translate to other studies, this step will be highly dependent on what is contained in the original object, the steps taken in the notebook are specific to the study of interest. 

### 3) Validating the final Anndata object
Once the requirements in both objects have been fulfilled, the objects then can be validated to ensure proper documentation for upload. 
```
git clone https://github.com/chanzuckerberg/single-cell-curation.git
cd single-cell-curation
python3 validate.py --input-file example_object_updated.h5ad
```
or
```
pip install cellxgene-schema
cellxgene-schema validate example_object_updated.h5ad
```

## Repository Structure
### Key Files
1. **Seurat_conversion.rmd**: R Markdown file documenting Seurat-to-AnnData conversion steps.
2. **BronchialCellxgene.ipynb**: Jupyter notebook detailing modifications to the bronchial AnnData object.
3. **NasalCellxgenecopy.ipynb**: Jupyter notebook detailing modifications to the nasal AnnData object.

## Additional Resources
- **Study DOI**: 10.1038/s41598-022-17832-6
- **Cellxgene Documentation**: [Link](https://cellxgene.cziscience.com/docs/032__Contribute%20and%20Publish%20Data)

## Citation
Xu, K., Shi, X., Husted, C., et al. (2022). Smoking modulates different secretory subpopulations expressing SARS-CoV-2 entry genes in the nasal and bronchial airways. Scientific Reports, 12, 18168. https://doi.org/10.1038/s41598-022-17832-6.
