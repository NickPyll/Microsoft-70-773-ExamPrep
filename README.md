# Microsoft-70-773-ExamPrep
Study Guide for Microsoft 70-773 - A supplement to [Analyzing Big Data with R Server](https://courses.edx.org/courses/course-v1:Microsoft+DAT213x+2T2017/info)

## Table of Contents

- [General Resources](#general-resources)
- [Chapter 1 - Introduction](#chapter-1)
  - [Chapter 1 Quiz](#quiz1)
- [Chapter 2 - Reading and Preparing Data](#chapter-2)  
  - [Chapter 2 Quiz A](#quiz2a)
  - [Chapter 2 Quiz B](#quiz2b)
- [Chapter 3 - Examining and Visualizing the Data](#chapter-3) 
  - [Chapter 3 Quiz A](#quiz3a)
  - [Chapter 3 Quiz B](#quiz3b)
- [Chapter 4 - Clustering and Modeling](#chapter-4)
  - [Chapter 4 Quiz](#quiz4)
  - [Modeling Lab](#modeling-lab)
- [Chapter 5 - Deploying and Scaling](#chapter-5)
  - [Chapter 5 Quiz](#quiz5)
- [Final Exam](#exam)  
  
# Chapter 1 - Introduction <a name="chapter-1"></a>

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

## Quiz 1 <a name="quiz1"></a>

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

## Quiz 1 Answers

  1. C
  2. B
  3. ABD
  4. ABC
  5. AC
  
# Chapter 2 - Reading and Preparing Data <a name="chapter-2"></a>
  - Read supported data file formats, such as text files, SAS, and SPSS
  - Convert data to XDF format
  - Identify trade-offs between XDF and flat text files
  - Read data through Open Database Connectivity (ODBC) data sources
  - Read in files from other file systems
  - Use an internal data frame as a data source
  - Process data from sources that cannot be read natively by R Server
  - Create complex formulas to perform multiple tasks in one pass through the data
  - Subset rows of data, modify and create columns by using the **Transforms** argument
  - Choose when to use on-the-fly transformations versus in-data transform trade-offs
  - Handle missing values through filtering or replacement
  - Generate a data frame or an XDF file
  - Process dates (**POSIXct**, **POSIXlt**)
  - Define a transform function
  - Reshape data by using a transform function
  - Use open source packages, such as lubridate
  - Pass in values by using **transformVars** and **transformEnvir**
  - Use internal **.rx** variables and functions for tasks, including cross-chunk communication
  - Sort data in various orders, such as ascending and descending
  - Use **rxSort** deduplication to remove duplicate values
  - Merge data sources using **rxMerge()**
  - Merge options and types
  - Identify when alternatives to **rxSort** and **rxMerge** should be used
  - Create features using RML functions, such as **featurizeText()**
  - Create indicator variables and arrays using RML functions, such as **categorical()** and **categoricalHash()**
  - Perform feature selection using RML functions
  
## Reading the Data

`rxImport` - Used to import various file types into XDF format
  - returns either list or data.frame
  - syntax `rxImport(inData, outFile = NULL, colClasses = NULL, overwrite = FALSE, append = "none")`
      - `outFile` a character string representing the output .xdf file, a `RxHiveData` data source, a `RxParquetData` data source or a `RxXdfData object`. If `NULL`, a data frame will be returned in memory
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
   
## Quiz 2a <a name="quiz2a"></a>

  1. Which two possible R data type can be returned by the `rxImport()` function?
      + A. vector
      + B. List
      + C. data.frame
      + D. factor
      
  2. Consider the `rxSummary()` function. Which notation should you use for the formula argument to summarize the column trip_duration?
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

## Quiz 2b <a name="quiz2b"></a>

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

# Chapter 3 - Examining and Visualizing the Data <a name="chapter-3"></a>
  - Compute crosstabs and univariate statistics
  - Choose when to use **rxCrossTabs** versus **rxCube**
  - Integrate with open source technologies by using packages such as dplyrXdf
  - Use **group by** functionality
  - Visualize in-memory data with base plotting functions and ggplot2
  - Create custom visualizations with **rxSummary** and **rxCube**
  - Visualize data with **rxHistogram** and **rxLinePlot**, including faceted plots
  
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
  
`rxCrossTabs` Get counts (and sums when argument on left side) for combinations of factor levels
  - syntax `rxCrossTabs(formula, data, <options>)`
  - `formula` is flexible and can allowed you to analyze:
    - only one variable `~ x`
    - two variables `~ x:y`
    - or more... `~ x:y:z`
    - sums of combinations... `sales ~ product:region`
    - `means = TRUE` argument can be used to get similar output as `rxCube`

`rxHistogram` Visual display of frequency distribution
  - syntax `rxHistogram(formula, data, <options>)`
  - can specify `startVal` and `endVal`

`rxLinePlot`

`rxCor` - displays correlation matrix
  - syntax `rxCor(formula, data)`
  
## Quiz 3a <a name="chapter-3a"></a>

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

`rxCube` Get counts (and means when argument on left side) for combinations of factor levels
  - syntax `rxCube(formula, data, <options>)`
  - `formula` is flexible and can allowed you to analyze:
    - only one variable `~ x`
    - two variables `~ x:y`
    - or more... `~ x:y:z`
    - means of combinations... `price ~ product:region` 

`rxFactors`- Another way to modify or create factors

## Quiz 3b <a name="chapter-3b"></a>

  1. Consider the airquality dataset, which is part of Base R datasets. Assume that you have the exact same data on an XDF file named airqualiy.xdf.  Which of the following code will produce similar values for Temp variable to `rxCube(Temp ~ Day:Month, "airquality.xdf")`?
      + A. `rxCrossTabs(Temp ~ Day:Month, "airquality.xdf")`
      + B. `rxCrossTabs(Temp ~ Month:Day, "airquality.xdf")`
      + C. `rxCrossTabs(Temp ~ Day:Month, "airquality.xdf", means = TRUE)`
      + D. `xCrossTabs(Temp ~ Day:Month, "airquality.xdf", means = FALSE)`

  2. Consider the airquality dataset, which is part of Base R datasets. Assume that you have the exact same data on an XDF file named airqualiy.xdf and the following code below:
```
    newlevs <- c("1973","1974","1975") 
    rxFactors("airquality.xdf", outFile = "airquality.xdf", 
          factorInfo = list(Year = list(newLevels = unique(newlevs))))
```    
Which of the following code will produce similar results?\

```
# A
rxDataStep("airquality.xdf", outFile = "airquality.xdf", 
           transforms = list(Year = list(newLevels = unique(newlevs))), 
           overwrite = TRUE)
# B
rxDataStep("airquality.xdf", outFile = "airquality.xdf", 
           transforms = list(Year = factor(Year, levels = newlevs)), 
           overwrite = TRUE)
# C
rxDataStep("airquality.xdf", outFile = "airquality.xdf", 
           transformObjects = list(Year = list(newLevels = unique(newlevs))), 
           overwrite = TRUE)
# D
rxDataStep("airquality.xdf", outFile = "airquality.xdf", 
           transforms = list(Year = factor(Year, levels = newlevels)), 
           transformObjects = list(newlevels = unique(newlevs)), 
           overwrite = TRUE)
```           

  3. Consider the airquality dataset, which is part of Base R datasets. Assume that you have the exact same data on an XDF file named airqualiy.xdf and the following code below:

```
    newlevs <- c("1973","1974","1975") 
    rxDataStep("airquality.xdf", outFile = "airquality.xdf", 
               transforms = list(Year = factor(Year, levels = newlevels)), 
               transformObjects = list(newlevels = unique(newlevs)), 
               overwrite = TRUE) 
 ```   
Which of the following code will produce similar results?

```
# A
rxFactors("airquality.xdf", outFile = "airquality.xdf", 
          transforms = list(Year = factor(Year, levels = newlevels)), 
          transformObjects = list(newlevels = unique(newlevs)), 
          overwrite = TRUE)
# B
rxFactors("airquality.xdf", outFile = "airquality.xdf", 
          transforms = list(Year = list(newLevels = unique(newlevs))), 
          overwrite = TRUE)
# C
rxFactors("airquality.xdf", outFile = "airquality.xdf", 
          factorInfo = list(Year = list(newLevels = unique(newlevs))))
# D
rxFactors("airquality.xdf", outFile = "airquality.xdf", 
          factorInfo = list(Year = factor(Year, levels = newlevs)))
```

  4. Which two are the differences between the output of `rxCrossTabs()` and `rxCube()`?
      + A. The output of `rxCrossTabs()` function is in wide format.
      + B. The output of `rxCrossTabs()` function is in long format.
      + C. The output of `rxCube()` function is in wide format.
      + D. The output of `rxCube()` function is in long format.

  5. What are the three possible values that can be returned by the `rxCube()` function?
      + A. a Text file
      + B. an RxXdfData object
      + C. a List
      + D. a data.frame

## Answers
  1. C
  2. D
  3. C
  4. AD
  5. BCD
  
# Chapter 4 - Clustering and Modeling <a name="chapter-4"></a>
  - Extract quantiles by using **rxQuantile**
  - Use **rxLinMod**, **rxGlm**, and **rxLogit** to estimate linear models
  - Set the family for a generalized linear model by using functions such as **rxTweedie**
  - Process data on the fly by using the appropriate arguments and functions, such as the **F** function and **Transforms** argument
  - Weight observations through frequency or probability weights
  - Choose between different types of automatic variable selections, such as greedy searches, repeated scoring, and byproduct of training
  - Identify the impact of missing values during automatic variable selection
  - Use **rxDTree**, **rxDForest**, and **rxBTrees** to build partitioning models
  - Adjust the weighting of false positives and misses by using loss
  - Select parameters that affect bias and variance, such as pruning, learning rate, and tree depth
  - Use **as.rpart** to interact with open source ecosystems
  - Use **rxPredict** to generate predictions
  - Perform parallel scoring using **rxExec**
  - Generate different types of predictions, such as link and response scores for GLM, response, prob, and vote for **rxDForest**
  - Generate different types of residuals, such as Usual, Pearson, and DBM
  - Summarize estimated models
  - Run arbitrary code out of process, such as parallel parameter tuning by using **rxExec**
  - Evaluate tree models by using **RevoTreeView** and **rxVarImpPlot**
  - Calculate model evaluation metrics by using built-in functions
  - Calculate model evaluation metrics and visualizations by using custom code, such as mean absolute percentage error and precision recall curves
  - Build and use a One-Class Support Vector Machine
  - Build and use linear and logistic regressions that use L1 and L2 regularization
  - Build and use a decision tree by using FastTree
  - Use FastTree as a recommender with ranking loss (NDCG)
  - Build and use a simple three-layer feed-forward neural network     
  
## Clustering
   
`rxKmeans`
  - clustering algorithm
  - `rxKmeans(formula, data, <options>)`
  - must specify **only one** of `numClusters` or `centers`
  
## Modeling

`rxLinMod`
  - `rxLinMod(formula, data, <options>)`
  - Lots of output available including `r.squared`, `f.pvalue`, `coef.p.value`

`rxDTree`

`rxDForest`

`rxSplit`

`rxCor`

`rxPredict`
  - input can be XDF, data.frame, or character string representing XDF
  - `computeResiduals = TRUE` is used to get residual values

`rxQuantile`
  - `rxQuantile("varName", data, probs = seq(0, 1, 0.25))`

## Quiz 4 <a name="quiz4"></a>

  1. Consider the airquality dataset, which is part of Base R datasets. You have the following code:

```
    z <- airquality[,c("Wind","Temp")] 
    cl <- kmeans(z, 2) 
    cl$cluster
```   
  Which of the following code will produce similar result using the `rxKmeans()` function?
     + A. `cl <- rxKmeans(~Wind+Temp,data=z,centers=2)`
     + B. `cl <- rxKmeans(~Wind+Temp,data=z,numClusters=2)`
     + C. `cl <- rxKmeans(~Wind+Temp,data=z,reportProgress=2)`
     + D. `cl <- rxKmeans(~Wind+Temp,data=z,maxIterations=2)`
     
  2. Which three arguments are mandatory when using the rxKmeans() function?
     + A. either centers or numClusters
     + B. formula
     + C. data
     + D. maxIterations

  3. Consider the father.son dataset from the UsingR package. You can create a linear regression model fitting the son's height (sheight) by the father's height (fheight) with the following code:

```
    regfit<-lm(sheight ~ fheight, data=father.son)
```    
  Which of the following code return similar result when using the rxLinMod() function?
     + A. `rxLinMod(sheight ~ fheight, data="father.son.xdf")`
     + B. `rxLinMod(sheight ~ fheight, data=father.son.xdf)`
     + C. `rxLinMod(sheight ~ fheight, data=father.son)`
     + D. `xLinMod(sheight ~ fheight, data=RxXdfData(father.son))`
     
  4. You have created a linear regression model named rxlm1. Which of the following shows more information about the model coefficients, together with the residual standard error, multiple R-squared, and adjusted R-squared?
      + A. `rxSummary(rxlm1)`
      + B. `summary(rxlm1)`
      + C. `rxlm1`
      + D. `rxlm1$summary`
      
  5. When using `rxLinMod()` function, which argument you need to set to return the variance-covariance matrix of the regression coefficients?
      + A. `coefLabelStyle`
      + B. `covariance`
      + C. `covData`
      + D. `covCoef`

  6. Which three scenarios are possible when using the `rxPredict()` function?
      + A. If a data frame is specified as the input data, a data frame is returned.
      + B. If a data frame is specified as the input data and the outData is NULL, an RxXdfData data source object is returned.
      + C. If an XDF file is specified as the input data, an RxXdfData data source object is returned
      + D. If an XDF file is specified as the input data and the outData is NULL, the predicted values are appended to the original data file.

  7. Which three types of data can be used when using the `rxDForest()` function?
      + A. A data.frame
      + B. A data source object
      + C. A character string defining the path to the input XDF file
      + D. A matrix

  8. Consider the father.son dataset from the UsingR package. You have split the dataset into training and test dataset named fstrain and fstest respectively. You can train a linear regression model using the training dataset fitting the son's height by the father's height with the following code: 

```
    regfit<-lm(sheight ~ fheight, data=fstrain)
```    
You can obtain the fitted values of the model using the following code:

```
   regfit$fitted.values 
```   
Which of the following code return similar result when using the `rxLinMod()` function?

```
# A
regfit2 <- rxLinMod(sheight ~ fheight, data=fstrain) 
regfit2$fitted.values

# B
regfit2 <- rxLinMod(sheight ~ fheight, data=fstrain) 
rxPredict(regfit2, data=fstrain)

# C
regfit2 <- rxLinMod(sheight ~ fheight, data=fstrain) 
rxPredict(regfit2, data=fstest)

# D
regfit2 <- rxLinMod(sheight ~ fheight, data=fstrain) 
rxPredict(regfit2, data=fstrain, computeResiduals = TRUE)

# E
regfit2 <- rxLinMod(sheight ~ fheight, data=fstrain) 
rxPredict(regfit2, data=fstest, computeResiduals = TRUE)
```

  9. When using `rxLinMod()` function, which value would you set to the `reportProgress` argument so that only the number of processed rows printed and updated are reported?

      + A. 0
      + B. 1
      + C. 2
      + D. 3
      
  10. With reference to the New York Taxi data, which of the following code will return an error?
      + A. `rxLinMod(tip_percent ~ trip_distance, data = input_xdf)`
      + B. `rxLinMod(tip_percent ~ trip_distance+payment_type, data = input_xdf)`
      + C. `rxLinMod(tip_percent ~ ., data = input_xdf)`
      + D. `rxLinMod(tip_percent ~ trip_distance:payment_type, data = input_xdf)`

## Answers
  1. B
  2. ABC
  3. C
  4. B
  5. D
  6. ACD
  7. ABC
  8. B?
  9. B
  10. C?

## Modeling Lab <a name="modeling-lab"></a>

```
For this lab, please use the mht_lab2.xdf provided.

Assuming you followed the course thus far, you will now have an XDF file similar to the mht_lab2.xdf provided. Just in case you're wondering, it is the same file as the one used in the previous lab. This contains the NYC taxi data that was downsized for the course. The XDF file has been enriched with several features such as tip_percent, pickup_hour, pickup_dow, and payment_type_desc to name a few, and has been filtered down to focus only on the Manhattan neighborhood. However, to answer the lab questions, make sure you use the mht_lab2.xdf, and not the XDF from the lecture.

Let's build a linear model for predicting tip_percent using trip_duration and the interaction of pickup_dow and pickup_hour using the whole data from the XDF provided.

input_xdf <- 'mht_lab2.xdf'
mht_xdf <- RxXdfData(input_xdf)

# 1. What is the adjusted R - squared of the model ?

rxlm <- rxLinMod(tip_percent ~ trip_duration + pickup_dow:pickup_hour, mht_xdf)
names(rxlm)

rxlm$adj.r.squared

# 2. Now, let's include payment_type_desc as a predictor in the model and build the model. What is the new adjusted R - squared of the model ?

rxlm2 <- rxLinMod(tip_percent ~ trip_duration + pickup_dow:pickup_hour + payment_type_desc, mht_xdf)
rxlm2$adj.r.squared

# You can see that the adjusted R-squared increased quite a bit from the previous model. As we've explored in previous module, almost all cash payment shows zero tip percent in the data, making payment_type_desc an important feature in our data.  Now, use rxPredict to put the predictions made by both models into the data as new columns called tip_pred_1 and tip_pred_2. Then use rxHistogram to show predictions between 0 to 50% and plot the predictions. These are several potential results of the histograms.
# 3. Which of the graph represent the histogram of tip_percent?

rxPredict(rxlm, data = mht_xdf, predVarNames = "tip_pred_1")
rxPredict(rxlm2, data = mht_xdf, predVarNames = "tip_pred_2")

rxHistogram(~tip_percent, mht_xdf)

# 4. Which of the graph represent the histogram of tip_pred_1?

rxHistogram(~tip_pred_1, mht_xdf)

# 5. Which of the graph represent the histogram of tip_pred_2?

rxHistogram(~tip_pred_2, mht_xdf, numBreaks = 50)
```

# Chapter 5 - Deploying and Scaling <a name="chapter-5"></a>
  - Change the compute context (**rxHadoopMR**, **rxSpark**, **rxLocalseq**, and **rxLocalParallel**)
  - Identify which compute context to use for different tasks
  - Use different data source objects, depending on the context (**RxOdbcData** and **RxTextData**)
  - Identify and use appropriate data sources for different data sources and compute contexts (HDFS and SQL Server)
  - Debug processes across different compute contexts
  - Identify use cases for **RevoPemaR**
  - Identify and execute tasks that can be run only in the local compute context
  - Identify tasks that are more efficient to run in the local compute context
  - Choose between **rxLocalseq** and **rxLocalParallel**
  - Profile across different compute contexts
  - Choose when to perform in-database versus out-of-database computations
  - Identify limitations of in-database computations
  - Use in-database versus out-of-database compute contexts appropriately
  - Use stored procedures for data processing steps
  - Serialize objects and write back to binary fields in a table
  - Write tables
  - Configure R to optimize SQL Server (**chunksize**, **numtasks**, and **computecontext**)
  - Effectively communicate performance properties to SQL administrators and architects (SQL Server Profiler)  
  - Use appropriate R Server functions in Spark; integrate with Hive, Pig, and Hadoop MapReduce
  - Integrate with the Spark ecosystem of tools, such as SparklyR and SparkR
  - Profile and tune across different compute contexts
  - Use **doRSR** for parallelizing code that was written using open source **foreach**
  - Deploy predictive models to SQL Server as a stored procedure
  - Deploy an arbitrary function to Azure Machine Learning by using the AzureML R package
  - Identify when to use DeployR  
  
**Create pointer to SQL table**

```
sqlConnString <- "Driver=SQL Server;Server=SETHMOTTDSVM;Database=RDB;Uid=ruser;Pwd=ruser"
sqlRowsPerRead <- 100000
sqlTable <- "sampledata"
ccColInfo <- list(
  name = list(type = "character"),
  sales = list(type = "numeric")
)

sample_data_sql <- RxSqlServerData(connectionString = sqlConnString, rowsPerRead = sqlRowsPerRead, table = sqlTable, colInfo = ccColInfo)
```
  
`fileSystem` argument points to an object created in  `RxRxHdfsFileSystem(hostName, port)`

**Manage Compute Context**

`RxComputeContext` - Create a compute context.
`rxInSqlServer` - Generate a SQL Server compute context that lets ScaleR functions run in SQL Server R Services. This compute context is currently supported only for SQL Server instances on Windows.
`rxGetComputeContext` - Get the current compute context.
`rxSetComputeContext` - Specify which compute context to use.
`RxSpark` - point to Spark cluster
`rxExec`


## Quiz 5 <a name="quiz5"></a>

  1. Which argument would you use to indicate the type of file system used when defining RxXdfData object?
      + A. file
      + B. fileSystem
      + C. RxHdfsFileSystem
      + D. RxNativeFileSystem
      
  2. You have the following code:

```
    sqlConnString <- "Driver=SQL Server;Server=SETHMOTTDSVM;Database=RDB;Uid=ruser;Pwd=ruser" 
    sqlRowsPerRead <- 100000 
    sqlTable <- "NYCTaxiBig" 
    nyc_sql <- RxSqlServerData(connectionString = sqlConnString, rowsPerRead = sqlRowsPerRead, table = sqlTable) 
```

  Which of the following code would you use to define a compute context that points to the SQL server? 
      + A. `sqlCC <- RxSqlServerData(connectionString = sqlConnString)`
      + B. `sqlCC <- RxInSqlServer(connectionString = sqlConnString)`
      + C. `sqlCC <- RxLocalSeq(connectionString = sqlConnString)`
      + D. `sqlCC <- rxExecuteSQLDDL(connectionString = sqlConnString)`
      
  3. Which three of the following code define a compute context that can be used with rxSetComputeContext?
      + A. `RxXdfData()`
      + B. `RxSpark()`
      + C. `RxHadoopMR()`
      + D. `RxInSqlServer()`
      
  4. Which three functions will only work with XDF files on local system, and not with data residing in SQL server?
      + A. `rxMerge`
      + B. `rxSummary`
      + C. `rxSort`
      + D. `rxSplit`
      
  5. What three typical scenarios you would do when analyzing data residing in SQL server using R?
      + A. Define connection to the SQL Server
      + B. Connect to the SQL server and query the database
      + C. Perform analysis directly in the SQL Server
      + D. Bring the data out and perform analysis

## Answers
  1. B
  2. B
  3. BCD
  4. 
  5.
  
## Final Exam <a name="exam"></a>

  1. Consider the ChickWeight dataset, which is part of Base R datasets. Which of the following code using `rxHistogram()` will produce similar result to `hist(subset(ChickWeight,Time==21)$weight,freq=TRUE,breaks=10)`?
      + A. `rxHistogram(~weight, ChickWeight, rowSelection = (Time==21), histType = "Percent", numBreaks = 10)`
      + B. `rxHistogram(~weight, ChickWeight, rowSelection = (Time==21), numBreaks = 10)`
      + C. `rxHistogram(~weight, ChickWeight, numBreaks = 10)`
      + D. `rxHistogram(~weight, ChickWeight)`
      
  2. You have the following code:
```
    file_1 <- 'data.csv'
```    
    You are working on a local compute context and you've already set the working directory, specified colClasses, and imported the necessary libraries. You want to import data.csv to a data.frame called data_df using rxImport() function. Which of the following code should you use?
      + A. `data_df <- rxImport(file_1, colClasses = col_classes)`
      + B. `file_1 <- rxImport(data_df, colClasses = col_classes)`
      + C. `rxImport(file_1, data_df, colClasses = col_classes)`
      + D. `rxImport(data_df, file_1, colClasses = col_classes)`
      
  3. Consider the cars dataset, which is part of Base R datasets. Which of the following code using `rxLinePlot()` will produce similar result to `plot(cars)`?
      + A. `rxLinePlot(cars)`
      + B. `rxLinePlot(dist~speed,cars,type="p")`
      + C. `rxLinePlot(dist~speed,cars)`
      + D. `rxLinePlot(dist~speed,cars,type="r")`

  4. You have the following code:
```
    file_1 <- 'data.xdf' 
    file_2 <- 'data.csv'
```    
    You are working on a local compute context and you've already set the working directory, specified colClasses, and imported the necessary libraries. You want to import data.csv to data.xdf using rxImport() function. Which of the following code should you use?
      + A. `file_2 <- rxImport(file_1, colClasses = col_classes, overwrite = TRUE)`
      + B. `rxImport(file_1, file_2, colClasses = col_classes, overwrite = TRUE)`
      + C. `file_1 <- rxImport(file_2, colClasses = col_classes, overwrite = TRUE)`
      + D. `rxImport(file_2, file_1, colClasses = col_classes, overwrite = TRUE)`

  5. You have the following code:
```
    file_1 <- 'data.csv' 
    data_csv <- RxTextData(file_1, colClasses = col_classes)
```
    You are working on a local compute context and you've already set the working directory, specified colClasses, and imported the necessary libraries. You want to show statistical summaries for a column called 'amount' from the data.csv file using rxSummary() function. Which three of the following code could you use?
      + A. `rxSummary( ~ amount, 'data.csv')`
      + B. `rxSummary( ~ amount, file_1)`
      + C. 'rxSummary( ~ amount, data_csv)`
      + D. `rxSummary( ~ amount, data.csv)`

  6. Consider the airquality dataset, which is part of Base R datasets. Assume that you have the exact same data on an XDF file named airqualiy.xdf.  Which of the following code will produce similar values for Temp variable to rxCube(Temp ~ Day:Month, "airquality.xdf")?
      + A. `rxCrossTabs(Temp ~ Day:Month, "airquality.xdf", means = TRUE)`
      + B. `rxCrossTabs(Temp ~ Day:Month, "airquality.xdf")`
      + C. `rxCrossTabs(Temp ~ Month:Day, "airquality.xdf")`
      + D. `rxCrossTabs(Temp ~ Day:Month, "airquality.xdf", means = FALSE)`

  7. Consider the airquality dataset, which is part of Base R datasets. Assume that you have the exact same data on an XDF file named airqualiy.xdf. Which of the following code using rxHistogram() will produce similar result to hist(airquality$Temp, freq = TRUE, breaks = 10)?

      + A. `rxHistogram( ~ Temp, "airquality.xdf", breaks = 10)`
      + B. `rxHistogram( ~ Temp, "airquality.xdf", histType = "Percent", numBreaks = 10)`
      + C. `rxHistogram( ~ Temp, "airquality.xdf", histType = "Percent", breaks = 10)`
      + D. `rxHistogram( ~ Temp, "airquality.xdf", numBreaks = 10)`
      
  8. Consider the airquality dataset, which is part of Base R datasets. Assume that you have the exact same data on an XDF file named airqualiy.xdf and the following code below:
```
    newlevs <- c("1973","1974","1975") 
    rxFactors("airquality.xdf", outFile = "airquality.xdf", 
          factorInfo = list(Year = list(newLevels = unique(newlevs))))
```    
    Which of the following code will produce similar results?
```
# A
rxDataStep("airquality.xdf", outFile = "airquality.xdf", 
           transforms = list(Year = list(newLevels = unique(newlevs))), 
           overwrite = TRUE)
# B
rxDataStep("airquality.xdf", outFile = "airquality.xdf", 
           transforms = list(Year = factor(Year, levels = newlevs)), 
           overwrite = TRUE)
# C
rxDataStep("airquality.xdf", outFile = "airquality.xdf", 
           transforms = list(Year = factor(Year, levels = newlevels)), 
           transformObjects = list(newlevels = unique(newlevs)), 
           overwrite = TRUE)
# D
rxDataStep("airquality.xdf", outFile = "airquality.xdf", 
           transformObjects = list(Year = list(newLevels = unique(newlevs))), 
           overwrite = TRUE)
```

  9. Consider the airquality dataset, which is part of Base R datasets. You have the following code:
```
    z <- airquality[,c("Wind","Temp")] 
    cl <- kmeans(z, 2) 
    cl$cluster
```    
    Which of the following code will produce similar result using the rxKmeans() function?
      + A. `cl <- rxKmeans(~Wind+Temp,data=z,centers=2)`
      + B. `cl <- rxKmeans(~Wind+Temp,data=z,reportProgress=2)`
      + C. `cl <- rxKmeans(~Wind+Temp,data=z,numClusters=2)`
      + D. `cl <- rxKmeans(~Wind+Temp,data=z,maxIterations=2)`
      
  10. You have the following code:
```
    file_1 <- 'data.csv' 
    file_2 <- 'data.xdf' 
    rxImport(file_1, file_2, colClasses = col_classes, overwrite = TRUE)
```

You are working on a local compute context and you've already set the working directory, specified colClasses, and imported the necessary libraries.  The data.xdf file contains a numeric column named amount. You want to modify the data.xdf file by adding a new column named HiLo. The value should be "High" if amount > 10 and "Low" otherwise. Which two of the following code could you use?

```
# A
rxDataStep(file_2, file_2, 
           transforms = list(HiLo = ifelse(amount > 10, "High", "Low")), 
           overwrite = TRUE)
# B
rxDataStep(file_2, file_2, 
           transformFunc = list(HiLo = ifelse(amount > 10, "High", "Low")), 
           overwrite = TRUE)
# C
rxDataStep(file_1, file_2, 
           transformFunc = list(HiLo = ifelse(amount > 10, "High", "Low")), 
           overwrite = TRUE)
# D
rxDataStep(file_1, file_2, 
           transforms = list(HiLo = ifelse(amount > 10, "High", "Low")), 
           overwrite = TRUE)
```

  11. What is the main advantage of an XDF file over a data.frame?
      + A. You can use open source R functions directly with an XDF file
      + B. An XDF file can contain data larger than the local computer's memory sizeC.
      + C. Computation using an XDF file is faster than using data.frame
      + D. An XDF file is compressed such that it fits to the local computer's memory size

  12. Consider the father.son dataset from the UsingR package. You can create a linear regression model fitting the son's height (sheight) by the father's height (fheight) with the following code `regfit<-lm(sheight ~ fheight, data=father.son)`
    Which of the following code return similar result when using the rxLinMod() function?
      + A. `rxLinMod(sheight ~ fheight, data="father.son.xdf")`
      + B. `rxLinMod(sheight ~ fheight, data=father.son.xdf)`
      + C. `rxLinMod(sheight ~ fheight, data=RxXdfData(father.son))`
      + D. `rxLinMod(sheight ~ fheight, data=father.son)`

  13. Which three statement explains how RevoScaleR mitigates R limitation working with data that is larger than the local computer's memory?
      + A. RevoScaleR points to the data that sits on the disk
      + B. RevoScaleR loads the data into memory a chunk at a time
      + C. RevoScaleR waits until all the data is loaded into memory
      + D. RevoScaleR processes the loaded data one chunk at a time

  14. With the reference to the New York Taxi data, which of the following code creates a linear regression model fitting the tip_percent by the interaction of trip_distance and payment_type?
      + A. `rxLinMod(tip_percent ~ trip_distance, data = input_xdf)`
      + B. `rxLinMod(tip_percent ~ trip_distance+payment_type, data = input_xdf)`
      + C. `rxLinMod(tip_percent ~ trip_distance:payment_type, data = input_xdf)`
      + D. `rxLinMod(tip_percent ~ ., data = input_xdf)`
      
  15. Which code would you use to set RevoScaleR options to report no progress?
      + A. `rxOptions(reportProgress = 0)`
      + B. `rxOptions(reportProgress = 1)`
      + C. `rxOptions(reportProgress = 2)`
      + D. `rxOptions(reportProgress = 3)`
      
  16. You have the following code:
```
    sqlConnString <- "Driver=SQL Server;Server=SETHMOTTDSVM;Database=RDB;Uid=ruser;Pwd=ruser" 
    sqlRowsPerRead <- 100000 
    sqlTable <- "NYCTaxiBig" 
    nyc_sql <- RxSqlServerData(connectionString = sqlConnString, rowsPerRead = sqlRowsPerRead, table = sqlTable)
```    
    Which of the following code would you use to define a compute context that points to the SQL server?
      + A. `sqlCC <- RxSqlServerData(connectionString = sqlConnString)`
      + B. `sqlCC <- RxLocalSeq(connectionString = sqlConnString)`
      + C. `sqlCC <- RxInSqlServer(connectionString = sqlConnString)`
      + D. `sqlCC <- rxExecuteSQLDDL(connectionString = sqlConnString)`
      
  17. Consider the airquality dataset, which is part of Base R datasets. Assume that you have the exact same data on an XDF file named airqualiy.xdf. Which two of the following code will display the means of the Temp variable with the Day as the rows and the Month as the columns?

```
# A
rxct <- rxCrossTabs(Temp ~ Day:Month, "airquality.xdf", means = TRUE) 
print(rxct)

# B
rxct <- rxCrossTabs(Temp ~ Day:Month, "airquality.xdf") 
print(rxct)

# C
rxct <- rxCrossTabs(Temp ~ Month:Day, "airquality.xdf", means = TRUE) 
print(rxct)

# D
rxct <- rxCrossTabs(Temp ~ Day:Month, "airquality.xdf") 
print(rxct, output = "means")
```

  18. Consider the airquality dataset, which is part of Base R datasets. Assume that you have the exact same data on an XDF file named airqualiy.xdf and the following code below:
```
    newlevs <- c("1973","1974","1975") 
    rxDataStep("airquality.xdf", outFile = "airquality.xdf", 
               transforms = list(Year = factor(Year, levels = newlevels)), 
               transformObjects = list(newlevels = unique(newlevs)), 
               overwrite = TRUE)
```    
    Which of the following code will produce similar results?

```
# A
rxFactors("airquality.xdf", outFile = "airquality.xdf", 
          transforms = list(Year = factor(Year, levels = newlevels)), 
          transformObjects = list(newlevels = unique(newlevs)), 
          overwrite = TRUE)
# B
rxFactors("airquality.xdf", outFile = "airquality.xdf", 
          factorInfo = list(Year = factor(Year, levels = newlevs)))
# C
rxFactors("airquality.xdf", outFile = "airquality.xdf", 
          factorInfo = list(Year = list(newLevels = unique(newlevs))))
# D
rxFactors("airquality.xdf", outFile = "airquality.xdf", 
          transforms = list(Year = list(newLevels = unique(newlevs))), 
          overwrite = TRUE)
```

  19. You have the following code:
```
    athreshold <- 10 
    file_1 <- 'data.csv' 
    file_2 <- 'data.xdf' 
    rxImport(file_1, file_2, colClasses = col_classes, overwrite = TRUE) 
    xforms <- function(data) {  
      data$HiLo <- ifelse(data$amount > threshold, "High", "Low") 
      data 
    } 
```    
    You are working on a local compute context and you've already set the working directory, specified colClasses, and imported the necessary libraries. The data.xdf file contains a numeric column named amount. You want to modify the data.xdf file by adding a new column named HiLo. The value should be "High" if amount > 10 and "Low" otherwise. Which of the following code could you use?

```
# A
rxDataStep(file_2, file_2, 
           transformFunc = xforms, 
           transformObjects = list(threshold = athreshold))
# B
rxDataStep(file_2, file_2, 
           transformFunc = xforms, 
           overwrite = TRUE)
# C
rxDataStep(file_2, file_2, 
           transformFunc = xforms, 
           transformObjects = list(threshold = athreshold), 
           overwrite = TRUE)
# D
rxDataStep(file_2, file_2, 
           transforms = xforms, 
           overwrite = TRUE)
```
  20. Consider the airquality dataset, which is part of Base R datasets. Assume that you have the exact same data on an XDF file named airqualiy.xdf. Which of the following code will return a data.frame that has temperature above 75?

```
# A
rxDataStep("airquality.xdf",  
           rowSelection = (HiLo == "High"), 
           transforms = list(HiLo = ifelse(Temp>75,"High","Low")))
# B
rxDataStep("airquality.xdf", 
           transforms = list(HiLo = ifelse(Temp>75,"High","Low")))
# C
rxDataStep("airquality.xdf", 
           rowSelection = (Temp > 75 & Temp <= 75))
# D
rxDataStep("airquality.xdf",   
           transformFunc = list(HiLo = ifelse(Temp>75,"High","Low")))
```

  21. Which two of the following code would you use to change the active compute context to the local computer?
      + A. `rxSetComputeContext("local")`
      + B. `rxSetComputeContext(local)`
      + C. `rxSetComputeContext("localpar")`
      + D. `rxSetComputeContext(localpar)`
      
  22. Consider the father.son dataset from the UsingR package. You have the following code `cor(father.son)`
    
    Which of the following code return similar result when using the rxCor() function?
      + A. `rxCor(father.son)`
      + B. `rxCor(sheight~fheight,father.son)`
      + C. `rxCor(RxXdfData(father.son))`
      + D. `rxCor(~sheight+fheight,father.son)`

  23. Consider the rxSummary() function. Which notation should you use for the formula argument to summarize the column trip_duration?
      + A. `= trip_duration`
      + B. `~ trip_duration`
      + C. `+ trip_duration`
      + D. `? trip_duration`
      
  24. Assume that you are creating a linear regression model by using the following code:
```
    model1 <- rxLinMod(tip_percent~trip_distance, data=input_xdf)
```    
    Which of the following code resembles the approach but using a simple regression tree algorithm?
      + A. `model2 <- rxDTree(tip_percent~trip_distance, data=input_xdf)`
      + B. `model2 <- rxDTree(tip_percent~trip_distance, data=input_xdf,ntree=10)`
      + C. `model2 <- rxLogit(tip_percent~trip_distance, data=input_xdf)`
      + D. `model2 <- rxDForest(tip_percent~trip_distance, data=input_xdf)`
      
  25. Which three steps are part of the preparation phase in the advanced analytics lifecycle?
      + A. Ingest
      + B. Transform
      + C. Model
      + D. Explore

  26. Consider the rxDataStep() function. Which three of the following assignments are valid for the transformPackages argument?
      + A. `transformPackages = c("stringr","lubridate")`
      + B. `transformPackages = "lubridate"`
      + C. `transformPackages = c("stringr")`
      + D. `transformPackages = list("stringr", "lubridate")`

  27. Consider the father.son dataset from the UsingR package. You have split the dataset into training and test dataset named fstrain and fstest respectively. You can train a linear regression model using the training dataset fitting the son's height by the father's height with the following code: 
```

    regfit<-lm(sheight ~ fheight, data=fstrain)
```    
    You can obtain the fitted values of the model using the following code:
```

    regfit$fitted.values
```    
Which of the following code return similar result when using the rxLinMod() function?

```
# A
regfit2 <- rxLinMod(sheight ~ fheight, data=fstrain) 
regfit2$fitted.values

# B
regfit2 <- rxLinMod(sheight ~ fheight, data=fstrain) 
rxPredict(regfit2, data=fstest)

# C
regfit2 <- rxLinMod(sheight ~ fheight, data=fstrain) 
rxPredict(regfit2, data=fstrain, computeResiduals = TRUE)

# D
regfit2 <- rxLinMod(sheight ~ fheight, data=fstrain) 
rxPredict(regfit2, data=fstrain)

# E
regfit2 <- rxLinMod(sheight ~ fheight, data=fstrain) 
rxPredict(regfit2, data=fstest, computeResiduals = TRUE)E)
```
  28. Consider the rxImport() function. Which argument performs similar function to the argument nrows in the read.csv() function?
      + A. varsToKeep
      + B. numRows
      + C. varsToDrop
      + D. rowSelection
      
## Answers

  1. ABC
  2. B
  3. B
  4. B
  5. D
  6. ABC
  7. A
  8. D
  9. C
  10. C
  11. AD
  12. B
  13. D
  14. ABD
  15. C
  16. A
  17. C
  18. AD
  19. C
  20. C
  21. A
  22. AC
  23. D
  24. B
  25. A
  26. ABD
  27. ABC
  28. D
      
## General Resources <a name="general-resources"></a>

* [Learn Analytics @ Microsoft](http://learnanalytics.microsoft.com)
* [Analyzing Big Data with R Server](https://courses.edx.org/courses/course-v1:Microsoft+DAT213x+2T2017/info)
* [Big Data Analytics with Revolution R Enterprise](https://campus.datacamp.com/courses/big-data-revolution-r-enterprise-tutorial)
* [Microsoft R Documentation](https://msdn.microsoft.com/en-us/microsoft-r/index)
      


