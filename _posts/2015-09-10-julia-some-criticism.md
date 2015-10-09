---
layout:     post
title:      Julia, some criticism
summary:    >
            I am in the second part of my relation with the Julia language: after being
            enthusiastic about it, I am now seeing some of its flaw. Some of them can
            be fixed, the other or part of the language design decisions.
---

Relations with new technologies are like love: you will go through three phases:

- **Passion**: At first you hear about it, and someday you start spending time with it. And
  you fall for it:  *Oh my god, this is so nice! Why did I not used that before?*. You
  tell everyone about it, and want everyone to try it;
- **Disillusion**: Then you start finding that the technology also has bad sides, that
  world is not a pink warm blanket in which you can hide forever. You see things you do
  not like, and everything you try to overcome it do not seems to have an effect.
- **Acceptation**: You start knowing what is the fit for this technology, and what is not.
  You still use it for some stuff, but also rely on other to get things done.

I am in the second part of my relation with the Julia language: after being enthusiastic
about it, I am now seeing some of its flaw. I believe some of them can be fixed, and the
other are inherently part of the language or the design decisions.

# What will not be fixed

These issues will not be fixed, because there are part of the language design. They are
issues for me for two reasons:
    - They make it harder to maintain a big codebase;
    - They make it harder to use Julia as a general purpose language.

That is totally fine for most of the scientists (who are the main targeted audience), who
use Julia for everyday data exploration and small scientific scripts. But these issues
arise when trying (as [I was doing](https://github.com/Luthaf/Jumos.jl)) to write large
simulation software using Julia. Simulation softwares are not throw away scripts, they
will hopefully live in the wild for more than 10 years. They must be extendibles, and easy
to maintain.

## Include-based code organization

There is no one to one correspondence between Julia modules and file organization on the
filesystem. One have to explicitly declare a new module `module Bar ... end`, and use the
`include` function to include code in the current module. This have two consequences:
    - modules are big;
    - code organization do not follow file organization.

Modules are big because it is up to the programmer to declare new module. So most of them
(and I include myself) will not do it and have a big
[god-module](https://en.wikipedia.org/wiki/God_object).

The fact that code organization do not follow file organization is bad because you have to
read through all the files to learn what is going on. This add another entry barrier for
open source collaborations where newcomers have to spend time figuring what is going on,
and make maintenance of old code harder.

This is the same model as C/C++ `#include` for header files. But this model is bad, and
add a cognitive charge on top of the programmer head. C++ folks want to add a proper
module  system since the C++11 TS discussions.

## Very bad namespacing system

> Namespaces are one honking great idea -- let's do more of those!
>
> -- <cite>[The Zen of Python](https://www.python.org/dev/peps/pep-0020/)</cite>

In Julia, when you do `using Package`, you bring all exported names from the package
inside your current namespace. This is (for all exported symbols) the corresponding
operation to Python `from package import *`. But what append if we already have another
variable with the same name? Let's see (julia 0.4):

```julia
julia> module Foo
         export bar
         bar = 4
       end
Foo

julia> bar = 7
7

julia> using Foo
WARNING: using Foo.bar in module Main conflicts with an existing identifier.

julia> bar
7
```

Yay, nice, we get a warning here. And the value of our variable is not changed. Bu what if
we accidentally declare a variable with an already existing identifier?

```julia
julia> module Foo
         export bar
         bar = 4
       end
Foo

julia> using Foo  # bar == 4

julia> bar = 7
7

julia> bar 7
```

Here, we silently replaced bar without any warning. What is a desirable feature within a
new scope (we do want to be able to define variables with already used names in
functions), is not desirable in the global scope. Replace variables with functions in the
upper example, and add to this the fact that Julia encourage you to use short variables
and function names, and your code starts breaking randomly. Other functions get called in
the middle of a script, without having explicitly replaced them (as in Python, when we do
monkey-patching).

This is bad for maintenance. You can not refactor easily your code, or it will start
breaking like this. As there is a late binding for variables, you will not see errors
until the runtime of your functions. So you better have a lot of unit tests to check your
code. But scientists notoriously use not enough unit tests.

On the top of that, `Base` and `Core` export a lot of symbols and you do not know what is
or not in scope.

```bash
$ julia -e "whos(Base); whos(Core)" | wc -l
1515
```

## Too much magic

I think there is too much magic in Julia code, in the sense that you do no get full
control over what append at the system level. Coming from the C++ and Fortran side, this
can be annoying when it comes to memory management (where do allocations take place? No
idea without an external tool); or explicit control over SIMD. What do the `@simd` macro
do? No one know without looking the generated LLVM IR, and we have to hope for the best
when using it.

Automagically generation of code is nice for high-level construct, but when you want to
squeeze the last bit of performance from the hardware it is not really the best we can do.

# What can be fixed

This is a list of smaller flaw, that will be fixed or are being fixed currently.

- Enormous `Base` module. There are too much things in `Base`. I mean, why is the `airy`
  function part of the standard library? This is being [worked
  on](https://github.com/JuliaLang/julia/issues/5155);
- Installing native dependencies is hard, but this is true for most of the languages.
  Julia `BinDeps` package is already a good step forward;
- No debugger. You have to use the good ol' `println` everywhere, and when you have used
  `lldb` once, it is hard to go backward. This may change when
  [Galium.jl](https://github.com/Keno/Gallium.jl) become usable

# What is still nice and shiny

For a scientific computing language, Julia is still very nice: (multi-dimmensionals)
arrays are first-class citizen; multiple dispatch is a very nice way to implement reuse
code; battery included for scientific needs; a nice community. Another very nice feature
is the code generation introspection tools, using `@code_llvm` and `@code_native` macros.

I will still use Julia, but not for large codebase: maintaining integrity, and forcing
myself to respect a code organization it is just too much work that could be done by a
compiler.
