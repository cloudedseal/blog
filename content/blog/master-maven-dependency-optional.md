---
date: '2026-01-18T16:53:14+08:00'
draft: false
title: 'Master Maven Dependency Optional'
tags: ["maven","java","build-tool"]
---

## optional dependency 是啥？

Maven `<optional>` 标签用于**控制依赖的传递性**。

> **`<optional>true</optional>` means:  
> “Do NOT pass this dependency to projects that depend on me.”**

That’s it.  
Nothing more. Nothing less.

It is **NOT** about runtime optionality.

* * *

The mental model (very important)
---------------------------------

Think of Maven dependencies as **edges in a graph**.

`optional=true` means:

```
Consumer ←X— Optional Dependency
```

The arrow **stops** at the module boundary.

* * *

What `optional` does NOT do (CRITICAL)
--------------------------------------

❌ Does NOT remove the dependency  
❌ Does NOT make it runtime-optional  
❌ Does NOT disable compilation  
❌ Does NOT affect your own project

If your project declares an optional dependency:

*   **You still compile with it**
*   **You still run with it**
*   **You still package with it**

Only _downstream projects_ are affected.

* * *

Example 1: WITHOUT optional (default behavior)
----------------------------------------------

### Library A (a framework)

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.33</version>
</dependency>
```

### Application B (depends on A)

Dependency graph:

```
App B
 └─ Library A
     └─ mysql-connector-java
```

➡ MySQL driver is **forced onto everyone**, even if they use PostgreSQL.

❌ Bad library design.

* * *

Example 2: WITH optional (correct design)
-----------------------------------------

### Library A

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.33</version>
  <optional>true</optional>
</dependency>
```

### Dependency graph now

```
App B
 └─ Library A
```

➡ MySQL is **NOT inherited**.

If App B wants MySQL:

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
```

This gives **choice** to the consumer.

* * *

Example 3: Optional + runtime failure (important!)
--------------------------------------------------

### Library A

```xml
<dependency>
  <groupId>org.postgresql</groupId>
  <artifactId>postgresql</artifactId>
  <optional>true</optional>
</dependency>
```

### App B

```java
DriverManager.getConnection("jdbc:postgresql://...");
```

But App B **did not declare** PostgreSQL explicitly.

➡ Result:

```
ClassNotFoundException: org.postgresql.Driver
```

This is expected.

`optional` means **“I might use this, you decide”**.

* * *

When should you use `<optional>`?
---------------------------------

### ✔ Correct use cases

1.  **SPI / plugin systems**
2.  **Multiple backend choices**
3.  **Framework integrations**
4.  **Adapters**
5.  **Logging bridges**

Examples:

*   Database drivers
*   Messaging clients
*   Cloud provider SDKs
*   Logging implementations

* * *

When you should NOT use it
--------------------------

❌ Applications  
❌ Internal modules  
❌ Libraries with hard requirements

If your code **cannot run without it**, it should NOT be optional.

* * *

Optional vs Provided (common confusion)
---------------------------------------

| Feature | optional | provided |
| --- | --- | --- |
| Used at compile | ✔ | ✔ |
| Included transitively | ❌ | ❌ |
| Included in runtime | ✔ | ❌ |
| Provided by container | ❌ | ✔ |

**Optional = consumer choice**  
**Provided = runtime environment supplies it**

* * *

Optional vs Exclusion (very different)
--------------------------------------

| Concept | optional | exclusion |
| --- | --- | --- |
| Direction | Outbound | Inbound |
| Scope | Library → consumer | Consumer → library |
| Who controls it | Producer | Consumer |

* * *

Real-world example: SLF4J
-------------------------

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
</dependency>

<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <optional>true</optional>
</dependency>
```

Why?

*   API is required
*   Implementation is optional
*   Consumer chooses logback / log4j / etc.

This is **excellent library design**.

* * *

How to SEE optional behavior
----------------------------

Run in consumer project:

```bash
mvn dependency:tree
```

You will see:

```
(logback-classic omitted - optional)
```

This is Maven explicitly telling you what happened.

* * *

Key rule to remember (burn this in)
-----------------------------------

> **`optional` only affects transitive dependency propagation.**

If you remember only one thing, remember that.

* * *

Summary (plain English)
-----------------------

*   `optional` ≠ optional at runtime
*   `optional` = “don’t force this on my users”
*   Libraries use it, applications almost never
*   Consumers must opt-in explicitly

* * *


