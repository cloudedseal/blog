---
date: '2026-01-18T12:43:04+08:00'
draft: false
title: 'Master Maven'
tags: ["maven","java","build-tool"]
---

1\. What Maven actually is (mental model)？
--------------------------------------------------

Maven is **not**:

*   just a build tool
*   just dependency download

Maven **is**:

> A **standardized build lifecycle + dependency resolver + repository protocol**

Everything in Maven revolves around:

*   **Coordinates** 坐标
*   **Lifecycle** 生命周期  
*   **Repositories**  仓库
*   **Metadata**    元数据


2\. Maven coordinates (the identity of everything)
--------------------------------------------------

Every artifact is uniquely identified by:

```
groupId:artifactId:version[:classifier][@packaging]
```

Example:

```
org.apache.commons:commons-lang3:3.12.0
```

### How coordinates map to paths (CRITICAL)

```
groupId      → path
artifactId   → directory
version      → directory
```

Example:

```
org.apache.commons
commons-lang3
3.12.0
```

Becomes:

```
org/apache/commons/commons-lang3/3.12.0/
```

File names:

```
commons-lang3-3.12.0.jar
commons-lang3-3.12.0.pom
```

This mapping is **non-negotiable**.  这种映射是不可协商的。

* * *

3\. Maven repository layout (Maven 2 layout)
--------------------------------------------

A **valid Maven repository** is simply a directory tree with strict rules.

Allowed files:

*   `.jar`, `.pom`, `.war`, `.aar`
*   `.sha1`, `.md5`
*   `maven-metadata.xml` (repo-generated)

❌ Forbidden files:

*   `_remote.repositories`
*   `.lastUpdated`
*   random text
*   local cache metadata

* * *

4\. Types of Maven repositories
-------------------------------

### Local repository

```
~/.m2/repository
```

Contains:

*   artifacts
*   **local-only metadata** (DO NOT upload)

### Remote repositories

*   Maven Central
*   Nexus
*   Artifactory

### Hosted repository types

| Type | Purpose |
| --- | --- |
| **Release** | Immutable versions |
| **Snapshot** | Mutable versions |
| **Proxy** | Cache of remote |
| **Group** | Aggregation |

* * *

5\. SNAPSHOT vs RELEASE (very important)
----------------------------------------

### Release

```
1.0
2.3.4
```

Rules:

*   Immutable
*   No overwrite
*   No snapshot metadata

### Snapshot

```
1.0-SNAPSHOT
```

Resolved internally as:

```
1.0-20240108.123456-3
```

Rules:

*   Mutable
*   Overwrite allowed
*   Requires metadata

Uploading snapshot artifacts to a **release repo** → ❌ error

* * *

6\. The POM (Project Object Model)
----------------------------------

The `pom.xml` is the **single source of truth**.

Minimal POM:

```xml
<project>
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>demo</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>
</project>
```

### What POM controls

*   Dependencies
*   Plugins
*   Repositories
*   Build lifecycle
*   Packaging
*   Inheritance

* * *

7\. Dependency resolution (how Maven thinks)
--------------------------------------------

When Maven sees a dependency:

```xml
<dependency>
  <groupId>aopalliance</groupId>
  <artifactId>aopalliance</artifactId>
  <version>1.0</version>
</dependency>
```

Maven:

1.  Checks local repo
2.  Checks remote repos (in order)
3.  Downloads:
    *   `.pom`
    *   `.jar`
    *   metadata

It **never uploads cache files**. That’s your job when migrating.

* * *

8\. Transitive dependencies
---------------------------

Maven resolves dependencies **recursively**.

If:

```
A → B → C
```

You get **A + B + C** automatically.

### Dependency conflicts

Resolved by:

*   nearest-definition wins
*   dependencyManagement overrides

Example:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
      <version>1.2</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

* * *

9\. Build lifecycle (this is huge)
----------------------------------

Maven doesn’t run “commands”, it runs **phases**.

### Default lifecycle

```
validate
compile
test
package
verify
install
deploy
```

Common usage:

```bash
mvn clean package
mvn clean install
mvn deploy
```

### What `install` does

*   Builds artifact
*   Installs into **local repo**

### What `deploy` does

*   Uploads to **remote repo**

* * *

10\. Plugins (how Maven actually does work)
-------------------------------------------

Maven itself does very little.

Everything is a **plugin**.

Examples:

*   `maven-compiler-plugin`
*   `maven-surefire-plugin`
*   `maven-deploy-plugin`

Plugins bind goals to lifecycle phases.

Example:

```xml
<plugin>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.11.0</version>
</plugin>
```

* * *

11\. Settings.xml (enterprise Maven)
------------------------------------

Located at:

```
~/.m2/settings.xml
```

Controls:

*   credentials
*   mirrors
*   proxies
*   profiles

### Credentials (important)

```xml
<servers>
  <server>
    <id>nexus</id>
    <username>user</username>
    <password>pass</password>
  </server>
</servers>
```

Referenced in POM:

```xml
<repository>
  <id>nexus</id>
  <url>https://repo</url>
</repository>
```

* * *

12\. Repository migration (what you’re doing)
---------------------------------------------

### Correct approach

*   Upload **only**:
    *   `.jar`
    *   `.pom`
    *   checksums
*   Preserve layout
*   Use repo root URL
*   Match snapshot/release rules

### Wrong approach

*   Upload `.m2` blindly
*   Upload `_remote.repositories`
*   Upload local metadata

You already learned this the hard way — good.

* * *

13\. Common Maven mistakes (avoid these)
----------------------------------------

❌ Editing files in `.m2`  
❌ Uploading cache metadata  
❌ Overwriting releases  
❌ Mixing snapshot + release repos  
❌ Putting paths inside repo URL

* * *

14\. Debugging Maven like a pro
-------------------------------

```bash
mvn -X clean install     # full debug
mvn dependency:tree
mvn help:effective-pom
```

These commands explain **everything Maven is doing**.


