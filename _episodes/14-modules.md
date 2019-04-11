---
title: "Accessing software"
teaching: 20
exercises: 15
questions:
- "How do we load and unload software packages?"
objectives:
- "Understand how to load and use a software package."
keypoints:
- "Load software with `module load softwareName`"
- "Unload software with `module purge`"
- "The module system handles software versioning and package conflicts for you automatically."
- "You can edit your `.tcshrc` or `.turing_tcshrc` file to automatically load a software package."
---

On a high-performance computing system, no software is loaded by default. If we want to use a
software package, we will need to "load" it ourselves.

Before we start using individual software packages, however, we should understand the reasoning
behind this approach. The two biggest factors are software incompatibilities and versioning.

Software incompatibility is a major headache for programmers. Sometimes the presence (or absence) 
of a software package will break others that depend on it. Two of the most famous examples are 
Python 2 and 3 and C compiler versions. Python 3 famously provides a `python` command that 
conflicts with that provided by Python 2. Software compiled against a newer version of the C 
libraries and then used when they are not present will result in a nasty `'GLIBCXX_3.4.20' not
found` error, for instance.

Software versioning is another common issue. A team might depend on a certain package version for
their research project - if the software version was to change (for instance, if a package was
updated), it might affect their results. Having access to multiple software versions allow a set of
researchers to prevent software versioning issues from affecting their results.

## Environment modules (Lmod)

Environment modules are the solution to these problems. A module is a self-contained software
package - it contains all of the files required to run a software package and loads required
dependencies.

To enable Lmod, first use `enable_lmod` and then see available software modules by using `module avail`

```
[yourUsername@turing1 ~]$ enable_lmod
[yourUsername@turing1 ~]$ module avail
```
{: .bash}
```
---------------------------- /cm/shared/mls/Core ----------------------------
   R/3.4                    cuda/10.0          matlab/2018  (D)
   R/3.5             (D)    gcc/4              mono/5.0
   anaconda2/4.2            gcc/5              nodejs/10.15
   anaconda2/4.3            gcc/6       (D)    octave/4.0
   anaconda2/2018    (D)    ghc/8.0            octave/4.2
   anaconda3/4.2            ghc/8.2     (D)    octave/4.4   (D)
   anaconda3/4.3            go/1.10            perl/5.24
   anaconda3/2018    (D)    go/1.11            perl/5.26
   binutils/2.28            go/1.12     (D)    perl/5.28    (D)
   binutils/2.29            icc/16             pgi/16
   binutils/2.30            icc/17             pgi/17
   binutils/2.31            icc/18.0           pgi/18       (D)
   binutils/2.32-bfd        icc/18      (D)    python/2.7
   binutils/2.32     (D)    icc/19             python/3.6   (D)
   clang/6.0                java/1.8           python/3.7
   clang/7.0         (D)    java/11.0   (D)    ruby/2.5
   cuda/7.5                 julia/1.0          ruby/2.6     (D)
   cuda/8.0                 lua/5.1            scala/2.11
   cuda/9.0                 lua/5.3     (D)    tcl/8.5
   cuda/9.1                 matlab/2016        tcl/8.6      (D)
   cuda/9.2          (D)    matlab/2017

------------------------ /cm/shared/mls/Applications ------------------------
   BEST/2.3                    mmtsb/2019.02
   FastME/2.1.5                mothur/1.38
   FoamExtended/4.0            mothur/1.39
   HybPhyloMaker/1.6           mothur/1.41        (D)


[removed most of the output here for clarity]

  Where:
   D:  Default Module

Use "module spider" to find all possible modules.
Use "module keyword key1 key2 ..." to search for all possible modules
matching any of the "keys".
```
{: .output}

## Loading and unloading software

To load a software module, use `module load`.
In this example we will use Python 3.

Initially, Python 3 is not loaded. 
We can test this by using the `which` command.
`which` looks for programs the same way that Bash does,
so we can use it to tell us where a particular piece of software is stored.
```
[yourUsername@turing1 ~]$ which python3
```
{: .bash}
```
python3: Command not found.
```
{: .output}

We can load the `python3` command with `module load`:

```
[yourUsername@turing1 ~]$ module load python
[yourUsername@turing1 ~]$ which python3
```
{: .bash}
```
/cm/shared/applications/Python/3.6.8/bin/python3
```
{: .output}

So what just happened?

