- 运算符式我们使用的一种符号，通常代替一个函数来执行一些事情，不仅仅式数学运算符(+-*/%),也有其他常用的运算符包括dereference(逆向引用)运算符，箭头运算符，+=运算符，逗号运算符，用于内存地址的&运算符，左移运算符(<<cout),new,delete,[]{}
- 运算符重载，是给运算符重载赋予新的含义，或者添加参数，或者创建，允许在程序中定义或更改运算符的行为 。运算符就是函数

```c++
#include <iostream>

struct Vector2 {
	float x, y;
	Vector2(float x, float y) 
		: x(x), y(y) {}

	Vector2 add(const Vector2& other) {
		return Vector2(x + other.x, y + other.y);
	}

	Vector2 operator+(const Vector2& other) {
		return add(other);
	}

	Vector2 multiply(const Vector2& other) {
		return Vector2(x * other.x, y * other.y);
	}

	Vector2 operator*(const Vector2& other) {
		return multiply(other);
	}

	bool operator==(const Vector2& other) {
		return x == other.x && y == other.y;
	}

	bool operator!=(const Vector2& other) {
		return !(*this == other); //或者 return !operator==(other);
	}
};

std::ostream& operator<<(std::ostream& stream, const Vector2& other) {
	stream << other.x << ", " << other.y;
	return stream;
}

void main() {
	Vector2 position(4.0f, 4.0f);
	Vector2 speed(0.5f, 1.5f);
	Vector2 powerup(1.1f, 1.1f);

	Vector2 result1 = position.add(speed.multiply(powerup));
	Vector2 result2 = position + speed * powerup;

	std::cout << result1 << std::endl; // 4.55, 5.65
	std::cout << result2 << std::endl; // 4.55, 5.65
	std::cout << (result1 == result2) << std::endl; // 1
	std::cout << (result1 != result2) << std::endl; // 0
	std::cin.get();
} 