- this是一个指向当前对象实例的指针，该方法属于这个对象实例

```c++
class Entity {
public:
	int x, y;
	Entity(int x, int y) {
		this->x = x; //(*this).x = x;
		this->y = y; //(*this).y = y;

	}
};
```

