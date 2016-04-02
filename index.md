# Practical Machine Learning Course Project
Robert Osterried  
April 3, 2016  

### Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset). 

### Data

The training data for this project are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

The data for this project come from this source: http://groupware.les.inf.puc-rio.br/har. If you use the document you create for this class for any purpose please cite them as they have been very generous in allowing their data to be used for this kind of assignment. 

### Project Objectives

The goal of your project is to predict the manner in which they did the exercise. This is the “classe” variable in the training set. You may use any of the other variables to predict with. You should create a report describing how you built your model, how you used cross validation, what you think the expected out of sample error is, and why you made the choices you did. You will also use your prediction model to predict 20 different test cases.

### Analysis

#### Load Training Data

Use 70% of the training data to train the model, and use the rest for cross-validation testing.


```r
library(caret)

trainUrl <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"

training <- read.csv(url(trainUrl), na.strings=c("NA","#DIV/0!",""))

inTrain <- createDataPartition(y=training$classe, p=0.7, list=FALSE)
myTraining <- training[inTrain, ]
myTesting <- training[-inTrain, ]
dim(myTraining);
```

```
## [1] 13737   160
```

#### Data Cleaning and Pre-processing

To cut down on data, first eliminate data with low variability.

```r
lowVar <- nearZeroVar(myTraining, saveMetrics = TRUE)
myTraining <- myTraining[, !lowVar$nzv]
dim(myTraining)
```

```
## [1] 13737   129
```
Then eliminate any columns with more than 50% NA's.

```r
myTraining[myTraining==""] <- NA
rateNa <- apply(myTraining, 2, function(x) sum(is.na(x)))/nrow(myTraining)
myTraining <- myTraining[!(rateNa>0.50)]
dim(myTraining)
```

```
## [1] 13737    59
```
Use the same columns for myTesting.

```r
myTesting <- myTesting[colnames(myTraining)]
```

### ML Prediction with Random Forest
Use the random forest model.

```r
library(randomForest)

randForstModel <- randomForest(classe ~. , data=myTraining, method="class")
```

Predict the in-sample error.

```r
predictions <- predict(randForstModel, myTesting, type = "class")
```

Test results with a confusion matrix.

```r
confusionMatrix(predictions, myTesting$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1674    0    0    0    0
##          B    0 1139    0    0    0
##          C    0    0 1026    0    0
##          D    0    0    0  964    0
##          E    0    0    0    0 1082
## 
## Overall Statistics
##                                      
##                Accuracy : 1          
##                  95% CI : (0.9994, 1)
##     No Information Rate : 0.2845     
##     P-Value [Acc > NIR] : < 2.2e-16  
##                                      
##                   Kappa : 1          
##  Mcnemar's Test P-Value : NA         
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            1.0000   1.0000   1.0000   1.0000   1.0000
## Specificity            1.0000   1.0000   1.0000   1.0000   1.0000
## Pos Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Neg Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
## Detection Rate         0.2845   0.1935   0.1743   0.1638   0.1839
## Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
## Balanced Accuracy      1.0000   1.0000   1.0000   1.0000   1.0000
```

### Predictions for Test Data

Load the test data, and select the same columns as above.

```r
testUrl <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"

testing <- read.csv(url(testUrl), na.strings=c("NA","#DIV/0!",""))

testing <- testing[colnames(myTraining[, -59])] # remove last column

# Coerce the columns of testing to be the same as myTraining.
testing <- rbind(myTraining[2, -59], testing)
testing <- testing[-1,]
```
Apply the above model against this test data.

```r
predictions2 <- predict(randForstModel, testing, type = "class")
predictions2
```

```
##  2  3  4  5 61  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 
##  A  A  A  A  A  A  D  A  A  A  A  A  A  A  A  A  A  A  A  B 
## Levels: A B C D E
```
These results are almost totally incorrect.  I have learned almost nothing from this "class", and have thus produced this "analysis".  Good thing I only paid $49US for it.
