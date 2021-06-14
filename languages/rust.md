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
   * [References and Borrowing](#references-and-borrowing)
   * [Slices](#slices)
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

This creates a mutable `String` with the initial contents `hello` and names it `s`. It is important to note that `s` is not *just*
a pointer; `s` is actually made up of three distinct parts:

   1. A pointer, which is a reference to the buffer that holds the data
   2. A length, which is the number of bytes *currently* stored in the buffer
   3. A capacity, which is the *total* size of the buffer, in bytes

These parts are stored on the program stack inside of the variable `s`. Note that the length will always be less than or
equal to the capacity. If a `String` has enough capacity, adding elements to it will not allocate any additional memory
for those elements since they don't need it

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

#### Move
What if we want to allow multiple variables to use memory we have allocated on the heap?

```
let s1 = String::from("hello");
let s2 = s1;
```

When we assign `s1` to `s2` we copy the `String` data from the *stack* into `s2`, **not** the actual data on the heap. This makes
sense, because copying heap data into new variables would be extremely expensive in terms of runtime performance

![Reference Copy in Rust](/images/rust/reference_copy.svg)

But if Rust automatically the calls `drop()` function to clean up memory at the end of the scope, wouldn't this lead to a
*double free* error now that two variables are referencing the same memory in the same scope? It absolutely would, which
is why there is one important thing to remember from earlier; the second rule of ownership: *There can only be one owner at a time*

To ensure memory safety here, Rust considers `s1` to **no longer be valid** so it does not need to free anything
when `s1` goes out of scope. Only `s2` is freed when the scope ends, so any references made to `s1` *after* it has
been copied to `s2` will cause a compiler error

```
let s1 = String::from("hello");
let s2 = s1;

println("{}, world!", s1);
```

Rust prevents you from using the invalidated reference with the following compiler error

```
$ cargo build
Compiling playground v0.0.1 (/playground)
error[E0382]: borrow of moved value: `s1`
--> src/main.rs:5:28
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value borrowed here after move

error: aborting due to previous error

For more information about this error, try `rustc --explain E0382`.
error: could not compile `ownership`

To learn more, run the command again with --verbose.
```

If you're familiar with the terms *shallow copy* and *deep copy*, the concept of copying only the stack data
without the heap data probably sounds like making a shallow copy. However, because Rust also *invalidates* the
first variable, this is actually known as a *move*. In this example, we would say that `s1` was *moved* into
`s2`

#### Clone
If you *do* want to deeply copy the heap data of the `String` and not just the stack data, you can use the
common method called `clone`

```
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

This works perfectly fine. There is no risk of a double free error here because both `s1` and `s2` are pointing
to different locations in the heap. Each of these references will be freed as normal when their scope comes to
an end

#### Ownership and Functions
Passing values to functions is simliar to assigning a value to a new variable. Doing so will *move* or *copy* the
data, just as the variable assignment does

```
fn main() {
   let s = String::from("hello");

   takes_ownership(s);

   let x = 5;

   makes_copy(5);
}

fn takes_ownership(some_string: String) {
   println!("{}", some_string);
}

fn makes_copy(some_integer: i32) {
   println!("{}", some_integer);
}
```

Rust will throw a compiler error in the code above if you try to reference `s` *after* the call to `takes_ownership()`
because `s` is no longer valid in the scope of `main()` - it was *moved* into the scope of `take_ownership()`, where
it had its memory freed when the new scope ended

There is nothing special about `x` in this example because `x` has its memory allocated on the stack and not the heap. Passing
it to the function `makes_copy()` performs a simple `copy()`, so `x` is still valid in the `main()` scope after the call to
`makes_copy()`

#### Transferring Ownership with Returns
This works the other way around too, that is, return values can transfer the ownership of references to the caller of their function

```
fn main() {
   let s1 = gives_ownership();

   let s2 = String::from("hello");

   let s3 = takes_and_gives_back(s1);
}

fn gives_ownership() -> String {
   let some_string = String::from("hello");

   some_string
}

fn takes_and_gives_back(some_string: String) -> {
   some_string
}
```

What happens to `s1`, `s2`, and `s3` when the scope of `main()` ends? Is there any point inside of the `main()` scope
where one or more of these references is considered invalid? When exactly is `drop()` called for each of these references?

   - `s1` - Valid until the end of `main()` where it goes out of scope and is dropped
   - `s2` - Valid until ownership is moved to `takes_and_gives_back()`. `s2` does not get dropped
            because the reference is invalidated through the indirect `move()` to `s3`
   - `s3` - Valid until the end of `main()` where it goes out of scope and is dropped

The key thing to take away from this is *when a variable that references heap data goes out of scope, the value will be cleaned
up by `drop()` unless ownership of the data was moved to be owned by another variable*

### References and Borrowing
If you want to let a function use a value but not take ownership of it, you need to pass it by *reference*
which is done by using the `&` operator

```
fn main() {
   let s1 = String::from("hello");

   let len = calculate_length(&s1);

   println!("The length of '{}' is {}!", s1, len);
}

fn calculate_length(s: &String) -> usize {
   s.len()
}
```

The `&` operator are *references* and allow you to refer to a value **without** taking ownership of it. So, whatever
scope gets the reference can **not** call `drop()` on it because it is not the owner of it. The ownership remains in
the caller. This is commonly referred to as *borrowing* in Rust

In C/C++, pass by reference is done in order to allow the called function to modify the variable passed to it. So in
Rust, what happens if we try to modify something that was passed by reference? That is, modify a reference inside of
a scope that is *borrowing* it

```
fn main() {
   let s = String::from("hello");
   change(&s);
}

fn change(some_string: &String) {
   some_string.push_str(", world");
}
```

This won't work, but not because of anything to do with ownership or borrowing. Remember from earlier how variables in
Rust are immutable by default? `s` was *not* declared with the `mut` keyword; it doesn't matter that we're trying to
modify the value inside of another scope, we wouldn't be able to modify the value inside of the original scope to
begin with

#### Mutable References
If you remember the brief description of the `String` type from before, one key difference in static string literals
and dynamic `String` types in Rust is that the latter can be declared with the `mut` keyword

```
fn main() {
   let mut s = String::from("hello");
   change(&mut s);
}

fn change(some_string: &mut String) {
   some_string.push_str(", world");
}
```

This code now works how we would expect it to since `change()` is now taking a *mutable* reference as an argument. The
scope that is borrowing the reference is now allowed to modify it

However, mutable references have one big restriction!

*You can only have one mutable reference to a particular piece of data in a particular scope*

Take the following snippet of code for example

```
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2);
```

This code will fail to compile because of this restriction; we are trying to create two references to a mutable
piece of data inside of the same scope

This restriction allows for mutation of references but in a *very* controlled fashion. The benefit of having this
restriction is that Rust is able to catch race conditions at *compile time*

With that in mind, will the following snippet of code compile?

```
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;

println!("{}, {}, and {}", r1, r2, s);
```

Yes, but it does give us a warning, which should give us some insight as to why this code compiles and the previous code doesn't

```
   Compiling playground v0.0.1 (/playground)
warning: variable does not need to be mutable
 --> src/main.rs:4:9
  |
4 |     let mut s = String::from("hello");
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default

warning: 1 warning emitted

   Finished dev [unoptimized + debuginfo] target(s) in 1.33s
    Running `target/debug/playground`
```

This code actually compiles because the new references `r1` and `r2` are *immutable*. This may seem strange because the they
are referencing a *mutable* piece of data - but that is where the warning comes in, Rust is telling us we don't actually
need `s` to be mutable at all because we aren't writing to it anywhere in our code!

Okay, so what happens if we do write to `s` after declaring `r1` and `r2` in this code?

```
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;

s.push_str(", world!");

println!("{}, {}, and {}", r1, r2, s);
```

Now this is a compiler error, because we effectively just programmed another race condition, which Rust catches for us!

```
   Compiling playground v0.0.1 (/playground)
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
  --> src/main.rs:9:5
   |
6  |     let r1 = &s;
   |              -- immutable borrow occurs here
...
9  |     s.push_str("{}, {}, and {}");
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ mutable borrow occurs here
10 |
11 |     println!("{} {} {}", s, r1, r2);
   |                             -- immutable borrow later used here

error: aborting due to previous error

For more information about this error, try `rustc --explain E0502`.
error: could not compile `playground`

To learn more, run the command again with --verbose.
```

So we're able to observe that another restriction of mutable references is *you cannot have
a mutable reference while you also have an immutable one*

It might seem like we got around this in the example before this one, but we really
didn't. Rust informed us via a warning that we only had immutable references to `s`,
so there was no need to declare `s` as mutable in the first place. Doing so actually only
sets us up for errors in the future if and when we try to modify the contents of `s`

One more key point to make regarding mutable references is that you are able to have
*multiple* mutable references, just not *simultaneous* ones. Take the following code
for example

```
fn main() {
   let mut s = String::from("hello");

   {
      let r1 = &mut s;
   }

   let r2 = &mut s;
}
```

This is perfectly valid in Rust because remember, `r1` is inside of a different scope than `r2`
and will be dropped when that scope ends. Multiple mutable references are allowed so long as
those references do not exist within the same scope

To summarize:

   - You can only have **one** mutable reference to a piece of data within a scope at a time
   - You can **not** have a mutable reference to a piece of data while you also have an immutable reference to the same data
   - You are able to have *multiple* mutable references to some data as long they are not *simultaneous* mutable references

#### Dangling References
A *dangling reference* happens when a pointer references a location in memory that
may have been given to and freed by someone else in the program. Rust is able to
*guarentee* that references will never be dangling; if you have a reference to
some data, the compiler will ensure that the data will not go out of scope before
the reference to that same data does

Let's try to create a dangling reference and observe how the compiler prevents
us from doing so

```
fn main() {
   let reference_to_nothing = dangle();

   println!("{}", *reference_to_nothing);
}

fn dangle() -> &String {
   let s = String::from("hello");

   &s
}
```

Here, we try to allocate memory inside of `dangle()` and return a
reference to that memory to the caller. On the surface, this may seem fine,
so what's the problem with this in Rust? Why does this code produce a compiler error?

Because `s` is **dropped** as soon as the scope of `dangle()` ends. Since
ownership of `s` was not given back to the caller, `dangle()` is actually
returning a pointer to *nothing* - the memory is freed as soon as the function ends

The solution to this is to return the `String` directly instead

```
fn main() {
   let reference_to_something = no_dangle();

   println!("{}", reference_to_something)
}

fn no_dangle() -> String {
   let s = String::from("hello");

   s
}
```

This works without any problems. Ownership of the memory is moved out of
the `no_dangle()` scope and nothing is deallocated

In Rust, you can **not** return a reference to a variable that is owned
by a function, regardless of how the function gained that ownership. You
must return an owned object instead, that is, `String` instead of `&String`
or `Vec<T>` instead of `&[T]` etc.

### Slices
Slices let you reference a contiguous sequence of elements within a collection
rather than the whole collection. For example, a *string slice* is a reference
to a part of a `String`

Let's write a function that takes a `String` and returns the first word found
in the `String`

Now, if we were to do this without string slices, we wouldn't have an easy way to
return a *part* of the `String`, so lets return the index of the last character
in the first word instead

```
fn first_word(s: &String) -> usize {
   let bytes = s.as_bytes();

   for (i, &item) in bytes.iter().enumerate() {
      if item == b' ' {
         return i;
      }
   }

   s.len()
}
```

There's quite a bit to take in here, so we'll break it down

```
fn first_word(s: &String) -> usize
```

Since we do not want this function to have ownership of `s`, we need to
pass it by reference. Otherwise, `first_word()` would free the memory
associated with `s` when its scope ends

```
let bytes = s.as_bytes();
```

Because we need to go through the `String` element by element, we
convert our `String` into an array of bytes using the `as_bytes()`
method available to the `String` type

```
for (i, &item) in bytes.iter().enumerate()
```

This creates an iterator over the array of bytes with the `iter()` method. We
won't go into too much detail on iterators right now; just know that `iter()`
returns each element in the array as a tuple where the first element of the
tuple is the index and the second element is the reference to the data found
at that index

```
if item == b' ' {
   return i;
}
```

Here, we are looking for a space character by using the byte literal syntax `b' '`. If
we find a space, we return the index of it. Otherwise, we will just return the entire
length of the `String` at the end of `first_word()` using `s.len()`

This function works perfectly fine and this code compiles and runs just as we would
expect it to

```
fn main() {
   let s = String::from("Hello World!");

   let idx = first_word(&s);
}
```

So then what *is* the problem with this code? Well, what if `s` was mutable instead?

```
fn main() {
   let mut s = String::from("Hello World!");

   let idx = first_word(&s);

   s.clear();
   s.push_str("This is now an entirely diffent string!");
}
```

This also compiles without any issues. But the problem is more apparent now; The `idx`
we get from `first_word()` is only meaningful within the context of the `&String` we
got it from! There is no guarentee that this index will be valid in the future of our
program if `s` is allowed to be modified

Having to worry about `idx` getting out of sync with the data in `s` is tedious and
error prone. And even if `s` is immutable, indexing directly into a `String` is not
supported in Rust. The solution to this problem is to use*string slices* instead

#### String Slices
```
let s = String::from("Hello World!");

let hello = &s[0..5];
let world = &s[6..11];
```

Rather than being a reference to the entire `String`, `hello` and `world` are references
to *portions* of the `String`

With this in mind, lets rewrite our `first_word()` function to return a string
slice instead of an index

Note that the 'string slice' type is denoted as `&str`

```
fn first_word(s: &String) -> &str {
   let bytes = s.as_bytes();

   for (i, &item) in bytes.iter().enumerate() {
      if item == b' ' {
         return &s[0..i];
      }
   }

   &s[..]
}
```

Instead of returning `i` directly when a space character is found, now we are returning
a string slice of the first index, `0`, up to the index of the space, `i`. If no space
is found, we return the entire `String` as a string slice with `&s[..]`

But wait, why are we able to return `&s`? I thought you couldn't return
references from functions because the memory the reference points to will
be freed as soon as the function ends? Two reasons:

   1) `s` was passed by *reference* to `first_word()`, so we did not transfer the ownership
      of `s` to the function scope. Because `first_word()` is borrowing `s`, it can't `drop()` it
      when the function ends. The reference we return is guarenteed to be valid
   2) The reference we are returning points to *statically sized* data. In other words, a string
      slice is nothing more than a string literal!

If you wanted to declare a new string slice `s` with the initial contents of `Hello World!`, you
would do so with the following line of code

```
let s = "Hello World!";
```

The type of `s` here is `&str`; it's a slice that points to a specific point on the program stack.
Since string literals are immutable, that means `&str` is an **immutable reference**!

With our new function, `main()` would now look like the following

```
fn main() {
   let s = String::from("Hello World!");
   let first_word = first_word(&s);

   println!("The first word of `s` is: ", first_word);
}
```

Again, this works perfectly fine. In fact, this is where we wanted to end up with this exercise. But
the question from before still stands; what if `s` is mutable instead? And if it is, what happens if
you try and modify `s` after calling `first_word(&s)`? For example

```
fn main() {
   let mut s = String::from("Hello World!");
   let first_word = first_word(&s);

   s.clear();

   println!("The first word of `s` is: ", first_word);
}
```

**Now** this is a compiler error because Rust can enforce the borrowing rules that we have for
mutable references! Because `clear()` needs to modify the contents of `s`, it needs a mutable
reference of `s`. However, since we also need to use an immutable reference to pass `&s` into
`first_word()`, this can **not** be done. Otherwise, a race condition would be possible between
`s` and `first_word`

### Other Slices
String slices are specific to, well, strings. In Rust, you are able to use slices to refer to
a subset of the elements found in a vector or an array regardless of what the type of the elements
actually is

```
let a = [1, 2, 3, 4, 5];
```

If you wanted to create a slice of the first 3 elements, you would do so with the following code

```
let slice_of_a = &a[0..2];
```

Where `slice_of_a` now has the type `&[i32]`

## Sources
   - [The Rust Programming Language](https://doc.rust-lang.org/stable/book)
   - [Rustc](https://doc.rust-lang.org/rustc)
   - [A Guide to Porting C/C++ to Rust](https://locka99.gitbooks.io/a-guide-to-porting-c-to-rust/content/)
   - [The Rust Standard Library](https://doc.rust-lang.org/std)
