# Rust知识总结-《The Rust Programming Language》

## 1.Getting Started

### 基本命令

* `rustup update`: 更新rustup版本到最新版本；
* `rustup self uninstall`: 删除Rust和rustup工具；
* `rustc --version`: 检查rust是否正常安装；

### cargo命令

* `cargo new xx`: 创建xx目录，并初始化项目，会包含依赖文件、目录结构等；
* `cargo new --vcs=git`: 初始化Git仓库，会同时创建.gitignore文件；
* cargo配置文件的格式是 `TOML(Tom's Obvious, Minimal Language)`，其中 `[package]`表示package相关的配置信息，例如包名等；`[dependencies]`表示项目的三方依赖；
* `cargo build`: 下载依赖库，并编译项目为可执行文件
* `cargo run`: 编译并运行
* `cargo check`: 代码错误检查，但是不会有编译物产出，用于开发期快速验证语法错误，避免每次都是用build浪费时间；
* `cargo build --release`: 生产环境编译，编译时会进行一些优化，编译目标是`target/release`替代开发期的`target/debug`。优化可以让Rust代码运行更快速，但是编译时间也会变长。因此，日常开发用 `cargo build`，线上发布用 `cargo build --release`，但是注意，做benchmark一定要用release！
* Cargo使用语义版本号规则，[Semantic Versioning](https://semver.org/)；
* Cargo从`registry`上拉取依赖库的最新版本信息，依赖库的版本信息都存储在 [crates.io](https://crates.io/) 上，Rust基于Crates.io来构建生态，让开发者可以把自己开发的开源Rust项目提交到这里，以供其他开发者使用；
* 与其他版本依赖库管理工具类似，Cargo的依赖库信息写在`Cargo.toml`，使用`Cargo.lock`文件保存已存在的库的信息以及依赖关系；
* `cargo update`用于更新依赖库版本，会忽略`Cargo.lock`文件中的版本信息，根据`Cargo.toml`中的配置来获取最新版本，并且把最新的版本信息写入到lock文件中。
* `cargo doc --open` 构建一个包含所有依赖库的可以本地浏览的开发文档；

## 2.Programming a Guessing Game

* Rust内置`std`标准库支持，其中，有一些标准库是默认导入的（prelude），不需要额外声明即可使用，[Prelude Libs](https://doc.rust-lang.org/std/prelude/index.html)
* 对于未默认导入的std标准库，需要显式导入才能使用，使用的关键词是 `use`。例如：`use std::io;`
* 函数定义关键词：`fn`，函数参数语法：`()`，函数包体语法：`{}`
* Rust中，变量默认是不可变的，需要添加关键字 `mut` 来定义变量
* `String`是Rust标准库提供的可变的UTF-8编码的文本，[string](https://doc.rust-lang.org/std/string/struct.String.html)
* `::new` 中的`::` 语法表示new是跟String类型关联的函数（`associated function`），关联函数表示在某个类型上实现的函数。
* `&`语法表示参数是一个引用，允许代码可以在多个地方访问这个变量，避免内存中频繁复制变量。
* Rust标准库中有多个命名为`Result`的类型，既有通用的[Result](https://doc.rust-lang.org/std/result/enum.Result.html)类型，也有特定版本的类似[io::Result](https://doc.rust-lang.org/std/io/type.Result.html)这样的类型。
  * `Result`的类型是枚举，可以有固定的变体variants，经常与`match`搭配使用。
  * 具体来说，`Results`有两个变体值：`Ok`和`Err`,Ok表示成功，Err表示失败。
  * Rusults类型也定义了对应方法，例如io::Results类型定义的[expect method](https://doc.rust-lang.org/std/result/enum.Result.html#method.expect)。
  * `match`表达式由多个分支组成，分支包含一个模式匹配规则`pattern`和匹配后要执行的代码，
* `println!`与C语言的printf类似，用于打印信息到控制台，在Rust中，使用`{}`符号作为placeholds占位符，可以把`{}`想象为螃蟹的钳子；
* Rust语法支持同名变量覆盖，避免作用类似但类型不同的变量的命名复杂化，比如 `guess`（数字） -> `guess_str`（字符串）

## 3.Common Programming Concepts

编程语言的常规基本概念：`variables`、`basic types`、`functions`、`comments`、`control flow`；

### 3.1 Variables and Mutability

#### 变量

默认情况下，变量是不可变的，这有利于充分利用Rust的安全和并发能力，但同时，也提供了可变变量的能力，即使用关键字 `mut` 修饰变量；

#### 常量

Rust中仍然有常量这个概念，常量与变量的区别有几个点：

* 常量不能使用`mut`修饰；
* 常量不是默认不可变，而是完全不可变；
* 使用关键字 `const` 定义常量，而变量使用关键字 `let`；
* 常量必须标注数据类型，这与变量的自动推导不同；

Rust中常量的命名惯例是大写和下划线组合，例如`const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;`。

#### shadowing

Rust支持变量定义覆盖，即Rust中的`shadowing`语法，后面定义的变量会覆盖前面定义的同名变量。

### 3.2 Data Types

Rust中所有值都有类型，数据类型主要包含：scalar标量/基础变量、compound组合变量。需要注意的是Rust是静态类型语言(static typed)，必须在编译器明确变量的类型。编译器通常情况下可以根据值的类型推导出(infer)变量的类型，以及如何正确使用这些变量。

#### Scalar Types

标量表示一个单独的值，Rust有4中主要的标量类型：整数integers、浮点数floating-point number、布尔类型Booleans、字符类型characters；

Rust中的内置整数类型有以下6中长度，有符号的使用`i`作为类型前缀，无符号的使用`u`作为类型前缀。有符号类型整数值范围：*-(2n - 1) to 2n - 1 - 1*，无符号类型的值范围：*0 to 2n - 1*。`arch`类型是根据架构类确定的，64位架构CPU就是64-bit。需要注意的是，Rust中整数默认是`i32`。

|  Length | Signed | Unsigned |
|:-------:|:------:|:--------:|
| 8-bit   | i8     | u8       |
| 16-bit  | i16    | u16      |
| 32-bit  | i32    | u32      |
| 64-bit  | i64    | u64      |
| 128-bit | i128   | u128     |
| arch    | isize  | usize    |

Rust中常量可以用以下方式来表示，分别是小数、16进制、8进制、二进制、字符：

| Number literals |   Example   |
|:---------------:|:-----------:|
| Decimal         | 98_222      |
| Hex             | 0xff        |
| Octal           | 0o77        |
| Binary          | 0b1111_0000 |
| Byte (u8 only)  | b'A'        |

Rust对于数字溢出的处理，如果使用`--release`编译选项，Rust不会panic，而是会`wrap around`到最小值。

##### Floating-Point Types

Rust有两种浮点数类型：`f32`和`f64`，默认类型是`f64`，因为现代CPU处理f32和f64速度是一样的，但是f64精准度更高。浮点数规范遵循 IEEE-754标准，f32是单精度，f64是双精度。

##### Numberic Operations

Rust对所有的数字类型，都是支持基本的数学操作，加、减、乘、除、求余等

##### Boolean Type

Rust有布尔类型：`true`和`false`.

##### The Charater Type

Rust的`char`字符类型，与大多数编程语言的常见的原始字符类型一样。使用*单引号*定义字符常量，使用*双引号*定义字符串。Rust字符类型有4个字节长度，表示一个unicode标量值。Rust中char值的范围是 `U+0000 to U+D7FF` 和 `U+E000 to U+10FFFF`。

#### Compound Types

组合类型可以组合多个值到一个类型，Rust主要有两种基础组合类型：tuples元组、arrays数组。

##### The Tuple Type

元组是常见的把多个不同类型的值组合为一个值的类型，元组有固定长度，一旦定义长度就不能再变化。

声明一个元组：`let tup: (i32, f64, u8) = (500, 6.4, 1);`；析构一个元组：`let (x, y, z) = tup;`，然后就可以使用变量名访问元组中的变量。

另外一种直接访问元组的方式是使用`.`和index来访问，例如 `tup.0`，表示范围元组变量tup的中第一个值。

`()`表示特殊类型的元组，即 `unit type`，表达式隐式会隐式范围unit value。

##### The Array Type

array数组是多个*同一类型*的值的集合。*Rust中的数组必须有固定长度！*，这与其他常见的语言不相同，因此，Rust中的数组在分配在栈上，而不是堆上。如果想要使用可变长度的数组，可以使用Rust标准库提供的vector类型。

可以使用中括号`[]`、元素类型、分号、数组中元素数量等一起来声明数组，例如 `let a: [i32; 5] = [1, 2, 3, 4, 5];`

使用中括号、分号、数量来声明一个数组，例如 `let a = [3; 5];`，声明一个包含5个3的数组。

Rust数组访问越界时，会抛出运行时错误，这与其他底层编程语言不同，例如C语言，Rust具备数据越界访问的感知能力。

### 3.3 Functions

Rust程序的入口是`main`函数，使用`fn`关键字定义。Rust使用`蛇形(snake case)`命名法作为惯用方式，包含函数和变量名，由小写字母和下划线组成。

#### Parameters

Rust与其他语言类似，也是分为形参paramter，实参arguments概念。Rust的函数签名中，必须定义每个参数的类型。

#### Statements and Expressions

Rust是expression-based编程语言，因此，必须要清晰理解`statements`声明和`expressions`表达式之间的区别：`Statements are instructions that perform some action and do not return a value. Expressions evaluate to a resulting value`。表达式执行得到结果，构成了我们要写的大部分Rust代码，例如 `x + 1`或`5 + 6`。

#### Functions with Return Values

Rust函数定义中，用箭头`->`符号来声明返回值的类型。

### 3.4 Comments

单行注释和多行注释都使用 `//`，例如

``` rust
// So we’re doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what’s going on.
```

### 3.5 Control Flow

#### `if`表达式

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

#### if let

鉴于if是表达式，可以在let声明语句的右侧使用if表达式来赋值操作，限制是if/else返回的值类型必须是一致的，例如：

```rust
let number = if condition { 5 } else { 6 };
```

#### Repetition with Loops

Rust提供了三种循环：`loop`、`while`、`for`。

##### loop用法

* 使用`loop {}`语法，可以搭配`break`和`continue`使用，类似Java中的do {} while(1)

* Rust的`loop`语法支持返回值：

```rust
let result = loop {
    counter += 1;

    if counter == 10 {
        break counter * 2;
    }
};
```

##### while条件循环

while条件循环用法与其他语言类似：

```rust
while number != 0 {
    println!("{}!", number);

    number -= 1;
}
```

##### for循环

for循环主要用于集合遍历，搭配是 for...in...，for循环具备安全性和便利性，是Rust中最常用的循环语法

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {}", element);
    }
}
```

for循环在一些执行一定次数循环的场景也可以使用，通过for和Range搭配：

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

## 4.Understanding Ownership

Ownership是Rust与其他编程语言最大区别之一，让Rust在没有GC的情况下具备了内存安全能力。

### 4.1 What is Ownership?

`Ownership`是Rust管理内存的一系列规则组合。与常规的GC方案或者开发者显式管理方案不同，Rust的内存管理方案是通过编译器检查一系列规则实现的。如果代码违反了任何规则，编译器就会报错。因此，Ownership的规则并不会降低程序的执行性能。

#### The Stack and The Heap

* 通常来说，编程语言并不会要求开发者时刻关注stack栈和heap堆。但是类似Rust这样的系统编程语言，值保存在堆或栈上会影响语言的表现，以及会影响你的决策。
* 所有存储在栈stack上的数据，在编译期都必须有明确的固定大小。对于编译期无法确定大小，或大小会变化的数据，则需要存在在堆heap上。
* 堆上的数据组织是不规则的，在堆上申请空间，然后把数据存入堆上，内存分配器会在堆上找到一个足够大的空白点，标注为已使用，然后返回一个指针给开发者，表示堆内存的地址。
* 在栈上压入数据速度要比堆上分配空间快很多，因为栈上压入数据，不需要内存分配器alloctor搜索足够存在大的新空间。相对而言，在堆上分配空间，需要分配器首先找到一个足够大的空间来存储数据，然后进行记录，为下一次分配做准备。
* 访问堆上的数据比访问栈的数据要慢很多，因为访问堆上数据需要通过指针来访问对应的地址。同一个地址段的数据访问速度比跳转访问更快。
* 调用函数时，传入的参数和函数的局部变量都会被压入到栈中，函数执行完毕以后，会从栈中弹出；
* `Ownership`的*主要目的是用来管理堆上的数据*，我们需要通过理解一些堆上的操作来理解Ownership，理解以后，就可以不需要再关注这些概念。比如，堆上分配的数据有哪些、减少堆上的重复数据、清理堆上未使用数据，这些就是ownership要解决的问题。

#### Ownership Rules

* `Each value in Rust has a variable that's called its owner`，Rust中的每一个值都有一个称之为owner（所有者）的变量variable，即变量是值的owner（所有者）
* `There can only be one owner at a time`，一个值同一时间只能有一个owner
* `When the owner goes out of scope, the value will be dropped.`，当owner离开一定的范围或作用域，对应的值就会被抛弃

#### Variable Scope

`A scope is the range within a program for which an item is valid`，Rust的作用域与其他语言类似，即作用域是程序中某个数据项有效的范围。

#### Memory and Allocation

字符串常量通常会在编译期硬编码到执行文件中，因此，字符串常量会很快和很高效。

在Rust语言中，变量拥有的内存会在变量离开作用域时自动释放，当变量离开作用域时，Rust会自动调用一个特殊的函数 `drop`，开发者可以把释放内存的代码放在这个函数里。Rust在离开作用时，会自动调用drop函数来达到自动释放的作用。

##### a) Ways Variables and Data Interact: Move

Rust中多个变量可以以多种形式与同一个数据产生交互，这里讲第一个交互方式 `move`。

首先，需要注意的是：`Ownership主要用来管理堆上的数据的`。一方面，*由于栈上的数据可以通过快速压入和弹出来访问数据，数据复制速度很快*，当把指向栈上的数据的变量赋值给另一变量时，对于常见的标量类型，通常会直接复制这个数据给另一变量。同时，对于堆上的数据，Rust才需要使用ownership规则来管理变量，栈上的数据由自动复制的特性，天然会满足ownership规则。

Rust中的String的存储结构如下图所示，例如`let s1 = String::from("hello");`，左侧一组数据(变量？)存储在栈上（指针指向右侧堆上的数据、长度、空间等），右侧数据存在堆上（字符串内容）:

```text
                  stack         heap

 ┌────────────┬────────┐        ┌───────┬───────┐
 │    name    │  value │        │ index │ value │
 ├────────────┼───┬────┤        ├───────┼───────┤
 │    ptr     │   │┼┼┼┼┼─────►  │   0   │   h   │
 ├────────────┼───┴────┤        ├───────┼───────┤
 │    len     │    5   │        │   1   │   e   │
 ├────────────┼────────┤        ├───────┼───────┤
 │  capacity  │    5   │        │   2   │   l   │
 │            │        │        ├───────┼───────┤
 └────────────┴────────┘        │   3   │   l   │
                                ├───────┼───────┤
                                │   4   │   o   │
                                └───────┴───────┘
```

通过上面的概念，我们理解到变量s1是如何存储的。如以下的代码可以进一步解释Move的概念，把变量s1赋值给变量s2，并且在赋值以后仍然在`println!`中使用变量s1。参考ownership的原则，因此，s1和s2离开作用域时，都会自动调用`drop`函数来释放字符串，这会会导致堆上存储value值的内存空间被双重释放。因此，这个操作在Rust中不符合ownership原则！所以，在编译时期这段代码就会报错。

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1); // 编译错误
```

这里，`Rust的解决方案是move`，即把第一个变量s1置为无效状态，即后续不能再使用变量s1，这也是上面代码报错的原因所在。这里我们对赋值的动作进一步解释，这里不能仅仅是是把s2当做s1的浅拷贝，应该是数据的所有权从s1转移到s2，s1变量失效了。*这里隐藏着一个Rust的设计原则，Rust永远不会为数据创建深拷贝deep copy，只会进行浅拷贝shallow copy*。因此，任何自动拷贝都会在运行时性能层面假设为廉价操作。PS: 其实这里的概念与C的浅copy与深copy概念类似，例如传参时都是值传递，都是传递栈上的变量，而不是堆上的数据

##### b) Ways Variables and Data Interact: Clone

Rust调用深拷贝可以使用`clone`方法，clone方法不仅赋值栈上的数据即变量，也会赋值堆上的数据。但这会导致一个问题，就是运行时开销成本过大，会影响执行效率。

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2); // 工作正常
```

##### c) Stack-Only Data: Copy

*对于存储在栈上的数据，例如integers整数数据，在编译期就能确定数据的大小，因此，对于真实值的复制速度非常快。换句话说，对于栈上的数据，常见的标量类型，会默认执行拷贝，深拷贝和浅拷贝并没有区别，因此也不需要通过move来遵守ownership规则，也不需要显式调用clone。*

例如：

```rust
let x = 5;
let y = x; // 直接拷贝

println!("x = {}, y = {}", x, y); // 工作正常
```

Rust为存储在栈上的数据类型，例如integers整数，实现了一个称之为`Copy trait`的特殊注解，通过这个trait支持了变量的复制。如果一个类型实现了`Copy trait`，变量在赋值给另外一个变量以后，会仍然有效。但如果某个类型实现了 `Drop trait`，就不能实现`Copy trait`。

通常来说，任何一组简单的标量值都实现Copy trait，需要内存分配或需要某种资源类型的不能实现copy，以下几种类型实现了Copy trait

* 整数类型
* 布尔类型
* 浮点数
* 字符类型
* 只包含实现Copy的数据类型的元组，例如 `(i32, i32)`可以，但是 `(i32, string)`不可以

#### Ownership and Functions

传递一个值给函数，在语义上，类似传递一个值给一个变量。因此，传递给函数的变量需要move或copy，规则与赋值操作类似。因此，加入我们把堆上的变量传递一个函数的调用以后，就发生了move操作，这个变量就不能再使用了。但对于栈上的数据，由于会发生copy，所以可以继续使用

```rust
let s = String::from("hello");  // s comes into scope
takes_ownership(s);             // s's value moves into the function...
                                // ... and so is no longer valid here
let x = 5;                      // x comes into scope
makes_copy(x);                  // x would move into the function,
                                // but i32 is Copy, so it's okay to still
                                // use x afterward
```

#### Return Values and Scope

返回值同样会转移ownership所有权，参考以下的代码，main函数执行完毕的时候，s1和s3的dropped会被调用，但是s2由于所有权被转移到叶子函数作用域中，s2的drop则不会被调用。

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.
```

`变量所有权使用都遵守同一个规则：赋值给另一个变量的时候，所有权就会转移！当包含堆数据的变量离开作用域时，就会调用drop函数进行清理，除非数据所有权已经转移到另一个变量！`

### 4.2 References and Borrowing

Rust中的引用与C语言的指针很类似，即指向别的变量拥有的数据的地址，但与C指针不同的是，Rust引用会保证指向的地址是特性类型的有效数据，保证不会指向野指针。与C语法类似，Rust语法中会使用`&`用于获取引用地址，`*`用于解引用。`由于引用本身并不会拥有值，所以在引用离开作用域时，并不会drop哪些引用指向的值，不会导致数据的释放，这就是引用借用reference borrowing。我理解这也是引用本身存在的最重要的意义，例如变量在子函数调用中传递时，我不需要把所有权先转移到子函数，然后子函数调用完毕后再把所有权转移回来给一个变量接收，这样才会保证传递给子函数的变量没有被释放。这时，只需要传递引用即可，而引用不会调用drop trait，不需要关心离开作用域后堆上内存的释放问题。简单来说，因为本身并没有所有权，只能用，不能释放，就是引用最大的价值，传递变量时简化内存管理`。总体来说，Rust编译器针对引用会有以下约束的校验：

* `在给定的任何时间，只能有一个可变引用或多个不可变引用`；
* `引用必须永远有效`；

用一个现实生活的例子来解释引用和借用这两个概念，以及他们对所有权的影响。我们把创建引用的动作叫作`borrowing借用`。在现实生活中，如果有人拥有一些东西，你去可以跟他们去“借用”这些东西。因此，当你用完的时候，你需要把这些东西归还给他们。你只是借用，而不是拥有所有权。

#### Mutalbe References可变引用

Rust引用中最容易出现问题或最难理解的点是可变引用使用。可变引用由于会修改应用的数据，在并发场景下就可能会导致资源竞争，进而引发数据安全问题。因此，在Rust Mutable References时，编译器会有以下的约束来规避任何潜在的资源竞争data races问题。

首先我们先明确一下，什么情况下会出现资源竞争问题，当同时出现以下三个现象时，就会有潜在的资源竞争问题：

1. 超过两个指针在同一时间访问同一个数据；
2. 其中至少有一个指针被用于写入数据；
3. 没有其他机制保证同步访问数据；

针对这个资源竞争的问题，Rust有第一个约束，即`在给定的任何时间，只能有一个可变引用或多个不可变引用`。具体来说，这个规则可以分为一下几个场景：

1. 在特定时间点上，只能有一个可变引用指向特定的数据。一方面，不允许有其他可变引用指向这个数据，另一方面，也不允许有其他不可变引用指向这个变量。因为可变引用可能会改变这个数据，从而造成data races；
2. 但Rust允许多个不可变引用，同时指向同一个数据。因为不可变引用不会修改这个数据，多个引用不会互相影响，就不不会造成资源竞争；

|             | 可变引用 | 不可变引用 |
|-------------|:---------:|:-----------:|
| 可变引用   | ❌         | ❌           |
| 不可变引用 | ❌         | ✅           |

尤其需要注意的是，`引用的作用域规则与变量不相同，引用的作用于范围是从引用被创建开始，一直持续到引用最后一次被用到`。这会带来一些特殊的case，从代码层面粗看起来会违反可变引用约束，但是事实上是符合规则，这就是因为引用的特殊作用域规则导致，例如下面的代码是可以正常执行的：

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem，因为可以允许多个不可变应用同时存在，这里没有问题；
    println!("{} and {}", r1, r2);
    // 由于变量r1和r2在这以后不会再被用到，因此这个引用r1和r2在这里会失效

    let r3 = &mut s; // 由于r1和r2已经失效，这里有效的引用只有可变引用r3，因此也是符合Rust引用约束的。
    println!("{}", r3);
}

```

#### Dangling References悬垂引用

Rust中不会出现C语言中那样的野指针问题，即悬垂引用问题，`a pointer that references a location in memory that may have been given to someone else--by freeing some memory while preserving a pointer to that memory.`，指针指向的内存地址，可能会被分配其他变量，或者在指针有效的时候，就直接释放内存。

在Rust中，*编译器会保证引用不会永远不会成为Dangling References*，即确保符合`引用永远必须有效`原则，我们看以下的代码，由于引用指向的变量会被提前释放，进而会出现悬垂引用问题，Rust针对这个情形就会出现编译报错：

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String { // dangle 函数调用会返回一个字符串的引用
    let s = String::from("hello");
    &s
}  // 这里我们会返回一个字符串的引用s，但是由于s离开函数作用域时，对应的值就会被dropped，内存就会被释放。这个时候，&s引用指向的内存其实是无效的，因此，Rust编译器就会识别这里会出现悬垂引用问题，编译会出现错误；
```

### 4.3 The Slice Type切片类型

Slices切片是一种`特殊的引用`，指向集合中的某一段连续的元素序列。因为是引用，所以`切片没有ownership所有权`。

Rust中使用语法 `[starting_index..ending_index]`来创建切片，start_index表示起始索引位置，ending_index表示结束索引位置加一。例如

```rust
    let s = String::from("hello world");

    let hello = &s[0..5]; // 引用字符串：hello，即字符串索引范围是0~4；
    let world = &s[6..11]; // 引用字符串：worrld，即字符串索引范围是6~10；
```

Rust的Range语法`..`，如果起始索引从0开始，可以省略起始索引；如果结束索引表示字符串的结尾，可以省略结束索引；例如：

```rust
let s = String::from("hello");
// 下面两种写法等同，省略了0；
let slice = &s[0..2];
let slice = &s[..2];
```

涉及字符串，就必须特别注意编码问题，Rust的字符串切片的边界必须位于有效的UTF-8字符边界以内。

Rust字符串切片与传统C语言中基于索引的子字符串表示法，有一个显著的优势。在过往基于索引的方法中，我们获得一个子字符串的范围索引，但是无法规避字符串本身操作导致的索引无效问题，例如下面的case：

```rust
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // 返回索引值5，表示第一个单词子字符串的索引范围是0~5

    s.clear(); // 通过这段代码清空字符串或修改字符串，编译器也不会报错；

    // 返回的索引值仍然是5，但是由于字符串本身已经被清空，这个索引值其实是无效的
    // 因此，过往需要我们手动去维护子字符串索引值和字符串之间的关系，并且容易疏漏和出现运行时错误；
}
```

但Rust的字符串切片语法可以在编译器规避这种问题，保证切片索引指向的字符串是始终有效的。上面的代码例子使用引用改写：

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}

fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s);
    s.clear(); // 编译报错，---- immutable borrow later used here
    println!("the first word is: {}", word);
}
```

Rust编译器在 `s.clear()`这一行会报错，提示`mutable borrow occurs here`。为什么会出现这行报错呢？我们回想一下借用规则约束，如果变量已经有不可用引用，就不能再有可变引用。针对这里，由于 `clear` 方法会截断String，所以过程中会产生一个可变引用。但又由于，不可变引用word这个时候仍然有效，前面讲过，引用的作用域是创建到最后一次被使用，这里word会在后面的`printlin!`函数使用。所以会产生一个情形，clear产生的可变引用和word不可变引用同时共存，这就是编译器提示这个错误的原因。

#### String Literals Are Slices

有一个需要注意到点，在Rust中，`字符串字面量是切片！！！`，例如`let s = "Hello, world!";`，这里s是`切片不可变引用str`。

#### String Slices as Parameters

设计Rust函数的API时，尽量使用字符串切片引用作为参数类型，可以充分利用Rust的`deref coercions`特性，可以使API更加通用且不会有任何损失；

#### Other Slices

切片除了字符串，也适合其他数据类型场景，例如 `let a = [1, 2, 3, 4, 5];`，可以使用 `&[i32]`表示对应的切片类型，内部存储了起始元素的引用以及长度，与字符串切片存储机制完全一样。

## 5.Using Structs to Structure Related Data

Rust结构体类似其他语言中的对象，是封装自定义数据结构的基础语法。

### 5.1 Defining and  Instantiating Structs

Rust结构体的语法，与其他现代语言语法非常类似，例如ES6/Swift/Go/TS等，这部分只需要记住几个语法点就行：

**语法一**：Rust的结构体定义和初始化语法，与TS的interface的定义和初始化语法比较类似：1) 用关键词 `struct`来定义结构体。2) 结构体名字是对数据集合的有意义的描述。3) 在大括号内部定义fileds字段，名字和类型组合而成的键值对；4) 如果结构体的实例是可变的，结构体的fields字段可以被修改；

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    // 结构体可变，所以字段也可以修改
    user1.email = String::from("anotheremail@example.com");
}
```

**语法二**：`Rust结构体支持字段初始化语法糖`，类似ES6对象的扩展语法类似，直接使用变量名作为结构体的字段名，且变量的值也是结构体对应字段的值，用于简化对象初始化代码，参考：[属性的简洁表示法](https://es6.ruanyifeng.com/#docs/object#%E5%B1%9E%E6%80%A7%E7%9A%84%E7%AE%80%E6%B4%81%E8%A1%A8%E7%A4%BA%E6%B3%95)，例如：

```rust
fn build_user(email: String, username: String) -> User {
    // 支持字段快捷初始化语法
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

**语法三**：Rust中有一个`struct update syntax`，用已有的结构体实例初始化其他结构体，这个用法与ES6中的数组和对象扩展运算符Spread Operator非常类似，参考：[扩展运算符](https://es6.ruanyifeng.com/#docs/array#%E6%89%A9%E5%B1%95%E8%BF%90%E7%AE%97%E7%AC%A6)。但需要注意的是，受限于Rust的内存管理机制，如果用于初始化*目的结构体*的*源结构体*的字段会发生所有权转移，即发生move操作，那么源结构体就会变成无效。例如下面例子中user1就在赋值以后就不能再使用了

```rust
fn main() {
    // 省略user1初始化代码
    
    // 场景1：user1不再有效。由于user1中的username字段的数据的所有权被转移给user2结构体，所以导致user1变量就不再有效；
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
    // 场景2：user1仍然有效。由于username和email都是直接赋值初始化的，并未发生从user1转移所有权给user2。
    // 剩下的active和sign_in_count字段，由于已经实现copy协议，会直接复制而不是转移所有权；
    let user2 = User {
        email: String::from("another@example.com"),
        username: String::from("username"),
        ..user1
    };
}
```

**语法四**：Rust支持一种特殊的语法，`tuple structs`，即元组结构体，元组结构体有结构体类型定义名字，但不能通过名字显示访问结构体的字段struct field。这个语法最有用的场景是，`给元组添加一个类型`，当需要把元组变成一个与其他元组不同的类型，但同时又不需要给其中的字段都额外添加一个多余的名字，就可以用这种语法简化结构体定义。例如 Color结构体

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

这里需要注意的是，`每个结构体定义都是特定的类型，即使结构体中的字段的名字和类型都是一样的，但它们仍然是两种结构体类型`。例如上面例子中的Color和Point，虽然都是由三个i32数字组成，但是它们是不同的类型。

**语法5**：Rust支持定义空结构体，即 `unit-like structs`。这个语法的适用场景后面介绍traits的时候会介绍；

### 5.2 Examples

Rust也支持Debug环境下直接打印对象，与OC的 `-(NSString *)description`方法和Java中的`toString()`方法的作用类似，可以把一个对象输出为一个字符串。在Rust语法中，则是通过`fmt::Display` trait支持的。Rust对于重要的类型都内置了Display默认实现，包括整数和字符串的打印都是通过这个Display trait支持的。

`println!("rect1 is {:?}", rect1);`，`:?`表示输出格式为Debug格式的；对于结构体，我们可以通过添加`#[derive(Debug)]`注解来添加Debug打印信息；

除了Debug派生trait之外，也可以通过 `dbg!宏`来打印值的Debug格式显示；

### 5.3 方法语法

与Go语法类似，方法与函数的差异在于方法是与结构体绑定在一起的，其他语法基本一致。Rust中的方法语法差异有，1) 方法必须定义在一个特定的环境中（结构体、枚举、trait派生对象等）；2) 方法的定义的第一个参数必须是self，指向对应定义环境的被调用实例对象，例如结构体实例等。

