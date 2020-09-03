Data Prep - Pacific Islands Trade
================
Avin Datt - PacifImpact
03 September 2020

# Scope

Ingest IMTS Data into PostgreSQL database.

# Reference

IMTS data is available in .xlsx format. It can be downloaded from [SPC
Statistics for Development
Division](https://sdd.spc.int/contact-us#disclaimer)

# Load Libraries

``` r
#### Load Libraries ####
# Load openxlsx library 
if (!require(openxlsx)) {
  install.packages("openxlsx")
  library(openxlsx)
}
```

    ## Loading required package: openxlsx

    ## Warning: package 'openxlsx' was built under R version 4.0.2

``` r
# Load purrr library
if (!require(purrr)) {
  install.packages("purrr")
  library(purrr)
}
```

    ## Loading required package: purrr

``` r
# Load data.table library
if (!require(data.table)) {
  install.packages("data.table")
  library(data.table)
}
```

    ## Loading required package: data.table

    ## 
    ## Attaching package: 'data.table'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     transpose
