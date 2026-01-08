Dataframe Manipulation in R
================

true

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

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(gapminder) # public dataset used for examples
library(viridis)   # library of colorblind-friendly palettes
```

    ## Loading required package: viridisLite

### Theme for ggplot

``` r
theme_set(theme_classic()) # feel free to not use this theme if you find a different one you prefer
```

## Some general notes for this document:

1.  if you see `%>%` in code, this is the pipe operator used in
    tidyverse. In the newest version of R (after 4.0) you can also use
    the standard pipe `|`
2.  Most examples will be either my own or public data commonly used in
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
seqstats <- read_excel("Sequencing_Stats_260107.xlsx", 
    sheet = "all_dualspike_metadata")
```

    ## New names:
    ## • `` -> `...7`

``` r
knitr::kable(head(seqstats))
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

# Getting Basic Information

## Basic operations:

Isolating a column by name: `seqstats$cell`

If there are spaces in the name, use “\`”, as in

``` r
seqstats$`experiment ID`
```

Print column names: `names(seqstats)` or `colnames(seqstats)`

Isolate column by position: `seqstats[1]`

## Cleanup column names in Kable with `gsub()`

Usually dataframe column names have \_ or . in their name for ease of
coding, but you might want to display the names in a more human-readable
way in the kable:

Example below replaces underscores in column names with spaces only for
knitr::kable() to display, WITHOUT modifying the original dataset.

``` r
knitr::kable(head(seqstats), col.names = gsub("[_]", " ", names(seqstats)))
```

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.2326846 | 0.0932468 | 0.4007435 | 8.252629 | 50.57160 | 0.9420306 | 0.9264793 | 22496.56 | 165939.1 | 6720.68 | 0.9951239 | 0.9883963 | 0.9991668547 | 0.9310996 | 0.9257074 | 0.9284035 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.2273306 | 0.0899316 | 0.3955981 | 7.959218 | 49.92229 | 0.9085379 | 0.9145838 | 22511.03 | 166041.9 | 6735.15 | 0.9957404 | 0.9905244 | 0.9997858429 | 0.8999290 | 0.9143879 | 0.9071584 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.2269665 | 0.1137764 | 0.5012917 | 10.069558 | 63.26023 | 1.1494315 | 1.1589369 | 22718.79 | 166251.4 | 6942.91 | 0.9969967 | 1.0210792 | 1.001047302 | 1.1736607 | 1.1601506 | 1.1669056 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.2037063 | 0.0532022 | 0.2611710 | 4.963025 | 34.02945 | 0.5665250 | 0.6234247 | 22547.08 | 165787.9 | 6860.25 | 0.9915105 | 1.0089226 | 0.9955388152 | 0.5715799 | 0.6206435 | 0.5961117 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.2208321 | 0.0455002 | 0.2060397 | 4.244537 | 26.84609 | 0.4845103 | 0.4918244 | 22465.16 | 165797.5 | 6778.33 | 0.9915680 | 0.9968748 | 0.9955964622 | 0.4829961 | 0.4896586 | 0.4863274 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.2000708 | 0.0531811 | 0.2658114 | 4.961059 | 34.63408 | 0.5663006 | 0.6345015 | 22570.42 | 165843.8 | 6883.59 | 0.9918449 | 1.0123552 | 0.9958744888 | 0.5732973 | 0.6318839 | 0.6025906 |

## Variable classes

``` r
str(seqstats)
```

    ## tibble [279 × 26] (S3: tbl_df/tbl/data.frame)
    ##  $ experiment ID                  : chr [1:279] "LP78" "LP78" "LP78" "LP78" ...
    ##  $ library ID                     : chr [1:279] "HelaS3_100sync_0inter_1_H3K9ac_1" "HelaS3_100sync_0inter_1_H3K9ac_2" "HelaS3_100sync_0inter_1_H3K9ac_3" "HelaS3_75sync_25inter_1_H3K9ac_1" ...
    ##  $ cell                           : chr [1:279] "HeLaS3" "HeLaS3" "HeLaS3" "HeLaS3" ...
    ##  $ IP                             : chr [1:279] "H3K9ac" "H3K9ac" "H3K9ac" "H3K9ac" ...
    ##  $ biorep                         : num [1:279] 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ techrep                        : num [1:279] 1 2 3 1 2 3 1 2 3 1 ...
    ##  $ ...7                           : num [1:279] NA NA NA NA NA NA NA NA NA NA ...
    ##  $ bwa hg38 -q 50                 : num [1:279] 8446795 8000672 7144407 9000686 10985910 ...
    ##  $ bwa dm6 -q 50                  : num [1:279] 787637 719513 812865 478856 499861 ...
    ##  $ bwa sac3 -q 50                 : num [1:279] 3384998 3165051 3581432 2350718 2263534 ...
    ##  $ dm6/sac3                       : num [1:279] 0.233 0.227 0.227 0.204 0.221 ...
    ##  $ dm6/hg38                       : num [1:279] 0.0932 0.0899 0.1138 0.0532 0.0455 ...
    ##  $ sac3/hg38                      : num [1:279] 0.401 0.396 0.501 0.261 0.206 ...
    ##  $ dm6/hg38 IP/input              : num [1:279] 8.25 7.96 10.07 4.96 4.24 ...
    ##  $ sac3/hg38 IP/input             : num [1:279] 50.6 49.9 63.3 34 26.8 ...
    ##  $ dm6/hg38 IP/input control norm : num [1:279] 0.942 0.909 1.149 0.567 0.485 ...
    ##  $ sac3/hg38 IP/input control norm: num [1:279] 0.926 0.915 1.159 0.623 0.492 ...
    ##  $ dm6 IP signal                  : num [1:279] 22497 22511 22719 22547 22465 ...
    ##  $ sac3 IP signal                 : num [1:279] 165939 166042 166251 165788 165798 ...
    ##  $ dm6 eff IP/input               : num [1:279] 6721 6735 6943 6860 6778 ...
    ##  $ sac3 eff IP/input              : num [1:279] 0.995 0.996 0.997 0.992 0.992 ...
    ##  $ dm6 eff control norm           : num [1:279] 0.988 0.991 1.021 1.009 0.997 ...
    ##  $ sac3 eff control norm          : chr [1:279] "0.9991668547" "0.9997858429" "1.001047302" "0.9955388152" ...
    ##  $ dm6 normfactor snr adj         : num [1:279] 0.931 0.9 1.174 0.572 0.483 ...
    ##  $ sac3 normfactor snr adj        : num [1:279] 0.926 0.914 1.16 0.621 0.49 ...
    ##  $ dual normfactor snr adj        : num [1:279] 0.928 0.907 1.167 0.596 0.486 ...

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
grep("_1hr", seqstats$ID)
```

    ## Warning: Unknown or uninitialised column: `ID`.

    ## integer(0)