```rust
impl Rectangle { // 指定定义上下文是结构体Rectangle
    fn area(&self) -> u32 { // 第一个参数必须是被调用结构体实例的引用
        self.width * self.height
    }
}
```

与其他语言的构造类方法类似，Rust支持Associated Functions关联函数语法，用于创建一个结构体的实例并返回，例如：

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
}
```

与Swift的扩展语法类似，Rust支持有多个`impl`块实现，可以把不同类型的方法在不同的impl块中实现。

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

## 6.Enums and Pattern Matching

Rust枚举类似函数式编程语言中的代数数据类型（algebraic data types），例如Haskell、OCaml。

### 6.1 Defining an Enum

#### 枚举基础定义

**语法一**，与Swift的枚举语法相似程度很高，除了常规的类型定义以外，每个枚举变体可以关联某种类型的数据，并且每个变体的数据的类型和数量都可以不同。

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}
let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

**语法二**：变体的类型支持结构体类型，如果仅仅是把数据聚合表示不同的类型，可以直接使用 struct tuple语法快速定义枚举的变体类型：

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}

// Tuple Struct
struct ChangeColorMessage(i32, i32, i32); 

enum Message {
    Quit,
    ChangeColor(ChangeColorMessage),
}

```

**语法三**，Rust的枚举，与结构体类似，直接使用 `impl` 为枚举定义方法。