To understand the output, first we need to understand the nature of the `$PATH` environment
variable. `$PATH` is a special environment variable that controls where a UNIX system looks for
software. Specifically `$PATH` is a list of directories (separated by `:`) that the OS searches
through for a command before giving up and telling us it can't find it. As with all environment
variables we can print it out using `echo`.

```
[yourUsername@turing1 ~]$ echo $PATH
```
{: .bash}
```
/cm/shared/applications/libffi/3.2.1/bin:/cm/shared/applications/openssl/1.1.1b/bin:/cm/shared/applications/libmpdec/2.4.2/bin:/cm/shared/applications/Python/3.6.8/bin:/cm/shared/applications/slurm/current/bin:/usr/lib64/qt-3.3/bin:/usr/local/bin:/bin:/usr/bin:/sbin:/usr/sbin:/cm/local/apps/environment-modules/3.2.10/bin
```
{: .output}

You'll notice a similarity to the output of the `which` command. In this case, there's only one
difference: the `/cm/shared/applications/Python/3.6.8/bin/` directory at
the beginning. When we ran `module load python`, it added this directory to the beginning of
our `$PATH`. Let's examine what's there:

```
[yourUsername@turing1 ~]$ ls /cm/shared/applications/Python/3.6.8/bin/
```
{: .bash}
```
2to3      easy_install      idle3    pip   pip3.6  pydoc3.6  python3    python3.6-config  python3.6m-config  pyvenv      wheel
2to3-3.6  easy_install-3.6  idle3.6  pip3  pydoc3  python    python3.6  python3.6m        python3-config     pyvenv-3.6
```
{: .output}

Taking this to it's conclusion, `module load` will add software to your `$PATH`. It "loads"
software. A special note on this - depending on which version of the `module` program that is
installed at your site, `module load` will also load required software dependencies. To 
demonstrate, let's use `module list`. `module list` shows all loaded software modules.

```
[yourUsername@turing1 ~]$ module list
```
{: .bash}
```

Currently Loaded Modules:
  1) libmpdec/2.4   2) openssl/1.1   3) libffi/3.2   4) python/3.6

```
{: .output}

Now, let's load the `tensorflow` module.

```
[yourUsername@turing1 ~]$ module load tensorflow
[yourUsername@turing1 ~]$ module list
```
{: .bash}
```
Currently Loaded Modules:
  1) libmpdec/2.4   3) libffi/3.2   5) libstdcxx/4    7) mkl/19       9) hdf5/1.10  11) tensorflow/1.13
  2) openssl/1.1    4) python/3.6   6) mkl-dnn/0.18   8) numpy/1.16  10) h5py/2.8

```
{: .output}

So in this case, loading the `tensorflow` module (a machine learning library), also loaded
`mkl/19`,`hdf5/1.10`,`mkl-dnn/0.18`,`numpy/1.16`, and `h5py/2.8` as well. 

Let's try unloading the `tensorflow` package.

```
[yourUsername@turing1 ~]$ module unload tensorflow
[yourUsername@turing1 ~]$ module list
```
{: .bash}
```
Currently Loaded Modules:
  1) libmpdec/2.4   2) openssl/1.1   3) libffi/3.2   4) python/3.6

```
{: .output}

So using `module unload` "un-loads" a module along with its dependencies.
If we wanted to unload everything at once, we could run `module purge` (unloads everything).

```
[yourUsername@turing1 ~]$ module purge
[yourUsername@turing1 ~]$ module list
```
{: .bash}
```
No modules loaded
```
{: .output}

## Software versioning

So far, we've learned how to load and unload software packages. This is very useful. However, we
have not yet addressed the issue of software versioning. At some point or other, you will run into
issues where only one particular version of some software will be suitable. Perhaps a key bugfix
only happened in a certain version, or version X broke compatibility with a file format you use. In
either of these example cases, it helps to be very specific about what software is loaded.

Let's examine the output of `module avail` more closely.

```
[yourUsername@turing1 ~]$ module avail
```
{: .bash}
```
---------------------------- /cm/shared/mls/Core ----------------------------
   R/3.4                    cuda/10.0          matlab/2018  (D)
   R/3.5             (D)    gcc/4              mono/5.0
   anaconda2/4.2            gcc/5              nodejs/10.15
   anaconda2/4.3            gcc/6       (D)    octave/4.0
   anaconda2/2018    (D)    ghc/8.0            octave/4.2

```
{: .output}

