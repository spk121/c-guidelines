# C++ Core Guidelines Remix for Mike's C Projects

04 Jan 2024

This is my remix of the
[C++ Core Guidelines](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md) that are edited
by Bjarne Stroustrop and Herb Sutter.  It is a mixture
of that text plus some of my opinions.
This remix is tailored for the C code I write
for my customers. 

To wit, the original C++ Core Guidelines are released
under a MIT-style license, as is this remix.  See the
LICENSE file

Much of this is verbatim or adapted from that document.

# Abstract

The purpose of me creating my own tailored document
from the C++ Core Guidelines specifically for the C
code I write professionally. It is just to keep me coding
and prevent me from pondering the right way to do things.
In this doc, I'll lay out a strategy and stick to it,
and I'll stop recreating and reinventing ways to
write C.

# I: Interfaces

An interface is a contract between two parts of a program.
Precisely stating what is expected of a supplier of a
service and a user of that service is essential.
Having good (easy-to-understand, encouraging efficient use,
not error-prone, supporting testing, etc.) interfaces
is probably the most important single aspect of code organization.

### I.1: Make input interfaces explicit

A function should not make control-flow decisions based
on the values of module or global-level variables.

An exception can be made for environment variables that
indicates logging levels or other details that do not
change the details of the operation.

##### Example, bad

Controlling the behavior of a function through a global
variable is implicit and potentially confusing.  For example:

    int round(double d)
    {
       // don't: "invisible" dependency
      return (round_up) ? ceil(d) : d;
    }

### I.1.1: Make output interfaces explicit

_Not valid for GR._

A function should avoid writing to global variables.

In other words, avoid passing information across an interface
through non-local or implicit state.

##### Example, bad

Reporting through non-local variables (e.g., `errno`) is
easily ignored.

### I.2: Avoid non-`const`` global variables

_Not valid for GR._

Non-`const` global variables hide dependencies and make the
dependencies subject to unpredictable changes.  Prefer passing
the data as a struct via a `const *`.

### I.3: Avoid singletons

_Not valid for GR._

Singletons are structs for which only one object is created.

_How do explain why singletons are to be avoided?_

### I.4: Make interfaces precisely and strongly typed

Types are the simplest and best documentation, improve
legibility due to their well-defined meaning, and are checked
at compile time. Also, precisely typed code is often optimized
better.

##### Example, bad

    void func(void *data); // weak and underqualified void *

##### Example, bad

Consider:

    draw_rect(100, 200, 100, 500); // what do numbers mean?

It is clear that this is a rectangle, but not clear if these
are points are lengths.

Better would be

    void draw_rect(Point top_left, Size height_width);

##### Example, bad

Consider:

    set_settings(true, false, 42); // what do these specify

This is more explicit

    AlarmSettings s;
    s.enabled = true;
    s.daily = false;
    s.volume = 42;
    setSettings (s);

For the case of a set of boolean values consider using a flags
`enum`; a pattern that expresses a set of boolean values.

    enum {
      lamp_off, lamp_on
    } LampState;



### I.10: Use exceptions to signal a failure to perform a required task

_Not valid for GR._

We have no specific exceptions in C, but, the notion is
that it should not be possible to ignore an error because
that could leave the system or a computation in an
undefined (or unexpected) state.

One workaround is to return an error code and use C compiler
features to warn if the return value is untested.

### I.11: Never transfer ownership by a raw pointer (T*)

Every object passed as a raw pointer is assumed to be owned
by the caller, so that its lifetime is handled by the caller.

This is hard to stick to in C. For small structs, you can
return the whole struct, but, constructor functions that
allocate and populate structures are common in C.

One could create an OWNER(x) macro to decorate the
return value type, perhaps.

    OWNER(X*) compute(int n);

Let's say
- Prefer to return structs by value, if they are not too large
- Otherwise, use the OWNER(x) macro

### I.12: Declare a pointer that must not be null as `NOT_NULL`

To help avoid dereferencing `NULL` pointers and to avoid
redundant checks for null pointers.

Example:

    int length(const char *p);  // not clear is null pointer is valid

    length(NULL);  // is this okay?

    // Func can assume that argument is not null
    int length(NOT_NULL(const char*) p);

    // Func must assume that p can be NULL
    int length(const char *p);

### I.13: Do not pass an array as a single pointer

(pointer, size)-style interfaces are error-prone.  Also,
a plain pointer (to array) must rely on some convention
to allow the callee to determine the size.

