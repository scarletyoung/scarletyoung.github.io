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
1. If *expr*'s type is a reference(pointer), ignore the reference(pointer) part.
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
```cpp
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
Color c = white; // error
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
For overriding to occur, serveral requirements must be met
* The base class function must be virtual.
* The base class and derived function names must be identical.
* The parameter types of the base and derived functions must be identical.
* The constness of the base and derived functions must be identical.
* The return types and exception specifications of the base and derived functions must be compatible.
* The functions's reference qualifier must be identical.(New in C++11).

Compiler may not provided warning about some of overriding problems.

In C++11, you can using keyword override to make explicit that a derived class function is supposed to override a base class version.

Using override keyword, compilers will kvetch about all the overriding-related problems.

Member function reference qualifiers make it possible to treat lvalue and rvalue object diiferently.

# Prefer const_iterators to iterators
The standard practice of using const whenever possible dictates that you should use const_iterators any time you need an iterator.

In C++11, it failed to add cbegin, cend, rbegin, rend, crbegin and crend. These functions are added in C++14;

Non-member begin returns for a const array is a pointer-to-const, and a pointer-to-const is a const_iterator for an array.

In maximally generic code, prefer non-member versions of begin, end, rbegin, etc., over their member function counterparts.

# Declare functions noexcept if they won't emit exceptions
Applying noexcept to functions premits compilers to generate better object code. In a noexcept function, optimizers need not keep the runtime stack in an unwindable state if an exception would propagate out of the function, nor must they ensure that objects in a noexcept function are destroyed in the inverse order of construction should an exception leave the function.

In C++11, move semantics can improve the performance of legacy code when move-enabled types are involved, but it may runs the risk of violating exception safety guarantee.

For example, when std::vector lacks space, we calls std::vector::push_back function to add a new element. The std::vector will allocates a new larger, chunck of memory to hold its elements, and it transfers the elements from the existing chunk of memory to the new one. 
In C++98, the transfer was accomplished by copying. In C++11, a natural optimization is to replace the copying with moves. But moves will modified the original std::vector.

Thus, std::vector::push_back takes advantage of this "move if you can, but copy if you must" strategy. Only if the move operation won't produce an exception(by checking if the operation is declared noexcept), calls to copy operations will be replace with calls to move operations.

By default, all memory deallocation functions and all destructors are implicitly noexcept. The only time a destructor is not implicitly noexcept is when a data member of the class is of a type that expressly states that its destructor may emit exceptions (e.g., declares it “noexcept(false)”).

# Use constexpr wheneve possible
constexpr indicates a value that's not only constant, it's known during compilation.

const doesn't offer the same guarantee as constexpr, because const objects need not be initialized with values know during compilation

All constexpr object are const, but not all const objects are constexpr.

For constexpr function
* constexpr functions can be used in contexts that demand compile-time constants. If the value of the arguments you pass to a constexpr function in such a context are known during compilation, the result will be computed during compilation. If any of the arguments' values is not known during compilation, your code will be rejected.
* When a constexpr function is called with one or more values that are not known during compilation, it act like a normal function.

```c++
constexpr   
int pow(int base, int exp) noexcept {}
constexpr auto numConds = 5;
std::array<int, pow(3, numConds)> results;
```
constexpr in front of pow means that if base and exp are compile-time constants, pow's result may be used as a compile-time constant.

```c++
auto base = readFromDB("base");
auto exp = readFromDB("exponent");
auto baesToExp = pow(base, exp);  // call pow function at runtime
```
In C++11, constexpr functions may contain no mroe than a single executable statement: a return. In C++14, this restrictions are substantially looser.

In C++11, all build-in types except void qualify, but user-defined types may be literal.

Declaring constexpr can migrate some computation done at runtime to compile time.

# Make const member functions thread safe
If a class contain a mutable member, this member can be modified even if object is const.

In this case, make const member functions thread safe unless you're certain they'll never used in a concurrent context.

Use of std::atomic varialbes may offer better performance than a mutex, but they're suited for manipulation of only a single variable or memory location.

# Understand special member function generation
The special member functions are the ones that C++ is willing to generate on its own. C++98 has four such functions: the default constructor, the destructor, the copy constructor and the copy assignment operator.

Generated special member functions are implicitly public and inline. They’re nonvirtual unless the function is a destructor in a derived class inheriting from a base class with a virtual destructor.

C++11 has two more special member function: the move constructor and the move assignment operator. Their signatures are
```c++
class Widget {
  public:
    Widget(Widget&& rhs);
    Widget& operator(Widget&& rhs);
};
```
Their behavior are analogous to copy siblings. But the move operaton is no guarantee that a move will actually take place. If types aren't move-enabled, copy operator will be used.

The move operations generation condition is a bit different with copy operations.
1. The two move operations are not independent. If you declare either, that prevents compilers from generating the other.
2. Furthermore, move operations won't be generated for any class that explicitly declares a copy operation.
3. Declaring a move operation in a class causes compilers to disable the copy operations.

So move operations are generated for classes (when needed) only if these three things are true:
* No copy operations are declared in the class.
* No move operations are declared in the class.
* No destructor is declared in the class.(Because if a class declares a destructor, the move operations that be automatically generated wouldn't do the right thing. If not, using default to indicate)

The copy constructor and copy assignment will be deleted if a move operation id declared.

However, member function templates never generation of special member.

# Use std::unique_ptr for exclusive-ownership resource management
std::unique_ptr is the same size as raw pointer. 
std::unique_ptr embodies exclusive ownership semantics. Moveing a std::unique_ptr transfers ownership from the source pointer to destination pointer. 
Copying a std::unique_ptr isn't allowed.

By default, destruction would take place via delete, bu during construction, std::unique_ptr objects can be configured to use custom deleters: arbitrary function to be invoked when it's time for their resources to be destroyed.

Deleter that are function pointers generally cause the size of a std::unique_ptr to grow from one word to two. The change in size depends on how much state is stored in the function object. This means that caputreless lambda expression is preferable.

The only situation I can conceive of when a std::unique_ptr<T[]> would make sense would be when you’re using a C-like API that returns a raw pointer to a heap array that you assume ownership of.

std::unique_ptr can easily and efficiently converts to a std::shared_ptr.

# Use std::shared_ptr for shared-ownership resource management
The existence of the reference count has performance implications
* std::shared_ptr is twice the size of a raw pointer.
* Memory for the reference count must by dynamicaly allocated.
* Increments and decrements of the reference count must be atomic

Moving std::shared_ptr is faster than copying them.

For std::shared_ptr, the type of the deleter is not part of the smart pointer
```c++
auto loggingDel = [] (Widget *pw) {
  makeLogEntry(pw);
  delete pw;
}
std::unique_ptr<Widget, decltype(loggingDel)> upw(new Widget, loggingDel);
std::shared_ptr<Widget> spw(new Widget, loggingDel);
```

Specifying a custom deleter doesn't change the size of a std::shared_ptr object.

std::shared_ptr contains two pointer, one for object and one for control block.
The control block contains additional data, such as reference count, a copy of the custom deleter etc.

The rules for control block creation are
* std::make_shared always creates a control block.
* A control block is created when a std::shared_ptr is constructed from a unique-ownership pointer.
* When a std::shared_ptr constructor is called with a raw pointer, it creates a control block.

According the above rules, we can know that constructing more than one std::shared_ptr from a single raw pointer will cause undefined behavior.

# Use std::weak_ptr for std::shared_ptr like pointer that can dangle
std::weak_ptr acts like a std::shared_ptr, but doesn't participate in the shared owner ship of the pointed-to resource. It's an augmentation of std::shared_ptr.

std::weak_ptr is typically created from std::shared_ptr.
```c++
auto spw = std::make_shared<Widget>();
std::weak_ptr<Widget> wpw(spw);
spw = nullptr;
```

std::weak_ptr lacks dereferencing operations, so the way to using it is like this
```c++
// version one
std::shared_ptr<Widget> spw1 = wpw.lock(); // if wpw's expired, sp1 is null;
// version two
std::shared_ptr<Widget> spw3 = (wpw); // if wpw's expired, throw std::bad_weak_ptr
```

std::weak_ptr often uses in cache, observer lists and the prevention of std::shared_ptr cycles, beacuse it can detect when they dangle.

std::weak_ptr objects are the same size as std::shared_ptr object, they make use of the same control blocks as std::shared_ptr. Operations such as construction, destruction, and assignment involve atomic reference count manipulations.

# Prefer std::make_unique and std::make_shared to direct use of new
std::make_unique is added in C++14.

The reason prefer make function is that
1. It can reduce code duplication.
  ```c++
  auto upw1(std::make_unique<Widget>());
  std::unique_ptr<Widget> upw2(new Widget);
  ```
2. make function has to do with exception safety
3. make function may improve efficiency. Using std::make_shared allows compilers to generate smaller, faster code that employs leaner data structures.
```c++
std::shared_ptr<Widget> spw(new Widget);
```
the above code may perform two allocation, one for object(through new) and one for control block of std::shared_ptr.
```c++
auto spw = std::make_shared_ptr<Widget>();
```
If std::make_shared is used instead, perform one allocation. std::make_shared will allocates a single chunck of memory to hold both the object and the control block.

using std::make_shared obviates the need for some of the bookkepping information in the control block, potentially reducing the total memory footprint for the program.

The limitation of make function is that
1. none of the make functions permit the specification of custom deleters.
2. Within the make functions, the perfect forwarding code uses parentheses, not braces. So if you want to construct your pointed-to object using a braced initializer, you must use new directly. Or create a std::initializer_list object manually.

For std::shared_ptr, there are two more limitation
1. Class defines its own version of operator new and operator delete.
2. The object type is quite large and the time between destruction of the last std::shared_ptr and the last std::weak_ptr is significant, a lag can occur between when an object is destroyed and when memory it occupied is freed.

# When using the Pimpl Idiom, define special member functions in the implementation file.
A example of using the Pimpl Idiom with smart pointer is that
```c++
// in widget.h
class Widget {
  public:
    Widget();
  private:
    struct Impl;
    std::unique_ptr<Impl> pImpl;
};
// in widget.cpp
#include "widget.h"
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl {
  std::string name;
  std::vector<double> data;
  Gadget g1, g2, g3;
};
Widget::Widget() : pImpl(std::make_unique<Impl>()) {}
```
This code compiles, but, alas, the most trivial client use doesn’t
```c++
#include "widget.h"
Widget w; // error!
```
This issue arises due to the code that's generated when w is destroyed. In the Widget definition, we didn't declare its destructor because we use std::unique_ptr to manager resource.

Thus, compiler will generate a destructor automatilly. Within that destructor, the compiler inserts code to call the destructor for Widget's data member pImpl.

pImpl is a std::unique_ptr using the default deleter. The default deleter is a function that use delete on the raw pointer inside the std::unique_ptr. Prior to using delete, implementation typically have the default deleter employ C++11's static_assert to ensure that the raw pointer doesn't point to a incomplete type.

Thus, static_assert fails. Due the function is implicitly inline. The message itself often refers to the line where w is created, because it's the source code explicitly creating the object that leads to its later implicit destruction(do not understand?)

Therefore, to fix the problem, just to make the generated-destructor to see the definition of the Impl.
```c++
// in widget.h
class Widget {
  public:
    Widget();
    ~Widget();
  private:
    struct Impl;
    std::unique_ptr<Impl> pImpl;
};
// in widget.cpp
#include "widget.h"
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl {
  std::string name;
  std::vector<double> data;
  Gadget g1, g2, g3;
};
Widget::Widget() : pImpl(std::make_unique<Impl>()) {}
Widget::~Widget() {}
```
This code compiles, but, alas, the most trivial client use doesn’t
```c++
#include "widget.h"
Widget w; // error!
```
This issue arises due to the code that's generated when w is destroyed. In the Widget definition, we didn't declare its destructor because we use std::unique_ptr to manager resource.

Thus, compiler will generate a destructor automatilly. Within that destructor, the compiler inserts code to call the destructor for Widget's data member pImpl.

pImpl is a std::unique_ptr using the default deleter. The default deleter is a function that use delete on the raw pointer inside the std::unique_ptr. Prior to using delete, implementation typically have the default deleter employ C++11's static_assert to ensure that the raw pointer doesn't point to a incomplete type.

Thus, static_assert fails. Due the function is implicitly inline. The message itself often refers to the line where w is created, because it's the source code explicitly creating the object that leads to its later implicit destruction(do not understand?)

Therefore, to fix the problem, just to make the generated-destructor to see the definition of the Impl. It means that the definition of the destructor must be in the file where Impl definition locate and after the Impl definition.

If you want to support move operator, the move operator has same requirement with destructor.

When using std::shared_ptr, there is no such problem.

The difference stems from the different way to support custom deleters. For std::unique_ptr, the type of the deleter is part of the type of the smart pointer, so pointed-to types must be complete when compiler-generated special function.

# Understand std::move and std::forward
std::move doesn't move anything. std::forward doesn't forward anything. At runtime, neither does anything at all. They generate no executable code.

std::move and std::forward are merely function that perform casts. std::move unconditionally casts its argument to an rvalue, while std::forward performs this cast only if a particular condition is fulfilled.

A simple implementation of std::move is
```c++
template<typename T>
tyename remove_reference<T>::type&&
move(T&& param) {
  using ReturnType = typename remove_reference<T>::type&&;
  return static_cast<ReturnType>(param);
}
```
remove_reference ensures that return value is rvalue.

rvalues are only usually candidates for moving. For example
```c++
class Annotation {
  public:
    explicit Annotation(const std::string text) : value(std::move(text)) {}
  private:
    std::string value;
};
```
In the code above, the text is not moved into value, it's copyied. Because text is an lvalue const std::string, and the result of the cast is an rvalue const std::string.

Moving a value out of an object generally modifies the object, so the language should not premit const objects to be passed to functions that could modify them.

There are two things that should be remember
1. Don't declare objects const if you want to able to move from them.
2. std::move doesn't move anything, it just cast a object to an rvalue.

std::forward is similar to std::move. It is a conditional cast. The condition is that it casts to an rvalue only if its argument was initialized with an rvalue.

# Distinguish universal references from rvalue references
"T&&" has two different meaning. One is rvalue reference. The other is either rvalue reference or lvalue reference. Furthermore, it can bind to virtually anything, called universal references.

Universal references arise in two contexts
1. function template parameters
2. auto declarations.
Both of them involve type deduction.

Universal reference is a reference, so it must be initialized. The initializer for a universal reference determins whether it represents an rvalue reference or an lvalue reference.

For a reference to be universal, type deduction is necessary, but it’s not sufficient.

The foundation of universal reference is an abstraction. The underlying truth is known as reference collapsing.

# Use std::move on rvalue references, std::forward on universal references
Using std::forward on rvalue reference is wordy, error-prone, and unidiomatic. Using std::move with universal reference may unexpectedly modify lvalues.
```c++
class Widget {
  public:
    template<typename T>
    void setName(T&& newName) {   // universal reference
      name = std::move(newName);
    }
  private:
    std::string name;
};
std::string getWidgetName();
Widget w;
auto n = getWidgetName();
w.setName(n); // move n into w. n's value will be unknown.
```
After the setName called, the value of n is unspecified.

One may using overloaded for const lvalues and for rvalues to avoid this problem.
```c++
class Widget {
  public:
    void setName(const std::string& newName) {
      name = newName;
    }
    void setName(std::string&& newName) {
      name = std::move(newName);
    }
};
```
This scheme has some drawbacks
1. More source code to write and maintain.
2. May be less efficient.
  ```c++
  w.setName("Adela Novak"); // In overload versions, it will create a temporary object and move to w's data member. But in universal reference version, w's member would be assigned directly from the string literal.
  ```
  In fact, replacing a template taking a universal reference with a pair of functions overloaded on lvalue references and rvalue references is likely to incur a runtime cost in some cases.
3. Some functions take an unlimited number of parameters, each of which could be an lvalue or rvalue.

If you're in a function that returns by value, and you're returning an object bound to an rvalue reference or a universal reference, you'll want to apply std::move or std::forward when you return the reference.

There's nothing to be lost by applying std::move to rvalue reference being returned from functions that return by value.The situation is similar for universal references and std::forward.

According to the rule above, someone may perform the same opimization on local variables. 
```c++
// original version
Widget makeWidget() {
  Widget w;
  ...
  return w;
}
// "optimize" version
Widget makeWidget() {
  Widget w;
  ...
  return std::move(w);
}
```
The "optimize" version has flaw.

The compiler will perform the same optimization called RVO(return value optimization). However, applying RVO has conditions
1. the type of the local object is the same as that returned by the function.
2. the local object is what's being returned.

Never apply std::move or std::forward to local objects if they would otherwise be eligible for the return value optimization.

# Avoid overloading on universal references
Suppose we write a function
```c++
std::multiset<std::string> names;
void logAndAdd(const std::string &name) {
  auto now = std::chrono::system_clock::now();
  log(now, "logAndAdd");
  names.emplace(name);
}
std::string petName("Darla");
logAndAdd(petName);
logAndAdd(std::string("Persephone"));
logAndAdd("Patty Dog");
```
For the second and third calls, the implementation of the function is inefficiency, because the parameter is an rvalue.

We can eliminate the inefficiency by rewriting logAndAdd to take a universal reference
```c++
template<typename T>
void logAndAdd(T&& name) {
  auto now = std::chrono::system_clock::now();
  log(now, "logAndAdd");
  names.emplace(std::forward<T>(name));
}
```

Suppose we have a overload function taking a int parameter.
```c++
std::string nameFromIdx(int idx);
void logAndAdd(int idx) {
  auto now = std::chrono::system_clock::now();
  log(now, "logAndAdd");
  names.emplace(nameFromIdx(idx));
}
short nameIdx;
logAndAdd(nameIdx);  // error!
```

There are two logAndAdd overloads. The one taking a universal reference can deduce T to be short, thus yielding an exact match. The overload with an int parameter can match the short argument only with a promotion.

According to the overload resolution rules, an exact match beats a match with a promotion, so the universal reference overload is invoked.

Suppose we write a class with constructor that do the same thing.
```c++
class Person {
  public:
    template<typename T>
    explicit Person(T&& n) : name(std::forward<T>(n)) {}
    explicit Person(int idx) : name(nameFromIdx(idx)){}
    Person(const Person& rhs);  // compiler-generated
    Person(Person&& rhs); // compiler-generated
  private:
    std::string name;
};
```
We know that C++ will generate both copy and move constructors even if the class contains a templatized constructor that could be instantiated to produce the signature of the copy or move constructor.

If we're trying to create a Person from another Person. We can use copy constructor like this.
```c++
Person p("Nancy");
auto cloneOfP(p);
```

But this code won't call the copy constructor. It will call the perfect-forwarding constructor and fail to compile.

The reason is as follow. cloneOfP is being initialized with a non-const lvalue. The templatized constructor can be instantiated to take non-const lvalue of type Person.

p could be passed to either the copy constructor or the instantiated tempalte. Calling copy constructor would require adding const to p to match the copy constructor's parameter's type, but calling the instantiated template requires no such addition. So the latter is better and will be call.

The interaction among perfect-forwarding constructors and compiler-generated copy and move operations develops even more wrinkles when inheritance enters the picture.
```c++
class SpecialPerson: public Person {
  public:
    SpecialPerson(const SpecialPerson& rhs) : Person(rhs) {}  // copy constructor; call base class forwarding constructor
    SpecialPerson(SpecialPerson& rhs) : Person(rhs) {}  // move constructor; call base class forwarding constructor
};
```
The reason is that the type of arguments to pass to base class is SpecialPerson.

In summary, overloading on universal reference is a bad idea.

The solution in next item

# Familiarize yourself with alternatives to overloading on universal references
## Abandon overloading
The first example can avoid overloading by using different names of the would-be overloads. But this doesn't work for constructor.

## Pass by const T&
Replace pass-by-universal-reference with pass-by-lvalue-reference-to-const. The drawback is to lose efficient.

## Pass by value
Replace pass-by-reference parameters with pass by value will dial up performance.

## Use Tag dispatch
The idea behind the tag dispatch is that a universal reference parameter generally provides an exact match for whatever's passed in, but if the universal reference is part of a parameter list containing other parameter that are not universal references, sufficiently poor matches on the non-universal reference parameters can knock an overload with a universal reference out of the running.
```c++
template<typename T>
void logAndAdd(T&& name) {
  logAndAddImpl(std::forward<T>(name), std::is_integral<typename std::remove_reference<T>::type>());
}
template<typename T>
void logAndAddImpl(T&& name, std::fales_type) {
  auto now = std::chrono::system_clock::now();
  log(now, "logAndAdd");
  names.emplace(std::forward<T>(name));
}
void logAndAddImpl(int idex, std::true_type) {
  logAndAdd(nameFromIdx(idx));
}
```
The type std::true_type and std::false_type are tags whose only purpose is to force overload resolution to go the way we want.

This is a standard building block of template metaprogramming.

The tag that determines which overload gets called. The tag values are designed so that no more than one overload will be a viable match.

## Constraining template that take universal references
std::enable_if gives you a way to force compilers to behave as if a particular template didn't exist. A template using std::enable_if is enabled only if the condition specified by std::enable_if is satisfied.
```c++
class Person {
  public:
    template<typename T, typename=typename std::enable_if<condition>::type>
    explicit Person(T&& n);
};
```
The condtion we want to specify is that T isn's Person.
We need to ignore reference, consts and volatile from T before checking to see if that type is the same as Person.

Thus, the condition is that
```c++
!std::is_same<Person, typename std::decay<T>::type>::value
```

There is still one situation needed to process, derivation.

What we really want is that any argument type other than Person or a type derived from Person is enable.
In this case, the condition is 
```c++
!std::is_base_of<Person, typename std::decay<T>::type>::value
```

The condition can be more complex. For example, we want to further constrain the templatized constructor so that it's disabled for integral arguments.
```c++
!std::is_base_of<Person, std::decay<T>::type>::value && !std::is_integral<std::remove_reference<T>::type>::value
```

## Trade-off
Perfect forwarding is more efficient, but it has drawbacks.


# Understand reference collapsing
In C++ a reference to a reference is illegal. But compiler may produce them in particular contexts. When compilers generate reference to references, reference collapsing dictates what happens next.

The references collapse rule is that if either reference is an lvalue reference, the result is an lvalue reference. Otherwise the result is an rvalue reference.

Reference collapsing is a key part of what makes std::forward work. A simple but not correct implementation of std::forward is that
```c++
template<typename T>
T&& forward(typename remove_reference<T>::type& param) {
  return static_cast<T&&>(param);
}
```

Reference collapsing occurs in four contexts
1. template instantiation.
2. type generation for auto variables.
3. the generation and use of typedefs and alia declarations.
4. uses of decltype.


A universal reference isn't a new kind of reference, it's actually an rvalue reference in a context where two conditions are satisfied
1. Type deduction distinguishes lvalue from rvalue. Lvalues of type T are deduced to have type T&, while rvalues of type T yield T as their deduced type.
2. Reference collapsing occurs.

# Assume that move operations are not present, not cheap, and not used.
Both moving and copying a std::array have linear-time computational complexity, because each element in the container must be copied or moved.

std::string offer constant-time moves an linear-time copies. But many string implementations employ the small string optimization(SSO). Small strings are stored in a buffer within the std::string object, so, moving small strings using an SSO-based implementation is no faster than copying them.

There are serveral scenarios in which C++11's move semantics do you no good.
* No move operation. The move request will become a copy request.
* Move not faster.
* Move not usable. The moving would take place requires a move operation that emits no exceptions, but that operation isn't declared noexcept.

This scenario where move semantics offers no efficiency gain
* Source object is lvalue: with few exceptions, only rvalues may be used as the source of a move operation.

# Familiarize yourself with perfect forwarding failurec cases
When it comes to general-pupose forwarding, we'll be dealing with parameters that are references.

Perfect forwarding means we don't just forward objects, we also forward their salient characteristics: their types, whether they’re lvalues or rvalues, and whether they’re const or volatile.

Several kinds of arguments lead to this kind of failure.

## Braced initializers
The use of a braced initializer is a perfrect forwarding failure case.

In direct call to f, compilers see the arguments passed at the call site, and they see the types of the parameters declared by f. They compare the arguments at the call site to the parameter declarations to see if they're compatible, and if necessary, they perform implicit conversions to make the call succeed.

When calling f indirectly through the forwarding function template fwd, compilers no longer compare the arguments passed at fwd's call site to the parameter declaration in f. They deduce the types of the arguments being passed to fwd, and the compare the deduced types to f's parameter declaration. Perfect forwarding fails when either of the following occurs.
* Compilers are unable to deduce a type for one or more of fwd's parameters.
* Compilers deduce the wrong type for one or more of fwd's parameters.

When passing a braced initializer to a function template parameter, becuase fwd's parameter isn't declared to be a std::initializer_list, this means that compilers are forbidden from deducing a type for the braced initializer expression.
???
## 0 or NULL as null pointers
Because when you try to pass 0 or NULL as a null pointer to a template, compiler will deduce a integral type instead of a pointer type, this result is that neither 0 nor NULL can be perfect-forwarded as a null pointer.

## Declaration-only integral static const data members
There is no need to define integral static const data members in class. That's because compilers perform const propagation on such members' value, thus eliminating the need to se aside memory for them.

If the static const data members' address were to be taken, then them would require storage, and unitl a definition for them was provided, the code would fail at link-time.

Passing integral static const data members by reference generally requires that they be defined, and that requirement can cause code using perfect forwarding to fail where the equivalent code without perfectwarding success.

## Overloaded function names and template names
Suppose we have a function f taking a function pointer as parameter and an overloaded function, processVal.
```c++
void f(int (*pf)(int));

