```c++
class Entity {
private:
    std::string m_name;
    int m_age;
public:
    Entity(const std::string& name) : m_name(name), m_age(0) {}
    
    Entity(int age) : m_name("Unknown"), m_age(age) {}
};

void PrintEntity(const Entity& entity) {
    //print entity
}

int main() {
    Entity e0 = "portlet"; //隐式转换调用Entity(const std::string& name)
    Entity e1 = 22; //隐式转换调用Enitity(int age)
    
    PrintEntity(22); 
    //PrintEntity("portlet"); 错误，"portlet"实际上是一个char数组，隐式转换只会转换一层
    PrintEntity(std::string("portlet"));
    PrintEntity(Entity("portlet"))
}
```

- explicit关键字放在构造函数前面，意味着禁止隐式转换

```c++
class Entity {
private:
    std::string m_name;    
    int m_age;
public:
    explicit Entity(const std::string& name) : m_name(name), m_age(0) {}
    
    explicit Entity(int age) : m_name("Unknown"), m_age(age) {}
};

void PrintEntity(const Entity& entity) {
    //print entity
}

int main() {
   Entity e0("portlet");
   Entity e1(22);
}
```

