---
date: '2025-04-30T10:48:42+08:00'
draft: false
title: 'Master Elf Sections'
---

> 分析环境 mint21
## ELF (linking view) 分析
1. 一个 ELF 文件有多种 view。
2. 从 linking 视角来理解 sections 到底是如何工作的。
3. 源代码的各个不同的部分编译后都被放到了哪个 section。
4. 本文主要分析  rodata text data bss 这 4 个 section。

## 示例代码

> gcc -g -o demo demo.c

[编译后的各个文件](https://github.com/cloudedseal/elf-demo/)

```c {fileName="demo.c"}
#include <stdio.h>
#include <stdlib.h>

// Global variable (initialized) → .data
int global_data = 42;

// Global variable (uninitialized) → .bss
int global_bss;

// Constant string → .rodata
const char *message = "Hello from .rodata!";

// Static local variable → gcc stored in .bss (not .bss, = 0 it's uninitialized)
void counter() {
    static int count = 0; 
    count++;
    printf("Counter: %d\n", count);
}

// Destructor function → .fini_array
__attribute__((destructor)) void finalize() {
    printf("Finalizing...\n");
}

// Constructor function → .init_array
__attribute__((constructor)) void initialize() {
    printf("Initializing...\n");
}

// Function in .text
void print_message() {
    printf("%s\n", message); // Uses PLT/GOT for printf
}

int main() {
    // Local static variable → stored in .data (initialized)
    static int local_static = 100;
    
    printf("Global Data: %d\n", global_data);
    printf("Global BSS: %d\n", global_bss);
    printf("Local Static: %d\n", local_static);
    
    print_message();
    
    for (int i = 0; i < 3; i++) {
        counter(); // Modifies static variable in .data
    }

    return 0;
}
```

### readelf -h 查看基本信息

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
  Entry point address:               0x1080
  Start of program headers:          64 (bytes into file)
  Start of section headers:          15656 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         37
  Section header string table index: 36

```

| header 信息 | 注意 |
|---------|----------|
| Data:  2's complement, `little endian` | 注意: 使用的是小端。数据高位在高地址 |
|   Number of section headers:       37  |    一共 37 个 section |


## readelf -t (Display the section details)

```bash
There are 37 section headers, starting at offset 0x3d28:

Section Headers:
  [Nr] Name
       Type              Address          Offset            Link
       Size              EntSize          Info              Align
       Flags
  [ 0] 
       NULL             0000000000000000  0000000000000000  0
       0000000000000000 0000000000000000  0                 0
       [0000000000000000]: 
  [ 1] .interp
       PROGBITS         0000000000000318  0000000000000318  0
       000000000000001c 0000000000000000  0                 1
       [0000000000000002]: ALLOC
  [ 2] .note.gnu.property
       NOTE             0000000000000338  0000000000000338  0
       0000000000000030 0000000000000000  0                 8
       [0000000000000002]: ALLOC
  [ 3] .note.gnu.build-id
       NOTE             0000000000000368  0000000000000368  0
       0000000000000024 0000000000000000  0                 4
       [0000000000000002]: ALLOC
  [ 4] .note.ABI-tag
       NOTE             000000000000038c  000000000000038c  0
       0000000000000020 0000000000000000  0                 4
       [0000000000000002]: ALLOC
  [ 5] .gnu.hash
       GNU_HASH         00000000000003b0  00000000000003b0  6
       0000000000000024 0000000000000000  0                 8
       [0000000000000002]: ALLOC
  [ 6] .dynsym
       DYNSYM           00000000000003d8  00000000000003d8  7
       00000000000000c0 0000000000000018  1                 8
       [0000000000000002]: ALLOC
  [ 7] .dynstr
       STRTAB           0000000000000498  0000000000000498  0
       0000000000000094 0000000000000000  0                 1
       [0000000000000002]: ALLOC
  [ 8] .gnu.version
       VERSYM           000000000000052c  000000000000052c  6
       0000000000000010 0000000000000002  0                 2
       [0000000000000002]: ALLOC
  [ 9] .gnu.version_r
       VERNEED          0000000000000540  0000000000000540  7
       0000000000000030 0000000000000000  1                 8
       [0000000000000002]: ALLOC
  [10] .rela.dyn
       RELA             0000000000000570  0000000000000570  6
       0000000000000108 0000000000000018  0                 8
       [0000000000000002]: ALLOC
  [11] .rela.plt
       RELA             0000000000000678  0000000000000678  6
       0000000000000030 0000000000000018  24                8
       [0000000000000042]: ALLOC, INFO LINK
  [12] .init
       PROGBITS         0000000000001000  0000000000001000  0
       000000000000001b 0000000000000000  0                 4
       [0000000000000006]: ALLOC, EXEC
  [13] .plt
       PROGBITS         0000000000001020  0000000000001020  0
       0000000000000030 0000000000000010  0                 16
       [0000000000000006]: ALLOC, EXEC
  [14] .plt.got
       PROGBITS         0000000000001050  0000000000001050  0
       0000000000000010 0000000000000010  0                 16
       [0000000000000006]: ALLOC, EXEC
  [15] .plt.sec
       PROGBITS         0000000000001060  0000000000001060  0
       0000000000000020 0000000000000010  0                 16
       [0000000000000006]: ALLOC, EXEC
  [16] .text
       PROGBITS         0000000000001080  0000000000001080  0
       00000000000001fb 0000000000000000  0                 16
       [0000000000000006]: ALLOC, EXEC
  [17] .fini
       PROGBITS         000000000000127c  000000000000127c  0
       000000000000000d 0000000000000000  0                 4
       [0000000000000006]: ALLOC, EXEC
  [18] .rodata
       PROGBITS         0000000000002000  0000000000002000  0
       0000000000000076 0000000000000000  0                 4
       [0000000000000002]: ALLOC
  [19] .eh_frame_hdr
       PROGBITS         0000000000002078  0000000000002078  0
       0000000000000054 0000000000000000  0                 4
       [0000000000000002]: ALLOC
  [20] .eh_frame
       PROGBITS         00000000000020d0  00000000000020d0  0
       000000000000012c 0000000000000000  0                 8
       [0000000000000002]: ALLOC
  [21] .init_array
       INIT_ARRAY       0000000000003da0  0000000000002da0  0
       0000000000000010 0000000000000008  0                 8
       [0000000000000003]: WRITE, ALLOC
  [22] .fini_array
       FINI_ARRAY       0000000000003db0  0000000000002db0  0
       0000000000000010 0000000000000008  0                 8
       [0000000000000003]: WRITE, ALLOC
  [23] .dynamic
       DYNAMIC          0000000000003dc0  0000000000002dc0  7
       00000000000001f0 0000000000000010  0                 8
       [0000000000000003]: WRITE, ALLOC
  [24] .got
       PROGBITS         0000000000003fb0  0000000000002fb0  0
       0000000000000050 0000000000000008  0                 8
       [0000000000000003]: WRITE, ALLOC
  [25] .data
       PROGBITS         0000000000004000  0000000000003000  0
       0000000000000020 0000000000000000  0                 8
       [0000000000000003]: WRITE, ALLOC
  [26] .bss
       NOBITS           0000000000004020  0000000000003020  0
       0000000000000010 0000000000000000  0                 4
       [0000000000000003]: WRITE, ALLOC
  [27] .comment
       PROGBITS         0000000000000000  0000000000003020  0
       000000000000002b 0000000000000001  0                 1
       [0000000000000030]: MERGE, STRINGS
  [28] .debug_aranges
       PROGBITS         0000000000000000  000000000000304b  0
       0000000000000030 0000000000000000  0                 1
       [0000000000000000]: 
  [29] .debug_info
       PROGBITS         0000000000000000  000000000000307b  0
       00000000000001ae 0000000000000000  0                 1
       [0000000000000000]: 
  [30] .debug_abbrev
       PROGBITS         0000000000000000  0000000000003229  0
       00000000000000e8 0000000000000000  0                 1
       [0000000000000000]: 
  [31] .debug_line
       PROGBITS         0000000000000000  0000000000003311  0
       00000000000000a2 0000000000000000  0                 1
       [0000000000000000]: 
  [32] .debug_str
       PROGBITS         0000000000000000  00000000000033b3  0
       000000000000013e 0000000000000001  0                 1
       [0000000000000030]: MERGE, STRINGS
  [33] .debug_line_str
       PROGBITS         0000000000000000  00000000000034f1  0
       0000000000000030 0000000000000001  0                 1
       [0000000000000030]: MERGE, STRINGS
  [34] .symtab
       SYMTAB           0000000000000000  0000000000003528  35
       0000000000000450 0000000000000018  20                8
       [0000000000000000]: 
  [35] .strtab
       STRTAB           0000000000000000  0000000000003978  0
       0000000000000245 0000000000000000  0                 1
       [0000000000000000]: 
  [36] .shstrtab
       STRTAB           0000000000000000  0000000000003bbd  0
       000000000000016a 0000000000000000  0                 1
       [0000000000000000]: 
 

```

## readelf -S (section headers)

```bash
There are 37 section headers, starting at offset 0x3d28:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000000318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.gnu.pr[...] NOTE             0000000000000338  00000338
       0000000000000030  0000000000000000   A       0     0     8
  [ 3] .note.gnu.bu[...] NOTE             0000000000000368  00000368
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .note.ABI-tag     NOTE             000000000000038c  0000038c
       0000000000000020  0000000000000000   A       0     0     4
  [ 5] .gnu.hash         GNU_HASH         00000000000003b0  000003b0
       0000000000000024  0000000000000000   A       6     0     8
  [ 6] .dynsym           DYNSYM           00000000000003d8  000003d8
       00000000000000c0  0000000000000018   A       7     1     8
  [ 7] .dynstr           STRTAB           0000000000000498  00000498
       0000000000000094  0000000000000000   A       0     0     1
  [ 8] .gnu.version      VERSYM           000000000000052c  0000052c
       0000000000000010  0000000000000002   A       6     0     2
  [ 9] .gnu.version_r    VERNEED          0000000000000540  00000540
       0000000000000030  0000000000000000   A       7     1     8
  [10] .rela.dyn         RELA             0000000000000570  00000570
       0000000000000108  0000000000000018   A       6     0     8
  [11] .rela.plt         RELA             0000000000000678  00000678
       0000000000000030  0000000000000018  AI       6    24     8
  [12] .init             PROGBITS         0000000000001000  00001000
       000000000000001b  0000000000000000  AX       0     0     4
  [13] .plt              PROGBITS         0000000000001020  00001020
       0000000000000030  0000000000000010  AX       0     0     16
  [14] .plt.got          PROGBITS         0000000000001050  00001050
       0000000000000010  0000000000000010  AX       0     0     16
  [15] .plt.sec          PROGBITS         0000000000001060  00001060
       0000000000000020  0000000000000010  AX       0     0     16
  [16] .text             PROGBITS         0000000000001080  00001080
       00000000000001fb  0000000000000000  AX       0     0     16
  [17] .fini             PROGBITS         000000000000127c  0000127c
       000000000000000d  0000000000000000  AX       0     0     4
  [18] .rodata           PROGBITS         0000000000002000  00002000
       0000000000000076  0000000000000000   A       0     0     4
  [19] .eh_frame_hdr     PROGBITS         0000000000002078  00002078
       0000000000000054  0000000000000000   A       0     0     4
  [20] .eh_frame         PROGBITS         00000000000020d0  000020d0
       000000000000012c  0000000000000000   A       0     0     8
  [21] .init_array       INIT_ARRAY       0000000000003da0  00002da0
       0000000000000010  0000000000000008  WA       0     0     8
  [22] .fini_array       FINI_ARRAY       0000000000003db0  00002db0
       0000000000000010  0000000000000008  WA       0     0     8
  [23] .dynamic          DYNAMIC          0000000000003dc0  00002dc0
       00000000000001f0  0000000000000010  WA       7     0     8
  [24] .got              PROGBITS         0000000000003fb0  00002fb0
       0000000000000050  0000000000000008  WA       0     0     8
  [25] .data             PROGBITS         0000000000004000  00003000
       0000000000000020  0000000000000000  WA       0     0     8
  [26] .bss              NOBITS           0000000000004020  00003020
       0000000000000010  0000000000000000  WA       0     0     4
  [27] .comment          PROGBITS         0000000000000000  00003020
       000000000000002b  0000000000000001  MS       0     0     1
  [28] .debug_aranges    PROGBITS         0000000000000000  0000304b
       0000000000000030  0000000000000000           0     0     1
  [29] .debug_info       PROGBITS         0000000000000000  0000307b
       00000000000001ae  0000000000000000           0     0     1
  [30] .debug_abbrev     PROGBITS         0000000000000000  00003229
       00000000000000e8  0000000000000000           0     0     1
  [31] .debug_line       PROGBITS         0000000000000000  00003311
       00000000000000a2  0000000000000000           0     0     1
  [32] .debug_str        PROGBITS         0000000000000000  000033b3
       000000000000013e  0000000000000001  MS       0     0     1
  [33] .debug_line_str   PROGBITS         0000000000000000  000034f1
       0000000000000030  0000000000000001  MS       0     0     1
  [34] .symtab           SYMTAB           0000000000000000  00003528
       0000000000000450  0000000000000018          35    20     8
  [35] .strtab           STRTAB           0000000000000000  00003978
       0000000000000245  0000000000000000           0     0     1
  [36] .shstrtab         STRTAB           0000000000000000  00003bbd
       000000000000016a  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)


