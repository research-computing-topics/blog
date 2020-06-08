Building GCC Compilers on an HPC Cluster
==============================================

.. post:: May 20, 2020
   :tags: gcc, gnu
   :category:
   :author: Shao-Ching Huang

Background
----------------

The GNU Compiler Collection (GCC) is a leading open-source compiler suite that
includes C, C++, Fortran and other languages. GCC is not to be confused with
``gcc``, which is the GNU C compiler, one of the components in GCC. On  the
Linux system of the cluster, the built-in GCC version may be too old  (e.g. on
CentOS 6, this is GCC 4.4.7) for building (compiling) some applications that
requires the language features not yet supported by that version of GCC. To
build these applications on an HPC cluster, such as UCLA Hoffman2 cluster, it is
necessary to install a newer version of GCC on the system. A common
misconception is to assume that installing a new GCC requires Linux system
administrator privilege. This is false. In fact, a regular user can install
virtually any version of GCC available, or most open-source application
programs, under the home directory independently using existing tools available
on the cluster. In this technical note, we discuss the process of installing GCC
9. We note by passing that the process of installing other versions of GCC is
similar, if not identical. The process of installing many other GNU software
packages is also largely the same.


GCC Evolution
------------------

Starting from GCC 5, GCC uses 2-number version naming convention (e.g. 9.3), in
contrast to the 3-number version number used in prior releases (e.g. GCC 4.4.7).
In the new 2-number versioning system, the first number is the major version
number, implying that new features are introduced into that release. The second
number is the minor version, denoting bug fixes that correct the reported issues
introduced in the initial major release.  We note that, while the main
documentation now uses the post-GCC-5 two-number versioning system, when
downloading the GCC source for compiling, as discussed in later sections, a
third number is still present in the file name (e.g. "0" in "gcc-9.3.0.tar.gz").

Here are some examples of the new features introduced in GCC major releases over
the years:

- C++11 is added in and after GCC 4.8.4.
- In GCC 5, the default mode for C is changed to ``-std=gnu11``
  instead of ``-std=gnu89``.
- In GCC 6, the default mode for C++ is changed to ``-std=gnu++14``
  instead of ``gnu++98``.
- In GCC 7, a number of C++ language related changes, e.g. enforcing stricter
  rules when using templates, and the implementation of most of OpenACC 2.0a
  specification is added.
- In GCC 8, the C and C++ compilers can emit more fix-it hints.
- In GCC 9, there are further improvements of command line options and
  diagnostics information, and the implements most of OpenACC 2.5 specification
  is added.
- In GCC 10, the implement most of OpenACC 2.6 specification is added.

More details about the changes in each release are available online. For
example, https://gcc.gnu.org/gcc-9/changes.html are the new changes for GCC 9.

Sometimes the build process needs to be modified (e.g. using alternative
compiler options) if the application assumes certain default behavior of GCC
that is different from that of the version being used.



Building GCC
-------------------


Download the source
```````````````````````

Different versions of GCC can be freely downloaded from one of the GNU mirror
sites, listed in ``https://gcc.gnu.org/releases.html``. Here we use the ``wget``
command to download GCC 9.3 from the mirror site
``https://mirrors.kernel.org/gnu/gcc`` into the current directory, such as your
scratch directory:

.. code-block::

    $ wget https://mirrors.kernel.org/gnu/gcc/gcc-9.3.0/gcc-9.3.0.tar.gz

The downloaded ``.tar.gz`` is a compressed file. It needs to be uncompressed and
expanded before use, described in the next section.



Decompress the .tar.gz file
`````````````````````````````

We can use the ``tar`` command to decompress (a.k.a. "un-gzip") and to expand
(a.k.a. "untar") the tar file in one go:

.. code-block::

    $ tar xvfz gcc-9.3.0.tar.gz
    $ cd gcc-9.3.0

Several arguments have been passed to the ``tar`` command:

- "x": expand the tar file back to its original file structure (i.e. individual
    files)
- "f": use the file
- "v": display the progress on screen
- "z": decompress the gzip'd ``.gz`` file

See https://www.gnu.org/software/tar/manual/ for more details about ``tar``.

In the ``gcc-9.3.0`` directory, there should be many files, including
``configure``, which is the script we will use next, as well as several
documentation files such as ``README`` and ``NEWS``.


Download the prerequisites
```````````````````````````

Building GCC requires a number of external libraries (e.g. `GMP
<https://gmplib.org/>`_). They need to be installed prior to installing GCC.
Care should be taken about the versions of these libraries because GCC requires
matching versions (or ranges of versions) of these libraries; incompatible
external libraries result in a broken GCC build. In the ``gcc-9.3.0/`` directory
from the last step, this script is run to download the required external
libraries:

.. code-block::

    $ ./contrib/download_prerequisites

The screen output should look similar to:

.. code-block::

    2020-05.../pub/gcc/infrastructure/gmp-6.1.0.tar.bz2 [2383840] -> "./gmp-6.1.0.tar.bz2" [1]
    2020-05.../pub/gcc/infrastructure/mpfr-3.1.4.tar.bz2 [1279284] -> "./mpfr-3.1.4.tar.bz2" [1]
    2020-05.../pub/gcc/infrastructure/mpc-1.0.3.tar.gz [669925] -> "./mpc-1.0.3.tar.gz" [1]
    2020-05.../pub/gcc/infrastructure/isl-0.18.tar.bz2 [1658291] -> "./isl-0.18.tar.bz2" [1]
    gmp-6.1.0.tar.bz2: OK
    mpfr-3.1.4.tar.bz2: OK
    mpc-1.0.3.tar.gz: OK
    isl-0.18.tar.bz2: OK
    All prerequisites downloaded successfully.


