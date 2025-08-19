---
title: Rust学习笔记
date: 2023-07-24 18:52:35
tags:
- Rust
---
# Rust学习笔记

学习资料来源：[Rust语言圣经(Rust Course)](https://course.rs/about-book.html)

## 基础入门

### 基本类型
- 数值类型
- 字符，布尔，单元类型

基本类型的创建都是创建在栈中

#### 语句与表达式
有`;`的是语句，没有的是表达式。
`;`很重要!!就是有没有返回值的重要!!!

该语句块返回一个返回值`y + 3`, Rust中返回值不需要`return`(也可以用`return`)
```rs
{
  let y = 1;
  y + 3
}
```

Rust中的三元
```rs
let y = if x % 2 == 1 {
        "odd"
    } else {
        "even"
    };
```

#### 函数
函数参数必须标注类型

函数没有返回值时，默认返回空元组`()`

`fn a() -> ! {}`表示函数永不返回(发散函数)

### 所有权和借用

> 释放内存的三种机制：

- 垃圾回收机制(GC)，在程序运行时不断寻找不再使用的内存，典型代表：Java、Go
- 手动管理内存的分配和释放, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++
- 通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查 Rust

栈：先进后出，栈中的所有数据都必须占用已知且固定大小的内存空间，假设数据大小是未知的，那么在取出数据时，你将无法取到你想要的数据

堆：先进先出，当向堆上放入数据时，需要请求一定大小的内存空间。操作系统在堆的某处找到一块足够大的空位，把它标记为已使用，并返回一个表示该位置地址的指针, 该过程被称为在堆上分配内存。接着，该指针会被推入栈中，因为指针的大小是已知且固定的，在后续使用过程中，你将通过栈中的指针，来获取数据在堆上的实际内存位置，进而访问该数据

#### 所有权

- Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者
- 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
- 当所有者(变量)离开作用域范围时，这个值将被丢弃(drop)

`String`存储在堆上
`String::from`中 `::`是调用操作符，表示调用`String`中的`from`方法。（暂且理解成`String.from`）

转移所有权

```rs
let s1 = String::from("hello");
let s2 = s1;
```

当 s1 赋予 s2 后，Rust 认为 s1 不再有效，因此也无需在 s1 离开作用域后 drop 任何东西，这就是把所有权从 s1 转移给了 s2，s1 在被赋予 s2 后就马上失效了。此时如果尝试访问s1，会报错.

上面的例子可以解读为 s1 被**移动**到了 s2 中

Rust 称呼 `let a = b` 为变量绑定

克隆（深拷贝）
```rs
let s1 = String::from("hello");
let s2 = s1.clone()
```

拷贝(浅拷贝)
```rs
let x = 5;
let y = x;
```
任何基本类型的组合可以 Copy ，不需要分配内存或某种形式资源的类型是可以 Copy 的

- 所有整数类型，比如 u32
- 布尔类型，bool，它的值是 true 和 false
- 所有浮点数类型，比如 f64
- 字符类型，char
- 元组，当且仅当其包含的类型也都是 Copy 的时候。比如，(i32, i32) 是 Copy 的，但 (i32, String) 就不是
- 不可变引用 &T ，例如转移所有权中的最后一个例子，但是注意: 可变引用 &mut T 是不可以 Copy的

#### 引用与借用

获取**变量的引用**，称之为**借用**(borrowing)

引用与解引用
```rs
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```
`&`符号即是引用，它们允许你使用值，但是不获取所有权

`let y = &x` 为 y对x的引用，比较时不能将整数类型与引用类型比较，因此必须对y解引用`*y`

可变引用

同一作用域，特定数据**只能有一个**可变引用：
```rs
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

这种限制的好处就是使 Rust 在编译期就避免数据竞争，数据竞争可由以下行为造成：

- 两个或更多的指针同时访问同一数据
- 至少有一个指针被用来写入数据
- 没有同步数据访问的机制

可变引用与不可变引用不能同时存在

引用的作用域 s 从创建开始，一直持续到它最后一次使用的地方，这个跟变量的作用域有所不同，变量的作用域从创建持续到某一个花括号 `}`

总的来说，借用规则如下：

- 同一时刻，你只能拥有要么一个可变引用, 要么任意多个不可变引用
- 引用必须总是有效的

### 复合类型

#### 字符串
切片: `[开始索引..终止索引]`

切片的单位是字节, 因此中午字符串不能按字数切片

```rs
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```
字符串切片是非常危险的操作，因为切片的索引是通过字节来进行，但是字符串又是 UTF-8 编码，因此你无法保证索引的字节刚好落在字符的边界上

操作字符串

- 追加: 在字符串尾部可以使用 `push()` 方法追加字符 char，也可以使用 `push_str()` 方法追加字符串字面量
这两个方法都是在原有的字符串上追加，并不会返回新的字符串。由于字符串追加操作要修改原来的字符串，则该字符串必须是可变的，即字符串变量必须由 mut 关键字修饰
- 插入: 使用 `insert()` 方法插入单个字符 char，也可以使用 `insert_str()` 方法插入字符串字面量
第一个参数是字符（串）插入位置的索引，第二个参数是要插入的字符（串），索引从 0 开始计数，如果越界则会发生错误。由于字符串插入操作要修改原来的字符串，则该字符串必须是可变的，即字符串变量必须由 mut 关键字修饰
- 替换：
  + `replace()` 该方法可适用于 String 和 &str 类型。接收两个参数，第一个参数是要被替换的字符串，第二个参数是新的字符串。该方法会替换所有匹配到的字符串。该方法是返回一个新的字符串，而不是操作原来的字符串
  + `replacen()` 该方法可适用于 String 和 &str 类型。 接收三个参数，前两个参数与 replace() 方法一样，第三个参数则表示替换的个数。该方法是返回一个新的字符串，而不是操作原来的字符串
  + `replace_range()` 该方法仅适用于 String 类型。接收两个参数，第一个参数是要替换字符串的范围（Range），第二个参数是新的字符串。该方法是直接操作原来的字符串，不会返回新的字符串。该方法需要使用 mut 关键字修饰。
- 删除：
  + `pop()` 删除并返回字符串的最后一个字符, 该方法是直接操作原来的字符串,但是存在返回值，其返回值是一个 Option 类型，如果字符串为空，则返回 None
  + `remove()` 删除并返回字符串中指定位置的字符, 该方法是直接操作原来的字符串。但是存在返回值，其返回值是删除位置的字符串，只接收一个参数，表示该字符起始索引位置。remove() 方法是按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误
  + `truncate()` 删除字符串中从指定位置开始到结尾的全部字符, 该方法是直接操作原来的字符串。无返回值。该方法 truncate() 方法是按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误。
  + `clear()` 清空字符串,该方法是直接操作原来的字符串。调用后，删除字符串中的所有字符，相当于 truncate() 方法参数为 0 的时候。
- 连接：使用 + 或者 += 连接字符串（还有一个`format`方法）
使用 + 或者 += 连接字符串，要求右边的参数必须为字符串的切片引用（Slice）类型。其实当调用 + 的操作符时，相当于调用了 std::string 标准库中的 add() 方法，这里 add() 方法的第二个参数是一个引用的类型。因此我们在使用 +， 必须传递切片引用类型。不能直接传递 String 类型。+ 是返回一个新的字符串，所以变量声明可以不需要 mut 关键字修饰

#### 元组
元组可以嵌套元组
```rs
let (x, y, z) = (1,2,3);
let uu = (1,2,3);
let x = uu.0;
let y = uu.1;
let z = uu.2;
```

#### 结构体
关键字 `struct`

```rs
struct User {
  username: String,
  email: String,
  age: u32,
  active: bool
}

