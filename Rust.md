# Rust

## 基本语法
### 变量
- Rust是静态类型语言，要提前指定类型，并且在编译时期被检查；Rust 有个称为“类型推断”的机制。如果它能推断出变量是什么类型，Rust 就不需要指定实际类型
- let绑定后的变量是**不可变**的
- 如若要绑定可变变量，需要使用**mut**关键字
```
let x = 5;
let (x, y) = (1, 2);
let x: i32 = 5;
x = 10; //报错
let mut y = 10;
y = 100; //不报错
```


### 函数
- main()函数是每个 Rust 程序的开始
- 当你看到一个 **!**，这意味着你调用了宏而不是一般的函数，Rust 的宏是明显不同于 C 的宏
- println!()
- panic!() 会导致当前执行线程的崩溃并给出特定的消息
- 必须声明函数参数的类型
```
fn print_sum(x: i32, y: i32) {
    println!("sum is: {}", x + y);
}
```
```
fn add_one(x: i32) -> i32 {
    x + 1
}
```
```
fn add_one(x: i32) -> i32 {
    return x + 1;
}
```
- 函数也是一种类型
```
fn foo(x: i32) -> i32 { x }
let x: fn(i32) -> i32 = foo;
```

### 引用包

-   ```external crate package```**包**（_Packages_）是 Cargo 的一个功能，它允许你构建、测试核分享 crate。
-   _Crates_  是一个模块的树形结构，它形成了库或二进制项目。
-  **模块**（_Modules_）和  _use_  关键字允许你控制作用域和路径的私有性  ```use package::module```
-  **路径**（_path_）是一个命名例如结构体、函数或模块等项的方式  ```package::module.function```

## 数据类型
### boolean
```
let x = true;
let y: bool = false;
```
###  char
unicode值
```
let x = 'x';
let two_hearts = '💕';
```
### 数值类型
有符号数和无符号数、固定长度和可变长度、浮点数和整数
i8 i16 i32 i64 isize 有符号整数
u8 u16 u32 u64 usize 无符号整数
f32 f64
```
let x = 42; // x has type i32
let y = 1.0; // y has type f64
```

### 数组
相同类型的元素和固定大小的顺序对象
```
let a = [1, 2, 3]; // a: [i32; 3]
let mut m = [1, 2, 3]; // m: [i32; 3]
let names = ["Graydon", "Brian", "Niko"]; // names: [&str; 3]

let a = [0, 1, 2, 3, 4];
let middle = &a[1..4]; // A slice of a: just the elements 1, 2, and 3
let complete = &a[..]; // A slice containing all of the elements in a
```

### 元组
固定大小的有序列表
```
let x = (1, "hello");
let x: (i32, &str) = (1, "hello");

let (x, y, z) = (1, 2, 3);

(0,); // single-element tuple
(0); // zero in parentheses

let tuple = (1, 2, 3);
let x = tuple.0;
let y = tuple.1;
let z = tuple.2;
```

### 迭代器
```
for x in 0..10 {
    println!("{}", x);
}
```
范围 **(0 . . 10)** 是一个迭代器。我们可以使用 **.next()** 方法反复调用迭代器


## 作用域
### 堆栈
- 栈是非常快的，也是 Rust 默认的内存分配方式。但是分配存在于本地函数调用，且在大小方面是有限的。另一方面，堆的速度相对比较慢，但是你的程序可以明确地分配堆内存。且它实际上是无限制的，可以在全局范围内访问。
- 堆
```
let x = Box::new(5);
```
### 所有权
- Rust 中的每一个值都有一个被称为其  **所有者**（_owner_）的变量。
- 值有且只有一个所有者。
- 当所有者（变量）离开作用域，这个值将被丢弃。
- 对象 -> 移动变量
```
let s1 = String::from("hello");
let s2 = s1; //此时s1就失效了，释放了自己
println!("{}, world!", s1);
```
```
let s1 = String::from("hello");
let s2 = s1.clone();
println!("s1 = {}, s2 = {}", s1, s2);
```
### 引用
- （默认）不允许修改引用的值
- 可变引用： &mut arg
- 在特定作用域中的特定数据有且只有一个可变引用
```
let  mut s = String::from("hello");
{  
	let r1 =  &mut s;
}  // r1 在这里离开了作用域，所以我们完全可以创建一个新的引用
let r2 =  &mut s;
```
- 不能在拥有不可变引用的同时拥有可变引用
- 在任意给定时间，**要么** 只能有一个可变引用，**要么** 只能有多个不可变引用
## 控制流
### if
```
let x = 5;

if x == 5 {
    println!("x is five!");
} else if x == 6 {
    println!("x is six!");
} else {
    println!("x is not five or six :(");
}
```
```
let x = 5;
let y = if x == 5 { 10 } else { 15 }; // y: i32
```
### for
```
for var in expression {
    code
}
```
```
for x in 0..10 {
    println!("{}", x); // x: i32
}
```
### while
```
let mut x = 5; // mut x: i32
let mut done = false; // mut done: bool

while !done {
	x += x - 3;
	println!("{}", x);

	if x % 5 == 0 {
		done = true;
	}
} 
```
### loop
```
loop {
```
相当于
```
while true {  
```
### continue和break在while循环和for循环都适用
### match
```
let x = 1;

match x {
1 => println!("one"),
2 => println!("two"),
3 => println!("three"),
_ => println!("anything"),
}
```
```
let x = 1;

match x {
1 | 2 => println!("one or two"),
3 => println!("three"),
_ => println!("anything"),
}
```
```
let x = '💅';

match x {
'a' ... 'j' => println!("early letter"),
'k' ... 'z' => println!("late letter"),
_ => println!("something else"),
}
```

## 特征
```
trait HasArea {
	fn area(&self) -> f64;
}

struct Circle {
	x: f64,
	y: f64,
	radius: f64,
}

impl HasArea for Circle {
	fn area(&self) -> f64 {
		std::f64::consts::PI * (self.radius * self.radius)
	}
}

struct Square {
	x: f64,
	y: f64,
	side: f64,
}

impl HasArea for Square {
	fn area(&self) -> f64 {
		self.side * self.side
	}
}

fn print_area<T: HasArea>(shape: T) {
	println!("This shape has an area of {}", shape.area());
}

fn main() {
	let c = Circle {
		x: 0.0f64,
		y: 0.0f64,
		radius: 1.0f64,
	};

	let s = Square {
		x: 0.0f64,
		y: 0.0f64,
		side: 1.0f64,
	};

	print_area(c);
	print_area(s);
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1OTU2MTQwMDddfQ==
-->