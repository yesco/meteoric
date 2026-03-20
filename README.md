# MeteoriC-compiler & IDE on 6502 (ORIC ATMOS)

&copy; 2026 Jonas S Karlsson

MeteoriC is a *minimalist* C-compiler for running **on-device** 6502. Like Turbo Pascal for CP/M 8080 computers, it includes an editor, compiler, and ability to run the program from inside the environment. Errors are indicated in the editor allowing for fast turn-around during experimentation.

> *Writing a C compiler in assembly for the MOS 6502 (the processor in the NES, Commodore 64, and Apple II) is a legendary challenge. It is a "Holy Grail" project because the 6502 architecture is famously hostile to C.*

# Alpha preview

This is the very first copy being shared among *early testers*.

Feedback is appreciated. Please refer to the rest of the docs.

Report any bugs, or issues. Take screen-shot of code if you found some that doesn't compile or do the right thing; beware of limitation of the implementation when it comes to expressions.

**Known issues of the alpha:**

* can only view/edit one page of code (no scroll)
* DEMO version with examples have limited memory
* very little error checking
* it may hang, it may crash, or give wrong result! - please report with examples!
* Arrays: have char array, but for `word` you can use `xmalloc,peek,poke,deek(addr),doke(addr,val)`! - *Note:* deek/doke take addresses, so for integer array need multiply index by two: doke(i*2+array, val);`
* `ORIC ATMOS 48K` required
* editing is using the HIRES memory as a buffer, if switching to hires mode, and back, the edit buffer is crapped. CTRL-Z re-initializes it with the default startup example. (TODO: save/restore)
* cassette saving and loaded implemented, but not sure if it works... (TODO: disk)
* no local variables (only global or function parameters)



# Speed Anecdote

Just did a test using the simulator, compiling `/Input/lps.c` - essentially 4711 lines with `++i;`.

```
./mc         =>   1.777 s   22432 bytes (for 6502)
cc65              0.775 CRASH! (out of code space!)
cc65 -Oirs   =>   0.857 s   37560 bytes (for 6502 opt)
oscar64 -O0  =>  >5 minutes! (turned off opt!)
oscar64 -O3  =>  23.724 s     152 bytes (for 6502, opt)
clang        =>   0.454 s   61904 bytes (for ARM)
tcc          =>   0.024 s  135464 bytes (for ARM)
```
So, MeteoriC, generates a binary of 22432 bytes, running the compiler in a simulated 6502 takes 1.777 seconds, which is pretty fast! That's 2651 lines per second. This with the overhead of interpreation which may be up to 10x the cost in time. MeteoriC has not yet gotten focus on speed-improvements of the compiler itself. So this figure will change.

CC65 crashes because the code generated is too big. It says 28500 bytes too big, and then it had already written a file of 35498 bytes, basically reaching the 64K limit. Turning on all optimizations, and even then I had to the file to 4689 lines of code, it generates a binary of 37560 bytes.

`Oscar64` is quite an enigma--at first, I ran it without optimization `-O0`, intending to give it the least work to do; but it sat there for more than 5 minutes! Not sure if it'll ever terminated. Then, changing tactics, I turned on optimization and the compiler did finish thejob: in about 24 seconds! However, what is the result is only 152 bytes. Taking 24 seconds makes is 13 times slower than MeteoriC,but it sure does clever optimizations.

I have not been able to compile clang-mos for termux environment so I just compiled it to arm and it was relative fast at 0.454 seconds. Binary of 61904 bytes.

Finally, no speed-test is complete without Bellard's `tcc` Tiny C Compiler; it generated 135464 bytes at a tremendous speed, in 0.024 seconds. TCC is famous for it's high-speed, but generating large, unoptimized code, as can be seen from that it's output file is more than double from that of clang.

In any case, MeteoriC holds up pretty good in terms of speed for this simple test.



# Origin

MeteoriC is written from scratch starting August 2025 by Jonas S Karlsson, in 6502 assembly using CA65. The idea about a minimal compiler is similar to [Small-C](https://en.wikipedia.org/wiki/Small-C), and [C/65](https://atariwiki.org/wiki/attach/C65Manual-Text/c65manual.pdf), and others.

It is probably the smallest C-compiler (ever) written for 6502, owing partly to its data-driven architecture. See more about it in the implementation section.

MeteoriC has restrictions and limitations. It does *not*, however, generate ASM code that needs to be assembled, but instead directly generates relevant binary machine code, ready to run, in memory. This is similar to what `Turbo Pascal` did on `CP/M`.

The compiler has been preceded by various 6502 experiments as well as interpreters and compilers written in C. One variant was prototyping compiling lisp-code directly to 6502 machine code.



# The C-language

MeteoriC is a subset of the C programming language as defined by Kernighan and Richie in the excellent book, “The C Programming Language” (K&amp;R2), published by Prentice-Hall updated with ANSI-style syntax. It's available online for free at [The C Programming Language](https://github.com/auspbro/ebook-c/raw/refs/heads/master/The.C.Programming.Language.2Nd.Ed%20Prentice.Hall.Brian.W.Kernighan.and.Dennis.M.Ritchie..pdf).

This manual that you're reading isn't a guide for C-programming. The user is assumed to know C. Be aware that MeteoriC isn't "fully standard-compliant", and may allow things which isn't allowed in standard C.

Generally, the idea is that a legal C-program should be compilable, *assuming* it is *within the supported subset*. Where there are deviations, they have been noted, or may be reported. `KISS` - Keep It Simple.

MeteoriC comes with an "optional" standard library, basically covering everything from libc etc that makes sense. Depending on how much of the library code that is enabled, it may use up to about 800 extra bytes. However, the compiler can be assembled in "library-less" mode, where only the runtime library is included (~100 bytes), and even that isn't needed if you don't need functions with parameter passing!

The compiler, when compiled for ORIC-ATMOS, will use some of the BASIC ROM routines, mostly for screen, and keyboard as well as graphics and sound, by providing a small API.

There are plans to generate ROMmmable code to replace, or provide an alternative to the BASIC ROM. This is particularly interesting for the LOCI-device that provides a modern file-system capabilities.

MeteoriC does not directly support external libraries, but can handle CC65 calling convention (_fastcall/AX).




# Usage and positioning

Goals:

* create, edit, run, experiment with C code on the machine
* interactive development, playing
* fast on-device interaction
* make it fun!

Non-Goals:

* it's probably not going to be a full compatible compiler with all the bells and whistles
* don't expect to take any big C-code and compile it, there are cross-compilers excelling at that
* no C++, LOL

Why another compiler? Well, a C-compiler is a challenge! Nevertheless for 6502. What's missing is the interactivity of BASIC or the feeling of Turbo Pascal that revolutionized programming on the nearly equivalent Z80, at least contemporary.

So, the goal is to be interactive, and have a decent integrated compiler, an editor, and to be able to run the programs directly, all on the machine without loading or switching to other programs.

Other softwares like this include `D-flat`, which is an amazing language and environment, providing a structured Integer BASIC, built-in line editor, but it's much faster than typical BASICs and provides really fast graphics-replacement routines. But it's not C!

For C, we have the excellent `CC65` that provides a very standards compliant cross-compiling compiler. It gets flack because of generated (bloated) slow code, sometimes a bit unfair, because it's very reliable and *correct*. Other realistic good alternatives are `SBCC` (gnu licence), `VBCC` (limited license), and finally `Oscar64` (also gnu) which seems like a rising super-star!

For ORIC ATMOS, the "outdated" *RCC*, based on an early *LCC*, part of *OSDK* is still used. It is well supported in the ORIC community but voices say "slow", and "big" code. It is also difficult to use under Linux, but the community has provided many tools and much example codes for it.

As we can C, sorry see, interactive alternatives are scarce.

We can mention that there is a lisp-interpreter, with only binary available for ORIC ATMOS.

So, here comes the `MeteoriC`-compiler!


# First example

Too much talking and not enough action, or code.

```
01: // A simple program that prints A-Z\n
02: // in many different ways

