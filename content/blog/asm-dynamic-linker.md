---
date: '2025-05-04T09:04:18+08:00'
draft: false
title: 'Asm Dynamic Linker'
---

## **dynamic linker** 简介
**dynamic linker** (also called the **dynamic loader**) **is a shared object** (`*.so` file) on Linux systems. It plays a critical role in the execution of dynamically linked programs.


## **What Is the Dynamic Linker?**
The dynamic linker is a special shared library responsible for:
- Loading the program's dependencies (shared libraries like `libc.so.6`).
- Resolving symbols (e.g., function addresses like `printf`).
- Performing relocations (adjusting addresses for shared libraries).
- Invoking constructors/destructors (via `.init_array` and `.fini_array`).

On Linux, the dynamic linker is typically named:
- **`ld-linux.so.2`** (for 32-bit x86).
- **`ld-linux-x86-64.so.2`** (for 64-bit x86_64).
- **`ld-linux-aarch64.so.1`** (for ARM64).


### **Why Is the Dynamic Linker a Shared Object?**
- **Efficiency:** It avoids duplicating the linker code in every executable.
- **Reusability:** All dynamically linked programs share the same dynamic linker.
- **Maintainability:** Updates to the dynamic linker (e.g., security fixes) apply to all programs.

###  **先有鸡先有蛋问题**
- The dynamic linker itself must be a `shared object`, but it is responsible for loading shared libraries. 动态链接器本身就是共享库
- To resolve this, the kernel `directly loads` the dynamic linker (`ld-linux-*.so`) into memory when starting a program. The dynamic linker then loads the rest of the dependencies (like `libc.so.6`) 内核直接负责装载 dynamic linker 这个共享库, 再由 dynamic linker 装载其他依赖

## demo 二进制文件查看

### `demo` 中动态链接器查看

`readelf -l demo` (look for the `INTERP` segment)

```bash
readelf -l demo | grep  -A 2 INTERP
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```

This path (`/lib64/ld-linux-x86-64.so.2`) is stored in the **`.interp` section** of the ELF file. 其中还有一个字符串结束符号 0x00。可以看到 **`.interp` section** 长度为 0x1c 一共 28bytes。


### file 查看 demo 信息
```bash
file demo
demo: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6fd65349be456b41544a47b67df68fa7968c3f4f, for GNU/Linux 3.2.0, with debug_info, not stripped

```

### Dynamic Linker VS Regular Shared Libraries
| Feature                      | Dynamic Linker (`ld-linux-*.so`)         | Regular Shared Library (e.g., `libc.so.6`) |
|-----------------------------|------------------------------------------|-------------------------------------------|
| **Loaded by**               | Kernel (directly)                        | Dynamic linker                            |
| **Purpose**                 | Load and link shared libraries             | Provide runtime functionality               |
| **Dependencies**            | None (self-contained)                    | May depend on other shared libraries        |
| **Entry Point**             | Called by kernel                         | Called by dynamic linker                    |


## **How Dynamic Linker Works?**



### **Step 1: Kernel Loads the Executable**
- The kernel reads the ELF header of `demo` and sees the `INTERP` segment pointing to `/lib64/ld-linux-x86-64.so.2`.
- The kernel **loads the dynamic linker itself** into memory.


### **Step 2: Dynamic Linker Takes Over**
- The kernel transfers control to the dynamic linker's entry point.
- The dynamic linker:
  1. Parses the executable's `.dynamic` section to find dependencies (e.g., `libc.so.6`).
  2. Loads these shared libraries into memory.
  3. Resolves symbols (e.g., `printf` in `libc`).
  4. Applies relocations (fixes addresses in `.got` and `.plt`).
  5. Calls constructor functions (e.g., `initialize()` in `.init_array`).
  6. Transfers control to the program's `main()`.
   
#### `gdb` 运行 `demo` 验证 `/lib64/ld-linux-x86-64.so.2` 先装载

