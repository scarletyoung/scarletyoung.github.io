---
title: Effective C++ Note
date: 2021/12/09
description: C++ prmie笔记，查漏补缺
---

# lambda表达式
lambda表达式可以表示为一个可调用的代码单元，可以认为是一个匿名的内联的函数，lambda表达式的格式为
```cpp
[capture list] (parameter list) -> return type {function body}
```
参数列表、返回类型和函数体的含义与普通函数相似。捕获列表通常是一些局部变量。参数列表和返回类型通常可以忽略，但是捕获列表和函数体必须要有。
一个lambda表达式的例子如下
```cpp
auto f = [] {return 43;}
cout << f() << endl;
```

## 捕获列表
捕获列表允许lambda表达式的函数体使用列表中的变量，不在成员列表中的变量则不可访问。

当定义一个lambda表达式的时候，编译器会为这个表达式生成一个新的类类型。当将一个lambda表达式传递给函数或者赋值给一个auto变量时，会创建这个类型的对象。

捕获列表可以看作这个类型的成员变量，因此和成员变量类似，当对象创建时初始化。

捕获列表可以按值捕获或按引用捕获，当按值捕获时，这个变量必须能被赋值（如cout）就不可复制，正如前面所说，复制在创建lambda时进行，lambda创建后，局部变量的修改不会影响lambda中的值。
```cpp
void fcn1() {
  size_t v1 = 42;
  auto f = [v1] {return v1;}
  v1 = 0;
  auto j = f(); // j is 42;
}
```

当按引用捕获时，局部变量的修改会影响lambda中的值，同时，需要保证当lambda执行时引用的对象必须存在。如下所示
```cpp
void fcn2() {
  size_t v1 = 42;
  auto f2 = [&v1] {return v1;}
  v1 = 0;
  auto j = f2();  // j is 0
}
```

由于lambda本质上时一个对象，因此，函数可以返回一个lambda表达式，此时，该lambda表达式不能使用引用捕获，否则，在lambda调用的时候，引用的对象不存在。

我们还可以使用隐式捕获，即不在捕获列表中指定捕获的变量，形式如下，前面介绍的可以称为显示捕获。
```cpp
int a = 0;
auto f1 = [=] {return a;}
auto f2 = [&] {return a;}
```
其中，=表示按值捕获，&表示按引用捕获。

我们还可以混合隐式捕获和显示捕获，在混合的情况下，第一项必须时隐式捕获，且显示捕获的方式不能和隐式捕获相同，即当隐式捕获时按值捕获时，显示捕获必须时按引用捕获，反之亦然。

综上所述，捕获列表有以下形式
| 形式 | 说明 |
|---|---|
| [] | 空捕获列表 |
| [names] | 指定捕获变量名称，加&表示按引用捕获 |
| [&] | 按引用隐式捕获 |
| [=] | 按值隐式捕获 | 
| [&, identifier_list] | |
| [=, identifier_list] | |

当按值捕获时，虽然是对变量进行了复制，但是lambda的函数体是无法修改变量的，如果需要修改，需要用mutable修饰，即
```cpp
void fcn3() {
  size_t v1 = 42;
  auto f = [v1]() mutable {return ++v1;}
  v1 = 0;
  auto j = f(); // j is 43;
}
```
当按引用捕获时，能否修改变量取决于引用的对象是否是一个常量。

## 返回类型
如果lambda函数体中包含除了return外的其他声明，那么lambda的返回值默认为void。当lambda函数体中只有一个return表达式时，lambda的返回类型会根据return表达式进行推断。

当函数体有多个表达式且返回值不为void时，我们需要指定lambda的返回类型，此时必须使用尾置返回类型，如下所示
```cpp
transform(vi.begin(), vi.end, vi.begin(), 
[](int i) -> int {
  return if (i < 0) return -i; else return i;
});
```

