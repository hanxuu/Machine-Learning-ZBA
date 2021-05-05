# PCA





## Introduction

* 聚类分析，它可以将相似的观测 归成一类
* 主成分分析(PCA)，它可以对相关变量进行归类，从而降低 数据维度，提高对数据的理解

维数灾难，因为模型估计所需的样本数量是随着输入特征的 数量指数增长的。这种数据集中会出现某些变量冗余，因为这些变量最后起的作用与其他变量基 本是重复的。比如收入水平与贫穷程度，或者抑郁度与焦虑度。那么我们的目标就是，通过PCA 从原始变量集合中找出一个更小的，但是能保留原来大部分信息的变量集合。这样可以简化数据 集，并经常能够发现数据背后隐藏的知识。这些新的变量(主成分)彼此高度不相关，除了可以 9 用于监督式学习之外，还经常用于数据可视化。

成分就是特征的规范化线性组合(James，2012)。在一个数据集中，第一主成分就 是能够最大程度解释数据中的方差的特征线性组合。第二主成分是另一种特征线性组合，它在方 向与第一主成分垂直这个限制条件下，最大程度解释数据中的方差。其后的每一个主成分(可以 构造与变量数相等数目的主成分)都遵循同样的规则。

Annahmen：

* 线性组合：如果你试图在一个变量之间 基本不相关的数据集上使用PCA，很可能会得到一个毫无意义的分析结果
* 变量的均值和方差是充分统计量。也就是说，数据应该服从正态分布，这样协方差矩阵即可充分 描述数据集。换言之，数据要满足多元正态分布。PCA对于非正态分布的数据具有相当强的鲁棒 性，甚至可以和二值变量一起使用，所以结果具有很好的解释性。

如何精简主成分，来达到降低数据维度这一首要初始目标:

在特征值大于1的情况下选择主成分

最优线性权重是通过线性代数运算得到特征向量而求出的，它们是最优解，因为 没有其他可能的权重组合可以比它们更好地解释方差。主成分的特征值是它在整个数据集中能够解释的方差的数量。


 

因为第一特征值可以解释最大数量的方差，它就有最大的特征值;第二主成分有第二大的特 征值，依此类推。所以，特征值大于1就表示这个主成分解释的方差比任何一个原始变量都要大。 如果通过标准化操作将特征值的总和变为1，就能够得到每个主成分解释的方差的比例。这也有 助于确定一个适当的分界点。












## 主成分旋转

旋转可以修改每个变量的载荷，这样有助于对主成分的解释。 旋转后的成分能够解释的方差总量是不变的，但是每个成分对于能够解释的方差总量的贡献会改 变。在旋转过程中，你会发现载荷的值或者更远离0，或者更接近0，这在理论上可以帮助我们识 别那些对主成分起重要作用的变量。这是一种将变量和唯一一个主成分联系起来的尝试。

最常用的主成分旋转方法被称为方差最大法。在方差最大法中，我们要使平方后的载荷的总方差最大。方差最大化过程会旋转 特征空间的轴和坐标，但不改变数据点的位置。







 
## 数据准备


```r
test <- read.csv("/Users/zehuibai/Documents/GitHub/Machine-Learning-ZBA/data/NHLtest.csv", header=T)
train <- read.csv("/Users/zehuibai/Documents/GitHub/Machine-Learning-ZBA/data/NHLtrain.csv", header=T)

str(train)
```

```
## 'data.frame':	30 obs. of  15 variables:
##  $ Team         : chr  "Anaheim" "Arizona" "Boston" "Buffalo" ...
##  $ ppg          : num  1.26 0.95 1.13 0.99 0.94 1.05 1.26 1 0.93 1.33 ...
##  $ Goals_For    : num  2.62 2.54 2.88 2.43 2.79 2.39 2.85 2.59 2.6 3.23 ...
##  $ Goals_Against: num  2.29 2.98 2.78 2.62 3.13 2.7 2.52 2.93 3.02 2.78 ...
##  $ Shots_For    : num  30.3 27.6 32 29.5 29.2 29.9 30.5 28.6 29.1 32 ...
##  $ Shots_Against: num  27.5 31 30.4 30.6 29 27.6 30.8 32.3 31.1 28.9 ...
##  $ PP_perc      : num  23 17.7 20.5 18.9 17 16.8 22.6 18 17.3 22.1 ...
##  $ PK_perc      : num  87.2 77.3 82.2 82.6 75.6 84.3 80.3 80.2 81 82.3 ...
##  $ CF60_pp      : num  111.6 97.7 118.3 97.4 94 ...
##  $ CA60_sh      : num  94.1 96.1 94.4 100.6 102.8 ...
##  $ OZFOperc_pp  : num  78.4 72.5 79.4 76.2 77.1 ...
##  $ Give         : num  9.78 5.67 8.6 6.34 9.8 ...
##  $ Take         : num  5.22 5.89 6.11 5.26 6.99 9.22 5.82 5.56 5.98 7.01 ...
##  $ hits         : num  27.2 22.1 26.4 23.4 20.7 ...
##  $ blks         : num  14.4 14 14.4 13.3 16.1 ...
```