#### Option Enum

*Programming language design is often thought of in terms of which features you include, but the features you exclude are important too.*

Rust语言与常见的语言不通，Rust并不支持null空类型。这个决策基于null发明者的总结，*Null References: The Billion Dollar Mistake*，总之，null是一个非常容易触发潜在问题的概念。Rust为了从根本上避免这个问题，不支持nulls。Rust提供的解决方案与Swift的Optional语法类似，Rust则是基于特殊的枚举类型来替换null，即标准库中定义的枚举`Option<T>`，[Enum std::option::Option](https://doc.rust-lang.org/std/option/enum.Option.html)。Option枚举可以安全同等实现null想要表达的概念，`a null is a value that is currently invalid or absent for some reason.`，即表示一个值是否有值或是否为空的状态。

```rust
enum Option<T> {
    None,
    Some(T),
}
```

需要注意的是，与Swift的Optional语法类似，`Option<T>`和`T`并不是同一个类型，必须得先把`Option<T>`转换为`T`才能使用，这个做法可以处理一个常见的null错误问题，`assuming that something isn't null when it actually is`，即常见的null判空错误。同样，Rust也支持通过`value?`从Option.Some中快速取值，

### 6.2 The match Control Flow Construct

#### 基础模式匹配语法

与Swift枚举switch类似，Rust语法支持强大的控制流结构，基于模式匹配(pattern match)强大的表达力，Rust可以通过match灵活地控制不同的执行路径，常见模式有常量值、变量名、通配等。模式执行的时候，会把要match的结果值，和每个分支的模式进行对比，如果能够匹配上，就会执行对应的代码。通常来说，如果对应的分支代码比较短，可以简化写法，不会使用括号，与ES6的箭头函数很类似。对于复杂的语句，通过括号写多行代码，但同样支持把最后一行代码的值用于返回值。

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

#### 匹配枚举绑定的值

Rust分支匹配常用的特性，模式匹配可以支持绑定值，常用于提取枚举变体中的值。使用上面的例子进一步延伸：

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => { // 提取模式匹配绑定的值
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

#### 匹配可选值

`macth`语法与`Option<T>`语法结合，可以用于提取Option的内部值。

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}
```

#### 模式匹配的全面性

与Swift的模式匹配语法类似，Rust中模式匹配要求必须覆盖全部的可能性的case，如果有遗漏，Rust编译器就会报错。因此，在使用枚举时，可以针对一些指定值进行特殊处理，然后对其他值采取一个共同的默认行为。具体来说，可以使用一个变量或`catch_all pattern`符号`_`来匹配其余全部模式：

```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    _ => (),
}
```

### 6.3 Concise Control Flow With if let

与Swift语法类似，Rust支持`if let`语法糖，用于简化模式只匹配一种值的场景的代码。

```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximmum is configured to be {}", max);
}
```

使用`if let`的优势是可以简化代码，但存在的问题是会失去模式匹配处理的全面性，因此需要根据实际使用情形来决定用那种语法。换句话说，`if let`其实是只匹配一个值而忽略其它值的`match`语法的语法糖。同时，`if let`和`else`可以搭配使用，与`match`和`_`搭配效果一样。例如下面的例子：

```rust
let mut count = 0;
// match和_搭配使用
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}
// 等价于：if let和else搭配使用
if let Coin::Quarter(state) = coin {
    printlin!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

## 7. Managing Growing Projects with Packages, Crates, and Modules

Rust提供了一系列用于代码组织的功能，包含决定哪些细节被暴露、哪些实现细节是私有的、程序中每个作用域有哪些命名。这些特性有时被统称为`module system`，即模块系统，具体来说包括：

* Packages包：一个用于编译构建、测试并分享单元包的Cargo功能；
* Crates单元包：一个用于生成库或可执行文件的树形模块结构；
* 模块（module）以及use关键字：用于控制文件结构、各种作用域、路径私有性；
* Paths路径：一个用于命名条目的方法，包括结构体、函数、模块等；

### 7.1 Packages and Crates

* *Package*: `一个包(package)包含一个或多个单元包(crates)`。一个包package必须包含一个`Cargo.toml`文件，用于描述如何构建这些单元包；
* *Crate*: `单元包提供一系列功能functionality集合`，Crate可以是binary crate二进制程序或library crate库。二进制单元包是可执行性文件，有`main`函数作为执行的入口，例如命令行工具就是binary crate。库单元包主要用于在不同工程之间共享代码，我们常见的开源库就是library crate，库不能被直接编译为可执行文件，也没有`main`函数入口。

Rust编译器从一个源文件source file开始编译，这个文件被称为`crate root`单元包根节点，同时，也是我们单元包的`root module`根模块。包有一个或多个单元包组成，并且有以下几个原则作为限制：

* `一个包最多只能包含一个库单元包（library crate）`;
* `一个包可以包含任意个数的二进制单元包（binary crate）`；
* `一个包至少要包含一个单元包，可以是二进制单元包或库单元包`；

Cargo创建package时，会遵循以下几个惯例：

* `src/main.rs` 如果包文件目录下有`src/main.rs`文件，cargo就认为，pacakge包含一个与package同名的binary crate二进制单元包，并且main.rs是单元包根节点(crate root)。
* `src/lib.rs` 如果包文件目录下包含了`src/lib.rs`文件，cargo就认为，pacakge包含一个与package同名的library crate库单元包，并且`src/lib.rs`是单元包的根节点crate root。
* Cargo会把单元包根节点文件信息传递给`rustc`，用于编译构建库或二进制；
* 如果一个package同时包含`src/main.rs`文件和 `src/lib.rs`文件，就表明这个package同时包含了两个crates单元包，一个二进制一个库，并且都与package同名；
* package支持包含多个二进制单元包，惯例是，通过在`src/bin`目录下放多个文件即可，每个文件都是一个独立的二进制单元包；

### 7.2 Defining Modules to Control Scope and Privacy

#### 模块快速入门基础知识

* `从单元包根节点(crate root)开始`：当编译一个单元包时，编译器首先会查找单元包根节点文件，通常来说，对于库单元包就是src/lib.rs，对于二进制单元包就是src/main.rs；
* `模块声明(Declaring Modules)`：在单元包根节点文件中，你可以声明一个模块命名，假设名字是garden，`mod garden`，编译器会在以下的几个地方，查找模块的实现：
  * 以内联形式包含的代码，即 `mod graden` 声明后面跟着的代码块；
  * 在文件 `src/garden.rs`找到的代码;
  * 在文件 `src/garden/mod.rs`找到的代码；
* `子模块声明(Declaring submodules)`：单元包内除了单元包根节点以外的其他文件中，你可以声明子模块。编译器会遵循一些惯例寻找子模块得实现，例如 `mod vegetables`，首先只会在父模块parent module对应的目录下寻找子模块，其次查找的顺序如下：
  * 以内联形式包含的代码，即`mod vegetables`声明后面跟着的代码块；
  * 在文件 src/garden/vegetables.rs
  * 在文件 src/garden/vegetables/mod.rs
* `模块中代码查找路径(paths to code in modules)`：一旦模块被编译为单元包的一部分，我们就可以在这个单元包的其它地方，在可见性规则允许的情况下，直接通过路径引用这个模块中代码。假设我们在单元包的其他地方要使用`veggetable`模块中的`Asparaus`类型，我们可以使用路径`crate::garden::vegetables::Asparagus`进行引用；
* `公有与私有(Private vs public)`：模块中的代码默认情况下是私有的。使用`pub mod`声明替换`mod`声明可以把模块变成公开。同样，对于公开模块中的items，也需要使用`pub`才能变成公开可用的。
* `use关键字(The use keyword)`：在一个作用域内，使用use关键词创建一个对items的快捷访问方式，减少长路径的重复出现。例如，在任意作用域内，使用 `use crate::garden::vegetables::Asparagus;` 引入`Asparagus`以后，就可以直接使用`Asparagus`类型，而不用每次都是用长路径。

例如对于上面的案例，我们创建一个`backyard`的二进制单元包来阐述这些知识，首先，路径树如下：

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

上面的案例里面，由于是二进制单元包，单元包的根节点crate root文件是 `src/main.rs`，这个文件的内容如下：

```rust
use crate::garden::vegetables::Asparagus; // 使用use导入Asparagus类型

pub mod garden; // 声明一个模块garden

fn main() {
    let plant = Asparagus {}; // 可以使用Asparagus直接访问，不用前缀路径
    println!("I'm growing {:?}!", plant);
}
```

按照模块的路径查找规则，`pub mod garden`模块，首先会找内联的代码实现，但模块声明后面并没有跟着 `{}` 包括的代码实现。因此接下来，就会从`src/garden.rs`文件中寻找garden模块的代码实现；

#### 使用模块组织代码

Rust模块支持把一组功能关联性比较高的代码组合在一起，方便维护和复用。模块可以控制items的可见性，通过`public`修饰的item可以被外部访问，通过`private`修饰的item只是内部实现细节，外部不可用。使用`cargo new --lib xxx`命令创建一个库，然后`src/lib.rs`中定义一些模块和函数签名。

### 7.3 Paths for Referring to an item in the Module Tree

Rust在module tree中查找items的方式，与文件系统非常类似，也是通过paths路径，也同时支持绝对路径和相对路径查找，路径的分隔符是`::`

* 绝对路径：对于外部的单元包，从这个单元包根节点crate root开始；对于当前单元包，直接通过`crate`字面量作为路径查找即可；
* 相对路径：相对路径会使用 `self`、`super`或当前模块内的标志符，并从当前模块开始进行查找

选择使用相对路径还是绝对路径，需要根据项目的需要来进行选择，后续是否会需要把调用者items和被调用者item一起管理或分开管理。

Rust通过模块Modules管理items可见性边界或者说实现访问控制，并且规则对所有类型的items都有效（functions/methods/structs/enums/modules/constants），并且全部默认是私有，对外不可见的。Rust的访问控制与其他语言类似，都是为了达到实现细节封装的目的。同时，由于Rust支持子模块，对于父模块和子模块的可见性，也有一些特殊的规则，*父模块不能使用子模块的私有items，只能访问public items，但子模块的items可以使用外部祖先模块（ancestor modules）的任意items*，这是因为，子模块用于封装和隐藏实现细节，对父模块需要不可见，但是对子模块而言，需要感知定义他们的父模块的环境context

使用`pub`关键词控制模块和模块内容的访问权限，只有都设置为`pub`，其它模块才可以访问。对外提供的公共库，可以遵循Rust的设计规范，[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/about.html)。

Rust支持使用`super`关键词引用父模块，然后构造路径访问对应的items，如下面的代码：

```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        // 访问父模块路径
        super::deliver_order();
    }

    fn cook_order() {}
}
```

使用`pub`关键词可以指定 `struct`结构体和`enums`枚举变为公开可访问。使用`pub`修饰struct可以让结构体变为外部可访问，但结构体的fields字段仍然是私有的，需要显式设置pub才能把这些field变成公开。但与结构体相反，使用pub修饰枚举为公开访问的时候，枚举所有的变体也都是公开的。

### 7.4 Bringing Paths into Scope with use keyword

*语法一*：Rust提供了`use`简化路径的导入语法，一旦使用use导入以后，在对应的作用域内，后续只需要使用shortcuts就能快速访问对应的items。 例如下面的例子，我们使用use导入host模块，后续就可以hosting进行访问即可，例如`hosting::add_to_waitlist`函数。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

需要注意的是，`use`创建的快捷方式只在特定的作用域内才有效，例如下面的例子就是一个无效的例子：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

mod customer {
    pub fn eat_at_restaurant() {
        // 编译错误，use导入的hosting模块只有在上一级的作用域内才有效
        hosting::add_to_waitlist();
    }
}
```

*语法二*：Rust中路径Paths的惯用用法。

* 如果需要导入函数，惯例用法是导入函数父模块到作用域内，然后调用函数的时候，指定父模块parent module即可，例如 导入函数的父模块`use crate::front_of_house::hosting;`，使用函数的时候指明父模块，`hosting::add_to_waitlist();`。
* 如果需要导入结构体、枚举或其他items，惯例用法是指定全路径。例如使用基础库的HashMap结构体时，导入的语法是 `use std::collections::HashMap;`，使用的语法是 `let mut map = HashMap::new();`；
* 用法惯例的例外是，如果导入两个同名的items到同一个作用域，这个时候可以通过导入父模块parent module的模式来导入。
* 对于两个同名的items导入情形，可以使用 `as` 关键词来指定一个新的局部名字或别名，例如 `use std::io::Result as IoResult;`。

*语法三*：Rust支持通过使用`pub use`关键词来重新(re-exporting)导出名字。正常情况下，我们使用use关键字将名字导入作用域时，这个名字会以私有的方式在新作用于内生效。但通过 `pub use` 关键词就可以让外部代码访问这些名字。

*语法四*：在`Cargo.toml`中导入这些外部模块，然后就可以通过`use`导入使用这些外部模块。需要注意的是，对于`std`标准库，我们不需要再`Cargo.toml`导入，但使用的时候仍然需要使用`use`关键字导入。

*语法五*：使用嵌套路径来优化多个模块的导入语法，与ES6的import优化相似，例如`use std::{cmp::Ordering, io};`

*语法六*：Rust支持使用通配符导入模块内全部items，常用于`tests`测试场景下使用，例如 `use std::colletctions::*;`

### 7.5 Separating Modules into Different Files

当模块逐渐变大，可以把定义拆分到不同的文件中，让代码更容易查找。在模块声明后面以分号结束，而不是代码块。根据模块声明的基本原则，Rust会查找与文件同名的文件中加载模块内容。例如：

```rust
// src/main.rs
mod front_of_house;

pub use crate::front_of_house::hosting; // 在front_of_house.rs或front_of_house/mod.rs目录查找对应的模块定义

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

```rust
// src/front_of_house.rs
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

### 总结

Rust允许将package拆分为多个crates单元包，并将单元包拆分为不同的模块，从而能使我们在其他模块中引用某个特定模块内定义的items。

## 8. Common Collections

Rust基础库包含了一组非常有用的数据结构，统称为集合Collections。与数组和元组分配在栈上空间不同，集合指向存储在堆上的数据，这些数据的数量在编译时间不会明确，而是在运行时动态增长或减少的。每一种类型的集合，都有不同的能力和成本，需要选择合适的数据结构：

* `vector`：被称之为向量的动态数组，可以连续存储任意多个值；
* `string`：字符的集合。
* `hash map`：将值关联到一个特定键上，是map映射的特殊实现；

### 8.1 Stroing Lists of Values with Vectors

Vectors动态数组允许在单个数据结构中存储多个相同类型的值，需要注意的是，*只支持存储同类型的值*。Vectors支持泛型 `Vec<T>`。惯用的创建方法：一、使用`Vec::new()`函数进行创建一个空的动态数据，这种类型需要在变量类型的声明中指明对应的泛型的类型；二、直接使用 `vec!`宏语法糖来创建，这个可以自动推导出类型，并且快速初始化vector；

```rust
// 创建一个空动态数组
let v: Vec<i32> = Vec::new();
// 通过宏创建动态数据
let v = vec![1, 2, 3];
```

*语法*：往vector中添加一个数据，`push`;

*语法*： 当离开作用域时，vector就会被释放，vector所包含的内容也会被释放和清理；

*语法*： 读取vector中的数据，可以通过索引`[]`或`get`方法。这两种访问的差异是，通过`[]`访问不存在的元素会导致panic；通过`get`访问不存在的元素，不会出现panic，并返回`None`。读取vector的值时，需要遵守所有权规则，例如下面的代码中会编译错误，因为会同时存在可变借用和不可变借用：

```rust
let mut v = vec![1, 2, 3, 4, 5];
let first = &v[0]; // 不可变借用
v.push(6); // 编译错误，因为这里会有可变借用
println!("The first element is: {}", first); // 不可变借用
```

*语法*：除了常规的for循环 + 索引方式访问vector，vector还支持 `for...in` 遍历访问，甚至支持通过 `&mut` 访问可变引用

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

*语法*：Vectors有一个比较重要的约束，只能存储同类型的数据，但在有些场景下，也存在一些不便利，比如需要存在不同类型的数据列表。Rust中可以通过枚举变体定义和match来优雅解决这个问题：

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

### 8.2 Storing UTF-8 Encoded Text With Strings

对于Rust新手来说，String是比较难理解的概念，主要因为三个阻塞点：Rust对潜在错误的易暴露特性，strings是一个超乎大多数开发者想象的的复杂数据结构，UTF-8编码处理。由于字符串本身就是bytes类型的collectsion集合，并通过一些功能性方法将bytes解析为文本。

#### What is a String ?

Rust语言的核心层只提供了一个字符串类型，即字符串切片 `str`，常见的使用形式借用形式 `&str`。`string slices`字符串切片和`string literals`字符串字面量都是引用类型。同时，Rust在std基础库提供了`String`类型，是可增长、可变、有所有权、UTF-8编码的字符串类型。因此，Rust开发者提到字符串概念时，指的可能是 `String`类型或`&str`类型，在Rust std基础库中这两个类型都被大量使用。

Rust std标准库有一系列的字符串类型，提供了更多的可选项，例如 `OsString`、`OsStr`、`CString`、`CStr`。

#### Creating a New String

* `Vec<T>`中很多操作都同样适用于`String`类型
* `String::new()`创建字符串；
* `to_string()`方法，从字面量创建`String`类型；
* `String::from()`方法，从字面量创建`String`类型，与`to_string()`能力是一样的。
* strings是UTF-8编码的，所以可以存储任意正常编码的数据；

#### Updating a String

* 使用`String`的`push_str`方法用于append字符串切片；
* 使用`String`的`push`方法用于append一个单字符；
* 使用 `+` 操作符或 `format!` 宏来连接 `String`类型的值。例如下面的例子

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```

上面的表达式需要特别注意，s1的所有权被转移了，s2是使用引用，这个实际的操作是取得s1的所有权，然后把s2的内容复制到s1中，这样设计更加高效。对于多个字符串相连接的操作，使用`+`操作符过于繁琐，可以使用 `format!` 更加高效。

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");
let s = format!("{}-{}-{}", s1, s2, s3);
```

#### Indexing into Strings

在许多编程语言中，通过索引访问单独字符是一个有效和常规的操作。但是，在Rust中，如果直接通过索引访问 `String`的部分内容，编译器就会报错。也就是说，*Rust字符串不支持索引访问*。

`String`的内部原理是基于`Vec<u8>`实现的，即经过编码的字符串都存储在一个u8 vector向量集合中。例如 `let hello = String::from("Здравствуйте");`，在Rust中的长度是24，这是由于每个Unicode标量值都占用2个字节的存储空间，而Rust返回的长度是占用的字节bytes空间，因此，如果直接访问`&hello[0]`会导致返回值时非法的，会导致运行时错误，这就是为什么Rust字符串不支持索引的原因。

在Rust的视角来看字符串的组成有三个角度：bytes字节，scalar values标量值，grapheme clusters字形簇（类似单词）。以印度语的 "नमस्ते" 为例，

* 按照u8字节视角，共有18个字节，
* 按照char字符视角，共有6个字符， "['न', 'म', 'स', '्', 'त', 'े']"
* 按照字形簇视角，共有4个印度语单词，"["न", "म", "स्", "ते"]"

Rust提供了多种解释电脑存储的原始字符串的方式，程序可以按照需要来选择合理的方式解释。从根本上来说，Rust不允许通过索引访问字符串的另一个原因是，Rust中的String并不能保证索引访问的速度。

#### Slicing Strings

Rust不支持通过索引访问单个字符，但支持通过范围语法来指定所需的字节内容，例如 `&hello[0..4]`。切记要小心谨慎使用范围语法来创建字符串切片，避免非法的索引使用会导致程序崩溃panic。

#### Methods for Iterating Over Strings

Rust中操作字符串片段的最好的方式是，首先明确你需要的是字符串characters还是字节bytes。如果要访问单个Unicode标量值，使用 `chars`方法来遍历。如果要访问原始字节，使用`bytes`方法来遍历；而对于字形簇，Rust标准库没有提供功能，但是可以在Crates上找到合适的功能，例如[unicode-segmentation](https://crates.io/crates/unicode-segmentation)

```rust
// 字符遍历
for c in "नमस्ते".chars() {
    println!("{}", c);
}
// 字节遍历
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

#### Strings Are Not So Simple

总之，字符串是非常复杂的，不同语言采取不同的选择，对开发者暴露不同的复杂度。Rust选择对开发者暴露更多的复杂性，因此，Rust语言选择把对所有程序的字符串数据处理都会保持正确的方案，作为Rust语言的默认行为，这意味着不会针对特定场景有特殊处理，需要Rust开发者提前理解UTF-8数据处理方式。这个trade-off暴露了更多复杂性，但也会阻止开发期的字符串处理错误。

### 8.3 Storing Keys with Associated Values in Hash Maps

最后一个要介绍的常见集合类型是 hash map，Rust中泛型定义是 `HashMap<K, V>`，很多编程语言也都有类似的数据结构，类似hash/map/object/hash table/dictionary/associative array。Map数据结构，在使用任意类型值作为索引key时，非常有用。

#### Creating a New Hash Map

使用`HashMap::new()`创建空的hash map，使用`insert`新增数据项。同时，由于HashMap使用频次较低，因此并没有包含到Rust的预引入范围内，需要通过`use`显式引入，Rust也没有内置构建宏支持。

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

hash map在堆上存储数据，同时，所有key都必须是同一个类型，所有的value也必须是同一个类型。

另外一种创建HashMap的方式，基于一个元组vectors使用迭代器和collect方法创建。例如下面的例子：

```rust
use std::collections::HashMap;

let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let mut scores: HashMap<_, _> =
    teams.into_iter().zip(initial_scores.into_iter()).collect();

```

#### Hash Maps and Ownership

* 如果插入的类型实现了Copy trait，例如i32，这些值会被复制到hash map中。
* 对于String这样的owned value类型，值会被转移，然后hash map将会成为这些值的新的所有者。
* 如果插入引用类型到hash map中，对应的值不会被转移到hash map中。这些值的引用在hash map存在期间，都必须有效。

#### Accessing Values in a Hash Map

* 使用`get`方法访问hash map，返回的是 `Option<&V>`类型，如果相应key对应的值不存在，`get`方法返回的是`None`值。
* 也可以结合for循环和迭代器的方式，访问hash map中的每一个键值对：

```rust
for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

#### Updating a Hash Map

覆盖hash map中的值，使用`insert`方法插入同样key就可以更新已有的值。

只有hash map中的key对应的值不存在的情况下，才插入值，可以使用 entry和or_insert方法结合，例如 `scores.entry(String::from("Yellow")).or_insert(50);`。

基于hash map中已有的值更新值，`or_insert`会返回一个变量的引用，然后通过解引用操作符 `*` 来更改已有的值。例如：

```rust
let count = map.entry(word).or_insert(0);
*count += 1;
```

#### Hashing Functions

默认情况下，HashMap使用一个叫做 `SipHash`的哈希函数，可以提供拒绝服务攻击能力（DDOS）。

## 9. Error Handling

Rust总体上把错误分为可恢复错误和不可恢复错误两类。大多数语言并不会区分这两种错误，而是使用同一种机制，例如异常，来处理这两种错误。而在Rust会采用两种不同的方案处理这两种错误：

* 可恢复错误：使用 `Result<T, E>` 处理可恢复错误；
* 不可恢复错误：使用 `panic!`宏来终止程序执行；

### 9.1 Unrecvoerable Errors with Panic

对于不可恢复的错误的退出处理，Rust支持unwinding和abort两种方式，unwinding方式会沿着调用方法调用栈，然后清理数据，但是会耗时比较久。而abort方式，会直接退出，然后把清理的职责交给操作系统来完成。如果要在release模式下，在遇到panic的时候直接abort，可以在`Cargo.toml`文件中添加以下的配置：

```toml
[profile.release]
panic = 'abort'
```

对于不可恢复的错误，Rust提供了 `panic!`宏来直接退出。panic! 在退出的时候，会打印堆栈和文件等一系列信息，帮助排查错误；

对于系统库导致的`panic!`错误，例如数据访问越界，可以使用 `RUST_BACKTRACE` 来打印错误的backtrace信息，例如 `RUST_BACKTRACE=1 cargo run`。如果想要打印backtrace栈回溯信息，就必须打开 `dubug symbols`来获取调试符号，默认情况下，如果没有添加 `--release` 选项，都会有符号；

### 9.2 Recoverable Errors with Result

对于可恢复错误，Rust提供了 `Reust<T, E>` 枚举类型支持，在Rust基础库中，Result通用的错误处理方案。

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```

#### Matching on Different Errors

在类Java编程语言中，基于不同的异常类来catch不同的error。Rust则是基于match来匹配不同的error类型，例如下面的例子，[file open](https://doc.rust-lang.org/std/fs/struct.OpenOptions.html#method.open)，error共有6个类型：NotFound、PermissionDenied、AlreadyExists、InvalidInput

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error)
            }
        },
    };
}
```

`match`语法使用起来会比较繁琐，可以使用 `unwrap_or_else` 语法简化使用：

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

#### Shortcuts for Panic on Error: unwrap and expect

Rust提供了错误处理的语法糖来简化match语法：unwrap和expect。

* `unwrap`语法*，如果发生错误直接调用`panic!`，结果正常就返回Ok值里面的值。
* `expect`语法*，与`unwrap`语法类似，但expect允许在发生错误时提供良好的错误信息，方便排查问题。

下面前面两种语法是等价的，第三种语法可以自定义错误信息：

```rust
let f = match f {
    Ok(file) => file,
    Err(error) => panic!("Problem opening the file: {:?}", error),
};
// 等价于
let f = File::open("hello.txt").unwrap();
// 允许自定义错误信息
let f = File::open("hello.txt").expect("Failed to open hello.txt");
```

#### Propagating Errors

在类Java语言中，可以通过抛异常来进行错误传递，即把底层调用的异常信息通过throw传递给上层进行处理，这种行为可以称之为 propagating errors传播错误。Rust支持`?`操作符，用于简化错误传播操作，使用`?`操作符替代Result的`match`语法处理错误。

```rust
// 写法一：基于match进行Rust错误传播
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e), // 返回io::Error类型
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e), // 返回io::Error类型
    }
}
```

```rust
// 写法二：基于?操作符进行错误传播
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?; // 自动转换错误类型为io::Error
    Ok(s)
}
```

`match`表达式与`?`操作符的区别是，`?`操作符会自动转换返回的错误值(error values)的类型，以便符合函数返回值的类型需要。`?`通过调用标准库`From` trait中定义的`from`函数，`from`函数会负责错误从一种类型转换为需要的类型。因此，当使用`?`操作符时，操作符会自动把函数调用返回错误的类型，转换为函数需要的错误类型，我们就不需要再处理不同错误之间的类型转换了，这在日常开发工作非常有用。一般来说，一个函数调用返回的错误类型只有一种，但在函数执行过程中可能可能会有多个原因导致错误发生，例如上面例子中，`read_user_from_file`函数只会返回io::Error错误哦，但调用`File::open`可能会发生错误，调用`read_to_string`也可能会发生错误，但通过`?`操作符支持自动错误类型转换，因此，`?`操作符可以大幅降低此类的错误类型转换工作。在Rust中，只错误类型实现了 `impl From<OtherError> for ReturnedError` 的trait中的`from`函数，就可以通过`?`函数自动转换错误类型。

进一步而言，Rust还是支持`?`操作符的链式调用，与Swift的optional chain语法类似。上面的代码可以使用下面的语法进行进一步简化：

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

#### Where The ? Operator Can Be Used

从前面的章节例子中，我们知道`?`操作符的行为与`match`表达式写法一致，会返回Result类型的返回值，即`read_username_from_file`函数的返回值与`read_to_string`返回的类型一样。因此，*`?` 操作符只能用于返回值与内部使用`?`操作的值的数据类型一样的函数*，The ? operator can only be used in functions whose return type is compatible with the value the ? is used on。例如下面的函数会报错，由于main函数的返回值是`()`类型，但是?操作的值是Result类型。解决这个问题方法有两种，一种是修改函数的返回值类型，与`?`操作符作用的值的类型一致。另一种是使用match表达式处理错误。

```rust
use std::fs::File;
fn main() {
    let f = File::open("hello.txt")?;
}
```

与Swift中的optional值类型相似，Rust基于`?`操作符也支持了 `Option<T>` 类型，可变值支持两种:`Some`和`None`。但需要注意的是，虽然都是基于`?`操作符，但`Option`和`Result`是两种类型，二者并不能混用，也就是说，如果函数返回值是Result类型，可以使用`?`处理Result。如果函数返回值是Option类型，可以使用`?`处理Option，但是不能混用，`?`操作符不会自动把Result转换为Option，反之亦然。

### 9.3 To panic! or Not to panic

#### Examples, Prototype Code, and Tests

* Examples：代码示例中，使用`unwrap`来处理错误，可以让代码更加清晰；
* Prototype Code: 开发原型时，使用`unwrap`和`handy`更加方便，同时，后续可以快速把类似的错误更改为更为健壮的错误处理；
* Tests: 测试代码中，使用 `panic!`、`unwrap`、`expect`是比较合理的方案；

#### Cases in Which You Have More Information Than the Compiler

对于一些实际逻辑上不可能会失败，但是编译器无法识别的场景，可以使用 `unwrap` 来绕过编译器的警告。例如下面的代码，由于`127.0.0.1`是常量，所以parse方法只会成功不会失败，使用unwrap只是为了绕过去编译告警

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1".parse().unwrap();
```

