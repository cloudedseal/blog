---
title: "Master Elf"
date: 2025-04-28T15:29:57+08:00
draft: false
---

## hello world

```c
#include <stdio.h>
int main() { 
    printf("hello world\n"); 
}
```

> gcc --save-temps  -o hello hello.c

## **What is ELF?**
ELF (Executable and Linkable Format) is a **standardized binary format** for storing compiled programs, libraries, and other binaries on Unix-like systems (e.g., Linux). It defines how machine code, data, symbols, relocation information, and metadata are organized in files like:
- Executables (`./hello`)
- Object files (`hello.o`)
- Shared libraries (`.so`)
- Core dumps

---

## **ELF File Structure**

An ELF file has two primary views:
### **Linking View (Sections)**
   - Used during `compilation/linking` to combine object files.
   - Contains **section headers** describing discrete parts of the file.

### **Execution View (Segments)**
   - Used at `runtime` to load the program into memory.
   - Contains **program headers** mapping sections to memory segments.

## Key Components
| Component           | Purpose                                                                 |
|---------------------|-------------------------------------------------------------------------|
| **ELF Header**      | Metadata about the file (architecture, type, entry point, etc.).       |
| **Program Headers** | Describe how to map segments to memory (used by the OS at runtime).     |
| **Section Headers** | List all sections (code, data, symbols, relocations, debug info, etc.). |
| **Sections**        | Logical units like `.text` (code), `.data` (initialized variables), etc.|
| **Segments**        | Physical chunks mapped into memory (e.g., `LOAD`, `DYNAMIC`, `STACK`).  |

---

## **ELF File Layout**
```
+---------------------+
|   ELF Header        |  ← Fixed-size metadata at file offset 0
+---------------------+
|   Program Headers   |  ← Describes segments (for runtime)
+---------------------+
|   Section 1 (.text) |  ← Machine code
+---------------------+
|   Section 2 (.data) |  ← Initialized global/static variables
+---------------------+
|   ...               |
+---------------------+
|   Section Headers   |  ← Table listing all sections
+---------------------+
```

---

## **Key ELf Concepts**

### **ELF Header**
Use `readelf -h hello` to see:
```bash
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x1060
  Start of program headers:          64 (bytes into file)
  Start of section headers:          13976 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```

### **Program Headers (Segments)**
Use `readelf -l hello` to see:
```bash
Elf file type is DYN (Position-Independent Executable file)
Entry point 0x1060
There are 13 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000002d8 0x00000000000002d8  R      0x8
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000628 0x0000000000000628  R      0x1000
  LOAD           0x0000000000001000 0x0000000000001000 0x0000000000001000
                 0x0000000000000175 0x0000000000000175  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x00000000000000f4 0x00000000000000f4  R      0x1000
  LOAD           0x0000000000002db8 0x0000000000003db8 0x0000000000003db8
                 0x0000000000000258 0x0000000000000260  RW     0x1000
  DYNAMIC        0x0000000000002dc8 0x0000000000003dc8 0x0000000000003dc8
                 0x00000000000001f0 0x00000000000001f0  RW     0x8
  NOTE           0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000030 0x0000000000000030  R      0x8
  NOTE           0x0000000000000368 0x0000000000000368 0x0000000000000368
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_PROPERTY   0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000030 0x0000000000000030  R      0x8
  GNU_EH_FRAME   0x0000000000002010 0x0000000000002010 0x0000000000002010
                 0x0000000000000034 0x0000000000000034  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000002db8 0x0000000000003db8 0x0000000000003db8
                 0x0000000000000248 0x0000000000000248  R      0x1

 Section to Segment mapping:
  Segment Sections...
   00     
   01     .interp 
   02     .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt 
   03     .init .plt .plt.got .plt.sec .text .fini 
   04     .rodata .eh_frame_hdr .eh_frame 
   05     .init_array .fini_array .dynamic .got .data .bss 
   06     .dynamic 
   07     .note.gnu.property 
   08     .note.gnu.build-id .note.ABI-tag 
   09     .note.gnu.property 
   10     .eh_frame_hdr 
   11     
   12     .init_array .fini_array .dynamic .got 
```

### segment LOAD 理解

#### **Explanation of the Two Lines in the `LOAD` Segment**
Here’s a breakdown of the two lines from the `LOAD` segment in your `readelf -l hello` output:

```
LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
               0x0000000000000628 0x0000000000000628  R      0x1000
```

---

#### **Line-by-Line Breakdown**

| Field         | Value                          | Meaning                                                                 |
|---------------|--------------------------------|-------------------------------------------------------------------------|
| **Type**      | `LOAD`                         | This segment is `loaded` into memory during execution.                    |
| **Offset**    | `0x0000000000000000`          | Start of this segment in the file (offset 0 bytes from the file start). |
| **VirtAddr**  | `0x0000000000000000`          | `Virtual memory address` where this segment is mapped (position-independent; resolved at runtime). |
| **PhysAddr**  | `0x0000000000000000`          | `Physical memory address` (ignored for executables; used in firmware).   |
| **FileSiz**   | `0x0000000000000628` (1576 bytes) | `Size` of the segment in the file.                                       |
| **MemSiz**    | `0x0000000000000628` (1576 bytes) | `Size` of the segment in memory (same as `FileSiz`; no zero-filled padding). |
| **Flags**     | `R`                            | Segment is **read-only**.                                             |
| **Align**     | `0x1000` (4096 bytes)          | Alignment requirement (must be `page-aligned` in memory).                |

---

