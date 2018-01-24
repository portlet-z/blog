## 主成分分析(Principal Component Analysis)

* 一个非监督的机器学习算法
* 主要用于数据的降维
* 通过降维，可以发现更便于人类理解的特征
* 其他应用：可视化，去噪

1. 对所有的样本进行demaen处理
2. 我们想要求一个轴的方向$w=(w1,w2)$使得我们所有的样本，映射到w以后，有

$Var(X_{project})=\frac{1}{m}\sum_{i=1}^m(X_{project}^{(i)}-\bar{X}_{project})^2$最大 或者 $Var(X_{project})=\frac{1}{m}\sum_{i=1}^m||X_{project}^{(i)}-\bar{X}_{project}||^2$最大

$\bar{X}_{project}=0$使得$Var(X_{project})=\frac{1}{m}\sum_{i=1}^m||X_{project}^{(i)}||^2$最大

方向向量$w$,$X^{(i)}.w=||X^{(i)}||.||w||.cos\theta$,由$||w||==1$,$X^{(i)}.w=||X^{(i)}||.cos\theta=||X_{project}^{(i)}||$

$Var(X_{project})=\frac{1}{m}\sum_{i=1}^m||X_{project}^{(i)}||^2=\frac{1}{m}\sum_{i=1}^{(i)}(X_1^{(i)}w_1+X_2^{(i)}w_2+…+X_n^{(i)}w_n)^2$ 

$=\frac{1}{m}\sum_{i=1}^m(\sum_{j=1}^nX_i^{(i)}w_j)^2$

一个目标函数的最优化问题，使用梯度上升法解决

* 目标，求$w$,使得$f(X)=\frac{1}{m}\sum_{i=1}^m(X_1^{(i)}w_1+X_2^{(i)}w_2+…+X_n^{(i)}w_n)^2$最大

$$
\nabla{f}=\begin{pmatrix}
\frac{\partial{f}}{\partial{w_1}} \\\\
\frac{\partial{f}}{\partial{w_2}} \\\\
\cdots \\\\
\frac{\partial{f}}{\partial{w_n}} \\\\
\end{pmatrix} = \frac{2}{m}\begin{pmatrix}
\sum_{i=1}^m(X_1^{(i)}w_1+X_2^{(i)}w_2+...+X_n^{(i)}w_n)X_1^{(i)} \\\\
\sum_{i=1}^m(X_1^{(i)}w_1+X_2^{(i)}w_2+...+X_n^{(i)}w_n)X_2^{(i)} \\\\
\cdots \\\\
\sum_{i=1}^m(X_1^{(i)}w_1+X_2^{(i)}w_2+...+X_n^{(i)}w_n)X_n^{(i)} \\\\
\end{pmatrix} =  \frac{2}{m}\begin{pmatrix}
\sum_{i=1}^m(X^{(i)}w)X_1^{(i)} \\\\
\sum_{i=1}^m(X^{(i)}w)X_2^{(i)} \\\\
\cdots \\\\
\sum_{i=1}^m(X^{(i)}w)X_n^{(i)} \\\\
\end{pmatrix}
$$

$$
\frac{2}{m}.(X^{(1)}w, X^{(2)}w,...,X^{(m)}w).\begin{pmatrix}
X_1^{(1)} & X_2^{(1)} & \cdots & X_n^{(1)} \\\\
X_1^{(2)} & X_2^{(2)} & \cdots & X_n^{(2)} \\\\
\cdots \\\\
X_1^{(m)} & X_2^{(m)} & \cdots & X_n^{(m)} \\\\
\end{pmatrix} = \frac{2}{m}.(Xw)^T.X \\\\
\nabla{f} = \frac{2}{m}X^T(Xw)
$$

* 求出第一主成分以后，如何求出下一个主成分

* 数据进行改变，将数据在第一个主成分上的分量去掉
  $$
  X^{(i)}.w = ||X_{project}^{(i)}|| , w为单位方向向量 \rightarrow X_{project}^{(i)}=||X_{project}^{(i)}||.w \\\\
  X^{'{(i)}}=X^{(i)} - X_{project}^{(i)}
  $$

*   在新的数据上求第一主成分



#### 高维数据向低维数据映射

 