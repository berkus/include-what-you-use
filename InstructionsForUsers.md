# Instructions for Users #

"Include what you use" means this: for every symbol (type, function, variable, or macro) that you use in foo.cc (or foo.cpp), either foo.cc or foo.h should #include a .h file that exports the declaration of that symbol. (Similarly, for foo\_test.cc, either foo\_test.cc or foo.h should do the #including.)  Obviously symbols defined in foo.cc itself are excluded from this requirement.

This puts us in a state where every file includes the headers it needs to declare the symbols that it uses.  When every file includes what it uses, then it is possible to edit any file and remove unused headers, without fear of accidentally breaking the upwards dependencies of that file.  It also becomes easy to automatically track and update dependencies in the source code.


## CAVEAT ##

This is alpha quality software -- at best (as of February 2011).  It was written to work specifically in the Google source tree, and may make assumptions, or have gaps, that are immediately and embarrassingly evident in other types of code.  For instance, we only run this on C++ code, not C or Objective C.  Even for Google code, the tool still makes a lot of mistakes.

While we work to get IWYU quality up, we will be stinting new features, and will prioritize reported bugs along with the many existing, known bugs.  The best chance of getting a problem fixed is to submit a patch that fixes it (along with a unittest case that verifies the fix)!


## How to Build ##

Include-what-you-use makes heavy use of Clang internals, and will occasionally break when Clang is updated. See the include-what-you-use Makefile for instructions on how to keep them in sync.

IWYU, like Clang, does not yet handle some of the non-standard constructs in Microsoft's STL headers. A discussion on how to use MinGW or Cygwin headers with IWYU is available on the [mailing list](https://groups.google.com/group/include-what-you-use/msg/a4d08378440edafd).

We support two build configurations: out-of-tree and in-tree.


### Building out-of-tree ###

In an out-of-tree configuration, we assume you already have compiled LLVM and Clang headers and libs somewhere on your filesystem, such as via the `libclang-dev` package. Out-of-tree builds are only supported with CMake (patches very welcome for the Make system).

  * Create a directory for IWYU development, e.g. `iwyu-trunk`
  * Get the IWYU source code, either from a [published tarball](http://code.google.com/p/include-what-you-use/downloads/list) or directly from [svn](http://code.google.com/p/include-what-you-use/source/checkout)
```
    # Unpack tarball
    iwyu-trunk$ tar xfz include-what-you-use-<version>.tar.gz

    # or checkout from SVN
    iwyu-trunk$ svn co http://include-what-you-use.googlecode.com/svn/trunk/ include-what-you-use
```
  * Create a build root
```
    iwyu-trunk$ mkdir build && cd build
```
  * Run CMake and specify the location of LLVM/Clang prebuilts
```
    # This example uses the Makefile generator,
    # but anything should work.
    iwyu-trunk/build$ cmake -G "Unix Makefiles" -DLLVM_PATH=/usr/lib/llvm-3.4 ../include-what-you-use
```
  * Once CMake has generated a build system, you can invoke it directly from `build`, e.g.
```
    iwyu-trunk/build$ make
```

This configuration is more useful if you want to get IWYU up and running quickly without building Clang and LLVM from scratch.


### Building in-tree ###

You will need the Clang and LLVM trees on your system, such as by [checking out](http://clang.llvm.org/get_started.html) their SVN trees (but don't configure or build before you've done the following.)

  * Put include-what-you-use in the source tree. Either [download](http://code.google.com/p/include-what-you-use/downloads/list) the include-what-you-use tarball and unpack it your `/path/to/llvm/tools/clang/tools` directory, or get the project directly from [svn](http://code.google.com/p/include-what-you-use/source/checkout):
```
    # Unpack tarball
    llvm/tools/clang/tools$ tar xfz include-what-you-use-<version>.tar.gz

    # or checkout from SVN
    llvm/tools/clang/tools$ svn co http://include-what-you-use.googlecode.com/svn/trunk/ include-what-you-use
```
  * Edit tools/clang/tools/Makefile and add include-what-you-use to the `DIRS` variable
  * Edit tools/clang/tools/CMakeLists.txt and `add_subdirectory(include-what-you-use)`
  * Once this is done, IWYU is recognized and picked up by both autoconf and CMake workflows as described in the Clang Getting Started guide

This configuration is more useful if you're actively developing IWYU against Clang trunk.


## How to Install ##

If you're building IWYU out-of-tree or installing pre-built binaries, you need to make sure it can find Clang built-in headers (`stdarg.h` and friends.)

Clang's default policy is to look in `path/to/clang-executable/../lib/clang/<clang ver>/include`. So if Clang 3.5.0 is installed in `/usr/bin`, it will search for built-ins in `/usr/lib/clang/3.5.0/include`.

Clang tools have the same policy by default, so in order for IWYU to analyze any non-trivial code, it needs to find Clang's built-ins in `path/to/iwyu/../lib/clang/3.5.0/include` where `3.5.0` is a stand-in for the version of Clang your IWYU was built against.

This weirdness is tracked in [issue 100](https://code.google.com/p/include-what-you-use/issues/detail?id=100), hopefully we can eliminate the manual patching.


## How to Run ##

The easiest way to run IWYU over your codebase is to run
```
  make -k CXX=/path/to/llvm/Debug+Asserts/bin/include-what-you-use
```
or
```
  make -k CXX=/path/to/llvm/Release/bin/include-what-you-use
```
(include-what-you-use always exits with an error code, so the build system knows it didn't build a .o file.  Hence the need for `-k`.)

We also include, in this directory, a tool that automatically fixes up your source files based on the iwyu recommendations.  This is also alpha-quality software!  Here's how to use it (requires python):
```
  make -k CXX=/path/to/llvm/Debug+Asserts/bin/include-what-you-use > /tmp/iwyu.out
  python fix_includes.py < /tmp/iwyu.out
```
If you don't like the way fix\_includes.py munges your #include lines, you can control its behavior via flags. `fix_includes.py --help` will give a full list, but these are some common ones:

  * `-b`: Put blank lines between system and Google #includes
  * `--nocomments`: Don't add the 'why' comments next to #includes

WARNING: include-what-you-use only analyzes .cc (or .cpp) files built by `make`, along with their corresponding .h files.  If your project has a .h file with no corresponding .cc file, iwyu will ignore it. include-what-you-use supports the AddGlobToReportIWYUViolationsFor()` function which can be used to indicate other files to analyze, but it's not currently exposed to the user in any way.


## How to Correct IWYU Mistakes ##

  * If fix\_includes.py has removed an #include you actually need, add it back in with the comment '`// IWYU pragma: keep`' at the end of the #include line.  Note that the comment is case-sensitive.
  * If fix\_includes has added an #include you don't need, just take it out.  We hope to come up with a more permanent way of fixing later.
  * If fix\_includes has wrongly added or removed a forward-declare, just fix it up manually.
  * If fix\_includes has suggested a private header file (such as `<bits/stl_vector.h>`) instead of the proper public header file (`<vector>`), you can fix this by inserting a specially crafted comment near top of the private file (assuming you can write to it): '`// IWYU pragma: private, include "the/public/file.h"`'.

All current IWYU pragmas (as of July 2012) are described in [IWYUPragmas](IWYUPragmas.md).