Data Prep - Palau Visitor Arrivals
================
Avin Datt - PacifImpact
03 September 2020

# Load Libraries

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

``` r
# Load openxlsx library 
if (!require(openxlsx)) {
  install.packages("openxlsx")
  library(openxlsx)
}
```

    ## Loading required package: openxlsx

# Read Palau Arrivals Data

``` r
# Read in Fiji File 
pl.arrivals <- read.xlsx(xlsxFile = './data/interim/palau-arrivals.xlsx',
                         sheet = 1)

head(pl.arrivals)
```

    ##   year month total-visitors
    ## 1 2007   Jun           6729
    ## 2 2007   Jul           8040
    ## 3 2007   Aug           9214
    ## 4 2007   Sep           7609
    ## 5 2007   Oct           6817
    ## 6 2007   Nov           6274

Melt the table to adhere to the key-value pair schema design.

``` r
# Melt the data to adhere to the schema design
pl.arrivals <- data.table(pl.arrivals)


pl.arrivals <- melt.data.table(pl.arrivals,
                               id = c("year",
                                      "month"), measure = c("total-visitors"))

head(pl.arrivals)
```

    ##    year month       variable value
    ## 1: 2007   Jun total-visitors  6729
    ## 2: 2007   Jul total-visitors  8040
    ## 3: 2007   Aug total-visitors  9214
    ## 4: 2007   Sep total-visitors  7609
    ## 5: 2007   Oct total-visitors  6817
    ## 6: 2007   Nov total-visitors  6274

Create Date from year and month.

``` r
# Concatenate
pl.arrivals[, 
            date := paste0(gsub(" ", "", year, fixed = TRUE),
                           "-",
                           gsub(" ", "",month, fixed = TRUE),
                           "-",
                           "01")]


# Format Date
pl.arrivals[, 
            date := as.Date(date, format = "%Y-%B-%d")]


# Remove year and month fields 
pl.arrivals[, 
            year := NULL][, 
                          month := NULL]


head(pl.arrivals)
```

    ##          variable value       date
    ## 1: total-visitors  6729 2007-06-01
    ## 2: total-visitors  8040 2007-07-01
    ## 3: total-visitors  9214 2007-08-01
    ## 4: total-visitors  7609 2007-09-01
    ## 5: total-visitors  6817 2007-10-01
    ## 6: total-visitors  6274 2007-11-01

Rename “variable” to “name”.

``` r
# Rename variable to name 
setnames(pl.arrivals, 
         "variable",
         "name")


head(pl.arrivals)
```

    ##              name value       date
    ## 1: total-visitors  6729 2007-06-01
    ## 2: total-visitors  8040 2007-07-01
    ## 3: total-visitors  9214 2007-08-01
    ## 4: total-visitors  7609 2007-09-01
    ## 5: total-visitors  6817 2007-10-01
    ## 6: total-visitors  6274 2007-11-01

Add metric key, frequency, country, category, source and property
informations.

``` r
# Metric 
pl.arrivals[, 
            `:=` (frequency = "monthly", 
                  country = "vn", 
                  category = "tourism", 
                  source = "vnso", 
                  properties = "",
                  metric_key = paste0("vn","-",name,"-",date))]


head(pl.arrivals)
```

    ##              name value       date frequency country category source properties
    ## 1: total-visitors  6729 2007-06-01   monthly      vn  tourism   vnso           
    ## 2: total-visitors  8040 2007-07-01   monthly      vn  tourism   vnso           
    ## 3: total-visitors  9214 2007-08-01   monthly      vn  tourism   vnso           
    ## 4: total-visitors  7609 2007-09-01   monthly      vn  tourism   vnso           
    ## 5: total-visitors  6817 2007-10-01   monthly      vn  tourism   vnso           
    ## 6: total-visitors  6274 2007-11-01   monthly      vn  tourism   vnso           
    ##                      metric_key
    ## 1: vn-total-visitors-2007-06-01
    ## 2: vn-total-visitors-2007-07-01
    ## 3: vn-total-visitors-2007-08-01
    ## 4: vn-total-visitors-2007-09-01
    ## 5: vn-total-visitors-2007-10-01
    ## 6: vn-total-visitors-2007-11-01

Reorder data.

``` r
# Change order of columns to align with schema
pl.arrivals <- pl.arrivals[,
                           .(metric_key,
                             frequency, 
                             country,
                             category, 
                             source,
                             name,
                             date, 
                             value,
                             properties)]


head(pl.arrivals)
```

    ##                      metric_key frequency country category source
    ## 1: vn-total-visitors-2007-06-01   monthly      vn  tourism   vnso
    ## 2: vn-total-visitors-2007-07-01   monthly      vn  tourism   vnso
    ## 3: vn-total-visitors-2007-08-01   monthly      vn  tourism   vnso
    ## 4: vn-total-visitors-2007-09-01   monthly      vn  tourism   vnso
    ## 5: vn-total-visitors-2007-10-01   monthly      vn  tourism   vnso
    ## 6: vn-total-visitors-2007-11-01   monthly      vn  tourism   vnso
    ##              name       date value properties
    ## 1: total-visitors 2007-06-01  6729           
    ## 2: total-visitors 2007-07-01  8040           
    ## 3: total-visitors 2007-08-01  9214           
    ## 4: total-visitors 2007-09-01  7609           
    ## 5: total-visitors 2007-10-01  6817           
    ## 6: total-visitors 2007-11-01  6274

# Write Arrivals to Database

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


# # Append pl.arrivals to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "country_metrics",
#                         pl.arrivals,
#                         append = TRUE,
#                         row.names = FALSE)


# Count rows after upload
rs2 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs2 <- dbFetch(rs2)


dbDisconnect(db.con)
```
