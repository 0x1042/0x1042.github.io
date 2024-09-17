# ML扫盲二

- [ML扫盲二](#ml扫盲二)
- [环境搭建](#环境搭建)
- [机器学习的分类](#机器学习的分类)
  - [监督学习  `supervised learning`](#监督学习--supervised-learning)
  - [无监督学习 `unsupervised learning`](#无监督学习-unsupervised-learning)
  - [半监督学习 `semi-supervised learning`](#半监督学习-semi-supervised-learning)
  - [强化学习 `reinforcement learning`](#强化学习-reinforcement-learning)
- [回归  regression](#回归--regression)
- [线性回归 linear regression](#线性回归-linear-regression)
  - [什么是线性回归](#什么是线性回归)
  - [损失函数 loss function](#损失函数-loss-function)
  - [随机梯度下降](#随机梯度下降)
  - [from zero](#from-zero)
  - [use tf](#use-tf)
- [非线性回归 Non-linear Regression](#非线性回归-non-linear-regression)
- [逻辑回归 logistic regression](#逻辑回归-logistic-regression)
  - [分类问题](#分类问题)
  - [网络结构](#网络结构)
  - [损失函数](#损失函数)
  - [use tf](#use-tf-1)
- [学习资料](#学习资料)

#  环境搭建 

```shell

# 开发机启动jupyter notebook 

nohup jupyter notebook --no-browser --port=8889 &

# 本地端口转发 
ssh -N -f -L localhost:8888:localhost:8889 ubuntu
```

# 机器学习的分类

## 监督学习  `supervised learning`

> **训练数据包含正确的结果（label），即希望学习或者预估的目标.**

- 监督学习建立一个学习过程，将预测结果与 “训练数据”（即输入数据）的实际结果进行比较，不断的调整预测模型，直到模型的预测结果达到一个预期的准确率，比如分类和回归问题等。

- 常用算法包括决策树、贝叶斯分类、最小二乘回归、逻辑回归、支持向量机、神经网络等。

## 无监督学习 `unsupervised learning`

> 无监督学习的训练数据都是未经标记的，算法会在没有指导的情况下自动学习.

- 输入数据没有标签，而是通过算法来推断数据的内在联系，比如聚类和关联规则学习
- 常用算法包括独立成分分析、K-Means 和 Apriori 算法等

## 半监督学习 `semi-supervised learning` 

输入数据部分标签，是监督学习的延伸，常用于分类和回归。常用算法包括图论推理算法、拉普拉斯支持向量机

## 强化学习 `reinforcement learning`

输入数据作为对模型的反馈，强调如何基于环境而行动，以取得最大化的预期利益。与监督式学习之间的区别在于，它并不需要出现正确的输入 / 输出对，也不需要精确校正次优化的行为。强化学习更加专注于在线规划，需要在探索（在未知的领域）和遵从（现有知识）之间找到平衡。


# 回归  regression 

回归（regression）是能为一个或多个自变量与因变量之间关系建模的一类方法。 在自然科学和社会科学领域，回归经常用来表示输入和输出之间的关系，即根据数据，确定两种及以上变量间相互依赖的定量关系 $y = f(x_1,x_2,...x_n)$

# 线性回归 linear regression 

## 什么是线性回归 

> 为了解释线性回归，我们举一个实际的例子： 我们希望根据房屋的面积（平方英尺）和房龄（年）来估算房屋价格（美元）。 为了开发一个能预测房价的模型，我们需要收集一个真实的数据集。 这个数据集包括了房屋的销售价格、面积和房龄。 在机器学习的术语中，该数据集称为训练数据集（training data set） 或训练集（training set）。 每行数据（比如一次房屋交易相对应的数据）称为样本（sample）， 也可以称为数据点（data point）或数据样本（data instance）。 我们把试图预测的目标（比如预测房屋价格）称为标签（label）或目标（target）。 预测所依据的自变量（面积和房龄）称为特征（feature）或协变量（covariate）  https://zh.d2l.ai/chapter_linear-networks/linear-regression.html

简单理解为 $y=wx+b$

线性回归(Linear Regression)算法就是寻找一条最优的直线来拟合数据(可以扩展到多维)。线性回归通常采用给定的函数值与模型预测值之差的平方和最小为损失函数, 并使用最小二乘法和梯度下降法来计算最终的拟合参数。 

给定一个数据集，我们的目标是寻找模型的权重，使得根据模型做出的预测大体符合数据里的真实价格。 输出的预测值由输入特征通过线性模型的仿射变换决定，仿射变换由所选权重和偏置确定

房价的例子中 $price = w_{area}*area + w_{age}*age + b$ , 即变量(面积和房龄) 与因变量(房屋价格)之间存在线性关系。

## 损失函数 loss function 

损失函数（loss function）能够量化目标的实际值与预测值之间的差距。 通常我们会选择非负数作为损失，且数值越小表示损失越小，完美预测时的损失为0

回归问题中常见的 loss function 是 mse(平方误差函数) 

## 随机梯度下降 

梯度下降（gradient descent）几乎可以优化所有深度学习模型，它通过不断地在损失函数递减的方向上更新参数来降低误差。

梯度下降最简单的用法是计算损失函数（数据集中所有样本的损失均值） 关于模型参数的导数（在这里也可以称为梯度）。 但实际中的执行可能会非常慢：因为在每一次更新参数之前，我们必须遍历整个数据集。 因此，我们通常会在每次需要计算更新的时候随机抽取一小批样本， 这种变体叫做小批量随机梯度下降（minibatch stochastic gradient descent）。

## from zero 

- step1 生成数据集 

```python
import random
import tensorflow as tf
from d2l import tensorflow as d2l

def synthetic_data(w, b, num_examples):  #@save
    """生成y=Xw+b+噪声"""
    X = tf.zeros((num_examples, w.shape[0]))
    X += tf.random.normal(shape=X.shape)
    y = tf.matmul(X, tf.reshape(w, (-1, 1))) + b
    y += tf.random.normal(shape=y.shape, stddev=0.01)
    y = tf.reshape(y, (-1, 1))
    return X, y

true_w = tf.constant([2, -3.4])
true_b = 4.2
features, labels = synthetic_data(true_w, true_b, 1000)
```

- step2 读取数据

```python
def data_iter(batch_size, features, labels):
    num_examples = len(features)
    indices = list(range(num_examples))
    # 这些样本是随机读取的，没有特定的顺序
    random.shuffle(indices)
    for i in range(0, num_examples, batch_size):
        j = tf.constant(indices[i: min(i + batch_size, num_examples)])
        yield tf.gather(features, j), tf.gather(labels, j)
```

- step3 初始化模型参数 

```python

# 从均值为0、标准差为0.01的正态分布中采样随机数来初始化权重， 并将偏置初始化为0。
w = tf.Variable(tf.random.normal(shape=(2, 1), mean=0, stddev=0.01),
                trainable=True)
b = tf.Variable(tf.zeros(1), trainable=True)
```

模型训练的过程就是不断更新w和b，直到这些参数足够拟合我们的数据， 每次更新都需要计算损失函数关于模型参数的梯度，有了这个梯度，我们就可以向减小损失的方向更新每个参数（使用自动微分来计算梯度）


- step4 定义模型 

```python
def linreg(X, w, b):  #@save
    """线性回归模型"""
    return tf.matmul(X, w) + b
```

- step5 定义损失函数 

```python
def squared_loss(y_hat, y):  #@save
    """均方损失"""
    return (y_hat - tf.reshape(y, y_hat.shape)) ** 2 / 2
```


- step6 定义优化器 

```python
def sgd(params, grads, lr, batch_size):  #@save
    """小批量随机梯度下降"""
    for param, grad in zip(params, grads):
        param.assign_sub(lr*grad/batch_size)
```

- step7 train

```python
lr = 0.03
num_epochs = 3
net = linreg
loss = squared_loss

for epoch in range(num_epochs):
    for X, y in data_iter(batch_size, features, labels):
        with tf.GradientTape() as g:
            l = loss(net(X, w, b), y)  # X和y的小批量损失
        # 计算l关于[w,b]的梯度
        dw, db = g.gradient(l, [w, b])
        # 使用参数的梯度更新参数
        sgd([w, b], [dw, db], lr, batch_size)
    train_l = loss(net(features, w, b), labels)
    print(f'epoch {epoch + 1}, loss {float(tf.reduce_mean(train_l)):f}')
```

## use tf 

```python

import numpy as np
import tensorflow as tf
from d2l import tensorflow as d2l

true_w = tf.constant([2, -3.4])
true_b = 4.2
features, labels = d2l.synthetic_data(true_w, true_b, 1000)

def load_array(data_arrays, batch_size, is_train=True):  #@save
    """构造一个TensorFlow数据迭代器"""
    dataset = tf.data.Dataset.from_tensor_slices(data_arrays)
    if is_train:
        dataset = dataset.shuffle(buffer_size=1000)
    dataset = dataset.batch(batch_size)
    return dataset

batch_size = 10
data_iter = load_array((features, labels), batch_size)
# 定义网络
initializer = tf.initializers.RandomNormal(stddev=0.01)
net = tf.keras.Sequential()
net.add(tf.keras.layers.Dense(1, kernel_initializer=initializer))

# 损失函数 
loss = tf.keras.losses.MeanSquaredError()
# 优化器 
trainer = tf.keras.optimizers.SGD(learning_rate=0.03)


num_epochs = 3
for epoch in range(num_epochs):
    for X, y in data_iter:
        with tf.GradientTape() as tape:
            l = loss(net(X, training=True), y)
        grads = tape.gradient(l, net.trainable_variables)
        trainer.apply_gradients(zip(grads, net.trainable_variables))
    l = loss(net(features), labels)
    print(f'epoch {epoch + 1}, loss {l:f}')

w = net.get_weights()[0]
print('w的估计误差：', true_w - tf.reshape(w, true_w.shape))
b = net.get_weights()[1]
print('b的估计误差：', true_b - b)
```

# 非线性回归 Non-linear Regression

简单理解为 $y=wx^2+b$

# 逻辑回归 logistic regression 

**线性回归可以预测多少的问题，但是无法解决是不是的问题，比如某个电子邮件是不是垃圾邮件，这个广告用户是否点击**

所以，出现了逻辑回归，也称为 `softmax 回归`

## 分类问题 

简单理解，label 是 0或者是1，模型的输出是 这个输入可能是0还是1的概率 

## 网络结构 

## 损失函数 

最大似然估计/交叉熵

## use tf


# 学习资料 

https://zh.d2l.ai/chapter_linear-networks/linear-regression.html