**`grepl()`** is very similar, but instead of returining indices of the
matches location, it returns a logical vector, with TRUE representing a
match, and FALSE if not a match

The same example with grepl:

``` r
grepl("_1hr", seqstats$ID)
```

    ## Warning: Unknown or uninitialised column: `ID`.

    ## logical(0)

## Search and return row of dataframe

Syntax: `dataframe[grep("string", dataframe$column), ]`

Use indexes to return row with match: works with `grep` and `grepl`

``` r
seqstats[grep("0hr", seqstats$ID), ] %>% knitr::kable()
```

    ## Warning: Unknown or uninitialised column: `ID`.

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|

## Multiple matches with logical operators

``` r
seqstats[grep("69a|69e", seqstats$ID), ] %>% knitr::kable()
```

    ## Warning: Unknown or uninitialised column: `ID`.

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|

# Split verbose column into multiple with `separate_wider_regex()`

Using `separate_wider_regex()` from tidyr package, info here:
<https://tidyr.tidyverse.org/reference/separate_wider_delim.html>

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

Regex:

- `^` = anchor beginning of string
- `$` = anchor end of the string
- `.` = matches any character but newline
- \[\[:alpha:\]\] = letters
- \[\[:digit:\]\] = digits
- \[\[:alnum:\]\] = letters and numbers
- \[\[:graph:\]\] = any character but not white space/blank

Special: `+` goes after a match, means it can be counted more than once!
Useful instead of denoting `[:digit:][:digit:]` for a two digit number

## Example with my sequencing data

My ChIP samples are labeled with the following format in the sample
sheet before sequencing:

