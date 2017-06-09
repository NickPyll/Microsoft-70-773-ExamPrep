```r

# List Available data
data()

# rxGetVarInfo
rxGetVarInfo(iris)

# rxGetInfo
rxGetInfo(iris, getVarInfo = TRUE)

# rxSummary
rxSummary(~., iris)
rxs <- rxSummary(~., iris)
names(rxs)
rxs$sDataFrame
rxs_df <- rxs$sDataFrame
names(rxSummary(~., iris))
rxSummary(~ Reliability:Country, car90)

# rxCrossTabs
rxGetInfo(car90, getVarInfo = TRUE)
rxCrossTabs(~ Reliability, car90)
rxCrossTabs(~ Reliability:Country, car90)
rxCrossTabs(~ Reliability:Country:Type, car90)
# returns sum when argument on left
rxCrossTabs(Price ~ Country:Type, car90)
# can use to output means also
rxct <- rxCrossTabs(Price ~ Country:Type, car90)
print(rxct, output = "means")
rxCrossTabs(Price ~ Country:Type, car90, means = TRUE)


# rxHistogram
rxHistogram(~ Price, car90)

# rxCube
rxCube(Price ~ Type, car90)
price_cube <- rxCube(Price ~ Country:Type, car90)
mileage_cube <- rxCube(Mileage ~ Country:Type, car90)
library(dplyr)
output <- bind_cols(list(price_cube, mileage_cube))
output <- output[, c('Country', 'Type', 'Counts', 'Price', 'Mileage')]

# rxKmeans
rxK <- rxKmeans(~Sepal.Length + Sepal.Width + Petal.Length + Petal.Width, iris, numClusters = 3, seed = 8675309)
iris_output <- cbind(iris, cluster = as.factor(rxK$cluster))
rxCrossTabs(~Species:cluster, iris_output)

# rxLinMod
rxlm <- rxLinMod(Price ~ Mileage + Weight, car90)
names(rxlm)
rxlm$f.pvalue
rxlm$r.squared
rxlm$coef.p.value
summary(rxlm)

## 70% of the sample size
smp_size <- floor(0.70 * nrow(car90))

## set the seed to make your partition reproductible
set.seed(8675309)
train_ind <- sample(seq_len(nrow(car90)), size = smp_size)
train <- car90[train_ind,]
test <- car90[-train_ind,]

rxlm <- rxLinMod(Price ~ Mileage + Weight, train)
rxPredict(rxlm, test)
rxPredict(rxlm, test, computeResiduals = TRUE)

regfit <- lm(Price ~ Mileage + Weight, train)
regfit$fitted.values

# rxQuantile
rxQuantile("Sepal.Length", iris)

# rxLinePlot
rxLinePlot(dist ~ speed, cars, type = "p")

# rxCor
rxCor(~ Price + Mileage, car90)

```
