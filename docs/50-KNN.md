# K-Nearest Neighbors  






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

<img src="50-KNN_files/figure-html/unnamed-chunk-1-1.png" width="672" />

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

<img src="50-KNN_files/figure-html/unnamed-chunk-1-2.png" width="672" />

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





使用KNN建模关键的一点就是选择最合适的参数（k或K）。在确定k值 方面，caret包又可以大展身手了。先建立一个供实验用的输入网格，k值从2到20，每次增加1。 使用expand.grid()和seq()函数可以轻松实现。在caret包中，作用于KNN函数的参数非常简 单直接，就是.k：



```r
grid1 <- expand.grid(.k = seq(2, 20, by = 1))
## 选择参数时，还是使用交叉验证。先建立一个名为control的对象，然后使用caret包中的 trainControl()函数
control = trainControl(method = "cv")

## 先设定随机数种子, ，使用caret包中train()函数建立计算最优k值的对象
## 使用train()函数建立对象时，需要指定模型公式、训练数据集名称和一个合适的方法(knn), 以建立对象并计算最优k值
set.seed(123)
knn.train <- train(type ~ ., data = train, 
                   method = "knn", 
                   trControl = control, 
                   tuneGrid = grid1)
## 调用这个对象即可得到我们追寻的最优k值，是17: The final value used for the model was k = 17.
knn.train
```

```
## k-Nearest Neighbors 
## 
## 385 samples
##   7 predictor
##   2 classes: 'No', 'Yes' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 345, 347, 347, 346, 347, 347, ... 
## Resampling results across tuning parameters:
## 
##   k   Accuracy   Kappa    
##    2  0.7291262  0.3579438
##    3  0.7661606  0.4218716
##    4  0.7688596  0.4318872
##    5  0.7660324  0.4220202
##    6  0.7657659  0.4140541
##    7  0.7791161  0.4501050
##    8  0.7713563  0.4371057
##    9  0.7816835  0.4559734
##   10  0.7868792  0.4661169
##   11  0.7841835  0.4627038
##   12  0.7736572  0.4406084
##   13  0.7789845  0.4408541
##   14  0.7712888  0.4150304
##   15  0.7790520  0.4401016
##   16  0.7869467  0.4552805
##   17  0.7739238  0.4262446
##   18  0.7686606  0.4104614
##   19  0.7738596  0.4303754
##   20  0.7791228  0.4437383
## 
## Accuracy was used to select the optimal model using the largest value.
## The final value used for the model was k = 16.
```

```r
## Interpretation
## 在输出的表格中还可以看到正确率和Kappa统计量的信 息，以及交叉验证过程中产生的标准差。
 # 正确率告诉我们模型正确分类的百分比。
 # Kappa又称科 恩的K统计量，通常用于测量两个分类器对观测值分类的一致性。Kappa可以使我们对分类问题 的理解更加深入，它对正确率进行了修正，去除了仅靠偶然性（或随机性）获得正确分类的因素。 计算这个统计量的公式是Kappa = (一致性百分比 期望一致性百分比)/(1 期望一致性百分比)。 一致性百分比是分类器的分类结果与实际分类相符合的程度（就是正确率），期望一致性百 分比是分类器靠随机选择获得的与实际分类相符合的程度。Kappa统计量的值越大，分类器的分 类效果越好，Kappa为1时达到一致性的最大值。

## 应用到测试数据集: 如何计算正确率和Kappa
knn.test <- knn(train[, -8], test[, -8], train[, 8], k = 17)
table(knn.test, test$type)
```

```
##         
## knn.test No Yes
##      No  77  26
##      Yes 16  28
```

```r
## 正确率: 用分类正确的观测数除以观测总数
(77+28)/147
```

```
## [1] 0.7142857
```

```r
## calculate Kappa
prob.agree <- (77+28)/147
prob.chance <- ((77+26)/147) * ((77+16)/147)
prob.chance
```

```
## [1] 0.4432875
```

```r
kappa <- (prob.agree - prob.chance) / (1 - prob.chance)
kappa
```

```
## [1] 0.486783
```

```r
## 解释Kappa:  ＜0.20 很差; 0.21 ~ 0.40 一般; 0.41 ~ 0.60 中等; 0.61 ~ 0.80 好; 0.81 ~ 1.00 很好
```

## 加权最近邻法

看是否可以使用 加权最近邻法得到更好的结果。加权最近邻法提高了离观测更近的邻居的影响力，降低了远离观 测的邻居的影响力。观测离空间点越远，对它的影响力的惩罚就越大。要使用加权最近邻法，需 要kknn包中的train.kknn()函数来选择最优的加权方式

train.kknn()函数使用我们前面介绍过的LOOCV选择最优参数，比如最优的K最近邻数 量、二选一的距离测量方式，以及核函数。

不加权的K最近邻算法使用的是欧式距离。在kknn包中，除了欧式距离， 还可以选择两点坐标差的绝对值之和。如果要使用这种距离计算方式，需要指定闵可夫斯基距 离参数。

有多种方法可以对距离进行加权, kknn包中有10种不同的加权方式，不加权也 是其中之一。它们是：retangular（不加权）、triangular、epanechnikov、biweight、triweight、consine、 inversion、gaussian、rank和optimal。

赋予权重之前，算法对所 有距离进行标准化处理，使它们的值都在0和1之间。triangular加权方法先算出1减去距离的差， 再用差作为权重去乘这个距离。epanechnikov加权方法是用3/4乘以(1 距离的平方)。



```r
## 两种加权方法triangular, epanechnikov和标准的不加权方法
## 先指定随机数种子，然后使用kknn()函数建立训练集对象, k值的最大值kmax、距离distance（1表示绝对值距离，2表示欧氏距离）、核函数kernel
set.seed(123)
kknn.train <- train.kknn(type ~ ., data = train, 
                         kmax = 25, distance = 2, 
                         kernel = c("rectangular", "triangular", "epanechnikov"))

## plot中X轴表示的是k值，Y轴表示的是核函数误分类观测百分比
plot(kknn.train)
```

<img src="50-KNN_files/figure-html/unnamed-chunk-3-1.png" width="672" />

```r
## 以调用对象看看分类误差和最优参数
kknn.train
```

```
## 
## Call:
## train.kknn(formula = type ~ ., data = train, kmax = 25, distance = 2,     kernel = c("rectangular", "triangular", "epanechnikov"))
## 
## Type of response variable: nominal
## Minimal misclassification: 0.212987
## Best kernel: rectangular
## Best k: 19
```

```r
## 从上面的数据可以看出，给距离加权不能提高模型在训练集上的正确率。而且从下面的代码 可以看出，它同样不能提高测试集上的正确率
kknn.pred <- predict(kknn.train, newdata = test)
table(kknn.pred, test$type)
```

```
##          
## kknn.pred No Yes
##       No  76  27
##       Yes 17  27
```
