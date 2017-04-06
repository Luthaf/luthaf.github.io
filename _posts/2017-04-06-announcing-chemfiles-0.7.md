---
layout:     post
title:      Announcing chemfiles 0.7
summary:    >
            I am very happy to announce the release of the version 0.7 of the
            chemfiles library. Chemfiles is a modern library for reading and
            writing theoretical chemistry trajectory files.
---

I am very happy to announce the release of the version 0.7 of the free and
open-source [chemfiles] library. Chemfiles is a modern library for reading and
writing theoretical chemistry trajectory files.

You can install the new version with conda in the [conda-forge] channel, using
[pre-compiled] binaries on Linux, or from the code [source][github].

<img src="/img/chemfiles.svg"
     alt="chemfiles logo"
     style="max-width: 300px; display: block; margin: auto;">

## The computation chemistry files mess

A recurrent pain point for anyone working in the theoretical and computational
chemistry field is the multiplication of file formats. Every software (would it
be GAUSSIAN, VASP, CP2K, LAMMPS, GROMACS, AMBER, or any other one really) comes
with its own set of files formats. And it is up to the user to adapt, and learn
how to work with the set of formats used by a specific software.

But every format contains basically the same information: a variation around
atomic positions, velocities, forces, names and bonding. They still have
different use cases: PDB for proteins and bio-molecules, as an easy to edit
file; NetCDF or TNG for storing trajectories with thousands of atoms to the
disk, XYZ for an easy to read and write format, *etc.*

But this multiplication of formats means that there is less interoperability
between different software. To run analysis on the results of a simulation I
have either to use a readily available algorithm, or need to learn how to read
the format of my current software. And when I try to use another software
(because a specific method is available there, or a different force-field); I
need to rewrite most of my analysis scripts. There is a lot of friction to
switch between software and just try to use different methods.

<img src="/img/files-formats.png"
     alt="list of file formats"
     style="max-width: 300px; display: block; margin: auto;">

Chemfiles is an attempt to provide an unified and simple interface for
programmers to work with all these file formats. It does not try to create a new
file format to unify all uses cases, it provides interoperability between
existing formats. In order to do so, chemfiles defines an internal
representation for the data that can exist in various formats (a `Frame`), and
transparently read and write files from this internal format. Which means that
you only need to learn how to use a `Frame`, and you can use it to read or write
every supported file format.

## Use cases for chemfiles

You can use chemfiles everywhere you need to either read or write theoretical
chemistry data, and you would like to support multiple formats. Specifically, I
can see three major area where chemfiles can be useful to you:

1. Analysis of trajectory data. You can use chemfiles to write your code only
   once, and support multiple file formats without more work. I am working on
   a set of analysis and data manipulation algorithms in the [cfiles] project,
   using chemfiles for reading and writing the files.

2. Writing simulation software. You are writing your own simulation software,
   and want to be able to provide multiple format as output format. Or maybe you
   want to be able to read multiple input formats for the initial configuration.

3. Molecular viewer programs. If you are working on a molecular viewer program,
   chemfiles can provide you the ability to read various formats by implementing
   a single and well-defined interface.

## Features

Here is a set of the features chemfiles offer, and what make this library useful:

- chemfiles offer interface to multiple languages. It is written in C++, but you
  can use if from C, C++, Python, Fortran, Julia and Rust;
- chemfiles can read both text (XYZ, PDB) and binary file formats (NetCDF, XTC,
  TNG);
- chemfiles support a varying number of atoms along a trajectory -- for example
  for Grand-Canonical Monte-Carlo simulations;
- chemfiles support separated files for topology and trajectories. You can give
  the topology in PDB format, and then read the positions from an XTC file.
- chemfiles support periodic boundary conditions for orthorhombic and triclinic
  unit cells;
- chemfiles provides a rich atom selection language. It is up to my knowledge
  the only software with support for multiple selections, so I'll talk a bit
  more about this later.
- chemfiles is cross platform. It is continuously tested on GNU/Linux, OS X and
  Windows;

Concerning the multiple selections, I believe they are an unique feature of
chemfiles. Selections are a way to get a list of atoms in the system matching a
given constrain: for example `"name H or mass < 1.2"` or `"type Zn and (x > 22.7
or y < 12)"` are selections. With multiple selections, you can select multiple
atoms at once: `"pairs: name(#1) H and name(#2) O"` will give you all the O-H
pairs in the system. You can get more than two atoms at once, and even work with
bonding information: for example `"angles: type(#2) C"` will give you all the
angles (set of 3 atoms linked together) with a carbon in the middle.

## What is new in chemfiles 0.7

The 0.7 version of chemfiles adds support for residues. Residues are group of
atoms that belong to the same logical unit. They can be amino-acids in proteins,
small molecules or something else. This version also add support for the TNG
format from the GROMACS project, as well as a bunch of improvements to the
interface of the library.

If you want to know more about this release, please have a look at the [release
notes]!

## Getting started

We have a complete documentation about how to use chemfiles [here][doc]. You can
read the installation instructions
[here](http://chemfiles.org/chemfiles/latest/installation.html). We provide
pre-compiled binaries for Linux systems, or through the `conda` package manager
for Linux and OS X. If you have any question, or if you need help, we have a
chat room using [gitter], feel free to ask anything there!

#### Contributing

The chemfiles project is open to your contributions! You can help in a lot of
different ways:

- The first and main way to help chemfiles is to use it for your use case, and
  tell us what is easy and what is not, so that we can improve the code;
- The second way to help is to share the word! If you think chemfiles is nice
  and useful, share it with your colleagues, students, friends, …
- you can read the documentation and signal to us parts you don't understand so
  that we can improve them; or event better proposing improvements;
- By fixing bug in the existing code. We maintain a list of easy issues to get
  started with the [E-easy] label on Github. We also offer to mentor you through
  to fix these issues. If you are interested, leave a message on the issue or on
  [gitter].
- Implementing new formats to enlarge the list of supported formats;

Beginners and newbies are  welcome at all levels: your insight is very important
to improve the documentation; and we offer to help you getting started with the
code and the tools we use.

We also have some documentation for contributors [here][contributing], and do
not hesitate to ask questions on [gitter]!

----

Now, you can go the [website][chemfiles], to the [source code][github], or the
the [chat room][gitter]. See you soon with the chemfiles project!

[chemfiles]: http://chemfiles.org
[conda-forge]: https://conda-forge.github.io
[pre-compiled]: http://chemfiles.org/chemfiles/latest/installation.html#pre-compiled-binaries
[github]: https://github.com/chemfiles/chemfiles
[cfiles]: https://github.com/chemfiles/cfiles
[release notes]: https://github.com/chemfiles/chemfiles/releases/tag/0.7.0
[doc]: http://chemfiles.org/chemfiles/latest/
[contributing]: https://github.com/chemfiles/chemfiles/blob/master/Contributing.md
[gitter]: https://gitter.im/chemfiles/chemfiles
[E-easy]: https://github.com/chemfiles/chemfiles/issues?q=is%3Aissue+is%3Aopen+label%3AE-easy
