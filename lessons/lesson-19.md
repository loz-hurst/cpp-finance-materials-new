---
title: Lesson 19 - Raw Pointers
---

# {{ page.title }}
{: .no_toc}

- TOC
{:toc}

## Today

We will be look at raw pointers and (C-style) arrays, as well as (C-style) variadic functions.

These are all features that are inherited from the C language, which C++ was based upon, and until C++11 were necessary to implement many data structures.  C++11 brought std::array and smart pointers, which replaced both for most, if not all, purposes.
{: .callout .history}

We are looking at these as you may find them in existing code or hear about them from software developers whose knowledge is not up to date with the last 9 years of C++ language evolution.

I reiterate what I said in week 6: **There is no reason to use pointers (or C-style arrays) in C++ since the C++11 standard.**
{: .callout .beware}

## Pointers

Pointers a a little like references, but lower level (closer to how the computer works) and with much less protection for you, the programmer.

Before we get into the details, let's look go back to basics with variables.  So far we have been thinking of variables as representing values (i.e. placeholders), and those values are usually mutable (meaning they can be changed).  To understand pointers, we have to first realise that variables are not place holders for variables but they are human-friendly labels for a piece of memory.  When your program is compiled the names of variables are, in almost all cases, removed entirely from the compiled program.

Consider the code to declare an integer called 'b':

```cpp
int b {5};
```

