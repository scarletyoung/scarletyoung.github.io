---
title: Effective Modern C++ Note
date: 2022/01/04
---

# Distinguish between pointers and references
How to decide to use pointers or references
1. There is no such thing as a null reference. But pointers can be null.
  ```c++
  char *pc = 0;
  char& rc = *ps;
  ```
  The code behavior above is undefined.
2. References must be initialized. Pointers are subject to no such restriction.
3. Reference can be more efficient because there is no need to test the validity of a reference before using it.
4. Pointers may be reassigned to refer to different object. A reference always refers to the object with which it is initialized.
  ```c++
  string s1("Nancy");
  string s2("Clancy");
  string& rs = s1;
  string *ps = &s1;
  rs = s2;  // rs refers to s1, and s1's value is change to "Clancy"
  ps = &s2; // ps points to s2, s1 is unchanged.
  ```

Use a reference whenever there will always be an object to refer to, whenever never want to refer to anything else and implementing some operatores such as operator[] whose syntactic requirements make the use of pointers undesirable should use a reference return.

In all other cases, stick with pointers.

# Prefer C++-style casts
Same as Effecitive C++ [Minimize casting](Effective%20C++.md#Minimize%20casting)

# Never treat arrays polymorphically
Suppose we have a class BST and a second class, BalancedBST, that inherits from BST
```c++
class BST {};
class BalancedBST: public BST{};
```
Consider a function to print out the content of each BST in an array of BSTs
```c++
void printBSTArray(ostream& s, const BST array[], int numElements) {
  for (int i = 0; i < numElements; ++i) {
    s << array[i];
  }
}
```
The function also can receive an array of BalancedBST objects
```c++
BalanceBST bBSTArray[10];
printBSTArray(count, bBSTArray, 10);  // can pass and run
```
Because the parameter array is declared to be of type array-of-BST, so the distance between array and array+i is array + i * sizeof(BST).

Derived class usually have more data members thar their base classes, so derived class objects usually larger that base class objects.

So, if you passes an array of BalancedBST objects to printBSTArray, the pointer arithmetic will be wrong for arrays of BalancedBST objects, the function behavior is undefined.

