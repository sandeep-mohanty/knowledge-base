# Mastering Java Garbage Collection: A Complete Guide from Beginner to Expert

Garbage Collection (GC) in Java is an automatic process that helps manage memory. It works by finding and removing objects in the heap memory that are no longer being used by the program. This frees up space so new objects can be created. Since garbage collection runs automatically inside the Java Virtual Machine (JVM), programmers don’t need to manually delete unused objects, which makes memory management easier and less error-prone.

---

### What is Garbage Collection?
Garbage Collection (GC) is like cleaning your house. Just as we collect and throw away trash to keep our home neat, computer programs also need to clean up unused data from memory. If we never collect garbage at home, it will pile up and make living difficult. Similarly, if a program never cleans up unused variables and objects (garbage), they will keep taking up memory space and slow down the system.

In programming, garbage refers to variables or objects that are no longer needed by the program. The Garbage Collector (GC) in languages like Java automatically finds and removes this unused memory, so the system can stay fast and efficient.

---

### What is OutOfMemoryError?
`OutOfMemoryError` happens when a program tries to use more memory than the computer (or JVM in Java) can give it. Think of it like trying to pour 2 liters of water into a 1‑liter bottle — it just won’t fit.

This usually occurs when:
* A program keeps creating new objects but there isn’t enough space left.
* Memory is used but never freed (called a **memory leak**).
* The program is working with very large files, images, videos, or huge amounts of data.



When this error happens, the program often crashes or stops working. 

**To fix it, you can:**
1.  Give the program more memory (by changing JVM settings).
2.  Make the program use memory more wisely (optimize the code).
3.  Use tools like memory profilers to find memory leaks.

---

### How does garbage collection work in Java?
In Java, all objects are created on a special memory area called the **heap**. When a program no longer needs an object (no variables reference it anymore), it becomes garbage and can be cleaned up.

The Garbage Collector (GC) is a background process in Java that automatically finds and removes unused objects to free memory. This way, programmers don’t have to manage memory manually.

**The garbage collection process usually has three main steps:**
1.  **Marking:** The GC starts from important objects (roots), like local variables and method parameters, and marks all objects that are still reachable.
2.  **Sweeping:** It removes (reclaims memory from) objects that are no longer reachable.
3.  **Compacting:** To avoid memory gaps (fragmentation), it sometimes moves the remaining objects closer together and frees up continuous space.



Because the JVM takes care of garbage collection automatically, Java programs are less likely to face memory issues compared to languages where developers must free memory themselves.

---

### Types of garbage collectors in Java
In Java, different types of garbage collection (GC) handle cleaning memory at different stages. The two main types are:

**1. Minor (or Incremental) Garbage Collection**
* Cleans the **young generation** part of the heap, where most new objects are created.
* Since most of these objects are short‑lived, they are quickly removed.
* Happens frequently but is usually very fast.

**2. Major (or Full) Garbage Collection**
* Cleans the **old generation** part of the heap, where long‑lived objects are stored.
* Runs less often but takes more time, since it deals with bigger, long‑lasting data.
* When this happens, it can pause the program for a while.



So, Minor GC quickly clears out short‑lived “trash,” while Major/Full GC deals with the bigger, long‑term memory cleanup.

---

### Benefits of Java Garbage Collection
* **No manual memory management:** Developers don’t have to write extra code to free memory. The JVM handles it automatically.
* **Prevents memory leaks:** Unused objects are removed, so memory isn’t wasted by data that is no longer needed.
* **Dynamic memory allocation:** Objects are created on the heap as needed, and GC ensures unused memory is recycled for new objects.
* **Better performance:** By keeping memory clean, programs run more smoothly without slowing down over time.
* **Memory optimization:** GC rearranges and frees up memory space so it can be used efficiently.

---

### Events That Trigger Java Garbage Collection
In Java, garbage collection (GC) usually happens automatically when the JVM decides memory needs to be cleaned. Some common triggers are:

* **Heap is getting full:** When you create a new object and there isn’t enough free space in the heap, the JVM runs GC to clear unused objects.
* **System.gc() call:** You can request GC by calling `System.gc()`, but it’s only a suggestion—the JVM decides whether to run it or not.
* **Old Generation filling up:** If the old generation (where long‑lived objects are stored) gets close to full, GC is triggered.
* **PermGen/Metaspace filling up:** In Java 7 and earlier, GC runs when PermGen is nearly full. From Java 8 onward, this applies to Metaspace.
* **Time‑based triggers:** In some cases, the JVM may run GC after a certain time period, even if there’s still free space, just to keep memory healthy.

---

### Requesting JVM to Run Garbage Collector
In Java, you can **ask** the JVM to run the Garbage Collector, but the JVM decides whether to run it immediately or not.

**The main ways are:**
1.  **System.gc():** Sends a request to the JVM to run garbage collection. It’s not guaranteed to run right away.
2.  **Runtime.getRuntime().gc():** Another way to request GC, works similarly to `System.gc()`.
3.  **JVM Flag (-XX:+DisableExplicitGC):** If this option is enabled, calling `System.gc()` or `Runtime.gc()` will be ignored. This is sometimes used in production to prevent unnecessary GC calls.

---

### What garbage collectors are available for Java?
Java provides different types of Garbage Collectors (GC), each with its own way of cleaning memory:

* **Serial Garbage Collector:** Uses a single thread to perform garbage collection. Simple and good for small applications running on single‑core machines.
    ```bash
    java -XX:+UseSerialGC -jar application.jar
    ```
* **Parallel Garbage Collector:** Uses multiple threads to speed up garbage collection. Best for applications that run on multi‑core processors.
* **Concurrent Mark Sweep (CMS) Collector:** Tries to minimize the pause time of applications by running most of the garbage collection work alongside (concurrently with) the program.
    ```bash
    java -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -jar application.jar
    ```
* **G1 (Garbage First) Collector:** Splits the heap into regions and cleans them efficiently. Works well for large heap memory and is the default collector in newer Java versions.
    ```bash
    java -XX:+UseG1GC -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:gc.log -jar application.jar
    ```



---

### How Can an Object Be Unreferenced?
An object becomes unreferenced in Java when no part of the program holds a reference (or pointer) to it anymore. This means there is no way to access that object through any variable or data structure. When this happens, the object is considered “dead” and becomes eligible for garbage collection.

**Here are some common ways an object can be unreferenced:**
1.  **Assigning null to a reference:**
    ```java
    Student student = new Student();
    student = null;
    ```
    Now, the Student object created is unreferenced because `student` no longer points to it.
2.  **Reassigning one reference to another:**
    ```java
    Student studentOne = new Student();
    Student studentTwo = new Student();
    studentOne = studentTwo; 
    ```
    The original object `studentOne` referred to has no references left and is unreferenced.
3.  **Objects created anonymously without storing reference:**
    ```java
    register(new Student());
    ```
    Since no variable stores the new `Student` object, it’s immediately eligible for garbage collection if not used.

Once an object is unreferenced, the Garbage Collector can reclaim its memory automatically since it knows the object is no longer needed by the program.

---

In this article, we have seen why Java needs a Garbage Collector to automatically clean unused memory. We also learned how the JVM runs garbage collection on its own and how we can ask it explicitly to run using methods like `System.gc()`. However, explicitly calling the garbage collector is usually bad for application performance and can cause unnecessary delays. Therefore, it’s best to let the JVM decide when to perform garbage collection instead of forcing it manually.