Let's take a closer look at the `gcc` module. GCC, or the GNU Compiler Collection, is a compiler system supporting various programming languages, such as C, C++, Fortran, and others.
Lots of software is dependent on the GCC version, and might not compile or run if the
wrong version is loaded. In this case, there are three different versions: `gcc/4`, `gcc/5` and
`gcc/6`. How do we load each copy and which copy is the default?

In this case, `gcc/6` has a `(D)` next to it. This indicates that it is the default - if we type
`module load gcc`, this is the copy that will be loaded.

```
[yourUsername@turing1 ~]$ module load gcc
[yourUsername@turing1 ~]$ gcc --version
```
{: .bash}
```
gcc (GCC) 6.5.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```
{: .output}

Now, let's run `module avail` again. 

```
[yourUsername@turing1 ~]$ module avail
```
{: .bash}
```
--------------------- /cm/shared/mls/Compiler/gcc/6 -----------------------
   CGAL/4.8           cuda/9.1          mkl/17             netcdf/4.6   (D)
   CGAL/4.9           cuda/9.2   (D)    mkl/18             netcdf/4.6.1
   CGAL/4.10   (D)    cuda/10.0         mkl/19      (D)    openblas/0.2 (D)
   charm++/6.7        fftw/3.3   (D)    mpich/3            openmpi/2.0
   cuda/7.5           hypre/2.13        mvapich/2.2        openmpi/2.1  (D)
   cuda/8.0           hypre/2.15 (D)    mvapich/2.3 (D)
   cuda/9.0           mkl/16            netcdf/4.4

```
{: .output}

We now see more software available that is dependent on the `gcc/6` module. Let's load `openmpi` and
run `module avail` again.

```
[yourUsername@turing1 ~]$ module load openmpi
[yourUsername@turing1 ~]$ module avail
```
{: .bash}
```
----------------- /cm/shared/mls/MPI/gcc/6/openmpi/2.1 --------------------
   ParMGridGen/1.0        gromacs/4.6             mkl/19        (D)
   amber/18               gromacs/2016            netcdf/4.4
   boost/1.54             gromacs/2018            netcdf/4.6    (D)
   boost/1.59             gromacs/2019     (D)    netcdf/4.6.1
   boost/1.61             hdf5/1.8                orca/4.1
   boost/1.62             hdf5/1.10               parmetis/4.0
   boost/1.63             hdf5/1.10.1      (D)    petsc/2.3
   boost/1.66             hpl-mkl/2.2             petsc/3.6
   boost/1.68             hpl-openblas/2.2        petsc/3.10    (D)
   boost/1.69      (D)    hypre/2.13              pnetcdf/1.11
   charmm/42              hypre/2.15       (D)    scalapack/2.0
   exafmm/2018            mkl/16                  scotch/6.0    (D)
   fftw/3.3        (D)    mkl/17
   ga/5                   mkl/18

```
{: .output}

In this case, we have additional packages available that are dependent on `gcc/6` and `openmpi/2.1`.  But what if we wanted to use openmpi version 2.0, instead?

```
[yourUsername@turing1 ~]$ module load openmpi/2.0
```
{: .bash}
```
The following have been reloaded with a version change:
  1) openmpi/2.1 => openmpi/2.0

```
{: .output}

Now let's run `module list` to see which modules are loaded. 


```
[yourUsername@turing1 ~]$ module list
```
{: .bash}
```
Currently Loaded Modules:
  1) binutils/2.32   2) gcc/6   3) openmpi/2.0

```
{: .output}

We now have successfully switches from OpenMPI 2.1 to OpenMPI 2.0. This way, we can load non-defaults or specific versions of a software package. 

> ## Loading a module by default
> 
> Adding a set of `module load` commands to all of your scripts and having to manually load modules
> every time you log on can be tiresome. Fortunately, there is a way of specifying a set of 
> "default  modules" that always get loaded, regardless of whether or not you're logged on or 
> running a job. Every user has a hidden file in their home directory: 
> `.turing_tcshrc` (you can see these files with `ls -la ~`). These scripts are run every time you 
> log on or run a job. Adding a `module load` command to this shell script means that 
> that module will always be loaded. Modify `.turing_tcshrc` to 
> load a commonly used module like Python. Does your `python3 --version` command from before still 
> need `module load` to work?
{: .challenge}

{% include links.md %}