In our compiled program, this line will generate the instructions to ask the operating system for a piece of memory the right size for an integer (4 bytes) and initialise it with the value 5.  In memory this might look like the following (b is just our program's internal label for that memory "slot"):

![b in a memory slot illustration](/assets/images/Memory-slots-int-b-5.png)

In the diagrams, each box represents a memory slot, the letters above are the internal labels of our program and the contents of the box the value actually stored in memory.  The slots for each variable as the same size - in reality the number of bytes allocated to a variable will vary based on type.
{: .callout .hint}

With references, which we have used before (e.g. as by-reference arguments to functions), you are effectively creating a new label for the same slot.  For example, if we make a reference to b called d it is like just adding a new label 'd' alongside the label 'b' in the diagram above:

```cpp
int & d {b};
```

![b and d illustration](/assets/images/Memory-slots-int-b-d-5.png)

Pointers are very different in this regard, they are variables in their own right and have their own memory slot.  In that memory slot is a memory address, and by de-referencing the point we are able to access the value in the memory *at the address in the pointer*.

It is very important to note that a pointer is just a memory address.  All pointers are the same size in memory (4 or 8 bytes on 32 or 64-bit computer systems respectively) and, crucially, the "type" of the pointer just tells C++ how to interpret the value at the target location.
{: .callout .technical}

A pointer is declared using an asterisk (\*) after its typename.  The address of a variable can be found using the ampersand (&) operator.  For example to create an pointer to b, called a, we would use this code:

```cpp
int * a {&b};
```

Here we see two more cases where the same symbol has subtly different use in C++.  We have previously met the asterisk (\*) as the de-reference operator, but now it is part of the type declaration is means the variable is a pointer to that type.  Likewise, we have seen the ampersand (&) for reference arguments - when used as part of the type declaration is means a reference to another variable but now it is used as an operator with a variable name to get it's address.
{: .callout .philosophy}

`a` is a pointer, which is just a memory address, but it is declared as with `int *`.  This means C++ will try and interpret whatever is stored at that address as an int when it is de-referenced 

Because `a` points to an int, we call it an *int pointer*.
{: .callout .terminology}

In memory, our a and b variables now looks like this:

![a pointing to b illustration](/assets/images/Memory-slots-a-points-to-int-b-5.png)

Unlike a reference we can update the value at `a` directly (`a` is just a variable, really) by not dereferencing it and assigning a new memory address to it (e.g. the address of another int).

References cannot be changed once initialised (and must be initialised when created).  Pointers can be changed and do not have to be initialised when created.
{: .callout .technical}

An uninitialised pointer will have whatever value was left behind in the memory allocated to it.  This will be interpreted as a memory address (the bits will always be able to be interpreted as a number) but could point to anywhere.  With pointers it is crucially important to always initialise them to a known value.
{: .callout .beware}

C++11 introduced the `nullptr` value.  It can be used to initialise pointers and explicitly indicates a pointer to nowhere (as opposed to one that could be pointing to anywhere).

Consider if we do not initialise out pointer `a`.  It will be allocated some memory and its value (which is the address of the value to read when dereferenced) will be whatever value is left behind by the last program that used the piece of memory:

```cpp
int * a;
```

![uninitialised a pointing to somewhere, anywhere illustration](/assets/images/Memory-slots-a-unitialised.png)

As you can see, `a` refers to a slot in memory that could contain anything and may or may not be allocated to our program.  What happens when `a` is dereferenced could vary from garbage data being returned to a segmentation fault.

If we initialise a properly, C++ will give an error if our program tries to access a nullptr (so it will fail predictably if we make a mistake):

```cpp
int * a {nullptr};
```

![nullptr a illustration](/assets/images/Memory-slots-a-nullptr.png)

This is another place pointers and references differ - a reference cannot point to nothing, they must be initialised to an object.  There is no such thing as a null reference (but there can be a reference to an object that no longer exists in memory, that reference will still be a label for the memory it used to be at).
{: .callout .technical}

Pointers can be passed like any other value, and because they are basic numbers (memory addresses) pointers themselves are safe to return from functions - but beware that the thing they point to might be destroyed (just like with references).

Consider this function which returns a pointer to a locally declared variable:

```cpp
int * my_function() {
    int b {5};
    int * a {&b};
    return a;
}
```

When the return code is run, a and b could be in memory just as we saw before:

![a pointing to b illustration](/assets/images/Memory-slots-a-points-to-int-b-5.png)

Imagine if this code is used to capture the return value of the function:

```cpp
int * c {my_function()};
```

A new variable, c, which is an int pointer will be created and initialised with the result of calling my_function.  This will initialise c with a copy of a (the pointer is returned by-value), which is the memory address of b.  However, as soon as the function completes, b is destroyed as is a so c ends up pointing to the address where b *used to be*.  A this point that memory may no longer be allocated to our program (in which case expect a segmentation fault when it is dereferenced) or may be reused by another variable:

![c pointing to where b used to be illustration](/assets/images/Memory-slots-c-points-to-old-b.png)

You can also create pointers that point to pointers.  To do this, you just add another asterisk (\*) to the typename, indicating that the type is now a pointer to an int pointer (or an int pointer pointer):

```cpp
int b {5};
int * a {&b};
int ** c {&a};
```

Here, c is a pointer to a pointer to an int.  In other words, stored in c is a memory address and at that memory address another memory address is stored, and at that 2nd memory address there is an int.

![c pointing to a pointing to b illustration](/assets/images/Memory-slots-c-points-a-points-to-b.png)

I order to access a, c must be dereferenced and in order to access the int that a points to a must then be dereferenced:

```cpp
int b_value {**c};
```

Some will refer to this a dereferencing c twice, or a double dereference. Strictly speaking, c only gets dereferenced once and then the result of that gets separately dereferenced - but most C++ programmers will be happy with either terminology.
{: .callout .terminology}

### Pointer arithmetic

Pointers are just values, although that value is a memory address.  These values are integers, normally the number of bytes from the start of the computer's memory at which the allocated memory is (but this does depend on the underlying computer).  Because they are just values we can do arithmetic on them.

When you add or subtract from a pointer, C++ will add (or subtract) space for that many *of the type of the pointer* from the memory address.  For example, it I add 4 to an int pointer, C++ will increase the memory address by space for 4 ints (if an int is 4 bytes, the memory address will be increased by 16 bytes).  In the same way, if I subtract 2 from a char pointer, C++ will decrease the memory address by space for 2 chars (if a char is 1 byte, the memory address will be decreased by 2 bytes).

This sounds complicated bit leads to very and intuitive code, if you have an int pointer then adding one to it will move to the integer next to it in memory.  However this only makes sense if you **know** that there is an integer next to it in memory, which is where arrays come in.

So, beware - C++ does no checks that when you manipulate a pointer you are doing something sensible.
{: .callout .beware}

For example:

```cpp
int b {5};
int * a {&b}; // a points to b
++a; // a points to the memory 1 int's width passed b
```

`a` now points to a piece of memory that is either used for something else or is not allocated to our program (in which case expect a segmentation fault):

![a pointing past b illustration](/assets/images/Memory-slots-a-points-past-b.png)

Another "gotcha" to be careful of: do not assume that elements of a struct are stored contiguously (one after another) in memory.  I have helped researchers whose code produced weird results because of this mistake.
{: .callout .beware}

```cpp
struct MyStruct {
    int a;
    int b;
};
```

```cpp
MyStruct a {1, 2};
int * b {&a.a};
// May or may not make b point to a.b - it's not guaranteed either way!
++b; 
```

### Pointers and objects

Like references, points to objects exhibit both a static and dynamic type, although with pointers the distinction is a little clearer.  The pointer itself is it's own variable and has its own type (e.g. int pointer), the thing it points to also has its own type which, in the case of inheritance, maybe a sub type of the pointer's type.

From the point of view of the programmer, creating a polymorphic pointer is very similar to a reference:

```cpp
DerivedClass derived_instance; // Instance of something derived from BaseClass
Base * base_pointer {&derived_instance}; // Pointer to the derived instance
Base & base_reference {derived_instance}; // Reference to the base instance
```

Remember that pointers themselves are memory addresses and need to be dereference, where as references can be directly used as aliases to the original variable:

```cpp
base_pointer->some_method(); // or (*base_pointer).some_method();
base_reference.some_method();
```

Virtual and non-virtual methods will work through pointers exactly the same as with references.

For straight-forward pointers to objects, there is no benefit compared to references (and many potential pitfalls) - so stick to references.
{: .callout .good_practice}

## (C-style) arrays

Finally we come onto C-style arrays.  Why have we left these until now?  The answer is that they are essentially just pointers, and so we need to know all about pointers first.

C-style arrays come with all the pitfalls and difficulties of using raw pointers (they are essentially the same thing).  Do not use them unless you have to.
{: .callout .bad_practice}

A C-style array is declared using square brackets after the variable name with the size of the array to be created.

Up until C++14 the size had to be a compile-time constant value (i.e. const or literal).  C++14 changed that, although some compilers (most notably GCC) supported it as a non-standard extension before then.
{: .callout .history}

```cpp
int a[5]
```

Make careful note: `a` is not itself an array of integers.  `a` is a **pointer** to a piece of memory large enough to store 5 integers sequentially.
{: .callout .beware}

This is what we really have in memory:

![a pointing past b illustration](/assets/images/Memory-slots-a-points-to-int-array.png)

This means a number of things, all of which are problems for us as programmers:

+ A C-style array does not know its own size
+ There is no way to detect the end of the array if looping over it
+ Passing an array to a function is like passing a pointer - no size information is available

If the array is part of a struct, however, and you copy the struct the memory of the actual array gets copied - that is every single element one-by-one.  Which is very inefficient but means that an array inside a struct is safe to pass by value.  It is not safe to directly pass a C-style array by value as that just copies the pointer.

In this function, for example, the size of the array is specified by completely ignore by C++ (the standard says this is how it must behave) so `my_array` will be a pointer to the start of which ever array was passed in - which may have more than 5 elements or, dangerously, less than 5 (in which case expect a segmentation fault):

```cpp
void my_function(int my_array[5]) {
    // C++ does not check that my_array has more, less or equal to 5 elements!

    //...
}
```

To access elements in an array we can use the `[]` subscript operators:

```cpp
const int SIZE {5}; // Size of the array
int a[SIZE];
// Initialise the array
for(int i {0}; i < SIZE; +=i) {
    a[i] = 0;
}
```

Note that the size of the array is a variable.  It is important that you use a constant variable to hold the size and use it everywhere you need to refer to the size.  This is because if you mistype a literal number (e.g. hit 6 instead of 5 for the loop condition) the program will compile and may or may not fail at runtime (depending on what memory it overflowed into).  If you mistype the name of a variable the program will not compile, so you know your bounds are correct if it does build.
{: .callout .good_practice}

If you go beyond the end of the array, this is called an *overflow*.  If you go beyond the start of the array (e.g. by decrementing a pointer from the end), this is called an *underflow*.
{: .callout .terminology}

If you want to know how easy it is to make a mistake with traditional arrays, just look at the number of security holes in software caused by array (typically used for buffers) over- and under-flows.  Like with pointers, there is no legitimate use case for them in modern C++.
{: .callout .philosophy}

To be safe, every time you access an array by index you must make sure that that index is within the bounds of the array (i.e. 0 <= index < array_size).  With a for loop, the loop initialiser and check will do that for use but for indexes from elsewhere, for example passed as a function argument, the checks must be done manually.

Note that you can use the subscription operator on a raw pointer too (because C-arrays and pointers are basically the same things):

```cpp
int b {5};
int * a {&b};
a[1]; // The int 1 int's width past b in memory.  Same as *(a++) or *(a+1).
```

The array variable can also allow access to elements via basic pointer arithmetic (because it is a pointer):

```cpp
const int SIZE {5}; // Size of the array
int a[SIZE];
// Initialise the array
for (int i {0}; i < SIZE; ++i) {
    *(a+i) = 0;
}
```

## Dynamic memory

So far everything we have done has let C++ (and the standard libraries) manage our memory for us.  This is good practice, however (as previously mentioned) it is possible to take over managing the memory ourselves.

Memory manually managed this way is *dynamic memory*.
{: .callout .terminology}

Using this memory is intimately tied to pointers as all of the access to dynamic memory is done through pointers.

Be very, very careful if you do this.  It is easy to make a mistake and not free-up the memory again when you have finished, in which case the memory will remain allocated to your program (until it exits and the operating system reclaims everything your program was using).

Allocating and not freeing memory correctly is called *leaking memory*.  This is because the memory is no longer used by your program but is still allocated to it an unavailable to anything else on the computer.
{: .callout .terminology}

Be aware that if your code or other libraries and functions throw exceptions, the exception handling might by-pass your code to release the memory.  This is common source of memory leaks.
{: .callout .beware}

If you start manually managing memory, make sure there is no path through your program, including via exceptions, that does not later free that memory back up.
{: .callout .good_practice}

Dynamic memory is allocated in a separate area to the variables C++ manages for us. There are two ways to create and release memory in that space, new & delete and malloc & free.  We will look at both briefly, but first note some important things that apply to both.

The space for dynamic memory is called *the heap* and the space that C++ manages is called *the stack*.
{: .callout .terminology}

It is very poor programming practice to rely on your memory being released when your program exits - always make sure your program releases all of its resources properly before it ends.
{: .callout .bad_practice}

As John Dibling says, if you are thinking of using dynamic memory directly you should think again.  And then have another think about an alternative approach:

> And in fact, in modern C++ using new should be a choice of last resort, and using malloc should be even more last-resort than that.
*(John Dibling, 24 August 2014, https://stackoverflow.com/a/25475733 accessed 21 March 2020)*

### Pointers, pointers everywhere!

Regardless of how you allocate the dynamic memory, what you are given is a pointer to that memory in a variable in the stack.  You need a pointer to that memory in order to free up the memory later.  If all copies of the pointer get destroyed (e.g. are local to a function) then you will not be able to free that memory.

If you do not have a pointer to the memory, you cannot free it.
{: .callout .beware}

This means you will have to find a way to keep a copy of the pointer, by returning it or passing it to other functions, until you are ready to release the memory.  If you are using dynamic memory inside a class, then you can keep the pointer in a class member and release the memory in the destructor.

Using dynamic memory and freeing the memory on destruction (or after reallocating due to a size change, in vector) is how most of the standard library containers work, such as vector, array and the smart pointers.
{: .callout .technical}

Once you allocate memory using either method, that memory (and it's contents) will not be destroyed, moved or changed unless you program explicitly says to do one of those things.

What ever you put in that memory will stay there until you tell C++ to free it, no matter what functions you call or return from in the meantime.

You need to make sure your program always frees the memory again but also that it only does it exactly once.
{: .callout .good_practice}

Trying to free the same memory more than once is called a *double free*.
{: .callout .terminology}

A double free will usually corrupt your program's internal memory - many implementations will cause your program to blow up immediately if you do it (glibc gives a "double free or corruption" error, for example).
{: .callout .beware}

### New & Delete

The first method of get dynamic memory is to use new & delete.  The argument to new is the object to create, and it returns a pointer to that object.

```cpp
const int SIZE {5};
// Allocate space for 5 consecutive integers on the heap, returns a pointer to it
int * my_array {new int[SIZE]};

for(int i{0}; i < SIZE; ++i) {
    my_array[i] = i; // use the pointer like an array
}

delete(my_array); // Free up the memory that was allocated
```

This can also be used with objects and structs:

```cpp
MyClass * my_class {new MyClass()};

// ...  (remembering my_class is a pointer, so needs dereferencing)

delete(my_class);
```

### Malloc & Free

This second method is slightly different to new and delete, but still returns a pointer to the memory.  Instead of telling it to create an object on the heap, however, we have to ask for a specific amount of memory (in bytes) to be allocated.  The method, malloc, to allocate the memory returns a void pointer (void \*) type which must be cast to whichever type you want to store in that memory.

Memory allocated with malloc must be freed by calling free with the pointer to that memory.

For example:

```cpp
const int SIZE {5};
// We can do this directly with malloc because char is 1 byte
char * my_string {static_cast<char *>(malloc(SIZE + 1))};
for(int i {0}; i < SIZE; ++i) {
    my_string[i] = '0' + i;
}
/*
 * Strings must be null-terminated -
 * this is what the extra malloc'd byte is for.
 */
my_string[SIZE] = '\0';

std::cout << my_string << std::endl;

free(my_string);
```

For anything bigger than char (which is 1 byte) we need to use the sizeof operator to find how much memory to allocate:

```cpp
const int SIZE {5};
int * my_ints {static_cast<int *>(malloc(sizeof(int) * SIZE))};
for(int i {0}; i < SIZE; ++i) {
    my_ints[i] = i;
}

for(int i {0}; i < SIZE; ++i) {
    std::cout << my_ints[i] << ", ";
}

free(my_ints);
```

Never try to use malloc with complex object (your own or from the standard library).  `malloc` does not call constructors, `free` does not call destructors.
{: .callout .bad_practice}

## Virtual destructors

The short message is, always make your destructors virtual.
{: .callout .good_practice}

If your object is created using new and the pointer to it is a base-type pointer (e.g. your class is derived from Base and the pointer is of type "Base \*"), then when that object is destroyed (by calling delete on the pointer) your destructor will only be called **if the base class destructor is virtual**.

To be safe, regardless of how your class is used, always make destructors virtual.

You might be using your class with new and delete without realising - std::vector and smart pointers use them internally to manage the life cycle of the objects they contain.  You will have problems if you have destructor in your class hierarchy and it is not virtual.
{: .callout .beware}

## Variadic functions

We looked at variadic templates two lessons ago and I mentioned there was another type of variadic function, from C.  I'm about to describe them here, but using them is quite complicated.

Variadic functions are defined by putting `, ...` after the named arguments.  For example:

```cpp
void MyFunction(const int count, ...);
```

You must have at least one name argument.
{: .callout .beware}

Since C98, the C language required the comma before the elipsis(...) but did not prior to that - so `void MyFucntion(int count...);` was valid.  The C++98 standard supported both for compatibility reasons.
{: .callout .history}

 You should not use the no-comma version, it is deprecated in C and only supported for historic reasons in C++.
{: .callout .bad_practice}

I'll provide an example of how to use them, rather than try and describe it:

```cpp
#include <stdarg.h> // the C header for variable length arguments

void MyFunction(const int count, ...) {
    // Argument count must be the number of variable arguments
    va_list args; // argument information structure
    /*
     * Prepare to process the variable arguments -
     * first argument must be an instance of va_list, the second
     * is the last named argument (to figure out which arguments)
     * are in the variable list.
     */
    va_start(args, count);
    for(int i {0}; i < count; ++i) {
        /*
         * Get the next argument.  First argument to va_arg is the 
         * va_list structure passed to va_start, the second is the
         * type of the next argument.  Note that we have to code
         * the type for each argument here. va_arg will return that
         * type.
         */
        std::string next_arg {va_arg(args, std::string)};
    }
    va_end(args); // Tell C(++) we have finished processing the arguments
}
```

As you can see, it is a pain to use.  Types of arguments have to be hard-coded into your functions as you loop over them.  Templates are more straight-forward and no less flexible.

Like pointers, the C style variadic functions cannot tell how many arguments are being passed into the function.  You have to figure out how many arguments there are another way, for example by having the count passed in as an argument as I have done or inferring it from some other information.
