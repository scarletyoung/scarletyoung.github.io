---
title: Effective stl
date: 2022/02/20
---

# Choose your containers with care

A quick review of containers

* The standard STL sequence containers: vector, string, deque, and list
* The standard STL associative containers: set, multiset, map and multimap
* The nonstandard sequence containers: slist and rope.
* The nonstandard associative containers: hash_set, hash_multiset, hash_map, and hash_multimap.
* vector\<char\> as a replacement for string
* vector as a replacement for the standard associative containers
* Serveral standard non-STL containers: arrays, bitset, valarray, stack, queue, and priority_queue.

A way of categorizing the STL containers is the contiguous-memory containers and node-based containers.

* Contiguous-memory containers store their elements in one or more chunks of memory, each chunk holding more than one container element. Insertion or erasure will move other element in the chunk. This kind of movement affects both performance and exception safety. The standard contiguous-memory containers are vector, string, and deque.
* Node-based containers store only a single element per chunk of memory. Linked lists, such as list and slish and all the standard associative containers are node-based.

Some questions about choosing among containers

* Do you need to be abled to insert a new element at an arbitrary position in the container? If so, you need a sequence container; associative contaier won't do.
* Do you care how elements are ordered in the container? In not, a hashed container becomes a viable choice.
* Must the container be part of standard C++?
* What category of iterators do you require? If they must be random access iterators, you're limited to vector, deque, and string(rope). If bidirectional iterators are required, slist must be avoided.
* Is it important to avoid movement of existing container elements when insertions or erasures take place? If so, contiguous-memory containers is not suitable.
* Does the data in the container need to be layout-compatible with C? If so, you're limited to vectors
* Is lookup speed a critical consideration? If so, hashed containers, sorted vectors and the standard associative containers are suitable.
* Do you mind if the underlying container uses reference counting? If so, you'll need to avoid string and rope.
* Do you need transactional semantics for insertions and erasures? If so, you'll want to use a node-based container. If you need transactional semantics for multiple-element insertions, you'll want to choose list.
* Do you need to minimize iterator, pointer, and reference invalidation? If so, you'll want to use node-based containers.
* Would it be helpful to have a sequence container with random access iterators where pointers and references to the data are not invalidated as long as nothing is erased and insertions take place only at the ends of the container? If so, ypu'll need to deque.

# Beware the illusion of the container-independent code

...

# Make copying cheap and correct for objects in containers.

Copy in, copy out: When you add an object to a container, what goes into the container is a copy of the object you specify. When you get an object from a container, what you get is a copy of what was contained. 

So, if you fill a container with objects where copying is expensive, the simple act of putting the objects into the container could prove to be a performance bottleneck.

In the presence of inheritance, of course, copying leads to slicing. That is, if you create a container of base class objects and you try to insert derived class objects into it, the derivedness of the objects will be removed as the objects are copied.

An easy way to make copying efficient, correct, and immune to the slicing problem is to create containers of pointers instead of containers of objects. But containers of pointers have their own problems.

# Call empty instead of checking size against zero

You should prefer the construct using empty. The reason is that empty is a constant-time operation for all standard containers, but for some list implementations, size takes linear time.

List has a unique splicing functions

```cpp
list<int> list1, list2;
list.splice(list.end(), list, find(list2.begin(), list2.end(), 5), find(list2.rbegin(), list2.rend(), 10).base());
```

This code above will move all nodes in list2 from the first occurrence of 5 through the last occurrence of 10 to the end of list1.

You want to make the splicing function and size funtion both a constant-time, but this is impossible.

If size is to be constant-time operation, each list member fucntion must update the sizes of the lists on which it operates. That include splice. But the only way for splice to update the sizes of the list it modifies is fo it to count the number of elements being spliced, and donig that would prevent splice from achieving the constant-time performance you want for it.

So, which operator is contant-time depends on its implementations. size may not constant-time, but empty is always constant-time.

# Prefer range member functions to their single-element counterparts

Whenever you have to completely replace the contents of a container, you should think of assignment.

A range member function is a member function that use two iterator parameters to specify a range of elements over which something should be done.

Almost all use of copy where the destination range is specified using an insert iterator can be replaced with calls to range member functions.

Some reasons to prefer range member functions to their single-element counterpart

* It's generally less work to write the code using the range member functions
* Range member functions tend to lead tocode that is clearer and more straightforward.
* For standard sequence containers, application of single-element member functions makes more demands on memory allocators, copies objects more frequently, and/or performs redundant operations compared to range member functions that achieve the same end.

There are three different performance taxes when you using the single-element version of insert member function

1. Unnecessary function call. Inlining may save you from this tax, but may not.
2. Inefficiently move the existing elements for sequence container. For node-based container, there are repeated superfluous assignments to pointer of some nodes in the list
3. Unnecessary memory allocation.

For some type of container, some performance taxex may not exist, but range member functions are never less efficient, so you have nothing to lose by preferring them.

Thera are some range member function

* Range construction: all standard containers offer a constructor of this form
  
  ```cpp
  container::container(InputIterator begin, InputIterator end);
  ```

* Range insertion: all standard sequence containers offer this form of insert
  
  ```cpp
  void container::insert(iterator position, InputIterator begin, InputIterator end);
  ```
  
  Associative containers' counterpart is 
  
  ```cpp
  void container::insert(InputIterator begin, InputIterator end);
  ```

* Range erasure: every standard container offers a range form of erase
  
  ```cpp
  // sequence container
  iterator container::erase(iterator begin, iterator end);
  // associative container
  void container::erase(iterator begin, iterator end);
  ```

* Range assignment: all standard sequence containers offer a range form of assign
  
  ```cpp
  void container::assign(InputIterator begin, InputIterator end);
  ```

# Be alert for C++'s most vexing parse

Suppose you have a file of ints and you'd like to copy those ints into a list. The code below may be used

```cpp
ifstream dataFile("ints.dat");
list<int> data(istream_iterator<int>(dataFile), istream_iterator<int>());
```

The code above will compile, but at runtime, it won't do anything.

Let's see some declarations before taking about why it won't work.
There are three different ways to declare a function f taking a double and returning an int.

```cpp
int f(double d);
int f(double (d));
int f(double);
```

There are three more declerations about a function g taking a pointer of function which takes nothing and returning a double.

```cpp
int g(double (*pf)());
int g(double pf());
int g(double ());
```

Back to the code.

```cpp
list<int> data(istream_iterator<int>(dataFile), istream_iterator<int>());
```

This declares a function data, whose return type is list<int>. The function takes two parameters

* The first parameter is named dataFile, It's type is istream_iterator\<int\>.
* The second parameter has no name. Its type is pointer to function taking nothing and returning an istream_iterator\<int\>.

For C++, anything that can be parsed as a function declaration will be.

The correct code is shown below

```cpp
list<int> data( (istream_iterator<int>(dataFile)), istream_iterator<int>());
```

The reason is that it's not leage to surround a formal parameter declaration with parentheses, but it is legal to surround an argument to a function call with parentheses.

Unfortunately, not all compilers receive this declarations. So, a better solution is that don't use anonymous istream_iterator objects and give those iterators names.

```cpp
ifstream dataFile("ints.dat");
istream_iterator<int> dataBegin(dataFile);
istream_iterator<int> dataEnd();
list<int> data(dataBegin, dataEnd);
```

# When using containers of newed pointers, remember to delete the pointer before the container is destroyed

When containers hold pointers to objects allocated with new, pointer will not automatically release when containers destroyed. The code below will leads to a resource leak.

```cpp
void doSomething() {
  vector<Widget*> vwp;
  for (int i = 0; i < SOME_MAGIC_NUMBERl; ++i) {
    vwp.push_back(new Widget);
  }
}
```

This is a feature. Only you know whether the pointers should be deleted.

You can add a deletion at the end of function, for example

```c++
void doSomething() {
  vector<Widget*> vwp;
  for (int i = 0; i < SOME_MAGIC_NUMBERl; ++i) {
    vwp.push_back(new Widget);
  }
  ...

  for (vector<Widget*>::iterator i = vwp.begin(); i!=vwp.end();++i) {delete *i;}
}
```

This will work, but it is not exception-safe.

A better way is to replace raw point with smart reference-counting opinter objects.

# Never create containers of auto_ptrs

Containers of auto_ptr are prohibited. Code attempting to use them shouldn'd comile.

