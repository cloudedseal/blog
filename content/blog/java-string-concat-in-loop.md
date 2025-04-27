---
title: "Java String Concat in Loop"
date: 2025-04-09T10:56:10+08:00
draft: false
tags: ["string","java","bytecode"]
---

# 分析环境
> jdk8

# 分析代码
```java
public class StringConcat {
    public static void main(String[] args) {
        String s = "$";
        for (int i = 0; i < 10; i++) {
            s += i;
        }
        System.out.println(s);
    }
}
```
# main 字节码

```java
  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String $
       2: astore_1
       3: iconst_0
       4: istore_2
       5: iload_2
       6: bipush        10
       8: if_icmpge     36
      11: new           #3                  // class java/lang/StringBuilder
      14: dup
      15: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
      18: aload_1
      19: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      22: iload_2
      23: invokevirtual #6                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      26: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      29: astore_1
      30: iinc          2, 1
      33: goto          5
      36: getstatic     #8                  // Field java/lang/System.out:Ljava/io/PrintStream;
      39: aload_1
      40: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      43: return

```

# main 字节码执行图解分析


![循环里字符串连接.drawio.svg](https://raw.githubusercontent.com/buybyte/pictures/main/img/java-string-concat-in-loop.drawio.svg)

> 由图可以看出，在外部定义的字符串变量 s，在 for 循环中和其他变量进行拼接，最终得到连接后的 s。<br>
> 循环的每一轮都会生成一个新的 StringBuilder 对象，进行两次 append 操作，append(s) 和 append(i)。最终这一轮的 s 是由 sb.toString() 得来的。<br>
> 如果循环次数是 1万次，需要创建的 StringBuilder 是 1 万个。这样是对性能肯定有影响的。短时间内产生大量的对象。<br>

# 结论

> 循环次数过多时，不要在循环里拼接需要的字符串。

# References
1. https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-6.html#jvms-6.5.ldc
2. https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-6.html#jvms-6.5.astore_n
3. https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-6.html#jvms-6.5.iload_n
4. https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-6.html#jvms-6.5.bipush
5. https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-6.html#jvms-6.5.if_icmp_cond
6. https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-6.html#jvms-6.5.new
7. https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-6.html#jvms-6.5.iinc
8. https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-6.html#jvms-6.5.getstatic
9. https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-6.html#jvms-6.5.invokevirtual
