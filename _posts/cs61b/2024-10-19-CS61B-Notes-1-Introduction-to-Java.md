---
layout: post
title: CS 61B (Sp21) Notes 1, Introduction to Java
categories: course
tags: [data structures, java]
author: <na_id>
math: true
media_subpath: /assets/img/cs61b/
toc:
  sidebar: right
---

## 1. Intro, Hello World Java
### 1.1 Java and Object Orientation
Java is an **object oriented language** with strict requirements:
- Every Java file must contain a **class** declaration*.
- **All code** lives inside a class*, even helper functions, global constants, etc.
- To run a Java program, you typically define a **main method** using `public static void main(String[] args)`

*: This is not completely true, e.g. we can also declare "**interfaces**" in `.java` files that may contain code.

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello world!");
    }
}
```

### 1.2 Java and Static Typing
Java is **statically typed**.
- All variables, parameters, and methods must have a **declared type**.
- That type can **never change**.
- Expressions also have a type, e.g. `larger(5, 10) + 3` has type int.
- The compiler checks that all the types in your program are compatible **before** the program ever runs.

### 1.3 Compilation
![compilation](1/compilation.png){: w="600"}

```sh
$ javac HelloWorld.java
$ java HelloWorld
Hello World! 
```

Why make a class file at all?
- `.class` file has been type checked. Distributed code is **safer**.
- `.class` files are 'simpler' for machine to execute. Distributed code is **faster**.

## 2. Defining and Using Classes
### 2.1 Classes
- Classes can contain not just functions (a.k.a. **methods**), but also data (a.k.a. **variables**).
- Classes can be instantiated as **objects**.

Static vs. Non-Static Members
- A variable or method defined in a class is also called a **member** of that class. 
- A class may have a mix of **static** and **non-static** members.
  - Static members are accessed using class name.
  - Non-static members cannot be invoked using class name.
  - Static methods must access instance variables via a specific instance.
    
<!-- ## 2.2 Lists
> A **list** is an ordered sequence of objects.

```java
import java.util.List;
import java.util.ArrayList;

public class ListDemo {
    public static void main(String[] args) {
        List<String> L = new ArrayList<>();
        L.add("a"); 
        L.add("b"); 
        String x = L.get(0);
    }
}
```

- **List** in Java is **Abstract Data Type (ADT)**
- Each implementation, e.g. **LinkedList**, is known as a **Concrete Implementation**.
  - Code for different types of list may be radically different.
  - All concrete implementations have at least [the operations guaranteed by every List](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/List.html).

![list](2/list.png){: w="400" h="200" } -->

<!-- ## 2.3 Arrays
> A **array** is a restricted version of the list ADT.
- Size that must be declared at the time the array is created.
- Size cannot change.
- No methods.

```java
public class ArrayDemo {
    public static void main(String[] args) {
        String[] x = new String[5]; // size 5
        x[0] = "a";
        x[1] = "b";
        System.out.println(x[0]);
    }
} 
```
-->

<!-- ## 2.4 Maps
> A **map** is collection of key-value pairs. Each key is guaranteed to be unique.

```java
import java.util.Map;
import java.util.TreeMap;
 
public class MapDemo {
    public static void main(String[] args) {
        Map<String, String> L = new TreeMap<>();
        L.put("dog", "woof");
        L.put("cat", "meow");
        String sound = L.get("cat");
  }
}
```

The most common Map implementations are **TreeMap** and **HashMap**. 
- Python uses an ArrayList (without telling you).
- Python uses a HashMap (without telling you). -->

## 3. Testing
[placeholder]

## 4. References, Recursion, and Lists
### 4.1 Primitive Types
#### 4.1.1 Bits
Information is stored in memory as a sequence of ones and zeros. Each Java type has a different way to interpret the bits:
- 8 primitive types in Java: byte, short, int, long, float, double, boolean, char

#### 4.1.2 Declaring a Variable (Simplified)
When you declare a variable of a certain type in Java:
- Your computer sets aside exactly enough bits to hold a thing of that type.
- Java creates an internal table that maps each **variable name** to a **location**.

![declaring](4/declaring.png){: w="500"}

#### 4.1.3 The Golden Rule of Equals (GRoE)
Given variables `y` and `x`, `y = x` copies all the bits from `x` into `y`.

### 4.2 Reference Types
Everything else, including arrays, is a reference type.

#### 4.2.1 Class Instantiations
When we instantiate an Object:
- Java first allocates a box of bits for each instance variable of the class and fills them with a default value (e.g. 0, null).
- The constructor then usually fills every such box with some other value.

![class instantiations](4/class_instantiations.png){: w="600"}

Can think of `new` as returning the **address** of the newly created object.
- Addresses in Java are 64 bits.
- Example: If object is created in memory location `2384723423`, then new returns `2384723423`.

![new](4/new.png){: w="600"}

#### 4.2.2 Reference Type Variable Declarations
When we declare a variable of any reference type:
- Java allocates exactly a box of size 64 bits, no matter what type of object.
- These bits can be either set to:
    - Null (all zeros).
    - The 64 bit address of a specific instance of that class (returned by `new`).

![reference type declaration](4/reference_type_declaration.png){: w="300"}

Reference types obey the Golden Rule of Equals: Just as with primitive types, the equals sign **copies the bits**.

![GRoE](4/groe.png){: w="300"}

### 4.3 Parameter Passing
Passing parameters obeys the same rule: Simply **copy the bits** to the new scope.

### 4.4 Instantiation of Arrays
Arrays are also Objects.
```java
int[] a = new int[]{0, 1, 2, 95, 4};
```
- Creates a 64 bit box for storing an int array address. (declaration)
- Creates a new Object, in this case an int array. (instantiation)
- Puts the address of this new Object into the 64 bit box named `a`. (assignment)

![array](4/array.png){: w="400"}

## 5. SLLists, Nested Classes, Sentinel Nodes
<!-- ## 4.1 SLLists -->

```java
public class SLList {
  private static class IntNode {
    public int item;
    public IntNode next;
    public IntNode(int i, IntNode n) {
      item = i;
      next = n;
    }
  }

