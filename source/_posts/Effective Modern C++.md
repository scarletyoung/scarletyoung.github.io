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

# Use the explicitly typed initializer idiom when auto deduces undesired types
```c++
std::vector<bool> features(const Widget& w);
Widget w;
auto highPriority = features(w)[5];
processWidget(w, highPriority); // undefined behavior
```
In the code using auto, the type of highPriority is no longer bool. Because operator[] for std::vector<bool> return an object of type std::vector<bool>::reference instead of bool&.

std::vector<bool>::reference exists because std::vector<bool> is specified to represent its bools in packed form, one bit per bool, but C++ forbids references to bits. Therefore, operator[] for std::vector<bool> returns an object that acts like a bool& and can implicit conversion to bool.

In the code above, the type of highPriority is deduced to std::vector<bool>::reference. The value depends on how std::vector<bool>::reference is implemented. One implementation is an object contain a pointer and an offset.

The call to features returns a temporary std::vector<bool> object. The operator[] is invoked on this temporary vector, and return a object manager by this temporary vector.

At the end of statement, the temporary is destroyed. Therefore highPriority contains a dangling pointer.

std::vector<bool>::reference is an example of a porxy class. Proxy classes are employed for a variety of purpose. Some proxy class is "invisible", and this type of classes don't play well with auto.

We can recognize proxy objects through library document and header file.

When auto meets proxy objects, auto isn't deducing the type you want it to deduce. The solution is to force different type deduction, i.e., using explicity typed initializer idiom. For example
```c++
auto highPriority = static_cast<bool>(features(w)[5]);
```

# Distinguish between () and {} when creating objects
Initialization values may be specified with parentheses, an euqal sign or braces
```c++
int a(0);
int b = 0;
int c{0};
int d = {0};  // same as using braces only
```
Braced initialization can specify the initial content of a container and specify default initialization values for non-static data members
```c++
class Widget {
  private:
    int x{0};   // pass
    int y = 0;  // pass
    int z(0);   // error
}
```

Uncopyable objects may be initialized using braces or parentheses, but not using "=";
```c++
std::atomic<int> ai1{0};  // pass
std::atomic<int> ai2(0);  // pass
std::atomic<int> ai3 = 0; // error
```
Therefore, braced initialization is called uniform initialization.

A feature of braced initialization is that it prohibits implicit narrowing conersions among build-in types.
```c++
double x,y,z;
int sum1{x+y+z};  // error
int sum2(x+y+z);  // pass
int sum3 = x+y+z; // pass
```

C++ has a rule that anything that can be parsed as a declaration must be interpreted as one. But braced initialization is immunity to it.
```c++
Widget w1(10);  // call widget constructor with argument 10;
Widget w2();    // not calling widget constructor with not argument, instead of declaring a function named w2 that returns a Widget. 
Widget w3{};    // call widget constructor with not argument
```

