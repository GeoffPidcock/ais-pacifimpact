Data Prep - Fiji AIS
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

# Read AIS Suva

Read the files extracted from UNGP for Suva, Fiji Islands.

``` r
# Suva files
file.list <- list.files(path = "./data/external/ais_fj_suva/", 
                        pattern='*.csv')


# Read files in bulk
suva.list <- lapply(paste0("./data/external/ais_fj_suva/",
                           file.list),
                  fread)
```

    ## Warning in FUN(X[[i]], ...): Found and resolved improper quoting out-
    ## of-sample. First healed line 9368: <<2019-10-13T21:46:10.000Z,
    ## 520290000,"RATU%SPRICI0JXH\" !2T",3DNG$$E,42,A,511,19,Fiji,
    ## 520,178.42631666666668,-18.131533333333334,POINT (178.42631666666668
    ## -18.131533333333334),HSC,"Carrying DG,HS or MP,IMO hazard or Pollutant Category
    ## Y",Tug,,DTU,08031657,6.4,0.1,61.6,0.0,294.0,Not Defined,15,S-AIS,1>>. If the
    ## fields are not quoted (e.g. field separator does not appear within any field),
    ## try quote="" to avoid this warning.

    ## Warning in FUN(X[[i]], ...): Found and resolved improper quoting out-of-
    ## sample. First healed line 76039: <<2020-05-07T20:19:51.000Z,542233110,PACIFIC
    ## 2,E6FN2,70,A,91,15,Niue,542,178.41962833333332,-18.13834,POINT
    ## (178.41962833333332 -18.13834),Cargo,,,,"\"",
    ## 00002460,0.0,0.5,162.4,0.0,0.0,Under Way Using Engine,0,T-AIS,3>>. If the fields
    ## are not quoted (e.g. field separator does not appear within any field), try
    ## quote="" to avoid this warning.

    ## Warning in FUN(X[[i]], ...): Found and resolved improper quoting out-of-
    ## sample. First healed line 14342: <<2019-10-10T19:31:03.000Z,412460295,SUI
    ## YUAN YU 30,BZWA32,30,B,35,4,China,412,178.41691166666666,-18.13216,POINT
    ## (178.41691166666666 -18.13216),Fishing,,,,"\"UW:UUUU>AFJ.^:0",
    ## 00002460,0.0,0.0,124.1,0.0,124.0,Unknown,16,S-AIS,18>>. If the fields are not
    ## quoted (e.g. field separator does not appear within any field), try quote="" to
    ## avoid this warning.

    ## Warning in FUN(X[[i]], ...): Found and resolved improper quoting out-
    ## of-sample. First healed line 210578: <<2019-10-14T16:59:13.000Z,
    ## 412460295,SUI YUAN YU 30,BZWA32,30,B,35,4,China,412,178.4257,-18.128335,POINT
    ## (178.4257 -18.128335),Fishing,,,,"\"UW:UUUU>AFJ.^:0",
    ## 00002460,0.0,0.0,273.9,0.0,273.0,Unknown,16,S-AIS,18>>. If the fields are not
    ## quoted (e.g. field separator does not appear within any field), try quote="" to
    ## avoid this warning.

