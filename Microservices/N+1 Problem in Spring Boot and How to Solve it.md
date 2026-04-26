# What is N+1 Problem in Spring Boot and How to Solve it?
**The N+1 problem is a sneaky database performance killer in JPA. What it is, why it happens, and how to fix it.**


![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*kfop5BKI1u389LxCNmFnmw.png)

**Table of Contents:**
* What is the N+1 Problem?
* Setting Up a Scenario
* Seeing the Problem in Action
* Fixing N+1 with @EntityGraph
* JOIN FETCH as an Alternative
* When BatchSize Can Help

### What is the N+1 Problem?
Alright, so the N+1 problem is when your application makes one initial query to fetch “parent” entities, then **N** additional queries for their “child” entities or related data.

Think of it: you want a list of authors and their books.
1. You query for all authors (that’s 1 query).
2. Now you have, say, 10 authors.
3. For **each** of those 10 authors, your code tries to access their list of books.
4. Because those books weren’t fetched initially, JPA (Hibernate) goes back to the database **10 more times**, once for each author.

So, 1 query + N queries. If N is 10,000, your database is going to hate you. This usually happens when you’re using **lazy loading** for your relationships, which is often the default for `@OneToMany` and `@ManyToOne` and generally a good thing, until it isn’t.

---

### Setting Up a Scenario
Let’s set up a quick example: **Author** and **Book** entities. One author can have many books. Lazy fetching is key here.

**Author.java**
```java
@Entity
public class Author {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "author", fetch = FetchType.LAZY) // Lazy by default
    private List<Book> books = new ArrayList<>();

    // Getters and Setters, constructor...
    // Yeah, I know, Lombok would clean this up. I just haven't added it to this project yet.
    // For brevity, I'll skip some boilerplate here.

    public Author() {}
    public Author(String name) { this.name = name; }
    public Long getId() { return id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public List<Book> getBooks() { return books; }
    public void setBooks(List<Book> books) { this.books = books; }
}
```

**Book.java**
```java
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;

    @ManyToOne(fetch = FetchType.LAZY) // Lazy by default
    @JoinColumn(name = "author_id")
    private Author author;

    // Getters and Setters, constructor...
    public Book() {}
    public Book(String title, Author author) {
        this.title = title;
        this.author = author;
    }
    public Long getId() { return id; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public Author getAuthor() { return author; }
    public void setAuthor(Author author) { this.author = author; }
}
```

**And our repositories:**
```java
// AuthorRepository.java
public interface AuthorRepository extends JpaRepository<Author, Long> {}

// BookRepository.java
public interface BookRepository extends JpaRepository<Book, Long> {}
```

To make sure we have some data, a **data.sql** file:
```sql
INSERT INTO author (name) VALUES ('Stephen King');
INSERT INTO author (name) VALUES ('Agatha Christie');
INSERT INTO author (name) VALUES ('J.K. Rowling');

INSERT INTO book (title, author_id) VALUES ('It', 1);
INSERT INTO book (title, author_id) VALUES ('The Shining', 1);
INSERT INTO book (title, author_id) VALUES ('Murder on the Orient Express', 2);
INSERT INTO book (title, author_id) VALUES ('And Then There Were None', 2);
INSERT INTO book (title, author_id) VALUES ('Harry Potter and the Sorcerer''s Stone', 3);
INSERT INTO book (title, author_id) VALUES ('Harry Potter and the Chamber of Secrets', 3);
```

We’ll also want to see the SQL queries. Add these to your **application.properties**:
```properties
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

---

### Seeing the Problem in Action
Now, let’s write a simple service method that’s going to trigger our N+1 problem.

**AuthorService.java**
```java
@Service
public class AuthorService {
    private final AuthorRepository authorRepository;
    public AuthorService(AuthorRepository authorRepository) {
        this.authorRepository = authorRepository;
    }

    @Transactional(readOnly = true)
    public List<Author> getAllAuthorsWithBooksProblematic() {
        List<Author> authors = authorRepository.findAll(); // Query 1
        for (Author author : authors) {
            // This line triggers a new query for each author's books
            // because 'books' is LAZY and not yet loaded.
            author.getBooks().size(); // Just accessing it
        }
        return authors;
    }
}
```

**And a controller endpoint to call it:**
```java
// AuthorController.java
@RestController
@RequestMapping("/authors")
public class AuthorController {
    private final AuthorService authorService;
    public AuthorController(AuthorService authorService) {
        this.authorService = authorService;
    }

