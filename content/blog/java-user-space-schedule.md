---
title: "Java User Space Schedule"
date: 2025-03-10T17:13:48+08:00
draft: false
---

> A user-space scheduler manages tasks or "threads" entirely within an application, without relying on the operating system (OS) scheduler. This is different from kernel-level scheduling, where the OS manages threads.
> 什么是用户态调度？可以简单理解为自定义一堆任务, 由用户决定怎样执行这些任务。不涉及操作系统的进程/线程调度。


## Task 自定义的任务

```java
public class Task {
    private String name;
    private int steps;

    public Task(String name, int steps) {
        this.name = name;
        this.steps = steps;
    }

    public boolean runStep() {
        if (steps <= 0) return false; // Task is done
        System.out.println(name + " is running. Steps left: " + steps);
        steps--;
        return true; // Task has more work
    }
}
```

## 自定义的调度

```java{ hl_lines=["16-19"] style=emacs }
import java.util.LinkedList;
import java.util.Queue;

public class UserSpaceScheduler {
    // 以 FIFO 顺序执行 Task
    private Queue<Task> taskQueue = new LinkedList<>();

    public void addTask(Task task) {
        taskQueue.add(task);
    }

    public void run() {
        while (!taskQueue.isEmpty()) {
            Task task = taskQueue.poll();
            boolean hasMoreWork = task.runStep();
            if (hasMoreWork) {
//                deciding which task runs next
                taskQueue.add(task); // Re-add to the queue if not done
            }
        }
    }

}
```

## Driver class

```java
public class Main {
    public static void main(String[] args) {
        UserSpaceScheduler scheduler = new UserSpaceScheduler();
        scheduler.addTask(new Task("Task A", 3));
        scheduler.addTask(new Task("Task B", 5));
        scheduler.addTask(new Task("Task C", 2));

        scheduler.run();
    }
}
```

## 如何工作

### Cooperative Multitasking
> Tasks are not OS threads. They are objects managed in a queue.
> 不是 OS 的线程

### Explicit Yielding
> The scheduler runs each task for one "step," then moves to the next. Tasks don’t block—they voluntarily give up control.

### No OS Involvement 
> The OS sees only the main thread running scheduler.run().