```r
names(train)
```

```
##  [1] "Team"          "ppg"           "Goals_For"     "Goals_Against"
##  [5] "Shots_For"     "Shots_Against" "PP_perc"       "PK_perc"      
##  [9] "CF60_pp"       "CA60_sh"       "OZFOperc_pp"   "Give"         
## [13] "Take"          "hits"          "blks"
```

```r
train.scale <- scale(train[, -1:-2])
nhl.cor = cor(train.scale)
# dev.off()
cor.plot(nhl.cor)
```

<img src="57-PCA_files/figure-html/unnamed-chunk-1-1.png" width="672" />






## 模型构建

对于模型构建过程，我们按照以下几个步骤进行:

* (1) 抽取主成分并决定保留的数量; 
* (2) 对留下的主成分进行旋转;
* (3) 对旋转后的解决方案进行解释; 
* (4) 生成各个因子的得分;
* (5) 使用得分作为输入变量进行回归分析，并使用测试数据评价模型效果。



### 主成分抽取


```r
## 通过psych包抽取主成分要使用principal()函数，这个函数的语法中要包括数据和是否要 进行主成分旋转:
pca <- principal(train.scale, rotate="none")
## 使用碎石图即可。碎石图可以帮助你评估能解释大部分数据方差的主成分，它用X轴表示 主成分的数量，用Y轴表示相应的特征值:
plot(pca$values, type="b", ylab="Eigenvalues", xlab="Component")
```

<img src="57-PCA_files/figure-html/unnamed-chunk-2-1.png" width="672" />

```r
## 需要在碎石图中找出使变化率降低的那个点，也就是我们常说的统计图中的“肘点”或弯曲 点。在统计图中，肘点表示在这个点上新增加一个主成分时，对方差的解释增加得并不太多。换 句话说，这个点就是曲线由陡变平的转折点
```


### 正交旋转与解释


```r
## 旋转背后的意义是使变量在某个主成分上的载荷最大化，这样可以减少(或消灭)主成分之间的相关性，有助于对主成分的解释。进行正交旋转的方法称为“方差最大法”。还有其他非正交旋转方法，这种方法允许主成分(因子)之间存在相关性
## 设定使用5个主成分，并需要进行正交旋转
pca.rotate <- principal(train.scale, nfactors = 5, rotate = "varimax")
pca.rotate
```

```
## Principal Components Analysis
## Call: principal(r = train.scale, nfactors = 5, rotate = "varimax")
## Standardized loadings (pattern matrix) based upon correlation matrix
##                 RC1   RC2   RC5   RC3   RC4   h2   u2 com
## Goals_For     -0.21  0.82  0.21  0.05 -0.11 0.78 0.22 1.3
## Goals_Against  0.88 -0.02 -0.05  0.21  0.00 0.82 0.18 1.1
## Shots_For     -0.22  0.43  0.76 -0.02 -0.10 0.81 0.19 1.8
## Shots_Against  0.73 -0.02 -0.20 -0.29  0.20 0.70 0.30 1.7
## PP_perc       -0.73  0.46 -0.04 -0.15  0.04 0.77 0.23 1.8
## PK_perc       -0.73 -0.21  0.22 -0.03  0.10 0.64 0.36 1.4
## CF60_pp       -0.20  0.12  0.71  0.24  0.29 0.69 0.31 1.9
## CA60_sh        0.35  0.66 -0.25 -0.48 -0.03 0.85 0.15 2.8
## OZFOperc_pp   -0.02 -0.18  0.70 -0.01  0.11 0.53 0.47 1.2
## Give          -0.02  0.58  0.17  0.52  0.10 0.65 0.35 2.2
## Take           0.16  0.02  0.01  0.90 -0.05 0.83 0.17 1.1
## hits          -0.02 -0.01  0.27 -0.06  0.87 0.83 0.17 1.2
## blks           0.19  0.63 -0.18  0.14  0.47 0.70 0.30 2.4
## 
##                        RC1  RC2  RC5  RC3  RC4
## SS loadings           2.69 2.33 1.89 1.55 1.16
## Proportion Var        0.21 0.18 0.15 0.12 0.09
## Cumulative Var        0.21 0.39 0.53 0.65 0.74
## Proportion Explained  0.28 0.24 0.20 0.16 0.12
## Cumulative Proportion 0.28 0.52 0.72 0.88 1.00
## 
## Mean item complexity =  1.7
## Test of the hypothesis that 5 components are sufficient.
## 
## The root mean square of the residuals (RMSR) is  0.08 
##  with the empirical chi square  28.59  with prob <  0.19 
## 
## Fit based upon off diagonal values = 0.91
```

