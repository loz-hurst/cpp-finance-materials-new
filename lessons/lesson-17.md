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


(see Term 2 Week 10 in old course)

https://en.wikibooks.org/wiki/C%2B%2B_Programming/Code/Design_Patterns

https://en.cppreference.com/w/cpp/numeric/valarray / https://en.wikipedia.org/wiki/Expression_templates

+ parameter packs/variable length argument lists

+ Add oprator>> to our input reader


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

And we can expand the pack (in certain places) using the name of the pack followed by an elipisis, e.g. `args...`.  One place where this can happen is in an initialiser list:

```cpp
template<typename ...Ts>
std::vector<std::common_type_t<Ts...>> make_vector(Ts... args) {
    return std::vector<std::common_type_t<Ts...>> {args...};
}
```

```cpp
auto my_vector = make_vector(1, 2, 3, 4, 5, 6);
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

### Passing initaliser lists

## Common design patterns

### Singleton

### Factory



## Lab exercises

### 1. Finish CSV and Fixed-width input reader specialisations from last week

The recognised format for CSV is documented in [RFC4180](https://www.ietf.org/rfc/rfc4180.txt). Note you will need to cope with quoted fields (which can include newlines-within-quotes).

Sometimes data is provided in a fixed-width data format, where each field is a set number of characters rather than being seperated by a delimiter. Write a new sub-class that can read such a file with the following format (we will generalise it next week):

+ 8 characters for a date (YYYYMMDD)
+ 3 character for the stock symbol
+ 8 characters for the current value - first 4 are pre-decimal point, next 4 are after (i.e. field is price x 10000)

For example:

```text
20200101UOB00012345
20200101GOG01000056
20200102LAH00000005
```

### 2. Add template for GetNextField in the base class that will return different types on request - default to `std::string` but provide `int` and `double` specialisations

This could also be (and possibly would be better) done by returning a custom 'InputField' type from GetNextField that internally has conversion operators to `std::string`, `int`, `double` etc.

### 3. (challenge) Add template for Fixed-width reader that specifies the width of each field at compile time

You will need to read about [template parameter packs](https://en.cppreference.com/w/cpp/language/parameter_pack) and use the list of sizes to initialise a data-structure (you can use `sizeof...()` to get the number of elements in the pack - a `const std::array` of that size would be a good structure to use to hold the field widths).
