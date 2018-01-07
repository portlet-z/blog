## CSS动画

#### transition补间动画

* 位置-平移(left/right/margin/transform)
* 方位-旋转(transform)
* 大小-缩放(transform)
* 透明度(opacity)
* 其他-线性变换(transform)

#### keyframe关键帧动画

* 相当于多个补间动画
* 与元素状态的变化无关
* 定义更加灵活

#### 逐帧动画

* 适用于无法补间计算的动画
* 资源较大
* 使用steps()

#### 过度动画和关键帧动画的区别

* 过度动画需要有状态变化
* 关键帧动画不需要状态变化
* 关键帧动画能控制更精细

#### CSS动画的性能

* 性能不坏
* 部分情况优于JS
* 但JS可以做到更好
* 部分高危属性 box-shadow等

