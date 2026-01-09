# R_tidyverse_cheatsheet


- [`grep()` and `grepl()`](#grep-and-grepl)
  - [Search and return row of
    dataframe](#search-and-return-row-of-dataframe)
  - [Multiple matches with logical
    operators](#multiple-matches-with-logical-operators)
- [`select()` to grab whole columns](#select-to-grab-whole-columns)
  - [`select()` with exact match](#select-with-exact-match)
  - [`select()` with partial match](#select-with-partial-match)
  - [select columns that match multiple
    arguments](#select-columns-that-match-multiple-arguments)
- [select entries in rows](#select-entries-in-rows)
  - [reverse select rows](#reverse-select-rows)
- [`filter()`](#filter)
  - [filter to select one instance within
    column](#filter-to-select-one-instance-within-column)
  - [filter based on math/other
    logicals](#filter-based-on-mathother-logicals)
  - [filter/grepl to get a partial
    match](#filtergrepl-to-get-a-partial-match)
  - [`filter()` reverse of selection](#filter-reverse-of-selection)
  - [`filter()` based on numeric data in
    columns](#filter-based-on-numeric-data-in-columns)
  - [Filter multiple matches](#filter-multiple-matches)
- [`mutate()`](#mutate)
- [Tidy(Long) formatted Data](#tidylong-formatted-data)
  - [Select columns by name](#select-columns-by-name)
  - [Select “everything but” columns](#select-everything-but-columns)
  - [Select columns using tidy helper
    functions](#select-columns-using-tidy-helper-functions)
  - [pivot and separate column into
    multiple](#pivot-and-separate-column-into-multiple)
- [Add column to number groups](#add-column-to-number-groups)
- [Change column names](#change-column-names)
- [Change names within columns](#change-names-within-columns)
  - [Change data in specific column by
    name](#change-data-in-specific-column-by-name)
- [Split verbose column into multiple with
  `separate_wider_regex()`](#split-verbose-column-into-multiple-with-separate_wider_regex)
- [groups](#groups)
  - [apply `slice_` functions to
    groups](#apply-slice_-functions-to-groups)
  - [apply function to each group with
    `group_map()`](#apply-function-to-each-group-with-group_map)
    - [Function to calculate variation in inputs within each
      paper](#function-to-calculate-variation-in-inputs-within-each-paper)
- [change column order](#change-column-order)
- [String manipulation](#string-manipulation)
  - [`str_remove` to remove part of a
    string](#str_remove-to-remove-part-of-a-string)
  - [`str_match` to keep part of a string based on regex
    pattern](#str_match-to-keep-part-of-a-string-based-on-regex-pattern)
  - [`str_extract` to keep part of a
    string](#str_extract-to-keep-part-of-a-string)

``` r
library(gapminder)
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

# `grep()` and `grepl()`

**`grep()`** returns indices of matched items

In the below code we are searching for:

1.  the string “bania” (only 1 partial match, Albania)
2.  within the gapminder dataset
3.  within this dataset, the country column

``` r
grep("bania", gapminder$country)
```

     [1] 13 14 15 16 17 18 19 20 21 22 23 24

**`grepl()`** is very similar, but instead of returining indices of the
matches location, it returns a logical vector, with TRUE representing a
match, and FALSE if not a match. Importantly, this will always return a
vector equal to the length of the searched column

The same example with grepl:

``` r
length(grepl("bania", gapminder$country))
```

    [1] 1704

## Search and return row of dataframe

Syntax: `dataframe[grep("string", dataframe$column), ]`

Use indexes to return row with match: works with `grep` and `grepl`

``` r
gapminder[grep("bania", gapminder$country), ] %>% knitr::kable()
```

| country | continent | year | lifeExp |     pop | gdpPercap |
|:--------|:----------|-----:|--------:|--------:|----------:|
| Albania | Europe    | 1952 |  55.230 | 1282697 |  1601.056 |
| Albania | Europe    | 1957 |  59.280 | 1476505 |  1942.284 |
| Albania | Europe    | 1962 |  64.820 | 1728137 |  2312.889 |
| Albania | Europe    | 1967 |  66.220 | 1984060 |  2760.197 |
| Albania | Europe    | 1972 |  67.690 | 2263554 |  3313.422 |
| Albania | Europe    | 1977 |  68.930 | 2509048 |  3533.004 |
| Albania | Europe    | 1982 |  70.420 | 2780097 |  3630.881 |
| Albania | Europe    | 1987 |  72.000 | 3075321 |  3738.933 |
| Albania | Europe    | 1992 |  71.581 | 3326498 |  2497.438 |
| Albania | Europe    | 1997 |  72.950 | 3428038 |  3193.055 |
| Albania | Europe    | 2002 |  75.651 | 3508512 |  4604.212 |
| Albania | Europe    | 2007 |  76.423 | 3600523 |  5937.030 |

## Multiple matches with logical operators

“bania” will only match “Albania”, but “geria” will match “Nigeria” and
“Algeria”

``` r
gapminder[grep("bania|geria", gapminder$country), ] %>% knitr::kable()
```

| country | continent | year | lifeExp |       pop | gdpPercap |
|:--------|:----------|-----:|--------:|----------:|----------:|
| Albania | Europe    | 1952 |  55.230 |   1282697 |  1601.056 |
| Albania | Europe    | 1957 |  59.280 |   1476505 |  1942.284 |
| Albania | Europe    | 1962 |  64.820 |   1728137 |  2312.889 |
| Albania | Europe    | 1967 |  66.220 |   1984060 |  2760.197 |
| Albania | Europe    | 1972 |  67.690 |   2263554 |  3313.422 |
| Albania | Europe    | 1977 |  68.930 |   2509048 |  3533.004 |
| Albania | Europe    | 1982 |  70.420 |   2780097 |  3630.881 |
| Albania | Europe    | 1987 |  72.000 |   3075321 |  3738.933 |
| Albania | Europe    | 1992 |  71.581 |   3326498 |  2497.438 |
| Albania | Europe    | 1997 |  72.950 |   3428038 |  3193.055 |
| Albania | Europe    | 2002 |  75.651 |   3508512 |  4604.212 |
| Albania | Europe    | 2007 |  76.423 |   3600523 |  5937.030 |
| Algeria | Africa    | 1952 |  43.077 |   9279525 |  2449.008 |
| Algeria | Africa    | 1957 |  45.685 |  10270856 |  3013.976 |
| Algeria | Africa    | 1962 |  48.303 |  11000948 |  2550.817 |
| Algeria | Africa    | 1967 |  51.407 |  12760499 |  3246.992 |
| Algeria | Africa    | 1972 |  54.518 |  14760787 |  4182.664 |
| Algeria | Africa    | 1977 |  58.014 |  17152804 |  4910.417 |
| Algeria | Africa    | 1982 |  61.368 |  20033753 |  5745.160 |
| Algeria | Africa    | 1987 |  65.799 |  23254956 |  5681.359 |
| Algeria | Africa    | 1992 |  67.744 |  26298373 |  5023.217 |
| Algeria | Africa    | 1997 |  69.152 |  29072015 |  4797.295 |
| Algeria | Africa    | 2002 |  70.994 |  31287142 |  5288.040 |
| Algeria | Africa    | 2007 |  72.301 |  33333216 |  6223.367 |
| Nigeria | Africa    | 1952 |  36.324 |  33119096 |  1077.282 |
| Nigeria | Africa    | 1957 |  37.802 |  37173340 |  1100.593 |
| Nigeria | Africa    | 1962 |  39.360 |  41871351 |  1150.927 |
| Nigeria | Africa    | 1967 |  41.040 |  47287752 |  1014.514 |
| Nigeria | Africa    | 1972 |  42.821 |  53740085 |  1698.389 |
| Nigeria | Africa    | 1977 |  44.514 |  62209173 |  1981.952 |
| Nigeria | Africa    | 1982 |  45.826 |  73039376 |  1576.974 |
| Nigeria | Africa    | 1987 |  46.886 |  81551520 |  1385.030 |
| Nigeria | Africa    | 1992 |  47.472 |  93364244 |  1619.848 |
| Nigeria | Africa    | 1997 |  47.464 | 106207839 |  1624.941 |
| Nigeria | Africa    | 2002 |  46.608 | 119901274 |  1615.286 |
| Nigeria | Africa    | 2007 |  46.859 | 135031164 |  2013.977 |

# `select()` to grab whole columns

## `select()` with exact match

``` r
gapminder_select <- gapminder %>% 
  # pick two years in dataset
  filter(year %in% c(1952, 2007)) %>%
  # only countries in Asia
  filter(continent %in% c("Asia")) %>%
  # select certain columns
  select(country,year,lifeExp, gdpPercap) 
knitr::kable(head(gapminder_select))
```

| country     | year | lifeExp |  gdpPercap |
|:------------|-----:|--------:|-----------:|
| Afghanistan | 1952 |  28.801 |   779.4453 |
| Afghanistan | 2007 |  43.828 |   974.5803 |
| Bahrain     | 1952 |  50.939 |  9867.0848 |
| Bahrain     | 2007 |  75.635 | 29796.0483 |
| Bangladesh  | 1952 |  37.484 |   684.2442 |
| Bangladesh  | 2007 |  64.062 |  1391.2538 |

## `select()` with partial match

Syntax: `select(dataframe, logical_expression)`

- dataframe can be piped in as shown below or defined explicityly

logical expressions: `dplyr` has lots of helpful functions for string
matching. Examples:

- `starts_with()`
- `ends_with()`
- `contains()`
- `matches()`: works with regular expressions (above functions do not)

Each example below would usually select entire columns that match the
logical operator, but I only show 6 rows here (with head())

``` r
gapminder %>% select(., ends_with("p")) %>% 
  head() %>% knitr::kable()
```

| lifeExp |      pop | gdpPercap |
|--------:|---------:|----------:|
|  28.801 |  8425333 |  779.4453 |
|  30.332 |  9240934 |  820.8530 |
|  31.997 | 10267083 |  853.1007 |
|  34.020 | 11537966 |  836.1971 |
|  36.088 | 13079460 |  739.9811 |
|  38.438 | 14880372 |  786.1134 |

``` r
gapminder %>% select(., contains("nt")) %>%
  head() %>% knitr::kable()
```

| country     | continent |
|:------------|:----------|
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |

``` r
gapminder %>% select(., starts_with("co")) %>% 
  head() %>% knitr::kable()
```

| country     | continent |
|:------------|:----------|
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |

## select columns that match multiple arguments

Select OR `|` operator

``` r
gapminder %>% select(., matches('co') | matches('try')) %>% head() %>% knitr::kable()
```

| country     | continent |
|:------------|:----------|
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |
| Afghanistan | Asia      |

Select AND `&` operator

``` r
gapminder %>% select(., matches('co') & matches('try')) %>% head() %>% knitr::kable()
```

| country     |
|:------------|
| Afghanistan |
| Afghanistan |
| Afghanistan |
| Afghanistan |
| Afghanistan |
| Afghanistan |

# select entries in rows

## reverse select rows

``` r
gapminder_select <-
  gapminder[!grepl("19", gapminder$year), ]

knitr::kable(head(gapminder_select))
```

| country     | continent | year | lifeExp |      pop | gdpPercap |
|:------------|:----------|-----:|--------:|---------:|----------:|
| Afghanistan | Asia      | 2002 |  42.129 | 25268405 |  726.7341 |
| Afghanistan | Asia      | 2007 |  43.828 | 31889923 |  974.5803 |
| Albania     | Europe    | 2002 |  75.651 |  3508512 | 4604.2117 |
| Albania     | Europe    | 2007 |  76.423 |  3600523 | 5937.0295 |
| Algeria     | Africa    | 2002 |  70.994 | 31287142 | 5288.0404 |
| Algeria     | Africa    | 2007 |  72.301 | 33333216 | 6223.3675 |

# `filter()`

## filter to select one instance within column

1.  only keep data from years 1952 and 2007 in column “Year”
2.  data from Asia in the column “Continent”

``` r
gapminder %>% 
  filter(year %in% c(1952, 2007)) %>% 
  filter(continent == "Asia") %>% 
  head() %>% knitr::kable()
```

| country     | continent | year | lifeExp |       pop |  gdpPercap |
|:------------|:----------|-----:|--------:|----------:|-----------:|
| Afghanistan | Asia      | 1952 |  28.801 |   8425333 |   779.4453 |
| Afghanistan | Asia      | 2007 |  43.828 |  31889923 |   974.5803 |
| Bahrain     | Asia      | 1952 |  50.939 |    120447 |  9867.0848 |
| Bahrain     | Asia      | 2007 |  75.635 |    708573 | 29796.0483 |
| Bangladesh  | Asia      | 1952 |  37.484 |  46886859 |   684.2442 |
| Bangladesh  | Asia      | 2007 |  64.062 | 150448339 |  1391.2538 |

## filter based on math/other logicals

``` r
gapminder %>%
  filter(lifeExp > 80) %>%
  knitr::kable()
```

| country          | continent | year | lifeExp |       pop | gdpPercap |
|:-----------------|:----------|-----:|--------:|----------:|----------:|
| Australia        | Oceania   | 2002 |  80.370 |  19546792 |  30687.75 |
| Australia        | Oceania   | 2007 |  81.235 |  20434176 |  34435.37 |
| Canada           | Americas  | 2007 |  80.653 |  33390141 |  36319.24 |
| France           | Europe    | 2007 |  80.657 |  61083916 |  30470.02 |
| Hong Kong, China | Asia      | 2002 |  81.495 |   6762476 |  30209.02 |
| Hong Kong, China | Asia      | 2007 |  82.208 |   6980412 |  39724.98 |
| Iceland          | Europe    | 2002 |  80.500 |    288030 |  31163.20 |
| Iceland          | Europe    | 2007 |  81.757 |    301931 |  36180.79 |
| Israel           | Asia      | 2007 |  80.745 |   6426679 |  25523.28 |
| Italy            | Europe    | 2002 |  80.240 |  57926999 |  27968.10 |
| Italy            | Europe    | 2007 |  80.546 |  58147733 |  28569.72 |
| Japan            | Asia      | 1997 |  80.690 | 125956499 |  28816.58 |
| Japan            | Asia      | 2002 |  82.000 | 127065841 |  28604.59 |
| Japan            | Asia      | 2007 |  82.603 | 127467972 |  31656.07 |
| New Zealand      | Oceania   | 2007 |  80.204 |   4115771 |  25185.01 |
| Norway           | Europe    | 2007 |  80.196 |   4627926 |  49357.19 |
| Spain            | Europe    | 2007 |  80.941 |  40448191 |  28821.06 |
| Sweden           | Europe    | 2002 |  80.040 |   8954175 |  29341.63 |
| Sweden           | Europe    | 2007 |  80.884 |   9031088 |  33859.75 |
| Switzerland      | Europe    | 2002 |  80.620 |   7361757 |  34480.96 |
| Switzerland      | Europe    | 2007 |  81.701 |   7554661 |  37506.42 |

## filter/grepl to get a partial match

`filter()` works with multiple logical operators, including `grepl()`.
The format is `filter(grepl("string", column_to_search))`. The string to
match can be a normal string or some combination with regex. For
example, below uses a string `^A` which indicates the string must start
with capital A.

``` r
gapminder %>% 
  filter(year %in% c(1952, 2007)) %>% 
  filter(grepl("^A", continent)) %>% 
  head() %>% knitr::kable()
```

| country     | continent | year | lifeExp |      pop | gdpPercap |
|:------------|:----------|-----:|--------:|---------:|----------:|
| Afghanistan | Asia      | 1952 |  28.801 |  8425333 |  779.4453 |
| Afghanistan | Asia      | 2007 |  43.828 | 31889923 |  974.5803 |
| Algeria     | Africa    | 1952 |  43.077 |  9279525 | 2449.0082 |
| Algeria     | Africa    | 2007 |  72.301 | 33333216 | 6223.3675 |
| Angola      | Africa    | 1952 |  30.015 |  4232095 | 3520.6103 |
| Angola      | Africa    | 2007 |  42.731 | 12420476 | 4797.2313 |

## `filter()` reverse of selection

%\>% filter(!Annotation %in% c(“Intergenic”, “intron”))

## `filter()` based on numeric data in columns

Remove rows with zeros in *at least* one of three columns:

``` r
counts_BRD4_HA_nozeros <- counts_BRD4_HA_tidy %>% 
  filter(HA_avg_5 != 0) %>% 
  filter(BRD4_avg_5 != 0) %>%
  filter(HA_avg_2.5 != 0) 
```

Remove rows with zeros in *all* columns selected

``` r
counts_BRD4_HA_nozeros <- counts_BRD4_HA_tidy %>%
  rowwise() %>%
  filter(sum(c(HA_avg_5, BRD4_avg_5, HA_avg_2.5)) != 0)
```

## Filter multiple matches

``` r
 LP88_LP91_seqstats %>%
  filter(grepl(".comb", ID) | grepl("hr_1", ID))

# OR
LP88_LP91_seqstats %>%
  filter(grepl(".comb|hr_1", ID))
```

Filter/grepl with special character

``` r
Sample <- c("HelaS3_0sync_100inter_1_H3K9ac_1.fly.normalized.tagdir.Coverage", 
            "HelaS3_0sync_100inter_1_H3K9ac_1.yeast.normalized.tagdir.Coverage", 
            "HelaS3_0sync_100inter_1_H3K9ac_1.dual.normalized.tagdir.Coverage", 
            "HelaS3_0sync_100inter_1_H3K9ac_1.notnorm.tagdir.Coverage")

data.frame(Sample) %>% filter(grepl("_..normalized", Sample))
```

    [1] Sample
    <0 rows> (or 0-length row.names)

Here the `_` is read as is, but the `.` can mean any one character, so
this matches:

HelaS3_0sync_100inter_1_H3K9ac_1.normalized.tagdir.Coverage <br>
HelaS3_0sync_100inter_1_H3K9ac_2.normalized.tagdir.Coverage <br>
HelaS3_0sync_100inter_1_H3K9ac_3.normalized.tagdir.Coverage <br>

etc.

Take another scenario:

``` r
data.frame(Sample) %>% filter(grepl(".[[:alpha:]]+.normalized", Sample)) 
```

                                                                 Sample
    1   HelaS3_0sync_100inter_1_H3K9ac_1.fly.normalized.tagdir.Coverage
    2 HelaS3_0sync_100inter_1_H3K9ac_1.yeast.normalized.tagdir.Coverage
    3  HelaS3_0sync_100inter_1_H3K9ac_1.dual.normalized.tagdir.Coverage

``` r
data.frame(Sample) %>%
  mutate(
    # detect normalization method
    method = case_when(
      str_detect(Sample, "\\.fly\\.normalized\\.tagdir\\.Coverage$")   ~ "flynorm",
      str_detect(Sample, "\\.yeast\\.normalized\\.tagdir\\.Coverage$") ~ "yeastnorm",
      str_detect(Sample, "\\.normalized\\.tagdir\\.Coverage$")         ~ "norm",
      TRUE ~ "other"
    ),
    
    # remove suffixes to get the base sample name
    base_sample = Sample %>%
      str_remove("\\.fly\\.normalized\\.tagdir\\.Coverage$") %>%
      str_remove("\\.yeast\\.normalized\\.tagdir\\.Coverage$") %>%
      str_remove("\\.dual\\.normalized\\.tagdir\\.Coverage$"),
  ) %>% knitr::kable()
```

| Sample | method | base_sample |
|:---|:---|:---|
| HelaS3_0sync_100inter_1_H3K9ac_1.fly.normalized.tagdir.Coverage | flynorm | HelaS3_0sync_100inter_1_H3K9ac_1 |
| HelaS3_0sync_100inter_1_H3K9ac_1.yeast.normalized.tagdir.Coverage | yeastnorm | HelaS3_0sync_100inter_1_H3K9ac_1 |
| HelaS3_0sync_100inter_1_H3K9ac_1.dual.normalized.tagdir.Coverage | norm | HelaS3_0sync_100inter_1_H3K9ac_1 |
| HelaS3_0sync_100inter_1_H3K9ac_1.notnorm.tagdir.Coverage | other | HelaS3_0sync_100inter_1_H3K9ac_1.notnorm.tagdir.Coverage |

# `mutate()`

# Tidy(Long) formatted Data

Tidy format: Instead of multiple columns with different variables, data
is stored with one column recording each instance of a variable, and a
second column recording all values. Helpful to group variables that all
use the same measurement/metrics.

Before example with the iris dataset:

``` r
knitr::kable(head(iris))
```

| Sepal.Length | Sepal.Width | Petal.Length | Petal.Width | Species |
|-------------:|------------:|-------------:|------------:|:--------|
|          5.1 |         3.5 |          1.4 |         0.2 | setosa  |
|          4.9 |         3.0 |          1.4 |         0.2 | setosa  |
|          4.7 |         3.2 |          1.3 |         0.2 | setosa  |
|          4.6 |         3.1 |          1.5 |         0.2 | setosa  |
|          5.0 |         3.6 |          1.4 |         0.2 | setosa  |
|          5.4 |         3.9 |          1.7 |         0.4 | setosa  |

`pivot_longer()` requires a few arguments:

- cols: the columns to transform/combine
- names_to: the names from each column (variables) will all be moved to
  entries of one column, with the specified name
- values_to: all values within the input columns are stored in this
  output column

## Select columns by name

``` r
iris %>% 
    pivot_longer(cols = c("Sepal.Length", "Sepal.Width"), 
                 names_to = "Sepal_Measures", 
                 values_to = "Values") %>%
  head() %>% knitr::kable()
```

| Petal.Length | Petal.Width | Species | Sepal_Measures | Values |
|-------------:|------------:|:--------|:---------------|-------:|
|          1.4 |         0.2 | setosa  | Sepal.Length   |    5.1 |
|          1.4 |         0.2 | setosa  | Sepal.Width    |    3.5 |
|          1.4 |         0.2 | setosa  | Sepal.Length   |    4.9 |
|          1.4 |         0.2 | setosa  | Sepal.Width    |    3.0 |
|          1.3 |         0.2 | setosa  | Sepal.Length   |    4.7 |
|          1.3 |         0.2 | setosa  | Sepal.Width    |    3.2 |

## Select “everything but” columns

Below pivots every column except for the Species column (as this column
is a different class than the rest).

``` r
iris %>% 
    pivot_longer(cols = !Species, 
                 names_to = "All_Measures", 
                 values_to = "Values") %>%
  head() %>% knitr::kable()
```

| Species | All_Measures | Values |
|:--------|:-------------|-------:|
| setosa  | Sepal.Length |    5.1 |
| setosa  | Sepal.Width  |    3.5 |
| setosa  | Petal.Length |    1.4 |
| setosa  | Petal.Width  |    0.2 |
| setosa  | Sepal.Length |    4.9 |
| setosa  | Sepal.Width  |    3.0 |

## Select columns using tidy helper functions

See select() with partial match to see helper functions.

``` r
iris %>% 
    pivot_longer(cols = starts_with("Petal"), 
                 names_to = "Petal_Measures", 
                 values_to = "Values") %>%
  head() %>% knitr::kable()
```

| Sepal.Length | Sepal.Width | Species | Petal_Measures | Values |
|-------------:|------------:|:--------|:---------------|-------:|
|          5.1 |         3.5 | setosa  | Petal.Length   |    1.4 |
|          5.1 |         3.5 | setosa  | Petal.Width    |    0.2 |
|          4.9 |         3.0 | setosa  | Petal.Length   |    1.4 |
|          4.9 |         3.0 | setosa  | Petal.Width    |    0.2 |
|          4.7 |         3.2 | setosa  | Petal.Length   |    1.3 |
|          4.7 |         3.2 | setosa  | Petal.Width    |    0.2 |

## pivot and separate column into multiple

# Add column to number groups

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

# Change column names

Goal: Replace any slashes “/” with \_ in column names using
`rename_with` (from tidyverse). I do this because the slashes being a
special character could cause issues later.

Code: `rename_with(~ gsub("/", "_", .x), contains("/"))`

# Change names within columns

## Change data in specific column by name

Example changed the sample names to remove the experiment number at the
beginning, as ggplot can’t take a number at the beginning of a variable.

``` r
seqstats_LP69_expanded$id <- gsub("69", "", as.character(seqstats_LP69_expanded$id))
knitr::kable(head(seqstats_LP69_expanded))
```

# Split verbose column into multiple with `separate_wider_regex()`

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

Regex:

- `^` = anchor beginning of string
- `$` = anchor end of the string
- `.` = matches any character but newline
- `[[:alpha:]]` = letters
- `[[:digit:]]` = digits
- `[[:alnum:]]` = letters and numbers
- `[[:graph:]]` = any character but not white space/blank

Special: `+` goes after a match, means it can be counted more than once!
Useful instead of denoting `[:digit:][:digit:]` for a two digit number

# groups

`group_by()` groups dataframe rows by values in a specified column.
Doesn’t otherwise change the dataframe, but you will see how many groups
there are in the printed tibble.

``` r
gapminder |> 
  group_by(year) |> 
  head()
```

    # A tibble: 6 × 6
    # Groups:   year [6]
      country     continent  year lifeExp      pop gdpPercap
      <fct>       <fct>     <int>   <dbl>    <int>     <dbl>
    1 Afghanistan Asia       1952    28.8  8425333      779.
    2 Afghanistan Asia       1957    30.3  9240934      821.
    3 Afghanistan Asia       1962    32.0 10267083      853.
    4 Afghanistan Asia       1967    34.0 11537966      836.
    5 Afghanistan Asia       1972    36.1 13079460      740.
    6 Afghanistan Asia       1977    38.4 14880372      786.

## apply `slice_` functions to groups

Extract specific rows from each group:

1.  `slice_head()` gives you the first entry from each group:

``` r
gapminder |> 
  group_by(year) |> 
  slice_head(n = 1) 
```

    # A tibble: 12 × 6
    # Groups:   year [12]
       country     continent  year lifeExp      pop gdpPercap
       <fct>       <fct>     <int>   <dbl>    <int>     <dbl>
     1 Afghanistan Asia       1952    28.8  8425333      779.
     2 Afghanistan Asia       1957    30.3  9240934      821.
     3 Afghanistan Asia       1962    32.0 10267083      853.
     4 Afghanistan Asia       1967    34.0 11537966      836.
     5 Afghanistan Asia       1972    36.1 13079460      740.
     6 Afghanistan Asia       1977    38.4 14880372      786.
     7 Afghanistan Asia       1982    39.9 12881816      978.
     8 Afghanistan Asia       1987    40.8 13867957      852.
     9 Afghanistan Asia       1992    41.7 16317921      649.
    10 Afghanistan Asia       1997    41.8 22227415      635.
    11 Afghanistan Asia       2002    42.1 25268405      727.
    12 Afghanistan Asia       2007    43.8 31889923      975.

2.  `slice_tail()` gives you the last entry from each group (in original
    df order)

``` r
gapminder |> 
  group_by(year) |> 
  slice_tail(n = 1) 
```

    # A tibble: 12 × 6
    # Groups:   year [12]
       country  continent  year lifeExp      pop gdpPercap
       <fct>    <fct>     <int>   <dbl>    <int>     <dbl>
     1 Zimbabwe Africa     1952    48.5  3080907      407.
     2 Zimbabwe Africa     1957    50.5  3646340      519.
     3 Zimbabwe Africa     1962    52.4  4277736      527.
     4 Zimbabwe Africa     1967    54.0  4995432      570.
     5 Zimbabwe Africa     1972    55.6  5861135      799.
     6 Zimbabwe Africa     1977    57.7  6642107      686.
     7 Zimbabwe Africa     1982    60.4  7636524      789.
     8 Zimbabwe Africa     1987    62.4  9216418      706.
     9 Zimbabwe Africa     1992    60.4 10704340      693.
    10 Zimbabwe Africa     1997    46.8 11404948      792.
    11 Zimbabwe Africa     2002    40.0 11926563      672.
    12 Zimbabwe Africa     2007    43.5 12311143      470.

3.  `slice_min(column, n = 1)` gives you the minimum value of the column
    specified (must by numeric) for each group

``` r
gapminder |> 
  group_by(year) |> 
  slice_min(pop, n = 1)
```

    # A tibble: 12 × 6
    # Groups:   year [12]
       country               continent  year lifeExp    pop gdpPercap
       <fct>                 <fct>     <int>   <dbl>  <int>     <dbl>
     1 Sao Tome and Principe Africa     1952    46.5  60011      880.
     2 Sao Tome and Principe Africa     1957    48.9  61325      861.
     3 Sao Tome and Principe Africa     1962    51.9  65345     1072.
     4 Sao Tome and Principe Africa     1967    54.4  70787     1385.
     5 Sao Tome and Principe Africa     1972    56.5  76595     1533.
     6 Sao Tome and Principe Africa     1977    58.6  86796     1738.
     7 Sao Tome and Principe Africa     1982    60.4  98593     1890.
     8 Sao Tome and Principe Africa     1987    61.7 110812     1517.
     9 Sao Tome and Principe Africa     1992    62.7 125911     1429.
    10 Sao Tome and Principe Africa     1997    63.3 145608     1339.
    11 Sao Tome and Principe Africa     2002    64.3 170372     1353.
    12 Sao Tome and Principe Africa     2007    65.5 199579     1598.

4.  `slice_max(column, n = 1)` takes max value of column specified

``` r
gapminder |> 
  group_by(year) |> 
  slice_max(pop, n = 1)
```

    # A tibble: 12 × 6
    # Groups:   year [12]
       country continent  year lifeExp        pop gdpPercap
       <fct>   <fct>     <int>   <dbl>      <int>     <dbl>
     1 China   Asia       1952    44    556263527      400.
     2 China   Asia       1957    50.5  637408000      576.
     3 China   Asia       1962    44.5  665770000      488.
     4 China   Asia       1967    58.4  754550000      613.
     5 China   Asia       1972    63.1  862030000      677.
     6 China   Asia       1977    64.0  943455000      741.
     7 China   Asia       1982    65.5 1000281000      962.
     8 China   Asia       1987    67.3 1084035000     1379.
     9 China   Asia       1992    68.7 1164970000     1656.
    10 China   Asia       1997    70.4 1230075000     2289.
    11 China   Asia       2002    72.0 1280400000     3119.
    12 China   Asia       2007    73.0 1318683096     4959.

## apply function to each group with `group_map()`

### Function to calculate variation in inputs within each paper

``` r
get_variation_gdpPercap <- function(df, .df) {
   dfnew <- df                                # copy input df
   minGdp <- min(df$gdpPercap, na.rm = T)  # get minimum value
   var <- as.numeric((df$gdpPercap)/minGdp)  # make var column
   dfnew %>%
     mutate(var = var)                          # add to df
}
```

``` r
# grouping df 
gapminder_varstats <- gapminder %>% 
  #filter(Mark == "input") %>%      # only inputs
  group_by(country) %>%             # need to group by paper
  group_map(get_variation_gdpPercap, .keep = TRUE) %>% # keep group variable
  bind_rows()
```

``` r
ggplot(gapminder_varstats) + 
  aes(x = continent, y = var, color = continent) +
  geom_point(position = "jitter", alpha = 0.7) + 
  labs(title = "Variation in GDPpercap by country over time", 
       x = "Continent", y = "GDPpercap relative to country minimum")
```

![](R_tidyverse_cheatsheet_files/figure-commonmark/unnamed-chunk-37-1.png)

Which country had the highest variation?

``` r
max_gdpvar <- max(gapminder_varstats$var)

gapminder_varstats[grep(max_gdpvar, gapminder_varstats$var), ]
```

    # A tibble: 1 × 7
      country           continent  year lifeExp    pop gdpPercap   var
      <fct>             <fct>     <int>   <dbl>  <int>     <dbl> <dbl>
    1 Equatorial Guinea Africa     2007    51.6 551201    12154.  32.4

# change column order

``` r
data_new$group <- factor(data_new$group,levels = c("B", "A", "C", "D"))
```

# String manipulation

## `str_remove` to remove part of a string

``` r
library(gapminder)
str_remove(gapminder$country, "stan") %>% head()
```

    [1] "Afghani" "Afghani" "Afghani" "Afghani" "Afghani" "Afghani"

Remove multiple parts of strings:

``` r
str_remove_all(Sample, "H3K27ac_|\\.Coverage")
```

Using regex:

``` r
Sample <- "HelaS3_75TSA_25DMSO_spike3_H3K27ac.Coverage"
```

## `str_match` to keep part of a string based on regex pattern

``` r
str_match(Sample, '([^_]+)(?:_[^_]+){3}$')[,2]
```

Combine with str_remove to get numeric variable for plotting

``` r
TSAratio <- str_remove(str_match(Sample, '([^_]+)(?:_[^_]+){3}$')[,2], "TSA")

TSAratio
```

    [1] "75"

## `str_extract` to keep part of a string

``` r
hist_tss_hg38_LP78_long_ordered <- hist_tss_hg38_LP78_long %>%
  mutate(
    # extract the number before 'inter'
    biorep_num = as.numeric(str_extract(biorep, "\\d+(?=inter)"))
  ) %>%
  arrange(desc(biorep_num))
```