When you copy an auto_ptr, ownership of the object pointed to by the auto_ptr is transferred to the copying auto_ptr, and the copied auto_ptr is set to NULL. To copy an auto_ptr is to change its value. It can lead to some very surprising behavior. For example

```c++
bool widgetAPCompare(const auto_ptr<Widget>&lhs, const auto_ptr<Widget>& rhs) {
  return *lhs < *rhs;
}
vector<auto_ptr<Widget>> widgets;
sort(widgets.begin(), widgets.end(), widgetAPCompare);
```

This code seems reasonable, but the result need not be reasonable at all. One or more of the auto_ptrs in widgets may have been set to NULL during the sort.

# Choose carefully among erasing options

Suppose you have a standard STL container, c, that holds ints, and you'd like to get rid of all the objects in c with the value 1963. The way to accomplish this task varies from container type to container type: no single approach works for all of them.

If the container is congiguous-memory, the best approach is the erase-remove idiom

```c++
c.erase(remove(c.begin(), c.end(), 1963), c.end());
```

This approach works for lists, too, but the list member function remove is more efficient

When c is a standard associative container, the proper way to approach the problem is to call erase.

Let's get rid of every object which satifies certain predication.

For the sequence containers, all we need to do is repalce each use of remove with remove_if.

```c++
bool badValue(int x);
c.erase(remove_if(c.begin(), c.end(), badValue), c.end());
```

For the standard associative containers. There are two ways to approach the problem, one easier to code, one more efficient.

The easier one is to use remove_copy_if to copy the values we want into a new container, then swaps the contents of the original container with those of the new one

```c++
AssocContainer<int> c;
AssocContainer<int> goodValues;
remove_copy_if(c.begin(), c.end(), inserter(goodValues, goodValues.end()), badValue);
c.swap(goodValues);
```

The efficient one is to write a loop to iterate over the elements in c and erasing elements.

```c++
AssocContainer<int> c;
for (AssocContainer<int>::iterator i = c.begin(); i!=c.end();) {
  if (badValue(*i)) c.erase(i++);
  else ++i;
}
```

We make the problem further. Instead of merely erasing each element for which badValue returns true, we also want write a message to a log file each time an element is erased.
For the associative container, this is easy.

```c++
ofstream logFile;
AssocContainer<int> c;
for (AssocContainer<int>::iterator i = c.begin(); i!=c.end();) {
  if (badValue(*i)) c.erase(i++);
  else ++i;
}
```

Now the vector, string and deque gives us trouble. We can't use the erase-remove idiom any longer. We also can't use the loop, because it yields undefined behavior. For such container, invoking erase not onbly invalidates all iterators pointing to the erased element, it also invalidates all iterators beyong the erased element.
The way to tackle this problme is that

```c++
for (SeqContainer<int>::iterator i = c.begin(); i!=c.end();) {
  if (badValue(*i)) {
    logFile << "Erasing " << *i << '\n';
    i = c.erase(i);
  } else ++i;
}
```

In summary

* To eliminate all objects in a container that have a particular
  value:
  * If the container is a vector, string, or deque, use the erase-remove idiom.
  * If the container is a list, use list::remove.
  * If the container is a standard associative container, use its erase member function.
* To eliminate all objects in a container that satisfy a particular predicate:
  * If the container is a vector, string, or deque, use the erase-remove_if idiom.
  * If the container is a list, use Iist::remove_if.
  * If the container is a standard associative container, use remove_copy_if and swap, or write a loop to walk the container elements, being sure to postincrement your iterator when you pass it to erase.
* To do something inside the loop (in addition to erasing objects):
  * If the container is a standard sequence container, write a loop to walk the container elements, being sure to update your iterator with erase's return value each time you call it.
  * If the container is a standard associative container, write a loop to walk the container elements, being sure to postincrement your iterator when you pass it to erase.

# Be aware of allocator conventions and restrictions

It's important that you know the boundaries of the field before you try to using allocator.

First, allocators provide typedefs for pointers and references in the memory model they defined. In the C++ standard, the default allocator for objects of type T offers the typedefs allocator\<T\>::pointer and allocator\<T\>::reference, and it is expected that user-defined allocators will provide these typedefs, too.

Doing so would require the ability to overload operator. ("operator dot"), and that's not permitted. In addition, creating objects that act like references is an example of the use of proxy objects, and proxy objects lead to a number of problems.

But this is not the main problem.  it's the fact that the Standard explicitly allows library implementers to assume that every allocator's pointer typedef is a synonym for T* and every allocator's reference typedef is the same as T&. Library implementers may ignore the typedefs and use raw pointers and references directly!

Second, allocators are objects, and that means they may have member functions, nested types and typedefs, etc., but the Standard says that an implementation of the STL is permitted to assume that all allocator objects of the same type are equivalent and always compare equal.
This can premit that memory allocated by one allocator object may be safely deallocated by another allocator object. For example

```c++
template<typename T>
class SpecialAllocator{};
typedef SpecialAllocator<Widget> SAW;
list<Widget, SAW> L1;
list<Widget, SAW> L2;
L1.splice(L1.begin(), L2);
```

Without being able to make such an assumption, splicing operations would be more difficult to implement.

But this restriction means that portable allocators may not have any nonstatic data members, at least not any that affect their behavior. And this is a runtime issue. Compilers will not give any tips when you violate this constraint.

Standardization committee allow libraries support non-equal instance, but the behaviors are implementation-defined.
When you want to develop a custom allocator with state, you must

1. know the STL implementations you are using support inequivalent allocators
2. delve into their documentation to determine whether the implementation-defined behavior of non-equal allocators is acceptable to you
3. not concern about porting your code.

The interface between allocators and operator new is different. Both take parameter specifying how much memory to allocate, but in the case of operator new, this parameter specifies a certain number of bytes, while in the case of allocator\<T\>::allocate, it specifies how many T object are to fit in the memory. For return types, operator new returns a void*, while allocator\<T\>::allocate returns a T*.

The difference in return type between operator new and aliocator\<T\>::allocate indicates a change in the conceptual model for uninitialized memory, and it again makes it harder to apply knowledge about implementing operator new to the development of custom allocators.

The most of the standard containers never make a single call to allcator with which they are instantiated.

```c++
list<int> L;  //allcator<int> is never asked to allocate memory
```

This oddity is true for list and all the standard associative containers(set, multiset, map, and multimap). That's because these are node-based containers, i.e., containers based on data structures in which a new node is dynamically allocated each time a value is to be stored.

The list will be made up of nodes, each of which holds a T object as well asw pointers to the next and previous nodes in the list

```c++
template<typename T, typename Allocator=allcator<T>>
class list {
  private:
    Allocator alloc;
    struct ListNode {
      T data;
      ListNode *prev;
      ListNode *next;
    };
};
```

When a new node is added to the list, we need to get memory for it from an allocator, but we don't need memory for a T, we need memory for a ListNode that contains a 1. That makes our Allocator object all but useless, because it doesn't allocate memory for ListNodes, it allocates memory for Ts.

Every allocator template A (e.g., std::allocator, SpecialAllocator, etc.) is expected to have a nested struct template called rebind. rebind takes a single type parameter, U, and defines nothing but a typedef. other. other is simply a name for A\<U\>. As a result, list\<T\> can get from its allocator for T objects (called Allocator) to the corresponding allocator for ListNode objects by referring to Allocator::rebind\<ListNode\>::other.

```c++
template<typename T>
class allocator {
  public:
    template<typename U>
    struct rebind {
      typedef allocator<U> other;
    };
};
```

Let us therefore summarize the things you need to remember if you ever want to write a custom allocator.

* Make your allocator a template, with the template parameter T representing the type of objects for which you are allocating memory
* Provide the typedefs pointer and reference but always have pointer be T* and reference be T&
* Never give your allocator per-object state. In general, allocators should have no nonstatic data members.
* Remember that an allocator's allocate member functions are passed the number of objects for which memory is required, not the number of bytes needed. Also remember that these functions return T* pointers.
* Be sure to provide the nested rebind template on which standard containers depend.

# Understand the legitimate uses of custom allocators

Default STL memory manager is too slow, wastes memory, or suffer excessive fragmentation for you STL need. Or you discover that allocator\<T\> takes precautions to be thread-safe, but you don't want to pay for the synchronization overhead you don't need. Or you know that objects in certain containers are typically used together, so you'd like to place them near one another in a special heap to maximize locality of reference. Or you'd like to set up a unique heap that corresponds to shared memory, then put one or more containers in that memory so they can be shared by other processes. Each of these scenarios corresponds to a situation where custom allocators are well suited to the problem.

