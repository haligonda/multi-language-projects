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

