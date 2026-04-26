# When NOT to use the repository pattern in EF Core

### Contents
If you design an application with a data source, the repository pattern often comes to mind as a prominent choice. In fact, many developers see it as the default choice. However, the pattern is not helping every time. This post pinpoints some cases where the repository pattern is not the best choice.

---

### What is a repository pattern?
The **repository pattern** is a design pattern that acts as an intermediate layer between data access and business logic. It abstracts the data source and implements the details, providing a clean representation of data manipulation as objects and lists.



Let us start by looking at how a repository pattern can be implemented with EF Core.
Start by adding a new model named **Movie**:
```csharp
public class Movie
{
    public Guid Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public string Director { get; set; } = string.Empty;
    public int ReleaseYear { get; set; }
    public double ImdbRating { get; set; }
    public DateTime CreatedAtUtc { get; set; } = DateTime.UtcNow;
}
```

Next, add a **IMovieRepository** interface with the basic methods for adding, getting, and saving movies:
```csharp
public interface IMovieRepository
{
    Task AddAsync(Movie movie);
    Task<Movie?> GetByIdAsync(Guid id);
    Task<List<Movie>> GetTopRatedAsync(double minRating);
    Task SaveChangesAsync();
}
```

Add an implementation of that interface using EF Core:
```csharp
public class MovieRepository : IMovieRepository
{
    private readonly AppDbContext _context;

    public MovieRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task AddAsync(Movie movie)
    {
        await _context.Movies.AddAsync(movie);
    }

    public async Task<Movie?> GetByIdAsync(Guid id)
    {
        return await _context.Movies.FindAsync(id);
    }

    public async Task<List<Movie>> GetTopRatedAsync(double minRating)
    {
        return await _context.Movies
            .Where(m => m.ImdbRating >= minRating)
            .OrderByDescending(m => m.ImdbRating)
            .ToListAsync();
    }

    public async Task SaveChangesAsync()
    {
        await _context.SaveChangesAsync();
    }
}
```

Finally, I'll add a service class that shows how to use the movie repository:
```csharp
public class MovieService
{
    private readonly IMovieRepository _repository;

    public MovieService(IMovieRepository repository)
    {
        _repository = repository;
    }

    public async Task<Guid> CreateMovieAsync(
        string title,
        string director,
        int releaseYear,
        double rating)
    {
        var movie = new Movie
        {
            Id = Guid.NewGuid(),
            Title = title,
            Director = director,
            ReleaseYear = releaseYear,
            ImdbRating = rating
        };

        await _repository.AddAsync(movie);
        await _repository.SaveChangesAsync();

        return movie.Id;
    }

    public async Task<List<Movie>> GetHighlyRatedMoviesAsync()
    {
        return await _repository.GetTopRatedAsync(8.0);
    }
}
```

If you are writing CRUD applications, implementing a data layer like this probably looks very familiar.

---

### What are the advantages of the Repository pattern?
The repository pattern promises several key advantages.
* **A clean separation of concerns** where data access logic is centralized.
* **Reusability**, where the same repo methods can be used without copying the same logic again.

---

### When to use the Repository pattern
Like any tool, it offers leverage only when in the right place. If you smell any scent in your code, go for the repository pattern.
1.  **Complex Query Logic:** When your application does not rely on simple data storage or fetching but requires enquiring logic such as validation, projection, object preparation, or calculations. Domains such as insurance, banking, healthcare, and IoT require calculations, so the repository pattern can be helpful.
2.  **Multiple Data Sources:** The repository pattern can win for you if you are aggregating multiple data sources but presenting them as a single source to the upper layers. Usage of different data sources, such as MSSQL, Postgres, and external APIs, is kept hidden from the business logic layer.
3.  **Sophisticated Caching:** The repository layer can be handy if an application demands sophisticated caching strategies and you don't want to pollute the business layers. Hence, the service layer can be unaware of how the cache is configured, or even of whether the data comes from the cache or another source.
4.  **Error-Critical Systems (Testing):** For unit testing, you can employ the repository pattern, especially in error-critical systems such as financial systems, medical devices, and safety systems. Repositories enable you to test business logic in **isolation** by swapping real data access with test doubles. You can verify complex business rules, edge cases, and error handling without the overhead, unpredictability, and slowness of database tests.

---

### When to avoid the Repository pattern
Well, we have seen the usefulness of the repository pattern. Now, rejoining our original question, "In what conditions can you avoid the repository pattern?"



1.  **Simple CRUD Applications:** If your app is just basic Create, Read, Update, Delete operations without complex business logic, you simply go without it. A simple creation or fetch will not require verbose code, and adding a new layer will overengineer it. For example, in the code, a repository pattern has simple operations:
```csharp
public class UserRepository : IUserRepository 
{
    public User GetById(int id) => _context.Users.Find(id);
    public void Add(User user) => _context.Users.Add(user);
}
```
2.  **Leveraging ORM Features:** With an ORM, you can avoid the abstraction layer. Most ORMs, such as Entity Framework Core, NHibernate, and Doctrine, already implement the repository pattern using `DbSet` and `AppDbContext`. You can simply deal with entities like collections and objects. If you don't have to add conditions, validation, and projections in the operations, you can choose simplicity. When wrapping an ORM in repositories, you are often hiding powerful features (like `IQueryable` for deferred execution or `Include` for eager loading) behind a more restrictive interface.
3.  **Small Projects:** Smaller projects also don't need to be tedious. If your project requires simple queries and consists of 10-15 tables, you are good to go without bombarding a small project with more code.
4.  **Performance-Critical Systems:** Any abstraction comes with overhead. In a performance-critical system, a repository may not be the best choice for the same reason. Repository layers can require memory allocation, additional method calls, or complex query translation, which may slow down the software. Repositories often lead to the N+1 query problem or over-fetching data because the repository method returns a generic object rather than a specific projection (`Select`) tailored to the view.
5.  **Small Microservices:** One more scenario where you can skip the repository pattern is in a microservice architecture. If a service is simple enough to have a small database and minimal operations, you don't need to trade off the repository pattern for maintenance and performance.
6.  **Reporting and Analytics:** While preparing reporting and analytics data, the repository pattern can be unnecessary. Mostly, the stored procedures, raw SQL queries, and database-specific optimizations do the whole job for us. The code only calls those underlying queries and returns. To keep things maintainable and, of course, speedy, you can avoid one layer.

---

### Conclusion
The repository pattern is something you have probably used on your development journey. Why not? It is one of the popular choices for abstracting data access. However, abstractions have a hidden cost that I highlighted in the blog. I identified a few scenarios where you can escape it and barely lose anything. If you still want to use the repository pattern without losing its limitation, the **specification pattern** is another player that can work. It allows for reusable query logic without the bloat of a traditional repository.