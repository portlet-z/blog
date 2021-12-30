- 类型很大可以适合用auto关键字，自动推导出变量的类型

```c++
#include <iostream>
#include <vector>

int main() {
	std::vector<std::string> strings;
	strings.push_back("Apple");
	strings.push_back("Orange");
	for (std::vector<std::string>::iterator it = strings.begin(); it != strings.end(); it++) {
		std::cout << *it << std::endl;
	}
	for (auto it = strings.begin(); it != strings.end(); it++) {
		std::cout << *it << std::endl;
	}
	std::cin.get();
}
```

- 以下两个场景下适合使用auto关键字，比如类型非常长的话，如果就是int, string这样的不要使用，会降低代码的可读性。当进入到复杂代码包含了模板等，不得不使用auto,因为你不知道类型是什么

```c++
#include <iostream>
#include <vector>
#include <unordered_map>
class Device {};
class DeviceManager {
private:
	std::unordered_map<std::string, std::vector<Device*>> devices;
public:
	const std::unordered_map<std::string, std::vector<Device*>>& getDevices() const {
		return devices; 
	}
};
int main() {
	DeviceManager dm;
	const std::unordered_map<std::string, std::vector<Device*>>& devices = dm.getDevices();

	using deviceMap = std::unordered_map<std::string, std::vector<Device*>>;
	const deviceMap& devices0 = dm.getDevices();

	typedef std::unordered_map<std::string, std::vector<Device*>> deviceMap1;
	const deviceMap1& devices1 = dm.getDevices();

	const auto& devices2 = dm.getDevices();
	
	std::cin.get();
}
```

- auto关键字可以用在C++11中的后置返回类型，或者在C++14中可以用于函数，

```c++
auto getName() {
    return "portlet";
}

auto getName() -> char* {
    return "portlet";
}
```

