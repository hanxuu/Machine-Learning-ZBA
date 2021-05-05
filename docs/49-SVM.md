# Support Vector Machine  






们在同一个数据集上应用KNN和SVM, 对混淆矩阵进行深入研究，并对评价 模型正确率的各个统计量进行比较.

要研究的数据来自美国国家糖尿病消化病肾病研究所，这个数据集包括532个观测，8 个输入特征以及1个二值结果变量（Yes/No）。这项研究中的患者来自美国亚利桑那州中南部，是 皮玛族印第安人的后裔。数据显示，在过去的30年中，科学家已经通过研究证明肥胖是引发糖尿 病的重要因素。选择皮玛印第安人进行这项研究是因为，半数成年皮玛印第安人患有糖尿病。而这些患有糖尿病的人中，有95%超重。研究仅限于成年女性，病情则按照世界卫生组织的标准进 行诊断，为Ⅱ型糖尿病。这种糖尿病的患者胰腺功能并未完全丧失，还可以产生胰岛素，因此又 称“非胰岛素依赖型”糖尿病。

是研究那些糖尿病患者，并对这个人群中可能导致糖尿病的风险因素进行预测。 久坐不动的生活方式和高热量的饮食习惯使得糖尿病已经成为美国的流行病。根据美国糖尿病协 会的数据，2010年，糖尿病成为美国排名第七的致死疾病，这个结果还不包括那些未被诊断出来 的病例。糖尿病还会大大增加其他疾病的发病概率，比如高血压、血脂异常、中风、眼疾和肾脏 疾病。糖尿病及其并发症的医疗成本非常巨大，据估计，美国2012年糖尿病治疗总成本大约为4900 亿美元。

## Data Preparation

数据集包含了532位女性患者的信息，存储在两个数据框中。数据集包含在MASS这个R包中，一个数据框是Pima.tr，另一个数据框的是Pima.te。我们不 将它们分别作为训练集和测试集，而是将其合在一起，然后建立自己的训练集和测试集

数据集变量如下:

    npreg：怀孕次数  
    glu：血糖浓度，由口服葡萄糖耐量测试给出  
    bp：舒张压（单位为mm Hg）  
    skin：三头肌皮褶厚度（单位为mm）  
    bmi：身体质量指数  
    ped：糖尿病家族影响因素  
    age：年龄  
    type：是否患有糖尿病（是/否）



```r
data(Pima.tr)
str(Pima.tr)
```

```
## 'data.frame':	200 obs. of  8 variables:
##  $ npreg: int  5 7 5 0 0 5 3 1 3 2 ...
##  $ glu  : int  86 195 77 165 107 97 83 193 142 128 ...
##  $ bp   : int  68 70 82 76 60 76 58 50 80 78 ...
##  $ skin : int  28 33 41 43 25 27 31 16 15 37 ...
##  $ bmi  : num  30.2 25.1 35.8 47.9 26.4 35.6 34.3 25.9 32.4 43.3 ...
##  $ ped  : num  0.364 0.163 0.156 0.259 0.133 ...
##  $ age  : int  24 55 35 26 23 52 25 24 63 31 ...
##  $ type : Factor w/ 2 levels "No","Yes": 1 2 1 1 1 2 1 1 1 2 ...
```

```r
data(Pima.te)
str(Pima.te)
```

```
## 'data.frame':	332 obs. of  8 variables:
##  $ npreg: int  6 1 1 3 2 5 0 1 3 9 ...
##  $ glu  : int  148 85 89 78 197 166 118 103 126 119 ...
##  $ bp   : int  72 66 66 50 70 72 84 30 88 80 ...
##  $ skin : int  35 29 23 32 45 19 47 38 41 35 ...
##  $ bmi  : num  33.6 26.6 28.1 31 30.5 25.8 45.8 43.3 39.3 29 ...
##  $ ped  : num  0.627 0.351 0.167 0.248 0.158 0.587 0.551 0.183 0.704 0.263 ...
##  $ age  : int  50 31 21 26 53 51 31 33 27 29 ...
##  $ type : Factor w/ 2 levels "No","Yes": 2 1 1 2 2 2 2 1 1 2 ...
```

```r
pima <- rbind(Pima.tr, Pima.te)
str(pima)
```

