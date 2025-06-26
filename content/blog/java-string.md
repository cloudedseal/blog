---
date: '2025-06-25T10:40:29+08:00'
draft: false
title: 'Java String'
---

> 起因：需要通过 JNI 调用外部程序。传 String 类型的数据，对使用的 java 字符串使用的字符集不清晰。

## 字符集用在哪？

### java 源代码

javac 编译`源代码`(文本文件)时可以指定源代码使用的字符集

```bash
javac -encoding <encoding>       Specify character encoding used by source files
```

### java 字节码文件中的字符串存储

常量字符串比如 `emoji` String s = "🤣" 使用的是 [modified UTF-8](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.4.7)

### java 字符串内部表示

所有 String 对象、常量字符串内部存储使用的是 `UTF-16`



## 默认的字符集
1. `defaultCharset` 在 jvm 启动时使用 `file.encoding` 设置。
2. `file.encoding` 启动时不指定，使用 OS 的默认字符集。
3. 所有使用字符集的地方，不指定，使用的就是默认的字符集。
   - String.getBytes()
   - OutputStreamWritter() 构造方法
   - InputStreamReader() 构造方法

```java {fileName="Charset.java"}
public static Charset defaultCharset() {
        if (defaultCharset == null) {
            synchronized (Charset.class) {
                String csn = AccessController.doPrivileged(
                    new GetPropertyAction("file.encoding"));
                Charset cs = lookup(csn);
                if (cs != null)
                    defaultCharset = cs;
                else
                    defaultCharset = forName("UTF-8");
            }
        }
        return defaultCharset;
    }
```

## Unicode


