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
Making a data member private can implement more precise control of the accessibility. For example
```c++
class AccessLevels {
  public:
    int getReadOnly() const {return readOnly;}
    void setReadWrite(int value) {readWrite=value;}
    int getReadWrite() {return readWrite;}
    void setWriteOnly(int value) {writeOnly = value;}
  private:
    int noAccess;
    int readOnly;
    int readWrite;
    int writeOnly;
};
```

Making a data member private can offer more flexibility. If a data member is public, when your ability to change it is extremely restricted, because too much code will be broken.

Public means unencapsulated, and unencapsulated means unchangeable.

For short getter and setter, you can declare them inline for speed up.

# Prefer non-member non-friend functions to member functions
Object-oriented principles dictate that data should be as encapsulated as possible.

See the example below
```c++
class WebBrowser {
  public:
    void clearCache();
    void clearHistory();
    void removeCookies();
};
```
Many users will want to perform all these actions together. There are two implementation
1. add a new member function
   ```c++
   class WebBrowser {
     publc:
      void clearEverything();
   };
   ```
2. add a non-member function
   ```c++
   void clearBrowser(WebBrowser& wb) {
     wb.clearCache();
     wb.clearHistory();
     wb.removeCookier();
   }
   ```

The better implementation is second. Because member function yields less encapsulation than the non-member function.
The reason is shown as below.

The more something is encapsulated, the fewer things can see it. The fewer things can see it, the greater flexibility we have to change it.

As a coarse-grained measure of how much code can see a piece of data, we can count the number of functions that can access that data. The more functions that can access it, the less encapsulated the data.

Thus, the member functions or the friend functions will yields less encapsulation.

In C++, a more natural approach would be to make clearBrowser a non-member function in the same namespace as WebBrowser.

A class like WebBrowser might have a large number of convenience functions, some related to bookmarks, others related to printing, etc. Most client will be interested in only a small sets of convenience functions. A practial is that declare bookmark-related convenience functsion in one header file, cookie-related convenience functions in a different headerf file, etc.
```c++
// header webbrowser.h
namespace WebBrowserStuff {
  class WebBrowser{};
}
// header webbrowserbookmarks.h
namespace WebBrowserStuff {
  // some bookmark-related convenience functions
}
//header webbrowsercookies.h
namespace WebBrowserSTuff {
  // some cookie-related convenience functions
}
```

# Declare non-member functions when type conversions should apply to all parameters
Considering a Rational class, you'd like to support arithmetic operations. It is natural to implement operator* inside the Ration class.
```c++
class Rational {
  public:
    Rational(int numerator=0, int denominator=1);

    int numerator() const;
    int denominator() const;

    const Rational operator*(const Rational &rhs) const;
  private:
    int numerator, denominator;
}
```
But when you try to do mixed-mode arthmetic, there is a problem
```c++
Rational oneHalf(1, 2);
Rational result;
result = oneHalf * 2;  // pass
result = 2 * oneHalf;  // error
```
The first one successes because the compiler call a non-expllicit constructor to create a temporary Rational object from 2.
```
const Rational temp(2);
result oneHalf * temp;
```

If you'd like to support mixed-mode arithmetic, you should make operator* a non-member function, thus allowing compilers to perform implict type conversions on all arguments.
```c++
class Rational {...};

const Rational operator*(const Rational& lhs, const Rational& rhs) {...}

Rational oneFourth(1, 4);
Rational result;
result = oneFourth * 2;  // pass
result = 2 * oneFourth;  // pass
```
The sequence of such operator functions is, member function, non-member functions(at namespace or global scope)

# Consider support for a non-throwing swap
The default swap algorithm implementation is shown as below
```c++
namespace std {
  template<typename T>
  void swap(T& a, T& b) {
    T temp(a);  //class should implement copy constructor and copy assignment operator
    a = b;
    b = temp;
  }
}
```

In the default swap implementation, it involves copying three objects. However, for some types, it is expensive. For example
```c++
class WidgetImpl {
  public:

  private:
    int a, b, c;
    std::vector<double> v;
};
class Widget {
  public:

  private:
    WidgetImpl *pImpl;
}
```
To swap the value of two Widget objects, we just need to swap their pImpl pointers instead of copying three objects.

If we want default swap algorithm to swap internal pImpl pointers, what we need to do is specialized std::swap function for Widget.
```c++
namespace std{
  template<>
  void swap<Widget>(Widget &a, Widget &b) {
    swap(a.pImpl, b.pImpl);
  }
}
```
The function above has a small problem, because the pImple is private member for Widget class.
To solution this problem is simple: declare a public member function called swap that do actual swapping, the specialize std::swap to call the member function.
```c++
class Widget {
  public:
    void swap(Widget &other) {
      using std::swap;
      swap(pImpl, other.pImpl);
    }
};
namespace std {
  template<>
  void swap<Widget>(Widget &a, Widget &b) {
    a.swap(b);
  }
}
```