#### **Purpose of This Segment**
This `LOAD` segment maps **read-only metadata** into memory, including:
- **`.interp`**: Path to the dynamic linker (`/lib64/ld-linux-x86-64.so.2`).
- **`.note.gnu.build-id`**: Unique identifier for debugging.
- **`.gnu.hash`, `.dynsym`, `.dynstr`**: Dynamic symbol table and hashing for efficient symbol resolution.
- **Relocation tables**: Used by the dynamic linker to adjust addresses at runtime.

It is critical for **dynamic linking** and **runtime symbol resolution**.

---

#### **Why Are `VirtAddr` and `PhysAddr` Zero?**
- **Position-Independent Executable (PIE)**:  
  The binary is compiled as `DYN` (shared object style), allowing the OS to load it at a **randomized base address** (ASLR).  
  - At runtime, the OS chooses a base address (e.g., `0x555555554000`), and all segments are mapped relative to this base.  
  - `VirtAddr = 0` means the segment’s virtual address is calculated as `base + offset`.

---

#### **Security Implications**
- **No Write or Execute Permissions**:  
  The `R` flag ensures this segment is **read-only**, protecting metadata from tampering.
- **Alignment (`0x1000`)**:  
  Ensures the segment starts at a **page boundary** (4KB), satisfying memory protection requirements.

---

#### **Example Mapping in Memory**
At runtime, the OS maps this segment as follows:
```
Virtual Address Range      Permissions   Purpose
[BASE_ADDR]                R             .interp, .note.gnu.build-id, .gnu.hash, .dynsym, etc.
[BASE_ADDR + 0x628]        ...           Next segment starts here.
```

---

#### **Key Takeaways**
1. **Metadata Storage**: Contains essential runtime data for dynamic linking.
2. **ASLR Compatibility**: Position-independent addressing enhances security.
3. **Read-Only Protection**: Prevents accidental or malicious modification of critical data.

---

### **Sections**
Use `readelf -S hello` to list sections:
```bash
Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
  [ 1] .text             PROGBITS         0000000000401000  00001000
       00000000000001b2  0000000000000000  AX       0     0     16
  ...
```
- `.text`: Executable code.
- `.rodata`: Read-only constants (e.g., strings).
- `.plt/.got`: Dynamic linking stubs.
- `.symtab`: Symbol table for debugging/linking.

---

## **How Does ELF Work?**
1. **Compilation**:  
   GCC generates an ELF object file (`hello.o`) with sections like `.text`, `.data`, and `.symtab`.
2. **Linking**:  
   The linker (`ld`) merges sections from multiple object files, resolves symbols, and creates an executable ELF file with program headers.
3. **Execution**:  
   The OS loads the executable using the program headers to map memory regions and execute code.

---

## **What Does the OS Do When Running `./hello`?**
Here’s the lifecycle from command line to execution:

### **User Runs the Command**
```bash
./hello
```
The shell calls `execve("./hello", ...)`, which triggers the kernel to load the ELF file.

### **Kernel Reads the ELF Header**
- Checks the magic number (`\x7fELF`) to confirm it’s an ELF file.
- Parses the ELF header to determine architecture (e.g., x86-64).

### **Kernel Loads Segments into Memory**
Using the **program headers**, the kernel:
- Maps `.text` (code) to `executable memory`.
- Maps `.data` and `.bss` to `writable memory`.
- Sets up the **stack** and environment variables.
- If there’s a `INTERP` segment, loads the `dynamic linker` (e.g., `/lib64/ld-linux-x86-64.so.2`).

### **Dynamic Linking (if needed)**
If the program uses shared libraries:
- The kernel `transfers control` to the dynamic linker.
- The linker resolves dependencies (e.g., `libc.so.6`), loads libraries, and relocates addresses.

### **Start Execution**
- The kernel jumps to the **entry point** specified in the ELF header (`Entry point address`).
- For C programs, this is typically `_start` (in `crt1.o`), which initializes the C runtime, calls `main()`, and exits cleanly.

---

## **Example: Full Flow for `hello`**

```bash
$ gcc -o hello hello.c  # Compiles to ELF executable
```
###  **File Type**:  
   ```bash
    file hello
        hello: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=ce69e228b62365b698bac3bf837cb1c5668a8079, for GNU/Linux 3.2.0, not stripped

    ldd hello
        linux-vdso.so.1 (0x00007fffdd6be000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007cdea5a00000)
        /lib64/ld-linux-x86-64.so.2 (0x00007cdea5db7000)
   ```
### **Entry Point**:  
   ```bash
   $ readelf -h hello | grep "Entry point"
        Entry point address:               0x1060
   ```
### **Execution Steps**:
   - Kernel maps the `.text` segment to `0x401000`.
   - Starts executing `_start` (`assembler` boilerplate).
   - Calls `__libc_start_main()` (glibc initialization).
   - Invokes `main()` → prints `"Hello, world!"`.
   - Returns to the kernel via `exit(0)`.

---

### **Tools to Inspect ELF Files**
| Tool       | Purpose                          |
|------------|----------------------------------|
| `readelf`  | Analyze headers, sections, symbols. |
| `objdump`  | Disassemble code, view relocations. |
| `nm`       | List symbols (functions/variables). |
| `gdb`      | Debug and inspect memory layout.   |
| `strings`  | Extract human-readable strings.    |

---

### **Summary Workflow**
1. **Source Code** → `gcc -c` → **ELF Object File** (`hello.o`)  
2. **Object Files** → `ld` → **ELF Executable** (`hello`)  
3. **OS Loads ELF** → Maps Memory Segments → Resolves Dependencies → Executes Code  

By understanding ELF, you gain insight into how software interacts with hardware, memory, and the operating system at the lowest level.

# References
1. [linkers](https://www.lurklurk.org/linkers/linkers.html)
2. [wiki-linker](https://en.wikipedia.org/wiki/Linker_(computing))

