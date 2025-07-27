---
date: '2025-05-02T09:18:20+08:00'
draft: false
title: 'Master Elf Sections Dynamic Link'
---

## **Dynamic Linking 涉及到的 section**

| **Section**         | **Type**       | **Purpose**                                                                 | **Example Command**                     |
|---------------------|----------------|-----------------------------------------------------------------------------|-----------------------------------------|
| **`.dynamic`**      | `DYNAMIC`      | Contains metadata for the dynamic linker (e.g., needed libraries, symbol tables, relocations). | `readelf -d demo`                       |
| **`.plt`**          | `PROGBITS`     | **Procedure Linkage Table**: Stubs for external function calls (e.g., `printf`). | `objdump -d demo`                       |
| **`.got` / `.got.plt`** | `PROGBITS` | **Global Offset Table**: Stores resolved addresses for functions and variables. | `objdump -R demo`                       |
| **`.dynsym`**       | `DYNSYM`       | Dynamic symbol table (subset of `.symtab`) for runtime symbol resolution.    | `readelf -s demo`                       |
| **`.dynstr`**       | `STRTAB`       | String table for symbol names in `.dynsym`.                                 | `readelf -p .dynstr demo`               |
| **`.gnu.hash`**     | `GNU_HASH`     | Optimized hash table for fast symbol lookup (replaces legacy `.hash`).       | `readelf -p .gnu.hash demo`             |
| **`.rela.dyn`**     | `RELA`         | Relocations for global variables (e.g., `message`).                          | `readelf -r demo`                       |
| **`.rela.plt`**     | `RELA`         | Relocations for function calls (e.g., `printf`).                            | `readelf -r demo`                       |
| **`.init_array`**   | `INIT_ARRAY`   | Array of constructor functions (marked with `__attribute__((constructor))`). | `readelf -s demo`                      |
| **`.fini_array`**   | `FINI_ARRAY`   | Array of destructor functions (marked with `__attribute__((destructor))`).   | `readelf -s demo`                      |
| **`.interp`**       | `PROGBITS`     | Path to the dynamic linker (e.g., `/lib64/ld-linux-x86-64.so.2`).           | `readelf -p .interp demo`               |

## interp /lib64/ld-linux-x86-64.so.2

> 动态链接程序的入口

## symbol table 符号表分析

The `objdump -t demo` command displays the **symbol table** of the ELF executable `demo`. This table lists `all symbols` (functions, variables, sections, etc.) defined or referenced in the binary. 

```bash

demo:     file format elf64-x86-64

SYMBOL TABLE:
0000000000000000 l    df *ABS*	0000000000000000              Scrt1.o
000000000000038c l     O .note.ABI-tag	0000000000000020              __abi_tag
0000000000000000 l    df *ABS*	0000000000000000              crtstuff.c
00000000000010b0 l     F .text	0000000000000000              deregister_tm_clones
00000000000010e0 l     F .text	0000000000000000              register_tm_clones
0000000000001120 l     F .text	0000000000000000              __do_global_dtors_aux
0000000000004020 l     O .bss	0000000000000001              completed.0
0000000000003db0 l     O .fini_array	0000000000000000              __do_global_dtors_aux_fini_array_entry
0000000000001160 l     F .text	0000000000000000              frame_dummy
0000000000003da0 l     O .init_array	0000000000000000              __frame_dummy_init_array_entry
0000000000000000 l    df *ABS*	0000000000000000              demo.c
0000000000004028 l     O .bss	0000000000000004              count.1
0000000000004014 l     O .data	0000000000000004              local_static.0
0000000000000000 l    df *ABS*	0000000000000000              crtstuff.c
00000000000021f8 l     O .eh_frame	0000000000000000              __FRAME_END__
0000000000000000 l    df *ABS*	0000000000000000              
0000000000003dc0 l     O .dynamic	0000000000000000              _DYNAMIC
0000000000002078 l       .eh_frame_hdr	0000000000000000              __GNU_EH_FRAME_HDR
0000000000003fb0 l     O .got	0000000000000000              _GLOBAL_OFFSET_TABLE_
0000000000004018 g     O .data	0000000000000008              message
0000000000000000       F *UND*	0000000000000000              __libc_start_main@GLIBC_2.34
0000000000000000  w      *UND*	0000000000000000              _ITM_deregisterTMCloneTable
0000000000004000  w      .data	0000000000000000              data_start
0000000000000000       F *UND*	0000000000000000              puts@GLIBC_2.2.5
0000000000004020 g       .data	0000000000000000              _edata
0000000000004010 g     O .data	0000000000000004              global_data
000000000000127c g     F .fini	0000000000000000              .hidden _fini
00000000000011d3 g     F .text	000000000000001a              print_message
0000000000004024 g     O .bss	0000000000000004              global_bss
0000000000000000       F *UND*	0000000000000000              printf@GLIBC_2.2.5
000000000000119f g     F .text	000000000000001a              finalize
00000000000011b9 g     F .text	000000000000001a              initialize
0000000000004000 g       .data	0000000000000000              __data_start
0000000000000000  w      *UND*	0000000000000000              __gmon_start__
0000000000004008 g     O .data	0000000000000000              .hidden __dso_handle
0000000000002000 g     O .rodata	0000000000000004              _IO_stdin_used
0000000000004030 g       .bss	0000000000000000              _end
0000000000001080 g     F .text	0000000000000026              _start
0000000000001169 g     F .text	0000000000000036              counter
0000000000004020 g       .bss	0000000000000000              __bss_start
00000000000011ed g     F .text	000000000000008e              main
0000000000004020 g     O .data	0000000000000000              .hidden __TMC_END__
0000000000000000  w      *UND*	0000000000000000              _ITM_registerTMCloneTable
0000000000000000  w    F *UND*	0000000000000000              __cxa_finalize@GLIBC_2.2.5
0000000000001000 g     F .init	0000000000000000              .hidden _init


```