```bash
gdb ./demo
.
.
.
Reading symbols from ./demo...
(gdb) break _start
Breakpoint 1 at 0x1080
(gdb) info program
The program being debugged is not being run.
(gdb) c
The program is not being run.
(gdb) run
Starting program: /home/yang/elf-demo/demo 

Breakpoint 1.2, 0x00007ffff7fe4540 in _start () from /lib64/ld-linux-x86-64.so.2
(gdb) info program
Last stopped for thread 1 (process 50955).
        Using the running image of child process 50955.
Program stopped at 0x7ffff7fe4540.
It stopped at breakpoint 1.
Type "info stack" or "info registers" for more information.
(gdb) info proc mappings
process 50955
Mapped address spaces:

          Start Addr           End Addr       Size     Offset  Perms  objfile
      0x555555554000     0x555555555000     0x1000        0x0  r--p   /home/yang/elf-demo/demo
      0x555555555000     0x555555556000     0x1000     0x1000  r-xp   /home/yang/elf-demo/demo
      0x555555556000     0x555555557000     0x1000     0x2000  r--p   /home/yang/elf-demo/demo
      0x555555557000     0x555555559000     0x2000     0x2000  rw-p   /home/yang/elf-demo/demo
      0x7ffff7fbf000     0x7ffff7fc3000     0x4000        0x0  r--p   [vvar]
      0x7ffff7fc3000     0x7ffff7fc5000     0x2000        0x0  r-xp   [vdso]
      0x7ffff7fc5000     0x7ffff7fc6000     0x1000        0x0  r--p   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
      0x7ffff7fc6000     0x7ffff7ff1000    0x2b000     0x1000  r-xp   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
      0x7ffff7ff1000     0x7ffff7ffb000     0xa000    0x2c000  r--p   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
      0x7ffff7ffb000     0x7ffff7fff000     0x4000    0x36000  rw-p   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
      0x7ffffffde000     0x7ffffffff000    0x21000        0x0  rw-p   [stack]
  0xffffffffff600000 0xffffffffff601000     0x1000        0x0  --xp   [vsyscall]
```
1. `break _start` 打断点
2. run 运行
3. info proc mappings 查看内存映射
   - `/home/yang/elf-demo/demo` 可执行文件
   - `/usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2` 动态链接器在 main 运行之前已被加载
```bash
ls -l /lib64/ld-linux-x86-64.so.2 
lrwxrwxrwx 1 root root 44 Jan 29 01:07 /lib64/ld-linux-x86-64.so.2 -> ../lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
```

#### `/lib64/ld-linux-x86-64.so.2` 装载其他依赖

1. ldd 查看 demo 依赖

```bash
ldd demo
        linux-vdso.so.1 (0x00007fffac3dc000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007be167000000)
        /lib64/ld-linux-x86-64.so.2 (0x00007be1672c1000)
```

2. gdb `initialize` 处打断点
   
```bash
(gdb) bt
#0  initialize () at demo.c:27
#1  0x00007ffff7c2a304 in call_init (env=<optimized out>, argv=0x7fffffffdc48, argc=1) at ../csu/libc-start.c:145
#2  __libc_start_main_impl (main=0x5555555551ed <main>, argc=1, argv=0x7fffffffdc48, init=<optimized out>, fini=<optimized out>, rtld_fini=<optimized out>, 
    stack_end=0x7fffffffdc38) at ../csu/libc-start.c:347
#3  0x00005555555550a5 in _start ()
(gdb) info proc mappings
process 50955
Mapped address spaces:

          Start Addr           End Addr       Size     Offset  Perms  objfile
      0x555555554000     0x555555555000     0x1000        0x0  r--p   /home/yang/elf-demo/demo
      0x555555555000     0x555555556000     0x1000     0x1000  r-xp   /home/yang/elf-demo/demo
      0x555555556000     0x555555557000     0x1000     0x2000  r--p   /home/yang/elf-demo/demo
      0x555555557000     0x555555558000     0x1000     0x2000  r--p   /home/yang/elf-demo/demo
      0x555555558000     0x555555559000     0x1000     0x3000  rw-p   /home/yang/elf-demo/demo
      0x7ffff7c00000     0x7ffff7c28000    0x28000        0x0  r--p   /usr/lib/x86_64-linux-gnu/libc.so.6
      0x7ffff7c28000     0x7ffff7db0000   0x188000    0x28000  r-xp   /usr/lib/x86_64-linux-gnu/libc.so.6
      0x7ffff7db0000     0x7ffff7dff000    0x4f000   0x1b0000  r--p   /usr/lib/x86_64-linux-gnu/libc.so.6
      0x7ffff7dff000     0x7ffff7e03000     0x4000   0x1fe000  r--p   /usr/lib/x86_64-linux-gnu/libc.so.6
      0x7ffff7e03000     0x7ffff7e05000     0x2000   0x202000  rw-p   /usr/lib/x86_64-linux-gnu/libc.so.6
```

