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
  
# Chapter 2 - Reading and Preparing Data
  - Read supported data file formats, such as text files, SAS, and SPSS
  - Convert data to XDF format
  - Identify trade-offs between XDF and flat text files
  - Read data through Open Database Connectivity (ODBC) data sources
  - Read in files from other file systems
  - Use an internal data frame as a data source
  - Process data from sources that cannot be read natively by R Server

## Reading the Data

`rxImport` - Used to import various file types into XDF format
  - returns either list or data.frame
  - syntax `rxImport(inData, outFile, colClasses = NULL, overwrite = FALSE, append = "none")`
      - `colClasses` is an optional argument which can be used to define variable types
      - If we specify overwrite as TRUE, any existing data in the output file will be overwritten by the results from this process.
      - `append` is an optional argument, which can be used to append multiple files (`append = "rows"`).

`rxGetInfo` can be used to get some basic info about the data, including:
  - Min/Max values
  - Variable Types
  - `numRows` option can provide a quick view

`RxXdfData` - Creates pointer to an XDF file 

`RxTextData` - Creates pointer to a CSV file

`rxSummary` - Summarizes variables
  - syntax `rxSummary(formula, data, <options>)`
    - `formula` is initiated with '~' and used to specify which variables to summarize, separated by '+' (eg `~ x+ y + z`, `~.` for all)
  - By default, provides:
    - Mean
    - Standard Deviation
    - Min
    - Max
    - ValidObs
    - MissingObs
  
   
**Creating Pointers to Files**
```
# CSV Pointer

input_csv <- 'examplefile.csv' 
pointer_csv <- RxTextData(input_csv, colClasses = col_classes) 

# XDF pointer

input_xdf <- 'examplefile.xdf'
pointer_xdf <- RxXdfData(input_xdf)

```

**Importing Different File Types**
```
# Read CSV into XDF

rxImport(input_csv, input_xdf, colClasses = col_classes, overwrite = TRUE, append = append)

# Read SAS into XDF

inFileSAS <- file.path(rxGetOption("sampleDataDir"), "claims.sas7bdat")
xdfFileSAS <- "claimsSAS.xdf"
claimsSAS <- rxImport(inData = inFileSAS, outFile = xdfFileSAS)
rxGetInfo(claimsSAS, getVarInfo=TRUE)

# Sometimes, SAS data files on Windows come in two pieces, a .sasb7dat file containing the data and a .sasb7cat file containing label information.  In this case, the below procedure is used.

myData <- rxImport(inData = "myfile.sas7bdat",
     outFile ="myfile.xdf",
     formatFile = "myfile.sas7bcat")

# Read SPSS into XDF

inFileSpss <- file.path(rxGetOption("sampleDataDir"), "claims.sav")
xdfFileSpss <- "claimsSpss.xdf"
claimsSpss <- rxImport(inData = inFileSpss, outFile = xdfFileSpss)
rxGetInfo(claimsSpss, getVarInfo=TRUE)

# Read data from ODBC Source

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
   
## Quiz

  1. Which two possible R data type can be returned by the `rxImport()` function?
      + A. vector
      + B. List
      + C. data.frame
      + D. factor
      
  2. Consider the rxSummary() function. Which notation should you use for the formula argument to summarize the column trip_duration?
      + A. `+ trip_duration`
      + B. `? trip_duration`
      + C. `= trip_duration`
      + D. `~ trip_duration`
      
  3. Consider the `rxSummary()` function. The data argument is used to specify the data to summarize. What three types of data can be used with the data argument?
      + A. A data.frame object
      + B. A data source object, such as the one created using RxTextData() function
      + C. A character string specifying an XDF file
      + D. A database backup file
      
  4. Consider the `rxImport()` function. Which argument performs similar function to the argument nrows in the `read.csv()` function?
      + A. `varsToKeep`
      + B. `varsToDrop`
      + C. `rowSelection`
      + D. `numRows`
      
  5. What is the main advantage of an XDF file over a data.frame?
      + A. An XDF file can contain data larger than the local computer's memory size
      + B. You can use open source R functions directly with an XDF file
      + C. Computation using an XDF file is faster than using data.frame incorrect
      + D. An XDF file is compressed such that it fits to the local computer's memory size
      
## Answers 
  1. BC
  2. D
  3. ABC
  4. D
  5. 
  
## Preparing the Data

`rxDataStep` 
  - modify existing columns or add new columns to the data (using `transforms =` or `transformFunc =`)
  - keep or drop certain columns from the data before writing to a new file
  - keep or drop certain rows of the data before writing to a new file
  - syntax - `rxDataStep(inData, outFile, overwrite = FALSE)`
  - **Note: `outfile` is an optional argument: leaving it out will result in a** `data.frame`. 
  - `transforms` argument used to create simple transformations.
  - `transformFunc` assumes a function has already been defined outside `rxDataStep`, and can be used for more complicated transformations
  
All of the summary and analytics functions in `RevoScaleR` allow us to create new columns on-the-fly. **Note: This method does NOT write the column to the XDF. Because of the lower IO overhead, this is more efficient for a single run.**

** more info on `transformFunc`**

## Quiz

  1. Consider the `rxGetInfo()` function. Which argument performs similar function to the argument `n` in the `head()` function?
      + A. startRow
      + B. numRows 
      + C. varsToDrop
      + D. varsToKeep
      
  2. Which Microsoft R function would you use to transform data from an input data set to an output data set?
      + A. `rxImport()`
      + B. `rxSummary()`
      + C. `rxGetInfor()`
      + D. `rxDataStep()`
      
  3. You have the following code:
  
  ```
  file_1 <- 'data.csv'
  file_2 <- 'data.xdf
  data_csv <- RxTextData(file_1, colClasses = col_classes)
  data_df <- rxImport(data_csv)
  rxImport(file_1, file_2, colClasses = col_classes, overwrite = TRUE)
  ```    
  
     You are working on a local compute context and you've already set the working directory, specified `colClasses`, and imported the necessary libraries.  You can use `str(data_df)` to show the structure of the `data_df` data.frame. Which of the following code should you use to achieve the closes result for `data.xdf` using `rxGetInfo()` function?

  + A. `rxGetInfo(file_2)`
  + B. `rxGetInfo(file_2, numRows = 6)`
  + C. `rxGetInfo(file_2, getVarInfo = TRUE)`
  + D. `rxGetInfo(file_2, getValueLabels = TRUE)`
  
  5. Consider the `rxDataStep()` function. Which three of the following assignments are valid for the `transformPackages` argument?
      + A. `transformPackages = list("stringr", "lubridate")`
      + B. `transformPackages = c("stringr","lubridate")`
      + C. `transformPackages = "lubridate"`
      + D. `transformPackages = c("stringr")`
 
## Answers
  1. B
  2. D
  3. C
  4. BCD
  5. B

# Chapter 3 - 

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