\[experiment##\]\[sampleID\]\_\[celltype\]\_\[timepoint\]\_\[treatment\]\_\[antibody\]\_\[rep\]

``` r
knitr::kable(head(seqstats))
```

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.2326846 | 0.0932468 | 0.4007435 | 8.252629 | 50.57160 | 0.9420306 | 0.9264793 | 22496.56 | 165939.1 | 6720.68 | 0.9951239 | 0.9883963 | 0.9991668547 | 0.9310996 | 0.9257074 | 0.9284035 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.2273306 | 0.0899316 | 0.3955981 | 7.959218 | 49.92229 | 0.9085379 | 0.9145838 | 22511.03 | 166041.9 | 6735.15 | 0.9957404 | 0.9905244 | 0.9997858429 | 0.8999290 | 0.9143879 | 0.9071584 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.2269665 | 0.1137764 | 0.5012917 | 10.069558 | 63.26023 | 1.1494315 | 1.1589369 | 22718.79 | 166251.4 | 6942.91 | 0.9969967 | 1.0210792 | 1.001047302 | 1.1736607 | 1.1601506 | 1.1669056 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.2037063 | 0.0532022 | 0.2611710 | 4.963025 | 34.02945 | 0.5665250 | 0.6234247 | 22547.08 | 165787.9 | 6860.25 | 0.9915105 | 1.0089226 | 0.9955388152 | 0.5715799 | 0.6206435 | 0.5961117 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.2208321 | 0.0455002 | 0.2060397 | 4.244537 | 26.84609 | 0.4845103 | 0.4918244 | 22465.16 | 165797.5 | 6778.33 | 0.9915680 | 0.9968748 | 0.9955964622 | 0.4829961 | 0.4896586 | 0.4863274 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.2000708 | 0.0531811 | 0.2658114 | 4.961059 | 34.63408 | 0.5663006 | 0.6345015 | 22570.42 | 165843.8 | 6883.59 | 0.9918449 | 1.0123552 | 0.9958744888 | 0.5732973 | 0.6318839 | 0.6025906 |

# Combine dataframes

## Tidyverse `cbind()`

## Tidyverse `rbind()`

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

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.2326846 | 0.0932468 | 0.4007435 | 8.252629 | 50.57160 | 0.9420306 | 0.9264793 | 22496.56 | 165939.1 | 6720.68 | 0.9951239 | 0.9883963 | 0.9991668547 | 0.9310996 | 0.9257074 | 0.9284035 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.2273306 | 0.0899316 | 0.3955981 | 7.959218 | 49.92229 | 0.9085379 | 0.9145838 | 22511.03 | 166041.9 | 6735.15 | 0.9957404 | 0.9905244 | 0.9997858429 | 0.8999290 | 0.9143879 | 0.9071584 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.2269665 | 0.1137764 | 0.5012917 | 10.069558 | 63.26023 | 1.1494315 | 1.1589369 | 22718.79 | 166251.4 | 6942.91 | 0.9969967 | 1.0210792 | 1.001047302 | 1.1736607 | 1.1601506 | 1.1669056 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.2037063 | 0.0532022 | 0.2611710 | 4.963025 | 34.02945 | 0.5665250 | 0.6234247 | 22547.08 | 165787.9 | 6860.25 | 0.9915105 | 1.0089226 | 0.9955388152 | 0.5715799 | 0.6206435 | 0.5961117 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.2208321 | 0.0455002 | 0.2060397 | 4.244537 | 26.84609 | 0.4845103 | 0.4918244 | 22465.16 | 165797.5 | 6778.33 | 0.9915680 | 0.9968748 | 0.9955964622 | 0.4829961 | 0.4896586 | 0.4863274 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.2000708 | 0.0531811 | 0.2658114 | 4.961059 | 34.63408 | 0.5663006 | 0.6345015 | 22570.42 | 165843.8 | 6883.59 | 0.9918449 | 1.0123552 | 0.9958744888 | 0.5732973 | 0.6318839 | 0.6025906 |

After:

``` r
seqstats %>% rename_with(~ gsub("/", "_", .x), contains("/")) %>% head() %>% knitr::kable()
```

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6_sac3 | dm6_hg38 | sac3_hg38 | dm6_hg38 IP_input | sac3_hg38 IP_input | dm6_hg38 IP_input control norm | sac3_hg38 IP_input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP_input | sac3 eff IP_input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.2326846 | 0.0932468 | 0.4007435 | 8.252629 | 50.57160 | 0.9420306 | 0.9264793 | 22496.56 | 165939.1 | 6720.68 | 0.9951239 | 0.9883963 | 0.9991668547 | 0.9310996 | 0.9257074 | 0.9284035 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.2273306 | 0.0899316 | 0.3955981 | 7.959218 | 49.92229 | 0.9085379 | 0.9145838 | 22511.03 | 166041.9 | 6735.15 | 0.9957404 | 0.9905244 | 0.9997858429 | 0.8999290 | 0.9143879 | 0.9071584 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.2269665 | 0.1137764 | 0.5012917 | 10.069558 | 63.26023 | 1.1494315 | 1.1589369 | 22718.79 | 166251.4 | 6942.91 | 0.9969967 | 1.0210792 | 1.001047302 | 1.1736607 | 1.1601506 | 1.1669056 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.2037063 | 0.0532022 | 0.2611710 | 4.963025 | 34.02945 | 0.5665250 | 0.6234247 | 22547.08 | 165787.9 | 6860.25 | 0.9915105 | 1.0089226 | 0.9955388152 | 0.5715799 | 0.6206435 | 0.5961117 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.2208321 | 0.0455002 | 0.2060397 | 4.244537 | 26.84609 | 0.4845103 | 0.4918244 | 22465.16 | 165797.5 | 6778.33 | 0.9915680 | 0.9968748 | 0.9955964622 | 0.4829961 | 0.4896586 | 0.4863274 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.2000708 | 0.0531811 | 0.2658114 | 4.961059 | 34.63408 | 0.5663006 | 0.6345015 | 22570.42 | 165843.8 | 6883.59 | 0.9918449 | 1.0123552 | 0.9958744888 | 0.5732973 | 0.6318839 | 0.6025906 |

## Remove All Numbers from Column Names

Format: `names(dataset) <- gsub("[[:digit:]]", "", names(dataset))`

Example

``` r
#first copy the dataset with a new name, so we don't mess with the original
seqstats_new <- seqstats

#change all column names, removing any number
names(seqstats_new) <- gsub("[[:digit:]]", "", names(seqstats))

# checking
seqstats_new %>% head()
```

    ## # A tibble: 6 × 26
    ##   `experiment ID` `library ID`     cell  IP    biorep techrep   ... `bwa hg -q `
    ##   <chr>           <chr>            <chr> <chr>  <dbl>   <dbl> <dbl>        <dbl>
    ## 1 LP78            HelaS3_100sync_… HeLa… H3K9…      1       1    NA      8446795
    ## 2 LP78            HelaS3_100sync_… HeLa… H3K9…      1       2    NA      8000672
    ## 3 LP78            HelaS3_100sync_… HeLa… H3K9…      1       3    NA      7144407
    ## 4 LP78            HelaS3_75sync_2… HeLa… H3K9…      1       1    NA      9000686
    ## 5 LP78            HelaS3_75sync_2… HeLa… H3K9…      1       2    NA     10985910
    ## 6 LP78            HelaS3_75sync_2… HeLa… H3K9…      1       3    NA      8336422
    ## # ℹ 18 more variables: `bwa dm -q ` <dbl>, `bwa sac -q ` <dbl>, `dm/sac` <dbl>,
    ## #   `dm/hg` <dbl>, `sac/hg` <dbl>, `dm/hg IP/input` <dbl>,
    ## #   `sac/hg IP/input` <dbl>, `dm/hg IP/input control norm` <dbl>,
    ## #   `sac/hg IP/input control norm` <dbl>, `dm IP signal` <dbl>,
    ## #   `sac IP signal` <dbl>, `dm eff IP/input` <dbl>, `sac eff IP/input` <dbl>,
    ## #   `dm eff control norm` <dbl>, `sac eff control norm` <chr>,
    ## #   `dm normfactor snr adj` <dbl>, `sac normfactor snr adj` <dbl>, …

### Rename Column by Order

`colnames(df)[1] <- "New Column 1 Name"`

Example:

``` r
#first copy the dataset with a new name, so we don't mess with the original
seqstats_newID <- seqstats

#changing the name: 
colnames(seqstats_newID)[1] <- "newID"

#checking:
knitr::kable(head(seqstats_newID))
```

| newID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.2326846 | 0.0932468 | 0.4007435 | 8.252629 | 50.57160 | 0.9420306 | 0.9264793 | 22496.56 | 165939.1 | 6720.68 | 0.9951239 | 0.9883963 | 0.9991668547 | 0.9310996 | 0.9257074 | 0.9284035 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.2273306 | 0.0899316 | 0.3955981 | 7.959218 | 49.92229 | 0.9085379 | 0.9145838 | 22511.03 | 166041.9 | 6735.15 | 0.9957404 | 0.9905244 | 0.9997858429 | 0.8999290 | 0.9143879 | 0.9071584 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.2269665 | 0.1137764 | 0.5012917 | 10.069558 | 63.26023 | 1.1494315 | 1.1589369 | 22718.79 | 166251.4 | 6942.91 | 0.9969967 | 1.0210792 | 1.001047302 | 1.1736607 | 1.1601506 | 1.1669056 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.2037063 | 0.0532022 | 0.2611710 | 4.963025 | 34.02945 | 0.5665250 | 0.6234247 | 22547.08 | 165787.9 | 6860.25 | 0.9915105 | 1.0089226 | 0.9955388152 | 0.5715799 | 0.6206435 | 0.5961117 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.2208321 | 0.0455002 | 0.2060397 | 4.244537 | 26.84609 | 0.4845103 | 0.4918244 | 22465.16 | 165797.5 | 6778.33 | 0.9915680 | 0.9968748 | 0.9955964622 | 0.4829961 | 0.4896586 | 0.4863274 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.2000708 | 0.0531811 | 0.2658114 | 4.961059 | 34.63408 | 0.5663006 | 0.6345015 | 22570.42 | 165843.8 | 6883.59 | 0.9918449 | 1.0123552 | 0.9958744888 | 0.5732973 | 0.6318839 | 0.6025906 |

Where the \# above is the column number (above changes the name of the
first column)

