---
title: "Statistical Consulting: HSPSModelR R Package Workflow"
author: "Brad Borget, Spencer Cook, Chad Schaeffer, Nic Stover, Dallin Webb"
date: "2019-03-07"
output: rmarkdown::html_vignette
vignette: >
  %\VignetteIndexEntry{Example workflow}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---

## Overview

**HSPSModelR** is an R package that reduces the time it takes to fit and compare many machine learning models to a cognostic dataset. It is designed to give the user a quick snapshot of which classification algorithms perform well, as well as extract variable importance. This package works only for binary classification problems.

<br>

## Installation


```r
devtools::install_github("BYUIDSS/HSPSModelR")
```


```r
library(caret)
library(dplyr)
library(HSPSModelR)
```

<br>

## Clean Data with `preprocess_data`

The `preprocess_data` function's main purpose is to reduce unnessesary predictor columns using correlation and variance. It also has the ability to impute missing data and change the target class into either factor or binary. 


```r
data(dhfr)

dhfr_reduced <- preprocess_data(dhfr, target = "Y", reduce_cols = TRUE)
```


```r
ncol(dhfr)
```

```
## [1] 229
```

```r
ncol(dhfr_reduced)
```

```
## [1] 112
```


This function does not scale or perform any other major data tranformation. Such transformations should be specified while training as to be more easily reverted for interpretation purposes. For more options, see `caret`'s pre-processing functions [here](http://topepo.github.io/caret/pre-processing.html){target="_blank"}.

<br>

## Split data


```r
set.seed(1)
index   <- caret::createDataPartition(dhfr_reduced$Y, p = .8, list = F)
train_x <- dhfr_reduced[ index, 1:111]
test_x  <- dhfr_reduced[-index, 1:111]
train_y <- dhfr_reduced[ index, "Y"]
test_y  <- dhfr_reduced[-index, "Y"]
```

<br>

## Install packages needed to train models with `install_train_packages`

In preparation for the `run_models` function, this function i

## Run multiple models with `run_models`

This function takes considerable time to run as it trains 68 classification methods from the caret package. To reduce this time, make sure to run `install_train_packages()` which installs all the libraries required to iterate through each of the 68 methods. 

Many preprocessing and training parameters can be specified such as preprocess, folds, iterations, etc. Since caret includes a lot of information in each `train` object, this function implements a trimming method to retain just enough information to be able to predict.


```r
install_train_packages()
```


```r
models_list <- run_models(train_x     = train_x,
                          train_y     = train_y,
                          seed        = 1,
                          num_folds   = 2,
                          trim_models = TRUE,
                          light       = TRUE,
                          prepro_methods = c("center","scale","YeoJohnson"))
```

```
## + Fold1: mstop=150, prune=no 
## - Fold1: mstop=150, prune=no 
## + Fold2: mstop=150, prune=no 
## - Fold2: mstop=150, prune=no 
## Aggregating results
## Selecting tuning parameters
## Fitting mstop = 150, prune = no on full training set
## + Fold1: ncomp=3 
## - Fold1: ncomp=3 
## + Fold2: ncomp=3 
## - Fold2: ncomp=3 
## Aggregating results
## Selecting tuning parameters
## Fitting ncomp = 3 on full training set
## + Fold1: mtry=  2 
## - Fold1: mtry=  2 
## + Fold1: mtry= 56 
## - Fold1: mtry= 56 
## + Fold1: mtry=111 
## - Fold1: mtry=111 
## + Fold2: mtry=  2 
## - Fold2: mtry=  2 
## + Fold2: mtry= 56 
## - Fold2: mtry= 56 
## + Fold2: mtry=111 
## - Fold2: mtry=111 
## Aggregating results
## Selecting tuning parameters
## Fitting mtry = 2 on full training set
## + Fold1: degree=1, nprune=17 
## earth glm inactive: did not converge after 25 iterations
## earth glm inactive: did not converge after 25 iterations
## earth glm inactive: did not converge after 25 iterations
## - Fold1: degree=1, nprune=17 
## + Fold2: degree=1, nprune=17 
## earth glm inactive: did not converge after 25 iterations
## - Fold2: degree=1, nprune=17 
## Aggregating results
## Selecting tuning parameters
## Fitting nprune = 17, degree = 1 on full training set
## + Fold1: cp=0.00, K=1, L=9 
## - Fold1: cp=0.00, K=1, L=9 
## + Fold1: cp=0.00, K=3, L=9 
## - Fold1: cp=0.00, K=3, L=9 
## + Fold1: cp=0.05, K=1, L=9 
## - Fold1: cp=0.05, K=1, L=9 
## + Fold1: cp=0.05, K=3, L=9 
## - Fold1: cp=0.05, K=3, L=9 
## + Fold1: cp=0.10, K=1, L=9 
## - Fold1: cp=0.10, K=1, L=9 
## + Fold1: cp=0.10, K=3, L=9 
## - Fold1: cp=0.10, K=3, L=9 
## + Fold2: cp=0.00, K=1, L=9 
## - Fold2: cp=0.00, K=1, L=9 
## + Fold2: cp=0.00, K=3, L=9 
## - Fold2: cp=0.00, K=3, L=9 
## + Fold2: cp=0.05, K=1, L=9 
## - Fold2: cp=0.05, K=1, L=9 
## + Fold2: cp=0.05, K=3, L=9 
## - Fold2: cp=0.05, K=3, L=9 
## + Fold2: cp=0.10, K=1, L=9 
## - Fold2: cp=0.10, K=1, L=9 
## + Fold2: cp=0.10, K=3, L=9 
## - Fold2: cp=0.10, K=3, L=9 
## Aggregating results
## Selecting tuning parameters
## Fitting K = 3, L = 6, cp = 0 on full training set
```

