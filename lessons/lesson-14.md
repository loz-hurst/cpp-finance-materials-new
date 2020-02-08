---
title: Lesson 14 - Dynamic Polymorphism
---

# {{ page.title }}
{: .no_toc}

- TOC
{:toc}

## Recap

Last lesson we looked at inheritance, how that works in C++ and creating derived a base Option class, as well as derived classes for European Call and Put options.

This week we will look at using inheritance for an important, and powerful, aspect of Object-Orientated Programming - "Dynamic Polymorphism" - as well as how casting works in contemporary C++.

First, however, we will look at "Forward Declarations".

## Forward Declarations

So far, when we needed to reference a class (e.g. in a function declaration that has it as an argument) we include the full header.  When we "mention" (e.g. by-reference argument) an object we can get away with just saying "this thing exists and is a class".

This is because a reference uses the same amount of memory regardless of the size of the thing it points to.  This is all the compiler cares about when building individual files.
{: .callout .technical}

To create a forward declaration, we just declare the class as being of type Class but no body:

```cpp
// Forward declaration - see MyObjects.hpp for full definition
class MyClass;
```

This is telling the compiler "MyClass" is a thing of type class, but nothing more about it.
{: .callout .terminology}

You can do this with other types too, if you need to (but remember the type of the thing you are forward declaring has to have been declared - `class` is built into C++ so we do not have to worry about this in the above example).

The forward declared object can then be used, e.g. in this function declaration with a by-reference argument:

```cpp
// Forward declaration - see MyObjects.hpp for full definition
class MyClass;

// SomeFunction takes a reference to a MyClass instance
void SomeFunction(const MyClass &);
```

