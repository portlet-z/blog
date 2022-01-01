- lambda本质上是我们定义一种叫做匿名函数的方式，我们用这种方式创建函数，不需要实际创建一个函数
- 只有你有一个函数指针，你都可以在C++中使用lambda,这就是它的工作原理

- 当向lambda传递某些变量时，我们应该用std::function修饰原始函数指针

```c++
#include <iostream>
#include <vector>
#include <functional>
void ForEach(const std::vector<int>& values, const std::function<void(const int&)>& func) {
	for (const int& value : values) {
		func(value);
	}
}
int main() {
	std::vector<int> values = { 1, 5, 4, 3, 2 };
    auto it = std::find_if(values.begin(), values.end(), [](const int& value) {return value > 2; });
	std::cout << *it << std::endl;
	int a = 5;
    //[=]传递外部所有的变量的值，[&]传递所有的变量的引用，[=a]传递a的copy, [&a]传递a的引用
	auto lambda = [&a](const int& value) { std::cout << a << std::endl; };
	ForEach(values, lambda);
	
	std::cin.get();
}
```