<br>

## Compare model performance with `get_performance`

This function takes a list of models and outputs a dataframe where each row represents a specific perfmance metric for a trained model (labeled by the training method) calculated from `caret::confusionMatrix`

The default ouput is in long format. If you would like it in wide, specify the `format` argument to be `"wide"`.


```r
model_performance <- get_performance(models_list, test_x, test_y)

model_performance
```

```
## # A tibble: 90 x 3
##    method           measure       score
##    <chr>            <chr>         <dbl>
##  1 rf               Accuracy      0.891
##  2 pls              Accuracy      0.875
##  3 earth            Accuracy      0.875
##  4 rotationForestCp Accuracy      0.875
##  5 glmboost         Accuracy      0.859
##  6 rf               AccuracyLower 0.788
##  7 pls              AccuracyLower 0.768
##  8 earth            AccuracyLower 0.768
##  9 rotationForestCp AccuracyLower 0.768
## 10 glmboost         AccuracyLower 0.750
## # ... with 80 more rows
```


```r
get_performance(models_list, test_x, test_y, format = "wide")
```

```
## # A tibble: 5 x 19
##   method Accuracy AccuracyLower AccuracyNull AccuracyPValue AccuracyUpper
##   <chr>     <dbl>         <dbl>        <dbl>          <dbl>         <dbl>
## 1 earth     0.875         0.768        0.625     0.00000828         0.944
## 2 glmbo~    0.859         0.750        0.625     0.0000323          0.934
## 3 pls       0.875         0.768        0.625     0.00000828         0.944
## 4 rf        0.891         0.788        0.625     0.00000186         0.955
## 5 rotat~    0.875         0.768        0.625     0.00000828         0.944
## # ... with 13 more variables: `Balanced Accuracy` <dbl>, `Detection
## #   Prevalence` <dbl>, `Detection Rate` <dbl>, F1 <dbl>, Kappa <dbl>,
## #   McnemarPValue <dbl>, `Neg Pred Value` <dbl>, `Pos Pred Value` <dbl>,
## #   Precision <dbl>, Prevalence <dbl>, Recall <dbl>, Sensitivity <dbl>,
## #   Specificity <dbl>
```

Further exploration can be performed to view a specific measurement such as the F1 measure. F1 is a measure of a test's accuracy that weights the average of precision and recall.


```r
model_performance %>%
  filter(measure == "F1")
```

```
## # A tibble: 5 x 3
##   method           measure score
##   <chr>            <chr>   <dbl>
## 1 rf               F1      0.916
## 2 pls              F1      0.907
## 3 rotationForestCp F1      0.902
## 4 earth            F1      0.9  
## 5 glmboost         F1      0.894
```

<br>

## Make Predictions


```r
get_common_predictions(models    = models_list, 
                       test_x    = test_x, 
                       threshold = 0.70)