```

## `.data` section 分析

```
[25] .data             PROGBITS         0000000000004000  00003000
       0000000000000020  0000000000000000  WA       0     0     8
```


### .data section 字段解释 
| Field | Value | Meaning |
|-------|-------|---------|
| **[25]** | Section Index | The `25th section` in the ELF file. Used internally by tools like `readelf`. |
| **.data** | Section Name | Holds **initialized global/static variables** (e.g., `global_data`, `local_static`, `message`). |
| **PROGBITS** | Section Type | Contains raw data (code, initialized data) stored in the file. Not dynamically generated at runtime. |
| **0000000000004000** | `Virtual Address (VA)` | At `runtime`, this section will be mapped to virtual address `0x4000` in memory. |
| **00003000** | File Offset | In the file (`demo`), this section starts at byte offset `0x3000` (12288 in decimal). |
| **0000000000000020** | Size | Size of the section in bytes: `0x20` (32 bytes). Spans from `0x3000` to `0x3020` in the file. |
| **0000000000000000** | Entry Size (EntSize) | If the section contains fixed-size entries (e.g., symbol tables), this specifies their size. Here it’s `0` → no fixed-size entries. |
| **WA** | Flags | Section attributes:  <br/>`W` = **Writable** (memory can be modified at runtime).<br>  A = **Allocated** (`loaded` into memory during execution). <br/> No `X` → this section is not executable. 
| **0** | Link | Relates to other sections; used for symbol tables or relocation (value `0` here = unused). |
| **0** | Info | Extra information (e.g., section index of related section); unused here. |
| **8** | Alignment | Memory alignment requirement: must start at an address divisible by `8`. |



### **Key Takeaways**
1. **Virtual Address (VA): `0x4000`**  
   - At `runtime`, the `.data` section is mapped to virtual address `0x4000`.
   - This is a **runtime memory address**, not a file offset.
   - Example: Your `message` variable is located at `0x4000 + 0x18 = 0x4018`.

2. **File Offset: `0x3000`**  
   - In the file (`demo`), the `.data` section starts at byte `12288` (hex `0x3000`).
   - Since the file size is **16320 bytes (~0x4000)**, this offset is valid.

3. **Size: `0x20` (32 bytes)**  
   - The section spans 32 bytes on disk (`0x3000` to `0x3020`) and in memory (`0x4000` to `0x4020`).

4. **Flags: `WA` (Writable + Allocated)**  
   - The OS loads this section into memory with **write permissions**.
   - You can modify variables like `global_data` at runtime.

5. **Alignment: `8`**  
   - Ensures efficient memory access by aligning the section to 8-byte boundaries.



### 查看 demo 中 0x3000(.data) 后 `0x20`(32) 字节的内容

```bash
xxd -l 32 -s 0x3000 demo
00003000: 0000 0000 0000 0000 0840 0000 0000 0000  .........@......
00003010: 2a00 0000 6400 0000 0420 0000 0000 0000  *...d.... ......
```

### global variable 之 message 分析

```c
const char *message = "Hello from .rodata!";
```

#### `message` 符号信息

```bash
nm demo | grep message
0000000000004018 D message
00000000000011d3 T print_message
```

- `message` is a pointer variable stored in .data.
- `message` virtual address = 0x4018
- `data section` 中的 virtual address, 
- `message` offset = VA(message) - VA(start of .data) = 0x4018 - 0x4000 = 0x18
- `message` 这个 global variable 的指针在 `demo` 的 offset 是 0x000003000 + 0x18 = 0x3018

1. 可知 0x3018 处 8 字节的内容为 `0420 0000 0000 0000`
2. ELF 小端存储, 所以值为 `0x2004`, 这就是 `message` 变量的值

#### 查找 `message` 字符串内容

```bash
xxd demo | grep -A 1 "Hello from"
00002000: 0100 0200 4865 6c6c 6f20 6672 6f6d 202e  ....Hello from .
00002010: 726f 6461 7461 2100 436f 756e 7465 723a  rodata!.Counter:
```
1. 可知 `Hello from .rodata!` 这一串字符所在文件 offset = `0x00002004`
2. 这就是 data section 中 global variable `message` 所指向的字符串

#### 查找 `message` 字符串内容在何处?

> 在 .rodata 只读数据 section 中
```
  [18] .rodata           PROGBITS         0000000000002000  00002000
       0000000000000076  0000000000000000   A       0     0     4
