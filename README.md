# UCB-HLS Build Notes

Source: [Julia's CBE revival](https://github.com/JuliaComputing/llvm-cbe)

The C backend resurrected by Julia works with LLVM release 3.7.0 (and clang 3.7.0/3.7.1) but not with HEAD.

Download release 3.7.0 tarballs for [LLVM](http://releases.llvm.org/3.7.0/llvm-3.7.0.src.tar.xz) and [clang](http://releases.llvm.org/3.7.0/cfe-3.7.0.src.tar.xz) [here](http://releases.llvm.org/download.html#3.7.0).

Steps:
* Clone our llvm fork
* Checkout branch `release_37`
* Copy in the clang 3.7.0 release source from prepackaged source tarballs to tools/clang
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

More notes:
* `#include <APInt-C.h>` is added to generated code by default; we can fetch it we need: https://github.com/JuliaComputing/llvm-cbe/issues/1

Use:
1. `${TOP}/llvm-37-build/bin/llvm-cbe <go_source_file_name>.s`
2. This will generate `<go_source_file_name>.s.cbe.s`. Edit this and remote the `#include <APInt-C.h>` - or find the file and add it to the include path, seeing note above.

### Compilation weirdness

I ran into this problem when compiling on a new 32-core AWS instance:

```
ubuntu@ip-172-31-28-222:~/llvm-37-build$ ninja
[277/2419] Building CXX object projects/llvm-cbe/lib/Target/CBackend/CMakeFiles/LLVMCBackendCodeGen.dir/CBackend.cpp.o
FAILED: /usr/bin/c++   -DGTEST_HAS_RTTI=0 -D_GNU_SOURCE -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS -Iprojects/llvm-cbe/lib/Target/CBackend -I/home/ubuntu/llvm-37/projects/llvm-cbe/lib/Target/CBackend -Iinclude -I/home/ubuntu/llvm-37/include -fPIC -fvisibility-inlines-hidden -Wall -W -Wno-unused-parameter -Wwrite-strings -Wcast-qual -Wno-missing-field-initializers -pedantic -Wno-long-long -Wno-maybe-uninitialized -Wnon-virtual-dtor -Wno-comment -std=c++11 -ffunction-sections -fdata-sections   -fno-exceptions -fno-rtti -MMD -MT projects/llvm-cbe/lib/Target/CBackend/CMakeFiles/LLVMCBackendCodeGen.dir/CBackend.cpp.o -MF projects/llvm-cbe/lib/Target/CBackend/CMakeFiles/LLVMCBackendCodeGen.dir/CBackend.cpp.o.d -o projects/llvm-cbe/lib/Target/CBackend/CMakeFiles/LLVMCBackendCodeGen.dir/CBackend.cpp.o -c /home/ubuntu/llvm-37/projects/llvm-cbe/lib/Target/CBackend/CBackend.cpp
In file included from /home/ubuntu/llvm-37/include/llvm/CodeGen/IntrinsicLowering.h:19:0,
                 from /home/ubuntu/llvm-37/projects/llvm-cbe/lib/Target/CBackend/CBackend.h:7,
                 from /home/ubuntu/llvm-37/projects/llvm-cbe/lib/Target/CBackend/CBackend.cpp:15:
/home/ubuntu/llvm-37/include/llvm/IR/Intrinsics.h:40:34: fatal error: llvm/IR/Intrinsics.gen: No such file or directory
compilation terminated.
[277/2419] Building CXX object utils/unittest/CMakeFiles/gtest.dir/googletest/src/gtest-all.cc.o
ninja: build stopped: subcommand failed.
```

But limiting ninja to use 12 jobs fixed it:
```
ubuntu@ip-172-31-28-222:~/llvm-37-build$ ninja -j 12
[2419/2419] Creating executable symlink bin/clang
```

I assume this is because some dependency isn't correctly computed.


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
