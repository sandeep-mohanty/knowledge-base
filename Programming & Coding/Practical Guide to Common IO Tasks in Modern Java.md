# A Practical Guide to Common I/O Tasks in Modern Java

Working with files, streams, and data is a common task for every Java developer — especially in web applications. In this article, we’ll explore the most frequent I/O tasks you’ll encounter in real-world projects, such as:
* Reading and writing text files
* Fetching text, images, or JSON from the web
* Traversing files in a directory
* Reading ZIP files
* Creating temporary files and folders

Java’s I/O APIs have evolved significantly since Java 8. Modern Java provides cleaner, safer, and more convenient ways to handle these tasks. In particular:
* **UTF-8** became the default charset starting with Java 18 (JEP 400).
* The `java.nio.file.Files` class, introduced in Java 7, gained many useful methods in later versions (Java 8, 11, and 12).
* `InputStream` received several enhancements in Java 9, 11, and 12.
* Older classes like `File` and `BufferedReader` are now largely outdated, though you may still see them in older tutorials.

This article focuses on modern, recommended APIs so you can write efficient and future-proof Java code.

---

## Reading Text Files in Java

### 1. Reading the whole file at once
The easiest way to read a text file in modern Java is:

```java
import java.nio.file.*;

var path = Path.of("/usr/sunil/textFile/words");
String content = Files.readString(path);
```
* `Path.of(…)` creates a path to the file.
* `Files.readString(path)` reads the whole file as one big string.

### 2. About Character Encodings
Before Java 18, you had to tell Java which character set to use (usually UTF-8):
```java
String content = Files.readString(path, StandardCharsets.UTF_8);
```
From Java 18 onward, UTF-8 is the default. That means you can just write `Files.readString(path)` — it’s simpler and safer!

### 3. Reading line by line
If you want each line separately:
```java
List<String> lines = Files.readAllLines(path);
```
This gives you a list of strings — one for each line. If the file is large and you don’t want to load everything into memory, use a stream:
```java
try (Stream<String> lines = Files.lines(path)) {
    lines.forEach(System.out::println);
}
```
Always use a **try-with-resources** block so the file closes automatically.

### 4. Reading words or numbers
To read words (not just lines), you can use a `Scanner`:
```java
Scanner scanner = new Scanner(Path.of("file.txt"));
scanner.useDelimiter("\\PL+"); // Split by any non-letter characters
while (scanner.hasNext()) {
    System.out.println(scanner.next());
}
```
Or, since Java 9+, use stream tokens:
```java
Stream<String> tokens = new Scanner(Path.of("file.txt"))
                            .useDelimiter("\\PL+")
                            .tokens();
```

### 5. Reading numbers
Be careful: number formats depend on locale (country settings).
* **Example:**
  * “100.000” = 100.0 in the US
  * “100.000” = 100000.0 in Germany

If you need to handle locale-specific numbers, use: `Integer.parseInt`/`Double.parseDouble`.

---

## Writing Text Files in Java

### 1. Writing a String to a File
If you want to write a simple string to a file, you can use:
```java
String content = "Hello World";
Files.writeString(path, content);
```

### 2. Writing Multiple Lines
If you have multiple lines, use a list of strings:
```java
List<String> lines = List.of("Line 1", "Line 2", "Line 3");
Files.write(path, lines);
```

### 3. Using PrintWriter for Formatted Output
If you need formatted output (similar to `printf`), use `PrintWriter`:
```java
PrintWriter writer = new PrintWriter(path.toFile());
writer.printf(locale, "Hello, %s. Next year you will be %d years old!%n", name, age + 1);
writer.close();
```

### 4. Writing with BufferedWriter
If you don’t need formatted output, `BufferedWriter` is a good alternative:
```java
BufferedWriter writer = Files.newBufferedWriter(path);
writer.write("Hello World");
writer.newLine();
writer.close();
```

---

## Reading Data from an Input Stream
A common use case for streams is reading data from a web URL.

### 1. Using HttpClient (Recommended for HTTP Requests)
```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://test.com/index.html"))
    .GET()
    .build();

HttpResponse<String> response =
    client.send(request, HttpResponse.BodyHandlers.ofString());

String result = response.body();
```
This approach is best when you need access to headers, status codes, or advanced HTTP features.

### 2. Reading Directly from a URL
If you only need the raw content, you can use a simpler approach:
```java
InputStream in = new URI("https://test.com/index.html").toURL().openStream();
byte[] bytes = in.readAllBytes();
String result = new String(bytes);
```
Or write it directly to a file:
```java
try (OutputStream out = Files.newOutputStream(path)) {
    in.transferTo(out);
}
```

### 3. Reading JSON or Images from a URL
Many libraries can read directly from a URL without manually handling streams.
* **Example with Jackson:**
```java
URL url = new URI("https://test/api/image/random").toURL();
Map<String, Object> result = JSON.std.mapFrom(url);
```
* **To load an image:**
```java
URL imageUrl = new URI(result.get("message").toString()).toURL();
BufferedImage image = ImageIO.read(imageUrl);
```
This approach is preferred because the library can automatically detect file types and handle decoding correctly.

---

## The Files API in Java
The `java.nio.file.Files` class provides a powerful and convenient way to work with files and directories. It supports common operations such as creating, reading, writing, copying, moving, and deleting files.

### Traversing Files and Directories
**1. Listing Files in a Directory:**
To list all entries (files and subdirectories) in a directory, use `Files.list()`:
```java
try (Stream<Path> entries = Files.list(pathToDirectory)) {
    entries.forEach(System.out::println);
}
```

**2. Traversing Subdirectories Recursively:**
If you want to traverse all subdirectories, use `Files.walk()`:
```java
try (Stream<Path> entries = Files.walk(pathToDirectory)) {
    List<Path> htmlFiles =
        entries.filter(p -> p.toString().endsWith(".html"))
               .toList();
}
```

### Working with ZIP Files
Java provides built-in ZIP file support through the file system API.

```java
try (FileSystem fs = FileSystems.newFileSystem(pathToZipFile)) {
    // Work with files inside the ZIP
}
```
Once opened, the ZIP file behaves like a regular file system.

* **Example: List Files in a ZIP Archive**
```java
try (Stream<Path> entries = Files.walk(fs.getPath("/"))) {
    List<Path> files = entries
        .filter(Files::isRegularFile)
        .toList();
}
```

* **Reading a File from a ZIP**
```java
String content = Files.readString(fs.getPath("/LICENSE"));
```
You can also add or replace files using `Files.write()` or `Files.writeString()`.

### Creating Temporary Files and Directories
Temporary files are useful for intermediate processing.
```java
Path tempFile = Files.createTempFile("myapp", ".txt");
Path tempDir  = Files.createTempDirectory("myapp");
```

---

Modern Java provides powerful, readable, and safe APIs for file handling. By using `java.nio.file`, you write less code, avoid common bugs, and improve maintainability.