```

### `initialized global variable` 之 global_data

#### global_data 符号信息
```bash
nm demo | grep global_data
0000000000004010 D global_data
```

```text
00003000: 0000 0000 0000 0000 0840 0000 0000 0000  .........@......
00003010: 2a00 0000 6400 0000 0420 0000 0000 0000  *...d.... ......
```
- `global_data` offset = VA(message) - VA(start of .data) = 0x4010 - 0x4000 = 0x10
- `global_data` 这个 global variable 的指针在 `demo` 的 offset 是 0x000003000 + 0x10 = 0x3010
- int global_data 为 4bytes。`2a00 0000` 转为小端 0x0000002a, 就是 42

#### 替换 global_data 的值

使用脚本[update.sh](https://github.com/cloudedseal/elf-demo/blob/main/update.sh)

##### 操纵 demo 文件中 offset = 0x3010 这 1 个 byte。🤣

> global_data 重新设置为 0xff

```bash
./demo 
Initializing...
Global Data: 42
Global BSS: 0
Local Static: 100
Hello from .rodata!
Counter: 1
Counter: 2
Counter: 3
Finalizing...

./update.sh demo 0x3010 0xff
Updated byte at offset 12304 (0x3010) to 0xff

./demo 
Initializing...
Global Data: 255 #🤣🤣🤣，这就是逆向。
Global BSS: 0
Local Static: 100
Hello from .rodata!
Counter: 1
Counter: 2
Counter: 3
Finalizing...

