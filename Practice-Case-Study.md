```r
# Iris dataset

# Use the Iris dataset to answer various questions using RevoScaleR functions.

#############################################################################
# QUESTIONS

# 1. Set reportProgress option.
#       a. Check to see what the current reportProgress value is.
#       b. Change the reportProgress value to 0.

# 2. How many variables are there in this dataset?  

# 3. What are the types and names of the variables in the data set?

# 4. What is the max value of Sepal.Length?

# 5. How many observations are there in the dataset?

# 6. What are the general measures of the petals in the dataset?

# 7. Are there any variables with missing values?

# 8. How many of each species are present in the dataset?

# 9. Create two histograms.
#       a. Petal.Length
#       b. Petal.Length by Species

# 10. Create two line plots.
#       a. Petal.Length vs. Petal.Width
#       b. Petal.Length vs. Petal.Width by Species

# 11. Use the code below to create a new variable called H2O.Consumption, then create a table of the total H2O Consumption by species.

#        set.seed(42)
#        mns <- c(5, 10, 20)
#        sds <- 1:3
#        specNum <- as.numeric(iris$Species)
#        iris$H2O.Consumption <- rnorm(nrow(iris), mns[specNum], sds[specNum])

# 12. Display the mean H2O.Consumption using rxCube AND rxCrossTabs.

# 13. Create a variable called High.H2O that is TRUE for observations with an H2O.Consumption >= 7.

# 14. Write a function to create three new variables.  Two called Petal.Ratio and Sepal.Ratio, by dividing length by width, and a third called H2O_Z (standardized H2O).

# 15. Perform a linear regression (using rxLinMod) to predict H2O.Consumption, assign the model to irisLM.
#       a. What are the values of r-squared and adjusted r-squared for this model?
#       b. Look at the p-values of the variables (HINT: names(irisLM) can tell you how to extract specific objects). Which variables have p-values greater than .05?
#       c. Run the regression again without the variables identified above and assign the object to irisLM2. What are the values of r-squared and adjusted r-squared for this model?
#       d. Use rxPredict to get predicted values for H2O.Consumption.
#       e. Create a histogram of the residuals.

# 16. Perform a logistic regression (using rxLogit) to classify species (do not use any of the H2O.Consumption variables).  Store the object as irisGLM.
#       NOTE: rxLogit requires a binary response, and the iris data contains three species.
#           Use the code below to limit the data to two categories:
#               rxDataStep(inData = iris_xdf, outFile = iris_xdf,
#                   transforms = list(setosa = ifelse(Species == 'setosa', 1, 0)),
#                   overwrite = TRUE
#                )
#       a. 
#

#############################################################################
# ANSWERS

# Check reportProgress value
paste("Answer 1a. The current value of reportProgress is ", rxGetOption('reportProgress'), ".")

# Change the value to 0.
rxOptions(reportProgress = 0)
paste("Answer 1b. The new value of reportProgress is ", rxGetOption('reportProgress'), ".")


# Get information about dataset
# - Variable Names
# - Variable Types
# - Min/Max for Numeric, or Levels for Factor
iris_info <- rxGetVarInfo(iris)

# This does the same as above, but also provides:
# - Number of Observations
# - Number of Variables
iris_info2 <- rxGetInfo(iris, getVarInfo = TRUE)

paste("Answer 2. There are ", iris_info2$numVars, "variables in this data set.")
paste("Answer 3. See below.")
names(iris_info)
iris_info
paste("Answer 4. The max value of Sepal.Length is ", iris_info$Sepal.Length$high, ".")
paste("Answer 5. There are ", iris_info2$numRows, "observations in the dataset named", iris_info2$objName, ".")

# Get summary statistics for dataset
# - Minimum, Maximum, Mean, Standard Deviation, Valid/Missing Observations
# - Can reference different parts of the object 
petal_stats <- rxSummary(~ Petal.Length + Petal.Width, iris)
paste("Answer 6. See below.")
petal_stats$sDataFrame
iris_stats <- rxSummary(~., iris)
paste("Answer 7. Of the ", iris_info2$numRows, "observations, ", iris_stats$nobs.valid, " are valid.  There are ", iris_info2$numRows - iris_stats$nobs.valid, "missing observations.")

# Frequency count
iris_freq <- rxCrossTabs(~Species, iris)
paste("Answer 8. See below.")
iris_freq$counts

# Display frequency distributions
rxHistogram(~Petal.Length, iris, title = 'Answer 9a. Histogram of Petal.Length')
rxHistogram(~Petal.Length | Species, iris, title = 'Answer 9b. Histogram of Petal.Length by Species')

# Explore bivariate relationship
rxLinePlot(Petal.Length ~ Petal.Width, iris, type = "p", title = 'Answer 10a. Lineplot of Petal.Length vs. Petal.Width')
rxLinePlot(Petal.Length ~ Petal.Width | Species, iris, type = "p", title = 'Answer 10b. Lineplot of Petal.Length vs. Petal.Width by Species')
rxLinePlot(Petal.Length ~ Petal.Width, groups = Species, iris, type = "p", title = 'Answer 10b. Lineplot of Petal.Length vs. Petal.Width by Species (same plot)')

# Create H2O.Consumption variable
set.seed(42)
mns <- c(5, 10, 20)
sds <- 1:3
specNum <- as.numeric(iris$Species)
iris$H2O.Consumption <- rnorm(nrow(iris), mns[specNum], sds[specNum])

# Calculate totals by group
iris_total_H2O <- rxCrossTabs(H2O.Consumption ~ Species, iris)
paste("Answer 11. See below.")
iris_total_H2O$sums

# Calculate means by group
paste("Answer 12. See below.")
rxCrossTabs(H2O.Consumption ~ Species, iris, means = TRUE)
rxCube(H2O.Consumption ~ Species, iris)

getwd()
iris_xdf <- 'iris.xdf'

# Create new variable with transformation.
paste('Answer 13. See below.')
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

paste("Answer 14. See below")
ratiofunction <- function(data) {
    data$Sepal.Ratio <- data$Sepal.Length / data$Sepal.Width
    data$Petal.Ratio <- data$Petal.Length / data$Petal.Width
    data$H2O_Z <- (data$H2O.Consumption - mymean) / mystd
    return(data)
}
rxDataStep(iris_xdf, iris_xdf, overwrite = TRUE,
    transformFunc = ratiofunction,
    transformObjects = list(mymean = meanH2O, mystd = sdH2O))

irisLM <- rxLinMod(H2O.Consumption ~
                    Sepal.Length +
                    Sepal.Width +
                    Sepal.Ratio +
                    Petal.Length +
                    Petal.Width +
                    Petal.Ratio, data = iris_xdf)

paste("Answer 15a. The r-squared for this model is ", irisLM$r.squared, ". The adjusted r-squared is ", irisLM$adj.r.squared, ".")

paste("Answer 15b. See below")
irisLM$coef.p.value

irisLM2 <- rxLinMod(H2O.Consumption ~
                    Sepal.Length +
                    #Sepal.Width +
                    #Sepal.Ratio +
                    Petal.Length +
                    Petal.Width +
                    Petal.Ratio, data = iris_xdf)

paste("Answer 15c. The r-squared for this model is ", irisLM2$r.squared, ". The adjusted r-squared is ", irisLM2$adj.r.squared, ".")

rxPredict(irisLM2, iris_xdf, iris_xdf, writeModelVars = TRUE, computeResiduals = TRUE, overwrite = TRUE)

rxHistogram(~H2O.Consumption_Resid, iris_xdf, title = 'Answer 15e. Histogram of H2O.Consumption Residuals')

#rxGetInfo(iris_xdf, getVarInfo = TRUE)
#rxGetInfo(iris_xdf, numRows = 150)

rxDataStep(inData = iris_xdf, outFile = iris_xdf,
    transforms = list(setosa = ifelse(Species == 'setosa', 1, 0)),
    overwrite = TRUE
)

irisGLM <- rxLogit(setosa ~
                    Sepal.Length +
                    Sepal.Width +
                    Sepal.Ratio +
                    Petal.Length +
                    Petal.Width +
                    Petal.Ratio, data = iris_xdf)

summary(irisGLM)
names(irisGLM)

rxPredict(modelObject = irisGLM, data = iris_xdf, outData = iris_xdf, type = "response")



KMout <- rxKmeans(~ Petal.Length + Petal.Width + Sepal.Ratio,
         data = iris_xdf,
         outFile = iris_xdf,
         numClusters = 3,
         writeModelVars = TRUE)
print(KMout)

regTreeOut <- rxDTree(Species ~ Sepal.Width + Sepal.Length + Petal.Width + Petal.Length,
                      data = iris_xdf,
                      maxdepth = 5)
## print out the object:
print(regTreeOut)

rxPredict(regTreeOut,
          data = iris_xdf,
          outData = iris_xdf,
          writeModelVars = TRUE,
          predVarNames = "default_RegPred")
