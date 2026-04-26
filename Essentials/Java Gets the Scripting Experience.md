# JBang: Finally, Java Gets the Scripting Experience It Deserves

There are many times, at when you need to create some scripts. From Bash scripts for dealing with terminal environment, or Creating a one time environment setup to importing/exporting from Database, S3, and so on.


For these needs, most developers resort to programming languages they are mostly sufficient with. If you are a Python or Javascript developer you are in luck. When a Python developer wants to make an HTTP request and parse JSON, they open a file, type 4 lines of code.

```python
import urllib.request
import json

with urllib.request.urlopen("https://api.example.com/data") as response:
    data = json.load(response)
```

Even when you require dependencies, scripting languages support native package manager where you can install dependencies easily. `pip` or `uv` for Python. `npm` for Javascript projects.

Compare this to Java.

Since Java 11, under JEP 330, you CAN run a single `.java` file directly using `java HelloWorld.java` without need to compile to `.class` file. However, the situation changes when you require dependencies.

I do encourage software engineers to be Polyglot programmers. With AI, creating scripts in other languages than your primary programming language have become so much easier. For many of scripting needs, I prefer Python.

---

### The Problem: Java’s Scaffolding Obsession

Let’s say you want to write a quick script that reads a JSON file using Jackson. Here’s what you traditionally need:

#### The Maven Way
First, you create the project structure:

```text
my-script/
├── pom.xml
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── JsonReader.java
```

**pom.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>json-reader</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.17.0</version>
        </dependency>
    </dependencies>
</project>
```

And **then** the actual code buried four directories deep:

```java
package com.example;

import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.File;
import java.util.Map;

public class JsonReader {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        Map data = mapper.readValue(new File("data.json"), Map.class);
        System.out.println(data);
    }
}
```

Running it?
`mvn compile exec:java -Dexec.mainClass="com.example.JsonReader"`

Lovely.

#### The Gradle Way
Slightly better, but still a multi-file affair:

```groovy
// build.gradle
plugins {
    id 'application'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.17.0'
}

application {
    mainClass = 'JsonReader'
}
```

Plus the same directory structure, the same class file. You run it with `gradle run`. Gradle will spend a few seconds thinking about its own existence before actually running your six lines of meaningful code.

---

### Enter JBang: One File to Rule Them All



```java
//DEPS com.fasterxml.jackson.core:jackson-databind:2.17.0

import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.File;
import java.util.Map;

public class JsonReader {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        Map data = mapper.readValue(new File("data.json"), Map.class);
        System.out.println(data);
    }
}
```

That’s it. One file. Run it with: `jbang JsonReader.java`

The `//DEPS` directive automatically installs the dependencies, adds them to the classpath, and runs your code. Magic.

---

### Comparing to Previously Available Groovy Scripts

JDK did have a lesser known feature that supported exactly this in Groovy.

#### Groovy Scripts
Apache Groovy has long been Java’s scripting escape valve. It offers a more concise syntax and genuine scripting capabilities with `@Grab` for dependency management:

```groovy
@Grab('com.fasterxml.jackson.core:jackson-databind:2.17.0')
import com.fasterxml.jackson.databind.ObjectMapper

def mapper = new ObjectMapper()
def data = mapper.readValue(new File("data.json"), Map)
println data
```

Run with `groovy script.groovy`. No project structure needed.

Groovy is excellent for scripting, and its `@Grab` annotation was a genuine innovation. But there are trade-offs compared to JBang. The biggest drawback is the unpopularity of the Groovy language itself.

Groovy started to get popular around 2008 which predates even Java 8. Much of its strength became native Java features most notably with Java 8 and lambda. Kotlin also took on many of its strength and there were not much use cases for using Groovy anymore. **EXCEPT FOR SCRIPTING** until now with JBang.

---

### JBang Directives

JBang has many features. Most of them through the directives (set of special comments at the start of the java class) We already covered `//DEPS` for installing dependencies. Let’s cover some of my favorites.

#### Direct Executing through SheBang
If you start your file with the following directive:
`///usr/bin/env jbang “$0” “$@” ; exit $?`
Then, you can execute the file without requiring to prefix it with `jbang` cli command. In other words, you can simply call `./JsonReader.java` as long as you make the file executable with `chmod`.

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//DEPS com.fasterxml.jackson.core:jackson-databind:2.17.0

import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.File;
import java.util.Map;

public class JsonReader {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        Map data = mapper.readValue(new File("data.json"), Map.class);
        System.out.println(data);
    }
}
```

#### Setting Java Version
```java
// Exact version
//JAVA 17

// Minimum version
//JAVA 11+

// For preview features
//JAVA 25+
//PREVIEW
```

#### Runtime Memory/GC Setting
```java
//RUNTIME_OPTIONS -Xmx4g -Xms1g -XX:+UseG1GC
```

#### Language-Specific Directives
You can even use other JVM languages. Not just Java.

**Using Kotlin**
```kotlin
///usr/bin/env jbang "$0" "$@" ; exit $?
//KOTLIN 2.0.21
//DEPS org.jetbrains.kotlin:kotlin-stdlib:2.0.21

fun main(args: Array<String>) {
    println("Hello from Kotlin ${args.firstOrNull() ?: "World"}")
}
```

**Using Groovy**
```groovy
///usr/bin/env jbang "$0" "$@" ; exit $?
//GROOVY 3.0.19
//DEPS org.codehaus.groovy:groovy:3.0.19

def name = args.length > 0 ? args[0] : "World"
println "Hello from Groovy $name"
```

---

### JBang Templates
JBang ships with built-in templates. Want a proper CLI with argument parsing?

`jbang init — template=cli mycli.java`

This generates a file pre-wired with **Picocli**. But you can write one from scratch too:

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//DEPS info.picocli:picocli:4.7.5

import picocli.CommandLine;
import picocli.CommandLine.Command;
import picocli.CommandLine.Option;
import picocli.CommandLine.Parameters;
import java.io.File;
import java.nio.file.Files;

@Command(name = "linecount", mixinStandardHelpOptions = true,
         description = "Counts lines in files")
public class linecount implements Runnable {

    @Parameters(description = "Files to count lines in")
    File[] files;

    @Option(names = {"-v", "--verbose"}, description = "Show per-file counts")
    boolean verbose;

    @Override
    public void run() {
        long total = 0;
        for (File f : files) {
            try {
                long count = Files.lines(f.toPath()).count();
                total += count;
                if (verbose) {
                    System.out.printf("%6d %s%n", count, f.getName());
                }
            } catch (Exception e) {
                System.err.println("Error reading " + f + ": " + e.getMessage());
            }
        }
        System.out.printf("%6d total%n", total);
    }

    public static void main(String[] args) {
        int exitCode = new CommandLine(new linecount()).execute(args);
        System.exit(exitCode);
    }
}
```
See template section of the [doc](https://www.jbang.dev/documentation/jbang/latest/templates.html) for more.

---

### Wrap Up
For years, the unspoken rule was: if you need a quick script, don’t use Java. JBang changes that. It doesn’t change the language — it removes the ceremony around it. Same Java, same libraries, zero scaffolding.

Think about all the throwaway scripts in your career — the one-time database migration, the CSV cleanup, the log parser at 2 AM. They lived in bash or Python because Java’s startup cost was too high for something you’ll run three times and delete. With JBang, that cost is zero.

JBang won’t replace Python or your build tool. But for the script that’s too complex for bash and too throwaway for a Maven project — it’s exactly right.