```
## 'data.frame':	532 obs. of  8 variables:
##  $ npreg: int  5 7 5 0 0 5 3 1 3 2 ...
##  $ glu  : int  86 195 77 165 107 97 83 193 142 128 ...
##  $ bp   : int  68 70 82 76 60 76 58 50 80 78 ...
##  $ skin : int  28 33 41 43 25 27 31 16 15 37 ...
##  $ bmi  : num  30.2 25.1 35.8 47.9 26.4 35.6 34.3 25.9 32.4 43.3 ...
##  $ ped  : num  0.364 0.163 0.156 0.259 0.133 ...
##  $ age  : int  24 55 35 26 23 52 25 24 63 31 ...
##  $ type : Factor w/ 2 levels "No","Yes": 1 2 1 1 1 2 1 1 1 2 ...
```

```r
## 通过箱线图进行探索性分析。为此，要使用结果变量"type"作为ID变量的值。和逻辑斯蒂 回归一样，melt()函数会融合数据并准备好用于生成箱线图的数据框
## 用facet_wrap()函数将统计图分两列显示
pima.melt <- melt(pima, id.var = "type")
ggplot(data = pima.melt, aes(x = type, y = value)) +
  geom_boxplot() + facet_wrap(~ variable, ncol = 2)
```

<img src="49-SVM_files/figure-html/unnamed-chunk-1-1.png" width="672" />

```r
## 为很难从中发现任何明显区别, 里最大的问题是，不同统计图的 单位不同，但却共用一个Y轴。对数据进行标准化处理并重新做图，可以解决这个问题，并生成 更有意义的统计图。
## R内建函数scale()，可以将数据转换为均值为0、标准差为1的标准 形式, 你对一个数据框应用了scale()函数，它 就自动变成一个矩阵。使用as.data.frame()函数，将其重新变回数据框
## 要对所有特征进行转换，只留 下响应变量type
pima.scale <- data.frame(scale(pima[, -8]))
#scale.pima = as.data.frame(scale(pima[,1:7], byrow=FALSE)) #do not create own function
str(pima.scale)
```

```
## 'data.frame':	532 obs. of  7 variables:
##  $ npreg: num  0.448 1.052 0.448 -1.062 -1.062 ...
##  $ glu  : num  -1.13 2.386 -1.42 1.418 -0.453 ...
##  $ bp   : num  -0.285 -0.122 0.852 0.365 -0.935 ...
##  $ skin : num  -0.112 0.363 1.123 1.313 -0.397 ...
##  $ bmi  : num  -0.391 -1.132 0.423 2.181 -0.943 ...
##  $ ped  : num  -0.403 -0.987 -1.007 -0.708 -1.074 ...
##  $ age  : num  -0.708 2.173 0.315 -0.522 -0.801 ...
```

```r
pima.scale$type <- pima$type

pima.scale.melt <- melt(pima.scale, id.var = "type")
ggplot(data=pima.scale.melt, aes(x = type, y = value)) + 
  geom_boxplot() + facet_wrap(~ variable, ncol = 2)
```

<img src="49-SVM_files/figure-html/unnamed-chunk-1-2.png" width="672" />

```r
## Interpretation: 出其他特征也随着 type发生变化，特别是age

## 有两对变量之间具有相关性：npreg/age和skin/bmi。如果能够正确训练模型，并能调整好 超参数，那么多重共线性对于这些方法通常都不是问题
cor(pima.scale[-8])
```

```
##             npreg       glu          bp       skin         bmi         ped
## npreg 1.000000000 0.1253296 0.204663421 0.09508511 0.008576282 0.007435104
## glu   0.125329647 1.0000000 0.219177950 0.22659042 0.247079294 0.165817411
## bp    0.204663421 0.2191779 1.000000000 0.22607244 0.307356904 0.008047249
## skin  0.095085114 0.2265904 0.226072440 1.00000000 0.647422386 0.118635569
## bmi   0.008576282 0.2470793 0.307356904 0.64742239 1.000000000 0.151107136
## ped   0.007435104 0.1658174 0.008047249 0.11863557 0.151107136 1.000000000
## age   0.640746866 0.2789071 0.346938723 0.16133614 0.073438257 0.071654133
##              age
## npreg 0.64074687
## glu   0.27890711
## bp    0.34693872
## skin  0.16133614
## bmi   0.07343826
## ped   0.07165413
## age   1.00000000
```

