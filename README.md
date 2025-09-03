# Fish-Metabarcoding-with-Qiime2-for-12S-locus

_Martin Holguin Osorio_  
_June 2021_  
_Version 3_  

## Metadata File Creation and Workspace Preparation
The `metadata.tsv` file must follow a specific order and organization to work correctly. More information in the [Qiime2 metadata tutorial](https://docs.qiime2.org/2020.11/tutorials/metadata/).  
It is recommended to use the **Keemei add-on** in Google Sheets to create the `metadata.tsv` file in the fastest and simplest way. More [info here](https://keemei.qiime2.org).

* After downloading `metadata.tsv` using Google Sheets:

```bash
# navigate and place the metadata file in the working directory 
cd /home/martin/eDNA/4/12S

# put raw reads into a new folder "datos"
mkdir datos

# create an output folder for results
mkdir salidas
```

## Data Import

After seeing the nature of this data (demultiplexed, with paired sequencing and barcodes in the sequences), I decide that I have to use another command
#to import the data, so I leave the names of the raw data (leaving them as they are, without altering anything) and place them in the “data” folder.
```
#importo los datos a qiime2
# import data into Qiime2
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path datos/ \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path salidas/secs_multiplexadas.qza
```

## Demultiplexing 
```
# as these data are already demultiplexed, only rename
mv salidas/secs_multiplexadas.qza salidas/secs_demultiplexadas.qza

# create summary visualization
qiime demux summarize \
  --i-data salidas/secs_demultiplexadas.qza \
  --o-visualization salidas/secs_demultiplexadas.qzv

# open viewer
qiime tools view salidas/secs_demultiplexadas.qzv

# note: file "untrimmed.qza" contains all unassigned sequences
```

## Noise Removal (Denoising with DADA2) 
```
# denoising with parameters from Mathon’s work to rescue more diversity and generate more colorful heat trees
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs salidas/secs_demultiplexadas.qza \
  --p-trim-left-f 0 \
  --p-trim-left-r 0 \
  --p-trunc-len-f 0 \
  --p-trunc-len-r 0 \
  --p-max-ee-f 2 \
  --p-max-ee-r 2 \
  --p-trunc-q 2 \
  --p-chimera-method none \
  --o-table salidas/tabla.qza \
  --o-representative-sequences salidas/secs_representativas.qza \
  --o-denoising-stats salidas/resumen_denoising.qza

# quantitative summary of significant sequences
qiime feature-table summarize \
  --i-table salidas/tabla.qza \
  --o-visualization salidas/tabla.qzv \
  --m-sample-metadata-file metadata.tsv

# denoising summary
qiime metadata tabulate \
  --m-input-file salidas/resumen_denoising.qza \
  --o-visualization salidas/resumen_denoising4.qzv

# representative sequences
qiime feature-table tabulate-seqs \
  --i-data salidas/secs_representativas.qza \
  --o-visualization salidas/secs_representativas.qzv

# view results
qiime tools view salidas/tabla.qzv

```


```
# Taxonomic Assignment with Sklearn
qiime feature-classifier classify-sklearn \
  --i-classifier ../../BdD/Locales/12S/classifier_12S_v1.qza \
  --i-reads salidas/secs_representativas.qza \
  --p-confidence 0.70 \
  --p-read-orientation same \
  --o-classification salidas/taxonomy_12S.qza

# taxonomy visualization
qiime metadata tabulate \
  --m-input-file salidas/taxonomy_12S.qza \
  --o-visualization salidas/taxonomy_12S.qzv

# taxonomy barplot
qiime taxa barplot \
  --i-table salidas/tabla.qza \
  --i-taxonomy salidas/taxonomy_12S.qza \
  --m-metadata-file metadata.tsv \
  --o-visualization salidas/taxa_bar_plots.qzv

# view
qiime tools view salidas/taxa_bar_plots.qzv
```

## Environmental Phylogeny

hago filogenia con la base de datos el comando phylogeny align-to-tree-mafft-fasttree hace todo el pipeline para generar la filogeniaPhylogeny is generated using the align-to-tree-mafft-fasttree pipeline, which includes:
*  qiime alignment mafft ...
*  qiime alignment mask ...
*  qiime phylogeny fasttree ...
*  qiime phylogeny midpoint-root ...
  
This command is itself a pipeline (shortcut for the above steps).
```
# build phylogeny
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences salidas/secs_representativas.qza \
  --output-dir salidas/filogenia_ambiental

# tree visualization with species detected in taxonomic assignment
qiime empress community-plot \
    --i-tree salidas/filogenia_ambiental/rooted_tree.qza \
    --i-feature-table salidas/tabla.qza \
    --m-sample-metadata-file metadata.tsv \
    --m-feature-metadata-file salidas/taxonomy_12S.qza \
    --o-visualization salidas/empress-tree.qzv

```