#### Guidelines for Error Handling

由于panic错误是不可恢复的，正常情况下，应该使用Result进行更稳健的错误处理。只有在一些程序运行可能出现比较极端的bad state时，才使用panic。这里的bad state指的是，当一些假设、保证、协议、不可变性发生破坏时，例如非法值、自相矛盾的值、值丢失等清醒出现在代码中，并且符合下列某个条件：

* bad state是不符合预期的，例如用户输入错误格式的数据；
* 在bad state发生以后，代码无法正常运行；
* 没有合适方式将bad state信息编码到我们使用的类型中；

#### Creating Custom Types for Validation

Rust类型系统提供了大量的类型校验工具，在错误处理过程中可以大幅简化代码。同时，我们针对自定义数据类型实现校验方法，避免校验逻辑分散在不同的调用函数中。

## 10. Generic Types, Traits and Lifetimes

第一部分内容讲解，如何通过泛型来提取函数，以减少代码重复问题；第二部分，讲解如何把traits和泛型结合；第三部分，讨论lifetime生命周期，这类泛型可以给编译器提供引用之间的关系，允许我们在借用值时通过编译器来确保这些引用的有效性。

### 10.1 Generic Data Types

在类似函数签名或结构体定义里面使用泛型，然后就可以对其他数据类型复用这些定义；

