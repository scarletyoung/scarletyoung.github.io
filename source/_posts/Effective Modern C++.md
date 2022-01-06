---
title: Effective Modern C++ Note
date: 2022/01/04
---

# Understand template type deduction
```c++
template<typename T>
void f(ParamType param);
f(expr);
```
During compilation, compilers use expr to deduce two types: one for T and one for *ParamType*.
The two types are usually different, because *ParamType* often contains adornment, such as const or reference qualifiers.
```c++
template<typename T>
void f(const T& param);
int x = 0;
f(x);   // T is int, but ParamType is const int &
```
It is natural to expect that the type deduced for T is the type of *expr*. But, the type deduced for T is dependent not just on the type of *expr*, but also on the form of *ParamType*.
There are three cases
1. *ParamType* is a pointer or reference type, but not a [universal reference](#Distinguish universal references from rvalue references)
2. *ParamType* is a universal reference.
3. *ParamType* is neither a pointer nor a reference.

## Case 1: *ParamType* is a pointer or reference type, but not a [universal reference](#Distinguish universal references from rvalue references)

In this case, type deduction works like this
1. I *expr*'s type is a reference(pointer), ignore the reference(pointer) part.
2. Then pattern-match *expr*'s type against *ParamType* to determine T.

```c++
// example one
template<typename T>
void f(T& param);
int x = 27;
const int cx = x;
const int& rx = x;
f(x);   //T is int, ParamType is int&
f(cx);  //T is const int, ParamType is const int&
f(rx);  //T is const int, ParamType is const int&

// example two
template<typename T>
void f(const T& param); // param is const T& now
int x = 27;
const int cx = x;
const int& rx = x;
f(x);   //T is int, ParamType is const int&
f(cx);  //T is int, ParamType is const int&
f(rx);  //T is int, ParamType is const int&
```

## Case 2: *ParamType* is a Universal Reference
The deduced scheme is that
1. If *expr* is an lvalue, both T and *ParamType* are deduced to be lvalue references
2. If *expr* is an rvalue, the Case 1 rules apply.

```c++
template<typename T>
void f(T&& param);
int x = 27;
const int cx = x;
const int& rx = x;
f(x);   // x is lvalue, T is int&, ParamType is int&
f(cx);  // x is lvalue, T is const int&, ParamType is const int&
f(rx);  // x is lvalue, T is const int&, ParamType is const int&
f(27);  // 27 is rvalue, T is int, ParamType is int&&
```

## Case 3: *ParamType* is neither a pointer or a reference
When *ParamType* is neither a pointer or a reference, that means the parameter is pass-by-value. The deduction rules is
1. If *expr*'s type is reference, ignore the reference part.
2. If *expr* is const or volatile, also ignore that.

```c++
template<typename T>
void f(T&& param);
int x = 27;
const int cx = x;
const int& rx = x;
f(x);   // T and ParamType are both int
f(cx);  // T and ParamType are both int
f(rx);  // T and ParamType are both int
```
The reason is simple. Due to pass-by-value, param is a copy of cx or rx.

Consider the case where *expr* is a const pointer to a const object
```c++
template<typename T>
void f(T param);
const char* const ptr = "Fun with pointer";
f(ptr); // ParamType is char * const;
```
Pointer can be change and what pointer point to is const.

## Array arguments
In many context, an array decays into a pointer to its first element.
```c++
const char name[] = "J. P. Briggs";
const char* ptrToName = name;
```
Array types are different form pointer types, even though they sometimes seem to be interchangeable.

There is no such thing as a function parameter that's an array. If a array is pass to a template taking a by-value parameter, the array parameter declaration is treated as a pointer parameters
```c++
template<typename T>
void f(T param);
f(name);  // T deduced as const char*
```

But functions can declare parameters that are references to arrays. If template f take its argument by reference. The type deduced for T is the actual type of the array. That type includes the size of array.
```c++
template<typename T>
void f(T& param);
f(name);  // T deduced as const char[13]. ParamType is const char (&)[13]
```

The ability to declare references to arrays enables creation of a template that deduces the number of elements that an array contains
```c++
template<typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) noexcept {
  return N;
}
```

## Function arguments
Function types can decay into function pointers, and the type deduction for array applies to type deduction for function.
```c++
void someFunc(int, double);
template<typename T>
void f1(T param);
template<typename T>
void f2(T param);
f1(someFunc); // type is void (*)(int, double)
f2(someFunc); // type is void [&](int, double)
```

# Understand auto type deduction
Auto type deduction is template type deduction. auto plays the role of T in the template, and type specifier for the variable act as *ParamType*.

There is one exception. When the initializer for a auto-declared variable is enclosed in braces, the deduced type is a std::initializer_list.
If you want to declare an int with an initial value of 27, C++98 gives you two syntactic choices
```c++
int x1 = 27;
int x2(27);
```
C++ add two choices
```c++
int x3 = {27};
int x4{27};
```
But if using auto instead of fixed types. The behavior may not what you want
```c++
// type is int
auto x1= 27;
auto x2(27);
// type is std::initializer_list<int> containing a single element instead of int
auto x3 = {27};
auto x4{27};
```

If the template passed a braced initializer, type deduction fails, and code is rejected.
```c++
template<typename T>
void f(T param);
f({11,23,9}); // error, deduction failed
```
The only real difference between auto and template type deduction is that auto assumes that a braced initializer represents a std::initializer_list, but template type deduction doesn't.

C++14 permits auto to indicate that a function's return type should be deduced, and C++14 lambdas may use auto in parameter declarations. However these uses of auto employ template type deduction, not auto type deduction.
```c++
auto createInitList() {
  return {1,2,3}; // error
}
std::vector<int> v;
auto resetV = [&v](const auto& newValue) {v = newValue};
resetV({1,2,3});  // error
```

# Understand decltype
In C++ 11, the primary use for decltype is declaring function templates where the function's return type depends on its parameter types.
```c++
//version one
template<typename Container, typename Index>
auto authAndAccess(Container &c, Index i) -> decltype(c[i]) {
  authenticateUser();
  return c[i];
}
```
In C++ 14, auto can be used to function's return type.
```c++
// version two
template<typename Container, typename Index>
auto authAndAccess(Container &c, Index i) { // In c++11, error occur during compilation. In C++14, compilation will pass
  authenticateUser();
  return c[i];
}
```
The two verions of code have small different. Because, most of containers will return a reference, T&, when calling operator[], in this case the auto type deduction is template type deduction and its return type will be T.

The correct declaration should use decltype(auto) instead of auto.
```c++
// version two
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container &c, Index i) { 
  authenticateUser();
  return c[i];
}
```
In the code above, auto specifies that the type is to be deduced, and decltype says that decltype rules should be used during the deduction.

Applying decltype to a name yields the declared type for that name. For lvalue expressions, decltype ensures that the type reported is always an lvalue reference.
```c++
decltype(auto) f1(){
  int x = 0;
  return x; // decltype(x) is int
}
decltype(auto) f1(){
  int x = 0;
  return (x); // decltype((x)) is int&
}
```

# Know how to view deduced types
nothing important

# Prefer auto to explicit type declarations
advantage of auto
1. auto variables must be initialized.
2. auto can represent types know only to compilers.
3. std::function approach is generally bigger and slower than the auto approach, and may yield out-of-memory exceptions.

disadvantage of auto
1. The type for each auto variable is deduced from its initializing expression, and some initializing expressions have types that are neither anticipated nor desired