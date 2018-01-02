## Python Numpy库学习

* Python List的特点

  > 创建语法

  ```python
  L = [i for i in range(10)]
  ```

  1. 可以直接修改索引处的值

     L[5] = 100

  2. 修改索引处的值类型也可以变幻

     L[5] = 'Hello World'

     这样修改不会出错

  3. 切片操作

     L[2:8] L[2:] L[:10] L[2:8:2]

* Numpy Array

  > 创建numpy list

  ```python
  import numpy as np
  nparr = np.array([i for i in range(10)])
  np.zeros(10)  #10个元素都为0
  np.zeros(10).dtype  #dtype('float64')
  np.zeros((3,5))  #3行5列的矩阵，每个元素都为0
  np.ones(10)  #10个元素都为1
  np.ones((3,5))  #3行5列的矩阵，每个元素都为1
  np.full((3,5), 666)  #3行5列的矩阵，每个元素都为666
  np.full(shape=(3,5), fill_value=666)  #同上
  np.arange(0,10)
  np.linspace(0,20,10)  #0到20 10个元素
  ## 随机值
  np.random.randint(0,10,10)  #0到10直接10个随机数
  np.random.randint(4,8, size=(3,5))  #3行5列的矩阵，每个元素的值都为4到8之间
  ## 设置随机种子
  np.random.seed(666)
  ## 随机浮点数
  np.random.normal(10,1, (3,5))
  ```

* numpy的索引操作

  ```python 
  x = np.arange(16)  #0到15的数组
  x[3:9]  #切片
  x[3:9:2]  #3,5,7
  ind = [3,5,8]
  x[ind]  #3，5，8
  ind = np.array([0,2], [1,3])
  x[ind]  #二维数组
  X = x.reshape(4,-1)  #4行4列的矩阵
  ```

* numpy中的运算

```python
import numpy as np
L = np.arange(10)
2*L #每个元素都乘以2
X = np.arange(1,16).reshape((3,5)) #3行5列的二维矩阵
X + 1 #矩阵中的每个元素都加1
X - 1 #矩阵中的每个元素都减1
np.sin(X) np.cos(X) np.tan(X)
X//2  X*2 X%2 X/2 X**2
np.abs(X) np.exp(X) np.log(X) np.log2(X) np.log10(X)
```

* 矩阵运算

  ```python
  A = np.arange(4).reshape(2,2)
  B = np.full((2,2), 10)
  A + B #矩阵中的每个元素相加
  A * B #矩阵中的每个元素相乘
  A.dot(B) #矩阵乘法运算和 B.dot(A) 结果不一致
  A.T #A的转置

  ```

  ​