1. `/usr/lib/x86_64-linux-gnu/libc.so.6` 这就是由 `/lib64/ld-linux-x86-64.so.2` 装载的依赖的动态库

### **Step 3: Lazy Binding**
> Lazy binding means that the actual address of `printf` isn't resolved until the first time it's called.


```
function call → PLT → GOT → resolver → dynamic linker resolves symbol → GOT updated
→ future calls jump directly to the resolved address. 
This is lazy binding.
```
  - When a function like `printf` is called for the first time:
  - The PLT entry jumps to the GOT.
  - The GOT initially points to `resolver code` in the dynamic linker.
  - The resolver resolves the symbol (e.g., finds `printf` in `libc`) and updates the GOT.
  - Subsequent calls jump directly to the resolved address.

## dynamic linker 启用环境变量观察过程

### LD_DEBUG 环境变量

```bash
LD_DEBUG=help ./demo
Valid options for the LD_DEBUG environment variable are:

  libs        display library search paths
  reloc       display relocation processing
  files       display progress for input file
  symbols     display symbol table processing
  bindings    display information about symbol binding
  versions    display version dependencies
  scopes      display scope information
  all         all previous options combined
  statistics  display relocation statistics
  unused      determined unused DSOs
  help        display this help message and exit

To direct the debugging output into a file instead of standard output
a filename can be specified using the LD_DEBUG_OUTPUT environment variable.

```


## `LD_DEBUG=libs ./demo 分析` 
The `LD_DEBUG=libs` output provides a detailed view of how the dynamic linker (`ld.so`) loads and initializes shared libraries for your `demo` executable. 


### **1. Library Search and Loading**
```bash
4039: find library=libc.so.6 [0]; searching
4039:  search cache=/etc/ld.so.cache
4039:   trying file=/lib/x86_64-linux-gnu/libc.so.6
```
- **Purpose**: The dynamic linker searches for required shared libraries (`libc.so.6` in this case).  
- **Process**:
  - It first checks the precomputed cache file `/etc/ld.so.cache` for efficiency.  
  - Then it directly tries the filesystem path `/lib/x86_64-linux-gnu/libc.so.6`.  
- **Result**: `libc.so.6` (the standard C library) is loaded into memory.



### **2. Initialization of Libraries**
```bash
4039: calling init: /lib64/ld-linux-x86-64.so.2
4039: calling init: /lib/x86_64-linux-gnu/libc.so.6
```
- **Purpose**: Run initialization code for shared libraries.  
- **Order**:
  1. **Dynamic Linker (`ld-linux-x86-64.so.2`)**: Initializes internal structures (e.g., GOT/PLT setup).  
  2. **`libc.so.6`**: Initializes the C library (e.g., stdio, threading, heap).  



### **3. Program Initialization**
```bash
4039: initialize program: ./demo
```
- **Purpose**: Execute constructor functions (marked with `__attribute__((constructor))`) before `main()`.  
- **In Your Code**:  
  - The `initialize()` function is called here, printing `"Initializing...\n"`.  



### **4. Transfer Control to Program**
```bash
4039: transferring control: ./demo
```
- **Purpose**: Jump to the program’s `main()` function.  
- **Output**:  
  - `Global Data: 42` (from `global_data` in `.data`)  
  - `Global BSS: 0` (from `global_bss` in `.bss`, zero-initialized)  
  - `Local Static: 100` (from `local_static` in `.data`)  
  - `Hello from .rodata!` (from `message` in `.rodata`)  
  - `Counter: 1`, `Counter: 2`, `Counter: 3` (from `counter()` with static `count` in `.bss`)  


### **5. Finalization**
```bash
4039: calling fini:  [0]
4039: Finalizing...
4039: calling fini: /lib/x86_64-linux-gnu/libc.so.6
4039: calling fini: /lib64/ld-linux-x86-64.so.2
```
- **Purpose**: Run destructor functions (marked with `__attribute__((destructor))`) after `main()`.  
- **Order**:
  1. **Program Destructors**:  
     - The `finalize()` function is called, printing `"Finalizing...\n"`.  
  2. **Library Destructors**:  
     - `libc.so.6`: Cleans up C library resources.  
     - `ld-linux-x86-64.so.2`: Cleans up dynamic linking infrastructure.  



