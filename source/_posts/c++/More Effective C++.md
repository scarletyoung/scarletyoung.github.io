---
title: More Effective C++ Note
date: 2022/01/04
---

# Distinguish between pointers and references
如何决定使用指针还是引用
1. 引用不可以为空，但是指针可以为空。
  ```cpp
  char *pc = 0;
  char& rc = *ps;
  ```
2. 引用必须被初始化，指针没有这个限制。
3. 引用更加高效，因为在使用引用前不需要测试引用是否有效。
4. 指针可能指向其他对象，而引用只能引用它初始化的对象
  ```cpp
  string s1("Nancy");
  string s2("Clancy");
  string& rs = s1;
  string *ps = &s1;
  rs = s2;  // rs refers to s1, and s1's value is change to "Clancy"
  ps = &s2; // ps points to s2, s1 is unchanged.
  ```

当总有一个对象可以引用时，当不想改变引用的对象时，或是语法要求使得不能使用指针，如operator\[\]时，因该使用引用。

其他情况则使用严格指针。

# Prefer C++-style casts
与Effecitive C++ 中的[Minimize casting](Effective%20C++.md#Minimize%20casting)相同。

# Never treat arrays polymorphically
假设我们有一个二叉搜索树类BST，以及从二叉搜索树继承来的平衡二叉搜索树类BalancedBST。
```cpp
class BST {};
class BalancedBST: public BST{};
```
有一个函数打印BST数组中的每个BST的内容。
```cpp
void printBSTArray(ostream& s, const BST array[], int numElements) {
  for (int i = 0; i < numElements; ++i) {
    s << array[i];
  }
}
```
这个函数也可以接受平衡二叉搜索树数据对象
```cpp
BalanceBST bBSTArray[10];
printBSTArray(count, bBSTArray, 10);  // can pass and run
```
因为参数被声明为BST数组类型，因此，数组中每个对象距离数组头的距离是固定的，即i+sizeof(BST)。
而派生类相较于基类通常会有更多的成员数据，因此派生类对象通常比基类对象大。

因此当传递BalancedBST对象数组到printBSTArray时，指针运算将会是错误的，因此，函数的行为是不确定的。

# Avoid gratuitous default constructors
对于一些对象而言，使用默认构造函数创建对象是不合理的。但是，如果一个类缺乏默认构造函数，它的使用会受到一些限制。
考虑以下的类，设备有一个强制的设备ID
```cpp
class EquipmentPiece {
public:
  EquipmentPiece(int IDNumber);
};
```
因为上述类缺乏默认构造函数，在三个环境中使用会出现问题
1. 创建数组的时候，因为无法指定数据中的构造函数参数，所以无法创建一个EquipmentPiece对象的数组，即以下代码是错误的
   ```cpp
   EquipmentPiece bestPieces[10];  // error
   EquipmentPiece *bestPieces = new EquipmentPiece[10];  // error
   ```
   一个常用绕过这个限制的方法是使用指针的数组，而不是对象的数组，即
   ```cpp
   typedef EquipmentPiece *PEP;
   PEP bestPieces[10];
   PEP *bestPieces = new PEP[10];
   for (int i = 0; i < 10; ++i) {
     bestPieces[i] = new EquipementPiece(ID);
   }
   ```
   上述方法有两个缺点，
   1. 需要手动删除指针数组指向的对象，否则会造成内存泄漏。
   2. 需要的内存增加了，要存储指针和对象。
      //TODO Place new
2. 一些基于模板的容器类会在内部创建一个模板参数类型的数组，因此要求用于实例化的模板的类型要提供一个默认构造函数，所以缺乏默认构造函数的类无法与这些容器类一起使用。例如，array类为
   ```cpp
   template<class T>
   class Array {
   public:
     Array(int size);
     ...
   private:
     T *data;
   };
   template<class T>
   Array<T>::Array(int size) {
     data = new T[size];
     ...
   }
   ```
3. 缺乏默认构造函数的虚基类使用起来很麻烦，因为，虚基类的构造函数参数要有派生类提供，这就要求派生类必须知道理解虚基类的构造参数含义，并提供参数。

基于以上原因，一些人认为所有类都必须提供一个默认构造函数，因此，EquipmentPiece被修改为如下形式
```cpp
class EquipmentPiece {
public:
  EquipmentPiece(int IDNumber = UNSPECIFIED);
private:
  static const int UNSPECIFIED; // magic number
};
```
但是，这样会使其他成员函数变得复杂，因为，不在能保证对象的ID是有意义的，所以使用前必须检查ID是否被正确初始化。当ID没有被初始化时，程序通常没有好的处理方法，只能抛出异常或终止程序。

同时由于增加了测试代码，执行时间和代码量增加，如果类的构造函数能够保证成员都被正确初始化，那么这些成本时可以避免的。

因此，需要避免一些没有意义的默认构造函数，尽管会带来一些限制。

# Be wary of user-defined conversion functions
对于自定义的类型，可以选择是否给编译器提供用于进行隐式转换的函数。有两类函数允许编译器执行这种函数，单参构造函数和隐式类型转换操作符。

单参构造函数是仅提供一个参数就可以调用的构造函数，可能是只有一个参数，或者是有多个参数但是第一个参数后面的参数有默认值，如下所示
```cpp
class Name {
public:
  Name(const string& s);
  ...
};
class Rational {
public:
  Rational(int numerator = 0, int denominator = 1);
  ...
};
```

隐式类型转换操作符是一个成员函数，函数名称是operator加上类型说明，如下所示，不需要指定函数的返回值，因为返回值得类型就是函数名称。
```cpp
class Rational {
public:
  ...
  operator double() const;  // implicity converts Rational to double
};
Rational r(1, 2);
double d = 0.5 * r;
```

尽管类型转换函数看起来非常方便，但是，通常不会想提供这些类型转换。

这种函数的一个基本问题是，这些函数经常会在你不希望调用他们的地方被调用，从而导致不正确的结果，并且这种问题很难诊断。

解决方法是将转换操作符替换为一个功能相同的成员函数，如下所示，这种成员函数需要显示的调用。
```cpp
class Rational {
public:
  double asDouble() const;
};
```

单参构造函数进行的隐式转换更难以消除，而且，它们引起的问题通常比隐式转换操作符引起的问题更严重。
```cpp
template<class T>
class Array{
public:
  Array(int lowBound, int hightBound);
  Array(int size);

  T& operator[](int index);
};
bool operator(const Array<int>& lhs, const Array<int>& rhs);
Array<int> a(10);
Array<int> b(10);
for (int i = 0; i < 10; ++i) {
  if (a == b[i]) {  // typo, should be a[i] == b[i]
    ...
  } else {
    ...
  }
}
```
在上述代码中，其实是希望比较a和b中的对象，但是由于打字错误，少打了一些字符，这种情况下，我们其实是希望编译器给出编译错误。但是，编译器注意到可以将int隐式转换为Array\<int\>，因此，进行了隐式转换，实际的代码如下，最终通过了编译，但是执行的方式与我们期望的不同。
```cpp
for (int i = 0; i < 10; ++i) {
  if (a == static_cast<Array<int>>(b[i])) ...
}
```

为了避免单参构造函数被隐式的调用，可以使用explicit关键字，被声明为explicit的构造函数无法被编译器隐式的调用，必须显示调用，如下所示
```cpp
template<class T>
class Array {
public:
  explicit Array(int size);
  ...
};
Array<int> a(10); // okey
if (a == b[i])  // error
if (a == Array<int>(b[i]))  //okey
```

如果编译器不支持explicit关键字，可以使用以下方法了避免单参构造函数的隐式转换，核心思想是任何转换序列都不允许包含一个以上的用户定义的转换。
```cpp
template<class T>
class Array {
public:
  class ArraySize {
  public:
    ArraySize(int num) : theSize(num) {}
    int size() const {return theSize;}
  private:
    int theSize;
  };
  Array(int lowBound, int highBound);
  Array(ArraySize size);
};
```
在这种情况下，无法将int转换为Array\<int\>，因为需要经过两次用户定义的转换，一次是从int转换为ArraySize，另一次是从ArraySize转换为Array\<int\>。

总之，除非确定非常需要转换函数，否则不要提供转换函数。

# Distinguish between prefix and postfix forms of increment and decrement operators
C++可以重载自增运算符和自减运算符，这两个运算符都有前缀形式和后缀形式。这里有个小小的语法问题，重载是通过参数类型来实现的，但是前缀形式和后缀形式都是无参的。为了解决这个问题，C++规定后缀形式需要一个int参数，当后缀形式的函数被调用时，编译器会传一个0作为参数。如下所示
```cpp
class UPInt {
public:
  UPInt& operator++();  // prefix ++
  const UPInt operator++(int);  // postfix ++
  UPInt& operator--();  // prefix --
  const UPInt operator--(int);  // postfix --
};
UPInt i;
++i;  // call i.operator++();
i++;  // call i.operator++(0);
--i;  // call i.operator--();
i--;  // call i.operator--(0);
```

这里需要注意的另一个地方是两个函数的返回值不同，前缀表达式返回一个对象的引用，后缀表达式返回一个常量对象。根据前缀和后缀表达式的语义，两个函数实现应该如下所示
```cpp
UPInt& UPInt::operator++() {
  *this += 1;
  return *this;
}
const UPInt UPInt::operator++(int) {
  UPInt oldValue = *this;
  ++(*this);
  return oldValue;
}
```
从实现来看，可以清楚的知道为什么后缀表达式返回的是一个对象。

我们先假设返回的不是常量，那么以下表达式是合法的
```cpp
UPInt i;
i++++;
```
因为它等同于
```cpp
i.operator++(0).operator++(0);
```
第二个函数调用会作用在第一个函数调用返回的对象上，实际对象会只增加一次。这与我们所期望的行为不同。
另一个更重要的原因是，对于重载的运算符，行为要和内置的运算符一致。即，对于一个整数而言，我们无法使用两次后缀自增运算符。
```cpp
int i;
i++++;  //error
```
因此，为了保持语义的一致以及正确的行为，让后缀自增运算符返回一个常量对象。当常量对象再次调用后缀自增运算符时，由于运算符函数是非常常量的，因此无法调用。

从效率方面来说，后缀自增运算符创建了一个临时对象，有对象创建和销毁的开销，因此，通常更加喜欢前缀表达式而不是后缀表达式。

# Never overload &&, ||, or ,
对于布尔表达式，C++采用来短路求值，即当整个表达式的值已经确定之后，后面的表达式将不会执行。
```cpp
char *p;
if ((p != 0) && (strlen(p) > 10)) ...
```
在上述代码中，当p为空指针时，后面的strlen函数将不会调用，可以避免在空指针上调用异常。

但是，当你重载了&&和||运算符的时候，表达式的行为会发生变化。虽然对用户来说，表达式没有发生变化，但是对编译器来说，表达式的行为如下所示
```cpp
if (expression1.operator&&(expression2)) ...
if (operator&&(expression1, expression2)) ...
```
表达式变成了函数调用，函数调用与短路求值有两个主要区别
1. 在函数调用的时候，所有表达式都必须评估，因此短路求值不存在。
2. 函数参数的表达式求值顺序不确定，而短路评估的求值顺序是从左到右。

因此，当重载了&&和||运算符的时候，布尔表达式的行为将发生变化，所以，不要重载它们。

对于,表达式也是如此，逗号表达式先计算左边的部分，再计算右边的部分，结果是右边部分的值。

另外，通常你不能重载以下操作符
* .
* .*
* ::
* ?:
* new
* delete
* sizeof
* typeid
* static_cast
* dynamic_cast
* const_cast
* reinterper_cast

# Understand the different meanings of new and delete
在下面的代码中，使用的new是new操作符，这个操作符是内建的，无法人为修改它的含义。
```cpp
string *ps = new string("Memory Management");
```
new操作符主要作两件事情
1. 为请求类型的对象分配足够的内存。
2. 调用构造函数来初始化分配内存中的对象。

我们可以修改的是如何为对象分配内存，new操作符调用函数来执行分配内存的情况，我们可以重写或重载这个函数来改变分配的行为，这个函数是operator new，它的声明如下所示
```cpp
void * operator new(size_t size);
```
它返回一个原始的未初始化的内存。这个函数也可以重载，但是第一个参数必须是size_t类型。

operator new仅负责分配内存，而new ooperator将接受operator new返回的原始的内存并转换为对象。当执行最开始的代码时，编译器生成的代码大致如下所示
```cpp
void *memory = operator new(sizeof(string));  // call operator new to get a raw memory for string object
call string::string("Memory Management") on *memory;  // initialize the object in the memory
string *ps = static_cast<string*>(memory);  // make ps point to the new object
```

一个特殊的operator new称为placement new。通常来讲，我们无法直接调用构造函数。但是，当我们有一些分配的原始内存，并且想要在这些内存上创建对象，就要使用placement new。
```cpp
class Widget {
public:
  Widget(int widgetSize);
  ...
};
Widget * constructWidgetInBuffer(void* buffer, int widgetSize) {
  return new (buffer) Widget(widgetSize);
}
```
这个函数返回一个Widget对象指针，这个对象时在buffer上构建的。

在new表达式中，为隐式调用的operator new指定了一个额外的参数buffer，隐式的operator new的定义大致如下，这个operator new就是placement new
```cpp
void * operator new(size_t, void *location) {
  return location;
}
```

综上所述，对于new operator和operator new，我们可以知道
* 当在堆上创建对象时，使用new operator。
* 当仅想要分配内存时，调用operator new。
* 当在堆上创建对象时，想要自定义内存分配的方式，重写operator new并使用new operator创建对象。
* 当想要在指定的内存上构建对象时，使用placement new。

## Deletion and Memory Deallocation
为了避免资源泄露，动态分配的内存都要进行回收。因此，delete operator和operator delete的关系和new operator和operator new的关系相同。

operator delete函数回收分配的内存，通常声明如下
```cpp
void operator delete(void *memoryToBeDeallocated);
```
而deleter operator则会析构对象并且回收内存，编译器生成的代码大致如下所示
```cpp
ps->~string();
opeartor delete(ps);
```

operator new和operator delete类似于malloc和free。

当使用placement new创建对象时，不应该使用delete operator来回收分配的内存。因为delete operator会调用operator delete来回收内存，但是，内存不一定时由operator new分配的。此时应该手动调用对象的析构函数，值执行相应的内存回收操作。如下所示
```cpp
void * mallocShared(size_t size);
void freeShared(void *memory);
void *sharedMemory = mallocShared(sizeof(Widget));
Widget *ps = constructWidgetInBuffer(sharedMemory, 10);
...
pw->~Widget();
freeShared(pw);
```

## Array
对于数组分创建，new operator在分配内存时不会调用operator new而是调用operator new[]，operator new[]和operator new类似。
在初始化对象的部分，new operator会为数组中的每个元素调用一次构造函数。
```cpp
string *ps = new string[10];  // call string constructor 10 times
```

在回收时也类似，delete operator为每个元素调用一次析构函数，并使用operator delete[]函数回收内存。

# Use desctructor to prevent resource leaks
假设有如下代码
```cpp
ALA* readALA(istream& s);
void processAdoptions(istream& dataSource) {
  while(dataSource) {
    ALA *pa = readALA(dataSource);
    pa->processAdoption();
    delete pa;
  }
}
```
在上述代码中，如果pa->processAdoption抛出了一个异常，异常会传播的调用者，此时所有在pa-processAdoption后面执行的代码会被跳过，这意味着pa不会被删除，因此产生了资源泄露。

为了防止资源泄露，可以加上try-catch块
```cpp
void processAdoptions(istream& dataSource) {
  while (dataSource) {
    ALA *pa = readALA(dataSource)
    try {
      pa->processAdoption();
    } catch (...) {
      delete pa;
      throw;
    }
    delete pa
  }
}
```
可以看到在上面的代码中，引入了重复代码。不利于维护。

如果可以将删除代码移动的局部对象的析构函数中，就可以避免这个问题，因为变量在离开作用域时会被销毁。解决方法是将指针pa替换为一个对象，通常称为智能指针。

使用智能指针的思想，我们可以将资源进行封装，在封装类的析构函数中释放资源，从而避免资源的泄露。

# Prevent resource leaks in constructors
C++仅会销毁完全构造的对象，一个完全销毁的对象是指构造函数运行完成的对象。

所以，当对象的构造函数在执行时抛出了异常，对象的析构函数不会被调用，因此，在构造函数抛出异常之前分配的对象无法被释放。如下所示
```cpp
class BookEntry {
public:
  BookEntry(const string& name, const string& address = "", const string& imageFileName = "", const string& audioClipFileName = "") : theName(name), theAddress(address), theIamge(0), theAudioClip(0) {
    if (imageFileName != "") {
      theImage = new Image(imageFileName);
    }
    if (audioClipFileName != "") {
      theAudioClip = new AudioClip(audioClipFileName);
    }
  }
  ~BookEntry();
private:
  string theName;
  string theAddress;
  List<PhoneNumber> thePhones;
  Image *theImage;
  AudioClip *theAudioClip;
};
```
在上述代码中，若AudioClip构造函数抛出了异常，Image对象不会被释放，造成了资源泄露。

此时可以在构造函数中增加try-catch来保证资源被正确释放。
如下所示
```cpp
BookEntry(const string& name, const string& address = "", const string& imageFileName = "", const string& audioClipFileName = "") : theName(name), theAddress(address), theIamge(0), theAudioClip(0) {
  try {
    if (imageFileName != "") {
      theImage = new Image(imageFileName);
    }
    if (audioClipFileName != "") {
      theAudioClip = new AudioClip(audioClipFileName);
    }
  } catch(...) {
    delete theImage;
    delete theAutoClip;
  }
}
```

但是当资源是常量的时候，对资源的初始化必须在初始化列表中执行，而在初始化列表中无法使用try-catch。此时可以将资源的初始化写在私有函数中，具体如下所示
```cpp
class BookEntry {
public:
  BookEntry(const string& name, const string& address = "", const string& imageFileName = "", const string& audioClipFileName = "") : theName(name), theAddress(address), theIamge(initImage(imageFileName)), theAudioClip(initAudoClip(audioCLipFileName)) {}
private:
  Image * initImage(const string& imageFileName) {
    if (imageFileName != "") {
      return new Image(imageFileName);
    } else {
      return 0;
    }
  }
  AudioClip * initAutoClip(const string& audioClipFileName) {
    try {
      if (audioClipFileName != "") {
        return new AudioClip(audioClipFileName);
      } else {
        return 0;
      }
    } catch (...) {
      delete theImage;
      throw;
    }
  }
private:
  const Image* theImage;
  const AudioClip* thiAudioClip;
}
```

当然，更好的方案是使用智能指针来管理资源。

# Prevent exceptions from leaving destructor
在析构函数调用的时候有两种情况，一种是正常情况，即变量离开作用域或析构函数被显示调用。另一种是异常处理机制调用来销毁对象。

在析构函数调用的时候，我们无法知道是哪一种情况。因此，在写析构函数的时候，必须假设异常时激活的，否则，当析构函数产生异常导致控制流离开了析构函数，那么在异常激活的情况下，程序会立即终止。
此外，当析构函数抛出异常的时候，异常之后的代码不会被执行，部分资源不会被释放。

# Understand how throwing an exception differs from passing a parameter or calling a virtual function
传递异常给catch语句和传递参数给函数有相似之处，但它们是完全不同的。
相似之处在于，都可以通过值、引用和指针传递参数和异常。
区别在于，当函数调用完成，控制流会返回到函数调用点，而抛出异常后，控制流不会返回到抛出异常点。

当一个对象被抛出时，无论异常是否捕获的方式是什么（值或引用），都会产生一个对象的副本，并将副本传递给catch语句中。
而当函数调用传递对象时，是否产生对象的副本取决于参数的声明方式。
这是因为当局部对象被抛出时，对象会离开作用域，因此它的析构函数会被调用。因此当抛出对象时，总是会复制对象，无论是否时局部对象。

因为总是会发生对象的复制，所以抛出异常会比函数调用慢的多。

当对象被复制作为异常的时候，由对象的复制构造函数执行复制操作，调用的是对象静态类型的复制构造函数而不是动态类型的复制构造函数，如下所示
```cpp
class Widget {};
class SpecialWidget : public Widget{};
void passAndThrowWidget() {
  SpecialWidget localSpecialWidget;
  Widget& rw = localSpecialWidget;
  throw rw; // throw an exception of type widget
}
```
这个行为可能不是我们所期望的，但是它与其他复制对象的行为保持了一致，即复制总是基于对象的静态类型而不是动态类型。

异常是对象的副本会影响异常在catch块中的传播，考虑如下代码
```cpp
catch(Widget& w) {
  ...
  throw;  // 重新抛出捕获的异常，不会产生新的对象
}
catch(Widget& w) {
  ...
  throw w;  // 将捕获的对象以Widget类型抛出，产生新对象
}
```
在第一个块中，会将捕获的异常重新抛出，异常的类型不会变，也不会产生新的对象副本。而在第二个块中，会抛出一个新的异常，类型为Widget，并且发生复制行为。

catch语句由三种方式捕获异常
```cpp
catch(Widget w)...
catch(Widget& w)...
catch(const Widget& w)...
```
当以值捕获时，会发生两次复制，一次是创建异常时产生的临时对象，二是将临时对象复制到w。
当以引用捕获时，在创建异常产生临时对象时发生复制行为。对于函数调用而言，传递临时对象给一个非const的引用参数是不允许的，但对异常来说是允许的。另外由于复制的发生，对于异常而言，非常量的引用和常量引用是相同的。
当以指针捕获时，传递的是指针的复制，需要注意的一点是不要抛出局部对象的指针。

当函数调用时，会发生隐式转换，如下所示
```cpp
double sqrt(double);
int i;
double sqrtOfi = sqrt(i);
```
但是，通常而言，这种转换不会再匹配异常时发生，如
```cpp
void f(int value) {
  try {
    throw value;
  } catch (double d) {
    ...
  }
}
```
在上述代码中，try块中抛出一个int异常，但是catch捕获的是double异常，因此，抛出的异常不会被catch块捕获。

在匹配异常时，由两种转换可以生效
1. 基于继承的转换，即捕获基类的catch块可以捕获派生类异常。
2. 从类型化的指针到无类型的指针，因此，捕获void*指针的catch块可以捕获任何指针类型的异常。

catch语句的执行是按声明的顺序，因此，当异常被前一个catch捕获后，后面的catch就无法捕获异常了。

# Catch exceptions by reference
如上一节所说，catch语句捕获异常的方式由三种，通过指针捕获，按值捕获和按引用捕获。

按指针捕获在理论上来说是最高效的方式，因为抛出指针是唯一一个不需要复制对象的方式。它的问题在于异常需要定义为为全局或静态变量，以保证抛出异常后，指针所指向的对象依然存在。
一个替代的方法是抛出指向新堆对象的指针，如下所示
```cpp
void someFunction() {
  throw new exception;
}
```
上述方法可以避免catch到的指针指向被销毁的对象，但是，会带来另一个问题，catch是否需要销毁接收到的指针？如果对象分配在堆上，则必须销毁，否则会造成资源泄露。如果对象没有分配在堆上，则不能销毁，否则会导致不确定的行为。

catch块无法知道对象分配的方式，因此无法确定是否删除指针。

另外，四个标准异常bad_alloc，bad_cast，bad_typeid，bad_exception都是对象，而不是对象指针。

按值捕获没有删除的问题，也能很好的应用于标准异常类型，但是，异常对象会复制两次。另外，还会产生切片问题，当捕获的是基类时，派生类对象的部分会被切掉，此时，虚函数会调用基类版本而不是派生类的版本。

按引用捕获则没有上述提到的所有问题。

# Use exception specifications judiciously
异常规范说明了一个函数可以抛出哪些异常，编译器在编译的过程中可以法线不一致的异常规范。另外，当一个函数抛出一个不在异常规范中的异常时，运行时会检测到这个行为，并且自动调用特殊函数unexpected。

但是，unexpected的默认行为是调用terminate，而terminate的默认行为是调用abort。因此，当违反异常规范时，默认行为是终止程序。此时，在活动堆栈帧中的局部变量不会被销毁。
不幸的是，上述情况很容易发生，因为编译器仅仅部分检查异常的使用是否与异常规范一致，当被调用函数的异常规范违反了调用函数的异常规范时，编译器不会检查这种情况。如下所示
```cpp
extern void f1(); // might throw anyting
void f2() throw(int) {
  ...
  f1(); // 即使f1可能抛出违反f2异常规范的异常时，这个调用也是合法的。
  ...
}
```

因此，在编码的过程中，需要尽量使这种不一致最小。

一个好的开始是不要把异常规范放在接受类型参数的模板上，考虑如下函数
```cpp
template<class T>
bool operator==(const T& lhs, const T& rhs) throw() {
  return &lhs == &rhs;
}
```
这个模板函数声明不会抛出任何异常，但是，由于取地址运算符&可能会被重载，因此可能会抛出异常，那么此时就违反了异常规范。

这个例子说明了没有办法了解模板的类型参数所抛出的异常，因此应该避免模板和异常规范混合使用。

第二个方式是在调用缺乏异常规范的函数的函数上省略异常规范，一个容易被忽略的地方是回调函数，如下所示
```cpp
typedef void (*CallBackPtr)(int eventXLocation, int eventYLocation, void *dataToPassBack);
class CallBack {
public:
  CallBack(CallBackPtr fPtr, void *dataToPassBack) : func(fPtr), data(dataToPassBack) {}
  void makeCallBack(int eventXLocation, int eventYLocation) const throw() {
    func(eventXLocation, eventYLocation, data);
  }
private:
  CallBackPtr func;
  void *data;
}
```
在makeCallBack中调用回调函数可能会违反异常规范，因为不知道回调函数会抛出什么异常。
一个解决方法是在声明函数指针是指定异常规范，如下所示
```cpp
typedef void (*CallBackPtr) (int eventXLocation, int eventYLocation, void *dataToPassBack) throw();
```
此时当注册回调函数时，若回调函数没有保证不抛出异常，则会出错。

第三个方法是避免调用unexpected是处理系统可能抛出的异常。

如果防止意外异常并不实际，可以利用C++允许使用不同类型的异常来替换未预料的异常。例如，将所有未预期的异常替换未UnexpectedExceptio对象，如下所示
```cpp
class UnexpectedException{};
void convertUnexpected() {
  throw UnexpectedException();
}
set_unexpected(convertUnexpected);  // 将默认unexpected函数替换未convertUnexpected
```

只要异常规范中包含UnexpectedException，那么异常传播就会继续进行。

另一种将意外异常转化为众所周知的类型的方法是依靠这样一个事实：如果意外函数的替换程序重新抛出当前的异常，该异常将被一个标准类型的新异常bad_exception所替换。
```cpp
void convertUnexpected() {
  throw;
}
set_unexpected(convertUnexpected);
```
此时，在异常规范中包含bad_exception即可。

异常规范还有另一个缺点，即使高层的调用者对所有可能抛出的异常做了应对，unexpected也可能会被调用。如下所示
```cpp
class Session {
public:
  ~Session() {
    try {
      logDestruction(this);
    } catch (...) {}
  }
private:
  static void logDestruction(Session *objAddr) throw();
};
```
Session析构器调用LogDestruction来记录Session对象的销毁，它明确地捕获了所有异常。但是，LogDestruction的异常规范说明它不会抛出任何异常。假设，LogDestruction调用的一些函数抛出了一个异常，且LogDestruction没有捕获这些异常，此时，违反了LogDestruction的异常规范，因此，unexpected会被调用，默认的情况下，它会终止程序。尽管这个行为确实是正确的，但不是我们所期望的。

综上所示，异常规范应该谨慎使用。（替换unexpected的默认行为是不是比较好？）

# Understand the costs of exception handling
在没有使用任何异常处理特征时，需要额外的空间来存储跟踪那些对象被完全构建的数据结果，并且需要额外的时间了保证保证这些数据结构是最新的。这些额外的花销通常是小的。

异常处理的第二个成本来自try块，不同的编译器使用不同的方式实现try块，因此成本也随编译器变化而变化。当使用try块但没有抛出异常时，程序的大小和运行时间大约增加5%~10%。因此，需要避免无意义的try块。

编译器为异常规范生成的代码和try块一样多，因此，异常成本的规范和try块相同。

与正常的函数返回相比，通过抛出异常来返回函数的速度可能会慢三个数量级。

为了尽量减少与异常相关的成本，在可行的情况下，在编译时不要支持异常；将尝试块和异常规范的使用限制在那些你真正需要的地方；并且只在真正例外的情况下抛出异常。

# Remember the 80-20 rule
软件的总体性能通常由一小部分代码决定。

这个规则表示当软件存在性能问题时，你需要找到影响性能的一小部分并且极大的提供它的性能。

确定性能瓶颈的方法是使用程序分析器。你需要使用一个能够直接测量感兴趣的资源的分析器，例如，如果程序运行的很慢，分析器需要能够测量出程序的每个部分的执行时间。

此外，程序分析器时测量程序在一组运行上的数据，如果输入数据不具有代表性，那么得出的结果也没有帮助。

因此，为了分析程序的性能瓶颈，需要一个好的程序分析器，并在一组具有代表性的数据上得出结果。

# Consider using lazy evaluation
当使用懒惰评估的时候，我们会推至计算过程知道需要计算结果的时候，如果不需要结果，则计算不会执行。

考虑以下代码
```cpp
class String{};
String s1 = "Hello":
String s2 = s1;
```
当使用s2初始化s1后，一个常见的实现会使得s1和s2都有一个"Hello"的副本。这是个费时的工作，它需要使用new运算符在堆上分配一个内存，调用strcpy将数据复制到s2中。

懒惰评估则不会立即复制s1的值给s2，而是s2共享s1的值，只需要记录下这个信息。当其中一个string被修改的时候，只修改一个string，在下面的语句中
```cpp
s2.convertToUpperCase();
```
仅有s2的值被修改，s1的值不会发生变化。
要实现这个效果，需要在实现convertToUpperCase函数的时候，将s2的值私有化，即，将s2共享的值（这里是s1的值）制作一个副本给s2私用。

如果s2的值不会被修改，则不需要对值进行复制，避免的复制的开销。

对于懒惰评估而言，一个重点是区分读写操作。因为，写操作需要进行复制，而读操作不需要，考虑以下代码
```cpp
String s = "Homer's Iliad";
cout << s[3];
s[3] = 'x';
```
上述两个语句都调用了operator\[\]，但是一个是读一个是写。不幸的是，在operator\[\]中我们无法区分是读还是写，但是通过懒惰评估和代理类，可以推迟决定是采取读动作还是写动作，直到我们能够确定哪个是正确的。

另一个懒惰评估的例子是延迟表达式求值，考虑如下代码
```cpp
template<class T>
class Matrix {};
Matrix<int> m1(1000, 1000);
Matrix<int> m2(1000, 1000);
Matrix<int> m3 = m1 + m2;
```
当在operator+中使用及早评估时，为了得到m3需要执行1,000,000次加法以及分配相应的内存。而在延迟评估策略中，则是在m3中设置一个数据结构表明m3的值是m1和m2的和，这个数据结构仅需要m1和m2的指针以及相应的操作。

要实现懒惰评估是一项艰巨的工作，它需要存储值之间的依赖关系，维护可以存储值、依赖关系或两者结合的数据结构以及重载运算符。但是，它往往能节省大量的时间和空间。

但是，当所有的计算都是必要的时候，懒惰评估反而会减慢运行速度并增加内存使用。因此，当分析显示一个类的实现是性能瓶颈，可以用一个基于懒惰评估的实现来替换。

# Amortize the cost of expected computations
这个改进性能的策略称为过早评估，即在要求作之前就做完。

考虑下面的代码，这个模板类代表大量数字数据的集合。
```cpp
template<class NumericalType>
class DataCollection {
public:
  NumericalType min() const;
  NumericalType max() const;
  NumericalType avg() const;
  ...
};
```
我们有三种方法实现这些函数
1. 使用及早评估，在函数调用时检查集合中的数据并返回合适的值。
2. 使用懒惰评估，函数返回一个数据结构，当返回值需要的时候，这个结构可以返回合适的值。
3. 使用过早评估，追踪集合中的这些最小值、最大值、平均值等信息，在需要的时候可以立即返回。

当这些函数被频繁调用的时候，第三个方法可以将这些计算平摊到每次调用上，因此，每次调用的成本将小于及早评估和惰性评估。

过早评估的思想是如果一个计算会被频繁请求，可以通过设计数据结构来特别有效地处理请求，从而降低每个请求的平均成本。

摊销的方法有
1. 缓存：对那些可能再次需要的值进行缓存。
2. 预取：利用程序和数据的局部性。如对vector扩容是每次扩大两倍而不是增加一个。

# Understand the origin of temporary objects.
C++中真正的临时对象是不可见的，通常是无名的非堆对象，这种无名对象通常由两种方式产生
1. 为了使函数调用成功而进行隐式类型转换。
2. 当函数返回对象。

理解临时对象的创建和销毁是很重要的，因为创建和销毁的成本会显著的影响程序的性能。

第一种情况发生于传递给函数的对象和函数声明的对象不一致的时候，考虑如下代码
```cpp
size_t countChar(const string& str, char ch);
char buffer[MAX_STRING_LEN];
char c
cin >> setw(MAX_STRING_LEN) >> buffer;
cout << "There ara " << countChar(buffer, c) << " occurrences of the character " << c << " in " << buffer << endl;
```
在调用countChar的时候，第一个参数的类型和声明的类型不同，想要函数调用成功，必须将第一个参数转换为string类型，因此，编译器创建了一个临时对象，传递给函数，当函数调用结束后，临时对象被自动销毁。
有两种方法可以避免这些方法
1. 如[Item 5](Be wary of user-defined conversion functions)所示，使得隐式转换不会发生。
2. 如[Item 21]()所示，修改代码使得隐式转换是不必要的。

这种隐式转换只会发生在以值传递和以常量引用传递，当以非常量引用传递时，隐式转换不会发生
```cpp
void uppercasify(string& str);
char subtleBookPlug[] = "Effective C++";
uppercasify(subtleBookPlug);  // error
```
原因在于，当传递的是非常量的参数时，函数可能会修改参数对象，此时，若进行了隐式转换，创建了临时对象，那么修改的是临时对象而不是传递的对象，这个行为与预期不符。而常量引用的参数没有这个问题，因此可以进行隐式转换。

第二种情况如下
```cpp
const Number operator+(const Number& lhs, const Number &rhs);
```
这个函数的返回值是一个临时对象。对于某些函数可以切换外另一个类似的函数来避免临时对象的开销，如上述函数可用operator+=替代。但是大部分函数是无法这么做的，因此，理论上是无法避免临时对象的开销。

但是，编译器会进行一些优化，最常用的是返回值优化。

# Facilitate the return value optimization
返回对象的函数通常会创建临时对象，意味着存在构造函数和析构函数的调用，会对程序的效率有所影响。一些函数就是需要返回对象来保证程序的正确性，但是，返回对象不是问题，返回对象的执行花费才是问题。

如果函数返回一个对象，现在编译器已经可能消除临时对象的花费。技巧是返回构造参数而不是对象，如下所示
```cpp
const Rational operator*(const Rational& lhs, const Rational& rhs) {
  return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
}
```
函数的返回表达式调用了构造函数，这个构造函数创建一个临时对象，函数的返回值要复制是这个临时对象的副本。但是，C++的规则允许编译器优化以消除临时对象，当调用代码如下时，编译器可以消除函数内部和函数返回值的临时对象
```cpp
Rational a = 10;
Rational b(1,2);
Rational c = a * b;
```
在这种情况下，编译器使用分配给对象c的内存构造返回表达式定义的对象。此时，没有构建任何临时对象。

在96年之后，ISO声明命名和未命名对象都可能被返回值优化策略优化，因此，下面的也可能产生优化后的代码
```cpp
const Rational operator*(const Rational& lhs, const Rational& rhs) {
  Rational r(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
  return r;
}
```

# Overload to avoid implicit type conversions
考虑如下代码
```cpp
class UPInt {
public:
  UPInt();
  UPInt(int value);
  ...
};
const UPInt operator+(const UPInt& lhs, const UPInt& rhs);
UPInt upi1, upi2;
UPInt upi3 = upi1 + upi2;
UPInt upi4 = upi1 + 10;
UPInt upi5 = 10 + upi2;
```
上述所有语句都会执行成功，包括最后两个，因为通过隐式转换将10转换为了UPInt对象。

但是，正如之前所说，隐式转换会创建临时对象，会带来额外的执行成本。

隐式对象转换的目的是使得调用成功，因此，为了使得混合类型的方法参数调用成功，我们可以声明多个函数，每个由不同的参数类型，如下所示
```cpp
const UPInt operator+(const UPInt& lhs, const UPInt& rhs);
const UPInt operator+(int lhs, const UPInt& rhs);
const UPInt operator+(const UPInt& lhs, int rhs);
```
但是声明以下函数是不行的
```cpp
const UPInt operator+(int lhs, int rhs);
```
因为C++规定，在重载运算符的时候必须至少有一个参数是用户自定义的类型。

有一个疑问，重载函数会有多个实现，不利于维护吧？

# Consider using op= instead of stand-alone op
```cpp
x = x + y;
x += y;
```
在上述代码中，如果x和y都是用户自定义类型，则无法保证通过编译。在C++中operator+、operator=和operator+=是没有关系的，如果想要这三个运算符存在且有相应的关系，必须自己实现，其他运算符也类似。

一个好的方法是保证赋值版本的运算符和独立版本的运算符的正确关系的实现是使用后者调用前者。如下所示
```cpp
class Rational {
public:
  Rational& operator+=(const Rational& rhs);
};
const Rational operator+(const Rational& lhs, const Rational& rhs) {
  return Rational(lhs) += rhs;
}
```
在这个情况下，仅需要维护赋值版本的运算符实现。

如果不介意将独立版本的运算符将全局作用域，可以使用模板来实现，从而避免写单独版本的运算符，例子如下所示
```cpp
template<class T>
const T operator+(const T& lhs, const T& rhs) {
  return T(lhs) += rhs;
}
```
对于某些编译器而言，可能会将T(lhs)视为转换，从而移除lhs的常量性，然后返回修改后的lsh的引用，在使用前需要测试以下编译器的行为。

在效率方面，有三个需要注意的地方。
1. 通常赋值版本的运算符通常更加高效，因为独立版本的运算符返回一个新对象，需要创建和销毁临时对象。而赋值版本的运算符不需要创建临时对象。
2. 通过提供运算符的赋值版本以及独立版本，你允许你的类的客户在效率和便利性之间做出权衡。可以在下面两个版本的代码中根据情况进行选择
  ```cpp
  Rational a,b,c,d,result;
  //version 1
  // 写、调试和维护容易
  result = a + b + c +d;
  //version 2
  // 更加高效
  result = a;
  result += b;
  result += c;
  result += d;
  ```
3. 独立版本运算符的不正确实现可能会导致无法利用返回值优化。

# Consider alernative libraries
TODO

# Understand the costs of virtual functions, multiple inheritance, virtual base classes, and RTTI
TODO

# Virtualizing constructors and non-member functions
尽管虚构造函数似乎没有意义，但是它们是相当有用的。例如，假设要写一个通讯应用，其中包含了文本或图像组件，代码大致如下
```cpp
class NLComponent {};
class TextBlock : public NLComponent {};
class Graphic : public NLComponent {};
class NewsLetter {
private:
  list<NLComponent*> components;
};
```
NewsLetter中包含了一个指针列表，指向了包含的组件。当NewsLetter对象不工作时，需要存储在硬盘上。为了从硬盘读取的数据直接创建对象，NewsLetter有一个以istream作为参数的构造函数。大致如下所示
```cpp
NewsLetter::NewsLetter(istream& str) {
  while (str) {
    // read the next component object from str;
    // add the object to the list
  }
}
```
或者是创建一个静态函数readComponent。

总之，在具体实现中，需要根据读取的数据创建TextBlock对象或Graphic对象。因为它根据输入创建了不同的对象，因此称为虚构造函数（工厂模式？）

一个特别的虚构造函数是虚构造赋值函数，通常如下所示
```cpp
class NLComponent {
public:
  virtual NLComponent * clone() const = 0;
};
class TextBlock: public NLComponent {
public:
  virtual TextBlock * clone() const {
    return new TextBlock(*this);
  }
};
class Graphic: public NLComponent {
public:
  virtual Graphic * clone() const {
    return new Graphic(*this);
  }
}
```
虚构造函数中调用真正的构造函数，深复制和浅复制取决于复制构造函数的实现。

## Making Non-Member Functions Act Virtual
假设需要实现TextBlock和Graphics类的输出操作符，由于需要根据动态类型来决定行为，因此需要实现虚的输出操作符。尽管可以通过虚拟重载输出操作符，但是，这个函数是将ostream参数作为左值，与实际使用方法不符。
也可以通过声明一个虚拟的print类来进行输出，但语法上就与其他的输出不一致类。

一个较好的方法如下所示
```cpp
class NLCOmponent {
public:
  virtual ostream& print(ostream& s) const = 0;
};
class TextBlock: public NLComponent {
public:
  virtual ostream& print(ostream& s) const;
};
class Graphic: public NLComponent {
public:
  virtual ostream& print(ostream& s) const;
};
inline ostream& operator<<(ostream& s, const NLComponent& c) {
  return c.print(s);
}
```

# Limiting the number of objects of a class

## Allowing Zero or One objects
当对象实例化的时候，一定有构造函数被调用。因此，防止生成特定类的对象的最简单方法是将构造函数声明为私有。
```cpp
class CantBeInstantiated {
private:
  CantBeInstantiated();
  CantBeInstantiated(const CantBeInstantizted&);
};
```

稍稍放松一下条件，我们想为类仅创建一个对象，具体如下
```cpp
class Printer {
public:
  friend Printer& thePrinter();
private:
  Printer();
  Printer(const Printer& rhs);
};
Printer& thePrinter() {
  static Printer p;
  return p;
}
```
这个设计分为三个部分
1. 私有的构造函数，阻止类对象创建
2. 全局函数thePrinter声明为该类的友元，使得thePrinter可以调用私有的构造函数。
3. thePrinter包含一个静态Printer对象，保证仅有一个对象被创建。

当然，thePrinter函数也可以声明为Printer的静态函数，或者增加命名空间等等加以限制。

thePrinter的实现有两个微妙之处需要探讨
1. 唯一的Printer静态对象在函数中而不是在类中。区别在于，类的静态对象总是会构造和销毁，即使没有不会使用它。而函数的静态对象会在函数第一次调用时创建。类静态对象的另一个问题是C++只保证特定翻译单元内的惊叹对象初始化顺序，对不同翻译单元中静态对象的初始化顺序没有说明。
2. 除了第一次调用外，thePrinter是一个只有一个返回语句的函数，但是却没有被声明为内联。因为内联会在调用的地方用函数体的副本替换，对于非成员函数而言，这个副本还包含了静态对象。因此，内联后，会有多个静态对象。

另一个限制对象数量的方法是记录存在对象的数量并且当对象数量超过阈值的时候在构造函数中抛出异常。示例代码如下
```cpp
class Printer {
public:
  class TooManyObjects{};
  Printer();
  ~Printer();
private:
  static size_t numObjects;
};
size_t Printer::numObjects = 0;
Printer::Printer() {
  if (numObjects >= 1) {
    throw TooManyObjects();
  }
  // normal construction
  ++numObjects;
}
Printer::~Printer() {
  //normal destruction
  --numObjects;
}
```
这个方法比较直观，并且可以容易地修改限制对象的数量。

这个方法也存在一个问题，当Printer类是基类的时候，子类对象的创建也会受到限制，这通常不是我们希望的行为。此外，当Printer对象作为另一个对象的成员时，此对象的创建也会受到限制。如下所示
```cpp
class CPFMachine {
private:
  Printer p;
};
CPFMachine m1; // fine;
CPFMachine m2; // throw TooManyObjects exception
```

此外，私有构造函数还可以阻止继承，如下所示
```cpp
class FSA {
public:
  static FSA * makeFSA() {
    return new FSA();
  }
private:
  FSA();
}
```

thePrinter方法在整个程序执行的过程中使用的是同一个对象，它不允许一下形式的代码，尽管它没有违反单个对象的限制。
```cpp
// create Printer object p1;
// use p1;
// delete p1;
// create Printer object p2;
// use p2;
// delete p2;
```
为了实现这个目的，可以结合上述两类方法，即
```cpp
class Printer {
public:
  class TooManyObjects{};
  static Printer * makePrinter() {
    return new Printer;
  }
  ~Printer() {
    ...
    --numObjects;
  }
private:
  static size_t numObjects;
  Printer() {
    if (numObjects >= 1) {
      throw TooManyObjects();
    }
    ...
    ++numObjects;
  }
  Printer(const Printer& rhs);
};
size_t Printer::numObjects = 0;
```
这个方法容易将修改限制数量。

当程序中存在许多类需要限制数量时，上述的内容需要重复许多遍，因此我们需要一个模板基类，使用模板可以确保每个类的计数互不影响。具体如下所示
```cpp
template<class BeingCounted>
class Counted {
public:
  class TooManyObjects{};
  static int objectCount() {return numObjects;}
protected:
  Counted() {
    init();
  }
  Counted(const Counted& rhs) (
    init();
  )
  ~Counted()；
private:
  static int numObjects;
  static const size_t maxObjects;
  void init() {
    if (numObjects >= maxObjects) {
      throw TooManyObjects();
    }
    ++numObjects;
  }
};
class Printer: private Counted<Printer> {
public:
  using Counted<Printer>::objectCount;
  using Counted<Printer>::TooManyObjects;
};
```
这个基类的构造函数和析构函数是protected的。Printer类是私有继承，如果是公有继承，则析构函数必须是虚的，还会增加额外的大小。Printer类现在使用Counted类来限制对象的数量。
由于计数相关的成员numObjects和maxObjects在Counted中是私有的，且使用的是私有继承，为了让客户可以知道相关的限制信息，在Printer类中重新声明为了公有。

# Requiring or prohibiting heap-based objects

## Requiring Heap-Based Objects
为了限制对象在对上创建，需要有一个方法来阻止客户都调用new。非堆的对象在定义的时候自动创建，则声明周期结束的时候销毁，因此只需要使这些隐式创建和销毁非法即可。

但是，声明构造函数和析构函数为私有则过于严苛了，更好的方法是仅声明析构函数为私有，同时引入一个伪析构函数来访问真正的析构函数。如下例所示
```cpp
class UPNumber {
public:
  UPNumber();
private:
  ~UPNumber();
};
UPNumber n; // error! illegal when n's dtor is later implicitly invoked.
UPNumber *p = new UPNumber; // fine
delete p; // error
p->destroy(); // fine
```

另一个方法是声明构造函数为私有，缺点是类必须将所有的构造函数都声明为私有，其中包括了默认构造函数，复制构造函数等，否则编译器自动生成的构造函数都是共有的。而声明析构函数为私有则比较容易，因为只有一个析构函数。

限制构造函数和析构函数可以阻止在非堆对象的创建，但是也会阻止继承和包含。

继承问题可以通过将析构函数声明为保护的，如下所示
```cpp
class UPNumber{
protected:
  ~UPNumber();
};
class NonNegativeUPNumber : public UPNumber{};  // okay,
class Asset {
public:
  Asset(int initValue) : value(new UPNumber(initValue)) {} //fine
  ~Asset() {
    value->destroy(); // fine
  }
private:
  UPNumber *value;
};
```

## Determining Whether an Object is On The Heap
在上面描述的类中，在非堆上定义NonNegativeUPNumber对象。此时，NonNegativeUPNumber对象的UPNumber部分不在堆上。我们希望派生类的对象的基类部分也在堆上。

我们无法通过修改new运算符，operator new来实现这个限制，考虑一下代码
```cpp
class UPNumber{
public:
  class HeapConstraintViolation {};
  static void * operator new(size_t size);
  UPNumber();
private:
  static bool onTheHeap;
};
bool UPNumber::onTheHeap = false;
void *UpNumber::operator new(size_t size) {
  onTheHeap = true;
  return ::operator new(size);
}
UPNumber::UPNumber() {
  if (!onTheHeap) {
    throw HeapConstraintViolation();
  }
  ...
  onTheHeap = false;
}
```
虽然是一个很好的思路，但是实际上存在一些问题，考虑下面的代码
```cpp
UPNumber *numberArray = new UPNumber[100];
```
第一个问题在于数组的分配调用的是operator new[]而不是operator new。更重要的问题是在数组有100个元素，需要调用100次构造函数，而内存分配只调用一次，因此只有第一次构造函数会调用成功，第二个构造函数时会抛出异常。

第二个问题在于比特设置的事务会失败，考虑一下代码。
```cpp
UPNumber *pn = new UPNumber(*new UPNumber);
```
在表达式右端执行了两个new运算符，因此有两个operator new调用和两个UPNumber构造函数调用。理想的执行顺序为
1. 调用第一个对象的operator new
2. 调用第一个对象的构造函数
3. 调用第二个对象的operator new
4. 调用第二个对象的构造函数

但是，C++并没有保证上述四个函数调用的执行顺序，因此，一些编译器可能会按一下顺序执行
1. 调用第一个对象的operator new
2. 调用第二个对象的operator new
3. 调用第一个对象的构造函数
4. 调用第二个对象的构造函数

此时，在第二个构造函数执行的时候，operator new设置的标志被清除了。因此，不认为第二个对象在堆上建立。

问题在于无法保证operator new和构造函数具有一致性。

在设计中，需要考虑三件事情
1. 不希望再全局作用域定义任何东西，像是operator new和operator delete
2. 效率
3. 具有多个基类或虚拟基类的对象有多个地址

。。。。

# Smart pointers
智能指针类似于哑指针，但是提供了更多的功能。当使用智能指针替换哑指针时，可以控制以下的指针行为
1. 构造和销毁：可以控制创建和销毁指针时的行为。如设置默认值为零来避免未初始化问题；当指针销毁时销毁指向的对象。
2. 复制和赋值：可以控制指针复制和赋值的行为。如采取深复制或浅复制，仅对指针进行复制或赋值，以及不允许复制等。
3. 解引用：当智能指针引用一个智能指针指向的对象时的行为。

智能指针有模板声明，模板类型是指针指向的对象类型。智能指针模板大致如下所示
```cpp
template<class T>
class SmartPtr {
public:
  SmartPtr(T* realPtr = 0);
  SmartPtr(const SmartPtr& rhs);
  ~SmartPtr();
  SmartPtr& operator=(const SmartPtr& rhs);
  T* operator->() const;
  T& operator*() const;
private:
  T *pointee;
};
```

当不允许复制和赋值时，复制构造函数和赋值运算符通常声明为私有（或delete）。智能指针包含一个指向T对象的哑指针。

由于所有权问题，智能指针的复制、赋值和销毁是复杂的。如果智能指针拥有一个对象，智能指针负责销毁这个对象。

# Reference counting
引用计数允许具有相同值的多个对象共享该值的单一表示。使用引用计数有两个动机
1. 简化堆对象的管理。
2. 节约内存，加速程序运行。

先从一个例子开始
```cpp
String a,b,c,d,e;
a = b = c = d = e = "Hello";
```
在普通实现中，上述代码有五个对象，每个对象有自己的值。此时，内存中存在5个Hello对象，可以很明显看的存在冗余。

在理想的情况下，内存中仅存在一个Hello值，所有的String对象共享这个值。当其中一个对象被赋予其他值时，值Hello不能被销毁，因为还有其他四个对象需要它，仅当最后一个对象也不需要的时候，才能销毁，因此，需要存储当前引用此值得对象数量，即引用计数。

## implementing Reference Counting
实现一个引用计数的String首先需要一个地方存放引用计数，引用计数不能存放在String对象中，因为引用计数是计算string值的数量而不是string对象的数量。因此，需要创建一个类来存储值和引用计数，称为StringValue。这个类是仅为String服务的，因此声明在String的私有部分。如下所示
```cpp
class String {
public:

private:
  struct StringValue {
    int refCount;
    char *data;
    StringValue(const char* initValue)；
    ~StringValue();
  };
  StringValue *value;
};
String::StringValue::StringValue(const char* initValue) : refCount(1) {
  data = new char[strlen(initValue) + 1];
  strcpy(data, initValue);
}
String:StringValue::~StringValue() {
  delete [] data;
}
```
StringValue是一个struct，将结构嵌套在类的私有部分是一种方便的方法，可以让该类的所有成员访问该结构，但拒绝其他人的访问。

对于String的实现，我们先从构造函数开始
```cpp
class String {
public:
  String(const char *initValue = "");
  String(const String& rhs);
};
String::String(const char *initValue) : value(new StringValue(initValue)) {}
String::String(const String& rhs) : value(rhs.value) {++value->refCount;}
```
第一个构造函数是使用char\*字符串创建一个string对象。在这种情况下，使用同一个char\*字符串创建string对象时，依然会产生两个StringValue对象。这个问题可以通过追踪已有的StringValue对象来实现。

第二个构造函数是复制构造函数，复制构造函数中不需要分配内存、复制字符串值，仅需要复制一个指针和增加一个引用计数。

String的析构函数也容易实现，在大部分情况下，析构函数仅需要修改引用计数。
```cpp
class String{
public:
  ~String();
};
String::~String() {
  if (--value->refCount == 0) delete value;
}
```

赋值运算符实现如下
```cpp
class String{
public:
  String& operator=(const String& rhs);
};
String& String::operator=(const String& rhs) {
  if (value == rhs.value) {
    return *this;
  }
  if (--value->refCount == 0) {
    delete value;
  }
  value = rhs.value;
  ++value->refCount;
  return *this;
}
```

## Copy-on-Write
考虑数组运算符（即[]），可以对字符串的单个字符进行读写。
```cpp
class String {
public:
  const char& operator[] (int index) const;
  char& operator[] (int index);
}
const char& String::operator[] (int index) const {
  return value->data[index];
}
char& String::operator[] (int index) {
  if (value->refCount > 1) {
    --value->refCount;
    value = new StringValue(value->data);
  }
  return value->data[index];
}
```

常量版本的函数非常简单，而非常量版本的函数则比较复杂，因为非常量版本的函数既可以读又可以写。在进行写操作的时候，不能影响其他共享这个值的对象。但是，在执行的时候无法知道是读还是写，因此，需要假设所有操作都是写操作。

## Pointer, Reference and Copy-on-Write
在上述实现中，存在一个问题，考虑以下代码
```cpp
String s1 = "Hello";
char *p = &s1[1];
```
此时，p指向的是StringValue中的char数组的第二个字符，即指向共享值第二个字符。若通过指针p来修改字符，则会影响所有共享此值得对象。遗憾的是我们无法检测这个问题。

一个解决方案如下
```cpp
class String {
private:
  struct StringValue {
    int refCount;
    bool shareable;
    char *data;
    StringValue(const char *initValue);
  };
public:
  String(const String& rhs);
  char& operator[] (int index);
};
String::StringValue::StringValue(const char *initValue) : refCount(1), shareable(true) {
  data = new char[strlen(initValue) + 1];
  strcpy(data, initValue);
}
String::String(const String& rhs) {
  if (rhs.value->shareable) {
    value = rhs.value;
    ++value->refCount;
  } else {
    value = new StringValue(rhs.value->data);
  }
}
char& String::operator[] (int index) {
  if (value->refCount > 1) {
    --value->refCount = new StringValue(value->data);
  }
  value->shareable = false;
  return value->data[index];
}
```
在每个StringValue中增加了一个标志表明对象是否是共享的，当非常量版本的函数调后，标志关闭，当标志关闭后，就不会在打开。

这个方案减少了对象之间的共享。

## A Reference-Count Base Class
为了将引用计数相关的代码写成一个上下文无关的代码，首先需要创建一个基类，如下所示
```cpp
class RCObject {
public:
  RCObject();
  RCObject(const RCObject &rhs);
  RCObject& operator=(const RCObject& rhs);
  virtual ~RCObject = 0;
  void addReference();
  void removeReference();
  void markUnshareable();
  bool isShareable() const;
  bool isShared() const;
private:
  int refCount;
  bool shareable;
};
RCObject::RCObject() : refCount(0), shareable(true) {}
RCObject::RCObject(const RCObject&) : refCount(0), shareable(true) {}
RCObject& RCObject::operator=(const RCObject &) {return *this;}
RCObject::~RCObject() {}
void RCObject::addReference() {++refCount;};
void RCObject::removeReference() {if (--refCount == 0) delete this;}
void RCObject::markUnshareable() {shareable = false}
bool RCObject::isShareable() const {return shareable;}
bool RCObject::isShared() const {return refCount > 1;}
```
任何想要实现引用计数的类需要继承这个类，上述类封装类引用计数并提供了增加和减少引用计数的函数。此外，还包括可否共享等标志。

这里需要注意一点的是，这个类是StringValue的基类，而不是String的基类。

在构造函数中，将refCount置为0，实际上置为1更合适，因为创建者肯定会引用它。

赋值运算符看上去有点奇怪，它没有做任何事，需要注意的是，这个类是StringValue的基类，改变的是实际的string的值，这个值改变的时候实际上并不会改变引用计数，因此基类的部分完全不会改变。

修改后的string类如下所示
```cpp
class String{
private:
  struct StringValue: public RCObject {
    char *data;
    StringValue(const char *initValue);
    ~StringValue();
  };
};
String::StringValue::StringValue(const char *initValue) {
  data = new char[strlen(initValue) + 1];
  strcpy(data, initValue);
}
String::StringValue::~StringValue() {
  delete [] data;
}
```

## Automating Reference Count Manipulations
在上述实现中，基类提供了操纵引用计数的函数，但是，这些函数依然要在其他类中手动进行调用，我们希望将这部分代码移动到一个可重用的类中，从而使得实际的不需要关心引用计数的细节。

首先必须清楚什么时候引用计数会发生变化，即复制一个指针、给指针重新赋值以及销毁一个指针。如果可以让指针自己检测这些行为的发生并自动地修改引用计数就可以达成目标，即使用智能指针
