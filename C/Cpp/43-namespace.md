- namespace的主要目的时避免命名冲突

```c++
#include <iostream>
#include <vector>
#include <functional>
namespace apple {
	void print(const char* text) {
		std::cout << text << std::endl;
	}
}
namespace orange {
    inline namespace inner {
		void inner_print() {
			std::cout << "inner" << std::endl;
		}
	}
	void print(const char* text) {
		std::string temp = text;
		std::reverse(temp.begin(), temp.end());
		std::cout << temp << std::endl;
        inner_print();
	}
    void print_again() {}
}
int main() {
	apple::print("Hello");
	orange::print("Hello");
    
    using orange::print; //只导入print方法
	print("Hello");
	//print_again(); 
    
    namespace a = apple; //别名
    a::print("Hello");
	std::cin.get();
}
```

- using namespace apple;这意味着从命名空间apple导入所有东西
- inline namespace，使得所有东西对父命名空间可用