**A word about where to run the build.** After downloading all of the files, we
*can proceed to configure and build GCC, described in the next sections. These
*steps are CPU-intensive. It is highly recommended to launch an interactive
*session (via the job scheduler) to perform the operations on a compute node, as
*the login nodes may be too loaded or memory-restricted for these tasks.


Configure
``````````

Before building (compiling) GCC, we need to configure it by passing several
parameters to the configure script and also let the configure script to detect
certain machine specific parameters of the cluster.

One requirement of GCC is that one cannot run the configure script directly in
the same directory where the ``configure`` is located. We need to create a
separate, empty directory and run the configure script from the new directory.
These steps can be done by the following commands:

.. code-block::

    $ cd ..             # go up one level from gcc-9.3.0
    $ mkdir gcc_build

Before proceeding to the next step, let's confirm that you have the two
directories at the same level:

.. code-block::

    $ ls
    gcc-9.3.0  gcc-9.3.0.tar.gz  gcc_build

The ``gcc-9.3.0`` is the result of expanding ``gcc-9.3.0.tar.gz``, and
``gcc_build`` is the new empty directory where we will run the configure script.
These two directories may be deleted after GCC is successfully built and
installed.


Enter the ``gcc_build`` directory and run the configure script (located in
another directory, ``gcc-9.3.0``), assuming that you will install GCC to your
directory ``$HOME/sw/gcc/9.3.0``:

.. code-block::

    $ cd gcc_build
    $ ../gcc-9.3.0/configure \
      --prefix=$HOME/sw/gcc/9.3.0 \
      --disable-multilib \
      --enable-languages=c,c++,fortran,jit \
      --enable-checking=release \
      --enable-host-shared

The meanings of the parameters are:

- ``--prefix``: the directory where GCC will be installed
- ``--disable-multilib``: we will build only the 64-bit version of GCC.
- ``--enable-languages``: only the compilers of the specified languages will
  be built; GCC contains more other languages.
- ``--enable-checking``: Enable additional checkings for stage1 of compiler
- ``--enable-host-shared``: Build host code as shared library;
  this is needed for jit.

See https://gcc.gnu.org/install/configure.html for more information about
configuring GCC.


Compile
````````````````````

After running configure, we are ready to build GCC. We are going to build GCC
using the system default compiler, e.g.

.. code-block::

    $ which gcc
    /usr/bin/gcc
    $ gcc --version
    gcc (GCC) 4.4.7 20120313 (Red Hat 4.4.7-23)
    Copyright (C) 2010 Free Software Foundation, Inc.
    This is free software; see the source for copying conditions.  There is NO
    warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

To build GCC, simply issue the ``make`` command in the `gcc_build` directory
(where we just ran the ``configure`` command in the previous section). We add
``-j 4`` to enable parallel build (using multiple CPU cores) to accelerate the
build process:

.. code-block::

    $ make -j 4

The ``make`` step takes a while to complete. If the ``make`` step is successful,
``make install`` will install GCC to the directory specified by ``--prefix`` in
the configure step.


Test
`````

It is a good idea to run GCC tests to see if it is correctly built. Running
tests takes a while to complete. To run GCC tests, run the following command in
the build directory (e.g. ``gcc_build`` from previous sections):

.. code-block::

    $ make -k check


Install
````````

From the build directory (e.g. ``gcc_build``), run this command to install GCC
to the target directory, specified by ``--prefix`` in the configure step:

.. code-block::

    $ make install



Using GCC
----------

To use the newly installed GCC, the corresponding file system paths need to be
added to the environment variables:

.. code-block::

    install_dir=$HOME/sw/gcc/9.3.0       # GCC install directory
    export PATH=$install_dir/bin:$PATH
    export LD_LIBRARY_PATH=$install_dir/lib64:\
    $install_dir/lib:$GCC_DIR/lib/gcc/x86_64-pc-linux-gnu/9.3.0:\
    $install_dir/libexec/gcc/x86_64-pc-linux-gnu/9.3.0:\
    $LD_LIBRARY_PATH
    unset install_dir

The temporary variable ``install_dir`` corresponds to the the path set by
``--prefix`` in the configure step, discussed previously. After using its value
in ``PATH`` and ``LD_LIBRARY_PATH``, it can be removed, or "unset" as shown
above. The purpose of the ``PATH`` environment variable is for the GCC commands
(e.g. typing ``gcc``) to be found automatically without a full path. The purpose
of the ``LD_LIBRARY_PATH`` environment variable is for the shared libraries
associated with GCC be found at run time. To make ``$PATH`` and
``$LD_LIBRARY_PATH`` permanent, this block can be added to ``~/.bashrc`` or
``~/.bash_profile``.

The Linux operating system uses the file system paths in ``PATH`` and
``LD_LIBRARY_PATH`` in the left-to-right order. We added the newly installed GCC
at the beginning of these environment variables, so they are found (and used) in
the current shell, even if there are paths to other versions of GCC later in the
paths.


Summary
----------

This technical note summarizes the essential steps of installing GCC compilers
within a regular user's own directory without requiring superuser (or system
administrator) privilege in an HPC cluster environment, such as UCLA Hoffman2
cluster. The procedure is expected to be the same, except for the version
numbers, for similar GCC versions, at least those released in recent past or in
near future.


Acknowledgement
-------------------

The author wishes to thank Dr. B. Winjum for his helpful comments on an earlier draft of this report.


Appendix
--------------

The script to run the entire procedure described in this note is available at:

https://gist.github.com/schuang/5df8dd3c7c17067cdeadc09d607f7cfa


