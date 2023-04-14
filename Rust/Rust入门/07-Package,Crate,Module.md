## Package, Crate, 定义Module

### Rust的代码组织
- 代码组织主要包括：
  - 哪些细节可以暴露，哪些细节是私有的
  - 作用域内哪些名称有效
  
- 模块系统
  - Package(包): Cargo的特性，让你构建，测试共享crate
  - Crate(单元包): 一个模块树，它可产生一个library或可执行文件
  - Module(模块): use: 让你控制代码的组织，作用域，私有路径
  - Path(路径): 为struct, function或module等项命名的方式
  
- Package和Crate
  - Crate的类型：binary, library
  - Crate Root: 是源代码文件，Rust编译器从这里开始，组成你的Crate的根Module
  - 一个Package: 
    - 包含1个Cargo.toml, 它描述了如何构建这些Crates
    - 只能包含0-1个library crate
    - 可以包含任意数量的binary crate
    - 但必须至少包含一个crate(binary 或 library)
    
- Cargo的惯例
  - src/main.rs: binary crate的crate root; crate名与package名相同
  - src/lib.rs: package包含一个library crate; library crate的crate root; crate名与package名相同
  - Cargo把crate root文件交给rustc来构建library 或 binary
  - 一个Package可以同时包含src/main.rs和src/lib.rs
    - 一个binary crate, 一个libary crate
    - 名称与package名相同
  - 一个Package可以有多个binary crate
    - 文件放在src/bin
    - 每个文件是单独的binary crate
    
- Crate的作用
  - 将相关功能组合到一个作用域内，便于在项目间进行共享：防止冲突
  - 例如rand crate, 访问它的功能需要通过它的名字:rand
  
- 定义module来控制作用域和私有性
  - module:
    - 在一个crate内，将代码进行分组
    - 增加可读性，易于复用
    - 控制项目(item)的私有性。public, private
  - 建立module:
    - mod关键字
    - 可嵌套
    - 可包含其他项(struct, enum, 常量, trait, 函数等)的定义
  - src/main.rs和src/lib.rs叫做crate roots:
    - 这两个文件(任意一个)的内容形成了名为crate的模块，位于整个模块树的根部
    - 整个模块树在隐式的crate模块下

```rust
mod front_of_house {
  mod hosting {
    fn add_to_waitlist() {}
    fn seat_at_table() {}
  }

  mod serving {
    fn take_order() {}
    fn serve_order() {}
    fn take_payment() {}
  }
}
```
  
  
  
  
  
  
