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
class MyClass;
```

This is telling the compiler "MyClass" is a thing of type class, but nothing more about it.
{: .callout .terminology}

You can do this with other types too, if you need to (but remember the type of the thing you are forward declaring has to have been declared - `class` is built into C++ so we do not have to worry about this in the above example).

The forward declared object can then be used, e.g. in this function declaration with a by-reference argument:

```cpp
class MyClass;

void SomeFunction(MyClass & my_class);
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
class MyClass {};
class MyOtherClass : public MyClass {};

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

class MyClass {
public:
    virtual void SayHello () const {
        std::cout << "Hello world" << std::endl;
    }    
};

class MyOtherClass : public MyClass {
    virtual void SayHello () const override {
        std::cout << "Hello from another world" << std::endl;
    }
};

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


