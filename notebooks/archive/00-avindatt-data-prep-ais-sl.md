Data Prep - Solomon Islands AIS
================
Avin Datt - PacifImpact
04 September 2020

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

# Read AIS Honiora

Read the files extracted from UNGP for Honiora, Solomon Islands.

``` r
# Honiora files
file.list <- list.files(path = "./data/external/ais_sl_honiora/", 
                        pattern='*.csv')


# Read files in bulk
honiora.list <- lapply(paste0("./data/external/ais_sl_honiora/",
                           file.list),
                  fread)


# turn into one table 
honiora.list <- rbindlist(honiora.list)


head(honiora.list)
```

    ##                         dtg      mmsi   vessel_name callsign vessel_type_code
    ## 1: 2019-09-11T10:56:43.000Z 440026270 UTA PRINCESS2     H4UB              100
    ## 2: 2019-09-11T10:57:04.000Z 440026270 UTA PRINCESS2     H4UB              100
    ## 3: 2019-09-11T11:00:35.000Z 440026270 UTA PRINCESS2     H4UB              100
    ## 4: 2019-09-11T10:53:31.000Z 533170293     FAIR KING     H4FG               70
    ## 5: 2019-09-11T10:54:31.000Z 533170293     FAIR KING     H4FG               70
    ## 6: 2019-09-11T10:55:43.000Z 557008900   TAIMAREHO 1     H4TN               55
    ##    vessel_class length width    flag_country flag_code longitude  latitude
    ## 1:            A      0     0     South Korea       440  159.9584 -9.429792
    ## 2:            A      0     0     South Korea       440  159.9584 -9.429792
    ## 3:            A      0     0     South Korea       440  159.9584 -9.429788
    ## 4:            A     50    12        Malaysia       533  160.0077 -9.422410
    ## 5:            A     50    12        Malaysia       533  160.0077 -9.422407
    ## 6:            A     66    10 Solomon Islands       557  159.9774 -9.425858
    ##                          UDF:st_asText(position)     vessel_type
    ## 1: POINT (159.95840333333334 -9.429791666666667)         Unknown
    ## 2:         POINT (159.958405 -9.429791666666667)         Unknown
    ## 3: POINT (159.95840833333332 -9.429788333333333)         Unknown
    ## 4:           POINT (160.00772833333335 -9.42241)           Cargo
    ## 5: POINT (160.00774666666666 -9.422406666666667)           Cargo
    ## 6: POINT (159.97742833333334 -9.425858333333334) Law Enforcement
    ##    vessel_type_cargo vessel_type_main vessel_type_sub destination     eta
    ## 1:                                                          YEOSU    2460
    ## 2:                                                          YEOSU    2460
    ## 3:                                                          YEOSU    2460
    ## 4:                                                      SUAFA-HON 7040600
    ## 5:                                                      SUAFA-HON 7040600
    ## 6:                                                          BUSAN 8121400
    ##    draught sog   cog rot heading             nav_status nav_status_code source
    ## 1:     0.0 0.2  90.4   0       0 Under Way Using Engine               0  S-AIS
    ## 2:     0.0 0.1  90.4   0       0 Under Way Using Engine               0  S-AIS
    ## 3:     0.0 0.1 349.5   0       0 Under Way Using Engine               0  S-AIS
    ## 4:     2.0 0.0   0.0   0     135 Under Way Using Engine               0  S-AIS
    ## 5:     2.0 0.0   0.0   0     135 Under Way Using Engine               0  S-AIS
    ## 6:     3.8 0.0   0.0   0      97            Not Defined              15  S-AIS
    ##    message_type
    ## 1:            1
    ## 2:            1
    ## 3:            1
    ## 4:            1
    ## 5:            1
    ## 6:            3

# Write AIS Honiora to PostgreSQL

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# # Append honiora.list to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "stg_ais",
#                         honiora.list,
#                         append = TRUE,
#                         row.names = FALSE)


rm(honiora.list)
```

# Read AIS Noro

Read the files extracted from UNGP for Noro, Solomon Islands.

``` r
# Noro files
file.list <- list.files(path = "./data/external/ais_sl_noro/", 
                        pattern='*.csv')


# Read files in bulk
noro.list <- lapply(paste0("./data/external/ais_sl_noro/",
                           file.list),
                  fread)


# turn into one table 
noro.list <- rbindlist(noro.list)


head(noro.list)
```

    ##                         dtg      mmsi    vessel_name callsign vessel_type_code
    ## 1: 2019-09-06T00:07:25.000Z 557008700 SOLOMON HUNTER     H4HC               30
    ## 2: 2019-09-06T00:10:05.000Z 319038000     NO COMMENT    ZGCW9              100
    ## 3: 2019-09-06T00:16:05.000Z 319038000     NO COMMENT    ZGCW9              100
    ## 4: 2019-09-06T01:02:44.000Z 416004453  YI SIANG NO.8                        30
    ## 5: 2019-09-06T01:04:04.000Z 319038000     NO COMMENT    ZGCW9              100
    ## 6: 2019-09-06T01:28:34.000Z 319038000     NO COMMENT    ZGCW9              100
    ##    vessel_class length width    flag_country flag_code longitude  latitude
    ## 1:            B     30     5 Solomon Islands       557  157.1966 -8.210468
    ## 2:            A     48     9  Cayman Islands       319  157.1950 -8.235000
    ## 3:            A     48     9  Cayman Islands       319  157.1950 -8.235000
    ## 4:            B      0     0          Taiwan       416  157.1988 -8.226127
    ## 5:            A     48     9  Cayman Islands       319  157.1950 -8.235000
    ## 6:            A     48     9  Cayman Islands       319  157.1950 -8.235000
    ##                          UDF:st_asText(position) vessel_type vessel_type_cargo
    ## 1: POINT (157.19656833333335 -8.210468333333333)     Fishing                  
    ## 2:                        POINT (157.195 -8.235)     Unknown                  
    ## 3:                        POINT (157.195 -8.235)     Unknown                  
    ## 4:          POINT (157.19877 -8.226126666666667)     Fishing                  
    ## 5:                        POINT (157.195 -8.235)     Unknown                  
    ## 6:                        POINT (157.195 -8.235)     Unknown                  
    ##    vessel_type_main vessel_type_sub destination      eta draught  sog   cog rot
    ## 1:              Tug                      LAVENA 10031500     3.0 10.3 192.9   0
    ## 2:   Pleasure Craft           Yacht        GIZO  9040800     2.4  0.0 161.0   0
    ## 3:   Pleasure Craft           Yacht        GIZO  9040800     2.4  0.0 162.0   0
    ## 4:                                                  2460     0.0  0.0 143.6   0
    ## 5:   Pleasure Craft           Yacht        GIZO  9040800     2.4  0.0 214.0   0
    ## 6:   Pleasure Craft           Yacht        GIZO  9040800     2.4  0.0 221.0   0
    ##    heading nav_status nav_status_code source message_type
    ## 1:       0    Unknown              16  S-AIS           18
    ## 2:       0     Moored               5  S-AIS           27
    ## 3:       0     Moored               5  S-AIS           27
    ## 4:       0    Unknown              16  S-AIS           18
    ## 5:       0     Moored               5  S-AIS           27
    ## 6:       0     Moored               5  S-AIS           27

# Write AIS Noro to PostgreSQL

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# # Append honiora.list to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "stg_ais",
#                         noro.list,
#                         append = TRUE,
#                         row.names = FALSE)


rm(noro.list)
```
