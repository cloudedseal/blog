---
date: '2026-01-18T14:06:16+08:00'
draft: false
title: 'Master Maven Resolve Dependency Conflicts'
tags: ["maven","java","build-tool"]
---


## Dependency conflicts
Dependency conflicts are one of the **most important things to understand in Maven**, because they cause:

*   random runtime bugs
*   `NoSuchMethodError`
*   `ClassNotFoundException`
*   “works on my machine” builds


* * *

Maven Dependency Conflicts — Explained by Example

1️⃣ What is a dependency conflict?
----------------------------------

A conflict happens when **two (or more) versions of the same dependency** appear in the dependency graph.

Example:

```
YourApp
 ├─ LibraryA → commons-logging:1.1
 └─ LibraryB → commons-logging:1.2
```

Maven must choose **one version**.

* * *

2️⃣ Maven’s conflict resolution rule (CRITICAL)
-----------------------------------------------

> **Nearest definition wins**  

“Nearest” = **fewest `hops` from your project** 近水楼台先得月

If distance is equal:

*   the **first declared dependency wins** 先到先得

* * *

3️⃣ Simple conflict example
---------------------------

### pom.xml

```xml
  <dependencies>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-configuration2</artifactId>
      <version>2.13.0</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
      <version>1.1</version>
    </dependency>
  </dependencies>
```

### Dependency graph

#### 项目只有 commons-configuration2 的依赖

```
[INFO] com.abc:maven-tutorial:jar:1.0-SNAPSHOT #第一层
[INFO] \- org.apache.commons:commons-configuration2:jar:2.13.0:compile # 第二层
[INFO]    +- org.apache.commons:commons-lang3:jar:3.20.0:compile
[INFO]    +- org.apache.commons:commons-text:jar:1.14.0:compile
[INFO]    \- commons-logging:commons-logging:jar:1.3.5:compile # 距离为 2
```
#### 项目有 commons-configuration2 + 直接依赖 commons-logging:1.1

```
[INFO] --- dependency:3.7.0:tree (default-cli) @ maven-tutorial ---
[INFO] com.abc:maven-tutorial:jar:1.0-SNAPSHOT
[INFO] +- org.apache.commons:commons-configuration2:jar:2.13.0:compile
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.20.0:compile
[INFO] |  \- org.apache.commons:commons-text:jar:1.14.0:compile
[INFO] \- commons-logging:commons-logging:jar:1.1:compile # 最近的距离获胜
[INFO]    +- log4j:log4j:jar:1.2.12:compile
[INFO]    +- logkit:logkit:jar:1.0.1:compile
[INFO]    +- avalon-framework:avalon-framework:jar:4.1.3:compile
[INFO]    \- javax.servlet:servlet-api:jar:2.3:compile

```

#### commons-logging 最终被选中的版本

Maven picks:

```
commons-logging:1.1
```


4️⃣ Conflict between transitive dependencies
--------------------------------------------

### pom.xml

```xml
<dependencies>
  <dependency>
    <groupId>libA</groupId>
    <artifactId>a</artifactId>
    <version>1.0</version>
  </dependency>

  <dependency>
    <groupId>libB</groupId>
    <artifactId>b</artifactId>
    <version>1.0</version>
  </dependency>
</dependencies>
```

### Transitive graph

```
YourApp
 ├─ libA → guava:18.0
 └─ libB → guava:31.1
```

### Resolution

*   Both are distance = 2
*   First declared wins

If `libA` is first:

```
guava:18.0  ← selected
```

⚠️ You may not even realize this unless you check.

* * *

5️⃣ 如何解决依赖冲突?
------------------------------------------

Run:

```bash
mvn dependency:tree
```

With conflicts highlighted:

```bash
mvn dependency:tree -Dverbose
```

Example output:

```
[INFO] com.abc:maven-tutorial:jar:1.0-SNAPSHOT
[INFO] +- org.apache.commons:commons-configuration2:jar:2.13.0:compile
[INFO] |  +- org.apache.commons:commons-lang3:jar:3.20.0:compile
[INFO] |  +- org.apache.commons:commons-text:jar:1.14.0:compile
[INFO] |  |  \- (org.apache.commons:commons-lang3:jar:3.18.0:compile - omitted for conflict with 3.20.0)
[INFO] |  \- (commons-logging:commons-logging:jar:1.3.5:compile - omitted for conflict with 1.1)
[INFO] \- commons-logging:commons-logging:jar:1.1:compile
[INFO]    +- log4j:log4j:jar:1.2.12:compile
[INFO]    +- logkit:logkit:jar:1.0.1:compile
[INFO]    +- avalon-framework:avalon-framework:jar:4.1.3:compile
[INFO]    \- javax.servlet:servlet-api:jar:2.3:compile

```

That line tells you **exactly what was dropped**.

* * *

6️⃣ 解决依赖冲突推荐方法: `dependencyManagement`
----------------------------------------------------------

This is the **gold standard**.

### Example

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.google.guava</groupId>
      <artifactId>guava</artifactId>
      <version>31.1-jre</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

### What this does

*   Forces **all guava dependencies** to use `31.1`
*   No matter who pulls it in
*   Without adding a direct dependency

This is how large projects stay sane.

* * *

7️⃣ 解决依赖冲突可选方法: Exclusions (use sparingly)
-----------------------------------------------

```xml
<dependency>
  <groupId>libA</groupId>
  <artifactId>a</artifactId>
  <version>1.0</version>
  <exclusions>
    <exclusion>
      <groupId>com.google.guava</groupId>
      <artifactId>guava</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

Now only libB’s Guava is used.

⚠️ Dangerous if libA truly needs Guava.

* * *

8️⃣ BOMs (Bill of Materials) — professional solution
----------------------------------------------------

Large frameworks publish **BOMs**.

Example: Spring BOM

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-framework-bom</artifactId>
      <version>5.3.30</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

Now:

*   All Spring libs
*   All tested versions
*   No conflicts

This is how **enterprise Maven** works.

* * *


9️⃣ How experts debug dependency hell?
--------------------------------------

```bash
mvn dependency:tree -Dincludes=groupId:artifactId
mvn dependency:analyze
mvn help:effective-pom
```

These commands reveal **the truth Maven is using**, not what you think it’s using.