This will work because std allow total template specialization.

But when class Widget and WidgetImpl is template, the scheme above does not work, because C++ doesn't allow partially specialize for function templates. If you want to partially specialize a function template, the usual approach is to simply add an overload, such as
```c++
namespace std {
  template<typename T>
  void swap(Widget<T> &a, Widget<T> &b) {
    a.swap(b);
  }
}
```
But std is a special namespace. We can totally specialize template in std, but it doesn't allow to add new tempaltes to std.

The answer is that we still declare a non-member swap that call the member swap instead of declaring a non-member to be a specialization or overloading of std::swap
```c++
namespace WidgetStuff {
  template<typename T>
  class Widget{};

  template<typename T>
  void swap(Widget<T> &a, Widget<T> &b) {
    a.swap(b);
  }
}
```
If any code calls swap on two Widget objects, the name lookup rules in C++ will find the Widget-specific version in WidgetStuff.

In summary, we need to write both a non-member version in the same namespace as your class and specialization of std::swap.

In the client's view, we do not know that a specialization of the general one or a T-specific one may or may not exist. The better way is shown as below
```c++
template<typename T>
void doSomething(T &obj1, T &obj2) {
  using std::swap;  // make std::swap available

  swap(obj1, obj2);  // make the compiler to choose the best swap.
}
```

C++'s name lookup rules is that compiler will find any T-specific swap at global scope or in the same namespace as the type T. If no T-specific swap exists, compilers will use swap in std. If std::swap has been specialized for T, the compiler will prefer a T-specific specialization of std::swap.

1. If the defulat implementation of swap is ok, you don't need to do anything.
2. Otherwise, do the following things
   1. Offer a public swap member function. This function should never throw an exception.
   2. Offer a non-member swap in the same namespace as you class or template.
   3. If writing a class, specialize std::swap for this class.
3. when calling swap, be sure to include a suing declaration to make std::swap visible in function, then call swap without any namespace qualification.

# Postpone variable definitions as long as possible
```c++
std::string encryptPassword(const std::string& password) {
  using namespace std;
  string encrypted;
  if (password.length() < MinimumPasswordLength) {
    throw logic_error("Password is too short");
  }
  ...
  return encrypted;
}
```
In the code above, if an exception is thrown, the object encrypted isn't unused. it is better to postponing encrypted's definition until it is needed.
```c++
std::string encryptPassword(const std::string& password) {
  ...
  string encrypted(password);
  encrypt(encrypted);
  return encrypted;
}
```

If a variable is used only inside a loop, there are two approach
```c++
// Approach A: define outside loop
Widget w
for (int i = 0; i < n; ++i) {
  w = ...
}
// Approach B: define inside loop
for (int i = 0; i < n; ++i) {
  Widget w(...);
}
```
The cost of these two approaches are as follow
* Approach A: 1 constructor + 1 destructor + n assignments
* Approach B: n constructors + n destructors

If an assignment costs less than a constructor-desctructor pair, Approach A is more efficient as n growing larger. Otherwise B is better.

# Minimize casting
C++ offers four new cast forms
* const_cast<T>(), only way to remove the constness of objects.
* dynamic_cast<T>(), used to perform safe downcasting. Only way that cannot be performed using the old-style syntax. Only way that may have a significant runtime cost.
* reinterpret_cast<T>(), intended for low-level casts, such as casting a pointer to int.
* static_cast<T>(), used to force implicit conversions.

C++ also support C-style casts, such as (T) expression and T(expression). But new forms are preferable.
1. mush easier to indentify in code.
2. compiler can diagnose usage error.

Type conversions often lead to code that is executed at runtime.
```c++
class Base{...};
class Derived: public Base {...};
Derived d;
Base *bp = &d;
```
In the code above, the base class pointer and the derived class pointer will not be the same. An offset is applied at runtime to the derived pointer to get the correct base pointer value.

The offset varies from compiler to compiler. So, you should generally avoid making assumptions about how things are laid out in C++, and you should certainly not perform casts based on such assumptions.

Suppose we want the virtual member function in derived classes call their base class couterparts first. 
```c++
class Window {
  public:
    virtual void onResize(){...}
};
class SpecialWindow: public Window {
  public:
    virtual void onResize() {
      static_cast<Window>(*this).onResize();
      ...
    }
}
```
The implementation in the code is wrong, because cast will create a new, temporary copy of the base class part of *this, then invokes onResize on the copy.

