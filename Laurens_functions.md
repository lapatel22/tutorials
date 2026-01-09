# Laurens_functions


- [Processing HOMER files](#processing-homer-files)
  - [get all coverage columns in tidy
    format](#get-all-coverage-columns-in-tidy-format)
  - [Yuwei sample naming](#yuwei-sample-naming)
  - [get coverage columns, NOT tidy
    format](#get-coverage-columns-not-tidy-format)
  - [Process HOMER annotated peak
    files](#process-homer-annotated-peak-files)
- [Plotting](#plotting)
  - [Seq stats plots](#seq-stats-plots)
  - [histograms](#histograms)
    - [area under curve for above
      histograms](#area-under-curve-for-above-histograms)
  - [Scatterplots](#scatterplots)
  - [Volcano plots (FC vs Log p
    value)](#volcano-plots-fc-vs-log-p-value)
  - [Quantile peaks](#quantile-peaks)
    - [Extract set of peaks you want by bin
      number](#extract-set-of-peaks-you-want-by-bin-number)
    - [plot max values of histogram](#plot-max-values-of-histogram)
    - [Calculate Signal at Peaks: Area under Histogram
      Curve](#calculate-signal-at-peaks-area-under-histogram-curve)
    - [Minmax normalize data (min to 0, max to
      1)](#minmax-normalize-data-min-to-0-max-to-1)
    - [Minmax normalization, set min/max to variable
      range](#minmax-normalization-set-minmax-to-variable-range)
    - [Calculated Rsquared](#calculated-rsquared)
- [split sample names with regex](#split-sample-names-with-regex)
  - [regex for my sample naming
    conventions](#regex-for-my-sample-naming-conventions)
- [workflow for all p300/H3K9ac binned
  histograms](#workflow-for-all-p300h3k9ac-binned-histograms)
- [Spike-in normalization](#spike-in-normalization)
  - [histogram normalization](#histogram-normalization)
  - [peak file normalization](#peak-file-normalization)
- [csRNAseq processing](#csrnaseq-processing)
  - [Tag Length Distributions (using example from
    chip)](#tag-length-distributions-using-example-from-chip)
  - [Base Frequency Distributions](#base-frequency-distributions)

``` r
library(tidyverse)
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ✔ purrr     1.0.2     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

# Processing HOMER files

## get all coverage columns in tidy format

``` r
process_histograms <- function(x, .x) {
    colnames(x)[1] <- "Distance_from_tss"
    x <- x %>% 
      ### first two rename_with uses are specific to my sample names
      # get rid of suffix to samples, leaving only sample name.Coverage
    rename_with(~ gsub(".nodup.hg38.tagdir", "", .x), contains("tagdir")) %>% 
      # get rid of any prefix before cell type (ex: directory path)
    rename_with(~ gsub(".+\\_HEK_", "HEK_", .x), contains("HEK")) %>%
      ### below two uses of rename_with are general to HOMER output
    rename_with(~ gsub("\\.[[:digit:]]$", "_minus", .x), contains("Tags")) %>% 
    rename_with(~ gsub("\\.\\.\\.", "_", .x), contains("Tags"))
    
    # select coverage columns only
    xcov <- x %>% select(contains("Coverage"))
    xcov$Distance_from_tss <- x$Distance_from_tss
    
    xcovlong <- 
    xcov %>% pivot_longer(
      cols = -"Distance_from_tss", 
      names_to = "Sample", 
      values_to = "Coverage")
  }
```

## Yuwei sample naming

``` r
process_histograms_YC <- function(x, .x) {
    colnames(x)[1] <- "Distance_from_tss"
    x <- x %>% 
    rename_with(~ gsub(".tagdir", "", .x), contains("tagdir")) %>% 
    rename_with(~ gsub("ChIP[[:digit:]]+\\.[[:digit:]]+\\_[[:alpha:]]+\\.[[:alpha:]][[:digit:]]\\_", "", .x), contains("HeLa")) %>%
    rename_with(~ gsub("\\.[[:digit:]]$", "_minus", .x), contains("Tags")) %>% 
    rename_with(~ gsub("\\.\\.\\.", "_", .x), contains("Tags"))
    
    xcov <- x %>% select(contains("Coverage"))
    xcov$Distance_from_tss <- x$Distance_from_tss
    
    xcovlong <- 
    xcov %>% pivot_longer(
      cols = -"Distance_from_tss", 
      names_to = "Sample", 
      values_to = "Coverage")
  }
```

## get coverage columns, NOT tidy format

``` r
process_histograms_cov <- function(x, .x) {
    colnames(x)[1] <- "Distance_from_tss"
    x <- x %>% 
    rename_with(~ gsub(".nodup.hg38.tagdir", "", .x), contains("tagdir")) %>% 
    rename_with(~ gsub(".+\\_HEK_", "HEK_", .x), contains("HEK")) %>%
    rename_with(~ gsub("\\.[[:digit:]]$", "_minus", .x), contains("Tags")) %>% 
    rename_with(~ gsub("\\.\\.\\.", "_", .x), contains("Tags"))
    
    xcov <- x %>% select(contains("Coverage"))
    
}
```

## Process HOMER annotated peak files

``` r
process_counts_annotpeaks <- function(counts_annotpeaks, .x) {
  colnames(counts_annotpeaks)[1] <- "PeakID"
  counts_annotpeaks <- counts_annotpeaks %>% 
    rename_with(~ gsub(".nodup.hg38.tagdir", "", .x), contains("tagdir")) %>% 
    rename_with(~ gsub(".+HEK", "HEK", .x), contains("Tag")) %>%
    rename_with(~ gsub("\\.[[:digit:]]$", "_minus", .x), contains("Tag")) %>% 
    rename_with(~ gsub("\\.Count.+", "", .x), contains("Tag"))
} 
```

Clean annotation:

``` r
clean_annotation_peakfile <- function(input_df) {
  Annotation <- input_df[['Annotation']]
  input_df$Annotation <- 
    gsub("\\(.*", "", as.character(input_df$Annotation))
  input_df$Annotation <- 
    gsub(" ", "", as.character(input_df$Annotation))
}
```

# Plotting

## Seq stats plots

tidy to compare all species together

``` r
seqstats_tidy <- 
    seqstats_IP %>% pivot_longer(
      cols = c(combined.tagdir.dm6, combined.tagdir.sac3, combined.tagdir.hg38), 
      names_to = "Species", 
      values_to = "Reads")
```

all species bar plots

``` r
seqstats_plots <- function(seqstats_tidy, antibody_name) {
  ggplot(seqstats_tidy %>% filter(antibody == antibody_name)) + 
    aes(fill=Species, y=Reads, x=as.factor(rep), group = interaction(timepoint, rep)) + 
      geom_bar(position="fill", stat="identity", alpha = 0.8) + 
    scale_fill_manual(values = c("grey50", "navy", "firebrick")) +
    labs(x = "Replicate", 
         y = "Read Proportion") + 
    facet_wrap(~ timepoint, scales = "free_x")
}
```

``` r
seqstats_plots(seqstats_tidy, "H3K27ac") + labs(title = "H3K27ac")
```

## histograms

Categorical coloring for 3 timepoints from Wang et al:

``` r
plot_histograms_natgen <- function(input_df, brewer_cols, color_names) {
  xvar <- colnames(input_df)[1]
  treatment_num <- length(unique(input_df[[3]]))
  treatment <- input_df[[3]]
  timepoint <- input_df[[4]]
  antibody_name <- input_df[[5]]
  cell_name <- input_df[[2]]
  brewer_pallet <- as.numeric(brewer_cols)

  input_df |>
    ggplot2::ggplot(ggplot2::aes(x = Distance_from_tss, y = Coverage, group=interaction(treatment, timepoint, antibody_name, replicate), color = timepoint)) +
    ggplot2::geom_line(alpha = 0.8, linewidth = 1.1) +
    ggplot2::labs(title = paste("Histogram of", antibody_name, "in", cell_name),
         x = xvar,
         y = deparse(colnames(input_df[7]))) +
    ggplot2::scale_color_manual(
      values = c("firebrick", "slateblue", "goldenrod2"),
      name = "Timepoint",
      labels = unique(input_df[[4]])) +
    ggplot2::theme_classic()
}
```

Categorical coloring for mitotic/interphase standard curve:

``` r
plot_histograms_stdcurve <- function(input_df, brewer_cols) {
  xvar <- colnames(input_df)[1]
  condition_num <- length(unique(input_df[[4]]))
  antibody_name <- input_df[[6]]
  cell_name <- input_df[[3]]
  brewer_pallet <- as.numeric(brewer_cols)

  input_df |>
    ggplot2::ggplot(ggplot2::aes(x = Distance_from_center, y = Coverage, group=id, color = treatment)) +
    ggplot2::geom_line(alpha = 0.9, linewidth = 1.1) +
    ggplot2::labs(title = paste("Histogram of", antibody_name, "in", cell_name),
         x = xvar,
         y = colnames(input_df[8])) +
    ggplot2::scale_color_manual(
      values = colorRampPalette(
        RColorBrewer::brewer.pal(brewer_pallet, "Blues"))(condition_num+4)[(4):(condition_num+4)],
      name = "Condition",
      labels = unique(input_df[[4]])) +
    ggplot2::theme_classic() +
    ggplot2::theme(legend.position = c(0.84, 0.76),
          legend.background = element_rect(
            size=0.7, linetype="solid",
            colour ="grey20"))
}
```

### area under curve for above histograms

Calculate Area Under Curve for HOMER histograms

- input df: histogram after 1) process_histograms and 2)
  sep_hist_byregex
- other parameters: – id: column created after sep_hist_byregex, unique
  id column – Coverage: column containing signal information – output
  df: contains all samples and the area under their histogram

``` r
histograms_signalarea <- function(input_df, id, Coverage, colname1) {
  # ensym() converts strings into symbols, so function inputs can be understood
  ### within ggplot
  id <- rlang::ensym(id)
  Coverage <- rlang::ensym(Coverage)
  colname1 <- rlang::ensym(colname1)
  # get a list of unique sample names
  samples <- unique(input_df$id)

  # make sure first column is named properly
  colnames(input_df)[1] <- "Distance_from_center"

  # get x variable
  x <- unique(input_df[[1]])

  # initialize empty matrix, then use to make dataframe output
  AUC_peaks <- matrix(data = "", nrow = length(samples), ncol = 1)
  AUC_peaks_df <- data.frame(AUC_peaks, row.names = samples)

  # fill in AUC dataframe for each sample
  for (i in 1:length(samples)) {

    y <- input_df |>
      dplyr::filter(id == samples[i]) |>
      dplyr::select(Coverage)
    y <- dplyr::pull(y, Coverage)

    AUC_peaks_df[i, ] <- DescTools::AUC(x, y, method = c("trapezoid"))

  }
 # uncomment below if this stops working
  #AUC_peaks_df

  # get data in numeric form
  AUC_peaks_df <- as.data.frame(sapply(AUC_peaks_df, as.numeric))
  #now get this also as an output:
  AUC_peaks_df

  # add in the treatment column for plotting
  treatment_df <- input_df |>
    dplyr::filter(Distance_from_center == "-2000") |>
    dplyr::select(treatment)

  treatment_vect <- dplyr::pull(treatment_df, treatment)

 # QC the treatment_vect
 # print(length(treatment_vect))

  # plot signal area
  ## first make colors vector
  color_vect <- unique(treatment_vect)

  # graph labels in title custom
  cell_type <- unique(input_df[[3]])
  AUC_name <- colnames(AUC_peaks_df[1])

  # plot
  AUC_peaks_df |>
    ggplot2::ggplot(ggplot2::aes(x = treatment_vect, y = AUC_peaks, color = treatment_vect)) +
    ggplot2::geom_point(size = 5, alpha = 0.7) +
    ggplot2::scale_color_manual(
      values = colorRampPalette(RColorBrewer::brewer.pal(9, "Blues"))(8)[3:8],
      name = "Condition",
      labels = as.character(unique(treatment_vect))) +
    ggplot2::theme_classic() +
    ggplot2::labs(title = paste(as.character(AUC_name), "in", as.character(cell_type), "cells"),
         x = "Condition",
         y = "Signal Area")

}
```

``` r
histograms_signalarea(hist_K9ac_allsamples_LP78_sepIP, colname1 = colname1) + 
  labs(title = "K9ac around Peaks", 
       subtitle = "Read normalized")
```

## Scatterplots

General scatterplot (log axes, xy line)

``` r
plot_scatter <- function(df, xvar, yvar) {
  ggplot(data = df) + 
    aes(x = .data[[xvar]], y = .data[[yvar]]) + 
    geom_point(alpha = 0.2, stroke = NA) + 
    scale_x_log10() + scale_y_log10() + geom_abline() 
}
```

Scatterplot with RColorBrewer colors based on categorical column

``` r
some_function2 <- function(df, quantile, xvar, yvar, regions, colorbrewer) {
  # below: extracts a particular subset of the string xvar, 
  # the subset based on which underscores its between
  cell_type <- str_match(xvar, '([^_]+)(?:_[^_]+){5}$')[,2]
  treatment <- str_match(xvar, '([^_]+)(?:_[^_]+){4}$')[,2]
  timepoint <- str_match(xvar, '([^_]+)(?:_[^_]+){3}$')[,2]
  antibody <- str_match(xvar, '([^_]+)(?:_[^_]+){1}$')[,2]
  regions <- rlang::ensym(regions)
  colorbrewer <- rlang::ensym(colorbrewer)
  df[, quantile] <- as.factor(df[, quantile]) 
  
ggplot(data = df) + 
    aes(x = .data[[xvar]], y = .data[[yvar]], 
        color = .data[[quantile]]) + 
    scale_x_log10() + scale_y_log10() + geom_abline() + 
    geom_point(alpha = 0.04, stroke = NA) + 
    labs(
      title = paste(antibody, "in", cell_type, "at", regions),
      x = str_remove(xvar, paste0(cell_type, "_")), 
      y = str_remove(xvar, paste0(cell_type, "_"))) +
   scale_color_manual(values = colorRampPalette(brewer.pal(9, paste0(colorbrewer)))(6)[3:6])
}
```

``` r
some_function2(counts_tss_hg38_dual, "HCT116_DMSO_0hr_1_H3K27ac_1.Tag", "HCT116_DMSO_0hr_2_H3K27ac_1.Tag", "all TSS", "Blues")
```

## Volcano plots (FC vs Log p value)

``` r
ggplot(diff_peaks_H3K27ac_4hrTRP_topsignal %>% 
  filter(DMSO.vs..4hrTRP.adj..p.value < 0.05)) + 
  aes(x = DMSO.vs..4hrTRP.Log2.Fold.Change, 
      y = -log(DMSO.vs..4hrTRP.adj..p.value), 
      color = (DMSO.vs..4hrTRP.Log2.Fold.Change > 1 | DMSO.vs..4hrTRP.Log2.Fold.Change < -1)) + 
  geom_point(alpha = 0.3, stroke = NA) + 
  geom_vline(xintercept = -1) + geom_vline(xintercept = 1) + 
  labs(title = "Adj P-value vs Log2FC of H3K27ac after 4hr TRP in HeLa-S3",
       subtitle = "HelaS3, quantified at top 50% H3K27ac peaks",
        x = "DMSO.vs.4hr.TRP.Log2FC", 
       y = "-Log(adj.p.value)") + 
  facet_wrap(~ Annotation) + 
  theme(legend.position = "none") + 
  scale_color_manual(values = c("grey", "blue4"))
```

## Quantile peaks

``` r
quantiling_peaks <- function(counts_peaks, ., avg_tags, nbin) {
  # Add column with average tags across all samples per peak
  counts_peaks <- counts_peaks %>% mutate(avg_tags = select(., ends_with("Tag")) %>% rowMeans())
  # if you want to select average of a certain sample/samples uncomment below:
  # counts_peaks <- counts_peaks %>% mutate(avg_cond = select(., contains("condition")) %>% rowMeans())
  # Sort by descending avg_tags with dplyr arrange()
  counts_peaks <- arrange(counts_peaks, desc(avg_tags))
  # Split into bins with dplyr ntile()
  counts_peaks <- counts_peaks %>% mutate(quantile = ntile(avg_tags, nbin))
}
```

Example Usage:

``` r
counts_p300_peaks <- read.delim("~/Research/YC_ChIP_16/counts_p300_peaks_hg38_YCChIP16.txt", header = T, sep = "\t")
```

``` r
counts_p300_peaks <- process_counts_annotpeaks(counts_p300_peaks)
counts_p300_peaks_table <- clean_annotation_peakfile(counts_p300_peaks)
counts_p300_peaks <- quantiling_peaks(counts_p300_peaks, nbin = 10)
# leaving out column with super long entries
knitr::kable(head(counts_p300_peaks %>% select(!"Focus.Ratio.Region.Size")))
```

Format: counts_p300_peaks_binned \<- quantiling_peaks(counts_p300_peaks,
10)

### Extract set of peaks you want by bin number

``` r
extract_peak_bin <- function(counts_peaks_binned, quantile_num) {
  # make empty vector length of # of quantiles
  quantile_files <- vector("numeric")
  length(quantile_files) <- quantile_num
  # print quantile_files to test
  
  #loop
for (i in 1:length(quantile_files)) {
  
  counts_peaks_binned_i <- counts_peaks_binned %>% 
    filter(quantile == i)
  write.table(counts_peaks_binned_i, 
              file = paste("~/Research/", deparse(substitute(counts_peaks_binned)) , i, ".txt", sep = ""), 
              quote = FALSE, row.names = FALSE, sep = "\t")
  counts_peaks_bin_q <- read.delim(
    paste("~/Research/", deparse(substitute(counts_peaks_binned)), i, ".txt", sep = ""), 
    header = T, sep = "\t")
  head(counts_peaks_bin_q)
}
  
}
```

Usage:

``` r
extract_peak_bin(counts_p300_peaks, 10)
```

Check for files:

``` r
list.files(path = "~/Research/", pattern = "counts_p300_peaks")
```

    character(0)

### plot max values of histogram

``` r
hist_q10peaks_p300_covlong_sep %>% 
  group_by(id) %>% 
  filter(Coverage == max(Coverage, na.rm=TRUE)) %>% ggplot(aes(x = as.numeric(amount), y = Coverage, color = amount)) +
  geom_point(size = 5, alpha = 0.8) +
  scale_color_manual(
    values = colorRampPalette(brewer.pal(9, "PuRd"))(6)[3:6],
    name = "Antibody", 
    labels = c("0.5ug", "1ug", "3ug", "5ug")) + 
  theme_classic() + 
  labs(title = "p300 Maximum at Q10 Peaks", 
       x = "p300 Antibody Amount") + 
   theme(legend.position = c(0.82, 0.8), 
                          legend.background = element_rect(
                                  size=0.7, linetype="solid", 
                                  colour ="grey20"))
```

### Calculate Signal at Peaks: Area under Histogram Curve

need DescTools

``` r
samples <- unique(hist_K9ac_allsamples_LP78_sepIP$id)
colnames(hist_K9ac_allsamples_LP78)[1] <- "Distance_from_center"
x <- hist_K9ac_allsamples_LP78$Distance_from_center
AUC_peaks <- matrix(data = "", nrow = length(samples), ncol = 1)
AUC_peaks <- data.frame(AUC_peaks, row.names = samples)

for (i in 1:length(samples)) {

  y <- hist_K9ac_allsamples_LP78_sepIP %>% 
    filter(id == samples[i]) %>%
    select(Coverage)
  y <- pull(y, Coverage)

AUC_peaks[i, ] <- AUC(x, y, method = c("trapezoid"))

}
```

### Minmax normalize data (min to 0, max to 1)

``` r
scale_AUC_minmaxnorm <- function(AUC_peaks) {
  AUC_peaks$AUC_peaks <- as.numeric(AUC_peaks$AUC_peaks)
  
  interphase <- rep(c(0, 25, 50, 75, 95, 100), each = 3)
  AUC_peaks$interphase <- as.numeric(interphase)
  
  # Min max normalization: Scale points from 0-1
  avg_0inter <- mean(c(AUC_peaks[1,1], AUC_peaks[2,1], AUC_peaks[3,1]))
  avg_25inter <- mean(c(AUC_peaks[4,1], AUC_peaks[5,1], AUC_peaks[6,1]))
  avg_50inter <- mean(c(AUC_peaks[7,1], AUC_peaks[8,1], AUC_peaks[9,1]))
  avg_75inter <- mean(c(AUC_peaks[10,1], AUC_peaks[11,1], AUC_peaks[12,1]))
  avg_95inter <- mean(c(AUC_peaks[13,1], AUC_peaks[14,1], AUC_peaks[15,1]))
  avg_100inter <- mean(c(AUC_peaks[16,1], AUC_peaks[17,1], AUC_peaks[18,1]))
  
  AUC_peaks_minmaxnorm <- AUC_peaks %>%
    mutate(minmaxnorm = (AUC_peaks-avg_0inter)/(avg_100inter-avg_0inter) )
}
```

Usage:

``` r
AUC_peaks_readnorm_minmaxnorm <- scale_AUC_minmaxnorm(AUC_peaks)
```

### Minmax normalization, set min/max to variable range

Example sets min to 1, max to 3

``` r
scale_AUC_scalenorm <- function(AUC_peaks) {
  AUC_peaks$AUC_peaks <- as.numeric(AUC_peaks$AUC_peaks)
  
  interphase <- rep(c(0, 25, 50, 75, 95, 100), each = 3)
  AUC_peaks$interphase <- as.numeric(interphase)
  
  # Minmax normalization: Scale points from 0-1
  avg_0inter <- mean(c(AUC_peaks[1,1], AUC_peaks[2,1], AUC_peaks[3,1]))
  avg_25inter <- mean(c(AUC_peaks[4,1], AUC_peaks[5,1], AUC_peaks[6,1]))
  avg_50inter <- mean(c(AUC_peaks[7,1], AUC_peaks[8,1], AUC_peaks[9,1]))
  avg_75inter <- mean(c(AUC_peaks[10,1], AUC_peaks[11,1], AUC_peaks[12,1]))
  avg_95inter <- mean(c(AUC_peaks[13,1], AUC_peaks[14,1], AUC_peaks[15,1]))
  avg_100inter <- mean(c(AUC_peaks[16,1], AUC_peaks[17,1], AUC_peaks[18,1]))
  
AUC_peaks <- AUC_peaks %>%
  mutate(minmaxnorm_base1 = 1+ ((AUC_peaks-avg_100inter )*3)/(avg_0inter-avg_100inter))

}
```

### Calculated Rsquared

``` r
get_Rsquared <- function(AUC_peaks) {
  AUC_peaks$expected <- rep(c(0, 0.25, 0.5, 0.75, 0.95, 1), each = 3)
  
  AUC_peaks$error <- AUC_peaks$minmaxnorm - AUC_peaks$expected
  
  mse = sum(((AUC_peaks$error)^2)/3)
  
  SSres = sum((AUC_peaks$error)^2)
  meanobserved <- mean(AUC_peaks$minmaxnorm)
  SStotal = sum((AUC_peaks$minmaxnorm-meanobserved)^2)
  rsquared = 1 - (SSres/SStotal)
  rsquared
}
```

Usage

``` r
get_Rsquared(AUC_peaks_readnorm_minmaxnorm)
```

# split sample names with regex

## regex for my sample naming conventions

My ChIP samples are labeled with the following format in the sample
sheet before sequencing:

\[experiment##\]\[sampleID\]\_\[celltype\]\_\[timepoint\]\_\[treatment\]\_\[antibody\]\_\[rep\]

Example: LP_69

We want to breakdown the ID column into components, for example the
sample in row 1:

- `69a_` stays in `id`
- `Hela` goes to `cell`
- `0` goes to `timepoint`
- `TRP` goes to `treatment`
- `H3K27ac` goes to `antibody`
- `1` goes to `replicate`

``` r
# when debugging, add too_few = "debug" as last argument in separate_wider_regex()

seqstats_LP69_IPsep <- seqstats_LP69[grep("TRP", seqstats_LP69$ID), ] %>% separate_wider_regex(cols = ID, patterns = c(
  id = "69[:lower:]",
  "\\_", 
  cell = "[:alpha:]+", 
  "\\_", 
  timepoint = "[:digit:]", 
  "[:lower:]+\\_", 
  treatment = "[:upper:]+", 
  "\\_", 
  antibody = ".+", 
  "\\_", 
  replicate = ".+$"))

head(seqstats_LP69_IPsep)
```

Fix the inputs: format `Hela_0hr_input`

``` r
# note the "input" designation is set to antibody column for easier comparison later
seqstats_LP69_inputsep <- seqstats_LP69[grep("input", seqstats_LP69$ID), ] %>% separate_wider_regex(cols = ID, patterns = c(cell = "[:alpha:]+", 
  "\\_", 
  timepoint = "[:digit:]", 
  "[:lower:]+\\_", 
  antibody = "[:lower:]+$"))
head(seqstats_LP69_inputsep)
```

Stitch IP and input dataframes together:

``` r
seqstats_LP69_expanded <- bind_rows(seqstats_LP69_IPsep, seqstats_LP69_inputsep)

knitr::kable(head(seqstats_LP69_expanded))
```

\#Apply function to groups within dataframe

``` r
# function to apply to each author: 
get_input_variation <- function(df, .df) {
   dfnew <- df                           # copy input dataframe
   mininput <- min(df$tagdir.spike_target, na.rm = T) # get min from spike_targ
   var <- as.numeric((df$tagdir.spike_target)/mininput) # make var column
   dfnew %>%
     mutate(var = var)                          # add to df
}
```

``` r
get_input_mean <- function(df, .df) {
   dfnew <- df                                # copy input dataframe
   df$var <- as.numeric(df$var)
   mean <- as.numeric(mean(df$var, na.rm = TRUE)) # make var column
   dfnew %>%
     mutate(mean = mean)                          # add to df
}
```

``` r
# grouping df 
input_var_stats <- alignstats %>% 
  filter(Mark == "input") %>%      # only inputs
  select(2,5:10, 13) %>%           # remove extraneous data
  group_by(Author) %>%             # need to group by paper
  group_map(get_input_variation, .keep = TRUE) %>% # keep group variable
  bind_rows()
```

``` r
input_var_stats_mult <- input_var_stats_mult %>%
  group_by(Author) %>%
 group_map(get_input_mean, .keep = TRUE) %>%
  bind_rows()
```

I used to use `cur_group_id` by itself to get a unique group identifier,
but this ranks the groups in alphabetical order. Instead, we want to
rank by increasing input variation (meanvar), then order from 1-X.

First need to `arrange` by increasing meanvar, then `group_by` Author
(set as factor with levels in order of groups after arrange function),
then set group id to `cur_group_id`

``` r
input_var_stats_mult <- input_var_stats_mult %>%
  arrange(meanvar) %>%
  group_by(Author = factor(Author, levels = unique(Author))) %>%
  mutate(grp.id = cur_group_id())
```

# workflow for all p300/H3K9ac binned histograms

``` r
extract_peak_bin(counts_p300_peaks, 10)
```

``` r
library(RColorBrewer)
quantile_files <- vector("numeric")
length(quantile_files) <- 10
  
files <- list.files(path = "~/Research/YC_ChIP_16/", pattern = "peaks_hg38_H3K9ac")

files_names <- str_remove(files, pattern = ".txt")

for (i in 1:length(quantile_files)) {
  if (!paste0("~/Research/YC_ChIP_16/", "hist_q", i, "peaks_hg38_H3K9ac", ".txt") %in% files) {
   next()
  }
  
  in_file <- paste("~/Research/YC_ChIP_16/", "hist_q", i, "peaks_hg38_H3K9ac", ".txt")
  
  out_plot <- paste("~/Research/YC_ChIP_16/", "hist_q", i, "peaks_hg38_H3K9ac",'_plot.png', sep="")
  
  screen_plate <- read.delim(in_file)
  
  colnames(screen_plate)[1] <- "Distance_from_tss"

  plate_clean <- screen_plate %>% 
    rename_with(~ gsub(".tagdir", "", .x), contains("tagdir")) %>% 
    rename_with(~ gsub("ChIP[[:digit:]]+\\.[[:digit:]]+\\_[[:alpha:]]+\\.[[:alpha:]][[:digit:]]\\_", "", .x), contains("HeLa")) %>%
    rename_with(~ gsub("\\.[[:digit:]]$", "_minus", .x), contains("Tags")) %>% 
    rename_with(~ gsub("\\.\\.\\.", "_", .x), contains("Tags"))

  plate_cov <- plate_clean %>% select(contains("Coverage"))
    
  plate_cov$Distance_from_tss <- plate_clean$Distance_from_tss
  
  plate_cov_long <- 
    plate_cov %>% pivot_longer(
      cols = -"Distance_from_tss", 
      names_to = "Sample", 
      values_to = "Coverage")
  
  plate_cov_long %>% 
    ggplot(
      aes(x = Distance_from_tss, y = Coverage, group = as.factor(id), color = as.factor(id))) +
    geom_line(alpha = 0.9, linewidth = 1.3) +
    scale_color_manual(
      values = colorRampPalette(brewer.pal(9, "YlGnBu"))(12)[4:11],
      name = "Antibody (ug)", 
      labels = c("0.04_rep1", "0.04_rep2", "0.2_rep1", "0.2_rep2", "0.4_rep1", "0.4_rep2", "1_rep1", "1_rep2")) + 
    theme_classic() + theme(legend.position = c(0.9, 0.72), 
                            legend.background = element_rect(
                                    size=0.7, linetype="solid", 
                                    colour ="grey20"))
  
  ggsave(out_plot, width=6, height=5)
                            
}
```

# Spike-in normalization

## histogram normalization

``` r
hist_K9ac_allsamples_LP78_tidyIP <- 
  hist_K9ac_allsamples_LP78_tidy %>% 
  filter(grepl("H3K9ac", Sample))

# copy the hist_tss_hg38_LH58_cov dataframe 
peakcov_fly_ip_input_norm <- hist_K9ac_allsamples_LP78_tidyIP

sampleID <- unique(hist_K9ac_allsamples_LP78_tidyIP$Sample)

seqstatID <- sub('.Coverage', "", sampleID )

# dataframe 1: hist_K9ac_allsamples_LP78_tidy (original read-normalized data) 
# dataframe 2: H3K9ac_mitotic_titration_seqtatsIP
# dataframe 3: peakcov_avg_ip_input_norm (output df)

# When Sample rows of df1 match Sample.ID in df2, multiply Coverage column in df1 by factor in df2, assign to df3

for (i in 1:nrow(hist_K9ac_allsamples_LP78_tidyIP)) {
  # check sample and normfactor match
  if (!hist_K9ac_allsamples_LP78_tidyIP[i, 2] %in% paste0(H3K9ac_mitotic_titration_seqtatsIP$Sample.ID, ".Coverage")) {
   print(paste("warning: No matching normalization factor found for sample", hist_K9ac_allsamples_LP78_tidyIP[i, 2]))

    next()
  }
  
  # make get current sampleID, remove .Coverage 
 seqstatIDi <- sub('.Coverage', "", hist_K9ac_allsamples_LP78_tidyIP[i, 2] )
 
 # get normalization factor from sequencing stats (df3)
 # Col 14 in seqstats dataframe contains fly normalization factor
 
 normfactori <- H3K9ac_mitotic_titration_seqtatsIP[grep(seqstatIDi, H3K9ac_mitotic_titration_seqtatsIP$Sample.ID), 14]
 
 # multiply read_norm coverage by norm factor, assign to new df
peakcov_fly_ip_input_norm[i, 3] <- 
  hist_K9ac_allsamples_LP78_tidyIP[i, 3]/(normfactori)
  
}
```

## peak file normalization

``` r
#### dual normalization at tss: for H3K4me3 and H3K27ac samples only!
counts_tss_hg38_samples <- counts_tss_hg38 %>% 
  select(contains(".Tag"))

counts_tss_hg38_dual <- counts_tss_hg38_samples

# dataframe 1: original read-normalized data
# dataframe 2: seq stats
# dataframe 3: peakcov_avg_ip_input_norm (output df)

# When Sample rows of df1 match Sample.ID in df2, multiply Coverage column in df1 by factor in df2, assign to df3
sampleID <- sub('.Tag', '', colnames(counts_tss_hg38_samples))

for (i in 1:ncol(counts_tss_hg38_samples)) {
  
  # make get current sampleID, remove .Coverage 
 sampleIDi <- sampleID[i]
 
 # get normalization factor from sequencing stats (df3)
 # Col 23 in seqstats df3 contains yeast normalization factor
 
 normfactori <- LP88_LP91_seqstats[grep(sampleIDi, LP88_LP91_seqstats$ID), 'avg.normfactor.adj']
 # multiply read_norm coverage by norm factor, assign to new df
counts_tss_hg38_dual[, i] <- 
  counts_tss_hg38_samples[, i]/(normfactori)
  
}

counts_tss_hg38_dual$quantile <- counts_tss_hg38$quantile
#### fly normalization at tss: for Rpb1 samples only!
counts_tss_hg38_fly <- counts_tss_hg38_samples

# dataframe 1: original read-normalized data
# dataframe 2: seq stats
# dataframe 3: peakcov_avg_ip_input_norm (output df)

# When Sample rows of df1 match Sample.ID in df2, multiply Coverage column in df1 by factor in df2, assign to df3
sampleID <- sub('_2_Rpb1_1.comb.Tag', '_2_Rpb1_2', colnames(counts_tss_hg38_samples))
sampleID <- sub('.Tag', '', sampleID)

for (i in 1:ncol(counts_tss_hg38_samples)) {
  
  # make get current sampleID, remove .Coverage 
 sampleIDi <- sampleID[i]
 
 # get normalization factor from sequencing stats (df3)
 # Col 23 in seqstats df3 contains yeast normalization factor
 
 normfactori <- LP88_LP91_seqstats[grep(sampleIDi, LP88_LP91_seqstats$ID), 'dm6.normfactor.adj']
 
 # multiply read_norm coverage by norm factor, assign to new df
counts_tss_hg38_fly[, i] <- 
  counts_tss_hg38_samples[, i]/(normfactori)
  
}

counts_tss_hg38_fly$quantile <- counts_tss_hg38$quantile
```

# csRNAseq processing

## Tag Length Distributions (using example from chip)

Step 1: all samples in one df

``` r
# get vector of sample tagdir names
files <- list.files(path = "/Users/laurenpatel/Documents/Research/LP_81/LP_81/dm6_tagdirs", pattern = "dm6.GCcheck-tagdir")

# initialize output matrix (proper row/col numbers)
taglengthdist <- data.frame(matrix(NA, 
                                   nrow = 73, ncol= (length(files) + 1)))


for (i in 1:length(files)) {
  
  # get name of input file, paste0 uses no spaces
  in_file <- paste0("/Users/laurenpatel/Documents/Research/LP_81/LP_81/dm6_tagdirs/", 
                    files[i], "/tagLengthDistribution.txt")

  # read each table
  sample_lengthdist_i <- read.table(in_file, sep="\t", header = T)
  
  colnames(sample_lengthdist_i)[1] <- "Tag.Length"
  
  # first column is all tag lengths, add one to the output df
  taglengthdist[,1] <- sample_lengthdist_i[,1]
  colnames(taglengthdist)[1] <- "Tag.Length"
  
  # add fraction of tags column from each sample to output df
  taglengthdist[,i + 1] <- sample_lengthdist_i["Fraction.of.Tags"]
  
  # give each fraction of tags column a name from the sample
  # str_remove will remove the extra suffixes from tagdir names in files list
  colnames(taglengthdist)[i + 1] <- str_remove(files[i], pattern = ".GCcheck-tagdir")
  
}
```

``` r
 # need to also remove prefix numbers, will mess up ggplot
names(taglengthdist) <- gsub("^[[:digit:]]+\\_", "", names(taglengthdist))

names(taglengthdist) <- gsub("-", "", names(taglengthdist))
```

Step 2: plotting function

``` r
plot_taglength <- function(taglengthdist, Tag.Length, input, sample, Tag.Dist, Sample) {
    taglengthdistfilt <- taglengthdist %>% 
      select(Tag.Length, input, sample) %>%
      pivot_longer(
        cols = c(input, sample),
        names_to = "Sample", 
        values_to = "Tag.Dist")
    
    ggplot(data = taglengthdistfilt) + 
      aes_string(x = Tag.Length, y = Tag.Dist, color = Sample) + 
      geom_line(linewidth = 1.1, alpha = 0.8) +
      scale_color_manual(values = c("mediumorchid3", "slateblue"), 
                         labels = c("input", "ChIP")) +
      labs(y = "Distribution of Tags") 
  }
```

NOTE: when a function contains an input variable that will be called by
ggplot (for example, an x or y value) you have to use `aes_string()`
instead of `aes()`. Then in the function, the input arguments need to be
in string format (in quotes).

Use:

``` r
plot_taglength(taglengthdist, "Tag.Length", "Sample", "Sample", "Tag.Dist", "Sample") + labs(title = "Sample ChIP")
```

## Base Frequency Distributions

``` r
# get vector of sample tagdir names
files <- list.files(path = "/Users/laurenpatel/Documents/Research/LP_81/LP_81/dm6_tagdirs", pattern = "dm6.GCcheck.GCcheck-tagdir")

# initialize output matrix (proper row/col numbers)
basedist <- data.frame(matrix(NA, 
                                   nrow = 1000, ncol= (length(files) + 2)))


for (i in 1:length(files)) {
  
  # get name of input file, paste0 uses no spaces
  in_file <- paste0("/Users/laurenpatel/Documents/Research/LP_81/LP_81/dm6_tagdirs/", files[i], "/tagFreqUniq.txt")

  # read each table
  sample_basedist_i <- read.table(in_file, sep="\t", header = T)
  
  colnames(sample_basedist_i)[1] <- "Offset"
  
  # put each sample basedist in tidy format (1 col each)
  sample_basedist_tidyi <- sample_basedist_i %>% 
    pivot_longer(cols = c("A", "C", "G", "T"), 
                 names_to = "Base", 
                 values_to = "Frequency")
  
  # first column is all tag lengths, add one to the output df
  basedist[,1] <- sample_basedist_tidyi[,1]
  colnames(basedist)[1] <- "Offset"
  
  # add one base column to output df (same in every sample df)
  basedist[,2] <- sample_basedist_tidyi["Base"]
  colnames(basedist)[2] <- "Base"
  
  # add Frequency column from each sample to output df
  basedist[,i + 2] <- sample_basedist_tidyi["Frequency"]
  
  # give each fraction of tags column a name from the sample
  # str_remove will remove the extra suffixes from tagdir names in files list
  colnames(basedist)[i + 2] <- str_remove(files[i], pattern = ".GCcheck-tagdir")
  
}
```

``` r
 # need to also remove prefix numbers, will mess up ggplot
names(basedist) <- gsub("^[[:digit:]]+\\_", "", names(basedist))

names(basedist) <- gsub("-", "", names(basedist))
```

``` r
plot_basedist <- function(basedist, Offset, Base, Sample) {
  ggplot(data = basedist) + 
    aes_string(x = Offset, y = Sample, color = Base, group = Base) +
    geom_line(linewidth = 1.1, alpha = 0.8) +
    scale_color_manual(values = c("firebrick", "goldenrod", "slateblue", "mediumorchid3"), 
                       labels = c("A", "C", "G", "T")) +
    labs(y = "Nucleotide Frequency", x = "Distance from TSS") +
    theme(legend.position = c(0.85, 0.75))
}
```

Use:

``` r
plot_basedist(basedist, "Offset", "Base", "HeLaS3_Interphase2uMTSA_S17") + 
  labs(title = "HeLaS3_Interphase_2uM_TSA")
```