1. unicode 规定了每个字符对应的码点。然而，这里所说的“字符” character 实际上并非完全等价于人们看起来的一个字或符号。对于部分语言文字、 emoji 符号，需要连续使用多个码点来表示一个不可分解的`文字元素`。
2.  unicode 一共分为 17 个 [plane](https://en.wikipedia.org/wiki/Code_point#In_character_encoding), 一个 plane 有 65536 (= 2^16) code points. 一共有 1114112 个 codepoint
3.  In Java, Strings are internally represented using UTF-16 , which means:
    -  `Basic Multilingual Plane (BMP)` characters (U+0000 to U+FFFF) are represented as `a single char` (16 bits).
    - `Supplementary Plane (SP)` characters (U+10000 to U+10FFFF, such as most emojis) are represented as a surrogate pair : `two char` values (high surrogate + low surrogate).

## 重新审视 java String API

### String(byte bytes[], Charset charset) 构造方法

```java
public String(byte bytes[], Charset charset) {
    this(bytes, 0, bytes.length, charset);
}
```

将 byte[] 数组用指定的字符集 charset 解码，最终 java 内部使用 `UTF-16` char[] 表示这个字符串

### string.length 
> 返回的 UTF-16 `Unicode code unit(2Bytes)` 的个数。
[测试代码](https://github.com/cloudedseal/java-examples/blob/main/snippet/src/main/java/charset/UnicodeTest.java)
1. 比如英文字母 [e(0x65)](https://symbl.cc/en/0065/) 位于 `BMP` 使用 1 个编码单元，长度为 1
2. 比如 [汉(0x6c49)](https://symbl.cc/en/6C49/) 位于 `BMP` 使用 1 个编码单元，长度为 1
3. 比如 [🤣(0x1f923)](https://symbl.cc/en/1F923-rolling-on-the-floor-laughing-rofl-emoji/)位于 `SP` 使用 2 个编码单元，长度为 2。称之为 `surrogate pairs`
4. 比如 `👨‍👩‍👧‍👦` 这个 `family-emoji` 符号(字素)，它包含有 7 个码点，分别是（十六进制） U+1F468 U+200D U+1F469 U+200D U+1F467 U+200D U+1F466。这五个码点连续出现时，就展示为这个 emoji 符号。这种人们看起来不应被分解的文字或符号整体，称为“字素” grapheme （在 unicode 中也被称为“扩展字素簇” extended grapheme cluster ）。长度为 11 = 2 + 1 + 2 + 1 + 2 + 1 + 2
   1. Code Point: [1F468](https://symbl.cc/en/1F468-man-emoji/) `man-emoji` 位于 `SP` 使用 2 个编码单元，长度为 2
   2. Code Point: [200D](https://symbl.cc/en/200D/) `Zero Width Joiner` 位于 `BMP` 使用 1 个编码单元，长度为 1 
   3. Code Point: [1F469](https://symbl.cc/en/1F469-woman-emoji/) `woman-emoji` 位于 `SP` 使用 2 个编码单元，长度为 2
   4. Code Point: [200D](https://symbl.cc/en/200D/) `Zero Width Joiner` 位于 `BMP` 使用 1 个编码单元，长度为 1 
   5. Code Point: [1F467](https://symbl.cc/en/1F467-girl-emoji/) `girl-emoji` 位于 `SP` 使用 2 个编码单元，长度为 2
   6. Code Point: [200D](https://symbl.cc/en/200D/) `Zero Width Joiner`位于 `BMP` 使用 1 个编码单元，长度为 1
   7. Code Point: [1F466](https://symbl.cc/en/1F466-boy-emoji/) `boy-emoji` 位于 `SP` 使用 2 个编码单元，长度为 2


## 📌 **Java APIs Affected by UTF-16 Internals**

| API | Behavior with UTF-16 |
|-----|----------------------|
| length() | Returns number of 16-bit `char`s (not Unicode code points) |
| charAt(int) | Returns a single `char` (may be part of a surrogate pair) |
| codePointAt(int) | Returns the full Unicode code point (handles surrogate pairs) |
| codePointCount(int, int) | Returns the number of Unicode code points in a range |
| substring(int, int) | May split surrogate pairs if not handled carefully |
| toCharArray() | Returns raw 16-bit `char`s (may include surrogate pairs) |
| String(char[]) | Constructs a `String` from 16-bit `char`s (must include full surrogate pairs) |
| getBytes(Charset) | Encodes the `String` to bytes in the specified charset (e.g., UTF-8, UTF-16) |
| new String(byte[], Charset) | Decodes bytes to a `String` using the specified charset |
| Pattern/Matcher | Regular expressions can match code points using `\X` or `Pattern.UNICODE_CHARACTER_CLASS` |
| Normalizer | Normalizes Unicode text (e.g., combining characters) |
| Character class | Provides utilities to handle surrogate pairs (e.g., `isHighSurrogate`, `isLowSurrogate`) |

## JNI String 类型数据怎么传更好?

1. 直接使用 String#getBytes(charset) 传指定字符集的字节数组更好。








## References

1. [unicode-table](https://symbl.cc/en/unicode-table/)
2. [unicode-table-CJK](https://symbl.cc/en/unicode/blocks/cjk-unified-ideographs/)
3. [code_unit 最少比特来表示一个编码单元](https://www.unicode.org/glossary/#code_unit)
4. [code_point-表中的坐标(一维、二维、三维...)](https://www.unicode.org/glossary/#code_point)
5. [grapheme-字素不完全等于字符](https://www.unicode.org/glossary/#grapheme)
6. [UTF-16](https://en.wikipedia.org/wiki/UTF-16)
7. [java 常量字符串](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.4.7)
8.  [unicode-编码解码](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/sun/nio/cs/)
9.  [UTF-8](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/sun/nio/cs/UTF_8.java)
10. [JNI-GetStringUTFChars(modified UTF-8 encoding)](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/functions.html#GetStringUTFChars)
11. [unicode-NULL](https://symbl.cc/cn/0000/)
12. [字体排印学的乱麻梳理](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU5MDExNDE5MA==&action=getalbum&album_id=3270895673529450496&subscene=159&subscene=126&scenenote=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU5MDExNDE5MA%3D%3D%26mid%3D2247483840%26idx%3D1%26sn%3Db3e8b10ceec3a68f37132511ab86199b%26chksm%3Dfcb044d993201ba9f9c86af082898b4d9fd186d1200596a144b4d582b013950db23c8d0ec6ba%26scene%3D126%26sessionid%3D1750901842%23rd&nolastread=1#wechat_redirect)
13. [string.length](https://hsivonen.fi/string-length/)