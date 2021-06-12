# Rust
## Table of Contents
1. [Overview](#overview)
2. [Coding Conventions](#popular-coding-conventions)
3. [Common Programming Concepts](#common-programming-concepts)
   * [Variables and Mutability](#variables-and-mutability)
   * [Data Types](#data-types)
   * [Functions](#functions)
   * [Control Flow](#control-flow)
4. [Ownership](#ownership)
   * [Scope](#scope)
   * [The Heap](#the-heap)
   * [Memory and Allocation](#memory-and-allocation)
5. [Sources](#sources)

## Overview
Rust is a multi-paradigm programming language designed for performance and safety

Syntactically, Rust is similar to C++, but is able to guarentee memory safety by using a *borrow checker* to validate
references. Thus, achieving memory safety without needing garbage collection or reference counting

The high-level ergonomics combined with the low-level control of Rust enables developers to write faster and more
reliable software

## Coding Conventions
Formatting:
   1. Lines should be at most 100 characters in length
   2. 4-space indents are preferred to tabs

Naming:
   1. `snake_case` should be used for function and variable names
   2. `SCREAMING_SNAKE_CASE` should be used for static and constant variable names
   3. `CamelCase` should be used for "type-level" constructs
   4. `T` should be used for type parameters
   5. Lifetime annotations should be short and lowercase, e.g., `'a`

Pull Requests:
   1. Isolate refactors into individual commits
   2. More commits is better, especially for large changes
   3. Individual commits should be correctly formatted even though the final commit is the only one that matters
   4. **No** merge commits. Merge conflicts should be resolved via rebases
   5. Individual commits do not have to compile, but it's really nice if they do. Bisecting is performed at a PR level
      instead of at a commit level

## Common Programming Concepts
### Variables and Mutability
By default, variables in Rust are *immutable*. You have the option to explicitely make variables mutable with
the `mut` keyword, but developers are encouraged to favor immutability wherever possible in order to take advantage
of Rust's safety and easy concurrency

```
fn main() {
   let x = 5;        // Declaring an immutable variable `x`
   let mut y = 5     // Declaring a mutable variable `y`
}
```

#### Immutable Variables vs Constants
Now, being unable to change the value of a variable probably sounds familiar, because other languages accomplish
this by using *constants*. Just like immutable variables, constants are values that are bound to a name and are not
allowed to change. However, there are a few differences between *constants* and *variables* in Rust:

   1. Constants aren't just immutable by default; they are **always** immutable
   2. Constants are declared using the `const` keyword instead of `let`
      * This means the data type must be specified at the time of declaration
   3. Constants can be declared in any scope, including the global scope
   4. Constants cannot store values that can only be computed at runtime
   5. Constants are valid for the entire lifetime of the program within
      the scope they were declared in

```
const KIBIBYTE: u32 = 1024;   // Declaring a global constant `KIBIBYTE` to store a `u32` with a value of `1024`

fn main() {
   let x = 5;                 // Declaring a local immutable variable `x` with a value of `5`
}
```

#### Shadowing Immutable Variables
In Rust, you can declare a new variable with the same name as the previous variable by *shadowing* the initial
variable. Shadowing is accomplished by repeating the use of the `let` keyword

```
fn main() {
   // Bind `x` to a value of `5`
   let x = 5;

   // "Shadow" `x` to make the value now `6`
   let x = x + 1;

   // "Shadow" `x` again to make the value now `12`
   let x = x * 2;
}
```

Shadowing is different from marking a variable as `mut` because shadowing effectively creates a new variable
entirely with the `let` keyword. So, you are not modifying the value of `x` so much as you are initializing
a completely new variable with the same name `x` that will hold a value computed by *shadowing* the original
value of `x`

Shadowing spares you from having to declare multiple variables of different names in order to perform type
conversions. For example:

```
fn main() {
   let spaces = "     ";            // `spaces` is a new string-type variable
   let spaces = spaces.len();       // `spaces` is a new integer-type variable
}
```

You do not have to declare separate variables such as `spaces_str` and `spaces_int` to hold these values
if you only care about the latter value. You can just reuse the simpler name `spaces`

### Data Types
Rust is a *statically typed* language, meaning that it must know the types of all variables at compile time. Variable
types can be broken down into two categories - **Scalar Types** and **Compound Types**; *scalar* types represent a single
value and *compound* types group multiple values into a single type

#### Scalar Types
Rust has four primary scalar types: integers, floats, booleans, and characters

| Type | Length | Keyword |
| :---: | :---: | :---: |
| Integer | 8-bit | `i8` or `u8` |
| Integer | 16-bit | `i16` or `u16` |
| Integer | 32-bit | `i32` or `u32` |
| Integer | 64-bit | `i64` or `u64` |
| Integer | 128-bit | `i128` or `u128` |
| Integer |arch-Native | `isize` or `usize` |
| Float | 32-bit | `f32` |
| Float | 64-bit | `f64` |
| Boolean | 8-bit | `bool` |
| Character | 32-bit | `char` |

#### Compound Types
Rust has two primary compound types: tuples and arrays

### Functions
Just like C or C++, Functions are pervasive in Rust. New functions can be declared using the `fn` keyword

### Control Flow
Control flow is simply deciding whether or not to run some code depending on a certain condition. The most
common control flow constructs in Rust are `if` expressions and loops

## Ownership
The central feature of Rust is **ownership**

All programs have to be able to manage the way they use memory at runtime. Some languages, such as Java, use a
garbage collector. Other languages, such as C, require the programmer to explicitly allocate and free the memory

Rust offers a unique third approach to memory management; Memory is managed through a system of ownership with a
set of rules that the compiler checks at compile time

The 'Rules of Ownership' in Rust are as follows:

   - Each value in Rust has a variable that is called the **owner**
   - There can only be **one** owner at a time
   - When the owner goes out of scope the value is **dropped**

### Scope
A scope is the range within a program for which an item is considered *valid*. Variables are
valid from the point at which they are declared within a scope until the scope ends

```
fn main() {
   let s = "hello";

   // --snip--

} // scope ends
```

Here, the variable `s` is declared as a string literal that stores the static value `hello`.
`s` will remain *valid* until the scope ends, which in this example is at the end of `main()`.
In other words, you cannot reference `s` outside the body of `main()`

This isn't really any different from how other programming languages handle scopes and variables.
Nobody should expect to be able to access `s` outside of the `main()` scope here, but keep in
mind that `s` was **statically** defined in this example and thus placed on the program stack.
Both the *contents* and the *size* of `s` are known by the compiler at compile time

So, what changes if we use **dynamically** allocated memory for `s` instead? That is, use memory
from the heap instead of the stack

### The Heap
In the case of static memory, the data held by a variable is hardcoded directly into the final
executable. This can be done because both the contents and the size of the variable are known by
the compiler at compile time, so the compiler is able to allocate the necessary space required on
the program stack to store the variable

However, if the size of the data stored inside of a variable can *change* at runtime, the data cannot
be put on the stack because the compiler has no way of knowing how much space might be required for it. This is why
we need *dynamic memory allocation*

The following conditions must be met in order to support a *mutable* and *growable* piece of data on the heap:

   1. The memory must be requested from the memory allocator at runtime
   2. The memory must be given back to the allocator at runtime when you are finished using it

These two conditions largely boil down to the same universal point in programming: You must pair **exactly** one `allocate()` with
**exactly** one `free()`

Rust takes a unique approach to ensure this: *Memory is automatically freed once the variable that owns it goes out of scope*

#### The `String` Type
In order to illustrate how dynamic memory allocations are handled in Rust, the following examples of ownership and scope
focus on a common data type found in Rust; the `String` type

   - The `String` type is dynamic and allocated on the heap
   - Memory for a `String` can be allocated from a string literal using the `from()` function
   - Unlike string literals, a `String` *can* be mutated (That is, be declared with the `mut` keyword)

An example of memory allocation in Rust using `String` is as follows

```
let s = String::from("hello");
```

This creates a mutable `String` with the initial contents "hello" and names it `s`. It is important to note that `s` is **not**
a pointer. `s` is just a mutable string object that lives on the heap. In fact, a `String` is made up of three distinct parts:

   1. A pointer, which is a reference to the buffer that holds the data
   2. A length, which is the number of bytes *currently* stored in the buffer
   3. A capacity, which is the *total* size of the buffer in bytes

These elements are stored on the program stack. Note that the length will always be less than or equal to the capacity. If a
`String` has enough capacity, adding elements to it will not allocate any additional memory for those elements since it already exists

### Memory and Allocation
Now, let's take the example from before with a string literal and re-write it to use a `String` type instead. We will also add
an additional scope within `main()` to further illustrate the lifetime of the memory we allocate and how Rust knows exactly when
to free that memory

```
func main() {
   // --snip--

   {
      // Allocate memory for `s`
      let s = String::from("hello");

      // do stuff with `s`

   } // end of scope; `drop()` is automatically called and `s` is no longer valid
}
```

As soon as the scope ends a special function `drop()` is automatically called and frees the memory. This may seem simple now,
but this pattern has a *profound* effect on the way that Rust code is written

What if we want to allow multiple variables to use memory we have allocated on the heap?

```
let s1 = String::from("hello");
let s2 = s1;
```

When we assign `s1` to `s2` we copy the `String` data from the stack into `s2`, **not** the actual data on the heap that the `String`
refers to

## Sources
   - [The Rust Programming Language](https://doc.rust-lang.org/stable/book)
   - [Rustc](https://doc.rust-lang.org/rustc)
   - [A Guide to Porting C/C++ to Rust](https://locka99.gitbooks.io/a-guide-to-porting-c-to-rust/content/)
