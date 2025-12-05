# xv6 Kernel Optimization: Shared Memory & Concurrency

This repository contains my modified version of the **xv6 operating system kernel** (RISC-V architecture).

The project focuses on two major areas of OS engineering: **Memory Management Optimization** and **Concurrency Control**. By modifying the kernel core, I implemented features to speed up system calls and resolve thread starvation issues.

## Key Features Implemented

### 1. Zero-Copy System Calls (Shared Memory)
**Problem:** System calls like `getpid()` are expensive. They require a context switch (User Mode → Kernel Mode), saving registers, and trap handling just to read a simple integer.
**Solution:** Implemented a **read-only shared memory page** mapped into every user process at a fixed virtual address (`USYSCALL`).
* **Mechanism:** The kernel writes data (like PID) to this page during process creation (`allocproc`).
* **Result:** User programs can read their PID instantly by accessing memory, bypassing the kernel trap entirely.
* **Files Modified:** `kernel/proc.c`, `kernel/proc.h`, `user/ulib.c`

### 2. Recursive Page Table Visualizer
**Problem:** Debugging virtual memory issues is difficult without seeing the mapping structure.
**Solution:** Developed a kernel tool (`vmprint`) that traverses the 3-level RISC-V **Sv39 page table**.
* **Mechanism:** A recursive function walks the page directory tree, identifies valid PTEs, and prints the hierarchy with indentation to show the structure (L2 → L1 → L0).
* **Result:** Provides a clear visual map of how virtual addresses translate to physical RAM.
* **Files Modified:** `kernel/vm.c`

### 3. Starvation-Free Reader-Writer Locks
**Problem:** Standard spinlocks allow only one CPU to access data at a time. This is inefficient for data that is frequently read but rarely written.
**Solution:** Implemented a **Reader-Writer Lock** with a **Writer Priority** scheduler.
* **Mechanism:**
    * **Concurrency:** Multiple readers can hold the lock simultaneously.
    * **Priority:** If a writer is waiting, new readers are blocked from entering. This prevents "writer starvation" in high-traffic scenarios.
* **Files Modified:** `kernel/spinlock.c`, `kernel/spinlock.h`

## How to Build & Run

### Prerequisites
You need a RISC-V toolchain and QEMU.
* **MacOS:** `brew install riscv-gnu-toolchain qemu`
* **Linux:** `sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu`

### Compiling
To compile the kernel and build the file system image:
```bash
make clean
make qemu