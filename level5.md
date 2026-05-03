# MicroVM — Level 5

Your level 4 VM is already brainfuck. The only difference: real brainfuck programs aren't arrays of numbers — they're strings like `++++++++[>+++++++++<-]>.`.

The mapping is:

| Character | Your opcode |
|-----------|------------|
| `+` | `INC` |
| `-` | `DEC` |
| `>` | `RIGHT` |
| `<` | `LEFT` |
| `.` | `PRINT` |
| `,` | `READ` |
| `[` | `JUMP_IF_ZERO` |
| `]` | `JUMP_IF_NOT_ZERO` |

All other characters are ignored (they're comments).

There's one problem: in your level 4 bytecode, `JUMP_IF_ZERO 35` says exactly where to jump. In brainfuck source code, `[` doesn't say where `]` is — you have to **find the matching bracket**.

You need to solve the bracket matching problem. Here's the approach:

**Before running the program**, scan through the entire source string. Build a table that maps every `[` position to its matching `]` position, and vice versa.

```c
// pseudocode
int match[program_length];

int depth = 0;
int opens[MAX_DEPTH];    // a stack! you know how these work now

for i = 0 to length-1:
    if program[i] == '[':
        opens[depth] = i
        depth += 1
    if program[i] == ']':
        depth -= 1
        match[i] = opens[depth]       // ] points back to [
        match[opens[depth]] = i       // [ points forward to ]
```

Look at that — it's a stack, just like level 2. You're using a stack to compile the bracket pairs. `[` pushes its position, `]` pops the matching `[` and wires them together.

After this, your VM loop barely changes. Instead of reading a numeric jump target from `program[pc+1]`, you look it up:

```c
// Level 4:
case JUMP_IF_ZERO:
    if (tape[dp] == 0) pc = program[pc+1];
    else pc += 2;

// Level 5 (brainfuck):
case '[':
    if (tape[dp] == 0) pc = match[pc] + 1;
    else pc += 1;          // note: pc += 1, not 2. no operand byte anymore.
```

**Test program — Hello World in brainfuck**

```
++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---.+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.
```

If this prints `Hello World!` your brainfuck interpreter is done.

**How to debug it**

If it doesn't work, start with these simpler programs:

1. `++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.` — that's 72 `+` signs then `.`. Should print `H`. If this doesn't work, your `INC`/`PRINT` is broken.

2. `++++++++[>+++++++++<-]>.` — this is the level 4 multiply loop as brainfuck (8 × 9 = 72). Should print `H`. If this doesn't work, your bracket matching or jumps are broken.

3. If program 2 works but Hello World doesn't, your bracket matcher probably fails on **nested** brackets. The Hello World program has `[` inside `[` — make sure your depth counter handles that.

---

That's the whole path. Five levels, each one adding exactly one concept:

1. Fetch-decode-execute
2. Stack
3. Control flow
4. Tape memory model
5. Source parsing (bracket matching)