In [Item 2](#understand-auto-type-deduction), we have known that the type of auto-declared variable with braced initializer is std::initializer_list. And if one or more constructors declare a parameter of type std::initizlizer_list, calls using the braced initialization syntax *strongly* prefer the overloads taking std::initializer_list.
```c++
class Widget {
  public:
    Widget(int i, bool);
    Widget(int i, double d);
    Widget(std::initializer_list<long double> il);
};
Widget w1(10, true);  // calls first constructor
Widget w2{10, true};  // calls third constructor
Widget w3(10, 0.5);   // calls second constructor
Widget w4{10, 0.5};   // calls third constructor
```
Even if the best-match std::initializer_list constructor can't be called,  compilers will ignore other constructors.
```c++
class Widget {
  public:
    Widget(int i, bool);
    Widget(int i, double d);
    Widget(std::initializer_list<bool> il);
};
Widget w1{10, true};  // calls third constructor but error
```
Only if there’s no way to convert the types of the arguments in a braced initializer to the type in a std::initializer_list do compilers fall back on normal overload resolution
```c++
class Widget {
  public:
    Widget(int i, bool);
    Widget(int i, double d);
    Widget(std::initializer_list<std::string> il);
};
Widget w1{10, true};  // calls first constructor but error
```

But there is an edge case. When using an empty set of braces to construct an object that supports default construction and also supports std::initializer_list construction. Default construction will be called.

# Prefer nullptr to 0 and NULL
0 is an int, not a pointer; NULL is given an integral type other than int. It's that neither 0 nor NULL has a pointer type. The problem is that overloading on pointer and integral types count lead to surprises. Passing 0 or Null to such overloads never alled a pointer overload
```c++
void f(int);
void f(bool);
void f(void*);
f(0); // call fi(int)
f(NULL);  // might not compile, but typically calls f(int). Never calls f(void*)
```
The uncertainty behavior of NULL is due to the implementations of the type of NULL. If NULL is defined to be 0L, the call is ambiguous, because conversion from long to int, long to bool, and 0L to void* are considered equally good.

nullptr doesn't have an integral type. Its type is std::nullptr_t. The type std::nullptr_t implicitly converts to all ray pointer types, and that's what makes nullptr act as if it were a pointer of all types. Calling the overloaded function f with nullptr calls the void* overload

Using nullptr instead of 0 or NULL thus avoids overload resolution surprises.

Using nullptr can also improve code clarity, especially when auto variables are involed.
```c++
auto result = findRecore();
if (result == 0) {  // do not know whether result is a pointer type or an integral type.

}
```

Template type deduction deduces the wrong types for 0 and NULL. Their deduction type is their rule type instead of null pointer.

# Prefer alias declarations to typedefs
```c++
typedef std::unique_ptr<std::unorderd_map<std::string, std::string>> UPtrMapSS; // in C++ 98
using UPtrMapSS = std::unique_ptr<std::unorderd_map<std::string, std::string>>; // in C++ 11
```

The different between typedef and alias declarations is in template. Alias declarations may be templatized, while typedef cannot.

Alias templates avoid the "::type" suffix and, in templates, the "typename" prefix often required to refer to typedef.

C++14 offers alias templates for all the C++11 type traits transformations.

# Prefer scoped enums to unscoped enums
The C++98-style enumerator names will leak into the scope containing their enum definition, called unscoped enums.
```c++
enum Color {black, white, red};
auto white = false; // error
```
Their new C++11 counterparts, scoped enums, don't leak names in this way
```c++
enum class Color {black, white, red};
auto white false; // fine
Color c = whilte; // error
Color c = Color::white; // fine
```

Another advantage of scoped enum is that their enumerators are much more strongly typed.

uscoped enums implicitly convert to integral types. However, there are not implicit conversions from enumerators in a scoped enum to any other type. If you want to perform a conversion, you can use a cast
```c++
enum class Color {black, white, red};
Color c = Color::white;
double d = static_cast<double>(c);
```

The third advantage of scoped enums is that C++98 only supports only enum definitions.

By default, the underlying type for scoped enums is int. If the default doesn't suit your, you can override it
```c++
enum class Status: std::unit32_t;
```

You can also specify the underlying type for an unscoped enum, and the result may be forward-declared.
```c++
enum Color: std::unit8_t;
```

# Prefer deleted functions to private underfined ones.
Deleted functions may not be used in any wayt, enve in member and friend function.

An important advantage of deleted functions is that any function may be deleted, while only member functions may be private.
```c++
bool isLucky(int number);
isLucky('a'); //fine
isLucky(true);  //fine
isLucky(3.5); // fine
```
If isLucky parameter must be integers, and we'd like to prevent calls such as these from compiling. We cans use deleted functions.
```c++
bool isLucky(int number);
bool isLucky(char) = delete;
bool isLucky(bool) = delete;
bool isLucky(double) = delete;  // reject both double and float
```
Although deleted functions can't be used, they are part of program. Thus, they are taken into account during overload resolution.

Another trick that deleted functions can perform is to prevent use of template instantiations that should be disabled.
```c++
template<typename T>
void processPointer(T* ptr);
template<>
void processPointer<void>(void*) = delete;
template<>
void processPointer<char>(char*) = delete;
```
This can not be achieved by declaring them private. Because it's not possible to give member function template specialization a different access level from that of the main template.

# Declare overriding functions override