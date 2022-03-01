---
title: "Kaggle house price using some ML algorithm"
author: "Tue Vu"
date: "2/28/2022"
output: html_document
---



## Kaggle house price prediction

This is the sample R Markdown file following the workshop to create the template for Data Science workflow that we learnt inside class:

### Step 1: Load library, Load data



```r
rm(list=ls())
library(caret)
```

```
## Loading required package: lattice
```

```
## 
## Attaching package: 'caret'
```

```
## The following object is masked from 'package:purrr':
## 
##     lift
```

```r
library(GGally)
```

```
## Registered S3 method overwritten by 'GGally':
##   method from   
##   +.gg   ggplot2
```

```r
library(ggplot2)
library(RColorBrewer)
setwd("/home/tuev/Kaggle/house-prices")

df_train <- read.csv("https://raw.githubusercontent.com/vuminhtue/SMU_Data_Science_workflow_R/master/data/Kaggle_house_prices/train.csv",head=T)
df_test <- read.csv("https://raw.githubusercontent.com/vuminhtue/SMU_Data_Science_workflow_R/master/data/Kaggle_house_prices/test.csv",head=T)
```


### Step 2: Select variables and check for missing value


```r
df_small <- df_train[c("OverallQual","OverallCond","YearBuilt","X1stFlrSF",
                       "FullBath","GarageCars","SaleCondition","SalePrice")]
ggpairs(df_small,aes(colour=SaleCondition))
```

```
## 
## 
## 
## 
## 
## 
## 
## 
```

<img src="figure/unnamed-chunk-3-1.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" width="100%" />

```r
sum(is.na(df_small))
```

```
## [1] 0
```

Convert SaleCondition to integer:


```r
df_small$SaleCondition <- as.integer(as.factor(df_small$SaleCondition))
```

### Step 3: Create partition for the data


```r
ind1 <- createDataPartition(y=df_small$SalePrice,p=0.6,list=FALSE,times=1)
training <- df_small[ind1,]
testing  <- df_small[-ind1,] 
```

### Step 4: Apply 1 ML algorithm to the data and calculate prediction

Apply Random Forest:


```r
model_rf <- train(SalePrice~., data=training,method="rf",
                  prox=TRUE)
predict_rf <- predict(model_rf,testing)
```

Apply Neural Network


```r
library(neuralnet)
```

```
## 
## Attaching package: 'neuralnet'
```

```
## The following object is masked from 'package:dplyr':
## 
##     compute
```

```r
smax <- apply(training,2,max)
smin <- apply(training,2,min)
trainNN <- as.data.frame(scale(training,center=smin,scale=smax-smin))
testNN <- as.data.frame(scale(testing,center=smin,scale=smax-smin))

set.seed(123)
ModNN <- neuralnet(SalePrice~.,trainNN, hidden=c(4,3),linear.output = T)

predictNN <- compute(ModNN,testNN)
predictNN_rescaled <- predictNN$net.result*(smax-smin)[1]+smin[1]
```


```r
plot(ModNN)
```


### Step 5: Evaluate the model output

Since the model output is continuous output, we use postResample to evaluate:


```r
print(postResample(predict_rf,testing$SalePrice))
```

```
##         RMSE     Rsquared          MAE 
## 3.555839e+04 8.102577e-01 2.251900e+04
```

```r
print(postResample(predictNN_rescaled,testing$SalePrice))
```

```
##         RMSE     Rsquared          MAE 
## 1.985016e+05 7.756623e-01 1.810589e+05
```

Display the scatter plots:


```r
testing$predicted_rf <- predict_rf
ggplot(testing,aes(x=SalePrice,y=predict_rf))+
  geom_point(aes(color=factor(YearBuilt)))+
  geom_smooth(method="loess")
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="figure/unnamed-chunk-10-1.png" title="plot of chunk unnamed-chunk-10" alt="plot of chunk unnamed-chunk-10" width="100%" />

Since the Random Forest gave better results, we use this algorithm to predict the testing data:



### Step 6: Predict the test data and write output


```r
df_small_test <- df_test[c("OverallQual","OverallCond","YearBuilt","X1stFlrSF",
                       "FullBath","GarageCars","SaleCondition")]
df_small_test$SaleCondition <- as.integer(as.factor(df_small_test$SaleCondition))

output <- predict(model_rf,df_small_test)
output <- cbind(df_test$Id,output)
colnames(output) <- c("ID","SalePrice")
write.csv(output,"Output.csv",row.names = FALSE)
```