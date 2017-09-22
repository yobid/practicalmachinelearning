---
title: "index.Rmd"
author: "Yoan Bidart"
date: "9/22/2017"
output: 
  html_document:
    keep_md: true
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

# Overview
We build a model to predict a type of activity from accelerometers' data. The data form this project comes from this source :  http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har. We created a tree and a random forest model on our train dataset and then calculated the accuracy on a test set. 

# Background 
Using e-devices to collect data about personal activity is now mainstream. A group collected the data from people doing an activity to find patterns to recognize if barbells exercises were made correctly. More information is available from the website here: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

# Downloading the data
```{r eval=FALSE}
download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", "training.csv")
download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", "testing.csv")
```

# Loading and processing the data
```{r}
data <- read.csv("training.csv")
validating <- read.csv("testing.csv")
library(caret)
library(rpart)
library(rpart.plot)
library(randomForest)
set.seed(202020)
# removing useless variables : time and user name and window
alldata <- data[,-(2:7)]
# remove variables that are mostly NA
NaCol <- sapply(alldata, function(x) mean(is.na(x)))>.95
alldata <- alldata[,NaCol==FALSE]
# remove values that have low variance
nzv <- nearZeroVar(alldata)
alldata <- alldata[,-nzv]
```
We did remove a bunch of variables, as they were not useful such as user description and time series, variables which were mostly NA, and low variance ones. 

# Creating training and testing set 
We used cross validation
```{r}
inTrain <- createDataPartition(alldata$classe, p=.7, list=FALSE)
train <- alldata[inTrain,]
test <- alldata[-inTrain,]
```

# Prediction models and accuracy

## Regression tree
```{r}
fit1 <- rpart(classe~., data=train, method="class")
rpart.plot(fit1)
pred1 <- predict(fit1, test, type="class")
confusionMatrix(pred1, test$classe)$overall
```
Accuracy is quite good but we tried to use random forest to see if it can be better.

## Random forest
```{r}
fit2 <- randomForest(classe~., data=train, model="class")
pred2<- predict(fit2, test, type="class")
confusionMatrix(pred2, test$classe)$overall
```
This model seems fine to predict the quiz.

# Prediction quizz
```{r}
quiz <- predict(fit2, validating, model="class")
head(quiz)
```
Thank you for reading this far !