#### In Function Definitions

利用泛型定义函数时，在函数签名中需要指明数据类型的地方使用泛型。例如，定义 `fn largest<T>(list: &[T]) -> T`

#### In Struct Definitions

在定义结构体的时候，对于一个或多个字段使用枚举类型参数来定义

```rust
struct Point<T, U> {
    x: T,
    y: U,
}
```

#### In Enum Definitions

对在枚举的变体，可以使用泛型来定义类型

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

#### In Method Definitions

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}
impl<X1, Y1> Point<X1, Y1> {
    // ...
} 
```

注意，方法定义时，`impl`关键字必须紧跟着泛型声明，以便能够在实现方法时指定类型，例如 `Point<X1, Y1>`。通过impl之后声明泛型类型，Rust可以识别尖括号内的类型是泛型而不是具体类型。

#### Performance of Code Using Generics

Rust泛型不会带来任何性能负担！Rust在编译时执行代码的单态化（monomorphization）。单态化是一个在编译期将泛型代码转换为特定代码的过程，它会将所有使用过的具体类型填入泛型参数从而得到具体类型的代码。

### 10.2 Traits: Defining Shared Behavior

trait特征用于定义在特定类型之间共享的功能，相对于类而言，复用颗粒度更小。借用`<Modern PHP>`中的形象描述，一个Bird对象可以飞，一个飞机对象也可以飞，在传统的OOP种，飞的代码在这两种对象之间无法复用，但是可以通过trait特征实现复用，例如可以定义一个用于描述飞能力的traits特征，在飞机和bird之间复用。因此，traits与其他语言中的interface类似，但又有不同，traits是可以包含实现的，interface只能有定义抽象。Rust特征语法与Swift的POP面向协议编程类似。

#### Defining a Trait

Trait特征，将一组方法签名组合在一起，来定义完成某种目标必要的行为集合。

```rust
pub trait Summay {
    fn summarize(&self) -> String;
}
```

在特定的数据类型上，可以针对这个类型实现trait特征，语法上与常规的方法实现类似。语法规则是 `impl (trait name) for (struct name)`。

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
```

*需要注意的是*，我们不能为外部类型定义外部traits特征，只能为当前crate内部的类型增加traits实现。例如，我们不能再`aggregator`单元包内为`VecT>`定义Display特征，因为Display和`Vec<T>`都定义在标准库中。这个限制的来源是`coherence`一致性规则，确保了其他人的代码不会破坏你的的代码。*如果没有这个规则，两个库可以分别对相同的类型定义相同的trait，Rust将无法决策使用哪一个版本*。

#### Default Implementations

在一些场景下，为traits特征中的某些方法或所有方法提供默认行为是非常有用的，使得我们无须为每一个类型的实现都提供自定义行为。这个提供了特征级别的代码复用能力，让复用的颗粒度更细。然后，当在某个特定类型实现trait特征时，我们可以保持或者重写这些方法的默认行为。

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

#### Traits as Parameters

*语法：impl trait*，可以使用traits代表特定类型，与Java中的interface用法类似，达到依赖抽象而不是依赖实现的目的，对于编码有重要价值和意义；

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

*语法：trait bound*，`impl rait`实际上是`trait bound`特征约束语法的语法糖，例如上面的例子等价于下面的trait bounds语法

```rust
// trait bounds
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

对于`impl Trait`和`trait bound`这两种语法，`impl Trait`适用于短小的实例，`trait bound`适用于复杂情形。例如下面的情形，方法有多个参数，并且强制参数是同一种类型，就必须的使用trait bounds语法：

```rust
pub fn notify<T: Summary>(item1: T, item2: T)
```

*语法：使用+指定多个trait bounds*，对应的impl trait和trait bound写法如下：

```rust
// impl trait
pub fn notify(item: &(impl Summary + Display))
// trait bounds
pub fn notify<T: Summary + Display>(item: &T)
```

*语法：使用where从句来简化trait约束*，与C++的构造函数的参数初始化语法类似

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
    // do something
}
// 等价于where语法
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
    // do something
}
```

#### Returning Types that Implement Traits

对于返回值，也可以应用 `impl trait`语法，来返回一个实现了trait的特定类型的值。例如语法：`fn returns_summarizable() -> impl Summary`

#### Using Trait Bounds to Conditionally Implement Methods

*语法：通过在带有泛型参数的impl代码块中使用trait约束，我们可以单独为实现了指定trait的类型编写方法*，例如下面的cmd_display方法，只针对实现了 Display和PartialOrd 特征的类型有效：

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

*语法：我们可以为实现了某个trait的类型有条件实现另一个trait*。对满足所有trait约束的所有类型实现trait，这种语法被称作覆盖实现（blanket implementation），这种机制被广泛适用于Rust基础库中。

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

通过上面的语法，就可以为基础库中所有实现Display trait的类型添加了一个新的trait实现，相当于扩展了Display trait。

### 10.3 Validating References with Lifetimes

Rust Lifetime生命周期是Rust独有的语法，有一定的理解成本。*Rust生命周期lifetime也是一种泛型，是编译器用来确保所有借用borrows都有效的结构体*。Rust中引用都有一个lifetime生命周期，在这个作用域内引用是有效的。大多数时候，声明周期是隐式的可以被编译器推导出来的。我们只有在生命周期有多种可能性的时候，才需要特别注解生命周期。Rust需要依赖我们通过泛型生命周期参数注解的关系信息，才能在运行时保证真正的引用关系是有效的。