let mut user1 = User {
  email: String::from("你好"),
  username: String::from("hello"),
  age: 18,
  active: true
}

let active = user1.active
```

结构体更新语法: `..` 语法表明凡是我们没有显式声明的字段，全部从 user1 中自动获取。需要注意的是 ..user1 必须在结构体的尾部使用
```rs
let user2 = User {
    email: String::from("another@example.com"),
    ..user1
};
println!("user1 : {}", user1) // 报错
println!("user1.email, {}", user1.email) // 正常
```

元组结构体
```rs
struct Position(i32, i32, i32)
let position1 = Position(32, 32, 32)
let Position(x, y ,z) = position1
```

单元结构体

定义一个类型，但是不关心该类型的内容, 只关心它的行为时，就可以使用 单元结构体
```rs
struct AlwaysEqual;
let subject = AlwaysEqual;
// 我们不关心 AlwaysEqual 的字段数据，只关心它的行为，因此将它声明为单元结构体，然后再为它实现某个特征
impl SomeTrait for AlwaysEqual {
}
```

输出结构体详情`#[derive(Debug)]`

```rs
#[derive(Debug)]
...
```
#### 枚举
通过`enum`定义枚举
```rs
enum Country {
  China("China"),
  America("America")
}
let country = Country::China

```
#### 数组
分为数组`array`和动态数组`Vector`
`array`长度固定，分配在栈上, 元素类型要统一，长度要固定
`Vector`长度可变，分配在堆上

类似字符串的`&str`和`String`

