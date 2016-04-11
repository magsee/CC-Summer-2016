<<<<<<< HEAD
# Selfie

Selfie is a project of the [Computational Systems Group](http://www.cs.uni-salzburg.at/~ck) at the Department of Computer Sciences of the University of Salzburg in Austria.

For further information and support please refer to http://selfie.cs.uni-salzburg.at

## Build Instructions

The first step is to produce a binary that runs on your computer. To do that on a Linux or Mac OS X system use `gcc` in a terminal to compile `selfie.c`:

```bash
gcc -w -m32 -D'main(a, b)=main(int argc, char **argv)' selfie.c -o selfie
```

This produces from `selfie.c` an executable called `selfie` as directed by the `-o` option. The executable contains the C\* compiler, the mipster emulator, and the hypster hypervisor. The `-w` option suppresses warnings that can be ignored for now. The `-m32` option makes the compiler generate a 32-bit executable (which may require installing gcc-multiarch or gcc-multilib, depending on your system). Selfie only supports 32-bit architectures. The `-D` option is needed to bootstrap the main function declaration. The `char` data type is not available in C\* but required by `gcc`.

We have not tested selfie on Windows systems ourselves but users have reported success. Since the only unusual requirement is to have a C compiler installed that supports generating 32-bit binaries, compiling selfie on Windows should be straightforward to do.

## Running Selfie

Once you have successfully compiled selfie you may invoke it in a terminal as follows:

```bash
./selfie { -c source | -o binary | -s assembly | -l binary } [ -m size ... | -d size ... | -y size ... ]
```

The order in which the options are provided matters for taking full advantage of self-referentiality.

The `-c` option invokes the C\* compiler on the given `source` file producing MIPSter code that is stored internally.

The `-o` option writes MIPSter code produced by the most recent compiler invocation to the given `binary` file.

The `-s` option writes MIPSter assembly of the MIPSter code produced by the most recent compiler invocation including approximate source line numbers to the given `assembly` file.

The `-l` option loads MIPSter code from the given `binary` file. The `-o` and `-s` options can also be used after the `-l` option. However, in this case the `-s` option does not generate approximate source line numbers.

The `-m` option invokes the mipster emulator to execute MIPSter code most recently loaded or produced by a compiler invocation. The emulator creates a machine instance with `size` MB of memory. The `source` or `binary` name of the MIPSter code and any remaining `...` arguments are passed to the main function of the code. The `-d` option is similar to the `-m` option except that mipster outputs each executed instruction, its approximate source line number, if available, and the relevant machine state.

The `-y` option invokes the hypster hypervisor to execute MIPSter code similar to the mipster emulator. The difference to mipster is that hypster creates MIPSter virtual machines rather than a MIPSter emulator to execute the code.

To compile `selfie.c` for mipster and hypster use the following command:

```bash
$ ./selfie -c selfie.c -o selfie.m
```

This produces a MIPSter binary file called `selfie.m` that implements selfie.

To execute `selfie.m` by mipster use the following command:

```bash
$ ./selfie -l selfie.m -m 1
```

This is semantically equivalent to executing `selfie` without any arguments:

```bash
$ ./selfie
```

To execute `selfie.m` by hypster use the following command:

```bash
$ ./selfie -l selfie.m -y 1
```

This is semantically equivalent to executing `selfie.m` by mipster and thus `selfie` without any arguments. There is a difference in output though since mipster reports code execution profiles whereas hypster does not.

### Self-compilation

Here is an example of how to perform self-compilation of `selfie.c`:

```bash
$ ./selfie -c selfie.c -o selfie1.m -m 2 -c selfie.c -o selfie2.m
$ diff -s selfie1.m selfie2.m
Files selfie1.m and selfie2.m are identical
```

Note that at least 2MB of memory is required.

### Self-execution

The following example shows how to perform self-execution of `selfie.c`. In this case we invoke the emulator to invoke itself which then invokes the compiler to compile itself:

```bash
$ ./selfie -c selfie.c -o selfie1.m -m 4 -l selfie1.m -m 2 -c selfie.c -o selfie2.m
$ diff -s selfie1.m selfie2.m
Files selfie1.m and selfie2.m are identical
```

Note that the example may take several hours to complete. Also, an emulator instance A running an emulator instance B needs more memory than B, say, 4MB rather than 2MB in the example here.

### Self-hosting

The previous example can also be done by running hypster on mipster. This is significantly faster since hypster does not create a second emulator instance on top of the first emulator instance. Instead, hypster creates a virtual machine to execute selfie that runs concurrently to hypster on the first emulator instance:

```bash
$ ./selfie -c selfie.c -o selfie1.m -m 4 -l selfie1.m -y 2 -c selfie.c -o selfie2.m
$ diff -s selfie1.m selfie2.m
Files selfie1.m and selfie2.m are identical
```

We may even run hypster on hypster on mipster which is still reasonably fast since there is still only one emulator instance involved and hypster itself does not add much overhead:

```bash
$ ./selfie -c selfie.c -o selfie1.m -m 8 -l selfie1.m -y 4 -l selfie1.m -y 2 -c selfie.c -o selfie2.m
$ diff -s selfie1.m selfie2.m
Files selfie1.m and selfie2.m are identical
```

### Workflow

To compile any C\* source and execute it right away in a single invocation of `selfie` without generating a MIPSter binary use:

```bash
$ ./selfie -c any-cstar-file.c -m 1 "arguments for any-cstar-file.c"
```

Equivalently, you may also use a selfie-compiled version of `selfie` and have the emulator execute selfie to compile any C\* source and then execute it right away with hypster on the same emulator instance:

```bash
$ ./selfie -c selfie.c -m 1 -c any-cstar-file.c -y 1 "arguments for any-cstar-file.c"
```

You may also generate MIPSter binaries both ways which will then be identical:

```bash
$ ./selfie -c any-cstar-file.c -o any-cstar-file1.m
$ ./selfie -c selfie.c -m 1 -c any-cstar-file.c -o any-cstar-file2.m
$ diff -s any-cstar-file1.m any-cstar-file2.m
Files any-cstar-file1.m and any-cstar-file2.m are identical
```

This can also be done in a single invocation of `selfie`:

```bash
$ ./selfie -c any-cstar-file.c -o any-cstar-file1.m -c selfie.c -m 1 -c any-cstar-file.c -o any-cstar-file2.m
$ diff -s any-cstar-file1.m any-cstar-file2.m
Files any-cstar-file1.m and any-cstar-file2.m are identical
```

The generated MIPSter binaries can then be loaded and executed as follows:

```bash
$ ./selfie -l any-cstar-file1.m -m 1 "arguments for any-cstar-file1.m"
```

#### Debugging

Console messages always begin with the name of the source or binary file currently running. The emulator also shows the amount of memory allocated for its machine instance and how execution terminated (exit code).

MIPSter assembly for `selfie` and any other C\* file is generated as follows:

```bash
$ ./selfie -c selfie.c -s selfie.s
```

If the assembly code is generated from a binary generated by the compiler (and not loaded from a file) approximate source line numbers are included in the assembly file.

Verbose debugging information is printed with the `-d` option, for example:

```bash
$ ./selfie -c selfie.c -d 1
```

Similarly, if the executed binary is generated by the compiler (and not loaded from a file) approximate source line numbers are included in the debug information.
=======
Workflow
--------

* Step 0: form a team of 2-3 members
* Step 1: give your team a name
* Step 2: get a github account for each member
* Step 3: one person per team forks the [CC-Summer-2016](https://github.com/cksystemsteaching/CC-Summer-2016/fork) repository
* Step 4: add the other team members to __your__ fork as collaborators
* Step 5: each team member clones that fork
* Step 6: check out the branch [selfie-master](https://github.com/cksystemsteaching/CC-Summer-2016/tree/selfie-master)
* Step 7: add a new file called TEAM to this branch
* Step 8: list the name of your team as well as your names in the TEAM file
* Step 9: implement solutions of assignments in this branch as well (see below)
* Step 10: commit regularly and push your changes to your fork
* Step 11: to submit solutions send pull requests from your fork via github.com to [selfie-master](https://github.com/cksystemsteaching/CC-Summer-2016/tree/selfie-master)

General Requirements
--------------------

All homework solutions:

* must be implemented in C\* in the selfie.c file,
* must compile without warnings with starc and execute with mipster,
* must not break any existing selfie functionality, and
* must be ready for presentation on your machine in class.

Assignment 0: Your Team!
------------------------

__Deadline__: March 10, 10am (hard, no extensions)

Suppose your team name is *TheCompilables*. Change selfie such that it prints "This is TheCompilables Selfie" in a separate line on the console before doing anything else. All other functionality should be unaffected.

Assignment 1: Bitwise Shift Instructions
----------------------------------------

__Deadline__: March 17, 10am (hard, no extensions)

Implement in mipster the four logical bitwise shift instructions of MIPS called `sll`, `srl`, `sllv`, and `srlv` according to the <a href="https://en.wikipedia.org/wiki/MIPS_instruction_set">MIPS</a> standard. In your implementation use the selfie library functions for bitwise shifting and the selfie interface functions for decoding. Also, make sure that the selfie disassembler and debugger produce proper output for these instructions.

Assignment 2: Bitwise Shift Operators (Scanning, Parsing)
---------------------------------------------------------

__Deadline__: April 7, 10am (hard, no extensions)

Implement in starc scanning and parsing of the two bitwise shift operators of C written `<<` and `>>`. Extend the scanner and parser of starc accordingly. In particular, invent unique tokens for the new symbols and make sure that the scanner properly distinguishes `<` from `<<` and `>` from `>>`. Then, add a new production rule for the new symbols to the C\* grammar (in the grammar.md file) such that their precedence is between `+` and `<` as specified for <a href="https://en.wikipedia.org/wiki/Operators_in_C_and_C%2B%2B">C operators</a>. Finally, add a procedure for parsing the new symbols to starc according to the new production rule (hint: use a copy of the procedure for parsing simple expressions as template). Test your implementation on syntactically correct as well as syntactically incorrect C\* programs using the two shift operators.

Assignment 3: Bitwise Shift Operators (Code Generation, Self-Compilation)
-------------------------------------------------------------------------

__Deadline__: April 14, 10am (hard, no extensions)

Implement in starc code generation for the shift operators `<<` and `>>` of Assignment 2 using the logical bitwise shift instructions of Assignment 1. The semantics of both operators should be logical bitwise shift meaning that zeros are shifted in on either end (even for `>>` of negative numbers). Change the implementation of the leftShift and rightShift procedures using `<<` and `>>` instead of `*` and `/` such that the original semantics of both procedures is preserved. Avoid calling the twoToThePowerOf procedure in your solution. Finally, demonstrate that self-compilation of selfie still works and check which procedures are now the most called procedures during self-compilation.
>>>>>>> upstream/master