The correct way to implement what you expect is shown as below
```c++
class Window {
  public:
    virtual void onResize(){...}
};
class SpecialWindow: public Window {
  public:
    virtual void onResize() {
      Window::onResize();
      ...
    }
}
```

The dynamic_cast generally cast base class pointer or reference to what you believe to be a derived class object.

A common implementation is based in part on string comparisons of class names. So the dyname_cast cost is expensive.

Avoid casts whenever practical, especially dynamic_casts in performance-sensitive code. If requires casting, try to develop a cast-free alternative.

When casting is necessary, try to hide it inside a function.

# Avoid returning handles to object internals.
```c++
class Point {
  public:
    Point(int x, int y);
    void setX(int newVal);
    void setY(int newVal);
};
struct RectData {
  Point ulhc;
  Point lrhc;
};
class Rectangle {
  public:
    // according to perfer pass-by-reference-to-const to pass-by-value, return reference is more efficient.
    Point& upperLeft() const {return pData->ulhc;}
    Point& lowerRight() const {return pData->lrhc;}
  private:
    std::shared_ptr<RectData> pData;
};
Point coord1(0,0);
Point coord2(100,100);
const Rectangle rec(coord1, coord2);
rec.uperLeft().setX(50);  // pass
```
In the example above, rec is declared as const, but we can modify its internal Point data member. This is what we won't.

Returning pointers or iterators will cause the same problem.

References, pointers and iterators are all handles, and returning a handle to an object's interals always runs the risk of compromising an object's encapsulation.

We could applying const to their return types to avoid modifying.
```c++
class Rectangle {
  public:
    const Point& upperLeft() const {return pData->ulhc;}
    const Point& lowerRight() const {return pData->lrhc;}
};
```

But it can be problematic in other ways. It can lead to dangling handles.

Avoid returning handles (references, pointers, or iterators) to object internals. Not returning handles increases encapsulation, helps const member functions act const , and minimizes the creation of dangling handles.

# Strive for exception-safe code
When an exception is thrown, there are two requirements for exception safety.
* Leak no resource.
* Don't allow data structures to become corrupted.

Exception-safe functions must offer one of three guarantees
* The base guarantee promise. If an exception is thrown, everything in the program remains in a valid state. No objects or data structures become corrupted, and all objects are in internally consistent state. But the exact state of program may not be predictable.
* The strong guarantee promise. If an exception is thrown, the state of the program is unchanged.
* The nothrow guarantee promise. Never to throw exception. All operations on build-in types are nothrow.

We can not distinguish the guarantee from the declaration of a function. All those guarantee are determined by the function's implmentation, not its declaration. 

There is a general design strategy that typically leads to the strong guarantee. The strategy is known as copy and swap.
Make a copy of the object you want to modify, then make all needed changes to the copy. After all the changes have been successfully completed, swap the modified object with the original in a non-throwing operation.

The copy-and-swap strategy has some disadvantage
* it doesn't guarantee that the overall function is strongly exception-safe. Consider a function throw a exception after a database modify function call. The database state will change and can not undo.
* copying object may be expensive.

A function can usually offer a guarante no stronger that the weakest guarantee of the functions it calls.

# Understand the ins and outs of inlining
The idea behind an inline function is to replace each call of that function with its code body. In general, overzealous inlining may increase program size. But if an inline function body is very short, the code generated for the function body may be smaller than the code generated for a function call.

Inline is a request to compiler, not a command. So the compiler will decide if inlining the function will lead to any benefits.
The requset can be given implicitly or explicitly.
* implicitly: define a function inside a class definition.
* explicitly: use inline keyword.

Inline function must typically be in header files, because most build environments do inlining during compilation.

Limit most inlining to small, frequently called functions.

# Minimize compilation dependencies between files
```c++
class Person {
  public:
    Person(const std::string& name, const Date& birthday, const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
  private:
    std::string theName;
    Date theBirthDate;
    Address theAddress;
};
```
Look at the code above. If any header file used in Person class changed, the Person class must be recompiled, as must any files that use Person.

The compiler must know the Person size to allocate enough space. The only way to calculate the size it to consult the class definition.

A solution is shown as below
```c++
#include <string>
#include <memory>
class PersonImpl;
class Date;
class Address;
class Person {
  public:
    Person(const std::string& name, const Date& birthday, const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
  private:
    std::shared_ptr<PersonImpl> pImpl;
};
```
The clients of Person are divorced from the details of dates, addresses and persons. In the code above, the client uses Person clas instead of PersonImpl class, only Person class uses PersonImpl class.

The key is replacement of dependencie on definitions with dependencies on declarations. Make your header files self-sufficient whenever it's practical and when it's not depend on declarations in other files, not definitions.
* Avoid using objects when object references and pointers will do.
* Depend on class declarations instead of class definitions whenever you can.
* Provide separate header files for declarations and definitions.