A example is shown as below.
Suppose you have special routines modeled after malloc and free for managing a heap of shared memory

```cpp
void* mallocShared(size_t, bytesNeeded);
void freeShared(void* ptr);
```

and you'd like to amke it possible to put the contents of stl containers in that shared memroy

```cpp
template<typename T> 
class SharedMemoryAllocator {
  public:
    pointer allocate(size_type numObjects, const void* localityHint=0) {
      return static_cast<pointer>(mallocShared(numObjects * sizeof(T)));
    }
    void deallocate(pointer ptrToMemory, size_type numObjects) {freeShared(ptrToMemory);}
};
```

# Have realistic expectations about the thread safety of STL containers

When it comes to thread safety and STL containers, you can hope for a library implementation that allows multiple readers on one con tainer and multiple writers on separate containers. You can't hope for the library to eliminate the need for manual concurrency control, and you can't rely on any thread support at all.

# Prefer vector and string to dynamically allocated arrays

The reason is

1. vector and string eliminate the burdens about release memory.
2. vector and string are full-fledged STL sequence containers, so they put at your disposal the complete arsenal of STL algorithms that work on such containers.

If you use reference-counted strings in a multithreaded environment, support for thread safety may be a preformance bottleneck.

To determine whether you're using a reference-counting implementation for string, it's often easiest to consult the documentation for your library. An alternative is to look at the source code for your libraries' implementations of string.

If the string implementations available to you are reference counted and you are running in a multithreaded environment where you've determined that string's reference counting support is a performance problem, you have at least three reasonable choices

1. check to see if your library implementatio nis one that makes it possible to disable reference counting, ofthe by changing the value of a preprocessor variable. This won't be portable.
2. find or develop an alternative string implementation that doesn't use reference counting
3. consider using a vector\<char\>

# Use reserve to avoid unnecessary reallocations

For vector and string, growth is handled by doing the equivalent of a realloc whenever more space is needed. This realloc-like operation has four parts

1. Allocate a new block of memory that is some multiple of the containers current capacity. In most implementations, their capacity is doubled each time.
2. Copy all the elements from the container's old memory into its new memory
3. Destroy the objects in the old memory.
4. Deallocate the old memory.

When reallocation happens, all iterators, pointers and references into the vector or string are invalidated.

The key to avoiding reallocations is to use reserve to set a container's capacity to a sufficiently large value as soon as possible.

There are common ways to use reserve to avoid unneeded reallocations

1. when you know exactly or approximately how many elements will ultimately end up in your container.
2. to reserve the maximum space you could ever need, and once you've added all your data, trim off any excess capacity.

# Be aware of variations in string

Every string implementation holds the following information

* This size of the string.
* The capacity of the memory holding the string's.
* The value of the string.

A string may hold

* A copy of its allocator.

string implementations that depend on reference counting also contain the reference count for value.

string implementations have more degrees of freedom than are apparent at first glance

* string values may or may not be reference counted.
* string objects may range in size from one to at least seven times the size of char* pointers.
* Creation of a new string value may require zero, one, or two dynamic allocations
* string objects may or may not share information on the string's size and capacity.
* strings may or may not support per-object allocators.
* Different implementations have different policies regarding minimum allocations for character buffers.

At the same time, if you're to make effective use of the STL, you need to be aware of the Variability in string implementations, especially if you're writing code that must run on different STL platforms and you face stringent performance requirements.

# Know how to pass vector and string data to legacy APIs

If you have a vector v and you need to get a pointer to the data in v that can be view as an array, just use &v[0]. However, if v.size() is zero, and &v[0] attempts to produce a pointer to something that does not exist.

You may use v.begin() in place of &v[0]. But the return type of begin is an iterator, not a pointer, and you should never use begin when you need to get a pointer to the data in a vector.

You can change the vector value through &v[0], but you must not try to create new elements in a vector's unused capacity. If it does, v will become internally inconsistent, because it won't know its correct size any longer.

If you have a vector that you'd like to initialize with elements from a C API, you can take advantage of the underlying layout compatibility of vectors and arrays by passing to the API the storage for the vector's elements

```cpp
size_t fillArray(double *pArray, size_t arraySize);
vector<double> vd(maxNumDoubles);
vd.resize(fillArray(&vd[0], vd.size()));
```

There are two reasons that the approach to getting a pointer to container data that works for vector isn't reliable for strings

1. the data for strings are not guaranteed to be stored in contiguous memory
2. the internal representation of a string is not guaranteed to end with a null character.

For a string s, the corresponding incantation is simply s.c_str(). Unlike vector, this works even if the string is of length zero. c_str wil return a pointer to a null character in this case.

You can't modify char* returned by c_str. Because there is no guarantee that c_str yields a pointer to the internal representation of the string data. It could return a pointer to an unmodifiable copy of the string's data.

If you want to initialize a string with data from a C API, however, you can do it easily enough. Just have the API put the data into a vector\<char\>, then copy the data from the vector to the string.

```cpp
size_t fillString(char* pArray, size_t arraySize);
vector<char> vc(maxNumChars);
size_t charsWritten = fillString(&vc[0], vc.size());
string s(vc.begin(), vc.begin() + charsWritten);
```

# Use "the swap trick" to trim excess capacity

Above c++ 11, just use shrink_to_fit member function.

In c++98, use the swap trick

```c++
class Contestant {};
vector<Contestant> contestants;
vector<Contestant>(contestants).swap(contestants);
```

The expression vector\<Contestant\>(contestants) creates a temporary vector that is a copy of contestants; vector's copy constructor does the work. However, vector's copy constructor allocates only as much memory as is needed for the elements being copied, so this temporary vector has no excess capacity. We then swap the data in the temporary vector with that in contestants, and by the time we're done, contestants has the trimmed capacity of the temporary, and the temporary holds the bloated capacity that used to be in contestants. At that point (the end of the statement), the temporary vector is destroyed, thus freeing the memory formerly used by contestants. 

The real effect of this method is depended on the stl implementation.

# Avoid using vector\<bool\>

There are two things wrong with vector\<bool\>

1. it's not an STL container
2. it's doesn't hold bools

For a container c, if c contains objects of types T and c support operator[], the following must compile:

```c++
T *p = &c[0];
```

vector\<bool\> does not fulfill this requirement. It contains not actual bools, but a packed representation of bool that is designed to save space. In a typical implementation, each bool stored in the vector occupies a single bit, and an eight-bit byte holds eight bools.

The most important difference between true bool and bitfields is that you can not create pointers or reference to  individual bits.

vector\<bool\>::operator[] returns an object that acts like a reference to a bit, a so-called proxy object.

```c++
template<typename Allocator>
vector<bool, Allocator> {
  public:
    class reference {};
    reference operator[] (size_type n);
};
```

The standard library provides two alternatives that suffice almost all the time.

1. deque\<bool\>: it offers almost everything a vector does and really contains bools.
2. bitset: it isn't an stl container, but it's part of the standard C++ library. Its size is fixed during compilation and offers no support for iterator. However, it offers a number of special member functions that make sense for collections of bits.

# Understand the difference between equality and equivalence

The notion of equally is based on operator==. If the expression "x==y" returns true, x and y have equal values.

Equivalence is based on the relative ordering of object values in a sorted range. Two objects x and y have equivalent values with respect to the sort order used by an associative container c if neither precedes the other in c's sort order.

```cpp
!(w1<w2)&&!(w2<w1)
```

# Specify comparison types of associative containers of pointers

If you want the string* pointers to be stored in the set in an order determined by the string values, you can't use the default comparison functor class less\<string*\>. You must instead write your own comparison functor class, one whose objects take string* pointers and order them by the values of the strings they point to.

```cpp
struct StringPtrLess:  public binary_function<const string*, const string*, bool> {
    bool operator()(const string* ps1, const string* ps2) const {
        return *ps1 < *ps2;
    }
};
```

Then, you can use StringPtrLess as ssp's comparison type

```cpp
typedef set<string*, StringPtrLess> StringPtrSet;
StringPtrSet ssp;
```

If you want to use an algorithm instead, you could write a function that knows how to dereference string* pointers, then use that function in conjunction with for_each

```cpp
void print(const string* ps) {
    cout << *ps << endl;
}
for_each(ssp.begin(), ssp.end(), print);
```

