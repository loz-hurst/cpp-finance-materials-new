---
title: Lesson 17 - Design Patterns
---

# {{ page.title }}
{: .no_toc}

Today's notebook: [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/loz-hurst/cpp-finance-materials-new.git/master?filepath=lessons%2Flesson-17.ipynb)

- TOC
{:toc}

## Recap

Last lesson we looked at Templates, how to define them for functions and classes and touched on some of the benefits and drawbacks of them.

This week, in the final lesson on core contemporary C++ topics, we will look at design patterns.

## Variable length arguments

So far functions have taken a fixed number of arguments, but sometimes we want to take a variable number.  We will look at two ways:

+ Templates for variable numbers of arguments (from C++11)
+ Pass an initialiser list as an argument

A third way, called variadic functions, from C we will discuss in the final lesson.

Variable numbers of arguments with templates are called <em>parameter packs</em>
{: .callout .terminology}

Generally, variable length argument lists have been discouraged in C++.  This could be for a range of reasons but the two obvious ones are that they are a pain to implement and there is a performance hit from processing arguments at runtime compared to using overloading to resolve the function call at compile time.  Using templates addresses both of these issues.
{: .callout .philosophy}

### Parameters packs

A template parameter that accepts zero or more template arguments is called a <em>template parameter pack</em>.
{: .callout .terminology}

A function parameter that accepts zero or more function arguments is called a <em>function parameter pack</em>.
{: .callout .terminology}

To create a function that takes multiple arguments we specify a template parameter pack and matching function parameter pack using an elipsis(`...`, three full-stops):

```cpp
template<typename ...Ts>
void MyFunction(Ts... args);
```

I most cases, the parameter pack must be the last argument in the template and function argument lists.
{: .callout .beware}

We can find out how many arguments are in the pack using the `sizeof...` operator:

```cpp
template<typename ...Ts>
void MyFunction(Ts... args) {
    std::cout << sizeof...(args) << std::endl;
}
```

And we can expand the pack (in certain places) using the name of the pack followed by an ellipsis, e.g. `args...`.  One place where this can happen is in an initialiser list:

```cpp
template<typename ...Ts>
std::vector<std::common_type_t<Ts...>> make_vector(Ts... args) {
    return std::vector<std::common_type_t<Ts...>> {args...};
}
```

```cpp
auto my_vector {make_vector(1, 2, 3, 4, 5, 6)};
```

The ability to expand like this allows a very convenient use of recursion techniques to process each parameter (if that is what is required) rather than counting and looping:

```cpp
template<typename T>
void PrintList(T arg) {
    std::cout << arg << std::endl;
}

template<typename T, typename ...Ts>
void PrintList(T arg, Ts... args) {
    std::cout << arg << ", ";
    PrintList(args...);
}
```

```cpp
PrintList(1, 2, 3, 4.5, "Hello", 5, 6);
```

The above will be a very familiar pattern if you have used functional programming languages before.
{: .callout .philosophy}

Note that the types within a parameter pack can be different, and they will retain their type.

