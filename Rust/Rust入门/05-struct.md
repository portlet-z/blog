## 定义和实例化struct

### 什么是struct

- struct,结构体
- 自定义的数据类型
- 为相关联的值命名，打包=>有意义的组合

### 定义struct

- 使用struct关键字，并为整个struct命名
- 在花括号内，为所有字段(field)定义名称和类型

```rust
struct User {
  username: String,
  email: String,
  sign_in_count: u64,
  active: bool, //最后一行逗号也需要
}
```

### 实例化struct

- 想要使用struct,需要创建struct的实例
  - 为每个字段指定具体值
  - 无需按声明的顺序进行指定

```rust
let user1 = User {
  email: String::from("11@qq.com"),
  username: String::from("someone"),
  active: true,
  sign_in_count: 1,
}
```

### 取得struct里面的某个值

- 使用点标记法
- user1.email = String::from("aa");
- 一旦struct实例是可变的，那么实例中所有的字段都是可变的

### struct作为函数的返回值

```rust
fn build_user(email: String, username: String) -> User {
  User {
    email,
    username,
    active: true,
    sign_in_count: 1,
  }
}
```

### struct更新语法

- 当你想基于某个struct实例来创建一个新实例的时候，可以使用struct更新语法

```rust
let user2 = User {
  email: String::from("another@qq.com"),
  username: String::from("another"),
  active: user1.active,
  sign_in_count: user1.sign_in_count,
}
let user2 = User {
  email: String::from("another@qq.com"),
  username: String::from("another"),
  ..user1
}
```

### Tuple struct

- 可定义类似tuple的struct，叫做tuple struct
  - Tuple struct整体有个名，但里面的元素没有名
  - 使用：想给整个tuple起名，并让它不同于其它tuple, 而且又不需要给每个元素起名
- 定义tuple struct：使用struct关键字，后边是名字，以及里面元素的类型
- struct Color(i32,i32,i32); let black = Color(0,0,0);
- struct Point(i32,i32,i32); let origin = Point(0,0,0);
- black和origin是不同的类型，是不同tuple struct的实例

### Unit-Like Struct(没有任何字段)

- 可以定义没有任何字段的struct,叫做Unit-Like struct(因为与(), 单元类型类似)
- 适用于需要再某个类型上实现某个trait, 但是里面又没有想要存储的数据

### struct数据的所有权

```rust
struct User {
  username: String,
  email: String,
  sign_in_count: u64,
  acitve: bool,
}
```

- 这里的字段使用了String而不是&str
  - 该struct实例拥有其所有的数据
  - 只要struct实例是有效的，那么里面的字段数据也是有效的
- struct里也可以存放引用，但这需要使用生命周期
  - 生命周期保证只要struct实例是有效的，那么里面的引用也是有效的
  - 如果struct里面存放引用，而不使用生命周期，就会报错

## struct方法

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect = Rectangle {
        width: 30,
        height: 50,
    };
    println!("{}", area(&rect));
    println!("{:#?}", rect);
}

fn area(rect: &Rectangle) -> u32 {
    rect.width * rect.height
}
```

- 方法和函数类型：fn关键字，名称，参数，返回值
- 方法与函数不同之处：
  - 方法是在struct(enum, trait对象)的上下文中的定义
  - 第一个参数是self, 表示方法被调用的struct的实例

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect = Rectangle {
        width: 30,
        height: 50,
    };
    println!("{}", rect.area());
    println!("{:#?}", rect);
}
```

### 定义方法

- 在impl块里定义方法
- 方法的第一个参数是&self, 也可以获得其所有权或可变借用。和其他参数意义
- 更良好的代码组织

### 方法调用的运算符

- C/C++： object->something()和(*object).something()一样
- Rust没有 -> 运算符
- Rust会自动引用或解引用：在调用方法时就会发生这种行为
- 在调用方法时，Rust根据情况自动添加&, &mut或*以便object可以匹配方法的签名
- 下面两行代码效果相同
  - p1.distance(&p2);
  - (&p1).distance(&p2);

### 方法参数

- 方法可以有多个参数

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect = Rectangle {
        width: 30,
        height: 50,
    };
    let rect1 = Rectangle {
        width: 31,
        height: 51,
    };
    println!("{}", rect.area());
    println!("{:#?}", rect);
    println!("{}", rect1.can_hold(&rect));
}
```

### 关联函数

- 可以在impl块里定义不把self作为第一个参数的函数，它们叫做关联函数（不是方法）
- 关联函数通常用于构造器 String::from()

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }

    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}

fn main() {
    let s = Rectangle::square(20);
    let rect = Rectangle {
        width: 30,
        height: 50,
    };
    let rect1 = Rectangle {
        width: 31,
        height: 51,
    };
    println!("{}", rect.area());
    println!("{:#?}", rect);
    println!("{}", rect1.can_hold(&rect));
}
```

- :: 符号
  - 关联函数
  - 模块创建的命名空间
- 多个impl块： 每个struct允许拥有多个impl块