```
##### 操纵 demo 文件中 offset = (0x3010,0x3013) 这 4 个 byte。🤣

> global_data 重新设置为 0x12345678, 注意是小端。高位高地址

```bash
./update.sh demo 0x3010 0x78
Updated byte at offset 12304 (0x3010) to 0x78
./update.sh demo 0x3011 0x56
Updated byte at offset 12305 (0x3011) to 0x56
./update.sh demo 0x3012 0x34
Updated byte at offset 12306 (0x3012) to 0x34
./update.sh demo 0x3013 0x12
Updated byte at offset 12307 (0x3013) to 0x12
./demo 
Initializing...
Global Data: 305419896
Global BSS: 0
Local Static: 100
Hello from .rodata!
Counter: 1
Counter: 2
Counter: 3
Finalizing...
```



### **.data section 总结**
| Section | File Offset | Size (Bytes) | Virtual Address | Runtime Permissions |
|--------|-------------|--------------|------------------|---------------------|
| `.data` | `0x3000`    | `0x20` (32)  | `0x4000`         | Writable + Allocated (`WA`) |


## `.bss` section 分析

```
[26] .bss              NOBITS           0000000000004020  00003020
       0000000000000010  0000000000000000  WA       0     0     4
```



### .bss section 字段解释 
| Field | Value | Meaning |
|-------|-------|---------|
| **[26]** | Section Index | The 26th section in the ELF file. Tools use this internally. |
| **.bss** | Section Name | Stands for **Block Started by Symbol**. Holds **uninitialized global/static variables** (e.g., `global_bss`, `count.1` in your code). |
| **NOBITS** | Section Type | Indicates this section does **not occupy space in the file** (`NOBITS` = no bytes on disk). Memory is allocated at runtime and initialized to zero. |
| **0000000000004020** | Virtual Address (VA) | At runtime, this section will be mapped to virtual address `0x4020`. |
| **00003020** | File Offset | In the file (`demo`), this section starts at byte `0x3020`. Since it’s `NOBITS`, this value is **unused** (placeholder for alignment/padding). |
| **0000000000000010** | Size | Size of the section **in memory**: `0x10` (16 bytes). The OS allocates this much zeroed memory `at runtime`. |
| **0000000000000000** | Entry Size (EntSize) | Not applicable here; set to `0` because `.bss` contains no fixed-size entries. |
| **WA** | Flags | Section attributes: <br/>`W` = **Writable** (variables can be modified at runtime).<br/>  `A` = **Allocated** (memory is reserved at runtime). <br/> No `X` → not executable.  |
| **0** | Link | Relates to other sections; unused here. |
| **0** | Info | Extra metadata; unused here. |
| **4** | Alignment | Must start at an address divisible by `4` in memory. |



### **Key Takeaways**
1. **Purpose of `.bss`**  
   Stores **uninitialized global/static variables**, such as:
   ```c
   int global_bss;            // Uninitialized global variable
   static int count.1;        // Static local variable in `counter()`
   ```
   These are zeroed out at runtime.

2. **No Space in File (`NOBITS`)**  
   - Unlike `.data`, `.bss` does **not store values in the file**.  🤔 不占用存储空间
   - The OS allocates and initializes this section to zero during program startup.  
   - Saves disk space: zero-filled regions don’t need explicit storage.

3. **Runtime Allocation**  
   - Virtual Address: `0x4020` (where `.bss` lives in memory).  
   - Size: `0x10` (16 bytes) → covers all uninitialized variables.  
   - Example: `global_bss` is 4 bytes and `count.1` is 4 bytes

4. **Flags: `WA` (Writable + Allocated)**  
   - Writable: Variables like `global_bss` can be modified at runtime.  
   - Allocated: Reserved in memory, but not stored in the file.

5. **File Offset: `0x3020`**  
   - Exists only for padding/alignment in the file.  
   - Ignored by the OS because `.bss` is `NOBITS`.

6. **Alignment: `4`**  
   - Ensures efficient memory access by aligning the section to 4-byte boundaries.



### **Why `.bss` Is Efficient**
- **Disk Space**: Does not duplicate zeros in the file (saves space).  
- **Memory**: Zeroed at runtime by the OS using `calloc`-like behavior.  
- **Security**: Prevents leaking old memory contents (ensures clean state).



### `uninitialized global variable` 之 global_bss

#### global_bss 符号信息
```bash
nm demo | grep global_bss
0000000000004024 B global_bss
```

### `uninitialized static local variable` 之 count

#### count 符号信息
```bash
nm demo | grep count.1
0000000000004028 b count.1
```

### 不占用存储空间的 bss

```
  [25] .data             PROGBITS         0000000000004000  00003000
       0000000000000020  0000000000000000  WA       0     0     8
  [26] .bss              NOBITS           0000000000004020  00003020
       0000000000000010  0000000000000000  WA       0     0     4
  [27] .comment          PROGBITS         0000000000000000  00003020
       000000000000002b  0000000000000001  MS       0     0     1
