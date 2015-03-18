# Instructions for Developers #

## Submitting Patches ##

We're still working this part out.  For now, you can create patches against svn-head and submit them as [new issues](http://code.google.com/p/include-what-you-use/issues/list).  Probably, we'll move to a scheme where people can submit patches directly to the SVN repository.


## Running the Tests ##

If fixing a bug in clang, please add a test to the test suite!  You
can create a file called `whatever.cc` (_not_ .cpp), and, if necessary, `whatever.h`, and `whatever-<extension>.h`.  You may be able to get away without adding any `.h` files, and just #including `direct.h` -- see, for instance, `tests/remove_fwd_decl_when_including.cc`.

To run the iwyu tests, run
```
  python run_iwyu_tests.py
```
It runs one test for each `.cc` file in the `tests/` directory.  (We have additional tests in `more_tests/`, but have not yet gotten the testing framework set up for those tests.)  The output can be a bit hard to read, but if a test fails, the reason why will be listed after the
`ERROR:root:Test failed for xxx` line.

When fixing `fix_includes.py`, add a test case to `fix_includes_test.py` and run
```
  python fix_includes_test.py
```

## Debugging ##

It's possible to run `include-what-you-use` in `gdb`, to debug that way.
Another useful tool -- especially in combination with `gdb` -- is to get
the verbose include-what-you-use output.  See `iwyu_output.h` for a
description of the verbose levels.  Level 7 is very verbose -- it
dumps basically the entire AST as it's being traversed, along with
iwyu decisions made as it goes -- but very useful for that:
```
  env IWYU_VERBOSE=7 make -k CXX=/path/to/llvm/Debug+Asserts/bin/include-what-you-use 2>&1 > /tmp/iwyu.verbose
```

## A Quick Tour of the Codebase ##

The codebase is strewn with TODOs of known problems, and also language constructs that aren't adequately tested yet.  So there's plenty to do!  Here's a brief guide through the codebase:

  * `iwyu.cc`: the main file, it includes the logic for deciding when a symbol has been 'used', and whether it's a full use (definition required) or forward-declare use (only a declaration required).  It also includes the logic for following uses through template instantiations.
  * `iwyu_driver.cc`: responsible for creating and configuring a Clang compiler from command-line arguments.
  * `iwyu_output.cc`: the file that translates from 'uses' into iwyu violations.  This has the logic for deciding if a use is covered by an existing #include (or is a built-in).  It also, as the name suggests, prints the iwyu output.
  * `iwyu_preprocessor.cc`: handles the preprocessor directives, the `#includes` and `#ifdefs`, to construct the existing include-tree.  This is obviously essential for include-what-you-use analysis.  This file also handles the iwyu pragma-comments.
  * `iwyu_include_picker.cc`: this finds canonical #includes, handling private->public mappings (like `bits/stl_vector.h` -> `vector`) and symbols with multiple possible #includes (like `NULL`). Mappings are maintained in a set of .imp files separately, for easier per-platform/-toolset customization.
  * `iwyu_cache.cc`: holds the cache of instantiated templates (may hold other cached info later).  This is data that is expensive to compute and may be used more than once.
  * `iwyu_globals.cc`: holds various global variables.  We used to think globals were bad, until we saw how much having this file simplified the code...
  * `iwyu_*_util(s).h` and `.cc`: utility functions of various types.  The most interesting, perhaps, is `iwyu_ast_util.h`, which has routines  that make it easier to navigate and analyze the clang AST.  There are also some STL helpers, string helpers, filesystem helpers, etc.
  * `iwyu_verrs.cc`: debug logging for iwyu.
  * `port.h`: shim header for various non-portable constructs.
  * `iwyu_getopt.cc`: portability shim for GNU getopt(_long). Custom getopt(_long) implementation for Windows.
  * `fix_includes.py`: the helper script that edits a file based on the iwyu recommendations.