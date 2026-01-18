---
date: '2026-01-18T16:11:25+08:00'
draft: false
title: 'Master Maven Pom Dependency'
tags: ["maven","java","build-tool"]
---


## Maven `<dependency>` 

### Mental model (remember this)
---------------------------------

> `<dependency>` defines **what**, **when**, and **how far** a library is visible

*   **what** → groupId / artifactId / classifier
*   **when** → scope
*   **how far** → optional / exclusions

* * *

Maven `<dependency>` — All Children Explained
=============================================

Full form:

```xml
<dependency>
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>...</version>
  <type>...</type>
  <classifier>...</classifier>
  <scope>...</scope>
  <optional>...</optional>
  <exclusions>...</exclusions>
</dependency>
```

* * *

1️⃣ `<groupId>` (REQUIRED)
--------------------------

### What it is

The **organization / namespace** of the artifact.


```xml
<groupId>org.springframework</groupId>
```

### Rules

*   Usually a reversed domain name
*   Maps directly to directory path

```
org.springframework → org/springframework
```

* * *

2️⃣ `<artifactId>` (REQUIRED)
-----------------------------

### What it is

The **name of the artifact**.


```xml
<artifactId>spring-context</artifactId>
```

### Rules

*   Unique within the groupId
*   No version, no classifier here

* * *

3️⃣ `<version>` (CONDITIONALLY REQUIRED)
----------------------------------------

### What it is

The **exact version** of the dependency.

*   Exact artifact version
*   Can be omitted if managed elsewhere

```xml
<version>5.3.30</version>
```

### When it’s optional

If controlled by:

*   `dependencyManagement`
*   a BOM
*   parent POM

### Best practice

✔ Centralize versions  
❌ Hardcode everywhere

* * *

4️⃣ `<type>` (OPTIONAL)
-----------------------

### What it is

The **artifact packaging type**.

Default:

```
jar
```

### Common values

| Type | Meaning |
| --- | --- |
| `jar` | Default Java library |
| `pom` | BOM or parent |
| `war` | Web app |
| `ear` | Enterprise archive |
| `zip` | Distribution |

### Example (BOM import)

```xml
<type>pom</type>
```

* * *

5️⃣ `<classifier>` (OPTIONAL, ADVANCED)
---------------------------------------

### What it is

A **variant** of the same artifact.

Example:

```
guava-31.1-jre.jar
guava-31.1-android.jar
```

### Usage

```xml
<classifier>sources</classifier>
```

Common classifiers:

*   `sources`
*   `javadoc`
*   `tests`
*   `linux-x86_64`

⚠️ Use only if you know what you’re doing.

* * *

6️⃣ `<scope>` (VERY IMPORTANT)
------------------------------

### What it is

Controls **where and when** the dependency is used.

### All scopes explained

#### `compile` (DEFAULT)

*   Available everywhere
*   Included transitively

```xml
<scope>compile</scope>
```

#### `provided`

*   Needed to compile
*   Not packaged
*   Provided by runtime (e.g. servlet container)

```xml
<scope>provided</scope>
```

#### `runtime`

*   Not needed to compile
*   Needed at runtime

```xml
<scope>runtime</scope>
```

#### `test`

*   Only for tests
*   Not transitive

```xml
<scope>test</scope>
```

#### `system` (AVOID)

*   Hardcoded local path
*   Breaks portability

```xml
<scope>system</scope>
<systemPath>/lib/foo.jar</systemPath>
```

#### `import` (SPECIAL)

*   Only valid in `dependencyManagement`
*   Used for BOMs

```xml
<scope>import</scope>
```

* * *

7️⃣ `<optional>` (MISUNDERSTOOD)
--------------------------------

### What it is

Prevents this dependency from being **pulled transitively**.

```xml
<optional>true</optional>
```

### What it does NOT do

❌ Does not remove the dependency from your build  
❌ Does not make it runtime-optional

### When to use

Libraries that support **plug-ins** or **extensions**.

* * *

8️⃣ `<exclusions>` (CONFLICT CONTROL)
-------------------------------------

### What it is

Explicitly removes unwanted transitive dependencies.

```xml
<exclusions>
  <exclusion>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
  </exclusion>
</exclusions>
```

### Use cases

*   Resolve conflicts
*   Remove legacy libs
*   Replace logging frameworks

⚠️ Overuse leads to fragile builds.

* * *

9️⃣ Hidden but related elements
-------------------------------

### `<systemPath>` (ONLY with `system`)

```xml
<systemPath>${project.basedir}/lib/foo.jar</systemPath>
```

❌ Strongly discouraged.

* * *