## Change data in column

Example changed the sample names to remove the experiment number at the
beginning, as ggplot can’t take a number at the beginning of a variable.

``` r
# seqstats$`experiment ID` <-gsub("78", "78_test", as.character(seqstats$`experiment ID`))
knitr::kable(head(seqstats))
```

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.2326846 | 0.0932468 | 0.4007435 | 8.252629 | 50.57160 | 0.9420306 | 0.9264793 | 22496.56 | 165939.1 | 6720.68 | 0.9951239 | 0.9883963 | 0.9991668547 | 0.9310996 | 0.9257074 | 0.9284035 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.2273306 | 0.0899316 | 0.3955981 | 7.959218 | 49.92229 | 0.9085379 | 0.9145838 | 22511.03 | 166041.9 | 6735.15 | 0.9957404 | 0.9905244 | 0.9997858429 | 0.8999290 | 0.9143879 | 0.9071584 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.2269665 | 0.1137764 | 0.5012917 | 10.069558 | 63.26023 | 1.1494315 | 1.1589369 | 22718.79 | 166251.4 | 6942.91 | 0.9969967 | 1.0210792 | 1.001047302 | 1.1736607 | 1.1601506 | 1.1669056 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.2037063 | 0.0532022 | 0.2611710 | 4.963025 | 34.02945 | 0.5665250 | 0.6234247 | 22547.08 | 165787.9 | 6860.25 | 0.9915105 | 1.0089226 | 0.9955388152 | 0.5715799 | 0.6206435 | 0.5961117 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.2208321 | 0.0455002 | 0.2060397 | 4.244537 | 26.84609 | 0.4845103 | 0.4918244 | 22465.16 | 165797.5 | 6778.33 | 0.9915680 | 0.9968748 | 0.9955964622 | 0.4829961 | 0.4896586 | 0.4863274 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.2000708 | 0.0531811 | 0.2658114 | 4.961059 | 34.63408 | 0.5663006 | 0.6345015 | 22570.42 | 165843.8 | 6883.59 | 0.9918449 | 1.0123552 | 0.9958744888 | 0.5732973 | 0.6318839 | 0.6025906 |

# random downsampling of data

``` r
seqstats[sample(nrow(seqstats), 10),]
```

# Cleanup data - set NAs to 0

