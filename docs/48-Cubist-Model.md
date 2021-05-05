# Cubist Model  


## Data Preparation



```r
library("Cubist")
library("mlbench")

## Data Preparation

data(BostonHousing)
BostonHousing$chas <- as.numeric(BostonHousing$chas) - 1

set.seed(1)
inTrain <- sample(1:nrow(BostonHousing), floor(.8*nrow(BostonHousing)))

## Predictors
train_pred <- BostonHousing[ inTrain, -14]
test_pred  <- BostonHousing[-inTrain, -14]

## Responder variable
train_resp <- BostonHousing$medv[ inTrain]
test_resp  <- BostonHousing$medv[-inTrain]
```


## Fit Continious Outcome

The modelTree method for Cubist shows the usage of each variable in either the rule conditions or the (terminal) linear model. In actuality, many more linear models are used in prediction that are shown in the output. Because of this, the variable usage statistics shown at the end of the output of the summary function will probably be inconsistent with the rules also shown in the output. At each split of the tree, Cubist saves a linear model (after feature selection) that is allowed to have terms for each variable used in the current split or any split above it. Quinlan (1992) discusses a smoothing algorithm where each model prediction is a linear combination of the parent and child model along the tree. As such, the final prediction is a function of all the linear models from the initial node to the terminal node. The percentages shown in the Cubist output reflects all the models involved in prediction (as opposed to the terminal models shown in the output).

> Cubist的modelTree方法显示了规则条件或（最终）线性模型中每个变量的用法。实际上，在预测中使用了更多的线性模型，这些模型显示在输出中。因此，摘要功能输出末尾显示的变量使用情况统计信息可能与输出中也显示的规则不一致。在树的每个分割处，Cubist都会保存一个线性模型（在特征选择之后），该线性模型允许对当前分割处或其上方任何分割处使用的每个变量都具有术语。 Quinlan（1992）讨论了一种平滑算法，其中每个模型预测都是沿着树的父模型和子模型的线性组合。这样，最终预测是从初始节点到终端节点的所有线性模型的函​​数。立体派输出中显示的百分比反映了预测中涉及的所有模型（与输出中显示的终端模型相对）。


```r
## Continious Outcome
## Building mOdel
model_tree <- cubist(x = train_pred, y = train_resp)
summary(model_tree)
```

```
## 
## Call:
## cubist.default(x = train_pred, y = train_resp)
## 
## 
## Cubist [Release 2.07 GPL Edition]  Wed May  5 21:53:01 2021
## ---------------------------------
## 
##     Target attribute `outcome'
## 
## Read 404 cases (14 attributes) from undefined.data
## 
## Model:
## 
##   Rule 1: [77 cases, mean 14.04, range 5 to 27.5, est err 2.14]
## 
##     if
## 	nox > 0.668
##     then
## 	outcome = -2.18 + 3.47 dis + 21.6 nox - 0.32 lstat + 0.0089 b
## 	          - 0.12 ptratio - 0.02 crim - 0.005 age
## 
##   Rule 2: [167 cases, mean 19.30, range 7 to 31, est err 2.15]
## 
##     if
## 	nox <= 0.668
## 	lstat > 9.53
##     then
## 	outcome = 43.02 - 0.94 ptratio - 0.27 lstat - 0.84 dis - 0.034 age
## 	          + 0.0082 b - 0.1 indus + 0.4 rm
## 
##   Rule 3: [29 cases, mean 25.08, range 18.2 to 50, est err 2.64]
## 
##     if
## 	rm <= 6.226
## 	lstat <= 9.53
##     then
## 	outcome = -18.07 + 3.91 crim - 1.67 lstat + 0.0834 b + 3.3 rm
## 
##   Rule 4: [143 cases, mean 29.49, range 16.5 to 50, est err 2.39]
## 
##     if
## 	dis > 2.3999
## 	lstat <= 9.53
##     then
## 	outcome = -28.26 + 11.1 rm + 0.62 crim - 0.58 lstat - 0.017 tax
## 	          - 0.059 age + 11.3 nox - 0.58 dis - 0.52 ptratio
## 
##   Rule 5: [14 cases, mean 40.54, range 22 to 50, est err 5.28]
## 
##     if
## 	rm > 6.226
## 	dis <= 2.3999
## 	lstat <= 9.53
##     then
## 	outcome = -5.27 + 3.14 crim - 5.18 dis - 1.22 lstat + 9.6 rm
## 	          - 0.0141 tax - 0.031 age - 0.39 ptratio
## 
## 
## Evaluation on training data (404 cases):
## 
##     Average  |error|               2.13
##     Relative |error|               0.31
##     Correlation coefficient        0.96
## 
## 
## 	Attribute usage:
## 	  Conds  Model
## 
## 	   82%   100%    lstat
## 	   57%    51%    nox
## 	   37%    93%    dis
## 	   10%    82%    rm
## 	          93%    age
## 	          93%    ptratio
## 	          63%    b
## 	          61%    crim
## 	          39%    indus
## 	          37%    tax
## 
## 
## Time: 0.0 secs
```

```r
## Make the prediction on the test datasets
Cubist.probs <- predict(model_tree, test_pred)

## Test set RMSE
sqrt(mean((Cubist.probs - test_resp)^2))
```

```
## [1] 3.722453
```

```r
## Test set R^2
cor(Cubist.probs, test_resp)^2
```

```
## [1] 0.7733923
```

### Variable Importance
 
the variable importance is a linear combination of the usage in the rule conditions and the model.





### Summary display

The tidyRules function in the tidyrules package returns rules in a tibble (an extension of dataframe) with one row per rule. The tibble provides these information about the rule: support, mean, min, max, error, LHS, RHS and committee. 


```r
library("tidyrules")
tr <- tidyRules(model_tree)
tr
```

```
## # A tibble: 5 x 9
##      id LHS          RHS               support  mean   min   max error committee
##   <int> <chr>        <chr>               <int> <dbl> <dbl> <dbl> <dbl>     <int>
## 1     1 nox > 0.668  (-2.18) + (3.47 …      77  14.0   5    27.5  2.14         1
## 2     2 nox <= 0.66… (43.02) - (0.94 …     167  19.3   7    31    2.15         1
## 3     3 rm <= 6.226… (-18.07) + (3.91…      29  25.1  18.2  50    2.64         1
## 4     4 dis > 2.399… (-28.26) + (11.1…     143  29.5  16.5  50    2.39         1
## 5     5 rm > 6.226 … (-5.27) + (3.14 …      14  40.5  22    50    5.28         1
```

```r
tr[, c("LHS", "RHS")]
```

```
## # A tibble: 5 x 2
##   LHS                          RHS                                              
##   <chr>                        <chr>                                            
## 1 nox > 0.668                  (-2.18) + (3.47 * dis) + (21.6 * nox) - (0.32 * …
## 2 nox <= 0.668 & lstat > 9.53  (43.02) - (0.94 * ptratio) - (0.27 * lstat) - (0…
## 3 rm <= 6.226 & lstat <= 9.53  (-18.07) + (3.91 * crim) - (1.67 * lstat) + (0.0…
## 4 dis > 2.3999 & lstat <= 9.53 (-28.26) + (11.1 * rm) + (0.62 * crim) - (0.58 *…
## 5 rm > 6.226 & dis <= 2.3999 … (-5.27) + (3.14 * crim) - (5.18 * dis) - (1.22 *…
```

### specific parts 

These results can be used to look at specific parts of the data. For example, the 4th rule predictions are:


```r
library("rlang")
library("dplyr")
char_to_expr <- function(x, index = 1, model = TRUE) {
  x <- x %>% dplyr::slice(index) 
  if (model) {
    x <- x %>% dplyr::pull(RHS) %>% rlang::parse_expr()
  } else {
    x <- x %>% dplyr::pull(LHS) %>% rlang::parse_expr()
  }
  x
}

rule_expr  <- char_to_expr(tr, 4, model = FALSE)
model_expr <- char_to_expr(tr, 4, model = TRUE)
```
  
  
## Ensembles By Committees 

The Cubist model can also use a boosting–like scheme called committees where iterative model trees are created in sequence. The first tree follows the procedure described in the last section. Subsequent trees are created using adjusted versions to the training set outcome: if the model over–predicted a value, the response is adjusted downward for the next model (and so on). Unlike traditional boosting, stage weights for each committee are not used to average the predictions from each model tree; the final prediction is a simple average of the predictions from each model tree.

> 其中按顺序创建迭代模型树。 第一棵树遵循最后一部分中描述的过程。 使用对训练集结果的调整后的版本来创建后续树：如果模型预测值过高，则针对下一个模型向下调整响应（依此类推）。 与传统的提升不同，每个委员会的阶段权重不会用于平均每个模型树的预测； 最终预测是来自每个模型树的预测的简单平均值。

### Nearest–neighbors Adjustmemt

Another innovation in Cubist using nearest–neighbors to adjust the predictions from the rule–based model. First, a model tree (with or without committees) is created. Once a sample is predicted by this model, Cubist can find it’s nearest neighbors and determine the average of these training set points.

> 使用最近邻来调整基于规则的模型中的预测。 首先，创建一个模型树（有或没有委员会）。 通过该模型预测样本后，Cubist可以找到其最近的邻居，并确定这些训练设定点的平均值。


```r
## Ensembles By Committees 
set.seed(1)
com_model <- cubist(x = train_pred, y = train_resp, committees = 5)
summary(com_model)
```

```
## 
## Call:
## cubist.default(x = train_pred, y = train_resp, committees = 5)
## 
## 
## Cubist [Release 2.07 GPL Edition]  Wed May  5 21:53:01 2021
## ---------------------------------
## 
##     Target attribute `outcome'
## 
## Read 404 cases (14 attributes) from undefined.data
## 
## Model 1:
## 
##   Rule 1/1: [77 cases, mean 14.04, range 5 to 27.5, est err 2.14]
## 
##     if
## 	nox > 0.668
##     then
## 	outcome = -2.18 + 3.47 dis + 21.6 nox - 0.32 lstat + 0.0089 b
## 	          - 0.12 ptratio - 0.02 crim - 0.005 age
## 
##   Rule 1/2: [167 cases, mean 19.30, range 7 to 31, est err 2.15]
## 
##     if
## 	nox <= 0.668
## 	lstat > 9.53
##     then
## 	outcome = 43.02 - 0.94 ptratio - 0.27 lstat - 0.84 dis - 0.034 age
## 	          + 0.0082 b - 0.1 indus + 0.4 rm
## 
##   Rule 1/3: [29 cases, mean 25.08, range 18.2 to 50, est err 2.64]
## 
##     if
## 	rm <= 6.226
## 	lstat <= 9.53
##     then
## 	outcome = -18.07 + 3.91 crim - 1.67 lstat + 0.0834 b + 3.3 rm
## 
##   Rule 1/4: [143 cases, mean 29.49, range 16.5 to 50, est err 2.39]
## 
##     if
## 	dis > 2.3999
## 	lstat <= 9.53
##     then
## 	outcome = -28.26 + 11.1 rm + 0.62 crim - 0.58 lstat - 0.017 tax
## 	          - 0.059 age + 11.3 nox - 0.58 dis - 0.52 ptratio
## 
##   Rule 1/5: [14 cases, mean 40.54, range 22 to 50, est err 5.28]
## 
##     if
## 	rm > 6.226
## 	dis <= 2.3999
## 	lstat <= 9.53
##     then
## 	outcome = -5.27 + 3.14 crim - 5.18 dis - 1.22 lstat + 9.6 rm
## 	          - 0.0141 tax - 0.031 age - 0.39 ptratio
## 
## Model 2:
## 
##   Rule 2/1: [58 cases, mean 14.03, range 5 to 36, est err 4.06]
## 
##     if
## 	dis > 1.4254
## 	dis <= 1.8956
## 	lstat > 5.91
##     then
## 	outcome = 86.06 - 20.56 dis - 0.47 lstat - 3 rm - 0.0201 b - 0.17 crim
## 	          - 4.1 nox
## 
##   Rule 2/2: [159 cases, mean 19.56, range 8.7 to 27.1, est err 1.84]
## 
##     if
## 	rm <= 6.251
## 	dis > 1.8956
##     then
## 	outcome = -0.22 + 3.8 rm - 0.076 age + 0.021 b + 11.6 nox - 0.56 ptratio
## 	          - 0.04 lstat - 0.06 dis
## 
##   Rule 2/3: [116 cases, mean 22.43, range 7.2 to 50, est err 2.23]
## 
##     if
## 	rm > 6.251
## 	lstat > 5.91
##     then
## 	outcome = -23.67 + 8.5 rm - 0.46 lstat - 0.3 crim - 0.0088 tax + 6.5 nox
## 	          - 0.2 dis - 0.19 ptratio + 0.0028 b - 0.008 age
## 
##   Rule 2/4: [10 cases, mean 25.07, range 5 to 50, est err 8.00]
## 
##     if
## 	rm <= 6.251
## 	dis <= 1.4254
##     then
## 	outcome = 174.32 - 119.61 dis - 1.31 lstat + 38.6 nox
## 
##   Rule 2/5: [8 cases, mean 27.56, range 22.5 to 50, est err 3.33]
## 
##     if
## 	rm <= 6.461
## 	lstat <= 5.91
##     then
## 	outcome = 22.34 + 6.98 crim
## 
##   Rule 2/6: [37 cases, mean 36.09, range 22.8 to 50, est err 3.40]
## 
##     if
## 	rm > 6.461
## 	tax > 265
## 	lstat <= 5.91
##     then
## 	outcome = 53.88 - 0.2698 b + 0.0588 tax + 11.5 rm + 0.52 crim - 1.83 dis
## 	          - 0.129 age - 0.3 lstat - 0.12 ptratio
## 
##   Rule 2/7: [47 cases, mean 36.19, range 24.4 to 50, est err 3.56]
## 
##     if
## 	rm > 6.461
## 	tax <= 265
##     then
## 	outcome = -23.33 - 0.0943 tax + 13.6 rm + 0.84 crim - 0.0276 b
## 	          - 0.21 lstat - 0.09 ptratio - 0.09 dis - 0.005 age + 0.006 zn
## 
## Model 3:
## 
##   Rule 3/1: [33 cases, mean 13.98, range 5 to 23.2, est err 2.95]
## 
##     if
## 	nox > 0.668
## 	b > 375.52
##     then
## 	outcome = 197.68 - 0.3916 b - 0.5 age + 36.1 nox - 0.36 lstat
## 	          - 0.08 crim
## 
##   Rule 3/2: [44 cases, mean 14.09, range 7 to 27.5, est err 2.58]
## 
##     if
## 	nox > 0.668
## 	b <= 375.52
##     then
## 	outcome = 14.76 - 0.3 lstat + 0.0196 b
## 
##   Rule 3/3: [241 cases, mean 17.54, range 5 to 31, est err 2.50]
## 
##     if
## 	lstat > 9.53
##     then
## 	outcome = 49.71 - 1.16 dis - 0.32 lstat - 1.06 ptratio - 14.8 nox
## 	          + 0.0091 b + 0.9 rm + 0.07 rad - 0.017 age - 0.0023 tax
## 	          - 0.03 crim
## 
##   Rule 3/4: [29 cases, mean 25.08, range 18.2 to 50, est err 3.71]
## 
##     if
## 	rm <= 6.226
## 	lstat <= 9.53
##     then
## 	outcome = -71.35 + 3.01 crim + 0.1693 b - 1.96 lstat + 5.5 rm
## 	          + 0.47 indus + 0.0173 tax
## 
##   Rule 3/5: [134 cases, mean 31.94, range 16.5 to 50, est err 2.79]
## 
##     if
## 	rm > 6.226
## 	lstat <= 9.53
##     then
## 	outcome = -13.33 + 1.72 crim + 9.9 rm - 0.87 lstat - 0.86 dis
## 	          - 0.73 ptratio - 0.045 age - 0.0033 tax
## 
## Model 4:
## 
##   Rule 4/1: [17 cases, mean 11.90, range 8.4 to 17.8, est err 1.86]
## 
##     if
## 	nox > 0.693
## 	lstat > 19.52
##     then
## 	outcome = 12.1 - 0.64 crim + 2 rm - 0.18 ptratio - 3.1 nox
## 
##   Rule 4/2: [38 cases, mean 12.67, range 5 to 23.7, est err 3.61]
## 
##     if
## 	nox <= 0.693
## 	dis > 1.4254
## 	lstat > 19.52
##     then
## 	outcome = 112.54 - 116.7 nox - 4.67 dis - 0.109 age - 0.0155 b
## 	          - 0.008 tax
## 
##   Rule 4/3: [70 cases, mean 18.02, range 9.6 to 27.5, est err 1.96]
## 
##     if
## 	crim > 1.42502
## 	dis > 1.4254
## 	lstat > 5.91
## 	lstat <= 19.52
##     then
## 	outcome = 28.61 - 0.9 lstat + 0.0105 b - 0.027 age - 0.06 crim + 3 nox
## 
##   Rule 4/4: [61 cases, mean 18.76, range 12.7 to 25, est err 1.92]
## 
##     if
## 	crim > 0.21977
## 	crim <= 1.42502
## 	rm <= 6.546
## 	lstat > 5.91
##     then
## 	outcome = -8.96 + 6.4 rm - 0.108 age - 0.09 lstat + 0.0017 b
## 	          - 0.07 ptratio - 0.07 dis - 0.0008 tax - 0.01 crim
## 
##   Rule 4/5: [112 cases, mean 21.60, range 13.6 to 29.4, est err 1.67]
## 
##     if
## 	crim <= 0.21977
## 	rm <= 6.546
## 	lstat <= 19.52
##     then
## 	outcome = -37.71 + 18.25 crim + 7.3 rm + 0.0451 b - 0.0149 tax
## 	          + 0.32 lstat - 0.068 age
## 
##   Rule 4/6: [9 cases, mean 22.30, range 5 to 50, est err 11.24]
## 
##     if
## 	rm <= 6.546
## 	dis <= 1.4254
## 	lstat > 5.91
##     then
## 	outcome = 216.66 - 123.39 dis - 1.52 lstat + 31.4 nox - 5.1 rm
## 
##   Rule 4/7: [8 cases, mean 27.56, range 22.5 to 50, est err 5.10]
## 
##     if
## 	rm <= 6.461
## 	lstat <= 5.91
##     then
## 	outcome = 93.41 - 2.79 lstat - 11.5 rm + 37.5 nox + 0.49 crim
## 	          - 0.0023 tax - 0.05 ptratio
## 
##   Rule 4/8: [118 cases, mean 32.26, range 7.5 to 50, est err 4.39]
## 
##     if
## 	rm > 6.546
##     then
## 	outcome = -12.45 - 0.0364 tax + 8.1 rm - 0.05 lstat - 0.12 dis
## 	          - 0.007 age - 0.08 ptratio - 0.02 indus - 0.01 crim
## 
##   Rule 4/9: [37 cases, mean 36.09, range 22.8 to 50, est err 4.24]
## 
##     if
## 	rm > 6.461
## 	tax > 265
## 	lstat <= 5.91
##     then
## 	outcome = 52.87 - 0.2742 b + 2.81 crim + 12 rm + 0.0478 tax - 0.151 age
## 	          - 1.73 dis - 0.06 lstat - 0.9 nox
## 
##   Rule 4/10: [33 cases, mean 37.05, range 25 to 50, est err 3.64]
## 
##     if
## 	tax <= 265
## 	lstat <= 5.91
##     then
## 	outcome = -21.76 - 0.1186 tax + 1.59 crim + 13.2 rm - 0.33 lstat
## 	          - 4.6 nox - 0.16 ptratio
## 
## Model 5:
## 
##   Rule 5/1: [324 cases, mean 19.78, range 5 to 50, est err 2.75]
## 
##     if
## 	rm <= 6.781
##     then
## 	outcome = 46.87 - 0.47 lstat - 1.05 dis - 14.9 nox - 0.78 ptratio
## 	          + 0.0104 b + 0.4 rm + 0.02 rad - 0.0006 tax
## 
##   Rule 5/2: [80 cases, mean 35.32, range 7.5 to 50, est err 4.52]
## 
##     if
## 	rm > 6.781
##     then
## 	outcome = -49.93 + 13 rm - 0.83 crim - 0.19 lstat - 0.52 dis
## 	          - 0.45 ptratio - 6.3 nox + 0.0049 b + 0.04 rad - 0.0017 tax
## 	          + 0.007 zn
## 
##   Rule 5/3: [77 cases, mean 35.75, range 22.5 to 50, est err 3.93]
## 
##     if
## 	lstat <= 5.91
##     then
## 	outcome = -10.9 + 4.64 crim - 2.66 lstat + 9.2 rm - 0.26 indus
## 	          - 0.84 dis - 0.12 ptratio + 0.02 rad - 1.5 nox - 0.0008 tax
## 	          + 0.0014 b
## 
## 
## Evaluation on training data (404 cases):
## 
##     Average  |error|               1.82
##     Relative |error|               0.26
##     Correlation coefficient        0.97
## 
## 
## 	Attribute usage:
## 	  Conds  Model
## 
## 	   62%    97%    lstat
## 	   57%    88%    rm
## 	   22%    84%    dis
## 	   16%    66%    nox
## 	   10%    68%    crim
## 	    7%    71%    tax
## 	    3%    79%    b
## 	          80%    ptratio
## 	          69%    age
## 	          31%    rad
## 	          17%    indus
## 	           5%    zn
## 
## 
## Time: 0.0 secs
```

```r
## Nearest–neighbors Adjustmemt
inst_pred <- predict(com_model, test_pred, neighbors = 5)
## RMSE
sqrt(mean((inst_pred - test_resp)^2))
```

```
## [1] 4.641258
```

```r
## R^2
cor(inst_pred, test_resp)^2
```

```
## [1] 0.6878503
```


## Optimize parameters

To tune the model over different values of neighbors and committees, the train function in the `caret package can be used to optimize these parameters. 


```r
## Optimize parameters
library("caret")
grid <- expand.grid(committees = c(1, 10, 50, 100),
                    neighbors = c(0, 1, 5, 9))
set.seed(1)
boston_tuned <- train(
  x = train_pred,
  y = train_resp,
  method = "cubist",
  tuneGrid = grid,
  trControl = trainControl(method = "cv")
)
boston_tuned
```

```
## Cubist 
## 
## 404 samples
##  13 predictor
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 364, 364, 364, 363, 364, 363, ... 
## Resampling results across tuning parameters:
## 
##   committees  neighbors  RMSE      Rsquared   MAE     
##     1         0          3.486995  0.8696332  2.410782
##     1         1          3.476423  0.8766470  2.353796
##     1         5          3.226631  0.8905880  2.172835
##     1         9          3.273334  0.8863545  2.200032
##    10         0          2.984583  0.9030420  2.107548
##    10         1          2.881682  0.9119488  2.071019
##    10         5          2.726023  0.9194164  1.907319
##    10         9          2.758086  0.9165772  1.910888
##    50         0          3.010918  0.9003286  2.114696
##    50         1          2.902492  0.9105268  2.061518
##    50         5          2.707697  0.9195486  1.881237
##    50         9          2.742850  0.9165186  1.900488
##   100         0          2.971313  0.9033629  2.093351
##   100         1          2.871736  0.9128822  2.043151
##   100         5          2.671169  0.9221820  1.862776
##   100         9          2.709607  0.9189411  1.879577
## 
## RMSE was used to select the optimal model using the smallest value.
## The final values used for the model were committees = 100 and neighbors = 5.
```

