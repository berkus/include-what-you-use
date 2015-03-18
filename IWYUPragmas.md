# IWYU pragmas #

IWYU pragmas are used to give IWYU information that isn't obvious from the source code, such as how different files relate to each other and which #includes to never remove or include.

All pragmas start with `"// IWYU pragma: "` or `"/* IWYU pragma: "`. They are case-sensitive and spaces are significant.


## IWYU pragma: keep ##

This pragma applies to a single `#include` statement. It forces IWYU to keep an inclusion even if it is deemed unnecessary.
```
main.cc:
  #include <vector> // IWYU pragma: keep
```
In this case, `std::vector` isn't used, so `<vector>` would normally be discarded, but the pragma instructs IWYU to leave it.


## IWYU pragma: export ##

This pragma applies to a single `#include` statement. It says that the current file is to be considered the provider of any symbol from the included file.
```
facade.h:
  #include "detail/constants.h" // IWYU pragma: export
  #include "detail/types.h" // IWYU pragma: export
  #include <vector> // don't export stuff from <vector>

main.cc:
  #include "facade.h"

  // Assuming Thing comes from detail/types.h and MAX_THINGS from detail/constants.h
  std::vector<Thing> things(MAX_THINGS);
```
Here, since `detail/constants.h` and `detail/types.h` have both been exported, IWYU is happy with the `facade.h` include for `Thing` and `MAX_THINGS`.

In contrast, since `<vector>` has not been exported from `facade.h`, it will be suggested as an additional include.


## IWYU pragma: begin\_exports/end\_exports ##

This pragma applies to a set of `#include` statements. It declares that the including file is to be considered the provider of any symbol from these included files. This is the same as decorating every `#include` statement with `IWYU pragma: export`.
```
facade.h:
  // IWYU pragma: begin_exports
  #include "detail/constants.h"
  #include "detail/types.h"
  // IWYU pragma: end_exports 

  #include <vector> // don't export stuff from <vector>
```

## IWYU pragma: private ##

This pragma applies to the current header file. It says that any symbol from this file will be provided by another, optionally named, file.
```
private.h:
  // IWYU pragma: private, include "public.h"
  struct Private {};

private2.h:
  // IWYU pragma: private
  struct Private2 {};

public.h:
  #include "private.h"
  #include "private2.h"

main.cc:
  #include "private.h"
  #include "private2.h"

  Private p;
  Private2 i;
```
Using the type `Private` in `main.cc` will cause IWYU to suggest that you include `public.h`.

Using the type `Private2` in `main.cc`  will cause IWYU to suggest that you include `private2.h`, but will also result in a warning that there's no public header for `private2.h`.


## IWYU pragma: no\_include ##

This pragma applies to the current source file. It declares that the named file should not be suggested for inclusion by IWYU.
```
private.h:
  struct Private {};

unrelated.h:
  #include "private.h"
  ...

main.cc:
  #include "unrelated.h"
  // IWYU pragma: no_include "private.h"

  Private i;
```
The use of `Private` requires including `private.h`, but due to the `no_include` pragma IWYU will not suggest `private.h` for inclusion. Note also that if you had included `private.h` in `main.cc`, IWYU would suggest that the `#include` be removed.

This is useful when you know a symbol definition is already available via some unrelated header, and you want to preserve that implicit dependency.

The `no_include` pragma is somewhat similar to `private`, but is employed at point of use rather than at point of declaration.


## IWYU pragma: no\_forward\_declare ##

This pragma applies to the current source file. It says that the named symbol should not be suggested for forward-declaration by IWYU.
```
public.h:
  struct Public {};

unrelated.h:
  struct Public;
  ...

main.cc:
  #include "unrelated.h" // declares Public
  // IWYU pragma: no_forward_declare Public

  Public* i;
```
IWYU would normally suggest forward-declaring `Public` directly in `main.cc`, but `no_forward_declare` suppresses that suggestion. A forward-declaration for `Public` is already available from `unrelated.h`.

This is useful when you know a symbol declaration is already available in a source file via some unrelated header and you want to preserve that implicit dependency, or when IWYU does not correctly understand that the definition is necessary.


## IWYU pragma: friend ##

This pragma applies to the current header file. It says that any file matching the given regular expression will be considered a friend, and is allowed to include this header even if it's private. Conceptually similar to `friend` in C++.

If the expression contains spaces, it must be enclosed in quotes.
```
detail/private.h:
  // IWYU pragma: private
  // IWYU pragma: friend "detail/.*"
  struct Private {};

detail/alsoprivate.h:
  #include "detail/private.h"

  // IWYU pragma: private
  // IWYU pragma: friend "main\.cc"
  struct AlsoPrivate : Private {};

main.cc:
  #include "detail/alsoprivate.h"

  AlsoPrivate p;
```

## Which pragma should I use? ##

Ideally, IWYU should be smart enough to understand your intentions (and intentions of the authors of libraries you use), so the first answer should always be: none.

In practice, intentions are not so clear -- it might be ambiguous whether an `#include` is there by clever design or by mistake, whether an `#include` serves to export symbols from a private header through a public facade or if it's just a left-over after some clean-up. Even when intent is obvious, IWYU can make mistakes due to bugs or not-yet-implemented policies.

IWYU pragmas have some overlap, so it can sometimes be hard to choose one over the other. Here's a guide based on how I understand them at the moment:

  * Use `IWYU pragma: keep` to force IWYU to keep any `#include` statement that would be discarded under its normal policies.
  * Use `IWYU pragma: export` to tell IWYU that one header serves as the provider for all symbols in another, included header (e.g. facade headers). Use `IWYU pragma: begin_exports/end_exports` for a whole group of included headers.
  * Use `IWYU pragma: no_include` to tell IWYU that the file in which the pragma is defined should never `#include` a specific header (the header may already be included via some other `#include`.)
  * Use `IWYU pragma: no_forward_declare` to tell IWYU that the file in which the pragma is defined should never forward-declare a specific symbol (a forward declaration may already be available via some other `#include`.)
  * Use `IWYU pragma: private` to tell IWYU that the header in which the pragma is defined is private, and should not be included directly.
  * Use `IWYU pragma: private, include "public.h"` to tell IWYU that the header in which the pragma is defined is private, and `public.h` should always be included instead.
  * Use `IWYU pragma: friend ".*favorites.*"` to override `IWYU pragma: private` selectively, so that a set of files identified by a regex can include the file even if it's private.

The pragmas come in three different classes;

  1. Ones that apply to a single `#include` statement (keep, export)
  1. Ones that apply to a file being included (private, friend)
  1. Ones that apply to a file including other headers (no\_include, no\_forward\_declare)

Some files are both included and include others, so it can make sense to mix and match.