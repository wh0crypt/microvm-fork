# MicroVM

MicroVM is a self-guided path for learning how computers work by building a tiny virtual machine, then gradually turning it into a [brainfuck](https://en.wikipedia.org/wiki/Brainfuck) interpreter.

The goal is not to memorize brainfuck. The goal is to understand the pieces underneath it:

- instruction fetch/decode/execute
- program counters
- stacks
- control flow
- memory/tape models
- parsing source code into executable behavior

Each level is a small extension of the previous one.

## Prerequisites

You should have a basic programming environment available:

- a text editor or IDE
- a terminal/shell
- a compiler or runtime for the language you want to use
- if following the examples directly, a C environment with `stdint.h`, `uint8_t`, `putchar()`, and `getchar()`

A little experience with variables, arrays, loops, and functions is helpful. You do not need to know how virtual machines or interpreters work yet.

The examples are written in C-style byte arrays, but the project can be implemented in any language that lets you work with numbers, arrays, and characters.

## How to use this project

Work through the levels in order:

1. Read the level document.
2. Implement or extend your VM.
3. Run the test programs.
4. If something breaks, trace the VM by hand: write down `pc`, stack/tape contents, and output after each instruction.
5. Move on only when the current level works.

Starter files, build commands, and a complete example setup will be added later.

## Levels

| Level | File | Main idea |
|-------|------|-----------|
| 1 | `level1.md` | Fetch/decode/execute loop and a program counter |
| 2 | `level2.md` | Stack operations and arithmetic |
| 3 | `level3.md` | Jumps, conditionals, and loops |
| 4 | `level4.md` | Brainfuck-style tape memory |
| 5 | `level5.md` | Brainfuck source parsing and bracket matching |

## Expected outputs

Use these as quick checks:

| Level | Program | Expected output |
|-------|---------|-----------------|
| 1 | Hello program | `Hello, World!` |
| 2 | Stack print program | `Hi` |
| 2 | ADD program | `C` |
| 3 | Infinite loop program | prints `A` forever |
| 3 | Counter loop challenge | `AAA` |
| 4 | 72 `INC`s | `H` |
| 4 | Multiply loop | `H` |
| 5 | Simple 72-plus brainfuck program | `H` |
| 5 | Real brainfuck multiply loop | `H` |
| 5 | Real brainfuck Hello World program | `Hello World!\n` |

## Suggested VM behavior

For a learning project, it is useful to make errors loud instead of mysterious.

Consider checking for:

- invalid opcodes
- stack overflow or underflow
- tape pointer moving before cell `0` or beyond the end of the tape
- jumps to invalid program positions
- programs that never halt
- unmatched `[` or `]` in brainfuck source
- `getchar()` returning end-of-file

The levels use `uint8_t` values, so arithmetic should wrap: `255 + 1` becomes `0`, and `0 - 1` becomes `255`.

## Debugging tip

Add an optional trace mode that prints the VM state before or after every instruction, for example:

```text
pc=8 op=JUMP_IF_ZERO dp=0 tape[0]=8 tape[1]=0 output=""
```

This makes bugs in jump targets, stack order, and tape movement much easier to see.