### General Format of Each Line
Each line in the symbol table has the following format:
```
ADDRESS Binding  TYPE   SECTION  SIZE   NAME
```

### Key Fields in the Output
1. **Address (`00000000000011d3`)**  
   - The `virtual memory address` of the symbol (hexadecimal)
   - For `external/undefined` symbols (`*UND*`), this is `0000000000000000`

2. **Binding (`l`, `g`, `!`, `w`, `u`)** 
   - global(默认的)、local、weak binding
   - **`l`**: Local symbol (`not visible outside the file`; e.g., static functions/variables).  
   - **`g`**: Global symbol (`externally visible`; e.g., `main`, `printf`).  
   - **`!`**: Weak symbol (lower precedence during linking; not shown here).  
   - **`w`**: Weak symbol (unresolved).  还没有解析
   - **`u`**: Unique global symbol (special case for symbol versioning).  

3. **Type (`df`, `O`, `F`)**  
   - **`df`**: `Debug file` symbol (indicates source code filenames like `demo.c`).  
   - **`O`**: `Object` symbol (data variable, e.g., `global_data`).  
   - **`F`**: `Function` symbol (e.g., `main`, `print_message`).  
   - **`UND`**: `Undefined` symbol (external reference, must be resolved at **link** time).  
   - **`ABS`**: `Absolute` symbol (not associated with any section; e.g., constants).  

4. **Section (`*ABS*`, `.text`, `.data`, `.bss`)**  
   - **`*ABS*`**: Absolute address (no section).                        不属于任何 section
   - **`.text`**: Executable code (functions).                          指令
   - **`.data`**: Initialized global/static variables.                  初始化数据
   - **`.bss`**: Uninitialized global/static variables.                 未初始化数据
   - **`.rodata`**: Read-only data (e.g., string literals).             只读数据
   - **`*UND*`**: Undefined section (symbol requires external linkage). 需要外部链接  

5. **Size (`0000000000000004`)**  
   - `Size of the symbol` (in bytes, hexadecimal). For example, `count.1` (a static variable) occupies 4 bytes.

6. **Name (`main`, `printf@GLIBC_2.2.5`)**  
   - `Symbol name`. For external symbols, versioning (e.g., `@GLIBC_2.2.5`) indicates the required library version.


### Key Symbols 总结
| Symbol | Type | Section | Description |
|-------|------|---------|-------------|
| `_start` | `gF` | `.text` | Entry point of the program (CRT startup code). |
| `main` | `gF` | `.text` | Your main function. |
| `print_message` | `gF` | `.text` | A custom function defined in `demo.c`. |
| `global_data` | `gO` | `.data` | Initialized global variable. |
| `global_bss` | `gO` | `.bss` | Uninitialized global variable. |
| `message` | `gO` | `.data` | String literal (e.g., `"Hello, World!"`). |
| `puts@GLIBC_2.2.5` | `u UND ` | `*UND*` | External reference to `puts` from glibc. |
| `__libc_start_main` | `UND` | `*UND*` | CRT function to initialize the program. |
| `_edata`, `_end` | `g` | `.data`, `.bss` | Special symbols marking end of sections. |


