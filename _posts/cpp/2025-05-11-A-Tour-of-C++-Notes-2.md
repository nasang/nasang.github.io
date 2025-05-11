---
layout: post
title: A Tour of C++ | Notes 2 Abstraction (Ch 5 - Ch 8)
categories: book
tags: [C++, language]
description: >
toc:
  sidebar: right
media_subpath: /assets/img/cs61b/
thumbnail: /assets/img/cpp/a_tour_of_c++.jpg
---

## 5 Classes

A class is a user-defined type designed to represent an entity within a program. The C++ language provides fundamental support for three important kinds of classes:
- Concrete classes
- Abstract classes
- Classes in class hierarchies

An astounding number of useful classes fall into one of these categories. Many others can be seen as variations of these or are constructed using combinations of their techniques.

### Concrete Types
The basic idea of **concrete classes** is that they behave "just like built-in types".

#### An Arithmetic Type
A classic example of a user-defined arithmetic type is the `complex` class:

```c++
class complex {
    double re, im; // representation: two doubles
public:
    complex(double r, double i) : re{r}, im{i} {}
    complex(double r) : re{r}, im{0} {}
    complex() : re{0}, im{0} {}

    double real() const { return re; }
    void real(double d) { re = d; }
    double imag() const { return im; }
    void imag(double d) { im = d; }

    complex& operator+=(complex z) {
        re += z.re;
        im += z.im;
        return *this;
    }

    // ...
};
```
- A constructor that can be called without arguments is known as a **default constructor**.
- The `const` qualifier on member functions like `real()` and `imag()` indicates that these functions do not modify the object.

#### A Container

A **container** is an object that holds a collection of elements. The class `Vector` is a container because objects of this type store such a collection:

```c++
class Vector {
public:
    Vector(int s) : elem{new double[s]}, sz{s} {
        for (int i = 0; i != s; ++i)
            elem[i] = 0;
    }

    ~Vector() { delete[] elem; }

    double& operator[](int i);
    int size() const;
private:
    double* elem;
    int sz;
};
```

- A **destructor** ensures that resources allocated in the constructor are properly released. It is declared using the complement operator ~ followed by the class name.
- `delete` frees a single object; `delete[]` frees an array.

This pattern of acquiring resources in the constructor and releasing them in the destructor is known as **Resource Acquisition Is Initialization (RAII)**. It eliminates the need for "naked" `new` or `delete` operations, making code safer and more maintainable.

Objects of class `Vector` obey the same rules for naming, scoping, memory allocation, and lifetime as built-in types:

```c++
Vector gv(10); // global variable; gv is destroyed at the end of the program
Vector* gp = new Vector(100); // Vector on free store; never implicitly destroyed
```

```c++
void fct(int n)
{
    Vector v(n);
    {
        Vector v2(2 * n);
    } // v2 is destroyed here
} // v is destroyed here
```

#### Initializing Containers

```c++
class Vector {
public:
    Vector();
    Vector(std::initializer_list<double>);

    void push_back(double);
    // ...
};
```

The `std::initializer_list` used here is a standard library type recognized by the compiler. When we write `{1, 2, 3}`, the compiler creates an `initializer_list` object to pass to the constructor.

```c++
Vector::Vector(std::initializer_list<double> lst)
    : elem{new double[lst.size()]}, sz{static_cast<int>(lst.size())}
{
    copy(lst.begin(), lst.end(), elem);
}
```

Since the standard library typically uses `unsigned` types for sizes and indices, we use `static_cast` to convert the `size_t` to `int`. Other casts include `reinterpret_cast`, `bit_cast` (to treat memory as bytes), and `const_cast` (to remove `const` qualifiers).

### Abstract Types

Types like `complex` and `Vector` are called concrete types because their implementation details (i.e., their representation) are part of the class definition. By contrast, an **abstract type** completely hides its implementation from the user.

```c++
class Container {
public:
    virtual double& operator[](int) = 0;
    virtual int size() const = 0;
    virtual ~Container() {}
};
```

- The `virtual` keyword means the function may be redefined in a derived class. Such functions are called **virtual functions**.

- The `=0` syntax defines a **pure virtual function**, meaning any derived class _must_ override it.

A class with a pure virtual function is called an **abstract class**.

The `Container` can be used like this:

```c++
void use(Container& c)
{
    const int sz = c.size();
    for (int i = 0; i != sz; ++i)
        cout << c[i] << '\n';
}
```

- As is common, the abstract class `Container` does not have a constructor, since it doesn't store data.
- It **does** have a virtual destructor so that derived classes can be correctly cleaned up through base class pointers.


To use `Container`, we must define a concrete subclass that implements its interface:

```c++
class Vector_container : public Container {
public:
    Vector_container(int s) : v(s) {}
    ~Vector_container() {}

    double& operator[](int i) override { return v[i]; }
    int size() const override { return v.size(); }
private:
    Vector v;
};
```

Here, `: public` means that `Vector_container` **is-a** `Container`. The member destructor `~Vector()` is implicitly invoked by the destructor of `Vector_container`.

```c++
void g()
{
    Vector_container vc(10);
    // ...
    use(vc);
}
```

Since `use()` only depends on the `Container` interface, it works equally well for any other class that implements that interface:

```c++
class List_container : public Container {
public:
    // ...
};

void h()
{
    List_container lc = {1, 2, 3};
    use(lc);
}
```


As a result, `use(Container&)` doesn’t need to be recompiled even if implementations of derived classes change.

### Virtual Functions

When `h()` calls `use()`, `List_container`'s `operator[]()` must be called. When `g()` calls `use()`, `Vector_container`'s `operator[]()` must be called. To achive this resolution, a `Container` object must contain infomation to allow it to select the right function to call at run time, - the **virtual function table**, or simply the `vtbl`.

### Class Hierarchies

## 6 Essential Operations