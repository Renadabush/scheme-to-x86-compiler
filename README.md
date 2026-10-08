# Scheme → x86-64 Compiler

A compiler for a substantial subset of Scheme, written in **OCaml**, that translates Scheme source code into **x86-64 assembly (NASM)** and produces a native Linux executable.

Built as the final project for the *Principles of Compilation* course (Ben-Gurion University).

---

## Table of Contents

- [Overview](#overview)
- [Compilation Pipeline](#compilation-pipeline)
- [Repository Structure](#repository-structure)
- [Supported Language Features](#supported-language-features)
- [Runtime & Memory Model](#runtime--memory-model)
- [Requirements](#requirements)
- [Build & Usage](#build--usage)
- [Example](#example)
- [Testing](#testing)
- [Authors](#authors)
- [Academic Integrity Note](#academic-integrity-note)

---

## Overview

The compiler takes a Scheme program, prepends a Scheme-written standard library (`init.scm`), and compiles everything into a single assembly file. That file is assembled and linked with a hand-written runtime, giving a standalone executable.

Highlights:

- Full front-end: **reader → tag parser → semantic analysis**
- Back-end **code generator** emitting x86-64 NASM
- First-class **closures** with lexical environments
- **Tail-call optimization**
- Lambdas with **optional / variadic arguments**
- Runtime primitives written directly in assembly
- Part of the standard library bootstrapped in Scheme itself (`init.scm`)

---

## Compilation Pipeline

```
 Scheme source  ──►  init.scm + user code
                         │
                         ▼
                 ┌───────────────┐
                 │    Reader     │  S-expressions (parser combinators, pc.ml)
                 └───────┬───────┘
                         ▼
                 ┌───────────────┐
                 │  Tag Parser   │  S-expr → expr  (macro expansion, special forms)
                 └───────┬───────┘
                         ▼
                 ┌───────────────┐
                 │   Semantics   │  lexical addressing, tail-call annotation,
                 │               │  variable boxing
                 └───────┬───────┘
                         ▼
                 ┌───────────────┐
                 │ Code Generator│  expr' → x86-64 assembly
                 └───────┬───────┘
                         ▼
   prologue-1.asm + prologue-2.asm + generated code + epilogue.asm
                         │
                         ▼
                   NASM  +  ld/gcc
                         │
                         ▼
                 Native executable
```

---

## Repository Structure

| File | Description |
|------|-------------|
| `compiler.ml` | The complete compiler: reader, tag parser, semantic analysis, and code generator |
| `pc.ml` | Parser-combinator library used by the reader |
| `init.scm` | Built-in procedures implemented in Scheme; compiled together with every user program |
| `prologue-1.asm` | First part of the assembly header: definitions and macros emitted *before* generated code |
| `prologue-2.asm` | Second part of the assembly header |
| `epilogue.asm` | Assembly support code and all primitive procedures, emitted *after* generated code |
| `makefile` | Builds executables from generated assembly files |
| `readme` | Original submission info file |

---

## Supported Language Features

- **Data types:** integers, rationals, reals, booleans, characters, strings, symbols, pairs/lists, vectors, void, nil
- **Special forms:** `define`, `lambda` (simple, optional-argument, and variadic), `if`, `or`, `and`, `begin`, `set!`, `let`, `let*`, `letrec`, `cond`, `quasiquote`/`unquote`, and more
- **Closures** capturing lexical environments
- **Tail-call optimization** for proper tail recursion
- **Variable boxing** for mutated captured variables
- **Primitive procedures** (type predicates, arithmetic, list/string/vector operations, `apply`, etc.) implemented in assembly
- **Library procedures** (e.g. `map`, `append`, `apply`, ...) implemented in Scheme in `init.scm`

---

## Runtime & Memory Model

- **Tagged values:** every Scheme object is represented by a tagged machine word/pointer.
- **Constants table:** literals are laid out in the data segment at compile time.
- **Global bindings table:** each top-level `define` maps to a labeled slot in memory.
- **Closures:** a closure packs the lexical environment together with a code pointer.
- **Lexical environment extension** is handled by `environment_lexical_extend`; optional/variadic argument handling uses `stack_fix_opt`.
- **`apply-bin`** is implemented in assembly (`apply_bin_ptr_code_L`); the full `apply` is built on top of it in `init.scm`.

---

## Requirements

- Linux x86-64 (or WSL on Windows)
- [OCaml](https://ocaml.org/) (4.x or later)
- [NASM](https://www.nasm.us/)
- `gcc` / `ld`
- `make`

---

## Build & Usage

> Verify the exact target names against the included `makefile`.

1. Put your Scheme program in a file, e.g. `foo.scm`.
2. Compile it:

   ```bash
   make foo
   ```

   This runs the compiler on `foo.scm` (with `init.scm` prepended), generates `foo.asm`, assembles it with NASM, and links an executable named `foo`.

3. Run it:

   ```bash
   ./foo
   ```

Manual steps (equivalent, for reference):

```bash
# 1. Generate assembly from Scheme (inside OCaml toplevel)
#    Compiler.compile_scheme_file "foo.scm" "foo.asm"

# 2. Assemble and link
nasm -f elf64 -o foo.o foo.asm
gcc -static -m64 -o foo foo.o
```

---

## Example

`fact.scm`:

```scheme
(define fact
  (lambda (n)
    (if (zero? n)
        1
        (* n (fact (- n 1))))))

(fact 10)
```

```bash
make fact
./fact
# 3628800
```

---

## Testing

Compare your executable's output against a reference Scheme implementation (e.g. Chez Scheme or Racket) on a variety of programs covering:

- Closures and higher-order functions
- Tail recursion (large loop counts should not overflow the stack)
- Optional and variadic lambdas
- Mutation (`set!`) of captured variables
- Lists, vectors, strings, and chars
- `apply` and library procedures from `init.scm`

---

## Authors

- **Your Name** - [@your-github-username](https://github.com/your-github-username)
- **Partner Name** - [@partner-github-username](https://github.com/partner-github-username)

---

## Academic Integrity Note

This project was completed as university coursework. If you are currently enrolled in a course with a similar assignment, **do not copy this code** — doing so violates academic-integrity policies and defeats the purpose of the exercise. It is shared here for portfolio and reference purposes only.

---

## Acknowledgments

Course materials, skeleton code, and project specification by **Meir Goldberg**, *Principles of Compilation*.
