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

## Reading the Data

`rxImport` - Used to import various file types into XDF format
  - returns either list or data.frame
  - syntax `rxImport(inData, outFile, colClasses = NULL, overwrite = FALSE, append = "none")`
      - `colClasses` is an optional argument which can be used to define variable types
      - If we specify overwrite as TRUE, any existing data in the output file will be overwritten by the results from this process.
      - `append` is an optional argument, which can be used to append multiple files (`append = "rows"`).
  - `stringsAsFactors` by default is set to true, automatically converting character variables to factors...
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
  - keep or drop certain columns from the data before writing to a new file `varsToDrop`, `varsToKeep`
  - keep or drop certain rows of the data before writing to a new file. `rowSelection` argument can be used to limit number of observations like a `where` clause
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

# Chapter 3 - Examining and Visualizing the Data

## Examining the Data

`rxSummary` Objects can be explored using `names(rxSummary(~., data))`
  - `sDataFrame` - Dsiplays default output (Mean, Min, Max, StdDev, ValidObs, MissingObs)
  - `nobs.missing`
  - `nobs.valid`
  - `categorical`
  - `params`
  - `formula`
  - `call`
  - `categorical.type`
  
`rxCrossTabs` Get counts of frequencies for combinations of factor levels
  - syntax `rxCrossTabs(formula, data, <options>)`
  - `formula` is flexible and can allowed you to analyze:
    - only one variable `~ x`
    - two variables `~ x:y`
    - or more... `~ x:y:z`

`rxHistogram` Visual display of frequency distribution
  - syntax `rxHistogram(formula, data, <options>)`
  - can specify `startVal` and `endVal`
  
## Quiz

  1. Which three elements might contain the values resulted from the `rxCrossTabs()` function call?
      + A. sums
      + B. counts
      + C. means
      + D. median

  2. What type of variables could be used in right-hand side (independent variables) on the formula arguments when using the `rxCrossTabs()` function?
      + A. Numeric
      + B. Character
      + C. Logical
      + D. Factor
      
  3. Consider the airquality dataset, which is part of Base R datasets. Assume that you have the exact same data on an XDF file named airqualiy.xdf. Which of the following code will return a data.frame with a new variable HiLo that contains "High" when Temp is above 75 and "Low" otherwise?
      + A. `rxDataStep("airquality.xdf", transforms = list(HiLo = ifelse(Temp>75,"High","Low")))`
      + B. `rxDataStep("airquality.xdf", rowSelection = (Temp > 75 & Temp <= 75))`
      + C. `rxDataStep("airquality.xdf", transformFunc = list(HiLo = ifelse(Temp>75,"High","Low")))`
      + D. `rxDataStep("airquality.xdf", rowSelection = (HiLo == "High"), transforms = list(HiLo = ifelse(Temp>75,"High","Low")))`

  4. You can set the `rowSelection` argument when using either `rxSummary()` or `rxDataStep()` function. What two considerations that you need to remember when setting this argument for these functions?
      + A. The row selection is performed before processing any data transformations. 
      + B. The row selection is performed after processing any data transformations.
      + C. The logical expression assigned to the rowSelection argument can only be defined inside the function call.
      + D. The logical expression assigned to the rowSelection can be defined outside of the function call using the expression function.
      
  5. Which two elements of the `rxSummary` object would tell you the number of missing observations in your data?
      + A. `nobs.valid`
      + B. `nobs.missing`
      + C. `sDataFrame`
      + D. `categorical`

## Answers
  1. ABC
  2. D
  3. A
  4. BD
  5. BC
  
## Visualizing the Data

# Chapter 4 - Clustering and Modeling

## Clustering
   



## General Resources <a name="general-resources"></a>

* [Learn Analytics @ Microsoft](http://learnanalytics.microsoft.com/)
edx
datacamp
Microsoft R Documentation(https://msdn.microsoft.com/en-us/microsoft-r/index)
      


