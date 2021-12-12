---
title: Effective C++ Note
date: 2021/12/09
---
# View C++ as a federation of languages
c++ contains four main sublanguages
1. c, block, statement, preprocessor, build-in data type, arrays, pointers, etc.
2. Object-Oriented C++, class, encapsulation, inheritance, polymorphism, virtual functions, etc.
3. template c++,
4. STL 

c++ is a federation of languages, so when you switch from one sublanguage to another, programming strategy should be change too.

# Prefer consts, enums, and inlines to #defines
reason
1. the symbolic name may never be seen by compilers and may not get entered into the symbolic table, So when an error occurs, the error message may not refer to the symbolic name and you'd have no idea where the constant came from.
2. #define could result in multiple copies, because of preprocessor's blind substitution of macro name with value.
3. #define can't be used to provide any kind of encapsulation.
4. macro function has so many drawbacks, for example
  ```
  #define CALL_WITH_MAX(a,b) f((a) > (b) ? (a) : (b))
  int a = 5, b = 0;
  CALL_WITH_MAX(++a, b);  \\ a is incremented twice
  CALL_WITH_MAX(++a, b+10);  \\ a is incremented once
  ```


solution
1. replace macro with a constant. but there are two special cases
   1. defining constant pointers, should use two const. For example
      
      ```c++
      const char* const name = 'abc';
      ```
   2. class-specific constant, should may a constant as a static member. For example

      ```c++
      class GamePlayer {
        private:
          static const int NumTurns = 5;
          int scores[NumTurns];
      };
      ```
      When type is integral(integers, chars, bools) type, you can declare and use them without definition. The defination will be put in an implementation file, not a header file.
      Some older compiler may not accept the syntax above. In this case, should put the initial value at the point of definition. But when you need the value of the constant during compilation of the class, such as in the declaration above. The solution is known as "the enum hack" shown below
      ```c++
      class GamePlayer {
        private:
          enum {NumTurns = 5};
          int scores[NumTurns];
      }
      ```
2. replace macro function with inline function. 

# Use const whenever possible
## Iterator with const
```c++
const std::vector<int>::iterator iter = vec.begin();  //iter acts like a T* const. it means that data can be changed but iter is const
std::vector<int>::const_iterator iter = vec.begin();  //inter acts like a const T*. It measn that iter can be changed but data(*iter) is const
```

## constant returned values
constant returned values sometimes can reduce the incidence of client errors. For example,
```c++
class Rational {...}
const Rational operator*(const Rational& lhs, const Rational& rhs);
Rational a,b,c;

if (a*b=c) ...  // a typo. but if returned value is not constant, then it will pass compilation.
```

## const member functions
The reason for using const member functions
1. can know which functions may be invoked on const objects.
1. can know which functions may modify an object and which may not.
2. work with const object.

const function and non-const function can be overloaded, for example
```c++
class TextBlock {
  public:
    const char& operator[](std::size_t position) const;  // for const object
    char& operator[](std::size_t position);  // for non-const object
  private:
    std::string text;
};
TextBlock tb("Hello");
const TextBlock ctb("World");
std::cout << tb[0];  // fine
tb[0] = 'x';  // fine
std::cout << ctb[0];  // fine
ctb[0] = 'x';  // error
```
Notice, the return type of the non-const operator[] is a reference. If the type is a simple char, it is illegal to modify the return value of a function that returns a build-in type.

There two type of const, bitwise constness and logical constness.
C++ is designed for bitwise constness, but there is a exception. For example
```c++
class CTextBlock {
  public:
    char& operator[](std::size_t position) const;  // inappropriate declaration
  private:
    char *pText;
};
const CTextBlock cctb("World");
char *pc = &cctb[0];
*pc = 'J';  // can pass
```
If any member value may be modified and it should be valid for const. You should add a mutable modifier. For example
```c++
class CTextBlock {
  public:
    std::size_t length() const;
  private:
    char *pText;
    multable std::size_t textLength;
    multable bool lenghtIsValid;
};
```

## Avoidng Duplicatio in const and Non-const Member Functions
In TextBlock class, we overloaded operator[] function for const and non-const. If we performed bounds checking, it will yields code duplication.
The solution is below
```c++
class TextBlock {
  public:
    const char& operator[](std::size_t position) const;
    char& operator[](std::size_t position) {
      return const_cast<char&>(  // cast the const char& to non-const char&
        static_cast<const TextBlock&>(*this)[position]  // we want to call the const op[], but we can not directly do that, because it will call itself and result in infinite recursion. Instead, we should cast this object to const type(cosnt TextBlock&) so that we can call const version.
      );
    }
};
```

