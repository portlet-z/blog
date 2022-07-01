- static关键字在C++中有两个意思，这取决于上下文，其中之一是在类或结构体外部使用static关键字，另一种是在类或结构体内部使用static 
- 类外面的static，意味着你声明为static的符号，链接将只是在内部，这意味这它只能对你定义它的翻译单元可见即当前cpp文件
- 然而类或结构体内部的静态变量意味着该变量实际上将于类的所有实例共享内存，这意味这该静态变量在你的类中创建的所有实例中与Java类中static的含义一样

```c++
#include <iostream>
extern int variable; // extern意味这它会在外部翻译单元中寻找variable

int main() {
    std::cout << variable << std::endl;
}
```

```c++
struct Entity {
    int x,y;
}

int main() {
    Entity e; //实例化e
    e.x = 2; //给x赋值为2
    e.y = 3; //给y赋值为3
    
    Entity e1 = {5, 8};//给x,y赋值
}
```

```c++
struct Entity {
    static int x,y;
};

int Entity::x; //x,y为static的，如果不加这两行声明e.x,e.y会在链接阶段报错
int Entity::y;

int main() {
    Entity e;
    e.x = 2;
    e.y = 3;
    
    Entity::x = 5;
    Entity::y = 8;
}
```

- local static 静态局部变量允许我们声明一个变量，它的生存期基本上相当于整个程序的生存期，然而它的作用范围被限制在这个函数内，但它其实和函数没有什么关系

```c++
void Function() {
    static int i = 0;
    i++;
    std::cout << i << std::endl;
}
int main() {
    Function();//1
    Function();//2
    Function();//3
}
```

```c++
//i定义为全局变量也可实现上述功能，但是任何地方都可以访问i,不安全
int i = 0;
void Function() {
    i++;
    std::cout << i << std::endl;
}
int main() {
    Function();//1
    Function();//2
    Function();//3
}
```

- static单例 

```c++
class Singleton {
private:
    static Singleton* instance;
public:
    static Singleton& Get() {return *instance;}
    void Hello() {}
};

Singleton* Singleton::instance = nullptr; //声明这个实例

int main() {
    Singleton::Get().Hello();
}
```

- local static 单例

```c++
class Singleton{
public:
    static Singleton& Get() {
        static Singleton instance;
        return instance;
    }
    void Hello() {}
};

int main() {
    Singleton::Get().Hello();
}
```

