---
layout:     post
title:      Announcing chemfiles 0.9
summary:    >
            Version 0.9 of chemfiles is here, with new formats and multiple
            ergonomic improvements.
---

We are very happy to announce the release of the 0.9 version of Chemfiles.
Chemfiles is a C++ library providing write and read access to chemistry file
formats. Chemfiles also has bindings to other languages and can be used from C,
Fortran, Python, Julia and Rust.

You can download and install this release from your favorite package manager:

```bash
# conda
conda install -c conda-forge chemfiles

# pip
pip install chemfiles

# cargo
[dependencies]
chemfiles = "0.9"

# Julia Pkg
> Pkg.add("Chemfiles")
```

We also have pre-built binaries for various Linux distributions in [OpenSuse
build service], and code source is available on [Github].

<img src="/img/chemfiles.svg"
     alt="chemfiles logo"
     style="max-width: 100px; display: block; margin: auto;">

# What is chemfiles?

As I discussed in the [first public
presentation](http://luthaf.fr/announcing-chemfiles-0.7.html) of chemfiles on
this blog, there are a lot of different file formats used in computational
chemistry. Most simulation and analysis softwares define their own format, but at
the same time most formats contains the same kind of information.

This multiplication of formats means that there is less interoperability between
different software. To run analysis on the results of a simulation wee have to
either use readily available algorithms, or learn how to read the format of our
current software. And when we try to use another software (because a specific
method is available there, or a different force-field); we need to rewrite most
of my analysis scripts. There is a lot of friction to switch between software
and just try to use different methods.

<img src="/img/files-formats.png"
     alt="list of file formats"
     style="max-width: 300px; display: block; margin: auto;">

Chemfiles is an attempt to provide an unified and simple interface for
programmers to work with all these file formats. It does not try to create a
[new file format](https://xkcd.com/927/) to unify all uses cases, instead it
provides interoperability between existing formats. In order to do so, chemfiles
defines an internal representation for the data that can exist in various
formats, and transparently read and write files from this internal format.
Which means that you only need to learn how to use chemfiles, and you can use it
to read or write every supported file format!

# What's new in version 0.9

We added support for three new formats, bringing the total number of [supported
formats] to 18! The new formats are the MacroMolecule Transmission Format
(**MMTF**), created by the people behind the protein data bank and PDB format.
It is a format designed for fast transmission over Internet, compact storage of
huge molecules, and fast loading. The two other new formats are the
Structure-Data File (**SDF**) format; and the Cambridge Structure Search and
Retrieval (**CSSR**) format.

Compressed files are now transparently read and written. Supported compression
algorithms include lzma (**.xz**) and gzip (**.gz**). Reading a compressed file
is as easy as:
```python
from chemfiles import Trajectory

file = Trajectory("name.xyz.gz")
frame = file.read()
```

---

The atoms selection language have been rewritten and extended to support more
complex expressions, such as mathematical constrains:
```python
from chemfiles import Selection

# Get the atoms in a certain region of space
selection = Selection("x^2 - y ^2 < sqrt(z^2 + 25)")
atoms = selection.evaluate(frame)
```

Other improvements include topology and bonding constrains; as well as
constrains on atomic properties:
```python
# Get the atoms bonded to an oxygen atom
selection = Selection("bonded(#1, name O)")

# Get the atoms defined with HETATM in PDB format
selection = Selection("[is_hetatm] and name S")
```

---

On a more technical side, the C API functions accessing data such as atoms, unit
cells, residues inside a frame or a topology no longer make a copy. This allows
to directly read and write inside the containing frame or topology, both from C
and from all the languages with a binding for chemfiles. This means that using
chemfiles from Python, Julia, Fortran or Rust should now be faster and more
ergonomic.

---

This release would not have been possible without all the contributions by
[@frodofine], many thanks to him!

You too can contribute to chemfiles! All backgrounds and skills are welcomed and
needed, from documentation improvements to C++ code changes. Testing chemfiles
and giving your feedback is very valuable too.

# Going further

To learn more, you can go have a look at the [documentation], go to the [Github]
page of the project, or the the [chat room][gitter]. Happy science!


[chemfiles]: http://chemfiles.org
[documentation]: http://chemfiles.org/chemfiles/0.9.1/
[supported formats]: http://chemfiles.org/chemfiles/0.9.1/formats.html
[Github]: https://github.com/chemfiles/chemfiles
[OpenSuse build service]: https://software.opensuse.org/download.html?project=home%3ALuthaf&package=chemfiles
[gitter]: https://gitter.im/chemfiles/chemfiles
[@frodofine]: https://github.com/frodofine