I didn't fully understand this item.

# Make sure public inheritance models "is-a"
Public inheritance means "is-a". Everything that applies to base classes must also apply to derived classes.

# Avoid hiding inherited names.
When a class inherits a base class, the deried class inherits the things declared in the base class. Actually, the scope of the derived class is nested inside its base class's scope. 

The names in the inner scopes hide names in outer scopes. C++'s name-hiding rules do just that: hind names. Whether the names correspond to the same or different types is immaterial.

For example
```c++
class Base {
  private:
    int x;
  public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
};
class Derived: public Base {
  public:
    virtual void mf1();
    void mf3();
    void mf4();
};
Derived d;
int x;
d.mf1();  // call Derived::mf1
d.mf1(x); // error Derived::mf1 hides Base::mf2
d.mf2();  // call Based::mf2
d.mf3();  // call Derived::mf3
d.mf3(x); // error 
```

name-hiding rule will applies regardless of whether the functions are virtual or non-virtual or the functions take different parameter types.

The solution is to do it with using declarations
```c++
class Base {
  private:
    int x;
  public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
};
class Derived: public Base {
  public:
    using Base::mf1;  // make mf1 and mf3 in Base class visible in Derived's scope
    using Base::mf3;
    virtual void mf1();
    void mf3();
    void mf4();
};
Derived d;
int x;
d.mf1(x); // call Base::mf1 
d.mf3(x); // call Base::mf3 
```

Something you won't want to inherit all the functions from base class. The correct way to do is shown as below
```c++
class Base {
  public:
    virtual void mf1() = 0;
    virtual void mf1(int);
};
class Derived: public Base {
  public:
    virtual void mf1() {Base::mf1();}
};
Derived d;
int x;
d.mf1(); // call Base::mf1 
d.mf1(x); // error 
```
When inheritance is combined with templates, it will incur another problem that will tell in other item.

# Differentiate between inheritance of interface and inheritance of implementation

The purpose of declaring a pure virtual function is to have derived classes inherit a function interface only.

It is possible to provide a definition for a pure vitual function, but the only way to call it would e qualify the call with the class name.

The purpose of declaring a simple virtual function is to have derived class inherit a function interface as well as a default implementation.

The purpose of declaring a non-virtual function is to have derived classes inherit a function interface as well as a mandatory implementation.

The differences in declarations for pure virtual, simple virtual, and non-virtual functions allow you to specify with precision what you want derived classes to inherit: interface only, interface and a default implementation, or interface and a mandatory implementation, respectively.

# Consider alternatives to virtual functions.

## non-virtual interface idiom
A form of the Template Method design pattern that wraps public non-virtual member functions around less accessible virtual functions

## Function Pointer
```c++
class GameCharacter {
  public:
    typedef int (*HealthCalcFunc)(const GameCharacter&);
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc) : healthFunc(hcf) {}
    int healthValue() const {reurn healthFunc(*this);}
  private:
    HealthCalcFunc healthFunc;
};
```
The strategy offers some flexibility
* Different instances of the same character type can have different health calculation functions.
* Health calculation functions for a particular character may be changed at runtime.

## std::function
Replace virtual function with std::function data member.

## Strategyh pattern
Replace virtual functions in one hierarchy with virtual functions in another hierarchy.


A disadvantage of moving functionality from a member function to a function outside the class is that the non-member function lacks access to the class’s non-public members.

# Never redefine an inherited non-virtual function
```c++
class B{
  public:
    void mf1();
    virtual mf2();
};
class D: public B {
  public:
    void mf1();
    virtual void mf2();
};
D x;
B* pb = &x;
D* pd = &x;
pb->mf1();  // B::mf1
pb->mf2();  // D::mf2
pd->mf1();  // D::mf1
pd->mf2();  // D::mf2
```

Non-virtual functions are statically bound. 
Virtual functions are dynamically bound.

# Never redefine a function's inherited default parameter value.
Virtual functions are dynamically bound, but default parameter values are statically bound.
```c++
class Shape {
  public:
    enum ShapeColor{Red, Green, Blue};
    virtual void draw(ShapeColor color=Red) const = 0;
};
class Rectangle: public Shape {
  public:
    virtual void draw(ShapeColor color=Green) const = 0;
};
Shape *pr = new Rectangle;
pr->draw();  // call Rectangle::draw(ShapeColor::Red) instead of Rectangle::draw(ShapeColor::Green)
```
The reason of using this strategy is for runtime efficiency. If default parameter values were dynamically bound, compilers would have to come up with a way to determine the appropriate default value for parameters of virtual functions at runtime, which would be slower and more compicated.

