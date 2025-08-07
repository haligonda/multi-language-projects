# Why Projects Often Use Multiple Programming Languages

## Overview

For simplicity, let's start by considering only system-level programming languages that compile down to machine code:

- **Go** - Fast compilation, garbage collection, built for concurrency
- **Rust** - Memory safety, zero-cost abstractions, systems programming
- **C** - Low-level control, minimal runtime, widely supported
- **C++** - Object-oriented, performance-critical applications, extensive libraries

## The Challenge

Generally, each programming language has its own dedicated compiler. So we can't take a Rust file and compile it using the Go compiler.

> **Question:** How do some projects consist of different programming languages when most programming languages have separate compilers, runtimes, and memory models? How can they coexist in the same binary?

## Example: Compiling C Code

Let's take the example of a compiler for C files -- **GCC** (GNU Compiler Collection). 

To compile a C file, you run:

```bash
gcc main.c -o main
# Syntax: compiler source_file -o output_executable_name
```

### Process Overview

The compilation and execution process involves two simple steps:

#### 1. **Compile the source code:**
```bash
gcc main.c -o main
```

#### 2. **Run the executable:**
```bash
./main
```

---

*This creates an executable binary that can run independently of the source code.*

## What Happens Under the Hood?

From our perspective, it's just two steps: one to compile the program and another to run it. But under the hood, the compiler is doing a whole lot more.

Internally, **GCC** goes through **4 main phases** to turn a C file into a working executable:

### 1. **Pre-processing**
This prepares the source code by:
- Removing comments
- Expanding macros
- Resolving conditional compilation
- **Crucially:** Resolving `#include` statements

When you use `#include`, the C pre-processor replaces that line with the contents of the header file and all the headers that it includes, effectively inserting that code into our file before compilation begins. The output is still C code, but pre-processed for the next step.

### 2. **Compilation** (Source → Assembly)
Next comes compilation, but **not directly into machine code**. Instead, the pre-processed code is translated into **assembly language** - which contains the instructions that the computer will execute, but still in a human-readable format.

> 💡 **Myth Busted:** A compiler doesn't always convert source code directly into machine code. In fact, many compilers convert source code into an intermediate representation like assembly or even into another programming language.

![Myth Busted](./mythbusters.png)

### 3. **Assembly** (Assembly → Machine Code)
The third step involves the **assembler**, which is technically another compiler. It takes the human-readable assembly code from the previous phase and translates it into **machine code** - the ones and zeros your CPU understands. The result is called an **object file**.

But here's the catch: **This object file isn't runnable yet.** GCC still needs to resolve the position within the binary where functions will be placed.

#### Example: A Simple Hello World Program
Let's take an example where we're just printing text to the console:

```c
#include <stdio.h>

int main(void) {
    printf("Hello, World!");
    return 0;
}
```

Here, the actual implementation of the `printf` function lives in the **C standard library**. So that library also needs to go through the same compilation steps.

### 4. **Linking** (Object Files → Executable)
Finally, **linking** happens. At this stage, we may have multiple object files:
- Some from our code
- Others from external libraries we included during development

The **linker's job** is to combine all these object files into a single, self-contained executable.

You might be wondering what's the point of this all modularization? Why break the process into so many phases if the compiler could just go directly from source to executable? Well, the reason we don't normally see all these intermediate steps is because compilers like GCC are configured by default to hide them. They just show you the final result, the executable.
```bash
gcc main.c -o main
```
```bash
./main.exe
```

But with right flags we can expose all those phases. 
```bash
gcc main.c -o main -save-temps
```
For example, using GCC, if you compile a program and add the `-save-temp` flag, you'll get not just the final executable, but also all the intermediate files.

main.c -> Pre-processor -> main.i -> Compiler -> main.o -> Assembler -> main.s/main.dll -> Linker -> main.exe

We can even stop the process at specific stage. For example the `-S` flag makes the process stop after generating assembly.
```bash
gcc main.c -S -o main
                                                          ________________________    
                                                        # |                      |
# main.c -> Pre-processor -> main.i -> Complier -> main.o X Assembler -> Linker  |___ -> main.s
```