Anytime you create associative containers of pointers, figure you're probably going to have to specify the container's comparison type, too. Most of the time, your comparison type will just dereference the pointers and compare the pointed-to objects.

That being the case, you might as well keep a template for such comparison functors close at hand.

```cpp
struct DereferenceLess {
    template<typename PtrType>
    bool operator()(PtrType pT1, PtrType pT2) const {return *pT1 < *pT2;}
};
```

# Always have comparison functions return false for equal values

The title is enough.

# Avoid in-place key modification in set and multiset

For objects of type set\<T\> or multiset\<T\>, the type of the elements stored in the container is simply T, not canst T. Hence, the elements in a set or multiset may be changed anytime you want to.

The reason why the elements in a set or multiset aren't const is that
```cpp
class Employee {
  public:
    const string& title() const;
    void setTitle(const string& title);
    int idNumber() const;
};
struct IDNumberLess : public binary_function<Employee, Employee, bool> {
  bool operator()(const Employee& lhs), const Employee& rhs) const {return lhs.idNumber() < rhs.idNumber();}
};
typedef set<Employee, IDNumberLess> EmpIDSet;
EmpIDSet se;
Employee selectedId;
EmpIDSet::iterator i = se.find(selectedID);
if (i != se.end()) {
  i->setTitle("Corporate Deity");  
}
```

However, for some stl implementation, this code above won't compile. 

Because of the equivocal state of the Standard and the differing interpretations it has engendered, code that attempts to modify elements in a set or multiset isn't portable. So, the guideline you should follow is 

* If portability is not a concern, you want to change the value of an element in a set or multiset, and your STL implementation will let you get away with it, go ahead and do it. Just be sure not to change a key part of the element, i.e., a part of the element that could affect the sortedness of the container.

* If you value portability, assume that the elements in sets and multisets cannot be modified, at least not without a cast.

We know that it is reasonable to change a non-key portion of an element in a set or multiset. The way to do it correctly and portably is that you must cast to a reference
```cpp
EmpIDSet::iterator i = se.find(selectedID);
if (i != se.end()) {
  const_cast<Employee&>(*i).setTitle("Corporate Deity");
}
```

The two following way will fail to behave the way people often expect it to.
```cpp
// version one
if (i != se.end()) {
  static_cast<Employee>(*i).setTitle("Corporate Deity");
}
// version two
if (i != se.end()) {
  ((Employee)(*i)).setTitle("Corporate Deity");
}
```
Actually, the two verions are equivalent. Both of these compile, but in runtime, they don't modify *i. The result of the cast is a temporary anonymouse object that is a copy of *i, and setTitle is invoked on the anonymous object. Both syntactic forms are equivalent to this:
```cpp
if (i != se.end()) {
  Employee tmpCopy(*i);
  tmpCopy.setTitle("Corporate Deity");
}
```

So, it is important to cast to a reference.

If you want to change an element in a set, multiset, map or multimap in a way that always works and is always safe, do it in five simple steps
1. Locate the container element you want to change.
2. Make a copy of the element to be modified.
3. Remove the element from the container, typically via a call to erase
4. Modify the copy so it has the value you want to be in the container
5. Insert the new value into the container.

# Consider replacing associative containers with sorted vectors.
If lookup speed is really important, it's almost certainly worthwhile to consider the nonstandard hashed containers as well.

If you want to make effective use of the STL, you need to understand when and how a vector can offer faster lookups than a standard associative container.

The standard associative containers are typically implemented as balanced binary search trees. A balanced binary search tree is a data structure that is optimized for a mixed combination of insertions, erasures, and lookups.

That is, it's designed for applications that do some insertions, then some lookups, then maybe some more insertions, then perhaps some erasures, then a few more lookups, then more insertions or erasures, then more lookups, etc. 

In general, there's no way to predict what the next operation on the tree will be.

But many applications use their data structures in three distinct phases
1. Setup. Create a new data structure by inserting lots of elements into it. During this phase, almost all operations are insertions and erasures. Lookups are rare or nonexistent.
2. Lookup.  Consult the data structure to find specific pieces of information. During this phase, almost all operations are lookups. Insertions and erasures are rare or nonexistent.
3. Reorganize. ModifY the contents of the data structure, perhaps by erasing all the current data and inserting new data in its place. Behaviorally, this phase is eqUivalent to phase 1. 

In these applications, a sorted vector is likely to offer better performance than an associative container. There are two reason
1. sorted vector can stored more object than associative container
2. Unlike contiguous-memory containers such as vector, node-based containers find it more difficult to guarantee that container elements that are close to one another in a container's traversal order are alose to another in physical memory.

Storing data in a sorted vector is likely to consume less memory than storing the same data in a standard associative container, and searching a sorted vector via binary search is likely to be faster than searching a standard associative container when page faults are taken into account.

The big drawback of a sorted vector is that it must remain sorted.

When you decide to replace a map or multimap with a vector. You'll need to write a custom comparison function for your pairs, because pair's operator\< looks at both components of the pair.
You also will need a second comparison function for performing lookups. The comparison function used for sorting will take two pair objects, but lookups are performed given only a key value. The comparison function for lookups, then, must take an object of the key type (the value being searched for) and a pair (one of the pairs stored in the vector) - two different types.  As an additional twist, you can't know whether the key value or the pair will be passed as the first argument, so you really need two comparison functions for lookups:
```cpp
typedef pair<string, int> Data;
class DataCompare {
  public:
    bool operator()(const Data& lhs, const Data& rhs) const {
      return keyLess(lhs.first, rhs.first);
    }
    bool operator()(const Data& lhs, const Data::first_type& k) const {
      return keyLess(lhs.first, k);
    }
    bool operator()(const Data::first_type& lhs, const Data& rhs) const {
      return keyLess(k, rhs.first);
    }
  private:
    bool keyLess(const Data::first_type& k1, const Data::first_type& k2) const {
      return k1 < k2;
    }
};
```

# Choose carefully between map::operator[] and map::insert when efficiency is important
map::operator[] is designed to facilitate "add or update" functionality. Given the expression
```cpp
map[k] = v;
```
checks to see if the key k is already in the map. If not, it's added, along with v as its corresponding value. If k is already in the map, its associated value is updated to v.

The operator[] returns a reference to the value object associated with k, v is then assigned to the object to which the reference refers. When an existing key's associated value is being updated because there's already a value object ot which operator[] can return a reference. But if k isn't yet in the map, it creates one from scratch by using the value type's default constructor. operator[] then returns a reference to this newly-created object.

When an "add" is perfromed, insert is more efficient than operator[]. However, the situation is reversed when we do an update.

When we call insert, we must construct and destruct an object of value_type. That coast us a pair constructor and desctructor. operator[] uses no pair object, so it constructs and destructs no pair and no object.

If you're updating an existing map element, operator[] is preferable, but if you're adding a new element, insert has the edge.

# Familiarize yourself with the nonstandard hashed containers
SGI
Dinkumware

# Prefer iterator to const_iterator, reverse_iterator, and const_reverse_iterator
The type iterator and reverse_iterator act like a T*, while const_iterator and const_reverse_iterator act like a const T*
Incrementing an iterator or a const_iterator moves you to the next element in the container in a traversal from front to back, while reverse_iterator and const_reverse_iterator from back to front.

Every standard continer offers functions analogouse to the following, though the return types vary with the container type.
```cpp
iterator insert(iterator position, const T& x);
iterator erase(iterator position);
iterator erase(iterator rangeBegin, iterator rangeEnd);
```
These functions demand parameters of type iterator, not const_iterator, not reverse_iterator, not const_reverse_iterator.

There are implicit conversions from iterator to const_iterator, from iterator to reverse_iterator, and from reverse_iterator to const_reverse_iterator.
A reverse_iterator may be converted into an iterator by using the base member function, same as const_reverse_iterator.

There is no way to get from a const_iterator to an iterator or from a const_reverse_iterator to a reverse_iterator. It means if you have a const_iterator or a const_reverse_iterator, you'll find it difficult to use those iterators with some container member functions.

The reason for prefering iterators to const and reverse iterators is that
* Some versions of insert and erase require iterators.
* It's not possible to implicitly convert a const iterator to an iterator.
* Conversion from a reverse_iterator to an iterator may require iterator adjustment after the conversion. 