03: word c;

04: word main() {

05:   puts("ABCDEFGHIJKKLMNOPQRSTUVWXYZ");
06:   putz("ABCDEFGHIJKKLMNOPQRSTUVWXYZ\n");

07:   // - biggest and least efficient
08:   for(c='A'; c<'Z'+1; ++c) putchar(c);
09:   putchar('\n');

10:   // - OK, OK
11:   c= 'A'-1; while(c<'Z'+1) putchar(++c);
12:   putchar('\n');

13:   // - smallest and most efficient  
14:   c= 'A'; do putchar(c++); while(c<'[');
15:   putchar('\n');
17:
16:   return 4711;
17: }
```

Notes:
* Line 01,02: `// comments only`
* Line 03: global `word` variable, no local variables yet, parameters OK
* Line 05,06: sometimes the most efficient!
* Line 04,16: the main returns a `word` (standard C: int), you can use this to have the IDE print the value when the program exits without using a print statement
* Line 05: `puts` adds a newline after the string
* Line 06: `putz` (non-standard) or standard-C: `fputs("ABC...", stdout)` doesn't add newline
* Line 08: `for`- loops jumps around a lot, and thus takes more space, and are slowest
* Line 12: `putchar('\n')` becomes a simple `jsr nl`
* Line 13, 14: `c<'Z'+1` is most costly than `c<'['`
* Line 14: `do...while(...)`; generates the *smallest code* and is the most efficient looping construct

# Supported subset

MeteoriC supports a decent subset of standard C. The goal is that code should be reasonably portable, and that it should provide the benefits of C on the device for interactive programming and experimentation.

The idea is that code that is legal C and using supported features under the limitations should compile and give same result. However, it comes with limitations. More about that in next chapters.

Here is an overview of features supported:

**Integrated IDE:**
* built-in help screens
* list symbols recognized (library functions)
* reasonably fast compiler (nearly 1000 "lines"/minute?)
* integrated full-screen editor
* editor can positions cursor directly at/near error
* integrated disassembler
* optional inline ASM; (prototype, takes extra 2KB)

**Variables:**
* global variables of type `word`, which is an `unsigned uint16_t` basically. Think of it as if you've written `typedef uint16_t word;`, which by the way you can and it's ignored!
* char-arrays (see arrays section)
* unlimited long names `word a_cucumber, b52, abba, foo_bar32, _x;`

**Expressions:**
* operators: `+ - & | ^ *2 /2 << >>` taking `v` or `const` as right hand parameter
* math: `*` (coming `/ %`) (using "mathlibrary")
* `x= 42;` assignment
* `a=b=c= 42;` multi variable assignments
* `+= -= &= |= ^= <<= >>= ... ;` all working with **only** simple constants or variables, even the shifting takes a variable number of shifts `3<<n`.

**Comparisons &amp; Logical:**
* comparisons: `== < >=` (more to come... `> !=`)
* `!var` or `!!var` or `!(expr)` works
* logic: `expr && expr` (not `||` yet)
* parenthesis can *only* be used around an expression, like `(expr && expr)` or `(expr == expr)` so this would work correctly:
* `((a==b+3) && (b==4))` - Is fine.

**Functions:**
* functions with up to 8 parameters (TODO: locals coming)
* recursive functions supported!
* no forward declaration of function (yet)

**Control constructs:**
* `if (...) ...` with optional `else ...`
* `{ block(); stmts(); }`
* `do ... while(...);` - most efficient/small
* `while(...) ...` - OK
* `for(...; ...; ...) ...` - expensive and big/slow
* `return;` or `return ...;`

**Arrays:**
* `char foo[42];` declares an array
* `char foo[42]={0};` ensures it's zeroed out
* `char foo[]="fish";` initialize, with 5 bytes. 
* `char foo[]={'f',105,0x73,0b1101000);` initialized with 4 bytes.
* **Note:** there is no `char` type so you can declare a single variable this way! NO: `char c;` :-D
* `sizeof(array)` gives size in bytes
* `sizeof(var)` gives 2 for other vars (no char yet)
* `xmalloc(bytes)` fail or give pointer to `bytes` allocated

**Accessing memory:**
* `*r= *a+3;` derefencing a word is equivalent to using a `char*`-style access (assign, read)
* `xmalloc(bytes)` - to allocate dynamic memory fail if run out (safer than `malloc`)
* `peek(a) poke(a,v)` - alternative byte/char memory access (char*)
* `deek(a) doke(a,v)` - word value access at any memory location `*(int*)(a)`. **Notice:** the address isn't aligned and it *doesn't* work like index, so to access an "array of integers" you'd have to `doke(i*2+arr, val);`.


## Libraries

One of the benefits of C is the standard libraries, like "`libc`". Most applicable functions have been implemented,  see the `User manual` section at the end.



# Features currently **not** supported

In the tradition of `Small-C`, `C/65`, `KickC`, and others the compiler supports a narrow, but useful subset of C.

Here is a list of what is *not* (currently) supported:

**Meta level:**
* `/* no slash-star comments */` use `// comments`
* `#define ...` or `#ifdef ...` (TODO: limited)
* `#include ....` (TODO: use for choosing library usage)
* In actually anything after `#` is ignored (for now)
* `typedef uint16_t word;` is implied; you can actually type it and it will be ignored!

