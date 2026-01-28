# MeteoriC-compiler & IDE on 6502 (ORIC ATMOS)

&copy; 2026 Jonas S Karlsson

MeteoriC is a *minimalist* C-compiler for running *on-device* 6502. Like Turbo Pascal for CP/M 8080 computers, it includes an editor, compiler, and ability to run the program from inside the environment. Errors are indicated in the editor allowing for fast turn-around during development.

# Origin

It's written from scratch starting August 2025 by Jonas S Karlsson, in 6502 assembly using CA65. The idea about minimal a minimal compiler is similarly, to [Small-C](https://en.wikipedia.org/wiki/Small-C), and 
[C/65](https://atariwiki.org/wiki/attach/C65Manual-Text/c65manual.pdf), and others.

It has restrictions and limitations. It does *not*, however, generate ASM code that needs to be assembled, but instead directly generates relevant binary machine code, ready to run, in memory.

# The C-language

MeteoriC is a subset of the C programming language as defined by Kernighan and Richie in the book, “The C Programming Language”, published by Prentice-Hall updated with latest langauge (ANSI-style) syntax.

Generally, the idea is that a legal C-program should be compilable, assuming it is within the supported subset. Where there are devications, they have been noted.

MeteoriC comes with an "optional" standard library, basically covering everything from libc etc, that makes sense. Depending on how much of the library code that is enabled, it may use up to about 600 extra bytes. However, binaries can be compiled in "libraryless" mode, where only the runtime library is included (~100 bytes). This also depends on if the code uses the ORIC-ATMOS ROM routines, or not. There are thoughts of being able to generate ROMmmable code to replace, or provide an alternative to the BASIC ROM.

It does not directly support external libraries, but can handle CC65 calling convention (_fastcall/AX).

# Supported Subset

* the datatype `word` (unsigned int, 16-bit)
* it can be used as `char*` in `*v=...;` and `3+*v`
* operators: `+ - & | ^ *2 /2 << >>`
* math: `*` (coming `/ %`) (requires "mathlibrary")
* `+= -= &= |= ^= <<= >>=`
* optional inline ASM;
* functions with parameters (locals coming)
* `if (...) ...` with optinal `else ...`
* `{ block(); stmts(); }` (no define local vars)
* `do ... while(...);` - most efficient/small
* `while(...) ...` - ok
* `for(...; ...; ...) ...` - expensive and big/slow
* `return;` or `return ...;`
* no linking neeeded

# Currently Not Supported

* `#define ...` or `#ifdef ...`
* `#include ....`
*  `break; continue;`
*  no separate compilation
* `long`
* `signed int`
* `float double`
* `struct union` and bit fields
* multi-dimensional arrays
* `switch statement`
* `&& !!`

It may seem restrictive, but operators have been choosen for ease of implementation as well as efficiency.

Still, the supported subset should be enough to implement the compiler itself. However, for space and speed reasons MeteoriC compiler is written purely in assembly to give the user the most available memory.
  
# Limitations

* preceedence
* 
