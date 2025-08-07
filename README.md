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
