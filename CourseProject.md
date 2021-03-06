---
title: "Personal Activity Measurement and Prediction - Practical Machine Learning Course Project"
author: "Andrii Daniliuk"
date: "Monday, February 17, 2015"
---

## Submission

---

The goal of the project is to predict the manner in which six young health selected participants did their exercise. They were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions: exactly according to the specification (Class A), throwing the elbows to the front (Class B), lifting the dumbbell only halfway (Class C), lowering the dumbbell only halfway (Class D) and throwing the hips to the front (Class E) [1]. 

For this purpose the "classe" variable in the training set is selected. As a result a report describing how the model is build, how the cross validation is used, what the expected out of sample error is, and why the choices of the model is done. The prediction model is used to predict 20 different test cases.

The training data for this project are available at 'https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv', the test data are available at 'https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv'.

---

## Data Preprocessing

---

First we need to be sure that all packages for the analysis as `caret`, `randomForest` and `rpart` packages are installed and ready for further use and :


```r
if (!require("caret")) { 
        install.packages("caret") 
}
if (!require("randomForest")) { 
        install.packages("randomForest") 
}
if (!require("rpart")) { 
        install.packages("rpart") 
}
library(caret)
library(randomForest)
library(rpart)
```

Now we shall create a folder for keeping the data there, set the seed at *90210*  for further reproducibility and load both training and testing data:


```r
mainDir <- "C:\\Users\\pc\\Documents\\R Prog"
subDir <- "PractMachLearn_CourseProject"
if (file.exists(subDir)){
    setwd(file.path(mainDir, subDir))
} else {
    dir.create(file.path(mainDir, subDir), showWarnings = FALSE)
    setwd(file.path(mainDir, subDir))
}
set.seed(90210)
getUrlTrain <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
getUrlTest <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
download.file(getUrlTrain, destfile = "pml-training.csv",method = "curl")
download.file(getUrlTest, destfile = "pml-testing.csv",method = "curl")
training <- read.csv("pml-training.csv",na.strings=c("","#DIV/0!","NA"))
testing <- read.csv("pml-testing.csv",na.strings=c("","#DIV/0!","NA"))
str(training); str(testing)
```

As we could notice due to `str(training)` results there are NA values in the training dataset coded as "", "#DIV/0" and "NA", so for better data transformation we changed all of them to "NA".

Next, we shall remove all the columns of the datasets which consist of NA values only:


```r
training <- training[,colSums(is.na(training))==0]
testing <- testing[,colSums(is.na(testing))==0]
```

Now we shall transform the datasets for the analysis excluding the variables which are not applicable for our task:


```r
pred_colTR<-grepl("belt|[^(fore)]arm|dumbbell|forearm", names(training))
train_pred_columns<-names(training)[pred_colTR]
training<-training[,c("classe",train_pred_columns)]
pred_colTE<-grepl("belt|[^(fore)]arm|dumbbell|forearm", names(testing))
test_pred_columns<-names(testing)[pred_colTE]
testing<-testing[,c("problem_id",train_pred_columns)]
```

Thus, both datasets are ready for further modeling.

---

## Model Selection

---

Cross-validation of the datasets shall be performed by subsampling our training set into 2 subsamples: subTraining (70% of the training set) and subTesting (30%). All the models will be fitted on the subTraining set and further tested on the subTesting one.

As known the cross-validation assists us in identifying the expected out-of-sample error which is 100% minus the accuracy of the validation (in %). The accuracy of the subsample itself means the proportion of correctly classified observations within the subTesting sample. Upon selection the accuratest model we will tested it on the testing set of data on the same principle for the out-of-sample error (100% * (1-accuracy of the testing set)).

So, let's do the subsampling:


```r
inSubTrain <- createDataPartition(y=training$classe,p=0.7,list=FALSE)
subTraining <- training[inSubTrain, ]
subTesting <- training[-inSubTrain, ]
```

First we shall check the `randomForest` model with the help of `confusionMatrix`:


```r
modFitRF<-randomForest(classe~.,data=subTraining)
predRF<-predict(modFitRF,newdata=subTesting)
RF<-confusionMatrix(predRF,subTesting$classe)[3]
```

Next let's predict on the decision tree model - `rpart` function:


```r
modFitDT<-rpart(classe~.,data=subTraining,method="class")
predDT<-predict(modFitDT,newdata=subTesting,type="class")
DT<-confusionMatrix(predDT,subTesting$classe)[3]
```

The third one will be the linear discriminant analysis - `lda` - model of the `caret` package to be checked:


```r
modFitLDA<-train(classe~.,method="lda",data=subTraining)
predLDA<-predict(modFitLDA,newdata=subTesting)
LDA<-confusionMatrix(predLDA,subTesting$classe)[3]
```

Finally we may compare the accuracy of all the models and find the one with the smallest out-of-sample error:


```r
compare<-data.frame(RF,DT,LDA)[1,]
names(compare)<-c("RF","DT","LDA")
compare
```

```
##                 RF        DT       LDA
## Accuracy 0.9964316 0.7592184 0.6914189
```

As calculated, the Random Forest model predicts the subsample with the highest accuracy (the out-of-sample error is less than 0.5%) compared to the Decision Tree (the out-of-sample error is more than 24%) and Linear Discriminant Analysis (almost 31%).

---

## Finalization and Submission

---

Finally, we may predict the testing dataset using the selected the Random Forest model:


```r
predFinal<-predict(modFitRF,newdata=testing)
predFinal
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```

Now the data are ready for the submission:


```r
submission <- function(p){
        n = length(p)
        for(i in 1:n){
                fname<-paste0("problem_id_",i,".txt")
                write.table(p[i],file=fname,quote=FALSE,row.names=FALSE,col.names=FALSE)
        }
}
submission(predFinal)
```

---

## Links and References

---

1. http://groupware.les.inf.puc-rio.br/har

---