```r
## 先检查响应变量中 Yes和No的比例。确保数据划分平衡是非常重要的，如果某个结果过于稀疏，就会导致问题，可 能引起分类器在优势类和劣势类之间发生偏离。对于不平衡的判定没有一个固定的规则。一个比 较好的经验法则是，结果中的比例至少应该达到2∶1
table(pima.scale$type)
```

```
## 
##  No Yes 
## 355 177
```

```r
## 比例为2∶1，现在可以建立训练集和测试集了。使用我们常用的语法，划分比例为70/30
set.seed(502)
ind <- sample(2, nrow(pima.scale), replace = TRUE, prob = c(0.7, 0.3))
train <- pima.scale[ind == 1, ]
test <- pima.scale[ind == 2, ]
str(train)
```

```
## 'data.frame':	385 obs. of  8 variables:
##  $ npreg: num  0.448 0.448 -0.156 -0.76 -0.156 ...
##  $ glu  : num  -1.42 -0.775 -1.227 2.322 0.676 ...
##  $ bp   : num  0.852 0.365 -1.097 -1.747 0.69 ...
##  $ skin : num  1.123 -0.207 0.173 -1.253 -1.348 ...
##  $ bmi  : num  0.4229 0.3938 0.2049 -1.0159 -0.0712 ...
##  $ ped  : num  -1.007 -0.363 -0.485 0.441 -0.879 ...
##  $ age  : num  0.315 1.894 -0.615 -0.708 2.916 ...
##  $ type : Factor w/ 2 levels "No","Yes": 1 2 1 1 1 2 2 1 1 1 ...
```

```r
str(test)
```

```
## 'data.frame':	147 obs. of  8 variables:
##  $ npreg: num  0.448 1.052 -1.062 -1.062 -0.458 ...
##  $ glu  : num  -1.13 2.386 1.418 -0.453 0.225 ...
##  $ bp   : num  -0.285 -0.122 0.365 -0.935 0.528 ...
##  $ skin : num  -0.112 0.363 1.313 -0.397 0.743 ...
##  $ bmi  : num  -0.391 -1.132 2.181 -0.943 1.513 ...
##  $ ped  : num  -0.403 -0.987 -0.708 -1.074 2.093 ...
##  $ age  : num  -0.7076 2.173 -0.5217 -0.8005 -0.0571 ...
##  $ type : Factor w/ 2 levels "No","Yes": 1 2 1 1 2 1 2 1 1 1 ...
```



使用e1071包构建SVM模型，先从线性支持向量分类器开始，然后转入非线性模型

e1071 包中有一个非常好的用于SVM的函数——tune.svm()，它可以帮助我们选择调优参数及核函 数。tune.svm()使用交叉验证使调优参数达到最优。我们先建立一个名为linear.tune的对象， 然后使用summary()函数看看其中的内容


```r
## linear tune
set.seed(123)
linear.tune <- tune.svm(type ~ ., data = train, 
                        kernel = "linear", 
                        cost = c(0.001, 0.01, 0.1, 1, 5, 10))
summary(linear.tune)
```

```
## 
## Parameter tuning of 'svm':
## 
## - sampling method: 10-fold cross validation 
## 
## - best parameters:
##  cost
##  0.01
## 
## - best performance: 0.2 
## 
## - Detailed performance results:
##    cost     error dispersion
## 1 1e-03 0.3192308 0.04698696
## 2 1e-02 0.2000000 0.04579145
## 3 1e-01 0.2102564 0.05714612
## 4 1e+00 0.2076248 0.06252977
## 5 5e+00 0.2102564 0.06321544
## 6 1e+01 0.2102564 0.06321544
```

```r
## 最优成本函数cost是1，这时的误分类误差率差不多为21%。我们在测试集 上进行预测和检验
best.linear <- linear.tune$best.model
tune.test <- predict(best.linear, newdata = test)
table(tune.test, test$type)
```

```
##          
## tune.test No Yes
##       No  82  24
##       Yes 11  30
```

```r
(82+30)/147
```

```
## [1] 0.7619048
```

