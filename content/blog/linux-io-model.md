---
date: '2025-05-29T15:44:41+08:00'
draft: false
title: 'Linux IO Model'
---

## cpu 基本概念

### CPU 执行的流程

> A CPU core is a thread execution unit – if it has no runnable thread, it `physically` cannot do work. 

```mermaid
flowchart LR
    A[CPU Core] -->|Fetches| B[Instruction Pointer]
    B -->|Executes| C[Current Thread's Code]
    C -->|Needs data| D[Registers/Memory]
    D -->|If data not ready| E[Stall Pipeline]
    E -->|If stall too long| F[Switch to another thread]
    F -->|If no threads ready| G[HALT Instruction]
    G --> H[CPU IDLE]
```

### 现代 CPU idle(空闲) 会怎样?

1. modern CPU 在空闲时会进入低功耗(低电压)状态，称为 C-state。
2. C-state 分为多个级别，C0 是 CPU 正在工作，C1、C2 等是不同的低功耗状态。
3. CPU 会根据负载情况自动调整 C-state。

## 等待发生在何处?

### **The Brutal Truth: Waiting `Never Disappears`**
**Physics law**: I/O operations (disk/network) are **orders of magnitude slower** than CPU.  
- Disk seek: ~10ms (10,000,000 CPU cycles)  
- Network roundtrip: ~100ms (100,000,000 CPU cycles)  

**No technology can eliminate this latency**—it's physics. 
Async I/O doesn't *remove* waiting; it **relocates who bears the cost of waiting**.


### **Where Waiting Happens: The 3 Layers**
#### **1. Hardware Layer (Unavoidable Waiting)**
- Disk controller waits for platter rotation → **kernel cannot bypass this**  
- Network card waits for packets → **kernel cannot bypass this**  
- *This waiting is PHYSICAL and happens regardless of software model.*

#### **2. Kernel Layer (The "Delegation" Illusion)**
- **What the kernel *actually* does**:  
  ```c
  // Simplified Linux kernel pseudo-code for async I/O
  void handle_async_read(request *req) {
      if (data_ready_in_hardware) {  // Hardware completed wait
          copy_data_to_user(req->buffer);
          trigger_user_notification(req->callback); // "Wake up" user-space
      } else {
          queue_request_for_later(req); // Defer until hardware finishes
      }
  }
  ```
- **The kernel *is* waiting**—but **not in your thread's context**.  
  - It uses **kernel threads** (for `aio_*`) or **completion rings** (`io_uring`) to track hardware progress.  
  - *Your thread isn't blocked*, but the kernel *is* managing the wait elsewhere.

#### **3. User-Space Layer (The Critical Nuance)**
- **Your thread *does* "wait"**—but in a **non-wasteful way**:  
  ```java
  // Java AsynchronousFileChannel example
  channel.read(buffer, 0, null, new CompletionHandler<>() {
      public void completed(Integer bytes, Object att) {
          // THIS is where you "wait" - but only AFTER data arrives
          System.out.println("Data processed: " + bytes);
      }
  });
  System.out.println("Thread continues immediately");
  ```
  - **Before data arrives**: Your thread executes other code → **no CPU wasted**.  
  - **When data arrives**: Your thread *must* run the callback → **this is "waiting" in disguise**.  


## 用户空间获取数据前提是什么?

1. 数据准备阶段
   - 内核准备数据，这就引出了问题，内核还没准备好数据，等(进程进入 sleep 状态)还是不等？
     - 比如内核 `sock` 的 [sk_receive_queue](https://elixir.bootlin.com/linux/v6.15/source/include/net/sock.h#L252) 没有数据
   - 内核没准备好数据，进程进入 sleep 状态 -> `阻塞 IO`
   - 内核没准备好数据，进程不进入 sleep 状态 -> `非阻塞 IO`
2. 数据拷贝阶段
   - 数据从内核空间复制到用户空间，这就引出了问题，谁来负责复制? 
     - 比如内核 `sock` 的 [sk_receive_queue](https://elixir.bootlin.com/linux/v6.15/source/include/net/sock.h#L252) 的数据由谁来复制到用户空间
   - 由`用户线程`的内核态来负责 -> `同步 IO`
   - 由`内核线程`负责 -> `异步 IO`
3. 根据以上两大阶段理解 block/non-block sync/async

## I/O 模型

> IO 的同步、异步、阻塞、非阻塞取决于相关 syscall 的实现。

### Blocking I/O

> The process is blocked until the I/O operation completes.
> 阻塞 

1. 用户调用阻塞的 `read` 后，没有数据进程就放弃 CPU，进入阻塞状态(sleep)

### Non-blocking I/O

> The system call returns `immediately`, either with the data or with an error indicating that the data is not ready. 
> The process can then `periodically check` (poll) for readiness.
> 非阻塞

1. 用户自己去调用非阻塞的 `read` 轮询


### I/O Multiplexing

> (e.g., select, poll, epoll)​​: The process can `block` on multiple I/O sources at `once`, and is woken up when `any` of them are ready. 
> This allows one thread to manage multiple I/O operations.

> 同步非阻塞, java 的 NIO 就是这个模型。有一个线程阻塞在 `epoll_wait()`

1. [select](https://man7.org/linux/man-pages/man2/select.2.html) 内核轮询，返回结果。用户再遍历获取哪些 IO ready。
2. [epoll](https://man7.org/linux/man-pages/man2/epoll_wait.2.html)

### Signal-Driven I/O

> The kernel notifies the process via a signal (e.g., SIGIO) when the I/O operation is ready to proceed.
> 信号驱动 SIGIO 只有一种状态

```bash
 kill -l | grep SIGIO
1)  SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR

```

### Asynchronous I/O

> The I/O operation is initiated, and the process continues. The kernel sends a signal when the entire I/O operation is completed.


> Async I/O = `Decoupling` request submission from result processing, where the duration of thread occupancy is minimized to the time required to handle the result—not the time required for the physical I/O operation.  发起请求, 处理结果

> 异步 io_uring





## References

1. [内核角度看 I/O 模型-强烈推荐](https://mp.weixin.qq.com/s?__biz=Mzg2MzU3Mjc3Ng==&mid=2247483737&idx=1&sn=7ef3afbb54289c6e839eed724bb8a9d6&chksm=ce77c71ef9004e08e3d164561e3a2708fc210c05408fa41f7fe338d8e85f39c1ad57519b614e&scene=178&cur_album_id=2559805446807928833&search_click_id=#rd)
2. [syscall 分类-block/non-block](https://mohitmishra786.github.io/chessman/2025/03/31/Technical-Guide-to-System-Calls-Implementation-and-Signal-Handling-in-Modern-Operating-Systems.html)
3. [fcntl-O_NONBLOCK](https://man7.org/linux/man-pages/man2/fcntl.2.html)
4. [IO model-Qwen](https://chat.qwen.ai/c/ca5a7f7b-0084-4812-97d4-2463748d9010)