The choice between iterator and reverse_iterator is depended on what you need. If you need a back-to-front traversal, you choose reverse_iterator and use base to convert it to an iterator when you want to make calls to container member functions that require iterators.

However, there are reasons to choose iterators even when you could use a const_iterator.
Let me show you code
```cpp
typedef deque<int> IntDeque;
typedef IntDeque::iterator Iter;
typedef IntDeque::const_iterator ConstIter;
Iter i;
ConstIter ci;

if (i == ci) {}
```

In the comparison of two type of iterator, the iterator should be implicitly converted into a const_iterator, and the comparision should be performed between two const_iterators.

With well-designed STL implementations, this is precisely what happends, but with other implementations, the code will not compile. The reason is that such implementation declare operator== for const_iterator as a member function instead of as a non-member function. The workaround is to swap the order of the iterators like
```cpp
if(ci == i) {}
```

This kind of problem can arise whenever you mix iterators and const_iterators (or reverse_iterators and const_reverse_iterators) in the same expression. The easiest way to guard against these kinds of problems is to minimize your mixing of iterator types, and it means that prefer iterators to const_iterators

# Use distance and advance to convert a container's const_iterators to iterators
Try to cast a const_iterator to an iterator will not compile. The reason is that iterator and const_iterator are different classes.

The cast might compile if the iterators' container were a vector or a string. That is because it is common for implementations of these containers to use pointers as iterators. Even in this cases, casting const iterators to iterators is ill-advised, beacuse its portability is doubtful.

For all container, reverse_iterators and const_reverse_iterator are true class, so you can't const_cast a const_reverse_iterator to a reverse_iterator.

There is a safe, portable way to get its corresonding iterator
```cpp
typedef deque<int> IntDeque;
typedef IntDeque::iterator Iter;
typedef IntDeque::const_iterator ConstIter;
IntDeque d;
ConstIter ci;
Iter i(d.begin());
advance(i, distance<ConstIter>(i, ci));
```
This approach is so simple and direct, it's startling. To get an iterator pointing to the same container element as a const_iterator, create a new iterator at the beginning of the container, then move it forward it until it's as far from the beginning of the container as the const_iterator is.

This technique is as efficient as the iterators allow it to be. For random access iterators, it's a constant-time operation. For bidirectional iterators, it's a linear-time operation.

# Understand how to use a reverse_iterator's base iterator
```cpp
vector<int> v;
v.reserve(5);
for (int i = 1; i <= 5; ++i) {
  v.push_back(i);
}
vector<int>::reverse_iterator ri = find(v.rbegin(), v.rend(), 3);
vector<int>::iterator i(ri.base());
```
After executing this code, things can be thought of as looking like this:
![](../img/Effective_STL_Figure_1.png)

To perform insertions or erasures, you must convert reverse_iterators into iterators via base, then use the iterators to get the jobs done.
* For purposes of insertion, ri and ri.base() are equivalent, and ri.base() is truly the iterator corresponding to ri.
* For purposes of erasure, ri and ri.base() are not equivalent, and ri.base() is not the iterator corresponding to ri.

To do erasure, an intuitive code is like this
```cpp
vector<int> v;
vector<int>::reverse_iterator ri = find(v.rbegin(), v.rend(), 3);
v.erase(--ri.base());
```

This code will work with every standard container except some implementation of vector and string. In such implementations, iterators are implementated as build-in pointers, so the result of ri.base() is a pointer. Both C and C++ dictate that pointers returned from functions shall not be modified, so for STL platforms where string and vector iterators are pointers, expressions like --ri.base() won't compile.

A portably way is to increment the reverse_iterator and then call base
```cpp
v.erase((++ri).base());
```

# Consider istreambuf_iterators for character-by-character input
Copy a text file into a string object
```cpp
ifstream inputFile("interestingData.txt");
string fileData((istream_iterator<char>(inputFile)), istream_iterator<char>());
```

This approach fails to copy whitespace in the file into the string. That's because istream_iterators use operator<< functions to do the actual reading, and by default, operator<< functions skip whitespace.

If you'd like to retain the whitespace, all you need to do is override the defualt
```cpp
ifstream inputFile("interestingData.txt");
inputFile.unset(ios::skipws);
string fileData((istream_iterator<char>(inputFile)), istream_iterator<char>());
```

A more efficient approach is to use istreambuf_iterators. istreambuf_iterator<char> objects go straight to the stream's buffer and read the next character directly.
```cpp
ifstream inputFile("interestingData.txt");
string fileData((istreambuf_iterator<char>(inputFile)), istreambuf_iterator<char>());
```

If you need to read the characters in a stream one by one, you don't need the power of formatted input, and you care about how long it takes to read the stream, typing three extra characters per iterator is a small price to pay for what is often a significant increase in performance. For unformatted character-by-character input, you should always consider istreambuf_terators.

# Make sure destination ranges are big enough
```cpp
int transmogrify(int x);
vector<int> values;
vector<int> result;
transform(values.begin(), values.end(), result.end(), transmogrify);
```
This code above has a bug. transform will apply transmogrity to values[0] and assign the result to *results.end(), and go on. This can lead only to disaster, because there is no object at *results.end(). The call to transform is wrong, because it's asking for assignments to be made to object that don't exist.

Programmers who make this kind of mistake almost always intend for the results of the algorithm they're calling to be inserted into the destination container. 

In this example, the way to say "please put transform's results at the end of the container called results" is to call back_inserter to generate the iterator specifYing the beginning of the destination range
```cpp
vector<int> results;
transform(values.begin(), values.end(), back_inserter(results), transmogrify);
```
The iterator returned by back_inserter causes push_back to be called, so you may use back_inserter with any container offering push_back.

If you'd prefer to have an algorithm insert things at the front of a container you can use front_inserter.

Because front_inserter causes each object added to results to be push_fronted, the order of the objects in results will be the reverse of the order of the corresponding objects in values.
If you want transform to put its output at the front of results, but you also want the output to be in the same order as the corresponding objects in values, just iterate over values in reverse order:
```cpp
list<int> results;
transform(values.rbegin(), values.rend(), front_nserter(result), transmogrify);
```

Regardless of whether you use back_inserter, fronCinserter, or inserter, each insertion into the destination range is done one object at a time. This can be expensive for contiguous-memory containers. And there's nothing you can do to change that.
When the container into which you're inserting is a vector or a string, you can minimize the expense by calling reserve in advance. You'll still have to absorb the cost of shifting elements up each time an insertion takes place, but at least you'll avoid the need to reallocate the container's underlying memory.

Even if you want to overwrite the values of existing container elements instead of inserting new ones, you still need to make sure your destination range is big enough.
You must either use resize to make sure your destination range is big enough.
```cpp
vector<int> values, results;
if (result.size() < values.size()) {
  result.resize(values.size());
}
transform(values.begin(), values.end(), results.begin(), transmogrify);
```
or you can clear results and then use an insert iterator
```cpp
results.clear();
results.reserve(values.size());
transform(values.begin(), values.end(), back_inserter(results), transmgorify);
```

Whenever you use an algorithm requiring specification of a destination range, ensure that the destination range is big enough already or is increased in size as the algorithm runs.

# Know your sorting options
sort is a wonderful algorithm, but sometimes you don't need a full sort. For example, if you would like to select the 20 highest-quality Widgets in the vector, you need to do only enough sorting to identify the 20 best Widgets. The remainder can remain unsorted. What you need is a partial sort
```cpp
bool qualityCompare(const Widget& lhs, const Widget& rhs);
partial_sort(widgets.begin(), widgets.begin() + 20, widgets.end(), qualityCompare);
```

If all you care about is the 20 best Widgets, bu you don't care their orders, nth_element is better.
```cpp
nth_element(widgets.begin(), widgets.begin()+20, widgets.end(), qualityCompare);
```
nth_element sorts a range so that the element at position n is the one that would be there if the range had been fully sorted.
The only problem is that partial_sort and nth_element are not stable.

nth_element can also be used to find the median value in a range or to find the value at a particular percentile.

The algorithms sort, stable_sort, partial_sort and nth_element require random access iterators, so they may be applied only to vectors, strings, deques, and arrays.

You can use lsit::sort to perform stable sort, but you can not use partial_sort or nth_element directly.
There are three indirect aproach
1. copy the elements into a container with random acces iterators.
2. create a container of list::iterators, and use algorithm on that container, then access the list elements via the iterators.
3. se the information in an ordered container of iterators to iteratively splice the list's elements into the positions you'd like them to be in.