But if you follow this rule and also offer default parameter values to users of both base and derived classes, code duplication occurs. If the default parameter value changed in base class, all derived classes that repeat it must also be changed.

In this case, it's wise to consider alternative designs to virtual functions talking above.

# Model "has-a" or "is-implemented-in-terms-of" through composition
Composition has meanings completely different from that of public inheritance.

In the application domain, composition means has-a. In the implementation domain, it means is-implemented-in-terms-of.

# Use private inheritance judiciously
private inheritance has two behavior.
* Compiler will generally not convert a derived class object into bass class object if the inheritance relationsip between the classes is private.
* Members inherited froma private base class become private members of the derived class.
```c++
class Person {};
class Student: private Person{};
void eat(const Person& p);
void study(const Student& s);
Person p;
Student s;
eat(p); // succeed
eat(s); // error
```

Pirvate inheritance means is-implemented-in-terms-of. If make a class D privately inherit from a class, because you are interested in taking advantage of some of the features available in class B.

Private inheritance is purely an implementation technique.

The choice strategy between private inheritance and composition is that use composition whenever you can, and use private inheritance whenever you must.

One of the case perfer private inheritance over composition is that you're dealing with a class that has no data in it. Such classes have no non-static data member, no virtual functions, and not virtual base class.

This class conceptually should use no space, but c++ decree that frestanding objects must have non-zero size.
```c++
class Empty {};
class HoldsAnInt {
  private:
    int x;
    Empyt e;
};
sizeof(HoldsAnInt) > sizeof(int)  // true
```
For most compilers, sizeof(Empty) is 1. But alignment requirements may cause compilers to add padding to class like HoldsAnInt.

This constriant doesn't apply to base class parts of derived class objects. so
```c++
class HoldsAnInt: private Empyt{
  private:
    int x;
};
sizeof(HoldsAnInt) == sizeof(int) //true
```
This is known as the empty base optimization(EBO). But the EBO is generally biable only under single inheritance.

Using private inheritance judiciously means empolying it when having considered all the alternatives, it's the best way to express the relationship between two classes in your software.

# Use multiple inheritance judiciously
One of the problem when using multiple inheritance is that it becomes possible to inherit the same name, which will lead to new opportunities for ambiguity.
```c++
class BorrowableItem {
  public:
    void checkOut();
};
class ElectronicGadget {
  private:
    bool checkOut();
};
class MP3Player :public BorrowableItem, public ElectronicGadget {};
MP3Player mp;
mp.checkOut();  // ambiguous
```
This ambiguous exist even though only one of the two functions is accessible. Because C++ first indentifies the function that's the best-match function before seeing whether a function is accessible.

To resolve the ambiguity, must specify which base class's function to call.

Multiple inheritance can lead to deadly MI diamound.

Classes using virtual inheritance are generally larger than they would be without using virtual inheritance. Access to data members in virtual base classes is also slower than to those in non-virtual base classes.

The rules governing the initialization of virtual base classes are more complicated and less intuitive than are those for non-virtual bases.
* classes derived from virtual bases that require initialization must be aware of their virtual bases, no matter how far distant the bases are
* when a new derived class is added to the hierarchy, it must assume initialization responsibilities for its virtual bases.

Advice on virtual base classes
* don't use virtual bases unless you need to.
* try to avoid putting data in virtual base classes.

Multiple inheritance is more complex than single inheritance. It can lead to new ambiguity issues and to the need for virtual inheritance.

Virtual inheritance imposes costs in size, speed, and complexity of initialization and assignment. It’s most practical when virtual base classes have no data.

Multiple inheritance does have legitimate uses. One scenario involves combining public inheritance from an Interface class with private inheritance from a class that helps with implementation.

# Understand implicit interfaces and compilet-time polymorphism.
The object-oriented programming revolves around explicit interfaces and runtime polymorphism.

The template and generic programming revolves around implicit interface and compile-time polymorphism.
1. Implicit interface is the set of expressions that must be valid in order for the template to compile.
2. Instantiating templates occurs during compilation.

Both classes and templates support interfaces and polymorphism. For classes, interfaces are explicit and centered on function signatures. Polymorphism occurs at the runtime throught virtual functions. For template parameters, interfaces are implicit and based on valid expressions. Polymorphism occurs during compilation throught template instantiation and function overloading resolution.

# Understand the two meanings of typename
When declaring template parameters, class and typename are inter-changeable.

