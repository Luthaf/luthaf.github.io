---
layout:     post
title:      Calling C++ from C and Fortran
summary:    >
   How to build a (fast) language-agnostic library ? Write it in C++,
   and call it from all the other languages. Today, how to call C++ from Fortran.
src: sources/cpp-from-fortran
---

> All the code in this article is available as a [gist](https://gist.github.com/Luthaf/4df78ca52b3caf7fbe0e),
if you want to use it.


The number of available languages nowadays is very big, and growing every day.
As a library writer, I would like to make sure that all the cool code I write
today will still be runnable in 10 or 20 years, if it is still useful. Ideally,
this very good library (let's call it `libfoo`) would be language agnostic,
such that any programmer can use it, whatever his favorite language be.

## Picking a language to write a library

In real world, I will always need to write some *glue* code between languages.
In order to make it easier to write this *glue* code, I can use C to write my
library. Any existing language does has a C interface, and can call C code in
one way or another.

But I don't really like C. I find that for my particular library, an object-oriented
(OO) language will help me, and will allow me to be more efficient. So, I need an OO
language with a very straightforward C interface. This interface will then be
called by all the bindings. And here comes the C++ !

As usual, when writing a C++ library, we should start by writing down our main
interface specification, in an header file. This library will be the `Foo` library,
built around the following `Foo.hpp` file:

```cpp
{% include {{page.src}}/Foo.hpp %}
```

This is the most boring library ever, with only one class, two useless function
member and a global function. The library implementation can be found in the
corresponding `Foo.cpp` file:

```cpp
{% include {{page.src}}/Foo.cpp %}
```

## Creating the C API for our code

The good thing with C++, is its very straightforward C interface. All we have to
do is use an `extern "C"` statement around the functions we need to wrap. So let's
create a mixed C/C++ header containing the exported C interface to `libfoo`.

We will wrap all the public functions, together with the constructor and the
destructor. The `foo.h` file contains the C interface, and can be parsed by either
a C compiler or a C++ compiler.

```cpp
{% include {{page.src}}/foo.h %}
```

This C wrapper will be implemented in C++. All we need is to use a pointer to
our `Foo` class, creating it dynamically with `new`. So let's put this code to
the `foo_capi.cpp` file.

```cpp
{% include {{page.src}}/foo_capi.cpp %}
```


## Wrapping the C API to Fortran

*Why fortran ? Why this old and dying language ?* You may ask. Well, three
reasons here: first, Fortran is still very used in the high performance computing
(HPC) world. You know, these guys running calculations for days on thousand of
processor. And this is not a metaphor: super-computers frequently have at least
1024 processor per computing node. Second, the last standard is from 2008, and
the previous one in 2003 introduced object-oriented programming. And third, I
like this language, because it is as simple as C for simple tasks, but a lot more
safer and easier to use.

I'll split the fortran interface in two file, because wrapping C can become very
verbose. The first one is `foo_cdef.f90`, and contains the interface declaration
for the external C function. This is the equivalent of the `foo.h` file.

```fortran
{% include {{page.src}}/foo_cdef.f90 %}
```

What is interesting here ? We use the `iso_c_binding` module from fortran 2003,
and bind the function to their C name. Another point is the `value` attribute
for all the parameter: in Fortran, all the arguments are passed by reference,
so we have to precise that we will use the pass-by-value convention.

And that's all we need to have something functional ! But let's make this
interface feels a bit more like fortran code. For this, we will use the
object-oriented (OO) capacities of the Fortran 2003 standard. Their is
something funny in wrapping C++ OO code to plain old C to build and OO module
in fortran … All this code goes to the `foo_mod.f90` file:

```fortran
{% include {{page.src}}/foo_mod.f90 %}
```

This library can now be used in a fortran program! As you can see, the library
interface is really easy to use, and looks a lot like the C++ interface.

```fortran
{% include {{page.src}}/test.f90 %}
```

Theoretically, the destructor should be automatically called when a variable
goes out of scope, enabling the RAII idiom in Fortran, but the gfortran
implementation (as of 4.9.2) is not complete for this point.

We can now compile the code, using this Makefile:

```makefile
{% include {{page.src}}/Makefile %}
```

You should note that as gfortran is the program which will call the linker, we
have to explicitly link to the C++ standard library. And now we can run our
program!

```sh
$ make
$ ./test.x
C API, create_foo
C++ side, constructor
          63  should be           63
   14.000000000000000       should be    14.000000000000000
C++ side, constructor
From Fortran! Foo(4, 2).bar(3) is: 7
C++ side, destructor
C API, delete_foo
C++ side, destructor
```

So, what is missing for this interface? We should add a couple of `try/cacth`
statements in the CAPI implementation, and replace it by a status code, in order
to play nicely with the C++ exceptions. Another amelioration point is for the
`foo_cdef.f90` and `foo_mod.f90` files, which are rather repetitive, boring and
error prone to write. It should be possible to use a C [parser](https://github.com/eliben/pycparser/) to generate these files from the
`foo.h` file. This would make another article!
