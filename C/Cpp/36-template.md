- template不同于Java中的泛型，比泛型功能更强大，模板有点像宏。然而泛型却非常受制于类型系统 
- 模板允许定义一个可以根据用途进行编译的模板，可以让编译器为你写代码，基于一套规则
- 例如，当我写一个函数时，我在这个函数里面使用模板，我实际在做的是，创建一个蓝本，因此当我决定要调用这个函数时，我可以指定特定的参数，这个参数决定了放入到模板中的实际代码

```c++
#include <iostream>
//template<class T> class在此处和template是等价的
template<typename T>
void Print(const T& value) {
    std::cout << value << std::endl;
}
int main() {
    Print<int>(5);
    Print(5);
    Print(5.5f);
    Print("Hello World");
    std::cin.get();
}
```

- 这个Print函数只是一个模板，只有当我们调用这个函数时，这个函数才会被实际创建，使用我们给定的模板参数（来创建）
- 模板绝不仅仅局限于类型或任何东西，也不会局限于函数，你可以创建一整个类基于模板。实际上大量的C++标准模板库同样完全使用了模板，下面一个例子，不用类型作为模板参数

```c++
#include <iostream>
template<int N>
class Array {
private:
    int array[N];
public:
    int getSize() {
        return N;
    }
};
int main() {
    Array<5> array;
    std::cout << array.getSize() << std::endl;
    std::cin.get();
}
```

```c++
#include <iostream>

template<typename T, int N>
class Array {
private:
    T array[N];
public:
    int getSize() {
        return N;
    }
};
int main() {
    Array<std::string, 5> array;
    std::cout << array.getSize() << std::endl;
    std::cin.get();
}
```

-  日志系统，素材系统比如堆图像进行渲染，当你有一个可以包含各种不同类型的统一缓冲区时，在一定程度上模板是非常有用的