```c++
//this is not valid C++ code
template<typename C>
void print2nd(const C& container) {
  if (container.size() >= 2) {
    C::const_iterator iter(container.begin());
    ++iter;
    int value = *iter;
    std::cout << value;
  }
}
```
Names in a template that are dependent on a template parameter are called dependent names. When a dependent name is nested inside a class, it is a nested dependent name. (What is non-nested dependent name look like?)
In the code above, C::const_iterator is a nested dependent name. More accuratly, it is a type.

Nested dependent names can lead to ambiguity. In the above code, until C is known, there is no way to know whether C::const_iterator is a type or isn't. So C++ consider a nested dependent name is not a type unless you tell it otherwise.

The correct version of the code above is shown as follow
```c++
template<typename C>
void print2nd(const C& container) {
  if (container.size() >= 2) {
    typename C::const_iterator iter(container.begin());
    ++iter;
    int value = *iter;
    std::cout << value;
  }
}
```

typename should be used to identify only nested dependent type names.

The exception to this rule is that typename must not precede nested dependent type names in a list of base classes or as a base class indentifier in a member initialization list.

# Know how to access names in templatized base classes
```c++
template<typename Company>
class MsgSender {
  public:
    void sendClear(const MsgInfo& info) {}
    void sendSecret(const MsgInfo& info) {}
};
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
  public:
    void sendClearMsg(const MsgInfo& info) {}
};
template<>
class MsgSender<CompanyZ> {
  public:
    void sendSecret(const MsgInfo& info);
}
```
The code above won't compile. The reason is that when compilers encounter the definition for the class template LoggingMsgSender, they don't know what class it inherits from. Company is a template parameter, one that won't be know until later.

Base class templates may be specialized and that such specializations may not offer the same interface as the general template. Thus, compile generally refuses to look in templatized base classes for inherited names.

To solve this problem, we have to somehow disable C++'s "don't look in templatized base classes" behavior. There are three way to do that
1. preface calls to base class function with "this->".
2. employ a using declaration.
3. explicitly specify that the function being called in the base class.

All of these approaches promise compilers that any subsequent specializations of the base class template will support the interface offered by the general template.

# Factor parameter-independent code out of templates
In template code repication is implicit: there's only one copy of the template sourc code, so you should sense the replication that may take place when a template is instantiated multiple times.

Templates generate multiple classes and multiple functions, so any template code not dependent on a template parameter causes bloat.

Bloat due to non-type template parameters can often be eliminated by replacing template parameters with function parameters or class data members.

Bloat due to type parameters can be reduced by sharing implementations for instantiation types with identical binary representations.

# Use member function templates to accept "all compatible types"
Real pointers supports impicit conversions. Derived class pointers implicitly convert into base class pointers, pointers to non-const objects convert into pointers to const object, etc.

A user-defined smart pointer classes as follow
```c++
template<typename T>
class SmartPtr {
  public:
    explicit SmartPtr(T *realPtr);
};
```
In the code above, we can't convert Derived class pointer to base class pointer. For example
```c++
class Top {}
class Bottom: public Top{}
Top *tp = new Bottom; // pass
SmartPtr<Top> stp = SmartPtr<Bottom>(new Bottom); // error
```
Because there is no inherent relationship among different instantiations of the same template.
To achieve conversion that we want, we can write a constructor. However, if the hierarchy is extended in the future, we need to add a new constructor.

Actually, we need a constructor template. Such templates are member function templates.
```c++
template<typename T>
class SmartPtr {
  public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other);
};
```
A small problem in the code above is that a conversion from SmartPtr<Bottom> to SmartPtr<Top> is legal, which is what we want. To restrict the conversion, we can utilize the build-in type implicit conversion.
```c++
template<typename T>
class SmartPtr {
  public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other):heldPtr(other.get()) {}
    T* get() const {return heldPtr;}
  private:
    T *heldPtr;
};
```
This will compile only if there is an implicit conversion from a U* pointer to a T* pointer.

It is not end. Compiler will generate copy constructor and copy opertor function if needed. A important thing is that Declaring a generalized copy constructor in a class doesn't keep compilers from generating their own copy constructor.

So you must declare both a generalized copy constructo as well as the normal copy constructor.
```c++
template<typename T>
class SmartPtr {
  public:
    SmartPtr(const SmartPtr const& r);
    template<typename U>
    SmartPtr(const SmartPtr<U>& other):heldPtr(other.get()) {}
    T* get() const {return heldPtr;}
  private:
    T *heldPtr;
};
```

# define non-member functions inside templates when type conversions are desired
Implicit type conversion functions are never considered during template argument deduction. Such conversions are used during function calls.

When writing a class template that offers functions related to the template that support implicit type conversions on all parameters, define those functions as friends inside the class template.

# Use traits classes for information about types
There are five categories of iterators, C++ has a tag struct for each of the five iterator categories.
* Input interators, move only forward, move only one step at a time, only read and read only once.
  ```c++
  struct input_interator_tag{};
  ```
