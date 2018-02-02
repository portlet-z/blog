## 逻辑回归 Logistic Regression

* 逻辑回归：解决分类问题
* 将样本的特征和样本发生的概率联系起来，概率是一个数
* 逻辑回归既可以看做是回归算法，也可以看做是分类算法
* 通常作为分类算法，只可以解决二分类问题

$$
\hat{y}=f(x) \\\\
\hat{p}=f(x) \qquad \hat{y}=\begin{cases}
1, \hat{p} >= 0.5 \\\\
0, \hat{p} <= 0.5
\end{cases}
$$

* Sigmoid函数
  $$
  \hat{p}=\sigma(\theta^T.x_b) \qquad \sigma(t)=\frac{1}{1+e^{-t}}
  $$

  > 值域(0,1)  t>0时，p>0.5    t<0时，p<0.5