```r
## 试验的第一个核函数是多项式核函数，需要调整优化两个参数：多项式的阶（degree） 与核系数（coef0）。设定多项式的阶是3、4和5，核系数从0.1逐渐增加到4
## SVM with e1071; tune the poly only
set.seed(123)
poly.tune <- tune.svm(type ~ ., data = train, 
                      kernel = "polynomial", 
                      degree = c(3, 4, 5), 
                      coef0 = c(0.1, 0.5, 1, 2, 3, 4))
summary(poly.tune)
```

```
## 
## Parameter tuning of 'svm':
## 
## - sampling method: 10-fold cross validation 
## 
## - best parameters:
##  degree coef0
##       3     3
## 
## - best performance: 0.2209177 
## 
## - Detailed performance results:
##    degree coef0     error dispersion
## 1       3   0.1 0.2339406 0.05673284
## 2       4   0.1 0.2416329 0.05898725
## 3       5   0.1 0.2441970 0.06195056
## 4       3   0.5 0.2418354 0.06226721
## 5       4   0.5 0.2468961 0.07055752
## 6       5   0.5 0.2414980 0.05339164
## 7       3   1.0 0.2339406 0.05280244
## 8       4   1.0 0.2649123 0.05419548
## 9       5   1.0 0.2625506 0.07622638
## 10      3   2.0 0.2235493 0.05464342
## 11      4   2.0 0.2493927 0.05941857
## 12      5   2.0 0.2755061 0.06597294
## 13      3   3.0 0.2209177 0.05595814
## 14      4   3.0 0.2520243 0.05094380
## 15      5   3.0 0.2701754 0.04813547
## 16      3   4.0 0.2261134 0.05428339
## 17      4   4.0 0.2493927 0.06747803
## 18      5   4.0 0.2857625 0.05870079
```

```r
best.poly <- poly.tune$best.model
poly.test <- predict(best.poly, newdata = test)
table(poly.test, test$type)
```

```
##          
## poly.test No Yes
##       No  75  25
##       Yes 18  29
```

```r
(81 + 26) / 147
```

```
## [1] 0.7278912
```

```r
## 测试径向基核函数，此处只需找出一个参数gamma， 在0.1 ~ 4中依次检验。如果gamma过小，模型就不能解释决策边界的复杂性；如果gamma过大， 模型就会严重过拟合。
## tune the rbf
set.seed(123)
rbf.tune <- tune.svm(type ~ ., data = train, 
                     kernel = "radial", 
                     gamma = c(0.1, 0.5, 1, 2, 3, 4))
summary(rbf.tune)
```

```
## 
## Parameter tuning of 'svm':
## 
## - sampling method: 10-fold cross validation 
## 
## - best parameters:
##  gamma
##    0.1
## 
## - best performance: 0.2184885 
## 
## - Detailed performance results:
##   gamma     error dispersion
## 1   0.1 0.2184885 0.05636224
## 2   0.5 0.2236842 0.06496235
## 3   1.0 0.2752362 0.06431054
## 4   2.0 0.3244939 0.04452924
## 5   3.0 0.3218623 0.04750687
## 6   4.0 0.3192308 0.04698696
```

```r
best.rbf <- rbf.tune$best.model
rbf.test <- predict(best.rbf, newdata = test)
table(rbf.test, test$type)
```

```
##         
## rbf.test No Yes
##      No  76  26
##      Yes 17  28
```

```r
(73+21)/147
```

```
## [1] 0.6394558
```

```r
## 找出两个参 数——gamma和核系数（coef0）
## tune the sigmoid
set.seed(123)
sigmoid.tune <- tune.svm(type ~ ., data = train, 
                         kernel = "sigmoid", 
                         gamma = c(0.1, 0.5, 1, 2, 3, 4),
                         coef0 = c(0.1, 0.5, 1, 2, 3, 4))
summary(sigmoid.tune)
```

