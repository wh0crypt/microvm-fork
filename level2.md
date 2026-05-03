# MicroVM — Level 2

Extend your VM with a **stack** (e.g. `uint8_t stack[256]` and a `sp` (stack pointer) starting at 0).

If you do not know how a stack works: https://claude.ai/public/artifacts/18651d38-5518-4a87-a58d-2cd2f16df998

Add three new opcodes:

| Byte | Name | Behavior |
|------|------|----------|
| `0x00` | `HALT` | Stop execution |
| `0x01` | `PRINT_CHAR` | Print `program[pc+1]` as ASCII. `pc += 2` |
| `0x02` | `PUSH` | Push `program[pc+1]` onto the stack. `pc += 2` |
| `0x03` | `ADD` | Pop two values, push their sum. `pc += 1` |
| `0x04` | `PRINT_TOP` | Pop the top of the stack and print it as ASCII. `pc += 1` |

Test program — prints `Hi` using the stack:

```c
uint8_t program[] = {
    2, 'H',       // PUSH 'H'
    4,             // PRINT_TOP
    2, 'i',       // PUSH 'i'
    4,             // PRINT_TOP
    0              // HALT
};
```

Test program — prints `C` (ASCII 67 = 65 + 2) using ADD:

```c
uint8_t program[] = {
    2, 65,         // PUSH 65  ('A')
    2, 2,          // PUSH 2
    3,             // ADD       (stack top = 67)
    4,             // PRINT_TOP (prints 'C')
    0              // HALT
};
```
