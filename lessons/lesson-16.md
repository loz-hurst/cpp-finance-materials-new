---
title: Lesson 16 - Templates
---

# {{ page.title }}
{: .no_toc}

Today's notebook: [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/loz-hurst/cpp-finance-materials-new.git/master?filepath=lessons%2Flesson-16.ipynb)

- TOC
{:toc}

## Recap

Last lesson we looked at advanced constructor concepts, special member functions, conversion operators and some small containers.

## Templates

Initially templates may seem like a really complex part of C++ but they are conceptually quite simple.  A template is essentially a way of saying "this function or class can work with many different types".

Consider, for example, a `max` function that takes two arguments and returns the largest one.  We could provide a couple of overloads for this, based on the types:

```cpp
// Returns the largest integer of the two arguments
int max (const int a, const int b) {
	return (a < b) ? b : a; // Return the larger value
}

// Returns the largest double of the two arguments
double max (const double a, const double b) {
	return (a < b) ? b : a; // Return the larger value
}
```

If we carry on with this, obviously we will need to provide many different specialisations - for `long`, `float`, `char` etc. etc..  In principal, however, this function will work correctly to compare any two things of the same type provided a `<` operator exists (i.e. operator< is defined somewhere).  Templates allow us to define a single version of this function that can work with any type.

### Creating template functions

The syntax for specifying a function template is to put the keyword `template` followed by the keyword `typename` or `class` and a place-holder (think of it like a variable representing the template type) enclosed in less-than/greater-than (`<>`) before the function name.  The place-holder can then be used to represent the type (called the specialisation) within the function template.  For example, for our `max` function:

```cpp
template <typename T> T Max(const T& a, const T& b) {
	return (a < b) ? b : a;
}
```

This template will work with any type provided a `bool operator< (const & T lhs, const & T rhs)` (or equivalent, e.g. member version) exists.

Note that because `T` could be any type, including a complex one, we are now taking it by reference as it will be more efficient in some cases.

Templates allow us to write a function once, and the C++ compiler will automatically generate type-specific version when they are used during compilation.
{: .callout .philosophy}

There is not difference at all between `template <typename `&hellip;`>` and `template <class `&hellip;`>`.  `typename` and `class` mean the same to C++ and just say "what follows refers to a type, not an instance of something with that type".
{: .callout .technical}

Likewise we could write a function to sum all the elements of any vector, provided the type inside the vector supports `operator+=` and can be constructed from an integer (which is certainly true for all the built in numeric types):

```cpp
template <typename T> T Sum(const std::vector<T>& vec) {
	T result {0}; // Initialise to 0
	for(const T& elem : vec) {
		result += elem;
	}
	return result;
}
```

We can even have multiple template types:

```cpp
template <typename T, typename U> size_t LongestLength (const std::vector<T>& vec1, const std::vector<U>& vec2) {
	return (vec1.size() < vec2.size()) ? vec2.size() : vec1.size();
}
```

Don't forget the C++ standard libarary already provides efficient versions of `max` and `accumulate` (which can be used to sum) in the `algorithm` and `numeric` libraries - use them and don't re-invent the wheel!  These are examples only.
{: .callout .bad_practice}

Templates can move some runtime polymorphic work to compile time, which increases runtime performance but at the expense of program size, increased compile time and can be hard to test and debug (as different versions of the code are generated automatically).
{: .callout .technical}

Templates are used to capture universal concepts - for example the concept of summing the elements of a vector or finding the larger of two variables are applicable to any type `T` that supports certain operations.  It is more pervasive than could be represented though inheritance in a single object hierarchy.
{: .callout .philosophy}

### Template parameters

Previously a function has parameters, the values (but not the type) of which are determined at run-time.  A function template adds another layer of parameters but these values are known at compile time - a function template has two kinds of parameters:

1. Template parameters (e.g. `typename T`)
2. Function parameters (e.g. `vec`)

Values can be provided via the template parameters, provided that their value is known at compile time.  There are strict restrictions on what types can be used for values passed as template arguments - `enum`, `bool`, and integer types are the most useful ones permitted (floating point types, like double, are comming in C++20).  We have already seen this in use with `std::array` which uses a template parameter to specify the array size - we will revisit this when we look at template classes, below.

Like with function parameters, we can provide default values for our template parameters (both types and values):

```cpp
template <typename T = int> T max (const T& a, const T& b);
```

### Template specialisation

Once we have specified a template function we can create a specialised version if it makes sense (usually for special cases or efficiency).  To do that we still write template but specify nothing for the template parameters and the specific types/values for the specialised version.

Having no template parameters in the specialisation is called a <em>full specialisation</em>.  Conceptually we could also provide some parameters and specify the others, to create a partial specialisation however C++ does not support partial specialisations.
{: .callout .terminology}

For example, to create a specialised version of `max` for boolean values we could write:

```cpp
template <> bool max (const bool& a, const bool& b) {
	return a || b;
}
```

We can only provide a specialised version after, and if, a generic version has been defined.
{: .callout .beware}

### Template overloading

It is also possible to overload a template function, for example to provide a sum function that works on two values of the same type or a vector:

```cpp
template <typename T> T sum(const T& val1, const T& val2) {
    T result {val1};
    result += val2;
    return result;
}

template <typename T> T sum(const std::vector<T>& vec) {
    T result {0}; // Initialise to 0
    for(const T& elem : vec) {
        result += elem;
    }
    return result;
}
```