partition and stable_partition requires only bidirectional iterators. You can use partition and stable_partition with any of the standard sequence containers.

In summary
* If you need to perform a full sort on a vector, string, deque or array, you can use sort or stable_sort
* If you have a vector, string, deque, or array and you need to put only the top n elements in order, partial_sort is available.
* If you have a vector, string, deque, or array and you need to identify the element at position n or you need to identify the top n elements without putting them in order, nth_element is at your beck and call.
*  If you need to separate the elements of a standard sequence container or an array into those that do and do not satisfy some criterion, you're probably looking for partition or stable_partition.
*  If your data is in a list, you can use partition and stable_partition directly, and you can use list::sort in place of sort and stable_sort. If you need the effects offered by partial_sort or nth_element, you'll have to approach the task indirectly.

# Follow remove-like algorithms by erase if you really want to remove somehting
The declaration for remove is that
```cpp
template<class ForwardIterator, class T>
ForwardIterator remove(ForwardIterator first, ForwardIterator last, const T& value);
```
remove receives a pair of iterators identifying the range of elements over which it should operator. It does not konw which container holds the elements it's looking at.

And the only way to eliminates elements from a container is to call a member function on that container, almost always some form of erase.

Thus, it is not possible for remove to eliminate elements from a container. That is why removeing elements from acontainer never changes the number of elements in the containers.

What remove really do is to move elements in the range it's given untial all the unremoved elements are at the front of the range. It returns an iterator pointing one past the last unremoved element.
For example, a vector is like this
![](../img/Effective STL/Figure_2.png)
And we execute a remove call like this, and store remove's return value in a new iterator called newEnd
```cpp
vector<int>::iterator newEnd(remove(v.begin(), v.end(), 99));
```
After the call, the v looks like
![](../img/Effective STL/Figure_3.png)

remove walk down the range, overwriting values that are to be removed with later values that are to be retained.

If you really want to remove something from container, use erase like this
```cpp
vector<int> v;
v.erase(remove(v.begin(), v.end(), 99), v.end());
```

There are two other remove-like algorithms: remove_if and unique.

# Be wary of remove-like algorithm on containers of pointers
If you store raw pointers in a container, you can not use erase and remove to remove values in the container. This will cause resource leak.

The reason explains in the item above. remove will overwrite values that are to be removed with later values that are to be retained. The pointer in the container is overwrited and the object that the pointer point to is not release.

In this case, you can use smart pointer, manual deleteion or some other technique.

# Note which algorithm expect sorted ranges
The algorithms that require the data on which they operate to be sorted
* binary_search: using binary search to look for values. guarantee logarithmic-time lookup only when they are passed random accesss iterators.
* lower_bound
* upper_bound
* equal_range
* set_union: require sorted range for linear-time performance.
* set_intersection
* set_difference
* set_symmetric_difference
* merge: read two sorted ranges and produce a new sorted range containing all the elements from both source ranges
* inplace_merge
* includes: require sorted range for linear-time performance.

The algorithms are typically used with sorted ranges, though they don't require them.
* unique: its behavior like remove. So, if you want to unique to eliminate all duplicates from range, you must first make sure that all duplicate values are next to one another.
* unique_copy

If you pass a sorted range to an algorithm that also takes a comparison function, be sure that the comparison function you pass behaves the same as the one you used to sort the range.
```cpp
vector<int> v;
sort(v.begin(), v.end(), greater<int>());
bool exists = binary_search(v.begin(), v.end(), 5, greater<int>());
```

All the algorithms that require sorted ranges determine whether two values are "the same" by using equivalence, just like the standard associative con tainers. In contrast, the default way in which unique and unique_copy determine whether two objects are "the same" is by using equality, though you can override this default by passing these algorithms a predicate defining an alternative definition of "the same."

# Implement simple case-insensitive string comparisons via mismatch or lexicographical_compare 
Case-insensitive string comparisons are either really easy or really hard, depending on how general you want to be. If you're willing to ignore internationalization issues and restrict your concern to the kinds of strings strcmp is designed for, the task is easy. If you want to be able to handle strings of characters in languages where strcmp wouldn't apply or where programs use a locale other than the default, the task is very hard.

Programmers desiring case-insensitive string comparisons often need two different calling interfaces, one similar to strcmp, the other akin to operator\<.

First, we need a way to determine whether two characters are the same, except for their case.
```cpp
int ciCharCompare(char c1, char c2) {
  int lc1 = tolower(static_cast<unsigned char>(c1));
  int lc2 = tolower(static_cast<unsigned char>(c2));
  if (lc1 < lc2) return -1;
  if (lc1 > lc2) return 1;
  return 0;
}
```

In both C and C++, char mayor may not be signed, and when char is signed, the only way to ensure that its value is representable as an unsigned char is to cast it to one before calling tolower. That explains the casts in the code above. It also explains the use of int instead of char to store tolower's return value.

Case-insenstive string comparison functions
```cpp
int ciStringCompareImpl(const string& s1, const string& s2) {
  typedef pair<string::const_iterator, string::const_iterator> PSCI;
  PSCI p = mismatch(s1.begin(), s1.end(), s2.begin(), not2(ptr_func(ciCharCompare)));
  if (p.first == s1.end()) {
    if (p.second == s2.end()) return 0；
    else return -1;
  }
  return ciCharCompare(*p.first, *p.second);
}
int ciStringCompare(const string& s1, const string& s2) {
  if (s1.size() <= s2.size()) return ciStringCompareImpl(s1, s2);
  else return -ciStringCompareImpl(s1, s2);
}
```
mismatch function returns a pair of iterators indicating the locations in the ranges where corresponding characters first fail to match


The second approach is
```cpp
bool ciCharLess(char c1, char c2) {
  return tolower(static_cast<unsigned>(c1)) < tolower(static_cast<unsigned>(c1));
}
bool ciStringCompare(const string& s1, const string& s2) {
  return lexicographical_compare(s1.begin(), s1.end(), s2.begin(), s2.end(), ciCharLess)
}
```

lexicographical_compare is a generalized version of strcmp. lexicographical_compare works with ranges of values of any type.lexicographical_compare may be passed an arbitrary predicate that determines whether two values satisfy a user-defined criterion.

In standard C library, there is case-insensitive string comparison function called stricmp or strcmpi. If you're willing to sacrifice some portability, you know that your strings never contain embedded nulls, and you don't care about internationalization, you may find that the easiest way to implement a case-insensitive string comparison doesn't use the STL at all.

# Understand the proper implementation of copy_if
copy_if add in C++11.

The implementation is like
```cpp
template<typename InputIterator, typename OutputInterator, typename Predicate>
OutputIterator copy_if(InputerIterator begin, InputIterator end, OutputIterator destBegin, Predicate p) {
  while(begin != end) {
    if (p(*begin)) *destBegin++ = *begin;
    ++begin;
  }
  return destBegin;
}
```

# Use accumulate or for_each to summarize ranges
Sometimes you need to boil an entire range down to a single number, or, more generally, a single object. 

* count: how many elements are in a range.
* count_if: how many elements satisfy a predicate
* min_element/max_element: The minimum and maximum values in a range.

If you need to summarize a range in some custom manner, you need something more flexible. It's called accumulate and located in \<numeric\>

accumulate exists in two forms
1. taking a pair of iterators and an initial value returns the initial value plus the sum of the values in the range demarcated by the iterators
   ```cpp
   list<double> ld;
   double sum = accumulate(ld.begin(), ld.end(), 0.0);
   ```
   you can use it even with istream_iterators and istreambuf_iterators.
2. taking an initial summary value and an arbitrary summarization function.
   To compute the sum, accumulate needs to know two things. First, it must know the starting sum. Second, it must know how to update this sum.
   ```cpp
   string::size_type stringLengthSum(string::size_type sumSoFar, const string& s) {
     return sumSoFar + s.size();
   }
   set<string> ss;
   string::size_type lengthSum = accumulate(ss.begin(), ss.end(), 0, stringLengthSum);
   ```

Like accumulate, for_each takes a range and a function (typically a function object) to invoke on each element of the range, but the function passed to for_each receives only a single argument (the current range element), and for_each returns its function when it's done.  Significantly, the function passed to (and later returned from) for_each may have side effects.

for_each differs from accumulate in two primary ways. First, the name accumulate suggests an algorithm that produces a summary of a range. for_each sounds like you simply want to do something to evelY element of a range. Using for_each to summarize a range is legitimate, but it's not as clear as accumulate.

