---
title: "Null Device"
date: 2025-01-03T10:53:57+08:00
draft: false
---

# NULL device 是啥？

The null device is a special file that `discards` all data written to it, but reports that the `write operation succeeded`.

# 用在何处？

> java 执行外部的命令, 但是不需要外部命令的执行结果. 直接丢弃 stdout, stderr.

```java
import java.io.*;

public class DiscardProcessOutput {
    public static void main(String[] args) {
        try {
            // Create the ProcessBuilder with the command to run
            ProcessBuilder processBuilder = new ProcessBuilder("someCommand");

            // Merge stdout and stderr and redirect them to /dev/null (or NUL in Windows)
            processBuilder.redirectErrorStream(true);
            if (System.getProperty("os.name").contains("Windows")) {
                processBuilder.redirectOutput(new File("NUL"));
            } else {
                processBuilder.redirectOutput(new File("/dev/null"));
            }

            // Start the process
            Process process = processBuilder.start();

            // Wait for the process to finish
            int exitCode = process.waitFor();
            System.out.println("Process exited with code: " + exitCode);

        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```








# References
1. [dev-null](https://www.putorius.net/introduction-to-dev-null.html)
2. [windows-NUL](https://ss64.com/nt/nul.html)
3. [man7 /dev/null](https://man7.org/linux/man-pages/man4/null.4.html)
4. [NUL in Windows seems to be actually a virtual path in any folder](https://stackoverflow.com/questions/313111/is-there-a-dev-null-on-windows)