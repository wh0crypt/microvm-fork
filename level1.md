# MicroVM — Level 1

Below is a program:

```c
uint8_t program[] = {
    1, 'H',
    1, 'e',
    1, 'l',
    1, 'l',
    1, 'o',
    1, ',',
    1, ' ',
    1, 'W',
    1, 'o',
    1, 'r',
    1, 'l',
    1, 'd',
    1, '!',
    1, '\n',
    0
};
```

Implement a virtual machine that executes a program:

```c
void run(uint8_t* program) {
    /* TODO: your task */
}
```

The VM has one piece of state: a **program counter** (`pc`), starting at 0.

The VM loops: fetch the opcode at `program[pc]` and execute it.

There are two opcodes:

| Byte | Name | Behavior |
|------|------|----------|
| `0x00` | `HALT` | Stop execution |
| `0x01` | `PRINT_CHAR` | Print `program[pc+1]` as an ASCII character. Advance `pc` by 2. |
