---
date: '2025-04-29T17:25:37+08:00'
draft: false
title: 'Gcc Compile C Prog'
---

### 分析代码
> os: Linux Mint 22.1
```c {fileName="hello.c"}
#include <stdio.h>
int main(void){
    printf("hello world");
}
```

### strace 追踪编译流程

```bash
strace -t -f -o hello.log -e trace=execve -s 2000 gcc -o hello hello.c
```
All `execve()` system calls during compilation, showing how GCC orchestrates the build process.

#### hello.log 内容

```
20029 13:48:52 execve("/usr/bin/gcc", ["gcc", "-o", "hello", "hello.c"], 0x7ffd60d08358 /* 65 vars */) = 0
20034 13:48:52 execve("/usr/libexec/gcc/x86_64-linux-gnu/13/cc1", ["/usr/libexec/gcc/x86_64-linux-gnu/13/cc1", "-quiet", "-imultiarch", "x86_64-linux-gnu", "hello.c", "-quiet", "-dumpbase", "hello.c", "-dumpbase-ext", ".c", "-mtune=generic", "-march=x86-64", "-fasynchronous-unwind-tables", "-fstack-protector-strong", "-Wformat", "-Wformat-security", "-fstack-clash-protection", "-fcf-protection", "-o", "/tmp/ccLfEVyN.s"], 0x7d14db0 /* 70 vars */) = 0
20034 13:48:53 +++ exited with 0 +++
20029 13:48:53 --- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=20034, si_uid=1000, si_status=0, si_utime=3 /* 0.03 s */, si_stime=17 /* 0.17 s */} ---
20036 13:48:53 execve("/usr/local/sbin/as", ["as", "--64", "-o", "/tmp/ccGZrdFz.o", "/tmp/ccLfEVyN.s"], 0x7d14db0 /* 70 vars */) = -1 ENOENT (No such file or directory)
20036 13:48:53 execve("/usr/local/bin/as", ["as", "--64", "-o", "/tmp/ccGZrdFz.o", "/tmp/ccLfEVyN.s"], 0x7d14db0 /* 70 vars */) = -1 ENOENT (No such file or directory)
20036 13:48:53 execve("/usr/sbin/as", ["as", "--64", "-o", "/tmp/ccGZrdFz.o", "/tmp/ccLfEVyN.s"], 0x7d14db0 /* 70 vars */) = -1 ENOENT (No such file or directory)
20036 13:48:53 execve("/usr/bin/as", ["as", "--64", "-o", "/tmp/ccGZrdFz.o", "/tmp/ccLfEVyN.s"], 0x7d14db0 /* 70 vars */) = 0
20036 13:48:53 +++ exited with 0 +++
20029 13:48:53 --- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=20036, si_uid=1000, si_status=0, si_utime=0, si_stime=1 /* 0.01 s */} ---
20037 13:48:53 execve("/usr/libexec/gcc/x86_64-linux-gnu/13/collect2", ["/usr/libexec/gcc/x86_64-linux-gnu/13/collect2", "-plugin", "/usr/libexec/gcc/x86_64-linux-gnu/13/liblto_plugin.so", "-plugin-opt=/usr/libexec/gcc/x86_64-linux-gnu/13/lto-wrapper", "-plugin-opt=-fresolution=/tmp/ccKbYYb2.res", "-plugin-opt=-pass-through=-lgcc", "-plugin-opt=-pass-through=-lgcc_s", "-plugin-opt=-pass-through=-lc", "-plugin-opt=-pass-through=-lgcc", "-plugin-opt=-pass-through=-lgcc_s", "--build-id", "--eh-frame-hdr", "-m", "elf_x86_64", "--hash-style=gnu", "--as-needed", "-dynamic-linker", "/lib64/ld-linux-x86-64.so.2", "-pie", "-z", "now", "-z", "relro", "-o", "hello", "/usr/lib/gcc/x86_64-linux-gnu/13/../../../x86_64-linux-gnu/Scrt1.o", "/usr/lib/gcc/x86_64-linux-gnu/13/../../../x86_64-linux-gnu/crti.o", "/usr/lib/gcc/x86_64-linux-gnu/13/crtbeginS.o", "-L/usr/lib/gcc/x86_64-linux-gnu/13", "-L/usr/lib/gcc/x86_64-linux-gnu/13/../../../x86_64-linux-gnu", "-L/usr/lib/gcc/x86_64-linux-gnu/13/../../../../lib", "-L/lib/x86_64-linux-gnu", "-L/lib/../lib", "-L/usr/lib/x86_64-linux-gnu", "-L/usr/lib/../lib", "-L/usr/lib/gcc/x86_64-linux-gnu/13/../../..", "/tmp/ccGZrdFz.o", "-lgcc", "--push-state", "--as-needed", "-lgcc_s", "--pop-state", "-lc", "-lgcc", "--push-state", "--as-needed", "-lgcc_s", "--pop-state", "/usr/lib/gcc/x86_64-linux-gnu/13/crtendS.o", "/usr/lib/gcc/x86_64-linux-gnu/13/../../../x86_64-linux-gnu/crtn.o"], 0x7d15df0 /* 72 vars */) = 0
20038 13:48:53 execve("/usr/bin/ld", ["/usr/bin/ld", "-plugin", "/usr/libexec/gcc/x86_64-linux-gnu/13/liblto_plugin.so", "-plugin-opt=/usr/libexec/gcc/x86_64-linux-gnu/13/lto-wrapper", "-plugin-opt=-fresolution=/tmp/ccKbYYb2.res", "-plugin-opt=-pass-through=-lgcc", "-plugin-opt=-pass-through=-lgcc_s", "-plugin-opt=-pass-through=-lc", "-plugin-opt=-pass-through=-lgcc", "-plugin-opt=-pass-through=-lgcc_s", "--build-id", "--eh-frame-hdr", "-m", "elf_x86_64", "--hash-style=gnu", "--as-needed", "-dynamic-linker", "/lib64/ld-linux-x86-64.so.2", "-pie", "-z", "now", "-z", "relro", "-o", "hello", "/usr/lib/gcc/x86_64-linux-gnu/13/../../../x86_64-linux-gnu/Scrt1.o", "/usr/lib/gcc/x86_64-linux-gnu/13/../../../x86_64-linux-gnu/crti.o", "/usr/lib/gcc/x86_64-linux-gnu/13/crtbeginS.o", "-L/usr/lib/gcc/x86_64-linux-gnu/13", "-L/usr/lib/gcc/x86_64-linux-gnu/13/../../../x86_64-linux-gnu", "-L/usr/lib/gcc/x86_64-linux-gnu/13/../../../../lib", "-L/lib/x86_64-linux-gnu", "-L/lib/../lib", "-L/usr/lib/x86_64-linux-gnu", "-L/usr/lib/../lib", "-L/usr/lib/gcc/x86_64-linux-gnu/13/../../..", "/tmp/ccGZrdFz.o", "-lgcc", "--push-state", "--as-needed", "-lgcc_s", "--pop-state", "-lc", "-lgcc", "--push-state", "--as-needed", "-lgcc_s", "--pop-state", "/usr/lib/gcc/x86_64-linux-gnu/13/crtendS.o", "/usr/lib/gcc/x86_64-linux-gnu/13/../../../x86_64-linux-gnu/crtn.o"], 0x7ffd0bf6c850 /* 72 vars */) = 0
20038 13:48:54 +++ exited with 0 +++
20037 13:48:54 --- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=20038, si_uid=1000, si_status=0, si_utime=4 /* 0.04 s */, si_stime=23 /* 0.23 s */} ---
20037 13:48:54 +++ exited with 0 +++
20029 13:48:54 --- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=20037, si_uid=1000, si_status=0, si_utime=0, si_stime=3 /* 0.03 s */} ---
20029 13:48:54 +++ exited with 0 +++
```