##### Example:

Consider:

    void copy_n(const int *p, int *q, int n);

Getting `n` wrong or misordering `p` and `q` could create
a nasty bug.

Consider using explicit spans:

  struct {
    int x[10];
    int len = 10;
  }

would seem odd.  But I like the idea of it.

### I.22: Avoid complex initialization of global objects

Sure. Concretely, this says to be suspicious of initializing
globals with non-constants.

### I.23: Keep the number of function arguments low

Too many parameters may be

1. Missing an abstraction. Several individual elements
   are actually a compound value.
2. Violating 'one function, one responsibility'

### I.24: Avoid adjacent parameters that can be invoked by the same arguments in either order with different meaning

Basically, functions like `memcpy` with their `to` and `from`
arguments next to one another.

Could avoid this by making a struct that encapsulates
the arguments

# F: Functions

A function specifies an action or a computation that takes the
system from one consistent state to the next. It is the
fundamental building block of programs.

It should be possible to name a function meaningfully, to
specify the requirements of its argument, and clearly state
the relationship between the arguments and the result. An
implementation is not a specification. Try to think about what
a function does as well as about how it does it. Functions are
the most critical part in most interfaces, so see the
interface rules.

### F.1: Package meaningful operations into carefully named functions

A function should be a well-specified action.

### F.2: A function should perform a single logical operation

A function that performs a single operation is simpler to
understand, test, and reuse.

Consider functions with more than one "out" parameter
suspicious. Try to use return values, including `tuple`
for multiple return values.

Consider "large" functions that don't fit on one editor screen
suspicious and in need of refactoring.

Consider functions with 7 or more parameters suspicious.

### F.3: Keep functions short and simple

Large functions are hard to read, more likely to contain complex code, and more likely to have variables in larger than minimal scopes. Functions with complex control structures are more likely to be long and more likely to hide logical errors.

### F.5: If a function is very small and time-critical: declare it inline

### F.8: Prefer pure functions

Pure functions are easier to reason about, sometimes easier to
optimize (and even parallelize), and sometimes can be memoized.

### F.10: If an operation can be reused, give it a name

If a function has logic to perform multiple operations,
try to extract useful functions from these operations.

### F.15: Prefer simple and conventional ways of passing information

|--------|---------------|---------------|-------------------|
|        | Cheap to copy | Cheap to Move | Expensive to Move |
|--------|---------------|---------------|-------------------|
| Out    | X f()         | X f()         | f(X *x)           |
| In/Out | f(X *x)       | f(X *x)       | f(X *x)           |
| In     | f(X x)        | f(const X *x) | f(const X *x)     |

### F.16: For "in" parameters, pass cheaply-copied types by value and others as `const *`

### F.17: For "in-out" parameters, pass as non-const pointer

### F.20: For "out" output values, prefer return values to output parameters

### F.21: To return multiple 'out' values, prefer returning a struct or tuple

But this is tough for functions that return a value and
an error code.  Could define tuples, but, returning
structs by value is also not very C-like.

### F.22: Use `T*` to designate a single object

Basically this says to not use pointers for passing
arrays or strings.  Arrays should be passed as pointer / len
structs.  C strings should be typedefs to `zstring` or
`czstring`.

### F.25: Use `zstring` to designate a C-style null-terminated string


### F.42: Return a `T*` to indicate a position (only)

This says that the only good use of returning a pointer
is to show the location of something in a struct.

It says that you never return a pointer to transfer ownership.
This goes along with the philosophy that the caller owns

### F.43: Never return a pointer or reference to a local object

Yeah, obviously.

##### Example, bad

    int *f(x)
    {
      int fx = 1;
      return &fx; // BAD
    }

This only applies to non-`static` local variables.

### F.49: Don't return `const T`

For C++, having a function that returns a `const` value
interferes with move semantics.  This is less of an
issue for C, but, I do like having a concrete rule for
this.

### F.56: Avoid unnecessary condition nesting

Prefer using a guard-clause to take care of exceptional
cases and return early, instead of the arrow anti-pattern.

##### Example, bad

    void foo() {
      if (a)
      {
        if (b)
        {
          // succeess
        }
        else
        {
          // failure
        }
      }
      else
      {
        // failure
      }
    }

Instead prefer

    void foo() {
      if (!a)
        //failure
      if (!b)
        //failure
      // success
    }

# C: Classes and class hierarchies

