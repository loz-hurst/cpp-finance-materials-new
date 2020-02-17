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

In C++, the *default constructor* is the constructor that can be called with no arguments.
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

A constructor that just calls another constructor is a *delegating constructor*
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

The *special member functions* are the functions that the compiler will implicitly provide if they are used but not explicitly declared by a class.
{: .callout .terminology}

The special member functions are (with signatures for a class called 'MyClass'):

default constructor
: `MyClass();`

  Only if no other constructor is explicitly declared

copy constructor
: `MyClass(const MyClass& other);`

  Only if no move constructor and move assignment operator are explicitly declared

  In C++11 and C++14, providing a copy constructor if a destructor is declared is deprecated - it will stop being provided if you declare a destructor, in a future revision of the standard.
  {: .callout .beware}

move constructor
: `MyClass(MyClass&& other) noexcept;`

  Only if no copy constructor, copy assignment operator, move assignment operator and destructor are explicitly declared.

copy assignment operator
: `MyClass& operator=(const MyClass& other);`

  Only if no move constructor and move assignment operator are explicitly declared.

move assignment operator
: `MyClass& operator=(MyClass&& other) noexcept;`

  Only if no copy constructor, copy assignment operator, move assignment operator and destructor are explicitly declared.

destructor
: `~MyClass() noexcept;`

These are all of the functions you can use `delete` and `default` with to explicitly say you want or do not want the default versions (if you do not override them).

Note that the move constructor, move assignment operator and destructor must *not* throw exceptions.
{: .callout .beware}

#### Rules of five (née three) & zero

It is generally accepted that to build correct objects that if you provide your own destructor, copy constructor, copy assignment operator, move constructor or move assignment operator you should provide all of them.

This is called the *rule of five*, because if you provide any one of these special members you must provide all five.
{: .callout .terminology}

Before C++11 this was called the rule of three, the Law of The Big Three or The Big Three.  This was because there was no "move" concept so you only had to provide three methods.  This is no longer true, you should provide five.
{: .callout .history}

Some argue that the rule of five only applies if you want your object to be movable, so you can just stick to the rule of three.  The [C++ Core Guidelines](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c21-if-you-define-or-delete-any-default-operation-define-or-delete-them-all) are clear that the full rule of five should be preferred and moves either explicitly made into more expensive copies or the class made move only.
{: .callout .bad_practice}

Underpinning this rule is the idea that if you are manually releasing or tidying up some resource (which is the most usual reason for needing a destructor), or can optimise by moving a reference instead of doing a copy, then that resource needs careful management and the default (shallow) copy and move behaviours will not work correctly for that resource.
{: .callout .philosophy}

Since C++11 the idea of "the rule of zero" has been gaining more traction (although it has been in the [C++ Core Guidelines](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-zero) since before then).  This principle is the one can avoid writing any custom copy/move/destruction semantics by using existing types (such as smart pointers and containers from the standard library) that already support appropriate copy/move semantics.

The rule of zero lets us rely on the default special members if we only use standard library containers and smart pointers.  They manage their own memory and will copy/move themselves as efficiently as possible without any extra effort by us.
{: .callout .philosophy} 

#### Conversion constructors

We have already met these.  

Any constructor that takes a single argument of a different type, with no default value, is a *conversion constructor*.
{: .callout .terminology}

Any such constructor can create an object from an object of a different type.
{: .callout .philosophy}

Remember that such constructors will be called implicitly by the compiler when an object of the type of the class is expected but one of the constructor's argument type is given.

To prevent them being called implicitly for type conversions mark the constructor `explicit`.  This is recommended unless you want implicit type conversions.
{: .callout .good_practice}

#### Copy constructors

A *copy constructor* takes a single argument of the same type as the class and initialises this one from it (making a copy).
{: .callout .terminology}

The argument to the copy constructor is almost always a constant reference (`const &`).  The compiler provided one will do a shallow copy of each data member.

```cpp
MyClass b;
MyClass a {b}; // Copy constructor called
a = b; // Copy assignment operator called
```

Typically the copy constructor will copy the contents of the old object into the new one.  Sometimes it may not be necessary to copy everything or some things may need special attention when copying and these are the situations when we need to provide our own copy constructor.

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

### Exceptions in constructors and destructors

As previously discussed, the only way for a constructor to signal an error is to throw an exception.

If an exception occurs (is thrown) the application should be left in a safe state.  If thrown from a constructor it is safe to assume that object is unusable but any other object should be left in a safe state and any work the constructor may have already done (e.g. setting up database connections) should be left in a sensible state now the object construction has failed.

