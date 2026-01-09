# Dataframe Manipulation in R


- [Quarto formatting:](#quarto-formatting)
- [Intro Steps](#intro-steps)
  - [Load Libraries and Presets](#load-libraries-and-presets)
    - [Theme for ggplot](#theme-for-ggplot)
  - [Some general notes for this
    document:](#some-general-notes-for-this-document)
- [Data Import](#data-import)
  - [From excel (try to avoid)](#from-excel-try-to-avoid)
  - [Import from TSV, CSV, etc](#import-from-tsv-csv-etc)
- [Getting Basic Information](#getting-basic-information)
  - [Basic operations:](#basic-operations)
  - [Cleanup column names in Kable with
    `gsub()`](#cleanup-column-names-in-kable-with-gsub)
  - [Variable classes](#variable-classes)
- [Searching](#searching)
  - [`grep()` vs `grepl()`](#grep-vs-grepl)
  - [Search and return row of
    dataframe](#search-and-return-row-of-dataframe)
- [Boolean Logic](#boolean-logic)
- [Regex in R](#regex-in-r)
  - [Example with my sequencing data](#example-with-my-sequencing-data)
  - [rename columns](#rename-columns)
  - [with select()](#with-select)
- [Combine dataframes](#combine-dataframes)
  - [Base R `merge()`](#base-r-merge)
  - [`bind_rows()`](#bind_rows)
- [Changing Names with `rename_with()` and
  `gsub()`](#changing-names-with-rename_with-and-gsub)
  - [Change column names](#change-column-names)
  - [Remove All Numbers from Column
    Names](#remove-all-numbers-from-column-names)
    - [Rename Column by Order](#rename-column-by-order)
  - [Change data in column](#change-data-in-column)
- [Actions: subsampling rows, set NAs to 0,
  etc.](#actions-subsampling-rows-set-nas-to-0-etc)
  - [random downsampling of data](#random-downsampling-of-data)
  - [Cleanup data - set NAs to 0](#cleanup-data---set-nas-to-0)
- [Tidyverse basics](#tidyverse-basics)
  - [`mutate()` can create new column based on
    others](#mutate-can-create-new-column-based-on-others)
  - [select columns](#select-columns)
  - [paring mutate and select](#paring-mutate-and-select)
  - [Pair Data](#pair-data)
  - [Long Format (TidyR format) with
    `pivot_longer()`](#long-format-tidyr-format-with-pivot_longer)
  - [Reorder column contents](#reorder-column-contents)
  - [Apply function to groups within
    dataframe](#apply-function-to-groups-within-dataframe)
    - [Function to calculate variation in inputs within each
      paper](#function-to-calculate-variation-in-inputs-within-each-paper)
- [Create data](#create-data)
  - [Empty vector](#empty-vector)
  - [Column filled with basic math](#column-filled-with-basic-math)
  - [rowMeans()](#rowmeans)
  - [Column filled using patterns](#column-filled-using-patterns)
  - [Add column to dataframe](#add-column-to-dataframe)
  - [Add column from dataframe (containing \> 1 column) to different
    dataframe](#add-column-from-dataframe-containing--1-column-to-different-dataframe)
  - [Make Dataframe from vectors](#make-dataframe-from-vectors)
  - [Add column to number groups](#add-column-to-number-groups)
- [Special cases](#special-cases)
  - [Convert Time Series to Dataframe with `as.matrix()` and
    `data.frame()`](#convert-time-series-to-dataframe-with-asmatrix-and-dataframe)
  - [Split verbose column into multiple with
    `separate_wider_regex()`](#split-verbose-column-into-multiple-with-separate_wider_regex)

# Quarto formatting:

If you are new to R, disregard this section. It’s purpose is only to
show the code necessary to make your own quarto document like this one.

| Input                                  | Output                  |
|----------------------------------------|:------------------------|
| YAML header line `code-overflow: wrap` | Long code overflows     |
| YAML header line `toc: true`           | Show table of contents  |
| `#| eval: false` code block top line   | Display code, don’t run |
| `>` at the beginning of a line         | Indents text            |
| \`display code here\`                  | `display code here`     |
| What is \` r 2 + 2\`                   | What is 4               |

# Intro Steps

## Load Libraries and Presets

If you are brand new to R, you will need to first install the libraries
shown with `install.packages("libraryname")`. However, downloading only
happens ONCE, and you shouldn’t put the install.packages line of code in
a doc like this that will be rerun, so I didn’t include it here :). Once
you have packages installed, you need to “load” them every time you open
R with `library(libraryname)` as I have done below.

``` r
library(readxl)    # used to load excel files in R
library(tidyverse) # see tidyverse tutorial, powerful set of tools for data maniputation
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

``` r
library(gapminder) # public dataset used for examples
library(viridis)   # library of colorblind-friendly palettes
```

    Loading required package: viridisLite

### Theme for ggplot

``` r
theme_set(theme_classic()) # feel free to not use this theme if you find a different one you prefer
```

## Some general notes for this document:

1.  if you see `%>%` in code, this is the pipe operator used in
    tidyverse. In the newest version of R (after 4.0) you can also use
    the standard pipe `|`
2.  Many examples will be either my own or public data commonly used in
    R tutorials. One example of public data is the gapminder dataset
    which is loaded above with `library(gapminder)`. It contains
    demographic/economic/health information on countries from 1952-2007.
3.  My own data is used when its a good example for
    bioinformatic-specific tasks, OR when it’s a good example of
    “human-annotated” data being difficult to feed into programs. There
    are lots of issues I learned hard way (for example, ggplot does not
    work properly if your variables for x/y start with a number, and my
    sample IDs usually start with an experiment number, go figure)
4.  I tried to load all relevant example data in the *Data Import*
    section. Because not everyone has these files, I also run commands
    to visualize their structure (what their rows/columns are, etc) when
    they are loaded. I also try to show before/after examples to show
    what each line of code does when possible, or at least a verbal
    explanation.

Commands useful to “visualize” (actually see) your raw data are
invaluable. You will see lots of them below but here are my most used:

For a given “dataset”:

1.  `View(dataset)` will automatically open the dataset in a new tab. If
    you run this line of code in a quarto doc, a popup window with the
    dataset will open while it executes. A cool feature is that if you
    run code to change a column or row name, it will update
    automatically if the code worked :)
2.  `head(dataset)` will output the first few lines of your dataset
    inside your document/console/wherever you run the code. The default
    for head() is 6 rows, but you can change this with
    `head(dataset, n = 10)` to make it 10 rows, etc. Also `tail()` does
    the same thing with the end of your dataset.
3.  If you are working within a Quarto doc like this one,
    `knitr::kable(dataset)` displays the table in a much more
    reader-friendly format than head(). You will see this shortly below.
    You can also do more fancy things like captions, etc.

# Data Import

## From excel (try to avoid)

I keep a master spreadsheet with sequencing statistics for all of my NGS
experiments:

``` r
seqstats_excel <- read_excel("Sequencing_Stats_260107.xlsx", 
    sheet = "all_dualspike_metadata")
```

    New names:
    • `` -> `...7`

``` r
knitr::kable(head(seqstats_excel))
```

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.2326846 | 0.0932468 | 0.4007435 | 8.252629 | 50.57160 | 0.9420306 | 0.9264793 | 22496.56 | 165939.1 | 6720.68 | 0.9951239 | 0.9883963 | 0.9991668547 | 0.9310996 | 0.9257074 | 0.9284035 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.2273306 | 0.0899316 | 0.3955981 | 7.959218 | 49.92229 | 0.9085379 | 0.9145838 | 22511.03 | 166041.9 | 6735.15 | 0.9957404 | 0.9905244 | 0.9997858429 | 0.8999290 | 0.9143879 | 0.9071584 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.2269665 | 0.1137764 | 0.5012917 | 10.069558 | 63.26023 | 1.1494315 | 1.1589369 | 22718.79 | 166251.4 | 6942.91 | 0.9969967 | 1.0210792 | 1.001047302 | 1.1736607 | 1.1601506 | 1.1669056 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.2037063 | 0.0532022 | 0.2611710 | 4.963025 | 34.02945 | 0.5665250 | 0.6234247 | 22547.08 | 165787.9 | 6860.25 | 0.9915105 | 1.0089226 | 0.9955388152 | 0.5715799 | 0.6206435 | 0.5961117 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.2208321 | 0.0455002 | 0.2060397 | 4.244537 | 26.84609 | 0.4845103 | 0.4918244 | 22465.16 | 165797.5 | 6778.33 | 0.9915680 | 0.9968748 | 0.9955964622 | 0.4829961 | 0.4896586 | 0.4863274 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.2000708 | 0.0531811 | 0.2658114 | 4.961059 | 34.63408 | 0.5663006 | 0.6345015 | 22570.42 | 165843.8 | 6883.59 | 0.9918449 | 1.0123552 | 0.9958744888 | 0.5732973 | 0.6318839 | 0.6025906 |

## Import from TSV, CSV, etc

Use `read.table()` function: first argument specify path to file, sep
argument tells R what separates the columns of your data (space, comma,
or tab). Example below is for tab separated TSV. header = T (header =
True) means the first row is the headers for columns.

``` r
TRP_input_tagLen <- read.table("tagLengthDistribution.txt", sep="\t", header = T)

knitr::kable(head(TRP_input_tagLen, n = 10))
```

| Tag.Length..Average.tag.length…70.748090. | Fraction.of.Tags |
|------------------------------------------:|-----------------:|
|                                         0 |         0.000000 |
|                                         1 |         0.000000 |
|                                         2 |         0.000057 |
|                                         3 |         0.000019 |
|                                         4 |         0.000248 |
|                                         5 |         0.000096 |
|                                         6 |         0.000096 |
|                                         7 |         0.000172 |
|                                         8 |         0.000287 |
|                                         9 |         0.000248 |

If you use read.delim it automatically defaults `sep = "\t"`, perfect
for tsv files.

``` r
seqstats <- read.delim("all_dualspike_metadata.tsv", sep = "\t")
knitr::kable(head(seqstats))
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

# Getting Basic Information

## Basic operations:

Isolating a column by name: `seqstats$cell`

If there are spaces in the name, use “\`”, as in

``` r
seqstats_excel$`experiment ID` %>% head()
```

    [1] "LP78" "LP78" "LP78" "LP78" "LP78" "LP78"

If you use `read.delim()`, spaces are converted to periods “.” so

``` r
seqstats$experiment.ID %>% head()
```

    [1] "LP78" "LP78" "LP78" "LP78" "LP78" "LP78"

Print column names: `names(seqstats)` or `colnames(seqstats)`

Isolate column by position: `seqstats[1]`

``` r
seqstats[1] %>% head()
```

      experiment.ID
    1          LP78
    2          LP78
    3          LP78
    4          LP78
    5          LP78
    6          LP78

## Cleanup column names in Kable with `gsub()`

Usually dataframe column names have \_ or . in their name for ease of
coding, but you might want to display the names in a more human-readable
way in the kable:

Example below replaces underscores in column names with spaces only for
knitr::kable() to display, WITHOUT modifying the original dataset.

``` r
knitr::kable(head(seqstats), col.names = gsub("[.]", " ", names(seqstats)))
```

| experiment ID | library ID | cell | IP | biorep | techrep | X | bwa hg38 q 50 | bwa dm6 q 50 | bwa sac3 q 50 | dm6 sac3 | dm6 hg38 | sac3 hg38 | dm6 hg38 IP input | sac3 hg38 IP input | dm6 hg38 IP input control norm | sac3 hg38 IP input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP input | sac3 eff IP input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

## Variable classes

``` r
str(seqstats)
```

    'data.frame':   199 obs. of  26 variables:
     $ experiment.ID                  : chr  "LP78" "LP78" "LP78" "LP78" ...
     $ library.ID                     : chr  "HelaS3_100sync_0inter_1_H3K9ac_1" "HelaS3_100sync_0inter_1_H3K9ac_2" "HelaS3_100sync_0inter_1_H3K9ac_3" "HelaS3_75sync_25inter_1_H3K9ac_1" ...
     $ cell                           : chr  "HeLaS3" "HeLaS3" "HeLaS3" "HeLaS3" ...
     $ IP                             : chr  "H3K9ac" "H3K9ac" "H3K9ac" "H3K9ac" ...
     $ biorep                         : int  1 1 1 1 1 1 1 1 1 1 ...
     $ techrep                        : num  1 2 3 1 2 3 1 2 3 1 ...
     $ X                              : logi  NA NA NA NA NA NA ...
     $ bwa.hg38..q.50                 : int  8446795 8000672 7144407 9000686 10985910 8336422 11129385 13553505 9924828 15148990 ...
     $ bwa.dm6..q.50                  : int  787637 719513 812865 478856 499861 443340 416659 509021 349112 418195 ...
     $ bwa.sac3..q.50                 : int  3384998 3165051 3581432 2350718 2263534 2215916 2308097 2532542 1631292 2170994 ...
     $ dm6.sac3                       : num  0.233 0.227 0.227 0.204 0.221 0.2 0.181 0.201 0.214 0.193 ...
     $ dm6.hg38                       : num  0.0932 0.0899 0.1138 0.0532 0.0455 ...
     $ sac3.hg38                      : num  0.401 0.396 0.501 0.261 0.206 ...
     $ dm6.hg38.IP.input              : num  8.25 7.96 10.07 4.96 4.25 ...
     $ sac3.hg38.IP.input             : num  50.6 49.9 63.3 34 26.8 ...
     $ dm6.hg38.IP.input.control.norm : num  0.942 0.908 1.149 0.567 0.484 ...
     $ sac3.hg38.IP.input.control.norm: num  0.926 0.915 1.159 0.623 0.492 ...
     $ dm6.IP.signal                  : num  22497 22511 22719 22547 22465 ...
     $ sac3.IP.signal                 : num  165939 166042 166251 165788 165798 ...
     $ dm6.eff.IP.input               : num  6721 6735 6943 6860 6778 ...
     $ sac3.eff.IP.input              : num  0.995 0.996 0.997 0.992 0.992 0.992 0.989 0.992 0.991 0.989 ...
     $ dm6.eff.control.norm           : num  0.988 0.991 1.021 1.009 0.997 ...
     $ sac3.eff.control.norm          : chr  "0.999" "1.000" "1.001" "0.996" ...
     $ dm6.normfactor.snr.adj         : num  0.931 0.9 1.174 0.572 0.483 ...
     $ sac3.normfactor.snr.adj        : num  0.926 0.914 1.16 0.621 0.49 0.632 0.507 0.458 0.402 0.335 ...
     $ dual.normfactor.snr.adj        : num  0.928 0.907 1.167 0.596 0.486 ...

- fct = categorical variable (factor)
- dbl = dibble
- int = continuous variable (integer)
- num = numeric
- logi = logical variable
- chr = character

Note that categorical variables containing numbers will automatically be
treated as a continuous variable, to force it to be categorical replace
the variable “Year” with “factor(Year)”

# Searching

## `grep()` vs `grepl()`

**`grep()`** is a built-in function that returns indices of matched
items or the items themselves

In the below code we are searching for:

1.  the string “\_1hr”
2.  within the seqstats dataset
3.  within this dataset, the ID column

``` r
grep("_1hr", seqstats$library.ID)
```

     [1]  27  28  33  34  39  40  45  46  51  52  56  58  61  63  66  68  71  73  76
    [20]  78  81  83  86  88  92  93  96  97 101 103 106 108 112 113 116 117 122 123
    [39] 126 127 131 133 136 138 141 143 146 148 152 153 156 157 162 163 166 167 172
    [58] 173 176 177 182 183 186 187 192 193 196 197

**`grepl()`** is very similar, but instead of returining indices of the
matches location, it returns a logical vector, with TRUE representing a
match, and FALSE if not a match

The same example with grepl:

``` r
grepl("_1hr", seqstats$library.ID)
```

      [1] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
     [13] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
     [25] FALSE FALSE  TRUE  TRUE FALSE FALSE FALSE FALSE  TRUE  TRUE FALSE FALSE
     [37] FALSE FALSE  TRUE  TRUE FALSE FALSE FALSE FALSE  TRUE  TRUE FALSE FALSE
     [49] FALSE FALSE  TRUE  TRUE FALSE FALSE FALSE  TRUE FALSE  TRUE FALSE FALSE
     [61]  TRUE FALSE  TRUE FALSE FALSE  TRUE FALSE  TRUE FALSE FALSE  TRUE FALSE
     [73]  TRUE FALSE FALSE  TRUE FALSE  TRUE FALSE FALSE  TRUE FALSE  TRUE FALSE
     [85] FALSE  TRUE FALSE  TRUE FALSE FALSE FALSE  TRUE  TRUE FALSE FALSE  TRUE
     [97]  TRUE FALSE FALSE FALSE  TRUE FALSE  TRUE FALSE FALSE  TRUE FALSE  TRUE
    [109] FALSE FALSE FALSE  TRUE  TRUE FALSE FALSE  TRUE  TRUE FALSE FALSE FALSE
    [121] FALSE  TRUE  TRUE FALSE FALSE  TRUE  TRUE FALSE FALSE FALSE  TRUE FALSE
    [133]  TRUE FALSE FALSE  TRUE FALSE  TRUE FALSE FALSE  TRUE FALSE  TRUE FALSE
    [145] FALSE  TRUE FALSE  TRUE FALSE FALSE FALSE  TRUE  TRUE FALSE FALSE  TRUE
    [157]  TRUE FALSE FALSE FALSE FALSE  TRUE  TRUE FALSE FALSE  TRUE  TRUE FALSE
    [169] FALSE FALSE FALSE  TRUE  TRUE FALSE FALSE  TRUE  TRUE FALSE FALSE FALSE
    [181] FALSE  TRUE  TRUE FALSE FALSE  TRUE  TRUE FALSE FALSE FALSE FALSE  TRUE
    [193]  TRUE FALSE FALSE  TRUE  TRUE FALSE FALSE

## Search and return row of dataframe

Syntax: `dataframe[grep("string", dataframe$column), ]`

Use indexes to return row with match: works with `grep` and `grepl`

``` r
seqstats[grep("0hr", seqstats$library.ID), ] %>% head() %>% knitr::kable()
```

|  | experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| 25 | LP81 | HelaS3_TRP_0hr_1_H3K27ac_DSG-FA | HeLaS3 | H3K27ac | 1 | 1 | NA | 3526473 | 56726 | 10545850 | 0.005 | 0.01609 | 2.99048 | 6.381 | 103.438 | 0.9572 | 0.9348 | 27942.8100 | 181238.2 | 12547.3500 | 181238.2 | 0.981 | 1.001 | 0.939 | 0.935 | 0.937 |
| 26 | LP81 | HelaS3_TRP_0hr_2_H3K27ac_DSG-FA | HeLaS3 | H3K27ac | 2 | 1 | NA | 2823944 | 56233 | 10599682 | 0.005 | 0.01991 | 3.75350 | 6.952 | 117.867 | 1.0428 | 1.0652 | 28434.9200 | 181029.8 | 13043.4600 | 181029.8 | 1.019 | 0.999 | 1.063 | 1.065 | 1.064 |
| 31 | LP81 | HelaS3_TRP_0hr_1_H3K4me1_DSG-FA | HeLaS3 | H3K4me1 | 1 | 1 | NA | 8538628 | 760248 | 5195480 | 0.146 | 0.08904 | 0.60847 | 35.318 | 21.046 | 0.8917 | 0.9382 | 596.9912 | NA | 0.0388 | NA | 1.000 | \#DIV/0! | 0.892 | NA | 0.892 |
| 32 | LP81 | HelaS3_TRP_0hr_2_H3K4me1_DSG-FA | HeLaS3 | H3K4me1 | 2 | 1 | NA | 8862085 | 1114426 | 6722173 | 0.166 | 0.12575 | 0.75853 | 43.901 | 23.819 | 1.1083 | 1.0618 | 596.4940 | NA | 0.0388 | NA | 1.000 | \#DIV/0! | 1.108 | NA | 1.108 |
| 37 | LP81 | HelaS3_TRP_0hr_1_H3K4me3_DSG-FA | HeLaS3 | H3K4me3 | 1 | 1 | NA | 3476232 | 671941 | 10859461 | 0.062 | 0.19330 | 3.12392 | 76.675 | 108.054 | 0.9197 | 0.9402 | 51680.0700 | 190755.6 | 36284.6100 | 190755.6 | 1.002 | 1.000 | 0.922 | 0.940 | 0.931 |
| 38 | LP81 | HelaS3_TRP_0hr_2_H3K4me3_DSG-FA | HeLaS3 | H3K4me3 | 2 | 1 | NA | 3019041 | 778905 | 11710955 | 0.067 | 0.25800 | 3.87903 | 90.068 | 121.809 | 1.0803 | 1.0598 | 51495.3100 | 190729.2 | 36103.8500 | 190729.2 | 0.998 | 1.000 | 1.078 | 1.060 | 1.069 |

# Boolean Logic

- !x = NOT x (everything else)
- x & y = x AND y
- x && y =
- x \| y = x OR y
- x \|\| y =
- xor(x, y) =

Example: get rows whose column `library.ID` contents matched the partial
string `H3K36me3` OR `Rpb1`.

``` r
seqstats[grep("H3K36me3|Rpb1", seqstats$library.ID), ] %>% head(n = 10) %>% knitr::kable()
```

|  | experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| 49 | LP81 | HelaS3_TRP_0hr_1_Rpb1_DSG-FA | HeLaS3 | Rpb1 | 1 | 1 | NA | 11813620 | 12549 | 122904 | 0.102 | 0.00106 | 0.01040 | 0.421 | NA | 0.8991 | NA | 21433.43000 | NA | 6037.97 | NA | 1.055 |  | 0.948 | NA | 0.948 |
| 50 | LP81 | HelaS3_TRP_0hr_2_Rpb1_DSG-FA | HeLaS3 | Rpb1 | 2 | 1 | NA | 5383291 | 7956 | 67369 | 0.118 | 0.00148 | 0.01251 | 0.516 | NA | 1.1009 | NA | 20801.62000 | NA | 5410.16 | NA | 0.945 |  | 1.041 | NA | 1.041 |
| 51 | LP81 | HelaS3_TRP_1hr_1_Rpb1_DSG-FA | HeLaS3 | Rpb1 | 1 | 1 | NA | 5027345 | 18036 | 132006 | 0.137 | 0.00359 | 0.02626 | 1.131 | NA | 2.4126 | NA | 19707.93000 | NA | 4422.62 | NA | 0.773 |  | 1.864 | NA | 1.864 |
| 52 | LP81 | HelaS3_TRP_1hr_2_Rpb1_DSG-FA | HeLaS3 | Rpb1 | 2 | 1 | NA | 4549165 | 10217 | 133292 | 0.077 | 0.00225 | 0.02930 | 0.817 | NA | 1.7425 | NA | 22460.57000 | NA | 6930.54 | NA | 1.211 |  | 2.110 | NA | 2.110 |
| 53 | LP81 | HelaS3_TRP_4hr_1_Rpb1_DSG-FA | HeLaS3 | Rpb1 | 1 | 1 | NA | 5077412 | 15491 | 191187 | 0.081 | 0.00305 | 0.03765 | 0.845 | NA | 1.8032 | NA | 21176.91000 | NA | 5471.54 | NA | 0.956 |  | 1.724 | NA | 1.724 |
| 54 | LP81 | HelaS3_TRP_4hr_2_Rpb1_DSG-FA | HeLaS3 | Rpb1 | 2 | 1 | NA | 2239773 | 14461 | 237312 | 0.061 | 0.00646 | 0.10595 | 1.920 | NA | 4.0974 | NA | 22284.58000 | NA | 6647.42 | NA | 1.161 |  | 4.758 | NA | 4.758 |
| 70 | LP88 | HCT116_DMSO_0hr_1_Rpb1_1 | HCT116-Rpb1 | H3K4me3 | 1 | 1 | NA | 23252966 | 52648 | 2848 | 18.486 | 0.00226 | 0.00012 | 0.320 | 0.048 | 1.0000 | NA | 69.05311 | NA | 1.53 | NA | 1.000 |  | 1.000 | NA | 1.000 |
| 71 | LP88 | HCT116_TRP_1hr_1_Rpb1_1 | HCT116-Rpb1 | H3K4me3 | 1 | 1 | NA | 12225783 | 72691 | 2711 | 26.813 | 0.00595 | 0.00022 | 0.813 | 0.066 | 2.5400 | NA | 64.33036 | NA | 1.42 | NA | 0.928 |  | 2.358 | NA | 2.358 |
| 72 | LP88 | HCT116_TRP_4hr_1_Rpb1_1 | HCT116-Rpb1 | H3K4me3 | 1 | 1 | NA | 8340865 | 147207 | 19465 | 7.563 | 0.01765 | 0.00233 | 2.323 | 0.811 | 7.2567 | NA | 56.79567 | NA | 1.26 | NA | 0.822 |  | 5.962 | NA | 5.962 |
| 73 | LP88 | HCT116_IAA_1hr_1_Rpb1_1 | HCT116-Rpb1 | H3K4me3 | 1 | 1 | NA | 11004948 | 127128 | 2334 | 54.468 | 0.01155 | 0.00021 | 1.263 | 0.067 | 3.9453 | NA | 55.01261 | NA | 1.19 | NA | 0.778 |  | 3.070 | NA | 3.070 |

For AND matches, we need to modify the notation, as
`grep("DMSO&Rpb1", ...)` will only find a match if there is a library ID
that contains the continuous string “DMSO Rpb1” in that exact order.
Also, `grep` will not work, we need `grepl`. We need to do:

``` r
seqstats[grepl("DMSO", seqstats$library.ID) & grepl("Rpb1", seqstats$library.ID), ] %>% 
  head() %>% knitr::kable()
```

|  | experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| 70 | LP88 | HCT116_DMSO_0hr_1_Rpb1_1 | HCT116-Rpb1 | H3K4me3 | 1 | 1 | NA | 23252966 | 52648 | 2848 | 18.486 | 0.00226 | 0.00012 | 0.320 | 0.048 | 1.0000 | NA | 69.05311 | NA | 1.53 | NA | 1.000 |  | 1.0000 | NA | 1.0000 |
| 120 | LP91 | HCT116_DMSO_0hr_2_Rpb1_1 | HCT116-Rpb1 | Rpb1 | 2 | 1 | NA | 8057207 | 8906 | 827 | 10.769 | 0.00111 | 0.00010 | 0.066 | 0.077 | 0.7286 | 0.7579 | 6669.26100 | NA | 1.03 | NA | 1.013 |  | 0.7380 | NA | 0.7380 |
| 121 | LP91 | HCT116_DMSO_0hr_2_Rpb1_2 | HCT116-Rpb1 | Rpb1 | 2 | 2 | NA | 7953774 | 15343 | 1338 | 11.467 | 0.00193 | 0.00017 | 0.116 | 0.127 | 1.2714 | 1.2421 | 6504.36400 | NA | 1.01 | NA | 0.987 |  | 1.2560 | NA | 1.2560 |
| 190 | LP95 | HCT116_DMSO_0hr_3_Rpb1_1 | HCT116-Rpb1 | Rpb1 | 3 | 1 | NA | 4807836 | 67749 | 1088 | 62.269 | 0.01409 | 0.00023 | 1.409 | 0.042 | 0.9294 | 1.3448 | 8170.40000 | NA | 1.28 | NA | 1.068 |  | 0.9923 | NA | 0.9923 |
| 191 | LP95 | HCT116_DMSO_0hr_4_Rpb1_1 | HCT116-Rpb1 | Rpb1 | 4 | 1 | NA | 4852590 | 110428 | 712 | 155.096 | 0.02276 | 0.00015 | 1.623 | 0.021 | 1.0706 | 0.6552 | 7270.40200 | NA | 1.12 | NA | 0.926 |  | 0.9915 | NA | 0.9915 |

# Regex in R

Regex:

- `^` = anchor beginning of string
- `$` = anchor end of the string
- `.` = matches any character but newline
- \[\[:alpha:\]\] = letters
- \[\[:digit:\]\] = digits
- \[\[:alnum:\]\] = letters and numbers
- \[\[:graph:\]\] = any character but not white space/blank

Special options:

- `+` goes after a match, means it can be counted more than once! Useful
  instead of denoting `[:digit:][:digit:]` for a two digit number
- `\` escapes any special character that otherwise has a meaning in
  regex. For example, a period usually means any character, but if
  preceeeded by `\` as in `\.` only a literal “.” will match

More complicated examples:

`\.+\.` will match any string between two periods

## Example with my sequencing data

My ChIP samples are labeled with the following format in the sample
sheet before sequencing:

\[experiment##\]\[sampleID\]\_\[celltype\]\_\[timepoint\]\_\[treatment\]\_\[antibody\]\_\[rep\]

``` r
knitr::kable(head(seqstats))
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

## rename columns

`gsub` is similar to find/replace in most applications.

Below, I am replacing “.q.50” with “MAPQ_50_cutoff”, but ONLY if the
column name also contains the string “bwa”.

``` r
seqstats %>% 
    rename_with(~ gsub(".q.50", "MAPQ_50_cutoff", .x), contains("bwa")) %>% head() %>% knitr::kable()
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38.MAPQ_50_cutoff | bwa.dm6.MAPQ_50_cutoff | bwa.sac3.MAPQ_50_cutoff | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

``` r
seqstats %>% 
  rename_with(~ gsub("cell", "new_cell", .x), contains("cell")) %>% head() %>% knitr::kable()
```

| experiment.ID | library.ID | new_cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

## with select()

``` r
seqstats %>%
  select(matches(".+")) %>% head() %>% knitr::kable()
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

``` r
seqstats %>%
  select(matches("[:alpha:]")) %>% head() %>% knitr::kable()
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

``` r
seqstats %>%
  select(matches("[:digit:]")) %>% head() %>% knitr::kable()
```

| experiment.ID | library.ID | IP | biorep | techrep | bwa.hg38..q.50 | bwa.dm6..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | H3K9ac | 1 | 1 | 8446795 | 787637 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | H3K9ac | 1 | 2 | 8000672 | 719513 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | H3K9ac | 1 | 3 | 7144407 | 812865 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | H3K9ac | 1 | 1 | 9000686 | 478856 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | H3K9ac | 1 | 2 | 10985910 | 499861 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | H3K9ac | 1 | 3 | 8336422 | 443340 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

# Combine dataframes

## Base R `merge()`

Structure: `merge(df_x, df_y, by = "key", all = TRUE/FALSE)`

1.  Can only merge two dataframes at a time (df_x, df_y)
2.  “key” is a column name that must be common between the two
    dataframes, used to “match” the rows even if the dfs are in a
    different order
3.  all = T/F decides whether missing rows are kept (filled 0) or
    discarded

## `bind_rows()`

Fairly narrow use, I use only if you are certain all columns match
exactly. For example in the case earlier where I split a dataframe into
only IP or input samples, then wanted to recombine

Copied from above:

``` r
seqstats <- bind_rows(seqstats, new_data)
```

See Tidyverse for `cbind()` and `rbind()`

# Changing Names with `rename_with()` and `gsub()`

## Change column names

Goal: Replace any slashes “/” with \_ in column names using
`rename_with` (from tidyverse). I do this because the slashes being a
special character could cause issues later.

Code: `rename_with(~ gsub("/", "_", .x), contains("/"))`

Breakdown:

- `rename_with()` is the general function
- using `gsub` as a “global substitutor”
- first argument of gsub is the string to be replaced: “/”
- second gsub argument is the thing to replace it with: “\_”
- third gsub argument is the dataset to be “operated” on. In this case
  because we used the tidyverse pipe operator %\>% .x refers to what’s
  being “piped in”
- rename_with() argument contains(“/”) means the gsub function is only
  applied to strings that contain the “/” string. Sometimes this
  argument is helpful if you don’t want to replace ALL “/”, but for
  example ones that have a certain word after. For example
  `contains("/sac3")` would have only changed the columns with “/sac3”
  in the column name, not “hg38/dm6”

Before:

``` r
seqstats %>% head() %>% knitr::kable()
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

After:

``` r
seqstats %>% rename_with(~ gsub("/", "_", .x), contains("/")) %>% head() %>% knitr::kable()
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

## Remove All Numbers from Column Names

Format: `names(dataset) <- gsub("[[:digit:]]", "", names(dataset))`

Example

``` r
#first copy the dataset with a new name, so we don't mess with the original
seqstats_new <- seqstats

#change all column names, removing any number
names(seqstats_new) <- gsub("[[:digit:]]", "", names(seqstats))

# checking
seqstats_new %>% head() %>% knitr::kable()
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg..q. | bwa.dm..q. | bwa.sac..q. | dm.sac | dm.hg | sac.hg | dm.hg.IP.input | sac.hg.IP.input | dm.hg.IP.input.control.norm | sac.hg.IP.input.control.norm | dm.IP.signal | sac.IP.signal | dm.eff.IP.input | sac.eff.IP.input | dm.eff.control.norm | sac.eff.control.norm | dm.normfactor.snr.adj | sac.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

### Rename Column by Order

`colnames(df)[1] <- "New Column 1 Name"`

Example:

``` r
#I'm copying the dataset with a new name, so we don't mess with the original
seqstats_newID <- seqstats

#changing the name of the first column: 
colnames(seqstats_newID)[1] <- "newID"
colnames(seqstats_newID)[2] <- "newcolumn2"

#checking:
knitr::kable(head(seqstats_newID))
```

| newID | newcolumn2 | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

## Change data in column

Example changed the sample names to remove the experiment number at the
beginning, as ggplot can’t take a number at the beginning of a variable.

``` r
# seqstats$`experiment ID` <-gsub("78", "78_test", as.character(seqstats$`experiment ID`))
knitr::kable(head(seqstats))
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 |

# Actions: subsampling rows, set NAs to 0, etc.

## random downsampling of data

Below keeps 10 random rows of the dataframe:

``` r
seqstats[sample(nrow(seqstats), 10),]
```

        experiment.ID                      library.ID        cell       IP biorep
    120          LP91        HCT116_DMSO_0hr_2_Rpb1_1 HCT116-Rpb1     Rpb1      2
    166          LP95     HCT116_IAA_1hr_3_H3K36me3_1 HCT116-Rpb1 H3K36me3      3
    197          LP95         HCT116_IAA_1hr_4_Rpb1_1 HCT116-Rpb1     Rpb1      4
    14           LP78 HelaS3_5sync_95inter_1_H3K9ac_2      HeLaS3   H3K9ac      1
    143          LP92      HCT116_IAA_1hr_2_H3K4me3_1 HCT116-Rpb1  H3K4me3      2
    169          LP95     HCT116_IAA_4hr_4_H3K36me3_1 HCT116-Rpb1 H3K36me3      4
    168          LP95     HCT116_IAA_4hr_3_H3K36me3_1 HCT116-Rpb1 H3K36me3      3
    182          LP95        HCT116_TRP_1hr_3_input_1 HCT116-Rpb1    input      3
    172          LP95      HCT116_TRP_1hr_3_H3K4me3_1 HCT116-Rpb1  H3K4me3      3
    123          LP91         HCT116_TRP_1hr_2_Rpb1_2 HCT116-Rpb1     Rpb1      2
        techrep  X bwa.hg38..q.50 bwa.dm6..q.50 bwa.sac3..q.50 dm6.sac3 dm6.hg38
    120     1.0 NA        8057207          8906            827   10.769  0.00111
    166     1.0 NA       10900478       1071176          38258   27.999  0.09827
    197     1.0 NA        4158781         39739           2473   16.069  0.00956
    14      2.0 NA       13886959        305756        1478948    0.207  0.02202
    143     1.1 NA       20105003        718860         201651    3.565  0.03576
    169     1.0 NA        7329545       1283137          79308   16.179  0.17506
    168     1.0 NA       10936219       1844417          87931   20.976  0.16865
    182     1.0 NA       24869141        274837         142554    1.928  0.01105
    172     1.0 NA        7876580       4494332         273457   16.435  0.57059
    123     2.0 NA       10074290         12017            520   23.110  0.00119
        sac3.hg38 dm6.hg38.IP.input sac3.hg38.IP.input
    120   0.00010             0.066              0.077
    166   0.00351            10.585              0.735
    197   0.00059             1.587              0.181
    14    0.10650             2.244             14.841
    143   0.01003             2.224              7.714
    169   0.01082             7.150              0.965
    168   0.00804            11.085              1.094
    182   0.00573                NA                 NA
    172   0.03472            51.631              6.057
    123   0.00005             0.056              0.032
        dm6.hg38.IP.input.control.norm sac3.hg38.IP.input.control.norm
    120                         0.7286                          0.7579
    166                         1.3781                          1.1938
    197                         1.0470                          5.7847
    14                          0.2561                          0.2719
    143                         1.3004                          1.6851
    169                         0.9308                          1.5687
    168                         1.4431                          1.7777
    182                             NA                              NA
    172                         1.1974                          1.1848
    123                         0.6168                          0.3112
        dm6.IP.signal sac3.IP.signal dm6.eff.IP.input sac3.eff.IP.input
    120    6669.26100             NA             1.03                NA
    166      96.91704       649.3339             0.01             0.009
    197    9789.04000             NA             1.51                NA
    14    22060.34000    165382.4000          6230.72             0.988
    143   39727.41000    126933.2000         33396.33         59027.870
    169      93.02468       665.5144             0.01             0.010
    168      97.16207       661.8124             0.01             0.010
    182    6533.58100     68582.7300               NA                NA
    172   30402.04000    128407.0000         23868.46         59824.270
    123    7494.06400             NA             1.17                NA
        dm6.eff.control.norm sac3.eff.control.norm dm6.normfactor.snr.adj
    120                1.013                                       0.7380
    166                1.010                 0.994                 1.3930
    197                1.133                                       1.1859
    14                 0.916                 0.992                 0.2350
    143                1.438                 1.113                 1.8700
    169                0.955                 1.012                 0.8890
    168                1.002                 1.011                 1.4460
    182                   NA                                           NA
    172                1.008                 1.007                 1.2070
    123                1.152                                       0.7100
        sac3.normfactor.snr.adj dual.normfactor.snr.adj
    120                      NA                  0.7380
    166                   1.187                  1.2900
    197                      NA                  1.1859
    14                    0.270                  0.2520
    143                   1.876                  1.8730
    169                   1.588                  1.2380
    168                   1.798                  1.6220
    182                      NA                      NA
    172                   1.194                  1.2000
    123                      NA                  0.7100

## Cleanup data - set NAs to 0

``` r
# set NAs in numeric columns to 0
seqstats[is.na(is.numeric(seqstats))] <- 0
```

# Tidyverse basics

## `mutate()` can create new column based on others

Example below (code not run) uses multiple tidyverse functions:

``` r
seqstats %>% 
  mutate(total_reads = bwa.hg38..q.50 + bwa.dm6..q.50 + bwa.sac3..q.50) %>%
  head()
```

      experiment.ID                       library.ID   cell     IP biorep techrep
    1          LP78 HelaS3_100sync_0inter_1_H3K9ac_1 HeLaS3 H3K9ac      1       1
    2          LP78 HelaS3_100sync_0inter_1_H3K9ac_2 HeLaS3 H3K9ac      1       2
    3          LP78 HelaS3_100sync_0inter_1_H3K9ac_3 HeLaS3 H3K9ac      1       3
    4          LP78 HelaS3_75sync_25inter_1_H3K9ac_1 HeLaS3 H3K9ac      1       1
    5          LP78 HelaS3_75sync_25inter_1_H3K9ac_2 HeLaS3 H3K9ac      1       2
    6          LP78 HelaS3_75sync_25inter_1_H3K9ac_3 HeLaS3 H3K9ac      1       3
       X bwa.hg38..q.50 bwa.dm6..q.50 bwa.sac3..q.50 dm6.sac3 dm6.hg38 sac3.hg38
    1 NA        8446795        787637        3384998    0.233  0.09325   0.40074
    2 NA        8000672        719513        3165051    0.227  0.08993   0.39560
    3 NA        7144407        812865        3581432    0.227  0.11378   0.50129
    4 NA        9000686        478856        2350718    0.204  0.05320   0.26117
    5 NA       10985910        499861        2263534    0.221  0.04550   0.20604
    6 NA        8336422        443340        2215916    0.200  0.05318   0.26581
      dm6.hg38.IP.input sac3.hg38.IP.input dm6.hg38.IP.input.control.norm
    1             8.253             50.572                         0.9420
    2             7.959             49.922                         0.9085
    3            10.070             63.260                         1.1494
    4             4.963             34.029                         0.5665
    5             4.245             26.846                         0.4845
    6             4.961             34.634                         0.5663
      sac3.hg38.IP.input.control.norm dm6.IP.signal sac3.IP.signal dm6.eff.IP.input
    1                          0.9265      22496.56       165939.1          6720.68
    2                          0.9146      22511.03       166041.9          6735.15
    3                          1.1589      22718.79       166251.4          6942.91
    4                          0.6234      22547.08       165787.9          6860.25
    5                          0.4918      22465.16       165797.5          6778.33
    6                          0.6345      22570.42       165843.8          6883.59
      sac3.eff.IP.input dm6.eff.control.norm sac3.eff.control.norm
    1             0.995                0.988                 0.999
    2             0.996                0.991                 1.000
    3             0.997                1.021                 1.001
    4             0.992                1.009                 0.996
    5             0.992                0.997                 0.996
    6             0.992                1.012                 0.996
      dm6.normfactor.snr.adj sac3.normfactor.snr.adj dual.normfactor.snr.adj
    1                  0.931                   0.926                   0.928
    2                  0.900                   0.914                   0.907
    3                  1.174                   1.160                   1.167
    4                  0.572                   0.621                   0.596
    5                  0.483                   0.490                   0.486
    6                  0.573                   0.632                   0.603
      total_reads
    1    12619430
    2    11885236
    3    11538704
    4    11830260
    5    13749305
    6    10995678

## select columns

By full name:

``` r
seqstats %>% 
   select(., IP) %>% head() %>% knitr::kable()
```

| IP     |
|:-------|
| H3K9ac |
| H3K9ac |
| H3K9ac |
| H3K9ac |
| H3K9ac |
| H3K9ac |

By partial match with `contains`

``` r
seqstats %>% 
   select(., contains("snr")) %>% head() %>% knitr::kable()
```

| dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|-----------------------:|------------------------:|------------------------:|
|                  0.931 |                   0.926 |                   0.928 |
|                  0.900 |                   0.914 |                   0.907 |
|                  1.174 |                   1.160 |                   1.167 |
|                  0.572 |                   0.621 |                   0.596 |
|                  0.483 |                   0.490 |                   0.486 |
|                  0.573 |                   0.632 |                   0.603 |

## paring mutate and select

1.  `mutate()` is used to create a new column. First argument is the
    column name, to be filled with some data =
2.  `select()` from the piped in dataframe selects specific columns
    matching arguments inside (), looks for partial string matches with
    `contains()`
3.  The columns selected in (2) are piped into `rowMeans()` which takes
    the average of the two columns for each row individually.
4.  Resulting data put in new column defined by `mutate()`

``` r
seqstats %>% 
  mutate(avg_normfactor = select(., contains("snr")) %>% 
           rowMeans()) %>% head() %>% knitr::kable()
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj | avg_normfactor |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 | 0.9283333 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 | 0.9070000 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 | 1.1670000 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 | 0.5963333 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 | 0.4863333 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 | 0.6026667 |

Before doing math, check which columns you are grabbing first! Here, we
are looking to compute the average of two columns:
`dm6.normfactor.snr.adj` and `sac3.normfactor.snr.adj` to double check
our `dual.normfactor.snr.adj` column is accurate.

We select for the snr/IP efficiency adjusted normalization factor
columns by matching the string `snr.adj`, but then this also selets for
the `dual.normfactor.snr.adj` column, which we do not want included.

``` r
seqstats %>% 
  select(.,  matches('snr.adj')) %>% head() %>% knitr::kable()
```

| dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj |
|-----------------------:|------------------------:|------------------------:|
|                  0.931 |                   0.926 |                   0.928 |
|                  0.900 |                   0.914 |                   0.907 |
|                  1.174 |                   1.160 |                   1.167 |
|                  0.572 |                   0.621 |                   0.596 |
|                  0.483 |                   0.490 |                   0.486 |
|                  0.573 |                   0.632 |                   0.603 |

We can modify our search, so that we match “snr.adj” AND (“sac3” OR
“dm6”)

``` r
seqstats %>% 
  select(., matches('sac3|dm6') & matches('snr.adj')) %>% head() %>% knitr::kable()
```

| dm6.normfactor.snr.adj | sac3.normfactor.snr.adj |
|-----------------------:|------------------------:|
|                  0.931 |                   0.926 |
|                  0.900 |                   0.914 |
|                  1.174 |                   1.160 |
|                  0.572 |                   0.621 |
|                  0.483 |                   0.490 |
|                  0.573 |                   0.632 |

``` r
seqstats %>% 
  mutate(avg_normfactor_adj = select(., matches('sac3|dm6') & matches('snr.adj')) %>% 
           rowMeans()) %>% head() %>% knitr::kable()
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | X | bwa.hg38..q.50 | bwa.dm6..q.50 | bwa.sac3..q.50 | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj | avg_normfactor_adj |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 | 0.9285 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 | 0.9070 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.227 | 0.11378 | 0.50129 | 10.070 | 63.260 | 1.1494 | 1.1589 | 22718.79 | 166251.4 | 6942.91 | 0.997 | 1.021 | 1.001 | 1.174 | 1.160 | 1.167 | 1.1670 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.204 | 0.05320 | 0.26117 | 4.963 | 34.029 | 0.5665 | 0.6234 | 22547.08 | 165787.9 | 6860.25 | 0.992 | 1.009 | 0.996 | 0.572 | 0.621 | 0.596 | 0.5965 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.221 | 0.04550 | 0.20604 | 4.245 | 26.846 | 0.4845 | 0.4918 | 22465.16 | 165797.5 | 6778.33 | 0.992 | 0.997 | 0.996 | 0.483 | 0.490 | 0.486 | 0.4865 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.200 | 0.05318 | 0.26581 | 4.961 | 34.634 | 0.5663 | 0.6345 | 22570.42 | 165843.8 | 6883.59 | 0.992 | 1.012 | 0.996 | 0.573 | 0.632 | 0.603 | 0.6025 |

## Pair Data

From here:
https://datavizpyr.com/connect-paired-points-with-lines-in-scatterplot-in-ggplot2/

Example from gapminder: pair together data points of different years in
same country by adding another column `paired`. This new column is
filled in with numbers sequentially from 1-(# of rows/2) as there are
two rows per country. These numbers are each repeated twice, leaving us
with a column with a unique identifier for each “paired” set of points.

``` r
gapminder_paired <- gapminder %>% mutate(paired = rep(1:(n()/2), each = 2), year = factor(year))

knitr::kable(head(gapminder_paired))
```

| country     | continent | year | lifeExp |      pop | gdpPercap | paired |
|:------------|:----------|:-----|--------:|---------:|----------:|-------:|
| Afghanistan | Asia      | 1952 |  28.801 |  8425333 |  779.4453 |      1 |
| Afghanistan | Asia      | 1957 |  30.332 |  9240934 |  820.8530 |      1 |
| Afghanistan | Asia      | 1962 |  31.997 | 10267083 |  853.1007 |      2 |
| Afghanistan | Asia      | 1967 |  34.020 | 11537966 |  836.1971 |      2 |
| Afghanistan | Asia      | 1972 |  36.088 | 13079460 |  739.9811 |      3 |
| Afghanistan | Asia      | 1977 |  38.438 | 14880372 |  786.1134 |      3 |

In the section Create data –\> Column filled using patterns I use this
to make new columns for my data.

## Long Format (TidyR format) with `pivot_longer()`

Pivot_longer arguments: - cols: can be all, or select (example is all
columns EXCEPT named Distance_from_tss) - names_to: names of columns
selected are now entries within this columns (usually these are sample
names for me) - values_to: the rows within each column that are now in
“Sample” are in this new column

Chose specific columns:

``` r
seqstats_tidy <- 
    seqstats %>% pivot_longer(
      cols = c(bwa.hg38..q.50, bwa.dm6..q.50, bwa.sac3..q.50), 
      names_to = "Species", 
      values_to = "Reads")

knitr::kable(head(seqstats_tidy))
```

| experiment.ID | library.ID | cell | IP | biorep | techrep | X | dm6.sac3 | dm6.hg38 | sac3.hg38 | dm6.hg38.IP.input | sac3.hg38.IP.input | dm6.hg38.IP.input.control.norm | sac3.hg38.IP.input.control.norm | dm6.IP.signal | sac3.IP.signal | dm6.eff.IP.input | sac3.eff.IP.input | dm6.eff.control.norm | sac3.eff.control.norm | dm6.normfactor.snr.adj | sac3.normfactor.snr.adj | dual.normfactor.snr.adj | Species | Reads |
|:---|:---|:---|:---|---:|---:|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|:---|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 | bwa.hg38..q.50 | 8446795 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 | bwa.dm6..q.50 | 787637 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 0.233 | 0.09325 | 0.40074 | 8.253 | 50.572 | 0.9420 | 0.9265 | 22496.56 | 165939.1 | 6720.68 | 0.995 | 0.988 | 0.999 | 0.931 | 0.926 | 0.928 | bwa.sac3..q.50 | 3384998 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 | bwa.hg38..q.50 | 8000672 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 | bwa.dm6..q.50 | 719513 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 0.227 | 0.08993 | 0.39560 | 7.959 | 49.922 | 0.9085 | 0.9146 | 22511.03 | 166041.9 | 6735.15 | 0.996 | 0.991 | 1.000 | 0.900 | 0.914 | 0.907 | bwa.sac3..q.50 | 3165051 |

## Reorder column contents

General Example

Replicate data with `data_new <- data`

Reordering group factor levels

``` r
data_new <- rep(c("A","B", "C", "D"), each = 5)
col2 <- rep(c("12", "10"), each = 10)
data_new$Values <- col2
```

    Warning in data_new$Values <- col2: Coercing LHS to a list

``` r
head(data_new)
```

    [[1]]
    [1] "A"

    [[2]]
    [1] "A"

    [[3]]
    [1] "A"

    [[4]]
    [1] "A"

    [[5]]
    [1] "A"

    [[6]]
    [1] "B"

``` r
data_new$group <- factor(data_new$group,levels = c("B", "A", "C", "D"))
head(data_new)
```

    [[1]]
    [1] "A"

    [[2]]
    [1] "A"

    [[3]]
    [1] "A"

    [[4]]
    [1] "A"

    [[5]]
    [1] "A"

    [[6]]
    [1] "B"

## Apply function to groups within dataframe

Dataset:

``` r
# import data
alignstats <- read.delim("~/Research/spike_commentary/public_data_alignments.txt")
# set replicate as factor not numeric
alignstats$Rep <- as.factor(alignstats$Rep)
```

### Function to calculate variation in inputs within each paper

``` r
# function to apply to each author: 
get_input_variation <- function(df, .df) {
   dfnew <- df                                # copy input dataframe
   mininput <- min(df$spike_target, na.rm = T) # get min from spike_targ
   var <- as.numeric((df$spike_target)/mininput) # make var column
   dfnew %>%
     mutate(var = var)                          # add to df
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

# Create data

## Empty vector

Below: creates a new vector, gives it a class (numeric) with length
equal to a vector previously made

`hg38_total_mapq10 <- vector("numeric", length(SRA))`

## Column filled with basic math

Adding three new columns:

1.  dm6_sac3 = ratio between dm6 and sac3 reads per sample
2.  dm6_hg38 = ratio between dm6 and hg38 reads per sample
3.  sac3_hg38 = ratio between sac3 and hg38 reads per sample

``` r
seqstats_230910_expanded <- seqstats_230910_expanded %>%
  mutate(dm6_sac3 = dm6_nodup_q10/sac3_nodup_q10) %>%
  mutate(dm6_hg38 = dm6_nodup_q10/hg38_dups) %>% 
  mutate(sac3_hg38 = sac3_nodup_q10/hg38_dups)
```

## rowMeans()

``` r
counts_BRD4_HA_tidy <- counts_BRD4_HA_tidy %>% 
  mutate(HA_avg_0.5 = select(., contains("HA_0.5ul")) %>% rowMeans()) %>%
  mutate(HA_avg_1 = select(., contains("HA_1ul")) %>% rowMeans()) 
```

## Column filled using patterns

Each means that the objects x, y, z in the vector (c(x, y, z), each = 3)
will be repeated by this pattern: x, x, x, y, y, y, z, z, z

Using “times” instead (c(x, y, z), times = 3) results in x, y, z, x, y,
z, x, y, z

You can use both to create more complex patterns as shown in the example
below:

``` r
# List with each element repeated together
Yeast_each<-rep(c("yeast_0.1k","yeast_1k","yeast_10k","yeast_100k","yeast_1000k", "yeast_10000k", "yeast_100000k"), each=2)
Yeast_each
```

     [1] "yeast_0.1k"    "yeast_0.1k"    "yeast_1k"      "yeast_1k"     
     [5] "yeast_10k"     "yeast_10k"     "yeast_100k"    "yeast_100k"   
     [9] "yeast_1000k"   "yeast_1000k"   "yeast_10000k"  "yeast_10000k" 
    [13] "yeast_100000k" "yeast_100000k"

``` r
# List itself repeated
Yeast_times<-rep(c("yeast_0.1k","yeast_1k","yeast_10k","yeast_100k","yeast_1000k", "yeast_10000k", "yeast_100000k"), times=2)
Yeast_times
```

     [1] "yeast_0.1k"    "yeast_1k"      "yeast_10k"     "yeast_100k"   
     [5] "yeast_1000k"   "yeast_10000k"  "yeast_100000k" "yeast_0.1k"   
     [9] "yeast_1k"      "yeast_10k"     "yeast_100k"    "yeast_1000k"  
    [13] "yeast_10000k"  "yeast_100000k"

``` r
# Combine for more complex patterns
Yeastnew <-rep(c(Yeast_each), times = 3)
Yeastnew
```

     [1] "yeast_0.1k"    "yeast_0.1k"    "yeast_1k"      "yeast_1k"     
     [5] "yeast_10k"     "yeast_10k"     "yeast_100k"    "yeast_100k"   
     [9] "yeast_1000k"   "yeast_1000k"   "yeast_10000k"  "yeast_10000k" 
    [13] "yeast_100000k" "yeast_100000k" "yeast_0.1k"    "yeast_0.1k"   
    [17] "yeast_1k"      "yeast_1k"      "yeast_10k"     "yeast_10k"    
    [21] "yeast_100k"    "yeast_100k"    "yeast_1000k"   "yeast_1000k"  
    [25] "yeast_10000k"  "yeast_10000k"  "yeast_100000k" "yeast_100000k"
    [29] "yeast_0.1k"    "yeast_0.1k"    "yeast_1k"      "yeast_1k"     
    [33] "yeast_10k"     "yeast_10k"     "yeast_100k"    "yeast_100k"   
    [37] "yeast_1000k"   "yeast_1000k"   "yeast_10000k"  "yeast_10000k" 
    [41] "yeast_100000k" "yeast_100000k"

## Add column to dataframe

This only works if the object Treatment is a single column

``` r
#df$Treatment <- Treatment
```

## Add column from dataframe (containing \> 1 column) to different dataframe

Dataframe 1 is the dataframe you want to add to

``` r
df1['newColName'] = df2['ExistingColumnInDF2']
```

## Make Dataframe from vectors

``` r
df <- data.frame(vector1, vector2, vector3)
```

## Add column to number groups

``` r
country_num <- gapminder %>% 
    group_by(country) %>% 
    group_indices(., country)
```

    Warning: The `...` argument of `group_indices()` is deprecated as of dplyr 1.0.0.
    ℹ Please `group_by()` first

``` r
gapmindernew <- gapminder

gapmindernew$country_num <- as.factor(country_num)

knitr::kable(head(gapmindernew))
```

| country     | continent | year | lifeExp |      pop | gdpPercap | country_num |
|:------------|:----------|-----:|--------:|---------:|----------:|:------------|
| Afghanistan | Asia      | 1952 |  28.801 |  8425333 |  779.4453 | 1           |
| Afghanistan | Asia      | 1957 |  30.332 |  9240934 |  820.8530 | 1           |
| Afghanistan | Asia      | 1962 |  31.997 | 10267083 |  853.1007 | 1           |
| Afghanistan | Asia      | 1967 |  34.020 | 11537966 |  836.1971 | 1           |
| Afghanistan | Asia      | 1972 |  36.088 | 13079460 |  739.9811 | 1           |
| Afghanistan | Asia      | 1977 |  38.438 | 14880372 |  786.1134 | 1           |

# Special cases

## Convert Time Series to Dataframe with `as.matrix()` and `data.frame()`

Example from seatbelts dataset: need to add a new column “date” and
convert the time series to a table with the year in “date”

``` r
data("Seatbelts")
seatbelts_df <- data.frame(as.matrix(Seatbelts), date = time(Seatbelts))
head(seatbelts_df)
```

      DriversKilled drivers front rear   kms PetrolPrice VanKilled law     date
    1           107    1687   867  269  9059   0.1029718        12   0 1969.000
    2            97    1508   825  265  7685   0.1023630         6   0 1969.083
    3           102    1507   806  319  9963   0.1020625        12   0 1969.167
    4            87    1385   814  407 10955   0.1008733         8   0 1969.250
    5           119    1632   991  454 11823   0.1010197        10   0 1969.333
    6           106    1511   945  427 12391   0.1005812        13   0 1969.417

## Split verbose column into multiple with `separate_wider_regex()`

Using `separate_wider_regex()` from tidyr package, info here:
https://tidyr.tidyverse.org/reference/separate_wider_delim.html

Syntax:

``` r
dataframe %>% separate_wider_regex(
  cols = columntosplit, 
  patterns = c(
    newcol1 = "regex",
    "regex", #if you want to discard anything between columns
    newcol2 = "regex"
  ))
```
