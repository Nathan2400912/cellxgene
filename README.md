# Example of converting of Seurat object into Anndata object for uploading into public repository Cellxgene 

The goal of this project was to convert 2 Seurat objects from a specific study: 
Smoking modulates different secretory subpopulations expressing SARS-CoV-2 entry genes in the nasal and bronchial airways
to an Anndata object for uploading into a single-cell RNA-seq (scRNA-seq) public repository called Cellxgene. 

# Background 
The objects used in this project contain scRNA-seq data. One object contains samples from nasal swabbings and the other from bronchial swabbings. The swabbings were conducted as part of the study involving participants with indeterminate pulmonary nodules undergoing lung cancer screening at Boston Medical Center and Lahey Hospital Medical Center.

## Procedures
There are three specific steps in this conversion process: 1) Converting the object from Seurat to Anndata; 2) Making sure the Anndata objects meet all the requirements needed by Cellxgene; 3) Validating the Cellxgene objects. Here is a brief description of what was done.

1) Converting the Seurat object to Anndata
There are many ways to convert a Seurat object into an Anndata object. However, the most convenient way I found to retain most of the information in the original object is first to convert the Seurat object into an h5Seurat object, then convert the corresponding h5Seurat object into an h5ad file. Here is an example:
```
# Packages required
library(Seurat)
library(SeuratData)
library(SeuratDisk)

# Before converting the object, make sure that you have the correct assay wanted in the Anndata object set as the current default
example <- readRDS("example_object.rds")
DefaultAssay(example) <- "RNA"

# Conversion code
SaveH5Seurat(example, filename = "example_object.h5Seurat")
Convert("example_object.h5Seurat", dest = "h5ad")
```

2) Adding in required components set by Cellxgene
After conversion, some components of the original object may be lost, such as pca/umap etc. Therefore, you can manually save your pca or umap as csv files and add them back into the Anndata object. Below is a simple example of how that could be done:
```
# Extract PCA matrix (embedding) from Seurat in R
pca_matrix <- example[["pca"]]@cell.embeddings

# Save it as a CSV file
write.csv(pca_matrix, "example_pca_matrix.csv")

# Adding them back into Anndata object
pca_matrix = pd.read_csv("example_pca_matrix.csv", index_col=0)  
example.obsm['X_pca'] = pca_matrix
```

Other than that, there are many requirements of the dataset laid out by cellxgene. Here is the website for the requirements:
[Cellxgene Requirements](https://cellxgene.cziscience.com/docs/032__Contribute%20and%20Publish%20Data)
To see how you can meet all these requirements, take a look at the uploaded Jupyter notebooks that contain the detailed steps of how I fulfilled the cellxgene requirements for both of the Anndata objects in this project. Note that the steps I took do not necessarily neccesate how you will perform this requirement, this step will be highly dependent on what is contained in your original object, the steps taken in the notebook are specific to our study of interest. 

```