Try not to mix overloading and specialisation.  The way C++ resolves overloads and specialisations may result in which function is used being counter-intuitive.
{: .callout .bad_practice}

### Template functions go in headers - entirely!

The compiler generates a new version of a template function for every different type(s) it is used with.
{: .callout .technical}

In order to generate this the compiler needs to know the <em>full</em> definition of the template, including the body (as that may use the template type).  The full function template definition is therefore required.

This means template functions should be in the header that is included everywhere the template function is used, do not separate the implementation from the declaration as normal.

The linker is aware of template functions, and so will ensure only one version is kept in the final program.
{: .callout .technical}

### Template type resolution

The types for a template function can be resolved by the compiler in three ways:

1. implicit deduction (a bit like `auto`)
2. explicit specification
3. using the default value(s)

#### Implicit deduction

If the compiler can figure out the type, it will automatically set it.  The complete rules are very complicated but for the simplest cases the type can be obvious:

```cpp
max(1, 2); // T will be determined to be an int
max(1.2, 3.5); // T will be a double
```

If the type cannot be deduced then a compile error will occur:

```cpp
max(1, 2.3); // Error - T is ambiguous
```

Like with overloading, the return types and the types used in the body of the function are not considered when deducing the type.
{: .callout .technical}

#### Explicit specification

We can explicitly specify the type for a template, as we have seen with many of the standard containers:

```cpp
max<int>(1, 2); // Explicitly using an int specialisation
max<double>(1, 2.3); // Explicit choice to use double - no ambiguity
```

But be very careful that the types are correct, especially constants:

```cpp
template <typename T> T new_max(T& a, T&b) {
	return (a < b) ? b : a;
}
```

```cpp
const int a {3}, b {5};
new_max<int>(a, b); // Error - cannot convert const int to int
```

This can be avoided by making the arguments to `new_max` constant - then the above code will work fine (another case where marking the arguments `const` makes the function more flexible).

#### Default values

If the `<>` are provided but no value for the type(s)/value(s) then the default will be used:

```cpp
template <typename T=int> T new_max(T& a, T&b) {
	return (a < b) ? b : a;
}
```

```cpp
new_max<>(2, 5); // Type is int, the default, because of the <>
```

### Class templates

We can also specify template classes, these work very similarly to template functions but work at a class level:

```cpp
template <typename T> class MyClass {
private:
	T my_member_;
public:
	MyClass(const T & value) : my_member_{value} {}
}
```

```
MyClass<int> cls {10};
```

Note that with template classes the type must always be explicitly specified.
{: .callout .technical}

We can create specialised versions of the class by providing specialised constructors:

```cpp
template <> class MyClass<bool> {
private:
	bool my_member_;
public:
	MyClass(const bool value) : my_member_{value} {}
};
```

Note that the specialised version of the class is treated as a completely separate implementation so everything has to be duplicated.
{: .callout .beware}

#### Applying class templates

Using our new found template knowledge, we can create our own 2-dimensional array that uses a `std::array` type to store the array (n.b. we could also have used a `std::array<T, N*M>` and calculated the location in `at`, but it would have required a custom (friend) type for `operator[]` to return in order to emulate `[][]`):

```cpp
// A 2-dimensional NxM matrix of elements of type T
template <typename T, size_t N, size_t M> class Matrix {
private:
	// The actual matrix data
    std::array<std::array<T, M>, N> data_;
public:
	/* 
	 * No argument constructor - delegates to the 1 argument constructor using
	 * T's no argument constructor
	 */
    Matrix();
    // 1 argument constructor - fills matrix with the value passed
    Matrix(const T& initial_value);
    // Get the element at n, m
    T& at(const size_t n, const size_t m);
    const T& at(const size_t n, const size_t m) const;
    /*
     * Get the array at row pos (i.e. n=pos), which can then the chained with
     * that's array's operator[] to emulate [][] 2-dimensioned access.
     */
    const std::array<T, M>& operator[] (const size_t pos) const;
    std::array<T, M>& operator[] (const size_t pos);
};

template <typename T, size_t N, size_t M> Matrix<T, N, M>::Matrix() : Matrix(T()) {}

template <typename T, size_t N, size_t M> Matrix<T, N, M>::Matrix(const T & initial_value) {
    for (size_t i {0}; i < data_.size(); ++i) {
        data_[i].fill(initial_value);
    }
}

template <typename T, size_t N, size_t M> T& Matrix<T, N, M>::at(const size_t n, const size_t m) {
    if ((n >= N) || (m >= M)) {
        throw std::out_of_range {"Index out of range"};
    }
    return data_[n][m];
}

template <typename T, size_t N, size_t M> const T& Matrix<T, N, M>::at(const size_t n, const size_t m) const {
    if ((n >= N) || (m >= M)) {
        throw std::out_of_range {"Index out of range"};
    }
    return data_[n][m];
}

template <typename T, size_t N, size_t M> const std::array<T, M>& Matrix<T, N, M>::operator[] (const size_t pos) const {
    return data_[pos];
}

template <typename T, size_t N, size_t M> std::array<T, M>& Matrix<T, N, M>::operator[] (const size_t pos) {
    return data_[pos];
}
```

```cpp
Matrix<int, 10, 10> my_matrix {0};

my_matrix.at(2, 2) = 5;

std::cout << my_matrix[2][2] << std::endl;
```

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