This is incredibly useful where you might want to see how high level C code translate to assembly or machine code.

In professional environments, this is also used to inspect performance critical code. You can look at the generated assembly to verify whether the compiler is producing efficient instructions.

Even more interesting we can start from any phase in the pipeline. We can pass GCC the assembly file and simply tell it assemble and link it.
```bash
gcc main.s -o main
```

---

## The Power of Modular Compilation

You might be wondering: what's the point of all this modularization? Why break the process into so many phases if the compiler could just go directly from source to executable? 

Well, the reason we don't normally see all these intermediate steps is because compilers like **GCC** are configured by default to hide them. They just show you the final result - the executable.

```bash
gcc main.c -o main
```
```bash
./main
```

### Exposing the Compilation Phases

But with the right flags, we can expose all those phases:

```bash
gcc main.c -o main -save-temps
```

For example, using **GCC**, if you compile a program and add the `-save-temps` flag, you'll get not just the final executable, but also all the intermediate files.

**Complete Pipeline:**
```
main.c → Pre-processor → main.i → Compiler → main.s → Assembler → main.o → Linker → main.exe
```

### Stopping at Specific Stages

We can even stop the process at specific stages. For example, the `-S` flag makes the process stop after generating assembly:

```bash
gcc main.c -S -o main.s
```

**Partial Pipeline:**
```
main.c → Pre-processor → main.i → Compiler → main.s ❌ (Stop here)
```

This is incredibly useful when you want to see how high-level C code translates to assembly or machine code.

In professional environments, this is also used to inspect performance-critical code. You can look at the generated assembly to verify whether the compiler is producing efficient instructions.

### Starting from Any Phase

Even more interesting: we can start from any phase in the pipeline. We can pass **GCC** an assembly file and simply tell it to assemble and link it:

```bash
gcc main.s -o main
```

## Multi-Language Projects in Action

This is huge because it means we can write part of our code in assembly, pass it to the compiler at different stages, and then the linker will take care of mixing them together into an executable file.

This already starts to answer our original question. Let's walk through an example:

### Example: C + Assembly Performance Optimization

Suppose we need to write a program that calculates how many prime numbers exist between zero and a given number, and we want it to be as fast as possible. We could write the whole thing in C, but let's say we don't trust the compiler optimizations. So we decide to:

1. Write the heavy calculation function directly in **assembly**
2. Call it from **C**
3. Pass both files to **GCC**, which will:
   - Compile and assemble the C code
   - Assemble the assembly code  
   - Link both object files into a single executable

🎉 **Congratulations!** We just compiled a multi-language project.

### Real-World Usage

This technique is used by real-world systems like:
- **Linux kernel**
- **FFmpeg**
- **OpenSSL**
- **Many embedded projects**

They often contain **C** for most of the logic, but fall back to **assembly** when performance really matters.

## Beyond Assembly: High-Level Language Mixing

So what about mixing high-level languages instead of assembly? For example, instead of writing a function in assembly, what if we implement part of our project in **FORTRAN**?

Well, this is totally possible and actually more common than it might seem. In this case, however, we usually need multiple steps:

### Example: C + FORTRAN

```bash
# Step 1: Compile FORTRAN file to object code
gfortran -c is_prime.f90

# Step 2: Compile C file to object code  
gcc -c count_prime.c

# Step 3: Link both object files into executable
gfortran count_prime.o is_prime.o -o count_prime

# Step 4: Run the multi-language program
./count_prime
```

Unlike assembly (which is already embedded in the C compilation pipeline), **FORTRAN** has:
- Its own pipeline
- Its own compiler
- Sometimes even its own runtime dependencies

## The Linker: The Universal Connector

And by now it should be crystal clear that **the answer to our original question** - how can different languages live inside a single executable - **comes down to the linker**.

You see, the different languages involved don't even need to come from the same compiler suite as **GCC**. Take **Rust** for example:
- It has a completely different toolchain from C
- Different compiler
- Different build system  
- Different philosophy altogether

And when it comes to producing the final binary, guess what? **Rust too relies on a linker.**


