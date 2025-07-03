---
date: '2025-07-03T11:47:45+08:00'
draft: false
title: 'Java How Jvm Works'
---

how modern JVMs (like HotSpot/OpenJDK, OpenJ9, or GraalVM) work? 
1. 这是一个经过多层次抽象的软件系统。
2. 想要彻底了解，需要理解每一层，显然要知道每一层是做什么的。
3. 每一层的边界在哪里。
4. 问题出于哪一层。


## **1. Java Layer**
- **Java Code**: application code (e.g., `MyClass.java`) is compiled into **bytecode** (`.class` files).
- **Bytecode Execution**: The JVM interprets this bytecode or compiles it to native machine code using a **Just-In-Time (JIT) compiler** (e.g., C1/C2 in HotSpot, or the newer Graal compiler).
- **Java APIs**: Even core Java APIs (e.g., `java.lang.Object`, `java.io.File`) are implemented in Java but rely on **native methods** (via JNI/`native` keywords) for low-level operations.



## **2. C/C++ Layer (JVM Implementation)**
- **JVM Internals**: The JVM itself (e.g., HotSpot) is primarily written in **C++**. This layer handles:
  - **Memory Management**: Garbage collection (G1, ZGC, Shenandoah, etc.).
  - **Threading**: Thread scheduling and synchronization.
  - **JIT Compilation**: Translating bytecode to optimized machine code.
  - **Class Loading**: Parsing `.class` files and verifying bytecode.
- **JNI (Java Native Interface)**: When Java code calls `native` methods (e.g., `java.lang.Thread.start()`), the JVM delegates to C++ code via JNI. This bridges Java and native code.
- **OS Abstraction**: The JVM uses platform-specific C/C++ code (e.g., `os_linux.cpp`, `os_windows.cpp`) to handle OS differences.



## **3. System Call Layer**
- **OS Interaction**: The JVM’s C++ code interacts with the operating system through **system calls** (e.g., `read()`, `write()`, `mmap()`). These are often wrapped by:
  - **Standard Libraries** (e.g., glibc on Linux, MSVCRT on Windows).
  - **Direct Syscall Wrappers**: For performance-critical paths (e.g., thread scheduling via `futex()` on Linux).
- **Kernel Mode**: System calls transition into **kernel mode** to access hardware resources (e.g., file I/O, network sockets, memory allocation).



## **Visualizing the Full Chain**
```
Java Application Code
   ↓ (Bytecode)
JVM (C++/C) → JIT Compiler → Machine Code
   ↓ (Native Methods, GC, Threading)
OS Libraries (glibc, MSVCRT) → System Calls → Kernel
```



## **Key Details to Clarify**
1. **Not All Paths Go Directly to System Calls**:
   - Many operations (e.g., arithmetic, object allocation) are handled purely in user space by the JVM/JIT.
   - Only I/O, threading, memory management, etc., require system calls.

2. **JNI and Native Libraries**:
   - Java applications can directly load native libraries (e.g., via `System.loadLibrary`) and call C/C++ code, bypassing the JVM’s internal layers. This adds another path:
     ```
     Java → JNI → Native Library (C/C++) → System Calls
     ```

3. **AOT Compilation (Ahead-of-Time)**:
   - Tools like **GraalVM Native Image** compile Java directly to native machine code ahead of time, skipping the JVM runtime. Here, the chain becomes:
     ```
     Java → Native Code (via AOT) → System Calls
     ```

4. **OS Abstraction in the JVM**:
   - The JVM hides OS differences. For example, thread creation in Java uses `pthread_create()` on Linux/macOS but `CreateThread()` on Windows, all abstracted by the JVM’s C++ code.



## **Example: File I/O in Java**
```java
FileInputStream fis = new FileInputStream("file.txt");
int data = fis.read(); // Java → C++ → System Call → Kernel
```
1. **Java Layer**: `FileInputStream.read()` (Java method).
2. **JNI**: Calls `JVM_Read()` (native method in JVM’s C++ code).
3. **C++ Layer**: Uses `read()` system call via OS-specific wrappers.
4. **System Call**: The kernel reads data from disk.



## **Summary**
The full stack is:

**`Java (Bytecode/App)`** → **`JVM (C++/C)`** → OS Libraries → System Calls → Kernel

This abstraction allows Java to maintain **write-once, run-anywhere** portability while leveraging low-level system capabilities through the JVM’s native implementation.


## References

1. [java 面向对象是怎样实现的?](https://www.zhihu.com/question/1921502817790726530/answer/1922327406754108184)
