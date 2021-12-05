## 01-Cpp 第一课

![create-project](.\images\create-project.png)

- 解决方案(Solution)就是一组相互关联的项目(Project),它们可以是各种项目类型。基本上，解决方案就像工作台 ，每个项目本质上就是一组文件，它们被编译成某种目标二进制文件。不管它是一个库还是一个实际的可执行文件。
- Location项目所放位置不用默认的用户名下，路径太长，直接放到C盘目录下即可

```c++
#include <iostream>

int main() {
	std::cout << "Hello World!" << std::endl;
	std::cin.get();
}
```