* Output interators, move only forward, move only one step at a time, only write and write only once.
  ```c++
  struct output_interator_tag{};
  ```
* forward interator, conbine input interators and output interator.
  ```c++
  struct forward_interator_tag : public input_interator_tag{};
  ```
* Bidirectional interators, add to forward interators the ability to move backward as well as forward.
  ```c++
  struct bidirectional_interator_tag : public forward_iterator_tag{};
  ```
* random access iterators, add to bidirectional iterators the ability to jump forward or backward an arbitrary distance in constant time.
  ```c++
  struct random_access_interator_tag : public bidirectional_interator_tag{};
  ```

Advance moves a specified iterator a specified distance. However, Given the different iterator capabilities, only random access interator can directly achieve iter += d and other interators must be to use a loop that interatively increments or decrements the iterator. The advance's implementation what we really want is likely like this
```c++
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d) {
  if (iter is a random access iterator) {
    iter += d;
  } else {
    if (d >= 0) {
      while (d--) {++iter;}
    } else {
      while(d++) --iter;
    }
  }
}
```
This requires being able to determine whether iter is a random access iterator, which in turn requires knowing whether its type, IterT, is a random access iterator type.. The traits allow you to get information about a type during compilation.

Traits is a technique and a convention followed by C++ programmers. One fo the demands is that it has to work as well for built-in types as it does for user-defined types. It means things like nesting information inside types won't do.

The standard technique is to put it into a template and one or more specializations of taht template. For interators, the template in the standard library is named iterator_traits
```c++
template<typename IterT>
struct iterator_traits;
```
The way iterator_traits works is that for each type IterT, a typedef named iterator_category is declared in the struct iterator_traits<IterT>. This typedef identifies the iterator category of IterT. interator_traits implements this in two parts.
1. any user-defined iterator type must contain a nested typedef named iterator_category that identifies the appropriate tag struct.
  ```c++
  template<...>
  class deque {
    public:
      class iterator {
        public:
          typedef random_access_iterator_tag iterator_category;
      }
  };
  ```
  iterator_traits just parrots back the iterator class's nested typedef
  ```c++
  template<typename IterT>
  struct iterator_traits {
    typedef typename IterT::iterator_category iterator_category;
  };
  ```
2. To handle iterators that are pointers. iterato_traits offer a partial tempalte specialization for pointer types. Pointers act as random access iterators
  ```c++
  template<typename T>
  struct iterator_traits<T*> {
    typedef random_access_iterator_tag iterator_catagory;
  }
  ```
In summary, To design and implement a traits class
1. Identify some information about types you'd like to make available
2. Choose a name to identify that information.
3. Provide a template and set of specializations that contain the information for the types you want to support.

Given iterator_traits, we can check the type at the runtime. However, IterT's type is known during compilation, so we can check at runtime using overload.
```c++
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::random_access_iterator_tag) {
  iter += d;
}
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::bidirectional_iterator_tag) {
  if (d >= 0) {while(d--) ++iter;}
  else {while(d++) --iter;}
}
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::input_iterator_tag) {
  if (d < 0) {
    throw std::out_of_range("Negative distance");
  }
  while (d--) ++iter;
}
```
So advance can use overloading resolution to call proper implementation
```c++
template<typename IterT, typename DistT>
void advance(IterT &iter, DistT d) {
  doAdvance(iter, d, typename std::iterator_traits<IterT>::iterator_category());
}
```
For using a traits class
1. Create a set of overloaded worker functions or function templates that differ in a traits parameter. Implement each function in accord with the traits information passed.
2. Create a master function or function template that calls the workers, pass information provided by traits class.

# Be aware of template metaprogramming
Template metaprogramming(TMP) is the process of writing tempalte-based C++ programs that execute during comilation. A template metaprogram is a program written in C++ that executes inside the C++ compiler.

TMP has two great strengths
1. It makes some things easy that would otherwise be hard or impossible.
2. Can shift work from runtime to compile-time.

TMP can accomplish
* Ensuring dimensional unit correctness.
* Optimizing matrix operations.
* Generating custom design pattern implementations.

The disadvantage of TMP
1. Syntax is unintuitive.
2. Tool support is weak.

# Understand the behavior of the new-handler
Befere operator new throws an exception in response to an unsatisfiable request for memory, it calls a client-specifiable error-handling function called a new-handler. Clients use set_new_handler function specify the out-of-memory-handling function.
```c++
namespace std{
  typedef void (*new_handler)();
  new_handler set_new_handler(new_handler p) throw();
}
```
throw() means that this function won't throw any exceptions. set_new_handler function takes a new_handler function pointer and return old function pointer.

