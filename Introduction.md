#Introduction

###Default constructor

 A constructor that can be called without any arguments. Such a constructor either has no parameters or has a **default value** for every parameter.



###Explicit constructor

A constructor that can prevents themselves from being used to perform **implicit** type conversions. However, they may still be used for **explicit** type conversions.

Constructors declared **explicit** are usually preferable to non-explicit ones, because they prevent compilers from performing **unexpected (often unintended) type conversions**.



### Copy constructor

A constructor that is used to **initialize** an object with a different object of the same type.



### Copy assignment operator

An operator that is used to **copy** the value from one object to another of the same type.

Though when you see what appears to be an assignment, it may also be used to call the copy constructor since it use it to initialize an object. For example:

```c++
Widget w3 = w2;    // invoke copy constructor!
```



Fortunately, copy construction is easy to distinguish from copy assignment:

**If a new object is being defined, a constructor has to be called; it can't be an assignment. If no new object is being defined, no constructor can be invoked, so it's an assignment.**



The copy constuctor is particularly important function, because it defines how an object is **passed by value**. *Pass-by-value* means "call the copy constructor"

**However, it's generally a bad idea to pass user-defined types by value. Pass-by-reference-to-const is typically a better choice.** (Why? See Item 20)



### Undefined behavior

A program with **undefined behavior** could erase your hard drive. But is's not probable. More likely is that the program will behave **erratically**.