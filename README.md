# UCB-HLS BUILD NOTES

* Clone our llvm fork
* Checkout branch `release_37`
* Copy in the clang 3.7.0 release source from the tarball to tools/clang
* Clone this repo into the `projects` directory

```
export TOP=$(pwd)
wget http://releases.llvm.org/3.7.0/cfe-3.7.0.src.tar.xz
git clone git@github.com:ucb-hls/llvm.git llvm-37
export LLVM_37=${TOP}/llvm-37
cd ${LLVM_37}
git checkout release_37
cd tools
tar -xvf ${TOP}/cfe-3.7.0.src.tar.xz
mv cfe-3.7.0.src clang
cd ../projects
git clone git@github.com:ucb-hls/llvm-cbe.git
cd ${TOP}
mkdir llvm-37-build
cd !$
cmake -GNinja $LLVM_37}
ninja
```

llvm-cbe
========

resurrected LLVM "C Backend", with improvements


INSTALLATION INSTRUCTIONS
=========================

This version of the LLVM-CBE library works with LLVM 3.7. You will have to
compile this version of LLVM before you try to use LLVM-CBE. This
guide will walk you through the compilation and installation of both
tools and show usage statements to verify that the LLVM-CBE library is
compiled correctly.

The library is known to compile on various Linux versions (Redhat,
Mageia, Ubuntu, Debian), Mac OS X, and Windows (Mingw-w64).

Step 1: Installing LLVM
=======================

LLVM-CBE relies on specific LLVM internals, and so it is best to use
it with a specific revision of the LLVM development tree. Currently,
llvm-cbe works with the LLVM 3.7 release versions and autotools.

Note: to convert C to LLVM IR to run the tests, you will also need a C compiler such as clang.

The first step is to compile LLVM on your machine
(this assumes an in-tree build, but out-of-tree will also work):

     cd $HOME
     git clone https://github.com/llvm-mirror/llvm
     cd llvm
     git checkout release_37
     ./configure
     make

Step 2: Compiling LLVM-CBE
==========================

Next, download and compile llvm-cbe from the same folder:

    cd $HOME/llvm/projects
    git clone https://github.com/JuliaComputing/llvm-cbe.git llvm-cbe
    cd ..
    make

Step 3: Usage Examples
======================

If llvm-cbe compiles, you should be able to run it with the following commands.
```
$ cd llvm/lib/test/selectionsort
$ ls
main.c
$ clang -S -emit-llvm main.c
$ ls
main.c main.ll
$ $(HOME)/llvm/Release/bin/llvm-cbe main.ll

```
Compile Generated C-Code and Run
================================
```
$ gcc -o main.cbe main.cbe.c
$ ls
main.c  main.cbe  main.cbe.c  main.ll
$ ./main.cbe
```
