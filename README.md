# index.Rmd
Yoan Bidart  
9/22/2017  



# Overview
We build a model to predict a type of activity from accelerometers' data. The data form this project comes from this source :  http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har. We created a tree and a random forest model on our train dataset and then calculated the accuracy on a test set. 

# Background 
Using e-devices to collect data about personal activity is now mainstream. A group collected the data from people doing an activity to find patterns to recognize if barbells exercises were made correctly. More information is available from the website here: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

# Downloading the data

```r
download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", "training.csv")
download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", "testing.csv")
```

# Loading and processing the data

```r
data <- read.csv("training.csv")
validating <- read.csv("testing.csv")
library(caret)
```

```
## Warning: package 'caret' was built under R version 3.4.1
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
library(rpart)
library(rpart.plot)
library(randomForest)
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```r
set.seed(202020)
# remove values that have low variance
nzv <- nearZeroVar(data)
alldata <- data[,-nzv]
# remove variables that are mostly NA
NaCol <- sapply(alldata, function(x) mean(is.na(x)))>.95
alldata <- alldata[,NaCol==FALSE]
# remove useless variables : user details and time
alldata <- alldata[,-(1:5)]
```
We did remove a bunch of variables, as they were not useful such as user description and time series, variables which were mostly NA, and low variance ones. 

# Creating training and testing set 
We used cross validation

```r
inTrain <- createDataPartition(alldata$classe, p=.7, list=FALSE)
train <- alldata[inTrain,]
test <- alldata[-inTrain,]
```

# Prediction models and accuracy

## Regression tree

```r
set.seed(12345)
control <- trainControl(method="cv", number=3, verboseIter=FALSE)
fit1 <- train(classe~., data=train, method="rpart", trControl=control)
rpart.plot(fit1$finalModel)
```

![](index_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
pred1 <- predict(fit1, newdata=test)
confusionMatrix(pred1, test$classe)$overall
```

```
##       Accuracy          Kappa  AccuracyLower  AccuracyUpper   AccuracyNull 
##      0.5673747      0.4473015      0.5546009      0.5800816      0.2844520 
## AccuracyPValue  McnemarPValue 
##      0.0000000      0.0000000
```
Accuracy is 52% but we tried to use random forest to see if it can be better.

## Random forest

```r
control <- trainControl(method="cv", number=3, verboseIter=FALSE)
fit2 <- train(classe~., data=train, method="rf", trControl=control)
fit2$finalModel
```

```
## 
## Call:
##  randomForest(x = x, y = y, mtry = param$mtry) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 27
## 
##         OOB estimate of  error rate: 0.19%
## Confusion matrix:
##      A    B    C    D    E class.error
## A 3906    0    0    0    0 0.000000000
## B    8 2650    0    0    0 0.003009782
## C    0    5 2390    1    0 0.002504174
## D    0    1    7 2244    0 0.003552398
## E    0    0    0    4 2521 0.001584158
```

```r
pred2<- predict(fit2, test)
confusionMatrix(pred2, test$classe)$overall
```

```
##       Accuracy          Kappa  AccuracyLower  AccuracyUpper   AccuracyNull 
##      0.9967715      0.9959161      0.9949628      0.9980551      0.2844520 
## AccuracyPValue  McnemarPValue 
##      0.0000000            NaN
```
We will choose this model to predict the quizz as the accuracy is 99.6%!

# Prediction quizz

```r
quiz <- predict(fit2, validating)
head(quiz)
```

```
## [1] B A B A A E
## Levels: A B C D E
```
Thank you for reading this far !