### **Key Observations**
1. **Initialization Order**:  
   - Libraries are initialized **depth-first**, starting from dependencies (e.g., `libc`) and moving to the main executable.  
   - Constructors run **after** all libraries are loaded but **before** `main()`.  

2. **Finalization Order**:  
   - Destructors run **in reverse order** of initialization:  
     - Program destructors first (`finalize()`), then library destructors (`libc`, `ld-linux`).  

3. **Dynamic Linker Role**:  
   - Manages the entire lifecycle:  
     - **Loading**: Resolves symbols and maps libraries into memory.  
     - **Relocation**: Adjusts addresses for position-independent code (PIC).  
     - **Initialization/Finalization**: Ensures proper setup/teardown of libraries and code.  

4. **Symbol Resolution**:  
   - Symbols like `printf` are resolved via `.plt`/`.got` (lazy binding unless `BIND_NOW` is set).  


### **Why This Matters**
- **Debugging**: Helps identify missing libraries, symbol conflicts, or initialization issues.  
- **Performance**: Reveals overhead from lazy vs. eager binding (`LD_BIND_NOW`).  
- **Security**: Shows how libraries are loaded and validated (e.g., `PIE`, `RELRO`).  


## LD_DEBUG=files ./demo 分析

```bash
LD_DEBUG=files ./demo
      4266:
      4266:     file=libc.so.6 [0];  needed by ./demo [0]
      4266:     file=libc.so.6 [0];  generating link map
      4266:       dynamic: 0x000073f09de02940  base: 0x000073f09dc00000   size: 0x0000000000211d90
      4266:         entry: 0x000073f09dc2a390  phdr: 0x000073f09dc00040  phnum:                 14
      4266:
      4266:
      4266:     calling init: /lib64/ld-linux-x86-64.so.2
      4266:
      4266:
      4266:     calling init: /lib/x86_64-linux-gnu/libc.so.6
      4266:
      4266:
      4266:     initialize program: ./demo
      4266:
Initializing...
      4266:
      4266:     transferring control: ./demo
      4266:
Global Data: 42
Global BSS: 0
Local Static: 100
Hello from .rodata!
Counter: 1
Counter: 2
Counter: 3
      4266:
      4266:     calling fini:  [0]
      4266:
Finalizing...
      4266:
      4266:     calling fini: /lib/x86_64-linux-gnu/libc.so.6 [0]
      4266:
      4266:
      4266:     calling fini: /lib64/ld-linux-x86-64.so.2 [0]
      4266:

```

### **Library File Resolution**
```bash
4266: file=libc.so.6 [0];  needed by ./demo [0]
4266: file=libc.so.6 [0];  generating link map
```
- **Purpose**: The dynamic linker resolves dependencies listed in the `.dynamic` section (e.g., `NEEDED` entries like `libc.so.6`).  
- **Link Map**:  
  - A `link_map` structure is created to track metadata for each loaded library.  
  - **Fields**:  
    - `dynamic`: Pointer to the `.dynamic` section of `libc.so.6` (used for symbol resolution).  
    - `base`: Load address in memory (`0x000073f09dc00000`).  
    - `size`: Size of the mapped library (`0x211d90` bytes).  
    - `entry`: Entry point (start address of `libc.so.6`).  
    - `phdr`: Pointer to program headers (used to load segments into memory).  
    - `phnum`: Number of program headers (`14` in this case).


## **LD_DEBUG=reloc ./demo 分析**

The `LD_DEBUG=reloc ./demo` output reveals how the **dynamic linker** processes **relocations** during program execution. Relocations are adjustments made to addresses in the binary and shared libraries when they're loaded into memory (due to **Position-Independent Code (PIC)** and **Address Space Layout Randomization (ASLR)**). Let’s break down the key steps:



### **1. Relocation Processing**
```bash
5881: relocation processing: /lib/x86_64-linux-gnu/libc.so.6
5881: relocation processing: ./demo
5881: relocation processing: /lib64/ld-linux-x86-64.so.2
```
- **Purpose**: The dynamic linker resolves **symbol addresses** in the following order:
  1. **`libc.so.6`**: Adjusts addresses for symbols like `printf`, `exit`, and global variables.  
  2. **`./demo`**: Fixes up addresses for global/static variables (e.g., `global_data`, `global_bss`, `local_static`) and PLT/GOT entries.  
  3. **`ld-linux-x86-64.so.2`**: Adjusts internal addresses for the dynamic linker itself.  

