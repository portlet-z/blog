- C语言风格的类型转换

```c++
int main() {
    double value = 5.25;
    int a = value; //隐式转换
    int b = (int)value; //强制类型转换
    std::cout << a << std::endl; //5
    std::cout << b << std::endl; //5
}
```

- C++语言风格的类型转换有以下4中转换。它们不做任何C风格类型转换不能做的事情，只是一些语法糖而已。此外会做一些额外的检查
  - static_cast
  - reinterpret_cast：当我不想转换任何东西，只是想把那种指针，解释成别的东西的时候用到
  - dynamic_cast：如果转换不成功，返回一个NULL
  - const_cast：移除或添加变量的const修饰符的。可以用它来隐式添加const,但大部分情况下是用来移除const的

```c++
#include <iostream>

class Base {
public:
	Base() {}
	virtual ~Base() {}
};

class Derived : public Base {
public:
	Derived() {}
	~Derived() {}
};

class AnotherClass : public Base {
public:
	AnotherClass() {}
	~AnotherClass() {}
};

int main() {
	Derived* derived = new Derived();
	Base* base = derived;
	AnotherClass* ac = dynamic_cast<AnotherClass*>(base); //NULL
	if (!ac) {
	}
	std::cin.get();
}
```