int processVal(int value);
int processVal(int value, int priority);

f(processVal);  //fine
```
The code above is fine because compilers know which processValue they need through matching f's parameter type.

However, fwd doesn't have any information about what type it needs, and that makes it impossible for compilers to determine which overload should be passed.

The same problem arises if we try to use a function template instead of an overloaded function name. Because a function template represents many functions.

The solution is to manually specify the overload or instantiation you want to have forwarded.
```c++
using ProcessFuncType = int(*)(int);
ProcessFuncType processValPtr = processVal;
fwd(processValPtr);
fwd(static_cast<ProcessFuncType>(workOnVal));
```

## Bitfields
When a bitfield is used as a function argument, the perfect forwarding will fail.

The reason is that bitfields may consist of arbitrary parts of machine words, but there's no way to create a pointer to arbitrary bits, and there's no way to bind a reference to arbitrary bits.

Any function that accepts a bitfield as an argument will receive a copy of the bitfield’s value. The only kinds of parameters to which a bitfield can be passed are by-value parameters and references-to-const. 

In the case of a reference-to-const parameter, the Standard requires that the reference actually bind to a copy of the bitfield’s value that’s stored in an object of some standard integral type (e.g., int). References-to-const don’t bind to bitfields, they bind to “normal” objects into which the values of the bitfields have been copied.

The key to passing a bitfield into a perfect-forwarding function is to take advantage of the fact that the forwarded-to function will always receive a copy of the bitfield’s value. You can thus make a copy yourself and call the forwarding function with the copy.

# Avoid default capture modes
There are two default capture modes in C++11: by-reference and by-value. 

Default by-reference capture can lead to dangling references. Default by-value capture lures you into thinking you're immune to that problem, and it lulls you into thinking your closures are self-contained.

A by-reference capture causes a closure to contain a reference to a lcoal variable or to a parameter that's available in the scope where the lambda is defined. If the lifetime of a closure created from that lambda exceeds the lifetime of the local varialbe or parameter, the reference in the closure will dangle. For example,
```c++
using FilterContainer = std::vector<std::function<bool(int)>>;
FilterContainer filters;