``` r
# turn into one table 
suva.list <- rbindlist(suva.list)


head(suva.list)
```

    ##                         dtg      mmsi     vessel_name callsign vessel_type_code
    ## 1: 2020-01-15T05:38:55.000Z 520314000      HANGTON116     3DOF               30
    ## 2: 2020-01-15T05:40:54.000Z 520314000      HANGTON116     3DOF               30
    ## 3: 2020-01-15T05:41:56.000Z 412674190   XIN SHI JI 10    BZUU2               30
    ## 4: 2020-01-15T05:38:13.000Z 412331499 LURONGYUANYU297    BCLN7               30
    ## 5: 2020-01-15T05:38:16.000Z 412331499 LURONGYUANYU297    BCLN7               30
    ## 6: 2020-01-15T05:38:17.000Z 412549105 LURONGYUANYU357    BZTI5               30
    ##    vessel_class length width flag_country flag_code longitude  latitude
    ## 1:            B      0     0         Fiji       520  178.4176 -18.12352
    ## 2:            B      0     0         Fiji       520  178.4176 -18.12352
    ## 3:            B     49    10        China       412  178.4128 -18.14012
    ## 4:            A      2     2        China       412  178.4257 -18.12991
    ## 5:            A      2     2        China       412  178.4257 -18.12991
    ## 6:            A      2     2        China       412  178.4269 -18.12695
    ##                           UDF:st_asText(position) vessel_type vessel_type_cargo
    ## 1:         POINT (178.417625 -18.123516666666667)     Fishing                  
    ## 2:         POINT (178.417625 -18.123516666666667)     Fishing                  
    ## 3:           POINT (178.4128 -18.140116666666668)     Fishing                  
    ## 4:          POINT (178.42574666666667 -18.129905)     Fishing                  
    ## 5:          POINT (178.42574666666667 -18.129905)     Fishing                  
    ## 6: POINT (178.42685333333333 -18.126951666666667)     Fishing                  
    ##    vessel_type_main vessel_type_sub destination  eta draught sog  cog rot
    ## 1:                                              2460       0 3.6 97.7   0
    ## 2:                                              2460       0 3.6 97.7   0
    ## 3:                                            R 2460       0 7.9 65.7   0
    ## 4:                                            X 2460       0 6.3 82.7   0
    ## 5:                                            X 2460       0 0.1  0.0   0
    ## 6:                                              2460       0 0.0  0.0   0
    ##    heading             nav_status nav_status_code source message_type
    ## 1:       0                Unknown              16  T-AIS           18
    ## 2:       0                Unknown              16  T-AIS           18
    ## 3:       0     Engaged In Fishing               7  T-AIS           19
    ## 4:       0                 Moored               5  S-AIS            1
    ## 5:       0     Engaged In Fishing               7  S-AIS            1
    ## 6:       0 Under Way Using Engine               0  S-AIS            1

# Write AIS Suva to PostgreSQL

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# # Append ws.imts.bot to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "stg_ais",
#                         suva.list,
#                         append = FALSE,
#                         row.names = FALSE)


rm(suva.list)
```

# Read AIS Lautoka

Read the files extracted from UNGP for Lautoka, Fiji Islands.

``` r
# Suva files
file.list <- list.files(path = "./data/external/ais_fj_lautoka/", 
                        pattern='*.csv')


# Read files in bulk
lautoka.list <- lapply(paste0("./data/external/ais_fj_lautoka/",
                           file.list),
                  fread)
```

    ## Warning in FUN(X[[i]], ...): Found and resolved improper quoting out-
    ## of-sample. First healed line 77096: <<2019-11-07T02:11:38.000Z,
    ## 542233110,PACIFIC 2,E6FN2,70,A,91,15,Niue,542,177.4014,-17.6183,POINT (177.4014
    ## -17.6183),Cargo,,,,"\"",00002460,0.0,6.9,62.0,0.0,0.0,Under Way Using Engine,
    ## 0,S-AIS,3>>. If the fields are not quoted (e.g. field separator does not appear
    ## within any field), try quote="" to avoid this warning.

    ## Warning in FUN(X[[i]], ...): Found and resolved improper quoting out-of-
    ## sample. First healed line 26821: <<2019-11-20T20:26:46.000Z,542233110,PACIFIC
    ## 2,E6FN2,70,B,91,15,Niue,542,177.42257666666666,-17.612985,POINT
    ## (177.42257666666666 -17.612985),Cargo,,,,"\"",
    ## 00002460,0.0,1.9,256.1,0.0,0.0,Unknown,16,S-AIS,18>>. If the fields are not
    ## quoted (e.g. field separator does not appear within any field), try quote="" to
    ## avoid this warning.

    ## Warning in FUN(X[[i]], ...): Found and resolved improper quoting out-of-
    ## sample. First healed line 42265: <<2019-12-29T19:53:05.000Z,542233110,PACIFIC
    ## 2,E6FN2,70,A,91,15,Niue,542,177.45711666666668,-17.593416666666666,POINT
    ## (177.45711666666668 -17.593416666666666),Cargo,,,,"\"",
    ## 00002460,0.0,0.1,325.3,0.0,0.0,Under Way Using Engine,0,S-AIS,3>>. If the fields
    ## are not quoted (e.g. field separator does not appear within any field), try
    ## quote="" to avoid this warning.

    ## Warning in FUN(X[[i]], ...): Found and resolved improper quoting out-of-sample.
    ## First healed line 43746: <<2020-02-02T19:08:17.000Z,249817000,"DTH'IPHWUB\"6RM
    ## \"",9HA2020,80,A,120,22,Malta,249,177.43913833333335,-17.602935,POINT
    ## (177.43913833333335 -17.602935),Tanker,,Oil And Chemical Tanker,Chemical Oil
    ## Products Tanker,"$5DA$GXH(04N , !0",03150800,5.4,0.0,217.0,0.0,199.0,Moored,
    ## 5,S-AIS,3>>. If the fields are not quoted (e.g. field separator does not appear
    ## within any field), try quote="" to avoid this warning.

``` r
# turn into one table 
lautoka.list <- rbindlist(lautoka.list)


