```r

# Iris dataset

# Use the Iris dataset to answer various questions using RevoScaleR functions.

#############################################################################
# QUESTIONS

# 1. How many variables are there in this dataset?  

# 2. What are the types and names of the variables in the data set?

# 3. What is the max value of Sepal.Length?

# 4. How many observations are there in the dataset?

# 5. What are the general measures of the petals in the dataset?

# 6. Are there any variables with missing values?

# 7. How many of each species are present in the dataset?

# 8. Create two histograms.
#       a. Petal.Length
#       b. Petal.Length by Species

# 9. Create two line plots.
#       a. Petal.Length vs. Petal.Width
#       b. Petal.Length vs. Petal.Width by Species

# 10. Use the code below to create a new variable called H2O.Consumption, then create a table of the total H2O Consumption by species.

#        set.seed(42)
#        iris$H2O.Consumption <- rnorm(150, 5, 1)

# 11. Display the mean H2O.Consumption using rxCube AND rxCrossTabs.

# 12. Create a variable called High.H2O that is TRUE for observations with an H2O.Consumption >= 7.

# 13. Write a function to create three new variables.  Two called Petal.Ratio and Sepal.Ratio, by dividing length by width, and a third called H2O_Z (standardized H2O).

# 14. 
#############################################################################
# ANSWERS

# Get information about dataset
# - Variable Names
# - Variable Types
# - Min/Max for Numeric, or Levels for Factor
iris_info <- rxGetVarInfo(iris)

# This does the same as above, but also provides:
# - Number of Observations
# - Number of Variables
iris_info2 <- rxGetInfo(iris, getVarInfo = TRUE)

paste("Answer 1. There are ", iris_info2$numVars, "variables in this data set.")
paste("Answer 2. See below.")
names(iris_info)
iris_info
paste("Answer 3. The max value of Sepal.Length is ", iris_info$Sepal.Length$high, ".")
paste("Answer 4. There are ", iris_info2$numRows, "observations in the dataset named", iris_info2$objName, ".")

# Get summary statistics for dataset
# - Minimum, Maximum, Mean, Standard Deviation, Valid/Missing Observations
# - Can reference different parts of the object 
petal_stats <- rxSummary(~ Petal.Length + Petal.Width, iris)
paste("Answer 5. See below.")
petal_stats$sDataFrame
iris_stats <- rxSummary(~., iris)
paste("Answer 6. Of the ", iris_info2$numRows, "observations, ", iris_stats$nobs.valid, " are valid.  There are ", iris_info2$numRows - iris_stats$nobs.valid, "missing observations.")

# Frequency count
iris_freq <- rxCrossTabs(~Species, iris)
paste("Answer 7. See below.")
iris_freq$counts

# Display frequency distributions
rxHistogram(~Petal.Length, iris, title = 'Answer 8a. Histogram of Petal.Length')
rxHistogram(~Petal.Length | Species, iris, title = 'Answer 8b. Histogram of Petal.Length by Species')

# Explore bivariate relationship
rxLinePlot(Petal.Length ~ Petal.Width, iris, type = "p", title = 'Answer 9a. Lineplot of Petal.Length vs. Petal.Width')
rxLinePlot(Petal.Length ~ Petal.Width | Species, iris, type = "p", title = 'Answer 9b. Lineplot of Petal.Length vs. Petal.Width by Species')
rxLinePlot(Petal.Length ~ Petal.Width, groups = Species, iris, type = "p", title = 'Answer 9b. Lineplot of Petal.Length vs. Petal.Width by Species (same plot)')

# Calculate totals by group
iris_total_H2O <- rxCrossTabs(H2O.Consumption ~ Species, iris)
paste("Answer 10. See below.")
iris_total_H2O$sums

# Calculate means by group
paste("Answer 11. See below.")
rxCrossTabs(H2O.Consumption ~ Species, iris, means = TRUE)
rxCube(H2O.Consumption ~ Species, iris)

getwd()
iris_xdf <- 'iris.xdf'

# Create new variable with transformation.
paste('Answer 12. See below.')
rxDataStep(inData = iris, outFile = iris_xdf,
           transforms = list(High.H2O = H2O.Consumption > 7),
           overwrite = TRUE
  )
head(RxXdfData(iris_xdf))

# Create new variable with transformation function
# - First need to find mean and stdev of H2O.Consumption
H2OSummary <- rxSummary(~H2O.Consumption, data = iris_xdf)
meanH2O <- H2OSummary$sDataFrame$Mean[1]
sdH2O <- H2OSummary$sDataFrame$StdDev[1]

paste("Answer 13. See below")
ratiofunction <- function(data) {
    data$Sepal.Ratio <- data$Sepal.Length / data$Sepal.WidthMean
    data$Petal.Ratio <- data$Petal.Length / data$Petal.Width
    data$H2O_Z <- (data$H2O.Consumption - mymean) / mystd
    return(data)
}
rxDataStep(iris_xdf, iris_xdf, overwrite = TRUE,
    transformFunc = ratiofunction,
    transformObjects = list(mymean = meanH2O, mystd = sdH2O))

```
