# How To Improve Performance Of My ASP.NET Core Web API In 18x Times Using HybridCache In .NET 9

After .NET 9 came out I was playing around with it and wanted to test the new `HybridCache`. So I decided to test how performant is one of my WEB APIs and how I can improve its performance by using `HybridCache`.

[](#application-we-will-be-exploring)

## Application We Will be Exploring

One of my applications used Minimal APIs and EF Core to work with Authors and their Books:

```csharp
public class Author
{
    public required Guid Id { get; set; }
    public required string Name { get; set; }
    public List<Book> Books { get; set; } = [];
}

public class Book
{
    public required Guid Id { get; set; }
    public required string Title { get; set; }
    public required int Year { get; set; }
    public Guid AuthorId { get; set; }
    public Author Author { get; set; } = null!;
}
```

This application targets .NET 8 and has the following Minimal API endpoints:

-   Books: Create, Update, Delete, GetById, GetBooksByTitle
-   Authors: Create, Update, Delete, GetById

[](#testing-performance-with-k6)

## Testing Performance with k6

Let's explore the "Get Book By Id" endpoint:

```csharp
public class GetBookByIdEndpoint : ICarterModule
{
    public void AddRoutes(IEndpointRouteBuilder app)
    {
        app.MapGet("/api/books/{id}", Handle);
    }

    private static async Task<IResult> Handle(
        [FromRoute] Guid id,
        ApplicationDbContext context,
        CancellationToken cancellationToken)
    {
        var entity = await context.Books
            .Include(b => b.Author)
            .FirstOrDefaultAsync(b => b.Id == id, cancellationToken);

        if (entity is null)
        {
            return Results.NotFound();
        }

        return Results.Ok(new BookResponse(entity.Id, entity.Title, entity.Year, entity.Author.Name));
    }
}
```

Let's migrate the application to .NET 9 and test this endpoint with `k6` to find out what is the RPS (Requests per second) of the endpoint.

[k6](https://k6.io/) - is an open-source tool and cloud service that makes load testing easy for developers and QA engineers. You can use JavaScript and Go programming languages to write the load tests. This library supports multiple types of load tests.

Here is the code with `k6` using JavaScript to create a load test:

```javascript
import http from "k6/http";
import { check } from "k6";

export let options = {
  noConnectionReuse: false,
  insecureSkipTLSVerify: true,
  scenarios: {
    max_load_test: {
      executor: 'constant-arrival-rate',
      rate: 30000, // Number of requests per second
      timeUnit: '1s', // The time unit over which 'rate' is defined
      duration: '1m', // Test duration (1 minute)
      preAllocatedVUs: 20, // Preallocate 20 Virtual Users
      maxVUs: 100, // Allow up to 100 Virtual Users
    },
  },
};

export default function () {
  const id = "236cf9dd-7664-480e-b7f4-99f4e3790d71";
  const response = http.get(`http://localhost:5001/api/books/${id}`);
  check(response, {
    "status is 200": (r) => r.status === 200,
    "body is not empty": (r) => r.body.length > 0,
  });
}
```

Here I set `rate: 30000` as the base number for desired requests per second for the tested endpoint.

Let's run the test (make sure you have k6 installed on your machine):

```bash
k6 run loadTest.js
```

And here are the results (1,370 RPS):

![Screenshot_1](https://antondevtips.com/media/code_screenshots/aspnetcore/hybrid-cache-performancePS is pretty low, you would say. Can we do better? Let's try.

First, that comes to my mind is to add `AsNoTracking` to the query:

```csharp
var entity = await context.Books
    .Include(b => b.Author)
    .AsNoTracking()
    .FirstOrDefaultAsync(b => b.Id == id, cancellationToken);
```

After running the test, I get around 1,450 RPS. A bit more requests but nothing special.

Let's try to replace EF Core with Dapper.

[](#testing-performance-after-replacing-ef-core-with-dapper)

## Testing Performance after replacing EF Core with Dapper

I switched to Dapper, a lightweight ORM known for its speed.

Dapper maps database results to objects efficiently without the heavy lifting that EF Core does.

Let's replace EF Core with Dapper:

```csharp
private static async Task<IResult> Handle(
    [FromRoute] Guid id,
    SqlConnectionFactory sqlConnectionFactory,
    CancellationToken cancellationToken)
{
    const string sql = @"
        SELECT
           b.id as Id, b.title as Title, b.year as Year, b.author_id as AuthorId,
           a.id AS AuthorId, a.name AS Name
       FROM
           devtips_hybridcache_redis.books b
       INNER JOIN
           devtips_hybridcache_redis.authors a ON b.author_id = a.id
       WHERE
           b.id = @Id";

    await using var connection = sqlConnectionFactory.CreateConnection();
    var result = await connection.QueryAsync<Book, Author, Book>(
        sql,
        (book, author) =>
        {
            book.Author = author;
            return book;
        },
        new { Id = id },
        splitOn: "AuthorId");

    var entity = result.FirstOrDefault();

    if (entity is null)
    {
        return Results.NotFound();
    }

    return Results.Ok(new BookResponse(entity.Id, entity.Title, entity.Year, entity.Author.Name));
}
```

This change resulted in a significant performance jump, reaching 6,299 RPS:

![Screenshot_2](https://antondevtips.com/media/code_screenshots/aspnetcore/hybrid-cache-performance 4x times faster than with EF Core.

The API is now much faster, but I was curious if I could push the boundaries even further.

But do we really need to improve the performance of this request? The answer is - it depends on the business requirements.

Imagine if you have an online bookstore with thousands of books.

When a user opens a new tab to view a concrete book's page - you need to send a request to the database. And what if there are thousands of users on the website at the same time? In such a case, you may need to get the load off your database.

That's why I decided to add caching.

[](#hybridcache-in-net-9)

## HybridCache in .NET 9

I decided to use a "Read Through Cache" pattern that works as follows:

1.  Tries to get entity from the cache
2.  If an entity was found in the cache - it is returned
3.  If an entity is not found, a query is sent to the database to retrieve it
4.  The retrieved entity is written to cache

When working with cache, we must be aware of the cache stampede problem: when multiple requests may receive a cache miss and will all call the database to retrieve the entity. Instead of calling the database once to get the entity and write it into cache.

You can introduce manual locking to solve this problem, but this approach doesn't scale so well.

That's why I am using a new caching library `HybridCache` available in .NET 9 that prevents a cache stampede problem.

`HybridCache` is a great replacement for old `IMemoryCache` and `IDistributedCache` as it combines them both and can work with:

-   In-memory cache
-   Distributed cache like Redis

To get started with `HybridCache`, add the following NuGet package:

```bash
dotnet add package Microsoft.Extensions.Caching.Hybrid
```

Here is how you can register it in DI:

```csharp
#pragma warning disable EXTEXP0018
services.AddHybridCache(options =>
{
    options.DefaultEntryOptions = new HybridCacheEntryOptions
    {
        Expiration = TimeSpan.FromMinutes(10),
        LocalCacheExpiration = TimeSpan.FromMinutes(10)
    };
});
#pragma warning restore EXTEXP0018
```

I need to disable the warning with pragma as `HybridCache` is in preview at the moment (it might be already released when you're reading this).

By default, it enables the InMemory Cache.

Let's update our endpoint to use the cache. You can use `GetOrCreateAsync` method that will check if entity exists in the cache and returns it. If the entity is not found - a delegate method is called and I get the entity from EF Core.

```csharp
private static async Task<IResult> Handle(
    [FromRoute] Guid id,
    HybridCache cache,
    ApplicationDbContext context,
    CancellationToken cancellationToken)
{
    var bookResponse = await cache.GetOrCreateAsync($"book-{id}",
        async token =>
        {
            var entity = await context.Books
                .Include(b => b.Author)
                .FirstOrDefaultAsync(b => b.Id == id, token);

            return entity is not null
                ? new BookResponse(entity.Id, entity.Title, entity.Year, entity.Author.Name)
                : null;
        },
        cancellationToken: cancellationToken
    );

    return bookResponse is not null ? Results.Ok(bookResponse) : Results.NotFound();
}
```

Here there is no need to use Dapper as the entity will be retrieved from the database only once.

Let's run our load tests and see how our performance improved:

![Screenshot_3](https://antondevtips.com/media/code_screenshots/aspnetcore/hybrid-cache-performance improved the performance to 25,590 RPS. By serving data from the cache instead of hitting the database every time, response times decreased dramatically.

[](#using-redis-with-hybridcache)

## Using Redis with HybridCache

Now let's add Redis and see if it can outperform the InMemory caching.

First, add the following NuGet package:

```bash
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis
```

Here is how you can register Redis in DI:

```csharp
services
    .AddStackExchangeRedisCache(options =>
    {
        options.Configuration = configuration.GetConnectionString("Redis")!;
    });
```

That's it, now `HybridCache` will automatically use Redis as distributed cache.

Let's run the tests and see what happens:

![Screenshot_4](https://antondevtips.com/media/code_screenshots/aspnetcore/hybrid-cache-performance slightly improved the performance to 25,798 RPS. While the increase was not as huge as previous improvements, Redis provides additional benefits like persistence and scalability.

This in total gives **18x** performance improvement compared to the first version.

[](#summary)

## Summary

By methodically analyzing and addressing performance bottlenecks, I transformed my ASP.NET Core Web API from handling 1,370 RPS to an impressive 25,798 RPS — **18x** times increase.

I used the following steps to increase the performance:

-   Use AsNoTracking in EF Core
-   Replaced EF Core with Dapper
-   Used InMemory cache using HybridCache
-   Used Redis cache using HybridCache