---

### **编译流程分析**

#### **Initial GCC Invocation**
```bash
execve("/usr/bin/gcc", ["gcc", "-o", "hello", "hello.c"], ...)
```
- **Process ID**: `20029`  
- **Action**: Invokes the GCC driver (`/usr/bin/gcc`).

---

#### **Compilation Phase**
```bash
execve("/usr/libexec/gcc/x86_64-linux-gnu/13/cc1", ["/usr/libexec/gcc/x86_64-linux-gnu/13/cc1", ...], ...)
```
- **Process ID**: `20034`  
- **Role**:  
  - `cc1` is the **GCC C compiler frontend**.  
  - Converts `hello.c` to **assembly code** (`/tmp/ccLfEVyN.s`).  
- **Flags Passed**:  
  Includes optimization settings (e.g., `-mtune=generic`, `-march=x86-64`), security features (`-fstack-protector-strong`, `-Wformat-security`), and others.  

**Result**:  
Assembly file generated at `/tmp/ccLfEVyN.s`.

---

#### **Assembly Phase**
Attempts to locate the **assembler (`as`)**:
```bash
execve("/usr/local/sbin/as", ...) = -1 ENOENT
execve("/usr/local/bin/as", ...) = -1 ENOENT
execve("/usr/sbin/as", ...) = -1 ENOENT
execve("/usr/bin/as", ...) = 0
```
- **Process ID**: `20036`  
- **Why Multiple Tries?**:  
  GCC searches standard paths for the assembler. Succeeds with `/usr/bin/as` (GNU Assembler).  