```rs
// [类型; 长度]
let a: [i32; 5] = [1,2,3,4,5]
let a = [3; 5]
// std::array::from_f
let array: [String; 8] = std::array::from_fn(|i| String::from("rust is good!"));

```
### 流程控制
if语句块可以充当表达式, 要保证每个分支返回的类型一样
```rs
let num = if true {
  5
}else {
  6
};
```

循环`for`、`while` 和 `loop`

```rs
for 元素 in 集合 {
  // 使用元素干一些你懂我不懂的事情
}
// 在循环中，修改该元素
for item in &mut collection {
  // ...
}
```

当有多层循环时，你可以使用 continue 或 break 来控制外层的循环。要实现这一点，外部的循环必须拥有一个标签 'label, 然后在 break 或 continue 时指定该标签

```rs
fn main() {
    let mut count = 0;
    'outer: loop {
        'inner1: loop {
            if count >= 20 {
                // 这只会跳出 inner1 循环
                break 'inner1; // 这里使用 `break` 也是一样的
            }
            count += 2;
        }
        count += 5;
        'inner2: loop {
            if count >= 30 {
                break 'outer;
            }

            continue 'outer;
        }
    }

    assert!(count == 30)
}
```
### 模式匹配
match 匹配
```rs
match target {
    模式1 => 表达式1,
    模式2 => {
        语句1;
        语句2;
        表达式2
    },
    _ => 表达式3
}
```
穷尽匹配，非重点匹配项可用 `_` 替代(相当于switch...case...中的 default)

if...let...
```rs
  let v = Some(3u8);
  match v {
      Some(3) => println!("three"),
      _ => (),
  }
  // 等价于
  if let Some(3) = v {
    println!("three");
  }
```
当你只要匹配一个条件，且忽略其他条件时就用 if let ，否则都用 match。

matches!宏
```rs
// matches!宏 返回 true|false
let foo = 'f';
assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));
```

### 方法method
使用`impl`定义方法

rust方法和其他语言的区别
```rs
struct Circle {
  x: i32,
  y: i32
}

impl Circle {
  // new是Circle的关联函数，因为它的第一个参数不是self，且new并不是关键字
  // 这种方法往往用于初始化当前结构体的实例
  fn new(x: i32, y:i32) -> Self {
    Circle {
      x: x,
      y: y
    }
  }
  // Circle的方法，&self表示借用当前的Circle结构体
  fn area(&self) -> f64 {

  }
}
```
![方法](https://pica.zhimg.com/80/v2-0d848e960f3279999eab4b1317f6538e_1440w.png)

关联函数
因为是函数，所以不能用 `.` 的方式来调用，我们需要用 `::` 来调用，例如 `let sq = Rectangle::new(3, 3);`。这个方法位于结构体的命名空间中：`::` 语法用于关联函数和模块创建的命名空间

### 泛型和特征
#### 泛型
- 提前声明，跟泛型函数定义类似，首先我们在使用泛型参数之前必需要进行声明 `Point<T>`，接着就可以在结构体的字段类型中使用 T 来替代具体的类型
- x 和 y 是相同的类型
```rs
struct Point<T> {
  x: T,
  y: T
}
```

方法中使用泛型

使用泛型参数前，依然需要提前声明：impl<T>，只有提前声明了，我们才能在Point<T>中使用它，这样 Rust 就知道 Point 的尖括号中的类型是泛型而不是具体类型。需要注意的是，这里的 Point<T> 不再是泛型声明，而是一个完整的结构体类型，因为我们定义的结构体就是 Point<T> 而不再是 Point
`impl<T>`
```rs
impl<T> Point<T> {
  fn x(&self) -> &T {
    &self.x
  }
}
```

const泛型: 针对值的泛型
```rs
fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {
    println!("{:?}", arr);
}
```

泛型的性能：Rust 通过在编译时进行泛型代码的 单态化(monomorphization)来保证效率。单态化是一个通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程。
即根据输入生成对应所有输入类型的处理函数，因此没有运行时开销，牺牲打包时间和容量

泛型特征
`<T: std::ops::Add<Output = T>>` add特征



### **生命周期**
标记的生命周期只是为了取悦编译器，让编译器不要难为我们
生命周期语法以`'`开头
```rs
&i32
&'a i32
&'a mut i32
```

三条消除规则
- 每一个引用参数都会获得独自的生命周期
- 若只有一个输入生命周期(函数参数中只有一个引用类型)，那么该生命周期会被赋给所有的输出生命周期，也就是所有返回值的生命周期都等于该输入生命周期
- 若存在多个输入生命周期，且其中一个是 &self 或 &mut self，则 &self 的生命周期被赋给所有的输出生命周期

静态生命周期`'static`, 拥有该生命周期的引用可以和整个程序活得一样久。