    @GetMapping("/problematic")
    public List<Author> getAuthorsProblematic() {
        return authorService.getAllAuthorsWithBooksProblematic();
    }
}
```

When you hit `/authors/problematic`, your console will show something like this (simplified):
```text
Hibernate: select author0_.id as id1_0_, author0_.name as name2_0_ from author author0_
Hibernate: select books0_.author_id as author_i3_1_0_, books0_.id as id1_1_0_, books0_.id as id1_1_1_, books0_.author_id as author_i3_1_1_, books0_.title as title2_1_1_ from book books0_ where books0_.author_id=?
Hibernate: select books0_.author_id as author_i3_1_0_, books0_.id as id1_1_0_, books0_.id as id1_1_1_, books0_.author_id as author_i3_1_1_, books0_.title as title2_1_1_ from book books0_ where books0_.author_id=?
Hibernate: select books0_.author_id as author_i3_1_0_, books0_.id as id1_1_0_, books0_.id as id1_1_1_, books0_.author_id as author_i3_1_1_, books0_.title as title2_1_1_ from book books0_ where books0_.author_id=?
```
One query for author, then **three** separate queries for book. That’s N+1. If we had 1000 authors, that’d be 1001 queries.

---

### Fixing N+1 with @EntityGraph
One of the cleanest ways to fix this in Spring Data JPA is using the `@EntityGraph` annotation. It tells JPA to eagerly fetch specific associations along with the root entity in a single query.

You add it directly to your repository method:
```java
// AuthorRepository.java
import org.springframework.data.jpa.repository.EntityGraph;

public interface AuthorRepository extends JpaRepository<Author, Long> {
    @EntityGraph(attributePaths = "books")
    List<Author> findAllWithBooks();
}
```

Now, update your service method and add a new controller endpoint:
```java
// AuthorService.java (updated method)
@Transactional(readOnly = true)
public List<Author> getAllAuthorsWithBooksEntityGraph() {
    List<Author> authors = authorRepository.findAllWithBooks(); // One smart query!
    for (Author author : authors) {
        author.getBooks().size(); // Books already loaded
    }
    return authors;
}

// AuthorController.java (new method)
@GetMapping("/entitygraph")
public List<Author> getAuthorsEntityGraph() {
    return authorService.getAllAuthorsWithBooksEntityGraph();
}
```

When you hit `/authors/entitygraph`, you’ll see something like this:
```text
Hibernate: select author0_.id as id1_0_0_, author0_.name as name2_0_0_, books1_.author_id as author_i3_1_1_, books1_.id as id1_1_1_, books1_.id as id1_1_2_, books1_.author_id as author_i3_1_2_, books1_.title as title2_1_2_ from author author0_ left outer join book books1_ on author0_.id=books1_.author_id
```
Just one beautiful **LEFT OUTER JOIN**. All authors and their books fetched in a single go. Much better. You can specify multiple `attributePaths` if you have more nested relationships.

---

### JOIN FETCH as an Alternative
If you prefer more fine-grained control over the query, you can use **JOIN FETCH** directly in a JPQL query. This achieves pretty much the same result as `@EntityGraph`.

You’d add a custom query to your **AuthorRepository**:
```java
// AuthorRepository.java (another method)
import org.springframework.data.jpa.repository.Query;

public interface AuthorRepository extends JpaRepository<Author, Long> {
    @EntityGraph(attributePaths = "books")
    List<Author> findAllWithBooks();

    @Query("SELECT a FROM Author a JOIN FETCH a.books")
    List<Author> findAllWithBooksJoinFetch();
}
```

Then, update your service method and add a controller endpoint:
```java
// AuthorService.java (yet another method)
@Transactional(readOnly = true)
public List<Author> getAllAuthorsWithBooksJoinFetch() {
    List<Author> authors = authorRepository.findAllWithBooksJoinFetch();
    for (Author author : authors) {
        author.getBooks().size();
    }
    return authors;
}

// AuthorController.java (last new method)
@GetMapping("/joinfetch")
public List<Author> getAuthorsJoinFetch() {
    return authorService.getAllAuthorsWithBooksJoinFetch();
}
```

The SQL output for `/authors/joinfetch` will look very similar. Just be careful: **JOIN FETCH** on multiple collections can cause a Cartesian product, inflating your result set. Something to watch out for.

---

### When BatchSize Can Help
Sometimes, a full join isn’t ideal. Maybe the related collection is huge, or you’re running into that Cartesian product issue. `@BatchSize` offers a middle ground. Instead of fetching each child collection individually (N queries), Hibernate can fetch them in batches. So, 1 query for parents, then (N/batch_size) queries for children. Much better.

You apply `@BatchSize` directly to the collection you want to batch fetch:
```java
// Author.java (updated @OneToMany)
import org.hibernate.annotations.BatchSize;

@Entity
public class Author {
    // ... other fields

    @OneToMany(mappedBy = "author", fetch = FetchType.LAZY)
    @BatchSize(size = 10) // Fetch books for 10 authors at a time
    private List<Book> books = new ArrayList<>();

    // ...
}
```

With `@BatchSize(10)` on our `books` collection, hitting the original `/authors/problematic` endpoint will show fewer queries. For our 3 authors, it’ll likely be 1+1 query. If you had 25 authors, with `BatchSize(10)`, it would be 1 + 3 queries. It works by using **IN** clauses.

Lazy loading is good, but you have to be mindful of **when** those lazy collections get accessed. Don’t just set `fetch = FetchType.EAGER` everywhere; it often causes worse performance and memory issues. Be selective with `@EntityGraph`, **JOIN FETCH**, or `@BatchSize`.

### Conclusion:
So, that’s the N+1 problem in a nutshell. It’s about too many database round trips with lazy-loaded relationships. Spring Boot and JPA offer `@EntityGraph`, **JOIN FETCH**, and `@BatchSize` to fix it. Always keep an eye on your SQL logs — they tell the real story. This worked for me, anyway.