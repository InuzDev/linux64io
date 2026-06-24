# Linux64io

A I/O helper library for NASM assembly programs on Linux and WSL.

It does direct Syscalls and readable functions and macros. The main requirements are:

- NASM
- GNU Binutils
- Linux or WSL

## Installation for the binutils and NASM

You can run in the terminal the following command:

```sh
sudo apt install nasm binutils
```

## Usage

In a assembly project (`.asm`), you put `linux64io.inc` in the same folder and in the assembly code that you desire to use the package just write the following snippet of code

```asm
%include "linux64io.inc"
```

## Macros

The library got the following macros, doesn't require complex setup, but doesn't allow a lot of control.

| Macro       | Example                  | Description                                |
| ----------- | ------------------------ | ------------------------------------------ |
| `PRINT_STR` | `PRINT_STR msg, msg_len` | Print a string defined in `.data`          |
| `PRINT_INT` | `PRINT_INT r12`          | Print an integer followed by a newline     |
| `READ_INT`  | `READ_INT r12`           | Read an integer from stdin into a register |

## Functions

This allows more control, calling them directly allows you to use pointers and the bytes written.

| Function        | Input                          | Output              | Description                   |
| --------------- | ------------------------------ | ------------------- | ----------------------------- |
| `print_str`     | rsi = pointer, rdx = length    | rax = bytes written | Print a string                |
| `print_int`     | rax = integer                  | —                   | Print an integer with newline |
| `print_newline` | —                              | —                   | Print a newline character     |
| `read_int`      | —                              | rax = integer       | Read an integer from stdin    |
| `read_str`      | rsi = buffer, rdx = max length | rax = bytes read    | Read a raw string from stdin  |

## Example Program

```nasm
%include "linux64io.inc"

section .data
    msg     db "Enter a number: "
    msg_len equ $ - msg
    res_msg db "Doubled: "
    res_len equ $ - res_msg

section .text
    global _start

_start:
    PRINT_STR msg, msg_len
    READ_INT r12            ; r12 = user input

    imul r12, r12, 2        ; double it

    PRINT_STR res_msg, res_len
    PRINT_INT r12

    mov rax, 60             ; exit
    xor rdi, rdi
    syscall
```

## Compile & Run

> Test code still in progress to include

```bash
# Single file
nasm -f elf64 your_program.asm -o your_program.o
ld your_program.o -o your_program
./your_program

# Using the Makefile (builds suma, factorial, primo)
make
./sumNum
./factorial
./primeNum

# Clean up object files and binaries
make clean
```

## Register Safety

Linux syscalls internally clobber `rcx` and `r11`. Use `r12`–`r15` to safely hold values across function calls — the library never touches them.

| Function        | Clobbers                         |
| --------------- | -------------------------------- |
| `print_str`     | rax, rdi                         |
| `print_int`     | rax, rdx, rdi, rsi, rbx, r9, r10 |
| `print_newline` | rax, rdi, rsi, rdx               |
| `read_int`      | rax, rcx, rdx, rdi, rsi          |
| `read_str`      | rax, rdi                         |
