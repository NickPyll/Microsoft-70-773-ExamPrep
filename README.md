# Microsoft-70-773-ExamPrep
Study Guide for Microsoft 70-773 - A supplement to [Analyzing Big Data with R Server](https://courses.edx.org/courses/course-v1:Microsoft+DAT213x+2T2017/info)

## Table of Contents

- [General Resources](#general-resources)

# Chapter 1 - Introduction

`package` - Collection of functions, data, and compiled code.

`data.frame` - R uses a data type called a `data.frame`, which must be loaded in memory.  This can be very limiting, especially as data sets get very large.

`RevoScaleR`
  - escapes R's traditional memory limitations
  - Scales predictive modeling using parallelization
  - Distributes computation cores & nodes
  - Minimizes data movement using in-database, in-Hadoop, and in-Spark execution
  - Loads data a **chunk** at a time onto disk (500k default)

## Analytics Life Cycle
  - Preparation
    - Ingest
    - Transform
    - Explore 
  - Modeling
    - Model
    - Deploy
  - Operationalization
    - Score
    - Visualize
    - Measure

`rxImport` - Used to import various file types into R
  - syntax 

`rxOptions` - Functions to specify and retrieve options needed for `RevoScaleR` computations. These need to be set only once to carry out multiple computations.
  - `reportProgress` default value to use for reportProgress argument for many RevoScaleR functions. Options are:
      - 0: no progress is reported. 
      - 1: the number of processed rows is printed and updated.
      - 2: rows processed and timings are reported.
      - 3: rows processed and all timings are reported.
  - `traceLevel` - useful in debugging

## Quiz

  1. In the advanced analytics lifecyle, what would you typically start with?
      + A. Transform the data
      + B. Explore the data
      + C. Ingest the data 
      + D. Model the data
  
  2. Which code would you use to set `RevoScaleR` options to report only the number of processed rows?
      + A. `rxOptions(reportProgress = 0)`
      + B. `rxOptions(reportProgress = 1)`
      + C. `rxOptions(reportProgress = 2)`
      + D. `rxOptions(reportProgress = 3)`

  3. Which three statement explains how `RevoScaleR` mitigates R limitation working with data that is larger than the local computer's memory?
      + A. `RevoScaleR` points to the data that sits on the disk
      + B. `RevoScaleR` loads the data into memory a chunk at a time
      + C. `RevoScaleR` waits until all the data is loaded into memory
      + D. `RevoScaleR` processes the loaded data one chunk at a time
      
  4. Which three steps are part of the preparation phase in the advanced analytics lifecycle?
      + A. Ingest
      + B. Transform
      + C. Explore
      + D. Model
  
  5. Which two development environments with R console can you use when working with Microsoft R?
      + A. Visual Studio
      + B. SQL Server Management Studio
      + C. RStudio
      + D. Microsoft Word

## Answers
  1. C
  2. B
  3. ABD
  4. ABC
  5. AC
  


# Chapter 2 Reading and Preparing Data
  - Read supported data file formats, such as text files, SAS, and SPSS
  - Convert data to XDF format
  - Identify trade-offs between XDF and flat text files
  - Read data through Open Database Connectivity (ODBC) data sources
  - Read in files from other file systems
  - Use an internal data frame as a data source
  - Process data from sources that cannot be read natively by R Server
  






Sometimes, SAS data files on Windows come in two pieces, a `.sasb7dat` file containing the data and a `.sasb7cat` file containing label information.  In this case, the below procedure is used.

```
myData <- rxImport(inData = "myfile.sas7bdat",
     outFile ="myfile.xdf",
     formatFile = "myfile.sas7bcat")
```

## Read SPSS into XDF

```
inFileSpss <- file.path(rxGetOption("sampleDataDir"), "claims.sav")
xdfFileSpss <- "claimsSpss.xdf"
claimsSpss <- rxImport(inData = inFileSpss, outFile = xdfFileSpss)
rxGetInfo(claimsSpss, getVarInfo=TRUE)
```

## Read data from ODBC Source

```
sConnectionStr <- "Driver={SQL Server};Server=win-database01; 
        Database=TestData;Uid=mktest;Pwd=sqlpwd;"
    claimsSQL = "SELECT * FROM claims"
    claimsDS <- RxOdbcData(sqlQuery = claimsSQL,
        connectionString=sConnectionStr)
    claimsFile <- RxXdfData("claimsFromODBC.xdf")
    rxImport(claimsDS, claimsFile, overwrite=TRUE)
    rxGetInfo(claimsFile, getVarInfo=TRUE, numRows=10)
    
```

MRS has two ways of dealing with flat files:
   1.  it can work directly with the flat files, meaning that it can read and write to flat files directly,
   2.  it can covert flat files to a format called XDF (XDF stands for external data frame).

Advantages/Disadvantages of XDF over CSV: </br>
  - *Advantages*
   1. XDF is compressed, and therefore much smaller than a CSV.
   2. XDF read and processed much faster than CSV.
  - *Disadvantages* </br>
   1. Only recognized by MRS.
   2. Runtime cost associated with conversion to XDF (though quickly offset by the reduced runtime of working with XDF file)
   
```{r}

# Import first 6 months of 2016 Yellow Taxi Data into XDF

input_xdf <- 'yellow_tripdata_2016.xdf'
library(lubridate)
most_recent_date <- ymd("2016-07-01") # the day of the months is irrelevant

st <- Sys.time()
for (ii in 1:6) {
    # get each month's data and append it to the first month's data
    file_date <- most_recent_date - months(ii)
    input_csv <- sprintf('C:/yellow_tripdata_%s.csv', substr(file_date, 1, 7))
    append <- if (ii == 1) "none" else "rows"
    rxImport(input_csv, input_xdf, colClasses = col_classes, overwrite = TRUE, append = append)
    print(input_csv)
}
Sys.time() - st # stores the time it took to import
```
As you can see below, it took about 17 minutes to load these six CSV files (~2 GB each) into an XDF
```
Rows Processed: 10906858
[1] "yellow_tripdata_2016-01.csv"
Rows Processed: 11382049 
[1] "yellow_tripdata_2016-02.csv"
Rows Processed: 12210952 
[1] "yellow_tripdata_2016-03.csv"
Rows Processed: 11934338 
[1] "yellow_tripdata_2016-04.csv"
Rows Processed: 11836853 
[1] "yellow_tripdata_2016-05.csv"
Rows Processed: 11135470 
[1] "yellow_tripdata_2016-06.csv"

Time difference of 16.90247 mins
```
Now that we have the data loaded into an XDF, let's benchmark performance by performing a summary on both data forms.

```{r}

nyc_xdf <- RxXdfData(input_xdf)
system.time(
rxsum_xdf <- rxSummary(~fare_amount, nyc_xdf) # provide statistical summaries for fare amount
)
rxsum_xdf

Rows Processed: 69406520         
   user  system elapsed 
   0.02    0.00    1.98 
Call:
rxSummary(formula = ~fare_amount, data = nyc_xdf)

Summary Statistics Results for: ~fare_amount
Data: nyc_xdf (RxXdfData Data Source)
File name: yellow_tripdata_2016.xdf
Number of valid observations: 69406520 
 
 Name        Mean     StdDev   Min    Max      ValidObs MissingObs
 fare_amount 12.91626 128.1172 -957.6 628544.7 69406520 0      

```

Summary of XDF (six months of data) took about two seconds.

```

input_csv <- 'C:/Users/npylypiw/Documents/Training/Microsoft 70-773/Taxi Data/yellow_tripdata_2016-01.csv ' # we can only use one month's data unless we join the CSVs
nyc_csv <- RxTextData(input_csv, colClasses = col_classes) # point to CSV file and provide column info
system.time(
  rxsum_csv <- rxSummary(~fare_amount, nyc_csv) # provide statistical summaries for fare amount
)
rxsum_csv

   
Rows Processed: 10906858         
   user  system elapsed 
   0.03    0.00   36.70 
Call:
rxSummary(formula = ~fare_amount, data = nyc_csv)

Summary Statistics Results for: ~fare_amount
Data: nyc_csv (RxTextData Data Source)
File name: C:/yellow_tripdata_2016-01.csv
Number of valid observations: 10906858 
 
 Name        Mean     StdDev Min    Max      ValidObs MissingObs
 fare_amount 12.48693 35.564 -957.6 111270.9 10906858 0  
 
 ```

The same procedure on a much smaller CSV (one month) took 37 seconds.

`rxGetInfo` can be used to get some basic info about the data, including:
  - Min/Max values
  - Variable Types
  - `numRows` option can provide a quick view
  
```
rxGetInfo(nyc_xdf, getVarInfo = TRUE, numRows = 5)

File name: C:\yellow_tripdata_2016.xdf 
Number of observations: 69406520 
Number of variables: 19 
Number of blocks: 6 
Compression type: zlib 
Variable information: 
Var 1: VendorID
       2 factor levels: 2 1
Var 2: tpep_pickup_datetime, Type: character
Var 3: tpep_dropoff_datetime, Type: character
Var 4: passenger_count, Type: integer, Low/High: (0, 9)
Var 5: trip_distance, Type: numeric, Low/High: (-3390583.8000, 19072628.8000)
Var 6: pickup_longitude, Type: numeric, Low/High: (-165.0819, 118.4089)
Var 7: pickup_latitude, Type: numeric, Low/High: (-77.0395, 66.8568)
Var 8: RatecodeID, Type: integer, Low/High: (1, 99)
Var 9: store_and_fwd_flag
       2 factor levels: N Y
Var 10: dropoff_longitude, Type: numeric, Low/High: (-161.6987, 106.2469)
Var 11: dropoff_latitude, Type: numeric, Low/High: (-77.0395, 405.3167)
Var 12: payment_type
       5 factor levels: 2 1 3 4 5
Var 13: fare_amount, Type: numeric, Low/High: (-957.6000, 628544.7400)
Var 14: extra, Type: numeric, Low/High: (-58.5000, 648.8700)
Var 15: mta_tax, Type: numeric, Low/High: (-2.7000, 89.7000)
Var 16: tip_amount, Type: numeric, Low/High: (-220.8000, 998.1400)
Var 17: tolls_amount, Type: numeric, Low/High: (-99.9900, 1410.3200)
Var 18: improvement_surcharge, Type: numeric, Low/High: (-0.3000, 11.6400)
Var 19: total_amount, Type: numeric, Low/High: (-958.4000, 629033.7800)
Data (5 rows starting with row 1):
  VendorID tpep_pickup_datetime tpep_dropoff_datetime passenger_count trip_distance pickup_longitude pickup_latitude RatecodeID store_and_fwd_flag
1        2  2016-06-09 21:06:36   2016-06-09 21:13:08               2          0.79        -73.98336        40.76094          1                  N
2        2  2016-06-09 21:06:36   2016-06-09 21:35:11               1          5.22        -73.98172        40.73667          1                  N
3        2  2016-06-09 21:06:36   2016-06-09 21:13:10               1          1.26        -73.99432        40.75107          1                  N
4        2  2016-06-09 21:06:36   2016-06-09 21:36:10               1          7.39        -73.98236        40.77389          1                  N
5        2  2016-06-09 21:06:36   2016-06-09 21:23:23               1          3.10        -73.98711        40.73317          1                  N
  dropoff_longitude dropoff_latitude payment_type fare_amount extra mta_tax tip_amount tolls_amount improvement_surcharge total_amount
1         -73.97746         40.75398            2         6.0   0.5     0.5       0.00            0                   0.3         7.30
2         -73.98164         40.67024            1        22.0   0.5     0.5       4.00            0                   0.3        27.30
3         -74.00423         40.74217            1         6.5   0.5     0.5       1.56            0                   0.3         9.36
4         -73.92947         40.85154            1        26.0   0.5     0.5       1.00            0                   0.3        28.30
5         -73.98591         40.76645            1        13.5   0.5     0.5       2.96            0                   0.3        17.76
```
## Create Column with rxDataStep

`rxDataStep` can be used to:
  - modify existing columns or add new columns to the data
  - keep or drop certain columns from the data before writing to a new file
  - keep or drop certain rows of the data before writing to a new file

**Note: `outfile` is an optional argument: leaving it out will result in a** `data.frame`. 

Here we can use `rxDataStep` to create a tip percentage calculated column.
```
rxDataStep(nyc_xdf, nyc_xdf, 
           transforms = list(tip_percent = ifelse(fare_amount > 0 & tip_amount < fare_amount, round(tip_amount*100 / fare_amount, 0), NA)),
           overwrite = TRUE)
rxSummary( ~ tip_percent, nyc_xdf)
```

All of the summary and analytics functions in `RevoScaleR` allow us to create new columns on-the-fly like below. **Note: This method does NOT write the column to the XDF. Because of the lower IO overhead, this is more efficient for a single run.**

```
rxSummary( ~ tip_percent2, nyc_xdf, 
  transforms = list(tip_percent2 = ifelse(fare_amount > 0 & tip_amount < fare_amount, round(tip_amount * 100 / fare_amount, 0), NA)))

```
Suppose we want to view counts by month, but do not necessarily need these columns written to the XDF.  We can use `rxCrossTabs` to create the transformed columns AND produce the Cross Tabulation.

```
rxCrossTabs( ~ month:year, nyc_xdf, 
             transforms = list(
               date = ymd_hms(tpep_pickup_datetime), 
               year = factor(year(date), levels = 2014:2016), 
               month = factor(month(date), levels = 1:12)), 
             transformPackages = "lubridate")
             
```
The output for this procedure is below.
```
Rows Processed: 69406520         
Call:
rxCrossTabs(formula = ~month:year, data = nyc_xdf, transforms = list(date = ymd_hms(tpep_pickup_datetime), 
    year = factor(year(date), levels = 2014:2016), month = factor(month(date), 
        levels = 1:12)), transformPackages = "lubridate")

Cross Tabulation Results for: ~month:year
Data: nyc_xdf (RxXdfData Data Source)
File name: yellow_tripdata_2016.xdf
Number of valid observations: 69406520
Number of missing observations: 0 
Statistic: counts 
 
month:year (counts):
     year
month 2014 2015     2016
   1     0    0 10906858
   2     0    0 11382049
   3     0    0 12210952
   4     0    0 11934338
   5     0    0 11836853
   6     0    0 11135470
   7     0    0        0
   8     0    0        0
   9     0    0        0
   10    0    0        0
   11    0    0        0
   12    0    0        0
```

## Complex Transformations

The above examples are relatively simple transformations, but there is a more efficient way to do complex or multiple transformations is to create a custom function and pass it to the `transformFunc` argument.

```
xforms <- function(data) { # transformation function for extracting some date and time features

  weekday_labels <- c('Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat')
  cut_levels <- c(1, 5, 9, 12, 16, 18, 22)
  hour_labels <- c('1AM-5AM', '5AM-9AM', '9AM-12PM', '12PM-4PM', '4PM-6PM', '6PM-10PM', '10PM-1AM')

  pickup_datetime <- ymd_hms(data$tpep_pickup_datetime, tz = "UTC")
  pickup_hour <- addNA(cut(hour(pickup_datetime), cut_levels))
  pickup_dow <- factor(wday(pickup_datetime), levels = 1:7, labels = weekday_labels)
  levels(pickup_hour) <- hour_labels

  dropoff_datetime <- ymd_hms(data$tpep_dropoff_datetime, tz = "UTC")
  dropoff_hour <- addNA(cut(hour(dropoff_datetime), cut_levels))
  dropoff_dow <- factor(wday(dropoff_datetime), levels = 1:7, labels = weekday_labels)
  levels(dropoff_hour) <- hour_labels

  data$pickup_hour <- pickup_hour
  data$pickup_dow <- pickup_dow
  data$dropoff_hour <- dropoff_hour
  data$dropoff_dow <- dropoff_dow
  data$trip_duration <- as.integer(as.duration(dropoff_datetime - pickup_datetime))

  data
}
```

Let's test our custom function on the sample data.

```
library(lubridate)
Sys.setenv(TZ = "US/Eastern") # not important for this dataset
head(xforms(nyc_sample_df)) # test the function on a data.frame
```
Here we can see that the five new columns have been added to the data.frame, and appear to be working correctly.

```
 VendorID tpep_pickup_datetime tpep_dropoff_datetime passenger_count trip_distance pickup_longitude pickup_latitude RatecodeID store_and_fwd_flag
1        2  2016-06-09 21:06:36   2016-06-09 21:13:08               2          0.79        -73.98336        40.76094          1                  N
2        2  2016-06-09 21:06:36   2016-06-09 21:35:11               1          5.22        -73.98172        40.73667          1                  N
3        2  2016-06-09 21:06:36   2016-06-09 21:13:10               1          1.26        -73.99432        40.75107          1                  N
4        2  2016-06-09 21:06:36   2016-06-09 21:36:10               1          7.39        -73.98236        40.77389          1                  N
5        2  2016-06-09 21:06:36   2016-06-09 21:23:23               1          3.10        -73.98711        40.73317          1                  N
6        2  2016-06-09 21:06:36   2016-06-09 21:19:21               1          2.17        -73.99520        40.73949          1                  N
  dropoff_longitude dropoff_latitude payment_type fare_amount extra mta_tax tip_amount tolls_amount improvement_surcharge total_amount pickup_hour
1         -73.97746         40.75398            2         6.0   0.5     0.5       0.00            0                   0.3         7.30    6PM-10PM
2         -73.98164         40.67024            1        22.0   0.5     0.5       4.00            0                   0.3        27.30    6PM-10PM
3         -74.00423         40.74217            1         6.5   0.5     0.5       1.56            0                   0.3         9.36    6PM-10PM
4         -73.92947         40.85154            1        26.0   0.5     0.5       1.00            0                   0.3        28.30    6PM-10PM
5         -73.98591         40.76645            1        13.5   0.5     0.5       2.96            0                   0.3        17.76    6PM-10PM
6         -73.99320         40.76264            1        10.5   0.5     0.5       2.36            0                   0.3        14.16    6PM-10PM
  pickup_dow dropoff_hour dropoff_dow trip_duration
1        Thu     6PM-10PM         Thu           392
2        Thu     6PM-10PM         Thu          1715
3        Thu     6PM-10PM         Thu           394
4        Thu     6PM-10PM         Thu          1774
5        Thu     6PM-10PM         Thu          1007
6        Thu     6PM-10PM         Thu           765
```

Now we are ready to execute the function on the XDF.

```
st <- Sys.time()
rxDataStep(nyc_xdf, nyc_xdf, overwrite = TRUE, transformFunc = xforms, transformPackages = "lubridate")
Sys.time() - st
```

Examining the new columns to see if we got desired results.  Let's look at a quick `rxSummary`

```
rxs1 <- rxSummary( ~ pickup_hour + pickup_dow + trip_duration, nyc_xdf)
# we can add a column for proportions next to the counts
rxs1$categorical <- lapply(rxs1$categorical, function(x) cbind(x, prop = round(prop.table(x$Counts), 2)))
rxs1
```
Summary Output is below
```
Rows Processed: 69406520         
Call:
rxSummary(formula = ~pickup_hour + pickup_dow + trip_duration, 
    data = nyc_xdf)

Summary Statistics Results for: ~pickup_hour + pickup_dow + trip_duration
Data: nyc_xdf (RxXdfData Data Source)
File name: yellow_tripdata_2016.xdf
Number of valid observations: 69406520 
 
 Name          Mean     StdDev   Min        Max      ValidObs MissingObs
 trip_duration 933.9168 119243.5 -631148790 11538803 69406520 0         

Category Counts for pickup_hour
Number of categories: 7
Number of valid observations: 69406520
Number of missing observations: 0

 pickup_hour Counts   prop
 1AM-5AM      3801430 0.05
 5AM-9AM     10630653 0.15
 9AM-12PM     9765429 0.14
 12PM-4PM    13473045 0.19
 4PM-6PM      7946899 0.11
 6PM-10PM    16138968 0.23
 10PM-1AM     7650096 0.11

Category Counts for pickup_dow
Number of categories: 7
Number of valid observations: 69406520
Number of missing observations: 0

 pickup_dow Counts   prop
 Sun         9267881 0.13
 Mon         8938785 0.13
 Tue         9667525 0.14
 Wed         9982769 0.14
 Thu        10398738 0.15
 Fri        10655022 0.15
 Sat        10495800 0.15
 ```
Separating two variables by a colon (`pickup_dow:pickup_hour`) instead of a plus sign (`pickup_dow + pickup_hour`) allows us to get summaries for each combination of the levels of the two factor columns, instead of individual ones.
 
 ```
 rxs2 <- rxSummary( ~ pickup_dow:pickup_hour, nyc_xdf)
rxs2 <- tidyr::spread(rxs2$categorical[[1]], key = 'pickup_hour', value = 'Counts')
row.names(rxs2) <- rxs2[ , 1]
rxs2 <- as.matrix(rxs2[ , -1])
rxs2
```
```
Rows Processed: 69406520         
    1AM-5AM 5AM-9AM 9AM-12PM 12PM-4PM 4PM-6PM 6PM-10PM 10PM-1AM
Sun 1040233  740157  1396409  1980752 1032434  1697529  1380367
Mon  304474 1630951  1268326  1838143 1133728  2096219   666944
Tue  278407 1840134  1382381  1882356 1151837  2390506   741904
Wed  313809 1854757  1417953  1880896 1142071  2508618   864665
Thu  354646 1871828  1428985  1922502 1165535  2634023  1021219
Fri  553159 1766482  1406979  1922542 1173163  2516285  1316412
Sat  956702  926344  1464396  2045854 1148131  2295788  1658585
```

A `levelplot` may be helpful to view these results in a more meaningful way.

```
levelplot(prop.table(rxs2, 2), cuts = 4, xlab = "", ylab = "", main = "Distribution of taxis by day of week")
```

# PICTURE
```
library(rgeos)
library(maptools)

nyc_shapefile <- readShapePoly('ZillowNeighborhoods-NY/ZillowNeighborhoods-NY.shp')
mht_shapefile <- subset(nyc_shapefile, str_detect(CITY, 'New York City-Manhattan'))

mht_shapefile@data$id <- as.character(mht_shapefile@data$NAME)
mht.points <- fortify(gBuffer(mht_shapefile, byid = TRUE, width = 0), region = "NAME")
mht.df <- inner_join(mht.points, mht_shapefile@data, by = "id")

library(dplyr)
mht.cent <- mht.df %>%
  group_by(id) %>%
  summarize(long = median(long), lat = median(lat))

library(ggrepel)
ggplot(mht.df, aes(long, lat, fill = id)) + 
  geom_polygon() +
  geom_path(color = "white") +
  coord_equal() +
  theme(legend.position = "none") +
  geom_text_repel(aes(label = id), data = mht.cent, size = 3)
```
Functions Used:
   - `RxXdfData` - Creates pointer to an XDF file 
   - `RxTextData` - Creates pointer to a CSV file
   - `rxSummary` - Summarizes variables
   - `rxImport` - Flexible function used to import various data types into R Server. Returns either `list` or `data.frame`
   - `rxOptions` - Used to alter option shown.  (reportProgress = 1) shows only number of processed rows

## Section 1.2 - Summarize data
  - Compute crosstabs and univariate statistics
  - Choose when to use **rxCrossTabs** versus **rxCube**
  - Integrate with open source technologies by using packages such as dplyrXdf
  - Use **group by** functionality
  - Create complex formulas to perform multiple tasks in one pass through the data
  - Extract quantiles by using **rxQuantile**
  
We already looked a little at `rxSummary`, but we can output the underlying data.  It is stored in an element called `sDataFrame` and can be very useful in extracting values when necessary.

```
rxs_all <- rxSummary(~., nyc_xdf)
head(rxs_all$sDataFrame)

                   Name       Mean      StdDev           Min           Max ValidObs MissingObs
1              VendorID         NA          NA            NA            NA 69406520          0
2  tpep_pickup_datetime         NA          NA            NA            NA        0          0
3 tpep_dropoff_datetime         NA          NA            NA            NA        0          0
4       passenger_count   1.660674    1.310478        0.0000        9.0000 69406520          0
5         trip_distance   4.850022 4044.503422 -3390583.8000 19072628.8000 69406520          0
6      pickup_longitude -72.920469    8.763351     -165.0819      118.4089 69406520          0
```

Cross Tabulations are useful when we want to summarize by two variables

  
  
## Section 1.3 - Visualize data
  - Visualize in-memory data with base plotting functions and ggplot2
  - Create custom visualizations with **rxSummary** and **rxCube**
  - Visualize data with **rxHistogram** and **rxLinePlot**, including faceted plots
  
# Chapter 2 - Process big data

## Section 2.1 - Process data with **rxDataStep**
  - Subset rows of data, modify and create columns by using the **Transforms** argument
  - Choose when to use on-the-fly transformations versus in-data transform trade-offs
  - Handle missing values through filtering or replacement
  - Generate a data frame or an XDF file
  - Process dates (**POSIXct**, **POSIXlt**)
  









Functions Used:
   - `rxGetInfo` - Shows variable types, as well as min/max values
   - `rxDataStep` - Can be used to perform data manipulations
   - `rxCrossTabs` - Performs Cross Tabulation
   
   
   
# Chapter 1 - Read and explore big data

## Section 1.1 - Read Data with R Server
  - Read supported data file formats, such as text files, SAS, and SPSS
  - Convert data to XDF format
  - Identify trade-offs between XDF and flat text files
  - Read data through Open Database Connectivity (ODBC) data sources
  - Read in files from other file systems
  - Use an internal data frame as a data source
  - Process data from sources that cannot be read natively by R Server
   
## Section 1.2 - Summarize data
  - Compute crosstabs and univariate statistics
  - Choose when to use **rxCrossTabs** versus **rxCube**
  - Integrate with open source technologies by using packages such as dplyrXdf
  - Use **group by** functionality
  - Create complex formulas to perform multiple tasks in one pass through the data
  - Extract quantiles by using **rxQuantile**
  
## Section 1.3 - Visualize data
  - Visualize in-memory data with base plotting functions and ggplot2
  - Create custom visualizations with **rxSummary** and **rxCube**
  - Visualize data with **rxHistogram** and **rxLinePlot**, including faceted plots
  
# Chapter 2 - Process big data

## Section 2.1 - Process data with **rxDataStep**
  - Subset rows of data, modify and create columns by using the **Transforms** argument
  - Choose when to use on-the-fly transformations versus in-data transform trade-offs
  - Handle missing values through filtering or replacement
  - Generate a data frame or an XDF file
  - Process dates (**POSIXct**, **POSIXlt**)
## Section 2.2 - Perform complex transforms that use transform functions
  - Define a transform function
  - Reshape data by using a transform function
  - Use open source packages, such as lubridate
  - Pass in values by using **transformVars** and **transformEnvir**
  - Use internal **.rx** variables and functions for tasks, including cross-chunk communication

## Section 2.3 - Manage data sets
  - Sort data in various orders, such as ascending and descending
  - Use **rxSort** deduplication to remove duplicate values
  - Merge data sources using **rxMerge()**
  - Merge options and types
  - Identify when alternatives to **rxSort** and **rxMerge** should be used
  
## Section 2.4 - Process text using RML packages
  - Create features using RML functions, such as **featurizeText()**
  - Create indicator variables and arrays using RML functions, such as **categorical()** and **categoricalHash()**
  - Perform feature selection using RML functions
  
# Chapter 3 - Build predictive models with ScaleR

## Section 3.1 - Estimate linear models
  - Use **rxLinMod**, **rxGlm**, and **rxLogit** to estimate linear models
  - Set the family for a generalized linear model by using functions such as **rxTweedie**
  - Process data on the fly by using the appropriate arguments and functions, such as the **F** function and **Transforms** argument
  - Weight observations through frequency or probability weights
  - Choose between different types of automatic variable selections, such as greedy searches, repeated scoring, and byproduct of training
  - Identify the impact of missing values during automatic variable selection
  
## Section 3.2 - Build and use partitioning models
  - Use **rxDTree**, **rxDForest**, and **rxBTrees** to build partitioning models
  - Adjust the weighting of false positives and misses by using loss
  - Select parameters that affect bias and variance, such as pruning, learning rate, and tree depth
  - Use **as.rpart** to interact with open source ecosystems
  
## Section 3.3 - Generate predictions and residuals
  - Use **rxPredict** to generate predictions
  - Perform parallel scoring using **rxExec**
  - Generate different types of predictions, such as link and response scores for GLM, response, prob, and vote for **rxDForest**
  - Generate different types of residuals, such as Usual, Pearson, and DBM
  
## Section 3.4 - Evaluate models and tuning parameters
  - Summarize estimated models
  - Run arbitrary code out of process, such as parallel parameter tuning by using **rxExec**
  - Evaluate tree models by using **RevoTreeView** and **rxVarImpPlot**
  - Calculate model evaluation metrics by using built-in functions
  - Calculate model evaluation metrics and visualizations by using custom code, such as mean absolute percentage error and precision recall curves
  
## Section 3.5 - Create additional models using RML packages
  - Build and use a One-Class Support Vector Machine
  - Build and use linear and logistic regressions that use L1 and L2 regularization
  - Build and use a decision tree by using FastTree
  - Use FastTree as a recommender with ranking loss (NDCG)
  - Build and use a simple three-layer feed-forward neural network
  
# Chapter 4 - Use R Server in different environments

## Section 4.1 - Use different compute contexts to run R Server effectively
  - Change the compute context (**rxHadoopMR**, **rxSpark**, **rxLocalseq**, and **rxLocalParallel**)
  - Identify which compute context to use for different tasks
  - Use different data source objects, depending on the context (**RxOdbcData** and **RxTextData**)
  - Identify and use appropriate data sources for different data sources and compute contexts (HDFS and SQL Server)
  - Debug processes across different compute contexts
  - Identify use cases for **RevoPemaR**

## Section 4.2 - Optimize tasks by using local compute contexts
  - Identify and execute tasks that can be run only in the local compute context
  - Identify tasks that are more efficient to run in the local compute context
  - Choose between **rxLocalseq** and **rxLocalParallel**
  - Profile across different compute contexts
  
## Section 4.3 - Perform in-database analytics by using SQL Server
  - Choose when to perform in-database versus out-of-database computations
  - Identify limitations of in-database computations
  - Use in-database versus out-of-database compute contexts appropriately
  - Use stored procedures for data processing steps
  - Serialize objects and write back to binary fields in a table
  - Write tables
  - Configure R to optimize SQL Server (**chunksize**, **numtasks**, and **computecontext**)
  - Effectively communicate performance properties to SQL administrators and architects (SQL Server Profiler)  
  
  
Write tables back to database

```
myTable <- RxOdbcData(table="mtcars", connectionString = connectionString)
rxDataStep(mtcars, myTable, overwrite=TRUE) 
head(myTable) 
rxGetInfo(myTable, getVarInfo=TRUE) 
myCars <- rxDataStep(myTable) 
head(myCars)
```

## Section 4.4 - Implement analysis workflows in the Hadoop ecosystem and Spark
  - Use appropriate R Server functions in Spark; integrate with Hive, Pig, and Hadoop MapReduce
  - Integrate with the Spark ecosystem of tools, such as SparklyR and SparkR
  - Profile and tune across different compute contexts
  - Use **doRSR** for parallelizing code that was written using open source **foreach**
  
## Section 4.5 - Deploy predictive models to SQL Server and Azure Machine Learning
  - Deploy predictive models to SQL Server as a stored procedure
  - Deploy an arbitrary function to Azure Machine Learning by using the AzureML R package
  - Identify when to use DeployR


## General Resources <a name="general-resources"></a>

* [Learn Analytics @ Microsoft](http://learnanalytics.microsoft.com/)

      
## Glossary <a name="glossary"></a>

rxCrossTabs - blah blah blah
rxImport