```

### demo 中 25、26、27 三个 section 布局

```txt
+---------------------+  0x3000 (start of .data)
|   0x20(32 bytes)
+---------------------+  0x3020 (end of .bss, start of .bss, end of .bss, start of .comment)
|   0x2b(42 bytes) 
+---------------------+  0x304b (end of .comment)
```

> .bss section 在 demo 中不占实际的存储空间

### **.bss section 总结**
| Section | File Offset | Size (Bytes) | Virtual Address | Runtime Permissions | Content Type             |
|--------|-------------|--------------|------------------|---------------------|--------------------------|
| `.bss` | `0x3020`    | `0x0` (no bits stored) | `0x4020`    | Writable + Allocated (`WA`) | Uninitialized variables (`zeroed at runtime`) |


## `.text` section 分析

### **.text 字段解释**
| Field | Value | Meaning |
|-------|--------|---------|
| **[16]** | Section index | `.text` is the 16th section in the ELF file. |
| **`.text`** | Section name | Contains **machine code** (compiled functions). |
| **`PROGBITS`** | Section type | Data is stored in the file (not `NOBITS`). |
| **`0000000000001080`** | **Virtual Address (VA)** | This section is mapped to virtual address `0x1080` (relative to the randomized PIE base address). |
| **`00001080`** | **File Offset** | Start of the `.text` section in the file: byte `0x1080` (4224 in decimal). |
| **`00000000000001fb`** | **Size** | Size of the `.text` section: `0x1fb` bytes (507 bytes). |
| **`0000000000000000`** | **Entry Size (`EntSize`)** | Not used for `.text` (applies to sections with fixed-size entries, like `.symtab`). |
| **`AX`** | Flags | Section attributes: <br/> `A` = **Allocated** (loaded into memory). <br/> `X` = **Executable** (code can run as machine instructions). |
| **`0`** | Link | Unused for `.text` (used for symbol tables). |
| **`0`** | Info | Unused for `.text` (used for `.text` with extra metadata). |
| **`16`** | Alignment | Must be aligned to a **16-byte boundary** in memory. |

 Virtual Address = `0000000000001080` 和 readelf -h 查看的 entry point address 一致


### **Key Takeaways**
1. **Purpose of `.text`**:
   - Contains **compiled machine code** (e.g., `main()`, `counter()`, `print_message()`).
   - Stored in the file and loaded into memory as executable code.

2. **Virtual Address (VA)**:
   - At runtime, the `.text` section starts at virtual address `0x1080` (relative to the base address chosen by ASLR).
   - Example: If the base address is `0x555555554000`, `.text` starts at `0x555555555080`.

3. **File Offset**:
   - In the file (`demo`), `.text` starts at byte `0x1080` (4224 in decimal).
   - This is where the compiler stores the **raw machine code** for your functions.

4. **Size: `0x1fb` (507 bytes)**:
   - Total size of the `.text` section in both the file and memory.
   - Spans from `0x1080` to `0x1080 + 0x1fb = 0x127b`.

5. **Flags: `AX` (Allocated + eXecutable)**:
   - Memory is marked **executable** (`X`), allowing CPU to run the code.
   - Memory is **allocated** (`A`), but not **writable** (`W`).

6. **Alignment: `16`**:
   - Ensures the `.text` section starts at a 16-byte aligned address in memory.
   - Optimizes instruction cache and CPU performance.



### `.text section` 之 counter function 分析

```c
void counter() {
    static int count = 0;
    count++;
    printf("Counter: %d\n", count);
}

```

#### counter 符号信息

```bash
nm demo | grep counter
0000000000001169 T counter
```

#### objdump 查看 counter 

```bash
objdump demo --disassemble=counter

demo:     file format elf64-x86-64


Disassembly of section .init:

Disassembly of section .plt:

Disassembly of section .plt.got:

Disassembly of section .plt.sec:

Disassembly of section .text:

