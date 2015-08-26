---
layout:     post
title:      First impressions on Rust
summary:   >
    Rust is a new programming language, created by the Mozilla foundation. All programming
    languages are just tools, and as an hammer can not be use to drill holes, not all
    programming languages target the same areas of application. Here I would like to share
    some thoughts on Rust design and usability.
---

[Rust](http://rust-lang.org/) is a new programming language, created by the mozilla
foundation. All programming languages are just tools, and as an hammer can not be use to
drill holes, not all programming languages target the same areas of application. Rust's
focus on safety, zero-cost abstractions and parallel computing without data races; and
target system programming with C efficiency, and C++ expressiveness.

I've been playing with Rust for a month, and here I would like to share some thoughts on
Rust design and usability.

# Rust model just feels right

> There is more than one way to do this.

That could be the *moto* of programming. In addition to the hundred of existing languages
out there, we have multiple paradigms to express our algorithms: from procedural and
structured programming to functional programming, including the omniscient object-oriented
paradigm. But in all these paradigms, one thing remains constant: we try as much as
possible to separate data and data structures from code and behavior.

This is not yet another programmer caprice, but a nice way to get easy to maintain, and
easy to adapt code.

And Rust is one of the first language I use which enforce this distinction at language
level. In Rust, we have `struct` to pack data together, and `taits` to express behavior.
And these two concepts are completely orthogonal, *i.e.* we can implement any trait we
want for any structure.

## Packing data and adding behavior

Let's take a little example by defining a 3D vector type: `Vect3D`. The only field will be
`data`, a Rust native array of float on 64-bits and of dimension 3.

```rust
struct Vect3D {
    data: [f64; 3]
}
```

This feels like C, but we can jump over and add some methods to Vect3D in an 'impl' block.

```rust
impl Vect3D {
    // 'new' is the conventional name for constructors
    fn new(x: f64, y: f64, z: f64) -> Vect3D {
        Vect3D{
            data: [x, y, z]
        } // No return statement, return the last expression if there is no ';'
    }

    // Scale the vector by the scalar value `s`
    fn scale(&self, s: f64) -> Vect3D {
        let x = self.data[0] * s; // Let introduce a variable, which type is inferred.
        let y = self.data[1] * s;
        let z = self.data[2] * s;
        Vect3D{
            data: [x, y, z] // Create and return a new scaled Vect3D
        }
    }
}
```
Which is already closer to C++.


A basic usage of our `Vect3D` type will looks like this:

```rust
fn main() {
    let v = Vect3D::new(1.0, 2.0, 3.0);
    // {:?} is the debug format statement
    println!("{:?}", v); // -> Vect3D { data: [1, 2, 3] }
    let u = v.scale(2.0);
    println!("{:?}", u); // -> Vect3D { data: [2, 4, 6] }
}
```

So the `impl` block allow us to add some specific behavior to our types, but what if we
want to abstract this behavior? How do we specify the way to index an object? How do we
abstract a whole class of "objects", like specifying the behavior of animals? In Rust,
trait are the answer this problematic.

## Zero-cost abstractions: traits defines behavior

In addition to data, behavior — and especially abstract behavior — provides the building
blocks for all programming task. We can find a way to define an abstract behavior and let
the users implement it in all languages: we have interfaces in Java, class inheritance
with pure abstract classes in C++ or Python, specials methods (like `__init__` or
`__add__`) in Python, methods definitions in Julia (`call`, `getindex`, …).

Rust uses traits to perform this. A trait in Rust is a group of functions with specific
names and signature and some associated data (associated types or constants for example).
This trait specify a behavior, and must be implemented for every specific `struct`. Here,
I implement the `Index` trait (the trait called by the `my_vector[3]` operation) for the
`Vect3D` type we defined earlier.

```rust
use std::ops::Index; // Bring the trait into scope

impl Index<usize> for Vect3D {
    type Output = f64;  // Indexing will return a 64-bit float

    // The index function is the only one that needs to be implemented.
    fn index(&self, index: usize) -> f64 {
        if index < 3 {
            self.data[index]
        } else {
            panic!("Out of bounds indexing!")
        }
    }
}
```

It is also easy to define our own trait, for example the `Animal` trait define the
behavior of some abstract animal.

```rust
trait Animal {
    // Declaring a function which will have to be implemented
    fn noise(&self) -> String;

    // Declaring a function with default implementation
    fn pet(&self) -> String {
        self.noise()
    }
}
```

We can then implement this animal trait for a `Cat` and a `Dog` struct, and play with the
implementations.

```rust
struct Cat;
struct Dog;

impl Animal for Cat {
    fn noise(&self) -> String {"meow"}
    fn pet(&self) -> String {"purrr"}
}

impl Animal for Dog {
    fn noise(&self) -> String {"bark, bark!"}
}

fn main() {
    let cat = Cat{};
    let dog = Dog{};

    cat.noise(); // -> "meow"
    dog.noise(); // -> "bark, bark!"

    cat.pet(); // -> "purr"
    dog.pet(); // -> "bark, bark!"
}
```

Rust also have support generic programming, like in C++ templates. Rust genericity is
based on trait, and use compile-time dispatch to create all the needed functions. Another
interesting point is that generic functions are checked at definition and not at
instantiation, removing all the horror of C++ template error messages.

## Use traits, they are good for you

I think this data *vs* behavior separation is a nice way to go, and gives us lot of things
at once:

- clean interface separation, without the class inheritance nightmare (where does
  this function come from already?);
- easy to maintain code. This is a consequence of the previous point, and of the
  data/behavior separation;
- zero-cost abstractions, because these traits are still plain old function, and the
  compiler is LLVM-baked.
- RAII based memory management, which free us from doing lot of resource management by
  hand.

Rust also have some other nice points:

- safety as a default: everything is immutable by default, and you have to ask for a
  mutable version is you need it. This is the other way around in C/C++ where you should
  to over-use the `const` keyword;
- nice modules (not based on the `#include` trick!) system, where you can write code
  in the order you want;
- all code is private by default;
- generic programming is supported in a clever way, based on traits;
- All the code is statically checked, and only runtime bugs are still present.

I think that Rust is what C++ is going to look like in 20 years, with a proper module
system, and more and more static verification of the code.

# Rust is not for beginners …

Rust still have some downsides:

- the **current** lack of some libraries, or the presence of libraries, but only in the
  unstable channel. This point should improve with time, as more and more people start to
  use the language;
- Rust is very complex, even more than C++.

You have to learn concepts like moving, borrowing, references, lifetime even before the
"hello world" example. And then you try to add generic, traits, and parallelism, and you
just go mad.

The compiler is not permissive at all, and yell at you every time when you start writing
code. At least, it provides useful suggestions, and clear explanations.

For me, Rust is great for big codebase, where parallelism is needed. And it can
replace C++ here, being far more easy to maintain and to keep alive. It also remove all
bugs from memory error, and even if I do not have years of experience in C++, this kind
of bug have already bitten me often!
