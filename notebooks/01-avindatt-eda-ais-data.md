EDA - Pacific AIS Data
================
Avin Datt - PacifImpact
06 September 2020

# Scope

1.  Explore data features. Understand summary and values.
2.  Apply algorithms to reduce noise as outlined by
    [UNSTATS](https://unstats.un.org/wiki/display/AIS/AIS+data+at+the+UN+Global+Platform).
3.  Create new data features using draught, time and other dimensions.
4.  Create common model to share that can be used in conjunction with
    trade, and economic research by PacifImpact.

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
# Load SmartEDA library
if (!require(SmartEDA)) {
  install.packages("SmartEDA")
  library(SmartEDA)
}
```

    ## Loading required package: SmartEDA

    ## Registered S3 method overwritten by 'GGally':
    ##   method from   
    ##   +.gg   ggplot2

# Read AIS Cook Islands.

Read the files extracted from UNGP for Cook Islands.

``` r
# Cook Islands. files
file.list <- list.files(
  path = "./data/external/ais_ck/", 
  pattern='*.csv'
  )


# Read files in bulk
cklist.list <- lapply(
  paste0("./data/external/ais_ck/",
         file.list),
  fread
  )


# turn into one table 
cklist.list <- rbindlist(
  cklist.list
  )


head(cklist.list)
```

<div class="kable-table">

| dtg                      |      mmsi | vessel\_name | callsign | vessel\_type\_code | vessel\_class | length | width | flag\_country | flag\_code |  longitude |   latitude | UDF:st\_asText(position)                        | vessel\_type | vessel\_type\_cargo       | vessel\_type\_main      | vessel\_type\_sub | destination |     eta | draught | sog |  cog | rot | heading | nav\_status            | nav\_status\_code | source | message\_type |
| :----------------------- | --------: | :----------- | :------- | -----------------: | :------------ | -----: | ----: | :------------ | ---------: | ---------: | ---------: | :---------------------------------------------- | :----------- | :------------------------ | :---------------------- | :---------------- | :---------- | ------: | ------: | --: | ---: | --: | ------: | :--------------------- | ----------------: | :----- | ------------: |
| 2020-07-02T18:24:29.000Z | 440691000 | MEDIA        | D7SI     |                 80 | A             |    105 |    16 | South Korea   |        440 | \-159.7842 | \-21.20419 | POINT (-159.78419 -21.20419)                    | Tanker       |                           | Oil And Chemical Tanker | Chemical Tanker   | CK RAR      | 7020200 |     5.9 |   0 | 17.2 |   0 |      14 | Moored                 |                 5 | T-AIS  |             3 |
| 2020-07-02T18:26:17.000Z | 518100443 | GRINNA2      | E5U3358  |                 99 | A             |     38 |     9 | Cook Islands  |        518 | \-159.7847 | \-21.20507 | POINT (-159.78471833333333 -21.205066666666667) | Other        | No Additional Information |                         |                   | RAROTONGA   | 6281600 |     3.1 |   0 |  0.0 |   0 |     129 | Under Way Using Engine |                 0 | T-AIS  |             1 |
| 2020-07-02T18:31:36.000Z | 518100443 | GRINNA2      | E5U3358  |                 99 | A             |     38 |     9 | Cook Islands  |        518 | \-159.7847 | \-21.20506 | POINT (-159.78471666666667 -21.20506333333333)  | Other        | No Additional Information |                         |                   | RAROTONGA   | 6281600 |     3.1 |   0 |  0.0 |   0 |     128 | Under Way Using Engine |                 0 | T-AIS  |             1 |
| 2020-07-02T18:33:18.000Z | 518100443 | GRINNA2      | E5U3358  |                 99 | A             |     38 |     9 | Cook Islands  |        518 | \-159.7847 | \-21.20507 | POINT (-159.78470333333334 -21.205065)          | Other        | No Additional Information |                         |                   | RAROTONGA   | 6281600 |     3.1 |   0 |  0.0 |   0 |     128 | Under Way Using Engine |                 0 | T-AIS  |             1 |
| 2020-07-02T18:36:37.000Z | 518100443 | GRINNA2      | E5U3358  |                 99 | A             |     38 |     9 | Cook Islands  |        518 | \-159.7847 | \-21.20507 | POINT (-159.78471333333334 -21.205065)          | Other        | No Additional Information |                         |                   | RAROTONGA   | 6281600 |     3.1 |   0 |  0.0 |   0 |     128 | Under Way Using Engine |                 0 | T-AIS  |             1 |
| 2020-07-02T18:43:58.000Z | 518100443 | GRINNA2      | E5U3358  |                 99 | A             |     38 |     9 | Cook Islands  |        518 | \-159.7847 | \-21.20506 | POINT (-159.78471666666667 -21.20506333333333)  | Other        | No Additional Information |                         |                   | RAROTONGA   | 6281600 |     3.1 |   0 |  0.0 |   0 |     127 | Under Way Using Engine |                 0 | T-AIS  |             1 |

</div>

## Exploration

1.  Convert dtg and eta to as.POSIXct(x, tz = "", …)

<!-- end list -->

``` r
# Change from char to posixct 
cklist.list[,
            `:=` (dtg = gsub("T",
                             " ",
                             dtg
                             )
                  )][,
                     `:=` (dtg = gsub(".000Z",
                             "",
                             dtg
                             )
                           )]


cklist.list[,
            `:=` (dtg = strptime(dtg, "%Y-%m-%d %H:%M:%S")
                  )]
```

    ## Warning in strptime(dtg, "%Y-%m-%d %H:%M:%S"): strptime() usage detected and
    ## wrapped with as.POSIXct(). This is to minimize the chance of assigning POSIXlt
    ## columns, which use 40+ bytes to store one date (versus 8 for POSIXct). Use
    ## as.POSIXct() (which will call strptime() as needed internally) to avoid this
    ## warning.

``` r
# check summary 
cklist.list[,
            summary(dtg)]
```

    ##                  Min.               1st Qu.                Median 
    ## "2019-09-06 00:00:02" "2019-10-15 01:33:28" "2019-12-25 19:29:51" 
    ##                  Mean               3rd Qu.                  Max. 
    ## "2020-01-17 06:06:41" "2020-03-18 12:16:00" "2020-09-01 00:37:12" 
    ##                  NA's 
    ##                   "2"

``` r
# Check out dataset quality
ExpData(
  data = cklist.list,
  type = 1
  )
```

<div class="kable-table">

| Descriptions                                          | Value       |
| :---------------------------------------------------- | :---------- |
| Sample size (nrow)                                    | 141833      |
| No. of variables (ncol)                               | 28          |
| No. of numeric/interger variables                     | 15          |
| No. of factor variables                               | 0           |
| No. of text variables                                 | 12          |
| No. of logical variables                              | 0           |
| No. of identifier variables                           | 0           |
| No. of date variables                                 | 1           |
| No. of zero variance variables (uniform)              | 0           |
| %. of variables having complete cases                 | 71.43% (20) |
| %. of variables having \>0% and \<50% missing cases   | 21.43% (6)  |
| %. of variables having \>=50% and \<90% missing cases | 7.14% (2)   |
| %. of variables having \>=90% missing cases           | 0% (0)      |

</div>

``` r
# Check out data feature quality
ExpData(
  data = cklist.list,
  type = 2
  )
```

<div class="kable-table">

| Index | Variable\_Name           | Variable\_Type | Per\_of\_Missing | No\_of\_distinct\_values |
| ----: | :----------------------- | :------------- | ---------------: | -----------------------: |
|     1 | dtg                      | POSIXct:POSIXt |          0.00001 |                   141313 |
|     2 | mmsi                     | integer        |          0.00000 |                       80 |
|     3 | vessel\_name             | character      |          0.00000 |                       86 |
|     4 | callsign                 | character      |          0.01631 |                       60 |
|     5 | vessel\_type\_code       | numeric        |          0.00009 |                       18 |
|     6 | vessel\_class            | character      |          0.00000 |                        2 |
|     7 | length                   | integer        |          0.00000 |                       59 |
|     8 | width                    | integer        |          0.00000 |                       27 |
|     9 | flag\_country            | character      |          0.03465 |                       40 |
|    10 | flag\_code               | integer        |          0.00000 |                       50 |
|    11 | longitude                | numeric        |          0.00000 |                     5067 |
|    12 | latitude                 | numeric        |          0.00000 |                     6567 |
|    13 | UDF:st\_asText(position) | character      |          0.00000 |                    41657 |
|    14 | vessel\_type             | character      |          0.00000 |                       13 |
|    15 | vessel\_type\_cargo      | character      |          0.45586 |                        4 |
|    16 | vessel\_type\_main       | character      |          0.55995 |                       12 |
|    17 | vessel\_type\_sub        | character      |          0.84972 |                       10 |
|    18 | destination              | character      |          0.03654 |                       67 |
|    19 | eta                      | integer        |          0.00000 |                      166 |
|    20 | draught                  | numeric        |          0.00000 |                       38 |
|    21 | sog                      | numeric        |          0.00000 |                      124 |
|    22 | cog                      | numeric        |          0.00000 |                     3591 |
|    23 | rot                      | numeric        |          0.00000 |                       56 |
|    24 | heading                  | numeric        |          0.00000 |                      360 |
|    25 | nav\_status              | character      |          0.00000 |                        5 |
|    26 | nav\_status\_code        | integer        |          0.00000 |                        5 |
|    27 | source                   | character      |          0.00000 |                        2 |
|    28 | message\_type            | integer        |          0.00000 |                        4 |

</div>

``` r
# Statistics for Numerical features 
data.table(
  ExpNumStat(
    cklist.list,
    by = "A",
    gp = NULL,
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)
```

<div class="kable-table">

| Vname              | Group |     TN |   nNeg |  nZero |   nPos | NegInf | PosInf | NA\_Value | Per\_of\_Missing |            sum |           min |           max |          mean |        median |           SD |        CV |          IQR | Skewness | Kurtosis |         LB.25% |        UB.75% | nOutliers |
| :----------------- | :---- | -----: | -----: | -----: | -----: | -----: | -----: | --------: | ---------------: | -------------: | ------------: | ------------: | ------------: | ------------: | -----------: | --------: | -----------: | -------: | -------: | -------------: | ------------: | --------: |
| cog                | All   | 141833 |      0 |  15387 | 126446 |      0 |      0 |         0 |            0.000 |   2.648614e+07 |         0.000 |       359.900 |       186.742 |       204.400 | 1.166830e+02 |     0.625 | 1.932000e+02 |  \-0.384 |  \-1.287 |      \-186.100 |       586.700 |         0 |
| draught            | All   | 141833 |      0 |   6512 | 135321 |      0 |      0 |         0 |            0.000 |   6.416894e+05 |         0.000 |        12.900 |         4.524 |         3.100 | 2.178000e+00 |     0.481 | 3.900000e+00 |    0.106 |  \-1.093 |        \-2.850 |        12.750 |        16 |
| eta                | All   | 141833 |      0 |      0 | 141833 |      0 |      0 |         0 |            0.000 |   1.011480e+12 |      2460.000 |  12311800.000 |   7131486.726 |   8260830.000 | 3.838625e+06 |     0.538 | 6.950300e+06 |  \-0.339 |  \-1.191 |  \-7254250.000 |  20546950.000 |         0 |
| flag\_code         | All   | 141833 |      0 |      0 | 141833 |      0 |      0 |         0 |            0.000 |   6.025123e+07 |       219.000 |       982.000 |       424.804 |       518.000 | 1.203680e+02 |     0.283 | 2.140000e+02 |  \-0.006 |  \-0.151 |       \-17.000 |       839.000 |       499 |
| heading            | All   | 141833 |      0 |  21461 | 120372 |      0 |      0 |         0 |            0.000 |   1.375850e+07 |         0.000 |       359.000 |        97.005 |       112.000 | 9.274400e+01 |     0.956 | 1.130000e+02 |    0.977 |    0.364 |      \-156.500 |       295.500 |      8029 |
| latitude           | All   | 141833 | 141833 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-3.007341e+06 |      \-21.206 |      \-21.190 |      \-21.203 |      \-21.205 | 3.000000e-03 |     0.000 | 1.000000e-03 |    2.294 |    4.072 |       \-21.206 |      \-21.203 |     21101 |
| length             | All   | 141833 |      0 |  12539 | 129294 |      0 |      0 |         0 |            0.000 |   1.089040e+07 |         0.000 |       490.000 |        76.783 |        38.000 | 6.055300e+01 |     0.789 | 8.000000e+01 |    1.117 |    1.087 |       \-82.000 |       238.000 |       825 |
| longitude          | All   | 141833 | 141833 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-2.266268e+07 |     \-159.797 |     \-159.775 |     \-159.784 |     \-159.785 | 1.000000e-03 |     0.000 | 0.000000e+00 |    3.092 |   21.606 |      \-159.785 |     \-159.784 |     22849 |
| mmsi               | All   | 141833 |      0 |      0 | 141833 |      0 |      0 |         0 |            0.000 |   6.030233e+13 | 219018897.000 | 982449587.000 | 425164326.211 | 518100443.000 | 1.201826e+08 |     0.283 | 2.132134e+08 |  \-0.005 |  \-0.137 | \-14933164.500 | 837920607.500 |       499 |
| rot                | All   | 141833 |    847 | 140304 |    682 |      0 |      0 |         0 |            0.000 | \-5.833590e+02 |      \-57.854 |        45.712 |       \-0.004 |         0.000 | 1.667000e+00 | \-405.360 | 0.000000e+00 |  \-0.210 |  319.920 |          0.000 |         0.000 |      1529 |
| sog                | All   | 141833 |      0 |  58099 |  83734 |      0 |      0 |         0 |            0.000 |   7.221630e+04 |         0.000 |        22.500 |         0.509 |         0.100 | 1.697000e+00 |     3.333 | 2.000000e-01 |    5.729 |   41.116 |        \-0.300 |         0.500 |     15627 |
| vessel\_type\_code | All   | 141833 |      0 |     14 | 141806 |      0 |      0 |        13 |            0.009 |   1.124205e+07 |         0.000 |       100.000 |        79.270 |        80.000 | 2.050300e+01 |     0.259 | 2.900000e+01 |  \-0.781 |  \-0.216 |         26.500 |       142.500 |        14 |
| width              | All   | 141833 |      0 |  12542 | 129291 |      0 |      0 |         0 |            0.000 |   1.845753e+06 |         0.000 |        60.000 |        13.014 |         9.000 | 7.707000e+00 |     0.592 | 9.000000e+00 |    0.760 |    0.932 |        \-4.500 |        31.500 |      6746 |

</div>

## Definitions Numerical Features

[Definitions available in
pdf](https://unstats.un.org/wiki/display/AIS/AIS+data+at+the+UN+Global+Platform?preview=/57999715/72778218/UNGP%20-%20AIS%20ADSB%20JupyterHubNotebookQueryGuide.pdf#AISdataattheUNGlobalPlatform-Step-by-stepguidelinesforexecutingthesamplescriptsonUNGP)

**cog:** Course over Ground \[Degrees\]

**draught:** Vessel Draught \[Metres\]

**eta:** Month, Day, Hour, and Minute of Estimated Time of Arrival in
UTC \[MMDDHHmm\]

**flag\_code:** Country of Registration Code

**heading:** Heading \[Degrees\]

**latitude:** WGS 84 Latitude Coordinate \[Decimal Degrees\]

**length:** Length of Bow to Main Tower and Main Tower to Stern
\[Meters\]

**longitude:** WGS 84 Longitude Coordinate \[Decimal Degrees\]

**mmsi:** Maritime Mobile Service Identity (MMSI)

**rot:** Rate of Turn \[Degrees / Min\]

**sog:** Speed over Ground \[Knots\]

**vessel\_type\_code**:\* Vessel Type Code

**width:** Length of Port to Main Tower and Main Tower to Starboard
\[Meters\]

``` r
# Statistics for Numerical features by Vessel Type
data.table(
  ExpNumStat(
    cklist.list,
    by = "GA",
    gp = "vessel_type",
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)[order(-TN),
  head(.SD,
       10),
  by = Vname]
```

    ## Warning in min(x, na.rm = T): no non-missing arguments to min; returning Inf

    ## Warning in max(x, na.rm = T): no non-missing arguments to max; returning -Inf

<div class="kable-table">

| Vname              | Group                       |     TN |   nNeg |  nZero |   nPos | NegInf | PosInf | NA\_Value | Per\_of\_Missing |            sum |           min |           max |          mean |        median |           SD |        CV |          IQR |  Skewness |  Kurtosis |         LB.25% |        UB.75% | nOutliers |
| :----------------- | :-------------------------- | -----: | -----: | -----: | -----: | -----: | -----: | --------: | ---------------: | -------------: | ------------: | ------------: | ------------: | ------------: | -----------: | --------: | -----------: | --------: | --------: | -------------: | ------------: | --------: |
| cog                | vessel\_type:All            | 141833 |      0 |  15387 | 126446 |      0 |      0 |         0 |            0.000 |   2.648614e+07 |         0.000 |       359.900 |       186.742 |       204.400 | 1.166830e+02 |     0.625 | 1.932000e+02 |   \-0.384 |   \-1.287 |      \-186.100 |       586.700 |         0 |
| cog                | vessel\_type:Other          |  62474 |      0 |   7939 |  54535 |      0 |      0 |         0 |            0.000 |   1.405621e+07 |         0.000 |       359.900 |       224.993 |       292.800 | 1.119130e+02 |     0.497 | 1.711000e+02 |   \-1.060 |   \-0.458 |      \-125.850 |       558.550 |         0 |
| cog                | vessel\_type:Cargo          |  41523 |      0 |      3 |  41520 |      0 |      0 |         0 |            0.000 |   8.052524e+06 |         0.000 |       359.900 |       193.929 |       204.400 | 9.300700e+01 |     0.480 | 1.480000e+02 |   \-0.185 |   \-0.642 |       \-95.000 |       497.000 |         0 |
| cog                | vessel\_type:Passenger      |  13836 |      0 |     10 |  13826 |      0 |      0 |         0 |            0.000 |   2.474556e+06 |         0.000 |       359.800 |       178.849 |       174.300 | 1.019660e+02 |     0.570 | 1.750000e+02 |     0.029 |   \-1.128 |      \-166.700 |       533.300 |         0 |
| cog                | vessel\_type:Sailing        |  11092 |      0 |   4907 |   6185 |      0 |      0 |         0 |            0.000 |   1.132198e+06 |         0.000 |       359.900 |       102.073 |        87.200 | 1.153390e+02 |     1.130 | 1.607000e+02 |     0.765 |   \-0.763 |      \-241.050 |       401.750 |         0 |
| cog                | vessel\_type:Tanker         |   8631 |      0 |      0 |   8631 |      0 |      0 |         0 |            0.000 |   4.382534e+05 |         0.700 |       358.200 |        50.777 |         7.900 | 8.474600e+01 |     1.669 | 0.000000e+00 |     1.792 |     1.999 |          7.900 |         7.900 |      2109 |
| cog                | vessel\_type:Fishing        |   2714 |      0 |   2270 |    444 |      0 |      0 |         0 |            0.000 |   8.725020e+04 |         0.000 |       355.200 |        32.148 |         0.000 | 7.779400e+01 |     2.420 | 0.000000e+00 |     2.169 |     3.175 |          0.000 |         0.000 |       444 |
| cog                | vessel\_type:Pleasure Craft |    747 |      0 |    166 |    581 |      0 |      0 |         0 |            0.000 |   1.044318e+05 |         0.000 |       359.900 |       139.802 |       185.200 | 1.021030e+02 |     0.730 | 1.756000e+02 |   \-0.021 |   \-0.889 |      \-253.600 |       448.800 |         0 |
| cog                | vessel\_type:Reserved       |    531 |      0 |      0 |    531 |      0 |      0 |         0 |            0.000 |   1.062986e+05 |         1.100 |       359.400 |       200.186 |       239.200 | 1.088880e+02 |     0.544 | 1.903000e+02 |   \-0.550 |   \-0.970 |      \-209.400 |       551.800 |         0 |
| cog                | vessel\_type:Port Tender    |    140 |      0 |      0 |    140 |      0 |      0 |         0 |            0.000 |   2.332370e+04 |         5.200 |       358.800 |       166.598 |       193.400 | 1.179880e+02 |     0.708 | 2.234000e+02 |     0.079 |   \-1.411 |      \-290.750 |       602.850 |         0 |
| draught            | vessel\_type:All            | 141833 |      0 |   6512 | 135321 |      0 |      0 |         0 |            0.000 |   6.416894e+05 |         0.000 |        12.900 |         4.524 |         3.100 | 2.178000e+00 |     0.481 | 3.900000e+00 |     0.106 |   \-1.093 |        \-2.850 |        12.750 |        16 |
| draught            | vessel\_type:Other          |  62474 |      0 |   1516 |  60958 |      0 |      0 |         0 |            0.000 |   1.909055e+05 |         0.000 |         5.100 |         3.056 |         3.100 | 5.770000e-01 |     0.189 | 1.000000e-01 |   \-2.841 |    19.375 |          2.850 |         3.250 |      3427 |
| draught            | vessel\_type:Cargo          |  41523 |      0 |     13 |  41510 |      0 |      0 |         0 |            0.000 |   2.655557e+05 |         0.000 |         7.600 |         6.395 |         6.900 | 1.352000e+00 |     0.211 | 5.000000e-01 |   \-1.947 |     2.334 |          5.750 |         7.750 |      5324 |
| draught            | vessel\_type:Passenger      |  13836 |      0 |    485 |  13351 |      0 |      0 |         0 |            0.000 |   9.605900e+04 |         0.000 |         8.100 |         6.943 |         7.700 | 1.783000e+00 |     0.257 | 2.200000e+00 |   \-2.267 |     5.530 |          2.600 |        11.400 |       485 |
| draught            | vessel\_type:Sailing        |  11092 |      0 |   1316 |   9776 |      0 |      0 |         0 |            0.000 |   2.737280e+04 |         0.000 |         2.800 |         2.468 |         2.800 | 9.050000e-01 |     0.367 | 0.000000e+00 |   \-2.359 |     3.563 |          2.800 |         2.800 |      1316 |
| draught            | vessel\_type:Tanker         |   8631 |      0 |      8 |   8623 |      0 |      0 |         0 |            0.000 |   5.801750e+04 |         0.000 |         7.000 |         6.722 |         7.000 | 6.540000e-01 |     0.097 | 0.000000e+00 |   \-3.111 |    14.133 |          7.000 |         7.000 |      1858 |
| draught            | vessel\_type:Fishing        |   2714 |      0 |   2286 |    428 |      0 |      0 |         0 |            0.000 |   2.125600e+03 |         0.000 |         5.000 |         0.783 |         0.000 | 1.814000e+00 |     2.316 | 0.000000e+00 |     1.889 |     1.577 |          0.000 |         0.000 |       428 |
| draught            | vessel\_type:Pleasure Craft |    747 |      0 |     94 |    653 |      0 |      0 |         0 |            0.000 |   1.420800e+03 |         0.000 |         2.600 |         1.902 |         2.100 | 7.410000e-01 |     0.390 | 0.000000e+00 |   \-2.004 |     2.575 |          2.100 |         2.100 |       193 |
| draught            | vessel\_type:Reserved       |    531 |      0 |    531 |      0 |      0 |      0 |         0 |            0.000 |   0.000000e+00 |         0.000 |         0.000 |         0.000 |         0.000 | 0.000000e+00 |       NaN | 0.000000e+00 |       NaN |       NaN |          0.000 |         0.000 |         0 |
| draught            | vessel\_type:Port Tender    |    140 |      0 |    137 |      3 |      0 |      0 |         0 |            0.000 |   2.310000e+01 |         0.000 |         7.700 |         0.165 |         0.000 | 1.119000e+00 |     6.782 | 0.000000e+00 |     6.610 |    41.689 |          0.000 |         0.000 |         3 |
| eta                | vessel\_type:All            | 141833 |      0 |      0 | 141833 |      0 |      0 |         0 |            0.000 |   1.011480e+12 |      2460.000 |  12311800.000 |   7131486.726 |   8260830.000 | 3.838625e+06 |     0.538 | 6.950300e+06 |   \-0.339 |   \-1.191 |  \-7254250.000 |  20546950.000 |         0 |
| eta                | vessel\_type:Other          |  62474 |      0 |      0 |  62474 |      0 |      0 |         0 |            0.000 |   4.925451e+11 |      2460.000 |  12250830.000 |   7884001.210 |   9281300.000 | 3.107656e+06 |     0.394 | 4.809900e+06 |   \-0.515 |   \-0.371 |  \-1903250.000 |  17336350.000 |         0 |
| eta                | vessel\_type:Cargo          |  41523 |      0 |      0 |  41523 |      0 |      0 |         0 |            0.000 |   3.334576e+11 |   1040700.000 |  12311500.000 |   8030672.463 |   8260830.000 | 3.224099e+06 |     0.401 | 4.849900e+06 |   \-0.866 |   \-0.316 |  \-1033250.000 |  18366350.000 |         0 |
| eta                | vessel\_type:Passenger      |  13836 |      0 |      0 |  13836 |      0 |      0 |         0 |            0.000 |   8.175384e+10 |      2460.000 |  12181600.000 |   5908776.770 |   3131700.000 | 4.467675e+06 |     0.756 | 8.949985e+06 |     0.379 |   \-1.728 | \-11183262.500 |  24616677.500 |         0 |
| eta                | vessel\_type:Sailing        |  11092 |      0 |      0 |  11092 |      0 |      0 |         0 |            0.000 |   6.204372e+10 |      2460.000 |  12302300.000 |   5593555.758 |   1231930.000 | 5.569506e+06 |     0.996 | 1.107037e+07 |     0.364 |   \-1.846 | \-15373625.000 |  28907855.000 |         0 |
| eta                | vessel\_type:Tanker         |   8631 |      0 |      0 |   8631 |      0 |      0 |         0 |            0.000 |   3.627228e+10 |   2051800.000 |  12311800.000 |   4202558.301 |   3142300.000 | 2.731513e+06 |     0.650 | 0.000000e+00 |     2.299 |     3.681 |    3142300.000 |   3142300.000 |      1850 |
| eta                | vessel\_type:Fishing        |   2714 |      0 |      0 |   2714 |      0 |      0 |         0 |            0.000 |   3.446542e+09 |      2460.000 |   9300500.000 |   1269912.218 |      2460.000 | 2.930340e+06 |     2.308 | 0.000000e+00 |     1.880 |     1.540 |       2460.000 |      2460.000 |       428 |
| eta                | vessel\_type:Pleasure Craft |    747 |      0 |      0 |    747 |      0 |      0 |         0 |            0.000 |   1.660873e+09 |      2460.000 |  10020000.000 |   2223390.515 |   1280800.000 | 2.814566e+06 |     1.266 | 0.000000e+00 |     2.059 |     2.441 |    1280800.000 |   1280800.000 |       193 |
| eta                | vessel\_type:Reserved       |    531 |      0 |      0 |    531 |      0 |      0 |         0 |            0.000 |   1.306260e+06 |      2460.000 |      2460.000 |      2460.000 |      2460.000 | 0.000000e+00 |     0.000 | 0.000000e+00 |       NaN |       NaN |       2460.000 |      2460.000 |         0 |
| eta                | vessel\_type:Port Tender    |    140 |      0 |      0 |    140 |      0 |      0 |         0 |            0.000 |   9.732120e+06 |      2460.000 |   3131700.000 |     69515.143 |      2460.000 | 4.547664e+05 |     6.542 | 0.000000e+00 |     6.610 |    41.689 |       2460.000 |      2460.000 |         3 |
| flag\_code         | vessel\_type:All            | 141833 |      0 |      0 | 141833 |      0 |      0 |         0 |            0.000 |   6.025123e+07 |       219.000 |       982.000 |       424.804 |       518.000 | 1.203680e+02 |     0.283 | 2.140000e+02 |   \-0.006 |   \-0.151 |       \-17.000 |       839.000 |       499 |
| flag\_code         | vessel\_type:Other          |  62474 |      0 |      0 |  62474 |      0 |      0 |         0 |            0.000 |   3.157211e+07 |       222.000 |       518.000 |       505.364 |       518.000 | 5.526900e+01 |     0.109 | 0.000000e+00 |   \-4.480 |    19.099 |        518.000 |       518.000 |      3423 |
| flag\_code         | vessel\_type:Cargo          |  41523 |      0 |      0 |  41523 |      0 |      0 |         0 |            0.000 |   1.376157e+07 |       256.000 |       710.000 |       331.420 |       304.000 | 7.156000e+01 |     0.216 | 0.000000e+00 |     2.228 |     2.977 |        304.000 |       304.000 |      5320 |
| flag\_code         | vessel\_type:Passenger      |  13836 |      0 |      0 |  13836 |      0 |      0 |         0 |            0.000 |   4.121183e+06 |       244.000 |       982.000 |       297.859 |       244.000 | 1.337130e+02 |     0.449 | 6.700000e+01 |     4.578 |    20.604 |        143.500 |       411.500 |       477 |
| flag\_code         | vessel\_type:Sailing        |  11092 |      0 |      0 |  11092 |      0 |      0 |         0 |            0.000 |   6.006614e+06 |       219.000 |       710.000 |       541.527 |       570.000 | 8.573300e+01 |     0.158 | 0.000000e+00 |   \-2.865 |     6.480 |        570.000 |       570.000 |      1305 |
| flag\_code         | vessel\_type:Tanker         |   8631 |      0 |      0 |   8631 |      0 |      0 |         0 |            0.000 |   3.578806e+06 |       219.000 |       440.000 |       414.646 |       440.000 | 6.688600e+01 |     0.161 | 0.000000e+00 |   \-2.475 |     4.364 |        440.000 |       440.000 |      1266 |
| flag\_code         | vessel\_type:Fishing        |   2714 |      0 |      0 |   2714 |      0 |      0 |         0 |            0.000 |   6.033520e+05 |       222.000 |       224.000 |       222.311 |       222.000 | 7.250000e-01 |     0.003 | 0.000000e+00 |     1.901 |     1.615 |        222.000 |       222.000 |       422 |
| flag\_code         | vessel\_type:Pleasure Craft |    747 |      0 |      0 |    747 |      0 |      0 |         0 |            0.000 |   2.476960e+05 |       222.000 |       982.000 |       331.588 |       339.000 | 8.883400e+01 |     0.268 | 0.000000e+00 |     5.165 |    36.360 |        339.000 |       339.000 |       188 |
| flag\_code         | vessel\_type:Reserved       |    531 |      0 |      0 |    531 |      0 |      0 |         0 |            0.000 |   2.750580e+05 |       518.000 |       518.000 |       518.000 |       518.000 | 0.000000e+00 |     0.000 | 0.000000e+00 |       NaN |       NaN |        518.000 |       518.000 |         0 |
| flag\_code         | vessel\_type:Port Tender    |    140 |      0 |      0 |    140 |      0 |      0 |         0 |            0.000 |   4.375400e+04 |       244.000 |       982.000 |       312.529 |       244.000 | 2.149600e+02 |     0.688 | 0.000000e+00 |     2.806 |     5.872 |        244.000 |       244.000 |        13 |
| heading            | vessel\_type:All            | 141833 |      0 |  21461 | 120372 |      0 |      0 |         0 |            0.000 |   1.375850e+07 |         0.000 |       359.000 |        97.005 |       112.000 | 9.274400e+01 |     0.956 | 1.130000e+02 |     0.977 |     0.364 |      \-156.500 |       295.500 |      8029 |
| heading            | vessel\_type:Other          |  62474 |      0 |  15024 |  47450 |      0 |      0 |         0 |            0.000 |   5.933415e+06 |         0.000 |       359.000 |        94.974 |       124.000 | 6.108300e+01 |     0.643 | 1.120000e+02 |   \-0.315 |     0.334 |      \-154.000 |       294.000 |       330 |
| heading            | vessel\_type:Cargo          |  41523 |      0 |   1213 |  40310 |      0 |      0 |         0 |            0.000 |   2.263568e+06 |         0.000 |       359.000 |        54.514 |        14.000 | 6.344800e+01 |     1.164 | 9.300000e+01 |     1.718 |     3.549 |      \-126.500 |       245.500 |       741 |
| heading            | vessel\_type:Passenger      |  13836 |      0 |    484 |  13352 |      0 |      0 |         0 |            0.000 |   2.088763e+06 |         0.000 |       359.000 |       150.966 |       116.000 | 9.804500e+01 |     0.649 | 1.590000e+02 |     0.486 |   \-0.992 |      \-157.500 |       478.500 |         0 |
| heading            | vessel\_type:Sailing        |  11092 |      0 |   1273 |   9819 |      0 |      0 |         0 |            0.000 |   3.050408e+06 |         0.000 |       349.000 |       275.010 |       321.000 | 1.004010e+02 |     0.365 | 2.700000e+01 |   \-2.316 |     3.530 |        253.500 |       361.500 |      1304 |
| heading            | vessel\_type:Tanker         |   8631 |      0 |      3 |   8628 |      0 |      0 |         0 |            0.000 |   3.495560e+05 |         0.000 |       359.000 |        40.500 |        14.000 | 6.730900e+01 |     1.662 | 0.000000e+00 |     2.689 |     6.788 |         14.000 |        14.000 |      1580 |
| heading            | vessel\_type:Fishing        |   2714 |      0 |   2296 |    418 |      0 |      0 |         0 |            0.000 |   2.523300e+04 |         0.000 |       359.000 |         9.297 |         0.000 | 3.883800e+01 |     4.177 | 0.000000e+00 |     6.000 |    40.220 |          0.000 |         0.000 |       418 |
| heading            | vessel\_type:Pleasure Craft |    747 |      0 |    742 |      5 |      0 |      0 |         0 |            0.000 |   1.145000e+03 |         0.000 |       334.000 |         1.533 |         0.000 | 2.039800e+01 |    13.308 | 0.000000e+00 |    14.085 |   202.404 |          0.000 |         0.000 |         5 |
| heading            | vessel\_type:Reserved       |    531 |      0 |    172 |    359 |      0 |      0 |         0 |            0.000 |   4.258900e+04 |         0.000 |       352.000 |        80.205 |        14.000 | 1.118330e+02 |     1.394 | 2.365000e+02 |     0.954 |   \-0.890 |      \-354.750 |       591.250 |         0 |
| heading            | vessel\_type:Port Tender    |    140 |      0 |    138 |      2 |      0 |      0 |         0 |            0.000 |   2.140000e+02 |         0.000 |       107.000 |         1.529 |         0.000 | 1.274300e+01 |     8.336 | 0.000000e+00 |     8.186 |    65.014 |          0.000 |         0.000 |         2 |
| latitude           | vessel\_type:All            | 141833 | 141833 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-3.007341e+06 |      \-21.206 |      \-21.190 |      \-21.203 |      \-21.205 | 3.000000e-03 |     0.000 | 1.000000e-03 |     2.294 |     4.072 |       \-21.206 |      \-21.203 |     21101 |
| latitude           | vessel\_type:Other          |  62474 |  62474 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-1.324730e+06 |      \-21.205 |      \-21.190 |      \-21.204 |      \-21.205 | 2.000000e-03 |     0.000 | 0.000000e+00 |     4.491 |    20.128 |       \-21.205 |      \-21.205 |      7511 |
| latitude           | vessel\_type:Cargo          |  41523 |  41523 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-8.804494e+05 |      \-21.205 |      \-21.190 |      \-21.204 |      \-21.205 | 2.000000e-03 |     0.000 | 0.000000e+00 |     4.345 |    19.089 |       \-21.205 |      \-21.204 |     14419 |
| latitude           | vessel\_type:Passenger      |  13836 |  13836 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-2.932698e+05 |      \-21.204 |      \-21.190 |      \-21.196 |      \-21.197 | 2.000000e-03 |     0.000 | 3.000000e-03 |     0.254 |     0.750 |       \-21.202 |      \-21.190 |       162 |
| latitude           | vessel\_type:Sailing        |  11092 |  11092 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-2.351988e+05 |      \-21.206 |      \-21.190 |      \-21.204 |      \-21.205 | 1.000000e-03 |     0.000 | 1.000000e-03 |     8.034 |    72.837 |       \-21.206 |      \-21.203 |       352 |
| latitude           | vessel\_type:Tanker         |   8631 |   8631 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-1.830051e+05 |      \-21.204 |      \-21.190 |      \-21.203 |      \-21.204 | 3.000000e-03 |     0.000 | 0.000000e+00 |     3.243 |     9.206 |       \-21.204 |      \-21.204 |      1658 |
| latitude           | vessel\_type:Fishing        |   2714 |   2714 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-5.754836e+04 |      \-21.205 |      \-21.190 |      \-21.204 |      \-21.205 | 2.000000e-03 |     0.000 | 0.000000e+00 |     6.689 |    45.802 |       \-21.205 |      \-21.205 |       444 |
| latitude           | vessel\_type:Pleasure Craft |    747 |    747 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-1.583839e+04 |      \-21.205 |      \-21.190 |      \-21.203 |      \-21.205 | 5.000000e-03 |     0.000 | 0.000000e+00 |     1.962 |     2.078 |       \-21.205 |      \-21.205 |       289 |
| latitude           | vessel\_type:Reserved       |    531 |    531 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-1.125877e+04 |      \-21.205 |      \-21.190 |      \-21.203 |      \-21.204 | 3.000000e-03 |     0.000 | 0.000000e+00 |     3.205 |     9.381 |       \-21.204 |      \-21.203 |        87 |
| latitude           | vessel\_type:Port Tender    |    140 |    140 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-2.967922e+03 |      \-21.204 |      \-21.193 |      \-21.199 |      \-21.199 | 3.000000e-03 |     0.000 | 3.000000e-03 |     0.148 |   \-0.204 |       \-21.206 |      \-21.193 |         0 |
| length             | vessel\_type:All            | 141833 |      0 |  12539 | 129294 |      0 |      0 |         0 |            0.000 |   1.089040e+07 |         0.000 |       490.000 |        76.783 |        38.000 | 6.055300e+01 |     0.789 | 8.000000e+01 |     1.117 |     1.087 |       \-82.000 |       238.000 |       825 |
| length             | vessel\_type:Other          |  62474 |      0 |      0 |  62474 |      0 |      0 |         0 |            0.000 |   2.542040e+06 |        22.000 |       180.000 |        40.690 |        38.000 | 1.176600e+01 |     0.289 | 0.000000e+00 |     4.069 |    15.073 |         38.000 |        38.000 |      3425 |
| length             | vessel\_type:Cargo          |  41523 |      0 |      1 |  41522 |      0 |      0 |         0 |            0.000 |   4.496009e+06 |         0.000 |       265.000 |       108.278 |       118.000 | 2.695700e+01 |     0.249 | 2.000000e+00 |   \-2.212 |     2.973 |        115.000 |       123.000 |      5318 |
| length             | vessel\_type:Passenger      |  13836 |      0 |    476 |  13360 |      0 |      0 |         0 |            0.000 |   2.838601e+06 |         0.000 |       294.000 |       205.161 |       220.000 | 5.327200e+01 |     0.260 | 5.700000e+01 |   \-1.935 |     5.240 |         95.500 |       323.500 |       477 |
| length             | vessel\_type:Sailing        |  11092 |      0 |   9818 |   1274 |      0 |      0 |         0 |            0.000 |   2.108900e+04 |         0.000 |       240.000 |         1.901 |         0.000 | 6.468000e+00 |     3.402 | 0.000000e+00 |    13.908 |   450.308 |          0.000 |         0.000 |      1274 |
| length             | vessel\_type:Tanker         |   8631 |      0 |      0 |   8631 |      0 |      0 |         0 |            0.000 |   9.029600e+05 |        86.000 |       110.000 |       104.618 |       105.000 | 4.374000e+00 |     0.042 | 0.000000e+00 |   \-3.396 |    12.483 |        105.000 |       105.000 |      1266 |
| length             | vessel\_type:Fishing        |   2714 |      0 |   2174 |    540 |      0 |      0 |         0 |            0.000 |   3.466200e+04 |         0.000 |       325.000 |        12.772 |         0.000 | 3.958600e+01 |     3.100 | 0.000000e+00 |     6.241 |    45.627 |          0.000 |         0.000 |       540 |
| length             | vessel\_type:Pleasure Craft |    747 |      0 |     17 |    730 |      0 |      0 |         0 |            0.000 |   2.552100e+04 |         0.000 |       269.000 |        34.165 |        30.000 | 1.896300e+01 |     0.555 | 0.000000e+00 |     6.337 |    67.390 |         30.000 |        30.000 |       182 |
| length             | vessel\_type:Reserved       |    531 |      0 |      0 |    531 |      0 |      0 |         0 |            0.000 |   1.646100e+04 |        31.000 |        31.000 |        31.000 |        31.000 | 0.000000e+00 |     0.000 | 0.000000e+00 |       NaN |       NaN |         31.000 |        31.000 |         0 |
| length             | vessel\_type:Port Tender    |    140 |      0 |     16 |    124 |      0 |      0 |         0 |            0.000 |   1.364000e+03 |         0.000 |        11.000 |         9.743 |        11.000 | 3.512000e+00 |     0.361 | 0.000000e+00 |   \-2.425 |     3.879 |         11.000 |        11.000 |        16 |
| longitude          | vessel\_type:All            | 141833 | 141833 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-2.266268e+07 |     \-159.797 |     \-159.775 |     \-159.784 |     \-159.785 | 1.000000e-03 |     0.000 | 0.000000e+00 |     3.092 |    21.606 |      \-159.785 |     \-159.784 |     22849 |
| longitude          | vessel\_type:Other          |  62474 |  62474 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-9.982382e+06 |     \-159.797 |     \-159.775 |     \-159.785 |     \-159.785 | 1.000000e-03 |     0.000 | 0.000000e+00 |     4.263 |    49.332 |      \-159.785 |     \-159.785 |      8596 |
| longitude          | vessel\_type:Cargo          |  41523 |  41523 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-6.634717e+06 |     \-159.797 |     \-159.775 |     \-159.784 |     \-159.784 | 1.000000e-03 |     0.000 | 0.000000e+00 |     2.427 |    44.704 |      \-159.785 |     \-159.784 |     14047 |
| longitude          | vessel\_type:Passenger      |  13836 |  13836 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-2.210755e+06 |     \-159.795 |     \-159.775 |     \-159.783 |     \-159.783 | 2.000000e-03 |     0.000 | 2.000000e-03 |     0.880 |     2.262 |      \-159.787 |     \-159.778 |      1026 |
| longitude          | vessel\_type:Sailing        |  11092 |  11092 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-1.772331e+06 |     \-159.796 |     \-159.775 |     \-159.785 |     \-159.785 | 1.000000e-03 |     0.000 | 1.000000e-03 |     5.553 |    50.810 |      \-159.786 |     \-159.783 |       215 |
| longitude          | vessel\_type:Tanker         |   8631 |   8631 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-1.379096e+06 |     \-159.787 |     \-159.775 |     \-159.784 |     \-159.784 | 1.000000e-03 |     0.000 | 0.000000e+00 |     5.338 |    36.621 |      \-159.784 |     \-159.784 |      1094 |
| longitude          | vessel\_type:Fishing        |   2714 |   2714 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-4.336547e+05 |     \-159.785 |     \-159.781 |     \-159.784 |     \-159.784 | 0.000000e+00 |     0.000 | 0.000000e+00 |     6.881 |    55.898 |      \-159.784 |     \-159.784 |       429 |
| longitude          | vessel\_type:Pleasure Craft |    747 |    747 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-1.193588e+05 |     \-159.795 |     \-159.776 |     \-159.784 |     \-159.784 | 1.000000e-03 |     0.000 | 0.000000e+00 |     2.504 |    22.785 |      \-159.785 |     \-159.784 |       277 |
| longitude          | vessel\_type:Reserved       |    531 |    531 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-8.484568e+04 |     \-159.795 |     \-159.777 |     \-159.785 |     \-159.784 | 1.000000e-03 |     0.000 | 1.000000e-03 |     1.306 |    16.295 |      \-159.788 |     \-159.782 |        23 |
| longitude          | vessel\_type:Port Tender    |    140 |    140 |      0 |      0 |      0 |      0 |         0 |            0.000 | \-2.236969e+04 |     \-159.788 |     \-159.778 |     \-159.784 |     \-159.783 | 2.000000e-03 |     0.000 | 2.000000e-03 |     0.136 |     1.462 |      \-159.788 |     \-159.779 |         7 |
| mmsi               | vessel\_type:All            | 141833 |      0 |      0 | 141833 |      0 |      0 |         0 |            0.000 |   6.030233e+13 | 219018897.000 | 982449587.000 | 425164326.211 | 518100443.000 | 1.201826e+08 |     0.283 | 2.132134e+08 |   \-0.005 |   \-0.137 | \-14933164.500 | 837920607.500 |       499 |
| mmsi               | vessel\_type:Other          |  62474 |      0 |      0 |  62474 |      0 |      0 |         0 |            0.000 |   3.157899e+13 | 222222222.000 | 518100443.000 | 505474148.290 | 518100443.000 | 5.523547e+07 |     0.109 | 0.000000e+00 |   \-4.481 |    19.113 |  518100443.000 | 518100443.000 |      3423 |
| mmsi               | vessel\_type:Cargo          |  41523 |      0 |      0 |  41523 |      0 |      0 |         0 |            0.000 |   1.378858e+13 | 256204000.000 | 710009510.000 | 332070918.027 | 304887000.000 | 7.134929e+07 |     0.215 | 3.980000e+05 |     2.228 |     2.977 |  303989000.000 | 305581000.000 |      5320 |
| mmsi               | vessel\_type:Passenger      |  13836 |      0 |      0 |  13836 |      0 |      0 |         0 |            0.000 |   4.127801e+12 | 244370000.000 | 982443703.000 | 298337768.716 | 244958000.000 | 1.336729e+08 |     0.448 | 6.663089e+07 |     4.582 |    20.627 |  144423663.500 | 410947227.500 |       477 |
| mmsi               | vessel\_type:Sailing        |  11092 |      0 |      0 |  11092 |      0 |      0 |         0 |            0.000 |   6.010938e+12 | 219025000.000 | 710000595.000 | 541916499.708 | 570411000.000 | 8.579268e+07 |     0.158 | 0.000000e+00 |   \-2.865 |     6.484 |  570411000.000 | 570411000.000 |      1305 |
| mmsi               | vessel\_type:Tanker         |   8631 |      0 |      0 |   8631 |      0 |      0 |         0 |            0.000 |   3.583915e+12 | 219018897.000 | 440691000.000 | 415237489.156 | 440691000.000 | 6.710372e+07 |     0.162 | 0.000000e+00 |   \-2.473 |     4.355 |  440691000.000 | 440691000.000 |      1266 |
| mmsi               | vessel\_type:Fishing        |   2714 |      0 |      0 |   2714 |      0 |      0 |         0 |            0.000 |   6.039263e+11 | 222222222.000 | 224154000.000 | 222522594.261 | 222222222.000 | 7.001497e+05 |     0.003 | 0.000000e+00 |     1.901 |     1.615 |  222222222.000 | 222222222.000 |       422 |
| mmsi               | vessel\_type:Pleasure Craft |    747 |      0 |      0 |    747 |      0 |      0 |         0 |            0.000 |   2.481021e+11 | 222222222.000 | 982449587.000 | 332131298.592 | 339652000.000 | 8.889213e+07 |     0.268 | 0.000000e+00 |     5.150 |    36.238 |  339652000.000 | 339652000.000 |       188 |
| mmsi               | vessel\_type:Reserved       |    531 |      0 |      0 |    531 |      0 |      0 |         0 |            0.000 |   2.750654e+11 | 518014000.000 | 518014000.000 | 518014000.000 | 518014000.000 | 0.000000e+00 |     0.000 | 0.000000e+00 |       NaN |       NaN |  518014000.000 | 518014000.000 |         0 |
| mmsi               | vessel\_type:Port Tender    |    140 |      0 |      0 |    140 |      0 |      0 |         0 |            0.000 |   4.387983e+10 | 244958000.000 | 982320121.000 | 313427347.779 | 244958009.000 | 2.147746e+08 |     0.685 | 0.000000e+00 |     2.806 |     5.872 |  244958009.000 | 244958009.000 |        16 |
| rot                | vessel\_type:All            | 141833 |    847 | 140304 |    682 |      0 |      0 |         0 |            0.000 | \-5.833590e+02 |      \-57.854 |        45.712 |       \-0.004 |         0.000 | 1.667000e+00 | \-405.360 | 0.000000e+00 |   \-0.210 |   319.920 |          0.000 |         0.000 |      1529 |
| rot                | vessel\_type:Other          |  62474 |      0 |  62474 |      0 |      0 |      0 |         0 |            0.000 |   0.000000e+00 |         0.000 |         0.000 |         0.000 |         0.000 | 0.000000e+00 |       NaN | 0.000000e+00 |       NaN |       NaN |          0.000 |         0.000 |         0 |
| rot                | vessel\_type:Cargo          |  41523 |      2 |  41521 |      0 |      0 |      0 |         0 |            0.000 | \-1.040100e+01 |       \-7.544 |         0.000 |         0.000 |         0.000 | 4.000000e-02 | \-158.042 | 0.000000e+00 | \-175.706 | 32408.544 |          0.000 |         0.000 |         2 |
| rot                | vessel\_type:Passenger      |  13836 |    844 |  12310 |    682 |      0 |      0 |         0 |            0.000 | \-5.707710e+02 |      \-57.854 |        45.712 |       \-0.041 |         0.000 | 5.338000e+00 | \-129.389 | 0.000000e+00 |   \-0.045 |    28.514 |          0.000 |         0.000 |      1526 |
| rot                | vessel\_type:Sailing        |  11092 |      1 |  11091 |      0 |      0 |      0 |         0 |            0.000 | \-2.187000e+00 |       \-2.187 |         0.000 |         0.000 |         0.000 | 2.100000e-02 | \-105.319 | 0.000000e+00 | \-105.304 | 11087.000 |          0.000 |         0.000 |         1 |
| rot                | vessel\_type:Tanker         |   8631 |      0 |   8631 |      0 |      0 |      0 |         0 |            0.000 |   0.000000e+00 |         0.000 |         0.000 |         0.000 |         0.000 | 0.000000e+00 |       NaN | 0.000000e+00 |       NaN |       NaN |          0.000 |         0.000 |         0 |
| rot                | vessel\_type:Fishing        |   2714 |      0 |   2714 |      0 |      0 |      0 |         0 |            0.000 |   0.000000e+00 |         0.000 |         0.000 |         0.000 |         0.000 | 0.000000e+00 |       NaN | 0.000000e+00 |       NaN |       NaN |          0.000 |         0.000 |         0 |
| rot                | vessel\_type:Pleasure Craft |    747 |      0 |    747 |      0 |      0 |      0 |         0 |            0.000 |   0.000000e+00 |         0.000 |         0.000 |         0.000 |         0.000 | 0.000000e+00 |       NaN | 0.000000e+00 |       NaN |       NaN |          0.000 |         0.000 |         0 |
| rot                | vessel\_type:Reserved       |    531 |      0 |    531 |      0 |      0 |      0 |         0 |            0.000 |   0.000000e+00 |         0.000 |         0.000 |         0.000 |         0.000 | 0.000000e+00 |       NaN | 0.000000e+00 |       NaN |       NaN |          0.000 |         0.000 |         0 |
| rot                | vessel\_type:Port Tender    |    140 |      0 |    140 |      0 |      0 |      0 |         0 |            0.000 |   0.000000e+00 |         0.000 |         0.000 |         0.000 |         0.000 | 0.000000e+00 |       NaN | 0.000000e+00 |       NaN |       NaN |          0.000 |         0.000 |         0 |
| sog                | vessel\_type:All            | 141833 |      0 |  58099 |  83734 |      0 |      0 |         0 |            0.000 |   7.221630e+04 |         0.000 |        22.500 |         0.509 |         0.100 | 1.697000e+00 |     3.333 | 2.000000e-01 |     5.729 |    41.116 |        \-0.300 |         0.500 |     15627 |
| sog                | vessel\_type:Other          |  62474 |      0 |  13216 |  49258 |      0 |      0 |         0 |            0.000 |   2.298560e+04 |         0.000 |        13.600 |         0.368 |         0.100 | 1.140000e+00 |     3.098 | 1.000000e-01 |     4.999 |    28.301 |        \-0.050 |         0.350 |      4934 |
| sog                | vessel\_type:Cargo          |  41523 |      0 |  25638 |  15885 |      0 |      0 |         0 |            0.000 |   2.069280e+04 |         0.000 |        11.500 |         0.498 |         0.000 | 1.572000e+00 |     3.155 | 1.000000e-01 |     4.814 |    25.758 |        \-0.150 |         0.250 |      7097 |
| sog                | vessel\_type:Passenger      |  13836 |      0 |   2156 |  11680 |      0 |      0 |         0 |            0.000 |   1.896170e+04 |         0.000 |        22.500 |         1.370 |         0.300 | 3.204000e+00 |     2.338 | 6.000000e-01 |     3.730 |    15.505 |        \-0.800 |         1.600 |      2099 |
| sog                | vessel\_type:Sailing        |  11092 |      0 |   6646 |   4446 |      0 |      0 |         0 |            0.000 |   1.705800e+03 |         0.000 |        16.700 |         0.154 |         0.000 | 6.600000e-01 |     4.289 | 1.000000e-01 |     9.500 |   121.167 |        \-0.150 |         0.250 |      1119 |
| sog                | vessel\_type:Tanker         |   8631 |      0 |   7391 |   1240 |      0 |      0 |         0 |            0.000 |   5.855500e+03 |         0.000 |        13.000 |         0.678 |         0.000 | 2.498000e+00 |     3.682 | 0.000000e+00 |     4.284 |    17.728 |          0.000 |         0.000 |      1240 |
| sog                | vessel\_type:Fishing        |   2714 |      0 |   2520 |    194 |      0 |      0 |         0 |            0.000 |   5.403000e+02 |         0.000 |         9.200 |         0.199 |         0.000 | 9.780000e-01 |     4.911 | 0.000000e+00 |     5.579 |    31.684 |          0.000 |         0.000 |       194 |
| sog                | vessel\_type:Pleasure Craft |    747 |      0 |    338 |    409 |      0 |      0 |         0 |            0.000 |   4.668000e+02 |         0.000 |        12.000 |         0.625 |         0.100 | 1.701000e+00 |     2.722 | 2.000000e-01 |     3.585 |    13.556 |        \-0.300 |         0.500 |       106 |
| sog                | vessel\_type:Reserved       |    531 |      0 |     77 |    454 |      0 |      0 |         0 |            0.000 |   5.271000e+02 |         0.000 |        11.900 |         0.993 |         0.100 | 2.377000e+00 |     2.395 | 2.000000e-01 |     2.790 |     6.529 |        \-0.200 |         0.600 |        78 |
| sog                | vessel\_type:Port Tender    |    140 |      0 |     13 |    127 |      0 |      0 |         0 |            0.000 |   3.143000e+02 |         0.000 |         8.500 |         2.245 |         0.800 | 2.585000e+00 |     1.152 | 3.925000e+00 |     0.999 |   \-0.516 |        \-5.588 |        10.113 |         0 |
| vessel\_type\_code | vessel\_type:All            | 141833 |      0 |     14 | 141806 |      0 |      0 |        13 |            0.009 |   1.124205e+07 |         0.000 |       100.000 |        79.270 |        80.000 | 2.050300e+01 |     0.259 | 2.900000e+01 |   \-0.781 |   \-0.216 |         26.500 |       142.500 |        14 |
| vessel\_type\_code | vessel\_type:Other          |  62474 |      0 |      0 |  62474 |      0 |      0 |         0 |            0.000 |   6.154119e+06 |        90.000 |        99.000 |        98.507 |        99.000 | 2.048000e+00 |     0.021 | 0.000000e+00 |   \-3.913 |    13.309 |         99.000 |        99.000 |      3423 |
| vessel\_type\_code | vessel\_type:Cargo          |  41523 |      0 |      0 |  41523 |      0 |      0 |         0 |            0.000 |   2.959684e+06 |        70.000 |        79.000 |        71.278 |        70.000 | 2.976000e+00 |     0.042 | 1.000000e+00 |     2.176 |     2.827 |         68.500 |        72.500 |      5311 |
| vessel\_type\_code | vessel\_type:Passenger      |  13836 |      0 |      0 |  13836 |      0 |      0 |         0 |            0.000 |   8.902350e+05 |        60.000 |        69.000 |        64.342 |        60.000 | 4.497000e+00 |     0.070 | 9.000000e+00 |     0.070 |   \-1.995 |         46.500 |        82.500 |         0 |
| vessel\_type\_code | vessel\_type:Sailing        |  11092 |      0 |      0 |  11092 |      0 |      0 |         0 |            0.000 |   3.993120e+05 |        36.000 |        36.000 |        36.000 |        36.000 | 0.000000e+00 |     0.000 | 0.000000e+00 |       NaN |       NaN |         36.000 |        36.000 |         0 |
| vessel\_type\_code | vessel\_type:Tanker         |   8631 |      0 |      0 |   8631 |      0 |      0 |         0 |            0.000 |   6.922100e+05 |        80.000 |        82.000 |        80.200 |        80.000 | 6.010000e-01 |     0.007 | 0.000000e+00 |     2.663 |     5.089 |         80.000 |        80.000 |       865 |
| vessel\_type\_code | vessel\_type:Fishing        |   2714 |      0 |      0 |   2714 |      0 |      0 |         0 |            0.000 |   8.142000e+04 |        30.000 |        30.000 |        30.000 |        30.000 | 0.000000e+00 |     0.000 | 0.000000e+00 |       NaN |       NaN |         30.000 |        30.000 |         0 |
| vessel\_type\_code | vessel\_type:Pleasure Craft |    747 |      0 |      0 |    747 |      0 |      0 |         0 |            0.000 |   2.763900e+04 |        37.000 |        37.000 |        37.000 |        37.000 | 0.000000e+00 |     0.000 | 0.000000e+00 |       NaN |       NaN |         37.000 |        37.000 |         0 |
| vessel\_type\_code | vessel\_type:Reserved       |    531 |      0 |      0 |    531 |      0 |      0 |         0 |            0.000 |   2.017800e+04 |        38.000 |        38.000 |        38.000 |        38.000 | 0.000000e+00 |     0.000 | 0.000000e+00 |       NaN |       NaN |         38.000 |        38.000 |         0 |
| vessel\_type\_code | vessel\_type:Port Tender    |    140 |      0 |      0 |    140 |      0 |      0 |         0 |            0.000 |   7.420000e+03 |        53.000 |        53.000 |        53.000 |        53.000 | 0.000000e+00 |     0.000 | 0.000000e+00 |       NaN |       NaN |         53.000 |        53.000 |         0 |
| width              | vessel\_type:All            | 141833 |      0 |  12542 | 129291 |      0 |      0 |         0 |            0.000 |   1.845753e+06 |         0.000 |        60.000 |        13.014 |         9.000 | 7.707000e+00 |     0.592 | 9.000000e+00 |     0.760 |     0.932 |        \-4.500 |        31.500 |      6746 |
| width              | vessel\_type:Other          |  62474 |      0 |      1 |  62473 |      0 |      0 |         0 |            0.000 |   5.841720e+05 |         0.000 |        28.000 |         9.351 |         9.000 | 1.757000e+00 |     0.188 | 0.000000e+00 |     6.279 |    51.851 |          9.000 |         9.000 |      3424 |
| width              | vessel\_type:Cargo          |  41523 |      0 |      3 |  41520 |      0 |      0 |         0 |            0.000 |   6.996120e+05 |         0.000 |        45.000 |        16.849 |        18.000 | 3.014000e+00 |     0.179 | 0.000000e+00 |   \-2.199 |     3.177 |         18.000 |        18.000 |      5318 |
| width              | vessel\_type:Passenger      |  13836 |      0 |    476 |  13360 |      0 |      0 |         0 |            0.000 |   3.972080e+05 |         0.000 |        34.000 |        28.708 |        30.000 | 6.715000e+00 |     0.234 | 3.000000e+00 |   \-2.759 |     8.773 |         24.500 |        36.500 |      3034 |
| width              | vessel\_type:Sailing        |  11092 |      0 |   9818 |   1274 |      0 |      0 |         0 |            0.000 |   6.493000e+03 |         0.000 |        46.000 |         0.585 |         0.000 | 1.775000e+00 |     3.033 | 0.000000e+00 |     4.628 |    54.209 |          0.000 |         0.000 |      1274 |
| width              | vessel\_type:Tanker         |   8631 |      0 |      0 |   8631 |      0 |      0 |         0 |            0.000 |   1.398260e+05 |        16.000 |        18.000 |        16.200 |        16.000 | 6.010000e-01 |     0.037 | 0.000000e+00 |     2.663 |     5.089 |         16.000 |        16.000 |       865 |
| width              | vessel\_type:Fishing        |   2714 |      0 |   2174 |    540 |      0 |      0 |         0 |            0.000 |   6.472000e+03 |         0.000 |        60.000 |         2.385 |         0.000 | 7.336000e+00 |     3.076 | 0.000000e+00 |     6.171 |    44.827 |          0.000 |         0.000 |       540 |
| width              | vessel\_type:Pleasure Craft |    747 |      0 |     17 |    730 |      0 |      0 |         0 |            0.000 |   5.727000e+03 |         0.000 |        34.000 |         7.667 |         7.000 | 2.470000e+00 |     0.322 | 0.000000e+00 |     2.828 |    26.870 |          7.000 |         7.000 |       182 |
| width              | vessel\_type:Reserved       |    531 |      0 |      0 |    531 |      0 |      0 |         0 |            0.000 |   4.248000e+03 |         8.000 |         8.000 |         8.000 |         8.000 | 0.000000e+00 |     0.000 | 0.000000e+00 |       NaN |       NaN |          8.000 |         8.000 |         0 |
| width              | vessel\_type:Port Tender    |    140 |      0 |     16 |    124 |      0 |      0 |         0 |            0.000 |   4.960000e+02 |         0.000 |         4.000 |         3.543 |         4.000 | 1.277000e+00 |     0.361 | 0.000000e+00 |   \-2.425 |     3.879 |          4.000 |         4.000 |        16 |

</div>

``` r
# Statistics for Categorigal features
data.table(
  ExpCTable(
    cklist.list,
    Target = NULL,
    margin = 1,
    clim = 50,
    nlim = 10,
    round = 2,
    bin = 3,
    per = TRUE
  )
)[order(-Frequency),
  head(.SD,
       10),
  by = Variable]
```

<div class="kable-table">

| Variable            | Valid                                                   | Frequency | Percent | CumPercent |
| :------------------ | :------------------------------------------------------ | --------: | ------: | ---------: |
| vessel\_class       | TOTAL                                                   |    141833 |      NA |         NA |
| vessel\_class       | A                                                       |    139367 |   98.26 |      98.26 |
| vessel\_class       | B                                                       |      2466 |    1.74 |     100.00 |
| flag\_country       | TOTAL                                                   |    141833 |      NA |         NA |
| flag\_country       | Cook Islands                                            |     65058 |   45.87 |      78.42 |
| flag\_country       | Antigua and Barbuda                                     |     36203 |   25.53 |      28.99 |
| flag\_country       | Tonga                                                   |      9791 |    6.90 |      99.83 |
| flag\_country       | South Korea                                             |      7365 |    5.19 |      92.62 |
| flag\_country       | Netherlands                                             |      7123 |    5.02 |      85.19 |
| flag\_country       |                                                         |      4914 |    3.46 |       3.46 |
| flag\_country       | Bahamas                                                 |      4254 |    3.00 |      32.51 |
| flag\_country       | Panama                                                  |      2450 |    1.73 |      87.41 |
| flag\_country       | Denmark                                                 |       866 |    0.61 |      79.03 |
| vessel\_type        | TOTAL                                                   |    141833 |      NA |         NA |
| vessel\_type        | Other                                                   |     62474 |   44.05 |      75.25 |
| vessel\_type        | Cargo                                                   |     41523 |   29.28 |      29.28 |
| vessel\_type        | Passenger                                               |     13836 |    9.76 |      85.01 |
| vessel\_type        | Sailing                                                 |     11092 |    7.82 |      93.83 |
| vessel\_type        | Tanker                                                  |      8631 |    6.09 |      99.92 |
| vessel\_type        | Fishing                                                 |      2714 |    1.91 |      31.19 |
| vessel\_type        | Pleasure Craft                                          |       747 |    0.53 |      85.54 |
| vessel\_type        | Reserved                                                |       531 |    0.37 |      86.01 |
| vessel\_type        | Port Tender                                             |       140 |    0.10 |      85.64 |
| vessel\_type\_cargo | TOTAL                                                   |    141833 |      NA |         NA |
| vessel\_type\_cargo | No Additional Information                               |     71037 |   50.08 |     100.00 |
| vessel\_type\_cargo |                                                         |     64656 |   45.59 |      45.59 |
| vessel\_type\_cargo | Carrying DG,HS or MP,IMO hazard or Pollutant Category X |      5275 |    3.72 |      49.31 |
| vessel\_type\_cargo | Carrying DG,HS or MP,IMO hazard or Pollutant Category Y |       865 |    0.61 |      49.92 |
| vessel\_type\_main  | TOTAL                                                   |    141833 |      NA |         NA |
| vessel\_type\_main  |                                                         |     79420 |   56.00 |      56.00 |
| vessel\_type\_main  | General Cargo Ship                                      |     36203 |   25.53 |      83.21 |
| vessel\_type\_main  | Passenger Ship                                          |     11163 |    7.87 |      96.88 |
| vessel\_type\_main  | Oil And Chemical Tanker                                 |      8230 |    5.80 |      89.01 |
| vessel\_type\_main  | Tug                                                     |      4417 |    3.11 |     100.00 |
| vessel\_type\_main  | Gas Tanker                                              |      1903 |    1.34 |      57.68 |
| vessel\_type\_main  | Fishing Vessel                                          |       438 |    0.31 |      56.34 |
| vessel\_type\_main  | Bulk Carrier                                            |        38 |    0.03 |      56.03 |
| vessel\_type\_main  | Pleasure Craft                                          |        14 |    0.01 |      96.89 |
| vessel\_type\_sub   | TOTAL                                                   |    141833 |      NA |         NA |
| vessel\_type\_sub   |                                                         |    120518 |   84.97 |      84.97 |
| vessel\_type\_sub   | Cruise Ship                                             |     11163 |    7.87 |      98.64 |
| vessel\_type\_sub   | Chemical Tanker                                         |      7365 |    5.19 |      90.77 |
| vessel\_type\_sub   | Lpg Tanker                                              |      1902 |    1.34 |      99.98 |
| vessel\_type\_sub   | Chemical Oil Products Tanker                            |       865 |    0.61 |      85.58 |
| vessel\_type\_sub   | Sailing Vessel                                          |        14 |    0.01 |      99.99 |
| vessel\_type\_sub   | Offshore Tug Supply Ship                                |         3 |    0.00 |      99.98 |
| vessel\_type\_sub   | Lng Tanker                                              |         1 |    0.00 |      98.64 |
| vessel\_type\_sub   | Offshore Support Vessel                                 |         1 |    0.00 |      99.98 |
| nav\_status         | TOTAL                                                   |    141833 |      NA |         NA |
| nav\_status         | Under Way Using Engine                                  |    116667 |   82.26 |      98.42 |
| nav\_status         | Moored                                                  |     17233 |   12.15 |      12.55 |
| nav\_status         | Not Defined                                             |      5122 |    3.61 |      16.16 |
| nav\_status         | Unknown                                                 |      2245 |    1.58 |     100.00 |
| nav\_status         | At Anchor                                               |       566 |    0.40 |       0.40 |
| source              | TOTAL                                                   |    141833 |      NA |         NA |
| source              | S-AIS                                                   |    108821 |   76.72 |      76.72 |
| source              | T-AIS                                                   |     33012 |   23.28 |     100.00 |
| nav\_status\_code   | TOTAL                                                   |    141833 |      NA |         NA |
| nav\_status\_code   | 0                                                       |    116667 |   82.26 |      82.26 |
| nav\_status\_code   | 5                                                       |     17233 |   12.15 |     100.00 |
| nav\_status\_code   | 15                                                      |      5122 |    3.61 |      86.27 |
| nav\_status\_code   | 16                                                      |      2245 |    1.58 |      87.85 |
| nav\_status\_code   | 1                                                       |       566 |    0.40 |      82.66 |
| message\_type       | TOTAL                                                   |    141833 |      NA |         NA |
| message\_type       | 1                                                       |    112809 |   79.54 |      79.54 |
| message\_type       | 3                                                       |     19499 |   13.75 |     100.01 |
| message\_type       | 27                                                      |      7059 |    4.98 |      86.26 |
| message\_type       | 18                                                      |      2466 |    1.74 |      81.28 |

</div>

## Definitions Categorical Features

**vessel\_class:** Class of Vessel (A/B). A = Targeted at large
commercial vessels. B = aimed at lighter commercial and leisure markets.
[Link](https://en.wikipedia.org/wiki/Automatic_identification_system)

**vessel\_type\_cargo:** Vessel Type Cargo

**vessel\_type\_sub:** Vessel Type Sub-Category

**flag\_country:** Country of Registration

**vessel\_type\_main:** Vessel Type Main

**vessel\_type\_sub:** Vessel Type Sub-Category

**nav\_status:** Navigational Status

**source:** Source of Position Report (S-AIS or T-AIS)

**nav\_status\_code:** Navigational Status Code

message\_type: AIS Position Message type (1,2,3,4,18,19,27). Look up
values available
[here](https://arundaleais.github.io/docs/ais/ais_message_types.html).

*1: Scheduled position report (Class A shipborne mobile equipment) *2:
Assigned scheduled position report; (Class A shipborne mobile equipment)
*3: Special position report, response to interrogation; (Class A
shipborne mobile equipment) *4: Position, UTC, date and current slot
number of base station *18: Standard position report for Class B
shipborne mobile equipment to be used instead of Messages 1, 2, 3 *19:
Extended position report for class B shipborne mobile equipment;
contains additional static information \*27: Scheduled position report;
Class A shipborne mobile equipment outside base station coverage

## Noise Reduction oo

Numerous methods are outlined to reduce noise. Replicate the procedures
outlined by
[UNSTATS](https://unstats.un.org/wiki/display/AIS/AIS+data+at+the+UN+Global+Platform).

### “Moving Ships” filter Algorithm

From UNSTATS:

> The distance travelled is calculated in the “Calculate the mount of
> motion” block by computing the minimum and maximum of latitude and
> longitudes for all ship positions over the selected period. The
> differences in latitude and longitude, the deltas, are compared
> against a predefined threshold values.

### “Time in Port” Indicator

From UNSTATS:

> The “Time in Port” indicator measures the total time spent by all
> ships within the boundaries of the port monthly over the defined
> period.

Frequency of calculation: Monthly

1.  Calculate time difference between sorted time messages for each MMSI
    saved as time deltas.
2.  Generate Port Index (arbitrary because selection is only for port
    bounds) + Time Period (Month) + Year

<!-- end list -->

``` r
# Calculate time difference 
# Sort by mmsi and dth 
cklist.list <- cklist.list[order(cklist.list$dtg),
            `:=` (time_diff_seconds = dtg - shift(dtg)),
            by = mmsi][order(cklist.list$dtg,
                             cklist.list$mmsi)]


# Create index
cklist.list[,
            index := paste0("ck-tip-",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


tim.ds <- cklist.list[,
                 .(time_in_port_seconds = sum(time_diff_seconds,
                                              na.rm = TRUE)),
                 by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)][time_in_port_seconds != 0]


# z <- tim.ds[,
#        .(sum_time_mins = sum(time_in_port_seconds, na.rm = TRUE)/60),
#        by = .(year,
#               month)]


# melt.data.table(ki.imts.bot,
#                                id = c("year",
#                                       "month"), measure = c("exports-fob-domestic",
#                                                             "exports-fob-reexport", 
#                                                             "exports-fob-total",
#                                                             "imports-cif",
#                                                             "trade-balance"))


tim.ds <- melt.data.table(tim.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "time_in_port_seconds")
```

### “Port Traffic” Indicator

From UNSTATS:

> The “Port Traffic” indicator captures how many unique ships have been
> observed in port based on their reported MMSI.

Frequency of calculation: Monthly

1.  Create time period index using mmsi + port + time period (month)
2.  Count Unique

<!-- end list -->

``` r
# Create index
cklist.list[,
            index := paste0("ck-ptr",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


ptr.ds <- cklist.list[, 
                  .(unique_count_mmsi = .N), 
                  by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)]


# z <- ptr.ds[,
#        .(sum  = sum(unique_count_mmsi, na.rm = TRUE)),
#        by = .(year,
#               month)]


ptr.ds <- melt.data.table(ptr.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "unique_count_mmsi")



tim.ds[, value := as.integer(value)]


ck_ais <- rbind(tim.ds, ptr.ds)


fwrite(ck_ais, "./data/processed/ck_ais.csv")
```

# Read Suva Fiji Islands.

Read the files extracted from UNGP for Lautoka, Fiji Islands.

``` r
# Suva files
file.list <- list.files(path = "./data/external/ais_fj_suva/", 
                        pattern='*.csv')


# Read files in bulk
suva.list <- lapply(paste0("./data/external/ais_fj_suva/",
                           file.list),
                  fread)


# turn into one table 
suva.list <- rbindlist(suva.list)


head(suva.list)
```

## Exploration

1.  Convert dtg and eta to as.POSIXct(x, tz = "", …)

<!-- end list -->

``` r
# Change from char to posixct 
suva.list[,
            `:=` (dtg = gsub("T",
                             " ",
                             dtg
                             )
                  )][,
                     `:=` (dtg = gsub(".000Z",
                             "",
                             dtg
                             )
                           )]


suva.list[,
            `:=` (dtg = strptime(dtg, "%Y-%m-%d %H:%M:%S")
                  )]


# check summary 
suva.list[,
            summary(dtg)]
```

``` r
# Check out dataset quality
ExpData(
  data = suva.list,
  type = 1
  )
```

``` r
# Check out data feature quality
ExpData(
  data = suva.list,
  type = 2
  )
```

``` r
# Statistics for Numerical features 
data.table(
  ExpNumStat(
    suva.list,
    by = "A",
    gp = NULL,
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)
```

``` r
# Statistics for Numerical features by Vessel Type
data.table(
  ExpNumStat(
    suva.list,
    by = "GA",
    gp = "vessel_type",
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)[order(-TN),
  head(.SD,
       10),
  by = Vname]
```

``` r
# Statistics for Categorigal features
data.table(
  ExpCTable(
    suva.list,
    Target = NULL,
    margin = 1,
    clim = 50,
    nlim = 10,
    round = 2,
    bin = 3,
    per = TRUE
  )
)[order(-Frequency),
  head(.SD,
       10),
  by = Variable]
```

## Noise Reduction

Numerous methods are outlined to reduce noise. Replicate the procedures
outlined by
[UNSTATS](https://unstats.un.org/wiki/display/AIS/AIS+data+at+the+UN+Global+Platform).

### “Moving Ships” filter Algorithm

From UNSTATS:

> The distance travelled is calculated in the “Calculate the mount of
> motion” block by computing the minimum and maximum of latitude and
> longitudes for all ship positions over the selected period. The
> differences in latitude and longitude, the deltas, are compared
> against a predefined threshold values.

### “Time in Port” Indicator

From UNSTATS:

> The “Time in Port” indicator measures the total time spent by all
> ships within the boundaries of the port monthly over the defined
> period.

Frequency of calculation: Monthly

1.  Calculate time difference between sorted time messages for each MMSI
    saved as time deltas.
2.  Generate Port Index (arbitrary because selection is only for port
    bounds) + Time Period (Month) + Year

<!-- end list -->

``` r
# Calculate time difference 
# Sort by mmsi and dth 
suva.list <- suva.list[order(suva.list$dtg),
            `:=` (time_diff_seconds = dtg - shift(dtg)),
            by = mmsi][order(suva.list$dtg,
                             suva.list$mmsi)]


# Create index
suva.list[,
            index := paste0("fj-suva-tip-",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


tim.ds <- suva.list[,
                 .(time_in_port_seconds = sum(time_diff_seconds,
                                              na.rm = TRUE)),
                 by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)][time_in_port_seconds != 0]


# z <- tim.ds[,
#        .(sum_time_mins = sum(time_in_port_seconds, na.rm = TRUE)/60),
#        by = .(year,
#               month)]


# melt.data.table(ki.imts.bot,
#                                id = c("year",
#                                       "month"), measure = c("exports-fob-domestic",
#                                                             "exports-fob-reexport", 
#                                                             "exports-fob-total",
#                                                             "imports-cif",
#                                                             "trade-balance"))


tim.ds <- melt.data.table(tim.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "time_in_port_seconds")
```

### “Port Traffic” Indicator

From UNSTATS:

> The “Port Traffic” indicator captures how many unique ships have been
> observed in port based on their reported MMSI.

Frequency of calculation: Monthly

1.  Create time period index using mmsi + port + time period (month)
2.  Count Unique

<!-- end list -->

``` r
# Create index
suva.list[,
            index := paste0("fj-suva-ptr",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


ptr.ds <- suva.list[, 
                  .(unique_count_mmsi = .N), 
                  by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)]


# z <- ptr.ds[,
#        .(sum  = sum(unique_count_mmsi, na.rm = TRUE)),
#        by = .(year,
#               month)]


ptr.ds <- melt.data.table(ptr.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "unique_count_mmsi")



tim.ds[, value := as.integer(value)]


fj_suva_ais <- rbind(tim.ds, ptr.ds)


fwrite(fj_suva_ais, "./data/processed/fj_suva_ais.csv")
```

# Read Lautoka Fiji Islands

Read the files extracted from UNGP for Lautoka, Fiji Islands.

``` r
# Suva files
file.list <- list.files(path = "./data/external/ais_fj_lautoka/", 
                        pattern='*.csv')


# Read files in bulk
lautoka.list <- lapply(paste0("./data/external/ais_fj_lautoka/",
                           file.list),
                  fread)


# turn into one table 
lautoka.list <- rbindlist(lautoka.list)


head(lautoka.list)
```

## Exploration

1.  Convert dtg and eta to as.POSIXct(x, tz = "", …)

<!-- end list -->

``` r
# Change from char to posixct 
lautoka.list[,
            `:=` (dtg = gsub("T",
                             " ",
                             dtg
                             )
                  )][,
                     `:=` (dtg = gsub(".000Z",
                             "",
                             dtg
                             )
                           )]


lautoka.list[,
            `:=` (dtg = strptime(dtg, "%Y-%m-%d %H:%M:%S")
                  )]


# check summary 
lautoka.list[,
            summary(dtg)]
```

``` r
# Check out dataset quality
ExpData(
  data = lautoka.list,
  type = 1
  )
```

``` r
# Check out data feature quality
ExpData(
  data = lautoka.list,
  type = 2
  )
```

``` r
# Statistics for Numerical features 
data.table(
  ExpNumStat(
    lautoka.list,
    by = "A",
    gp = NULL,
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)
```

``` r
# Statistics for Numerical features by Vessel Type
data.table(
  ExpNumStat(
    lautoka.list,
    by = "GA",
    gp = "vessel_type",
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)[order(-TN),
  head(.SD,
       10),
  by = Vname]
```

``` r
# Statistics for Categorigal features
data.table(
  ExpCTable(
    lautoka.list,
    Target = NULL,
    margin = 1,
    clim = 50,
    nlim = 10,
    round = 2,
    bin = 3,
    per = TRUE
  )
)[order(-Frequency),
  head(.SD,
       10),
  by = Variable]
```

## Noise Reduction

Numerous methods are outlined to reduce noise. Replicate the procedures
outlined by
[UNSTATS](https://unstats.un.org/wiki/display/AIS/AIS+data+at+the+UN+Global+Platform).

### “Moving Ships” filter Algorithm

From UNSTATS:

> The distance travelled is calculated in the “Calculate the mount of
> motion” block by computing the minimum and maximum of latitude and
> longitudes for all ship positions over the selected period. The
> differences in latitude and longitude, the deltas, are compared
> against a predefined threshold values.

``` 
```

### “Time in Port” Indicator

From UNSTATS:

> The “Time in Port” indicator measures the total time spent by all
> ships within the boundaries of the port monthly over the defined
> period.

Frequency of calculation: Monthly

1.  Calculate time difference between sorted time messages for each MMSI
    saved as time deltas.
2.  Generate Port Index (arbitrary because selection is only for port
    bounds) + Time Period (Month) + Year

<!-- end list -->

``` r
# Calculate time difference 
# Sort by mmsi and dth 
lautoka.list <- lautoka.list[order(lautoka.list$dtg),
            `:=` (time_diff_seconds = dtg - shift(dtg)),
            by = mmsi][order(lautoka.list$dtg,
                             lautoka.list$mmsi)]


# Create index
lautoka.list[,
            index := paste0("fj-suva-tip-",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


tim.ds <- lautoka.list[,
                 .(time_in_port_seconds = sum(time_diff_seconds,
                                              na.rm = TRUE)),
                 by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)][time_in_port_seconds != 0]


# z <- tim.ds[,
#        .(sum_time_mins = sum(time_in_port_seconds, na.rm = TRUE)/60),
#        by = .(year,
#               month)]


# melt.data.table(ki.imts.bot,
#                                id = c("year",
#                                       "month"), measure = c("exports-fob-domestic",
#                                                             "exports-fob-reexport", 
#                                                             "exports-fob-total",
#                                                             "imports-cif",
#                                                             "trade-balance"))


tim.ds <- melt.data.table(tim.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "time_in_port_seconds")
```

### “Port Traffic” Indicator

From UNSTATS:

> The “Port Traffic” indicator captures how many unique ships have been
> observed in port based on their reported MMSI.

Frequency of calculation: Monthly

1.  Create time period index using mmsi + port + time period (month)
2.  Count Unique

<!-- end list -->

``` r
# Create index
lautoka.list[,
            index := paste0("fj-suva-ptr",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


ptr.ds <- lautoka.list[, 
                  .(unique_count_mmsi = .N), 
                  by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)]


# z <- ptr.ds[,
#        .(sum  = sum(unique_count_mmsi, na.rm = TRUE)),
#        by = .(year,
#               month)]


ptr.ds <- melt.data.table(ptr.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "unique_count_mmsi")



tim.ds[, value := as.integer(value)]


fj_ltka_ais <- rbind(tim.ds, ptr.ds)


fwrite(fj_ltka_ais, "./data/processed/fj_ltka_ais.csv")
```

# Read Noro Solomon Islands

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

## Exploration

1.  Convert dtg and eta to as.POSIXct(x, tz = "", …)

<!-- end list -->

``` r
# Change from char to posixct 
noro.list[,
            `:=` (dtg = gsub("T",
                             " ",
                             dtg
                             )
                  )][,
                     `:=` (dtg = gsub(".000Z",
                             "",
                             dtg
                             )
                           )]


noro.list[,
            `:=` (dtg = strptime(dtg, "%Y-%m-%d %H:%M:%S")
                  )]


# check summary 
noro.list[,
            summary(dtg)]
```

``` r
# Check out dataset quality
ExpData(
  data = noro.list,
  type = 1
  )
```

``` r
# Check out data feature quality
ExpData(
  data = noro.list,
  type = 2
  )
```

``` r
# Statistics for Numerical features 
data.table(
  ExpNumStat(
    noro.list,
    by = "A",
    gp = NULL,
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)
```

``` r
# Statistics for Numerical features by Vessel Type
data.table(
  ExpNumStat(
    noro.list,
    by = "GA",
    gp = "vessel_type",
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)[order(-TN),
  head(.SD,
       10),
  by = Vname]
```

``` r
# Statistics for Categorigal features
data.table(
  ExpCTable(
    noro.list,
    Target = NULL,
    margin = 1,
    clim = 50,
    nlim = 10,
    round = 2,
    bin = 3,
    per = TRUE
  )
)[order(-Frequency),
  head(.SD,
       10),
  by = Variable]
```

## Noise Reduction

Numerous methods are outlined to reduce noise. Replicate the procedures
outlined by
[UNSTATS](https://unstats.un.org/wiki/display/AIS/AIS+data+at+the+UN+Global+Platform).

### “Moving Ships” filter Algorithm

From UNSTATS:

> The distance travelled is calculated in the “Calculate the mount of
> motion” block by computing the minimum and maximum of latitude and
> longitudes for all ship positions over the selected period. The
> differences in latitude and longitude, the deltas, are compared
> against a predefined threshold values.

### “Time in Port” Indicator

From UNSTATS:

> The “Time in Port” indicator measures the total time spent by all
> ships within the boundaries of the port monthly over the defined
> period.

Frequency of calculation: Monthly

1.  Calculate time difference between sorted time messages for each MMSI
    saved as time deltas.
2.  Generate Port Index (arbitrary because selection is only for port
    bounds) + Time Period (Month) + Year

<!-- end list -->

``` r
# Calculate time difference 
# Sort by mmsi and dth 
noro.list <- noro.list[order(noro.list$dtg),
            `:=` (time_diff_seconds = dtg - shift(dtg)),
            by = mmsi][order(noro.list$dtg,
                             noro.list$mmsi)]


# Create index
noro.list[,
            index := paste0("fj-suva-tip-",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


tim.ds <- noro.list[,
                 .(time_in_port_seconds = sum(time_diff_seconds,
                                              na.rm = TRUE)),
                 by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)][time_in_port_seconds != 0]


# z <- tim.ds[,
#        .(sum_time_mins = sum(time_in_port_seconds, na.rm = TRUE)/60),
#        by = .(year,
#               month)]


# melt.data.table(ki.imts.bot,
#                                id = c("year",
#                                       "month"), measure = c("exports-fob-domestic",
#                                                             "exports-fob-reexport", 
#                                                             "exports-fob-total",
#                                                             "imports-cif",
#                                                             "trade-balance"))


tim.ds <- melt.data.table(tim.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "time_in_port_seconds")
```

### “Port Traffic” Indicator

From UNSTATS:

> The “Port Traffic” indicator captures how many unique ships have been
> observed in port based on their reported MMSI.

Frequency of calculation: Monthly

1.  Create time period index using mmsi + port + time period (month)
2.  Count Unique

<!-- end list -->

``` r
# Create index
noro.list[,
            index := paste0("fj-suva-ptr",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


ptr.ds <- noro.list[, 
                  .(unique_count_mmsi = .N), 
                  by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)]


# z <- ptr.ds[,
#        .(sum  = sum(unique_count_mmsi, na.rm = TRUE)),
#        by = .(year,
#               month)]


ptr.ds <- melt.data.table(ptr.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "unique_count_mmsi")



tim.ds[, value := as.integer(value)]


sl_noro_ais <- rbind(tim.ds, ptr.ds)


fwrite(sl_noro_ais, "./data/processed/sl_noro_ais.csv")
```

# Read Port Vila Vanuatu

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

## Exploration

1.  Convert dtg and eta to as.POSIXct(x, tz = "", …)

<!-- end list -->

``` r
# Change from char to posixct 
port.v.list[,
            `:=` (dtg = gsub("T",
                             " ",
                             dtg
                             )
                  )][,
                     `:=` (dtg = gsub(".000Z",
                             "",
                             dtg
                             )
                           )]


port.v.list[,
            `:=` (dtg = strptime(dtg, "%Y-%m-%d %H:%M:%S")
                  )]


# check summary 
port.v.list[,
            summary(dtg)]
```

``` r
# Check out dataset quality
ExpData(
  data = port.v.list,
  type = 1
  )
```

``` r
# Check out data feature quality
ExpData(
  data = port.v.list,
  type = 2
  )
```

``` r
# Statistics for Numerical features 
data.table(
  ExpNumStat(
    port.v.list,
    by = "A",
    gp = NULL,
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)
```

``` r
# Statistics for Numerical features by Vessel Type
data.table(
  ExpNumStat(
    port.v.list,
    by = "GA",
    gp = "vessel_type",
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)[order(-TN),
  head(.SD,
       10),
  by = Vname]
```

``` r
# Statistics for Categorigal features
data.table(
  ExpCTable(
    port.v.list,
    Target = NULL,
    margin = 1,
    clim = 50,
    nlim = 10,
    round = 2,
    bin = 3,
    per = TRUE
  )
)[order(-Frequency),
  head(.SD,
       10),
  by = Variable]
```

## Noise Reduction

Numerous methods are outlined to reduce noise. Replicate the procedures
outlined by
[UNSTATS](https://unstats.un.org/wiki/display/AIS/AIS+data+at+the+UN+Global+Platform).

### “Moving Ships” filter Algorithm

From UNSTATS:

> The distance travelled is calculated in the “Calculate the mount of
> motion” block by computing the minimum and maximum of latitude and
> longitudes for all ship positions over the selected period. The
> differences in latitude and longitude, the deltas, are compared
> against a predefined threshold values.

### “Time in Port” Indicator

From UNSTATS:

> The “Time in Port” indicator measures the total time spent by all
> ships within the boundaries of the port monthly over the defined
> period.

Frequency of calculation: Monthly

1.  Calculate time difference between sorted time messages for each MMSI
    saved as time deltas.
2.  Generate Port Index (arbitrary because selection is only for port
    bounds) + Time Period (Month) + Year

<!-- end list -->

``` r
# Calculate time difference 
# Sort by mmsi and dth 
port.v.list <- port.v.list[order(port.v.list$dtg),
            `:=` (time_diff_seconds = dtg - shift(dtg)),
            by = mmsi][order(port.v.list$dtg,
                             port.v.list$mmsi)]


# Create index
port.v.list[,
            index := paste0("fj-suva-tip-",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


tim.ds <- port.v.list[,
                 .(time_in_port_seconds = sum(time_diff_seconds,
                                              na.rm = TRUE)),
                 by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)][time_in_port_seconds != 0]


# z <- tim.ds[,
#        .(sum_time_mins = sum(time_in_port_seconds, na.rm = TRUE)/60),
#        by = .(year,
#               month)]


# melt.data.table(ki.imts.bot,
#                                id = c("year",
#                                       "month"), measure = c("exports-fob-domestic",
#                                                             "exports-fob-reexport", 
#                                                             "exports-fob-total",
#                                                             "imports-cif",
#                                                             "trade-balance"))


tim.ds <- melt.data.table(tim.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "time_in_port_seconds")
```

### “Port Traffic” Indicator

From UNSTATS:

> The “Port Traffic” indicator captures how many unique ships have been
> observed in port based on their reported MMSI.

Frequency of calculation: Monthly

1.  Create time period index using mmsi + port + time period (month)
2.  Count Unique

<!-- end list -->

``` r
# Create index
port.v.list[,
            index := paste0("fj-suva-ptr",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


ptr.ds <- port.v.list[, 
                  .(unique_count_mmsi = .N), 
                  by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)]


# z <- ptr.ds[,
#        .(sum  = sum(unique_count_mmsi, na.rm = TRUE)),
#        by = .(year,
#               month)]


ptr.ds <- melt.data.table(ptr.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "unique_count_mmsi")



tim.ds[, value := as.integer(value)]


vn_pv_ais <- rbind(tim.ds, ptr.ds)


fwrite(vn_pv_ais, "./data/processed/vn_portvila_ais.csv")
```

# Read Luganville Vanuatu

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

## Exploration

1.  Convert dtg and eta to as.POSIXct(x, tz = "", …)

<!-- end list -->

``` r
# Change from char to posixct 
lugv.list[,
            `:=` (dtg = gsub("T",
                             " ",
                             dtg
                             )
                  )][,
                     `:=` (dtg = gsub(".000Z",
                             "",
                             dtg
                             )
                           )]


lugv.list[,
            `:=` (dtg = strptime(dtg, "%Y-%m-%d %H:%M:%S")
                  )]


# check summary 
lugv.list[,
            summary(dtg)]
```

``` r
# Check out dataset quality
ExpData(
  data = lugv.list,
  type = 1
  )
```

``` r
# Check out data feature quality
ExpData(
  data = lugv.list,
  type = 2
  )
```

``` r
# Statistics for Numerical features 
data.table(
  ExpNumStat(
    lugv.list,
    by = "A",
    gp = NULL,
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)
```

``` r
# Statistics for Numerical features by Vessel Type
data.table(
  ExpNumStat(
    lugv.list,
    by = "GA",
    gp = "vessel_type",
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)[order(-TN),
  head(.SD,
       10),
  by = Vname]
```

``` r
# Statistics for Categorigal features
data.table(
  ExpCTable(
    lugv.list,
    Target = NULL,
    margin = 1,
    clim = 50,
    nlim = 10,
    round = 2,
    bin = 3,
    per = TRUE
  )
)[order(-Frequency),
  head(.SD,
       10),
  by = Variable]
```

## Noise Reduction

Numerous methods are outlined to reduce noise. Replicate the procedures
outlined by
[UNSTATS](https://unstats.un.org/wiki/display/AIS/AIS+data+at+the+UN+Global+Platform).

### “Moving Ships” filter Algorithm

From UNSTATS:

> The distance travelled is calculated in the “Calculate the mount of
> motion” block by computing the minimum and maximum of latitude and
> longitudes for all ship positions over the selected period. The
> differences in latitude and longitude, the deltas, are compared
> against a predefined threshold values.

### “Time in Port” Indicator

From UNSTATS:

> The “Time in Port” indicator measures the total time spent by all
> ships within the boundaries of the port monthly over the defined
> period.

Frequency of calculation: Monthly

1.  Calculate time difference between sorted time messages for each MMSI
    saved as time deltas.
2.  Generate Port Index (arbitrary because selection is only for port
    bounds) + Time Period (Month) + Year

<!-- end list -->

``` r
# Calculate time difference 
# Sort by mmsi and dth 
lugv.list <- lugv.list[order(lugv.list$dtg),
            `:=` (time_diff_seconds = dtg - shift(dtg)),
            by = mmsi][order(lugv.list$dtg,
                             lugv.list$mmsi)]


# Create index
lugv.list[,
            index := paste0("fj-suva-tip-",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


tim.ds <- lugv.list[,
                 .(time_in_port_seconds = sum(time_diff_seconds,
                                              na.rm = TRUE)),
                 by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)][time_in_port_seconds != 0]


# z <- tim.ds[,
#        .(sum_time_mins = sum(time_in_port_seconds, na.rm = TRUE)/60),
#        by = .(year,
#               month)]


# melt.data.table(ki.imts.bot,
#                                id = c("year",
#                                       "month"), measure = c("exports-fob-domestic",
#                                                             "exports-fob-reexport", 
#                                                             "exports-fob-total",
#                                                             "imports-cif",
#                                                             "trade-balance"))


tim.ds <- melt.data.table(tim.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "time_in_port_seconds")
```

### “Port Traffic” Indicator

From UNSTATS:

> The “Port Traffic” indicator captures how many unique ships have been
> observed in port based on their reported MMSI.

Frequency of calculation: Monthly

1.  Create time period index using mmsi + port + time period (month)
2.  Count Unique

<!-- end list -->

``` r
# Create index
lugv.list[,
            index := paste0("fj-suva-ptr",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


ptr.ds <- lugv.list[, 
                  .(unique_count_mmsi = .N), 
                  by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)]


# z <- ptr.ds[,
#        .(sum  = sum(unique_count_mmsi, na.rm = TRUE)),
#        by = .(year,
#               month)]


ptr.ds <- melt.data.table(ptr.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "unique_count_mmsi")



tim.ds[, value := as.integer(value)]


vn_lv_ais <- rbind(tim.ds, ptr.ds)


fwrite(vn_lv_ais, "./data/processed/vn_lugv_ais.csv")
```

# Read Honoira Solomon Islands

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

## Exploration

1.  Convert dtg and eta to as.POSIXct(x, tz = "", …)

<!-- end list -->

``` r
# Change from char to posixct 
honiora.list[,
            `:=` (dtg = gsub("T",
                             " ",
                             dtg
                             )
                  )][,
                     `:=` (dtg = gsub(".000Z",
                             "",
                             dtg
                             )
                           )]


honiora.list[,
            `:=` (dtg = strptime(dtg, "%Y-%m-%d %H:%M:%S")
                  )]


# check summary 
honiora.list[,
            summary(dtg)]
```

``` r
# Check out dataset quality
ExpData(
  data = honiora.list,
  type = 1
  )
```

``` r
# Check out data feature quality
ExpData(
  data = honiora.list,
  type = 2
  )
```

``` r
# Statistics for Numerical features 
data.table(
  ExpNumStat(
    honiora.list,
    by = "A",
    gp = NULL,
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)
```

``` r
# Statistics for Numerical features by Vessel Type
data.table(
  ExpNumStat(
    honiora.list,
    by = "GA",
    gp = "vessel_type",
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)[order(-TN),
  head(.SD,
       10),
  by = Vname]
```

``` r
# Statistics for Categorigal features
data.table(
  ExpCTable(
    honiora.list,
    Target = NULL,
    margin = 1,
    clim = 50,
    nlim = 10,
    round = 2,
    bin = 3,
    per = TRUE
  )
)[order(-Frequency),
  head(.SD,
       10),
  by = Variable]
```

## Noise Reduction

Numerous methods are outlined to reduce noise. Replicate the procedures
outlined by
[UNSTATS](https://unstats.un.org/wiki/display/AIS/AIS+data+at+the+UN+Global+Platform).

### “Moving Ships” filter Algorithm

From UNSTATS:

> The distance travelled is calculated in the “Calculate the mount of
> motion” block by computing the minimum and maximum of latitude and
> longitudes for all ship positions over the selected period. The
> differences in latitude and longitude, the deltas, are compared
> against a predefined threshold values.

### “Time in Port” Indicator

From UNSTATS:

> The “Time in Port” indicator measures the total time spent by all
> ships within the boundaries of the port monthly over the defined
> period.

Frequency of calculation: Monthly

1.  Calculate time difference between sorted time messages for each MMSI
    saved as time deltas.
2.  Generate Port Index (arbitrary because selection is only for port
    bounds) + Time Period (Month) + Year

<!-- end list -->

``` r
# Calculate time difference 
# Sort by mmsi and dth 
honiora.list <- honiora.list[order(honiora.list$dtg),
            `:=` (time_diff_seconds = dtg - shift(dtg)),
            by = mmsi][order(honiora.list$dtg,
                             honiora.list$mmsi)]


# Create index
honiora.list[,
            index := paste0("fj-suva-tip-",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


tim.ds <- honiora.list[,
                 .(time_in_port_seconds = sum(time_diff_seconds,
                                              na.rm = TRUE)),
                 by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)][time_in_port_seconds != 0]


# z <- tim.ds[,
#        .(sum_time_mins = sum(time_in_port_seconds, na.rm = TRUE)/60),
#        by = .(year,
#               month)]


# melt.data.table(ki.imts.bot,
#                                id = c("year",
#                                       "month"), measure = c("exports-fob-domestic",
#                                                             "exports-fob-reexport", 
#                                                             "exports-fob-total",
#                                                             "imports-cif",
#                                                             "trade-balance"))


tim.ds <- melt.data.table(tim.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "time_in_port_seconds")
```

### “Port Traffic” Indicator

From UNSTATS:

> The “Port Traffic” indicator captures how many unique ships have been
> observed in port based on their reported MMSI.

Frequency of calculation: Monthly

1.  Create time period index using mmsi + port + time period (month)
2.  Count Unique

<!-- end list -->

``` r
# Create index
honiora.list[,
            index := paste0("fj-suva-ptr",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


ptr.ds <- honiora.list[, 
                  .(unique_count_mmsi = .N), 
                  by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)]


# z <- ptr.ds[,
#        .(sum  = sum(unique_count_mmsi, na.rm = TRUE)),
#        by = .(year,
#               month)]


ptr.ds <- melt.data.table(ptr.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "unique_count_mmsi")



tim.ds[, value := as.integer(value)]


sl_honiora_ais <- rbind(tim.ds, ptr.ds)


fwrite(sl_honiora_ais, "./data/processed/sl_honiora_ais.csv")
```

# Read Kiribati

Read the files extracted from UNGP for Betio, Kiribati.

``` r
# betio.v.list files
file.list <- list.files(path = "./data/external/ais_ki_betio/", 
                        pattern='*.csv')


# Read files in bulk
betio.v.list <- lapply(paste0("./data/external/ais_ki_betio/",
                           file.list),
                  fread)


# turn into one table 
betio.v.list <- rbindlist(betio.v.list)


betio.v.list <- betio.v.list[,1:28]


# Remove bad values that cause the write to db to fail
betio.v.list <- betio.v.list[!vessel_type_code %in% c("General Cargo Ship", "S-AIS", "TARAWA")]


# Change vessel_type_code to num type 
betio.v.list[,
             vessel_type_code := as.numeric(vessel_type_code)]


head(betio.v.list)
```

## Exploration

1.  Convert dtg and eta to as.POSIXct(x, tz = "", …)

<!-- end list -->

``` r
# Change from char to posixct 
betio.v.list[,
            `:=` (dtg = gsub("T",
                             " ",
                             dtg
                             )
                  )][,
                     `:=` (dtg = gsub(".000Z",
                             "",
                             dtg
                             )
                           )]


betio.v.list[,
            `:=` (dtg = strptime(dtg, "%Y-%m-%d %H:%M:%S")
                  )]


# check summary 
betio.v.list[,
            summary(dtg)]
```

``` r
# Check out dataset quality
ExpData(
  data = betio.v.list,
  type = 1
  )
```

``` r
# Check out data feature quality
ExpData(
  data = betio.v.list,
  type = 2
  )
```

``` r
# Statistics for Numerical features 
data.table(
  ExpNumStat(
    betio.v.list,
    by = "A",
    gp = NULL,
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)
```

``` r
# Statistics for Numerical features by Vessel Type
data.table(
  ExpNumStat(
    betio.v.list,
    by = "GA",
    gp = "vessel_type",
    Qnt = NULL,
    Nlim = 10,
    MesofShape = 2,
    Outlier = TRUE,
    round = 3,
    dcast = FALSE,
    val = NULL
  )
)[order(-TN),
  head(.SD,
       10),
  by = Vname]
```

``` r
# Statistics for Categorigal features
data.table(
  ExpCTable(
    betio.v.list,
    Target = NULL,
    margin = 1,
    clim = 50,
    nlim = 10,
    round = 2,
    bin = 3,
    per = TRUE
  )
)[order(-Frequency),
  head(.SD,
       10),
  by = Variable]
```

## Noise Reduction

Numerous methods are outlined to reduce noise. Replicate the procedures
outlined by
[UNSTATS](https://unstats.un.org/wiki/display/AIS/AIS+data+at+the+UN+Global+Platform).

### “Moving Ships” filter Algorithm

From UNSTATS:

> The distance travelled is calculated in the “Calculate the mount of
> motion” block by computing the minimum and maximum of latitude and
> longitudes for all ship positions over the selected period. The
> differences in latitude and longitude, the deltas, are compared
> against a predefined threshold values.

### “Time in Port” Indicator

From UNSTATS:

> The “Time in Port” indicator measures the total time spent by all
> ships within the boundaries of the port monthly over the defined
> period.

Frequency of calculation: Monthly

1.  Calculate time difference between sorted time messages for each MMSI
    saved as time deltas.
2.  Generate Port Index (arbitrary because selection is only for port
    bounds) + Time Period (Month) + Year

<!-- end list -->

``` r
# Calculate time difference 
# Sort by mmsi and dth 
betio.v.list <- betio.v.list[order(betio.v.list$dtg),
            `:=` (time_diff_seconds = dtg - shift(dtg)),
            by = mmsi][order(betio.v.list$dtg,
                             betio.v.list$mmsi)]


# Create index
betio.v.list[,
            index := paste0("fj-suva-tip-",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


tim.ds <- betio.v.list[,
                 .(time_in_port_seconds = sum(time_diff_seconds,
                                              na.rm = TRUE)),
                 by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)][time_in_port_seconds != 0]


# z <- tim.ds[,
#        .(sum_time_mins = sum(time_in_port_seconds, na.rm = TRUE)/60),
#        by = .(year,
#               month)]


# melt.data.table(ki.imts.bot,
#                                id = c("year",
#                                       "month"), measure = c("exports-fob-domestic",
#                                                             "exports-fob-reexport", 
#                                                             "exports-fob-total",
#                                                             "imports-cif",
#                                                             "trade-balance"))


tim.ds <- melt.data.table(tim.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "time_in_port_seconds")
```

### “Port Traffic” Indicator

From UNSTATS:

> The “Port Traffic” indicator captures how many unique ships have been
> observed in port based on their reported MMSI.

Frequency of calculation: Monthly

1.  Create time period index using mmsi + port + time period (month)
2.  Count Unique

<!-- end list -->

``` r
# Create index
betio.v.list[,
            index := paste0("fj-suva-ptr",
                                "-",
                                month = lubridate::month(dtg), 
                                "-",
                                year = lubridate::year(dtg))]


ptr.ds <- betio.v.list[, 
                  .(unique_count_mmsi = .N), 
                  by = .(index,
                        month = lubridate::month(dtg),
                        year = lubridate::year(dtg),
                        vessel_type,
                        flag_country,
                        vessel_class)]


# z <- ptr.ds[,
#        .(sum  = sum(unique_count_mmsi, na.rm = TRUE)),
#        by = .(year,
#               month)]


ptr.ds <- melt.data.table(ptr.ds, 
                          id = c("index",
                                "month",
                                 "year",
                                 "vessel_type",
                                 "flag_country",
                                 "vessel_class"),
                          measure = "unique_count_mmsi")



tim.ds[, value := as.integer(value)]


ki_betio_ais <- rbind(tim.ds, ptr.ds)


fwrite(ki_betio_ais, "./data/processed/ki_betio_ais.csv")
```
