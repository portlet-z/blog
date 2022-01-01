- std::array和c语言风格的数组在内存中是一样的，它们都存储在栈中。std::vector是在堆上

```c++
#include <iostream>
#include <array>
template<int N>
void PrintArray(const   std::array<int, N>& data) {
	for (int i = 0; i < data.size(); i++) {
		std::cout << data[i] << std::endl;
	}
} 
int main() {
	std::array<int, 5> data;
	data[0] = 1;
	PrintArray(data);
	std::cin.get();
}
```

