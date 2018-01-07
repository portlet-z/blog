## kNN  k近邻算法

> 欧拉距离公式

$$
\sqrt{(x^a - x^b)^2 + (y^a - y^b)^2}
$$

$$
\sqrt{(x^a - x^b)^2 + (y^a - y^b)^2 + (z^a - z^b)^2}
$$

$$
\sqrt{(X_1^a-X_1^b)^2+(X_2^a-X_2^b)^2+...+(X_n^a-X_n^b)^2}
$$

$$
\sqrt{\sum_{i=1}^n(X_i^a-X_i^b)^2}
$$

> 简单的算法实现

```python
import numpy as np
from math import sqrt
from collections import Counter

def kNN_classify(k, X_train, y_train, x):
    assert 1 <= k <= X_train.shape[0], 'k must be valid'
    assert X_train.shape[0] == y_train.shape[0], 'the size of X_train must equal to the size of y_train'
    assert X_train.shape[1] == x.shape[0], 'the feature number of x must be equal to X_train'
    
    distances = [sqrt(np.sum((x_train - x) ** 2)) for x_train in X_train]
    nearest = np.argsort(distances)
    
    topK_y = [y_train[i] for i in nearest[:k]]
    votes = Counter(topK_y)
    
    return votes.most_common(1)[0][0]
```



> sklearn中kNN的算法调用

```python
from sklearn.neighbors import KNeighborsClassifier
kNN_classifier = KNeighborsClassifier(n_neighbors=6)
kNN_classifier.fit(X_train, y_train)
y_predict = kNN_classifier.predict(x.reshape(1,-1))
y_predict[0]
```

> 仿照sklearn中kNN算法方式封装下

```python
import numpy as np
from math import sqrt
from collections import Counter

class kNNClassifier:
    def __init__(self, k):
        """初始化kNN分类器"""
        assert k>=1, 'k must be valid'
        self.k = k
        self._X_train = None
        self._y_train = None
        
    def fit(self, X_train, y_train):
        """根据训练数据集X_train和y_train训练kNN分类器"""
        assert X_train.shape[0] == y_train.shape[0], 'the size of X_train must be equal to the size of y_train'
        assert self.k <= X_train.shape[0], 'the size of X_train must be at least k.'
        self._X_train = X_train
        self._y_train = y_train
        return self
   
	def predict(self, X_predict):
        """给定待预测数据集X_predict,返回表示X_predict的结果向量"""
        assert self._X_train is not None and self._y_train is not None, 'must fit before predict!'
        assert X_predict.shape[1] == self._X_train.shape[1], 'the feature number of X_predict must be equals to X_train'
        
        y_predict = [self._predict(x) for x in X_predict]
        return np.array(y_predict)
    
    def _predict(self, x):
        """给定单个待预测数据x,返回x的预测结果值"""
        assert x.shape[0] == self._X_train.shape[1], 'the feature number of x must be equal to X_train'
        distances = [sqrt(np.sum((x_train - x) ** 2)) for x_train in self._X_train]
        nearest = np.argsort(distances)
        topK_y = [self._y_train[i] for i in nearest[:self.k]]
        votes = Counter(topK_y)
        return votes.most_common(1)[0][0]
    
    def __repr__(self):
        return "kNN(k=%d)" % self.k
```

#### 判断机器学习算法的性能(model_selection.py)

```python
import numpy as np

def train_test_split(X, y, test_ratio=0.2, seed=None):
    """将数据X和y按照test_ration分割成X_train, X_test, y_train, y_test"""
    assert X.shape[0] == y.shape[0], "the size of X must be equal to the size of y"
    assert 0.0 <= test_ratio <= 1.0, "test_ratio must be valid"

    if seed:
        np.random.seed(seed)

    shuffled_indexes = np.random.permutation(len(X))

    test_size = int(len(X) * test_ratio)
    test_indexes = shuffled_indexes[:test_size]
    train_indexes = shuffled_indexes[test_size:]

    X_train = X[train_indexes]
    y_train = y[train_indexes]

    X_test = X[test_indexes]
    y_test = y[test_indexes]

    return X_train,X_test,y_train,y_test
```

* metrics.py

```python
import numpy as np

def accuracy_score(y_true, y_predict):
    '''计算y_true和y_predict之间的准确率'''
    assert y_true.shape[0] == y_predict[0], "the size of y_true must be equal to the size of y_predict"
    
    return sum(y_true == y_predict) / len(y_true)
```

#### 超参数

