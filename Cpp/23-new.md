- new实际上一个operator(操作符 )，就像加减乘除一样，这意味这可以重载这个操作符，并改变它的行为

```c++
Entity* e = new Entity();
Entity* e = (Entity*)malloc(sizeof(Entity)); //这两行代码唯一不一样的是第一个代码会调用Entity的构造函数
```

- new创建的对象在堆上，需要手动删除delete, delete也是一个operator

```c++
int* a = new int;
delete a;

int* b = new int[50];
delete[] b;
```

- placement new,就是要决定前面的内存来自哪里，并没有真正的分配内存，在这种情况下，只需要调用构造函数，并在一个特定的内存地址中初始对象

```c++
int* b = new int[50]; //200 bytes
Entity* e = new(b) Entity(); //假设Entity的大小是200bytes
```