### 根据主成分建立因子得分


```r
pca.scores <- data.frame(pca.rotate$scores)
head(pca.scores)
```

```
##           RC1          RC2        RC5        RC3        RC4
## 1 -2.21526408  0.002821488  0.3161588 -0.1572320  1.5278033
## 2  0.88147630 -0.569239044 -1.2361419 -0.2703150 -0.0113224
## 3  0.10321189  0.481754024  1.8135052 -0.1606672  0.7346531
## 4 -0.06630166 -0.630676083 -0.2121434 -1.3086231  0.1541255
## 5  1.49662977  1.156905747 -0.3222194  0.9647145 -0.6564827
## 6 -0.48902169 -2.119952370  1.0456190  2.7375097 -1.3735777
```

```r
## 得到每个球队在每个因子上的得分，这些得分的计算非常简单，每个观测的变量值乘以载荷 然后相加即可。现在可以将响应变量(ppg)作为一列加入数据
pca.scores$ppg <- train$ppg
```


### 回归分析


```r
nhl.lm <- lm(ppg ~ ., data = pca.scores)
summary(nhl.lm)
```

```
## 
## Call:
## lm(formula = ppg ~ ., data = pca.scores)
## 
## Residuals:
##       Min        1Q    Median        3Q       Max 
## -0.163274 -0.048189  0.003718  0.038723  0.165905 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  1.111333   0.015752  70.551  < 2e-16 ***
## RC1         -0.112201   0.016022  -7.003 3.06e-07 ***
## RC2          0.070991   0.016022   4.431 0.000177 ***
## RC5          0.022945   0.016022   1.432 0.164996    
## RC3         -0.017782   0.016022  -1.110 0.278044    
## RC4         -0.005314   0.016022  -0.332 0.743003    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.08628 on 24 degrees of freedom
## Multiple R-squared:  0.7502,	Adjusted R-squared:  0.6981 
## F-statistic: 14.41 on 5 and 24 DF,  p-value: 1.446e-06
```

```r
nhl.lm2 <- lm(ppg ~ RC1 + RC2, data = pca.scores)
summary(nhl.lm2)
```

```
## 
## Call:
## lm(formula = ppg ~ RC1 + RC2, data = pca.scores)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.18914 -0.04430  0.01438  0.05645  0.16469 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  1.11133    0.01587  70.043  < 2e-16 ***
## RC1         -0.11220    0.01614  -6.953  1.8e-07 ***
## RC2          0.07099    0.01614   4.399 0.000153 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.0869 on 27 degrees of freedom
## Multiple R-squared:  0.7149,	Adjusted R-squared:  0.6937 
## F-statistic: 33.85 on 2 and 27 DF,  p-value: 4.397e-08
```

```r
plot(nhl.lm2$fitted.values, train$ppg, 
     main="Predicted versus Actual",
     xlab="Predicted",ylab="Actual")
```

<img src="57-PCA_files/figure-html/unnamed-chunk-5-1.png" width="672" />

```r
## 使用ggplot2包生成一张带有球队名 字的散点图
train$pred <- round(nhl.lm2$fitted.values, digits = 2)
p <- ggplot(train, aes(x = pred,
                           y = ppg,
                           label = Team)) 
p + geom_point() + 
  geom_text(size=3.5, hjust=0.1, vjust=-0.5, angle=0) + 
  xlim(0.8, 1.4) + ylim(0.8, 1.5) +
  stat_smooth(method="lm", se=FALSE)
```

<img src="57-PCA_files/figure-html/unnamed-chunk-5-2.png" width="672" />

```r
## 评价模型误差
sqrt(mean(nhl.lm2$residuals^2))
```

```
## [1] 0.08244449
```

```r
## 模型在样本外数据上的效果
test.scores <- data.frame(predict(pca.rotate, test[, c(-1:-2)]))
test.scores$pred <- predict(nhl.lm2, test.scores)

test.scores$ppg <- test$ppg
test.scores$Team <- test$Team

p <- ggplot(test.scores, aes(x = pred,
                       y = ppg,
                       label = Team)) 
p + geom_point() + 
  geom_text(size=3.5, hjust=0.4, vjust = -0.9, angle = 35) + 
  xlim(0.75, 1.5) + ylim(0.5, 1.6) +
  stat_smooth(method="lm", se=FALSE)
```

<img src="57-PCA_files/figure-html/unnamed-chunk-5-3.png" width="672" />

```r
resid <- test.scores$ppg - test.scores$pred
sqrt(mean(resid^2))
```

```
## [1] 0.1011561
```