head(lautoka.list)
```

    ##                         dtg      mmsi      vessel_name callsign
    ## 1: 2020-08-15T13:06:04.000Z 477477400 NEW GUINEA CHIEF    VROO8
    ## 2: 2020-08-15T13:21:07.000Z 477477400 NEW GUINEA CHIEF    VROO8
    ## 3: 2020-08-15T13:30:06.000Z 477477400 NEW GUINEA CHIEF    VROO8
    ## 4: 2020-08-15T14:36:05.000Z 477477400 NEW GUINEA CHIEF    VROO8
    ## 5: 2020-08-15T15:00:06.000Z 477477400 NEW GUINEA CHIEF    VROO8
    ## 6: 2020-08-15T15:15:06.000Z 477477400 NEW GUINEA CHIEF    VROO8
    ##    vessel_type_code vessel_class length width flag_country flag_code longitude
    ## 1:               73            A    176    28    Hong Kong       477  177.4384
    ## 2:               73            A    176    28    Hong Kong       477  177.4384
    ## 3:               73            A    176    28    Hong Kong       477  177.4384
    ## 4:               73            A    176    28    Hong Kong       477  177.4384
    ## 5:               73            A    176    28    Hong Kong       477  177.4384
    ## 6:               73            A    176    28    Hong Kong       477  177.4384
    ##     latitude               UDF:st_asText(position) vessel_type
    ## 1: -17.60277 POINT (177.43845 -17.602766666666668)       Cargo
    ## 2: -17.60277 POINT (177.43845 -17.602766666666668)       Cargo
    ## 3: -17.60278 POINT (177.43845 -17.602783333333335)       Cargo
    ## 4: -17.60278 POINT (177.43845 -17.602783333333335)       Cargo
    ## 5: -17.60278 POINT (177.43845 -17.602783333333335)       Cargo
    ## 6: -17.60277 POINT (177.43845 -17.602766666666668)       Cargo
    ##                                          vessel_type_cargo vessel_type_main
    ## 1: Carrying DG,HS or MP,IMO hazard or Pollutant Category Z   Container Ship
    ## 2: Carrying DG,HS or MP,IMO hazard or Pollutant Category Z   Container Ship
    ## 3: Carrying DG,HS or MP,IMO hazard or Pollutant Category Z   Container Ship
    ## 4: Carrying DG,HS or MP,IMO hazard or Pollutant Category Z   Container Ship
    ## 5: Carrying DG,HS or MP,IMO hazard or Pollutant Category Z   Container Ship
    ## 6: Carrying DG,HS or MP,IMO hazard or Pollutant Category Z   Container Ship
    ##    vessel_type_sub destination     eta draught sog   cog rot heading nav_status
    ## 1:                     LAUTOKA 8150800     7.7 0.0 262.6   0     198     Moored
    ## 2:                     LAUTOKA 8150800     7.7 0.1 240.4   0     198     Moored
    ## 3:                     LAUTOKA 8150800     7.7 0.0 357.5   0     199     Moored
    ## 4:                     LAUTOKA 8150800     7.7 0.1 286.5   0     197     Moored
    ## 5:                     LAUTOKA 8150800     7.7 0.1  36.9   0     199     Moored
    ## 6:                     LAUTOKA 8150800     7.7 0.1  53.6   0     198     Moored
    ##    nav_status_code source message_type
    ## 1:               5  T-AIS            3
    ## 2:               5  T-AIS            3
    ## 3:               5  T-AIS            3
    ## 4:               5  T-AIS            3
    ## 5:               5  T-AIS            3
    ## 6:               5  T-AIS            3

# Write AIS Lautoka to PostgreSQL

``` r
# Connect to a specific postgres database
db.con <- dbConnect(RPostgres::Postgres(),
                    dbname = 'aishackathon', 
                    host = 'ais-hack.cirquhp75zcc.us-east-2.rds.amazonaws.com', 
                    port = 5432, 
                    user = Sys.getenv("userid"),
                    password = Sys.getenv("pwd"))


# # Append ws.imts.bot to country_metrics
# RPostgres::dbWriteTable(db.con,
#                         "stg_ais",
#                         lautoka.list,
#                         append = TRUE,
#                         row.names = FALSE)


count <- dbGetQuery(db.con, 'select count(*) from stg_ais')
# 8841715 records
```