- **Action**:  
  Assembles `/tmp/ccLfEVyN.s` into an object file: `/tmp/ccGZrdFz.o`.

---

#### **Linking Phase**
```bash
execve("/usr/libexec/gcc/x86_64-linux-gnu/13/collect2", [...], ...)
```
- **Process ID**: `20037`  
- **Role**:  
  - Wrapper for the linker (`ld`).  
  - Prepares metadata for C++ static constructors/destructors (not needed here since `hello.c` is plain C).  
  - Invokes the **real linker** (`/usr/bin/ld`).  

**Internal Linker Call**:
```bash
execve("/usr/bin/ld", [...], ...)
```
- **Process ID**: `20038`  
- **Key Flags**:  
  - `-dynamic-linker /lib64/ld-linux-x86-64.so.2`: Sets the dynamic loader.  
  - `-pie`: Creates a **Position-Independent Executable** (security feature).  
  - `-z now` and `-z relro`: Enhance runtime security.  
- **Inputs Linked**:  
  - CRT (C Runtime) files (`Scrt1.o`, `crti.o`, `crtbeginS.o`, `crtendS.o`, `crtn.o`).  
  - Object file: `/tmp/ccGZrdFz.o`.  
  - Standard libraries: `-lgcc`, `-lgcc_s`, `-lc` (libc).  

**Result**:  
Final executable `hello` is created.

---

### **Success Indicators**
- All `execve()` calls return `0` (success).  
- Processes exit cleanly (`+++ exited with 0 +++`).  
- No errors detected in the toolchain path or linking.  

---

### **Security & Optimization Notes**
- **Stack Protection**: Enabled via `-fstack-protector-strong`.  
- **Hardened Linking**:  
  - `-z now`: Immediate binding of symbols.  
  - `-z relro`: Read-only relocations after relocation.  
  - `-pie`: ASLR-friendly executable.  
- **Build ID**: Embedded via `--build-id` for debugging/tracing.  

---

### **Critical Paths**
| Tool          | Path                                | Purpose                          |
|---------------|-------------------------------------|----------------------------------|
| `cc1`         | `/usr/libexec/gcc/x86_64-linux-gnu/13/cc1` | Compiles C to assembly           |
| `as`          | `/usr/bin/as`                       | Assembles `.s` to `.o`           |
| `collect2`    | `/usr/libexec/gcc/x86_64-linux-gnu/13/collect2` | Manages linker setup             |
| `ld`          | `/usr/bin/ld`                       | Links objects into final binary  |

---

### **Conclusion**
The log confirms a **standard GCC compilation pipeline**:
1. **Preprocessing + Parsing**: Handled by `cc1`.  
2. **Assembly**: Done by `as`.  
3. **Linking**: Orchestrated by `collect2` → `ld`.  

