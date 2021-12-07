

- log.h

```c++
#include <iostream>

void Log(const char* message) {
	std::cout << message << std::endl;
}
```

- Log.cpp

```c++
#include "Log.h"

void InitLog() {
	Log("Init Log");
}
```

- Main.cpp

```c++
#include <iostream>
#include "Log.h"

int main() {
	Log("Hello World");
	std::cin.get();
}
```

1. 单独编译Log.cpp通过，单独编译Main.cpp通过

2. Build整个项目时会报error LNK2005(?Log@@YAXPEBD@Z) already defined in Log.obj

3. 构建错误原因：Log.cpp和Main.cpp文件都引入了Log.h文件，预处理时会把Log.h中的代码都会拷贝到Log.cpp和Main.cpp文件中。链接时会发现Log函数有两个一样的

4. 解决方案

   - Log.h中用inline修饰Log函数。用inline修饰的函数在链接阶段会把函数体部分拷贝到引用此函数的代码中

     ```c++
     #include <iostream>
     
     inline void Log(const char* message) {
     	std::cout << message << std::endl;
     }
     ```

   - Log.h中用static修饰Log函数。用static修饰的函数作用域只作用在当前源文件中

     ```c++
     #include <iostream>
     
     static void Log(const char* message) {
         std::cout << message << std::endl;
     }
     ```

   - Log.h文件中不要定义函数的实现，只定义函数的声明。