Second, accumulate returns the summary we want directly, while for_each returns a function object, and we must extract the summary information we want from this object. In C++ terms, that means we must add a member function to the functor class to let us retrieve the summary information we're after.
```cpp
struct Point{};

class PointAverage : public unary_function<Point, void> {
  public:
    PointAverage(): xSum(0), ySum(0), numPoints(0) {}
    void operator()(const Point& p) {
      ++numPoints;
      xSum += p.x;
      ySum += p.y;
    }
    Point result() const {
      return Point(xSum/numPoints, ySum/numPoints);
    }
  private:
    size_t numPoints;
    double xSum;
    double ySum;
};

list<Point> lp;
Point avg = for_each(lp.begin(), lp.end(), PointAverate()).result();
```

# Design functor classes for pass-by-value
Neither C nor C++ allows you to truly pass functions as parameters to other functions. Instead, you must pass pointers to functions. For example, 
```cpp
void qsort(void *base, size_t, nmemb, size_t size, int(*cmpfcn)(const void*, const void*));
```
The function pointers are passed by value.

STL function objects are modeled after function pointers, so function objects are passed by value when passed to and from functions.
```cpp
Function for_each(InputIterator first, InputIterator last, Function f);
```

Although, you can explicity specify the parameter types like this
```cpp
class DothingSomething : public unary_function<int, void>{
  void operator()(int x){}
};
typedef deque<int>::iterator DequeIntIter;
deque<int> di;
DoSomething d;
for_each<DequeIntIter, DoSomething&>(di.begin(), di.end(), d);
```
Users of the STL almost never do this kind of thing and some implementations of some STL algorithms won't even compile if function objects are passed by reference.

Thus, you should make sure that your function objects behave well when passed by value. This implies two things
1. Your function objects need to be small.
2. Your function objects must be monomorphic. It means that they must not use virtual functions.

Surely there's a way to let function objects be big and/or polymorphic, yet still allow them to mesh with the pass-functors-by-value convention that pervades the STL.

Take the data and/or the polymorphism you'd like to put in your functor class, and move it into a different class. Then give your functor class a pointer to this new class.

Now, the primary thing to keep in mind is that the copy constructor of functor classes must be carefully designed.

# Make predicates pure functions
A predicate is a function that returns bool.
A pure junction is a function whose return value depends only on its parameters.
A predicate dass is a functor class whose operatorO function is a predicate.

Regardless of how you write your predicates. they should always be pure functions.

# Make functor classes adaptable
Suppose I have a list of Widget* pointers and a function to determine whether such a pointer identifies a Widget that is interesting. It's easy to find the first pointer to an interesting Widget in the list
```cpp
list<Widget*> widgetPtrs;
bool isInteresting(const Widget *pw);
list<Widget*>::iterator i = find_if(widgetPtrs.begin(), widgetPtrs.end(), isInteresting);
if (i != widgetPtrs.end()) {

}
```

We can not only apply not1 to isInteresting to find the first pointer to a Widget that is not interesting. We must apply ptr_fun to isInteresting before applying not1
```cpp
list<Widget*>::iterator i = find_if(widgetPtrs.begin(), widgetPtrs.end(), not1(ptr_fun(isInteresting)));
```

This introduces some questions. Why do I have to apply ptr_fun to isInteresting before applying not1? What does ptr_fun do for me, and how does it make the above work?

The only thing ptr_fun does is make some typedef available. These typedefs are required by not1. Before a lowly function pointer, isInteresting lacks the typedefs that not1 demands.

Each of the four standard function adapters(not1, not2, bind1st, and bind2nd) requires the existence of certain typedefs. Function objects that provide the necessary typedefs are said to be adaptable, while function objects lacking these typedefs are not adaptable.Adaptable function objects can be used in more contexts than can function objects that are not adaptable, so you should make your function objects adaptable whenever you can.

The typedefs in question are argument_type, first_argument_type, second_argument_type, and result_type, but it's not quite that straightforward, because different kinds of functor classes are expected to provide different subsets of these names. In all honesty, unless you're writing your own adapters, you don't need to know anything about these typedefs. That's because the conventional way to provide them is to inherit them from a base class, or, more precisely, a base struct. For functor classes whose operator() takes one argument, the struct to inherit from is std::unary_function. For functor classes whose operator() takes two arguments, the struct to inherit from is std::binary_function.

unary_function and binary-function are templates, so you must inherit from structs they generate, and that requires that you specify some type arguments.
```cpp
template<typename T>
class MeetsThreshold: public std::unary_function<Widget, bool> {
  private:
    const T thread;
  public:
    MeetsThreshold(const T& threshold);
    bool operator()(const Widget&) const;
};
struct WidgetNameCompare: std::binary_function<Widget, Widget, bool> {
  bool operator()(const Widget& lhs, const Widget& rhs) const;
};
```
In general, non-pointer types passed to unary_function or binary_function have consts and references stripped off.

The rules change when operatorO takes pointer parameters.The types passed to binary_function are the same as the types taken by operator()
```cpp
template<typename T>
class MeetsThreshold: public std::unary_function<const Widget*, bool> {
  private:
    const T thread;
  public:
    MeetsThreshold(const T& threshold);
    bool operator()(const Widget*) const;
};
struct WidgetNameCompare: std::binary_function<const Widget*, const Widget*, bool> {
  bool operator()(const Widget* lhs, const Widget* rhs) const;
};
```

# Understand the reasons for ptr_fun, mem_fun, and mem_fun_ref
One of the primary tasks of ptr_fun, mem_fun and mem_fun_ref is to paper over one of C++'s inherent syntactic inconsistencies. For example.

If I have a function f and an object x, I wish to invoke f on x, and I'm outside x's member functions, C++ gives me three different syntaxes for making the call
```cpp
f(x);   // when f is a non-member function
x.f();  // When f is a member function and x is an object or a reference to an object.
p->f(); // When f is a member function and p is a pointer to x
```

Now, I have a function that can test Widgets and a container of Widgets. To test every Widget in container, I can use for_each.
```cpp
void test(Widget& w);
vector<Widget> vw;
for_each(vw.begin(), vw.end(), test);
```

However, if test is a member function of Widget, I'd also be able to use for_each to invoke Widget::test on each object in container. The code may be like this
```cpp
for_each(vw.begin(), vw.end(), &Widget::test);
// other version
vector<Widget* > lpw;
for_each(lpw.begin(), lpw.end(), &widget::test);
```
unfortunately, the code won't compile. Because functions and function objects are always invoked using the syntactic form for non-member functions.

mem_fun and mem_fun_ref arrange for member functions to be called using syntax of non-member function.

The way mem_fun and mem_fun_ref do this is simple. They are function templates, and several variants of mem_fun and mem_fun_ref template exist, corresponding to different numbers of parameters and the constness of the member functions they adapt.
```cpp
template<tyename R, typename C>
mem_fun_t<R, C> mem_fun(R(C::*pmf)());
```
mem_fun takes a pointer to a member function ,pmf, and returns an object of type mem_fun_t. This is a functor class that holds the member function pointer and offers an operator() that invokes the pointed to member function on the object passed to operator().

The objects produced by mem_fun and mem_fun_ref provide important typedefs.

For ptr_func, you can use it every time you pass a function to an STL component. The STL won't care, and there is no runtime penalty.
An alternative strategy with respect to ptr_fun is to use it only when you're forced to. If you omit it when the typedefs are necessary, your compilers will balk at your code. Then you'll have go back and add it.

For mem_fun and mem_fun_ref, you mus employ them whenever you pass a member functiont to an STL component.

# Make sure less\<T\> means operator\<
As a general rule, trying to modify components in std is indeed forbidden, but programmers are allowed to specialize templates in std for user-defined types. 

Don't mislead all those programmers by playing games with the definition of less. If you use less (explicitly or implicitly), make sure it means operator\<. If you want to sort objects using some other criterion, create a special functor class that's not called less.

# Prefer algorithm calls to hand-written loops
Internally, algorithms are loops, so  many tasks you might naturally code as loops could also be written using algorithms.
```cpp
class Widget {
  public:
    void redraw() const;
}
list<Widget> lw;
for (list<Widget>::iterator i = lw.begin(); i != lw.end(); ++i) {
  i->redraw();
}
```

