- 枚举就是一个数值集合

```c++
enum Example {
    A, B, C
}; //默认从0自增 A=0, B=1, C=2

enum Example : unsigned char {
    A, B, C
};

enum Example {
    A = 2, B = 4, C = 9
}; 
```

