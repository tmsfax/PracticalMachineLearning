Predictive Model for Weight-Lifting Motions
========================================================


```r
library(lattice)
library(ggplot2)
library(caret)
library(rpart)
set.seed(42)
```


# Data Processing

The data is loaded into R and invalid values are set to NA.


```r
trainingRawData = read.csv("pml-training.csv", na.strings = c("NA", "#DIV/0!"))
testingRawData = read.csv("pml-testing.csv", na.strings = c("NA", "#DIV/0!"))
```


Several variables are present in the data set for reference, but which are not intended for prediction. Those variables are the index of the sample, the name of the subject, and information about the time series. For this project, new data frames are constructed without those variables.

Also, the testing data set does not have values for most of the variables, so those columns that have no value in the testing data set are removed from the training data set as well.


```r
desiredColumns = c("roll_belt", "pitch_belt", "yaw_belt", "total_accel_belt", 
    "gyros_belt_x", "gyros_belt_y", "gyros_belt_z", "accel_belt_x", "accel_belt_y", 
    "accel_belt_z", "magnet_belt_x", "magnet_belt_y", "magnet_belt_z", "roll_arm", 
    "pitch_arm", "yaw_arm", "total_accel_arm", "gyros_arm_x", "gyros_arm_y", 
    "gyros_arm_z", "accel_arm_x", "accel_arm_y", "accel_arm_z", "magnet_arm_x", 
    "magnet_arm_y", "magnet_arm_z", "roll_dumbbell", "pitch_dumbbell", "yaw_dumbbell", 
    "total_accel_dumbbell", "gyros_dumbbell_x", "gyros_dumbbell_y", "gyros_dumbbell_z", 
    "accel_dumbbell_x", "accel_dumbbell_y", "accel_dumbbell_z", "magnet_dumbbell_x", 
    "magnet_dumbbell_y", "magnet_dumbbell_z", "roll_forearm", "pitch_forearm", 
    "yaw_forearm", "total_accel_forearm", "gyros_forearm_x", "gyros_forearm_y", 
    "gyros_forearm_z", "accel_forearm_x", "accel_forearm_y", "accel_forearm_z", 
    "magnet_forearm_x", "magnet_forearm_y", "magnet_forearm_z")

training = subset(trainingRawData, select = c(desiredColumns, "classe"))
testing = subset(testingRawData, select = c(desiredColumns, "problem_id"))
```


# Training

A model was built with rpart and cross validation. I would have used a more sophisticated technique such as random forests, except that training was too slow.

The cross-validation accuracy on the training set is shown below and the accuracy on the test set is expected to be similar.


```r
fitControl <- trainControl(method = "repeatedcv", number = 10, repeats = 10)
modFit = train(classe ~ ., data = training, method = "rpart", trControl = fitControl)
modFit
```

```
## CART 
## 
## 19622 samples
##    52 predictors
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold, repeated 10 times) 
## 
## Summary of sample sizes: 17659, 17661, 17661, 17659, 17661, 17660, ... 
## 
## Resampling results across tuning parameters:
## 
##   cp    Accuracy  Kappa  Accuracy SD  Kappa SD
##   0.04  0.5       0.4    0.01         0.02    
##   0.06  0.4       0.2    0.06         0.1     
##   0.1   0.3       0.05   0.04         0.06    
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was cp = 0.04.
```


This is the final tree used for the model:


```r
plot(modFit$finalModel, uniform = T)
text(modFit$finalModel, use.n = T, cex = 0.8)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 


# Testing

This table shows the misclassification error on the training data. Misclassification is not shown for the test data because the true values are not included.


```r
pred = predict(modFit, training)
table(pred, training$classe)
```

```
##     
## pred    A    B    C    D    E
##    A 5080 1581 1587 1449  524
##    B   81 1286  108  568  486
##    C  405  930 1727 1199  966
##    D    0    0    0    0    0
##    E   14    0    0    0 1631
```


This shows the predicted values of the test data:


```r
predictedTestResponses = predict(modFit, testing)
predictedTestResponses
```

```
##  [1] C A C A A C C A A A C C C A C A A A A C
## Levels: A B C D E
```



# Appendices

## Source Data

The WLE data set used for this project was released as part of this paper:

Velloso, E.; Bulling, A.; Gellersen, H.; Ugulino, W.; Fuks, H. Qualitative Activity Recognition of Weight Lifting Exercises. Proceedings of 4th International Conference in Cooperation with SIGCHI (Augmented Human '13) . Stuttgart, Germany: ACM SIGCHI, 2013.

More information is avaiale [here](http://groupware.les.inf.puc-rio.br/har#weight_lifting_exercises#ixzz35OWhUXw4)

## Hardware and Software Information

This project was developed and executed using R on Ubuntu. Additional details are shown below.


```r
sessionInfo()
```

```
## R version 3.1.0 (2014-04-10)
## Platform: x86_64-pc-linux-gnu (64-bit)
## 
## locale:
##  [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C              
##  [3] LC_TIME=en_US.UTF-8        LC_COLLATE=en_US.UTF-8    
##  [5] LC_MONETARY=en_US.UTF-8    LC_MESSAGES=en_US.UTF-8   
##  [7] LC_PAPER=en_US.UTF-8       LC_NAME=C                 
##  [9] LC_ADDRESS=C               LC_TELEPHONE=C            
## [11] LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C       
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] e1071_1.6-3     rpart_4.1-8     caret_6.0-30    ggplot2_0.9.3.1
## [5] lattice_0.20-29 knitr_1.5      
## 
## loaded via a namespace (and not attached):
##  [1] BradleyTerry2_1.0-5 brglm_0.5-9         car_2.0-20         
##  [4] class_7.3-10        codetools_0.2-8     colorspace_1.2-4   
##  [7] compiler_3.1.0      digest_0.6.4        evaluate_0.5.5     
## [10] foreach_1.4.2       formatR_0.10        grid_3.1.0         
## [13] gtable_0.1.2        gtools_3.4.1        iterators_1.0.7    
## [16] lme4_1.1-6          MASS_7.3-33         Matrix_1.1-3       
## [19] minqa_1.2.3         munsell_0.4.2       nlme_3.1-117       
## [22] nnet_7.3-8          plyr_1.8.1          proto_0.3-10       
## [25] Rcpp_0.11.1         RcppEigen_0.3.2.1.2 reshape2_1.4       
## [28] scales_0.2.4        splines_3.1.0       stringr_0.6.2      
## [31] tools_3.1.0
```