0000000000001169 <counter>:
    1169:       f3 0f 1e fa             endbr64
    116d:       55                      push   %rbp
    116e:       48 89 e5                mov    %rsp,%rbp
    1171:       8b 05 b1 2e 00 00       mov    0x2eb1(%rip),%eax        # 4028 <count.1>
    1177:       83 c0 01                add    $0x1,%eax
    117a:       89 05 a8 2e 00 00       mov    %eax,0x2ea8(%rip)        # 4028 <count.1>
    1180:       8b 05 a2 2e 00 00       mov    0x2ea2(%rip),%eax        # 4028 <count.1>
    1186:       89 c6                   mov    %eax,%esi
    1188:       48 8d 05 89 0e 00 00    lea    0xe89(%rip),%rax        # 2018 <_IO_stdin_used+0x18>
    118f:       48 89 c7                mov    %rax,%rdi
    1192:       b8 00 00 00 00          mov    $0x0,%eax
    1197:       e8 d4 fe ff ff          call   1070 <printf@plt>
    119c:       90                      nop
    119d:       5d                      pop    %rbp
    119e:       c3                      ret

Disassembly of section .fini:
```

### `.text section` 之 print_message function 分析

```c
void print_message() {
    printf("%s\n", message);
}
```

#### print_message 符号信息

```bash
nm demo | grep print_message
00000000000011d3 T print_message
```

#### objdump 查看 print_message 

```bash
objdump demo --disassemble=print_message

demo:     file format elf64-x86-64


Disassembly of section .init:

Disassembly of section .plt:

Disassembly of section .plt.got:

Disassembly of section .plt.sec:

Disassembly of section .text:

00000000000011d3 <print_message>:
    11d3:       f3 0f 1e fa             endbr64
    11d7:       55                      push   %rbp
    11d8:       48 89 e5                mov    %rsp,%rbp
    11db:       48 8b 05 36 2e 00 00    mov    0x2e36(%rip),%rax        # 4018 <message>
    11e2:       48 89 c7                mov    %rax,%rdi
    11e5:       e8 76 fe ff ff          call   1060 <puts@plt>
    11ea:       90                      nop
    11eb:       5d                      pop    %rbp
    11ec:       c3                      ret

Disassembly of section .fini:
```

### `.text section` 之 main function 分析

#### main 符号信息

```
nm demo | grep main
                 U __libc_start_main@GLIBC_2.34
00000000000011ed T main
```

#### objdump 查看 main 

```bash
 objdump demo --disassemble=main

demo:     file format elf64-x86-64


Disassembly of section .init:

Disassembly of section .plt:

Disassembly of section .plt.got:

Disassembly of section .plt.sec:

Disassembly of section .text:

00000000000011ed <main>:
    11ed:       f3 0f 1e fa             endbr64
    11f1:       55                      push   %rbp
    11f2:       48 89 e5                mov    %rsp,%rbp
    11f5:       48 83 ec 10             sub    $0x10,%rsp
    11f9:       8b 05 11 2e 00 00       mov    0x2e11(%rip),%eax        # 4010 <global_data>
    11ff:       89 c6                   mov    %eax,%esi
    1201:       48 8d 05 3b 0e 00 00    lea    0xe3b(%rip),%rax        # 2043 <_IO_stdin_used+0x43>
    1208:       48 89 c7                mov    %rax,%rdi
    120b:       b8 00 00 00 00          mov    $0x0,%eax
    1210:       e8 5b fe ff ff          call   1070 <printf@plt>
    1215:       8b 05 09 2e 00 00       mov    0x2e09(%rip),%eax        # 4024 <global_bss>
    121b:       89 c6                   mov    %eax,%esi
    121d:       48 8d 05 30 0e 00 00    lea    0xe30(%rip),%rax        # 2054 <_IO_stdin_used+0x54>
    1224:       48 89 c7                mov    %rax,%rdi
    1227:       b8 00 00 00 00          mov    $0x0,%eax
    122c:       e8 3f fe ff ff          call   1070 <printf@plt>
    1231:       8b 05 dd 2d 00 00       mov    0x2ddd(%rip),%eax        # 4014 <local_static.0>
    1237:       89 c6                   mov    %eax,%esi
    1239:       48 8d 05 24 0e 00 00    lea    0xe24(%rip),%rax        # 2064 <_IO_stdin_used+0x64>
    1240:       48 89 c7                mov    %rax,%rdi
    1243:       b8 00 00 00 00          mov    $0x0,%eax
    1248:       e8 23 fe ff ff          call   1070 <printf@plt>
    124d:       b8 00 00 00 00          mov    $0x0,%eax
    1252:       e8 7c ff ff ff          call   11d3 <print_message>
    1257:       c7 45 fc 00 00 00 00    movl   $0x0,-0x4(%rbp)
    125e:       eb 0e                   jmp    126e <main+0x81>
    1260:       b8 00 00 00 00          mov    $0x0,%eax
    1265:       e8 ff fe ff ff          call   1169 <counter>
    126a:       83 45 fc 01             addl   $0x1,-0x4(%rbp)
    126e:       83 7d fc 02             cmpl   $0x2,-0x4(%rbp)
    1272:       7e ec                   jle    1260 <main+0x73>
    1274:       b8 00 00 00 00          mov    $0x0,%eax
    1279:       c9                      leave
    127a:       c3                      ret

Disassembly of section .fini:
```



### `.text section` 之 initialize function 分析


#### objdump 查看 initialize

```bash
objdump demo --disassemble=initialize

demo:     file format elf64-x86-64


Disassembly of section .init:

Disassembly of section .plt:

Disassembly of section .plt.got:

Disassembly of section .plt.sec:

Disassembly of section .text:

