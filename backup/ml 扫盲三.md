# 神经网络搭建步骤 

- [神经网络搭建步骤](#神经网络搭建步骤)
  - [加载数据](#加载数据)
  - [定义网络模型](#定义网络模型)
  - [定义损失函数](#定义损失函数)
  - [定义优化器](#定义优化器)
  - [训练](#训练)
  - [测试](#测试)
  - [保存模型](#保存模型)


```mermaid
flowchart LR

数据-->网络结构
网络结构-->损失函数
损失函数-->优化器
优化器-->训练
训练-->测试
测试-->导出模型
```

## 加载数据

```python
import torch
from torchvision import datasets, transforms
import torch.optim as optim 
import torch.nn as nn
import torch.utils.data as data

train_data = datasets.MNIST(root='data/mnist', train=True, transform=transforms.ToTensor(), download=True)
test_data = datasets.MNIST(root='data/mnist', train=False, transform=transforms.ToTensor(), download=True)
```

> 由于数据规模，不可能将所有的数据加载一次训练。常规的做法是将数据分batch。

```python
batch_size = 128
train_loader = data.DataLoader(dataset=train_data, batch_size=batch_size, shuffle=True)
test_loader = data.DataLoader(dataset=test_data, batch_size=batch_size, shuffle=False)
```

## 定义网络模型

```python
class MLP(nn.Module):
    """
    constructor
    input size: 输入数据的维度
    hidden size: 隐藏层的大小
    output_size: 输出的分类数量 (0-9)
    """

    def __init__(self, input_size, hidden_size, output_size):
        super(MLP, self).__init__()
        # 第一个全连接层
        self.fc1 = nn.Linear(input_size, hidden_size)
        # 定义激活函数
        self.relu = nn.ReLU()

        # 定义第二个全连接层
        self.fc2 = nn.Linear(hidden_size, hidden_size)

        # 定义第三个全连接层
        self.fc3 = nn.Linear(hidden_size, output_size)

    def forward(self, x):
        """
        前向传播函数
        :param x: 
        :return: 
        """
        # 第一层运算
        out = self.fc1(x)
        # 将上一层的输出传给激活函数
        out = self.relu(out)
        # 将上一层的输出传给fc2
        out = self.fc2(out)
        # 同样需要激活函数
        out = self.relu(out)
        out = self.fc3(out)
        return out


input_size = 28 * 28
hidden_size = 512
output_size = 10

model = MLP(input_size, hidden_size, output_size)
```

## 定义损失函数

```python
# 定义损失函数 
# 分类问题，使用交叉墒函数
criterion = nn.CrossEntropyLoss()
```

## 定义优化器

```python
# 定义优化器
lr = 0.001
optimizer = optim.Adam(model.parameters(), lr=lr)
```

## 训练

```python
num_epochs = 10
for epoch in range(num_epochs):
    for i, (images, labels) in enumerate(train_loader):
        # 将image转换成向量
        images = images.reshape(-1, 28 * 28)
        # 将数据送到网络中
        outputs = model(images)
        # 计算损失
        loss = criterion(outputs, labels)
        # 梯度清零
        optimizer.zero_grad()
        # 反向传播
        loss.backward()
        # 更新参数
        optimizer.step()

        if (i + 1) % 100 == 0:
            print(f'Epoch [{epoch + 1}/{num_epochs}], Step [{i + 1}/{len(train_loader)}], Loss: {loss.item():.4f}', )
```

## 测试

```python
# test

with torch.no_grad():
    correct = 0
    total = 0

    for images, labels in test_loader:
        images = images.reshape(-1, 28 * 28)
        outputs = model(images)
        # 取出最大值对应的索引 即预测值
        _, predicted = torch.max(outputs.data, 1)
        total += labels.size(0)
        correct += (predicted == labels).sum().item()
    print(f'Accuracy of the network on the 10000 test images: {100 * correct / total}')
```

## 保存模型

```python
torch.save(model, 'mnist.pkl')
```