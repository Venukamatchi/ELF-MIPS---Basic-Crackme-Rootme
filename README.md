# Beginner’s Guide: Reversing a MIPS Crackme (Root-Me Example)

This guide walks you through the basics of analyzing a “crackme” binary on MIPS architecture, using only free tools like **radare2**. We’ll go from identifying the file, to extracting the password, and understanding what you see along the way. **No spoilers, just process and key ideas.**

---

## Table of Contents

- [What is a Crackme?](#what-is-a-crackme)
- [Understanding the Challenge](#understanding-the-challenge)
- [Identifying the Binary](#identifying-the-binary)
- [Why it may not run (Exec format error)](#why-it-may-not-run-exec-format-error)
- [Emulating or Analyzing the Binary](#emulating-or-analyzing-the-binary)
- [Introduction to radare2](#introduction-to-radare2)
- [Finding Strings](#finding-strings)
- [Finding the Main Function](#finding-the-main-function)
- [Disassembling and Reading the Logic](#disassembling-and-reading-the-logic)
- [Understanding Password Checks](#understanding-password-checks)
- [Reconstructing the Password](#reconstructing-the-password)
- [Common MIPS Instructions](#common-mips-instructions-quick-reference)
- [Practice Steps](#practice-steps)
- [Further Learning](#further-learning)

---

## What is a Crackme?

A **crackme** is a small program meant to test your reverse engineering skills. The goal is usually to figure out the “correct” input (often a password or serial) by analyzing the program, not by brute-forcing or guessing.

---

## Understanding the Challenge

- You’re given a binary file, often stripped (no symbols).
- The challenge is to find the validation password by analyzing the file.
- It may run on an architecture different from your computer (e.g., MIPS vs x86).

---

## Identifying the Binary

Use the `file` command:
```sh
file ch27.bin
```
You might see:
```
ELF 32-bit LSB executable, MIPS, MIPS32 rel2 ...
```
This tells you:
- It’s a 32-bit MIPS binary.
- It’s not made for your x86/x64 PC.

---

## Why it may not run (Exec format error)

Running `./ch27.bin` on your PC gives:
```
cannot execute binary file: Exec format error
```
**Reason:** Your CPU cannot run MIPS binaries natively.

---

## Emulating or Analyzing the Binary

- **Emulating**: Use tools like `qemu-mips` to try running MIPS binaries (may not always work, depending on binary specifics).
- **Analyzing**: Use reverse engineering tools to read the code and logic.

---

## Introduction to radare2

**radare2** (r2) is a powerful open-source reverse engineering framework.

### Install:
On Debian/Ubuntu:
```sh
sudo apt install radare2
```

### Start analyzing:
```sh
r2 -A ch27.bin
```
The `-A` does automatic analysis.

---

## Finding Strings

To see in-program text prompts and error messages:
```
izz
```
Look for lines like:
- “Enter password please”
- “well done!”
- “fail!”

These indicate where user input and password checking occur in the code.

---

## Finding the Main Function

List all functions:
```
afl
```
Look for a function named `main` or a function with most instructions (often at a higher address, e.g. `0x00400868`).

---

## Disassembling and Reading the Logic

Disassemble main:
```
pdf @ sym.main
```
or
```
pdf @ 0x00400868
```

Read through the assembly:
- Find where input is read (`fgets`, `scanf`).
- Find where `strlen` is called (checks input length).
- Look for comparison instructions.

---

## Understanding Password Checks

### What to look for:
- **Length checks**: Is the input the right length?
- **Character checks**: Is each position in the input checked for a specific value?
- **Loops**: Are multiple positions checked in a loop?
- **Direct comparisons**: Are some positions compared to constants directly?

### Example patterns:
- `addiu v0, zero, 0x63` sets `v0` to 'c'
- `beq v1, v0, ...` checks if a byte equals that value

---

## Reconstructing the Password

- Write down all constraints you find (which index must be which character).
- If a loop checks a range (e.g. input[8] to input[16] all must be 'i'), note that.
- Fill in a table or string with all required values.
- For positions not checked, you can often use any character.

---

## Common MIPS Instructions (Quick Reference)

- `addiu`: Add immediate unsigned (e.g., `addiu v0, zero, 0x63`) → sets a register to a value.
- `lb`: Load byte from memory.
- `beq`: Branch if equal (conditional jump).
- `bne`: Branch if not equal.
- `jalr`: Jump and link register (call function).
- `sw`, `lw`: Store/load word (write/read 4 bytes).

---

## Practice Steps

1. Run `file` to identify the binary format.
2. Run `r2 -A ch27.bin` to auto-analyze.
3. Use `izz` to find strings.
4. Use `afl` to find main.
5. Use `pdf @ sym.main` to read the logic.
6. Identify how/where input is read.
7. Find length checks and character checks.
8. Write down the password constraints.
9. Try reconstructing the password.

---

## Further Learning

- [radare2 Book](https://radareorg.github.io/radare2book/)
- [Reverse Engineering StackExchange](https://reverseengineering.stackexchange.com/)
- [Root-Me - Crackme MIPS Challenges](https://www.root-me.org/en/Challenges/Cracking/ELF32-MIPS-Basic)
- [Ghidra](https://ghidra-sre.org/) - for a graphical reversing experience.

---

## **Remember**  
Reverse engineering is about patience, curiosity, and practice.  
Take your time reading the code, and don’t be afraid to Google MIPS assembly instructions as you encounter them!