# Make sure that objects are initialized before they're used
C++对象的初始化规则通常比较复杂，最好的方法就是在使用之前进行初始化。
对于对象的成员，我们应该使用初始化列表进行初始化而不是在构造函数内赋值，初始化列表通常比较高效，初始化列表中的参数会作为成员构造器的参数进行对象创建，而在构造器内赋值时，会先调用成员对象的默认构造器（实际上时在进入构造体之前调用的）然后在进行赋值（构造器内实际上是赋值而不是初始化）。
当初始化列表中没有参数的时候，会调用成员对象的默认构造器，如下所示
```c++
ABEntry::ABEntry(): theName(),theAddress(),thePhones() {}
```
但是，当成员对象是引用或常量时，则必须在初始化列表中进行初始化，因为，他们不能进行赋值。
对于从文件中或数据库中获取对象值的类来说，较好的方法是使用一个私有的函数进行赋值，其他构造函数调用这个函数。而对于其他方式来说，使用初始化列表是更好的选择。

初始化列表的顺序是成员在类中声明的顺序，而不是在初始化列表中的顺序。

静态对象，包括全局对象，命名空间内定义的对象，类内的静态对象、函数内声明的静态对象以及在文件范围内声明为静态的对象，从构造后到程序结束是一直存在。函数内的静态对象成为局部静态对象，其他称为非局部静态对象。
一个翻译单元是产生单个目标文件的源代码，通常是一个源文件加上include的所有文件。
若在一个翻译单元中的非局部静态对象使用了另一个翻译单元的非局部静态对象，这个对象可能是未初始化的，因为在两个不同翻译单元的非局部静态变量的初始化顺序是未定义的。
一个解决方式是将非局部静态对象移动到函数中定义，使其成为局部的静态变量，使用函数调用获取对象的该引用，类似于单例。示例如下
```c++
//file_system.cpp
class FileSystem{...};
FileSystem& tfs() {
  static FileSystem fs;
  return fs;
}
// direcotry.cpp
class Directory{...};
Direcotry::Directory(params) {
  ...
  std::size_t disks = tfs().numDisks();
  ...
};
Directory& tempDir() {
  static Direcotry td(params);
  return td;
}
```

# Know what functions c++ silently writes and calls
For a class, if you don't declare them yourself, compilers will declare their own version of a copy constructor, a copy assignment operator, and a destructor(If they are needed). If you do not declare any constructor, compilers will also declare a default constructor.
The following two declaration is essentially the same.
```c++
// declaration one
class Empty{};
// declaration two
class Empty{
  public:
    Empty(){}
    Empty(const Emtpy& rhs){}
    ~Empty(){}

    Empty& operator=(const Empty& rhs){}
};
```

The generated destructor usually is non-vitual unless its base class declares a virtual destructor.
The copy constructor and the copy assignment operator will simply copy each non-static data member. But if a class contains a reference member or a const member, compiler won't generate a default copy assignment operator. You must define the copy assignment operator yourself.
If the copy assignment operator of the base class is private, compilers will not refuse to generator default copy assignment operator.

# Explicitly dissallow the use of compiler-generated functions you do not want.
If you do not want to support copying, you can't just simply not declaring copy constructor and copy assignment operator, beacuse compilers will generate them when needed.

One scheme is declare the copy constructor and copy assignment operator private and do not define them. For example
```c++
class HomeForSale {
  public:

  private:
    HomeForSale(const HomeForSale&);
    HomeForSale& operator=(const HomeForSale&);
}
```
The solution above has a problem: it can pass compiler and cause link-time error.

The another scheme that move the link-time error up to compile time is that declaring the copy contructor and copy assignment operator private in the base class instead of HomeForSale class. For example
```c++
class Uncopyable {
  protected:
    Uncopyable(){}
    ~Uncopyalbe(){}
  private:
    Uncopyable(const Uncopyable&);
    Uncopyable& operator=(const Uncopyable&);
};
class HomeForSale : private Uncopyable{

};
```
This work because compilers will try to generator copy constructor and copy assignment operator and these functions will try to call their base counterparts and fails.
This solution include some subtleties
1. Multiple inheritance.