While C has no classes _per se_, it does have structs.
A struct is a user-defined type, for which the programmer
can define the representation, operations, and interfaces.
Struct hierarchies are used to organize related structures
into hierarchical structures.

### C.3: Represent the distinction between an interface and an implementation using a class

Not exactly applicable, but, essentially this is saying that
getter / setter functions are better than tweaking structs
directly so we can change the representation of structs
without affecting its use.

### C.7: Don't define a struct or enum and declare a variable of its type in the same statement

### C.9: Minimize exposure of members

Encapsulation. Information hiding.

### C.10: Prefer concrete types over struct hierarchies

Don't create structs in a struct hierarchy unless
necessary.  Prefer each struct being a concrete type.

### C.90: Rely on constructors and assignment operators, not memset or memcpy

Basically, prefer explicit initialization over memset.

But some of our structs are so large, that memset will have
to do.

I have considered creating a `null` version of a large structure to use in a copy. I'm not sure about it

    struct Object nullObject = {};
    struct Object obj1 = nullObject;

Don't be afraid to initialize a small struct via `=` assignment.

### C.131: Avoid trivial getters and setters

If we're operating on a struct where members can be get and
set independently, just access the members. Don't write
getters and setters.

### C.180: Use unions to save memory

### C.181: Avoid naked unions

A union should probably be a member of a struct, where the
struct provides an indicator of the type of the union

##### Example, bad

    union Value {
      int x;
      double d;
    };
    Value v;
    v.d = 987.6;

##### Example, good

Instead make a tagged union

    enum Tag { number, text };
    struct {
      enum Tag tag;
      union {
        int i;
        zstring s;
      };
    }

### C.182 Use anonymous `union`s to implement tagged unions

    enum Tag { number, text };
    struct {
      enum Tag tag;
      union {    // Note anonymous union here.
        int i;
        zstring s;
      };
    }

### C.183: Don't use a union for type punning

Bjarne is suspicious of using unions for type punning.  For example,
using a union to get a value either as a 32-bit integer or
as two 16-bit integers. He thinks an explicit cast is better.

### Enum.1: prefer enumerations over macros

If using a bunch of `#define` as essentially an enum,
prefer making an enum.

##### Example, bad

    #define RED 1
    #define GREEN 2
    #define BLUE 3

    int color = BLUE;

Instead prefer

    enum Color {color_red, color_green, color_blue };
    enum Color k = color_blue;
### Enum.2: Use enumerations to represent sets of related named constants

### Enum.5: Don't use ALL_CAPS for enumerators

To avoid clashes with macros

### Enum.7: Specify the underlying type of an enumeration only when necessary

Try to use the `int` default type.

### Enum.8: Specify enumerator values only when necessary

Use the default values when you can.

# R: Resource Management

### R.2: In interfaces, use raw pointers to denote individual objects (only)

Each interface pointer should be a single object, not an
array.  If you need to pass an array, pass a struct
with both `data` and `length` fields.

### R.3: A raw pointer (a `X*`) is non-owning

If a function returns a raw pointer, it is assumed that
the pointer data is owned by the function that returned the
pointer.  The receiver of the return value may modify the
contents of the pointer, but, should not free it.

If this is not true, use the `OWNER()` macro.

### R.10: Avoid `malloc()` and `free()`

Prefer stack variables to `malloc` and `free`, since the
deallocation is handled automatically.

# ES: Expressions and statements

### ES.2: Prefer suitable abstractions to language features

Don't work too hard at using an almost-correct standard function
when custom code is the correct abstraction

### ES.3: Don't repeat yourself

### ES.5: Keep scopes small

Declare variables in the smallest available scope.

### ES.6: Declare names in `for`` statement initializers and conditions to limit scope

Example

    for(int i = 0; i < 10; i ++)

### ES.7: Keep common and local names short, and keep uncommon and non-local names longer

### ES.8: Avoid similar looking names

### ES.9: Avoid all caps names

### ES.10: Declare only one name per declaration

not like

   int a, b, c, d;

### ES.11: Avoid shadowing variables

### ES.20: Always initialize an object

Declare and initialize a variable in one line.

### ES.21: Don't introduce a variable before you need to use it

Avoid C89 style where all variables are declared at the top
of a function.  Instead declare a variable when it is needed.

### ES.22: Don't declare a variable before you have a value to initialize it with

Avoid the style where a variable is declared well in advance
of knowing its initial value.  Instead declare it when its
initial value is known.

