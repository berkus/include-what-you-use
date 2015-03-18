"Include what you use" means this: for every symbol (type, function variable, or macro) that you use in foo.cc, either foo.cc or foo.h should #include a .h file that exports the declaration of that symbol.  The include-what-you-use tool is a program that can be built with the clang libraries in order to analyze #includes of source files to find include-what-you-use violations, and suggest fixes for them.

The main goal of include-what-you-use is to remove superfluous #includes.  It does this both by figuring out what #includes are not actually needed for this file (for both .cc and .h files), and replacing #includes with forward-declares when possible.

Recent news:

## 25 November 2014 ##

iwyu 0.3 compatible with llvm+clang 3.5 is released. In this version we have
  * Added rudimentary support for C code.
  * Improved MSVC support for templated code and precompiled headers.
  * Added support for public STL #includes, which improves the IWYU experience for libc++ users.

For the full list of closed issues see https://code.google.com/p/include-what-you-use/issues/list?q=closed-after%3A2014%2F02%2F23&can=1

The source code can be downloaded from [include-what-you-use-3.5.src.tar.gz](http://include-what-you-use.com/downloads/include-what-you-use-3.5.src.tar.gz). It is equivalent to `clang_3.5` tag.

## 22 February 2014 ##

iwyu version compatible with llvm+clang 3.4 is released. The source code can be downloaded from [include-what-you-use-3.4.src.tar.gz](http://include-what-you-use.com/downloads/include-what-you-use-3.4.src.tar.gz).  It is equivalent to `clang_3.4` tag.

## 11 August 2013 ##

We are moving downloads to Google Drive. iwyu version compatible with llvm+clang 3.3 can be found at [include-what-you-use-3.3.tar.gz](https://docs.google.com/file/d/0ByBfuBCQcURXQktsT3ZjVmZtWkU/edit). It is equivalent to `clang_3.3` tag.

### 6 December 2011 ###

Now that clang 3.0 is out, I released a version of iwyu that works against clang 3.0.  It is equivalent to [r330](https://code.google.com/p/include-what-you-use/source/detail?r=330).  It is available in the 'downloads' section on the side pane.  To use, just `cd` to `llvm/tools/clang/tools` in your llvm/clang tree, and untar `include-what-you-use-3.0-1.tar.gz` from that location.  Then cd to `include-what-you-use` and type `make`.  (A cmakefile is also available.)  You can run `make check-iwyu` to run the iwyu test suite.

### 24 June 2011 ###

It was just pointed out to me the tarball I built against llvm+clang 2.9 doesn't actually compile with llvm+clang 2.9.  I must have made a mistake packaging it.  I've tried again; according to my tests, anyway, the new version works as it's supposed to.

### 8 June 2011 ###

I finally got around to releasing a tarball that builds against llvm+clang 2.9.  See the 'downloads' section on the side pane.  This is a rather old version of iwyu at this point, so you'll do much better to download a current clang+llvm and the svn-root version of include-what-you-use, and build from that.  See [README.txt](http://code.google.com/p/include-what-you-use/source/browse/trunk/README.txt?r=260) for more details.

## 13 April 2011 ##

Work has been continuing at a furious pace on include-what-you-use.  It's definitely beta quality by now. :-)  Well, early beta.  I've not been making regular releases, but the SVN repository is being frequently updated, so don't take the lack of news here to mean a lack of activity.

### 4 February 2011 ###

I'm very pleased to announce the very-alpha, version 0.1 release of include-what-you-use.  See the wiki links on the right for instructions on how to download, install, and run include-what-you-use.

I'm releasing the code as it is now under a "release early and often" approach.  It's still very early in iwyu, and the program will probably have mistakes on any non-trivial piece of code.  Furthermore, it still has google-specific bits that may not make much sense in an opensource release.  This will all get fixed over time.  Feel free to dig in and suggest patches to help the fixing along!

If you want to follow the discussion on include-what-you-use, and/or keep up to date with changes, subscribe to the
[Google Group](http://groups.google.com/group/include-what-you-use).