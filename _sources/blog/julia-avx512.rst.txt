Julia AVX512 Issue and Workaround
=======================================

.. post:: June 8, 2020
   :author: Shao-Ching Huang
   :tags: blog
   :category:


Problem description
-------------------------

`Julia <https://docs.julialang.org/>`_ is a high-level dynamic programming
language designed for numerical computing and computational sciences.  One of
the core components in Julia is numerical linear algebra. Like many other
computational software packages, Julia follows a layered approach and depends
on a third-party optimized implementation of `BLAS (Basic Linear Algebra
Subprograms)
<https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms>`_.  By
default, Julia uses `OpenBLAS <https://github.com/xianyi/OpenBLAS>`_, one of
the popular implementations of BLAS.  Recently we noticed one anormaly when
running Julia on Intel "Skylake" CPUs that are AVX512-capable but have AVX512
disabled (e.g.  this can happen on a virtual machine) with error message:

.. code-block:: text

        $ julia
        Invalid instruction at 0x7fc4868b3ba8: 0x62, 0xf1, 0x7d, 0x48, 0xef, 0xc0, 0xc3, 0x90, 0x89, 0xc2, 0x83, 0xe2, 0xe0, 0x0f, 0x8e
        signal (4): Illegal instruction
        in expression starting at none:0
        dot_compute at /u/local/apps/julia/1.2.0/bin/../lib/julia/libopenblas64_.so (unknown line)
        Allocations: 3163 (Pool: 3154; Big: 9); GC: 0
        Illegal instruction

Note that the same Julia executable works on the AVX512-enabled CPUs as expected.


As the error message indicates, the problem is traced back to OpenBLAS. After
some research, it appears to be a problem of the AVX512 detection mechanism in
OpenBLAS which has been fixed after OpenBLAS 0.3.6, as stated in OpenBLAS
release notes (https://github.com/xianyi/OpenBLAS/releases), e.g.

- OpenBLAS 0.3.6: "the AVX512 DGEMM kernel has been disabled again due to unsolved problems"
- OpenBLAS 0.3.7: "the build-time logic for detection of AVX512 availability in the processor and compiler was fixed"
- OpenBLAS 0.3.8: "a new AVX512 DGEMM kernel was added and the AVX512 SGEMM kernel was significantly improved"

As of this writing, the OpenBLAS version shipped with Julia 1.4.2
(https://github.com/JuliaLang/julia/releases/download/v1.4.2/julia-1.4.2-full.tar.gz)
is OpenBLAS 0.3.5 but a patch ``openblas-skylakexdgemm.patch`` is available

.. code-block:: text

        $ cat deps/openblas.version
        OPENBLAS_BRANCH=v0.3.5
        OPENBLAS_SHA1=eebc18928715775c9ed254684edee16e4efe0342


We note that Julia 1.1.1 from Anaconda uses OpenBLAS 0.3.9 appears to be free
of this problem.  We performed two tests to confirm that issue comes from
OpenBLAS. 


Test 1
-----------------

We replaced the shared library file  of OpenBLAS 0.3.5
``libopenblas64_.0.3.5.so`` with OpenBLAS 0.3.9 ``libopenblasp-r0.3.9.so`` (and
changed the symbolic links ``libopenblas64_.so`` and ``libopenblas64_.so.0``
accordingly), the same Julia executable binary that exhibits the error message
above now works (but has an initialization error due to
``openblas_get_config64_`` because of the brute-force shared library file
replacement.

.. code-block:: text

        $ ./julia 
        WARNING: Error during initialization of module LinearAlgebra:
        ErrorException("could not load symbol "openblas_get_config64_":
        /tmp/tmp.v12ytpLZik/1.2.0/bin/../lib/julia/libopenblas64_.so: undefined symbol: openblas_get_config64_")
                       _
           _       _ _(_)_     |  Documentation: https://docs.julialang.org
          (_)     | (_) (_)    |
           _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
          | | | | | | |/ _` |  |
          | | |_| | | | (_| |  |  Version 1.2.0 (2019-08-20)
         _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
        |__/                   |

        julia> 



Test 2
----------------

Another test is performed by passing the environment varialbe
``OPENBLAS_CORETYPE`` to "fool" OpenBLAS that the CPU is not Skylake (e.g.
"haswell", as shown below) so it does not execute Skylake-specific functions
causing the crash. The impact on performance is left as a future exercise.

.. code-block:: bash

        $ julia
        Invalid instruction at 0x7f77ef7e3ba8: 0x62, 0xf1, 0x7d, 0x48, 0xef, 0xc0, 0xc3, 0x90, 0x89, 0xc2, 0x83, 0xe2, 0xe0, 0x0f, 0x8e

        signal (4): Illegal instruction
        in expression starting at none:0
        dot_compute at /u/local/apps/julia/1.2.0/bin/../lib/julia/libopenblas64_.so (unknown line)
        Allocations: 3162 (Pool: 3154; Big: 8); GC: 0
        Illegal instruction (core dumped)

.. code-block:: bash

        $ OPENBLAS_CORETYPE=haswell julia
                       _
           _       _ _(_)_     |  Documentation: https://docs.julialang.org
          (_)     | (_) (_)    |
           _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
          | | | | | | |/ _` |  |
          | | |_| | | | (_| |  |  Version 1.2.0 (2019-08-20)
         _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
        |__/                   |

        julia> 


Summary
---------------

On AVX512-enabled CPUs with AVX512 disabled, Julia could crash due to an
OpenBLAS issue. The issue can be fixed by using either a patch or a new
OpenBLAS version.

