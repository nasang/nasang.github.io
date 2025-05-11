---
layout: post
title: A Tour of C++ | Notes 1 Basics (Ch 1 - Ch 4)
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

## 2 User-Defined Types
- **Built-in types** are built from _fundamental types_, `const` modifier, and _declarator operators_.
- **User-defined types** are built out of other types using C++'s abstraction mechanisims, referred to as _classes_ and _enumerations_.

### Structures

The first step in creating a new type is often to group related elements into a `struct`:

```c++
struct Vector {
  double* elem;
  int sz;
};
```

To use it:

```c++
void vector_init(Vector& v, int s)
{
  v.elem = new double[s];
  v.sz = s;
}
```

`new` allocates memory on the _free store_ (a.k.a _dynamic memory_, or _heap_), which must be explicitly deallocated with `delete`.

Use `.` to access `struct` members through a name, or a reference and `->` through a pointer.

```c++
void f(Vector v, Vector& rv, Vector* pv)
{
  int i1 = v.sz;
  int i2 = rv.sz;
  int i3 = pv->sz;
}
```

### Classes

While a `struct` expose all members by default, a `class` allows encapsulation: separating interface (`public` members) from implementation (`private` members).

```c++
class Vector {
public:
  Vector(int s): elem{new double[s]}, sz{s} {}
  double& operator[](int) {return elem[i];}
  int size() {return sz;}

private:
  double* elem;
  int sz;
}
```

The number of elements in a `Vector` can vary across objects and even over the lifetime of a single object. However, the `Vector` instance itself always has a fixed size. This illustrates a common C++ technique: using a fixed-size object as a _handle_ to manage a variable-size block of data allocated elsewhere.


A **constructor** (a member function with the same name as its class) initializes members using a _member initializer list_: `elem{new double[s]}, sz{s}`.

We can define constructors and other member functions for a `sturct` as well. There is no fundamental difference between a `struct` and a `class`; a `struct` is simply a `class` with members `public` by default.

### Enumerations
Enumerations are used to represent small sets of named constants (integer values), improving code readability and safety.

```c++
enum class Color { red, blue, green };
enum class Traffic_light { green, yellow, red };
```

`enum class`  introduces **strongly typed** and **scoped** enums.

```c++
Color c1 = red;                // error: which red?
Color c2 = Traffic_light::red; // error: not a Color
Color c3 = Color::red;         // OK
auto c4 = Color::red;          // OK
```

Implicit conversion to/from integers is not allowed:

```c++
int i = Color::red;  // error: Color::red is not an int
Color c = 2;         // error: 2 is not a color 

Color x = Color{5}; // OK, explicit cast
Color x {4}; // OK
```

We can define operators for an `enum class`.

```c++
Traffic_light& operator++(Traffic_light& t)
{
  using enum Traffic_light; // avoid repetition of the enumeration name

  switch(t) {
    case green: return t=yellow;
    case yellow: return t=red;
    case red: return t=green;
  }
}
```

A "plain" `enum` (i.e., without `class`) exposes its enumerators directly into the surrounding scope and allows implicit conversion to and from integers. While this makes them less well behaved, they remain common in modern code due to their long-standing presence in both C++ and C.

```c++
enum Color { red, green, blue };
int col = green; // col gets the value 1
```

### Unions

A `union` is a `struct` in which all memebers are allocated at the same address so that the `union` occupies only as much space as its largest member.

```c++
union Value {
  Node* p;
  int i;
};
```

The language dones't keep track of which kind of value is held by a `union`, so the programmer must do that:

```c++
enum class Type { ptr, num };

struct Entry {
  string name;
  Type t;
  Value v;// use v.p if t==Type::ptr; use v.i if t==Type::num
}

void f(Entry* pe)
{
  if (pe->t == Type:num)
    cout << pe->v.i;
  // ...
}
```

Maintaing the correspondence between a _type field_, sometimes called a _discriminant_ or a _tag_, (here, `t`) and the type held in a `union` is error-prone. 

To avoid this, encapsulate the logic in a class, or better, use a `variant` from the standard library.

A `variant` hold one value from a set of types:

```c++
struct Entry {
  string name;
  variant<Node*, int> v;
};

void f(Entry* pe)
{
  if (holds_alternative<int>(pe->v))
    cout << get<int>(pe->v);
}
```

For many uses, a `variant` is simpler and safer to use than a `union`.

## 3 Modularity

C++ represents interfaces using declarations. A **declaration** specifies everything needed to use a function or a type:

```c++
double sqrt(double);
```

The function’s **definition** can be provided elsewhere:

```c++
double sqrt(double d)  // definition of sqrt()
{
  // ... algorithm
}
```

### Separate Compilation

#### Header Files

We place declarations that specify the interface to a piece of code we consider a module in a file with a name indicating its intended use — a **header file**:

```c++
// Vector.h

class Vector {
public:
  Vector(int s);
  double& operator[](int i);
  int size();
private:
  double* elem;
  int sz;
}
```

Users then `#include` that file to access the interface:

```c++
// user.cpp

#include "Vector.h"

double sqrt_sum(const Vector& v)
{
  // ...
}
```

To help the compiler ensure consistency, the `.cpp` file providing the implementation of `Vector` also includes the `.h` file:

```c++
// Vector.cpp

#include "Vector.h"

Vector::Vector(int s):elem{new double[s]}, sz{s}
{
}

// ...
```

The code in `use.cpp` and `Vector.cpp` shares the `Vector` interface from `Vector.h`, but the two files are otherwise independent and can be compiled separately.

A `.cpp` file compiled by itself (including the `h` files it `#include`s) is called a **translation unit**.

The use of header files and `#include` is a very old way of simulating modularity with significant disadvantages.

