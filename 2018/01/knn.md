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

> 算法实现

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