*  `break; continue;` (TODO)
* `extern` - separate compilation (TODO: dynamic libraries!)
* `long`
* `int` `signed int` (TODO: considering, but it's slower/more code and will get you in trouble! `<`)
* `int main()` is allowed for compatibility, as `void main()`, but only here, not for functions in general. (yet)
* `float double tribble squabble` LOL
* `struct union` (TODO: thinking about struct)
* bit fields (nono)
* no local arrays, make them global, or use `xmalloc()`
* no word arrays: `word foo[...]=...` not supported:
  - use xmalloc to get memory pointer/address
  - `deek doke` to access `word` values (these are efficiently compiled!)
  - `peek poke` for byte `char` access (efficient)
* multi-dimensional arrays (see above)
* `switch statement` (TODO: hmmm)
* `...? ...: ...` (for now use `if`, TODO: will come)
* precedence in expressions are enforced to occur in highest order to lowest order. `4*5+3` is fine `3+4*5` is prohibited, as it is more complicated (and unnecessary).
* `!!` (TODO: yeah, will do, maybe `...? ...: ...` first)
* `(a==b+3) && (b==4)` - parenthsis have limited support, but can "stop" parsing where it should. In this case without parens it would be understood as: `(a==(b+3 && b)==4`. (left-to-right)
* `main` is special. It has to be the **last function**, and you can't recurse on or refer to it. LOL

This may feel as if it's not a complete C - and it's not! But operators have been chosen for ease of implementation as well as efficiency.

Still, the supported subset is plenty enough to implement the compiler itself. However, for space and speed reasons MeteoriC compiler is written purely in assembly to give the user the most available memory. Some IDE experimental features are coded in C, mostly information functions. These are optional and may be removed.
  

# Limitations

**Compilation:**
* less strict: if it looks like C it might "eat it" and generate some code
* does not (currently) check arguments number/types of functions, it'll crash if wrong! (TODO:?)
* few error messages, it'll show how far it got

**Limited globals:**
* global integral variables are limited to zero page. This both saves code size, and improves speed. This is not a limitation on arrays, they are different and stored with the code.

**Functions:**
* No local varibles yet! This is the same for `main`.
* (recursive) function calls have limited stack space, it uses the hardware stack, in the future non-recursive functions will have an extremely fast `_regcall` implementation.
* `main` has to be the last function. There can be only one, and it cannot be referred to by name, or called.


## Expressions

MeteoriC has a quirk; it uses pure left-to-right evaluation as this is the most efficient way. This, however, may give a semantic clash with *operator preceedence* levels.

In C, operators of higher or equal preceedence are to be evaluated first. For example in `a+b*c`: `b*c` is multiplied first and then `a` is added. Normal optimizing C compilers will re-order the expression for efficiency. If they aren't optimizing they'll push `a` on the stack ; push `'b' one stack; push `c` one the stack; call `multiply` that pops two items from the stack, nd then pushes the result; only to pop it off again and the final `a` and then add it; and push it on the stack! It's easy to make it correct but it comes at a very high cost; which is why many Forth-interpreters have so much overhead.

As a compromise, in MeteoriC, we're restricing the *order* of operations to keep it *legal* and *semantically correct*.

*(TODO: this isn't fully implemented yet; currently only `* +` are "controlled")*

In an expression like `COMPLEX op1 simple op2 simple ...` , `op1` must have **higher or equal precedence** than `op2`. Use this ordered list (from Highest to Lowest) to determine what is legal:

|operator|name|
|---|---|
| `(... , ...)`	| Parenthesis |
| `++ -- ! -`   | Unary (one arg) |
| `* / %`	| Multiplicative |
| `+ -`         | Additive |
| `<< >>`       | Shift |
| `< > <= >=`   | Relational |
| `== !=`       | Equality |
| `&`           | Bitwise AND |
| `^`           | Bitwise XOR |
| `|`           | Bitwise OR |
| `&&`          | Logical AND |
| `||`          | Logical OR |
| `+= -+ /+ *= <<= >>=` | Assignment Modifier |
| `=`           | Assignement

### COMPLEX

Can be any of:
```
(...)
array[...]
ptr[...]
variable
0xf00d
4711
'c'

   OR

++variable --variable variable-- variable++
variable= ...

   These are special as they are very efficient

variable+= bytevalue;
variable-= bytevalue;

variable<<= simple;
variable>>= simple;

```

### Legal expresions

Here are some illuminating example. In all these cases, assume the evaluator is looking at the raw expression: `A op1 B op2 C`:

In these examples, MeteoriC will have no issues and behave exactly like the standard C compiler, as they are purely left-to-right evaluation.

`A * B + C`

C sees * is higher than +, so it does A * B first. Matches L-to-R.

`A + B << C`

C sees + is higher than <<, so it does A + B first. Matches L-to-R.

`A == B & C`

C sees == is higher than &, so it does A == B first. Matches L-to-R.

`A & B | C`

C sees & is higher than |, so it does A & B first. Matches your L-to-R.

`A | B && C`

C sees | is higher than &&, so it does A | B first. Matches L-to-R.

**Note:**

The left hand side of an `LHS op RHS` can alwasy be written as `(LHS) op RHS`, however, no parenthis can be used for `RHS` -- there you'd need to break out the `RHS` expressions and assign it to another variable first.

So you can "break" the rule above by writing:

`(A + B) * C`


### Illegal expressions

In these examples, a standard C compiler will try to do the second operation first. Meteoric evaluator will prohibit these.

`A + B * C` ==> write instead `B * C + A`

The Conflict: C wants to do `B * C` first. L-to-R would do `A + B` first.

`A | B & C` ==> write instead `B & C | A`

The Conflict: C wants to do `B & C` first (AND is higher than OR). L-to-R would do `A | B` first.

`A && B == C` ==> write instead `B == C && A`

The Conflict: C wants to do `B == C` first. L-to-R would `A && B` first.

`v= A << B + C` ==> write instead `n= B+C; v= A << n;`

The Conflict: C wants to do B + C first (Math is higher than Shifting). L-to-R would do `A << B` first.

### The "Same Level" Rule

If `op1` and `op2` are the same (e.g., `A / B * C`), it is always legal order because C's math and logic operators (except assignments) default to left-to-right associativity when the precedence is equal.



# Efficiency

6502 is a challenging platform for compilation of high-level languages. It has a small stack, limited addressing modes, and all arithmetic is byte-only. Nevertheless, new compilers like `Oscar64` is doing marvels, both in speed and code size.

The following forms are exceptionally efficient and optimized on 6502.

**Inc &amp; dec operators:**
* `++i; i++;` - same cost (6 bytes)!
* `--i; i--;` - same cost (+2 bytes cmp ++)

**In an expression:**
* `++i +1` `i++ +2` `i-- +2` all same cost (+ 10 B for the ++ --)
* `--i +3` worst! (+ 2 bytes)

**Shifting:**
* `a<<= 1;` - 2 instructions!
* `a>>= 1;` - same
* `a<<= 2;` and `>>=` all the way up to 7 are inlined &amp; fast

**Others:**
* `BIGGER * smaller` - faster
* `... op CONST` - probably better than the other way around
* `... op const` - some times ops optimized for 0..255
* `... >= CONST` -  9 bytes!
* `... <  CONST` - 11 bytes
* `... <  ...`   - an extra +10 bytes
* `... >= ...`   - an extra + 10 bytes

**Indexing char-arrays:**
```
  (this is more or less from Input/arrays.c)

  s: $7BAB "foobar"
                    (code)
  INDEX       CHECK BYTES
  =====       ===== =====
  s[3]        : b=b   5     // super! LDA $addr
  s[(char)a]  : b=b   7     // good:  LDY $addr,x
  s[(char)(a)]: b=b  10     // generic expression

  s[300]      : ?==  18      \
  s[a]        : b=b  18       }- must do int arith
  s[(a)]      : b=a  18      /
