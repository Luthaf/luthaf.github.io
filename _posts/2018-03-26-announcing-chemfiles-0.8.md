---
layout:     post
title:      Announcing chemfiles 0.8
summary:    >
            Short of a year after chemfiles 0.7, here comes version 0.8!
            Fresh with more formats and new functionalities. Get it now!
---

We are very happy to announce the release of the 0.8.0 version of Chemfiles.
Chemfiles is a C++ library providing write and read access to chemistry file
formats. Chemfiles also has bindings to other languages and can be used from C,
Fortran, Python, Julia and Rust.

If you don't know Chemfiles, you can learn more about why we are writing it,
what problem it solves and what are some of its use cases [here](http://luthaf.fr/announcing-chemfiles-0.7.html).

You can download and install this release from your favorite package manager:

```bash
# conda
conda install -c conda-forge chemfiles

# pip
pip install chemfiles

# cargo
[dependencies]
chemfiles = "0.8"

# Julia Pkg
> Pkg.add("Chemfiles")
```

We also have pre-built binaries for various Linux distributions in [OpenSuse
build service], and code source is available on [Github].

<img src="/img/chemfiles.svg"
     alt="chemfiles logo"
     style="max-width: 100px; display: block; margin: auto;">

# What's new in version 0.8

We added support for four new formats: LAMMPS data files, Tinker .arc files
(also called tinker XYZ format), MOL2 files and Molden files. LAMMPS Data files
are used to specify starting configuration for the [LAMMPS] molecular dynamics
package, and .arc files are used as input and output by the [Tinker] familly of
molecular dynamics software. [Molden] is a software for visualizing the output
from quantum calculations, and a lot of *ab initio* software support creating
Molden files.

We also switched from Mozilla Public License to BSD license, in order to make it
easier to use chemfiles in all projects, commercial and open source.

Chemfiles also gained support for storing and retrieving arbitrary properties on
atoms and frames. Properties are values associated with a name and an atom/a
frame. For example, the forces acting on an atom or its beta factors can be
atomic properties, while the author or the instrument used to generate a file
can be set as Frame properties. These properties will help keeping the interface
of Chemfiles small and easy to use, by having a single way to specify any
additional data that can be in a specific format.

This release also adds support for reading configuration files (by default
located in .chemfilesrc). For now, only type renaming is implemented in the
configuration files, but new features are coming! Type renaming is useful for
some formats where numbers (2, 4, 5) or long strings (Hw, Ca1, Oxc) are used
instead of actual atomic types (H, C, O). As Chemfiles uses the atomic types to
get the radii and assign bonds, type renaming allow chemfiles to get the right
type and perform bonds detections or files conversion.

This release would not have been possible without many contributors, both to the
core C++ library and the languages bindings! Many thanks to all of them:
[@pgbarletta], [@lscalfi], [@frodofine] and [@g-bauer].

You too can contribute to chemfiles! All backgrounds and skills are welcomed and
needed, from documentation improvements to C++ code changes. Testing chemfiles
and giving your feedback is very valuable too.

# Going further

To learn more, you can go the [website][chemfiles], to the [Github] page of the
project, or the the [chat room][gitter]. See you soon with the chemfiles project!


[chemfiles]: http://chemfiles.org
[Github]: https://github.com/chemfiles/chemfiles
[OpenSuse build service]: https://software.opensuse.org/download.html?project=home%3ALuthaf&package=chemfiles
[LAMMPS]: http://lammps.sandia.gov/
[Tinker]: https://dasher.wustl.edu/tinker/
[Molden]: http://www.cmbi.ru.nl/molden/
[gitter]: https://gitter.im/chemfiles/chemfiles
[@pgbarletta]: https://github.com/pgbarletta
[@lscalfi]: https://github.com/lscalfi
[@frodofine]: https://github.com/frodofine
[@g-bauer]: https://github.com/g-bauer