void addDivisorFilter() {
  auto calc1 = computeSomeValue1();
  auto calc2 = compuateSomeValue2();
  auto divisor = computeDivisor(calc1, calc2);
  filter.emplace_back([&] (int value) {return value % divisor == 0;});
}
```
If you know that a closure will be used immediately and won’t be copied, there is no risk that references it holds will outlive the local variables and parameters in the environment where its lambda is created.

To solve this problem is a default by-value capture.
```c++
filters.emplace_back([=] (int value) {return value % divisor == 0});
```

But if you capture a pointer by value, you copy the pointer into the colsures arising from the lambda, but you don't prevent code outside the lambda from deleteing the pointer and causing your copies to dangle. For example
```c++
class Widget {
  public:
    void addFilter() const {
      filters.emplace_back([=] (int value) {return value % divisor == 0;});
    }
  private:
    int divisor;
}
```
Maybe you think divisor will be passed by-value into the lambda. It's wrong. Captures apply only to non-static local variables visible in the scope where the lambda is created.

So the real code shoulde be
```c++
void Widget::addFilter() const {
  auto currentObjectPtr = this;
  filters.emplace_back([currentObjectPtr] (int value) {return value % currentObjectPtr->divisor == 0;});
}
```
A raw pointer this is captured by-value, and when object is deleted, dangling pointer occur.

This problem can be solved by making a local copy of the data member
```c++
void Widget::addFilter() const {
  auto divisorCopy = divisor;
  filters.emplace_back([divisorCopy] (int value) {return value % divisorCopy == 0;});
}
```

In C++14, a better way to capture a data member is to use generalized lambda capture
```c++
void Widget::addFilter() const {
  filter.emplace_back([divisor=divisor] (int value) {return value % divisor==0;})
}
```

An additional drawback to default by-value captures is that they can suggest that the corresponding closures are self-contained and insulated from changes to data outside the closures.
But lambda maybe dependent on objects with static storage duration. These objects can be used inside lambda, but they can't be capture. Default by-value caputer may mislead that these objects are be capture by value.

# Use init capture to move objects into closures
In C++11, you can not move an object into a closure. This supports is added in C++14, called init capture.

Using an init capture makes it possible for you to specify
1. the name of a data member in the closure class generated from the lambda
2. an expression initializing that data member.

```c++
auto pw = std;:make_unique<Widget>();
auto func [pw=std::move(pw)] {return pw->isValidated() && pw->isArchived();}; // init capture
```
The scope on the left of the "=" is different from the scope on the right. The scope on the left is that of the closure class. The scope on the right is the same as where the lambda is being defined.

The code above can be written in C++11 like this
```c++
class IsValAndArch {
  public:
    using DataType = std::unique_ptr<Widget>;
    explicit IsValAndArch(DataType &&ptr) :pw(std::move(ptr)) {}
    bool operator()() const {
      return pw->isValidated() && pw->isArchived();
    }
  private:
    DataType pw;
};
```
If you want to stick with lambdas, move capture can be emulated in C++11 by
1. moving the object to be captured into a function object produced by std::bind
2. giving the lambda a reference to the captured object

```c++
std::vector<double> data;
// c++14 version
auto func = [data = std::move(data)] {...}
// c++11 version
auto func = std::bind([] (const std::vector<double>& data){}, std::move(data));
```

To prevent that copy of data from being modified inside the lambda, the lambda's parameter is declared reference-to-const. But, if the lambda were declared mutable, operator() in its closure class would not be declared const.

Some fundamental points should be clear
1. It's not possible to move-construct an object into a C++11 closure, but it is possible to move-construct an object into a C++11 bind object.
2. Emulating move-capture in C++11 consists of move-constructing an object into a bind object, then passing the move-constructed object to the lambda by reference.
3. Because the lifetime of the bind object is the same as that of the closure, it's possible to treat objects in the bind object as if they were in the closure.

# Use decltype on auto&& parameters to std::forward them
In C++14, lambda can use auto in their parameter specifications. Because operator() in lambda's closure class is template. For example
```c++
auto f = [] (auto x) {return func(normalized(x));};
```
The closure class's function call operator looks like this
```c++
class CompilerGeneratedClassName {
  public:
    template<typename T>
    auto operator()(T x) const {
      return func(normalized(x));
    }
};
```
In the code above, if the normalized treads lvalue different from rvalues, this lambda isn't written properly, because it always pass an lvalue to normalized.

The correct way is to have lambda perfect-forward x to normalized.

For using std::forward, we must know the type of x. We can use decltype to get the type we need.
```c++
auto f = [] (auto&& param) {
  return func(normalized(std::forward<decltype(param)>(param)));
};
```

# Prefer lambdas to std::bind
The reason is
1. lambdas are more readable.
2. compilers have no way to determine which of the overloaded cuntion they should pass to std::bind. Function name must be cast to the proper function pointer type.
3. using labmdas generates faster code that using std::bind. Because lambda can inline function while std::bind use function pointer that are less possible to inline.

In C++14, lambda is always better. However, in C++11, std::bind can be justified in two constrained situation
1. Move capture
2. Polymorphic function object

# Prefer task-based programming to thread-based
You have two basic choice to run a function doAsyncWork asynchronously.
1. thread-based approach
  ```c++
  int doAsyncWork();
  std::thread t(doAsyncWork);
  ```
2. task-based approach
  ```c++
  auto fut = std::async(doAsyncWork);
  ```

The task-based approach is typically superior to its thread-based counterpart.

The first reason is that doAsyncWork produces a return value, and with the thread-based invocation, there's no straightforward way to get access to it.

The second reason is that if the system is oversubscribed or is out of threads, the call of std::async doesn't guarantee that it will create a new software thread. It will permits the scheduler to arragne for the specified function to be run on the thread requesting the result. This can prevent oversubscription and more efficient.

Compared to thread-based programming, a task-based design spares you the travails of manual thread management, and it provides a natural way to examine the results of asynchronously executed functions.

There are some situation where using threads directly may appropriate
* You need access to the API of the underlying threading implementation. Because std::thread objects offer native_handle member function.
* You need to and are able to optimize thread usage for you application.
* You need to implement threading technology beyond the C++ concurrency API.

# Specify std::launch::async if asynchronicity is essential
There are two standard policies for std::async to execute a function f.
* std::launch::async launch policy, f must be run asynchronously.
* std::launch::deferred launch policy, f may run only when get or wait is called on the future returned by std::async. When get or wait is invoked, f will execute synchronously. If neither get nor wait is called, f will never run.

std::async's default launch policy is neither of these. it's these or-ed together. It means that the following two calls have exactly the same meaning
```c++
auto fut1 = std::async(f);
auto fut2 = std::async(std::launch::async | std::launch::deferred, f);
```

Given a thread executing std::async statement
* It's not possible to predict whether f will run concurrently with t.
* It's not possible to predict whether f runs on a thread different from the thread invoking get or wiat on fut.
* It may not be possible to predict whether f runs at all.

The default launch policy has several drawbacks
* If f reads or writes such thread-local storage, it's not possible to predict which thread's variables will be access
* It will affect wait-based loops using timeouts, because calling wait_for or wait untile on a task that's deferred yields the value std::launch::deferred
  ```c++
  void f() {
    std::this_thread::sleep_for(1s);
  }
  auto fut = std::async(f);
  while(fut.wait_for(100ms) 1= std::future_status::ready) { // may run forever
    // ...
  }
  ```

Using std::async with the default launch policy for a task is fine as long as the following condition are fulfilled
* The task need not run concurrent with the thread calling get or wait
* It doesn't matter which thread's thread_local variables are read or written.
* Either there's a guarantee that get or wait will be call on the future returned by std::async or it's acceptable that the task may neven execute.
* Code using wait_for or wait_until task the possibility of deferred status into account.

# Make std::threads unjoinable on all paths
Every std::thread object is in one of two states: joinable or unjoinable. A joinable std::thread corresponds to an underlying asynchronous thread of execution that is or could be running. An unjoinable is a std::thread that's not joinable.

Unjoinable std::thread object includes
* Default-constructed std::threads
* std::thread objects that have been moved from
* std::thread that has been joined
* std::thread that has been detached

std::thread's joinability is important because if the destructor for a joinalbe thread is invoked, execution of the program is terminated.

The reasons for this behavior of std::thread destructor is that the two other obvious options are worse
* join-on-destructor can lead to difficult-to-debug performance anomalies
* detach-on-destruction can lead to difficult-to-debug undefined behavior.

Any time you want to perform some action along every path out of a block, the normal approach is to put that action in the destructor of a local object. Such objects are known as RAII objects, and the classes they come from are known as RAII classes.

A simple ThreadRAII object is like that
```c++
class ThreadRAII {
public:
  enum class DtorAction{join, detach};

