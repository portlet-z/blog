- union有点像类类型，或者结构体类型，只不过它一次只能占用一个成员的内存。如果我们有一个结构体，声明4个浮点数，我们可以有4*4个字节在这个结构体中，总共16字节。一个union只能有一个成员，如果在union声明4个浮点数abcd，union的大小仍然是4个字节。如果我改变a为5，bcd也会设为5
- union中不能使用虚方法，还有其他一些限制。通常union做的事情是和类型双关紧密相关的。
- 当你想给同一个变量取两个不同的名字时，union是非常有用的。
- union通常是匿名使用的，但是匿名union不能含有成员函数

```c++
int main() {
	struct Union {
		union {
			float a;
			int b;
		};
	};
	Union u;
    //IEEE754浮点数2.0f二进制表示为 0x40000000
    //第一位为符号数，2-9位为指数，10-32为底数
    //2.0 = 1 * 2^1, 指数为1, 1 = E - 127, E = 128;底数为1，对应的值为1 - 1 = 0
    //0, 0100 0000, 0000 0000 0000 0000 0000 000
    // ==> 0100 0000 0000 0000 0000 0000 0000 0000 => 0x40000000
	u.a = 2.0f;
	std::cout << u.a << std::endl; //2
	std::cout << u.b << std::endl; //1073741824
	std::cin.get();
}
```

```c++
#include <iostream>
struct Vector2 {
	float x, y;
};
struct Vector4 {
	union {
		struct {
			float x, y, z, w;
		};
		struct {
			Vector2 a, b;
		};
	};
	
};
void PrintVector2(const Vector2& vector){
	std::cout << vector.x << ", " << vector.y << std::endl;
}
int main() {
	Vector4 vector = { 1.0f, 2.0f, 3.0f, 4.0f };	
	PrintVector2(vector.a); //1, 2
	PrintVector2(vector.b); //3. 4
	vector.z = 500.0f;
	PrintVector2(vector.a); //1, 2
	PrintVector2(vector.b); //500, 4
	std::cin.get();
}
```