```

```
## # A tibble: 14 x 7
##       ID agreeance glmboost   pls    rf earth rotationForestCp
##    <int>     <dbl>    <dbl> <dbl> <dbl> <dbl>            <dbl>
##  1    21     0.909    0.901 0.710 0.962 0.999            0.974
##  2     3     0.901    0.985 0.680 0.926 0.991            0.920
##  3    13     0.892    0.953 0.702 0.832 0.999            0.974
##  4    11     0.886    0.980 0.723 0.752 1.000            0.974
##  5     8     0.883    0.940 0.709 0.792 1.000            0.974
##  6    10     0.875    0.966 0.734 0.828 0.980            0.866
##  7     7     0.873    0.902 0.710 0.788 0.992            0.974
##  8    18     0.866    0.945 0.682 0.832 0.980            0.889
##  9    15     0.863    0.946 0.708 0.688 1.000            0.974
## 10    22     0.852    0.965 0.650 0.672 1.000            0.974
## 11    23     0.747    0.665 0.515 0.802 0.989            0.764
## 12     5     0.744    0.823 0.611 0.692 0.956            0.640
## 13    12     0.725    0.587 0.547 0.852 0.956            0.685
## 14     9     0.722    0.590 0.533 0.848 0.956            0.685
```

The get_common_predictions function takes a list of models and outputs a vector of rows whose desired outcomes were agreed upon by a prespecified ratio of models. Downloading the HSPSModelR package and using run_models() then get_common_predictions() is the fastest way to get from raw data to quality results.

<br>

## Extract variable importance with `var_imp_*()`

The `var_imp_*` functions output tibbles where rows rank the individual contribution of features to final predictions. `var_imp_overall` ranks the most influential features from among the entire list of models, whereas `var_imp_raw` gives the most important variable in sequence from each model listed in the function.


```r
suppressMessages( var_imp_overall(models_list) ) 
```

```
## # A tibble: 111 x 4
##    features            rank rank_place rank_scaled
##    <chr>              <dbl>      <int>       <dbl>
##  1 moe2D_PEOE_VSA.0.1   551          1       1    
##  2 moe2D_PEOE_VSA.4.1   549          2       0.996
##  3 moe2D_vsa_other      535          3       0.969
##  4 moe2D_BCUT_SMR_0     505          4       0.911
##  5 moe2D_PEOE_VSA_NEG   491          5       0.884
##  6 moe2D_PEOE_RPC..1    484          6       0.871
##  7 moe2D_PEOE_VSA.3     464          7       0.832
##  8 moe2D_BCUT_PEOE_1    463          8       0.830
##  9 moe2D_GCUT_SLOGP_3   444          9       0.793
## 10 moeGao_chi4cav_C     436         10       0.778
## # ... with 101 more rows
```

```r
suppressMessages( var_imp_raw(models_list) )
```

```
## # A tibble: 555 x 4
##    model    features           Overall  rank
##    <chr>    <chr>                <dbl> <int>
##  1 glmboost moe2D_PEOE_VSA.0.1   100       1
##  2 glmboost moe2D_PEOE_VSA.4.1    55.5     2
##  3 glmboost moe2D_vsa_other       45.7     3
##  4 glmboost moeGao_chi4cav_C      41.0     4
##  5 glmboost moe2D_BCUT_PEOE_1     33.4     5
##  6 glmboost moe2D_PEOE_VSA_NEG    32.9     6
##  7 glmboost moe2D_SlogP_VSA1      30.3     7
##  8 glmboost moe2D_kS_aaNH         30.3     8
##  9 glmboost moe2D_PEOE_RPC..1     22.1     9
## 10 glmboost moe2D_a_ICM           20.8    10
## # ... with 545 more rows
```

<br>

## Visualize variable importance with `gg_var_imp()`

`gg_var_imp()` is available to quickly visualize these variables, allowing you to specify the top number of ranked variables with the `top_num` argument.


```r
vars <- var_imp_overall(models_list)
```

```
## 
## NOTE: Coefficients from a Binomial model are half the size of coefficients
##  from a model fitted via glm(... , family = 'binomial').
## See Warning section in ?coef.mboost
```

```
## 5 out of 5 models selected by caret::varImp()
```

```r
gg_var_imp(vars, top_num = 20)
```

<img src="/project/statistical_consulting_files/figure-html/unnamed-chunk-13-1.png" width="672" />

<br>

