-  定义一个指针变量，如果不用const修饰，我们可以修改指针的内容，另外我们也可以修改指针

```c++
int age = 20;
int* a = new int;
*a = 2; //修改指针指向的内容
a = &age; //修改指针
```

- 禁止修改指针指向的内容

```c++
int age = 20;
const int* a = new int; //或者 int const* a = new int;
// *a = 2; 编译出错禁止修改指针指向的内容
a = &age; //指针还是可以修改的
```

- 禁止修改指针

```c++
int age = 20;
int* const a = new int;
*a = 2; //修改指针指向的内容
//a = &age; 编译出错，禁止修改指针
```

- 禁止修改指针以及指针的内容

```c++
int age = 20;
const int* const a = new int;
//*a = 2; //禁止修改指针指向的内容
//a = &age; //禁止修改指针
```

- 类中以及方法中的const

```c++
class Entity {
private:
    int X, Y;
public:
    int getX() const { //const修饰表示我们不能修改类中的成员变量
        // X = 2; 编译出错在这个方法内不能修改类的成员变量
        return X;
    }
};

void PrintEntity(const Entity& e) { //const修饰引用表示禁止修改指针
    std::cout << e.getX() << std::endl; //getX() 必须要用const 修饰表示禁止修改类
}
```

```c++
class Entity {
private:
    int* X, *Y;
public:
    const int* const getX() const { //第一个const表示禁止修改指针指向的内容，第二个表示禁止修改指针，第三个表示禁止修改类的成员变量
        return X;
    }
};
```



