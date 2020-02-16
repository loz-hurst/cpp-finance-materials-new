---
title: Lesson 15 - Advanced construction and small types
---

# {{ page.title }}
{: .no_toc}

Today's notebook: [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/loz-hurst/cpp-finance-materials-new.git/master?filepath=lessons%2Flesson-15.ipynb)

- TOC
{:toc}

## Recap

Last lesson we looked at polymorphism, most usefully in the context of a base-type reference behaving in multiple different ways depending on the actual type being referenced.

We also looked at creating a polymorphic input data reader, in the lab session (which we will carry on today).

## Constructors

We have talked a lot about constructors (and a little about destructors) since we began looking at object-orientation.

Constructors maybe invoked explicitly (e.g. when we initialise a new instance) or implicitly (e.g. assignment, conversion, copied (return, by-value, &hellip;)).  The compiler will provide some very simple default ones.

We can write our own constructors to change how our objects are created, e.g. to initialise things inside our object or if our object will manually manage memory (at the end of this course).

If we specify any constructors in our class definition then the compiler will not provide **any** constructors for us.
{: .callout .technical}


### Overloading constructors

As we have seen in previous lessons, constructors can be overloaded like any other function.  We can provide versions that different numbers of arguments or different types, to construct our objects from different initial data.

To do this, we can just provide multiple constructors in our definition just as though we were overloading a function:

```cpp
// Some example class that contains a couple of ints.
class MyClass {
private:
	int a_; // A member int
	int b_; // Another member int
public:
	// A constructor to initialise the two members to different values
	MyClass(const int a, const int b) : a_{a}, b_{b} {}
	// An overloaded constructor to initialise both members to the same value
	MyClass(const int value) : a_{value}, b_{value} {}
};
```

### The default constructor

In C++, the default constructor is the constructor that can be called with no arguments.
{: .callout .terminology}

The default constructor is what allows us when we declare a new instance without providing any initialisation.

```cpp
MyClass my_instance; // Will be created by calling the default constructor
```

If we do not define any constructors then a default constructor with an empty body (i.e. that does nothing) is provided.  However if we define **any** constructors then C++ will not provide a default constructor.  It is possible to have a class without a default constructor (and easy to do it by accident) just through providing another constructor.

If we want to use the default constructor but also provide our own overridden constructor, we can explicitly tell the compiler to provide the default by assigning `default` to it - a bit like assigning `0` for a pure virtual function:

```cpp
// Some class that provides a default and not-default constructor
class MyClass {
public:
	MyClass() = default;
	MyClass(const std::string message) { std::cout << message << " from MyClass constructor" << std::endl; }
}

```

### Deleting constructors

Very rarely we might want to prevent a class being instantiated at all.  One example is if everything (data members and methods) are static.

A namespace can make more sense than a purely static class, unless templates are required (which we will look at next lesson).
{: .callout .technical}

If we want to stop the class being instantiated then we need to make sure there are no constructors but, so far, unless we declare one of our own C++ is going to implicitly provide a default one.  We can, however, explicitly delete the default constructor by assigning `delete` to it - this will tell C++ not to provide a default.

```cpp
// A class with a single static member
class MyClass {
private:
	static int a_ {0};
public:
	// Prevent instantiation
	MyClass() = delete;
	// Sets the member
	static void SetA(const int a) {a_ = a;}
	// Gets the member
	static int GetA() const {return a_;}
};
```

Note this would be functionally equivalent to:

```cpp
namespace MyClass {
	// Anonymous namespace, cannot be accessed from outside this translation unit
	namespace {
		int a_ {0};
	}
	
	void SetA(const int a) {
		a_ = a;
	}

	int GetA() {
		return a_;
	}
}
```

### Delegating constructors

A constructor that just calls another constructor is called a "delegating constructor"
{: .callout .terminology}

We saw when we first look at inheritance how to call "up" to a base (parent) class constructor during object creation.  Using the same method we can call another constructor in the same object, if appropriate.  Consider the previous example that contained a couple of ints:

```cpp
// Some example class that contains a couple of ints.
class MyClass {
private:
	int a_; // A member int
	int b_; // Another member int
public:
	// A constructor to initialise the two members to different values
	MyClass(const int a, const int b) : a_{a}, b_{b} {}
	// Delegates to the two-member constructor with both values the value passed to this one
	MyClass(const int value) : MyClass(value, value) {}
};

```

### Special member functions

The special member functions are the functions that the compiler will implicitly provide if they are used but not explicitly declared by a class.
{: .callout .terminology}

The special member functions are (with signatures for a class called 'MyClass'):

default constructor
: `MyClass();`
  Only if no other constructor is explicitly declared

copy constructor
: Only if no move constructor and move assignment operator are explicitly declared

  In C++11 and C++14, providing a copy constructor if a destructor is declared is deprecated - it will stop being provided if you declare a destructor, in a future revision of the standard.
  {: .callout .beware}

move constructor
: Only if no copy constructor, copy assignment operator, move assignment operator and destructor are explicitly declared.

copy assignment operator
: Only if no move constructor and move assignment operator are explicitly declared.

move assignment operator
: Only if no copy constructor, copy assignment operator, move assignment operator and destructor are explicitly declared.
destructor

These are all of the functions you can use `delete` and `default` with to explicitly say you want or do not want the default versions (if you do not override them).

Note that the move constructor, move assignment operator and destructor must *not* throw exceptions.
{: .callout .beware}

### Temporary objects

As we know, objects can be created, deleted, copied, moved, assigned and modified.  We can apply many different operators to objects (that support them), such as +, <, ==, =, <<, etc.

C++ will implicitly create temporary objects when it needs to, in obvious places (such as passing objects by value, returning objects from methods) and in some not so obvious places (such as performing mathematical operators such as +).

We want to write code to avoid creating unnecessary temporary objects, as they take compute time that can often be avoided.
{: .callout .philosophy}

- We try to pass objects using references (`const &`) where we could pass by value, saving a temporary being created.
- We can pre-compute values that will not change, to avoid repeatedly creating temporaries in the same calculation.
- Throw exceptions when a constructor fails, rather than let a (probably bad) object carry on being created.

However, like optimisation, we have to be sensible:

```cpp
a = b + c; // operator+(b, c) creates a temporary that is then assigned to be
a = b; a += c; // no temporary is created, a internally updates itself without a separate instance being created
```

The first line is clear what the intention is ("add b and c, store in a"), the second is _possibly_ (see the next paragraph!) more efficient but it is not clear that the programmer's intention is "add b and c, store in a".

Unless this line of code is run more times than any other in the entire program, this is an example of premature optimisation - do not do this unless you have confirmed there is no bigger improvement to performance to be made elsewhere _and_ the performance benefit is enough to sacrifice maintainability (programmer's time is also valuable).
{: .callout .bad_practice}

Despite the expense of temporaries, the compilers are very good at optimising common patterns and can sometimes eliminate some temporaries entirely:

```cpp
class MyClass {};
// Returns a new MyClass by value - creating a copy
MyClass function() { return MyClass(); }
int main() {
	MyClass d {function()};
}
```

On the surface it seems like a temporary will be created, returned from `function()`, that is then used to initialise `d`.  However, the compilers are allowed to optimise away certain copy-constructor calls and treat this as though the line in `main()` were `MyClass d {MyClass()};`.

This optimisation is allowed even if the constructor has side-effects - it will still only get called once even though those side effects would have happened twice the first time.  We can see this by adding a static counter to the class to count the number of times constructors are called.
{: .callout .technical}


## Lab exercises

### 1. Finish off the input file reader class with CSV and TSV specialised versions from last week.

You will need to consider which functionality will be common to both kinds (open the file, check it is good, check for not reading past the end of the file) and what will be specialised (the delimiters are the key difference).

The TSV format is documented by IANA as the mime type [text/tab-separated-values](https://www.iana.org/assignments/media-types/text/tab-separated-values).  It is simpler than CSV, so start with this one.

The recognised format for CSV is documented in [RFC4180](https://www.ietf.org/rfc/rfc4180.txt).  Note you will need to cope with quoted fields (which can include newlines-within-quotes).

### 2. Add a fixed-width field format reader.