### ES.23: Prefer = {} initializers

For structs with a handful of elements, initialize them
using `= {}`

### ES.25: Declare an object const unless you want to modify its value later on

Get in the habit of using `const` by default, and only not
using `const` if you know that a value will be modified.

### ES.26: Don't use a variable for two unrelated purposes

### ES.30: Don't use macros for program text manipulation

This is both for stringification macros and macros that are C statements.

### ES.31: Don't use macros for C functions

### ES.32: Use ALL_CAPS for macros

### ES.33: If you must use macros, give them unique names

### ES.42: Keep use of pointers simple and straightforward

Use pointers for single objects

### ES.45: Don't use magic constants. Use symbolic constants

This rule is just to avoid using naked numbers in loops
and initializaions

### ES.46: Avoid lossy (narrowing, truncating) arithmetic conversions

### ES.48: Avoid casts

### ES.50: Don't cast away `const`

### ES.55: Avoid the need for range checking

This is difficult in C, and requires structs in which the range
is part of the struct.

### ES.63: Don't slice

Don't copy only part of an object using assignement or initialization

### ES.65: Don't dereference an invalid pointer

### ES.70: Prefer switch to `if` when switching on an integer or enum

### ES.72: Prefer `for` to `while` when there is an obvious loop variable

### ES.73: Prefer `while` to `for` when there is no obvious loop variable

### ES.74: Avoid `do`

### ES.75: Avoid `goto`

### ES.76: Minimize break and continue in loops

### ES.78: Don't rely on implicit fallthrough in switch

### ES.79: Use `default` to handle switch common cases (only)

If there is no common case, still use default with a comment

### ES.85: Make empty statements visible

Put semicolons on their own line, and perhaps a comment

### ES.86: Avoid modifying loop control variables inside the body of for loops

### ES.87: Don't add redundant == or != in conditions for boolean types or null pointer checks

### ES.100: Don't mix signed and unsigned arithmetic

### ES.101: Use unsigned types for bit manipulation

### ES.102: Use signed types for arithmetic

### ES.107 Prefer signed integers for array indices, if they are large enough

### CP.3: Minimize explicit sharing of writable data

If you don't share writable data, you can't have a data race.
The less sharing you do, the less chance you have to forget to
synchronize access (and get data races). The less sharing you do,
the less chance you have to wait on a lock (so performance can improve).

### CP.31: Pass small amounts of data between tasks by value

To avoid semaphore and locking

### CP.200: Use `volatile` only to talk to non-C memory

Hardware registers and the like. Don't use volatile
in C-compiler-created varaibles.

### E.28: Avoid error handling based on global state

Avoid `errno` types of error handling.

# Con: Constants and immutability

### Con.1: by default, make objects immutable

`const` when you can

### Con.3: by default, pass pointers and references to const

When you know a called function will not modify the value

f(const void *)

### Con.4: use const to define values that do not change after construction

const int x = 1;

# SF: Source files.

### SF.1: Use the .c suffix for code files and the .h suffix for interface files

### SF.2: a header file must not contain object definitions or non-inline function definitions

### SF.3: use header files for all declarations used in multiple source files

### SF.4: include header files before other declarations in a file

### SF.5: A C file must include the .h file that defines its interface

### SF.8: Use include guards for all header files

### SF.9: Avoid cyclic dependencies in header files

Eliminate cycles.

### SF.10: Avoid dependencies on implicitely included names

Make sure to use all the necessary include files in a C file.
Don't rely on the fact that one include files includes another.

### SF.11: Header files should be self contained

Make sure to include all the necessary include files in a H file.
Don't rely on the fact that one include file often appear after another

### SF.13: Use portable header identifiers in #include statements

Each #include "" should use the specific case of the file
and not rely on case insensitivity of the file system.

Also, use forward slashes and never backslashes

### SL.con.4: Don't use memset or memcpy for argument that are trivially copyiable

it is preferred to set a small struct using equals

### SL.str.3: Use zstring or czstring to refer to a C-style, zero-terminated, sequence of characters

### SL.str.4: Use char* to refer to a single character

### A.1: Separate stable code from less stable code

### A.2: Express potentially reusable parts as a library

### A.4: There should be no cycles among libraries

### NR.1: Don't insist that all declarations should be at the top of a function

C89 is dead

### NR.2: Don't insist on having only a single return statement in a function

It is fine to fail early












