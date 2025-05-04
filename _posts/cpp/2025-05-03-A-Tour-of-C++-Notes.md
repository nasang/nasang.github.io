---
layout: post
title: Notes of A Tour of C++
categories: book
tags: [C++, language]
description: >
toc:
  sidebar: right
media_subpath: /assets/img/cs61b/
thumbnail: /assets/img/cpp/a_tour_of_c++.jpg
---

## 1 The Basics

the notation of C++, C++'s model of memory and computation, the basic mechanisms for organizing code into a program

### Programs

C++ is a **complied language**: a program's `source (code) files` have to be compiled by a compiler, producing `object files`, which are combined by a linker yielding a `executable program`. A executable program is not portable, as it is created for a specific hardware/system combination.

C++ is a **statically typed** language: the type of every entity (e.g., object, value, name, and expression) must be known to the compiler at its point of use. The `type` of an object determines the set of `operations` applicable to it and its `layout in memory`.

#### Hello, World!

Every C++ program must have exatly one global function named `main()`, which is the first function the program executes.

```c++
import std;

int main() 
{
  std::cout << "Hello, World!\n";
}
```

- `import std;` instructs the compiler to make the declarations of the standard library available. 
  - The `import` derective is new in C++20, and presenting all of the standard library as a module `std` is not yet standard.

- The operator `<<` ("put to") writes its second argument onto its first.
  - In this case, the _string literal_ `"Hello, World!\n"` is written to the _standard output stream_ `std::cout`.

Note: Normally it is not allowed for the control flow to reach the end of a non-void function without returning something. The `main()` function is handled differently, as specified in the standard. ([StackOverflow](https://stackoverflow.com/questions/19293642/why-does-the-main-function-work-with-no-return-value))

### Functions

People sfecify how an `operation` is to be done by defining a `function`, and the main way of getting something done in a C++ program is to `call` a function. A function cannot be called unless it has been `declared`.

**Declaration of a function**: `<return_type> <function_name>(<argument_type_0>, <argument_type_1>, ...);`. It may contain argument names, which can be helpful to the reader, but is simply ignored by the compiler unless the declaration is also a definition.

```c++
double sqrt(double);
double square(double d);
```

**Type of a function**: `<return_type>(<argument_type_0>, <argument_type_1>, ...)`. For a class's member function, the class name is also part of the type: `<return_type> <class_name>::(<argument_type_0>, <argument_type_1>, ...)`.

```c++
double get(const vector<double>& vec, int index); 
// type: double(const vector<double>&, int)

char& String::operator[](int index); 
// type: char& String::(int)
```

**Argument passing**: the semantics of argument passing are identical to the semantics of the initialization, i.e., argument types are `checked` and implicit argument `type conversion` takes palce when necessary.

**Function overloading**: defining multiple functions with the `same name`, but with `different argument types`. Each function should implement the same semantics. The compiler will choose the `most appropriate` function to invoke for each call.

```c++
void print(int);
void print(double);

void user()
{
  print(42); // call print(int)
  print(9.65); // call print(double)
}
```

If two alternative functions could be called, but neither is better than the other, the call is deemed `ambiguous` and the compiler gives an error.

```c++
void print(int, double);
void print(double, int);

void user2()
{
  print(0, 0); // error: ambiguous
}
```

### Types, Variables, and Arithmetic

### Scope and Lifetime

### Constants

### Pointers, Arrays, and References

### Tests

### Mapping to Hardware

### Advice

## 2 User-Defined Types