```
## 
## Parameter tuning of 'svm':
## 
## - sampling method: 10-fold cross validation 
## 
## - best parameters:
##  gamma coef0
##    0.1   0.1
## 
## - best performance: 0.2101889 
## 
## - Detailed performance results:
##    gamma coef0     error dispersion
## 1    0.1   0.1 0.2101889 0.06844133
## 2    0.5   0.1 0.2881242 0.07399055
## 3    1.0   0.1 0.2985830 0.07442363
## 4    2.0   0.1 0.2856275 0.03959643
## 5    3.0   0.1 0.2827935 0.06092125
## 6    4.0   0.1 0.2935223 0.08685399
## 7    0.1   0.5 0.2334683 0.08787718
## 8    0.5   0.5 0.2933198 0.08615339
## 9    1.0   0.5 0.2987179 0.05894058
## 10   2.0   0.5 0.3009447 0.06724507
## 11   3.0   0.5 0.3037112 0.07594313
## 12   4.0   0.5 0.2962213 0.08011745
## 13   0.1   1.0 0.2728745 0.08381160
## 14   0.5   1.0 0.3066127 0.06722716
## 15   1.0   1.0 0.2775304 0.05618176
## 16   2.0   1.0 0.3060054 0.07541155
## 17   3.0   1.0 0.2933198 0.06786442
## 18   4.0   1.0 0.2985830 0.08166238
## 19   0.1   2.0 0.2155196 0.07736128
## 20   0.5   2.0 0.3692308 0.08603337
## 21   1.0   2.0 0.3481781 0.06052096
## 22   2.0   2.0 0.2570175 0.04241475
## 23   3.0   2.0 0.2905533 0.05586494
## 24   4.0   2.0 0.3141026 0.06505962
## 25   0.1   3.0 0.3192308 0.04698696
## 26   0.5   3.0 0.3508097 0.05834655
## 27   1.0   3.0 0.3717949 0.08106407
## 28   2.0   3.0 0.3351552 0.04363826
## 29   3.0   3.0 0.3064777 0.05576306
## 30   4.0   3.0 0.2959514 0.05254327
## 31   0.1   4.0 0.3192308 0.04698696
## 32   0.5   4.0 0.3533063 0.06060383
## 33   1.0   4.0 0.3848178 0.08806008
## 34   2.0   4.0 0.3506748 0.06107104
## 35   3.0   4.0 0.3010121 0.06866140
## 36   4.0   4.0 0.2957490 0.06078291
```

```r
best.sigmoid <- sigmoid.tune$best.model
sigmoid.test <- predict(best.sigmoid, newdata = test)
table(sigmoid.test, test$type)
```

```
##             
## sigmoid.test No Yes
##          No  74  23
##          Yes 19  31
```

```r
(82+35)/147
```

```
## [1] 0.7959184
```

```r
## 在测试集上表现得更好, 可以选 择sigmoid核函数作为最优预测。
## 研究了两种不同类型的建模技术，从各方面来看，KNN都处于下风。KNN在测试 集上最好的正确率只有71%左右，相反，通过SVM可以获得接近80%的正确率(使用sigmoid核函数的SVM模型)。
```

## 模型选择

通过混淆矩阵来比较各种模型, 友caret包的confusionMatrix() 函数, 使用过InformationValue包中的同名函数。但caret包中的这 个函数会生成我们评价和选择最优模型所需的所有统计量。先从建立的最后一个模型开始，使用 的语法和基础的table()函数一样，不同之处是要指定positive类

其他统计量介绍:

    No Information Rate：最大分类所占的比例——63%的人没有糖尿病
    Mcnemar's Test：我们现在不关心这个统计量，它用于配对分析，主要用于流行病学 的研究
    Sensitivity：敏感度，真阳性率；在本案例中，表示没有糖尿病并且被正确识别的 比例。
    Specificity：特异度，真阴性率；在本案例中，表示有糖尿病并且被正确识别的比例
    Pos Pred Value：阳性预测率，被认为有糖尿病的人中真的有糖尿病的概率。
      PPV =敏感度 *患病率/((敏感度 *患病率) + (1-敏感度) * (1-患病率))
    Neg Pred Value：阴性预测率，被认为没有糖尿病的人中真的没有糖尿病的概率
      NPV=敏感度 * (1-患病率)/(((1-敏感度) * (患病率)) + (敏感度) * (1-患病率) )
    Prevalence：患病率，某种疾病在人群中流行度的估计值: 第二列   （Yes列）中的数之和除以总观测数（矩阵中所有数之和）。 
    Detection Rate：真阳性预测中被正确识别的比例
    Detection Prevalence：预测的患病率，在本案例中，底行中的数的和除以总观测数。
    Balanced Accuracy：所有类别正确率的平均数。用来表示由于分类器算法中潜在的偏     差造成的对最频繁类的过度预测。可以简单地用(敏感度 + 特异度)/2来计算