```
The last three cases comes out to:
- 4 bytes to load index
- 10 bytes to ADD index and address of `s`, and then storing it in `tos`
- 4 bytes to `LDX #0; LDA ($addr,x)` indirect, using `($addr,x)` instead of `($addr),y` saves 2 bytes as we get `x` set to 0 for free!

**Pointer (to an array):**
```
  INDEX       CHECK BYTES
  =====       ===== =====
  p[3]        : b=?  14     // slight, ok
  p[(char)a]  : b=?  14     // ok
  p[(char)(X] : b=?  17     // inbetween

  p[300]      : ?==  20     \
  p[a]        : b=b  20      }- bad boy
  p[(EXPR)]   : b=b  20     /
```
As can be observed; this is substantially more code and less efficient; in most cases the pointer needs to be loaded to zero page, and in the first two cases we can benefit from indirect indexing, the third, ok.

The conclusion is that indexing directly in global array is substantially cheaper, particularly if index < 256 `(char)`. If possible: *don't pass pointers around but prefer indices*.

**array[index]=...;**
```
INDEX       CHECK BYTES
  =====       ===== =====
  s[3]=        : A=A  5       # precalc address
  s[(char)a]=  : B=B  7       # cool
  s[(char)(X)]=: C=C  16      # nah, wtf? TODO:
  s[300]=      : D=D  5       # precalc address
  s[a]=const   : E=E  17      # runtime half add
  s[a]=EXPR    : Q=Q  19      # some cleverness
  s[(EXPR)]=   : F=F  26      # worst full runtime add
```

**Update element using pointer[index]:**
```
  INDEX       CHECK BYTES
  =====       ===== =====
  p[3]=        : A=A   6       # precalc, was 28
  p[(char)a]=  : B=B   6       # yeah! `STA (p),Y`
  p[300]=      : D=D  17       # runtime half add
  p[a]=        : E=E  18       # runtime half add
  p[(EXPR)]=   : F=F  27       # worst full runtime add
```
This is not yet an optimized operation, it was just added. It currently has one generic rule `ptr[...]`. This is where we notice the cost of "generic" un-optimized operators, and how much it matters!


**if statements:**
* `if (v==0) ...` - most efficient
* `if (v==...) ...` - 13 bytes better than `if (...==v) ...`
* `if (v&42) ...` - fewer bytes for small constant

**while statements (better to worse):**
* `while(v) ...` - fast
* `while(v==42) ...` 
* `while(v==x) ...` 
* `while(v<42) ...`
* `while(v==...`) ...
* `while(v<...`) ...
* `while(...==...) ...` - worst: expensive+bigger
* `while(...<...) ...` - worst: expensive+bigger

**do...while:**
* `do ... while(v);` - small and FASTEST
* `do ... while(!v);` - same same
* `do ... while(v<42);` - OK
* `do ... while(v<x);` - OK
* `while(1) ...` - only 3 bytes! (`jmp $????`)
* `while(...<...) ...` - expensive overhead +9 + 
* `while(...==...) ...` - expensive overhead +9 + 

**function calls:**
* `foo()` no parameters, no local == jsr

**static (fixed) arrays:**
* `char arr[4711];`
* `arr[3]` - most efficient
* `arr[(char)i]` - perfect: `LDA arr,x`
* `arr[(char)...]`- same...
* `arr[...]` - same expensive as using a pointer, as it needs to add index to the pointer and store in zero page

**dynamic arrays, or using pointers:**
* a pointer needs to be dereferenced, on 6502 this means copying it to zero page, so it's always going to have more overhead compared to a static
* `char* ptr= "foobar";` - (not supported yet, but less efficient)
* `ptr[3]` - ok efficient
* `ptr(char)i]` - perfect: `LDA arr,x`
* `ptr[(char)...]`- same...
* `ptr[...]` - expensive, we will happily use any integral variable as pointer, LOL
* **Notice:** we will happy use any variable value as "ptr"




# Implementation &amp Internals

MeteoriC-compiler is a recursive-descent interpretive parser. This means it's data-drive *interpreting* rules as data. These rules, when matching will use inline templates to generated machine code directly. The templates have directives and markup to allow for specialization.

This is maybe an unusual design, however, it minimizes the amount of assembly code to be written. The rules are expressed in a limited variant of `BNF`.

In principle it can be repurposed to parse many different languages, however, it's specifically targeted for 6502. It's currently about 1865 bytes. The basic rules are currently about 6900 bytes, of which 1500 bytes are special *optimizing* rules. These decrease code size and improve speed by providing specialized rules for common edge cases. They are specialized for small byte size integer constants; specific operators; or even special values, like 0. They make the code about 20% smaller and faster.


## Generated code

The binary code is generated directly based on templates which come with each rule. Some rules are more specific and generates better code for specific cases.


## Integral Variables

Single variables, like word, char, and pointers are all stored in zero page. This does limit the number of globals, however, it makes code up to 20% smaller, and maybe 5% faster.

The zero page is also used for parameter passing, (TODO:) either direct (when safe and correct), or using the stack temporary. Read more about calling convention in later section.


## Strings &amp; Array data

Array data is inlined with the code. This goes for strings too. Initializing a global `char foo[]=...;` is cheaper than assigning a constant to a poitner during runtime. The latter will incur an overhead of 3+4=7 bytes, because it needs to jump over the string!


## Calling convention

The code generator uses a few different code-calling conventions internally:
* direct calling ATMOS BASIC ROM using `jsr $addr`
* ORIC ATMOS graphics/music routines fixed parameter passing calls
* recursion safe function calling - this employs a totally new calling convention
* TODO: not using stack - automatic zero page parameter passing, this can be done in some sitatuations when safe and non-recursive.


## Memory map

The current `malloc()` just allocates bytes linearly directly after the program. As it's limited by ORIC ATMOS 16KB ROM, 8KB HIRES screen, 2KB CHARSET, and 1K of low memory (ZP, stack, page 2, IO page, reserved page 4).

```
   T e x t M o d e      |        H i r e s M o d e       
                 _______|_____________ _ _ _ _ _ _ _ _ _ _
                 |                 |     
                 |     BASIC       |            
                 |                 |
                 |           R     |  $C000-$FFFF   16 KB
                 |           O     |    
                 |           M     |
                 |                 |
                 |_________________|__ _ _ _ _ _ _ _ _ _ _
17 Bytes unused! |____?___?___?____|  $BF40-$BF  42 Bytes
~1 KB $BF68-$BFDF|_3_lines_of_text_|__ _ _ _ _ _ _ _ _ _ _
~1 KB $BB80-$____|__Text__|        |
                 | char   |        |
~2 KB $B400-$B5FF|____set_| Hires  |  $A000-$BF3F  80000 B!
                 |        |        |
                 |        |        |
      $9800-$B400|  GRAB  |________|__ _ _ _ _ _ _ _ _ _ _
      (+ 256 B)  |        |Hires   |  $9800-$$A000  ~2 KB
~7.5 KB          |________|char_set|__ _ _ _ _ _ _ _ _ _ _ 
                 |                 |
                 |                 |
                 |                 |
                 |                 |
                 |                 |  $0500-$97FF (+256 B)
                 |                 |
                 |                 |
                 |                 |
                 |                 |
                 |_________________|__ _ _ _ _ _ _ _ _ _ _
                 |__(Disc) Page 4__|  $0400-$04FF  256 B
                 |___I/O Page 3____|  $0300-$03FF  256 B
                 |_____Page 2______|  $0200-$02FF  256 B
                 |______Stack______|  $0100-$01FF  256 B
                 |____Zero Page____|  $0000-$00FF  256 B

Technially, the 2KB charset has two parts,
first 1K normal character definition,
then  1K alt graphics definition.

But each of those starts with 256 Bytes unused
(chars 0-31, 128-159).

So for program space we have $0500-$98FF=
= 3788 bytes
=  148 pages
=   37 KB
 
------------------------------------------------------------
    | $0501   | asmstart $1562 
    |_________|_______________________________________
    |         |                                       |
    |  basic  |                                       |
    | starter |                                       |
    |_________|_______________________________________|
    |
```

