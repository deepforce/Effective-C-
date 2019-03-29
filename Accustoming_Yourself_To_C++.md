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



### Item 2: Prefer `consts`, `enums`, and `inlines` to `#define`s.

This Item might better be called "**prefer the compiler to the preprocessor**" because #define may be treated as if it's not part of the language per se.

When you do something like this:

​	`#define ASPECT_RATIO 1.653`

the symbolic name ASPECT_RATIO may never be seen by compilers; **it may be removed by the preprocessor before the source code ever gets to a compiler.**

As a result, the name ASPECT_RATIO **may not get entered into the symbol table**. This can confusing if you get an error during compilation involving the use of the constant, because the error message may refer to 1.653, not ASPECT_RATIO. If ASPECT_RATIO were defined in a header file you didn't write, you'd have no idea where that 1.653 came from, and you'd waster time tracking it down. 

The solution is to replace the macro with a constant:

​	`const double AspectRatio = 1.653 //uppercase names are usually for macros`

In the case of a **floating point constant**, use of the constant may yield smaller code than using a `#define`. That's because the preprocessor's blind substitution of the macro name ASPECT_RATIO with 1.653 could **result in multiple copies** of 1.653 in your object code, while the use of the constant AspectRatio should **never result in more than one copy**.

When replacing `#define` with constants, two special cases are worth mentioning. 

1. Defining constant pointers.
   Because constant definitions are typically put in header files, it's important that **the *pointer* be declared `const`**, usually in addition to what the pointer points to. For example, to define a constant `char*-`based string in a header file, you have to write `const` twice:

   ​	`const char * const authorName = "Scott Meyers";`

   However, `string` objects are generally preferable to their `char*`-based progenitors, so `authorName` is often better defined this way:

   ​	`const std::string authorMame("Scott Meyers");`

2. Class-specific constants.

   To limit the scope of a constant to a class, you must make it a member, and **to ensure there's at most one copy of the constant, you must make it `static` member**:

   ```c++
       class GamePlayer {
           private:
           static const int NumTurns = 5; // constant declaration
           int scores[NumTurns];          // use of constant
           ...
       };
   ```

   What you see above is a *declaration* for `NumTurns`, not a definition.

   **Usually, C++ requires that you provide a definition for anything you use, but class-specific constants that are static and of integral type are an exception.** As long as you don't **take their address**, you can declare them and use them without providing a definition. If you do take the address of a class constant, or if your compiler incorrectly insists on a definition even if you don't take the address, you provide a separate definition like this:

   ​	`const int GamePlayer::NumTurns; // deinition of NumTurns`

   You put this in an implementation file, not a header file. Because the initial value of class constants is provided where the constant is declared, **no initial value is permitted at the point of definition**.

   

   **There is no way to create a class-specific constant using a `#define`, because `#define` don't respect scope.** Once a macro is defined, it's in force for the rest of the compilation. Which means that **not only can't `#define` be used for class-specific constants**, they also **can't be used to provide any kind of encapsulation**. 

   

   Older compilers may not accept the syntax above, because it used to be illegal to provide an initial value for a static class member at its point of declaration. Furthermore, in-class initialization is allowed only for integral types and only for constants. In cases where the above syntax can't be used, you can put the initial value at the point of definition:

   ```c++
   	class CostEstimate {
           private:
           	static const double FudgeFactor;		// declaration of static class constant; goes in header file
           	...									
   	};
   	const double CostEstimate::FudgeFactor = 1.35;  // definition of static class constant; goes in impl. file
   ```



The only exception is **when you need the value of a class constant during compilation of the class**, such as in the declaration of the array `GamePlayer::scores` above (where compilers insist on knowing the size of the array during compilation). Then the accepted way to compensate for compilers that forbid the in-class specification of initial values for static integral class constants is to use what is affectionately known as "**the `enum` hack**." This technique takes advantage of the fact that the values of an enumerated type can be used where `int`s are expected, so `GamePlayer` could just as well be defined like this:

```c++
	class GamePlayer {
    private:
        enum {NumTurns = 5};	// "the enum hack" -- makes NumTurns a symbolic name for 5
       	int scores[NumTurns];	// fine
        ...
    };
```

The `enum` hack is worth knowing about for several reasons.

1. the `enum` hack behaves in some ways more like a `#define` than `const` does, and sometimes that's what you want. For example, it's legal to take the address of a `const,` but it's not legal to take the address of an `enum`, and it's typically not legal to take the address of a `#define`, either. **If you don't want to let people get a pointer or reference to one of your integral constants, an `enum` is a good way to enforce that constraint.** Also, though good compilers won't set aside storage for `const` objects of integral types (unless you create a pointer or reference to the object), sloppy compilers may, and you **may not be willing to set aside memory for such objects**. Like `#define`s, `enum`s never result in that kind of unnecessary memory allocation.
2. Lots of code employs it, so you need to recognize it when you see it. In fact, the `enum` hack is a fundamental technique of template metaprogramming.



Getting back to the preprocessor, another common (mis)use of the `#define` directive is using it to implement macros that look like functions but that don't incur the overhead of a function call. 

Here's a macro that calls some function `f` with the greater of the macro's arguments:

```c++
	// call f with the maximum of a and b
	#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
```

Macros like this have so many drawbacks, just thinking about them is painful. Whenever you write this kind of macro, **you have to remember to parenthesize all the arguments in the macro body**. Otherwise you can run into trouble when somebody calls the macro with an expression. But even if you get that right, look at the weird things that can happen:

```c++
	int a = 5, b = 0;
	CALL_WITH_MAX(++a, b);			// a is incremented twice
	CALL_WITH_MAX(++a, b+10);		// a is incremented once
```

Here, **the number of times that `a` is incremented before calling `f` depends on what it is being compared with!**

Fortunately, you don't need to put up with this nonsense. You can get all the efficiency of a macro plus all the predictable behavior an type safety of a regular function by using a template for an `inline` function:

```c++
	template<typename T>								// because we don't know what T is, we pass by reference-to-const
	inline void callWithMax(const T& a, const T& b)
	{
        f(a>b ? a:b);
    }
```

This template generates a whole family of functions, each of which takes two objects of the same type and calls `f` with the greater of the two objects. There's no need to parenthesize parameters inside the function body, no need to worry about evaluating parameters multiple times, etc. Furthermore, because `callWithMax` is a real function, it obeys scope and access rules. For example, it makes perfect sense to talk about an inline function that is private to a class. In general, there's no way to do that with a macro. 

