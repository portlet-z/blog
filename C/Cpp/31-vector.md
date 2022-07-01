- vector本质上是一个动态数组

```c++
#include <iostream>
#include <vector>
struct Vertex {
    float x, y, z;
};
std::ostream& operator<<(std::ostream& stream, const Vertex& vertex) {
    stream << vertex.x << ", " << vertex.y << ", " << vertex.z;
    return stream;
}
int main() {
    //std::vector<int> ints; 与Java不同的是此处泛型可以传递基本类型
    std::vector<Vertex> vertices;
    vertices.push_back({ 1, 2, 3 });
    vertices.push_back({ 4, 5, 6 });
    vertices.push_back({ 7, 8, 9 });
    for (int i = 0; i < vertices.size(); i++) {
        std::cout << vertices[i] << std::endl;
    }
    //移除第二个元素
    vertices.erase(vertices.begin() + 1);
    //遍历是用&可以防止每次遍历时复制一个Vertex
    for (Vertex& v : vertices) {
        std::cout << v << std::endl;
    }
    std::cin.get();
}
```

- vector的优化：指定vector的大小

```c++
#include <iostream>
#include <vector>
struct Vertex {
    float x, y, z;
    Vertex(float x, float y, float z) : x(x), y(y), z(z) {

    }
    Vertex(const Vertex& vertex) : x(vertex.x), y(vertex.y), z(vertex.z) {
        std::cout << "Copied!" << std::endl;
    }
};
std::ostream& operator<<(std::ostream& stream, const Vertex& vertex) {
    stream << vertex.x << ", " << vertex.y << ", " << vertex.z;
    return stream;
}
int main() {
    //std::vector<int> ints; 与Java不同的是此处泛型可以传递基本类型
    std::vector<Vertex> vertices;
    vertices.push_back(Vertex(1, 2, 3)); // Copied!
    vertices.push_back(Vertex(4, 5, 6)); // Copied! Copied! 
    vertices.push_back(Vertex(7, 8, 9)); // Copied! Copied! Copied!
    // 会打印6次Cpoied!
    // vector容量默认为1，push了3个元素，会扩容两次
    // 每次新建了一个Vertex对象，然后将此对象复制到vector对象里，会发生一次复制
    
    std::vector<Vertex> vertices1;
    vertices1.reserve(3);//指定vector的大小
    //这种情况下不是传递我们已经构建的vertex对象，我们只是传递了构造函数的参数列表
    //它告诉我们的vector,在我们实际的vector内存中，使用以下参数，构造一个vertex对象
    //不会打印Copied!
    vertices1.emplace_back(1, 2, 3);
    vertices1.emplace_back(4, 5, 6);
    vertices1.emplace_back(7, 8, 9);
    std::cin.get();
}
```

