Data Prep - Cook Islands AIS
================
Avin Datt - PacifImpact
04 September 2020

# Scope

AIS data was acquired via UNGP but due to the fact cluster resources
were scarce, AIS data had to be extracted from Spark and then stored in
a database for the wider PacifImpact team to access.

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

# Read AIS Cook Islands.

Read the files extracted from UNGP for Cook Islands.

``` r
# Cook Islands. files
file.list <- list.files(path = "./data/external/ais_ck/", 
                        pattern='*.csv')


# Read files in bulk
cklist.list <- lapply(paste0("./data/external/ais_ck/",
                           file.list),
                  fread)


# turn into one table 
cklist.list <- rbindlist(cklist.list)


head(cklist.list)
```

    ##                         dtg      mmsi vessel_name callsign vessel_type_code
    ## 1: 2020-07-02T18:24:29.000Z 440691000       MEDIA     D7SI               80
    ## 2: 2020-07-02T18:26:17.000Z 518100443     GRINNA2  E5U3358               99
    ## 3: 2020-07-02T18:31:36.000Z 518100443     GRINNA2  E5U3358               99
    ## 4: 2020-07-02T18:33:18.000Z 518100443     GRINNA2  E5U3358               99
    ## 5: 2020-07-02T18:36:37.000Z 518100443     GRINNA2  E5U3358               99
    ## 6: 2020-07-02T18:43:58.000Z 518100443     GRINNA2  E5U3358               99
    ##    vessel_class length width flag_country flag_code longitude  latitude
    ## 1:            A    105    16  South Korea       440 -159.7842 -21.20419
    ## 2:            A     38     9 Cook Islands       518 -159.7847 -21.20507
    ## 3:            A     38     9 Cook Islands       518 -159.7847 -21.20506
    ## 4:            A     38     9 Cook Islands       518 -159.7847 -21.20507
    ## 5:            A     38     9 Cook Islands       518 -159.7847 -21.20507
    ## 6:            A     38     9 Cook Islands       518 -159.7847 -21.20506
    ##                            UDF:st_asText(position) vessel_type
    ## 1:                    POINT (-159.78419 -21.20419)      Tanker
    ## 2: POINT (-159.78471833333333 -21.205066666666667)       Other
    ## 3:  POINT (-159.78471666666667 -21.20506333333333)       Other
    ## 4:          POINT (-159.78470333333334 -21.205065)       Other
    ## 5:          POINT (-159.78471333333334 -21.205065)       Other
    ## 6:  POINT (-159.78471666666667 -21.20506333333333)       Other
    ##            vessel_type_cargo        vessel_type_main vessel_type_sub
    ## 1:                           Oil And Chemical Tanker Chemical Tanker
    ## 2: No Additional Information                                        
    ## 3: No Additional Information                                        
    ## 4: No Additional Information                                        
    ## 5: No Additional Information                                        
    ## 6: No Additional Information                                        
    ##    destination     eta draught sog  cog rot heading             nav_status
    ## 1:      CK RAR 7020200     5.9   0 17.2   0      14                 Moored
    ## 2:   RAROTONGA 6281600     3.1   0  0.0   0     129 Under Way Using Engine
    ## 3:   RAROTONGA 6281600     3.1   0  0.0   0     128 Under Way Using Engine
    ## 4:   RAROTONGA 6281600     3.1   0  0.0   0     128 Under Way Using Engine
    ## 5:   RAROTONGA 6281600     3.1   0  0.0   0     128 Under Way Using Engine
    ## 6:   RAROTONGA 6281600     3.1   0  0.0   0     127 Under Way Using Engine
    ##    nav_status_code source message_type
    ## 1:               5  T-AIS            3
    ## 2:               0  T-AIS            1
    ## 3:               0  T-AIS            1
    ## 4:               0  T-AIS            1
    ## 5:               0  T-AIS            1
    ## 6:               0  T-AIS            1

# Write AIS Cook Islands. to PostgreSQL

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# # Append cklist.list to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "stg_ais",
#                         cklist.list,
#                         append = TRUE,
#                         row.names = FALSE)


rm(cklist.list)
```
