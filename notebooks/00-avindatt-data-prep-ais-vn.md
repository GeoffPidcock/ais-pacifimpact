Data Prep - Vanuatu AIS
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

# Read AIS Port Vila

Read the files extracted from UNGP for Port Vila, Vanuatu.

``` r
# port.v.list files
file.list <- list.files(path = "./data/external/ais_vn_port_vila/", 
                        pattern='*.csv')


# Read files in bulk
port.v.list <- lapply(paste0("./data/external/ais_vn_port_vila/",
                           file.list),
                  fread)


# turn into one table 
port.v.list <- rbindlist(port.v.list)


head(port.v.list)
```

    ##                         dtg      mmsi vessel_name callsign vessel_type_code
    ## 1: 2020-03-27T09:06:19.000Z 557000200  RSIPV AUKI     H4MS               37
    ## 2: 2020-03-27T10:54:20.000Z 557000200  RSIPV AUKI     H4MS               37
    ## 3: 2020-03-27T11:21:21.000Z 557000200  RSIPV AUKI     H4MS               37
    ## 4: 2020-03-27T11:54:20.000Z 557000200  RSIPV AUKI     H4MS               37
    ## 5: 2020-03-27T15:06:20.000Z 557000200  RSIPV AUKI     H4MS               37
    ## 6: 2020-03-27T17:39:38.000Z 557000200  RSIPV AUKI     H4MS               37
    ##    vessel_class length width    flag_country flag_code longitude  latitude
    ## 1:            B     31    10 Solomon Islands       557  168.3117 -17.75527
    ## 2:            B     31    10 Solomon Islands       557  168.3117 -17.75525
    ## 3:            B     31    10 Solomon Islands       557  168.3116 -17.75526
    ## 4:            B     31    10 Solomon Islands       557  168.3116 -17.75525
    ## 5:            B     31    10 Solomon Islands       557  168.3117 -17.75528
    ## 6:            B     31    10 Solomon Islands       557  168.3117 -17.75528
    ##                           UDF:st_asText(position)    vessel_type
    ## 1: POINT (168.31166166666668 -17.755271666666665) Pleasure Craft
    ## 2:                  POINT (168.311665 -17.755245) Pleasure Craft
    ## 3: POINT (168.31162333333333 -17.755261666666666) Pleasure Craft
    ## 4:                   POINT (168.31162 -17.755255) Pleasure Craft
    ## 5: POINT (168.31165333333334 -17.755283333333335) Pleasure Craft
    ## 6:          POINT (168.31168 -17.755278333333333) Pleasure Craft
    ##    vessel_type_cargo vessel_type_main vessel_type_sub destination  eta draught
    ## 1:                                                                2460       0
    ## 2:                                                                2460       0
    ## 3:                                                                2460       0
    ## 4:                                                                2460       0
    ## 5:                                                                2460       0
    ## 6:                                                                2460       0
    ##    sog   cog rot heading nav_status nav_status_code source message_type
    ## 1: 0.0 251.1   0     210    Unknown              16  S-AIS           18
    ## 2: 0.0 134.6   0     210    Unknown              16  S-AIS           18
    ## 3: 0.1  81.2   0     210    Unknown              16  S-AIS           18
    ## 4: 0.0 261.3   0     210    Unknown              16  S-AIS           18
    ## 5: 0.0 275.7   0     211    Unknown              16  S-AIS           18
    ## 6: 0.0 253.8   0     211    Unknown              16  S-AIS           18

# Write AIS Port Vila to PostgreSQL

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# # Append port.v.list to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "stg_ais",
#                         port.v.list,
#                         append = TRUE,
#                         row.names = FALSE)


rm(port.v.list)
```

# Read AIS Luganville

Read the files extracted from UNGP for Luganville, Vanuatu.

``` r
# lugv.list files
file.list <- list.files(path = "./data/external/ais_vn_luganville/", 
                        pattern='*.csv')


# Read files in bulk
lugv.list <- lapply(paste0("./data/external/ais_vn_luganville/",
                           file.list),
                  fread)


# turn into one table 
lugv.list <- rbindlist(lugv.list)


head(lugv.list)
```

    ##                         dtg      mmsi   vessel_name callsign vessel_type_code
    ## 1: 2020-07-31T06:25:44.000Z 503056000 TIWI ISLANDER     VHJM               90
    ## 2: 2020-07-31T07:40:44.000Z 503056000 TIWI ISLANDER     VHJM               90
    ## 3: 2020-07-31T10:16:10.000Z 503056000 TIWI ISLANDER     VHJM               90
    ## 4: 2020-07-31T10:25:12.000Z 503056000 TIWI ISLANDER     VHJM               90
    ## 5: 2020-07-31T11:13:06.000Z 503056000 TIWI ISLANDER     VHJM               90
    ## 6: 2020-07-31T11:19:06.000Z 503056000 TIWI ISLANDER     VHJM               90
    ##    vessel_class length width flag_country flag_code longitude  latitude
    ## 1:            B     20     5    Australia       503  167.1595 -15.52493
    ## 2:            B     20     5    Australia       503  167.1594 -15.52493
    ## 3:            B     20     5    Australia       503  167.1594 -15.52493
    ## 4:            B     20     5    Australia       503  167.1594 -15.52494
    ## 5:            B     20     5    Australia       503  167.1594 -15.52490
    ## 6:            B     20     5    Australia       503  167.1594 -15.52489
    ##                           UDF:st_asText(position) vessel_type vessel_type_cargo
    ## 1: POINT (167.15949833333335 -15.524928333333333)       Other                  
    ## 2: POINT (167.15941166666667 -15.524926666666667)       Other                  
    ## 3: POINT (167.15941666666666 -15.524926666666667)       Other                  
    ## 4: POINT (167.15943666666666 -15.524936666666667)       Other                  
    ## 5:         POINT (167.159425 -15.524901666666667)       Other                  
    ## 6:         POINT (167.159425 -15.524893333333333)       Other                  
    ##    vessel_type_main vessel_type_sub destination  eta draught sog   cog rot
    ## 1:            Other   Landing Craft             2460       0   0 256.0   0
    ## 2:            Other   Landing Craft             2460       0   0  64.9   0
    ## 3:            Other   Landing Craft             2460       0   0 213.7   0
    ## 4:            Other   Landing Craft             2460       0   0 213.7   0
    ## 5:            Other   Landing Craft             2460       0   0 225.3   0
    ## 6:            Other   Landing Craft             2460       0   0 225.3   0
    ##    heading nav_status nav_status_code source message_type
    ## 1:       0    Unknown              16  T-AIS           18
    ## 2:       0    Unknown              16  T-AIS           18
    ## 3:       0    Unknown              16  T-AIS           18
    ## 4:       0    Unknown              16  T-AIS           18
    ## 5:       0    Unknown              16  T-AIS           18
    ## 6:       0    Unknown              16  T-AIS           18

# Write AIS Luganville to PostgreSQL

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# Append lugv.list to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "stg_ais",
#                         lugv.list,
#                         append = TRUE,
#                         row.names = FALSE)


count <- dbGetQuery(db.con, 'select count(*) from stg_ais')
# 14013983 records
```
