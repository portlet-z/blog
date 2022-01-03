```c++
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
	std::vector<int> values = { 3, 5, 1, 2, 4 };
	//std::sort(values.begin(), values.end()); //默认从小到大排序
    // std::sort(values.begin(), values.end(), std::greater<int>()); //从大到小排序
    std::sort(values.begin(), values.end(), [](int a, int b) {return a > b; }); //自定义排序
	for (int& value : values) {
		std::cout << value << std::endl;
	}
	std::cin.get();
}
```