A new-handler function must do one of the following
* Make more memory available. One way to implement this strategy is to allocate a large block of memory at program start-up, then release it for use in the program the first time the new-handler is invoked.
* Install a different new-handler.
* Deinstall the new-handler. If no new-handler installed, operator new will throw an exception when memeory allocation fails.
* Throw an exception. Throw bad_alloc exception or some exception derived from bad_alloc.
* Not return. typically by calling abort or exit.

Sometimes you'd like to handle memory allocation failures depending on the class of the oject being allocated. The implementation will look like the following
```c++
class Widget {
  public:
    static std::new_handler set_new_handler(std::new_handler p) throw();
    static void* operator new(std::size_t size) throw(std::bad_alloc);
  private:
    static std::new_handler currentHandler;
};
```
The set_new_handler function implementation is like the standard version of set_new_handler
```c++
std::new_Handler Widget::set_new_handler(std::new_handler p) throw() {
  std::new_handler oldHandler = currentHandler;
  currentHandler = p;
  return oldHandler;
}
```

The operator new will do the following:
1. Call the standard set_new_handler with Widget's error-handling function.
2. Call the global operator new to perform the actual memory allocation. If failed, Widget's new-handler will be called. Whether exception thrown or not, original new-handler is always reinstated. Thus, treat the global new-handler as a resource and use resource-managing objects to prevent resource leaks.
  ```c++
  class NewHandlerHolder {
    public:
      explicit NewHandlerHolder(std::new_handler nh) : handler(nh) {}
      ~NewHandlerHolder() {
        std::set_new_handler(handler);
      }
    private:
      std::new_handler handler;
      NewHandlerHolder(const NewHandlerHolder&);
      NewHandlerHolder& operator=(const NewHandlerHolder&);
  }
3. If allocation successed, return a pointer to the allocated memory. The destructor of resource-managing object will restore the global new-handler.

The operator new implementation like this
```C++
void * Widget::operator new(std::size_t size) throw(std::bad_alloc) {
  NewHandlerHolder h(std::set_new_handler(currentHandler));
  return ::operator new(size);
}
```

The implementation is class-independent. A better way is to create a template base class. The reason of using template is that each interiting class has its own static data.
```c++
template<typename T>
class NewHandlerSupport {
  public:
    static std::new_handler set_new_handler(std::new_handler p) throw();
    static void* operator new(std::size_t size) throw(std::bad_alloc);
  private:
    static std::new_handler currentHandler;
};
class Widget: public NewHandlerSupport<Widget> {};
```

C++ required that operator new reutrn null in a early version when it was unable to allocate the requseted memory. Now, operator new will throw a bad_alloc exception. In order to returing null, we can use nothrow
```c++
Widget *pw2 = new (std::nothrow) Widget;
```
If allocation failed, nothrow from will just return null. But if allocation successed, the constructor is called, and if in constructor operator new is called and faild, excpetion will be throw and propagated as usual.

# Understand when it make sence to replace new and delete.
The reason for replacing the compiler-provided version of operator new or operator delete is that
* To detect usage errors.
* To collect usage statistics.
* To increase the speed of allocation and deallocation. General-purpose allocators are often a lot slower than custom version. Be sure to profile your program to confim that these functions are turly a bottleneck before custom operator new and operator delete.
* To reduce the space overhead of default memory management.
* To compensate for suboptimal alignment in the default allocator. Some compilers don't guarantee eight-byte alignment for dynamic allocations of doubles.
* To cluster related objects near one another. Some particular data structures are generally used together and cluster them can minimize the frequency of page faults.
* To obtain unconventional behavior. Do something that compiler-provided version don't offer.

# Adhere to convention when writing new and delete
Implementing a conformant operator new requirement
* Having the right return value. Return a pointer to allocation memory or throw an exception of type bad_alloc.
* Calling the new-handling function when insufficient memory is available.
* Being prepared to cope with the requests for no memory.
* Avoid inadvertently hiding the "normal" form of new.

There is no way to get at the new-handling function pointer directly, so you have to call set_new_handler to find out what it is.

Writing a custom memory manager is to optimize allocation for objects of a specific class, not for a class or any of its derived classes. Given an operator new for a class X, the behavior of that function is typically tuned for objects of size sizeof(X). But the operator new in a base class will be called to allocate memory for an object of a derived class. So Base's class-specific operator new must design to cope with this
```c++
class Base {
  public:
    static void* operator new(std::size_t size) throw(std::bad_alloc) {
      if (size != sizeof(Base)) {
        return ::operator new(size);
      }
    }
};
```
The operator new[] and the operator have same problem.

# Write placement delete if you write placement new