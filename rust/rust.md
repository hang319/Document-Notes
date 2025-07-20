# Rust

## 一、基础

#### （1）、工程基本操作

```bash 
cargo new greeting

cargo build

cargo run
```

- cargo new <project-name>：创建一个新的 Rust 项目。
- cargo build：编译当前项目。
- cargo run：编译并运行当前项目。
- cargo check：检查当前项目的语法和类型错误。
- cargo test：运行当前项目的单元测试。
- cargo update：更新 Cargo.toml 中指定的依赖项到最新版本。
- cargo --help：查看 Cargo 的帮助信息。
- cargo publish：将 Rust 项目发布到 crates.io。
- cargo clean：清理构建过程中生成的临时文件和目录。


```bash
println!("a is {}", a) #输出a

println!("a is {0}, a is {0}", a) #输出两次a

println!("{{}}");  # 输出 {}
```


#### （2）、变量

```bash
let a  = 1; # 不可变变量

let mut b = 1; # 可变变量

let a: u64 = 123; # 无符号64位整型
```

#### （3）、数据类型

- i32: 32位有符号整数
- u32: 32位无符号整数
- f64: 64位浮点数
- bool: 布尔值
- char: 字符

```bash
let x: i32 = 5;
let y: f64 = 5.0;
let is_true: bool = true;
let c: char = 'a';
```

#### （4）、函数

```bash
fn add(a: i32, b:i32) -> i32 { a+b }
```

#### （5）、条件判断

```bash
let num = 7;
if num > 5 {
    println(大于 5");
} else {
    println("小于等于 5");
}
```

#### （6）、循环

```bash
let mut count = 0;
loop {
    count += 1;
    if count == 10 {
        break;
    }
}

let mut num = 3;
while num > 0 {
    println!("{}", num);
    num -= 1;
}

for num in 1..4 {
    println!("{}", num);
}
```