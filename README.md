# tutorials
Tutorials for working and plotting in R, with examples from genomics data and public datasets.

[Getting started with R](https://github.com/lapatel22/tutorials/blob/main/dataframe_manipulation_quarto.md) <br>
[The Tidyverse package in R](https://github.com/lapatel22/tutorials/blob/main/R_tidyverse_cheatsheet.md) <br>
[GGplot tutorial](https://github.com/lapatel22/tutorials/blob/main/GGplot_quarto.md) <br>

  Covers basic plot types in GGplot, and helpful shortcuts: 
    - custom faceting
    - using categorical variables for plot aesthetics
    - Axes scaling, breaks 
    - Theme aesthetics, labelling for published figures (custom sizing, rotations, legends)
    - Custom color schemes (Rcolorbrewer, unikn)
    - Combining plots for published figures

[Custom plotting functions](https://github.com/lapatel22/tutorials/blob/main/Laurens_functions.md) <br>
  Includes code for plotting common HOMER outputs, including:
    - preprocessing of annotatePeaks histograms or counts files
    - plotting HOMER annotatePeaks histograms
    - calculating signal at histograms with Area-Under-Curve (used by ChIP-wrangler to estimate IP efficiency)
    - Scatterplots of annotatePeaks counts files
    - DESeq2-based volcano plots
    - Quantiling peaks based on annotatePeaks counts files, extracting peak sets for separate analysis
    - Minmax normalization, Rsquared calculations
    - Plotting QC data from tag directories (Nucleotide frequencies and tag length distributions)
