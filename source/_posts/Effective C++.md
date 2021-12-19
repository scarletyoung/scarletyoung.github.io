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

# Declare destructor virtual in polymorphic base classes
In C++, when a derived class object is deleted through a pointer to a base class with a non-virtual destructor, result are undefined. The derived part of the object may never be destroyed at the runtime, thus leading to a curious partially destroyed object.

The solution is that give the base class a virtual destructor. Any class with virtual functions should almost certainly have a virtual destructor, Because if a class does not contain virtual functions, it is not meant to be used as a base class.

When a class is not intended to be a base class, making the destructor virtual is ususally a bad idea. Assume we have a class below
```C++
class Point {
  public:
    Point(int xCoord, int yCoord);
    ~Point();
  private:
    int x, y;
};
```
If an int occupies 32 bits, then a Point object is 64 bits. Furthermore, such a Point object can be pass as a 64-bit quantity to functions written in other languages, such as C.

But if Point's desctructor is virtual, the object will contain a additional pointer called vptr(virtual table pointer). The Point objects will increase in size. As a result, it is no longer possible to pass Points to and from functions written in other languages.

You must provide a definition for the pure virtual destrcutor. Compiler will generate a call to base class desctructor from its derived classes' destructors. If you don't, the linker will complain.

This rule applies only to polymorphic base classes.

# Prevent exceptions from leaving desctructors
Emitting exceptions from destructors is not a good idea. 
Consider below the class
```c++
class Widget {
  public:
    ~Widget() {...}
};
void doSomething() {
  std::vector<Widget> v;
  ...
}
```
When the vector v is destroyed, it will destroying all the Widgets in it. Assume an exception is thrown during destruction of the first one. The other Widgets still have to be destroyed. So, if another destructor is called and throws an exception, there are two simultaneously active exceptions. In this situation, program execution either terminates or yields underfined behavior.

There are two solutions
1. Terminate the program.
   ```c++
   DBConn::~DBConn() {
     try {
       db.close();
     } catch(...) {
       ...
       std::abort();
     }
   }
   ```
2. swallow the exception. When choosing this scheme, the program must be able to reliably continue execuftion even after an error has been encountered and ignored.
   ```c++
   DBConn::~DBConn() {
     try {
       db.close();
     } catch(...){
       ...
     }
   }
   ```

The two solutions above have a disadvantage. The program has not opportunity to react to the problems that my arise.
A better strategy is shown as below
```c++
class DBConn {
  public:
    void close() {
      db.close();
      closed = true;
    }
    ~DBConn() {
      if (!closed) {
        try {
          db.close();
        } catch(...) {...}
      }
    }
};
```

# Never call virtual functions during construction or desctuction
Let's see the code below
```c++
class Transaction {
  public:
    Transaction() {
      logTransaction();
    }
    virtual void logTransaction() const = 0;
}
class BuyTransaction: public Transaction {
  public:
    virtual void logTransaction() const;
}
class SellTransaction: public Transaction {
  public:
    virtual void logTransaction() const;
}
```
When you create a derived class object, the base class constructor will be call before the derived clss constructor. If you call virtual fucntions in the base class constructor, it will not call the derived class version, because derived class data members have not been initialized.

Actually, during base class constructor of a derived class object, the type of the object is that of the base class.

The same reasoning applies during destruction.

The solution is that to turn logTransaction into a non-virutal function, then require that derived class constructors pass the necessary log information to the Transaction constructor.
   ```c++
   class Transaction {
     public:
      explicit Transaction(const std::string& logInfo) {
        logTransaction(logInfo);
      }
      void logTransaction(const std;:string& logInfo) const;
   };
   class BuyTransaction: public Transaction {
     public:
      BuyTransaction(parameters):Transaction(createLogString(parameters)) {...}
     private:
      static std::string createLogString(parameters);  // make the functions static in case of referring to uninitialized datamembers.
   }
   ``` 

# Have assignment operators return a reference to *this.
C++ supports chain of assignments and the assignment is right-associative. 
```c++
int x, y, z;
x = y = z = 15;  // equal to x = (y = (z = 15))
```
The assignment returns a reference to its left-hand argument.

When you implement assignment operators for your classes, you should follow the convention.

# Handle assignment to self in operator=
When you implement a operator= in a class, it may occurs self-assignment and this will cause some problem. For example
```c++
class Bitmap{};
class Widget {
  public:
    Widget& operator=(const Widget& rhs) {
      delete pb;  // if *this and rhs is same object, a problem occur.
      pb = new Bitmap(*rhs.pb);
      return *this;
    }
  private:
    Bitmap *pb;
}
```

The tradiional way to prevent this error is to check for assignment to self via an identity test at the top of operator=
```c++
Widget& operator=(const Widget& rhs) {
  if (this == &rhs)
    return *this
  delete pb;
  pb = new Bitmap(*rhs.pb);
  return *this;
}
```
The scheme above can avoid self-assignment, but it was exception-unsafe. If the "new Bitmap" expression yields an exception, the Widget will hold a pointer to a deleted Bitmap.