```r
confusionMatrix(sigmoid.test, test$type, positive = "Yes")
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction No Yes
##        No  74  23
##        Yes 19  31
##                                          
##                Accuracy : 0.7143         
##                  95% CI : (0.634, 0.7857)
##     No Information Rate : 0.6327         
##     P-Value [Acc > NIR] : 0.02308        
##                                          
##                   Kappa : 0.3756         
##                                          
##  Mcnemar's Test P-Value : 0.64343        
##                                          
##             Sensitivity : 0.5741         
##             Specificity : 0.7957         
##          Pos Pred Value : 0.6200         
##          Neg Pred Value : 0.7629         
##              Prevalence : 0.3673         
##          Detection Rate : 0.2109         
##    Detection Prevalence : 0.3401         
##       Balanced Accuracy : 0.6849         
##                                          
##        'Positive' Class : Yes            
## 
```

```r
## 结果和线性SVM进行对比
confusionMatrix(tune.test, test$type, positive = "Yes")
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction No Yes
##        No  82  24
##        Yes 11  30
##                                           
##                Accuracy : 0.7619          
##                  95% CI : (0.6847, 0.8282)
##     No Information Rate : 0.6327          
##     P-Value [Acc > NIR] : 0.0005615       
##                                           
##                   Kappa : 0.4605          
##                                           
##  Mcnemar's Test P-Value : 0.0425225       
##                                           
##             Sensitivity : 0.5556          
##             Specificity : 0.8817          
##          Pos Pred Value : 0.7317          
##          Neg Pred Value : 0.7736          
##              Prevalence : 0.3673          
##          Detection Rate : 0.2041          
##    Detection Prevalence : 0.2789          
##       Balanced Accuracy : 0.7186          
##                                           
##        'Positive' Class : Yes             
## 
```



## 特征选择

此处忽略了一件事，即没有进行任何特征选择。我们做的工作就是把特征堆在一起， 作为所谓的输入空间，然后让SVM这个黑盒去计算，最后给出一个预测分类。使用SVM的一个 主要问题就是，它给出的结果非常难以解释. 使用caret包进行粗略的特征选择。因为对于那些像SVM一样使用 黑盒技术的方法来说，特征选择确实是个艰巨的挑战。这也是使用这些技术时可能遇到的主要困 难

还有一些其他办法可以进行特征选择, 需要做的就是反复实验。再次用到caret包，因为它可以基于kernlab包在线性SVM中 进行交叉验证. 

设定随机数种子，在caret包中的rfeControl()函数中指定交叉验 证方法，使用rfe()函数执行一个递归的特征选择过程，最后检验模型在测试集上的运行情况。 在rfeControl()中，你需要根据使用的模型指定functions参数。可以使用几种不同的functions 参数，此处使用lrFuncs



```r
set.seed(123)
rfeCNTL <- rfeControl(functions = lrFuncs, method = "cv", number = 10)
## 要指定输入数据和响应因子、通过参数sizes指定输入 特征的数量以及kernlab包中的线性方法（此处是svmLinear）。method还有其他一些选项
svm.features <- rfe(train[, 1:7], train[, 8],
                   sizes = c(7, 6, 5, 4), 
                   rfeControl = rfeCNTL, 
                   method = "svmLinear")
svm.features
```

```
## 
## Recursive feature selection
## 
## Outer resampling method: Cross-Validated (10 fold) 
## 
## Resampling performance over subset size:
## 
##  Variables Accuracy  Kappa AccuracySD KappaSD Selected
##          4   0.7845 0.4774    0.06121  0.1619         
##          5   0.7898 0.4855    0.05589  0.1551        *
##          6   0.7870 0.4774    0.05563  0.1561         
##          7   0.7897 0.4826    0.05499  0.1545         
## 
## The top 5 variables (out of 5):
##    glu, ped, npreg, bmi, age
```

```r
svm.5 <- svm(type ~ glu + ped + npreg + bmi + age, 
             data = train, 
             kernel = "linear")
svm.5.predict = predict(svm.5, newdata=test[c(1,2,5,6,7)])
table(svm.5.predict, test$type)
```

```
##              
## svm.5.predict No Yes
##           No  79  21
##           Yes 14  33
```

```r
## 全特征模型的正确率是76.2%, 表现不怎么样, 要回到全特征模型。
```

