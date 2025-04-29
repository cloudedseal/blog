---
title: "Master Elf"
date: 2024-05-30T15:29:57+08:00
draft: false
---

### hello world

```c
#include <stdio.h>
int main() { 
    printf("hello world\n"); 
}
```

> gcc --save-temps  -o hello hello.c

### **What is ELF?**
ELF (Executable and Linkable Format) is a **standardized binary format** for storing compiled programs, libraries, and other binaries on Unix-like systems (e.g., Linux). It defines how machine code, data, symbols, relocation information, and metadata are organized in files like:
- Executables (`./hello`)
- Object files (`hello.o`)
- Shared libraries (`.so`)
- Core dumps

---

### **ELF File Structure**
An ELF file has two primary views:
#### 1. **Linking View (Sections)**
   - Used during `compilation/linking` to combine object files.
   - Contains **section headers** describing discrete parts of the file.

#### 2. **Execution View (Segments)**
   - Used at `runtime` to load the program into memory.
   - Contains **program headers** mapping sections to memory segments.

#### Key Components
| Component           | Purpose                                                                 |
|---------------------|-------------------------------------------------------------------------|
| **ELF Header**      | Metadata about the file (architecture, type, entry point, etc.).       |
| **Program Headers** | Describe how to map segments to memory (used by the OS at runtime).     |
| **Section Headers** | List all sections (code, data, symbols, relocations, debug info, etc.). |
| **Sections**        | Logical units like `.text` (code), `.data` (initialized variables), etc.|
| **Segments**        | Physical chunks mapped into memory (e.g., `LOAD`, `DYNAMIC`, `STACK`).  |

---

### **ELF File Layout**
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

### **Key ELf Concepts**
#### 1. **ELF Header**
Use `readelf -h hello` to see:
```bash
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64          ← 64-bit
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file) ← FILE TYPE
  Machine:                           Advanced Micro Devices X86-64
  Entry point address:               0x401000       ← Start address
  Program header table offset:       64 (bytes)
  Section header table offset:       9392 (bytes)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         11
  Size of section headers:           64 (bytes)
  Number of section headers:         29
  Section name string table index:   28
```

#### 2. **Program Headers (Segments)**
Use `readelf -l hello` to see:
```bash
Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x00000000000007cc 0x00000000000007cc  R E    200000
  ...
```
This segment tells the OS to map the first 0x7cc bytes of the file to virtual address `0x400000` with **Read + Execute** permissions.

#### 3. **Sections**
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

### **How Does ELF Work?**
1. **Compilation**:  
   GCC generates an ELF object file (`hello.o`) with sections like `.text`, `.data`, and `.symtab`.
2. **Linking**:  
   The linker (`ld`) merges sections from multiple object files, resolves symbols, and creates an executable ELF file with program headers.
3. **Execution**:  
   The OS loads the executable using the program headers to map memory regions and execute code.

---

### **What Does the OS Do When Running `./hello`?**
Here’s the lifecycle from command line to execution:

#### 1. **User Runs the Command**
```bash
./hello
```
The shell calls `execve("./hello", ...)`, which triggers the kernel to load the ELF file.

#### 2. **Kernel Reads the ELF Header**
- Checks the magic number (`\x7fELF`) to confirm it’s an ELF file.
- Parses the ELF header to determine architecture (e.g., x86-64).

#### 3. **Kernel Loads Segments into Memory**
Using the **program headers**, the kernel:
- Maps `.text` (code) to executable memory.
- Maps `.data` and `.bss` to writable memory.
- Sets up the **stack** and environment variables.
- If there’s a `INTERP` segment, loads the dynamic linker (e.g., `/lib64/ld-linux-x86-64.so.2`).

#### 4. **Dynamic Linking (if needed)**
If the program uses shared libraries:
- The kernel transfers control to the dynamic linker.
- The linker resolves dependencies (e.g., `libc.so.6`), loads libraries, and relocates addresses.

#### 5. **Start Execution**
- The kernel jumps to the **entry point** specified in the ELF header (`Entry point address`).
- For C programs, this is typically `_start` (in `crt1.o`), which initializes the C runtime, calls `main()`, and exits cleanly.

---

### **Example: Full Flow for `hello`**
```bash
$ gcc -o hello hello.c  # Compiles to ELF executable
```
1. **File Type**:  
   ```bash
   $ file hello
   hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=..., for GNU/Linux 3.2.0, not stripped
   ```
2. **Entry Point**:  
   ```bash
   $ readelf -h hello | grep "Entry point"
   Entry point address:               0x401000
   ```
3. **Execution Steps**:
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