### Overview of MeteoriC's memory

This same information can be gotten in the app by pressing CTRL-V. It's very compact!

Note that this will change such that code generated will be after the library as indicated:


```
Program area to be used:  $0500-$97FF (+256 B)

(+ (- #x980 #x0500) 256) = 37888 = 37 K

ORIC BASIC says at startup: 37637


===============================================

          B A S I C   L O A D E R


BASIC start : $0501     51 Bytes BASIC overhead
  _exit     : $051a           (cc65?)

-----------------------------------------------

          L I B R A R Y   A R E A 


 (choosing library less, this will no be here)
          (things will move around)

asmstart    : $0534   830 Bytes library
bios              
runtime     : $05b4   (113 bytes)
  runtimeend: $0625
libs
  stdio
  ctype
  stdlib
  string
  math
_init        : $0872

-----------------------------------------------

          C O M P I L E D   C O D E


                    TODO:


    (this requires relocation of compiler)

-----------------------------------------------

                C O M P I L E R

            
     (which is an BNF-interpreter + rules)



BNF           
  compile    : $089c  2078 Bytes BNF parser
  rulesstart : $1091  6117 Bytes C-rules
IDE          : $2875
  editor     : $2903
  editorend  : $2b01
  ...
  memset     : $35fe
  ideend     : $3671
debug        : $3671
  debugend   : $38d7
asmend       : $38d7

-----------------------------------------------

         E X A M P L E   A R E A 


             (to be removed)

input        : $38d7  6909 Bytes for
  intpuend   : $53d1     a-z examples!

-----------------------------------------------

              C  -  L O A D E R 



         (these is still remnants of
        cc65 help routines - too big!)

          (TODO: rewrite in asm?)


C            : $54d3 
  disasm     : $54db  1574 Bytes
  prettyprint: $5b01  1956 Bytes
  printvariab: $62a5  1010 Bytes

clib         : $6697   9678 Bytes library code@
 printf 690 B                  cc65 :-(
             : $6949
             : $8c65
__MAIN_SIZE__: $92f3 ???
```


## Optimization Limitations

The compiler on this platform comes with inherent limitations; these are mostly stemming from limited memory and speed.

Great compilers run on bigger systems and does whole-program analysis; this allows them to totally restructure and specialize the code; functions called once are inlined, parameter passing removed if only passing constants; common subexpression elimination, etc.

In general, this is just not applicable on the 6502 itself. Instead we employ rules, as previously mentioned.

These rules captures common patterns in code, but ultimately can only optimize those that are easy to be identified, i.e. obvious ones.

But I'm gonna try to push it as far as it can go!


## `@words` version "numbers"

MeteoriC has a new type of minimal automatic version numbers.

One could manually use `v0.98a` or some common system like that, but I need to be able `at a glance` be able to figure out how old a version in the web-emulator is, and if my load of a new version worked. Sometimes, I need to clear the cache and kill chrome as it doesn't actually clear the cache correctly at all times!
        
For this, once could use an ISO-data ala `2025-02-17_15:33`; these are sortable so easy to keep track of in the filesystem and order (when tarred-up for distribution f.ex.).

I wanted version numbers to have the same function; tob e lexically sortable, but also be short on the screen, and even better if they were *somewhat memorable*.

Instead of ISO-dates we're using **"alphamal"**, a 5-letter counting system. It's counting minutes from the first commit of the main file of the compiler.

The version always has the form `@abcde` and starts at the MeteoriC's epoch of `250727_2211` which is `@aaaaa`. The time is in *Thailand time*. Every minute this "version" name changes, and it's automatically updated and picked up each time the compiler is compiled.

For fun, the `./mmepoc` program can print upcoming version names that are English words. If I want a specific word, I'd either have to fake it, or wait till the right time and then do the compilation.

Regarding the range: 26^5 minutes is good for 22 years.



## ...

More info coming...




# TODO: Future/Planned

* optimizations coming for non-recursive funcs, basically using static parameter locations.
* `||` 
* better handling of precedence
* generated stand-alone .tap files
* ROM-less variant of compiler and code
* permanent storage (disk), files, source code, etc
* operating system? LOL
* `#embed "file" [offset(N)] [limit(N)]` - to include binary data.
* `alloca()` with automatic deallocation(?)
* tail-recursion optimization



# IDE User Manual

The MeteoriC program "lives" in the editor. It allows for full-screen editing and compilation (currently: ORIC ATMOS).

At any point the built-in help can be viewed by:
```
^Help summary (editing, navigation, lang, symbols)
```


## Editing

Arrow keys for movement, `backspace, delete` (forward).

Emacs commands are extras:
```
line: ^Prev ^Next ^A=stArt of line ^End of line
char: ^Back ^Forward ^Delete BackSpace```
```

### Deleting text

Apart from the normal backspace and ^Delete (forward) key, there is
```
^Kill line (till end of line)
^Yank it all back
^Gclear yank buffer
```

### ORIC ATMOS

```
^T - caps toggle (ORIC style, lol)
```

## Command mode

`ESC` - toggles between editing and command mode
(CTRL functions available all the time through)

In the command mode the following commands exists, they can be used with single letter, or by combination CTRL-letter, the latter works inside the editor, too.


```
? - mini help
h - bigger help        (CTRL-H)
r - ^Run program       (CTRL-R)
c - ^Compile program   (CTRL-C)
e - error (goto error)
x - e^Xtras            (CTRL-X)
Q - quit
```

**`CTRL-X` extras menu:**
```
b - ^Buffers: show examples    (CTRL-X CTRL-B)
f - ^Files open from tape      (CTRL-X CTRL-F)

**(TODO: not working? unverified):**
s - ^Save current file to tape (CTRL-X CTRL-S)
w - ^Write/save new file as (new name)
                               (CTRL-X CTRL-W)
```


## Experimental features

```
v - ^Vinfo of compiler/program/libraries (CTRL-V)
o - ^Out - print variables
d - disasm compiled program              (CTRL-Q)