- **What Happens During Relocation?**  
  - The dynamic linker reads **relocation sections** like `.rela.dyn` (for global variables) and `.rela.plt` (for function calls) to update addresses.  
  - For example:  
    - A `R_X86_64_GLOB_DAT` relocation updates the GOT entry for `printf` to point to its address in `libc.so.6`.  
    - A `R_X86_64_RELATIVE` relocation adjusts addresses in `./demo` based on its load address (ASLR).  



### **2. Initialization of Libraries**
```bash
5881: calling init: /lib64/ld-linux-x86-64.so.2
5881: calling init: /lib/x86_64-linux-gnu/libc.so.6
```
- **Purpose**: Run initialization code for shared libraries after relocations are applied.  
- **Order**:  
  1. **Dynamic Linker (`ld-linux`)**: Initializes internal structures (e.g., GOT/PLT setup).  
  2. **`libc.so.6`**: Initializes the C library (e.g., stdio, heap).  



### **3. Program Initialization**
```bash
5881: initialize program: ./demo
Initializing...
```
- **Purpose**: Execute constructor functions (marked with `__attribute__((constructor))`) before `main()`.  
- **In Your Code**:  
  - The `initialize()` function is called here, printing `"Initializing...\n"`.  



### **4. Transfer Control to Program**
```bash
5881: transferring control: ./demo
```
- **Purpose**: Jump to the program’s `main()` function.  
- **Output**:  
  - `Global Data: 42` (from `global_data` in `.data`)  
  - `Global BSS: 0` (from `global_bss` in `.bss`, zero-initialized)  
  - `Local Static: 100` (from `local_static` in `.data`)  
  - `Hello from .rodata!` (from `message` in `.rodata`)  
  - `Counter: 1`, `Counter: 2`, `Counter: 3` (from `counter()` with static `count` in `.bss`)  

- **Relocation Details**:  
  - **Static Variables**: Addresses of `global_data`, `global_bss`, and `local_static` are resolved via `.rela.dyn` relocations.  
  - **Function Calls**: `printf` is resolved via `.rela.plt` relocations using the **PLT/GOT mechanism**.  



### **5. Finalization**
```bash
5881: calling fini: [0]
Finalizing...
5881: calling fini: /lib/x86_64-linux-gnu/libc.so.6
5881: calling fini: /lib64/ld-linux-x86-64.so.2
```
- **Purpose**: Run destructor functions (marked with `__attribute__((destructor))`) after `main()`.  
- **Order**:  
  1. **Program Destructors**:  
     - The `finalize()` function is called, printing `"Finalizing...\n"`.  
  2. **Library Destructors**:  
     - `libc.so.6`: Cleans up C library resources.  
     - `ld-linux-x86-64.so.2`: Cleans up dynamic linking infrastructure.  



### **Key Concepts in Relocation**
1. **Why Relocation Is Needed**:  
   - Shared libraries are compiled as **position-independent code (PIC)** to support ASLR.  
   - At runtime, the dynamic linker adjusts addresses to match the actual load address in memory.  

2. **Types of Relocations**:  
   - **`R_X86_64_RELATIVE`**: Adjusts addresses relative to the library’s base address (e.g., for `./demo`).  
   - **`R_X86_64_GLOB_DAT`**: Updates GOT entries for global symbols (e.g., `printf` in `libc.so.6`).  
   - **`R_X86_64_JUMP_SLOT`**: Updates PLT entries for lazy binding (e.g., resolving `printf` on first call).  

3. **Sections Involved**:  
   - `.rela.dyn`: Relocations for global variables and data.  
   - `.rela.plt`: Relocations for function calls (lazy binding).  
   - `.got`: Global Offset Table (stores resolved addresses for variables/functions).  
   - `.plt`: Procedure Linkage Table (trampolines for lazy symbol resolution).  



### **Example: Resolving `printf`**
1. **Initial Call**:  
   - `printf` in `main()` jumps to `.plt`, which then jumps to `.got`.  
   - If unresolved, `.got` redirects to the dynamic linker.  

2. **Dynamic Linker**:  
   - Uses `.rela.plt` to find the symbol index for `printf`.  
   - Looks up `printf` in `libc.so.6` using `.gnu.hash` and `.dynsym`.  
   - Updates the `.got` entry with `printf`’s runtime address.  

3. **Subsequent Calls**:  
   - Directly use the resolved address in `.got` (no dynamic linker needed).  