在[Rust By Example](https://doc.rust-lang.org/rust-by-example/scope/lifetime.html)文档中，有一段关于lifetime的解释很有意思。有几个关键点信息：lifetime是用来编译器用来检查borrows有效性的结构体（borrow checker）；变量的生命周期，开始时间点是变量创建时，结束时间点是变量被销毁时；borror的liftime生命周期和scopes作用域经常被一起使用，但是他们并不相等，例如，borrow的生命周期通常通过在哪里被声明来决定的，因此，borrow借用在lender放款人被销毁之前都是有效的。与此同时，borrow的作用域则是通过引用在哪里被使用来决定。

```text
A lifetime is a construct the compiler (or more specifically, its borrow checker) uses to ensure all borrows are valid. Specifically, a variable's lifetime begins when it is created and ends when it is destroyed. While lifetimes and scopes are often referred to together, they are not the same.

Take, for example, the case where we borrow a variable via &. The borrow has a lifetime that is determined by where it is declared. As a result, the borrow is valid as long as it ends before the lender is destroyed. However, the scope of the borrow is determined by where the reference is used.
```

#### Preventing Dangling References with Lifetimes & Borrow Checker

`lifetime生命周期语法的主要目的是阻止悬垂引用dangling references`，悬垂引用会导致一个引用指向错误的数据。为了解决这个问题，`Rust编译器提供了一个称之为borrow checker的机制`，用于比较不同的作用域，来判断借用是否是有效的。例如下面的demo会报错，因为变量x的生命周期`'b`，会短于变量r的生命周期`'a`。在编译期间，Rust编译器会对比这两个生命周期的size长度，就会发现变量r的生命周期`'a`会指向一个生命周期为`'b`的变量，但由于`'a`大于`'b`，Rust编译器判断会出现悬垂引用，因此会编译时报错。

```rust
    {
        let r;                // ---------+-- 'a
                              //          |
        {                     //          |
            let x = 5;        // -+-- 'b  |
            r = &x;           //  |       |
        }                     // -+       |
                              //          |
        println!("r: {}", r); //          |
    }                         // ---------+
```

我们可以进一步修改上面的例子，调整生命周期`'a`小于生命周期`'b`。

```rust
    {
        let x = 5;            // ----------+-- 'b
                              //           |
        let r = &x;           // --+-- 'a  |
                              //   |       |
        println!("r: {}", r); //   |       |
                              // --+       |
    }                         // ----------+
```

#### Generic Lifetimes in Functions

如前面所述，一般情况下，生命周期是隐式的，不需要我们去注解，Rust编译器就可以推导出来。但是有一些情况下，生命周期可能有多种可能性，这个时候就需要我们显示标注生命周期信息。

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

上面的例子编译器会报错，`因为borrow checker不能决定判断返回值的生命周期是哪一个`。如果返回x值，返回值的生命周期与x一致，如果返回y值，返回值的生命周期与y一致。因此，我们需要添加一些生命周期参数来定义引用之间的生命周期关系，borrow checker才能进行分析。

#### Lifetime Annotation Syntax

Rust支持`Lifetime Annotation`生命周期注解语法，用于描述多个引用的生命周期之间的相互关系。需要特别注意的是，`生命周期注解不会改变引用的存活时长，只是描述关系`。就像函数的参数的类型一样，Rust函数也支持生命周期参数用于标注变量的生命周期。生命周期的语法规则是，以单引号（`'`）开头，名字是小写并且非常短，与泛型的命名类似（除了泛型大写）。一般情况下，我们会使用 `'a` 来标注第一个生命周期。把生命周期注解放在`&`引用后面，然后通过一个空格分隔引用的类型。例如：`&'a i32`、`&'a mut i32`。

需要注意的是，*单独的生命周期注解并没有意义，因为注解是用来向编译器描述多个引用的泛型生命周期参数是如何相互影响的*。因此，接下来我们用生命周期注解来修改上一小节编译失败的问题，告诉borrow checker，返回值的生命周期与变量x和变量y的生命周期一样。

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

需要注意的是，函数签名中的生命周期注解不会改变任何值的生命周期，无论是传入的参数还是返回值。反之，我们告诉了borrow checker应该拒绝那些不符合这些约束的值。当注解函数的生命周期时，注解只存在于函数签名中，而不会出现在函数体中。生命周期注解与变量类型信息一样，成为函数接口协议的一部分。通过函数中的生命周期协议，Rust编译器做的分析工作可以更加简单，可以减少编译器的推导和检查工作。

#### Thinking in Terms of Lifetimes

如何使用生命周期参数取决于函数在做什么。*从根本上来说，生命周期语法是用来关于函数的不同参数、返回值等变量之间的声明周期关联的，一旦有足够的关联信息，Rust就有足够信息来允许内存安全的操作，并且不允许那些可能产生悬垂指针或者其他内存安全的操作。Ultimately, lifetime syntax is about connecting the lifetimes of various parameters and return values of functions. Once they’re connected, Rust has enough information to allow memory-safe operations and disallow operations that would create dangling pointers or otherwise violate memory safety.*

#### Lifetime Annotations in Struct Definitions

如果结构体包含引用，我们需要对结构体的每一个引用都添加生命周期注解。例如下面例子：

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

上面这个例子中，结构体只有一个字段 `part`，用于持有字符串slice引用，我们同样定义了一个生命周期引用`'a`，用在结构体类型声明以后和结构体主体内。这个注解表示，`ImportantExcerpt`实例不能在它的字段`part`引用的生命周期之外存活，即结构体实例的生命周期必须短于字段引用的生命周期，这是由于结构体实例的存在依赖于字段所指向引用的有效性。

#### Lifetime Elision

我们知道，每个引用都有一个生命周期，并且需要为使用引用的函数或结构体指定生命周期参数。在早期的Rust版本中，需要在所有的场景中都显示写名生命周期参数，但经过一段时间，Rust团队意识到Rust开发者需要在特定的情形下反复写生命周期参数，但其实这些情形是可预测以及有固定规律可言的。针对这些情形，Rust提供了 `lifetime elision rules`，即生命周期省略原则，Rust开发者不需要显示写生命周期参数，反之，Rust编译器会考虑这些情形。

需要注意的是，*Rust省略规则不包含完整的推理逻辑*，如果Rust编译器应用了elision rules以后，生命周期规则上仍然存在歧义，就会报错，把决策权转交给开发者来解决。函数或方法的参数的生命周期称之为输入生命周期`input lifetimes`，返回值的生命周期称之为输出生命周期`output lifetimes`。当我们没有显示写出生命周期注解时，Rust编译器使用三条规则来检查引用的申明周期，如果通过这三条规则检查仍然无法确定生命周期，编译器就会停止然后报错。这三条规则会作用于 `fn`函数定义和`impl`块

* 规则一：编译器给每一个引用类型的函数或方法参数都赋值一个对应的生命周期参数，换句话说，有多少个参数就有多少个生命周期参数，例如 `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`
* 规则二：如果只有一个输入生命周期参数input lifetime parameter（`'a`），这个生命周期参数(`'a`)会被赋值给所有的输出生命周期参数，例如 `fn foo<'a>(x: &'a i32) -> &'a i32`
* 规则三：如果有多个输入生命周期参数，并且其中一个是 `&self` 或 `&mut self`，因为这是个方法，所以`self`的生命周期会被赋值给所有的输出生命周期参数。这个规则使得方法很容易被读写，因为只需要少数的符号。

```rust
// 原始函数
fn first_word(s: &str) -> &str {}
// 规则一
fn first_word<'a>(s: &'a str) -> &str {}
// 规则二
fn first_word<'a>(s: &'a str) -> &'a str {}
```

#### Lifetime Annotations in Method Definitions

结构体field的生命周期字段必须在`impl`块中声明，与泛型类似，在impl关键字以后声明，然后必须在结构体名字以后使用，因为这些生命周期参数也是结构体类型的一部分。在`impl`块内的方法签名中，引用必须与结构体fileds的生命周期绑定，或者它们之间是相互独立的。除此以外，由于lifetime elision规则，大多数情况下生命周期注解都是没有必要明确写出来的。

#### The Static Lifetime

static lifetime是一种特殊的生命周期，语法是`'static`，表明受到影响的引用的存活周期是程序的全生命周期。例如常量文本存储在可执行文件中，因此生命周期可以是static的：

```rust
let s: &'static str = "I have a static lifetime.";
```

静态生命周期的使用需要慎重，大多数情况下，不是真的需要使用生命周期修饰，而是悬垂引用导致的内存泄露，因此在使用之前需要谨慎甄别。

## 11. Writing Automated Tests

Dijkstra: `Program testing can be a very effecitve way to show the presence of bugs, but it is hopelessly inadequate for showing their absence`

### 11.1 How to Write Tests

test函数通常来说包含三个动作：

* Set up any needed data or state，初始化需要的数据或状态
* Run the code you want to test，运行你想要测试的代码
* Assert the results are what you expect，对期望的结果设置断言

Rust中的测试需要通过 `test` 属性进行标注，属性Rust代码片段的元数据信息。创建测试函数时，需要在 `fn` 声明的上一行，添加 `#[test]` 注解，当执行 `cargo test` 命令时，Rust会构建一个包含注解函数的二进制测试文件，执行完毕以后报告每个测试函数是否通过。

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 3;
        assert_eq!(result, 4);
    }
}
```

* `cargo new xxx --lib`，当使用cargo命令创建一个库工程时，一个包含test函数的test模块会被自动创建。
* `cargo test`，使用这个命令来运行工程中的所有的test函数，即那些被 `#[test]` 注解的函数。
* 对于测试需要fail的case，可以使用 `panic!`宏来让测试失败并输出信息。
* `assert!`宏是Rust标准库提供的断言，用于断言表达式结果为true的宏。如果表达式结果为true，test可以通过；如果表达式结果为false，assert宏调用panic宏来让测试fail。
* `assert_eq!`宏用于断言两个值是相等的宏，`assert_ne!`宏用于断言两个值是不相等的宏。这两个宏的底层是通过 `==`和`!=` 来实现的，如果断言失败，就用debug输出格式打印参数信息，这意味着对比的两个值需要实现 `PartialEq`和`Debug` traits特征。针对自定义的结构体和枚举，通常可以通过 `#[derive(PartialEq, Debug)]`注解来实现。
* Rust提供了`should_panic`属性来校验代码中的错误处理是否符合我们的预期，

```rust
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

### 11.2 Controlling How Tests Are Run

默认情况下，`cargo test`命令会并发执行工程中的所有测试，并且把测试运行期间的输出全部捕捉。但同时，Rust也为这个命令提供了很多可选项，可以通过 `cargo test --help`来查询全部的选项值。

* `--test-thread`标志来控制测试二进制执行的线程数据，如果需要串行执行，可以设置为`--test-thread=1`
* `--show-output`标志，默认情况下如果测试执行成功，控制台打印信息不会输出。但是添加这个flag，即使tests都执行成功的情况下，仍然会输出标准控制台的各种信息。
* `cargo test full_test_name`，指定test的完整名字的情况下，只会运行特定的一个test；
* `cargo test test_name_prefix`，指定test的名字前缀的情况下，会运行满足条件的多个test；
* 通过在test函数添加 `#[ignore]`属性和`--ignored`标志，可以控制不运行那些test函数；

### 11.3 Test Organization

Rust社区认为测试主要分为两类：单元测试unit tests和集成测试integration tests。单元测试小并且聚焦，一次只测试一个模块，并且支持测试私有接口。集成测试会从库外部进行测试，测试方式与其它模块调用库的方式相似，只能使用公开接口，并且会在一个test中测试多个模块。

#### Unit Tests

Rust单元测试的目标是独立测试每一个模块，确保模块的功能能够符合预期。Rust中把单测的代码也放在`src`目录下，在每个文件添加一个`tests`模块，然后用`cfg[test]`注解来标注测试函数。

##### The Tests Module and #[cfg(test)]

使用 `#[cfg(test)]` 注解来标注test module，在执行`cargo test`命令时，Rust编译器会只编译并运行这些注解标注的module。`cfg`属性代表configuration配置，Rust接下来的item只会在在对应的配置下才会被包含，在 `#[cfg(test)]` 配置下，配置选项是test，这个选项是Rust为编译和运行tests提供的选项。因此，通过cfg属性，Cargo只有在执行 `cargo test` 时才会编译test code。

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```

##### Testing Private Functions

test module与其他module是一样的，因此，在测试私有函数时，可以使用Rust同样的访问控制规则。例如，使用 `use super::*` 来测试parnt module的函数。

#### Integration Tests

Rust的集成测试是完全从外部来测试library。集成测试的案例用法，与其它正常库的用法是一样的，也只能调用library的公开API。单元测试通过的代码，在集成阶段也可能会出现问题。对于集成测试，需要在项目根目录下，创建一个 `tests` 目录，与`src`目录同级。Rust编译器会通过`tests`目录识别这个目录下的文件是测试文件，因此不需要再通过 `#[cfg(test)]` 注解标注module。

```rust
// tests/integration_test.rs
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

可以通过 `--test` 参数来限制指定特定的集成测试，例如 `cargo test --test integration_test`

## 13. Functional Language Features: Iterators and Closures

### 13.1 Closures: Anonymous Functions that Capture Their Environment

Rust闭包是一个匿名函数，闭包可以赋值一个变量或当做参数传递给其他函数。可以在一个地方创建闭包，在另一个上下文环境完全不同的地方调用闭包，与函数不同，闭包可以捕捉定义它们的作用域的值。

Rust闭包与函数的定义方式不同，Rust闭包的执行环境通常可以做类型推断，因此定义闭包时可以简化写法。Rust编译器会自动推断闭包的定义，包含每个参数的基本类型以及返回值的类型。

```rust
// 函数定义
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
// 闭包定义
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

闭包从环境中捕捉值有三种方式，与函数的参数的所有权的处理方式相同：`borrowing immutably, borrowing mutably, taking ownership`。闭包处理捕捉的值的所有权时，采取哪种方式取决于在闭包包体内怎么操作这些值。

* borrowing mutably：需要注意的是，根据引用的基本原则，一旦在闭包使用了变量，且发生了对这个变量的可变引用，因此，在闭包被调用前，就不能在发生对这个变量的任何引用了。
* taking ownership：如果希望闭包能够完全从执行环境中获取values的所有权，可以在闭包参数定义列表前使用 `move` 关键字定义。这个比较常见的场景是在多线程闭包中使用；

#### Moving Captured Values Out of Closures and the Fn Traits

一旦定义了一个闭包，闭包就会从执行环境中捕捉一个值的引用或所有权（例如，影响哪些可以转移到闭包中）。而当闭包被调用执行时，这些值会如何被操作，则是通过闭包包体内的代码来定义的（例如，影响哪些可以从闭包中转移出来）。具体的行为有：从闭包中移出一个被捕捉的值、修改捕捉的值，不会转移或修改这些值，不会从执行环境中捕捉任何值。

闭包如何捕捉和处理来自上下文中的值values，取决于闭包实现的traits有哪些，函数和结构体则通过traits来决定使用哪种闭包类型。闭包会自动实现一个、两个、或全部三个`Fn` traits，定义几个取决于闭包包体如何处理外部值values的：

* `FnOnce` 对只被调用一次的闭包起作用。所有闭包都会至少实现这个trait，因为所有的闭包都会被调用。如果一个闭包，会从闭包包体中转移出捕捉的值values，这个闭包只会实现FnOnce这一个trait，不会再实现其他Fn Traits，因此，此类型闭包只会被调用一次。
* `FnMut` FnMut 应用的闭包类型是，*不会从闭包包体中移出已捕捉的值values*，但可能会修改那些已捕捉的值values。因此，此类型的闭包可以被调用多次。
* `Fn` 对那些不会从执行上下文环境中捕捉任何值的闭包起作用。这些闭包在不影响执行环境的情况下，可以被调用多次，这个特性对并发多次调用闭包非常重要。

需要注意的是：函数也可以实现三个`Fn`特点traits，如果我们想要的是，不要求从执行环境中捕捉值，我们可以使用函数名，而不是实现了一个Fn的traits的闭包。

### 13.2 Processing a Series of Items with Iterators

Rust的迭代器是懒加载的，只有当你调用方法消耗迭代器的时候，才会有实际的效果。

