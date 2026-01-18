---
date: '2026-01-18T16:45:25+08:00'
draft: false
title: 'Master Maven Dependency Management'
tags: ["maven","java","build-tool"]
---

## Maven Dependency System  

1\. Core identity elements (WHAT the dependency is)
---------------------------------------------------

### `<groupId>`

*   Namespace / organization
*   Maps directly to directory path
*   Immutable identity component

Example:

```xml
<groupId>org.apache.commons</groupId>
```

* * *

### `<artifactId>`

*   Artifact name
*   Unique within a groupId

Example:

```xml
<artifactId>commons-lang3</artifactId>
```

* * *

### `<version>`

*   Exact artifact version
*   Can be omitted if managed elsewhere

Examples:

```xml
<version>3.12.0</version>
<version>1.0-SNAPSHOT</version>
```

Special forms:

*   SNAPSHOT → mutable
*   Ranges → discouraged (`[1.0,2.0)`)

* * *

### `<type>`

*   Packaging format
*   Default: `jar`

Used mainly for:

*   BOMs (`pom`)
*   non-JAR artifacts

* * *

### `<classifier>`

*   Variant of same artifact
*   Same coordinates, different purpose

Examples:

```
sources, javadoc, tests, linux-x86_64
```

* * *

2\. Visibility & lifecycle control (WHEN it’s used)
---------------------------------------------------

### `<scope>` (most critical concept)

| Scope | Compile | Runtime | Test | Transitive |
| --- | --- | --- | --- | --- |
| compile | ✔ | ✔ | ✔ | ✔ |
| provided | ✔ | ❌ | ✔ | ❌ |
| runtime | ❌ | ✔ | ✔ | ✔ |
| test | ❌ | ❌ | ✔ | ❌ |
| system | ✔ | ✔ | ✔ | ❌ |
| import | ❌ | ❌ | ❌ | ❌ |

Key rules:

*   `provided` = container provides it
*   `test` never leaks
*   `import` only for BOMs

* * *

3\. Dependency propagation control (HOW FAR it spreads)
-------------------------------------------------------

### `<optional>`

*   Prevents transitive propagation
*   Still present locally

Example:

```xml
<optional>true</optional>
```

Used by:

*   plugin-style libraries
*   integrations

* * *

### `<exclusions>`

*   Removes specific transitive dependencies

Example:

```xml
<exclusion>
  <groupId>commons-logging</groupId>
  <artifactId>commons-logging</artifactId>
</exclusion>
```

⚠️ Exclusions are surgical tools — not band-aids.

* * *

4\. Management layer (WHO decides versions)
-------------------------------------------

### `<dependencyManagement>`

*   Central version control
*   Does NOT add dependencies

Example:

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

* * *

### BOMs (Bill of Materials)

Imported using:

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-framework-bom</artifactId>
  <version>5.3.30</version>
  <type>pom</type>
  <scope>import</scope>
</dependency>
```

Purpose:

*   Lock compatible versions
*   Prevent conflicts

* * *

5\. Resolution mechanics (HOW Maven chooses)
--------------------------------------------

### Conflict resolution

*   Nearest dependency wins
*   First declared wins if equal depth

Maven **never fails** on conflicts — it silently chooses.

* * *

### Dependency mediation

Occurs when:

*   Same artifact appears multiple times
*   Different versions requested

Controlled by:

*   `dependencyManagement`
*   exclusions
*   ordering

* * *

6\. Repositories & metadata (WHERE it comes from)
-------------------------------------------------

### Artifact files

*   `.jar`
*   `.pom`
*   `.war`
*   checksums

### Metadata

*   `maven-metadata.xml`
*   Snapshot timestamp resolution

Metadata rules differ between:

*   snapshot repos
*   release repos

* * *

7\. Special cases & advanced behaviors
--------------------------------------

### Version ranges (avoid)

```xml
<version>[1.0,2.0)</version>
```

*   Non-reproducible
*   Break CI

* * *

### Optional + provided combo

Used for:

*   APIs (SPI)
*   container integrations

* * *

### System scope

Hardcoded path:

```xml
<systemPath>/lib/foo.jar</systemPath>
```

❌ Never use in real projects.

* * *

8\. Dependency inheritance
--------------------------

Children inherit:

*   dependencyManagement
*   dependencies
*   exclusions

Unless overridden.

* * *

9\. Dependency vs Plugin dependency (often confused)
----------------------------------------------------

| Project Dependency | Plugin Dependency |
| --- | --- |
| On app classpath | On plugin classpath |
| Affects runtime | Affects build only |

Plugins have their own `<dependencies>` block.

* * *

10\. Practical debugging tools (must know)
------------------------------------------

```bash
mvn dependency:tree
mvn dependency:analyze
mvn help:effective-pom
mvn -X
```

These show **what Maven actually resolved**.

* * *

11\. Mental model (final)
-------------------------

> Maven dependency resolution answers 5 questions:

1.  What artifact?
2.  Which version?
3.  When is it needed?
4.  How far does it propagate?
5.  Who decides the version?

If you can answer those, Maven becomes predictable.

* * *

12\. Professional rules to live by
----------------------------------

✅ Centralize versions  
✅ Prefer BOMs  
✅ Inspect dependency trees  
❌ Upload local cache files  
❌ Override randomly  
❌ Ignore conflicts

* * *

