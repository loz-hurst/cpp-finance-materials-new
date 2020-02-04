<style type="text/css">
.callout {
  border-left: 5px solid black;
  border-top: 1px solid silver;
  border-bottom: 1px solid silver;
  padding-left: 3px;
}
.callout.technical {
  border-left-color: darkslategray;
}
.callout.technical::before {
  content: "\2699";
}
.callout.terminology {
  border-left-color: darkkhaki;
}
.callout.technical::before {
  content: "\d83ddde3";
}
</style>

# Lesson 14 - Dynamic Polymorphism

## Recap

Last lesson we looked at inheritence, how that works in C++ and creating derived a base Option class, as well as derived classes for European Call and Put options.

This week we will look at using inheritence for an important, and powerful, aspect of Object-Orientated Programming - "Dynamic Polymorphism" - as well as how casting works in contemporary C++.

First, however, we will look at "Forward Declarations".

## Foward Declarations

So far, when we needed to reference a class (e.g. in a function declaration that has it as an argument) we `#include` the full header.  When we "mention" (e.g. a pointer, which we will cover later in the course, or by-reference argument) an object we can get away with just saying "this thing exists and is a class".

This is because a pointer or reference uses the same amount of memory regardless of the size of the thing it points to.  This is all the compiler cares about when building individual files.
{: .callout .technical }

To create a forward declaration, we just declare the class as being of type Class but no body:

```cpp
class MyClass;
```

This is telling the compiler "MyClass" is a thing of type class, but nothing mmore about it.
{: .callout .terminology }

You can do this with other types too, if you need to (but remember the type of the thing you are forward declaring has to have been declared - `class` is built into C++ so we do not have to worry about this in the above example).

The forward declared object can then be used, e.g. in this function declaration with a by-reference argument:

```cpp
class MyClass;

void SomeFunction(MyClass & my_class);
```

{::options parse_block_html="true" /}

<div class="callout philosophy">

We can use forward declarations to reduce the number of headers included.  This has multiple benefits such as:

+ reduces compile times, as the number of files to be pre-processed is smaller (less work for the pre-processor) and the resultant file to be compiled, after pre-processing, is smaller (less work for the compiler).
+ reduces the number of files that have to be recompiled when the forward-declared object (and hence its header) changes.  This is especially useful if using compiler caching that detect unchanged tranlation units.

</div>

You should not use forward declarations to circumvent compile-time warnings and errors - if your code will not compile with the full header you have an architectural problem with your program that needs fixing.
{: .callout .bad_practice }