``` r
# set NAs in numeric columns to 0
seqstats[is.na(is.numeric(seqstats))] <- 0
```

# Tidyverse basics

## `mutate()` can create new column based on others

Example below (code not run) uses multiple tidyverse functions:

``` r
seqstats %>% 
  mutate(total_reads = `bwa hg38 -q 50` + `bwa dm6 -q 50` + `bwa sac3 -q 50`) %>%
  head()
```

    ## # A tibble: 6 × 27
    ##   `experiment ID` `library ID` cell  IP    biorep techrep  ...7 `bwa hg38 -q 50`
    ##   <chr>           <chr>        <chr> <chr>  <dbl>   <dbl> <dbl>            <dbl>
    ## 1 LP78            HelaS3_100s… HeLa… H3K9…      1       1    NA          8446795
    ## 2 LP78            HelaS3_100s… HeLa… H3K9…      1       2    NA          8000672
    ## 3 LP78            HelaS3_100s… HeLa… H3K9…      1       3    NA          7144407
    ## 4 LP78            HelaS3_75sy… HeLa… H3K9…      1       1    NA          9000686
    ## 5 LP78            HelaS3_75sy… HeLa… H3K9…      1       2    NA         10985910
    ## 6 LP78            HelaS3_75sy… HeLa… H3K9…      1       3    NA          8336422
    ## # ℹ 19 more variables: `bwa dm6 -q 50` <dbl>, `bwa sac3 -q 50` <dbl>,
    ## #   `dm6/sac3` <dbl>, `dm6/hg38` <dbl>, `sac3/hg38` <dbl>,
    ## #   `dm6/hg38 IP/input` <dbl>, `sac3/hg38 IP/input` <dbl>,
    ## #   `dm6/hg38 IP/input control norm` <dbl>,
    ## #   `sac3/hg38 IP/input control norm` <dbl>, `dm6 IP signal` <dbl>,
    ## #   `sac3 IP signal` <dbl>, `dm6 eff IP/input` <dbl>,
    ## #   `sac3 eff IP/input` <dbl>, `dm6 eff control norm` <dbl>, …

## select columns

By full name:

``` r
seqstats %>% 
   select(., IP) %>% head()
```

    ## # A tibble: 6 × 1
    ##   IP    
    ##   <chr> 
    ## 1 H3K9ac
    ## 2 H3K9ac
    ## 3 H3K9ac
    ## 4 H3K9ac
    ## 5 H3K9ac
    ## 6 H3K9ac

By partial match with `contains`

``` r
seqstats %>% 
   select(., contains("snr")) %>% head()
```

    ## # A tibble: 6 × 3
    ##   `dm6 normfactor snr adj` `sac3 normfactor snr adj` `dual normfactor snr adj`
    ##                      <dbl>                     <dbl>                     <dbl>
    ## 1                    0.931                     0.926                     0.928
    ## 2                    0.900                     0.914                     0.907
    ## 3                    1.17                      1.16                      1.17 
    ## 4                    0.572                     0.621                     0.596
    ## 5                    0.483                     0.490                     0.486
    ## 6                    0.573                     0.632                     0.603

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

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj | avg_normfactor |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 8446795 | 787637 | 3384998 | 0.2326846 | 0.0932468 | 0.4007435 | 8.252629 | 50.57160 | 0.9420306 | 0.9264793 | 22496.56 | 165939.1 | 6720.68 | 0.9951239 | 0.9883963 | 0.9991668547 | 0.9310996 | 0.9257074 | 0.9284035 | 0.9284035 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 8000672 | 719513 | 3165051 | 0.2273306 | 0.0899316 | 0.3955981 | 7.959218 | 49.92229 | 0.9085379 | 0.9145838 | 22511.03 | 166041.9 | 6735.15 | 0.9957404 | 0.9905244 | 0.9997858429 | 0.8999290 | 0.9143879 | 0.9071584 | 0.9071584 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 7144407 | 812865 | 3581432 | 0.2269665 | 0.1137764 | 0.5012917 | 10.069558 | 63.26023 | 1.1494315 | 1.1589369 | 22718.79 | 166251.4 | 6942.91 | 0.9969967 | 1.0210792 | 1.001047302 | 1.1736607 | 1.1601506 | 1.1669056 | 1.1669056 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 9000686 | 478856 | 2350718 | 0.2037063 | 0.0532022 | 0.2611710 | 4.963025 | 34.02945 | 0.5665250 | 0.6234247 | 22547.08 | 165787.9 | 6860.25 | 0.9915105 | 1.0089226 | 0.9955388152 | 0.5715799 | 0.6206435 | 0.5961117 | 0.5961117 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 10985910 | 499861 | 2263534 | 0.2208321 | 0.0455002 | 0.2060397 | 4.244537 | 26.84609 | 0.4845103 | 0.4918244 | 22465.16 | 165797.5 | 6778.33 | 0.9915680 | 0.9968748 | 0.9955964622 | 0.4829961 | 0.4896586 | 0.4863274 | 0.4863274 |
| LP78 | HelaS3_75sync_25inter_1_H3K9ac_3 | HeLaS3 | H3K9ac | 1 | 3 | NA | 8336422 | 443340 | 2215916 | 0.2000708 | 0.0531811 | 0.2658114 | 4.961059 | 34.63408 | 0.5663006 | 0.6345015 | 22570.42 | 165843.8 | 6883.59 | 0.9918449 | 1.0123552 | 0.9958744888 | 0.5732973 | 0.6318839 | 0.6025906 | 0.6025906 |