Rust中的迭代器都实现了一个标准库中的trait特征 `Iterator`。同时，标准库的Iterator特征默认也提供了非常丰富的方法实现，例如 filter/map/find/last/sum等方法，都是在这个标准实现的，[https://doc.rust-lang.org/std/iter/trait.Iterator.html](https://doc.rust-lang.org/std/iter/trait.Iterator.html)。其中Iterator trait中的方法可以分为两类：

* consuming adaptors 消费类适配方法：会调用next方法，会消耗迭代器；
* iterator adaptors 迭代器适配方法：不消耗迭代器，但是会通过改变原始迭代器的某个方面，然后产生新的不同的迭代器。例如 `map`方法，会返回一个新迭代器，然后集合collect方法可以返回一个全新的集合类型。

通过这些方法，我们可以实现类似函数式编程的概念，把多个iterator adaptors连接起来进行调用，以友好可读的方式实现复杂行为操作。但因为iterator adpators是lazy的，所以必须使用consuming adaptors才能真正执行。

### 13.3 Improving Our I/O Project

大多数Rust程序员都会更倾向于使用迭代器模式，迭代器的代码可读性更高，不用关注用于遍历的琐碎逻辑，而只需要关注高层次的逻辑信息即可。

### 13.4 Comparing Performance: Loops vs Iterators

Rust有个继承自C++的重要设计理念：zero-cost abstractions （What you dont'use, you don't pay for it. And further: What you do use, you clound't hand code any better.）。Rust的迭代器正好符合这个理念，迭代器这样高层次的抽象代码，会被编译为与你手写的底层for循环代码有一样的性能，即高层次抽象并不会带来额外的运行时开销。

## 14. More about Cargo and Crates.io

Rust的 `release profiles` 正式发布配置是预定义的，同时，开发者也可以通过修改配置文件来也针对一些选项配优化编译。Cargo有两个主要的profile，`dev` profile 是在调用`cargo build`命令时使用的配置文件，`release` profile是在调用`cargo build --release`命令时使用的配置文件。同时，默认情况下，这两个配置文件已经做了很好的预定义配置。

修改配置的话，可以在 `Cargo.toml`文件中的`[profile.*]`章节内添加自定义配置，例如 `opt-level`选项。

### 14.1 Publishing a Crate to Crates.io

与Java的Mavaen，Node的NVM类似，Rust也有一个库的登记中心网站，即 [crates.io](crates.io)。我们可以把我们的包的源码公开发布到这个网站上。

#### Making Useful Documentation Comments

Rust使用 `//` 进行代码注释，但Rust还有一个特殊的文档注释类型，即`documentation comment`，这个注释信息会生成HTML文档，会包含对开发者公开的API的信息，使用`///`来创建文档注释，并且支持Markdown用于格式化文字。Rust提供了`cargo doc --open`命令来展示当前单元包的HTML文档。具体来说，文档注释支持的Markdown heading类型有以下几种：

* Examples：API调用示例代码。同时，Rust也是用这个部分的内容用于做文档测试，当我们调用`cargo test`的时候，也会执行这里的代码用于测试。
* Panics：标准那些场景可能会产生panic
* Erors：如果函数返回Result，这里描述那些场景会产生哪些错误类型。
* Safety：如果函数调用是Unsafe的，这部分可以用来解释为什么函数是unsafe的，需要调用者覆盖哪些异常场景

Rust使用 `//!`注释来说明单元包或模块的信息；

#### Exporting a Convenient Public API with pub use

发布crate单元包时需要特别关注公开API的设计结构，以降低API使用者的上手成本。使用Rust的pub和use关键字来导出和使用公开API，在开发期间我们使用这种语法可以通过比较深的层次来管理代码，但是针对单元包的公开API使用者而言，调用时就会特别冗长，例如使用长路径`use my_crate::some_module::another_module::UsefulType;`，而不是短路径`use my_crate::UsefulType;`。针对这个问题，可以通过重新导出公开项(re-export items)的方式来解决这个问题，即使用`pub use`声明来做re-export。

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use self::kinds::PrimaryColor; // 使用 pub use来重新导出公开API，简化调用成本
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

总之，在top level即库的最外层使用 `pub use` 来重新导出nestd modules，即内部层级比较深的库，可以大幅提升API调用者的开发体验。

#### Setting Up a Crates.io Account

发布公共单元包的基本步骤：

* 在[crates.io](crates.io)上创建一个账户；
* 访问[https://crates.io/me/](https://crates.io/me/)，获取一个API访问token；
* 在终端内通过 `cargo login` 命令行来登录，例如 `cargo login abcdefghijklmnopqrstuvwxyz012345`；

#### Adding Metadata to a New Crate

在Cargo.toml的package section添加关于库的基本信息：

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

license公开协议类型可以参考Linux基金会网站，[Linux Foundation’s Software Package Data Exchange (SPDX)](https://spdx.org/licenses/)。

#### Publishing to Crates.io

使用 `cargo publish` 把库发布到crates.io网站上，需要注意的是，发布时要非常谨慎，因为发布是永久性的，不可以撤销版本或删除代码。

#### Publishing a New Version of an Existing Crate

Rust的版本规则也是语义版本规则，[https://semver.org/](https://semver.org/)。

#### Deprecating Versions from Crates.io with cargo yank

Cargo可以通过 `cargo yank` 命令来设置某个发布的版本为过期版本。一旦被yank后，新项目就不能再使用这个版本，但是已有项目可以继续使用。同时，也会破坏Cargo.lock文件，后续更新版本时，就会更新lock文件。

`cargo yank --undo`，也可以通过undo选项来撤销过期设置。

### 14.2 Cargo Workspace

Rust项目逐步变得复杂以后，也可以把代码分割为多个库单元包，Cargo提供的方案与iOS项目类似，也是workspaces，用于协同开发多个关联比较紧密的包。

`A workspace is a set of packages that share the same Cargo.lock and output directory.`，工作空间是可以共享Cargo.lock文件和输出目录的一系列包的集合。在Cargo.toml文件的`[workspace]`章节中，添加members数组，来配置多个包。注意：workspace的构建产物会放在根目录的`target`目录下。

如果是内部库之间相互依赖，可以在对应的Cargo.toml中添加依赖配置：

```toml
[dependencies]
add_one = { path = "../add_one" }
```

需要注意的是，如果依赖外部库，workspace有且只有一个 `Cargo.lock`文件，这是因为需要确保不同单元包依赖的库的版本是一致的。

Cargo在设计之初就具备让开发者可扩展子命令的能力。

## 15. Smart Pointers

指针是一个通用概念，即一个包含内存地址的变量，而这个内存地址则会指向其他数据。Rust中的最常见的指针类型是引用reference。引用通过&符号来表示，并且借用他们指向的值。除了指向的数据不一样，指针没有其他任何特殊能力。相对指针来说，智能指针（Smart pointers）是一个特殊数据结构，表现上与指针类似，但同时会有附加的元数据和能力。Rust的智能指针起源于C++。Rust有所有权和借用概念，引用和智能指针有一些额外的差异：`引用只能借用数据，但不能拥有数据。 但大多数情况下，智能指针拥有它所指向的数据`。

智能指针通常是基于结构体实现的，主要通过实现`Deref`和`Drop` traits具备一些特殊能力。 `Deref`允许智能指针结构体实例表现像一个引用。`Drop` 则允许智能指针自定义实例离开作用域时的逻辑代码。Rust中智能指针是很常见的case：

* `Box<T>` 在堆上申请空间创建值
* `Rc<T>` 支持多个所有权的引用计数类型
* `Ref<T>` 和 `RefMut<T>` ，通过`RefCell<T>`来进行访问，这个类型强制借用规则在运行时期间生效，而不是编译期。

### 15.1 Using 1`Box<T>` to Point to Data on the Heap

`Box<T>`的主要功能是提供间接引用和堆内存分配，所以与其它智能指针相比，并没有性能开销；，实际应用场景主要有三个：

* 场景一：When you have a type whose size can’t be known at compile time and you want to use a value of that type in a context that requires an exact size。即对于一些数据类型，在编译期间没法确定大小，但是使用时又必须要求有明确的大小。
  * 场景：Recursive Types，某个类型的值内部拥有一个同类型的值，这个类型如果在栈上存储，Rust编译器就需要在编译器计算这个类型的大小，但是由于是递归类型，所以无法计算出来大小，这个时候就需要通过Box放到堆上，同时优于Box是指针大小是固定，所以Rust编译器就能计算出来类型的大小。比较典型的是链表实现
* 场景二：When you have a large amount of data and you want to transfer ownership but ensure the data won’t be copied when you do so。传输大量数据的所有权，但又想确保这些数据不能被拷贝的场景
  * 场景：栈上的数据直接传递所有权时，会直接进行copy，这对性能很不利，这时候可以使用堆上数据来代替
* 场景三：When you want to own a value and you care only that it’s a type that implements a particular trait rather than being of a specific type。使用trait作为类型的使用场景。

Recursive Types可以参考如下的案例：

```rust
enum List {
    Cons(i32, Box<List>), // Box<T> 指针大小是固定的，Rust编译器可以计算出来List类型的大小，从而决定栈帧上为List类型变量分配空间大小
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

### 15.2 Treating Smart Pointers Like References with the `Deref Trait`

智能指针通过实现 `Deref` 获得与常规引用一样的行为表现，使用引用的操作符也适用于智能指针，典型的就是解引用dereference operator `*`。Rust标准库中定义的`Deref` trait，包含了一个 `deref`方法，定义如下：`fn deref(&self) -> &Self::Target;`，这个方法会返回内部数据的*引用*。

如果不实现`Deref` trait，Rust编译器只能解引用常规的`&`引用。但对于实现了`Deref` trait的任意数据类型，调用`deref`方法就用于解引用的`&`引用。例如，对于实现了Deref的类型y， `*y`等价于`*(y.deref())`，Rust编译器会在编译器自动转换为调用deref。

`Deref coercion`特性，用于自动转换一个实现Deref trait的类型的引用，到另一种类型的引用，用在函数和方法的参数的隐性自动转换。比较典型的案例是，&String和&str之间的自动转换，即参数类型为&str的时候，可以传递&String类型，这是因为String实现Deref trait，会返回&str类型。

同样，针对可变引用类型，可以通过DerefMut trait来重写可以引用的 `*` 操作符。

* From `&T` to `&U` when `T: Deref<Target=u>`
* From `&mut T` to `&mut U` when `T: DerefMut<Target=U>`
* From `&mut T` to `&U` when `T: Deref<Target=U>`。这里比较特殊，Rust允许自动转换可变引用到不可变引用，但是反过来是不行的。可变引用转换到不可变引用不会破坏借用规则，由于Rust会确保只有一个可变引用，所以理论上来说，这种转换是可行的，不会破坏借用规则。但优于不可变引用有多个，不能保证当前要转换的引用是唯一的，如果把这个引用转换为可变，则会违反只允许存在一个可变引用，且不能有不可变应用的规则

### 15.3 Running Code on Cleanup With the `Drop Trait`

智能指针第二个比较重要的trait是`Drop Trait`，允许智能指针自定义值要离开作用域时，会有哪些自定义行为，例如释放网络连接资源等。比较典型的使用案例就是`Box<T>`类型，在Box离开作用域时，Rust会自动调用`drop`方法，做一些释放前的清理工作，类似析构函数，释放数据结构、关闭网络连接和文件句柄、清理缓存等。

`drop`调用是Rust自动调用行为，通常来说，我们不能直接调用这个方法，Rust编译器会报错。在一些场景时，我们需要提前释放一些数据时，例如并发锁场景。这个场景可以通过`std::mem::drop`函数来解决。

### 15.4 `RC<T>` the Reference Counted Smart Pointer

`RC<T>`是基于引用计数的拥有多个owner的智能指针，用于在多个owner之间共享*不可变的只读数据*。与Objective-C的内存管理类似，当引用计数降到0时，值就会被清理clean up。例如，graph data structure，多条边可能指向同一个节点，这时阶段就有多个拥有者，只有当所有边都不指向这个节点时，这个节点才能释放。注意，`RC<T>`只适用于单线程。

通过调用`RC::clone`方法增加引用计数，与大多数类型的`clone`方法实现不同，`RC::clone`方法不会对所有数据进行深拷贝，只是对引用计数进行增加，不会像深拷贝那样耗时。

与OC类似，Rust提供了Rc::strong_count和Rc::weak_count方法来打印当前的引用计数值。

### 15.5 `RefCell<T>` and the Interior Mutability Pattern

Rust语言的`Interior mutability`设计模式，允许修改不可变value中value。`Mutating the value inside an immutable value is the interior mutability pattern`。

`RefCell<T>`是基于这个设计模式的实现，与引用和`Box<T>`在编译器进行borrowing rules的检查不同，`RefCell<T>`允许在运行时对borrowing rules进行check，同时，这也意味着会有unsafe code的存在。需要注意的是，`RefCell<T>`也是需要满足借用规则的，只是在运行时检测而已。主要的使用场景是，由于Rust编译器本身静态分析，相对来说比较保守，会导致一些场景的实现方式很麻烦，所以Rust提供了这种运行时规则检测能力。

`Box<T>` vs `Rc<T>` vs `RefCell<T>`对比：

* `Rc<T>` enables mutiple owners of same data; `Box<T>` and `RefCell<T>` have single owners. 即 `Rc<T>` 允许对同一个数据有多个所有者，`Box<T>` 和 `RefCell<T>`则只有一个所有者。
* `Box<T>` allows immutable or mutable borrow checked at compiled time; `Rc<T>` allow immutable borrows checked at compiled time; `RefCell<T>` allows immutable or mutable borrows checked at runtime. `Box<T>` 在编译期间允许可变和不可变借用规则检查，`Rc<T>` 在编译期间允许不可变借用检查，`RefCell<T>` 在运行时允许不可变或可变借用检查。
* Because `RefCell<T>` allows mutable borrows checked at runtime, you can mutate the value inside the `RefCell<T>` even when the `RefCell<T>` is immutable. 因为`RefCell<T>` 允许在运行时进行可变检查，所以即使`RefCell<T>`引用是不可变的，也能够修改内部值。

```rust

pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &'a T, max: usize) -> LimitTracker<'a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You've used up over 75% of your quota!");
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>, // 这里使用允许可以修改内部的Vec<String>
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message)); // 由于使用了RefCell，所以可以向量里增加数据
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

需要注意的是，`RefCell<T>`仍然需要遵循借用规则，即只有一个可变引用或多个不可变应用，只是借用规则检查是发生在运行时的。与引用操作符的`&`和`&mut`语法类似，`RefCell<T>`有对应的`borrow`和`borrow_mut`方法，可以运行时进行借用。`borrow`方法返回一个`Ref<T>`类型的智能指针，`borrow_mut`返回一个`RefMut<T>`类型的智能指针，这两个返回的指针类型都实现了`Deref`，也就是说可以像常规指针一样被操作。`RefCell<T>`会记录有多少个活跃的`Ref<T>`和`RefMut<T>`智能指针。

既然`RefCell<T>`有运行时借用规则，与编译时检查的编译错误类似，如果运行时违反了借用规则，则会出现panic现象。`RefCell<T>`相对于编译检查，存在把内存管理问题带入生产环境、轻微性能问题等。

结合`Rc<T>`和`RefCell<T>`的能力，即可以有多个owner，又能修改值。如下：

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

### 15.6 Reference Cycles Can Leak Memory

Rust的内存管理仍然可能存在循环引用导致的内存泄露问题，例如，使用`Rc<T>`和`RefCell<T>`导致内存泄露，使用`Weak<T>`来解决循环引用问题。

## 16. Fearless Concurrency

Rust在并发编程的目标是安全和高效。注意，并发编程Concurrent Programming和并行编程Parallel Programming概念上是有差别的，并发编程是程序的不同部分可以独立执行，并行编程是程序的不同部分可以在同一时间执行。借助于Rust的ownership和type checking能力，很多并发编程错误在编译期就能被发现，从而避免潜在的线上问题。同时，与Erlang这种高级语言的仅支持小部分并发方式而换取效率的策略不同，Rust定位是低级语言，需要在各种场景下有最好的性能，尽可能减少硬件层的抽象，因此，Rust提供了一系列并发能力来解决问题。（PS，我比较喜欢一些资深专家写的图书的原因也在于此，不会只讲语法，而是会讲如何思考的以及经验总结）

Rust的线程模型是1:1模型，Rust的一个线程对应的就是OS的一个线程，这与Java和Go的语言层并发与OS的并发不是一一对应不同。

并发编程，常见的并发问题有三种：

* Race Conditions资源竞争，多线程同时访问资源或数据时，访问顺序是随机的；
* Deadlocks死锁，两个线程相互等待，两个线程都无法执行下去；
* 在特定场景偶像的Bugs，并且难以复现和可靠修复；

### 16.1 Using Threads to Run Code Simultaneously

Rust线程的基础接口：

* 创建线程接口：`thread::spawn`函数，入参是一个闭包，这个闭包里包含了子线程要执行的逻辑；
* 线程休眠：`thread::sleep`函数
* 等待线程执行结束：`JoinHandle`对象，调用`thread::spawn`函数会返回一个handle句柄，当调用句柄的`join`方法时，就会等待线程执行完毕才会继续执行；

```rust
use std::{thread, time::Duration};

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("spawned thread: {}", i);
            thread::sleep(Duration::from_millis(1));
        }
    });
    handle.join().unwrap();
    for i in 1..5 {
        println!("main thread: {}", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

同时，在多线程中捕获外部变量，需要使用`move`，即直接转移所有权到多线程闭包中：

```rust
fn main() {
    let v = vec![1, 2, 3];
    let handle = thread::spawn(move || {
        println!("{:?}", v);
    });
    handle.join().unwrap();
}
```

### 16.2 Using Message Passing to Transfer Data Between Threads

当前比较流行的确保安全并发的方式是消息通道，与Go语言的channels并发类似，"Do not communicate by sharing money; instead, share memory by communicating"。Rust在基础库也提供了channel实现，用于在不同线程之间传递数据。可以把channel想象为一个单向水流，你在上游放入一个物品，下游就会收到这个东西。因此，channel有两个参与者：a transmitter and a receiver。

并发库：`mpsc`基础库，概念缩写`mutiple producer, signel consumer`，意思是，Rust标准库实现的channel，可以有多个sendings来生产values，但是只能有一个receiving来接收和消费这些values。
创建channel：通过`mpsc::channel`函数来创建，返回值是一个元组，第一个值是发送端-the transmitter，第二个是接收端-the receiver。需要多个发送者时，可以调用发送者的`clone`方法获取；

```rust
fn main() {
    let (tx, rx) = mpsc::channel();
    // 第一个发送者；
    let tx1 = tx.clone(); // 通过clone获取多个发送者；
    thread::spawn(move || {
        let vals = vec![
            String::from("hi1"),
            String::from("hi2"),
            String::from("hi3"),
            String::from("hi4"),
        ];
        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_millis(1));
        }
    });
    // 第二个发送者
    thread::spawn(move || {
        let vals = vec![
            String::from("hello1"),
            String::from("hello2"),
            String::from("hello3"),
            String::from("hello4"),
        ];
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_millis(1));
        }
    });
    // 接收者
    for recived in rx {
        println!("Got: {}", recived);
    }
}
```

### 16.3 Shared-State Concurrcy

与Go语言的"do not communicate by sharing memory"原则不同，Rust提供了共享内存的并发方案-Mutux互斥量，Mutex是`mutual exclusion`的缩写，互斥量在同一时间只允许一个线程访问特定的数据。如果想要访问互斥量中的数据，线程必须先获取互斥量的锁，来表明需要访问数据。Mutux之所以使用起来比较困难，因为使用时必须时刻记着两个原则：

* 在使用数据之前，必须尝试获取锁，只有成功获取锁才能去访问数据；
* 当使用完Mutux保护的数据后，必须解锁，解锁后其它线程才能访问这个数据；

Mutux创建的API是`Mutex::new`关联函数，获取锁的API是Muxex的`lock`方法。

```rust
use std::{sync::{Arc, Mutex}, thread};

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    for handle in handles {
        handle.join().unwrap();
    }
    println!("Result: {}", *counter.lock().unwrap());
}
```

`Mutex<T>` 提供了内部可变性interior mutability，类似RefCell提供的能力。

### 16.4 Extensible Concurrency with the Sync and Send Traits

Rust语言层面提供了很少的并发特性，包括Mutex之和channel在内的并发特性都是基础库提供的，这其实也意味着你可以实现自己的并发特性。

* `Send`特性：允许在多个线程之间转移所有权，几乎全部的Rust类型都实现了`Send`特性，允许通过`move`在多线程之间转移所有权。例外就是，`Rc<T>`类型，由于这个类型并没有实现`Send`特性，所以只能在单线程环境中执行。
* `Sync`特性：允许多个线程之间引用同一个不可变值，即 `type T is Sync if &T (an immutable reference to T) is send`，意味着这个类型的引用可以在多线程之间安全传递。与`Send`类似，Rust的primitive types基本类型(标量类型，例如整数和浮点；组合类型，元组和数组)是Sync的，同时，完全由Sync类型组合而成的类型也是Sync的。

由于由Send和Sync特性组成类型默认是Send和Sync的，我们不需要手动实现这些特性，与其它marker traits一样，他们甚至没有可以实现的方法，只是用来标记与并发相关的特性而已。同时，手工实现这些traits实现的是不安全代码。

## 17. Object Oriented Programming Features of Rust

Rust受到很多编程范式的影响，从创建的OOP概念定义而言，即"Gang of Four"在设计模式的定义，Rust的sturct和enum也是面向对象的：

`Object-oriented programs are made up of objects. An object packages both data and the prodedures that operate on that data. The procedures are typically called methods or oprations.`

Rust封装encapsulation实现，默认情况下，所有modules、types、funtions、methods代码都是私有的，只有通过`pub`关键字定义才能是公开可访问的。

Rust并不支持继承Inheritance语法，除了通过macro宏，没有办法直接继承父结构体的field成员和method方法实现。选择OOP的主要有两个原因：代码复用 reuse of code、基于继承的类型替换（即polymorphism多态）。代码复用可以通过Rust trait复用替换，多态也可以基于trait来替换。

现代编程语言基本上都不再支持继承语法，例如Go、Swift等，这也是由于继承本身会带来不必要的代码共享。第一，子类大多数情况下并不需要继承父类的全部特性，但由于继承特性，不得不共享所有代码，因此继承导致设计灵活性比较低。第二，方法调用关系过于复杂，例如父类调用子类的方法实现，导致莫名其妙的错误。（PS，这里深有体会，我们实际项目中，基类基本上是没人敢修改、修改就出问题的存在，太多不需要的代码耦合在一起，修改代码影响范围完全不可控，你无法阅读代码显式了解一个类的全部功能，太多行为都是隐式的）

与Rust泛型的单态化monomorphization的静态派发（static dispatch）不会带来性能问题不同。Rust的Trait Objects是动态派发dynamic dispatch的，编译器在编译期间无法确定需要调用哪个方法，因此需要在运行时来动态派发调用方法。

Rust设计模式实现，可以参考网站，设计模式讲解和Rust代码示例很完整，[https://refactoring.guru/design-patterns/catalog](https://refactoring.guru/design-patterns/catalog)

## 18. Patterns and Matching

Patterns模式是一个特殊语法，Rust使用这个语法来匹配不同结构的类型，既有复杂的也有简单的，基于模式可以获得更灵活的程序控制流，例如match表达式、if let表达式等。模式由以下几个元素组成：

* Literals
* Destructured arrays, enums, structs, or tuples
* Variables
* Wildcards
* Placeholders

### 18.1 All the Places Patterns Can Be Used

模式的使用场景，有match、if let、while let、for循环、let表达式、函数参数：

* 1、`match`分支：在match的分支中使用Pattern，需要注意的是，mactch需要做到完全匹配，分支要覆盖值的所有可能性；

```rust
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

