Data Prep - Pacific Islands Trade
================
Avin Datt - PacifImpact
03 September 2020

# Scope

Ingest IMTS Data available from SPC Statistics for Development Division
into PostgreSQL database for the following countries:

  - Fiji
  - Cook Islands
  - Solomon Islands
  - Kiribati
  - Palau

IMTS Data for Vanuatu and Samoa available via the SPC Statistics for
Development Division do not meet the projects eligibility criteria. As
such, their appropriate trade data will be gathered from alternate
sources, referenced below.

# Reference

IMTS data for Fiji, Cook Islands and Solomon Islands is available in
.xlsx format. It can be downloaded from [SPC Statistics for Development
Division](https://sdd.spc.int/contact-us#disclaimer).

Eligible trade data for Samoa can be downloaded from [Samoa Bureau of
Statistics](https://www.sbs.gov.ws/images/sbs-documents/Economics/Merchandise-Trade-Report/2020/OVERSEAS_MERCHANDISE_TRADE_JUNE_2020.pdf).

Eligible IMTS trade data for Vanuatu can be downloaded from [Vanuatu
National Statistics
Office](https://vnso.gov.vu/index.php/statistics-by-topic/trade#latest-trade-news).

# Load Libraries

``` r
# Load openxlsx library 
if (!require(openxlsx)) {
  install.packages("openxlsx")
  library(openxlsx)
}
```

    ## Loading required package: openxlsx

``` r
# Load data.table library
if (!require(data.table)) {
  install.packages("data.table")
  library(data.table)
}
```

    ## Loading required package: data.table

``` r
# Load DBI library
if (!require(DBI)) {
  install.packages("DBI")
  library(DBI)
}
```

    ## Loading required package: DBI

``` r
# Load DBI library
if (!require(RPostgres)) {
  install.packages("RPostgres")
  library(RPostgres)
}
```

    ## Loading required package: RPostgres

# Read IMTS Balance of Trade Fiji Data

Read in Balance of Trade - All Items for Fiji.

``` r
# Read in Fiji File 
fj.imts.bot <- read.xlsx(xlsxFile = './data/external/Fiji_IMTS_Tables_2020.xlsx',
                         sheet = 1, 
                         startRow = 28,
                         skipEmptyCols = TRUE,
                         detectDates = TRUE,
                         rows = c(28:111),
                         cols = c(1:7))

head(fj.imts.bot)
```

    ##   Monthly       X2     X3     X4     X5     X6      X7
    ## 1    2014  January  48975  75949 124924 366592 -241668
    ## 2    <NA> February  65365  69844 135209 303470 -168261
    ## 3    <NA>    March  60586  84929 145515 343389 -197874
    ## 4    <NA>    April  67204  63355 130559 363233 -232674
    ## 5    <NA>      May 115355 101169 216524 368526 -152002
    ## 6    <NA>     June 103352 107488 210840 491190 -280350

Add and fix missing variable names.

``` r
# fj.imts.bot to data.table 
fj.imts.bot <- data.table(fj.imts.bot)


# Fix Monthly label to Year
setnames(fj.imts.bot, 
         "Monthly",
         "year")


# X2 should be Month 
setnames(fj.imts.bot, 
         "X2",
         "month")


# Exports FOB Domestic 
setnames(fj.imts.bot, 
         "X3",
         "exports-fob-domestic")


# Exports FOB Re-Export 
setnames(fj.imts.bot, 
         "X4",
         "exports-fob-reexport")


# Exports FOB Total 
setnames(fj.imts.bot, 
         "X5",
         "exports-fob-total")


# Imports CIF
setnames(fj.imts.bot, 
         "X6",
         "imports-cif")



# Trade Balance
setnames(fj.imts.bot, 
         "X7",
         "trade-balance")


head(fj.imts.bot)
```

    ##    year    month exports-fob-domestic exports-fob-reexport exports-fob-total
    ## 1: 2014  January                48975                75949            124924
    ## 2: <NA> February                65365                69844            135209
    ## 3: <NA>    March                60586                84929            145515
    ## 4: <NA>    April                67204                63355            130559
    ## 5: <NA>      May               115355               101169            216524
    ## 6: <NA>     June               103352               107488            210840
    ##    imports-cif trade-balance
    ## 1:      366592       -241668
    ## 2:      303470       -168261
    ## 3:      343389       -197874
    ## 4:      363233       -232674
    ## 5:      368526       -152002
    ## 6:      491190       -280350

Update Year values.

``` r
# Update by using row number references
# 2014
fj.imts.bot[1:12, 
            year := 2014]


# 2015
fj.imts.bot[13:24, 
            year := 2015]


# 2016
fj.imts.bot[25:36, 
            year := 2016]


# 2017
fj.imts.bot[37:48, 
            year := 2017]


# 2018
fj.imts.bot[49:60, 
            year := 2018]


# 2019
fj.imts.bot[61:72, 
            year := 2019]


# 2020
fj.imts.bot[73:77, 
            year := 2020]


head(fj.imts.bot)
```

    ##    year    month exports-fob-domestic exports-fob-reexport exports-fob-total
    ## 1: 2014  January                48975                75949            124924
    ## 2: 2014 February                65365                69844            135209
    ## 3: 2014    March                60586                84929            145515
    ## 4: 2014    April                67204                63355            130559
    ## 5: 2014      May               115355               101169            216524
    ## 6: 2014     June               103352               107488            210840
    ##    imports-cif trade-balance
    ## 1:      366592       -241668
    ## 2:      303470       -168261
    ## 3:      343389       -197874
    ## 4:      363233       -232674
    ## 5:      368526       -152002
    ## 6:      491190       -280350

Melt the table to adhere to the key-value pair schema design.

``` r
# Melt the data to adhere to the schema design
fj.imts.bot <- melt.data.table(fj.imts.bot,
                               id = c("year",
                                      "month"), measure = c("exports-fob-domestic",
                                                            "exports-fob-reexport", 
                                                            "exports-fob-total",
                                                            "imports-cif",
                                                            "trade-balance"))


head(fj.imts.bot)
```

    ##    year    month             variable  value
    ## 1: 2014  January exports-fob-domestic  48975
    ## 2: 2014 February exports-fob-domestic  65365
    ## 3: 2014    March exports-fob-domestic  60586
    ## 4: 2014    April exports-fob-domestic  67204
    ## 5: 2014      May exports-fob-domestic 115355
    ## 6: 2014     June exports-fob-domestic 103352

Create Date from year and month.

``` r
# Concatenate
fj.imts.bot[, 
            date := paste0(gsub(" ", "", year, fixed = TRUE),
                           "-",
                           gsub(" ", "",month, fixed = TRUE),
                           "-",
                           "01")]


# Format Date
fj.imts.bot[, 
            date := as.Date(date, format = "%Y-%B-%d")]


# Remove year and month fields 
fj.imts.bot[, 
            year := NULL][, 
                          month := NULL]


head(fj.imts.bot)
```

    ##                variable  value       date
    ## 1: exports-fob-domestic  48975 2014-01-01
    ## 2: exports-fob-domestic  65365 2014-02-01
    ## 3: exports-fob-domestic  60586 2014-03-01
    ## 4: exports-fob-domestic  67204 2014-04-01
    ## 5: exports-fob-domestic 115355 2014-05-01
    ## 6: exports-fob-domestic 103352 2014-06-01

Rename “variable” to “name”.

``` r
# Rename variable to name 
setnames(fj.imts.bot, 
         "variable",
         "name")


head(fj.imts.bot)
```

    ##                    name  value       date
    ## 1: exports-fob-domestic  48975 2014-01-01
    ## 2: exports-fob-domestic  65365 2014-02-01
    ## 3: exports-fob-domestic  60586 2014-03-01
    ## 4: exports-fob-domestic  67204 2014-04-01
    ## 5: exports-fob-domestic 115355 2014-05-01
    ## 6: exports-fob-domestic 103352 2014-06-01

Add metric key, frequency, country, category, source and property
informations.

``` r
# Metric 
fj.imts.bot[, 
            `:=` (frequency = "monthly", 
                  country = "fj", 
                  category = "trade", 
                  source = "imts", 
                  properties = "{'currency' = 'fjd 000'}",
                  metric_key = paste0("fj","-",name,"-",date))]


head(fj.imts.bot)
```

    ##                    name  value       date frequency country category source
    ## 1: exports-fob-domestic  48975 2014-01-01   monthly      fj    trade   imts
    ## 2: exports-fob-domestic  65365 2014-02-01   monthly      fj    trade   imts
    ## 3: exports-fob-domestic  60586 2014-03-01   monthly      fj    trade   imts
    ## 4: exports-fob-domestic  67204 2014-04-01   monthly      fj    trade   imts
    ## 5: exports-fob-domestic 115355 2014-05-01   monthly      fj    trade   imts
    ## 6: exports-fob-domestic 103352 2014-06-01   monthly      fj    trade   imts
    ##                  properties                         metric_key
    ## 1: {'currency' = 'fjd 000'} fj-exports-fob-domestic-2014-01-01
    ## 2: {'currency' = 'fjd 000'} fj-exports-fob-domestic-2014-02-01
    ## 3: {'currency' = 'fjd 000'} fj-exports-fob-domestic-2014-03-01
    ## 4: {'currency' = 'fjd 000'} fj-exports-fob-domestic-2014-04-01
    ## 5: {'currency' = 'fjd 000'} fj-exports-fob-domestic-2014-05-01
    ## 6: {'currency' = 'fjd 000'} fj-exports-fob-domestic-2014-06-01

Reorder data.

``` r
# Change order of columns to align with schema
fj.imts.bot <- fj.imts.bot[,
                           .(metric_key,
                             frequency, 
                             country,
                             category, 
                             source,
                             name,
                             date, 
                             value,
                             properties)]


head(fj.imts.bot)
```

    ##                            metric_key frequency country category source
    ## 1: fj-exports-fob-domestic-2014-01-01   monthly      fj    trade   imts
    ## 2: fj-exports-fob-domestic-2014-02-01   monthly      fj    trade   imts
    ## 3: fj-exports-fob-domestic-2014-03-01   monthly      fj    trade   imts
    ## 4: fj-exports-fob-domestic-2014-04-01   monthly      fj    trade   imts
    ## 5: fj-exports-fob-domestic-2014-05-01   monthly      fj    trade   imts
    ## 6: fj-exports-fob-domestic-2014-06-01   monthly      fj    trade   imts
    ##                    name       date  value               properties
    ## 1: exports-fob-domestic 2014-01-01  48975 {'currency' = 'fjd 000'}
    ## 2: exports-fob-domestic 2014-02-01  65365 {'currency' = 'fjd 000'}
    ## 3: exports-fob-domestic 2014-03-01  60586 {'currency' = 'fjd 000'}
    ## 4: exports-fob-domestic 2014-04-01  67204 {'currency' = 'fjd 000'}
    ## 5: exports-fob-domestic 2014-05-01 115355 {'currency' = 'fjd 000'}
    ## 6: exports-fob-domestic 2014-06-01 103352 {'currency' = 'fjd 000'}

# Read IMTS Balance of Trade Cook Islands Data

Read in Balance of Trade - All Items for Cook Islands.

``` r
# Read in Fiji File 
ck.imts.bot <- read.xlsx(xlsxFile = './data/external/Cook_IMTS_Tables_2020.xlsx',
                         sheet = 1, 
                         startRow = 15,
                         skipEmptyCols = TRUE,
                         detectDates = TRUE,
                         rows = c(15:58),
                         cols = c(1:7))


head(ck.imts.bot)
```

    ##   Monthly        X2   X3 X4   X5    X6     X7
    ## 1    2017   January  625  0  625 13121 -12496
    ## 2    <NA> February   847  0  847 12601 -11754
    ## 3    <NA>     March 3968  0 3968 14995 -11027
    ## 4    <NA>     April 2274  0 2274 10481  -8207
    ## 5    <NA>      May   171  0  171 16231 -16060
    ## 6    <NA>      June 3164  0 3164 17060 -13896

Add and fix missing variable names.

``` r
# ck.imts.bot to data.table 
ck.imts.bot <- data.table(ck.imts.bot)


# Fix Monthly label to Year
setnames(ck.imts.bot, 
         "Monthly",
         "year")


# X2 should be Month 
setnames(ck.imts.bot, 
         "X2",
         "month")


# Exports FOB Domestic 
setnames(ck.imts.bot, 
         "X3",
         "exports-fob-domestic")


# Exports FOB Re-Export 
setnames(ck.imts.bot, 
         "X4",
         "exports-fob-reexport")


# Exports FOB Total 
setnames(ck.imts.bot, 
         "X5",
         "exports-fob-total")


# Imports CIF
setnames(ck.imts.bot, 
         "X6",
         "imports-cif")



# Trade Balance
setnames(ck.imts.bot, 
         "X7",
         "trade-balance")


head(ck.imts.bot)
```

    ##    year     month exports-fob-domestic exports-fob-reexport exports-fob-total
    ## 1: 2017   January                  625                    0               625
    ## 2: <NA> February                   847                    0               847
    ## 3: <NA>     March                 3968                    0              3968
    ## 4: <NA>     April                 2274                    0              2274
    ## 5: <NA>      May                   171                    0               171
    ## 6: <NA>      June                 3164                    0              3164
    ##    imports-cif trade-balance
    ## 1:       13121        -12496
    ## 2:       12601        -11754
    ## 3:       14995        -11027
    ## 4:       10481         -8207
    ## 5:       16231        -16060
    ## 6:       17060        -13896

Remove rogue empty row.

``` r
# Remove rows with no month value 
ck.imts.bot <- ck.imts.bot[!is.na(month)]


head(ck.imts.bot)
```

    ##    year     month exports-fob-domestic exports-fob-reexport exports-fob-total
    ## 1: 2017   January                  625                    0               625
    ## 2: <NA> February                   847                    0               847
    ## 3: <NA>     March                 3968                    0              3968
    ## 4: <NA>     April                 2274                    0              2274
    ## 5: <NA>      May                   171                    0               171
    ## 6: <NA>      June                 3164                    0              3164
    ##    imports-cif trade-balance
    ## 1:       13121        -12496
    ## 2:       12601        -11754
    ## 3:       14995        -11027
    ## 4:       10481         -8207
    ## 5:       16231        -16060
    ## 6:       17060        -13896

Update Year values.

``` r
# Update by using row number references
# 2017
ck.imts.bot[1:12, 
            year := 2017]


# 2015
ck.imts.bot[13:24, 
            year := 2018]


# 2016
ck.imts.bot[25:36, 
            year := 2019]


# 2017
ck.imts.bot[37:40, 
            year := 2020]


head(ck.imts.bot)
```

    ##    year     month exports-fob-domestic exports-fob-reexport exports-fob-total
    ## 1: 2017   January                  625                    0               625
    ## 2: 2017 February                   847                    0               847
    ## 3: 2017     March                 3968                    0              3968
    ## 4: 2017     April                 2274                    0              2274
    ## 5: 2017      May                   171                    0               171
    ## 6: 2017      June                 3164                    0              3164
    ##    imports-cif trade-balance
    ## 1:       13121        -12496
    ## 2:       12601        -11754
    ## 3:       14995        -11027
    ## 4:       10481         -8207
    ## 5:       16231        -16060
    ## 6:       17060        -13896

Melt the table to adhere to the key-value pair schema design.

``` r
# Melt the data to adhere to the schema design
ck.imts.bot <- melt.data.table(ck.imts.bot,
                               id = c("year",
                                      "month"), measure = c("exports-fob-domestic",
                                                            "exports-fob-reexport", 
                                                            "exports-fob-total",
                                                            "imports-cif",
                                                            "trade-balance"))

head(ck.imts.bot)
```

    ##    year     month             variable value
    ## 1: 2017   January exports-fob-domestic   625
    ## 2: 2017 February  exports-fob-domestic   847
    ## 3: 2017     March exports-fob-domestic  3968
    ## 4: 2017     April exports-fob-domestic  2274
    ## 5: 2017      May  exports-fob-domestic   171
    ## 6: 2017      June exports-fob-domestic  3164

Create Date from year and month.

``` r
# Concatenate
ck.imts.bot[, 
            date := paste0(gsub(" ", "", year, fixed = TRUE),
                           "-",
                           gsub(" ", "",month, fixed = TRUE),
                           "-",
                           "01")]


# Format Date
ck.imts.bot[, 
            date := as.Date(date, format = "%Y-%B-%d")]


# Remove year and month fields 
ck.imts.bot[, 
            year := NULL][, 
                          month := NULL]


head(ck.imts.bot)
```

    ##                variable value       date
    ## 1: exports-fob-domestic   625 2017-01-01
    ## 2: exports-fob-domestic   847 2017-02-01
    ## 3: exports-fob-domestic  3968 2017-03-01
    ## 4: exports-fob-domestic  2274 2017-04-01
    ## 5: exports-fob-domestic   171 2017-05-01
    ## 6: exports-fob-domestic  3164 2017-06-01

Rename “variable” to “name”.

``` r
# Rename variable to name 
setnames(ck.imts.bot, 
         "variable",
         "name")


head(ck.imts.bot)
```

    ##                    name value       date
    ## 1: exports-fob-domestic   625 2017-01-01
    ## 2: exports-fob-domestic   847 2017-02-01
    ## 3: exports-fob-domestic  3968 2017-03-01
    ## 4: exports-fob-domestic  2274 2017-04-01
    ## 5: exports-fob-domestic   171 2017-05-01
    ## 6: exports-fob-domestic  3164 2017-06-01

Add metric key, frequency, country, category, source and property
informations.

``` r
# Metric 
ck.imts.bot[, 
            `:=` (frequency = "monthly", 
                  country = "ck", 
                  category = "trade", 
                  source = "imts", 
                  properties = "{'currency' = 'nzd 000'}",
                  metric_key = paste0("ck","-",name,"-",date))]


head(ck.imts.bot)
```

    ##                    name value       date frequency country category source
    ## 1: exports-fob-domestic   625 2017-01-01   monthly      ck    trade   imts
    ## 2: exports-fob-domestic   847 2017-02-01   monthly      ck    trade   imts
    ## 3: exports-fob-domestic  3968 2017-03-01   monthly      ck    trade   imts
    ## 4: exports-fob-domestic  2274 2017-04-01   monthly      ck    trade   imts
    ## 5: exports-fob-domestic   171 2017-05-01   monthly      ck    trade   imts
    ## 6: exports-fob-domestic  3164 2017-06-01   monthly      ck    trade   imts
    ##                  properties                         metric_key
    ## 1: {'currency' = 'nzd 000'} ck-exports-fob-domestic-2017-01-01
    ## 2: {'currency' = 'nzd 000'} ck-exports-fob-domestic-2017-02-01
    ## 3: {'currency' = 'nzd 000'} ck-exports-fob-domestic-2017-03-01
    ## 4: {'currency' = 'nzd 000'} ck-exports-fob-domestic-2017-04-01
    ## 5: {'currency' = 'nzd 000'} ck-exports-fob-domestic-2017-05-01
    ## 6: {'currency' = 'nzd 000'} ck-exports-fob-domestic-2017-06-01

Reorder data.

``` r
# Change order of columns to align with schema
ck.imts.bot <- ck.imts.bot[,
                           .(metric_key,
                             frequency, 
                             country,
                             category, 
                             source,
                             name,
                             date, 
                             value,
                             properties)]

head(ck.imts.bot)
```

    ##                            metric_key frequency country category source
    ## 1: ck-exports-fob-domestic-2017-01-01   monthly      ck    trade   imts
    ## 2: ck-exports-fob-domestic-2017-02-01   monthly      ck    trade   imts
    ## 3: ck-exports-fob-domestic-2017-03-01   monthly      ck    trade   imts
    ## 4: ck-exports-fob-domestic-2017-04-01   monthly      ck    trade   imts
    ## 5: ck-exports-fob-domestic-2017-05-01   monthly      ck    trade   imts
    ## 6: ck-exports-fob-domestic-2017-06-01   monthly      ck    trade   imts
    ##                    name       date value               properties
    ## 1: exports-fob-domestic 2017-01-01   625 {'currency' = 'nzd 000'}
    ## 2: exports-fob-domestic 2017-02-01   847 {'currency' = 'nzd 000'}
    ## 3: exports-fob-domestic 2017-03-01  3968 {'currency' = 'nzd 000'}
    ## 4: exports-fob-domestic 2017-04-01  2274 {'currency' = 'nzd 000'}
    ## 5: exports-fob-domestic 2017-05-01   171 {'currency' = 'nzd 000'}
    ## 6: exports-fob-domestic 2017-06-01  3164 {'currency' = 'nzd 000'}

# Read IMTS Balance of Trade Solomon Islands Data

Read in Balance of Trade - All Items for Solomon Islands.

``` r
# Read in Fiji File 
sl.imts.bot <- read.xlsx(xlsxFile = './data/external/Solomon_Islands_IMTS_Tables_2020.xlsx',
                         sheet = 1, 
                         startRow = 25,
                         skipEmptyCols = TRUE,
                         detectDates = TRUE,
                         rows = c(25:41),
                         cols = c(1:7))



head(sl.imts.bot)
```

    ##   Monthly       X2       X3        X4       X5       X6         X7
    ## 1    2018  January 456929.4  111.6982 457041.1 414343.6  42697.536
    ## 2      NA February 344153.8  620.2164 344774.0 342681.5   2092.568
    ## 3      NA    March 416248.3  490.8431 416739.2 318768.4  97970.802
    ## 4      NA    April 412862.3 4537.3310 417399.6 305915.3 111484.310
    ## 5      NA      May 376429.7  996.8101 377426.5 375513.1   1913.329
    ## 6      NA     June 286506.1 2610.8171 289116.9 365524.5 -76407.510

Add and fix missing variable names.

``` r
# sl.imts.bot to data.table 
sl.imts.bot <- data.table(sl.imts.bot)


# Fix Monthly label to Year
setnames(sl.imts.bot, 
         "Monthly",
         "year")


# X2 should be Month 
setnames(sl.imts.bot, 
         "X2",
         "month")


# Exports FOB Domestic 
setnames(sl.imts.bot, 
         "X3",
         "exports-fob-domestic")


# Exports FOB Re-Export 
setnames(sl.imts.bot, 
         "X4",
         "exports-fob-reexport")


# Exports FOB Total 
setnames(sl.imts.bot, 
         "X5",
         "exports-fob-total")


# Imports CIF
setnames(sl.imts.bot, 
         "X6",
         "imports-cif")



# Trade Balance
setnames(sl.imts.bot, 
         "X7",
         "trade-balance")

head(sl.imts.bot)
```

    ##    year    month exports-fob-domestic exports-fob-reexport exports-fob-total
    ## 1: 2018  January             456929.4             111.6982          457041.1
    ## 2:   NA February             344153.8             620.2164          344774.0
    ## 3:   NA    March             416248.3             490.8431          416739.2
    ## 4:   NA    April             412862.3            4537.3310          417399.6
    ## 5:   NA      May             376429.7             996.8101          377426.5
    ## 6:   NA     June             286506.1            2610.8171          289116.9
    ##    imports-cif trade-balance
    ## 1:    414343.6     42697.536
    ## 2:    342681.5      2092.568
    ## 3:    318768.4     97970.802
    ## 4:    305915.3    111484.310
    ## 5:    375513.1      1913.329
    ## 6:    365524.5    -76407.510

Update Year values.

``` r
# Update by using row number references
# 2018
sl.imts.bot[1:12, 
            year := 2018]


# 2019
sl.imts.bot[13:15, 
            year := 2019]


head(sl.imts.bot)
```

    ##    year    month exports-fob-domestic exports-fob-reexport exports-fob-total
    ## 1: 2018  January             456929.4             111.6982          457041.1
    ## 2: 2018 February             344153.8             620.2164          344774.0
    ## 3: 2018    March             416248.3             490.8431          416739.2
    ## 4: 2018    April             412862.3            4537.3310          417399.6
    ## 5: 2018      May             376429.7             996.8101          377426.5
    ## 6: 2018     June             286506.1            2610.8171          289116.9
    ##    imports-cif trade-balance
    ## 1:    414343.6     42697.536
    ## 2:    342681.5      2092.568
    ## 3:    318768.4     97970.802
    ## 4:    305915.3    111484.310
    ## 5:    375513.1      1913.329
    ## 6:    365524.5    -76407.510

Melt the table to adhere to the key-value pair schema design.

``` r
# Melt the data to adhere to the schema design
sl.imts.bot <- melt.data.table(sl.imts.bot,
                               id = c("year",
                                      "month"), measure = c("exports-fob-domestic",
                                                            "exports-fob-reexport", 
                                                            "exports-fob-total",
                                                            "imports-cif",
                                                            "trade-balance"))

head(sl.imts.bot)
```

    ##    year    month             variable    value
    ## 1: 2018  January exports-fob-domestic 456929.4
    ## 2: 2018 February exports-fob-domestic 344153.8
    ## 3: 2018    March exports-fob-domestic 416248.3
    ## 4: 2018    April exports-fob-domestic 412862.3
    ## 5: 2018      May exports-fob-domestic 376429.7
    ## 6: 2018     June exports-fob-domestic 286506.1

Create Date from year and month.

``` r
# Concatenate
sl.imts.bot[, 
            date := paste0(gsub(" ", "", year, fixed = TRUE),
                           "-",
                           gsub(" ", "",month, fixed = TRUE),
                           "-",
                           "01")]


# Format Date
sl.imts.bot[, 
            date := as.Date(date, format = "%Y-%B-%d")]


# Remove year and month fields 
sl.imts.bot[, 
            year := NULL][, 
                          month := NULL]


head(sl.imts.bot)
```

    ##                variable    value       date
    ## 1: exports-fob-domestic 456929.4 2018-01-01
    ## 2: exports-fob-domestic 344153.8 2018-02-01
    ## 3: exports-fob-domestic 416248.3 2018-03-01
    ## 4: exports-fob-domestic 412862.3 2018-04-01
    ## 5: exports-fob-domestic 376429.7 2018-05-01
    ## 6: exports-fob-domestic 286506.1 2018-06-01

Rename “variable” to “name”.

``` r
# Rename variable to name 
setnames(sl.imts.bot, 
         "variable",
         "name")


head(sl.imts.bot)
```

    ##                    name    value       date
    ## 1: exports-fob-domestic 456929.4 2018-01-01
    ## 2: exports-fob-domestic 344153.8 2018-02-01
    ## 3: exports-fob-domestic 416248.3 2018-03-01
    ## 4: exports-fob-domestic 412862.3 2018-04-01
    ## 5: exports-fob-domestic 376429.7 2018-05-01
    ## 6: exports-fob-domestic 286506.1 2018-06-01

Add metric key, frequency, country, category, source and property
informations.

``` r
# Metric 
sl.imts.bot[, 
            `:=` (frequency = "monthly", 
                  country = "sb", 
                  category = "trade", 
                  source = "imts", 
                  properties = "{'currency' = 'sbd 000'}",
                  metric_key = paste0("sb","-",name,"-",date))]


head(sl.imts.bot)
```

    ##                    name    value       date frequency country category source
    ## 1: exports-fob-domestic 456929.4 2018-01-01   monthly      sb    trade   imts
    ## 2: exports-fob-domestic 344153.8 2018-02-01   monthly      sb    trade   imts
    ## 3: exports-fob-domestic 416248.3 2018-03-01   monthly      sb    trade   imts
    ## 4: exports-fob-domestic 412862.3 2018-04-01   monthly      sb    trade   imts
    ## 5: exports-fob-domestic 376429.7 2018-05-01   monthly      sb    trade   imts
    ## 6: exports-fob-domestic 286506.1 2018-06-01   monthly      sb    trade   imts
    ##                  properties                         metric_key
    ## 1: {'currency' = 'sbd 000'} sb-exports-fob-domestic-2018-01-01
    ## 2: {'currency' = 'sbd 000'} sb-exports-fob-domestic-2018-02-01
    ## 3: {'currency' = 'sbd 000'} sb-exports-fob-domestic-2018-03-01
    ## 4: {'currency' = 'sbd 000'} sb-exports-fob-domestic-2018-04-01
    ## 5: {'currency' = 'sbd 000'} sb-exports-fob-domestic-2018-05-01
    ## 6: {'currency' = 'sbd 000'} sb-exports-fob-domestic-2018-06-01

Reorder data.

``` r
# Change order of columns to align with schema
sl.imts.bot <- sl.imts.bot[,
                           .(metric_key,
                             frequency, 
                             country,
                             category, 
                             source,
                             name,
                             date, 
                             value,
                             properties)]


head(sl.imts.bot)
```

    ##                            metric_key frequency country category source
    ## 1: sb-exports-fob-domestic-2018-01-01   monthly      sb    trade   imts
    ## 2: sb-exports-fob-domestic-2018-02-01   monthly      sb    trade   imts
    ## 3: sb-exports-fob-domestic-2018-03-01   monthly      sb    trade   imts
    ## 4: sb-exports-fob-domestic-2018-04-01   monthly      sb    trade   imts
    ## 5: sb-exports-fob-domestic-2018-05-01   monthly      sb    trade   imts
    ## 6: sb-exports-fob-domestic-2018-06-01   monthly      sb    trade   imts
    ##                    name       date    value               properties
    ## 1: exports-fob-domestic 2018-01-01 456929.4 {'currency' = 'sbd 000'}
    ## 2: exports-fob-domestic 2018-02-01 344153.8 {'currency' = 'sbd 000'}
    ## 3: exports-fob-domestic 2018-03-01 416248.3 {'currency' = 'sbd 000'}
    ## 4: exports-fob-domestic 2018-04-01 412862.3 {'currency' = 'sbd 000'}
    ## 5: exports-fob-domestic 2018-05-01 376429.7 {'currency' = 'sbd 000'}
    ## 6: exports-fob-domestic 2018-06-01 286506.1 {'currency' = 'sbd 000'}

# Read Samoa Bureau of Statistics Trade Data

This data has been extracted manual from the reference source outlined
earlier.

The process followed:

1.  Extract data from PDF
2.  Manipulate into the required data schema using MS Excel.
3.  Save as .csv to be uploaded to PostgreSQL Database.

<!-- end list -->

``` r
# Read in Samoa File 
ws.imts.bot <- fread('./data/interim/Samoa IMTS.csv')


head(ws.imts.bot)
```

    ##    V1      date          name  value country category frequency source currency
    ## 1:  0 1/01/2018 total-exports   6181     WSM    trade   monthly    sbs 000 tala
    ## 2:  1 1/01/2018 total-imports  74546     WSM    trade   monthly    sbs 000 tala
    ## 3:  2 1/01/2018 trade-balance -68365     WSM    trade   monthly    sbs 000 tala
    ## 4:  3 1/02/2018 total-exports   5784     WSM    trade   monthly    sbs 000 tala
    ## 5:  4 1/02/2018 total-imports  68186     WSM    trade   monthly    sbs 000 tala
    ## 6:  5 1/02/2018 trade-balance -62402     WSM    trade   monthly    sbs 000 tala

Manipulate to fit required data schema. Add appropriate attributes and
reorder the dataset to match the required schema.

``` r
# Keep only date, source, name, value, frequency
ws.imts.bot[, 
            `:=` (V1 = NULL,
                  country = NULL,
                  currency = NULL)]

# Metric 
ws.imts.bot[, 
            `:=` (country = "ws", 
                  properties = "{'currency' = 'wst 000'}",
                  metric_key = paste0("ws","-",name,"-",date))]


# Change order of columns to align with schema
ws.imts.bot <- ws.imts.bot[,
                           .(metric_key,
                             frequency, 
                             country,
                             category, 
                             source,
                             name,
                             date, 
                             value,
                             properties)]

head(ws.imts.bot)
```

    ##                    metric_key frequency country category source          name
    ## 1: ws-total-exports-1/01/2018   monthly      ws    trade    sbs total-exports
    ## 2: ws-total-imports-1/01/2018   monthly      ws    trade    sbs total-imports
    ## 3: ws-trade-balance-1/01/2018   monthly      ws    trade    sbs trade-balance
    ## 4: ws-total-exports-1/02/2018   monthly      ws    trade    sbs total-exports
    ## 5: ws-total-imports-1/02/2018   monthly      ws    trade    sbs total-imports
    ## 6: ws-trade-balance-1/02/2018   monthly      ws    trade    sbs trade-balance
    ##         date  value               properties
    ## 1: 1/01/2018   6181 {'currency' = 'wst 000'}
    ## 2: 1/01/2018  74546 {'currency' = 'wst 000'}
    ## 3: 1/01/2018 -68365 {'currency' = 'wst 000'}
    ## 4: 1/02/2018   5784 {'currency' = 'wst 000'}
    ## 5: 1/02/2018  68186 {'currency' = 'wst 000'}
    ## 6: 1/02/2018 -62402 {'currency' = 'wst 000'}

# Read Vanuatu National Statistics Trade Data

This data has been extracted manual from the reference source outlined
earlier.

The process followed:

1.  Extract data from PDF (Page 6, Table 1)
2.  Manipulate into the required data schema using MS Excel.
3.  Save as .csv to be uploaded to PostgreSQL Database.

<!-- end list -->

``` r
# Read in Vanuatu File 
vn.imts.bot <- fread('./data/interim/Vanuatu IMTS cleaned.csv')


head(vn.imts.bot)
```

    ##    country category source date               name       port    value
    ## 1:     VUT    trade   vnso 2016      trade-balance            -35962.0
    ## 2:     VUT    trade   vnso 2016      balance-ratio                 0.1
    ## 3:     VUT    trade   vnso 2016 luganville-exports luganville   3940.0
    ## 4:     VUT    trade   vnso 2016  port-vila-exports  port-vila   1506.0
    ## 5:     VUT    trade   vnso 2016   combined-exports              5446.0
    ## 6:     VUT    trade   vnso 2016         re-exports                 0.0
    ##    frequency     currency
    ## 1:  annually million-vatu
    ## 2:  annually million-vatu
    ## 3:  annually million-vatu
    ## 4:  annually million-vatu
    ## 5:  annually million-vatu
    ## 6:  annually million-vatu

Manipulate to fit required data schema. Add appropriate attributes and
reorder the dataset to match the required schema.

``` r
# Keep only date, source, name, value, frequency
vn.imts.bot[, 
            `:=` (country = NULL,
                  category = NULL, 
                  port = NULL, 
                  currency = NULL)]

# Metric 
vn.imts.bot[, 
            `:=` (country = "vn", 
                  category = "trade", 
                  properties = "{'currency' = 'vuv 000'}",
                  metric_key = paste0("vn","-",name,"-",date))]


# Change order of columns to align with schema
vn.imts.bot <- vn.imts.bot[,
                           .(metric_key,
                             frequency, 
                             country,
                             category, 
                             source,
                             name,
                             date, 
                             value,
                             properties)]

head(vn.imts.bot)
```

    ##                    metric_key frequency country category source
    ## 1:      vn-trade-balance-2016  annually      vn    trade   vnso
    ## 2:      vn-balance-ratio-2016  annually      vn    trade   vnso
    ## 3: vn-luganville-exports-2016  annually      vn    trade   vnso
    ## 4:  vn-port-vila-exports-2016  annually      vn    trade   vnso
    ## 5:   vn-combined-exports-2016  annually      vn    trade   vnso
    ## 6:         vn-re-exports-2016  annually      vn    trade   vnso
    ##                  name date    value               properties
    ## 1:      trade-balance 2016 -35962.0 {'currency' = 'vuv 000'}
    ## 2:      balance-ratio 2016      0.1 {'currency' = 'vuv 000'}
    ## 3: luganville-exports 2016   3940.0 {'currency' = 'vuv 000'}
    ## 4:  port-vila-exports 2016   1506.0 {'currency' = 'vuv 000'}
    ## 5:   combined-exports 2016   5446.0 {'currency' = 'vuv 000'}
    ## 6:         re-exports 2016      0.0 {'currency' = 'vuv 000'}

Where the the values are for annual frequency, update the date field if
it only contains a year value. Make formatting of the date field
consistent.

``` r
# Update date 
vn.imts.bot[date == "2016",
            date := "2016-12-01"]

# Concatenate date with just month and year to create full date like other datasets
vn.imts.bot[frequency != "annually", 
            date := paste0("01-",
                           date)]


# Format Date frequency annual
vn.imts.bot.a <- vn.imts.bot[frequency == "annually"][, 
                                                      date := as.Date(date)]


# Format Date frequency all others
vn.imts.bot.b <- vn.imts.bot[frequency != "annually"][, 
                                                      date := as.Date(date, format = "%d-%b-%y")]


# Stack back together 
vn.imts.bot <- rbind(vn.imts.bot.a, 
                         vn.imts.bot.b)

# Remove temp ds 
rm(vn.imts.bot.a,
   vn.imts.bot.b)


head(vn.imts.bot)
```

    ##                    metric_key frequency country category source
    ## 1:      vn-trade-balance-2016  annually      vn    trade   vnso
    ## 2:      vn-balance-ratio-2016  annually      vn    trade   vnso
    ## 3: vn-luganville-exports-2016  annually      vn    trade   vnso
    ## 4:  vn-port-vila-exports-2016  annually      vn    trade   vnso
    ## 5:   vn-combined-exports-2016  annually      vn    trade   vnso
    ## 6:         vn-re-exports-2016  annually      vn    trade   vnso
    ##                  name       date    value               properties
    ## 1:      trade-balance 2016-12-01 -35962.0 {'currency' = 'vuv 000'}
    ## 2:      balance-ratio 2016-12-01      0.1 {'currency' = 'vuv 000'}
    ## 3: luganville-exports 2016-12-01   3940.0 {'currency' = 'vuv 000'}
    ## 4:  port-vila-exports 2016-12-01   1506.0 {'currency' = 'vuv 000'}
    ## 5:   combined-exports 2016-12-01   5446.0 {'currency' = 'vuv 000'}
    ## 6:         re-exports 2016-12-01      0.0 {'currency' = 'vuv 000'}

# Read IMTS Balance of Trade Palau Data

Read in Balance of Trade - All Items for Palau.

``` r
# Read in Fiji File 
pl.imts.bot <- read.xlsx(xlsxFile = './data/external/Palau_IMTS_Tables_2020_.xlsx',
                         sheet = 1, 
                         startRow = 8,
                         skipEmptyCols = TRUE,
                         detectDates = TRUE,
                         rows = c(8:34),
                         cols = c(1:7))

head(pl.imts.bot)
```

    ##   Monthly        X2       X3  X4       X5       X6        X7
    ## 1  2018.0   January 680698.6 n/a 680698.6  9016884  -8336185
    ## 2    <NA> February  278271.1 n/a 278271.1 13527780 -13249509
    ## 3    <NA>     March 353315.0 n/a 353315.0 17111059 -16757744
    ## 4    <NA>     April 223003.0 n/a 223003.0 14258575 -14035572
    ## 5    <NA>      May  317412.0 n/a 317412.0 16048618 -15731206
    ## 6    <NA>      June 288296.1 n/a 288296.1 14540280 -14251984

Add and fix missing variable names.

``` r
# pl.imts.bot to data.table 
pl.imts.bot <- data.table(pl.imts.bot)


# Fix Monthly label to Year
setnames(pl.imts.bot, 
         "Monthly",
         "year")


# X2 should be Month 
setnames(pl.imts.bot, 
         "X2",
         "month")


# Exports FOB Domestic 
setnames(pl.imts.bot, 
         "X3",
         "exports-fob-domestic")


# Exports FOB Re-Export 
setnames(pl.imts.bot, 
         "X4",
         "exports-fob-reexport")


# Exports FOB Total 
setnames(pl.imts.bot, 
         "X5",
         "exports-fob-total")


# Imports CIF
setnames(pl.imts.bot, 
         "X6",
         "imports-cif")



# Trade Balance
setnames(pl.imts.bot, 
         "X7",
         "trade-balance")


head(pl.imts.bot)
```

    ##      year     month exports-fob-domestic exports-fob-reexport exports-fob-total
    ## 1: 2018.0   January             680698.6                  n/a          680698.6
    ## 2:   <NA> February              278271.1                  n/a          278271.1
    ## 3:   <NA>     March             353315.0                  n/a          353315.0
    ## 4:   <NA>     April             223003.0                  n/a          223003.0
    ## 5:   <NA>      May              317412.0                  n/a          317412.0
    ## 6:   <NA>      June             288296.1                  n/a          288296.1
    ##    imports-cif trade-balance
    ## 1:     9016884      -8336185
    ## 2:    13527780     -13249509
    ## 3:    17111059     -16757744
    ## 4:    14258575     -14035572
    ## 5:    16048618     -15731206
    ## 6:    14540280     -14251984

Update Year values.

``` r
# Update by using row number references
# 2014
pl.imts.bot[1:12, 
            year := 2018]


# 2015
pl.imts.bot[13:24, 
            year := 2019]


head(pl.imts.bot)
```

    ##    year     month exports-fob-domestic exports-fob-reexport exports-fob-total
    ## 1: 2018   January             680698.6                  n/a          680698.6
    ## 2: 2018 February              278271.1                  n/a          278271.1
    ## 3: 2018     March             353315.0                  n/a          353315.0
    ## 4: 2018     April             223003.0                  n/a          223003.0
    ## 5: 2018      May              317412.0                  n/a          317412.0
    ## 6: 2018      June             288296.1                  n/a          288296.1
    ##    imports-cif trade-balance
    ## 1:     9016884      -8336185
    ## 2:    13527780     -13249509
    ## 3:    17111059     -16757744
    ## 4:    14258575     -14035572
    ## 5:    16048618     -15731206
    ## 6:    14540280     -14251984

Melt the table to adhere to the key-value pair schema design.

``` r
# Melt the data to adhere to the schema design
pl.imts.bot <- melt.data.table(pl.imts.bot,
                               id = c("year",
                                      "month"), measure = c("exports-fob-domestic",
                                                            "exports-fob-reexport", 
                                                            "exports-fob-total",
                                                            "imports-cif",
                                                            "trade-balance"))
```

    ## Warning in melt.data.table(pl.imts.bot, id = c("year", "month"), measure =
    ## c("exports-fob-domestic", : 'measure.vars' [exports-fob-domestic, exports-fob-
    ## reexport, exports-fob-total, imports-cif, trade-balance] are not all of the
    ## same type. By order of hierarchy, the molten data value column will be of type
    ## 'character'. All measure variables not of type 'character' will be coerced too.
    ## Check DETAILS in ?melt.data.table for more on coercion.

``` r
head(pl.imts.bot)
```

    ##    year     month             variable     value
    ## 1: 2018   January exports-fob-domestic 680698.59
    ## 2: 2018 February  exports-fob-domestic 278271.12
    ## 3: 2018     March exports-fob-domestic    353315
    ## 4: 2018     April exports-fob-domestic    223003
    ## 5: 2018      May  exports-fob-domestic    317412
    ## 6: 2018      June exports-fob-domestic  288296.1

Create Date from year and month.

``` r
# Concatenate
pl.imts.bot[, 
            date := paste0(gsub(" ", "", year, fixed = TRUE),
                           "-",
                           gsub(" ", "",month, fixed = TRUE),
                           "-",
                           "01")]


# Format Date
pl.imts.bot[, 
            date := as.Date(date, format = "%Y-%B-%d")]


# Remove year and month fields 
pl.imts.bot[, 
            year := NULL][, 
                          month := NULL]


head(pl.imts.bot)
```

    ##                variable     value       date
    ## 1: exports-fob-domestic 680698.59 2018-01-01
    ## 2: exports-fob-domestic 278271.12 2018-02-01
    ## 3: exports-fob-domestic    353315 2018-03-01
    ## 4: exports-fob-domestic    223003 2018-04-01
    ## 5: exports-fob-domestic    317412 2018-05-01
    ## 6: exports-fob-domestic  288296.1 2018-06-01

Rename “variable” to “name”.

``` r
# Rename variable to name 
setnames(pl.imts.bot, 
         "variable",
         "name")


head(pl.imts.bot)
```

    ##                    name     value       date
    ## 1: exports-fob-domestic 680698.59 2018-01-01
    ## 2: exports-fob-domestic 278271.12 2018-02-01
    ## 3: exports-fob-domestic    353315 2018-03-01
    ## 4: exports-fob-domestic    223003 2018-04-01
    ## 5: exports-fob-domestic    317412 2018-05-01
    ## 6: exports-fob-domestic  288296.1 2018-06-01

Add metric key, frequency, country, category, source and property
informations.

``` r
# Metric 
pl.imts.bot[, 
            `:=` (frequency = "monthly", 
                  country = "pl", 
                  category = "trade", 
                  source = "imts", 
                  properties = "{'currency' = 'usd 000'}",
                  metric_key = paste0("pl","-",name,"-",date))]


head(pl.imts.bot)
```

    ##                    name     value       date frequency country category source
    ## 1: exports-fob-domestic 680698.59 2018-01-01   monthly      pl    trade   imts
    ## 2: exports-fob-domestic 278271.12 2018-02-01   monthly      pl    trade   imts
    ## 3: exports-fob-domestic    353315 2018-03-01   monthly      pl    trade   imts
    ## 4: exports-fob-domestic    223003 2018-04-01   monthly      pl    trade   imts
    ## 5: exports-fob-domestic    317412 2018-05-01   monthly      pl    trade   imts
    ## 6: exports-fob-domestic  288296.1 2018-06-01   monthly      pl    trade   imts
    ##                  properties                         metric_key
    ## 1: {'currency' = 'usd 000'} pl-exports-fob-domestic-2018-01-01
    ## 2: {'currency' = 'usd 000'} pl-exports-fob-domestic-2018-02-01
    ## 3: {'currency' = 'usd 000'} pl-exports-fob-domestic-2018-03-01
    ## 4: {'currency' = 'usd 000'} pl-exports-fob-domestic-2018-04-01
    ## 5: {'currency' = 'usd 000'} pl-exports-fob-domestic-2018-05-01
    ## 6: {'currency' = 'usd 000'} pl-exports-fob-domestic-2018-06-01

Reorder data.

``` r
# Change order of columns to align with schema
pl.imts.bot <- pl.imts.bot[,
                           .(metric_key,
                             frequency, 
                             country,
                             category, 
                             source,
                             name,
                             date, 
                             value,
                             properties)]


head(pl.imts.bot)
```

    ##                            metric_key frequency country category source
    ## 1: pl-exports-fob-domestic-2018-01-01   monthly      pl    trade   imts
    ## 2: pl-exports-fob-domestic-2018-02-01   monthly      pl    trade   imts
    ## 3: pl-exports-fob-domestic-2018-03-01   monthly      pl    trade   imts
    ## 4: pl-exports-fob-domestic-2018-04-01   monthly      pl    trade   imts
    ## 5: pl-exports-fob-domestic-2018-05-01   monthly      pl    trade   imts
    ## 6: pl-exports-fob-domestic-2018-06-01   monthly      pl    trade   imts
    ##                    name       date     value               properties
    ## 1: exports-fob-domestic 2018-01-01 680698.59 {'currency' = 'usd 000'}
    ## 2: exports-fob-domestic 2018-02-01 278271.12 {'currency' = 'usd 000'}
    ## 3: exports-fob-domestic 2018-03-01    353315 {'currency' = 'usd 000'}
    ## 4: exports-fob-domestic 2018-04-01    223003 {'currency' = 'usd 000'}
    ## 5: exports-fob-domestic 2018-05-01    317412 {'currency' = 'usd 000'}
    ## 6: exports-fob-domestic 2018-06-01  288296.1 {'currency' = 'usd 000'}

# Read IMTS Balance of Trade Kiribati Data

Read in Balance of Trade - All Items for Kiribati

``` r
# Read in Kiribati File 
ki.imts.bot <- read.xlsx(xlsxFile = './data/external/Kiribati_IMTS_Tables_2020.xlsx',
                         sheet = 1, 
                         startRow = 14,
                         skipEmptyCols = TRUE,
                         detectDates = TRUE,
                         rows = c(14:67),
                         cols = c(1:7))


head(ki.imts.bot)
```

    ##   Monthly       X2                 X3        X4         X5       X6        X7
    ## 1    2016  January 1859.5960000000002   8.15420 1867.75020 14488.23 -12620.48
    ## 2      NA February 15.651199999999998  76.00935   91.66055 12139.84 -12048.17
    ## 3      NA    March            208.036  31.88060  239.91660 15219.95 -14980.03
    ## 4      NA    April  670.9549999999999  10.93100  681.88600 12813.96 -12132.07
    ## 5      NA      May 207.89599999999996  20.66900  228.56500 15843.96 -15615.40
    ## 6      NA     June 502.08099999999996 381.93280  884.01380 16768.72 -15884.71

Add and fix missing variable names.

``` r
# ki.imts.bot to data.table 
ki.imts.bot <- data.table(ki.imts.bot)


# Remove empty record
ki.imts.bot <- ki.imts.bot[!is.na(X2)]



# Fix Monthly label to Year
setnames(ki.imts.bot, 
         "Monthly",
         "year")


# X2 should be Month 
setnames(ki.imts.bot, 
         "X2",
         "month")


# Exports FOB Domestic 
setnames(ki.imts.bot, 
         "X3",
         "exports-fob-domestic")


# Exports FOB Re-Export 
setnames(ki.imts.bot, 
         "X4",
         "exports-fob-reexport")


# Exports FOB Total 
setnames(ki.imts.bot, 
         "X5",
         "exports-fob-total")


# Imports CIF
setnames(ki.imts.bot, 
         "X6",
         "imports-cif")



# Trade Balance
setnames(ki.imts.bot, 
         "X7",
         "trade-balance")


head(ki.imts.bot)
```

    ##    year    month exports-fob-domestic exports-fob-reexport exports-fob-total
    ## 1: 2016  January   1859.5960000000002              8.15420        1867.75020
    ## 2:   NA February   15.651199999999998             76.00935          91.66055
    ## 3:   NA    March              208.036             31.88060         239.91660
    ## 4:   NA    April    670.9549999999999             10.93100         681.88600
    ## 5:   NA      May   207.89599999999996             20.66900         228.56500
    ## 6:   NA     June   502.08099999999996            381.93280         884.01380
    ##    imports-cif trade-balance
    ## 1:    14488.23     -12620.48
    ## 2:    12139.84     -12048.17
    ## 3:    15219.95     -14980.03
    ## 4:    12813.96     -12132.07
    ## 5:    15843.96     -15615.40
    ## 6:    16768.72     -15884.71

Update Year values.

``` r
# Update by using row number references
# 2014
ki.imts.bot[1:12, 
            year := 2016]


# 2015
ki.imts.bot[13:24, 
            year := 2017]


# 2016
ki.imts.bot[25:36, 
            year := 2018]


# 2017
ki.imts.bot[37:48, 
            year := 2019]


head(C)
```

    ##                                                                              
    ## 1 function (object, contr, how.many, ...)                                    
    ## 2 {                                                                          
    ## 3     if (isFALSE(as.logical(Sys.getenv("_R_OPTIONS_STRINGS_AS_FACTORS_")))) 
    ## 4         object <- as.factor(object)                                        
    ## 5     if (!nlevels(object))                                                  
    ## 6         stop("object not interpretable as a factor")

Melt the table to adhere to the key-value pair schema design.

``` r
# Melt the data to adhere to the schema design
ki.imts.bot <- melt.data.table(ki.imts.bot,
                               id = c("year",
                                      "month"), measure = c("exports-fob-domestic",
                                                            "exports-fob-reexport", 
                                                            "exports-fob-total",
                                                            "imports-cif",
                                                            "trade-balance"))
```

    ## Warning in melt.data.table(ki.imts.bot, id = c("year", "month"), measure =
    ## c("exports-fob-domestic", : 'measure.vars' [exports-fob-domestic, exports-fob-
    ## reexport, exports-fob-total, imports-cif, trade-balance] are not all of the
    ## same type. By order of hierarchy, the molten data value column will be of type
    ## 'character'. All measure variables not of type 'character' will be coerced too.
    ## Check DETAILS in ?melt.data.table for more on coercion.

``` r
head(ki.imts.bot)
```

    ##    year    month             variable              value
    ## 1: 2016  January exports-fob-domestic 1859.5960000000002
    ## 2: 2016 February exports-fob-domestic 15.651199999999998
    ## 3: 2016    March exports-fob-domestic            208.036
    ## 4: 2016    April exports-fob-domestic  670.9549999999999
    ## 5: 2016      May exports-fob-domestic 207.89599999999996
    ## 6: 2016     June exports-fob-domestic 502.08099999999996

Create Date from year and month.

``` r
# Concatenate
ki.imts.bot[, 
            date := paste0(gsub(" ", "", year, fixed = TRUE),
                           "-",
                           gsub(" ", "",month, fixed = TRUE),
                           "-",
                           "01")]


# Format Date
ki.imts.bot[, 
            date := as.Date(date, format = "%Y-%B-%d")]


# Remove year and month fields 
ki.imts.bot[, 
            year := NULL][, 
                          month := NULL]


head(ki.imts.bot)
```

    ##                variable              value       date
    ## 1: exports-fob-domestic 1859.5960000000002 2016-01-01
    ## 2: exports-fob-domestic 15.651199999999998 2016-02-01
    ## 3: exports-fob-domestic            208.036 2016-03-01
    ## 4: exports-fob-domestic  670.9549999999999 2016-04-01
    ## 5: exports-fob-domestic 207.89599999999996 2016-05-01
    ## 6: exports-fob-domestic 502.08099999999996 2016-06-01

Rename “variable” to “name”.

``` r
# Rename variable to name 
setnames(ki.imts.bot, 
         "variable",
         "name")


head(ki.imts.bot)
```

    ##                    name              value       date
    ## 1: exports-fob-domestic 1859.5960000000002 2016-01-01
    ## 2: exports-fob-domestic 15.651199999999998 2016-02-01
    ## 3: exports-fob-domestic            208.036 2016-03-01
    ## 4: exports-fob-domestic  670.9549999999999 2016-04-01
    ## 5: exports-fob-domestic 207.89599999999996 2016-05-01
    ## 6: exports-fob-domestic 502.08099999999996 2016-06-01

Add metric key, frequency, country, category, source and property
informations.

``` r
# Metric 
ki.imts.bot[, 
            `:=` (frequency = "monthly", 
                  country = "ki", 
                  category = "trade", 
                  source = "imts", 
                  properties = "{'currency' = 'aud 000'}",
                  metric_key = paste0("ki","-",name,"-",date))]


head(ki.imts.bot)
```

    ##                    name              value       date frequency country
    ## 1: exports-fob-domestic 1859.5960000000002 2016-01-01   monthly      ki
    ## 2: exports-fob-domestic 15.651199999999998 2016-02-01   monthly      ki
    ## 3: exports-fob-domestic            208.036 2016-03-01   monthly      ki
    ## 4: exports-fob-domestic  670.9549999999999 2016-04-01   monthly      ki
    ## 5: exports-fob-domestic 207.89599999999996 2016-05-01   monthly      ki
    ## 6: exports-fob-domestic 502.08099999999996 2016-06-01   monthly      ki
    ##    category source               properties                         metric_key
    ## 1:    trade   imts {'currency' = 'aud 000'} ki-exports-fob-domestic-2016-01-01
    ## 2:    trade   imts {'currency' = 'aud 000'} ki-exports-fob-domestic-2016-02-01
    ## 3:    trade   imts {'currency' = 'aud 000'} ki-exports-fob-domestic-2016-03-01
    ## 4:    trade   imts {'currency' = 'aud 000'} ki-exports-fob-domestic-2016-04-01
    ## 5:    trade   imts {'currency' = 'aud 000'} ki-exports-fob-domestic-2016-05-01
    ## 6:    trade   imts {'currency' = 'aud 000'} ki-exports-fob-domestic-2016-06-01

Reorder data.

``` r
# Change order of columns to align with schema
ki.imts.bot <- ki.imts.bot[,
                           .(metric_key,
                             frequency, 
                             country,
                             category, 
                             source,
                             name,
                             date, 
                             value,
                             properties)]


head(ki.imts.bot)
```

    ##                            metric_key frequency country category source
    ## 1: ki-exports-fob-domestic-2016-01-01   monthly      ki    trade   imts
    ## 2: ki-exports-fob-domestic-2016-02-01   monthly      ki    trade   imts
    ## 3: ki-exports-fob-domestic-2016-03-01   monthly      ki    trade   imts
    ## 4: ki-exports-fob-domestic-2016-04-01   monthly      ki    trade   imts
    ## 5: ki-exports-fob-domestic-2016-05-01   monthly      ki    trade   imts
    ## 6: ki-exports-fob-domestic-2016-06-01   monthly      ki    trade   imts
    ##                    name       date              value               properties
    ## 1: exports-fob-domestic 2016-01-01 1859.5960000000002 {'currency' = 'aud 000'}
    ## 2: exports-fob-domestic 2016-02-01 15.651199999999998 {'currency' = 'aud 000'}
    ## 3: exports-fob-domestic 2016-03-01            208.036 {'currency' = 'aud 000'}
    ## 4: exports-fob-domestic 2016-04-01  670.9549999999999 {'currency' = 'aud 000'}
    ## 5: exports-fob-domestic 2016-05-01 207.89599999999996 {'currency' = 'aud 000'}
    ## 6: exports-fob-domestic 2016-06-01 502.08099999999996 {'currency' = 'aud 000'}

# Append to PostgresSQL Database

Connect to the hosted PostgreSQL database.

Table used is country\_metrics.

## Fiji IMTS Balance of Trade Upload

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# Count rows before upload
rc1 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs1 <- dbFetch(rc1)



# # Append fj.imts.bot to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "country_metrics",
#                         fj.imts.bot,
#                         append = TRUE,
#                         row.names = FALSE)


# Count rows after upload
rs2 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs2 <- dbFetch(rs2)


dbDisconnect(db.con)
```

## Solomon Islands IMTS Balance of Trade Upload

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# Count rows before upload
rc3 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs3 <- dbFetch(rc3)


# # Append sl.imts.bot to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "country_metrics",
#                         sl.imts.bot,
#                         append = TRUE,
#                         row.names = FALSE)


# Count rows after upload
rs4 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs4 <- dbFetch(rs4)


dbDisconnect(db.con)
```

## Cook Islands IMTS Balance of Trade Upload

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# Count rows before upload
rc5 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs5 <- dbFetch(rc5)


# # Append ck.imts.bot to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "country_metrics",
#                         ck.imts.bot,
#                         append = TRUE,
#                         row.names = FALSE)


# Count rows after upload
rs6 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs6 <- dbFetch(rs6)


dbDisconnect(db.con)
```

## Vanuatu IMTS VNS Balance of Trade Upload

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# Count rows before upload
rc7 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs7 <- dbFetch(rc7)


# # Append vn.imts.bot to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "country_metrics",
#                         vn.imts.bot,
#                         append = TRUE,
#                         row.names = FALSE)


# Count rows after upload
rs8 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs8 <- dbFetch(rs8)


dbDisconnect(db.con)
```

## Samoa IMTS SBS Balance of Trade Upload

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# Count rows before upload
rc9 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs9 <- dbFetch(rc9)


# # Append ws.imts.bot to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "country_metrics",
#                         ws.imts.bot,
#                         append = TRUE,
#                         row.names = FALSE)


# Count rows after upload
rs10 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs10 <- dbFetch(rs10)


dbDisconnect(db.con)
```

## Palau IMTS Balance of Trade Upload

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# Count rows before upload
rc11 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs11 <- dbFetch(rc11)


# # Append pl.imts.bot to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "country_metrics",
#                         pl.imts.bot,
#                         append = TRUE,
#                         row.names = FALSE)


# Count rows after upload
rs12 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs12 <- dbFetch(rs12)


dbDisconnect(db.con)
```

## Kiribati IMTS Balance of Trade Upload

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# Count rows before upload
rc13 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs13 <- dbFetch(rc13)


# # Append ki.imts.bot to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "country_metrics",
#                         ki.imts.bot,
#                         append = TRUE,
#                         row.names = FALSE)


# Count rows after upload
rs14 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs14 <- dbFetch(rs14)


dbDisconnect(db.con)
```

## Clean up

Remove temp variables used to test.

``` r
# Remove vars not needed.
rm(rc1,
   rs2, 
   rc3,
   rs4,
   rc5,
   rs6,
   rc7,
   rs8,
   rc9,
   rs10)
```

# Final Result: country\_metrics

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# Count rows before upload
country_metrics <- data.table(RPostgres::dbGetQuery(db.con, "SELECT * FROM country_metrics"))


# Preview
tail(country_metrics)
```

    ##                     metric_key frequency country category source          name
    ## 1: ki-trade-balance-2019-07-01   monthly      ki    trade   imts trade-balance
    ## 2: ki-trade-balance-2019-08-01   monthly      ki    trade   imts trade-balance
    ## 3: ki-trade-balance-2019-09-01   monthly      ki    trade   imts trade-balance
    ## 4: ki-trade-balance-2019-10-01   monthly      ki    trade   imts trade-balance
    ## 5: ki-trade-balance-2019-11-01   monthly      ki    trade   imts trade-balance
    ## 6: ki-trade-balance-2019-12-01   monthly      ki    trade   imts trade-balance
    ##          date        value               properties
    ## 1: 2019-07-01   -14239.733 {'currency' = 'aud 000'}
    ## 2: 2019-08-01 -13452.04065 {'currency' = 'aud 000'}
    ## 3: 2019-09-01 -13470.76348 {'currency' = 'aud 000'}
    ## 4: 2019-10-01 -15435.80864 {'currency' = 'aud 000'}
    ## 5: 2019-11-01 -11282.19025 {'currency' = 'aud 000'}
    ## 6: 2019-12-01 -13355.90965 {'currency' = 'aud 000'}
