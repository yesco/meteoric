# MeteoriC-compiler & IDE on 6502 (ORIC ATMOS)

&copy; 2026 Jonas S Karlsson

MeteoriC is a *minimalist* C-compiler for running *on-device* 6502. Like Turbo Pascal for CP/M 8080 computers, it includes an editor, compiler, and ability to run the program from inside the environment. Errors are indicated in the editor allowing for fast turn-around during development.


# Origin

It's written from scratch starting August 2025 by Jonas S Karlsson, in 6502 assembly using CA65. The idea about a minimal compiler is similar to [Small-C](https://en.wikipedia.org/wiki/Small-C), and 
[C/65](https://atariwiki.org/wiki/attach/C65Manual-Text/c65manual.pdf), and others.

It has restrictions and limitations. It does *not*, however, generate ASM code that needs to be assembled, but instead directly generates relevant binary machine code, ready to run, in memory.


# The C-language

MeteoriC is a subset of the C programming language as defined by Kernighan and Richie in the book, “The C Programming Language”, published by Prentice-Hall updated with latest langauge (ANSI-style) syntax.

Generally, the idea is that a legal C-program should be compilable, assuming it is within the supported subset. Where there are devications, they have been noted.

MeteoriC comes with an "optional" standard library, basically covering everything from libc etc, that makes sense. Depending on how much of the library code that is enabled, it may use up to about 600 extra bytes. However, binaries can be compiled in "libraryless" mode, where only the runtime library is included (~100 bytes). This also depends on if the code uses the ORIC-ATMOS ROM routines, or not. There are thoughts of being able to generate ROMmmable code to replace, or provide an alternative to the BASIC ROM.

It does not directly support external libraries, but can handle CC65 calling convention (_fastcall/AX).

# Usage and positioning

Why another compiler? Well, a C-compiler is a challange! Nethertheless for 6502. What's missing is the interactivety of BASIC or the feeling of Turbo Pascal that revolutionized programming on the nearly equivalent Z80, at least contemporary.

So, the goal is to be interactive, and have a decent integrated compiler, and editor and to be able to run the programs directly, all on the machine without loading or switching to other programs.

`D-flat` is an amazing language, providing a structured basic, built-in editor ala BASIC with line-numbers, but it's much faster than typical BASICs. But it's not C...

For C, we have the excellent `CC65` that provides a very standards compliant cross-compiling compiler. It gets flack because of generated (bloated) slow code, sometimes a bit unfair. Other realistic good alternatives are `SBCC`, `VBCC` (limited licence), and finally `Oscar64`. For ORIC ATMOS, the "outdated" `RCC`, based on early `LCC`. It is well supported in the ORIC community but voices say "slow". It is also difficult to use under linux.

As we can C, sorry see, interactive alternatives are scarce.

We can mention that there is a lisp-interpreter, only binary available for ORIC ATMOS.

So, here comes the `MeteoriC`-compiler!


# Supported subset

MeteoriC supports a decent subset of standard C. The goal is that code should be reasonably portable, and that it should provide the benefits of C on the device for interactive programming and experimentation.

The idea is that code that is legal C and using supported features under the limitations should compile and give same result. However, it comes with limitations. More about that in next chapters.

Here is an overview of features supported:

* the datatype `word` (unsigned int, 16-bit)
* it can be used as `char*` in `*v=...;` and `3+*v`
* unlimted long names `word a, b, abba, foo_bar32, _x;`
* operators: `+ - & | ^ *2 /2 << >>`
* math: `*` (coming `/ %`) (requires "mathlibrary")
* `x= 42;` assignement
* `a=b=c= 42;` multi variable assignments
* `+= -= &= |= ^= <<= >>= *=2; /=2;`
* functions with up to 8 parameters (locals coming)
* `if (...) ...` with optinal `else ...`
* `{ block(); stmts(); }`
* `do ... while(...);` - most efficient/small
* `while(...) ...` - ok
* `for(...; ...; ...) ...` - expensive and big/slow
* `return;` or `return ...;`
* integrated editor jumping to error
* optional inline ASM; (takes extra 2KB)


# Features currently not supported

In the tradtion of `Small-C`, `C/65`, `KickC`, and others the 

* `#define ...` or `#ifdef ...` (TODO: limited)
* `#include ....` (TODO: use for choosing library usage)
*  `break; continue;` (TODO)
* `extern` - separate compilation (TODO: dynamic libraries!)
* `long`
* `signed int` (TODO: considering)
* `float double`
* `struct union` (TODO: thinking about it)
* and bit fields
* multi-dimensional arrays
* `switch statement` (TODO: hmmm)
* `...? ...: ...` (TODO: will come)
* `&& !!` (TODO: yeah, will do, maybe `...? ...: ...` first)

It may seem restrictive, but operators have been choosen for ease of implementation as well as efficiency.

Still, the supported subset should be enough to implement the compiler itself. However, for space and speed reasons MeteoriC compiler is written purely in assembly to give the user the most available memory.
  
# Limitations

* preceedence not supported; evaluation (at the moment) is LEFT-to-RIGHT: `3+4*2`=> `14`!
* expressions support limited right-hand side data
<br>`COMPLEX op simple op simple ...`
* no paresentitaion supported
* `COMPLEX` can be 
  - `variable`
  - `*variable` assumed to be pointer to char
  - `++var` or `--var` or `var++` or `var--`
  - `fun(...)`
  - `"fourtytwo"`
  - `simple` (see below)
* `simple` must be a constant
  - `42`
  - `'*'`
  - `0x2a`
  - `052`
* assignments
  - are not expressions
  - `var += simple;` with most operators


# Efficiency

6502 is a challanging platform for compilation of high-level languages. It has a small stack, limited addressing modes, and all arithmetic is byte-only. Nevertheless, new compilers like `Oscar64` is doing marvels, both in speed and code size.

The following forms are exceptionally efficient and optimized on 6502.

Inc &amp; dec operators:
* `++i; i++;` - same cost (6 bytes)!
* `--i; i--;` - same cost (+2 bytes cmp ++)
* in an expression:
  - `++i +1` `i++ +2` `i-- +2` all same cost (+ 10 B for the ++ --)
  - `--i +3` worst! (+ 2 bytes)

Shifting:
* `a<<= 1;` - 2 instructions!
* `a>>= 1;` - same

if statements:
* `if (v==0) ...` - most efficient
* `if (v==...) ...` - 13 bytes better than `if (...==v) ...`
* `if (v&42) ...` - fewer bytes for small constant

while statements:
* `while(v) ...` - fast
* `while(v==42) ...` 
* `while(v==x) ...` 
* `while(v<42) ...` 
* `while(...<...) ...` - expensive overhead
* `while(...==...) ...` - expensive overhead

do...while:
* `do ... while(v);` - small and FASTEST
* `do ... while(!v);` - same same
* `do ... while(v<42);` - ok
* `do ... while(v<x);` - ok
* `while(...<...) ...` - expensive overhead +9 + 
* `while(...==...) ...` - expensive overhead +9 + 

function  calls:
* `foo()` no parameters, no local == jsr

# Internals

MeteoriC-compiler is a recursive-descent interpretive parser. This means it's datadrive *interpreting* rules as data. These rules, when matching will use inline templates to generated machine code directly. The templates have directives and markup to allow for specialization.

This is maybe an unusual design, however, it minimizes the amount of assembly code to be written. The rules are expressed in a limited variant of `BNF`.

In principle it can be repurposed to parse many different languages, however, it's specifically targetted for 6502. It's currently about 1865 bytes. The basic rules, are currently about 6900 bytes, of which 1500 are special *optimizing* rules. These decrease code size and improve speed by providing specialized rules for common edge cases. They are specialized for small byte size integer constants; specific operators; or even special values, like 0.

## Generated code

## Optimization Limitations

The compiler comes with inherent limitations, these are mostly stemming from limited memory and speed.

Great compilers run on bigger systems and does whole-program analyzis; this allows them to totally restructure and specialize the code; functions called once are inlined, parameter passing removed if only passing constants; common subexpression elimination, etc.

## ...

More info coming...


# Future/Planned

* optimizations coming for non-recursive funcs, basically using static parameter locations.
* `&& ||`
* better handling of preceedence
* genrated stand-alone .tap files
* ROM-less variant of compiler and code
* permanent storage, files, source code, etc
* operating system? LOL
* user input?

