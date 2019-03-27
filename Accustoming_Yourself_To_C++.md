# Accustoming yourself to C++

### Item 1:  View C++ as a federation of language

Today's C++ is a multiparadigm programming language, one supporting a combination of procedural, object-oriented, functional, generic, and metaprogramming features.

To make sense of C++, you have to recognize its primary sublanguages. Fortunately, there are only four:

* **C**: Way down deep, C++ is still based on C. Blocks, statements, the preprocessor, built-in data types, arrays, pointers, etc., all come from C. When you find yourself working with the C part of C++, the rules for effective programming reflect C's more limited scope: **no templates, no exceptions, no overloading, etc.**
* **Object-Oriented C++**: This part of C++ is what C with Classes was all about: **classes** (including constructors and destructors), **encapsulation, inheritance, polymorphism, virtual functions** (dynamic binding), etc. 
* **Template C++**: This is the generic programming part of C++, the one that most programmers have the least experience with. In fact, templates are so powerful, they give rise to a completely new programming paradigm, *template metaprogramming)* (TMP). The rules for TMP rarely interact with mainstream C++ programming.
* **The STL**: The STL is a template library, of course, but it's a very special library.It's conventions regrading containers, iterators, algorithms, and function objects mesh beautifully, but templates and libraries can built around other ideas, too.

Keep these four sublanguages in mind, and don't be surprised when you encounter situations where effective programming requires that you change strategy when you switch from one sublanguage to another.

For example, **pass-by-value** is generally more efficient than **pass-by-reference** for built-in types, but when you move from the **C part of C++** to **Object-Oriented C++**, **the existence of user-defined constructors and destructors** means that pass-by-reference-to-const is usually better.

This is especially the case when working in **Template C++**, because there, you don't even know the type of object you're dealing with. When you cross **STL**, however, you know that **iterators and function objects are modeled on pointers in C**, so for iterators and function objects in the STL, the old C pass-by-value rule applies again.