# Rust ARM Binary Execution (Apple Silicon Only)

## Introduction

This project demonstrates how to **generate, place, and execute raw ARM64 machine code**
from **Rust** on **Apple Silicon (ARM64 macOS)**.

The example manually constructs an ARM binary for a simple function and executes it
by allocating executable memory at runtime.

> ⚠️ **Architecture & OS limitation**  
> This approach relies on macOS-specific memory permissions (`MAP_JIT`) and
> **only works on Apple Silicon Macs**.

---

## Setup & Requirements

- **OS**: macOS (Apple Silicon only)
- **Architecture**: ARM64 (ARMv8-A)
- **Rust**: `1.71.1` (2021 edition)
- **Crate**: `libc = 0.2.149`

---

## Overview

Execution flow:

1. Write a simple C++ function
2. Compile it to ARM64 assembly
3. Convert assembly to raw ARM instructions
4. Copy instructions into executable memory
5. Jump to the memory region as a function

---

## 1. Target Function (C++)

We start with a trivial function:

```cpp
int fn(int num) {
    return num;
}
```

---

## 2. Compile to ARM64 Assembly

Compiled with **clang 17.0.1** targeting **ARMv8-A**.  
Assembly generated via **Compiler Explorer**.

```asm
fn(int):    // demangled fn(int)
    sub     sp, sp, #16
    str     w0, [sp, #12]
    ldr     w0, [sp, #12]
    add     sp, sp, #16
    ret
```

---

## 3. Convert Assembly to Machine Code

The assembly is converted into raw ARM64 instructions using
[armconverter](https://armconverter.com/).

```text
0xD10043FF
0xB9000FE0
0xB9400FE0
0x910043FF
0xD65F03C0
```

Each value represents one 32-bit ARM instruction.

---

## 4. Execute ARM Binary in Rust

The following Rust code:

- Allocates executable memory using `mmap`
- Copies ARM instructions into memory
- Changes memory protection to executable
- Casts the memory to a function pointer and calls it

```rust
use libc::*;
use std::ffi::c_void;

fn main() {
    // Compiled ARM64 instructions of `int fn(int)`
    let mut code: [u32; 5] = [
        0xD10043FF,
        0xB9000FE0,
        0xB9400FE0,
        0x910043FF,
        0xD65F03C0,
    ];

    unsafe {
        // Allocate writable JIT memory
        let ptr = mmap(
            std::ptr::null_mut(),
            4 * code.len(),
            PROT_WRITE,
            MAP_JIT | MAP_ANON | MAP_PRIVATE,
            -1,
            0,
        );

        assert!(ptr != MAP_FAILED);

        // Copy machine code into memory
        std::ptr::copy(code.as_mut_ptr(), ptr as *mut u32, code.len());

        // Change memory protection to executable
        mprotect(ptr, 4 * code.len(), PROT_EXEC);

        // Convert pointer into function pointer
        let func = std::mem::transmute::<*mut c_void, fn(i32) -> i32>(ptr);

        println!("{}", func(3)); // expected output: 3
    }
}
```

---

## Security Notes

- **Never map memory as writable + executable at the same time**
- macOS enforces W^X via `MAP_JIT`
- This example is effectively **runtime code injection**
- Intended for **experimentation, JIT research, or compiler work only**

---

## References

- Simple JIT in C  
  https://blog.reverberate.org/2012/12/hello-jit-world-joy-of-simple-jits.html
- Compiler Explorer  
  https://godbolt.org/
- ARM Converter  
  https://armconverter.com/
