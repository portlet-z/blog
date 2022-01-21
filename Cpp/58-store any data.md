```c++
#include <iostream>
#include <any>

void* operator new(size_t size) {
	return malloc(size);
}

int main() {
	std::any data;
	data = std::string("portlet");
	std::string& string = std::any_cast<std::string&>(data);
	std::cout << string << std::endl;
	std::cin.get();
}
```