  private IntNode sentinel;
  private int size;
  
  public SLList() {
    sentinel = new IntNode(-1, null);
    size = 0;
  }

  public void addFirst(int x) {
    sentinel.next = new IntNode(x, sentinel.next);
    size += 1;
  }
  
  public int getFirst() {
    return sentinel.next.item;
  }

  public void addLast(int x) {
    IntNode p = sentinel;
    while (p.next != null) {
      p = p.next;
    }
    p.next = new IntNode(x, null);
    size += 1;
  }

  public int size() {
    return size;
  }
}
```

![SLList](5/sllist.png){: w="500"}

## 6. DLLists, Arrays
### 6.1 Doubly Linked Lists
One downside of SLLists is that **inserting at the back** of an SLList is much slower than the front.
We can have a `last` pointer to make `add` and `get` operations at the back fast. However, this is insufficient to support fast `remove`. We may want to add backwards links from every node.
> Reverse pointers allow all operations (`add`, `get`, `remove`) to be fast.

![Naive DLList](6/naive_dllist.png){: w="500"}

This approach has an annoying special case: `last` sometimes points at the sentinel, and sometimes points at a 'real' node. 

- Solution 1: have two sentinels
- Solution 2: circular sentinel

![Circular Sentinel](6/circular_sentinel.png){: w="500"}

### 6.2 Arrays
Arrays consist of:
- A **fixed** integer length (cannot change!)
- A sequence of `N` memory boxes where `N=length`, such that:
    - All of the boxes hold the **same type** of value (and have same # of bits).
    - The boxes are numbered `0` through `length-1`.

A **2D Array** is an array of array addresses.
![2D Array](6/2d_array.png){: w="600"}


## 7. ALists, Resizing
### 7.1 Naive Array Lists
Retrieval from any position of an array is very fast. How to build an array version of a list?

AList **Invariants**:
- The position of the next item to be inserted is always `size`.
- `size` is always the number of items in the AList.
- The last item in the list is always in position `size - 1`.

```java
public class AList {
  private int[] items;    
  private int size;
 
  public AList() {
    items = new int[100];  size = 0;
  }
 
  public void addLast(int x) {
    items[size] = x;
    size += 1;
  }
 
  public int getLast() {
    return items[size - 1];
  }
 
  public int get(int i) {
    return items[i];
  }

  public int size() {
    return size;
  }
}
```

### 7.2 Resizing Arrays
When the array gets too full (i.e., `size==items.length`), just make a new array:
![Resizing](7/resizing.png){: w="500"}

```java
public void addLast(int x) {
  if (size == items.length) {
    // resize(size + 1); // Unusably bad.
    resize(size * RFACTOR); // Great performance. 
    // This is how the Python list is implemented.
  }
  items[size] = x;
  size += 1;
}

private void resize(int capacity) {
  int[] a = new int[capacity];
  System.arraycopy(items, 0, a, 0, size);
  items = a;
}
```

An AList should not only be efficient in time, but also efficient in space.
- Define the "usage ratio" `R = size / items.length`;
- Half array size when `R < 0.25`.

<!-- > **Obscurantism**
The user of a class does not and should not know how it works.
{: .prompt-info} -->

## 8. Inheritance, Implements
<!-- ## 8.1 Method Overloading
Java allows multiple methods with same name, but different parameters.

```java
public static String longest(AList<String> list) {
    ...
}

