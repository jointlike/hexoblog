---
title: 关于特征工程
---

指的是把原始数据转变为模型的训练数据的过程，目的是为了获得更好的训练数据特征，提高机器学习的上限，使得机器学习模型逼近这个上限（附图）。


# 数据特征准备


1. 基于业务需求与理解，要实现目标需求需要哪些数据，确定变量
2. 可用性评估：获取难度，覆盖率，准确率
3. 获取方法
4. 存储方法



# 一般步骤流程

处理流程不分先后，以达到训练模型的最佳期望为目的。

1. 数据预处理（清洗与构建）

2. 数据补充

3. 数据降维

4. 特征监控



## 数据预处理（清洗与构建）

1. 数据规整：清洗异常样本，如删除极值


2. 数据转换：标准化，归一化，离散化（二值化），one-hot(dummy coding)哑编码，其他变换

2.1 标准化（无量纲化）
基于特征矩阵的列，将特征值转换至服从标准正态分布,去除均值和方差缩放，通过(X-X_mean)/std计算每个属性(每列)，进而使所有数据聚集在0附近，方差为1，公式为：x' = (X-X_mean)/std
使用sklearn.preprocessing库的StandardScaler类对数据进行标准化的代码如下：
```
from sklearn.preprocessing import StandardScaler
# 标准化，返回值为标准化后的数据
StandardScaler().fit_transform(iris.data)
```

2.2 归一化：区间缩放
区间缩放法的思路有多种，常见的一种为利用两个最值进行缩放即基于最大最小值，将特征值转换到[0, 1]区间上，公式表达为：x' = x-min/max-min
使用preproccessing库的MinMaxScaler类对数据进行区间缩放的代码如下：
```
from sklearn.preprocessing import MinMaxScaler
#区间缩放，返回值为缩放到[0, 1]区间的数据
MinMaxScaler().fit_transform(iris.data)
```

2.3 归一化：正则化
标准化是依照特征矩阵的列处理数据，其通过求z-score的方法，将样本的特征值转换到同一量纲下。归一化是依照特征矩阵的行处理数据，其目的在于样本向量在点乘运算或其他核函数计算相似性时，拥有统一的标准，也就是说都转化为“单位向量”。规则为l2的归一化公式如下：x' = x/(Ej->m x[j]^2)^1/2

使用preproccessing库的Normalizer类对数据进行归一化的代码如下：
```
from sklearn.preprocessing import Normalizer
#归一化，返回值为归一化后的数据
Normalizer().fit_transform(iris.data)
```

2.4 二值化
定量特征二值化的核心在于设定一个阈值，大于阈值的赋值为1，小于等于阈值的赋值为0，公式表达如下：
x'= { 1, x > threshold; 0, x <= threashold

使用preproccessing库的Binarizer类对数据进行二值化的代码如下：
```
from sklearn.preprocessing import Binarizer
#二值化，阈值设置为3，返回值为二值化后的数据
Binarizer(threshold=3).fit_transform(iris.data)
```

2.5 哑编码
对定性特征用定量表示，为避免数值大小对模型产生影响所有用dummy coding
使用preproccessing库的OneHotEncoder类对数据进行哑编码的代码如下：
```
from sklearn.preprocessing import OneHotEncoder
#哑编码，对IRIS数据集的目标值，返回值为哑编码后的数据
OneHotEncoder().fit_transform(iris.target.reshape((-1,1)))
```

2.6 数据变换
常见的数据变换有基于多项式的、基于指数函数的、基于对数函数的。4个特征，度为2的多项式转换公式如下：

x1,x2,x3,x4,x5,x6,x7,x8,x9,x10,x11,x12,x13,x14,x15 = 1,x1,x2,x3,x4,x1^2,x1*x2,x1*x3,x1*x4,x2^2,x2*x3,x2*x4,x3^2,x3*x4,x4^2

使用preproccessing库的PolynomialFeatures类对数据进行多项式转换的代码如下：
```
from sklearn.preprocessing import PolynomialFeatures
#多项式转换
#参数degree为度，默认值为2
PolynomialFeatures().fit_transform(iris.data)
```
基于单变元函数的数据变换可以使用一个统一的方式完成，使用preproccessing库的FunctionTransformer对数据进行对数函数转换的代码如下：
```
from numpy import log1p
from sklearn.preprocessing import FunctionTransformer
#自定义转换函数为对数函数的数据变换
#第一个参数是单变元函数
FunctionTransformer(log1p).fit_transform(iris.data)
```

3. 数据补充
3.1 缺失值计算

特征值为NaN（not a number）,表示数据缺失。
使用preproccessing库的Imputer类对数据进行缺失值计算的代码如下：
```
from numpy import vstack, array, nan
from sklearn.preprocessing import Imputer
#缺失值计算，返回值为计算缺失值后的数据
#参数missing_value为缺失值的表示形式，默认为NaN
#参数strategy为缺失值填充方式，默认为mean（均值）
Imputer().fit_transform(vstack((array([nan, nan, nan, nan]), iris.data)))
```

3.2 特征衍生：

- 对原始数据加工，生成符合训练需求的变量



## 数据选择方法

1. filter：方差选择法，相关系数，卡方检验，信息增益、互信息
过滤法，按照发散性或者相关性对各个特征进行评分，设定阈值或者待选择阈值的个数，选择特征。
2. wrapper： 递归特征消除法
包装法，根据目标函数（通常是预测效果评分），每次选择若干特征，或者排除若干特征。
3. embedded： 基于惩罚项的特征选择法，基于树形模型的特征选择法
嵌入法，先使用某些机器学习的算法和模型进行训练，得到各个特征的权值系数，根据系数从大到小选择特征。类似于Filter方法，但是是通过训练来确定特征的优劣。



## 数据降维

- PCA主成分分析法 
- LDA线性判别分析法



## 数据特征验证与监控

1. 有效性分析：重要性，权重
2. 监控重要特征