### Special Sections and Symbols
- **[`crtstuff.c`](https://github.com/gcc-mirror/gcc/blob/master/libgcc/crtstuff.c) and `Scrt1.o`**: Compiler-generated CRT (C Runtime) code for initialization/finalization.
- [**`_ITM_*`**](https://github.com/gcc-mirror/gcc/blob/master/libgcc/crtstuff.c#L194-L195): Symbols for Intel Transactional Memory (unused unless compiling with `-fgnu-tm`).
- **`.init_array`/`.fini_array`**: Functions to run before/after `main()` (e.g., constructors/destructors).
- **`.eh_frame`**: `Exception handling metadata` (stack unwinding for exceptions or debugging).
- **`.got`**: Global Offset Table (for PIC/position-independent code).



### print_message 分析
```
00000000000011d3 g     F .text  000000000000001a              print_message
```
- **Address**: `0x11d3` (location of `print_message` in memory).  
- **Binding**: `F` (function). 
- **Type**: `g` (global), 
- **Section**: `.text` (code section).  
- **Size**: `0x1a` (26 bytes).  print_message 代码共 26bytes
- **Name**: `print_message` (自定义的函数).  

### printf@GLIBC_2.2.5 分析
```
0000000000000000       F *UND*  0000000000000000              printf@GLIBC_2.2.5
```

| Field | Value | Explanation |
|-------|-------|-------------|
| **Address** | `0000000000000000` | Undefined symbols have no valid address (resolved at runtime). |
| **Binding** | **(implicit `g`)** | Global binding (default for undefined symbols like `printf`). Not explicitly shown here. |
| **Type** | `F` | Function (symbol is a function, not a variable). |
| **Section** | `*UND*` | Undefined (external reference; resolved by the linker/loader). |
| **Size** | `0000000000000000` | Size of the symbol (not applicable for undefined symbols). |
| **Name** | `printf@GLIBC_2.2.5` | Symbol name with versioning (from `glibc`). |


## `.dynstr section 分析`
```
  [ 7] .dynstr           STRTAB           0000000000000498  00000498
       0000000000000094  0000000000000000   A       0     0     1
```
### `.dynstr 二进制查看`

```bash
xxd -s 0x498 -l 0x94 demo
00000498: 0070 7574 7300 5f5f 6c69 6263 5f73 7461  .puts.__libc_sta
000004a8: 7274 5f6d 6169 6e00 5f5f 6378 615f 6669  rt_main.__cxa_fi
000004b8: 6e61 6c69 7a65 0070 7269 6e74 6600 6c69  nalize.printf.li
000004c8: 6263 2e73 6f2e 3600 474c 4942 435f 322e  bc.so.6.GLIBC_2.
000004d8: 322e 3500 474c 4942 435f 322e 3334 005f  2.5.GLIBC_2.34._
000004e8: 4954 4d5f 6465 7265 6769 7374 6572 544d  ITM_deregisterTM
000004f8: 436c 6f6e 6554 6162 6c65 005f 5f67 6d6f  CloneTable.__gmo
00000508: 6e5f 7374 6172 745f 5f00 5f49 544d 5f72  n_start__._ITM_r
00000518: 6567 6973 7465 7254 4d43 6c6f 6e65 5461  egisterTMCloneTa
00000528: 626c 6500                                ble.
```

### `.dynstr` 查看分析

> .dynstr base = 0x498
```bash
readelf -p .dynstr demo

String dump of section '.dynstr':
  [     1]  puts
  [     6]  __libc_start_main
  [    18]  __cxa_finalize
  [    27]  printf
  [    2e]  libc.so.6
  [    38]  GLIBC_2.2.5
  [    44]  GLIBC_2.34
  [    4f]  _ITM_deregisterTMCloneTable
  [    6b]  __gmon_start__
  [    7a]  _ITM_registerTMCloneTable

```
- [     1]  puts 距离 .dynstr 开始处偏移量为 1, 也就是 0x498 + 1 = 0x499。长度为 5。包含一个 NULL terminator



## `.strtab section` 分析

The `.strtab` section in an ELF file is the **string table** that stores **null-terminated strings** referenced by other sections, such as the **symbol table** (`.symtab`) and **section names**. 
These strings include:

- Function names (e.g., `main`, `printf`)
- Variable names (e.g., `global_data`)
- Section names (e.g., `.text`, `.data`)
- Other identifiers used in the binary.

It acts as a centralized repository for strings, allowing sections like `.symtab` to reference names via **offsets (indices)** into `.strtab` instead of embedding strings directly.

### **`.strtab section` 字段解释**
From your `readelf -S demo` output:
```
[35] .strtab           STRTAB           0000000000000000  00003978
     0000000000000245  0000000000000000           0     0     1
```

| Field               | Value              | Explanation |
|---------------------|--------------------|-------------|
| **Name**            | `.strtab`          | String table for symbol/section names. |
| **Type**            | `STRTAB`           | Indicates a string table. |
| **Address**         | `0x0000000000000000` | Not loaded into memory (no runtime use). |
| **Offset**          | `0x00003978`       | File offset to the start of the string table (14,712 bytes into the file). |
| **Size**            | `0x0000000000000245` (581 bytes) | Total size of the string table. |
| **Entry Size**      | `0x0000000000000000` | Not applicable (strings are variable-length). |
| **Flags**           | None               | No special flags (not allocated in memory). |
| **Link**            | `0`                | Not linked to another section (standalone). |
| **Info**            | `0`                | No additional metadata. |
| **Alignment**       | `1`                | Byte-aligned (strings are stored contiguously). |


### **Key Concepts**
1. **Purpose of `.strtab`:**
   - Stores `human-readable names` for `symbols` and `sections`.
   - Reduces redundancy by allowing multiple entries to reference the same string `via offset`.

2. **How Symbols Use `.strtab`:**
   - The symbol table (`.symtab`) references `.strtab` using the `st_name` field.
   - Example: A symbol like `main` in `.symtab` has an `st_name` value (e.g., `0x123`), which is an offset into `.strtab`. 

3. **String Encoding:**
   - Strings are null-terminated (`\0`).
   - The first byte (`offset 0x0`) is always an empty string (`""`), used as a placeholder.

4. **Not Loaded at Runtime:**
   - The `.strtab` section is **not marked as allocatable** (`A` flag), so it is not loaded into memory during program execution.
   - It is primarily used during linking and debugging.


### **`.strtab` 探析**

#### **`readelf -p demo`**
```bash
readelf -p .strtab demo

String dump of section '.strtab':
  [     1]  Scrt1.o
  [     9]  __abi_tag
  [    13]  crtstuff.c
  [    1e]  deregister_tm_clones
  [    33]  __do_global_dtors_aux
  [    49]  completed.0
  [    55]  __do_global_dtors_aux_fini_array_entry
  [    7c]  frame_dummy
  [    88]  __frame_dummy_init_array_entry
  [    a7]  demo.c
  [    ae]  count.1
  [    b6]  local_static.0
  [    c5]  __FRAME_END__
  [    d3]  _DYNAMIC
  [    dc]  __GNU_EH_FRAME_HDR
  [    ef]  _GLOBAL_OFFSET_TABLE_
  [   105]  __libc_start_main@GLIBC_2.34
  [   122]  _ITM_deregisterTMCloneTable
  [   13e]  puts@GLIBC_2.2.5
  [   14f]  _edata
  [   156]  global_data
  [   162]  _fini
  [   168]  print_message
  [   176]  global_bss
  [   181]  printf@GLIBC_2.2.5
  [   194]  finalize
  [   19d]  initialize
  [   1a8]  __data_start
  [   1b5]  __gmon_start__
  [   1c4]  __dso_handle
  [   1d1]  _IO_stdin_used
  [   1e0]  _end
  [   1e5]  counter
  [   1ed]  __bss_start
  [   1f9]  main
  [   1fe]  __TMC_END__
  [   20a]  _ITM_registerTMCloneTable
  [   224]  __cxa_finalize@GLIBC_2.2.5
  [   23f]  _init
```



#### **Differences Between `.strtab` and `.dynstr`**
| Feature          | `.strtab`                  | `.dynstr`                  |
|------------------|----------------------------|----------------------------|
| **Used by**      | `.symtab` (full symbol table) | `.dynsym` (dynamic symbol table) |
| **Loaded at Runtime?** | No (not allocatable)     | Yes (used by dynamic linker) |
| **Purpose**      | Debugging, static linking  | Dynamic linking             |
| **Size**         | Larger (includes all symbols) | Smaller (subset for runtime) |



### **Why `.strtab` Matters**
1. **Symbol Resolution:**  
   The symbol table (`.symtab`) relies on `.strtab` to associate names (e.g., `printf`) with addresses.
2. **Debugging:**  
   Tools like `gdb` and `nm` use `.strtab` to display human-readable names during debugging.
3. **Stripped Binaries:**  
   If `.strtab` is removed (via `strip`), symbols lose their names, making reverse engineering harder.


### **`.strtab section` 总结**
- **`.strtab`** is a string table storing names for symbols and sections.
- It is referenced by `.symtab` via offsets (e.g., `st_name`).
- Use `readelf -p .strtab demo` to view its contents.
- It is not loaded at runtime but is crucial for linking and debugging.


## `.shstrtab section` 分析

The `.shstrtab` section in an ELF file is the **section header string table**, which stores **null-terminated strings** representing the **names of all sections** in the binary. This section is critical for interpreting the **section headers** in the ELF file, as it allows tools like `readelf`, `objdump`, and debuggers to map `numeric section indices` to `human-readable names` (e.g., `.text`, `.data`, `.bss`). link 时使用。运行时不使用。


### **`.shstrtab section` 字段解释**

```
[36] .shstrtab         STRTAB           0000000000000000  00003bbd
     000000000000016a  0000000000000000           0     0     1
```

| Field               | Value              | Explanation |
|---------------------|--------------------|-------------|
| **Name**            | `.shstrtab`        | Section header string table. |
| **Type**            | `STRTAB`           | Indicates a `string table`. |
| **Address**         | `0x0000000000000000` | Not loaded into memory (no runtime use). |
| **Offset**          | `0x00003bbd`       | File offset to the start of the string table (15,277 bytes into the file). |
| **Size**            | `0x000000000000016a` (362 bytes) | Total size of the string table. |
| **Entry Size**      | `0x0000000000000000` | Not applicable (strings are `variable-length`). |
| **Flags**           | None               | No special flags (not allocated in memory). |
| **Link**            | `0`                | Not linked to another section (standalone). |
| **Info**            | `0`                | No additional metadata. |
| **Alignment**       | `1`                | Byte-aligned (strings are stored `contiguously`). |


### **Key Concepts**
1. **Purpose of `.shstrtab`:**
   - Stores **section names** as `null-terminated` strings.
   - Allows section headers to reference names via **offsets** instead of embedding strings directly.
   - Example: A section header for `.text` has an `sh_name` field pointing to the offset of `.text` in `.shstrtab`.

2. **Structure of Strings:**
   - Strings are **null-terminated** (`\0`).
   - The first byte (`offset 0x0`) is always an empty string (`""`), used as a placeholder.
   - Subsequent strings are concatenated, e.g.:
     ```
     [0x00] ""          \0
     [0x01] ".text"     \0
     [0x06] ".data"     \0
     [0x0c] ".bss"      \0
     ...
     ```

3. **How Section Headers Use `.shstrtab`**
   - Each section header has an `sh_name` field, which is an **offset** into `.shstrtab`.
   - Example: A section header with `sh_name = 0x01` points to the string `.text`.

4. **Not Loaded at Runtime:**
   - The `.shstrtab` section is **not marked as allocatable** (`A` flag), so it is not loaded into memory during program execution.
   - It is primarily used during `linking` and analysis.


### **`.shstrtab` 探析**

#### **`readelf -p`: Print String Table**
```bash
readelf -p .shstrtab demo

String dump of section '.shstrtab':
  [     1]  .symtab
  [     9]  .strtab
  [    11]  .shstrtab
  [    1b]  .interp
  [    23]  .note.gnu.property
  [    36]  .note.gnu.build-id
  [    49]  .note.ABI-tag
  [    57]  .gnu.hash
  [    61]  .dynsym
  [    69]  .dynstr
  [    71]  .gnu.version
  [    7e]  .gnu.version_r
  [    8d]  .rela.dyn
  [    97]  .rela.plt
  [    a1]  .init
  [    a7]  .plt.got
  [    b0]  .plt.sec
  [    b9]  .text
  [    bf]  .fini
  [    c5]  .rodata
  [    cd]  .eh_frame_hdr
  [    db]  .eh_frame
  [    e5]  .init_array
  [    f1]  .fini_array
  [    fd]  .dynamic
  [   106]  .data
  [   10c]  .bss
  [   111]  .comment
  [   11a]  .debug_aranges
  [   129]  .debug_info
  [   135]  .debug_abbrev
  [   143]  .debug_line
  [   14f]  .debug_str
  [   15a]  .debug_line_str

  ```
- This lists all section names stored in `.shstrtab`, such as:
  - `.interp` (dynamic linker path)
  - `.note.gnu.property` (security notes)
  - `.text`, `.data`, `.bss`, `.rodata`, `.plt`, `.got`, etc.


#### `.shstrtab` 二进制查看

```bash
xxd -s 0x3bbd -l 0x16a demo
00003bbd: 002e 7379 6d74 6162 002e 7374 7274 6162  ..symtab..strtab
00003bcd: 002e 7368 7374 7274 6162 002e 696e 7465  ..shstrtab..inte
00003bdd: 7270 002e 6e6f 7465 2e67 6e75 2e70 726f  rp..note.gnu.pro
00003bed: 7065 7274 7900 2e6e 6f74 652e 676e 752e  perty..note.gnu.
00003bfd: 6275 696c 642d 6964 002e 6e6f 7465 2e41  build-id..note.A
00003c0d: 4249 2d74 6167 002e 676e 752e 6861 7368  BI-tag..gnu.hash
00003c1d: 002e 6479 6e73 796d 002e 6479 6e73 7472  ..dynsym..dynstr
00003c2d: 002e 676e 752e 7665 7273 696f 6e00 2e67  ..gnu.version..g
00003c3d: 6e75 2e76 6572 7369 6f6e 5f72 002e 7265  nu.version_r..re
00003c4d: 6c61 2e64 796e 002e 7265 6c61 2e70 6c74  la.dyn..rela.plt
00003c5d: 002e 696e 6974 002e 706c 742e 676f 7400  ..init..plt.got.
00003c6d: 2e70 6c74 2e73 6563 002e 7465 7874 002e  .plt.sec..text..
00003c7d: 6669 6e69 002e 726f 6461 7461 002e 6568  fini..rodata..eh
00003c8d: 5f66 7261 6d65 5f68 6472 002e 6568 5f66  _frame_hdr..eh_f
00003c9d: 7261 6d65 002e 696e 6974 5f61 7272 6179  rame..init_array
00003cad: 002e 6669 6e69 5f61 7272 6179 002e 6479  ..fini_array..dy
00003cbd: 6e61 6d69 6300 2e64 6174 6100 2e62 7373  namic..data..bss
00003ccd: 002e 636f 6d6d 656e 7400 2e64 6562 7567  ..comment..debug
00003cdd: 5f61 7261 6e67 6573 002e 6465 6275 675f  _aranges..debug_
00003ced: 696e 666f 002e 6465 6275 675f 6162 6272  info..debug_abbr
00003cfd: 6576 002e 6465 6275 675f 6c69 6e65 002e  ev..debug_line..
00003d0d: 6465 6275 675f 7374 7200 2e64 6562 7567  debug_str..debug
00003d1d: 5f6c 696e 655f 7374 7200                 _line_str.


```

### **Why `.shstrtab` Matters**
1. **Human-Readable Names:**  
   Without `.shstrtab`, section headers would be numeric, making analysis harder.
2. **Efficient Storage:**  
   Avoids duplicating strings across section headers.
3. **Tool Compatibility:**  
   Required by tools like `readelf`, `objdump`, and debuggers to interpret section headers.
4. **Binary Inspection:**  
   Essential for reverse engineering, debugging, and understanding ELF structure.




## `.dynsym section` 分析

The `.dynsym` section in an ELF file is the **dynamic symbol table**, which contains **symbols required for dynamic linking** (e.g., external functions like `printf` from `libc`). This section is crucial for `resolving symbols at runtime`. 运行时解析符号使用

### **`.dynsym section`字段解释**

From your `readelf -S demo` output:
```
[ 6] .dynsym           DYNSYM           00000000000003d8  000003d8
     00000000000000c0  0000000000000018   A       7     1     8
```

| Field               | Value              | Explanation |
|---------------------|--------------------|-------------|
| **Name**            | `.dynsym`          | Dynamic symbol table. |
| **Type**            | `DYNSYM`           | Indicates a dynamic symbol table. |
| **Address**         | `0x00000000000003d8` | Virtual address in memory. |
| **Offset**          | `0x000003d8`       | File offset to the start of the section. |
| **Size**            | `0x00000000000000c0` (192 bytes) | Total size in bytes. |
| **Entry Size**      | `0x0000000000000018` (24 bytes) | Size of each symbol entry. |
| **Flags**           | `A` (Alloc)        | Section occupies memory during execution. |
| **Link**            | `7`                | Index of the `.dynstr` section (holds symbol names). |
| **Info**            | `1`                | Number of local symbols (e.g., `UND` symbols). |
| **Alignment**       | `8`                | Alignment requirement for the section. |

1. link=7 链接到 index = 【7】的 section

### **Key Concepts**
1. **Purpose of `.dynsym`:**
   - Stores **symbols needed for dynamic linking**, such as:
     - External functions (e.g., `printf`, `puts`).
     - Global variables from shared libraries.
   - Symbols are resolved by the dynamic linker (`ld-linux.so`) at runtime.
   - Unlike the full symbol table (`.symtab`), `.dynsym` is a **subset** optimized for runtime use.

2. **Structure of Entries:**
   Each entry in `.dynsym` is 24 bytes (0x18) long, with fields like:
   - **Name**: Offset into `.dynstr` for the symbol name (e.g., `"printf"`).
   - **Value**: Symbol address (0 for unresolved symbols).
   - **Size**: Size of the symbol (0 for functions).
   - **Type & Binding**: Metadata (e.g., `FUNC GLOBAL DEFAULT UND`).
   - **Visibility**: Symbol visibility (default, hidden, etc.).



### **`.dynsym` 查看**


#### **1. `readelf --dyn-syms`**
```bash
readelf --dyn-syms demo

Symbol table '.dynsym' contains 8 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND _[...]@GLIBC_2.34 (2)
     2: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterT[...]
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts@GLIBC_2.2.5 (3)
     4: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
     5: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
     6: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMC[...]
     7: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND [...]@GLIBC_2.2.5 (3)
```


#### **2. `objdump -T`**
```bash
objdump -T demo 

demo:     file format elf64-x86-64

DYNAMIC SYMBOL TABLE:
0000000000000000      DF *UND*  0000000000000000 (GLIBC_2.34) __libc_start_main
0000000000000000  w   D  *UND*  0000000000000000  Base        _ITM_deregisterTMCloneTable
0000000000000000      DF *UND*  0000000000000000 (GLIBC_2.2.5) puts
0000000000000000      DF *UND*  0000000000000000 (GLIBC_2.2.5) printf
0000000000000000  w   D  *UND*  0000000000000000  Base        __gmon_start__
0000000000000000  w   D  *UND*  0000000000000000  Base        _ITM_registerTMCloneTable
0000000000000000  w   DF *UND*  0000000000000000 (GLIBC_2.2.5) __cxa_finalize
```


### **How `.dynsym` Works in Dynamic Linking**
1. **Symbol Resolution:**
   - The dynamic linker uses `.dynsym` entries to locate symbols in shared libraries (e.g., `printf` in `libc`).
   - Each symbol has a name (from `.dynstr`) and metadata (type, binding).

2. **Lazy Binding:**
   - When `printf` is first called, the PLT stub jumps to the GOT entry (`.got`).
   - The GOT entry initially points to the resolver (in `.plt`), which triggers the dynamic linker to resolve the symbol.
   - The dynamic linker uses `.dynsym` to look up `printf` and updates the GOT entry with its address in `libc`.

3. **Key Sections Involved:**
   - `.dynsym`: Contains unresolved symbols (e.g., `printf`).
   - `.dynstr`: String table for symbol names (linked via `.dynsym`'s `Link` field).
   - `.rela.plt`: Relocations for PLT entries (e.g., `printf`'s GOT offset).
   - `.got`: Stores resolved addresses after dynamic linking.






## lazy binding 是怎样工作的?

### demo 执行流程
1. The kernel loads `/lib64/ld-linux-x86-64.so.2` (the dynamic linker).
2. The dynamic linker loads `libc.so.6` (used by `printf`).
3. It resolves `printf` in `libc` and updates the GOT.
4. Constructor `initialize()` runs before `main()`.
5. Destructor `finalize()` runs after `main()` exits.


**lazy binding** for the `printf` function in your code, using the `readelf` output and ELF mechanics. This process involves the **Procedure Linkage Table (PLT)**, **Global Offset Table (GOT)**, and the **dynamic linker**.

---

### **1. Key Sections Involved**
| Section | Address | Purpose |
|--------|---------|---------|
| `.plt` | `0x1020` | Stubs for external function calls (e.g., `printf`). |
| `.got` | `0x3fb0` | Stores resolved addresses of external symbols. |
| `.rela.plt` | `0x678` | Relocations for PLT entries (e.g., `printf`). |
| `.dynsym` | `0x3d8` | Dynamic symbol table (includes unresolved symbols like `printf`). |

---

### **2. Lazy Binding Workflow**
Lazy binding defers symbol resolution until the first call. Here's how `printf` is resolved:

#### **Step 1: Initial Call to `printf`**
- When `print_message()` calls `printf`, it jumps to the **PLT entry** for `printf`:
  ```c
  print_message() {
      printf("%s\n", message); // Jumps to PLT entry for printf
  }
  ```

#### **Step 2: PLT Entry for `printf`**
- The `.plt` section contains a stub for `printf`. 
  
  ```assembly
  0x1020: jmp    *0x2f90(%rip)   # Jump to GOT entry for printf
  0x1026: pushq  $0x0            # Argument for resolver
  0x102b: jmp    0x1000          # Jump to dynamic linker resolver
  ```
- The GOT entry (`0x3fb0 + offset`) initially points to the resolver code in `.plt`.

#### **Step 3: Dynamic Linker Resolves `printf`**
- The resolver (in `.plt`) calls the dynamic linker (`ld-linux-x86-64.so.2`) to:
  1. Look up `printf` in the **symbol table** (`.symtab`, `.dynsym`).
  2. Find the address of `printf` in `libc.so.6`.
  3. Update the GOT entry to point directly to `libc`'s `printf`.

#### **Step 4: Subsequent Calls to `printf`**
- After resolution, the GOT entry points directly to `libc`'s `printf`, bypassing the resolver:
  ```assembly
  0x1020: jmp    *0x2f90(%rip)   # Now jumps directly to libc's printf
  ```

---

### **3. Observing Lazy Binding in Your Code**

#### **Step 1: Find the PLT/GOT Entry for `printf`**
From `readelf -s demo`:
```bash
Symbol table '.symtab' contains 46 entries:
...
30: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND printf@GLIBC_2.2.5
```
- `printf` is an **undefined symbol** (UND), meaning it will be resolved dynamically.

From `readelf -r demo` (relocations):
```bash
Relocation section '.rela.plt' at offset 0x678 contains 2 entries:
    Offset          Info           Type           Sym. Value    Sym. Name + Addend
0x3fb8  0x0000000000020007 R_X86_64_JUMP_SLOT   0               printf + 0
```
- The GOT entry at `0x3fb8` will hold the resolved address of `printf`.

#### **Step 2: Use `gdb` to Trace Lazy Binding**
1. Run the program in GDB:
   ```bash
   gdb ./demo
   ```
2. Set a breakpoint at the PLT entry for `printf`:
   ```gdb
   (gdb) break *0x1020  # First PLT entry (for printf)
   (gdb) run
   ```
3. First call to `printf`:
   - The program stops at `0x1020` (PLT stub).
   - Step into the resolver:
     ```gdb
     (gdb) stepi
     => 0x1026: pushq  $0x0   # Push relocation index
     (gdb) stepi
     => 0x102b: jmp    0x1000 # Jump to resolver (dynamic linker)
     ```
4. After resolution:
   - The GOT entry at `0x3fb8` is updated to `libc`'s `printf` address:
     ```gdb
     (gdb) x/gx 0x3fb8
     0x3fb8: 0x7ffff7e12345  # Address of printf in libc
     ```

#### **Step 3: Verify with `objdump`**
Disassemble the `.plt` section:
```bash
objdump -d demo | grep -A 10 '<printf@plt>'
```
Output:
```assembly
0000000000001020 <printf@plt>:
    1020: ff 25 90 2f 00 00   jmpq   *0x2f90(%rip)   # 0x3fb8 <printf@got>
    1026: 68 00 00 00 00      pushq  $0x0
    102b: e9 d0 ff ff ff      jmpq   1000 <.plt>
```
- The first `jmpq` jumps to the GOT entry at `0x3fb8`.
- The second instruction pushes the relocation index for `printf`.

---

### **4. Summary of Lazy Binding Steps**
1. **Initial Call:**  
   `printf` → `.plt` stub → `.got` entry (points to resolver).
2. **Resolver Invoked:**  
   Resolver calls dynamic linker to resolve `printf` in `libc`.
3. **GOT Updated:**  
   Dynamic linker writes `libc`'s `printf` address to `.got`.
4. **Subsequent Calls:**  
   `.plt` → `.got` → `libc`'s `printf` directly.

---

### **5. Why Lazy Binding Matters**
- **Performance:** Avoids resolving all symbols upfront.
- **Memory Efficiency:** Only resolves symbols that are actually used.
- **Security:** Reduces attack surface by deferring resolution.