### **Summary**
- **Relocation ensures** that addresses in the binary and shared libraries are adjusted at runtime to work with ASLR and PIC.  
- **Dynamic linker processes** relocations in `.rela.dyn` (variables) and `.rela.plt` (functions) to resolve symbols like `printf`.  
- **Constructors/destructors** (`initialize`/`finalize`) are handled via `.init_array`/`.fini_array` after relocations are applied.  

## `printf` 重定位分析
1. gcc -g -no-pie -o demo demo.c
2. 查看重定位信息
```bash
   readelf -r demo

Relocation section '.rela.dyn' at offset 0x4d8 contains 2 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000403fd8  000100000006 R_X86_64_GLOB_DAT 0000000000000000 __libc_start_main@GLIBC_2.34 + 0
000000403fe0  000400000006 R_X86_64_GLOB_DAT 0000000000000000 __gmon_start__ + 0

Relocation section '.rela.plt' at offset 0x508 contains 2 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000404000  000200000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0
000000404008  000300000007 R_X86_64_JUMP_SLO 0000000000000000 printf@GLIBC_2.2.5 + 0
```
> 0x000000404008 就是 `printf` 在文件中偏移量
  
3. gdb ./demo 分析
```bash
Reading symbols from ./demo...
(gdb) break _start 
Breakpoint 1 at 0x401070
(gdb) break
break        break-range  
(gdb) run
Starting program: /home/yang/elf-demo/demo 

Breakpoint 1.2, 0x00007ffff7fe4540 in _start () from /lib64/ld-linux-x86-64.so.2
(gdb) break _dl_runtime_resolve_xsave
_dl_runtime_resolve_xsave   _dl_runtime_resolve_xsavec  
(gdb) break _dl_runtime_resolve_xsave
Breakpoint 2 at 0x7ffff7fda220: file ../sysdeps/x86_64/dl-trampoline.h, line 71.
(gdb) info sharedlibrary 
From                To                  Syms Read   Shared Object Library
0x00007ffff7fc6000  0x00007ffff7ff0195  Yes         /lib64/ld-linux-x86-64.so.2
(gdb) dis
disable      disassemble  disconnect   display      
(gdb) disassemble printf
Dump of assembler code for function printf@plt:
   0x0000000000401060 <+0>:	endbr64
   0x0000000000401064 <+4>:	jmp    *0x2f9e(%rip)        # 0x404008 <printf@got.plt>
   0x000000000040106a <+10>:	nopw   0x0(%rax,%rax,1)
End of assembler dump.


(gdb) x/xg 0x404008
0x404008 <printf@got.plt>:	0x0000000000401040 # 重定位前的地址

(gdb) info stack
#0  0x00007ffff7fe4540 in _start () from /lib64/ld-linux-x86-64.so.2
#1  0x0000000000000001 in ?? ()
#2  0x00007fffffffe1a9 in ?? ()
#3  0x0000000000000000 in ?? ()
                                               
(gdb) break print_message 
Breakpoint 3 at 0x4011c8: file demo.c, line 32.
(gdb) continue 
Continuing.
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1.1, 0x0000000000401070 in _start ()
(gdb) continue 
Continuing.
Initializing...
Global Data: 42
Global BSS: 0
Local Static: 100

Breakpoint 3, print_message () at demo.c:32
32	    printf("%s\n", message); // Uses PLT/GOT for printf
(gdb) x/xg 0x404008
0x404008 <printf@got.plt>:	0x00007ffff7c600f0 # 重定位后的地址

(gdb) info stack 
#0  print_message () at demo.c:32
#1  0x0000000000401244 in main () at demo.c:43

(gdb) break counter
Breakpoint 4 at 0x40115e: file demo.c, line 16.
(gdb) continue 
Continuing.
Hello from .rodata!

Breakpoint 4, counter () at demo.c:16
16	    count++;
(gdb) info stack 
#0  counter () at demo.c:16
#1  0x0000000000401257 in main () at demo.c:46
(gdb) x/xg 0x404008
0x404008 <printf@got.plt>:	0x00007ffff7c600f0


```



## **dynamic linker 总结**
- The dynamic linker (`ld-linux-*.so`) is a **shared object** that the kernel loads directly.
- It bootstraps the loading of all other shared libraries and manages symbol resolution.
- Without it, dynamically linked programs (like most C programs using `glibc`) would not work.



## Reference

1. [dynamic-linker](https://man7.org/linux/man-pages/man8/ld.so.8.html)