```r
## The profiles of the tuning parameters 
ggplot(boston_tuned)
```

<img src="48-Cubist-Model_files/figure-html/unnamed-chunk-6-1.png" width="672" />



## Logistic CV



```r
library("MASS")
data(biopsy)
biopsy$ID = NULL
names(biopsy) = c("thick", "u.size", "u.shape", "adhsn", 
                  "s.size", "nucl", "chrom", "n.nuc", "mit", "class")
biopsy.v2 <- na.omit(biopsy)

## 用0表示良性，用1表示恶性
y <- ifelse(biopsy.v2$class == "malignant", 1, 0)

set.seed(123) #random number generator
ind <- sample(2, nrow(biopsy.v2), replace = TRUE, prob = c(0.7, 0.3))
train <- biopsy.v2[ind==1, ] #the training data set
test <- biopsy.v2[ind==2, ] #the test data set

trainY <- y[ind==1]
testY <- y[ind==2]



set.seed(123)
ctrl=(trainControl(method="repeatedcv", repeats=5))
c<-c(100)
n<-c(3,4)

cubit.fit<-train(as.matrix(train[-10]),
                 trainY, 
                 method="cubist",
                 preProcess = c("center", "scale"),
                 tuneGrid = expand.grid(committees=c,neighbors=n),
                 trControl = ctrl)
                
summary(cubit.fit)
```