Destructors must never throw exceptions, as the exception will cause the rest of the destructor and any other destructors in the object's inheritance hierarchy to not be run.
{: .callout .beware}

## Conversion operators

We have seen how to convert from other types to our class (conversion constructors) but we can also specify how to convert from our type to another.

A special operator called the *conversion operator* allows us to specify how to convert a class to another type.
{: .callout .terminology}

To define a conversion operator we use the keyword `operator` followed by the type-id to be converted to.  This may be a custom type or built in one.  The operator returns a converted object.  It may be marked `explicit` (only used to explicitly convert) and/or `const` (will not modify the object).

For example:

```cpp
class MyClass {
public:
	// Converts MyClass to int
	operator int() const {return 7;}
	// Converts MyClass to double
	explicit operator double() const {return 2.3;}
};
class MyOtherClass {
public:
	// Converts MyOtherClass to MyClass
	operator MyClass() const { return MyClass(); }
};

int main() {
	MyClass cls;
	int a {cls}; // Fine, we have a conversion to int
	a = cls; // Fine, we have an implicit conversion permitted
	double b {cls}; // Fine, we have a conversion to double
	b = cls; // Error - conversion operator is explicit only
}
```

When converting between your own types you can either have a conversion constructor in the target type, or a conversion operator in the source.

If you have both a conversion constructor and conversion operator for the types, the compiler will not know which to use and your program will not compile.
{: .callout .beware}

## Small Types

### Pair

Pair, which we have seen before with map, can be used to hold any two related items even if they are of different types.  They are in the `utility` header.

```cpp
std::pair<std::string, double> my_pair {"Hello", 3.8};
```

### Tuple

Tuple is a generalised version of pair that can hold any number of values (instead of just two).  Each can be a different type.  It is in the `tuple` header.

For example:

```cpp
std::tuple<std::string, double, int, std::string> my_tuple {"Hello", 3.8, 5, "world"};
my_tuple = std::make_tuple("Hi", 5.4, 3, "there");
```

Unlike pair, which has `first` and `second` accessors, tuple does not provide any way to access elements built into the object.  Instead we have to use the `std::get` function, which it provides:

```cpp
std::string first {std::get<0>(my_tuple)}; // Get the first element
// Get the double (only works if there is exactly 1 element of type double):
double dbl {std::get<double>(my_tuple)}; 
```

Tuple also provides a function `std::tie` to unpack the entire tuple:

```cpp
std::string greeting1, greeting2;
int my_int {0};
double my_dbl {0};
std::tie(greeting1, my_dbl, my_int, greeting2) = my_tuple;
```

If we do not want to unpack a specific item from the tuple we have to use `std::ignore` (also from the tuple library) to indicate that:

```cpp
std::tie(greeting1, std::ignore, std::ignore, greeting2) = my_tuple;
```

### Array

Array is a fixed-size array provided by the `array` header.  It can only be used if the size of the array will be known at compile time and is intended to be a zero-overhead replacement for C arrays (i.e. should give exactly the same performance) but can be copied, assigned and passed by value unlike C arrays.  Most importantly, it knows its own size where as a C array does not (a C array is just a pointer, we will look at this in the last lesson).

```cpp
std::array<int, 3> my_array; // Declare an array of ints, size 3
std::array<int, 2> my_other_array {2, 3}; // Initialised array of 2 ints
```

Like C arrays, the items inside a std::array are guaranteed to be contiguous in memory.
{: .callout .technical}

## Lab exercises

### 1. Finish off the input file reader class with CSV and TSV specialised versions from last week.

You will need to consider which functionality will be common to both kinds (open the file, check it is good, check for not reading past the end of the file) and what will be specialised (the delimiters are the key difference).

The TSV format is documented by IANA as the mime type [text/tab-separated-values](https://www.iana.org/assignments/media-types/text/tab-separated-values).  It is simpler than CSV, so start with this one.

The recognised format for CSV is documented in [RFC4180](https://www.ietf.org/rfc/rfc4180.txt).  Note you will need to cope with quoted fields (which can include newlines-within-quotes).

### 2. Add a fixed-width field format reader.

Sometimes data is provided in a fixed-width data format, where each field is a set number of characters rather than being seperated by a delimiter.  Write a new sub-class that can read such a file with the following format (we will generalise it next week):

- 8 characters for a date (YYYYMMDD)
- 3 character for the stock symbol
- 8 characters for the current value - first 4 are pre-decimal point, next 4 are after (i.e. field is price x 10000)

For example:
```text
20200101UOB00012345
20200101GOG01000056
20200102LAH00000005
```