  ThreadRAII(std::thread&& t, DtorAction a) : action(a), t(std::move(t)) {}

  ~ThreadRAII() {
    if (j.joinable()) {
      if (action == DtorAction::join) {
        t.join();
      } else {
        t.detach();
      }
    }
  }

  std::thread& get() {return t;}
private:
  DtorAction actoin;
  std::thread t;
};
```

Declare std::thread objects last in lists of data members.

# Be aware of varying thread handle destructor behavior
Future destructors normally just destroy the future’s data members.

The final future referring to a shared state for a non-deferred task launched via std::async blocks until the task completes.
```
// TODO
```

# Consider void futures for one-shot event communication
For simple event communication, condvar-based designs require a superfluous mutex, impose constraints on the relative progress of detecting and reacting tasks, and require reacting tasks to verify that the event has taken place.

Designs employing a flag avoid those problems, but are based on polling, not blocking.

A condvar and flag can be used together, but the resulting communications mechanism is somewhat stilted.

Using std::promises and futures dodges these issues, but the approach uses heap memory for shared states, and it’s limited to one-shot communication.

```
// TODO
```

# Use std::atomic for concurrency, volatile for special memory.
std::atomic is for data accessed from multiple threads without using mutexes. It’s a tool for writing concurrent software.

volatile is for memory where reads and writes should not be optimized away. It’s a tool for working with special memory.

```
// TODO
```

copy operation for std::atomic are deleted.


std::atomic is useful for concurrent programming, but not for accessing special memory.
volatile is useful for accessing special memory, but not for concurrent programming.

# Consider pass by value for copyable parameters that are cheap to move and always copied.
A member function addName might copy its parameter into a private container. For efficiency, such a function should copy lvalue arugments, but move rvalue arguments
```c++
class Widget {
  public:
    void addName(const std::string& newName) {names.push_back(newName);}
    void addName(std::string&& newName) {names.push_back(std::move(newName));}
  private:
    std::vector<std::string> names;
};
```
There are two functions should be declared, implemented and maintained.

An alternative is to make addName a function template taking a universal reference
```c++
class Widget {
  public:
    template<typename T>
    void addName(T&& newName) {
      names.push_back(std::forward<T>(newName));
    }
};
```
The use of universal reference has serveral problem
1. Implementation must be in a header file
2. may yield several function in object code.
3. Instatiates differently for std::string and types that are convertiable to std::string
4. Improper argument types may be passed.

To avoid the problems avove, a better way is pass parameters by value.
```c++
class Widget {
  public:
    void addName(std::string newName) {
      names.push_back(std::move(newName));
    }
};
```

For the efficiency, in C++98, passing by value may less effecitive. But in c++11, addName will be copy constructed only for lvalues. For rvalues, it will be move constructed.

For copyable, cheap-to-move parameters that are always copied, pass by value may be nearly as efficient as pass by reference, it’s easier to implement, and it can generate less object code.

Copying parameters via construction may be significantly more expensive than copying them via assignment.

Pass by value is subject to the slicing problem, so it’s typically inappropriate for base class parameter types.

# Consider emplacement instead of insertion
Insertion functions take objects to be inserted, while emplacement functions take constructor arguments for objects to be inserted.

In theory, emplacement function can do everything insertfunction can, and they sometimes do it more efficiently and they never do it less efficiently.

But in pratice, there are situations where the insertion functions run faster.

If all the following are true, emplacement will almost certainly out‐ perform insertion:
* The value being added is constructed into the container, not assigned. Node-based containers virtually always use construction to add new values, and most standard containers are node-based. The only ones that aren't are std::vector, std::deque, and std::string.
* The argument types being passed differ from the type held by container.
* The container is unlikely to reject the new value as a deplicate. In order to detect whether a value is already in the container, emplacement implementations typically create a node with the new value so that they can compare the value of this node with existing container nodes.

When deciding whether to use emplacement functions, two other issues are worth keeping in mind. The first regards resource management. The second noteworthy aspect of emplacement functions is their interaction with explicit constructors

Emplacement functions use direct initialization, which means they may use explicit constructors. Insertion functions employ copy initialization.

Emplacement functions may perform type conversions that would be rejected by insertion functions.