00000000000011b9 <initialize>:
    11b9:       f3 0f 1e fa             endbr64
    11bd:       55                      push   %rbp
    11be:       48 89 e5                mov    %rsp,%rbp
    11c1:       48 8d 05 6b 0e 00 00    lea    0xe6b(%rip),%rax        # 2033 <_IO_stdin_used+0x33>
    11c8:       48 89 c7                mov    %rax,%rdi
    11cb:       e8 90 fe ff ff          call   1060 <puts@plt>
    11d0:       90                      nop
    11d1:       5d                      pop    %rbp
    11d2:       c3                      ret

Disassembly of section .fini:

```


### `.text section` 之 finalize function 分析

#### objdump 查看 finalize
```bash
objdump demo --disassemble=finalize

demo:     file format elf64-x86-64


Disassembly of section .init:

Disassembly of section .plt:

Disassembly of section .plt.got:

Disassembly of section .plt.sec:

Disassembly of section .text:

000000000000119f <finalize>:
    119f:       f3 0f 1e fa             endbr64
    11a3:       55                      push   %rbp
    11a4:       48 89 e5                mov    %rsp,%rbp
    11a7:       48 8d 05 77 0e 00 00    lea    0xe77(%rip),%rax        # 2025 <_IO_stdin_used+0x25>
    11ae:       48 89 c7                mov    %rax,%rdi
    11b1:       e8 aa fe ff ff          call   1060 <puts@plt>
    11b6:       90                      nop
    11b7:       5d                      pop    %rbp
    11b8:       c3                      ret

Disassembly of section .fini:
```

### **Security Implications**
- **NX Bit (No-eXecute)**:  
  The `.text` section is executable (`X`), but **not writable** (`W`), preventing code injection.
- **ASLR (Address Space Layout Randomization)**:  
  Virtual addresses like `0x1080` are **offsets from a randomized base address**:
  ```
  Runtime Address of _init = `Base` + 0x1000
  Runtime Address of main = `Base` + 0x11ed
  ```

### **Memory Mapping at Runtime**
When you run `./demo`, the OS maps the `.text` section into memory with execute permissions:
```bash
cat /proc/self/maps
```
Output (example):
```
555555555000-555555556000 r-xp 0x1000 ./demo
```
- `r-xp`: Read + eXecute, not Writable.
- `0x1000`: File offset for `.text`.
- `0x1fb`: Size of `.text` (507 bytes).


### **.text 总结**
| Section | Purpose | Virtual Address | File Offset | Size (Bytes) | Permissions | Notes |
|--------|--------|----------------|-------------|-------------|-------------|-------|
| `.text` | Executable code | `0x1080` | `0x1080` | `0x1fb` (507) | `r-x` | Not writable (security) |

This section is critical for program execution, containing all your compiled functions and runtime initialization code.

## `.rodata` section 分析

```
[18] .rodata           PROGBITS         0000000000002000  00002000
       0000000000000076  0000000000000000   A       0     0     4
```


### **`.rodata` section 字段解释**
| Field | Value | Meaning |
|-------|--------|---------|
| **[18]** | Section index | `.rodata` is the 18th section in the ELF file. |
| **`.rodata`** | Section name | Holds **read-only data** (constants, string literals, etc.). |
| **`PROGBITS`** | Section type | Data is stored in the file (not dynamically generated). |
| **`0000000000002000`** | **Virtual Address (VA)** | At runtime, this section is mapped to memory address `0x2000` (relative to the randomized PIE base address). |
| **`00002000`** | **File Offset** | In the file (`demo`), `.rodata` starts at byte `0x2000` (8192 in decimal). |
| **`0000000000000076`** | **Size** | Size of `.rodata` section: `0x76` (118 bytes). |
| **`0000000000000000`** | **Entry Size (`EntSize`)** | Not applicable (no fixed-size entries). |
| **`A`** | Flags | Section is **Allocated** (loaded into memory). No `W` or `X` → **read-only**. |
| **`0`** | Link | Unused for `.rodata`. |
| **`0`** | Info | Unused for `.rodata`. |
| **`4`** | Alignment | Must be aligned to **4-byte boundaries** in memory. |


### **Purpose of `.rodata`**
The `.rodata` section stores **read-only data** that doesn’t change during execution:
- **String literals**: e.g., `"Hello from .rodata!"`.
- **Constant variables**: e.g., `const int x = 42;`.
- **Jump tables**: Used for `switch()` statements.
- **Build metadata**: e.g., `__abi_tag`, `BuildID`.


### **Why Is `.rodata` Read-Only?**
- **Security**: Prevents runtime modification of constants (e.g., format strings, lookup tables).
- **Optimization**: Shared across processes (e.g., `libc`'s string constants).
- **Memory Protection**: Violations (e.g., writing to `.rodata`) cause **segmentation faults**.


### **Memory Mapping at Runtime**
When the program runs:
- The OS maps `.rodata` to virtual address `0x2000` (relative to the randomized base address).
- `.rodata` is marked **read-only** in memory (no write permissions).

Example `/proc/self/maps` output:
```
555555556000-555555557000 r--p 0x2000 ./demo
```
- `r--p`: Read-only, not writable or executable.


### **Security Implications**
- **NX Bit (No-eXecute)**:  
  `.rodata` is **not executable**, preventing code injection.
- **Writable Protection**:  
  Attempting to modify `.rodata` at runtime (e.g., via a casted pointer) will crash the program:
  ```c
  char *s = (char *)"Hello from .rodata!";
  s[0] = 'h'; // Will cause a segmentation fault
  ```
- **ASLR Compatibility**:  
  `.rodata` is part of the randomized memory layout in PIE binaries.



### **Key Takeaways**
| Concept               | Value                     | Notes |
|-----------------------|---------------------------|-------|
| **Section Type**       | `PROGBITS`                | Contains actual data in the file. |
| **Virtual Address**    | `0x2000`                 | Runtime address (relative to PIE base). |
| **File Offset**       | `0x2000`                 | String `"Hello from .rodata!"` is stored here on disk. |
| **Size**              | `0x76` (118 bytes)        | Total space used by `.rodata`. |
| **Flags**             | `A` (Allocated)            | Loaded into memory as read-only. |
| **Alignment**         | `4`                       | Must start at a 4-byte aligned address. |


### 查看 demo 中 0x2000(.rodata) 后 `0x76`(118) 字节的内容

```bash
xxd -l 0x76 -s 0x2000 demo
00002000: 0100 0200 4865 6c6c 6f20 6672 6f6d 202e  ....Hello from .
00002010: 726f 6461 7461 2100 436f 756e 7465 723a  rodata!.Counter:
00002020: 2025 640a 0046 696e 616c 697a 696e 672e   %d..Finalizing.
00002030: 2e2e 0049 6e69 7469 616c 697a 696e 672e  ...Initializing.
00002040: 2e2e 0047 6c6f 6261 6c20 4461 7461 3a20  ...Global Data: 
00002050: 2564 0a00 476c 6f62 616c 2042 5353 3a20  %d..Global BSS: 
00002060: 2564 0a00 4c6f 6361 6c20 5374 6174 6963  %d..Local Static
00002070: 3a20 2564 0a00                           : %d..
```

### demo 中 0x2000(.rodata) 后 `0x76`(118) 字节的内容分析

1. 0x20 space 展示用 `@` 代替
2. 0x0a line feed 展示用 `#` 代替
3. 0x00 null 作为字符串结束标志，展示用 `😂` 代替
   