```
## 
## Call:
## cubist.default(x = x, y = y, committees = param$committees)
## 
## 
## Cubist [Release 2.07 GPL Edition]  Wed May  5 21:54:12 2021
## ---------------------------------
## 
##     Target attribute `outcome'
## 
## Read 474 cases (10 attributes) from undefined.data
## 
## Model 1:
## 
##   Rule 1/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 1/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 1/3: [287 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= -0.1056389
## 	chrom <= -0.1989626
## 	n.nuc <= 1.636011
##     then
## 	outcome = -0
## 
##   Rule 1/4: [14 cases, mean 0.5, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
## 	mit <= 0.1942086
##     then
## 	outcome = 0.1 + 0.825 u.shape - 0.551 u.size + 0.073 nucl
## 
##   Rule 1/5: [7 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	chrom > -0.1989626
## 	n.nuc <= 1.636011
## 	mit <= 0.1942086
##     then
## 	outcome = 0.5 - 0.247 mit + 0.199 nucl + 0.192 thick + 0.152 chrom
## 	          + 0.032 n.nuc + 0.015 u.size + 0.011 s.size
## 
##   Rule 1/6: [10 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > 1.176124
## 	chrom <= -0.1989626
##     then
## 	outcome = 0.8 + 0.101 nucl + 0.061 thick + 0.039 s.size + 0.039 chrom
## 	          + 0.038 n.nuc + 0.016 u.size
## 
##   Rule 1/7: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 1/8: [16 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	mit > 0.1942086
##     then
## 	outcome = 1
## 
##   Rule 1/9: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 1/10: [10 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 1/11: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 1/12: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 2:
## 
##   Rule 2/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 2/2: [277 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.4555328
##     then
## 	outcome = 0
## 
##   Rule 2/3: [305 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	nucl <= 0.6322382
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0
## 
##   Rule 2/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.466 thick
## 
##   Rule 2/5: [9 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.3977132
## 	u.size <= 0.2327753
## 	nucl > -0.4555328
## 	nucl <= 0.6322382
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.9 - 4.143 u.size + 0.019 thick + 0.015 n.nuc
## 
##   Rule 2/6: [4 cases, mean 0.8, range 0 to 1, est err 0.5]
## 
##     if
## 	u.size > 1.178508
## 	u.size <= 1.808996
## 	nucl > 0.6322382
## 	nucl <= 1.176124
##     then
## 	outcome = 7.8 - 4.401 u.size
## 
##   Rule 2/7: [6 cases, mean 0.8, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 0.2327753
## 	u.size <= 1.178508
## 	nucl > 0.6322382
## 	nucl <= 1.176124
##     then
## 	outcome = 2.5 - 2.575 u.size
## 
##   Rule 2/8: [30 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > 0.6322382
##     then
## 	outcome = 1
## 
##   Rule 2/9: [81 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 0.9913985
##     then
## 	outcome = 1
## 
##   Rule 2/10: [35 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	nucl <= 0.6322382
##     then
## 	outcome = 1
## 
##   Rule 2/11: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 2/12: [54 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.808996
##     then
## 	outcome = 1
## 
##   Rule 2/13: [78 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
## Model 3:
## 
##   Rule 3/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 3/2: [275 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= 0.2174117
## 	s.size <= 0.7219615
## 	nucl <= -0.18359
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0
## 
##   Rule 3/3: [260 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 3/4: [13 cases, mean 0.3, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape <= -0.4286895
## 	nucl > -0.7274755
##     then
## 	outcome = 0.3 + 0.567 thick + 0.397 chrom
## 
##   Rule 3/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.453 thick
## 
##   Rule 3/6: [5 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > 0.2327753
## 	nucl > 0.9041809
## 	nucl <= 1.176124
##     then
## 	outcome = 1.3 + 4.121 thick - 1.787 u.size
## 
##   Rule 3/7: [7 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size <= 0.2327753
## 	u.shape > -0.4286895
## 	s.size > 0.7219615
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.9 + 0.053 nucl + 0.03 u.size + 0.022 thick + 0.02 n.nuc
## 	          + 0.01 s.size
## 
##   Rule 3/8: [9 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	n.nuc > 0.9913985
##     then
## 	outcome = 0.5 + 0.146 nucl + 0.13 thick + 0.06 chrom + 0.06 n.nuc
## 	          + 0.043 s.size + 0.007 u.shape
## 
##   Rule 3/9: [6 cases, mean 0.8, range 0 to 1, est err 1.1]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.shape > -0.4286895
## 	u.shape <= 0.2174117
## 	s.size <= 0.7219615
## 	nucl <= -0.18359
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1.3 + 1.666 u.shape + 0.553 n.nuc - 0.249 nucl + 0.246 s.size
## 	          + 0.019 thick + 0.013 chrom
## 
##   Rule 3/10: [8 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	u.shape > 0.2174117
## 	s.size <= 0.7219615
##     then
## 	outcome = 0.9 + 0.052 u.shape + 0.044 thick + 0.038 chrom + 0.022 nucl
## 
##   Rule 3/11: [27 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape > -0.4286895
## 	nucl > -0.18359
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1
## 
##   Rule 3/12: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 3/13: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 3/14: [7 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	nucl > 0.9041809
## 	nucl <= 1.176124
##     then
## 	outcome = 0.8 + 0.086 thick + 0.013 nucl + 0.007 u.size
## 
##   Rule 3/15: [37 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	nucl <= 0.9041809
##     then
## 	outcome = 1
## 
## Model 4:
## 
##   Rule 4/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 4/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 4/3: [35 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > -0.5175617
## 	thick <= 0.1863816
## 	u.shape > -0.7517401
## 	adhsn <= 0.3774847
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = -0 + 0.113 nucl + 0.097 thick
## 
##   Rule 4/4: [35 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= -0.5175617
## 	u.shape > -0.7517401
## 	adhsn <= 0.3774847
## 	nucl <= 1.176124
##     then
## 	outcome = -0.1 + 0.682 nucl - 0.513 thick + 0.312 adhsn
## 
##   Rule 4/5: [6 cases, mean 0.2, range 0 to 1, est err 0.5]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.3 + 1.024 thick
## 
##   Rule 4/6: [48 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.3774847
## 	nucl > -0.7274755
##     then
## 	outcome = 0.9 + 0.059 nucl
## 
##   Rule 4/7: [113 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.1863816
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 4/8: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 4/9: [10 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 4/10: [14 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	n.nuc <= 1.636011
##     then
## 	outcome = 1.2 - 0.106 thick - 0.047 mit
## 
##   Rule 4/11: [13 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc > 1.636011
##     then
## 	outcome = 0.9 + 0.05 nucl + 0.023 thick + 0.011 n.nuc
## 
##   Rule 4/12: [11 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	u.size <= 0.2327753
## 	adhsn > 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 0.9 + 0.107 u.size + 0.016 nucl
## 
##   Rule 4/13: [7 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 0.7 + 0.151 nucl + 0.073 thick + 0.054 n.nuc + 0.049 u.size
## 	          + 0.044 s.size
## 
##   Rule 4/14: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 4/15: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 5:
## 
##   Rule 5/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = -0
## 
##   Rule 5/2: [269 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape <= -0.4286895
## 	adhsn <= 1.064348
##     then
## 	outcome = -0
## 
##   Rule 5/3: [277 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.4555328
##     then
## 	outcome = 0
## 
##   Rule 5/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.478 thick
## 
##   Rule 5/5: [15 cases, mean 0.7, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape > -0.4286895
## 	adhsn <= 1.064348
## 	nucl > -0.4555328
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.8 - 1.634 u.size + 0.008 u.shape
## 
##   Rule 5/6: [7 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	n.nuc > 1.313705
##     then
## 	outcome = 0.4 + 0.169 nucl + 0.125 thick + 0.061 n.nuc + 0.058 s.size
## 	          + 0.057 chrom
## 
##   Rule 5/7: [32 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.9 + 0.052 nucl + 0.039 thick + 0.019 n.nuc + 0.018 s.size
## 	          + 0.017 chrom
## 
##   Rule 5/8: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 5/9: [93 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.4555328
##     then
## 	outcome = 1
## 
##   Rule 5/10: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 5/11: [73 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 1.064348
##     then
## 	outcome = 1
## 
## Model 6:
## 
##   Rule 6/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 6/2: [24 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	chrom <= -0.6054638
## 	n.nuc <= -0.6201341
##     then
## 	outcome = 0
## 
##   Rule 6/3: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.456 thick
## 
##   Rule 6/4: [27 cases, mean 0.4, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	nucl <= 1.176124
## 	chrom > -0.6054638
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.5 - 1.196 adhsn + 0.885 nucl + 0.556 n.nuc + 0.358 thick
## 
##   Rule 6/5: [13 cases, mean 0.6, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	chrom > -0.6054638
## 	n.nuc > 0.3467855
##     then
## 	outcome = 0.4 + 0.212 u.shape + 0.193 nucl - 0.142 adhsn + 0.08 thick
## 
##   Rule 6/6: [49 cases, mean 0.7, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	n.nuc > -0.6201341
##     then
## 	outcome = 0.4 + 0.186 nucl + 0.124 u.shape + 0.093 thick + 0.082 mit
## 	          + 0.046 n.nuc + 0.037 u.size + 0.027 s.size + 0.015 chrom
## 	          + 0.006 adhsn
## 
##   Rule 6/7: [7 cases, mean 0.7, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	n.nuc > 0.9913985
##     then
## 	outcome = 0.4 + 0.32 u.shape + 0.192 nucl
## 
##   Rule 6/8: [9 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.7 + 0.095 nucl + 0.07 thick + 0.038 n.nuc + 0.03 u.shape
## 	          + 0.028 s.size + 0.02 adhsn
## 
##   Rule 6/9: [32 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 1 - 0.022 chrom
## 
##   Rule 6/10: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 6/11: [9 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	nucl > 1.176124
## 	n.nuc <= 0.3467855
##     then
## 	outcome = 0.7 + 0.212 u.shape + 0.184 mit + 0.156 nucl + 0.069 thick
## 
##   Rule 6/12: [73 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 1.064348
##     then
## 	outcome = 1
## 
##   Rule 6/13: [15 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 1.242297
## 	u.size <= 0.2327753
##     then
## 	outcome = 1 + 0.097 mit
## 
## Model 7:
## 
##   Rule 7/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 7/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 7/3: [6 cases, mean 0.2, range 0 to 1, est err 0.5]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.3 + 1.025 thick
## 
##   Rule 7/4: [16 cases, mean 0.3, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc <= -0.2978275
##     then
## 	outcome = -1.2 - 2.411 n.nuc + 0.506 chrom
## 
##   Rule 7/5: [15 cases, mean 0.3, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	chrom <= 0.2075386
## 	n.nuc > -0.2978275
## 	n.nuc <= 1.313705
##     then
## 	outcome = -0.5
## 
##   Rule 7/6: [4 cases, mean 0.8, range 0 to 1, est err 1.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	adhsn <= 0.7209161
## 	s.size <= -0.5858465
## 	nucl > -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1.5 + 0.162 thick + 0.121 nucl + 0.104 n.nuc
## 
##   Rule 7/7: [9 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	s.size > -0.5858465
## 	chrom > 0.2075386
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.8 + 0.21 nucl + 0.037 n.nuc + 0.033 thick + 0.017 u.size
## 	          + 0.017 chrom + 0.013 s.size
## 
##   Rule 7/8: [38 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 1 + 0.014 nucl + 0.008 thick + 0.005 n.nuc
## 
##   Rule 7/9: [6 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.5383533
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc > 1.313705
##     then
## 	outcome = 1.1
## 
##   Rule 7/10: [4 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1.1
## 
##   Rule 7/11: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 7/12: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 8:
## 
##   Rule 8/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 8/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 8/3: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 0.1 + 0.104 nucl + 0.059 u.size + 0.056 thick + 0.043 n.nuc
## 	          + 0.026 s.size
## 
##   Rule 8/4: [21 cases, mean 0.3, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size <= 0.7219615
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0
## 
##   Rule 8/5: [5 cases, mean 0.4, range 0 to 1, est err 0.8]
## 
##     if
## 	u.size <= -0.08246895
## 	s.size > -0.1499105
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 1.2 + 0.968 u.size + 0.391 nucl + 0.385 u.shape + 0.24 thick
## 
##   Rule 8/6: [15 cases, mean 0.5, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= -0.08246895
## 	u.shape > -0.7517401
## 	s.size <= -0.1499105
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 2 + 1.905 s.size + 1.769 u.size + 0.611 n.nuc
## 
##   Rule 8/7: [6 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > 0.5383533
## 	thick <= 1.242297
## 	u.size > -0.08246895
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 1 + 0.095 s.size
## 
##   Rule 8/8: [106 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 8/9: [71 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	s.size > 0.7219615
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 8/10: [59 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > 1.176124
##     then
## 	outcome = 0.9 + 0.02 adhsn + 0.017 nucl + 0.009 thick + 0.008 s.size
## 	          + 0.007 n.nuc
## 
##   Rule 8/11: [28 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
## 	u.shape <= 1.186563
##     then
## 	outcome = 1 - 0.016 nucl
## 
##   Rule 8/12: [8 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc > 1.636011
##     then
## 	outcome = 1 - 0.025 thick - 0.012 n.nuc
## 
##   Rule 8/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 9:
## 
##   Rule 9/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = -0
## 
##   Rule 9/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 9/3: [43 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.2 + 0.476 nucl + 0.038 thick + 0.009 n.nuc
## 
##   Rule 9/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.477 thick
## 
##   Rule 9/5: [4 cases, mean 0.5, range 0 to 1, est err 0.7]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1 + 0.299 nucl + 0.148 thick + 0.102 n.nuc
## 
##   Rule 9/6: [17 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.7 + 0.173 nucl + 0.007 thick
## 
##   Rule 9/7: [13 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size <= 0.2327753
## 	n.nuc > 0.9913985
##     then
## 	outcome = -3.2 + 1.833 n.nuc + 0.299 nucl
## 
##   Rule 9/8: [32 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.9 + 0.049 nucl + 0.032 thick + 0.02 n.nuc + 0.016 chrom
## 	          + 0.012 s.size
## 
##   Rule 9/9: [26 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
##     then
## 	outcome = 1 + 0.014 nucl
## 
##   Rule 9/10: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
## Model 10:
## 
##   Rule 10/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 10/2: [235 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.4286895
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 10/3: [35 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= 0.2174117
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.4 + 3.395 u.shape
## 
##   Rule 10/4: [4 cases, mean 0.2, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape > 0.2174117
## 	u.shape <= 1.186563
## 	adhsn <= 0.03405333
## 	chrom <= 0.2075386
## 	n.nuc > -0.2978275
## 	mit <= -0.3671017
##     then
## 	outcome = 0.1 + 0.135 nucl - 0.095 u.shape + 0.077 thick + 0.044 n.nuc
## 	          + 0.035 s.size + 0.034 chrom
## 
##   Rule 10/5: [6 cases, mean 0.3, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.08246895
## 	u.size <= 0.2327753
## 	u.shape <= 0.2174117
## 	adhsn <= 0.7209161
##     then
## 	outcome = -0.1 + 0.348 u.shape + 0.07 chrom + 0.066 thick + 0.058 nucl
## 	          + 0.039 s.size
## 
##   Rule 10/6: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.536 thick
## 
##   Rule 10/7: [22 cases, mean 0.5, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= -0.08246895
## 	u.shape <= 0.2174117
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 0.4 + 0.826 u.shape - 0.395 adhsn + 0.144 thick + 0.02 chrom
## 	          + 0.016 nucl + 0.011 s.size
## 
##   Rule 10/8: [23 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.2174117
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 1
## 
##   Rule 10/9: [40 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
##     then
## 	outcome = 1 + 0.008 u.size
## 
##   Rule 10/10: [15 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	u.shape <= 0.2174117
##     then
## 	outcome = 1
## 
##   Rule 10/11: [5 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.594268
## 	u.shape <= 0.2174117
## 	adhsn <= 0.7209161
##     then
## 	outcome = 1 + 0.032 u.shape - 0.025 adhsn + 0.012 thick
## 
##   Rule 10/12: [10 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.03405333
## 	chrom <= 0.2075386
## 	n.nuc > -0.2978275
## 	mit <= -0.3671017
##     then
## 	outcome = 0.7 + 0.129 nucl + 0.074 thick + 0.042 n.nuc + 0.033 s.size
## 	          + 0.032 chrom + 0.018 u.shape
## 
##   Rule 10/13: [59 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.2174117
## 	mit > -0.3671017
##     then
## 	outcome = 1
## 
##   Rule 10/14: [88 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.2174117
## 	chrom > 0.2075386
##     then
## 	outcome = 1
## 
##   Rule 10/15: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 11:
## 
##   Rule 11/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 11/2: [202 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.16559
## 	u.shape <= -0.4286895
## 	nucl <= -0.4555328
##     then
## 	outcome = -0
## 
##   Rule 11/3: [32 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= -0.4286895
## 	nucl <= -0.4555328
##     then
## 	outcome = 0.1 + 0.35 thick + 0.288 u.size + 0.128 chrom
## 
##   Rule 11/4: [8 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= -0.16559
## 	u.size > -0.7129574
## 	u.shape <= 0.2174117
## 	nucl > -0.4555328
## 	nucl <= 0.9041809
##     then
## 	outcome = -0
## 
##   Rule 11/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.417 thick
## 
##   Rule 11/6: [10 cases, mean 0.4, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape > -0.4286895
## 	u.shape <= 0.2174117
## 	nucl <= -0.4555328
##     then
## 	outcome = 0.7 + 2.962 u.shape + 0.635 nucl
## 
##   Rule 11/7: [5 cases, mean 0.4, range 0 to 1, est err 0.6]
## 
##     if
## 	u.size <= 1.493752
## 	u.shape > 0.2174117
## 	adhsn <= 0.03405333
## 	chrom <= 0.2075386
## 	mit <= -0.3671017
##     then
## 	outcome = -0
## 
##   Rule 11/8: [37 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > -0.16559
## 	u.shape <= 0.2174117
## 	nucl > -0.4555328
##     then
## 	outcome = 1 + 0.027 thick
## 
##   Rule 11/9: [123 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	adhsn > 0.03405333
##     then
## 	outcome = 1
## 
##   Rule 11/10: [25 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= 0.2174117
## 	nucl > 0.9041809
##     then
## 	outcome = 3.3 - 1.348 nucl + 0.015 u.shape + 0.007 thick + 0.007 n.nuc
## 
##   Rule 11/11: [118 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	chrom > 0.2075386
##     then
## 	outcome = 1
## 
##   Rule 11/12: [59 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.493752
##     then
## 	outcome = 1
## 
##   Rule 11/13: [59 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.2174117
## 	mit > -0.3671017
##     then
## 	outcome = 1
## 
## Model 12:
## 
##   Rule 12/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 12/2: [270 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	adhsn <= 0.7209161
## 	nucl <= -0.4555328
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0
## 
##   Rule 12/3: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 12/4: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.2 + 1.014 thick
## 
##   Rule 12/5: [11 cases, mean 0.2, range 0 to 1, est err 0.2]
## 
##     if
## 	thick > -0.16559
## 	thick <= 0.5383533
## 	u.shape <= 0.5404623
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
## 	n.nuc <= 1.313705
##     then
## 	outcome = -1
## 
##   Rule 12/6: [13 cases, mean 0.4, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= -0.16559
## 	u.shape > -0.7517401
## 	u.shape <= 0.5404623
## 	adhsn <= 0.7209161
## 	nucl > -0.4555328
##     then
## 	outcome = -0.8 - 1.778 thick + 0.789 nucl
## 
##   Rule 12/7: [21 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > -0.16559
## 	u.shape <= 1.186563
## 	adhsn <= -0.3093781
## 	nucl > -0.4555328
##     then
## 	outcome = 1 + 0.007 nucl
## 
##   Rule 12/8: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.5404623
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 12/9: [71 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.313705
##     then
## 	outcome = 1
## 
##   Rule 12/10: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 12/11: [11 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	chrom <= -0.1989626
##     then
## 	outcome = 0.9 + 0.022 nucl + 0.012 thick + 0.009 n.nuc + 0.008 u.size
## 	          + 0.005 s.size
## 
##   Rule 12/12: [34 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
##     then
## 	outcome = 1 + 0.012 thick
## 
##   Rule 12/13: [4 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1.1 - 0.183 u.shape
## 
##   Rule 12/14: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 12/15: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 13:
## 
##   Rule 13/1: [243 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= -0.18359
##     then
## 	outcome = 0
## 
##   Rule 13/2: [265 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.4555328
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0
## 
##   Rule 13/3: [8 cases, mean 0.1, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size <= -0.7129574
## 	s.size <= -0.5858465
## 	nucl > -0.18359
##     then
## 	outcome = -2 - 3.465 s.size
## 
##   Rule 13/4: [12 cases, mean 0.2, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > -0.18359
##     then
## 	outcome = 0.3 + 2.51 s.size
## 
##   Rule 13/5: [10 cases, mean 0.5, range 0 to 1, est err 0.6]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 0.6322382
## 	n.nuc > -0.2978275
## 	n.nuc <= 0.02447898
##     then
## 	outcome = 1.4 + 1.852 u.size + 1.58 nucl - 1.184 chrom
## 
##   Rule 13/6: [21 cases, mean 0.6, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.4555328
## 	nucl <= 0.6322382
##     then
## 	outcome = 0.2 + 0.565 thick - 0.377 nucl + 0.007 n.nuc + 0.005 chrom
## 
##   Rule 13/7: [11 cases, mean 0.7, range 0 to 1, est err 0.5]
## 
##     if
## 	s.size <= 0.7219615
## 	nucl > 0.6322382
## 	nucl <= 1.176124
##     then
## 	outcome = 0.8 - 1.456 s.size
## 
##   Rule 13/8: [8 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size <= 0.2327753
## 	n.nuc > 0.02447898
## 	n.nuc <= 0.669092
##     then
## 	outcome = 1
## 
##   Rule 13/9: [10 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= -0.08246895
## 	nucl > 1.176124
##     then
## 	outcome = 0.8 + 0.091 nucl + 0.061 thick + 0.049 u.size + 0.035 n.nuc
## 
##   Rule 13/10: [95 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	n.nuc > 0.669092
##     then
## 	outcome = 1
## 
##   Rule 13/11: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 13/12: [35 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	nucl <= 0.6322382
##     then
## 	outcome = 1
## 
##   Rule 13/13: [11 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	s.size > 0.7219615
## 	nucl > 0.6322382
## 	nucl <= 1.176124
##     then
## 	outcome = 0.7 + 0.072 nucl + 0.062 thick + 0.033 n.nuc + 0.028 s.size
## 	          + 0.022 chrom
## 
## Model 14:
## 
##   Rule 14/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 14/2: [18 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	n.nuc > -0.6201341
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0
## 
##   Rule 14/3: [270 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	adhsn <= 0.7209161
## 	nucl <= -0.4555328
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0
## 
##   Rule 14/4: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 14/5: [300 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	chrom <= 0.2075386
## 	n.nuc <= 1.313705
##     then
## 	outcome = -0
## 
##   Rule 14/6: [6 cases, mean 0.7, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > 0.5404623
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.7 + 0.162 nucl
## 
##   Rule 14/7: [106 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 14/8: [52 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	adhsn > 0.3774847
##     then
## 	outcome = 1
## 
##   Rule 14/9: [107 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > -0.4555328
## 	chrom > 0.2075386
##     then
## 	outcome = 1
## 
##   Rule 14/10: [71 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.313705
##     then
## 	outcome = 1
## 
##   Rule 14/11: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 14/12: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 14/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 15:
## 
##   Rule 15/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 15/2: [213 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.16559
## 	u.shape <= -0.1056389
## 	nucl <= -0.4555328
##     then
## 	outcome = 0
## 
##   Rule 15/3: [231 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= -0.16559
## 	u.shape <= -0.1056389
## 	nucl <= 1.176124
## 	chrom <= 0.6140398
##     then
## 	outcome = 0
## 
##   Rule 15/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.466 thick
## 
##   Rule 15/5: [25 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > -0.16559
## 	u.size > -0.7129574
## 	u.shape <= -0.1056389
## 	nucl <= 1.176124
##     then
## 	outcome = 0.2 + 0.276 thick + 0.259 nucl + 0.216 u.size + 0.053 n.nuc
## 	          - 0.035 u.shape + 0.022 mit
## 
##   Rule 15/6: [15 cases, mean 0.5, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.1056389
## 	nucl > -0.4555328
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
## 	mit <= 0.1942086
##     then
## 	outcome = -0.2 + 1.716 u.shape - 1.278 u.size
## 
##   Rule 15/7: [5 cases, mean 0.8, range 0 to 1, est err 1.5]
## 
##     if
## 	thick > -0.16559
## 	thick <= 1.242297
## 	u.shape <= -0.1056389
## 	nucl > -0.4555328
## 	nucl <= 1.176124
## 	chrom <= 0.6140398
## 	n.nuc <= 1.636011
##     then
## 	outcome = 2.2 + 1.727 u.shape - 0.1 chrom + 0.049 nucl + 0.034 n.nuc
## 	          + 0.032 thick + 0.011 u.size + 0.009 mit
## 
##   Rule 15/8: [6 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape > -0.1056389
## 	u.shape <= 0.8635129
## 	nucl <= -0.4555328
##     then
## 	outcome = 0.9 + 0.113 u.size + 0.038 n.nuc
## 
##   Rule 15/9: [10 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= -0.08246895
## 	nucl > 1.176124
##     then
## 	outcome = 0.9 + 0.061 nucl + 0.041 thick + 0.03 u.size + 0.021 chrom
## 	          + 0.017 n.nuc
## 
##   Rule 15/10: [17 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape > -0.1056389
## 	nucl <= -0.4555328
##     then
## 	outcome = 1
## 
##   Rule 15/11: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 15/12: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 15/13: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 15/14: [11 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.1056389
## 	nucl > -0.4555328
## 	chrom > 0.6140398
##     then
## 	outcome = 1
## 
##   Rule 15/15: [5 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	nucl > -0.4555328
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
## 	mit > 0.1942086
##     then
## 	outcome = 1 + 0.017 n.nuc
## 
## Model 16:
## 
##   Rule 16/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 16/2: [260 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 16/3: [14 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
##     then
## 	outcome = -0
## 
##   Rule 16/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.453 thick
## 
##   Rule 16/5: [17 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	s.size <= -0.1499105
## 	nucl > -0.7274755
##     then
## 	outcome = 1.4 + 2.348 s.size
## 
##   Rule 16/6: [5 cases, mean 0.6, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > 0.2327753
## 	nucl > 0.9041809
## 	nucl <= 1.176124
##     then
## 	outcome = 1.4 + 5.906 thick - 2.375 u.size
## 
##   Rule 16/7: [9 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= -0.3093781
## 	s.size > -0.1499105
## 	nucl > -0.7274755
##     then
## 	outcome = 0.6 + 0.196 s.size + 0.113 nucl + 0.016 thick
## 
##   Rule 16/8: [12 cases, mean 0.9, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > 0.5383533
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 1.1 - 0.648 u.size - 0.459 nucl + 0.026 thick
## 
##   Rule 16/9: [26 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
##     then
## 	outcome = 1 + 0.048 u.size
## 
##   Rule 16/10: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 16/11: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 16/12: [7 cases, mean 1.0, range 1 to 1, est err 0.5]
## 
##     if
## 	thick > 0.5383533
## 	nucl > 0.9041809
## 	nucl <= 1.176124
##     then
## 	outcome = 2.3 - 0.658 thick + 0.007 nucl
## 
##   Rule 16/13: [12 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	adhsn > 0.7209161
##     then
## 	outcome = 1 + 0.059 u.size
## 
## Model 17:
## 
##   Rule 17/1: [235 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.4286895
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 17/2: [118 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.8695334
## 	u.shape <= -0.4286895
##     then
## 	outcome = -0
## 
##   Rule 17/3: [23 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape > -0.4286895
## 	u.shape <= 1.186563
## 	s.size <= 0.2860255
## 	nucl <= 1.176124
## 	n.nuc <= -0.2978275
##     then
## 	outcome = -0
## 
##   Rule 17/4: [17 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > -0.8695334
## 	u.shape <= -0.4286895
## 	s.size <= -0.5858465
## 	nucl > -0.7274755
##     then
## 	outcome = -2 - 3.587 s.size + 0.359 thick
## 
##   Rule 17/5: [41 cases, mean 0.2, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape <= -0.4286895
## 	nucl > -0.7274755
##     then
## 	outcome = 0.3 + 0.36 thick + 0.174 s.size + 0.066 nucl + 0.01 u.size
## 	          + 0.005 n.nuc
## 
##   Rule 17/6: [26 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 1.178508
## 	u.shape > -0.4286895
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.5 + 0.254 n.nuc + 0.189 adhsn
## 
##   Rule 17/7: [13 cases, mean 0.8, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= -0.08246895
## 	u.shape > -0.4286895
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1.1 + 0.256 n.nuc + 0.19 adhsn
## 
##   Rule 17/8: [43 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	u.shape > -0.4286895
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1 - 0.008 nucl
## 
##   Rule 17/9: [9 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size <= -0.08246895
## 	u.shape > -0.4286895
## 	nucl > 1.176124
##     then
## 	outcome = 0.7 + 0.122 nucl + 0.073 u.size + 0.064 thick + 0.039 n.nuc
## 	          + 0.023 s.size + 0.022 chrom
## 
##   Rule 17/10: [145 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.4286895
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1
## 
##   Rule 17/11: [104 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.4286895
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 17/12: [99 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.4286895
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 17/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 18:
## 
##   Rule 18/1: [205 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 18/2: [160 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.5175617
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 18/3: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 18/4: [7 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 0.5404623
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
## 	n.nuc <= 0.02447898
##     then
## 	outcome = -0.2
## 
##   Rule 18/5: [72 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > -0.5175617
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0.6 + 1.522 nucl
## 
##   Rule 18/6: [15 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= -0.3093781
## 	nucl > -0.7274755
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.3 + 0.317 nucl + 0.128 thick
## 
##   Rule 18/7: [8 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape <= 0.5404623
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
## 	n.nuc > 0.02447898
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.9
## 
##   Rule 18/8: [106 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 18/9: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.3774847
##     then
## 	outcome = 1
## 
##   Rule 18/10: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.5404623
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 18/11: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 18/12: [6 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.5383533
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc > 1.313705
##     then
## 	outcome = 0.9 + 0.058 nucl + 0.005 thick
## 
##   Rule 18/13: [38 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 1 + 0.01 nucl
## 
##   Rule 18/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 19:
## 
##   Rule 19/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 19/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 19/3: [227 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= -0.16559
## 	adhsn <= 0.03405333
## 	s.size <= -0.1499105
## 	chrom <= 0.2075386
##     then
## 	outcome = -0 + 0.18 chrom
## 
##   Rule 19/4: [16 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > -0.4555328
##     then
## 	outcome = 0.1 + 0.069 nucl + 0.036 thick + 0.024 u.size + 0.022 s.size
## 	          + 0.021 u.shape + 0.021 chrom
## 
##   Rule 19/5: [42 cases, mean 0.2, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	adhsn <= 0.03405333
## 	nucl > -0.7274755
## 	chrom <= 0.2075386
##     then
## 	outcome = -0.1 + 0.373 thick
## 
##   Rule 19/6: [6 cases, mean 0.5, range 0 to 1, est err 0.7]
## 
##     if
## 	thick <= 0.5383533
## 	adhsn <= 0.03405333
## 	s.size > -0.1499105
## 	nucl > -0.7274755
## 	chrom > -0.6054638
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.6 + 0.251 nucl + 0.008 thick
## 
##   Rule 19/7: [4 cases, mean 0.8, range 0 to 1, est err 1.4]
## 
##     if
## 	thick > -0.16559
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	adhsn <= 0.03405333
## 	s.size <= -0.1499105
## 	nucl > -0.7274755
## 	chrom > -0.6054638
##     then
## 	outcome = 1.6
## 
##   Rule 19/8: [4 cases, mean 0.8, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	nucl <= 0.9041809
## 	chrom > 0.2075386
##     then
## 	outcome = 0.9 + 0.129 nucl + 0.07 thick + 0.057 s.size + 0.044 u.shape
## 	          + 0.041 chrom + 0.032 u.size
## 
##   Rule 19/9: [15 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > 0.9041809
##     then
## 	outcome = 0.9 + 0.095 nucl
## 
##   Rule 19/10: [117 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.03405333
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 19/11: [106 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 19/12: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.3774847
##     then
## 	outcome = 1
## 
##   Rule 19/13: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 19/14: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 19/15: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 20:
## 
##   Rule 20/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 20/2: [202 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.16559
## 	adhsn <= -0.6528094
##     then
## 	outcome = 0
## 
##   Rule 20/3: [14 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	adhsn > -0.6528094
## 	adhsn <= 0.7209161
## 	nucl <= 0.6322382
## 	n.nuc <= 0.02447898
## 	mit <= -0.3671017
##     then
## 	outcome = 0
## 
##   Rule 20/4: [260 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 20/5: [7 cases, mean 0.1, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 1.594268
## 	u.size <= 0.2327753
## 	adhsn > -0.6528094
## 	adhsn <= 0.7209161
## 	mit > -0.3671017
##     then
## 	outcome = -0.8 + 0.091 nucl + 0.06 thick + 0.021 n.nuc + 0.017 mit
## 
##   Rule 20/6: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.474 thick
## 
##   Rule 20/7: [6 cases, mean 0.7, range 0 to 1, est err 0.5]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
## 	nucl <= 0.6322382
## 	n.nuc > 0.02447898
## 	mit <= -0.3671017
##     then
## 	outcome = 0.7 + 0.255 thick + 0.027 nucl
## 
##   Rule 20/8: [67 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 0.6322382
## 	mit <= -0.3671017
##     then
## 	outcome = 1
## 
##   Rule 20/9: [59 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.493752
##     then
## 	outcome = 1
## 
##   Rule 20/10: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 20/11: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 20/12: [50 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.594268
##     then
## 	outcome = 1
## 
##   Rule 20/13: [11 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	thick > -0.16559
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= -0.6528094
## 	nucl > -0.7274755
##     then
## 	outcome = 1.2 - 0.137 thick - 0.109 nucl + 0.099 n.nuc
## 
##   Rule 20/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 21:
## 
##   Rule 21/1: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 21/2: [47 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= -0.16559
## 	u.shape > -0.7517401
## 	nucl <= 0.9041809
##     then
## 	outcome = 0
## 
##   Rule 21/3: [25 cases, mean 0.2, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > -0.16559
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl <= 0.9041809
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.7 + 0.697 nucl + 0.289 s.size
## 
##   Rule 21/4: [6 cases, mean 0.2, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.2 + 0.998 thick
## 
##   Rule 21/5: [26 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size <= 0.2327753
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl <= 0.9041809
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.8 + 0.117 nucl + 0.077 thick + 0.049 u.size + 0.042 n.nuc
## 	          + 0.03 s.size
## 
##   Rule 21/6: [11 cases, mean 0.7, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > 0.9041809
## 	chrom <= 0.2075386
##     then
## 	outcome = -3.4 + 2.765 nucl - 1.71 thick
## 
##   Rule 21/7: [105 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1
## 
##   Rule 21/8: [13 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	u.shape <= 1.186563
## 	nucl > 0.9041809
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.6 + 0.238 nucl + 0.019 thick + 0.012 u.size + 0.012 s.size
## 	          + 0.011 n.nuc
## 
##   Rule 21/9: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	nucl > 0.9041809
## 	chrom > 0.2075386
##     then
## 	outcome = 1
## 
##   Rule 21/10: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 22:
## 
##   Rule 22/1: [226 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 22/2: [34 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	u.shape > -0.7517401
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0 + 0.012 s.size - 0.007 u.size + 0.006 nucl
## 
##   Rule 22/3: [54 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > -0.16559
## 	u.shape <= 1.186563
## 	s.size <= -0.5858465
## 	nucl <= 1.176124
## 	n.nuc <= -0.2978275
##     then
## 	outcome = -0
## 
##   Rule 22/4: [39 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= -0.16559
## 	u.shape > -0.7517401
## 	nucl <= 1.176124
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0
## 
##   Rule 22/5: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.1 + 0.932 thick
## 
##   Rule 22/6: [10 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > -0.5858465
## 	nucl <= 1.176124
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.2 - 1.22 u.shape + 0.786 nucl + 0.677 s.size
## 
##   Rule 22/7: [20 cases, mean 0.6, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.890325
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
## 	n.nuc <= 1.636011
## 	mit <= 0.1942086
##     then
## 	outcome = 0.4 + 0.558 adhsn + 0.431 nucl
## 
##   Rule 22/8: [8 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > 0.890325
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
## 	mit <= 0.1942086
##     then
## 	outcome = 0.8 + 0.107 nucl + 0.055 thick + 0.047 chrom + 0.044 s.size
## 	          + 0.029 n.nuc
## 
##   Rule 22/9: [10 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > 1.176124
## 	chrom <= -0.1989626
##     then
## 	outcome = 0.7 + 0.134 nucl + 0.07 thick + 0.059 chrom + 0.056 s.size
## 	          + 0.037 n.nuc
## 
##   Rule 22/10: [55 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > -0.2978275
## 	mit > 0.1942086
##     then
## 	outcome = 1
## 
##   Rule 22/11: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 22/12: [4 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl > 1.176124
## 	nucl <= 1.448066
## 	chrom > -0.1989626
##     then
## 	outcome = 0.5 + 0.145 nucl + 0.107 chrom + 0.084 thick + 0.043 n.nuc
## 	          + 0.042 s.size + 0.02 u.size
## 
##   Rule 22/13: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 22/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 23:
## 
##   Rule 23/1: [239 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.5858465
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 23/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 23/3: [57 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.1 + 0.473 s.size
## 
##   Rule 23/4: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 0.1 + 0.078 nucl + 0.038 thick + 0.034 u.size + 0.031 n.nuc
## 	          + 0.021 s.size + 0.016 u.shape
## 
##   Rule 23/5: [14 cases, mean 0.3, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= -0.16559
## 	u.shape > -0.7517401
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = -0.5 - 1.077 thick + 0.937 nucl
## 
##   Rule 23/6: [7 cases, mean 0.6, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	adhsn > 0.3774847
## 	adhsn <= 0.7209161
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.6 + 0.115 nucl
## 
##   Rule 23/7: [14 cases, mean 0.6, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > -0.16559
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0 + 1.166 thick
## 
##   Rule 23/8: [123 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 0.6322382
##     then
## 	outcome = 1
## 
##   Rule 23/9: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 23/10: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 23/11: [6 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
## 	nucl <= 0.6322382
##     then
## 	outcome = 0.9 + 0.046 nucl + 0.022 thick + 0.02 u.size + 0.018 n.nuc
## 	          + 0.012 s.size + 0.009 u.shape
## 
##   Rule 23/12: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 23/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 24:
## 
##   Rule 24/1: [225 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 24/2: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = -0
## 
##   Rule 24/3: [252 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.5858465
## 	nucl <= -0.4555328
##     then
## 	outcome = -0
## 
##   Rule 24/4: [8 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape <= -0.4286895
## 	s.size > -0.5858465
## 	nucl <= -0.4555328
##     then
## 	outcome = 2.7 + 3.738 nucl
## 
##   Rule 24/5: [12 cases, mean 0.3, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	adhsn <= 1.064348
## 	nucl > -0.4555328
## 	chrom <= -0.1989626
##     then
## 	outcome = 0.1 - 0.337 u.size + 0.186 u.shape + 0.097 nucl
## 
##   Rule 24/6: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.476 thick
## 
##   Rule 24/7: [12 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	nucl > -0.4555328
## 	chrom > -0.1989626
##     then
## 	outcome = 0.9 - 0.174 u.size + 0.096 u.shape + 0.05 nucl
## 
##   Rule 24/8: [32 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.8 + 0.072 nucl + 0.047 thick + 0.026 n.nuc + 0.021 u.shape
## 	          + 0.018 s.size + 0.017 chrom
## 
##   Rule 24/9: [162 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape > -0.4286895
## 	s.size > -0.5858465
##     then
## 	outcome = 1
## 
##   Rule 24/10: [81 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 0.9913985
##     then
## 	outcome = 1
## 
##   Rule 24/11: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 24/12: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 24/13: [73 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 1.064348
##     then
## 	outcome = 1
## 
##   Rule 24/14: [7 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	nucl > -0.4555328
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.8 + 0.115 nucl + 0.067 n.nuc - 0.063 u.size + 0.035 u.shape
## 
## Model 25:
## 
##   Rule 25/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 25/2: [35 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.1 + 0.062 nucl + 0.034 thick + 0.023 n.nuc + 0.01 s.size
## 	          + 0.009 chrom
## 
##   Rule 25/3: [43 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.2 + 0.483 nucl + 0.026 thick + 0.008 n.nuc
## 
##   Rule 25/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.452 thick
## 
##   Rule 25/5: [17 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.7 + 0.18 nucl + 0.006 thick + 0.005 n.nuc
## 
##   Rule 25/6: [13 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size <= 0.2327753
## 	n.nuc > 0.9913985
##     then
## 	outcome = -3.6 + 2.014 n.nuc + 0.321 nucl
## 
##   Rule 25/7: [32 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.9 + 0.042 nucl + 0.031 thick + 0.017 n.nuc + 0.014 s.size
## 	          + 0.013 chrom
## 
##   Rule 25/8: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 25/9: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
## Model 26:
## 
##   Rule 26/1: [200 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
## 	adhsn <= -0.6528094
##     then
## 	outcome = -0
## 
##   Rule 26/2: [203 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl <= 0.08835271
## 	chrom <= -0.6054638
##     then
## 	outcome = -0
## 
##   Rule 26/3: [265 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= -0.1056389
## 	nucl <= -0.4555328
## 	chrom <= 0.2075386
##     then
## 	outcome = 0 + 0.059 nucl + 0.044 chrom + 0.016 thick
## 
##   Rule 26/4: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 26/5: [32 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
## 	adhsn > -0.6528094
##     then
## 	outcome = 0.4 + 1.943 nucl
## 
##   Rule 26/6: [14 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	adhsn > -0.3093781
## 	nucl <= 1.176124
## 	chrom > -0.6054638
## 	n.nuc <= 1.636011
##     then
## 	outcome = -0.1 + 0.617 adhsn
## 
##   Rule 26/7: [5 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 0.08835271
## 	chrom <= -0.6054638
##     then
## 	outcome = 0.4 + 0.34 nucl
## 
##   Rule 26/8: [13 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl > -0.4555328
## 	nucl <= 1.176124
## 	chrom > -0.6054638
##     then
## 	outcome = 1.1
## 
##   Rule 26/9: [7 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl <= 1.176124
## 	chrom > 0.2075386
##     then
## 	outcome = 0.8 + 0.316 nucl + 0.197 thick + 0.05 n.nuc + 0.037 adhsn
## 	          + 0.021 u.size + 0.012 s.size
## 
##   Rule 26/10: [150 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.1056389
## 	chrom > -0.6054638
##     then
## 	outcome = 1
## 
##   Rule 26/11: [42 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > 1.176124
##     then
## 	outcome = 0.8 + 0.078 nucl + 0.047 thick + 0.029 u.size + 0.028 n.nuc
## 	          + 0.016 chrom
## 
##   Rule 26/12: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 26/13: [28 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
## 	u.shape <= 1.186563
##     then
## 	outcome = 2.2 - 0.646 thick + 0.023 nucl + 0.011 u.size + 0.007 n.nuc
## 
##   Rule 26/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 27:
## 
##   Rule 27/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 27/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 27/3: [9 cases, mean 0.1, range 0 to 1, est err 0.2]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > -0.18359
##     then
## 	outcome = 0.2 + 0.141 nucl + 0.076 thick + 0.063 u.shape + 0.055 n.nuc
## 	          + 0.037 s.size + 0.035 chrom + 0.011 adhsn
## 
##   Rule 27/4: [32 cases, mean 0.5, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.4 + 0.354 chrom + 0.049 nucl + 0.015 thick + 0.012 s.size
## 	          + 0.011 n.nuc
## 
##   Rule 27/5: [31 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	n.nuc > 1.313705
##     then
## 	outcome = 1 + 0.012 nucl
## 
##   Rule 27/6: [54 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
##     then
## 	outcome = 1 + 0.01 nucl + 0.009 thick
## 
##   Rule 27/7: [4 cases, mean 1.0, range 1 to 1, est err 0.3]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.9 + 0.13 s.size + 0.093 nucl + 0.04 thick + 0.03 n.nuc
## 
##   Rule 27/8: [40 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
##     then
## 	outcome = 0.9 + 0.045 nucl
## 
##   Rule 27/9: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 28:
## 
##   Rule 28/1: [225 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 28/2: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 28/3: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 28/4: [241 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= -0.16559
## 	u.size <= 0.2327753
## 	nucl <= 0.9041809
##     then
## 	outcome = -0.1 + 0.145 u.shape + 0.135 nucl + 0.083 n.nuc + 0.06 thick
## 
##   Rule 28/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.481 thick
## 
##   Rule 28/6: [6 cases, mean 0.7, range 0 to 1, est err 1.2]
## 
##     if
## 	thick > -0.16559
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
## 	nucl <= 0.9041809
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1.5
## 
##   Rule 28/7: [7 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 0.9041809
## 	n.nuc > 0.9913985
##     then
## 	outcome = 0.6 + 0.151 thick + 0.104 u.shape + 0.097 nucl
## 
##   Rule 28/8: [16 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > -0.16559
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
##     then
## 	outcome = 1.8 - 4.311 thick
## 
##   Rule 28/9: [12 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size > -0.7129574
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 28/10: [48 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > -0.16559
## 	u.size > -0.7129574
## 	nucl > -0.7274755
## 	nucl <= 0.9041809
##     then
## 	outcome = 1 - 0.01 nucl - 0.009 thick
## 
##   Rule 28/11: [32 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.9 + 0.048 nucl + 0.038 thick + 0.023 chrom + 0.019 s.size
## 	          + 0.014 n.nuc
## 
##   Rule 28/12: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 28/13: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 28/14: [5 cases, mean 1.0, range 1 to 1, est err 0.3]
## 
##     if
## 	thick <= -0.16559
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > 0.9041809
##     then
## 	outcome = 0.9 + 0.169 u.shape + 0.158 nucl
## 
## Model 29:
## 
##   Rule 29/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 29/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 29/3: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.438 thick
## 
##   Rule 29/4: [15 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape <= 0.2174117
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
##     then
## 	outcome = 0.4 + 2.711 u.shape - 1.289 s.size
## 
##   Rule 29/5: [5 cases, mean 0.4, range 0 to 1, est err 0.6]
## 
##     if
## 	u.size <= 1.493752
## 	u.shape > 0.2174117
## 	adhsn <= 0.03405333
## 	chrom <= 0.2075386
## 	mit <= -0.3671017
##     then
## 	outcome = -0.1 + 0.099 nucl + 0.071 thick + 0.037 u.shape + 0.036 chrom
## 	          + 0.035 n.nuc + 0.025 s.size
## 
##   Rule 29/6: [23 cases, mean 0.7, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= 0.2174117
## 	adhsn <= -0.3093781
## 	nucl > -0.7274755
##     then
## 	outcome = -0 - 0.96 adhsn + 0.267 thick + 0.222 n.nuc + 0.131 u.shape
## 	          + 0.039 nucl + 0.016 s.size
## 
##   Rule 29/7: [21 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= 0.2174117
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 29/8: [59 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.493752
##     then
## 	outcome = 1
## 
##   Rule 29/9: [96 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.2174117
## 	adhsn > 0.03405333
##     then
## 	outcome = 1
## 
##   Rule 29/10: [112 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	adhsn > -0.3093781
##     then
## 	outcome = 1
## 
##   Rule 29/11: [59 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.2174117
## 	mit > -0.3671017
##     then
## 	outcome = 1
## 
##   Rule 29/12: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 29/13: [88 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.2174117
## 	chrom > 0.2075386
##     then
## 	outcome = 1
## 
## Model 30:
## 
##   Rule 30/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 30/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.1499105
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 30/3: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.473 thick
## 
##   Rule 30/4: [14 cases, mean 0.5, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	s.size > -0.1499105
## 	nucl <= 0.9041809
##     then
## 	outcome = 0.4 + 0.165 nucl + 0.134 thick + 0.079 chrom + 0.065 u.shape
## 	          + 0.044 n.nuc + 0.031 s.size
## 
##   Rule 30/5: [22 cases, mean 0.6, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	s.size <= -0.1499105
## 	nucl > -0.7274755
## 	nucl <= 0.9041809
##     then
## 	outcome = 1 + 1.291 s.size + 0.191 nucl + 0.063 thick
## 
##   Rule 30/6: [32 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.8 + 0.055 nucl + 0.05 thick + 0.025 chrom + 0.019 n.nuc
## 	          + 0.015 u.shape + 0.013 s.size
## 
##   Rule 30/7: [115 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 0.9041809
##     then
## 	outcome = 1
## 
##   Rule 30/8: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
## Model 31:
## 
##   Rule 31/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 31/2: [222 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= -0.16559
## 	u.shape <= -0.4286895
##     then
## 	outcome = -0
## 
##   Rule 31/3: [16 cases, mean 0.3, range 0 to 1, est err 0.2]
## 
##     if
## 	thick > -0.16559
## 	u.size > -0.7129574
## 	u.shape <= -0.4286895
##     then
## 	outcome = 0.1 + 0.598 thick + 0.392 chrom
## 
##   Rule 31/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.398 thick
## 
##   Rule 31/5: [21 cases, mean 0.5, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.shape > -0.4286895
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl <= 1.176124
## 	n.nuc <= 0.669092
##     then
## 	outcome = 0.9 - 1.192 u.size + 0.767 nucl - 0.76 u.shape + 0.696 n.nuc
## 	          + 0.34 thick
## 
##   Rule 31/6: [8 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.4286895
## 	adhsn <= 0.7209161
## 	nucl <= 1.176124
## 	n.nuc > 0.669092
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.7 + 0.168 nucl + 0.109 thick - 0.077 u.size + 0.045 n.nuc
## 	          + 0.04 u.shape + 0.032 chrom
## 
##   Rule 31/7: [99 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.4286895
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 31/8: [89 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	nucl > 1.176124
## 	chrom > -0.1989626
##     then
## 	outcome = 1
## 
##   Rule 31/9: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 31/10: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 31/11: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 31/12: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 32:
## 
##   Rule 32/1: [277 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.4555328
##     then
## 	outcome = 0
## 
##   Rule 32/2: [13 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size <= 0.2327753
## 	s.size <= -0.5858465
## 	nucl > -0.4555328
## 	nucl <= 0.08835271
##     then
## 	outcome = 0.1 + 0.273 thick + 0.012 u.shape - 0.006 adhsn
## 
##   Rule 32/3: [41 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= -0.4555328
##     then
## 	outcome = 0.3 - 0.298 u.size + 0.272 n.nuc + 0.25 adhsn + 0.15 s.size
## 
##   Rule 32/4: [23 cases, mean 0.2, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= -0.5175617
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	nucl > -0.4555328
##     then
## 	outcome = 0.3 + 0.622 u.shape - 0.178 thick
## 
##   Rule 32/5: [7 cases, mean 0.6, range 0 to 1, est err 0.6]
## 
##     if
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	nucl > -0.4555328
## 	n.nuc > 0.669092
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.8 + 0.112 thick
## 
##   Rule 32/6: [20 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > -0.5175617
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	s.size > -0.5858465
## 	nucl > -0.4555328
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.7 + 0.121 nucl + 0.08 thick + 0.008 u.shape
## 
##   Rule 32/7: [26 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	thick > -0.5175617
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	nucl > 0.08835271
##     then
## 	outcome = 0.9 + 0.082 thick
## 
##   Rule 32/8: [6 cases, mean 0.8, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 1.493752
## 	nucl > 0.3602955
## 	nucl <= 1.176124
##     then
## 	outcome = -5.7 + 3.172 u.size
## 
##   Rule 32/9: [10 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size > 0.2327753
## 	u.size <= 1.493752
## 	nucl > 0.3602955
## 	nucl <= 1.176124
##     then
## 	outcome = 0.6 + 0.119 nucl + 0.061 u.size + 0.056 thick + 0.05 n.nuc
## 	          + 0.028 s.size
## 
##   Rule 32/10: [33 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	nucl <= 0.3602955
##     then
## 	outcome = 1
## 
##   Rule 32/11: [9 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	n.nuc > 1.636011
##     then
## 	outcome = 1 - 0.136 u.size
## 
##   Rule 32/12: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 32/13: [78 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
## Model 33:
## 
##   Rule 33/1: [226 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl <= 0.08835271
##     then
## 	outcome = -0
## 
##   Rule 33/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 33/3: [270 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	adhsn <= 0.7209161
## 	nucl <= -0.4555328
## 	n.nuc <= 1.313705
##     then
## 	outcome = -0
## 
##   Rule 33/4: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1 + 0.844 thick
## 
##   Rule 33/5: [11 cases, mean 0.2, range 0 to 1, est err 0.2]
## 
##     if
## 	thick > -0.16559
## 	thick <= 0.5383533
## 	u.shape <= 0.5404623
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
## 	n.nuc <= 1.313705
##     then
## 	outcome = -0.1
## 
##   Rule 33/6: [13 cases, mean 0.4, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= -0.16559
## 	u.shape > -0.7517401
## 	u.shape <= 0.5404623
## 	adhsn <= 0.7209161
## 	nucl > -0.4555328
##     then
## 	outcome = -0.8 - 1.703 thick + 0.939 nucl
## 
##   Rule 33/7: [6 cases, mean 0.7, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > 0.5404623
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.7 + 0.176 nucl
## 
##   Rule 33/8: [71 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.313705
##     then
## 	outcome = 1
## 
##   Rule 33/9: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 33/10: [4 cases, mean 1.0, range 1 to 1, est err 0.5]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1.3
## 
##   Rule 33/11: [4 cases, mean 1.0, range 1 to 1, est err 0.3]
## 
##     if
## 	thick > -0.16559
## 	thick <= 0.5383533
## 	u.shape <= 0.5404623
## 	adhsn <= -0.3093781
## 	nucl > -0.4555328
## 	n.nuc <= 1.313705
##     then
## 	outcome = 1.2
## 
##   Rule 33/12: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 33/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 34:
## 
##   Rule 34/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 34/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 34/3: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 0.1 + 0.116 nucl + 0.067 u.size + 0.063 thick + 0.045 n.nuc
## 	          + 0.027 s.size
## 
##   Rule 34/4: [15 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= -0.08246895
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.8 + 0.932 n.nuc + 0.513 thick
## 
##   Rule 34/5: [11 cases, mean 0.5, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.1 - 1.003 thick + 0.019 n.nuc + 0.01 chrom
## 
##   Rule 34/6: [19 cases, mean 0.5, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.4 + 0.199 thick + 0.165 n.nuc + 0.131 chrom - 0.012 u.size
## 
##   Rule 34/7: [106 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 34/8: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 34/9: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 34/10: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 34/11: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 34/12: [90 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.08246895
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 34/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 35:
## 
##   Rule 35/1: [225 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 35/2: [170 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.5175617
## 	u.size <= -0.7129574
##     then
## 	outcome = -0
## 
##   Rule 35/3: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 35/4: [85 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > -0.5175617
## 	u.size <= -0.7129574
##     then
## 	outcome = 0.3 + 0.717 nucl
## 
##   Rule 35/5: [5 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.3977132
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	n.nuc > 0.3467855
## 	n.nuc <= 1.313705
##     then
## 	outcome = -0.1 + 0.279 u.shape
## 
##   Rule 35/6: [8 cases, mean 0.4, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= -0.3977132
## 	nucl > -0.7274755
##     then
## 	outcome = 0.1 + 0.308 u.shape - 0.226 adhsn + 0.125 thick
## 
##   Rule 35/7: [13 cases, mean 0.5, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.3977132
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc <= 0.3467855
##     then
## 	outcome = 0.6 - 1.718 u.size + 0.5 thick + 0.193 u.shape - 0.101 adhsn
## 
##   Rule 35/8: [7 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.594268
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	n.nuc > 1.313705
##     then
## 	outcome = 0.9 + 0.058 u.shape + 0.053 nucl + 0.05 thick
## 
##   Rule 35/9: [13 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	thick <= 1.594268
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
##     then
## 	outcome = 0.9 + 0.096 thick
## 
##   Rule 35/10: [108 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 35/11: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 35/12: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 35/13: [50 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.594268
##     then
## 	outcome = 1
## 
## Model 36:
## 
##   Rule 36/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 36/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 36/3: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.428 thick
## 
##   Rule 36/4: [19 cases, mean 0.5, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	s.size <= 0.2860255
## 	nucl > -0.7274755
## 	nucl <= 0.6322382
## 	n.nuc <= 0.669092
##     then
## 	outcome = 0.8 + 1.052 n.nuc + 0.857 nucl + 0.013 thick
## 
##   Rule 36/5: [6 cases, mean 0.5, range 0 to 1, est err 0.6]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	s.size > 0.2860255
## 	nucl <= 0.6322382
## 	n.nuc <= 0.669092
##     then
## 	outcome = 1 + 0.268 nucl + 0.154 n.nuc + 0.151 thick + 0.083 adhsn
## 	          + 0.005 chrom
## 
##   Rule 36/6: [7 cases, mean 0.6, range 0 to 1, est err 0.6]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 0.6322382
## 	n.nuc > 0.669092
##     then
## 	outcome = 0.9
## 
##   Rule 36/7: [22 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	nucl > 0.6322382
## 	nucl <= 1.176124
##     then
## 	outcome = 1 + 0.007 nucl
## 
##   Rule 36/8: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 36/9: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
## Model 37:
## 
##   Rule 37/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 37/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.1499105
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 37/3: [5 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > -0.5175617
## 	thick <= 0.5383533
## 	u.size <= 1.808996
## 	adhsn <= 0.7209161
## 	s.size > -0.1499105
## 	nucl <= 1.448066
## 	n.nuc <= 1.636011
##     then
## 	outcome = -1
## 
##   Rule 37/4: [28 cases, mean 0.2, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= -0.5175617
## 	u.size > -0.7129574
## 	u.size <= 1.808996
## 	nucl <= 1.448066
##     then
## 	outcome = 0.4 + 0.527 nucl
## 
##   Rule 37/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.451 thick
## 
##   Rule 37/6: [16 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > -0.5175617
## 	u.size > -0.7129574
## 	u.size <= 1.808996
## 	s.size <= -0.1499105
## 	nucl > -0.7274755
## 	nucl <= 1.448066
## 	chrom <= 0.6140398
##     then
## 	outcome = 0.8 + 0.601 chrom + 0.366 thick + 0.293 s.size
## 
##   Rule 37/7: [8 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > 0.5383533
## 	thick <= 1.242297
## 	u.size <= 1.808996
## 	adhsn <= 0.7209161
## 	s.size > -0.1499105
## 	nucl <= 1.448066
##     then
## 	outcome = 0.7 + 0.13 nucl + 0.089 thick + 0.043 n.nuc + 0.04 u.shape
## 	          + 0.033 chrom + 0.031 s.size + 0.01 u.size
## 
##   Rule 37/8: [22 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > -0.5175617
## 	u.size > -0.7129574
## 	u.size <= 1.808996
## 	s.size <= -0.1499105
## 	nucl > -0.7274755
## 	nucl <= 1.448066
##     then
## 	outcome = 0.7 + 0.099 nucl + 0.083 thick + 0.048 u.shape + 0.042 chrom
## 	          + 0.029 s.size
## 
##   Rule 37/9: [92 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.448066
##     then
## 	outcome = 1
## 
##   Rule 37/10: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 37/11: [17 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
## 	u.size <= 1.808996
## 	nucl <= 1.448066
##     then
## 	outcome = 1 + 0.03 s.size - 0.014 u.size + 0.01 thick
## 
##   Rule 37/12: [15 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size <= 1.808996
## 	nucl <= 1.448066
## 	n.nuc > 1.636011
##     then
## 	outcome = 1 + 0.033 thick
## 
##   Rule 37/13: [84 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	nucl > 1.448066
## 	chrom > -0.1989626
##     then
## 	outcome = 1
## 
##   Rule 37/14: [54 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.808996
##     then
## 	outcome = 1
## 
## Model 38:
## 
##   Rule 38/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 38/2: [223 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.16559
## 	u.shape <= 0.2174117
## 	adhsn <= -0.3093781
##     then
## 	outcome = -0.3
## 
##   Rule 38/3: [260 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 38/4: [14 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape <= 0.2174117
## 	adhsn > -0.3093781
## 	adhsn <= 1.064348
##     then
## 	outcome = -0.1 + 0.545 u.shape
## 
##   Rule 38/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.469 thick
## 
##   Rule 38/6: [11 cases, mean 0.7, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= -0.3093781
## 	s.size > -0.1499105
##     then
## 	outcome = 0.7 + 0.295 u.shape + 0.217 nucl
## 
##   Rule 38/7: [12 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	u.shape > 0.2174117
##     then
## 	outcome = -0.1 + 0.507 nucl + 0.455 u.shape + 0.009 thick + 0.009 n.nuc
## 	          + 0.008 chrom
## 
##   Rule 38/8: [9 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > 1.493752
## 	nucl <= 1.448066
##     then
## 	outcome = -5.7 + 3.172 u.size
## 
##   Rule 38/9: [11 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > 0.2327753
## 	u.size <= 1.493752
## 	nucl <= 1.448066
##     then
## 	outcome = 0.9 + 0.041 nucl + 0.031 thick + 0.018 n.nuc + 0.017 chrom
## 	          + 0.014 s.size
## 
##   Rule 38/10: [12 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	thick > -0.16559
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= -0.3093781
## 	nucl > -0.7274755
##     then
## 	outcome = 1.1
## 
##   Rule 38/11: [92 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.448066
##     then
## 	outcome = 1
## 
##   Rule 38/12: [73 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 1.064348
##     then
## 	outcome = 1
## 
##   Rule 38/13: [76 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 38/14: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
## Model 39:
## 
##   Rule 39/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 39/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 39/3: [6 cases, mean 0.2, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.3 + 1.075 thick
## 
##   Rule 39/4: [26 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.1863816
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = -0 + 0.731 nucl - 0.347 thick
## 
##   Rule 39/5: [8 cases, mean 0.5, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1 + 6.682 u.shape - 1.203 s.size + 0.011 nucl + 0.006 thick
## 
##   Rule 39/6: [15 cases, mean 0.7, range 0 to 1, est err 0.8]
## 
##     if
## 	thick <= 0.1863816
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1 + 1.246 adhsn - 0.66 s.size
## 
##   Rule 39/7: [138 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1
## 
##   Rule 39/8: [113 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.1863816
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 39/9: [59 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > 1.176124
##     then
## 	outcome = 0.8 + 0.054 nucl + 0.032 thick + 0.019 s.size + 0.019 n.nuc
## 	          + 0.018 chrom + 0.009 u.size
## 
##   Rule 39/10: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 40:
## 
##   Rule 40/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 40/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 40/3: [278 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	u.size <= -0.3977132
## 	n.nuc <= 1.313705
##     then
## 	outcome = -0
## 
##   Rule 40/4: [6 cases, mean 0.2, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.1 + 0.89 thick
## 
##   Rule 40/5: [6 cases, mean 0.3, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.3977132
## 	u.shape > -0.7517401
## 	adhsn <= 0.7209161
## 	s.size <= -0.5858465
## 	nucl > -0.7274755
##     then
## 	outcome = 0.1 + 0.339 chrom
## 
##   Rule 40/6: [8 cases, mean 0.4, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.3977132
## 	u.shape <= 0.5404623
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
## 	n.nuc <= 1.313705
##     then
## 	outcome = -0.4 + 0.122 chrom
## 
##   Rule 40/7: [11 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > 0.5404623
## 	u.shape <= 1.186563
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.6 + 0.183 nucl + 0.074 thick + 0.057 u.size + 0.051 s.size
## 	          + 0.046 chrom + 0.043 n.nuc
## 
##   Rule 40/8: [31 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	n.nuc > 1.313705
##     then
## 	outcome = 1 + 0.011 nucl
## 
##   Rule 40/9: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.3774847
##     then
## 	outcome = 1
## 
##   Rule 40/10: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 40/11: [4 cases, mean 1.0, range 1 to 1, est err 2.4]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1 - 6.695 u.shape + 1.205 s.size
## 
##   Rule 40/12: [13 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.3977132
## 	u.shape <= 0.5404623
## 	adhsn <= -0.3093781
## 	s.size > -0.5858465
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 40/13: [20 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.1863816
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
##     then
## 	outcome = 1.1 - 0.02 n.nuc
## 
##   Rule 40/14: [40 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
##     then
## 	outcome = 1 - 0.009 n.nuc - 0.008 thick
## 
##   Rule 40/15: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 41:
## 
##   Rule 41/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 41/2: [235 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.4286895
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 41/3: [274 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	adhsn <= 0.7209161
## 	nucl <= -0.4555328
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0
## 
##   Rule 41/4: [41 cases, mean 0.2, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape <= -0.4286895
## 	nucl > -0.7274755
##     then
## 	outcome = 0.3 + 0.339 thick + 0.299 u.size + 0.282 nucl
## 
##   Rule 41/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.469 thick
## 
##   Rule 41/6: [38 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.shape > -0.4286895
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.4555328
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.9 + 0.08 nucl
## 
##   Rule 41/7: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 41/8: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 41/9: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 41/10: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 42:
## 
##   Rule 42/1: [191 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl <= -0.4555328
## 	chrom <= -0.6054638
##     then
## 	outcome = 0
## 
##   Rule 42/2: [269 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	s.size <= -0.1499105
## 	nucl <= -0.4555328
##     then
## 	outcome = -0
## 
##   Rule 42/3: [253 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= -0.7129574
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0
## 
##   Rule 42/4: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 42/5: [47 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	chrom <= -0.6054638
##     then
## 	outcome = -0.1 + 0.62 nucl
## 
##   Rule 42/6: [14 cases, mean 0.3, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	s.size > -0.1499105
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = -0.8 + 0.273 nucl - 0.045 u.size - 0.036 adhsn + 0.019 thick
## 
##   Rule 42/7: [4 cases, mean 0.8, range 0 to 1, est err 0.6]
## 
##     if
## 	thick > 0.5383533
## 	thick <= 1.242297
## 	adhsn <= 0.7209161
## 	s.size > -0.1499105
## 	nucl <= 1.176124
## 	chrom > -0.6054638
## 	n.nuc <= 1.636011
##     then
## 	outcome = 1.1
## 
##   Rule 42/8: [10 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	s.size <= -0.1499105
## 	nucl > -0.4555328
## 	nucl <= 1.176124
## 	chrom > -0.6054638
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.7 + 0.369 nucl - 0.325 adhsn + 0.247 thick - 0.202 u.size
## 	          + 0.041 chrom + 0.014 n.nuc
## 
##   Rule 42/9: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 42/10: [9 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	n.nuc > 1.636011
##     then
## 	outcome = 1 - 0.014 adhsn + 0.007 chrom + 0.005 thick + 0.005 nucl
## 
##   Rule 42/11: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 42/12: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 42/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 43:
## 
##   Rule 43/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 43/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 43/3: [17 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
## 	nucl <= 0.9041809
## 	n.nuc <= 0.669092
##     then
## 	outcome = 0.8 + 1.244 n.nuc + 0.465 thick
## 
##   Rule 43/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.466 thick
## 
##   Rule 43/5: [4 cases, mean 0.5, range 0 to 1, est err 0.8]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.9
## 
##   Rule 43/6: [7 cases, mean 0.6, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	nucl <= 0.9041809
## 	n.nuc > 0.669092
##     then
## 	outcome = 0.7 + 0.136 u.shape - 0.103 adhsn + 0.052 nucl
## 
##   Rule 43/7: [6 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	n.nuc > 1.313705
##     then
## 	outcome = 0.8 + 0.133 nucl + 0.114 u.shape + 0.068 mit + 0.024 s.size
## 
##   Rule 43/8: [115 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 0.9041809
##     then
## 	outcome = 1
## 
##   Rule 43/9: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 43/10: [12 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	adhsn > 0.7209161
##     then
## 	outcome = 1 + 0.009 u.size + 0.008 nucl - 0.006 adhsn
## 
##   Rule 43/11: [50 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.594268
##     then
## 	outcome = 1
## 
## Model 44:
## 
##   Rule 44/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 44/2: [36 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= 0.2174117
## 	nucl <= 0.08835271
## 	chrom <= -0.1989626
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.1 + 0.073 u.shape + 0.023 chrom + 0.019 thick + 0.015 n.nuc
## 	          + 0.011 nucl + 0.006 mit
## 
##   Rule 44/3: [257 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 0.2174117
## 	nucl <= -0.7274755
## 	chrom <= -0.1989626
##     then
## 	outcome = 0
## 
##   Rule 44/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.453 thick
## 
##   Rule 44/5: [17 cases, mean 0.7, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape <= 0.2174117
## 	adhsn <= 1.407779
## 	nucl > 0.08835271
##     then
## 	outcome = -0 + 0.593 nucl
## 
##   Rule 44/6: [14 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape > 0.2174117
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1.1 - 0.659 u.size + 0.05 nucl + 0.034 u.shape + 0.033 thick
## 	          + 0.028 chrom + 0.017 s.size
## 
##   Rule 44/7: [11 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= 0.2174117
## 	nucl <= 0.08835271
## 	chrom > -0.1989626
##     then
## 	outcome = 0.7 + 0.156 u.shape + 0.126 nucl + 0.097 thick + 0.097 n.nuc
## 	          + 0.083 u.size + 0.059 chrom
## 
##   Rule 44/8: [10 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= 0.2174117
## 	nucl > -0.7274755
## 	chrom <= -0.1989626
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1.1 + 0.129 s.size
## 
##   Rule 44/9: [130 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.2174117
##     then
## 	outcome = 1
## 
##   Rule 44/10: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 44/11: [62 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 1.407779
##     then
## 	outcome = 1
## 
##   Rule 44/12: [15 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	u.shape <= 0.2174117
##     then
## 	outcome = 1
## 
##   Rule 44/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 45:
## 
##   Rule 45/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 45/2: [111 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.8695334
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 45/3: [239 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.5858465
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 45/4: [35 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.1 + 0.937 s.size
## 
##   Rule 45/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.437 thick
## 
##   Rule 45/6: [24 cases, mean 0.5, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.1 - 0.581 u.size + 0.409 u.shape + 0.224 thick - 0.212 adhsn
## 	          + 0.157 nucl
## 
##   Rule 45/7: [11 cases, mean 0.7, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 0.5383533
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc > 0.02447898
##     then
## 	outcome = 1.2
## 
##   Rule 45/8: [9 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	n.nuc > 0.9913985
##     then
## 	outcome = 0.5 + 0.16 nucl + 0.14 thick + 0.058 n.nuc + 0.057 s.size
## 	          + 0.043 adhsn
## 
##   Rule 45/9: [9 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.8 + 0.083 nucl + 0.071 thick + 0.03 s.size + 0.03 n.nuc
## 	          + 0.022 adhsn
## 
##   Rule 45/10: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 45/11: [14 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1 - 0.177 u.shape - 0.104 u.size + 0.04 thick - 0.038 adhsn
## 
##   Rule 45/12: [12 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	adhsn > 0.7209161
##     then
## 	outcome = 1 + 0.037 u.size
## 
## Model 46:
## 
##   Rule 46/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 46/2: [252 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.5858465
## 	nucl <= -0.4555328
##     then
## 	outcome = -0
## 
##   Rule 46/3: [43 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	chrom <= -0.1989626
## 	n.nuc <= 0.9913985
##     then
## 	outcome = -0
## 
##   Rule 46/4: [8 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	s.size > -0.5858465
## 	s.size <= 0.2860255
## 	nucl <= -0.4555328
##     then
## 	outcome = 2.7 + 3.677 nucl
## 
##   Rule 46/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.443 thick
## 
##   Rule 46/6: [5 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	s.size > 0.2860255
## 	nucl <= -0.4555328
##     then
## 	outcome = 1.1 + 0.396 nucl + 0.092 thick + 0.062 u.shape + 0.053 chrom
## 	          + 0.048 n.nuc
## 
##   Rule 46/7: [10 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	nucl > -0.4555328
## 	chrom > -0.1989626
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.9 + 0.181 nucl + 0.094 thick + 0.085 u.shape + 0.065 chrom
## 	          + 0.042 n.nuc
## 
##   Rule 46/8: [10 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.4555328
## 	n.nuc > 0.9913985
##     then
## 	outcome = -4.1 + 2.118 n.nuc + 0.549 nucl
## 
##   Rule 46/9: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 46/10: [73 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 1.064348
##     then
## 	outcome = 1
## 
##   Rule 46/11: [49 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.4555328
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1
## 
## Model 47:
## 
##   Rule 47/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 47/2: [35 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= -0.1056389
## 	s.size <= -0.5858465
##     then
## 	outcome = 0
## 
##   Rule 47/3: [10 cases, mean 0.2, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= -0.1056389
## 	s.size > -0.5858465
## 	nucl <= -0.4555328
##     then
## 	outcome = 2.8 + 3.829 nucl
## 
##   Rule 47/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.443 thick
## 
##   Rule 47/5: [12 cases, mean 0.5, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.1056389
## 	nucl > -0.4555328
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0 - 1.489 thick + 0.835 adhsn
## 
##   Rule 47/6: [11 cases, mean 0.5, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.shape <= -0.1056389
## 	nucl > -0.4555328
## 	nucl <= 1.176124
##     then
## 	outcome = 1.9 + 3.535 u.shape + 0.746 thick
## 
##   Rule 47/7: [6 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape > -0.1056389
## 	u.shape <= 0.8635129
## 	nucl <= -0.4555328
##     then
## 	outcome = 1 + 0.351 nucl + 0.09 s.size + 0.073 thick + 0.05 n.nuc
## 	          + 0.046 u.shape
## 
##   Rule 47/8: [17 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape > -0.1056389
## 	nucl <= -0.4555328
##     then
## 	outcome = 1
## 
##   Rule 47/9: [22 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > 1.176124
##     then
## 	outcome = 0.5 + 0.162 nucl + 0.102 thick + 0.063 n.nuc + 0.047 u.shape
## 	          + 0.045 u.size + 0.04 chrom + 0.039 s.size
## 
##   Rule 47/10: [93 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.4555328
##     then
## 	outcome = 0.9 + 0.051 thick
## 
##   Rule 47/11: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 47/12: [6 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	nucl > -0.4555328
## 	nucl <= 1.176124
## 	n.nuc > 1.636011
##     then
## 	outcome = 0.9 + 0.161 thick + 0.01 nucl
## 
## Model 48:
## 
##   Rule 48/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 48/2: [260 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 48/3: [6 cases, mean 0.3, range 0 to 1, est err 0.5]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.1 + 0.046 nucl + 0.024 thick + 0.022 u.size + 0.018 n.nuc
## 	          + 0.012 s.size + 0.01 chrom
## 
##   Rule 48/4: [14 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
##     then
## 	outcome = 0 + 0.072 nucl - 0.036 adhsn + 0.023 thick
## 
##   Rule 48/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.457 thick
## 
##   Rule 48/6: [14 cases, mean 0.4, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= -0.3093781
## 	nucl > -0.7274755
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.3 - 1.056 n.nuc + 1.048 chrom + 1.029 u.size + 0.641 s.size
## 	          - 0.534 nucl + 0.525 thick
## 
##   Rule 48/7: [9 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	n.nuc > 1.313705
##     then
## 	outcome = 0.5 - 0.296 adhsn + 0.186 thick + 0.185 nucl
## 
##   Rule 48/8: [13 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	thick > 0.5383533
## 	thick <= 1.594268
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
##     then
## 	outcome = 1.1
## 
##   Rule 48/9: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1 + 0.005 nucl
## 
##   Rule 48/10: [12 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	u.size <= 0.2327753
## 	adhsn > 0.7209161
##     then
## 	outcome = 1.1 - 0.093 u.size - 0.045 thick - 0.028 s.size
## 
##   Rule 48/11: [10 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 1.594268
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
##     then
## 	outcome = 0.8 - 0.144 adhsn + 0.091 thick + 0.088 nucl
## 
## Model 49:
## 
##   Rule 49/1: [235 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.4286895
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 49/2: [16 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.8695334
## 	u.shape <= -0.4286895
## 	nucl > -0.7274755
##     then
## 	outcome = 0 + 0.024 thick + 0.022 s.size + 0.011 nucl
## 
##   Rule 49/3: [27 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.4286895
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.2 + 0.346 s.size
## 
##   Rule 49/4: [42 cases, mean 0.2, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size <= -0.3977132
## 	nucl > -0.7274755
##     then
## 	outcome = 0.2 + 0.349 thick + 0.211 s.size + 0.013 nucl + 0.009 u.size
## 
##   Rule 49/5: [29 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > -0.8695334
## 	s.size <= -0.5858465
## 	nucl > -0.7274755
##     then
## 	outcome = -1.9 - 3.386 s.size + 0.392 thick
## 
##   Rule 49/6: [25 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.4286895
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.9
## 
##   Rule 49/7: [7 cases, mean 0.9, range 0 to 1, est err 1.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= -0.08246895
## 	u.shape > -0.4286895
## 	nucl <= 1.176124
## 	n.nuc > 0.02447898
## 	n.nuc <= 1.636011
##     then
## 	outcome = 1.9
## 
##   Rule 49/8: [9 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.08246895
## 	u.shape > -0.4286895
## 	nucl > 1.176124
##     then
## 	outcome = 0.6 + 0.118 nucl + 0.076 u.size + 0.066 thick + 0.043 n.nuc
## 	          + 0.026 chrom + 0.022 s.size
## 
##   Rule 49/9: [145 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.4286895
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1
## 
##   Rule 49/10: [99 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.4286895
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 49/11: [4 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.3977132
## 	u.shape <= -0.4286895
## 	s.size > -0.5858465
## 	nucl > -0.7274755
##     then
## 	outcome = 0.8 + 0.093 thick + 0.06 s.size + 0.05 nucl + 0.025 u.size
## 	          + 0.012 n.nuc + 0.007 chrom
## 
##   Rule 49/12: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 50:
## 
##   Rule 50/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 50/2: [7 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 0.5404623
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
## 	n.nuc <= 0.02447898
##     then
## 	outcome = -0.1
## 
##   Rule 50/3: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 50/4: [15 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= -0.3093781
## 	nucl > -0.7274755
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.5 + 0.513 chrom + 0.32 thick + 0.279 s.size
## 
##   Rule 50/5: [11 cases, mean 0.7, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape <= 1.186563
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
## 	n.nuc > 0.02447898
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.9 + 0.071 nucl
## 
##   Rule 50/6: [11 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > 0.5404623
## 	u.shape <= 1.186563
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.6 + 0.151 nucl + 0.076 thick + 0.067 u.size + 0.063 n.nuc
## 	          + 0.026 s.size + 0.025 chrom
## 
##   Rule 50/7: [106 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 50/8: [38 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 1 + 0.009 thick + 0.007 nucl
## 
##   Rule 50/9: [71 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.313705
##     then
## 	outcome = 1
## 
##   Rule 50/10: [4 cases, mean 1.0, range 1 to 1, est err 0.5]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
## 	s.size <= -0.5858465
##     then
## 	outcome = 1 + 0.455 u.shape
## 
##   Rule 50/11: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 50/12: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 51:
## 
##   Rule 51/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = -0
## 
##   Rule 51/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 51/3: [36 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	adhsn <= 0.7209161
## 	chrom <= -0.1989626
## 	n.nuc <= -0.2978275
##     then
## 	outcome = -0 + 0.122 nucl
## 
##   Rule 51/4: [7 cases, mean 0.3, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size <= 1.808996
## 	adhsn <= 0.7209161
## 	s.size > -0.5858465
## 	chrom <= 0.2075386
## 	n.nuc > -0.2978275
## 	n.nuc <= 1.313705
##     then
## 	outcome = -0.9
## 
##   Rule 51/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.466 thick
## 
##   Rule 51/6: [6 cases, mean 0.5, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.6 + 0.166 nucl + 0.099 thick + 0.083 u.size + 0.052 n.nuc
## 
##   Rule 51/7: [7 cases, mean 0.7, range 0 to 1, est err 0.5]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 1.808996
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.7 + 0.174 u.size + 0.143 nucl + 0.074 thick + 0.054 n.nuc
## 	          + 0.031 s.size + 0.021 chrom
## 
##   Rule 51/8: [4 cases, mean 0.8, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	adhsn <= 0.7209161
## 	s.size <= -0.5858465
## 	nucl > -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1.1
## 
##   Rule 51/9: [12 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 1.808996
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	chrom > 0.2075386
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.8 + 0.185 thick + 0.114 chrom + 0.074 nucl
## 
##   Rule 51/10: [46 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	u.size <= 1.808996
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 1 + 0.015 nucl
## 
##   Rule 51/11: [71 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.313705
##     then
## 	outcome = 1
## 
##   Rule 51/12: [4 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	adhsn <= -0.3093781
## 	nucl > -0.7274755
## 	chrom > -0.1989626
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 1.1
## 
##   Rule 51/13: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 51/14: [54 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.808996
##     then
## 	outcome = 1
## 
## Model 52:
## 
##   Rule 52/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 52/2: [36 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 0.1863816
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 0.9041809
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0
## 
##   Rule 52/3: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.443 thick
## 
##   Rule 52/4: [7 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape <= -0.1056389
## 	nucl <= 0.9041809
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.3 + 0.147 nucl + 0.119 thick + 0.06 chrom + 0.056 u.shape
## 	          + 0.051 s.size + 0.049 n.nuc
## 
##   Rule 52/5: [20 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 0.9041809
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.1 - 3.4 u.size + 0.833 u.shape + 0.509 adhsn
## 
##   Rule 52/6: [5 cases, mean 0.6, range 0 to 1, est err 0.6]
## 
##     if
## 	thick > 0.1863816
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 0.9041809
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.7 + 0.239 thick + 0.161 n.nuc + 0.123 adhsn
## 
##   Rule 52/7: [32 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.9 + 0.042 nucl + 0.034 thick + 0.017 chrom + 0.016 u.shape
## 	          + 0.014 s.size + 0.014 n.nuc
## 
##   Rule 52/8: [25 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > 0.9041809
##     then
## 	outcome = 0.9 + 0.059 u.shape + 0.047 nucl + 0.025 thick + 0.024 chrom
## 	          + 0.016 s.size
## 
##   Rule 52/9: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 52/10: [13 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.594268
## 	u.size <= 0.2327753
##     then
## 	outcome = 1 - 0.015 nucl
## 
## Model 53:
## 
##   Rule 53/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 53/2: [40 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= -0.4286895
##     then
## 	outcome = 0.3 + 0.201 nucl + 0.147 thick + 0.13 s.size + 0.09 u.shape
## 	          + 0.048 chrom
## 
##   Rule 53/3: [54 cases, mean 0.2, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.2 + 0.153 chrom + 0.121 n.nuc + 0.102 nucl
## 
##   Rule 53/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.443 thick
## 
##   Rule 53/5: [20 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > 0.5383533
## 	u.size > -0.7129574
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl <= 0.9041809
##     then
## 	outcome = 0.6 + 0.179 nucl + 0.117 u.shape + 0.084 thick + 0.071 chrom
## 	          + 0.06 s.size + 0.016 u.size + 0.01 n.nuc
## 
##   Rule 53/6: [13 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.4286895
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	chrom > 0.2075386
##     then
## 	outcome = 0.7 + 0.149 nucl + 0.061 chrom + 0.057 n.nuc + 0.05 thick
## 	          + 0.048 u.size
## 
##   Rule 53/7: [41 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	u.size > -0.7129574
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
##     then
## 	outcome = 1 + 0.015 u.size
## 
##   Rule 53/8: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 53/9: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1 + 0.006 nucl
## 
## Model 54:
## 
##   Rule 54/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 54/2: [257 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
## 	chrom <= -0.1989626
##     then
## 	outcome = -0
## 
##   Rule 54/3: [294 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 0.1863816
## 	u.size <= 0.2327753
## 	nucl <= 0.9041809
##     then
## 	outcome = -0
## 
##   Rule 54/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.46 thick
## 
##   Rule 54/5: [11 cases, mean 0.5, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 0.9041809
## 	chrom > -0.1989626
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.1 + 0.091 nucl + 0.036 thick + 0.034 n.nuc
## 
##   Rule 54/6: [18 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > 0.1863816
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 0.9041809
##     then
## 	outcome = 1.1
## 
##   Rule 54/7: [9 cases, mean 0.8, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.5 - 1.428 thick + 0.498 adhsn
## 
##   Rule 54/8: [25 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > 0.9041809
##     then
## 	outcome = 1
## 
##   Rule 54/9: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 54/10: [5 cases, mean 1.0, range 1 to 1, est err 0.9]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
## 	nucl <= 0.9041809
## 	chrom <= -0.1989626
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1.6
## 
##   Rule 54/11: [6 cases, mean 1.0, range 1 to 1, est err 0.5]
## 
##     if
## 	thick > 1.594268
## 	u.size <= 0.2327753
## 	nucl <= 0.9041809
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1.4 - 0.978 u.size - 0.368 nucl - 0.14 n.nuc
## 
## Model 55:
## 
##   Rule 55/1: [235 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.4286895
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 55/2: [179 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.5175617
## 	u.shape <= -0.4286895
##     then
## 	outcome = 0
## 
##   Rule 55/3: [253 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0
## 
##   Rule 55/4: [283 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 0.1863816
## 	u.size <= 0.2327753
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 55/5: [10 cases, mean 0.2, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > -0.5175617
## 	u.shape <= -0.4286895
## 	s.size <= -0.5858465
## 	nucl > -0.7274755
##     then
## 	outcome = -2.3 - 3.753 s.size + 0.389 thick - 0.337 u.shape + 0.206 nucl
## 
##   Rule 55/6: [27 cases, mean 0.6, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape > -0.4286895
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.5 + 0.239 nucl + 0.198 n.nuc + 0.125 chrom
## 
##   Rule 55/7: [5 cases, mean 0.6, range 0 to 1, est err 1.4]
## 
##     if
## 	thick <= 0.1863816
## 	u.size > -0.7129574
## 	u.shape > -0.4286895
## 	nucl <= 0.08835271
## 	chrom <= 0.2075386
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1.5
## 
##   Rule 55/8: [9 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.9
## 
##   Rule 55/9: [12 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size <= 0.2327753
## 	u.shape > -0.4286895
## 	n.nuc > 0.9913985
##     then
## 	outcome = -2.3 + 1.479 n.nuc - 0.981 mit + 0.02 nucl + 0.009 chrom
## 
##   Rule 55/10: [16 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size <= 0.2327753
## 	u.shape > -0.4286895
## 	chrom > 0.2075386
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.8 + 0.111 nucl + 0.055 thick + 0.054 n.nuc + 0.033 chrom
## 	          + 0.022 s.size + 0.019 u.size
## 
##   Rule 55/11: [144 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > -0.5175617
## 	s.size > -0.5858465
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 55/12: [115 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.4286895
## 	chrom > 0.2075386
##     then
## 	outcome = 1
## 
##   Rule 55/13: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 55/14: [5 cases, mean 1.0, range 1 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.1863816
## 	u.size <= 0.2327753
## 	u.shape > -0.4286895
## 	nucl > 0.08835271
## 	chrom <= 0.2075386
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1 + 0.325 n.nuc + 0.202 nucl
## 
##   Rule 55/15: [9 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 1.594268
## 	u.size <= 0.2327753
## 	u.shape > -0.4286895
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.9 + 0.027 nucl - 0.021 n.nuc
## 
## Model 56:
## 
##   Rule 56/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 56/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 56/3: [4 cases, mean 0.2, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.1056389
## 	u.shape <= 0.5404623
## 	adhsn <= 0.7209161
## 	s.size > -0.5858465
## 	n.nuc > -0.2978275
## 	n.nuc <= 1.313705
##     then
## 	outcome = -0.6
## 
##   Rule 56/4: [16 cases, mean 0.3, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.2 + 0.416 nucl + 0.077 chrom
## 
##   Rule 56/5: [4 cases, mean 0.8, range 0 to 1, est err 0.8]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	adhsn <= 0.7209161
## 	s.size <= -0.5858465
## 	nucl > -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1.2 + 0.061 thick + 0.038 nucl
## 
##   Rule 56/6: [19 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1
## 
##   Rule 56/7: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.5404623
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 56/8: [71 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.313705
##     then
## 	outcome = 1
## 
##   Rule 56/9: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 56/10: [34 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
##     then
## 	outcome = 1 - 0.046 adhsn
## 
##   Rule 56/11: [4 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1.1
## 
##   Rule 56/12: [40 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
##     then
## 	outcome = 1 + 0.012 nucl
## 
##   Rule 56/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 57:
## 
##   Rule 57/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 57/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 57/3: [7 cases, mean 0.3, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.08246895
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	chrom <= 0.6140398
##     then
## 	outcome = 0.1 + 0.183 nucl + 0.091 thick + 0.085 s.size + 0.045 u.shape
## 	          + 0.044 n.nuc + 0.04 chrom + 0.026 u.size
## 
##   Rule 57/4: [19 cases, mean 0.5, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= -0.08246895
## 	u.shape > -0.7517401
## 	adhsn <= 0.03405333
## 	nucl > -0.7274755
## 	chrom <= 0.6140398
##     then
## 	outcome = 0.3 - 1.116 adhsn + 1.002 u.size + 0.686 chrom + 0.414 thick
## 
##   Rule 57/5: [8 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	adhsn > 0.3774847
## 	adhsn <= 0.7209161
##     then
## 	outcome = 0.9 + 0.045 nucl
## 
##   Rule 57/6: [6 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	chrom > 0.6140398
## 	n.nuc <= 0.02447898
##     then
## 	outcome = 0.6 + 0.214 nucl + 0.041 thick + 0.036 s.size + 0.021 n.nuc
## 	          + 0.018 u.shape + 0.016 chrom + 0.014 u.size
## 
##   Rule 57/7: [6 cases, mean 0.8, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	adhsn > 0.03405333
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	chrom <= 0.6140398
##     then
## 	outcome = 0.6 + 0.805 u.size + 0.378 nucl + 0.158 chrom + 0.075 thick
## 
##   Rule 57/8: [114 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	nucl > -0.7274755
## 	n.nuc > 0.02447898
##     then
## 	outcome = 1
## 
##   Rule 57/9: [106 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 57/10: [48 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 57/11: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 57/12: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 57/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 58:
## 
##   Rule 58/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 58/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 58/3: [43 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.1 + 0.575 nucl
## 
##   Rule 58/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.469 thick
## 
##   Rule 58/5: [13 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
## 	n.nuc > -0.2978275
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.8 + 0.121 nucl
## 
##   Rule 58/6: [13 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size <= 0.2327753
## 	n.nuc > 0.9913985
##     then
## 	outcome = -3.1 + 1.805 n.nuc + 0.275 nucl
## 
##   Rule 58/7: [108 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 58/8: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 58/9: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
## Model 59:
## 
##   Rule 59/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 59/2: [29 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 0.890325
## 	u.size > -0.7129574
## 	u.shape <= -0.1056389
## 	s.size <= -0.5858465
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0
## 
##   Rule 59/3: [41 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.890325
## 	u.size > -0.7129574
## 	u.shape <= -0.1056389
## 	nucl <= -0.18359
##     then
## 	outcome = 0 + 0.25 n.nuc + 0.136 u.size
## 
##   Rule 59/4: [23 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.890325
## 	u.shape <= -0.1056389
## 	s.size > -0.5858465
## 	nucl <= -0.18359
##     then
## 	outcome = 0.4 + 0.259 nucl + 0.24 u.shape + 0.18 n.nuc + 0.109 thick
## 	          + 0.046 u.size + 0.01 s.size + 0.009 chrom
## 
##   Rule 59/5: [6 cases, mean 0.3, range 0 to 1, est err 0.5]
## 
##     if
## 	thick > -0.5175617
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
## 	mit <= 0.1942086
##     then
## 	outcome = -0.9
## 
##   Rule 59/6: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.541 thick
## 
##   Rule 59/7: [11 cases, mean 0.6, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > -0.5175617
## 	u.size > -0.7129574
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	s.size <= 0.2860255
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
## 	mit <= 0.1942086
##     then
## 	outcome = 0.4 + 0.252 thick + 0.089 nucl + 0.054 u.shape + 0.029 chrom
## 	          + 0.028 n.nuc + 0.022 s.size
## 
##   Rule 59/8: [19 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= -0.1056389
## 	nucl > -0.18359
##     then
## 	outcome = 1
## 
##   Rule 59/9: [11 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= -0.5175617
## 	u.size > -0.7129574
## 	u.shape > -0.1056389
##     then
## 	outcome = 1
## 
##   Rule 59/10: [54 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= 1.186563
## 	nucl > -0.18359
## 	chrom > 0.2075386
##     then
## 	outcome = 1 + 0.023 u.shape + 0.009 nucl + 0.007 n.nuc
## 
##   Rule 59/11: [91 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.890325
##     then
## 	outcome = 1
## 
##   Rule 59/12: [61 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	mit > 0.1942086
##     then
## 	outcome = 1
## 
##   Rule 59/13: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 59/14: [48 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	nucl > 1.176124
##     then
## 	outcome = 0.9 + 0.024 nucl + 0.015 thick + 0.009 n.nuc + 0.008 u.shape
## 	          + 0.007 s.size
## 
##   Rule 59/15: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 60:
## 
##   Rule 60/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 60/2: [252 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.5858465
## 	nucl <= -0.4555328
##     then
## 	outcome = 0
## 
##   Rule 60/3: [37 cases, mean 0.2, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= -0.16559
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
##     then
## 	outcome = 0 + 0.552 nucl - 0.438 u.size + 0.199 u.shape
## 
##   Rule 60/4: [13 cases, mean 0.3, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	s.size > -0.5858465
## 	nucl <= -0.4555328
##     then
## 	outcome = 1.9 + 2.575 nucl + 0.135 s.size
## 
##   Rule 60/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.408 thick
## 
##   Rule 60/6: [26 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	thick > -0.16559
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	nucl > -0.4555328
##     then
## 	outcome = 1 - 0.034 u.size + 0.015 u.shape + 0.009 nucl
## 
##   Rule 60/7: [14 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	thick > -0.16559
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	nucl > -0.4555328
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1 + 0.026 nucl + 0.022 thick + 0.01 chrom + 0.01 n.nuc
## 	          + 0.008 u.shape + 0.006 s.size
## 
##   Rule 60/8: [81 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 0.9913985
##     then
## 	outcome = 1
## 
##   Rule 60/9: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 60/10: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 60/11: [11 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	thick > -0.16559
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	adhsn <= -0.3093781
## 	nucl > -0.4555328
##     then
## 	outcome = 1.3 - 0.194 nucl - 0.14 u.size + 0.088 u.shape
## 
## Model 61:
## 
##   Rule 61/1: [198 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl <= -0.18359
## 	chrom <= -0.6054638
##     then
## 	outcome = 0
## 
##   Rule 61/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 61/3: [83 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	s.size <= -0.1499105
## 	nucl <= 1.176124
## 	chrom > -0.6054638
## 	n.nuc <= -0.2978275
##     then
## 	outcome = -0
## 
##   Rule 61/4: [49 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= -0.3977132
## 	u.shape > -0.7517401
## 	s.size <= -0.1499105
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0
## 
##   Rule 61/5: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.2 + 1.013 thick
## 
##   Rule 61/6: [7 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > -0.18359
## 	chrom <= -0.6054638
##     then
## 	outcome = 0.5 + 1.178 u.shape + 0.043 nucl + 0.014 thick + 0.01 u.size
## 	          + 0.01 n.nuc + 0.007 s.size + 0.005 chrom
## 
##   Rule 61/7: [20 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > 0.08835271
## 	nucl <= 1.176124
## 	chrom > -0.6054638
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.7 + 0.138 thick + 0.108 s.size + 0.081 chrom
## 
##   Rule 61/8: [15 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size <= -0.1499105
## 	nucl <= 1.176124
## 	chrom > -0.6054638
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.8 + 0.307 nucl + 0.173 s.size + 0.059 thick + 0.035 chrom
## 
##   Rule 61/9: [148 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	chrom > -0.1989626
##     then
## 	outcome = 1
## 
##   Rule 61/10: [135 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > -0.1499105
## 	chrom > -0.6054638
##     then
## 	outcome = 1
## 
##   Rule 61/11: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 61/12: [11 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	u.shape <= 1.186563
## 	s.size > -0.1499105
## 	nucl <= 1.176124
## 	n.nuc > 1.636011
##     then
## 	outcome = 1.1 - 0.026 s.size - 0.018 chrom
## 
##   Rule 61/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 62:
## 
##   Rule 62/1: [225 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 62/2: [22 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > -0.7274755
## 	nucl <= 0.08835271
##     then
## 	outcome = 0 + 0.023 thick + 0.016 nucl
## 
##   Rule 62/3: [270 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	adhsn <= 0.7209161
## 	nucl <= -0.4555328
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0
## 
##   Rule 62/4: [260 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 62/5: [44 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	adhsn <= 0.7209161
## 	chrom <= -0.1989626
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0
## 
##   Rule 62/6: [10 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
## 	chrom > -0.6054638
##     then
## 	outcome = 0.1 + 0.099 nucl + 0.052 thick
## 
##   Rule 62/7: [5 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.08246895
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
##     then
## 	outcome = 0.1 - 0.688 u.size + 0.398 u.shape + 0.079 nucl + 0.041 thick
## 	          + 0.03 n.nuc + 0.016 chrom
## 
##   Rule 62/8: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.465 thick
## 
##   Rule 62/9: [4 cases, mean 0.8, range 0 to 1, est err 0.8]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= -0.08246895
## 	u.shape > -0.4286895
## 	nucl > -0.4555328
## 	chrom <= -0.1989626
##     then
## 	outcome = 1.3 - 0.352 adhsn + 0.342 u.shape + 0.274 thick
## 
##   Rule 62/10: [9 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	n.nuc > 0.9913985
##     then
## 	outcome = 0.6 + 0.204 nucl + 0.108 thick
## 
##   Rule 62/11: [32 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.9 + 0.051 nucl + 0.031 thick + 0.018 u.size + 0.018 n.nuc
## 
##   Rule 62/12: [20 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	thick > 0.5383533
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 0.6 + 0.242 thick - 0.195 adhsn + 0.189 u.shape
## 
##   Rule 62/13: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 62/14: [7 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= -0.08246895
## 	adhsn <= 0.7209161
## 	nucl > -0.4555328
## 	chrom > -0.1989626
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1.1 + 0.284 thick
## 
##   Rule 62/15: [12 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	adhsn > 0.7209161
##     then
## 	outcome = 1 + 0.076 u.size + 0.063 nucl - 0.05 adhsn
## 
## Model 63:
## 
##   Rule 63/1: [203 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl <= 0.08835271
## 	chrom <= -0.6054638
##     then
## 	outcome = 0
## 
##   Rule 63/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.1499105
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 63/3: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 63/4: [15 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= -0.16559
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	chrom > -0.6054638
##     then
## 	outcome = 1.3 + 1.913 u.shape + 0.876 s.size
## 
##   Rule 63/5: [5 cases, mean 0.4, range 0 to 1, est err 0.6]
## 
##     if
## 	u.shape <= 0.2174117
## 	s.size > -0.1499105
## 	nucl > 0.08835271
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = -0.1 + 0.103 nucl + 0.036 n.nuc + 0.031 thick + 0.029 s.size
## 	          + 0.027 u.size
## 
##   Rule 63/6: [9 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape > 0.2174117
## 	u.shape <= 1.186563
## 	s.size > -0.1499105
## 	nucl > 0.08835271
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.7 + 0.126 s.size + 0.123 thick + 0.075 n.nuc
## 
##   Rule 63/7: [5 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 0.08835271
## 	chrom <= -0.6054638
##     then
## 	outcome = 0.6 + 0.214 nucl + 0.026 n.nuc + 0.023 thick + 0.022 s.size
## 	          + 0.02 u.size
## 
##   Rule 63/8: [7 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > -0.16559
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	s.size <= -0.1499105
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	chrom > -0.6054638
##     then
## 	outcome = 1.3 + 0.8 u.shape + 0.408 s.size + 0.268 nucl + 0.082 thick
## 
##   Rule 63/9: [8 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > 1.176124
## 	chrom > -0.6054638
## 	chrom <= -0.1989626
##     then
## 	outcome = 0.6 + 0.18 nucl + 0.097 thick + 0.071 u.size + 0.07 n.nuc
## 	          + 0.04 s.size
## 
##   Rule 63/10: [26 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > -0.1499105
## 	nucl <= 0.08835271
## 	chrom > -0.6054638
##     then
## 	outcome = 1
## 
##   Rule 63/11: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 63/12: [28 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.1056389
## 	s.size <= -0.1499105
## 	nucl > -0.7274755
## 	chrom > -0.6054638
##     then
## 	outcome = 1
## 
##   Rule 63/13: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 63/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 64:
## 
##   Rule 64/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 64/2: [235 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.4286895
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 64/3: [268 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.4286895
## 	adhsn <= 0.7209161
## 	mit <= 0.1942086
##     then
## 	outcome = 0
## 
##   Rule 64/4: [35 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= 0.2174117
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.3 + 3.095 u.shape
## 
##   Rule 64/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.466 thick
## 
##   Rule 64/6: [6 cases, mean 0.7, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= -0.8695334
## 	u.size > -0.7129574
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 64/7: [14 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape > 0.2174117
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1.2 - 0.747 u.size + 0.019 nucl + 0.01 thick + 0.006 n.nuc
## 
##   Rule 64/8: [18 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > -0.8695334
## 	u.size > -0.7129574
## 	u.size <= 1.178508
## 	u.shape > -0.4286895
## 	u.shape <= 0.2174117
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	mit <= 0.1942086
##     then
## 	outcome = -0.2 - 1.742 mit - 1.663 u.shape + 0.441 thick - 0.238 adhsn
## 	          + 0.148 n.nuc + 0.146 nucl
## 
##   Rule 64/9: [55 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > -0.8695334
## 	nucl > -0.7274755
## 	mit > 0.1942086
##     then
## 	outcome = 1
## 
##   Rule 64/10: [58 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= 1.186563
## 	nucl > 1.176124
##     then
## 	outcome = 1 + 0.005 chrom
## 
##   Rule 64/11: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 64/12: [11 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.2174117
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	n.nuc > 0.9913985
##     then
## 	outcome = 1 - 0.015 n.nuc
## 
##   Rule 64/13: [7 cases, mean 1.0, range 1 to 1, est err 0.3]
## 
##     if
## 	u.size > 1.178508
## 	u.shape <= 0.2174117
##     then
## 	outcome = 3.3 - 1.11 u.size + 0.019 nucl + 0.017 u.shape + 0.011 thick
## 	          + 0.009 n.nuc
## 
##   Rule 64/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 65:
## 
##   Rule 65/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 65/2: [252 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0
## 
##   Rule 65/3: [47 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.1863816
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 0.6322382
##     then
## 	outcome = 0.1 + 0.282 nucl
## 
##   Rule 65/4: [6 cases, mean 0.3, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.2 - 0.438 u.size + 0.324 nucl + 0.183 thick + 0.1 n.nuc
## 	          + 0.038 chrom
## 
##   Rule 65/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.445 thick
## 
##   Rule 65/6: [30 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > 0.1863816
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 65/7: [15 cases, mean 0.9, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.493752
## 	chrom <= 0.2075386
##     then
## 	outcome = -5.7 + 3.172 u.size
## 
##   Rule 65/8: [22 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > 0.890325
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
##     then
## 	outcome = 0.8 + 0.089 thick + 0.062 nucl
## 
##   Rule 65/9: [68 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	u.size <= 1.493752
##     then
## 	outcome = 1
## 
##   Rule 65/10: [13 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.1863816
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > 0.6322382
##     then
## 	outcome = 0.8 - 0.122 u.size + 0.112 nucl + 0.082 thick + 0.028 n.nuc
## 	          + 0.011 chrom
## 
##   Rule 65/11: [95 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	chrom > 0.2075386
##     then
## 	outcome = 1
## 
## Model 66:
## 
##   Rule 66/1: [15 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	nucl <= -0.4555328
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0
## 
##   Rule 66/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 66/3: [262 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 66/4: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.2 + 1.022 thick
## 
##   Rule 66/5: [8 cases, mean 0.5, range 0 to 1, est err 0.5]
## 
##     if
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1 + 9.758 u.shape - 1.808 s.size
## 
##   Rule 66/6: [10 cases, mean 0.5, range 0 to 1, est err 0.5]
## 
##     if
## 	thick > -0.5175617
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.4555328
## 	nucl <= 0.3602955
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.7 - 2.608 nucl
## 
##   Rule 66/7: [11 cases, mean 0.5, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= -0.5175617
## 	u.shape > -0.7517401
## 	adhsn <= 0.7209161
## 	nucl > -0.4555328
##     then
## 	outcome = -2.2 - 4.473 thick + 1.243 u.shape - 0.469 n.nuc
## 
##   Rule 66/8: [26 cases, mean 0.7, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > -0.5175617
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	s.size <= 0.7219615
## 	nucl > -0.4555328
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.8 + 0.158 thick + 0.011 nucl + 0.007 n.nuc
## 
##   Rule 66/9: [126 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 0.3602955
##     then
## 	outcome = 1
## 
##   Rule 66/10: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 66/11: [19 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
##     then
## 	outcome = 1 - 0.011 nucl + 0.005 thick
## 
##   Rule 66/12: [40 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
##     then
## 	outcome = 1 - 0.005 thick
## 
##   Rule 66/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 67:
## 
##   Rule 67/1: [226 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 67/2: [191 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl <= -0.4555328
## 	chrom <= -0.6054638
##     then
## 	outcome = 0
## 
##   Rule 67/3: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.1499105
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 67/4: [124 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= -0.8695334
## 	s.size <= -0.1499105
##     then
## 	outcome = 0
## 
##   Rule 67/5: [83 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	s.size <= -0.5858465
## 	nucl <= 0.08835271
## 	chrom > -0.6054638
##     then
## 	outcome = 0
## 
##   Rule 67/6: [47 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	chrom <= -0.6054638
##     then
## 	outcome = -0.1 + 0.62 nucl
## 
##   Rule 67/7: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.1 + 0.935 thick
## 
##   Rule 67/8: [7 cases, mean 0.4, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape <= 1.186563
## 	s.size > -0.1499105
## 	nucl > 0.08835271
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
## 	mit <= 0.1942086
##     then
## 	outcome = 0
## 
##   Rule 67/9: [5 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > -0.1499105
## 	nucl <= 0.08835271
## 	chrom > -0.6054638
## 	n.nuc <= 1.636011
## 	mit <= 0.1942086
##     then
## 	outcome = 0.8 + 0.052 thick + 0.038 chrom + 0.025 n.nuc + 0.023 s.size
## 
##   Rule 67/10: [35 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size <= -0.1499105
## 	nucl > 0.08835271
##     then
## 	outcome = 1
## 
##   Rule 67/11: [76 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	u.shape > -0.7517401
## 	s.size > -0.1499105
##     then
## 	outcome = 1
## 
##   Rule 67/12: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 67/13: [9 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	chrom > -0.6054638
## 	n.nuc <= 1.636011
## 	mit > 0.1942086
##     then
## 	outcome = 1 - 0.031 nucl
## 
##   Rule 67/14: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 67/15: [5 cases, mean 1.0, range 1 to 1, est err 0.3]
## 
##     if
## 	s.size > -0.5858465
## 	s.size <= -0.1499105
## 	nucl > -0.7274755
## 	nucl <= 0.08835271
## 	chrom > -0.6054638
##     then
## 	outcome = 1.1 + 0.792 nucl + 0.16 thick
## 
##   Rule 67/16: [49 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl > 1.176124
## 	chrom > -0.1989626
##     then
## 	outcome = 1.1 - 0.053 nucl
## 
##   Rule 67/17: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 68:
## 
##   Rule 68/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 68/2: [4 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick > -0.5175617
## 	thick <= 0.1863816
## 	u.shape > -0.1056389
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 0
## 
##   Rule 68/3: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 68/4: [194 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= -0.5175617
## 	adhsn <= 0.3774847
## 	nucl <= 1.176124
##     then
## 	outcome = 0
## 
##   Rule 68/5: [13 cases, mean 0.3, range 0 to 1, est err 0.2]
## 
##     if
## 	thick > -0.5175617
## 	thick <= 1.242297
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	chrom <= -0.1989626
##     then
## 	outcome = 1.1 + 2.353 u.shape + 0.083 nucl + 0.012 thick + 0.01 s.size
## 	          + 0.008 chrom
## 
##   Rule 68/6: [8 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	adhsn > 0.3774847
## 	adhsn <= 0.7209161
##     then
## 	outcome = 0.5 + 0.152 nucl + 0.132 thick + 0.063 chrom + 0.059 u.size
## 	          + 0.057 s.size
## 
##   Rule 68/7: [4 cases, mean 0.8, range 0 to 1, est err 2.1]
## 
##     if
## 	thick <= -0.5175617
## 	u.shape > -0.1056389
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 2
## 
##   Rule 68/8: [21 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	chrom > -0.1989626
##     then
## 	outcome = 1
## 
##   Rule 68/9: [53 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 0.1863816
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 68/10: [17 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 0.3774847
## 	nucl > 1.176124
##     then
## 	outcome = 1 + 0.005 nucl
## 
##   Rule 68/11: [4 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1.1 + 0.025 s.size
## 
##   Rule 68/12: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 68/13: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 68/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 69:
## 
##   Rule 69/1: [225 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 69/2: [15 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	adhsn <= 1.064348
## 	nucl > -0.7274755
## 	nucl <= -0.4555328
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0
## 
##   Rule 69/3: [260 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 69/4: [186 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= -0.5175617
## 	chrom <= -0.1989626
##     then
## 	outcome = 0
## 
##   Rule 69/5: [85 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > -0.5175617
## 	u.size <= -0.7129574
##     then
## 	outcome = 0.3 + 0.738 nucl
## 
##   Rule 69/6: [6 cases, mean 0.3, range 0 to 1, est err 0.5]
## 
##     if
## 	u.size <= -0.7129574
## 	chrom > -0.1989626
##     then
## 	outcome = 0
## 
##   Rule 69/7: [21 cases, mean 0.5, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	nucl > -0.4555328
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.5 - 0.841 u.size + 0.42 u.shape + 0.105 nucl + 0.077 thick
## 	          + 0.053 chrom
## 
##   Rule 69/8: [8 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	n.nuc > 1.313705
##     then
## 	outcome = 0.7 + 0.175 nucl + 0.169 u.shape
## 
##   Rule 69/9: [43 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1 + 0.007 nucl + 0.005 thick
## 
##   Rule 69/10: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 69/11: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 69/12: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 69/13: [73 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 1.064348
##     then
## 	outcome = 1
## 
## Model 70:
## 
##   Rule 70/1: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 70/2: [260 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl <= -0.7274755
## 	chrom <= 0.6140398
##     then
## 	outcome = 0
## 
##   Rule 70/3: [11 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > -0.5175617
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	chrom <= 0.6140398
## 	mit <= -0.3671017
##     then
## 	outcome = -0
## 
##   Rule 70/4: [18 cases, mean 0.4, range 0 to 1, est err 0.5]
## 
##     if
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	chrom <= 0.6140398
## 	mit <= -0.3671017
##     then
## 	outcome = 0
## 
##   Rule 70/5: [23 cases, mean 0.5, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	chrom <= 0.6140398
##     then
## 	outcome = 0.8 + 1.016 u.shape + 0.679 chrom + 0.303 thick
## 
##   Rule 70/6: [14 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= 0.2327753
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	chrom > 0.6140398
##     then
## 	outcome = 0.6 + 0.162 nucl + 0.092 thick + 0.053 n.nuc + 0.044 s.size
## 	          + 0.043 chrom + 0.036 u.size
## 
##   Rule 70/7: [5 cases, mean 0.8, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= -0.5175617
## 	u.size > -0.7129574
## 	u.shape > -0.1056389
## 	nucl <= 1.176124
##     then
## 	outcome = 1.3 + 0.34 mit + 0.13 nucl
## 
##   Rule 70/8: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 70/9: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 70/10: [8 cases, mean 1.0, range 1 to 1, est err 0.4]
## 
##     if
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	chrom <= 0.6140398
## 	mit > -0.3671017
##     then
## 	outcome = 0.9 + 0.636 mit
## 
##   Rule 70/11: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 71:
## 
##   Rule 71/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 71/2: [14 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 71/3: [55 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	u.shape > -0.7517401
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.3 + 1.836 s.size + 0.216 u.size - 0.052 u.shape
## 
##   Rule 71/4: [23 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape > -0.7517401
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.1 + 0.1 nucl + 0.047 u.shape + 0.044 u.size + 0.04 thick
## 	          + 0.025 n.nuc + 0.005 s.size
## 
##   Rule 71/5: [8 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.08246895
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
## 	nucl <= 0.9041809
##     then
## 	outcome = -0 + 0.156 nucl + 0.093 u.size + 0.059 n.nuc + 0.034 thick
## 	          + 0.016 s.size + 0.014 adhsn
## 
##   Rule 71/6: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.411 thick
## 
##   Rule 71/7: [22 cases, mean 0.5, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= -0.08246895
## 	u.shape <= 0.2174117
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 71/8: [15 cases, mean 0.7, range 0 to 1, est err 0.5]
## 
##     if
## 	thick > -0.5175617
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= -0.08246895
## 	u.shape > -0.7517401
## 	u.shape <= 0.2174117
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 1.2 + 1.134 u.shape + 0.021 thick
## 
##   Rule 71/9: [9 cases, mean 0.9, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.594268
## 	u.size <= 0.2327753
## 	u.shape > 0.2174117
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 0.5 + 0.957 u.shape - 0.009 adhsn
## 
##   Rule 71/10: [20 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 1.594268
## 	u.size <= 0.2327753
## 	u.shape > -0.7517401
## 	nucl > 0.9041809
##     then
## 	outcome = 1 - 0.015 adhsn + 0.013 u.shape
## 
##   Rule 71/11: [59 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.493752
##     then
## 	outcome = 1
## 
##   Rule 71/12: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 71/13: [12 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	adhsn > 0.7209161
##     then
## 	outcome = 0.9 + 0.054 nucl + 0.043 u.size + 0.02 n.nuc
## 
##   Rule 71/14: [10 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.594268
## 	u.size <= 0.2327753
## 	adhsn <= 0.7209161
##     then
## 	outcome = 1 - 0.01 adhsn + 0.009 u.shape
## 
##   Rule 71/15: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 72:
## 
##   Rule 72/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 72/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 72/3: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.2 + 1.012 thick
## 
##   Rule 72/4: [19 cases, mean 0.3, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	adhsn <= 0.03405333
## 	nucl > -0.7274755
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.3 - 0.82 adhsn + 0.699 chrom + 0.569 thick
## 
##   Rule 72/5: [8 cases, mean 0.5, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1 + 1.681 u.shape - 0.303 s.size + 0.018 nucl + 0.011 thick
## 	          + 0.006 chrom
## 
##   Rule 72/6: [7 cases, mean 0.6, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape <= 1.186563
## 	adhsn > 0.03405333
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.8
## 
##   Rule 72/7: [12 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	chrom > 0.2075386
##     then
## 	outcome = 0.8 + 0.077 nucl + 0.049 thick + 0.033 s.size + 0.027 chrom
## 	          + 0.021 u.size
## 
##   Rule 72/8: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 72/9: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 72/10: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 73:
## 
##   Rule 73/1: [226 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 73/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 73/3: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.1 + 0.934 thick
## 
##   Rule 73/4: [12 cases, mean 0.2, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	chrom <= -0.1989626
##     then
## 	outcome = 1.5 + 3.094 u.shape + 0.789 chrom
## 
##   Rule 73/5: [11 cases, mean 0.5, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = -0.7 - 1.815 thick + 0.893 nucl
## 
##   Rule 73/6: [9 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	chrom > -0.1989626
##     then
## 	outcome = 1 + 0.95 u.shape + 0.093 n.nuc + 0.067 thick + 0.043 chrom
## 	          + 0.02 nucl + 0.007 s.size + 0.005 u.size
## 
##   Rule 73/7: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 73/8: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 73/9: [9 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	n.nuc > 1.636011
##     then
## 	outcome = 0.9 + 0.04 n.nuc
## 
##   Rule 73/10: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 73/11: [4 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1 + 0.033 nucl + 0.014 thick + 0.012 s.size + 0.01 n.nuc
## 	          + 0.009 u.size + 0.009 chrom
## 
##   Rule 73/12: [89 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	nucl > 1.176124
## 	chrom > -0.1989626
##     then
## 	outcome = 1
## 
##   Rule 73/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 74:
## 
##   Rule 74/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 74/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 74/3: [237 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= -0.16559
## 	u.shape <= 0.2174117
## 	adhsn <= 1.064348
## 	s.size <= 0.2860255
##     then
## 	outcome = -0
## 
##   Rule 74/4: [278 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= -0.3977132
## 	s.size <= 0.2860255
##     then
## 	outcome = -0
## 
##   Rule 74/5: [11 cases, mean 0.5, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 0.2174117
## 	adhsn <= 1.064348
## 	s.size <= 0.2860255
## 	n.nuc > 0.3467855
##     then
## 	outcome = -0.2 + 0.254 nucl + 0.144 thick + 0.143 chrom + 0.066 s.size
## 
##   Rule 74/6: [10 cases, mean 0.6, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	chrom <= 1.427042
## 	mit <= 0.1942086
##     then
## 	outcome = 1.9 - 0.916 u.shape - 0.905 s.size + 0.006 nucl
## 
##   Rule 74/7: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 74/8: [8 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 0.2174117
## 	adhsn > 1.064348
## 	nucl > -0.7274755
## 	chrom <= 1.427042
##     then
## 	outcome = 0.9 + 0.07 nucl
## 
##   Rule 74/9: [42 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.2174117
## 	s.size <= 0.2860255
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 74/10: [4 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	thick > -0.16559
## 	thick <= 1.242297
## 	u.size > -0.3977132
## 	u.shape <= 0.2174117
## 	adhsn <= 1.064348
## 	s.size <= 0.2860255
## 	nucl > -0.7274755
## 	n.nuc <= 0.3467855
##     then
## 	outcome = 1.1
## 
##   Rule 74/11: [5 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	mit > 0.1942086
##     then
## 	outcome = 1.1
## 
##   Rule 74/12: [10 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 74/13: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 74/14: [42 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	chrom > 1.427042
##     then
## 	outcome = 1
## 
##   Rule 74/15: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 75:
## 
##   Rule 75/1: [235 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.4286895
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 75/2: [179 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.5175617
## 	u.shape <= -0.4286895
##     then
## 	outcome = 0
## 
##   Rule 75/3: [260 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 75/4: [10 cases, mean 0.2, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > -0.5175617
## 	u.shape <= -0.4286895
## 	s.size <= -0.5858465
## 	nucl > -0.7274755
##     then
## 	outcome = -2.7 - 4.327 s.size + 0.432 thick - 0.374 u.shape + 0.229 nucl
## 
##   Rule 75/5: [7 cases, mean 0.3, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.08246895
## 	u.size <= 0.2327753
## 	nucl <= 0.9041809
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0
## 
##   Rule 75/6: [4 cases, mean 0.5, range 0 to 1, est err 0.8]
## 
##     if
## 	u.size <= 0.2327753
## 	u.shape > -0.4286895
## 	nucl <= -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1 + 0.156 nucl + 0.089 thick + 0.086 u.size + 0.053 n.nuc
## 	          + 0.038 chrom + 0.033 s.size
## 
##   Rule 75/7: [9 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 1 + 0.012 nucl + 0.008 thick + 0.006 chrom + 0.006 n.nuc
## 
##   Rule 75/8: [17 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= -0.08246895
## 	u.shape > -0.4286895
## 	nucl > -0.7274755
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1.1
## 
##   Rule 75/9: [17 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	u.shape > -0.4286895
## 	nucl > 0.9041809
##     then
## 	outcome = 3 - 1.167 nucl
## 
##   Rule 75/10: [144 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > -0.5175617
## 	s.size > -0.5858465
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 75/11: [81 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 0.9913985
##     then
## 	outcome = 1
## 
##   Rule 75/12: [115 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.4286895
## 	chrom > 0.2075386
##     then
## 	outcome = 1
## 
##   Rule 75/13: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 75/14: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
## Model 76:
## 
##   Rule 76/1: [239 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.5858465
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 76/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 76/3: [60 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size <= -0.5858465
## 	nucl <= 0.9041809
##     then
## 	outcome = 0.2 + 0.693 nucl + 0.107 n.nuc
## 
##   Rule 76/4: [11 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.1863816
## 	u.size <= 0.2327753
## 	u.shape > -0.7517401
## 	s.size > -0.5858465
## 	nucl <= 0.9041809
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.1 + 0.261 thick + 0.111 s.size
## 
##   Rule 76/5: [7 cases, mean 0.7, range 0 to 1, est err 0.5]
## 
##     if
## 	u.size <= 0.2327753
## 	u.shape <= 1.186563
## 	nucl <= 0.9041809
## 	n.nuc > 0.9913985
##     then
## 	outcome = 0.4 + 0.189 thick + 0.183 nucl + 0.099 n.nuc + 0.073 u.size
## 	          + 0.057 s.size
## 
##   Rule 76/6: [11 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > 0.9041809
## 	chrom <= 0.2075386
##     then
## 	outcome = -3.4 + 2.74 nucl - 1.494 thick
## 
##   Rule 76/7: [10 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > 0.1863816
## 	u.size <= 0.2327753
## 	u.shape > -0.7517401
## 	s.size > -0.5858465
## 	nucl <= 0.9041809
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.8 + 0.193 nucl + 0.157 thick + 0.104 n.nuc + 0.076 u.size
## 	          + 0.071 s.size
## 
##   Rule 76/8: [37 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	nucl <= 0.9041809
##     then
## 	outcome = 1
## 
##   Rule 76/9: [13 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	u.shape <= 1.186563
## 	nucl > 0.9041809
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.6 + 0.173 nucl + 0.05 thick + 0.032 n.nuc + 0.028 s.size
## 	          + 0.027 chrom + 0.022 u.size
## 
##   Rule 76/10: [45 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl > 0.9041809
## 	chrom > 0.2075386
##     then
## 	outcome = 0.9 + 0.063 nucl + 0.007 thick
## 
##   Rule 76/11: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 77:
## 
##   Rule 77/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 77/2: [252 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.5858465
## 	nucl <= -0.4555328
##     then
## 	outcome = 0
## 
##   Rule 77/3: [7 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	s.size > -0.5858465
## 	nucl <= -0.4555328
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.1 + 0.155 nucl - 0.061 u.size + 0.06 s.size
## 
##   Rule 77/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.498 thick
## 
##   Rule 77/5: [10 cases, mean 0.5, range 0 to 1, est err 0.7]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
## 	n.nuc <= 0.02447898
##     then
## 	outcome = 1.9 + 2.83 u.size + 2.453 nucl - 2.084 chrom
## 
##   Rule 77/6: [29 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.4555328
## 	nucl <= 1.176124
##     then
## 	outcome = 0.2 + 0.31 nucl + 0.214 thick - 0.122 u.size + 0.12 s.size
## 
##   Rule 77/7: [9 cases, mean 0.8, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.5 + 0.392 adhsn
## 
##   Rule 77/8: [11 cases, mean 0.8, range 0 to 1, est err 0.5]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 1.176124
## 	n.nuc > 0.02447898
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 2.1 - 1.618 n.nuc - 0.412 s.size + 0.257 nucl
## 
##   Rule 77/9: [13 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size <= 0.2327753
## 	n.nuc > 0.9913985
##     then
## 	outcome = 0.9 + 0.019 nucl + 0.014 thick + 0.007 n.nuc + 0.006 s.size
## 	          + 0.005 chrom
## 
##   Rule 77/10: [22 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > 1.176124
##     then
## 	outcome = 0.7 + 0.103 nucl + 0.051 thick + 0.036 n.nuc + 0.035 chrom
## 	          + 0.006 s.size
## 
##   Rule 77/11: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
## Model 78:
## 
##   Rule 78/1: [226 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 78/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 78/3: [270 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	adhsn <= 0.7209161
## 	nucl <= -0.4555328
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0
## 
##   Rule 78/4: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.1 + 0.931 thick
## 
##   Rule 78/5: [13 cases, mean 0.4, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= -0.16559
## 	u.shape > -0.7517401
## 	u.shape <= 0.5404623
## 	adhsn <= 0.7209161
## 	nucl > -0.4555328
##     then
## 	outcome = -0.5 - 1.069 thick + 0.812 nucl
## 
##   Rule 78/6: [9 cases, mean 0.6, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > -0.16559
## 	thick <= 0.5383533
## 	u.shape <= 1.186563
## 	adhsn > -0.3093781
## 	adhsn <= 0.7209161
## 	nucl > -0.4555328
##     then
## 	outcome = 0.1 + 0.371 chrom
## 
##   Rule 78/7: [8 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > 0.5404623
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
##     then
## 	outcome = 0.9 + 0.025 nucl + 0.014 thick + 0.011 n.nuc + 0.009 u.size
## 	          + 0.007 u.shape + 0.006 chrom
## 
##   Rule 78/8: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 78/9: [6 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > -0.16559
## 	thick <= 0.5383533
## 	u.shape <= 0.5404623
## 	adhsn <= -0.3093781
## 	nucl > -0.4555328
##     then
## 	outcome = 1.1
## 
##   Rule 78/10: [6 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.5383533
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc > 1.313705
##     then
## 	outcome = 1.1
## 
##   Rule 78/11: [4 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.9 + 0.101 nucl + 0.069 s.size + 0.051 thick + 0.026 chrom
## 	          + 0.023 n.nuc + 0.019 u.size
## 
##   Rule 78/12: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 78/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 79:
## 
##   Rule 79/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 79/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 79/3: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 0.1 + 0.099 nucl + 0.056 u.size + 0.052 thick + 0.039 n.nuc
## 	          + 0.026 s.size
## 
##   Rule 79/4: [15 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= -0.08246895
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.8 + 0.854 n.nuc + 0.497 thick + 0.03 nucl
## 
##   Rule 79/5: [11 cases, mean 0.5, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = -0.3 - 1.417 thick + 0.506 nucl
## 
##   Rule 79/6: [4 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.08246895
## 	u.shape <= -0.1056389
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.6 + 0.257 n.nuc + 0.111 nucl + 0.097 thick
## 
##   Rule 79/7: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 79/8: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 79/9: [8 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc > 1.636011
##     then
## 	outcome = 1.2 - 0.069 n.nuc + 0.021 nucl
## 
##   Rule 79/10: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 79/11: [10 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 79/12: [77 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	s.size > -0.1499105
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 79/13: [90 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.08246895
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 79/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 80:
## 
##   Rule 80/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 80/2: [179 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.5175617
## 	u.shape <= -0.4286895
##     then
## 	outcome = 0
## 
##   Rule 80/3: [6 cases, mean 0.3, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= 0.2327753
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.8 + 0.938 u.size + 0.693 chrom + 0.213 s.size
## 
##   Rule 80/4: [19 cases, mean 0.5, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > -0.5175617
## 	u.size <= 0.2327753
## 	s.size <= -0.5858465
## 	nucl > -0.7274755
##     then
## 	outcome = -2.3 - 3.966 s.size + 0.551 nucl
## 
##   Rule 80/5: [16 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > -0.5175617
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	s.size > -0.5858465
## 	nucl > -0.7274755
## 	n.nuc <= 1.636011
##     then
## 	outcome = 1 - 0.532 n.nuc
## 
##   Rule 80/6: [165 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.4286895
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 80/7: [144 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > -0.5175617
## 	s.size > -0.5858465
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 80/8: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
## Model 81:
## 
##   Rule 81/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 81/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 81/3: [38 cases, mean 0.6, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.594268
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 1.064348
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 0.4 + 0.181 n.nuc + 0.157 thick + 0.149 chrom
## 
##   Rule 81/4: [11 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size <= -0.08246895
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 0.6 + 0.155 nucl + 0.078 thick + 0.069 n.nuc + 0.063 s.size
## 
##   Rule 81/5: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 81/6: [6 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	thick > 1.594268
## 	s.size <= -0.1499105
## 	nucl <= 1.176124
##     then
## 	outcome = 0.8 - 1.441 s.size
## 
##   Rule 81/7: [50 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.594268
##     then
## 	outcome = 1
## 
##   Rule 81/8: [4 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1 + 0.024 nucl + 0.012 thick + 0.011 n.nuc + 0.01 s.size
## 
##   Rule 81/9: [73 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 1.064348
##     then
## 	outcome = 1
## 
##   Rule 81/10: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 82:
## 
##   Rule 82/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 82/2: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.4555328
## 	n.nuc <= -0.6201341
##     then
## 	outcome = 0
## 
##   Rule 82/3: [53 cases, mean 0.2, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	nucl <= 1.448066
##     then
## 	outcome = -0.1
## 
##   Rule 82/4: [15 cases, mean 0.3, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= -0.4555328
## 	n.nuc > -0.6201341
##     then
## 	outcome = 0.3 + 0.418 n.nuc + 0.263 adhsn + 0.248 thick
## 
##   Rule 82/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.454 thick
## 
##   Rule 82/6: [13 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.3977132
## 	u.size <= 0.2327753
## 	nucl > -0.4555328
## 	nucl <= 1.448066
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1 - 2.674 u.size + 0.288 nucl
## 
##   Rule 82/7: [8 cases, mean 0.8, range 0 to 1, est err 0.9]
## 
##     if
## 	thick <= 0.5383533
## 	nucl > 0.9041809
## 	nucl <= 1.176124
##     then
## 	outcome = 1.6 + 6.552 thick - 2.854 u.size
## 
##   Rule 82/8: [81 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 0.9913985
##     then
## 	outcome = 1
## 
##   Rule 82/9: [92 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.448066
##     then
## 	outcome = 1
## 
##   Rule 82/10: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 82/11: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 82/12: [37 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
## 	nucl <= 0.9041809
##     then
## 	outcome = 1
## 
##   Rule 82/13: [6 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	adhsn > 1.064348
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1 + 0.013 nucl + 0.009 thick + 0.005 n.nuc
## 
##   Rule 82/14: [7 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	nucl > 0.9041809
## 	nucl <= 1.176124
##     then
## 	outcome = 1.3 - 0.17 thick + 0.008 nucl
## 
##   Rule 82/15: [6 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	nucl > -0.4555328
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1.1
## 
## Model 83:
## 
##   Rule 83/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 83/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 83/3: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 0.1 + 0.107 nucl + 0.071 u.size + 0.06 thick + 0.041 n.nuc
## 	          + 0.023 chrom
## 
##   Rule 83/4: [45 cases, mean 0.2, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= -0.1056389
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 0.9 + 1.486 u.shape + 0.244 nucl + 0.162 thick
## 
##   Rule 83/5: [8 cases, mean 0.4, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.1863816
## 	u.shape > -0.1056389
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = -0.7 - 1.868 thick + 1.01 nucl
## 
##   Rule 83/6: [8 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	adhsn > 0.3774847
## 	adhsn <= 0.7209161
##     then
## 	outcome = 0.5 + 0.174 nucl + 0.093 thick + 0.065 n.nuc + 0.047 chrom
## 	          + 0.042 u.size + 0.042 s.size
## 
##   Rule 83/7: [113 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.1863816
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 83/8: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 83/9: [4 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1.1 + 0.01 nucl + 0.005 thick
## 
##   Rule 83/10: [19 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
##     then
## 	outcome = 0.8 + 0.099 thick + 0.038 mit + 0.008 adhsn
## 
##   Rule 83/11: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 83/12: [6 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
## 	s.size > 2.465705
##     then
## 	outcome = 0.8 + 0.052 nucl + 0.033 thick + 0.025 u.size + 0.023 n.nuc
## 
##   Rule 83/13: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 84:
## 
##   Rule 84/1: [225 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 84/2: [170 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.5175617
## 	u.size <= -0.7129574
##     then
## 	outcome = -0
## 
##   Rule 84/3: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 84/4: [85 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > -0.5175617
## 	u.size <= -0.7129574
##     then
## 	outcome = 0.3 + 0.722 nucl
## 
##   Rule 84/5: [4 cases, mean 0.5, range 0 to 1, est err 0.5]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.7 + 0.204 nucl - 0.144 u.size + 0.143 u.shape + 0.128 thick
## 	          + 0.108 chrom + 0.088 mit
## 
##   Rule 84/6: [10 cases, mean 0.5, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
## 	mit > -0.3671017
##     then
## 	outcome = 0.3 - 0.612 u.size + 0.393 u.shape + 0.202 nucl
## 
##   Rule 84/7: [13 cases, mean 0.5, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	mit <= -0.3671017
##     then
## 	outcome = 1 + 1.607 u.size + 0.766 thick + 0.457 nucl
## 
##   Rule 84/8: [29 cases, mean 0.6, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.1863816
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
##     then
## 	outcome = 0.1 + 0.514 nucl - 0.051 u.size + 0.033 u.shape
## 
##   Rule 84/9: [32 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > 0.2327753
## 	chrom <= 0.2075386
##     then
## 	outcome = 0.9 + 0.032 nucl + 0.028 thick + 0.019 chrom + 0.017 u.shape
## 
##   Rule 84/10: [113 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.1863816
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 84/11: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 84/12: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
## Model 85:
## 
##   Rule 85/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 85/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 85/3: [9 cases, mean 0.1, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= -0.4286895
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 0.1 + 0.137 thick + 0.109 nucl
## 
##   Rule 85/4: [46 cases, mean 0.2, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	chrom <= 0.2075386
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.1 + 0.243 u.shape + 0.117 chrom
## 
##   Rule 85/5: [5 cases, mean 0.6, range 0 to 1, est err 1.0]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.4286895
## 	u.shape <= 0.5404623
## 	s.size <= -0.5858465
## 	nucl > -0.7274755
## 	chrom <= 0.2075386
##     then
## 	outcome = 1.3
## 
##   Rule 85/6: [38 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 0.9 + 0.046 thick + 0.036 nucl
## 
##   Rule 85/7: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.5404623
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 85/8: [115 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.4286895
## 	chrom > 0.2075386
##     then
## 	outcome = 1
## 
##   Rule 85/9: [71 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.313705
##     then
## 	outcome = 1
## 
##   Rule 85/10: [4 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1.1
## 
##   Rule 85/11: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 85/12: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 86:
## 
##   Rule 86/1: [225 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 86/2: [170 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= -0.5175617
## 	u.size <= -0.7129574
##     then
## 	outcome = -0
## 
##   Rule 86/3: [14 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 86/4: [269 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= -0.1056389
## 	nucl <= -0.4555328
##     then
## 	outcome = 0 + 0.014 u.shape + 0.009 nucl
## 
##   Rule 86/5: [85 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > -0.5175617
## 	u.size <= -0.7129574
##     then
## 	outcome = 0.3 + 0.718 nucl
## 
##   Rule 86/6: [60 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	chrom <= -0.1989626
##     then
## 	outcome = -0
## 
##   Rule 86/7: [14 cases, mean 0.3, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = -0.6 + 0.168 nucl + 0.109 u.size + 0.095 thick + 0.067 n.nuc
## 	          + 0.041 chrom
## 
##   Rule 86/8: [4 cases, mean 0.8, range 0 to 1, est err 1.0]
## 
##     if
## 	thick <= -0.5175617
## 	u.size > -0.7129574
## 	u.shape > -0.1056389
## 	adhsn <= 0.7209161
## 	nucl <= 1.176124
##     then
## 	outcome = 1.3 + 0.144 thick + 0.129 nucl + 0.114 n.nuc + 0.086 adhsn
## 
##   Rule 86/9: [10 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl <= 1.176124
## 	chrom > -0.1989626
##     then
## 	outcome = 0.9 + 0.152 thick + 0.152 nucl + 0.086 n.nuc + 0.046 adhsn
## 	          + 0.029 chrom + 0.009 u.shape
## 
##   Rule 86/10: [105 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	u.shape > -0.7517401
##     then
## 	outcome = 1
## 
##   Rule 86/11: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 86/12: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 86/13: [11 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
##     then
## 	outcome = 1.1 - 0.044 thick - 0.023 nucl
## 
##   Rule 86/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
##   Rule 86/15: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 86/16: [90 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.08246895
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
## Model 87:
## 
##   Rule 87/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 87/2: [24 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	s.size <= -0.5858465
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.1 + 0.075 nucl + 0.042 thick + 0.026 u.shape + 0.024 n.nuc
## 	          + 0.007 s.size + 0.006 adhsn
## 
##   Rule 87/3: [35 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.1 + 0.737 s.size
## 
##   Rule 87/4: [19 cases, mean 0.3, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
## 	nucl <= 0.6322382
## 	n.nuc <= 1.313705
##     then
## 	outcome = 0.2 + 0.426 thick + 0.034 nucl + 0.021 u.shape + 0.007 n.nuc
## 
##   Rule 87/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.41 thick
## 
##   Rule 87/6: [7 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	n.nuc > 1.313705
##     then
## 	outcome = 0.6 + 0.203 u.shape + 0.185 nucl
## 
##   Rule 87/7: [122 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 0.6322382
##     then
## 	outcome = 1
## 
##   Rule 87/8: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1 + 0.006 nucl + 0.005 thick
## 
##   Rule 87/9: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
## Model 88:
## 
##   Rule 88/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 88/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 88/3: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.2 + 1.011 thick
## 
##   Rule 88/4: [14 cases, mean 0.3, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= -0.16559
## 	u.shape > -0.7517401
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = -0.2 + 0.742 nucl - 0.658 thick
## 
##   Rule 88/5: [8 cases, mean 0.5, range 0 to 1, est err 0.2]
## 
##     if
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1 + 4.513 u.shape - 0.836 s.size
## 
##   Rule 88/6: [14 cases, mean 0.6, range 0 to 1, est err 0.4]
## 
##     if
## 	thick > -0.16559
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.2 + 0.885 thick - 0.059 adhsn + 0.04 chrom + 0.026 n.nuc
## 
##   Rule 88/7: [10 cases, mean 0.7, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.3774847
## 	adhsn <= 0.7209161
##     then
## 	outcome = 0.8 + 0.104 nucl + 0.007 thick + 0.005 s.size
## 
##   Rule 88/8: [28 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > 1.176124
##     then
## 	outcome = 1 - 0.005 adhsn
## 
##   Rule 88/9: [8 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc > 1.636011
##     then
## 	outcome = 0.7 - 0.238 adhsn + 0.161 chrom + 0.105 n.nuc
## 
##   Rule 88/10: [19 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
##     then
## 	outcome = 1 + 0.007 thick
## 
##   Rule 88/11: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 88/12: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 89:
## 
##   Rule 89/1: [226 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 89/2: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 89/3: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.1 + 0.934 thick
## 
##   Rule 89/4: [7 cases, mean 0.3, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.08246895
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 0 + 0.343 nucl + 0.152 adhsn + 0.078 u.size
## 
##   Rule 89/5: [20 cases, mean 0.5, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= -0.08246895
## 	u.shape > -0.7517401
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 0.8 + 1.101 u.size + 0.487 nucl + 0.32 thick + 0.178 n.nuc
## 
##   Rule 89/6: [8 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > 0.5383533
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 0.8 + 0.06 nucl + 0.03 n.nuc + 0.02 thick + 0.016 u.size
## 	          + 0.01 s.size + 0.006 chrom
## 
##   Rule 89/7: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.3774847
##     then
## 	outcome = 1
## 
##   Rule 89/8: [101 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 89/9: [4 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.8 + 0.091 s.size + 0.078 nucl + 0.037 thick + 0.028 u.size
## 	          + 0.028 n.nuc + 0.013 chrom
## 
##   Rule 89/10: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 89/11: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 89/12: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 90:
## 
##   Rule 90/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 90/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 90/3: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 0.1 + 0.072 nucl + 0.042 thick + 0.031 n.nuc + 0.028 u.shape
## 	          + 0.022 s.size + 0.016 chrom + 0.012 adhsn
## 
##   Rule 90/4: [12 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.1056389
## 	adhsn <= 0.3774847
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = -0.2
## 
##   Rule 90/5: [16 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.6 + 0.549 chrom + 0.545 n.nuc + 0.34 thick
## 
##   Rule 90/6: [7 cases, mean 0.6, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	adhsn > 0.3774847
## 	adhsn <= 0.7209161
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0.5 + 0.162 nucl + 0.08 thick + 0.057 n.nuc + 0.042 s.size
## 	          + 0.039 u.shape + 0.032 chrom + 0.017 adhsn
## 
##   Rule 90/7: [4 cases, mean 0.8, range 0 to 1, est err 1.0]
## 
##     if
## 	thick <= -0.5175617
## 	u.shape > -0.1056389
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 1.3 - 0.212 adhsn + 0.111 chrom + 0.096 nucl + 0.058 n.nuc
## 
##   Rule 90/8: [106 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	s.size > 0.2860255
##     then
## 	outcome = 1
## 
##   Rule 90/9: [32 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	adhsn <= 0.3774847
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 90/10: [5 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	thick > 0.5383533
## 	thick <= 1.242297
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = 1.1
## 
##   Rule 90/11: [8 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc > 1.636011
##     then
## 	outcome = 1 - 0.012 adhsn + 0.006 chrom + 0.005 nucl
## 
##   Rule 90/12: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 90/13: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 90/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 91:
## 
##   Rule 91/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 91/2: [253 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= -0.1056389
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 91/3: [37 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 0.1863816
## 	u.size > -0.7129574
## 	adhsn <= 0.3774847
## 	nucl <= 1.176124
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.1 + 0.261 nucl + 0.022 u.size + 0.006 thick
## 
##   Rule 91/4: [7 cases, mean 0.3, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.08246895
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 0 + 0.204 nucl + 0.147 thick + 0.055 n.nuc + 0.048 chrom
## 
##   Rule 91/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.474 thick
## 
##   Rule 91/6: [8 cases, mean 0.6, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	adhsn > 0.3774847
## 	adhsn <= 0.7209161
##     then
## 	outcome = 0.6 + 0.144 nucl + 0.069 u.size + 0.06 thick + 0.048 n.nuc
## 	          + 0.026 chrom
## 
##   Rule 91/7: [8 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 0.1863816
## 	u.size > -0.7129574
## 	u.size <= -0.08246895
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1 + 0.53 u.shape + 0.006 thick + 0.005 nucl
## 
##   Rule 91/8: [5 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	nucl <= -0.7274755
##     then
## 	outcome = 1.3 - 1.215 u.shape + 0.045 u.size
## 
##   Rule 91/9: [15 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > 0.1863816
## 	u.size <= -0.08246895
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 91/10: [97 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 91/11: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 91/12: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 91/13: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 91/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 92:
## 
##   Rule 92/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 92/2: [252 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0
## 
##   Rule 92/3: [13 cases, mean 0.2, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.size <= 0.2327753
## 	mit > -0.3671017
##     then
## 	outcome = 0 + 0.076 nucl + 0.058 thick + 0.022 u.shape + 0.012 n.nuc
## 	          + 0.011 s.size + 0.009 chrom
## 
##   Rule 92/4: [47 cases, mean 0.2, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.5383533
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= 1.448066
## 	n.nuc <= 0.9913985
## 	mit <= -0.3671017
##     then
## 	outcome = 0.3 + 0.665 nucl + 0.019 thick
## 
##   Rule 92/5: [6 cases, mean 0.3, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.2 + 0.212 nucl - 0.158 u.size + 0.115 u.shape + 0.089 mit
## 	          + 0.079 thick + 0.06 n.nuc + 0.058 chrom
## 
##   Rule 92/6: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.467 thick
## 
##   Rule 92/7: [13 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	u.size <= 0.2327753
## 	n.nuc > 0.9913985
##     then
## 	outcome = -3.8 + 2.097 n.nuc + 0.34 nucl
## 
##   Rule 92/8: [59 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.493752
##     then
## 	outcome = 1
## 
##   Rule 92/9: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 92/10: [92 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.448066
##     then
## 	outcome = 1
## 
##   Rule 92/11: [50 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	nucl > -0.7274755
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 1
## 
## Model 93:
## 
##   Rule 93/1: [203 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	nucl <= 0.08835271
## 	chrom <= -0.6054638
##     then
## 	outcome = -0
## 
##   Rule 93/2: [53 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	s.size <= -0.5858465
## 	nucl <= 1.176124
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.1 + 0.209 n.nuc
## 
##   Rule 93/3: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 93/4: [11 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= -0.7129574
## 	u.shape > -0.7517401
## 	chrom > -0.6054638
##     then
## 	outcome = 1.2 + 1.971 nucl - 1.318 chrom
## 
##   Rule 93/5: [30 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.1863816
## 	u.size > -0.7129574
## 	u.shape > -0.7517401
## 	adhsn <= 0.03405333
## 	nucl <= 1.176124
## 	mit <= 0.1942086
##     then
## 	outcome = 0.1 + 0.184 thick + 0.143 adhsn + 0.131 n.nuc
## 
##   Rule 93/6: [6 cases, mean 0.2, range 0 to 1, est err 0.5]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.3 + 1.029 thick
## 
##   Rule 93/7: [9 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 1.178508
## 	u.shape <= 1.186563
## 	adhsn > 0.03405333
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
## 	mit <= 0.1942086
##     then
## 	outcome = 0.9
## 
##   Rule 93/8: [5 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 0.08835271
## 	chrom <= -0.6054638
##     then
## 	outcome = 0.7 + 0.176 nucl + 0.024 thick + 0.018 s.size + 0.017 n.nuc
## 
##   Rule 93/9: [13 cases, mean 0.8, range 0 to 1, est err 0.2]
## 
##     if
## 	thick > 0.1863816
## 	u.size <= 1.178508
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
## 	mit <= 0.1942086
##     then
## 	outcome = 1 + 0.03 nucl + 0.015 thick + 0.011 s.size + 0.011 n.nuc
## 
##   Rule 93/10: [30 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	s.size > -0.5858465
## 	chrom > -0.6054638
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 1
## 
##   Rule 93/11: [59 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	chrom > -0.6054638
## 	mit > 0.1942086
##     then
## 	outcome = 1
## 
##   Rule 93/12: [59 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > 1.176124
##     then
## 	outcome = 0.9 + 0.036 nucl + 0.024 thick + 0.017 s.size + 0.011 n.nuc
## 
##   Rule 93/13: [77 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.178508
##     then
## 	outcome = 1
## 
##   Rule 93/14: [11 cases, mean 1.0, range 1 to 1, est err 0.1]
## 
##     if
## 	thick > 1.242297
## 	u.shape <= 1.186563
## 	nucl <= 1.176124
##     then
## 	outcome = 1 - 0.098 u.shape + 0.097 u.size + 0.066 nucl
## 
##   Rule 93/15: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 94:
## 
##   Rule 94/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = -0
## 
##   Rule 94/2: [35 cases, mean 0.1, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	nucl <= -0.7274755
##     then
## 	outcome = 0.1 + 0.08 nucl + 0.067 u.shape + 0.039 mit + 0.024 thick
## 
##   Rule 94/3: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.462 thick
## 
##   Rule 94/4: [8 cases, mean 0.4, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= -0.3977132
## 	nucl > -0.7274755
## 	nucl <= 0.9041809
##     then
## 	outcome = 0
## 
##   Rule 94/5: [12 cases, mean 0.4, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.3977132
## 	u.size <= 0.2327753
## 	nucl > -0.7274755
## 	nucl <= 0.9041809
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.7 - 3.69 u.size + 0.755 thick + 0.524 n.nuc
## 
##   Rule 94/6: [9 cases, mean 0.8, range 0 to 1, est err 0.4]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 0.2327753
## 	n.nuc > 0.9913985
##     then
## 	outcome = -2.8 + 1.94 n.nuc
## 
##   Rule 94/7: [10 cases, mean 0.9, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	adhsn <= 1.064348
## 	nucl > 0.9041809
## 	n.nuc <= 0.9913985
##     then
## 	outcome = 0.5 - 1.074 u.size + 0.856 u.shape
## 
##   Rule 94/8: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1 + 0.007 nucl
## 
##   Rule 94/9: [15 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
## 	u.size <= 0.2327753
##     then
## 	outcome = 1 - 0.09 u.size - 0.022 n.nuc
## 
##   Rule 94/10: [10 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size <= 0.2327753
## 	adhsn > 1.064348
##     then
## 	outcome = 1 + 0.019 u.shape + 0.013 nucl
## 
## Model 95:
## 
##   Rule 95/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 95/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 95/3: [25 cases, mean 0.2, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= -0.5175617
## 	u.size > -0.7129574
## 	u.shape <= 0.2174117
##     then
## 	outcome = 0.6 + 1.104 u.shape + 0.235 adhsn + 0.09 n.nuc
## 
##   Rule 95/4: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.408 thick
## 
##   Rule 95/5: [64 cases, mean 0.6, range 0 to 1, est err 0.3]
## 
##     if
## 	thick > -0.5175617
## 	u.size > -0.7129574
## 	u.shape <= 0.2174117
##     then
## 	outcome = 0.2 + 0.326 thick + 0.131 chrom + 0.07 nucl + 0.058 u.shape
## 	          + 0.042 u.size + 0.022 n.nuc
## 
##   Rule 95/6: [26 cases, mean 0.7, range 0 to 1, est err 0.5]
## 
##     if
## 	thick > -0.5175617
## 	thick <= 1.594268
## 	u.size > -0.7129574
## 	u.size <= 0.2327753
## 	u.shape <= 0.2174117
## 	chrom > -0.6054638
##     then
## 	outcome = 0.7 - 1.282 u.size + 1.122 u.shape + 0.068 thick + 0.039 chrom
## 
##   Rule 95/7: [12 cases, mean 0.8, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 1.178508
## 	u.shape > 0.2174117
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.1 + 0.579 chrom + 0.019 thick
## 
##   Rule 95/8: [23 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.2174117
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 1
## 
##   Rule 95/9: [127 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > 0.2327753
##     then
## 	outcome = 1
## 
##   Rule 95/10: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 95/11: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 95/12: [31 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.178508
## 	nucl <= 1.176124
## 	n.nuc > -0.2978275
##     then
## 	outcome = 1 - 0.007 nucl
## 
##   Rule 95/13: [50 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.594268
##     then
## 	outcome = 1
## 
## Model 96:
## 
##   Rule 96/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = -0
## 
##   Rule 96/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = -0
## 
##   Rule 96/3: [6 cases, mean 0.2, range 0 to 1, est err 0.6]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.5 + 1.217 thick
## 
##   Rule 96/4: [18 cases, mean 0.4, range 0 to 1, est err 0.1]
## 
##     if
## 	thick <= 0.1863816
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0 + 0.609 nucl
## 
##   Rule 96/5: [4 cases, mean 0.8, range 0 to 1, est err 0.6]
## 
##     if
## 	thick > 0.1863816
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 1.1
## 
##   Rule 96/6: [85 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > -0.7274755
## 	n.nuc > -0.2978275
##     then
## 	outcome = 0.9 + 0.049 thick + 0.033 n.nuc + 0.028 chrom
## 
##   Rule 96/7: [50 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
##     then
## 	outcome = 0.9 + 0.027 s.size - 0.017 u.size
## 
##   Rule 96/8: [59 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	nucl > 1.176124
##     then
## 	outcome = 1 + 0.016 nucl + 0.007 thick + 0.007 n.nuc
## 
##   Rule 96/9: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 97:
## 
##   Rule 97/1: [254 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 97/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 97/3: [299 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape <= 0.5404623
## 	adhsn <= 0.7209161
## 	nucl <= 0.6322382
## 	n.nuc <= 1.313705
##     then
## 	outcome = -0.9
## 
##   Rule 97/4: [14 cases, mean 0.3, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 0.5383533
## 	u.shape > -0.7517401
## 	u.shape <= 0.5404623
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	n.nuc <= -0.2978275
##     then
## 	outcome = 0.5 + 0.477 u.shape + 0.419 chrom + 0.333 thick + 0.017 nucl
## 	          + 0.008 s.size + 0.005 n.nuc
## 
##   Rule 97/5: [123 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > -0.7517401
## 	nucl > 0.6322382
##     then
## 	outcome = 1
## 
##   Rule 97/6: [38 cases, mean 1.0, range 0 to 1, est err 0.1]
## 
##     if
## 	thick > 0.5383533
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
##     then
## 	outcome = 0.9 + 0.024 thick + 0.022 nucl
## 
##   Rule 97/7: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape > 0.5404623
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 97/8: [71 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.313705
##     then
## 	outcome = 1
## 
##   Rule 97/9: [4 cases, mean 1.0, range 1 to 1, est err 0.2]
## 
##     if
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	s.size > 0.2860255
## 	nucl <= -0.7274755
##     then
## 	outcome = 1.1 - 0.141 u.shape
## 
##   Rule 97/10: [40 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn > 0.7209161
##     then
## 	outcome = 1 - 0.015 thick + 0.015 adhsn + 0.014 u.shape - 0.014 nucl
## 
##   Rule 97/11: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 98:
## 
##   Rule 98/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 98/2: [274 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	adhsn <= 0.7209161
## 	nucl <= -0.4555328
## 	n.nuc <= 1.636011
##     then
## 	outcome = 0
## 
##   Rule 98/3: [222 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick <= -0.16559
## 	u.shape <= -0.4286895
##     then
## 	outcome = 0
## 
##   Rule 98/4: [54 cases, mean 0.1, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > -0.16559
## 	u.shape <= -0.4286895
##     then
## 	outcome = 0.2 + 0.377 nucl + 0.369 u.size + 0.368 thick
## 
##   Rule 98/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.6 + 0.466 thick
## 
##   Rule 98/6: [156 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape > -0.4286895
## 	nucl > -0.4555328
##     then
## 	outcome = 1
## 
##   Rule 98/7: [14 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	n.nuc > 1.636011
##     then
## 	outcome = 1 + 0.005 nucl
## 
##   Rule 98/8: [19 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
##     then
## 	outcome = 1.2 - 0.117 thick - 0.014 adhsn
## 
##   Rule 98/9: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 98/10: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## Model 99:
## 
##   Rule 99/1: [247 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl <= 0.08835271
##     then
## 	outcome = 0
## 
##   Rule 99/2: [268 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.size <= 1.808996
## 	u.shape <= -0.1056389
## 	nucl <= -0.4555328
##     then
## 	outcome = 0 + 0.014 nucl
## 
##   Rule 99/3: [6 cases, mean 0.2, range 0 to 1, est err 0.5]
## 
##     if
## 	thick <= 1.242297
## 	u.size > -0.7129574
## 	u.shape <= -0.4286895
## 	nucl > -0.4555328
## 	nucl <= 1.176124
##     then
## 	outcome = 0.6 + 1.081 thick
## 
##   Rule 99/4: [14 cases, mean 0.3, range 0 to 1, est err 0.3]
## 
##     if
## 	thick <= 0.5383533
## 	u.size <= 1.808996
## 	u.shape > -0.1056389
## 	adhsn <= 0.7209161
## 	nucl <= 1.176124
## 	n.nuc <= 1.636011
##     then
## 	outcome = -1
## 
##   Rule 99/5: [8 cases, mean 0.4, range 0 to 1, est err 0.3]
## 
##     if
## 	u.size <= -0.7129574
## 	nucl > 0.08835271
##     then
## 	outcome = 0.5 + 0.436 thick
## 
##   Rule 99/6: [9 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	u.size > -0.7129574
## 	u.size <= 1.808996
## 	u.shape <= -0.1056389
## 	nucl > 1.176124
##     then
## 	outcome = 0.8 + 0.077 nucl + 0.043 thick + 0.036 u.size + 0.022 n.nuc
## 
##   Rule 99/7: [18 cases, mean 0.9, range 0 to 1, est err 0.1]
## 
##     if
## 	u.size > -0.7129574
## 	u.shape > -0.4286895
## 	u.shape <= -0.1056389
## 	nucl > -0.4555328
##     then
## 	outcome = 1
## 
##   Rule 99/8: [92 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	thick > 0.5383533
## 	u.shape > -0.1056389
##     then
## 	outcome = 1
## 
##   Rule 99/9: [100 cases, mean 1.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size > -0.7129574
## 	nucl > 1.176124
##     then
## 	outcome = 1
## 
##   Rule 99/10: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 99/11: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 99/12: [55 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	n.nuc > 1.636011
##     then
## 	outcome = 1
## 
##   Rule 99/13: [54 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.size > 1.808996
##     then
## 	outcome = 1
## 
## Model 100:
## 
##   Rule 100/1: [239 cases, mean 0.0, range 0 to 0, est err 0.0]
## 
##     if
## 	s.size <= -0.5858465
## 	nucl <= -0.7274755
##     then
## 	outcome = 0
## 
##   Rule 100/2: [232 cases, mean 0.0, range 0 to 1, est err 0.0]
## 
##     if
## 	u.shape <= -0.7517401
##     then
## 	outcome = 0
## 
##   Rule 100/3: [6 cases, mean 0.2, range 0 to 1, est err 0.3]
## 
##     if
## 	u.shape <= -0.7517401
## 	nucl > 0.08835271
##     then
## 	outcome = 1.2 + 1.019 thick
## 
##   Rule 100/4: [6 cases, mean 0.2, range 0 to 1, est err 0.0]
## 
##     if
## 	u.size <= -0.08246895
## 	u.shape > -0.7517401
## 	s.size > -0.5858465
## 	nucl <= -0.7274755
##     then
## 	outcome = -0.3 - 3.172 u.size
## 
##   Rule 100/5: [12 cases, mean 0.2, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	chrom <= -0.1989626
##     then
## 	outcome = 1.4 + 2.491 u.shape + 1.102 chrom
## 
##   Rule 100/6: [9 cases, mean 0.4, range 0 to 1, est err 0.6]
## 
##     if
## 	thick <= 0.1863816
## 	u.shape > -0.1056389
## 	u.shape <= 1.186563
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	nucl <= 1.176124
##     then
## 	outcome = 0
## 
##   Rule 100/7: [8 cases, mean 0.6, range 0 to 1, est err 0.7]
## 
##     if
## 	thick <= 1.242297
## 	u.shape <= 1.186563
## 	adhsn > 0.3774847
## 	adhsn <= 0.7209161
##     then
## 	outcome = -0.4 + 7.023 u.shape - 4.45 u.size
## 
##   Rule 100/8: [4 cases, mean 0.8, range 0 to 1, est err 0.6]
## 
##     if
## 	u.size > -0.08246895
## 	u.shape <= 1.186563
## 	nucl <= -0.7274755
##     then
## 	outcome = 1 + 0.102 nucl + 0.073 thick + 0.057 chrom + 0.049 s.size
## 	          + 0.049 n.nuc
## 
##   Rule 100/9: [7 cases, mean 0.9, range 0 to 1, est err 0.2]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= -0.1056389
## 	adhsn <= 0.7209161
## 	nucl > -0.7274755
## 	nucl <= 1.176124
## 	chrom > -0.1989626
##     then
## 	outcome = 0.9 + 0.444 u.shape + 0.181 nucl + 0.091 thick + 0.06 n.nuc
## 	          + 0.046 chrom + 0.04 s.size + 0.023 u.size
## 
##   Rule 100/10: [17 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick <= 1.242297
## 	u.shape > -0.7517401
## 	u.shape <= 1.186563
## 	adhsn <= 0.3774847
## 	nucl > 1.176124
##     then
## 	outcome = 1 + 0.007 nucl
## 
##   Rule 100/11: [53 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 0.1863816
## 	adhsn <= 0.3774847
## 	nucl > -0.7274755
##     then
## 	outcome = 1
## 
##   Rule 100/12: [61 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	thick > 1.242297
##     then
## 	outcome = 1
## 
##   Rule 100/13: [85 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	adhsn > 0.7209161
##     then
## 	outcome = 1
## 
##   Rule 100/14: [71 cases, mean 1.0, range 1 to 1, est err 0.0]
## 
##     if
## 	u.shape > 1.186563
##     then
## 	outcome = 1
## 
## 
## Evaluation on training data (474 cases):
## 
##     Average  |error|                0.0
##     Relative |error|               0.07
##     Correlation coefficient        0.97
## 
## 
## 	Attribute usage:
## 	  Conds  Model
## 
## 	   61%     8%    nucl
## 	   37%     3%    u.shape
## 	   36%     2%    u.size
## 	   22%     7%    thick
## 	   19%     3%    s.size
## 	   12%     1%    adhsn
## 	   12%     4%    n.nuc
## 	    8%     3%    chrom
## 	    1%           mit
## 
## 
## Time: 0.2 secs
```

```r
dotPlot(varImp(cubit.fit), main="Cubist Predictor importance")
```

<img src="48-Cubist-Model_files/figure-html/unnamed-chunk-7-1.png" width="672" />
