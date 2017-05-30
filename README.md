# Microsoft-70-773-ExamPrep
Study Guide for Microsoft 70-773

## Table of Contents

- [General Resources](#general-resources)
- [Exam Description](#exam-description)
- [Glossary](#glossary)

# Chapter 1 - Read Data with R Server

```{r}
# Designating column classes
col_classes <- c('VendorID' = "factor",
                 'tpep_pickup_datetime' = "character",
                 'tpep_dropoff_datetime' = "character",
                 'passenger_count' = "integer",
                 'trip_distance' = "numeric",
                 'pickup_longitude' = "numeric",
                 'pickup_latitude' = "numeric",
                 'RateCodeID' = "factor",
                 'store_and_fwd_flag' = "factor",
                 'dropoff_longitude' = "numeric",
                 'dropoff_latitude' = "numeric",
                 'payment_type' = "factor",
                 'fare_amount' = "numeric",
                 'extra' = "numeric",
                 'mta_tax' = "numeric",
                 'tip_amount' = "numeric",
                 'tolls_amount' = "numeric",
                 'improvement_surcharge' = "numeric",
                 'total_amount' = "numeric")

# Import first 1000 rows
nyc_sample_df <- read.csv('C:/yellow_tripdata_2016-01.csv', nrows = 1000, colClasses = col_classes)
# View first 10 rows
head(nyc_sample_df)

```

MRS has two ways of dealing with flat files:
   1.  it can work directly with the flat files, meaning that it can read and write to flat files directly,
   2.  it can covert flat files to a format called XDF (XDF stands for external data frame).

Advantages of XDF over CSV:
   1. XDF is compressed, and therefore much smaller than a CSV.
   2. XDF read and processed much faster than CSV.
 
Disadvantages of XDF:
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

New Functions:
   - `RxXdfData` - 
   - `RxTextData` -
   - `rxSummary`


## General Resources <a name="general-resources"></a>

* [Learn Analytics @ Microsoft](http://learnanalytics.microsoft.com/)

## Exam Description <a name="exam-description"></a>

1. Read and explore big data
   1. Read data with R Server
      - Read supported data file formats, such as text files, SAS, and SPSS; convert data to XDF format; identify trade-offs between XDF and flat text files; read data through Open Database Connectivity (ODBC) data sources; read in files from other file systems; use an internal data frame as a data source; process data from sources that cannot be read natively by R Server
   2. Summarize data
      - Compute crosstabs and univariate statistics, choose when to use **rxCrossTabs** versus **rxCube**, integrate with open source technologies by using packages such as dplyrXdf, use **group by** functionality, create complex formulas to perform multiple tasks in one pass through the data, extract quantiles by using **rxQuantile**
   3. Visualize data
      - Visualize in-memory data with base plotting functions and ggplot2; create custom visualizations with **rxSummary** and **rxCube**; visualize data with **rxHistogram** and **rxLinePlot**, including faceted plots
2. Process big data
   1. Process data with **rxDataStep**
      - Subset rows of data, modify and create columns by using the **Transforms** argument, choose when to use on-the-fly transformations versus in-data transform trade-offs, handle missing values through filtering or replacement, generate a data frame or an XDF file, process dates (**POSIXct**, **POSIXlt**)
   2. Perform complex transforms that use transform functions
      - Define a transform function; reshape data by using a transform function; use open source packages, such as lubridate; pass in values by using **transformVars** and **transformEnvir**; use internal **.rx** variables and functions for tasks, including cross-chunk communication
   3. Manage data sets
      - Sort data in various orders, such as ascending and descending; use **rxSort** deduplication to remove duplicate values; merge data sources using **rxMerge()**; merge options and types; identify when alternatives to **rxSort** and **rxMerge** should be used
   4. Process text using RML packages
      - Create features using RML functions, such as **featurizeText()**; create indicator variables and arrays using RML functions, such as **categorical()** and **categoricalHash()**; perform feature selection using RML functions
3. Build predictive models with ScaleR
   1. Estimate linear models
      - Use **rxLinMod**, **rxGlm**, and **rxLogit** to estimate linear models; set the family for a generalized linear model by using functions such as **rxTweedie**; process data on the fly by using the appropriate arguments and functions, such as the **F** function and **Transforms** argument; weight observations through frequency or probability weights; choose between different types of automatic variable selections, such as greedy searches, repeated scoring, and byproduct of training; identify the impact of missing values during automatic variable selection
   2. Build and use partitioning models
      - Use **rxDTree**, **rxDForest**, and **rxBTrees** to build partitioning models; adjust the weighting of false positives and misses by using loss; select parameters that affect bias and variance, such as pruning, learning rate, and tree depth; use **as.rpart** to interact with open source ecosystems
   3. Generate predictions and residuals
      - Use **rxPredict** to generate predictions; perform parallel scoring using **rxExec**; generate different types of predictions, such as link and response scores for GLM, response, prob, and vote for **rxDForest**; generate different types of residuals, such as Usual, Pearson, and DBM
   4. Evaluate models and tuning parameters
      - Summarize estimated models; run arbitrary code out of process, such as parallel parameter tuning by using **rxExec**; evaluate tree models by using **RevoTreeView** and **rxVarImpPlot**; calculate model evaluation metrics by using built-in functions; calculate model evaluation metrics and visualizations by using custom code, such as mean absolute percentage error and precision recall curves
   5. Create additional models using RML packages
      - Build and use a One-Class Support Vector Machine, build and use linear and logistic regressions that use L1 and L2 regularization, build and use a decision tree by using FastTree, use FastTree as a recommender with ranking loss (NDCG), build and use a simple three-layer feed-forward neural network
4. Use R Server in different environments
   1. Use different compute contexts to run R Server effectively
      - Change the compute context (**rxHadoopMR**, **rxSpark**, **rxLocalseq**, and **rxLocalParallel**); identify which compute context to use for different tasks; use different data source objects, depending on the context (**RxOdbcData** and **RxTextData**); identify and use appropriate data sources for different data sources and compute contexts (HDFS and SQL Server); debug processes across different compute contexts; identify use cases for **RevoPemaR**
   2. Optimize tasks by using local compute contexts
      - Identify and execute tasks that can be run only in the local compute context, identify tasks that are more efficient to run in the local compute context, choose between **rxLocalseq** and **rxLocalParallel**, profile across different compute contexts
   3. Perform in-database analytics by using SQL Server
      - Choose when to perform in-database versus out-of-database computations, identify limitations of in-database computations, use in-database versus out-of-database compute contexts appropriately, use stored procedures for data processing steps, serialize objects and write back to binary fields in a table, write tables, configure R to optimize SQL Server (**chunksize**, **numtasks**, and **computecontext**), effectively communicate performance properties to SQL administrators and architects (SQL Server Profiler)
   4. Implement analysis workflows in the Hadoop ecosystem and Spark
      - Use appropriate R Server functions in Spark; integrate with Hive, Pig, and Hadoop MapReduce; integrate with the Spark ecosystem of tools, such as SparklyR and SparkR; profile and tune across different compute contexts; use **doRSR** for parallelizing code that was written using open source **foreach**
   5. Deploy predictive models to SQL Server and Azure Machine Learning
      - Deploy predictive models to SQL Server as a stored procedure, deploy an arbitrary function to Azure Machine Learning by using the AzureML R package, identify when to use DeployR
      
## Glossary <a name="glossary"></a>

rxCrossTabs - blah blah blah