* 2、`if let`条件表达式：if let表达式使用场景与match类似，但更简洁方便，用于只需要匹配少量的case场景，不需要像match一样匹配所有可能的值。

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

* 3、`while let`条件循环：与if let语法类似，允许在while循环中来做模式匹配

```rust
    let mut stack = Vec::new();

    stack.push(1);
    stack.push(2);
    stack.push(3);

    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
```

* 4、`for`循环：Rust中的for in循环语法，其实遍历的值语法也是模式语法。

```rust
    let v = vec!['a', 'b', 'c'];

    for (index, value) in v.iter().enumerate() {
        println!("{} is at index {}", value, index);
    }
```

* 5、`let`表达式：与for循环类似，let表达式其实模式语法，简单的变量声明语句其实也是模式语法

```rust
    let (x, y, z) = (1, 2, 3);
```

* 6、函数参数：与for和let表达式语法是模式一样，函数的参数其实也是模式，即使是最简单的参数变量声明

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}
```

### 18.2 Refutability: Whether a Pattern Might Fail to Match

模式Patterns分为两类：refutable和irrefutable。可以匹配传入的任何值的Pattern是refutable，因为匹配不会存在失败的可能性，例如变量声明`let x = 5;`。允许在某些情况下匹配失败的Pattern是irrefutable，因为有可能无法匹配到值，例如if let语法，`if let Some(x) = a_value`，这里如果a_value是None，匹配就会失败。

* 函数参数、let声明、for循环：只能接受irrefutable的模式，因为这种情况下，如果匹配失败，程序不能再执行任何有意义的事情；
* `if let`和`while let`表达式：可以接受refutable和irrefutable模式，但是编译器会对irrefutable模式有告警，因为正常情况下，表达式条件存在的意义是会根据成功或失败执行不同的动作，如果是irrefutable的，执行路径是固定的，条件就没有存在的意义；
* `match`分支：match分支除了最后一个分支之外，其它分支都只能使用refutable模式，最后一个分支因为需要匹配其它全部条件，所以可以是irrefutable模式；

### 18.3 Pattern Syntax

*Matching Literals*语法：匹配常量

```rust
    let x = 1;

    match x {
        1 => println!("one"),
        2 => println!("two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
```

*Matching Named Variables*语法：匹配命名变量

```rust
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {y}"),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {y}", x);
```

*Multiple Patterns*语法：使用 `|` or语法来匹配多个值

```rust
    let x = 1;

    match x {
        1 | 2 => println!("one or two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
```

*Matching Ranges of values with..=*语法：允许匹配一个范围的值

```rust
    let x = 5;
    match x {
        1..=5 => println!("one through five"),
        _ => println!("something else"),
    }

    let x = 'c';
    match x {
        'a'..='j' => println!("early ASCII letter"),
        'k'..='z' => println!("late ASCII letter"),
        _ => println!("something else"),
    }
```

*Destructuring to Break Apart Values*语法：与ES6中的解构赋值语法是一样的，Rust的解构语法支持structs、Enums、Nested Structs and Enums、Structs and Tuples等类型。同时，Rust甚至支持各种混合、匹配、内嵌复杂类型的解构语法，例如 `let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });`

1、结构体structs解构语法

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    // 创建变量a和变量b
    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
    // 简化写法，直接罗列结构体成员变量的名字，创建变量x和变量y
    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
    // match匹配语法
    match p {
        Point { x, y: 0 } => println!("On the x axis at {x}"),
        Point { x: 0, y } => println!("On the y axis at {y}"),
        Point { x, y } => {
            println!("On neither axis: ({x}, {y})");
        }
    }
}
```

2、枚举Enums解构语法

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.");
        }
        Message::Move { x, y } => {
            println!("Move in the x direction {x} and in the y direction {y}");
        }
        Message::Write(text) => {
            println!("Text message: {text}");
        }
        Message::ChangeColor(r, g, b) => {
            println!("Change the color to red {r}, green {g}, and blue {b}",)
        }
    }
}
```

3、内嵌结构体和枚举解构语法

```rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("Change color to red {r}, green {g}, and blue {b}");
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!("Change color to hue {h}, saturation {s}, value {v}")
        }
        _ => (),
    }
}
```

4、混合类型结构

```rust
let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
```

*Ignoring Values in a Pattern*语法：模式语法中的忽略值语法非常有用，例如match最后一个分支的兜底匹配。主要有几个形式：使用`_`模式忽略值、在其它模式中使用`_`模式、使用下划线开头的名字、使用`..`忽略value的剩余部分。

1、使用`_`忽略全部值：`fn foo(_: i32, y: i32) {}`

2、使用内嵌`_`忽略部分值

```rust
    let numbers = (2, 4, 8, 16, 32);
    match numbers {
        (first, _, third, _, fifth) => {
            println!("Some numbers: {first}, {third}, {fifth}")
        }
    }
```

3、使用下划线开头的名字来忽略未使用变量：例如 `let _x = 5`，x仍然会绑定到一个变量，内存所有权管理仍然生效。主要使用场景是设计应用原型阶段，暂时忽略变量；

4、使用`..`忽略值的其它部分：

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {first}, {last}");
        }
    }
}
```

*Extra Conditionals with Match Guards*语法：只适用于match分支，在match模式后跟着if条件判断，用于更复杂的语法表达。例如：`Some(x) if x % 2 == 0`

```rust
    let num = Some(4);

    match num {
        Some(x) if x % 2 == 0 => println!("The number {} is even", x),
        Some(x) => println!("The number {} is odd", x),
        None => (),
    }
```

*@Bindings*语法：允许对正在进行模式匹配的值，创建一个新变量来指向这个值。

```rust
    enum Message {
        Hello { id: i32 },
    }

    let msg = Message::Hello { id: 5 };

    match msg {
        Message::Hello {
            id: id_variable @ 3..=7,
        } => println!("Found an id in range: {}", id_variable),
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        }
        Message::Hello { id } => println!("Found some other id: {}", id),
    }
```