## Pair Data

From here:
<https://datavizpyr.com/connect-paired-points-with-lines-in-scatterplot-in-ggplot2/>

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
      cols = c(`bwa hg38 -q 50`, `bwa dm6 -q 50`, `bwa sac3 -q 50`), 
      names_to = "Species", 
      values_to = "Reads")

knitr::kable(head(seqstats_tidy))
```

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj | Species | Reads |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|:---|---:|
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 0.2326846 | 0.0932468 | 0.4007435 | 8.252629 | 50.57160 | 0.9420306 | 0.9264793 | 22496.56 | 165939.1 | 6720.68 | 0.9951239 | 0.9883963 | 0.9991668547 | 0.9310996 | 0.9257074 | 0.9284035 | bwa hg38 -q 50 | 8446795 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 0.2326846 | 0.0932468 | 0.4007435 | 8.252629 | 50.57160 | 0.9420306 | 0.9264793 | 22496.56 | 165939.1 | 6720.68 | 0.9951239 | 0.9883963 | 0.9991668547 | 0.9310996 | 0.9257074 | 0.9284035 | bwa dm6 -q 50 | 787637 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_1 | HeLaS3 | H3K9ac | 1 | 1 | NA | 0.2326846 | 0.0932468 | 0.4007435 | 8.252629 | 50.57160 | 0.9420306 | 0.9264793 | 22496.56 | 165939.1 | 6720.68 | 0.9951239 | 0.9883963 | 0.9991668547 | 0.9310996 | 0.9257074 | 0.9284035 | bwa sac3 -q 50 | 3384998 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 0.2273306 | 0.0899316 | 0.3955981 | 7.959218 | 49.92229 | 0.9085379 | 0.9145838 | 22511.03 | 166041.9 | 6735.15 | 0.9957404 | 0.9905244 | 0.9997858429 | 0.8999290 | 0.9143879 | 0.9071584 | bwa hg38 -q 50 | 8000672 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 0.2273306 | 0.0899316 | 0.3955981 | 7.959218 | 49.92229 | 0.9085379 | 0.9145838 | 22511.03 | 166041.9 | 6735.15 | 0.9957404 | 0.9905244 | 0.9997858429 | 0.8999290 | 0.9143879 | 0.9071584 | bwa dm6 -q 50 | 719513 |
| LP78 | HelaS3_100sync_0inter_1_H3K9ac_2 | HeLaS3 | H3K9ac | 1 | 2 | NA | 0.2273306 | 0.0899316 | 0.3955981 | 7.959218 | 49.92229 | 0.9085379 | 0.9145838 | 22511.03 | 166041.9 | 6735.15 | 0.9957404 | 0.9905244 | 0.9997858429 | 0.8999290 | 0.9143879 | 0.9071584 | bwa sac3 -q 50 | 3165051 |

## Reorder column contents

General Example

Replicate data with `data_new <- data`

Reordering group factor levels

``` r
data_new <- rep(c("A","B", "C", "D"), each = 5)
col2 <- rep(c("12", "10"), each = 10)
data_new$Values <- col2
```

    ## Warning in data_new$Values <- col2: Coercing LHS to a list

``` r
head(data_new)
```

    ## [[1]]
    ## [1] "A"
    ## 
    ## [[2]]
    ## [1] "A"
    ## 
    ## [[3]]
    ## [1] "A"
    ## 
    ## [[4]]
    ## [1] "A"
    ## 
    ## [[5]]
    ## [1] "A"
    ## 
    ## [[6]]
    ## [1] "B"

``` r
data_new$group <- factor(data_new$group,levels = c("B", "A", "C", "D"))
head(data_new)
```

    ## [[1]]
    ## [1] "A"
    ## 
    ## [[2]]
    ## [1] "A"
    ## 
    ## [[3]]
    ## [1] "A"
    ## 
    ## [[4]]
    ## [1] "A"
    ## 
    ## [[5]]
    ## [1] "A"
    ## 
    ## [[6]]
    ## [1] "B"

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

    ##  [1] "yeast_0.1k"    "yeast_0.1k"    "yeast_1k"      "yeast_1k"     
    ##  [5] "yeast_10k"     "yeast_10k"     "yeast_100k"    "yeast_100k"   
    ##  [9] "yeast_1000k"   "yeast_1000k"   "yeast_10000k"  "yeast_10000k" 
    ## [13] "yeast_100000k" "yeast_100000k"

``` r
# List itself repeated
Yeast_times<-rep(c("yeast_0.1k","yeast_1k","yeast_10k","yeast_100k","yeast_1000k", "yeast_10000k", "yeast_100000k"), times=2)
Yeast_times
```

    ##  [1] "yeast_0.1k"    "yeast_1k"      "yeast_10k"     "yeast_100k"   
    ##  [5] "yeast_1000k"   "yeast_10000k"  "yeast_100000k" "yeast_0.1k"   
    ##  [9] "yeast_1k"      "yeast_10k"     "yeast_100k"    "yeast_1000k"  
    ## [13] "yeast_10000k"  "yeast_100000k"

``` r
# Combine for more complex patterns
Yeastnew <-rep(c(Yeast_each), times = 3)
Yeastnew
```

    ##  [1] "yeast_0.1k"    "yeast_0.1k"    "yeast_1k"      "yeast_1k"     
    ##  [5] "yeast_10k"     "yeast_10k"     "yeast_100k"    "yeast_100k"   
    ##  [9] "yeast_1000k"   "yeast_1000k"   "yeast_10000k"  "yeast_10000k" 
    ## [13] "yeast_100000k" "yeast_100000k" "yeast_0.1k"    "yeast_0.1k"   
    ## [17] "yeast_1k"      "yeast_1k"      "yeast_10k"     "yeast_10k"    
    ## [21] "yeast_100k"    "yeast_100k"    "yeast_1000k"   "yeast_1000k"  
    ## [25] "yeast_10000k"  "yeast_10000k"  "yeast_100000k" "yeast_100000k"
    ## [29] "yeast_0.1k"    "yeast_0.1k"    "yeast_1k"      "yeast_1k"     
    ## [33] "yeast_10k"     "yeast_10k"     "yeast_100k"    "yeast_100k"   
    ## [37] "yeast_1000k"   "yeast_1000k"   "yeast_10000k"  "yeast_10000k" 
    ## [41] "yeast_100000k" "yeast_100000k"

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

    ## Warning: The `...` argument of `group_indices()` is deprecated as of dplyr 1.0.0.
    ## ℹ Please `group_by()` first
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

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

# Boolean Logic

- !x = NOT x (everything else)
- x & y = x AND y
- x && y =
- x \| y = x OR y
- x \|\| y =
- xor(x, y) =

Example: get rows whose column `library ID` contents matched the partial
string `DPY30` OR `RBBP5`.

``` r
seqstats[grep("DPY30|RBBP5", seqstats$`library ID`), ] %>% head(n = 10) %>% knitr::kable()
```

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP101 | LP101_DPY30mAID_IAA_0hr_1_H3K4me3_1 | mESC-Dpy30mAID | H3K4me3 | 1 | 1 | 8701023 | NA | 2244620 | 140474 | 0.2579720 | 0.0161445 | 82.48645 | 6.900959 | 1.000000 | 1.0000000 | 60691.44 | 264642.7 | 3.310585 | 1.358330 | 1.0000000 | 1.0000000 | 1.0 | 1.0000000 | 1.000000 | NA |
| LP101 | LP101_DPY30mAID_IAA_0hr_1_input_1 | mESC-Dpy30mAID | NA | 1 | 1 | 14835269 | NA | 45652 | 33690 | 0.0030773 | 0.0022709 | NA | NA | NA | NA | 18332.54 | 194829.5 | NA | NA | NA | NA | NA | NA | NA | NA |
| LP101 | LP101_DPY30mAID_IAA_0hr_1_input_2 | mESC-Dpy30mAID | NA | 1 | 2 | 19515475 | NA | 62013 | 46993 | 0.0031776 | 0.0024080 | NA | NA | NA | NA | 18071.94 | 193999.7 | NA | NA | NA | NA | NA | NA | NA | NA |
| LP101 | LP101_DPY30mAID_IAA_24hr_1_H3K4me3_1 | mESC-Dpy30mAID | H3K4me3 | 1 | 1 | 11356645 | NA | 2929585 | 146334 | 0.2579622 | 0.0128853 | 116.79127 | 5.486450 | 1.415884 | 0.7950272 | 62918.10 | 267300.0 | 3.425070 | 1.378007 | 1.0345815 | 1.0144863 | 1.464847756504565 | 0.8065443 | 1.135696 | NA |
| LP101 | LP101_DPY30mAID_IAA_24hr_1_H3K4me3_2 | mESC-Dpy30mAID | H3K4me3 | 1 | 2 | 9478740 | NA | 2404464 | 119671 | 0.2536692 | 0.0126252 | 114.84761 | 5.375694 | 1.392321 | 0.7789779 | 62518.37 | 267515.4 | 3.328545 | 1.385352 | 1.0054250 | 1.0198939 | 1.3998743927569224 | 0.7944748 | 1.097175 | NA |
| LP101 | LP101_DPY30mAID_IAA_24hr_1_input_1 | mESC-Dpy30mAID | NA | 1 | 1 | 11939641 | NA | 26745 | 26761 | 0.0022400 | 0.0022414 | NA | NA | NA | NA | 18369.87 | 193975.8 | NA | NA | NA | NA | NA | NA | NA | NA |
| LP101 | LP101_DPY30mAID_IAA_24hr_1_input_2 | mESC-Dpy30mAID | NA | 1 | 2 | 12097963 | NA | 26343 | 29710 | 0.0021775 | 0.0024558 | NA | NA | NA | NA | 18782.49 | 193102.8 | NA | NA | NA | NA | NA | NA | NA | NA |
| LP101 | LP101_RBBP5FKBP_dTAG13_0hr_1_H3K4me3_1 | mESC-RBBP5FKBP | H3K4me3 | 1 | 1 | 8759236 | NA | 1145797 | 88874 | 0.1308102 | 0.0101463 | 49.30893 | 7.368531 | 1.000000 | 1.0000000 | 64402.67 | 270471.3 | 3.500723 | 1.388588 | 1.0000000 | 1.0000000 | 1.0 | 1.0000000 | 1.000000 | NA |
| LP101 | LP101_RBBP5FKBP_dTAG13_0hr_1_input_1 | mESC-RBBP5FKBP | NA | 1 | 1 | 12642161 | NA | 33538 | 17408 | 0.0026529 | 0.0013770 | NA | NA | NA | NA | 18396.96 | 194781.6 | NA | NA | NA | NA | NA | NA | NA | NA |
| LP101 | LP101_RBBP5FKBP_dTAG13_24hr_1_H3K4me3_1 | mESC-RBBP5FKBP | H3K4me3 | 1 | 1 | 6110805 | NA | 2864394 | 123819 | 0.4687425 | 0.0202623 | 182.00847 | 10.756930 | 3.691187 | 1.4598472 | 62839.92 | 268884.4 | 3.392374 | 1.380411 | 0.9941119 | 0.9941119 | 3.669452561443117 | 1.4512515 | 2.560352 | NA |

For AND matches, we need to modify the notation, as
`grep("DMSO&Rpb1", ...)` will only find a match if there is a library ID
that contains the continuous string “DMSO Rpb1” in that exact order.
Also, `grep` will not work, we need `grepl`. We need to do:

``` r
seqstats[grepl("DMSO", seqstats$`library ID`) & grepl("Rpb1", seqstats$`library ID`), ] %>%
head() %>% knitr::kable()
```

| experiment ID | library ID | cell | IP | biorep | techrep | …7 | bwa hg38 -q 50 | bwa dm6 -q 50 | bwa sac3 -q 50 | dm6/sac3 | dm6/hg38 | sac3/hg38 | dm6/hg38 IP/input | sac3/hg38 IP/input | dm6/hg38 IP/input control norm | sac3/hg38 IP/input control norm | dm6 IP signal | sac3 IP signal | dm6 eff IP/input | sac3 eff IP/input | dm6 eff control norm | sac3 eff control norm | dm6 normfactor snr adj | sac3 normfactor snr adj | dual normfactor snr adj |
|:---|:---|:---|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|:---|---:|---:|---:|
| LP88 | HCT116_DMSO_0hr_1_Rpb1_1 | HCT116-Rpb1 | H3K4me3 | 1 | 1 | NA | 23252966 | 52648 | 2848 | 18.48596 | 0.0022641 | 0.0001225 | 0.3201269 | 0.0480709 | 1.0000000 | NA | 69.05311 | NA | 1.533166 | NA | 1.0000000 | NA | 1.0000000 | NA | 1.0000000 |
| LP91 | HCT116_DMSO_0hr_2_Rpb1_1 | HCT116-Rpb1 | Rpb1 | 2 | 1 | NA | 8057207 | 8906 | 827 | 10.76904 | 0.0011053 | 0.0001026 | 0.0664754 | 0.0773184 | 0.7285511 | 0.7578813 | 6669.26100 | NA | 1.031603 | NA | 1.0125172 | NA | 0.7376706 | NA | 0.7376706 |
| LP91 | HCT116_DMSO_0hr_2_Rpb1_2 | HCT116-Rpb1 | Rpb1 | 2 | 2 | NA | 7953774 | 15343 | 1338 | 11.46712 | 0.0019290 | 0.0001682 | 0.1160112 | 0.1267198 | 1.2714489 | 1.2421187 | 6504.36400 | NA | 1.006096 | NA | 0.9874828 | NA | 1.2555339 | NA | 1.2555339 |
| LP95 | HCT116_DMSO_0hr_3_Rpb1_1 | HCT116-Rpb1 | Rpb1 | 3 | 1 | NA | 4807836 | 67749 | 1088 | 62.26930 | 0.0140914 | 0.0002263 | 1.4091118 | 0.0421450 | 0.9293630 | 1.3447684 | 8170.40000 | NA | 1.278321 | NA | 1.0677682 | NA | 0.9923443 | NA | 0.9923443 |
| LP95 | HCT116_DMSO_0hr_4_Rpb1_1 | HCT116-Rpb1 | Rpb1 | 4 | 1 | NA | 4852590 | 110428 | 712 | 155.09551 | 0.0227565 | 0.0001467 | 1.6233133 | 0.0205349 | 1.0706370 | 0.6552316 | 7270.40200 | NA | 1.116058 | NA | 0.9261240 | NA | 0.9915426 | NA | 0.9915426 |
| LP85 | bias_HCT116_DMSO_0hr_Rpb1_1 | HCT116-Rpb1 | Rpb1 | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA | NA |

# Special cases

## Convert Time Series to Dataframe with `as.matrix()` and `data.frame()`

Example from seatbelts dataset: need to add a new column “date” and
convert the time series to a table with the year in “date”

``` r
data("Seatbelts")
seatbelts_df <- data.frame(as.matrix(Seatbelts), date = time(Seatbelts))
head(seatbelts_df)
```

    ##   DriversKilled drivers front rear   kms PetrolPrice VanKilled law     date
    ## 1           107    1687   867  269  9059   0.1029718        12   0 1969.000
    ## 2            97    1508   825  265  7685   0.1023630         6   0 1969.083
    ## 3           102    1507   806  319  9963   0.1020625        12   0 1969.167
    ## 4            87    1385   814  407 10955   0.1008733         8   0 1969.250
    ## 5           119    1632   991  454 11823   0.1010197        10   0 1969.333
    ## 6           106    1511   945  427 12391   0.1005812        13   0 1969.417
