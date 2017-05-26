# Microsoft-70-773-ExamPrep
Study Guide for Microsoft 70-773

## Table of Contents

- [General Resources](#general-resources)
- [Exam Description](#exam-description)
- [Glossary](#glossary)

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
