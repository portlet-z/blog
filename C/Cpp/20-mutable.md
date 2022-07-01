- 与const一起使用的情况

```c++
class Entity {
private:
    std::string name;
    mutable int count = 0; //用mutable修饰可以在const修饰的函数中改变
public:
    std::string& getName() const {
        count++;
    	return name;
	}
};

int main() {
    const Entity e;
    e.getName();
}
```

- lambda中使用的情况(不推荐，工程代码不要这样写)

```c++
int x = 8;
auto f = [&]() mutable { //中括号中&表示捕获引用(即传递变量的引用) =表示捕获值(即传递变量的值) &x传递x的引用，=x传递x的值
    std::cout << x++ << std::endl; //修改了x,这个lambda表达式要用mutable修饰
}
```

