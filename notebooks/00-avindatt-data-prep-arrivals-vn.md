Data Prep - Vanuatu Visitor Arrivals
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

# Read Vanuatu Arrivals Data

``` r
# Read in Fiji File 
vn.arrivals <- read.xlsx(xlsxFile = './data/interim/vanuatu-arrivals.xlsx',
                         sheet = 1)

head(vn.arrivals)
```

    ##   year month total-visitors
    ## 1 2019   Mar          19263
    ## 2 2019   Apr          19185
    ## 3 2019   May          15819
    ## 4 2019   Jun          18162
    ## 5 2019   Jul          23340
    ## 6 2019   Aug          20677

Melt the table to adhere to the key-value pair schema design.

``` r
# Melt the data to adhere to the schema design
vn.arrivals <- data.table(vn.arrivals)


vn.arrivals <- melt.data.table(vn.arrivals,
                               id = c("year",
                                      "month"), measure = c("total-visitors"))

head(vn.arrivals)
```

    ##    year month       variable value
    ## 1: 2019   Mar total-visitors 19263
    ## 2: 2019   Apr total-visitors 19185
    ## 3: 2019   May total-visitors 15819
    ## 4: 2019   Jun total-visitors 18162
    ## 5: 2019   Jul total-visitors 23340
    ## 6: 2019   Aug total-visitors 20677

Create Date from year and month.

``` r
# Concatenate
vn.arrivals[, 
            date := paste0(gsub(" ", "", year, fixed = TRUE),
                           "-",
                           gsub(" ", "",month, fixed = TRUE),
                           "-",
                           "01")]


# Format Date
vn.arrivals[, 
            date := as.Date(date, format = "%Y-%B-%d")]


# Remove year and month fields 
vn.arrivals[, 
            year := NULL][, 
                          month := NULL]


head(vn.arrivals)
```

    ##          variable value       date
    ## 1: total-visitors 19263 2019-03-01
    ## 2: total-visitors 19185 2019-04-01
    ## 3: total-visitors 15819 2019-05-01
    ## 4: total-visitors 18162 2019-06-01
    ## 5: total-visitors 23340 2019-07-01
    ## 6: total-visitors 20677 2019-08-01

Rename “variable” to “name”.

``` r
# Rename variable to name 
setnames(vn.arrivals, 
         "variable",
         "name")


head(vn.arrivals)
```

    ##              name value       date
    ## 1: total-visitors 19263 2019-03-01
    ## 2: total-visitors 19185 2019-04-01
    ## 3: total-visitors 15819 2019-05-01
    ## 4: total-visitors 18162 2019-06-01
    ## 5: total-visitors 23340 2019-07-01
    ## 6: total-visitors 20677 2019-08-01

Add metric key, frequency, country, category, source and property
informations.

``` r
# Metric 
vn.arrivals[, 
            `:=` (frequency = "monthly", 
                  country = "vn", 
                  category = "tourism", 
                  source = "vnso", 
                  properties = "",
                  metric_key = paste0("vn","-",name,"-",date))]


head(vn.arrivals)
```

    ##              name value       date frequency country category source properties
    ## 1: total-visitors 19263 2019-03-01   monthly      vn  tourism   vnso           
    ## 2: total-visitors 19185 2019-04-01   monthly      vn  tourism   vnso           
    ## 3: total-visitors 15819 2019-05-01   monthly      vn  tourism   vnso           
    ## 4: total-visitors 18162 2019-06-01   monthly      vn  tourism   vnso           
    ## 5: total-visitors 23340 2019-07-01   monthly      vn  tourism   vnso           
    ## 6: total-visitors 20677 2019-08-01   monthly      vn  tourism   vnso           
    ##                      metric_key
    ## 1: vn-total-visitors-2019-03-01
    ## 2: vn-total-visitors-2019-04-01
    ## 3: vn-total-visitors-2019-05-01
    ## 4: vn-total-visitors-2019-06-01
    ## 5: vn-total-visitors-2019-07-01
    ## 6: vn-total-visitors-2019-08-01

Reorder data.

``` r
# Change order of columns to align with schema
vn.arrivals <- vn.arrivals[,
                           .(metric_key,
                             frequency, 
                             country,
                             category, 
                             source,
                             name,
                             date, 
                             value,
                             properties)]


head(vn.arrivals)
```

    ##                      metric_key frequency country category source
    ## 1: vn-total-visitors-2019-03-01   monthly      vn  tourism   vnso
    ## 2: vn-total-visitors-2019-04-01   monthly      vn  tourism   vnso
    ## 3: vn-total-visitors-2019-05-01   monthly      vn  tourism   vnso
    ## 4: vn-total-visitors-2019-06-01   monthly      vn  tourism   vnso
    ## 5: vn-total-visitors-2019-07-01   monthly      vn  tourism   vnso
    ## 6: vn-total-visitors-2019-08-01   monthly      vn  tourism   vnso
    ##              name       date value properties
    ## 1: total-visitors 2019-03-01 19263           
    ## 2: total-visitors 2019-04-01 19185           
    ## 3: total-visitors 2019-05-01 15819           
    ## 4: total-visitors 2019-06-01 18162           
    ## 5: total-visitors 2019-07-01 23340           
    ## 6: total-visitors 2019-08-01 20677

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


# # Append vn.arrivals to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "country_metrics",
#                         vn.arrivals,
#                         append = TRUE,
#                         row.names = FALSE)


# Count rows after upload
rs2 <- RPostgres::dbSendQuery(db.con, "SELECT count(*) as count FROM country_metrics")


count_recs2 <- dbFetch(rs2)


dbDisconnect(db.con)
```
