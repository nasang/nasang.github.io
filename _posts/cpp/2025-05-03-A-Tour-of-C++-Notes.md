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

Disclaimer: This note is based on [_A Tour of C++ (Third Edition)_](https://www.stroustrup.com/tour3.html) by [Bjarne Stroustrup](https://www.stroustrup.com/index.html) and is intended for learning purposes only. All rights to the original work belong to the author and publisher.

## 1 The Basics

An overview of C++ syntax, memory model, and basic mechanisms for organizing code.

### Programs

C++ is a **compiled language**: a program's `source (code) files` are compiled into `object files`, which are then linked to produce an `executable program`. Executables are not portable, as they are built for a specific hardware/system combination.

C++ is a **statically typed** language: the type of every entity (e.g., object, value, name, and expression) must be known to the compiler at the point of use. The `type` of an object determines the set of applicable `operations` and its `layout in memory`.

#### Hello, World!

Every C++ program must have exatly one global function named `main()`, which is the program's entry point.

```c++
import std;

int main() 
{
  std::cout << "Hello, World!\n";
}
```

- `import std;` makes the declarations from the standard library available. 
  - This `import` directive is new in C++20. Presenting the entire standard library as a module named `std` is not yet fully standardardized.

- The operator `<<` ("put to") writes its second argument onto its first.
  - In this case, the _string literal_ `"Hello, World!\n"` is written to the _standard output stream_ `std::cout`.

Note: Normally, non-void functions must explicitly return a value, but `main()` is an exception as specified by the standard. ([Reference](https://stackoverflow.com/questions/19293642/why-does-the-main-function-work-with-no-return-value))

### Functions

In C++, functions specify how operations are performed and are the primary means of executing code. A function must be declared before it can be called.

**Declaration of a function**: `<return_type> <function_name>(<argument_type_0>, <argument_type_1>, ...);`. Argument names are optional in declarations (but helpful to readers); they are ignored by the compiler unless the declaration is also a definition.

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

**Argument passing**: Follows the semantics of initialization. Argument types are `checked`, and `implicit type conversions` may occur.

**Function overloading**: Multiple functions can share the `same name` if they differ in `argument types`. Each function should implement the same semantics. The compiler will choose the `most appropriate` function to invoke for each call:

```c++
void print(int);
void print(double);

void user()
{
  print(42); // call print(int)
  print(9.65); // call print(double)
}
```

If two matches are equally good, the call is `ambiguous`, and the compiler emits an error:

```c++
void print(int, double);
void print(double, int);

void user2()
{
  print(0, 0); // error: ambiguous
}
```

### Types, Variables, and Arithmetic

A **declaration** is a statement that introduces an entity into the program and specifies its type:
- A `type` defines a set of possible values and a set of operations (for an object).
- An `object` is some memory that holds a value of some type.
- A `value` is a set of bits interpreted according to to a type.
- A `variable` is a named object.

Type sizes are implementation-defined (i.e., can vary among different machines). Use `sizeof(type)` to get the size in bytes. For example, `sizeof(char)` equals `1`, and `sizeof(int)` is often `4`. When we want a type of a specific size, we use a standard-library type alias, such as `int32_t`.

Numbers can be floating-point or integers.
- Floating-point: `3.14`, `314e-2`
- Integer literals are by default decimal.
  - `0b` prefix: binary, e.g., `0b10101010`
  - `0x` prefix: hexadecimal, e.g., `0xBAD12CE3`
  - `0` prefix: octal (base 8), e.g., `0334`

We can use a signal quote `'` as a digit separator for readability, e.g., `3.14159'26525'89793`.

#### Arithmetic

```c++
x^y; // bitwise exclusive OR
~x; // bitwise complement
```

#### Initialization

An object must be initialized before it can be used. C++ offers several syntaxes for initialization, such `=`, and a universal form based on `{}` initializer lists.

```c++
double d1 = 2.3;
double d2 {2.3};
double d3 = {2.3}; // the = is optional with {...}

vector<int> v {1, 2, 3};
```

**Narrowing converions**, such as `double` to `int` and `int` to `char` are allowed and implicitly applied with `=`, but not allowed with `{}` to prevent information loss.

When the type can be deduced from the initializer, you can omit it using `auto`:

```c++
auto b = true; // bool
auto ch = 'x'; // char
auto i = 123; // int
```

With `auto`, we tend to use the `=` because there is no potentially troublesome type conversion involved.


### Scope and Lifetime

```c++
vector<int> vec; // global

void fct(int arg) // fct is global
                  // arg is local
{
  string motto {"Who dares wins"}; // motto is local
  auto p = new Record{"Hume"}; // p points to an unnamed Record (created by new)
  // ...
}

struct Record {
  string name; // name is a member of Record
  // ...
}
```

### Constants

### Pointers, Arrays, and References

An **array** is a contiguous block of memory containing elements of the same type. It is the most primitive form of data collection and closely reflects what the hardware provides.

In a _declaration_: 
- `[]` means `"array of"`
- `*` means `"pointer to"`

```c++
char v[6]; // array of 6 characters
char* p; // pointer to character
```

In an _expression_: 
- Unary prefix `&` means `"address of"`
- Unary prefix `*` means `"content of"`:

```c++
char* p = &v[3]; // p points to v's fourth element
char x = *p; // *p is the content that p points to
```

C++ supports a **range-for-statement**, for iterating over sequences:

```c++
void print()
{
  int v[] = {0, 1, 2, 3, 4, 5}; 
  // we don't have to speficy an array bound when we initialize it with a list

  for (auto x: v)  // copies each element into x
    cout << x << '\n';

  for (auto x: {10, 20, 30}) // iterates over an initializer list
    cout << x << '\n';
  // ... 
}
```

To **avoid copying** and instead work with each element directly, use a reference:

```c++
void increment()
{
  int v[] = {0, 1, 2, 3, 4, 5}; 

  for (auto& x: v) // x refers to each element
    ++x;
  // ...
}
```

A **reference** behaves like a pointer except that:
- No need to access the value with `*`
- Cannot be reseated to refer to another object once initialized

Reference are particualarly useful for specifying function arguments. This avoids copying the entire object and allows in-place modification. 

```c++
void sort(vector<double>& v);
```

To allow **read-only access without copying**, use a `const reference` (i.e., a reference to a `const`):

```c++
double sum(const vector<double>& v);
```

In declarations, operators like `[]`, `*`, and `&` are called **declarator operators**:

```c++
T a[n]  // T[n]: a is an array of n elements of type T
T* p    // T*: p is a pointer to T
T& r    // T&: r is a reference to T
T f(A)  // T(A): f is a function taking an argument of type A and returning a result of type T
```

#### The Null Pointer

`nullptr` represents a null pointer—used when no valid object is available to point to. It replaces the older conventions `0` and `NULL`, which could be confusing because they are also valid integer values.

```c++
int count_x(const char* p, char x)
{
  if (p == nullptr)
    return 0;
  int count = 0;
  while(*p) { // equivalent to while(*p != 0)
    if(*p == x) 
      ++count;
    ++p; // advance the pointer to the next element
  }
  return count;
}
```

Here, `char*` denotes a C-style string. Since characters in a string literal are immutable, so the function parameter is declared as `const char*`.

Thre is no null referece. A reference must refer to a valid object.

### Tests

C++ provides a set of statements for expressing selection and looping, such as `if`, `switch`, `while`, and `for`.

```c++
bool accept()
{
  cout << "Do you want to proceed (y or n)?\n";
  char answer = 0;
  cin >> answer;

  if (anser == 'y')
    return true;
  return false;
}
```

The `>>` operator (`"get from"`) is used for input; `cin` is the standard input stream.

An `if`-statement can introduce a variable and test it.

```c++
void do_something(vector<int>& v)
{
  if (auto n = v.size(); n != 0) {
    // ...
  }
  // ...
}
```
A name declared in a condition is in scope on both braches of the `if`-statement. This helps keep the scope of the variable limited, improving redeability and reducing errors.

Prefer omitting explicit mention of the condition, when testing a variable against `0` or the `nullptr`.

```c++
void do_something(vetor<int>& v)
{
  if (auto n = v.size()) {
    // ...
  }
  // ...
}
```


### Mapping to Hardware

C++ offers a direct mapping to hardware. A fundamental operation is typically implemented with a single `machine operation`. For example, adding two `int`s, executes an integer add machine instruction.

C++ views a machine's memory as a sequence of memory locations where it can store _typed_ objects and address them using `pointer`s. The numeric value of a pointer corresponds to a machine address.

The basic machine model of C and C++ is based on `computer hardware`. The simple mapping of fundamental language constructs to hardware is curcial for the `raw low-level performance` for which C and C++ have been famous for decades.

#### Assignment

An `assignment` of a built-in type is a simple machine `copy` operation. The assigned-to object gets the value from the assigned object, yielding two **independent** objects with the same value.

```c++
int x = 2;
int y = 3;
x = y; // y's value is copied to x
```

```c++
int x = 2;
int y = 3;
int* p = &x;
int* q = &y;
p = q; // q's value is copied to p; so p == q, and *p == *q
```

A reference and a pointer both refer/point to an object and both are represented in memory as a machine address. However, **assignment to a reference does not change what the reference refers to but assigns to the referenced object**.

```c++
int x = 2;
int y = 3;
int& r = x;  // r refers to x 
int& r2 = y; // r2 refers to y
r = r2; // read through r2, write through r: x becomes 3
```

### Initialization
The assigned-to object must have a value, for an assignment to work correctly. On the other hand, initialization is to make an uninitialized piece of memory into a valid object.

We cannot have an unintialized reference.

```c++
int& r2; // error: uninitialized reference
r2 = 99;
```

If we could, `r2=99` would assign `99` to some unspeficied memory location, eventually leading to bad results or a crash.

```c++
int x = 7;
int& r {x}; // bind r to x (r refers to x)
int& r2 = x; // bind r to x (r refers to x); not any form of value copy
```

The basic semantices of `argument passing` and `function value return` are that of initialization.

## 6 Essential Operations