```bash
00002000: 0100 0200 4865 6c6c 6f20 6672 6f6d 202e  ....Hello@from@.
00002010: 726f 6461 7461 2100 436f 756e 7465 723a  rodata!😂Counter:
00002020: 2025 640a 0046 696e 616c 697a 696e 672e  @%d#😂Finalizing.
00002030: 2e2e 0049 6e69 7469 616c 697a 696e 672e  ..😂Initializing.
00002040: 2e2e 0047 6c6f 6261 6c20 4461 7461 3a20  ..😂Global@Data:@
00002050: 2564 0a00 476c 6f62 616c 2042 5353 3a20  %d#😂Global@BSS:@
00002060: 2564 0a00 4c6f 6361 6c20 5374 6174 6963  %d#😂Local@Static
00002070: 3a20 2564 0a00                           :@%d#😂
```

The `.rodata` section's **4-byte alignment** ensures that the **section itself** starts at a 4-byte aligned address in both the file and memory, but **individual strings inside it do not need to be 4byte aligned**. 


### **Section Alignment vs. Internal Data Alignment**

####  **Section Alignment**
- **`Align: 4`** in `readelf -S` means the `.rodata` section must start at an address divisible by 4 in memory and in the file.
- In your file:
  ```
  File Offset: 0x2000 (8192) → divisible by 4 (0x2000 % 4 = 0).
  ```
- In memory:
  ```
  Virtual Address: 0x2000 → also divisible by 4 (0x2000 % 4 = 0).
  ```
- This alignment is enforced for **efficient memory access**, not for internal strings.

####  **Internal Strings Are Packed**
- The compiler places strings **sequentially** in `.rodata` without padding between them.
- Example from your hex dump:
  ```
  00002000: 0100 0200 4865 6c6c 6f20 6672 6f6d 202e  
            ^^^^^^^^^^   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
            __abi_tag     "Hello from .rodata!\0"
  ```
  - `"Hello from .rodata!\0"` starts at `0x2004` (after the 4-byte header).
  - Next string `"Global Data: %d\n\0"` starts at `0x2018` (no padding between `0x2014` and `0x2018`).


### **How Alignment Affects Storage**
Even though the section is aligned to 4 bytes, the compiler **packs strings tightly** to save space. Alignment only matters for:
1. **Section Start**: Ensures the entire `.rodata` section begins at a 4byte boundary.
2. **Data Access**: If code accesses parts of `.rodata` as **4byte aligned values** (e.g., via pointer arithmetic), the compiler ensures those values are aligned.

### 修改 `message` 信息

`Hello from .rodata!` 修改为
`ABCDE from .rodata!`

```bash
./update.sh demo 0x2004 0x41
Updated byte at offset 8196 (0x2004) to 0x41
~/elf-demo$ ./update.sh demo 0x2005 0x42
Updated byte at offset 8197 (0x2005) to 0x42
~/elf-demo$ ./update.sh demo 0x2006 0x43
Updated byte at offset 8198 (0x2006) to 0x43
~/elf-demo$ ./update.sh demo 0x2007 0x44
Updated byte at offset 8199 (0x2007) to 0x44
~/elf-demo$ ./update.sh demo 0x2008 0x45
Updated byte at offset 8200 (0x2008) to 0x45

~/elf-demo$ ./demo 
Initializing...
Global Data: 42
Global BSS: 0
Local Static: 100
ABCDE from .rodata! # 修改成功🤣
Counter: 1
Counter: 2
Counter: 3
Finalizing...
```


### **.rodata section  总结**
- **4byte alignment** applies to the **section itself**, not individual strings.
- **`__abi_tag`** adds 4 bytes to `.rodata`.
- **Null terminators** add 7 bytes (1 per string).

## data、bss、rodata、text

> data、rodata、text 在 demo 中位置关系

![data、rodata、text](https://raw.githubusercontent.com/cloudedseal/pictures/main/img/demo-elf-sections-part1.drawio.svg)