We can use forward declarations to reduce the number of headers included.  This has multiple benefits, one of the key ones is reduced compile time as the number of files to be pre-processed is smaller (less work for the pre-processor) and the resultant file to be compiled, after pre-processing, is smaller (less work for the compiler).  It also reduces the number of files that have to be recompiled when the forward-declared object (and hence its header) changes, which is especially useful if using a compiler cache (e.g. [ccache](https://ccache.dev/)) that detect unchanged translation units.
{: .callout .philosophy}

You can only use forward declarations where the object is just mentioned, using it (e.g. calling any of its methods) requires the full object definition and this an include of the header.

Simply declaring a variable of that object's type **is using** it (it will create an instance using a constructor).
{: .callout .beware}

Do not use forward declarations to circumvent compile-time warnings and errors - if your code will not compile with the full header you have an architectural problem with your program that needs fixing.  You probably need an interface (see last week's lesson) if you have a circular arrangement of types, for example.
{: .callout .bad_practice}

## Polymorphism

We have already seen polymorphism in action with overloaded functions - one function can have more than one form (signature).  This is a kind of polymorphism.

Polymorphism is the ability for an object to have several different forms.
{: .callout .terminology}

When designing our software we want to avoid the use of large switch or if statements to test if an object is one of several things in order to use them.  Polymorphism allows us to avoid this by creating an interface that several types of object can provide and that can be used to interact with all of them.  For example, when calculating pay-off; instead of testing if an European Option is a Call or Put, polymorphism allows us to say "all options can calculate a pay-off" and use either in the same way to find the pay-off without worrying about which specific one is being used.
{: .callout .philosophy}

The sort of polymorphism we are about to look at in more detail happens through inheritance, which we introduced last lesson.

The polymorphism that happens when a function is overloaded, or templates are used, is called *static* polymorphism.  This happens at compile time, the compiler works out what form is used according to rules.  The polymorphism we are about to look at is called *dynamic* polymorphism and the program works out at runtime which code to use.
{: .callout .terminology}

## Dynamic Polymorphism

In C++ we think of dynamic polymorphism as occurring when the same method call can have different behaviour.  This is realised through virtual methods in base classes that are overridden by sub-classes.

The base class is said to provide an *interface*.  That is, a common set of methods for interacting with any object derived from that class (i.e. anything that *is-a* that type).
{: .callout .terminology}

Recall from last lesson that virtual method do not have to be implemented in the base class, in which case the base class is "abstract".

The derived class an provide a specialised version of a virtual method that overrides what is in the base class.

A derived class that provides a version of a purely virtual method in its base class is said to *implement* that method.
{: .callout .terminology}

This is one of the most important ideas and aspects of object-orientated programming: One interface, multiple implementations.

### Types

As you are very familiar with by now, C++ requires every name (variable, function, etc.) we create to have a type.  In the case of a reference, it actually has two: a *static* type and a *dynamic* type.  They may be the same or different.

The *static* type of a reference is the type specified when it is declared.
{: .callout .terminology}

The *dynamic* type of a reference is determined by the type of the object being referred to, when a reference refers to something valid.
{: .callout .terminology}

For example:

```cpp
// My base class
class MyClass {};
// My Derived class
class MyOtherClass : public MyClass {};

// Some function that takes a reference to an instance of MyClass
void MyFunction (const MyClass & cls) {}

int main() {
	MyClass cls1;
	MyOtherClass cls2;

	MyFunction(cls1);
	MyFunction(cls2);

	return 0;
}
```

The first time MyFunction is called, the parameter cls will have a static type of MyClass (as that is what cls was declared as) and a dynamic type of MyClass too (as that is the type of the thing being referenced).  The second time, cls will still have a static type of MyClass (its declaration has not changed) but a dynamic type of MyOtherClass (as that is the type of cls2, which is what is being referenced in the second call).

### Virtual functions

We briefly met virtual functions last lesson when discussing the terms "abstract class" and "purely abstract class" (also known as "interfaces").

When a virtual function is invoked through a reference to a base class, C++ will run the last override of that signature in the dynamic type of the reference.  That is; the last version of that function defined in the class hierarchy at or below the the reference's static type.

If we add the keyword `override` to the end of the declaration of the derived function, the compiler will check that it does actually override something and generate a compile-time error if it does not.

Getting in the habit of always adding `override` to functions will help prevent the common mistake of mistyping function names or parameters lists.
{: .callout .good_practice}

For example:

```cpp
#include <iostream>

// Base class for all things that can say hello
class MyClass {
public:
	// Says hello on cout
    virtual void SayHello () const {
        std::cout << "Hello world" << std::endl;
    }    
};

// Another class that derives from MyClass to say hello
class MyOtherClass : public MyClass {
	// Says a different hello on cout
    virtual void SayHello () const override {
        std::cout << "Hello from another world" << std::endl;
    }
};

// Calls SayHello in the given class reference
void MyFunction (const MyClass & cls) {
    cls.SayHello();
}

int main() {
	MyClass cls1;
	MyOtherClass cls2;

	MyFunction(cls1); // Hello world is printed
	MyFunction(cls2); // Hello from another world is printed

	return 0;
}
```

### Dynamic binding

The process of finding the implementation at runtime is called *dynamic binding* (sometimes also known as "late binding").
{: .callout .terminology}

Dynamic binding of virtual functions only applies with indirection.  It does not happen if a reference is not used:

```cpp
// Note cls is now NOT a reference
void MyFunction (const MyClass cls) {
    cls.SayHello();
}

int main() {
	MyOtherClass cls1;

	MyFunction(cls2); // Prints "Hello World" from MyClass

	return 0;
}
```

Remember when passing by value a copy is made, and that copy will be a new object of the declared type.  Only a reference will not create a new object, so it makes sense that only a reference might refer to something with a different type to its static one and therefore have a dynamic type.
{: .callout .technical}

The compiler looks for member functions based on the static type - the actual type of the object the reference will point to is irrelevant at compile time:

```cpp
int i;

MyClass cls1;
MyOtherClass cls2;

while( std::cout << "Enter an integer: " && ! (std::cin >> i) )
{
    std::cin.clear();
    std::string line;
    std::getline(std::cin, line);
    std::cout << "I am sorry, but '" << line << "' is not an integer" << std::endl;
}

// Compiler can't tell whether MyClass or MyOtherClass will be used at compile time
MyFunction((i>0) ? cls1 : cls2);
```

Dynamic binding only occurs when a function is virtual.

If during compilation a member function is found that is a virtual function **and** it is called through a reference then the decision of which function will actually be executed is deferred until runtime.
{: .callout .technical}

## Virtual Inheritance

Last lesson we looked at inheritance in detail and briefly mentioned the challenge of multiple inheritance.  Suppose we have this class structure:

```cpp
// Class Animal - base class for an animal
class Animal {
public:
	// Makes the animal breath
	void Breath();
};

// A base class for all animals with legs
class LeggedAnimal : public Animal {};
// A base class for all animals with wings
class WingedAnimmal : public Animal {};

// A pigeon, which has 2 legs and wings
class Pigeon : public LeggedAnimal, public WingedAnimmal {
public:
	// Make pigeon fly
	void Fly() {
		// Pigeons need to breath while in flight
		Breath();
	}
};
```

Pigeon inherits from both LeggedAnimal and WingedAnimal, and both of them inherit from Animal.  In C++, Pigeon ends up with two distinct Animal sub-objects created one from LeggedAnimal and one from WingedAnimal.  So which one should get used when Pigeon calls Breath?  The answer is nobody can tell, and neither can the compiler - which will generate a compile-time error.

This example is called *the diamond problem* (sometimes *the deadly diamond of death*) because of the diamond shape made if you draw out the class inheritance structure with (in our example) Animal at the top, LeggedAnimal and WingedAnimal in the middle and Pigeon at the bottom.
{: .callout .terminology}

It is possible to resolve this ambiguity by explicitly referencing one of the Animals but Pigeon will still inherit two sub-objects which is inefficient and makes no sense (Pigeon is **a** animal, not two animals):

```cpp
void Pigeon::Fly() {
	// Use the Breath from the 'LeggedAnimal' side of the tree
	LeggedAnimal::Breath();
}
```

On top of being in efficient, having two Animals means each has its own internal state - what happens if your code calls Breath from LeggedAnimal and another piece of code, either in WingedAnimal or another derived class, calls it from the other side?

To deal with this, C++ allows us to use the virtual keyword when inheriting:

```cpp
class LeggedAnimal : virtual public Animal {/* ... */};
class WingedAnimal : virtual public Animal {/* ... */};
```

Using the virtual keyword when specifying the base classes is called *virtual inheritance*.
{: .callout .terminology}

Here what virtual does is different to with functions - virtual inheritance at compile time.  The compiler will ensure that only one Animal sub-object regardless of how many times in a derived objects hierarchy it is virtually inherited.  If any objects inherited Animal non-virtually they would still get a distinct sub-object created.

The rule is: one sub-object for all the times a class is virtually inherited in a hierarchy and one sub-object per non-virtual inheritance.

Multiple inheritance can be very problematic and is best avoided until you are adept at designing hierarchies but that might require redesign of your structure sometimes.  Also consider if composition is a better model for your problem.
{: .callout .philosophy}

If you think it might make sense for multiple things to derive from your class and for one thing to be derived from more than one of them, it is best to make the inheritance virtual from the start.
{: .callout .good_practice}

## Casting

Casting is the act of changing the type of an object.
{: .callout .terminology}

Now we have hierarchies of objects we might want to change the static type of an instance from one type to another.  C++ provides four cast operators to help with this.  They are all of the form `cast_type<T>(exp)` where exp is an expression whose result is to be converted to type T.

The four operators are:

`dynamic_cast<T>`
: Cast a reference into a reference to an object in the same hierarchy.  Checks the case is valid at runtime.

`static_cast<T>`
: Same as `dynamic_cast<T>` but only checks at compile time.

`reinterpret_cast<T>`
: Casts between any two objects.  Does no checking - dangerous.

`const_cast<T>`
: Enables const-ness to be changed.

### dynamic_cast

This can only be used with references to classes.  If it cannot case to a valid object of the destination type then throws an exception.

```cpp
#include <iostream>

// My base class - virtual destructor so C++ knows it's a polymorphic base
class MyClass {
public:
    virtual ~MyClass() = default;
};

// My derived class
class MyOtherClass : public MyClass {};

int main() {
	MyOtherClass cls1;
	MyClass cls2;
	
	MyClass & cls3 {cls1};
	MyClass & cls4 {cls2};
	
	// This is fine - the dynamic type of cls3 is MyOtherClass
	MyOtherClass & other_cls1 {dynamic_cast<MyOtherClass&>(cls3)};
	std::cout << "Converted cls1, now for cls2..." << std::endl;

	// Throws an exception at runtime - cannot convert dynamic type MyClass to MyOtherClass
	MyOtherClass & other_cls2 {dynamic_cast<MyOtherClass&>(cls4)};
}
```

In order for C++ to decide that the base class is polymorphic (and therefore can be used with dynamic_cast) is must have at least one virtual method.
{: .callout .technical}

dynamic_cast cannot be used with multiple inheritance or non-public (private/protected) inheritance.
{: .callout .beware}

For dynamic_cast to be available, the C++ compiler needs to have run-time type information (or run-time type identification) - RTTI - enabled.  For some compilers it is always available, for others it is optional and can the turned on and off.  Bjarne Stroustrup deliberately did not include this in the original C++ design because he felt the feature was frequently abused.
{: .callout .philosophy}

### static_cast

This is the preferred way to cast when wanting (supported) explicit conversions, e.g. deliberately converting a double to int (ordinarily this would be a compiler warning), or using a conversion constructor.

```cpp
double a {2.1};
int b {static_cast<int>(a)}; // No warning - we have explicitly said we want to convert

MyOtherClass cls1;
MyClass cls2 {static_cast<MyClass>(cls1)};
```

Only use static_cast if the cast make sense.  It can be used to do stupid things!
{: .callout .beware}

```cpp
MyClass cls1;
// Unlike dynamic_cast, this will not error but will cause bad things to happen at run-time
MyOtherClass & cls2 {static_cast<MyOtherClass&>(cls1)};
```

static_cast is computationally cheaper than a dynamic_cast because it does no runtime checks - so only use it to cast down the hierarchy if you know the cast is sensible and need to be cheap.

Prefer dynamic_cast for casting down the hierarchy in most situations - it is safer as it will throw exceptions if the object cannot be cast.
{: .callout .good_practice}

### reinterpret_cast

This can cast unrelated reference to one-another.

reinterpret_cast does no checking, at compile- or run-time.  It can be extremely dangerous.
{: .callout .beware}

```cpp
// Some class called A
class A {};
// Some unrelated class called B
class B {};

int main() {
	A cls1;
	B & cls2 {reinterpret_cast<B&>(cls1)}; // No error even though A cannot be converted to B!
}
```

This is the C++ equivalent to C style casting:

```c
// Don't do this, it is C code - just for illustration
double a = 2.1;
int b = (int)a;
```

reinterpret_cast cannot cast between const and non-const.

### const_cast

This can convert a const reference to a non-const one *provided the underlying object is not const*.

If the original object is declared const, even a const_cast cannot change that.  It can only change the "const-ness" of a *reference*.
{: .callout .philosophy}

```cpp
#include <iostream>

// An example class
class MyClass {
private:
	int i_; // Just an integer
public:
	// Constructor - initialise i_ to 0
	MyClass() : i_{0} {}
	// const function that can still set i_
	void Func(const int value) const;
	// Getter for i_
	int GetI() { return i_; }
};

// Const function that changes the object - BAD!
void MyClass::Func(const int value) const {
	// "this->i_ = value;" or "i_ = value;" would be a compiler error because the function is const

	// Make 'this' non-const using an explicit const_cast
	const_cast<MyClass*>(this)->i_ = value;
}

int main() {
	int a {5};
	const int & b {a}; // const reference to a

	// "b = 10;" would be an error because b is const
	const_cast<int&>(b) = 10;
	std::cout << a << std::endl; // a is now 10

	MyClass cls1;
	cls1.Func(7); // works fine despite changing the object AND being a const function
	std::cout << cls1.GetI() << std::endl;
}
```

### Final thoughts on casting

Casting should be used sparingly and with great care.  If you thinking of doing a cast, think carefully if you are doing the right thing and if there is a better way (e.g. using polymorphism).

In general do NOT use const_cast or reinterpret_cast.  The former breaks const-correctness and the latter is very dangerous as it turns anything into anything else with no checks that the conversion is possilbe.
{: .callout .bad_practice}