A exception-safe and assignment-safe solution is shown also below
```c++
Widget& operator=(const Widget& rhs) {
  Bigmap *pOrig = pb;
  pb = new Bitmap(*rhs.pb);
  delete pOrig;
  return *this;
}
```
A alternative to solution is use the technique know as "copy and swap"
```c++
class Widget {
  public:
    void swap(Widget& rhs);  // exchange rhs's data and *this's data.
    Widget& operator=(const Widget& rhs) {
      Widget temp(rhs);
      swap(temp)
      return *this;
    }
    // another way
    Widget& operator=(Widget rhs) {  // this will create a copy of rhs.
      swap(temp);
      return *this;
    }
}
```

# Copy all parts of an object
Copying functions should be sure to copy all of an object's data members and all of its base class parts.

Don't try to implement one of the copying functions in terms of the other. Instead, put common functionality in a third function that both call.

# Use objects to manage resources
There are two critical aspects of using objects to manager resources
- Resources are acquired and immediately turned over to resource-managing objects.
- Resource-managing objects use their destructors to ensure that resources are released.

This item suggests that if you're releasing resources manually, you're doing something wrong.

# Think carefully about copying behavior in resource-managing classes
There are some copying behaviors as below
1. Prohibit copying
2. Reference-count the underlying resource
3. Copy the underlying resource
4. Transfer ownership of the underlying resouce

So the copying function behavior should depend on what you want.

# Provide access to raw resources in resource-managing classes.
When you use classes to manage resources, you should access the resources with resource-managing classes, never sullying your hands with direct access to raw resources. But many APIs refer to access to raw resources. So you should provide a way to convert object into the raw resource it contains in resource-managing classes.

There two general ways to do it: explicit conversion and implicit conversion.
1. explicit conversion means that you should offer a get member function to return the raw pointer. But everytime you want to use raw resource, you must requset the get function. That in turn, would increase the chances of leaking fronts.
2. implicit conversion function is shown as below
   ```c++
   class Font {
     public:
      operator FontHandle() const {reutrn f;}  //implicit conversion function
   }
   ```
   The downside is that implicit conversions increase the chance of error. For example
   ```c++
   Font f1(getFont());

   FontHandle f2 = f1;  // f2 can access f1's resource. When f1 is destroyed, f2 will dangle.
   ```
   The decision about to offer explicit conversion or to offer implicit conversion is one that depends on the situation.
   explicit conversion is safer, but implicit conversion is more convenient for client. It is a tradeoff.

# Use the same form in corresponding uses of new and delete
When employing a new expression, two things happen. 
1. allocate memory
2. call one or more constructors for that memory

The same as empolying a delete express
1. one or more destructors are called.
2. the memory is deallocated.

If you use [] in a new expression, you must use [] in the corresponding delete expression. If you don’t use [] in a new expression, you mustn’t use [] in the corresponding delete expression.

# Store newed objects in smart pointers in standalone statements
Consider a function call as below
```c++
processWidget(std::shared_ptr<Widget>(new Widget), priority());
```

Before calling processWidget, compiler has to evaluate the arguments being passed as its parameters. In this cases, compiler has three things to do
1. Call priority
2. Execute new Widget
3. Call shared_ptr constructor.

In C++, compilers are granted considerable latitude in determining the order in which these things are to be done.

Consider a sequence of operations as below
1. Execute new Widget
2. Call priority
3. Call shared_ptr constructor

If priority function yields an exception, the pointer returned from new Widget will be lost.

The way to avoid problems is simple: use a separate statement to create the Widget and store in a smart pointer.
```c++
std::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority)
```

# Make interface easy to use correctly and hard to use incorrectly

Good interfaces are easy to use correctly and hard to use incorrectly. You should strive for these characteristics in all your interfaces.

Ways to facilitate correct use include consistency in interfaces and behavioral compatibility with build-in types

Ways to prevent errors include creating new types, restricting operations on types, constraining object values, and eliminating client resource management responsibilites.

# Treat class design as type design
Class design is type design. You should answer the questions below
* How should objects of your new type be created and destroyed? About how to design class's constructors and destructor.
* How should object initialization differ from object assignment? About how to design copy constructor and assignment operator.
* What does it mean for objects of your new type to be passed by value? About copy constructor.
* What are the restrictions on legal values for your new type. About value check.
* Does your new type fit into an inheritance graph? inheritance function, virtual or non-virtual.
* What kind of type conversions are allowed for your new type? conversion function design.
* What operators and functions make sense for the new type? 
* What standard functions should be disallowed? which function is private.
* Who shoud have access to members of your new type? public, protect or private member.
* What is the "undeclared interface" of your new type? 
* How general is your new type
* Is a new type really what you need.

# Prefer pass-by-reference-to-const to pass-by-value

For build-in types, it's often more efficient to pass it by value than by reference.

For iterators and function objects in STL, pass by value is more efficient too, because they are designed to be passed by value.

For other types, pass by reference is a better choice. It's typically more efficient and it avoid the slicing problem.

# Dot't try to return a reference when you must return an object

Reference just a name for some existing object.

Never return a pointer or reference to a local stack object, a reference to a heap-allocated object, or a pointer or reference to a local static object if there is a chance that more one such object will be need.

# Declare data members private