There is a lot more to [parameter packs](https://en.cppreference.com/w/cpp/language/parameter_pack) but this is enough to use them for variable length argument lists.
{: .callout .philosophy}

### Passing initialiser lists

Another approach to passing variable length argument lists is to pass an initialiser list as a single argument.  This is synonymous with passing a structure, like a vector, but an initialiser list is specifically intended for passing arguments (typically to a constructor) so is more intuitive and there is less overhead from the structure.

Like a vector, everything in an initialiser list type object <strong>must</strong> be the same type - note this is different to using uniform initialisation (which uses the same braced-syntax) where the types can be different provided they match a constructor's argument types.
{: .callout .beware}

The initialiser list provides const references to the original object - you cannot pass and modify arguments using this approach but no copy is made either so it is very efficient.
{: .callout .technical}

To do this, we simply use an argument of type `std::initializer_list` (from the `initializer_list` header) and pass the arguments in braces:

```cpp
template<typename T>
void MyFunction(std::initializer_list<T> args) {
    for (const auto& arg: args) {
        std::cout << "Argument: " << arg << std::endl;
    }
}
```

```cpp
MyFunction({"Hello", "World"});
MyFucntion({1, 2, 3, 4});
```

If we wanted to restrict the type passed we could remove the template and replace `<T>` with the type, e.g. `std::initializer_list<int>` or `std::initializer_list<std::string>`.
{: .callout .hint}

## Common design patterns

Design patterns are a huge topic, we could probably do an entire 20-credit module just on this.

Broadly speaking, by "design patterns" we mean "ways of structuring your code, your objects and the interactions between them".  Regardless of how you have done it to date, there is probably a recognised name for the approach you have used.

In 1994 a book, called Design Patterns: Elements of Reusable Object-Oriented Software, described 23 "classic" patterns for software design in an effort to provide a handbook for structuring software.  It [has been cited](https://en.wikibooks.org/wiki/C%2B%2B_Programming/Code/Design_Patterns) as the trigger for the concept of "design patterns" being formally recognised in software engineering.

At the time of writing, Wiki Books has [a page describing 26 different design patterns](https://en.wikibooks.org/wiki/C%2B%2B_Programming/Code/Design_Patterns) many of which you may recognise from your journey on this course.  Some patterns are more common in certain languages than others, for example the observer pattern is common in JavaScript (and other event-driven) applications.

We will look at two specific patterns that we have already mentioned on the course, the singleton and factory pattern.

### Singleton

The singleton pattern ensures that only one instance of a class ever exists.

An example where this would be useful could be to obtain a licence key from a licence file or server for our application.  This might require some significant work, to open and read the licence file or communicate to acquire a licence from a server, but there after the application can re-use the licence until it ends.

To do this, we define a class with a private constructor (so it can only be instantiated by itself or a friend) and either a static member function or a friend function to obtain the instance.

```cpp
class MySingleton {
public:
    // Any accessor methods
    static MySingleton& GetInstance() {
        static MySingleton my_instance; // Create an instance
        return my_instance;
    }
private:
    MySingleton() {} // Default constructor is private
    MySingleton(const MySingleton& other) = delete; // Do not allow the instance to be copied
    MySingleton(const MySingleton&& other) = delete; // Do not allow the instance to be moved
    MySingleton& operator=(const StringSingleton &rhs) const = delete; //disallow copy-assignment operator
    MySingleton& operator=(const StringSingleton &&rhs) const = delete; //disallow move-assignment operator
    ~MySingleton() {} // Private destructor controls who can destroy the object
    // Any instance data/variables
};
```

If we wanted to re-validate a licence periodically, this could also now be trivially added by adding a variable to hold the time the licence needs re-checking and see if it has passed each time an instance is requested or in an appropriate accessor method.
{: .callout .hint}

Other examples where this is particularly useful are:

+ Application wide configuration (load once on first access, use everywhere after that).
+ Shared-memory programming (typically threading) where locks are required (a single lock object than needs to be independently acquired and released by different parts of the program running at the same time).

### Factory

The factory pattern is where direct creation of objects is prevented and instead consumers are forced to use a specific method, typically static member or friend, to construct the object and provide it to them.

It is called the <em>factory pattern</em> and the method that makes the objects a <em>factory method</em> because it "manufactures" the objects and produces them ready for use.
{: .callout .terminology}

Most typically the exact type of the object being constructed is only known at runtime and the factory will apply some logic to work out which specific instance to create.

```cpp
/*
 * Constructs an option of the specified type and returns an Option reference.
 */
Option& MyOptionFactory(OptionType type, OptionData initial_data) {
    switch(type) {
        case OptionType::EurCall:
            return EuropeanCall {initial_data};
        case OptionType::EurPut:
            return EuropeanPut {initial_data};
        case OptionType::AsianCall:
            return AsianCall {initial_data};
        // etc.
    }
}
```

It is very useful for abstracting this sort of logic out of your main code, and helps to make sure that you stick to the generic 'Option' interface which should work with any type of option (provided you have structured your class hierarchy well).

## Lab exercises

### 1. Add template for GetNextField in the base class that will return different types on request - default to `std::string` but provide `int` and `double` specialisations

This could also be (and possibly would be better) done by returning a custom 'InputField' type from GetNextField that internally has conversion operators to `std::string`, `int`, `double` etc.

### 2. Add template for Fixed-width reader that specifies the width of each field at compile time

Last week we covered templates and this week we've looked at variable length parameter lists, so now we can write a template version of fixed-width reader that takes a variable list of template arguments describing the widths of all the fields in the file (and hence the number of fields).

As the number of fields can be determined at compile time, we can use  a `const std::array` of that size to efficiently hold this information.

### 3. Write a factory to create the appropriate InputReader from a delimited-file file-name

We can select the right one based on the file extension.

### 4. Add an overload to this factory method to create a fixed-width InputReader if passed the widths of the fields.
