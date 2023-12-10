# 神经网络简介

- [神经网络简介](#神经网络简介)
  - [训练过程](#训练过程)
  - [传播](#传播)
  - [如何计算损失](#如何计算损失)
  - [如何优化](#如何优化)
  - [梯度下降](#梯度下降)
  - [学习率](#学习率)
  - [激活函数](#激活函数)
  - [tensor](#tensor)
- [操作过程](#操作过程)
- [单变量线性回归](#单变量线性回归)
  - [构建模型](#构建模型)
  - [训练](#训练)
  - [测试](#测试)
  - [多变量线性回归模型](#多变量线性回归模型)
    - [模型训练](#模型训练)
    - [验证](#验证)
- [参考](#参考)

![神经网络](https://github.com/0x1042/0x1042.github.io/assets/7525242/86419435-239d-424d-afd2-021096a0c932)


给定1000个相亲对象的数据**特征**(**feature**),和对应的满意程度**标签**(**label**)，训练完成后，给定新的相亲对象数据来预测满意程度，即预估(**predict**)

## 训练过程 

> 训练模型的目标是从所有样本中找到一组平均损失“较小”的权重（w）和偏差（b）。

- 初始化，随机生成一些权重和偏差
- 计算损失，给定特征，计算出标签，得到它与实际的真实标签相差多远
- 优化，微调权重和偏置，使损失变小

## 传播

- 前向传播：将训练数据的特征送入网络，得到标签（预估的结果）
- 反向传播：计算损失并优化

## 如何计算损失 

- 使用损失函数(均方误差/对数损失/交叉熵...)

## 如何优化

- 使用优化器(随机梯度下降SGD，Adam...)

## 梯度下降

梯度下降法的基本思想可以类比为一个下山的过程，假设一个人在山上，此时他要以最快的速度下山，就需要梯度下降来帮助自己下山。具体来说，就是以自己现在所处的位置为基准，寻找一个山势最陡峭的方向，沿着高度下降的方向走，就能以最快速度到山底。

同理，将损失函数看作一座山，我们的目标就是找到这个损失函数的最小值（山底），那么我们就可以在初始点找到该点函数的梯度，沿着函数值下降的方向对参数进行更新，这就是梯度下降法。

梯度是一个向量（矢量），具有大小和方向，表示某一函数在该点处的方向导数沿着该方向取得最大值，即函数在该点处沿着该方向（此梯度的方向）变化最快，变化率最大。

梯度始终指向损失函数中增长最为迅猛的方向，梯度下降法算法会沿着负梯度的方向走一步，以便尽快降低损失。

## 学习率

> 可以理解为步长 

那么沿着负梯度方向进行下一步探索，前进多少才合适呢？用梯度乘以一个称为学习率（有时也称为步长）的标量，以确定下一个点的位置。所以学习率是指导我们该如何通过损失函数的梯度调整网络权重的一个参数（也成为超参数）。学习率越低，损失函数的变化速度就越慢。

类似加水和面，加多少水就是学习率

## 激活函数 

线性模型在处理非线性问题时往往手足无措，这时我们需要引入激活函数来解决线性不可分问题。激活函数（Activation function），又名激励函数，往往存在于神经网络的输入层和输出层之间，作用是给神经网络中增加一些非线性因素，使得神经网络能够解决更加复杂的问题，同时也增强了神经网络的表达能力和学习能力。


## tensor 

在TensorFlow中，所有的数据都通过张量的形式来表示。从功能的角度，张量可以简单理解为多维数组，零阶张量表示标量（scalar），也就是一个数；一阶张量为向量（vector），也就是一维数组；n阶张量可以理解为一个n维数组。需要注意的是，张量并没有真正保存数字，它保存的是计算过程。

# 操作过程

1. 初始化一个神经网络模型 
2. 为神经网络模型添加层
3. 设计层的神经元个数和`input shape`
4. 训练

# 单变量线性回归

> y = ax + w ，目标是得到合理的a和w，使得对于任意的输入，得到最接近原始值的y

```python

# x点（输入）
x_data = [1, 2, 3, 4]
# y点（标签）
y_data = [70, 80, 90, 100]

# 散点图
plt.scatter(x_data, y_data) 
```

## 构建模型 

```python
# 定义一个神经网络模型
# Sequential 连续的(这一层的输入一定是上一层的输出)
model = models.Sequential()

# 给模型添加层 
model.add(layers.Dense(units=1, input_dim=1))
model.summary()

# 定义损失函数和优化器 
# mean_squared_error: 均方误差 
# SGD 随机梯度下降，学习率是0.1
model.compile(loss=MSE, optimizer=SGD(0.01), metrics=['accuracy'])
```

## 训练 

```python
inputs = tf.Variable(name="x_data", dtype=tf.float32, initial_value=x_data)
labels = tf.Variable(name="y_data", dtype=tf.float32, initial_value=y_data)

history = model.fit(x=inputs, y=labels, batch_size=10, epochs=1000)
```

## 测试 

```python
predict_y = model.predict(x=[1024])
# [[10450.493]]
print(predict_y)
```

## 多变量线性回归模型 

> 房价预测

### 模型训练 

```python
import tensorflow as tf
import numpy as np
import pandas as pd
from tensorflow import keras
import matplotlib.pyplot as plt
from tensorflow.keras.losses import MSE

# 数据处理 
data = pd.read_csv('data/dataset_train.csv')
data.head(10)

x = data.iloc[0:, 1:9]

y = data.median_house_value

# 定义模型 

model = keras.models.Sequential()
model.add(keras.layers.Dense(1, input_shape=(x.shape[1],)))
model.summary()

# 添加优化器和损失函数 
model.compile(loss=MSE, optimizer='adam',    metrics=['acc'])

# 训练
hist = model.fit(x=x, y=y, epochs=200, workers=16, use_multiprocessing=True)

model.save(filepath='./house')
```

### 验证

```python
test = pd.read_csv('data/dataset_test.csv')
test.head(10)

# 切割测试集为x和y，用模型进行预测
test_x = test.iloc[0:, 1:9]
test_y = test.median_house_value
predict_y = model.predict(test_x)
x = np.linspace(0, 100, 100)

# 绘制预测结果
plt.plot(x, test_y[:100], "x-", label="real")
plt.plot(x, predict_y[:100], "+-", label="predict")

```
![house](https://github.com/0x1042/0x1042.github.io/assets/7525242/eeebac15-828d-4195-9c85-5b2a54be8498)



# 参考 

[tfbook](https://minghuiwu.gitbook.io/tfbook/di-si-zhang-dan-bian-liang-xian-xing-hui-gui-tesnsorflow-shi-zhan/di-si-zhang-dan-bian-liang-xian-xing-hui-gui-tesnsorflow-shi-zhan/4.8-xian-xing-hui-gui-wen-ti-tensorflow-shi-zhan)
[deep-learning-with-python-cn](https://cnbeining.github.io/deep-learning-with-python-cn/4-advanced-multi-layer-perceptrons-and-keras/ch15-understand-model-behavior-during-training-by-plotting-history.html)
[用tf搭建多元线性回归模型](http://120.25.234.115:8999/archives/%E7%94%A8tf%E6%90%AD%E5%BB%BA%E5%A4%9A%E5%85%83%E7%BA%BF%E6%80%A7%E5%9B%9E%E5%BD%92%E6%A8%A1%E5%9E%8B)