public static String longest(SLList<String> list) {
    ...
}
``` -->
### 8.1 Interface Inheritance
Our `AList` and `SLList` classes have the following methods (exact same method signatures for both classes). `SLLists` and `ALists` are both clearly some kind of "list". 
`List` is a **hypernym** of `SLList` and `AList`.

![list_methods](8/list_methods.png){: w="500"}

#### 8.1.1 Interface and Implements Keywords
Use the new keyword `interface` instead of `class` to define a `List61B`.
Interface is a specification of **what** a List is able to do, not how to do it.

![list_interface](8/list_interface.png){: w="500"}

Use the `implements` keyword to tell the Java compiler that SLList and AList are **hyponyms** of List61B.

![Implements](8/implements.png){: w="450"}

#### 8.1.2 Overriding vs. Overloading
**Method signature** is the <u>method name</u> and the <u>number and type of its parameters</u>.
- **overriding**: If a "subclass" has a method with the **exact same signature** as in the "superclass", we say the subclass overrides the method.
- **overloading**: Methods with the **same name but different signatures** are overloaded.

![Overriding Overloading](8/over.png){: w="500"}

We mark overriding methods with the `@Override` annotation.

```java
public class AList<Item> implements List61B<Item> {
   ...

   @Override
   public void addLast(Item x) { 
      ... 
```
Why use `@Override`?
- Protects against typos. The code won't compile if it is not actually an overriding method. 
- Reminds programmer that method definition came from somewhere higher up in the inheritance hierarchy.

Even if you don't write `@Override`, subclass **still overrides** the method.

#### 8.1.3 Interface Inheritance
Specifying the capabilities of a subclass using the `implements` keyword is known as **interface inheritance**.
Subclasses **must** override all of these methods. Will fail to compile otherwise. Interface inheritance is a powerful tool for **generalizing** code.

If `X` is a superclass of `Y`, then memory boxes for `X` may contain `Y`.
An `AList` is-a `List`. Therefore `List` variables can hold `ALList` addresses.

![Copy](8/copy.png){: w="500"}

### 8.2 Implementation Inheritance: Default Methods
- **Interface inheritance**: Subclass inherits signatures, but NOT implementation.
- **Implementation inheritance**: Subclasses can inherit signatures AND implementation.
  - Use the `default` keyword to specify a method that subclasses should inherit from an interface.
    ```java
    public interface List61B<Item> {
      ...
      default public void print() {
        for (int i = 0; i < size(); i += 1) {
          System.out.print(get(i) + " ");
        }
        System.out.println();
      }
    }
    ```
  - If you don't like a default method, you can **override** it.
    ```java
    public interface SLList<Item> implements {
      @Override
      public void print() {
        for (Node p = sentinel.next; p != null; p = p.next) {
          System.out.print(p.item + " ");
        }
        System.out.println();
      }
    }
    ```

### 8.3 Static and Dynamic Type
- Compile-time type (static type): the type specified at **declaration**. Never changes.
- Run-time type (dynamic type): the type specified at **instantiation** (e.g. when using `new`).

![Types](8/types.png){: w="520"}

Suppose we call a method of an object using a variable with compile-time type `X` and run-time type `Y`.
Then if `Y` **overrides** the method, `Y`'s method is used instead.

## 9. Extends, Casting, Higher Order Functions
### 9.1 The Extends Keyword
If you want one class to be a hyponym of another **class** (instead of an interface), you use `extends`.

#### 9.1.1 RotatingSLList
We'd like to build `RotatingSLList` that can perform any `SLList` operation as well as `rotateRight()`: Moves back item the front.

![rotatinglist](9/rotatinglist.png){: w="520"}

Because of `extends`, `RotatingSLList` inherits all members of `SLList`:
- All instance and static variables.
- All methods.
- All nested classes.
Constructors are not inherited.


#### 9.1.2 VengefulSLList
We want to build an SLList that:
  - Remembers all Items that have been destroyed by `removeLast`.
  - Has an additional method `printLostItems()`, which prints all deleted items.

![vengefulsllist](9/vengefulsllist.png){: w="520"}

#### 9.1.3 Constructor Behavior 
**Constructors** are not inherited. However, the rules of Java say that all constructors must start with a call to one of the super class's constructors.
- You can explicitly call the constructor with the keyword `super`.
- If you don’t explicitly call the constructor, Java will automatically do it for you.
- If you want to use a super constructor other than the no-argument constructor, can give parameters to super.

![super_1](9/super_1.png){: w="520"}

![super_2](9/super_2.png){: w="520"}


### 9.2 Encapsulation
**Module**: A set of methods that work together as a whole to perform some task or set of related tasks. A module is said to be **encapsulated** if its implementation is completely hidden, and it can be accessed only through a documented interface.

Implementation inheritance (e.g. extends) breaks encapsulation.

![bark](9/bark.png){: w="520"}

### 9.3 Type Checking and Casting

#### 9.3.1 Compile-Time Type Checking
- Compiler allows **method calls** based on compile-time type of variable.
  - `sl`’s runtime type: `VengefulSLList`. But cannot call `printLostItems`. 
- Compiler also allows **assignments** based on compile-time types.
  - Even though `sl`’s runtime-type is `VengefulSLList`, cannot assign to `vsl2`.

![type_checking](9/type_checking.png){: w="550"}

#### 9.3.2 Compile-Time Types and Expressions
Expressions have compile-time types:
- An expression using the `new` keyword has the specified compile-time type.

![expression_1](9/expression_1.png){: w="500"}

- Method calls have compile-time type equal to their declared type.

![expression_2](9/expression_2.png){: w="520"}


#### 9.3.3 Casting
Java has a special syntax for specifying the compile-time type of any expression: Put **desired type in parenthesis** before the expression.
- Compile-time type Dog: `maxDog(frank, frankJr);`
- Compile-time type Poodle: `(Poodle) maxDog(frank, frankJr);`

![casting_1](9/casting_1.png){: w="520"}

Casting is a powerful but dangerous tool. Tells Java to treat an expression as having a different compile-time type.

![casting_2](9/casting_2.png){: w="520"}
_If we run the code above, we get a ClassCastException at runtime._

### 9.4 Higher Order Functions
**Higher Order Function**: A function that treats another function as data. e.g. takes a function as input.

#### 9.4.1 Old School (Java 7 and earlier)
Fundamental issue: Memory boxes (variables) cannot contain pointers to functions. Can use an interface instead. Note: The old way is still widely used, e.g. Comparators.

![hof_1](9/hof_1.png){: w="630"}

#### 9.4.2 Java 8 or Later
In Java 8, new types were introduced: now can can hold references to methods.

![hof_2](9/hof_2.png){: w="630"}

## 10. Subtype Polymorphism vs. HoFs
### 10.1 Subtype Polymorphism (多态)
The biggest idea of the last couple of lectures: Subtype Polymorphism. 
Polymorphism: "providing a single interface to entities of different types".

![compare](10/compare.png){: w="500"}

### 10.2 Comparable
The built-in `Comparable` interface is already defined and used by tons of libraries.

![comparable](10/comparable.png){: w="300"}

![dog_comparable](10/dog_comparable.png){: w="500"}

### 10.3 Comparators
Suppose we want to write a program that prints a string representation of the larger of two objects according to some specific comparison function.

![comparator_1](10/comparator_1.png){: w="500"}

![dog_comparator](10/dog_comparator.png){: w="500"}

## 11. Exceptions, Iterators, Object Methods
### 11.1 Exceptions
When something goes really wrong, break the normal flow of control.

![exception](11/exception.png){: w="500"}

### 11.2 Iteration
Java allows us to iterate through Lists and Sets using a convenient shorthand syntax sometimes called the "foreach" or "enhanced for" loop.

![foreach](11/foreach.png){: w="400"}

How Iteration Really Works?

![iteration_1](11/iteration_1.png){: w="500"}

To support ugly iteration:
- Add an `iterator()` method to `ArraySet` that returns an `Iterator<T>`.
- `The Iterator<T>` that we return should have a useful `hasNext()` and `next()` method.

![iteration_2](11/iteration_2.png){: w="500"}

To support the enhanced for loop, we need to make `ArraySet` implement the `Iterable` interface.

![iteration_3](11/iteration_3.png){: w="500"}

### 11.3 Object Methods
#### 11.3.1 toString()
The `toString()` method provides a string representation of an object. The [implementation of `toString()` in Object](https://github.com/AdoptOpenJDK/openjdk-jdk11/blob/999dbd4192d0f819cb5224f26e9e7fa75ca6f289/src/java.base/share/classes/java/lang/Object.java#L245) is the the name of the class, then an @ sign, then the memory location of the object.

![to_string_1](11/to_string_1.png){: w="530"}

One approach to implement `toString` for `ArraySet`. This code is slow Adding even a single character to a string creates an entirely new string. It’s because Strings are "immutable".

![to_string_2](11/to_string_2.png){: w="530"}

Much faster approach is shown below. Append operation for a `StringBuilder` is fast. 

![to_string_3](11/to_string_3.png){: w="530"}

#### 11.3.2 Equals
`==` compares the bits. For references, `==` means "referencing the same object." [Default implementation](https://github.com/AdoptOpenJDK/openjdk-jdk11/blob/999dbd4192d0f819cb5224f26e9e7fa75ca6f289/src/java.base/share/classes/java/lang/Object.java#L157) of `.equals` uses `==`.

Implement `equals` for `ArraySet`.

![equal](11/equal.png){: w="530"}