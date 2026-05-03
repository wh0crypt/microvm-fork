# MicroVM — Level 4

Your VM has a stack. Brainfuck doesn't — it has a **tape**. Think of it like a long row of boxes, each holding one number, with an arrow pointing at the current box.

```
tape:  [0] [0] [0] [0] [0] [0] ...
        ^
        dp (data pointer)
```

You're going to **replace the stack** with this. Remove `PUSH`, `ADD`, `PRINT_TOP` and the `stack[]` array entirely. Replace them with:

| Byte | Name | Behavior |
|------|------|----------|
| `0x01` | `INC` | `tape[dp] += 1` (wraps: 255 + 1 = 0). `pc += 1` |
| `0x02` | `DEC` | `tape[dp] -= 1` (wraps: 0 - 1 = 255). `pc += 1` |
| `0x03` | `RIGHT` | `dp += 1`. `pc += 1` |
| `0x04` | `LEFT` | `dp -= 1`. `pc += 1` |
| `0x05` | `PRINT` | Print `tape[dp]` as an ASCII character. `pc += 1` |
| `0x06` | `READ` | `tape[dp] = getchar()`. `pc += 1` |
| `0x07` | `JUMP_IF_ZERO` | If `tape[dp] == 0`, set `pc` to `program[pc+1]`. Otherwise `pc += 2`. |
| `0x08` | `JUMP_IF_NOT_ZERO` | If `tape[dp] != 0`, set `pc` to `program[pc+1]`. Otherwise `pc += 2`. |
| `0x00` | `HALT` | Stop. |

Your VM state is now:

```c
uint8_t tape[256];   // all zeroes at start
int dp = 0;                // data pointer
int pc = 0;                // program counter
```

Notice two things:

1. Every value-changing operation works on `tape[dp]` — the cell the arrow is pointing at. To work with a different cell, you move the arrow with `RIGHT`/`LEFT`.

2. We now have `JUMP_IF_NOT_ZERO` as well as `JUMP_IF_ZERO`. In level 3 you had to use two jumps to make a loop. With both of these, loops become easier — you'll see why below.

**Why two jump instructions?**

A brainfuck loop works like this:

- `[` means "if current cell is zero, skip past the matching `]`"
- `]` means "if current cell is NOT zero, jump back to the matching `[`"

So `JUMP_IF_ZERO` is `[` and `JUMP_IF_NOT_ZERO` is `]`. We're still using numeric jump targets for now — level 5 handles the bracket matching.

**Test program 1 — print 'H' the hard way**

'H' is ASCII 72. Instead of `PUSH 72`, we have to **increment the current cell 72 times**, then print. Painful but simple:

```c
uint8_t program[] = {
    1, 1, 1, 1, 1, 1, 1, 1, 1, 1,   // 10 x INC
    1, 1, 1, 1, 1, 1, 1, 1, 1, 1,   // 20
    1, 1, 1, 1, 1, 1, 1, 1, 1, 1,   // 30
    1, 1, 1, 1, 1, 1, 1, 1, 1, 1,   // 40
    1, 1, 1, 1, 1, 1, 1, 1, 1, 1,   // 50
    1, 1, 1, 1, 1, 1, 1, 1, 1, 1,   // 60
    1, 1, 1, 1, 1, 1, 1, 1, 1, 1,   // 70
    1, 1,                             // 72
    5,                                // PRINT (prints 'H')
    0                                 // HALT
};
```

That's awful. Real brainfuck programs don't do this — they use loops.

**Test program 2 — print 'H' with a loop**

The idea: put 8 in cell 0, put 9 in cell 1, then loop: add 9 to cell 1 while decrementing cell 0. When cell 0 hits zero, cell 1 holds 8 × 9 = 72 = 'H'.

Trace it on paper:

```
Start:     tape: [0] [0]     dp=0

Set up cell 0 = 8:
INC x8     tape: [8] [0]     dp=0

Move to cell 1:
RIGHT      tape: [8] [0]     dp=1

Set up cell 1 = 9:
INC x9     tape: [8] [9]     dp=1

Now the loop. Go back to cell 0:
LEFT       tape: [8] [9]     dp=0

--- loop start (pc = 18) ---
JUMP_IF_ZERO to pc 28 (exit)
    tape[0] is 8, not zero, so keep going.
DEC        tape: [7] [9]     dp=0
RIGHT      tape: [7] [9]     dp=1
INC x9     tape: [7] [18]    dp=1     <- added 9 to cell 1
LEFT       tape: [7] [18]    dp=0     <- back to counter
JUMP_IF_NOT_ZERO to pc 18 (loop start)
    tape[0] is 7, not zero, so jump back.
--- loop continues until cell 0 is 0 ---

After 8 iterations:
           tape: [0] [72]    dp=0

JUMP_IF_ZERO fires, skips to pc 28.
RIGHT      dp=1
PRINT      prints tape[1] = 72 = 'H'
HALT
```

Write this as a bytecode array. The hardest part is getting the `pc` values right for the two jumps. Count every byte carefully — `INC` is 1 byte, but `JUMP_IF_ZERO` is 2 bytes (opcode + target).

**Hint**

```c
uint8_t program[] = {
//  pc
    1,1,1,1,1,1,1,1,          //  0-7:   INC x8 (cell 0 = 8)
    3,                         //  8:     RIGHT
    1,1,1,1,1,1,1,1,1,        //  9-17:  INC x9 (cell 1 = 9)
    4,                         // 18:     LEFT
    // -- loop start at pc 19 --
    7, ???,                    // 19:     JUMP_IF_ZERO ???  (exit loop)
    2,                         // 21:     DEC
    3,                         // 22:     RIGHT
    1,1,1,1,1,1,1,1,1,        // 23-31:  INC x9 (add 9 to cell 1)
    4,                         // 32:     LEFT
    8, ???,                    // 33:     JUMP_IF_NOT_ZERO ??? (loop back)
    // -- loop end --
    3,                         // 35:     RIGHT
    5,                         // 36:     PRINT
    0                          // 37:     HALT
};
```

Fill in the two `???`. One points forward to after the loop, one points backward to the top of the loop.