- **Compilation time**: If `header.h` is included in 101 translation units, its contents will be processed by the compiler 101 times.

- **Order dependencies**: The order in which headers are included can affect the meaning of the code (e.g., macros or conflicting declarations).

- **Inconsistency**: Slight differences in duplicated declarations across files can lead to crashes or subtle bugs.

- **Transitivity**: All code needed to express a declaration in a header must be included there, leading to code bloat as headers include other headers.

#### Modules

C++20 introduces `module` to provide proper modularity:

```c++
export module Vector;

export class Vector {
public:
  Vector(int s);
  double& operator[](int i);
  int size();
private:
  double* elem;
  int sz;
};

Vector::Vector(int s)
  :elem{new double[s]}, sz{s}
{
}

//...

export bool operator==(const Vector& v1, const Vector& v2)
{
  //...
}
```

This defines a `module` named `Vector`, which exports the `Vector` class, its member functions, and a non-member function defining opertor `==`.

You can `import` it where needed:

```c++
// user.cpp

import Vector;

double sqrt_sum(Vector& v)
{
  // ...
}
```
Advantages of modules:
- A module is compiled once, not in every translation unit that uses it.
- Modules can be imported in any order without affecting behavior.
- Importing something into a module does not implicitly expose it to users of the module — imports are not transitive.


Modules also simplify implementation hiding: only `export`ed declarations are accessible to users.

```c++
export module vector_printer;

import std;

export
template<typename T>
void print(std::vector<T>& v)
{
  std::cout << "{\n";
  for (const T& val: v)
    std::cout << " " <<  val << '\n';
  std::cout << '}';
}
```

By importing this module, we don't suddenly gain access to all of the standard library.

### Namespaces

C++ provides **namespaces** to group related declarations and prevent name collisions:

```c++
namespace My_code {
  class complex {
    // ...
  };

  complex sqrt(complex);
  // ...

  int main();
}

int My_code::main()
{
  complex z1 {1, 2};
  auto z2 = sqrt(z1);
  std::cout << '{' << z2.real() << ',' << z2.imag() << "}\n";
  // ...
}

int main()
{
  return My_code::main();
}
```

To access a name in another namespace, qualify it with the namespace name (e.g., `std::cout`, `My_code::main`).

If this becomes tedious, use a **using-declaration**:
```c++
void my_code(vector<int>& x, vector<int>& y)
{
  using std::swap;
  // ...
  swap(x, y);
  other::swap(x, y);
  // ...
}
```

To access all names in a namespace, use a **using-directive**: `using namespace std;`. 

```c++
export module vector_printer;

import std;
using namespace std;

export
template<typename T>
void print(vector<T>& v)
{
  cout << "{\n";
  for (const T& val: v)
    cout << " " << val << '\n';
  cout << '}';
}
```

This brings all names into the current scope but can cause name clashes, so use with caution.

### Function Arguments and Return Values

Function calls are the primary and recommended way to pass information between program parts. Key considerations:
- Is the object copied or shared?
- If shared, is it mutable?
- Is the object moved, leaving an "empty" object behind?

The deault behavior of both argument passing and value return is "make a copy", but many copies can implicitly be optimized to moves.

#### Argument Passing

We typically pass small objects by-value and larger ones by-reference. As a rule of thumb, an object is considered "small" if its size is not larger than "2 to 3 pointers". When passing large objects for performance reasons, but without the intent to modify them, we pass them by-`const`-reference.

```c++
int sum(const vector<int>& v)
{
  int s = 0;
  for (const int i: v)
    s += i;
  return s;
}

vector fib = {1, 2, 3, 5};
int x = sum(fib);
```

#### Value Return

Returning by value is the default and often efficient. Return by reference only when we want to grant a caller access to something that is not local to the function.

```c++
class Vector {
public:
  // ...
  double& operator[](int i){ return elem[i]; } // return reference to ith element
private:
  double* elem;
  // ...
};
```

The `i`th element of a `Vector` exists independently of the call of the subscript operator, so we can returen a reference to it.

Avoid returning references to local variables:
```c++
int& bad()
{
  int x;
  // ...
  return x; // bad: return a reference to the local variable x
}
```
All major C++ compilers will catch the obvious error in `bad()`.

For large return values, C++ uses **move semantics** or **copy elision**:

```c++
Matrix operator+(const Matrix& x, const Matrix& y)
{
  Matrix res;
  // ... for all res[i,j], res[i,j]=x[i,j]+y[i,j]
  return res;
}

Matrix m1, m2;
Matrix m3 = m1 + m2; // not copy
```

Instead of copying, we provide `Matrix` with a **move constructor** that can efficiently transfer its contents out of `operator+()`. Even if a move constructor isn't explicitly defined, the compiler can often optimize away the copy through **copy elision**, constructing the Matrix directly at its destination.

#### Return Type Deduction
C++ can deduce return types using `auto`:
```c++
auto mul(int i, double d) { return i*d; } 
// auto means "deduce the return type"
```

You can also use the **suffix return type** syntax:

```c++
auto mul(int i, double d) -> double { return i*d; }
// auto means "the return type will be mentioned later or be deduced"
```

#### Structured Binding 

The mechanism for giving local names to members of a class object.

```c++
struct Entry {
  string name;
  int value;
};

Entry read_entry(istream& is)
{
  string s;
  int i;
  is >> s >> i;
  return {s, i};
}

auto e = read_entry(cin);
cout << "{" << e.name << "," << e.value << "}\n";
```

Here, `{s,i}` is used to construct the `Entry` return value. Similarly, we can "unpack" an `Entry`'s members into local variables:

```c++
auto [n, v] = read_entry(is);
cout << "{" << n << "," << v << "}\n";
```