Thare are three reasons for calling an algorithm is usually preferable to any hand-written loop
1. Effeciency: Algorithms are often more efficient than the loops programmers produce.
2. Correctness: Writing loops is more subject to errors than is calling algorithms.
3. Maintainability: Algorithm calls often yield code that is clearer and more straightforward than the corresponding explicit loops.

From an efficiency perspective, algorithms can beat explicit loops in three ways, two major, on minor.
The minor way involves the elimination of redundant computations. For example, in the code above, lw.end() will be can each time around the loop and it is rendundant.
The first major argument is that library implementers can take advantage of their knowledge of container implementations to optimize traversals in a way that no library user ever could. 

For example, the objects in a deque are typically stored (internally) in one or more fixed-size arrays. Pointer-based traversals of these arrays are faster than iterator-based traversals, but only library implementers can use pointer-based traversals.

The second major efficiency argument is that all but the most trivial STL algorithms use computer science algorithms that are more sophisticated - sometimes much more sophisticated - than anything the average C++ programmer will be able to come up with.

From an correctness perspective,

From an mantainability perspective, algorithm names suggest what they do. "for," "while," and "do" don't.

# Prefer member functions to algorithms with the same names
Some containers have member functions with the sanme names as STL algorithms. Most of the time, you'll want to use the member functions instead of the algorithms.
There are two reasons
1. the member functions are faster.
2. they integrate better with the containers than do the altorithms. That's because algorithms and member functions that share the same name typically do no do exactly the same thing.

Suppose you have a set\<int\> holding a million values and you'd like to find the first occurrence of the value 727, if there is one.
```cpp
set<int> s;
set<int>::iterator i = s.find(727);
if (i != s.end()) ...

set<int>::iterator i = find(s.begin(), s.end(), 727);
if (i != s.end()) ...
```
The find member function runs in logarithmic time, while the find algorithm runs in linear times.

Efficiency isn't the only difference between member and algorithm find. STL algorithms determine whether two objects have the same value by checking for equality, while associative containers use equivalence as their "sameness" test. Hence, the find algorithm searches for 727 using equality, while the find member function searches using equivalence. 

For the standard associative containers, choosing member functions over algorithms with the same names offers several benefits.
1. you get logarithmic-time instead of linear-time performance.
2. you determine whether two values are the same using equivalence, which is the natural definition for associative containers.
3. when working with maps and multimaps, you automatically deal only with key values instead of with pairs.

For the list container,
1. under the assumption that manipulating pointers is less expensive than copying objects, list's version of these functions should offer better performance.
2. list member functions often behave differently from their algorithm counterparts.
3. some algorithm, such as sort, can not be applied to list.

# Distinguish among count, find, binary_search, lower_bound, upper_bound and equal_range
In selecting a search strategy, much depends on whether your iterator define a sorted range. If they do, you can get speedy lookups via binary_search, lower_bound, upper_bound, and equal_range. If the iterators don't demarcate a sorted range, you're limited to the linear-time algorithms count, count_if, find, and find_if. 

If you have an unsorted range, your choices are count or find. count answers the question, "Is the value there, and if so, how many copies are there?", while find answers the question, "Is it there, and if so, where is it?"
When search is successfunl, count is less efficient, because find stops once it's found a match, while count must continue to the end of the range looking for additional matches.

When you need to know not just whether a value exists but also which object has that value, you need find.

For sorted ranges, there are more efficiency algorithm running in logarithmic time.

The shift from unsorted ranges to sorted ranges leads to another shift: from using equality to determine whether two values are the same to using equivalence.

binary_search answers the questing, "Is it there". If you need more information that that, you need a different algorithm.

If you have a sorted range and you questing is, "Is it there, and if so, where is it?", you want equal_range.

When you ask lower_bound to look for a value, it returns an iterator pointing to either the first copy of that value(if it's found), or to the proper insertion location for that value(if it's not). lower_bound answers the question, "Is it there? If so, where is the first copy, and if it's not, where would it go?"

Unlike find, you can't just test lower_bound's return value against the end iterator. Instead, you must test the object lower_bound identifies to see if that's the value you want. However, lower_bound searched using equivalence. Test for equivalence and equality may yield different results.

A more easire way is using equal_range. equal_range returns a pair of iterators that demarcate the range of values equivalent to the one you searched for, the first equal to the iterator lower_bound would return, the second equal to the one upper_bound would return.
```cpp
typedef vector<Widget>::iterator VWIter;
typedef pair<VWIter, VWIter> VWIterPair;
VWIterPair p = equal_range(vw.begin(), vw.end(), w);
if (p.first != p.second) {...}
```

There are two important observations about equal_range return value.
1. If the two iterators are the same, that means the range of objects is empty.
2. eqaul_range's return value is that the distance between its iterators is equal to the number of objects in the range.

# Consider function objects instead of functions as algorithm parameters
Passing STL function objects to algorithms typically yields code that is more efficient than passing real functions.

If a function objects's operator() function has been declared inline, the body of that function is available to compilers and most copmilers will happily inline that function during template instantiation of the call algorithm.

When we try to pass a function as a parameter, compilers cilently convert the function into a pointer to that function. Each time it's used inside an algorithm, compilers make an indirect function call. Most compilers won't try to inline calls to functions that are invoke through function pointer, even if such functions have been declared inline.

The another reason is to get your programs to compile.

One widely used STL platform rejects the following code to print to cout the length of each string in a set.
```cpp
set<string> s;
transform(s.begin(), s.end(), ostream_iterator<string::size_type>(cout "\n"), mem_fun_ref(&string::szie));
```
The cause of the problem is that this particular STL platform has a bug in its handling of const member functions.

Another reason to prefer function objects to functions is that they can help you avoid subtle language pitfalls.

# Avoid producing write-only code
Write-only code is easy to write, but it's hard to read and understand.

# Always #include the proper headers
The Standard for C++ fails to dictate which standard headers must or may be #included by other standard headers.

Any time you refer to elements of namespace std, you are responsible for having #included the appropriate headers. If you omit them, your code might compile anyway, but you'll still be missing necessary headers, and other STL platforms may justly reject your code.

To help you remember what's reqUired when, here's a qUick summary
of what's in each standard STL-related header:
* Almost all the containers are declared in headers of the same name, i.e., vector is declared in \<vector\>, list is declared in \<list\>, etc. The exceptions are \<set\> and \<map\>. \<set\> declares both set and multiset, and \<map\> declares both map and multimap.
* All but four algorithms are declared in \<algorithm\>. The exceptions are accumulate, inner_product, adjacent_difference, and partial_sum. Those algorithms are declared in \<numeric\>.
* Special kinds of iterators, including istream_iterators and istreambuf_iterators, are declared in \<iterator\>.
* Standard functors (e.g., less\<T\> and functor adapters (e.g., not1, bind2nd) are declared in \<functional\>.

Any time you use any of the components in a header, be sure to provide the corresponding #include directive, even if your development platform lets you get away without it.

# Learn to decipher STL-related compiler diagnostics
Here are a few other hints that should help you make sense of STL-related compiler messages:
* For vector and string, iterators are usually pointers, so compiler diagnostics will likely refer to pointer types if you've made a mistake with an iterator. For example, if your source code refers to vector\<double\>::iterators, compiler messages will almost certainly mention double* pointers.
* Messages' mentioning back_inserCiterator. fronCinsert_iterator, or inserCiterator almost always mean you've made a mistake calling back_inserter, fronCinserter, or inserter, respectively. If you didn't call these functions, some function you called (directly or indirectly) did.
* Similarly, if you get a message mentioning binderl st or binder2nd, you've probably made a mistake using bindl st or bind2nd.
* Output iterators do their outputting or inserting work inside assignment operators, so if you've made a mistake with one of these iterator types, you're likely to get a message complaining about something inside an assignment operator you've never heard of.
* If you get an error message originating from inside the implementation of an STL algorithm, there's probably something wrong with the types you're trying to use with that algorithm.
*  If you're using a common STL component like vector, string, or the for_each algorithm, and a compiler says it has no idea what you're talking about, you've probably failed to #include a required header file.

# Familiarize yourself with STL-related web site
* The SGI STL site: http://www.sgi.com/tech/stl
* The STLport sit: http://www.stlport.org
* The Boost site: http://www.boost.org

SGI's STL web site offers comprehensive documentation on every component of the STL.

The SGI site offers something else for STL programmers: a freely downloadable implementation of the STL. 