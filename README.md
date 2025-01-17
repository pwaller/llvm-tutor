llvm-tutor
=========
[![Build Status](https://travis-ci.org/banach-space/llvm-tutor.svg?branch=master)](https://travis-ci.org/banach-space/llvm-tutor)

Example LLVM passes - based on LLVM-8

**llvm-tutor** (a.k.a **lt**) is a collection of self-contained reference LLVM
passes developed as a tutorial.  It targets novice and aspiring LLVM
developers. It strives to be:
  * **Complete:** There's a functional `CMakeLists.txt` and CI that provides a
    working reference set-up.
  * **Out of source:** It builds against a binary LLVM installation.
  * **Modern:** It's based on the latest version of LLVM -  you won't get stuck
    trying to download/build an old version of LLVM or learning an outdated API.

There's a CI set-up for this project so both you and I can be confident that
the examples do build and work fine. I've also tried to leave plenty of
comments everywhere (in source files, build scripts and tests) - both for my
future self as well as you. I encourage you to study and modify the code.

tl;dr
-----
The best place to start is the `lt` pass implemented in
`StaticCallCounter.cpp`. Build it first:
```
$ cmake -DLT_LLVM_INSTALL_DIR=<either_build_or_installation_dir_of_llvm_8>  <source_dir>
```
and then run:
```
$ opt -load <build_dir>/lib/liblt-lib.so --lt -analyze <bitcode-file>
```

Status
------
This is still **WORK IN PROGRESS**.

The list of currently available passes:
   * direct call counter: static and dynamic calls

Requirements
------------
These are the only requirement for **llvm-tutor**:
  * development version of LLVM-8
  * C++ compiler that supports C++14
  * CMake 3.4.3 or higher

Keep in mind that you only need the components required for developing
LLVM-based projects, e.g. header files, development libraries, CMake scripts.
Installing `clang-8` (and other LLVM-based tools) won't hurt, but is neither
required nor sufficient. There are additional requirements to be able to run
tests, which are documented [here](#test_requirements).

### Obtaining LLVM-8
Basically, there are two options here: either build from sources (works
regardless of the operating system, but is slow and complex) or download an
installation package (very quick and easy, but works only on systems for which
there are LLVM package maintainers).

#### Installing on Ubuntu
If you're using `Ubuntu` then install `llvm-8-dev` (other dependencies will be
_pulled_ automatically). Note that this very recent version of LLVM is not yet
available in the official repositories. On `Ubuntu Xenial`, you can [install
LLVM](https://blog.kowalczyk.info/article/k/how-to-install-latest-clang-6.0-on-ubuntu-16.04-xenial-wsl.html)
from the official [repository](http://apt.llvm.org/):
```bash
$ wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -
$ sudo apt-add-repository "deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial-8.0 main"
$ sudo apt-get update
$ sudo apt-get install -y llvm-8-dev
```
Note that this won't install the dependencies required for [testing](#testing).
Ubuntu maintainers have stopped those them starting with LLVM-3.8 (more info
[here](https://bugs.launchpad.net/ubuntu/+source/llvm-toolchain-3.8/+bug/1700630)).
In order to run LIT tests you will have to build the dependencies from sources.
However, you don't need them to be able to build and run the passes developed
here.

#### Installing on Mac OS X
On Darwin you can install LLVM-8 with [Homebrew](https://brew.sh/):
```bash
$ brew install llvm@8
```
This will install all the required header files, libraries and binaries in
`/usr/local/opt/llvm/`. Currently this will also install the binaries required
for testing.

#### Build From Sources
You can also choose to [build LLVM](https://llvm.org/docs/CMake.html) from
sources. It might be required if there are no precompiled packages for your
OS.

Platform Support
----------------
This project is currently being tested on Linux and Mac OS X. It should build
and run seamlessly on Windows, though some tweaks in CMake might be required.
It is regularly tested against the following configurations (extracted from the
CI files: `.travis.yml`):
  * Linux Ubuntu 16.04 (LLVM-7)
  * Mac OS X 10.14.4 (AppleClang 11)

Locally I used GCC-8.2.1 and LLVM-7 for development (i.e. for compiling). Please
refer to the CI logs (links at the top of the page) for reference setups.

Build Instructions
------------------
It is assumed that **llvm-tutor** will be built in `<build-dir>` and that the
top-level source directory is `<source-dir>`.

You can build all the examples as follows:
```bash
$ cd <build-dir>
$ cmake -DLT_LLVM_INSTALL_DIR=<either_build_or_installation_dir_of_llvm_8>  <source_dir>
$ make
```

The `LT_LLVM_INSTALL_DIR` variable should be set to either the installation or
build directory of LLVM-8. It is used to locate the corresponding
`LLVMConfig.cmake` script.

Usage
-----
Once you've [built](#build-instructions) this project, you can verify every
pass individually.

### Counting Function Calls
The `lt-cc` executable implements two basic direct call counters: static, and
dynamic. You can test it with one of the provided examples, e.g.:
```bash
$ clang  -emit-llvm -c <source_dir>/test/example_1.c
$ <build_dir>/bin/lt-cc -static example_1.bc
```

or, for dynamic call analysis:
```bash
$ <build_dir>/bin/lt-cc  -dynamic  example_1.bc -o example_1
$ ./example_1
```

Testing
-------
There's a handful of LIT tests added to this project. These tests are meant to
prevent any future breakage in **llvm-tutor**. They also serve as reference for
setting them up in the future (i.e. you will also find here LIT configuration
scripts which are part of the required set-up).

### Test Requirements

Before running the tests you need to make sure that you have the following
tools installed:
  * [**FileCheck**](https://llvm.org/docs/CommandGuide/lit.html) (LIT
    requirement, it's used to check whether tests generate the expected output)
  * [**opt**](http://llvm.org/docs/CommandGuide/opt.html) (the modular LLVM
    optimizer and analyzer, used to load and run passes from shared objects)
  * [**lit**](https://llvm.org/docs/CommandGuide/lit.html) (LLVM tool for executing
    the tests)

Neither of the requirements are satisfied on Ubuntu Xenial - sadly there are no
packages that would provide `opt` or `FileCheck` for LLVM-8.0. Although older
versions are available (e.g. bundled with LLVM-4.0), this project has been
developed against LLVM-8.0 and ideally you want a matching version of both
`opt` and `FileCheck`.

### Running The Tests
First, you will have to specify the location of the tools required for
running the tests (e.g. `FileCheck`). This is done when configuring the
project:

```bash
$ cmake -DLT_LLVM_INSTALL_DIR=<either_build_or_installation_dir>
-DLT_LIT_TOOLS_DIR=<location_of_filecheck> <source_dir>
```
Next, you can run the tests like this:
```bash
$ lit <build_dir>/test
```
Voilà! (well, assuming that `lit` is in your _path_).

### Wee disclaimer
The requirements for running LIT tests are _not_ met on Ubuntu (unless you
build LLVM from sources) and for this reason the CI is configured to skip tests
there. As far as Linux is concerned, I've only been able to run the tests on my
host. If you encounter any problems on your machine - please let me know and I
will try to fix that. The CI _does_ run the tests when building on Mac OS X.

License
--------
The MIT License (MIT)

Copyright (c) 2019 Andrzej Warzyński

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
