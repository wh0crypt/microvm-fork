# MicroVM — Level 3

Right now your VM can only go forward — the program counter always increases. In this level you'll make it go **backwards**, which gives you loops.

Add two new opcodes:

| Byte | Name | Behavior |
|------|------|----------|
| `0x05` | `JUMP` | Set `pc` to `program[pc+1]`. (Don't add 2 — you're replacing `pc` entirely.) |
| `0x06` | `JUMP_IF_ZERO` | Peek the top of the stack. If it was `0`, set `pc` to `program[pc+1]`. If it wasn't `0`, just do `pc += 2` and keep going. |

That's it. Two opcodes. But they change everything.

**What `JUMP` does**

Normally your `pc` moves forward. `JUMP` says "go to this position in the program instead." Think of it like a `goto`.

**What `JUMP_IF_ZERO` does**

It's an `if` statement. It pops a value and asks: was that zero? If yes, jump. If no, keep going. This is how you make decisions and exit loops.

**Test program 1 — infinite loop (prints AAAA... forever)**

```c
uint8_t program[] = {
    2, 65,         // 0: PUSH 65 ('A')
    4,             // 2: PRINT_TOP
    5, 0,          // 3: JUMP 0  (go back to the start)
    0              // 5: HALT    (never reached)
};
```

The numbers on the left (0, 2, 3, 5) are the `pc` positions. `JUMP 0` means "set `pc` to 0", which is the `PUSH` instruction. So the program loops forever: push A, print, jump back, push A, print, jump back...

**Test program 2 — print 'A' three times then stop**

This one is harder. Read it carefully and trace through by hand with a piece of paper. Write down `pc`, the stack, and the output after every instruction.

```c
uint8_t program[] = {
    2, 3,          // 0: PUSH 3         (loop counter)
    2, 65,         // 2: PUSH 65 ('A')
    4,             // 4: PRINT_TOP      (prints 'A', counter is now on top)
    2, 1,          // 5: PUSH 1
    3,             // 7: ADD            (wait — this is counter + 1, not counter - 1!)
};
```

Hmm, we don't have a `SUB` instruction. So to subtract 1 from our counter, we can add 255 instead — because our values are `unsigned char`, so `3 + 255` wraps around to `2`. It's the same trick CPUs use.

Here's the full program:

```c
uint8_t program[] = {
    2, 3,          //  0: PUSH 3           <- loop counter
    2, 65,         //  2: PUSH 65 ('A')
    4,             //  4: PRINT_TOP        <- prints 'A', counter back on top
    2, 255,        //  5: PUSH 255
    3,             //  7: ADD              <- counter = counter + 255 (wraps: subtract 1)
    6, 2,          //  8: JUMP_IF_ZERO 2   <- if counter hit 0, skip to pc=2... wait
    0              // 10: HALT
};
```

Actually, think about what should happen when the counter hits zero. We want to **stop**, not keep looping. So:

- If counter is **not** zero → go back to the PUSH 65 and loop again
- If counter **is** zero → fall through to HALT

But `JUMP_IF_ZERO` jumps when the value **is** zero. So we need it the other way around. One approach: put the `JUMP` (unconditional) to loop back, and put `JUMP_IF_ZERO` to skip over that jump to the HALT.

Take a crack at writing this. If you get stuck, think about it like this: you need two jumps — one that says "if done, skip to HALT" and one that says "otherwise, go back to the top."

**Hints if you're stuck**

The structure looks like:

```
PUSH 3              <- counter
PUSH 'A'            <- the character
PRINT_TOP
PUSH 255
ADD                  <- decrement counter
JUMP_IF_ZERO ___     <- if zero, jump to HALT
JUMP ___             <- otherwise, jump back to PUSH 'A'
HALT
```

Fill in the two `___` with the right `pc` values.
