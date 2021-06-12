# Rust
## Table of Contents
1. [Overview](#overview)
2. [Coding Conventions](#popular-coding-conventions)
3. [Common Programming Concepts](#common-programming-concepts)
   * [Variables and Mutability](#variables-and-mutability)
   * [Data Types](#data-types)
   * [Functions](#functions)
   * [Control Flow](#control-flow)
4. [Sources](#sources)

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

## Sources
   - [The Rust Programming Language](https://doc.rust-lang.org/stable/book)
   - [Rustc](https://doc.rust-lang.org/rustc)