(^Garnish program (pretty print) - not accessible)
```


## TODO: 

```
Not yet: ^Search   ^J indent ^O ^Machinecode(^Q)
```


## Compilation and Running

At any time, the compile status is shown at the upper right corner:

* Yellow means it's being compiled
* Green indicate it's compiled successfully
* Red means there was an error

The compiler can be invoked from the editor with `^Compile` (CTRL-C). If it's green the compiled program can be run with `^Run` (CTRL-R).

During compilation a series of '.' and ',' are outputted, partly to know it's alive. '.' indicates a new statement, and ',' an op processed. Approximately.

When an example is loaded, it's not yet compiled, so there is no status.


## Stopping/Reset

When the compiled program is running the keyboard/interrupts are disabled. This makes the ORIC run as fast as it can, normally in ORIC BASIC, the keyboard is scanned continuously, possibly slowing down the computer by 5-10%! In our case, this means that `CTRL-C` cannot be used to break execution. Instead, to `break`, you can use the `NMI-button` on ORIC. It's *inconveniently* located under the machine, LOL. If you have *LOCI* or other external storage devices they may have more convenient reset button.

`NMI` can also be used to reset the compiler. It resets the stack to "zero" and returns to the command mode. The edited program remains in memory, however, if your program has run amok it may have overwritten essential data and program code.


### Compilation errors

When there is a compilation error, typically there is no "human readable" error message. Instead, the code is printed till and beyond the error, the point where the error occurred is marked with red background and white text. This is as far as it got. Recursive descent parsers are known to have trouble generating clear indicates of what went wrong, as they typically backtrack. But I found this to be usable information. Don't forget to red the code before and after.

Undefined-defined variables will be high-lighted, that's easy. However, missing things like ';' or even "} when(...);" may not be all clear.

There are no clear-text error, as they take precious space, instead a `%L` code may be printed and error displayed.

**Documented errors:**

* `%L` - Local(/parameter) variables cannot be used in this context. Typically, a recursive/safe function when calling other functions will have their parameters push, thus they may not have a fixed address. (TODO: maybe allow for `_register` marked functions)
* `%E` - unexpected End of input
* `%Z` - Zero 
* `%I` - illegal variable TODO: maybe not used anymore?
* `%F` - general Failure; Maybe elect new General?

**Other compiler %Errors**

If there is an error a newline '%' letter error-code is printed, this error code is most likely a compiler code/rule error. Please report! Screenshot and code that you're compiling.

* `%S` - Stack messed up during compilation
* `%R` - Rule error; unexpected rule popped from stack

**Runtime errors**

Traditionally in C language, very little error checking is performed. It's mostly up to users to add `assert(...)` statements. Some more modern compilers like `zig` may insert array index checking, like `PASCAL` used to do.

When running the compiled program, few situation can be detected and cought, depending on flags set.

* `%S` - Stack overflow. Sentinel byte at bottom of stack disturbed, not safe to return from (recursive) function.
* `%M` - Malloc, not enough memory (amount requested shown)
* `%A` - Assert violation. Source location reported. e) in the IDE will go to it!

## Debugging

Debugging is limited. "print"-style debugging is your friend. LOL.

### Tracepoints

Putting a single "_" surrounded by spaces will create a "tracepoint". This is where the execution stops temporary, the source code line is printed with the next part of the line to be executed highlighted in green color. 

TODO: inprogress: any integral variables that changed since last tracepoint will be printed with their old and new values.

### Breakpoint

"_" is a precursor for real "breakpoints"; these will probably be "__" and will stop and allow you to do *some* inspections and ops.

Stay tuned!

### Trace functions

Functions such marked ("__trace") will be tracable--showing each entry call+parameters and the corresponding return value. Nested function calls will be outlined with indentation.



# Libraries

As mentioned, one of the benefits of C is the all present `stdlib` standard libraries. These are rather minimal expectations and were maybe at the time a giant step forward. Today, they may be consider minimal. They are basic but very handy.


## Bios

In the tradition of `SectorLisp`, `SectorForth`, etc, we assume a prevalent always present `BIOS`. The basic routines would be `putchar(char` and `getchar()`. Putchar has the additional expectation to make a newline with the `'\n'` char, which on most "terminals" involves the sequence `CR LF`. For ORIC I've added a `TAB` (`'\t'`).

These take up anything from 0 bytes (library less) to 80ish bytes, and maybe 300 bytes for ROM-less.


## Standard libraries

Typically, in a C program, you'd include the appropriate library to use it. C has become more strict in this regard. You can do this in MeteoriC, too. However, currently, any line starting with `#` is ignored!

In the future, these includes will determine which libraries are included, so you'll be able to select only what is needed.

```
#include <stdio.h>
```

You can view the current size at the bottom of the screen of `CTRL-V`.

```
There are 9 libraries relevant to ORIC/6502
- bios      :  72 B - getchar putchar
- misc      :  31 B - nl spc clrscr routines i.e. putchar(' ')
- runtime   :  91 B - runtime routines (RECURSION)

- stdio.h   : 114 B - putu puth putz puts (putd)
 (printf.h) :       - not yet, maybe as inline!
- ctype.h   :  97 B - isdigit is???
- stdlib.h  :  77 B - 
- string.h  : 164 B - strlen strcpy...
- libmath.h :  41 B - * (multiplication) TODO: div/mod
- assert.h  :       - TODO: in progress
```


## #include <stdio.h>


```
getchar()               // returns char from keyboard
putchar(c);             // prints c to terminal
putcraw(c);             // 

puts(s);                // print string and \n
fputs(s, stdout);       // print string and NO \n
putz(s);                // EXTENDED: same as fputs

              // takes user input, with limited editing
              // (only backspace works)
char* fgets(buff,size,stdio);
              // EXTENDED: edit existing string
              // (cursor at end of string at start)
char* fgets_edit(buff,size,stdio); 



; getline(&buffer,&len,stdio); // TODO: -> count bytes
; char* readline(char* prompt); // TODO: history?



PRINTF SUPPORT - nah, not yet (see next section)

However, these works!

printf("%u", X);        // == putu(X); (print word)
printf("%x", X);        // == puth(X); (print hex)
printf("%s", X);        // == putz(X); (string no newline) 
fputs(X, stdout);       // == putz(X0; (string+no newline)
puts(X);                // == puts(X); (string+newline)

// if SIGNED support has been enabled
printf("%d", X);             // == putd
```


### Formatting strings ("printf")

`printf` is notarious "heavyduty", both in code size and in low speed. Most compilers clock well over 1000 bytes for the implementation, and quite often printing simple integers becomes really slow.

#### Byte Size Comparison

These are at best "guestimates" by Google AI:

**cc65:** A full printf implementation typically consumes ~1,200 to 1,500 bytes. Because cc65 uses a hand-optimized assembly library, it is relatively compact for its feature set, but it still represents a significant portion of a 64KB memory map.

**vbcc:** The printf footprint is generally larger, often exceeding 2,000 bytes when including full floating-point support. However, vbcc is highly modular; if you don't use floats or complex formatting, the linker can sometimes strip it down to a more manageable size.

**Oscar64:** Known for extreme optimization, its printf is often the smallest among modern 6502 C compilers, typically clocking in at under 1,000 bytes for a standard integer-only build. It uses aggressive code analysis to eliminate unused formatting logic during the compilation phase. 

### MeteoriC variant

In MeteoriC we try to avoid this problem. We've already introduced nearly optimial and really cheap `putu() putd() putx() putz() puts()` printing functions. However, for formatting, that's another game. This is the first attemp capturing the formatting capabilities, on the cheap, saving both memory and speed. Our library formatting functions implementation is about 250 bytes!

Here is a sketch of these new formating functions; results shown on the left, and on the right, we show the equivalent `printf`:s.


```
printf has the form: printf("%[-][0][<w>][.<p>][u|d|x|s|c]<post>", ...);

instead we use:

    1234 = p(recision) = max chars to print from string/min from number!
 1234567 = w(idth)     = minimal width of the whole field

== numbers ==

"42     " : putfu(42  ,-7, 0, "<post>"); == printf("%-7d<post>", var);
"     42" : putfu(42  , 7, 0, "<post>"); == printf("%7d<post>", var);
"4711   " : putfu(4711,-7, 0, "<post>"); == printf("%-7d<post>", var);
"   4711" : putfu(4711, 7, 0, "<post>"); == printf("%-7d<post>", var);

"-1     " : putfd(-1,-7, 0, "<post>");   == printf("%-7d<post>", var);
" -32767" : putfd(-32767,7,0,"<post>");  == printf("%7d<post>", var);

== strings ==

"    foO" : putfs("foO", 7, 0, "<post>"); == printf("%-7.3s<post>", var);
"foO    " : putfs("foO",-7, 0, "<post>"); == printf("%-7.3s<post>", var);

"     fo" : putfs("foO", 7, 2, "<post>"); == printf("%-7.3s<post>", var);
"fo     " : putfs("foO",-7, 2, "<post>"); == printf("%-7.3s<post>", var);

These are still TODO: (leading zeroes):
"   0042" : putfu(42, 7, 4, "<post>");    == printf("%7.4u<post>", var);
"0000042" : putfu(42, 7, 7, "<post>");    == printf("%07d<post>", var);
"0042   " : putfu(42,-7, 0, "<post>");    == printf("%-7.4d<post>", var);
"002a   " : putfx(42,-7, 0, "<post>");    == printf("%-7.4d<post>", var);
"   002a" : putfx(42, 7, 0, "<post>");    == printf("%7.4d<post>", var);

See file: Input/putf-formatting.c
```
These are compiled to quite efficient code, and the post string is put inline after the call minimizing the overhead. Basically, you could say tha
t we'd pre-parsed the format string already.

**NOTE:** the `w` and `p` numbers can not be bigger than 127, as we use a single byte to process these. In practice this shouldn't be a problem unless one relies on printf-abuse, LOL. I think it's been proven `printf` (in gcc?) is Turing-complete!

In the future, we may automate the generation of this code from a normal `printf(format, ...)`.


## # #include <ctype.h>

```
Inline-able (if even if LIBRARYLESS!)
- isdigit
- isalpha
- isspace
These are "all-or-nothing"
- isspace
- isxdigit
- isdigit
- isalnum
- isalpha
- isupper
- islower
- ispunct
- tolower
- toupper

```


## #include <stdlib.h>

The cursory heap manager is just a stop-gap implementation. Use `xmalloc()`, unless your program handles 0 returned from `malloc()`. On modern *big* computers, memory allocation almost never fails, but you'd be suprised how fast you can run out of memory on a 6502!

The current `malloc()` just allocates bytes linearly directly after the program. As it's limited by ORIC ATMOS 16KB ROM, 8KB HIRES screen, 2KB CHARSET, and 1K of low memory (ZP, stack, page 2, IO page, reserved page 4).

See section on Implementation for a more clear memory map.

```
xmalloc(n) - allocate; stop and give error if not enough memory
free(p)    - TODO: currently, does nothing!
realloc(p) - TODO: hmmmm
malloc(n)  - return 0 if not have enough (CHECK!)
release(p) - release all memory allocat since p malloc:ed!

Numbers randomization

srand(n)    - set random seed, default 1
rand()      - get pseudorandom number
```
Note: `srand(0)` won't give any numbers, just 0...
NoteII: `rand()` is simple but gives almost all numbers "except" numbers < 10 or so... LOL.


## #include <string.h>

These are mostly working... LOL

```
strlen
strcpy
strcat
strchr
stpcpy
strcmp - untested!
strstr - TODO:

```


== #include <libmath.h>

For `int`egers, and `word` there aren't many functions that are defined. However, `*` multiply is implemented in the library.

```
* - multiply - DONE - tested
/ - divide   - TODO:
% - mod      - TODO:
```


## #include <assert.h>

For debugging, and writing good code, one may need to "assert" the assumptions. If a value isn't supposed to be zero, like `malloc`-allocation. You can write an assert.

Here we show an assert that doesn't allow the value to be 13:
```
(this is example d, try it out `^L d`)

// debug assert(13) test!
#include <assert.h>

word i;

word main() {
  puts("-- before assert");
  i=0;
  while(++i) {
    putu(i); putchar(' ');
    // don't become 13!
    assert(i-13); 

  }
  puts("--- after assert: Won't get here");
}
```
The program will output the numbers 0 to 13 then it'll give a runtime error and move to display the offending line! Press key `e` to the error in the source.


## Other libraries for consideration

```
- stddef.h  :       - nah
- limits.h  :       - nah (INT_MAX INT_MIN, ffs)
- system.h  :       - nah (exec? maybe need an OS)
- unistd.h  :       - nah (complicated file system stuff)
```


## ORIC ATMOS API

These routines provide a smile parameter passing interface and calls the BASIC ROM routines.

Refer to the ORIC ATMOS MANUAL for parameters.

```
- graphics.h: TODO: ORIC ATMOS optimized graphics
```

```
GRAPHICS: x=0..239 y=0..199 c=0..2
  hires()
  text()
  clrscr()
  curset(x, y, c)
  curmov(dx, dy, c)
  draw(dx, dy, c)
  circle(r, c)
  point(x, y)
  hchar(...)
  fill(...) 
  paper(0-7)
  ink(0-7)
  pattern(0-255)
```

```
SOUND:
  play(...)
  music(...)
  sound(...)
  ping(), shoot(), zap(), explode(),
  tick(), tock()
```
  
Not done

```
FILE:
  ; cload(...) - TODO
  ; csave(...) - TODO
  cwrite(0..255)
  cread()->0..255 - TODO: erh, should be function
  ; cwritehdr() - TODO
  ; creadsync() - TODO
```


## Library-less

The library is useful, but adds up to about 600 bytes if fully included. It is possibly to compile programs using some known library functions without incurring any library overhead!

This is known as library-less. The compiler is then instructed to generate inline code instead. This is possible for printing a string, memcpy() and few other basic operations, like isalpha().

On ORIC ATMOS, and other computers,the BASIC ROM can be used as a "BIOS" of sorts. It may loose some functionality like '\n' when embedded in a string that would be expected to 

To incur as little storage overhead as possible, the compiler can, on ORIC ATMOS using only the BASIC ROM dispense with overhead of the "fancy BIOS", that "corrects" some issues.

In this mode, the compiler maps some common simple idioms to
direct code using the BASIC ROM.

```
INPUT & OUTPUT
- putchar(' ')             // (jsr spc)
- putchar('\n')            // works! (jsr nl)
- putchar(X)               // \n does not do CR LF
- putz(S)                  // ONLY putz not puts
  (no printing numbers unsigned/decimal/hex)
```

```
MEMORY STUFF
- peek(A) -> byte
- poke(A, byte)
- deek(A) -> word          // ORIC ism! (word read)
- doke(A, word)            // ORIC ism! (word write)

- memset(arr, const, const)   // inline 10 Bytes! (<256)
- memset(EXPR, EXPR, EXPR)    // Generic: inline!
- BZERO(ARR)                  // inline 10-26 Bytes!
- BFILL(ARR, val)             // inline 10-28 Bytes!

- memcpy(CONST, CONST, const) // const<256 => 14 B!
- memcpy(X,X,X)               // inline    => 23 B


Comparision of different "fill methods":
(ARR is array >= 256 B, arr is < 256 B)

  52 B:  i=0; do ARR[i++]=const; while(i<CONST);
  41 B:  memset(ARR, expr, expr);
  28 B:  BFILL(ARR, const);
  26 B:  BZERO(ARR);
  10 B:  BZERO(arr);                // bytes <256 !
  10 B:  BFILL(arr, const);         // bytes <256 !
  10 B:  memset(arr, const, const); // bytes <256 !


equivalent C code:

  #define BFILL(arr,val) memset(arr,val,sizeof(arr))
  #define BZERO(arr)     BFILL(arr,0)
```
You may ask why many of these functions are inlined. One reason is to minimize overhead; if not needed then it won't be loaded minimizing library overhead! Second, as can be seen, if using fixed sizes or arrays directly which are < 256 bytes in size the generated inline code is 10 bytes only. Using a library; just the function call, if generic, would generate 24 bytes to pass 3 parameters and do an `jsr`. And that's not even considering the extra cycles by function call and crappier memory access methods.


From <ctype.h>, the follwoing functions can be generated inline:
```
CTYPE! (minimal)
- isdigit()              // inline 11 B
- isalpha()              // inline 13 B
- isspace()              // inlien 8  B
```
That is slightly higher than using subroutines, however, if all of `<ctype.h>` would be included that would take about 100 bytes.

A very simple malloc is inlined, basically just giving out memory directly after the program. No checks, and free() doesn't do anything/reclaim memory.
```
STDLIB
- xmalloc(X)               // gives pointer after code
- malloc(X)                // gives pointer after code
- free(X)                  // does nothing
(these are like sbrk, just increase a pointer)
NOTE: will most likely crash the IDE
      (TODO:? use for stand alone code generated)
```

On ORIC ATMOS, all ORIC API functions are available without any extra cost.

**TODO: conio.h** - these are not implemented yet
```
These are the functions available in CC65:
2.12 conio.h

bgcolor(char c)
bordercolor(char c)
cclear(char len)
cclearxy(char x, char y, char len)
cgetc()
chline(char len)
chlinexy(char x, char y, char len)
clrscr()
cprintf(char* format, ...)
cputc(char c)
cputcxy(char x, char y, char c)
cputs(char* s)
cputsxy(char x, char y, char* s)
cursor(char on)
cvline(char len)
cvlinexy(char x, char y, char len)
gotox(char x)
gotoxy(char x, char y)
gotoy(char y)
kbhit()
revers(char on)
screensize(char* x, char* y)
textcolor(char c)
vcprintf(char* format, va_list *va)
wherex()
wherey()
```



# Command line version (sim65 only)

The sim65 compiled version is meant for testing and faster development. It currently doesn't have an editor.

* TODO: ANSI/vt100 editor
* Using run wrapper `./oric-terminal` translates ORIC color codes
* TODO: Interactive commands (mapping bad)
* command line options

## Command line options

If no arguments are given it enters interactive command mode. Currently, it's limited, and cumbersome (TODO: redesign).

```
Usage: ./mc [-f filename] [-c] [-r[times]] [filename ...]

filename
      If a filename is given without
      options, it's loaded, compiled,
      and run once.
        Several files can be processed.

-f filename
       Loads a file as input to buffer.

-c     Compiles current buffer.
-r[N]  Runs program N times, default 1.
-d     Disasm of program.
-Q     Quit (stop processing parameters)

-pN    Print Names (defined order)
-pe    Print env debug info (reverse vars & addr)

-sNAME Snapshot   series: snap-NAME-00.ram
-s_ACT Snapshot instance: snap-NAME-0?_ACT
```


# Licence

The intent of this section is to make it clear that no claims are made on the user's compiled program rights, or to its source code. It's considered wholly, and fully owned by its author.

MeteoriC compiler is released under AGPLv3 or later version. Bugfixes are welcome and feature improvements. These **must** be under AGPL, or may otherwise not be combined with this work. Some compilers, limit the rights of users to fix bugs, or modify the compiler, MeteoriC does not.

Refer to the file `LICENSE`in the root directory of the project for the full AGPL license.

**Summary of Rights**
- **Does a user have to share their own app's source code?<br/>** No (because of the Exception).  
- **Does a user have to share changes they made to your compiler?**<br/>Yes (because of the AGPL). 
- **Does the "Binary-to-Binary" process trigger AGPL?**<br/>No, the Exception explicitly permits this "incorporation" without forcing the license on the output.

To avoid confusion, "The contents of the `/Input` or `/Examples` directory are excluded from the AGPL and are dedicated to the public domain, see the file `UNLICENSE."

## Goals

**Protects the Compiler:** If someone modifies the compiler’s source code, its optimization engine, or adds a new feature to the libraries, that code and changes must be shared under AGPLv3.

**The "Source Reading" Distinction:** By specifying that the exception applies to the act of compilation, we clarify that the user's code isn't "infected" just because it passed through the compiler.

**Handles Binary-to-Binary:** The compiler uses a binary to generate a new binary; this exception covers those "binary artifacts" that end up inside the user's final product. Without this, those binary chunks would carry the AGPL "copyleft" requirement into the user's program.


## The Compiler

"This program is free software: you can redistribute it and/or modify it under the terms of the GNU Affero General Public License as published by the Free Software Foundation, version 3, with the Exception for Generated Output."

Refer to the file `LICENSE`in the root directory of the project for the full AGPL license.

## "The MeteoriC Compiling Exception"

More formallly; about Your Program, and the binary output from the MeteoriC compiler from Your Program:

"As a special exception to the AGPLv3, the act of compiling, linking, or otherwise using this Compiler to generate an executable binary from user-provided input does not, by itself, cause the resulting binary to
be covered by the GNU AGPL.

This exception applies even if the Compiler, in the process of generating the binary, incorporates pre-compiled machine code or binary artifacts from the Compiler’s standard libraries into the output.

This exception does not grant permission to distribute or modify the source code of the Compiler or its libraries except under the terms of the AGPLv3. It is intended solely to ensure that the user's own original code and the resulting machine-readable binaries remain the property of the user."

This is exception is also spelled out in `LICENSE_EXCEPTION"-file.


# That's it folks!
