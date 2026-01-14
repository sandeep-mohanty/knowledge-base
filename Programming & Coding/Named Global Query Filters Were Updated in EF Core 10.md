# Named Global Query Filters Were Updated in EF Core 10

**Global query filters** in Entity Framework Core (EF Core) is a powerful feature that can be effectively used to manage data access patterns.

**Global query filters** are LINQ query predicates applied to EF Core entity models. These filters are automatically applied to all queries that involve the corresponding entities. This is especially useful in **multi-tenant** applications or scenarios requiring **soft deletion**.

In EF Core 10, Global Query Filters were renamed to **Named Query Filters**.

This update now solves a significant limitation EF Core previously had: support for multiple global query filters on a single entity.

In this post, we will explore:

- Query Filters for a Soft Delete use case  
- Query Filters for a Multi-Tenant application  
- Named Query Filters  

Let's dive in.

---

## Query filters for a soft delete use case

Soft deletion is implemented by adding an `is_deleted` column to the database table for required entities. Whenever an entity is considered deleted, this column is set to `true`.

### Example entities

```csharp
public class Author
{
    public required Guid Id { get; set; }
    public required string Name { get; set; }
    public required string Country { get; set; }
    public required List<Book> Books { get; set; } = [];
}

public class Book
{
    public required Guid Id { get; set; }
    public required string Title { get; set; }
    public required int Year { get; set; }
    public required bool IsDeleted { get; set; }
    public required Guid TenantId { get; set; }
    public required Author Author { get; set; }
}
```

### DbContext setup

```csharp
public class ApplicationDbContext : DbContext
{
    public DbSet<Author> Authors { get; set; } = default!;
    public DbSet<Book> Books { get; set; } = default!;

    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Book>()
            .HasQueryFilter(x => !x.IsDeleted);

        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<Author>(entity =>
        {
            entity.ToTable("authors");
            entity.HasKey(x => x.Id);
            entity.HasIndex(x => x.Name);

            entity.Property(x => x.Id).IsRequired();
            entity.Property(x => x.Name).IsRequired();
            entity.Property(x => x.Country).IsRequired();

            entity.HasMany(x => x.Books)
                .WithOne(x => x.Author);
        });

        modelBuilder.Entity<Book>(entity =>
        {
            entity.ToTable("books");
            entity.HasKey(x => x.Id);
            entity.HasIndex(x => x.Title);

            entity.Property(x => x.Id).IsRequired();
            entity.Property(x => x.Title).IsRequired();
            entity.Property(x => x.Year).IsRequired();
            entity.Property(x => x.IsDeleted).IsRequired();

            entity.HasOne(x => x.Author)
                .WithMany(x => x.Books);
        });
    }
}
```

### Minimal API endpoint

```csharp
app.MapGet("/api/books", async (ApplicationDbContext dbContext) =>
{
    var nonDeletedBooks = await dbContext.Books.ToListAsync();
    return Results.Ok(nonDeletedBooks);
});
```

### Ignoring filters

```csharp
app.MapGet("/api/all-books", async (ApplicationDbContext dbContext) =>
{
    var allBooks = await dbContext.Books
        .IgnoreQueryFilters()
        .Where(x => x.IsDeleted)
        .ToListAsync();

    return Results.Ok(allBooks);
});
```

---

## Query filters for a multi-tenant application

### Updated entity

```csharp
public class Book
{
    // Other properties ...

    public required Guid TenantId { get; set; }
}
```

### DbContext with tenant filter

```csharp
public class ApplicationDbContext : DbContext
{
    private readonly Guid? _currentTenantId;

    public DbSet<Author> Authors { get; set; } = default!;
    public DbSet<Book> Books { get; set; } = default!;

    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options,
        ITenantService tenantService) : base(options)
    {
        _currentTenantId = tenantService.GetCurrentTenantId();
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Book>()
            .HasQueryFilter(x => x.TenantId == _currentTenantId);

        base.OnModelCreating(modelBuilder);
    }
}
```

### Tenant service

```csharp
public interface ITenantService
{
    Guid? GetCurrentTenantId();
}

public class TenantService : ITenantService
{
    private readonly Guid? _currentTenantId;

    public TenantService(IHttpContextAccessor accessor)
    {
        var headers = accessor.HttpContext?.Request.Headers;
        _currentTenantId = headers.TryGetValue("Tenant-Id", out var value) is true
            ? Guid.Parse(value.ToString())
            : null;
    }

    public Guid? GetCurrentTenantId() => _currentTenantId;
}
```

### Minimal API endpoint

```csharp
app.MapGet("/api/tenant-books", async (ApplicationDbContext dbContext) =>
{
    var tenantBooks = await dbContext.Books.ToListAsync();
    return Results.Ok(tenantBooks);
});
```

---

## Named Query Filters

### DbContext with multiple filters

```csharp
public class ApplicationDbContext : DbContext
{
    public const string SoftDeleteFilter = nameof(SoftDeleteFilter);
    public const string TenantFiler = nameof(TenantFiler);

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Book>()
            .HasQueryFilter(SoftDeleteFilter, x => !x.IsDeleted)
            .HasQueryFilter(TenantFiler, x => x.TenantId == _currentTenantId);

        // ...
    }
}
```

### Ignoring specific filters

```csharp
app.MapGet("/api/tenant-books", async (ApplicationDbContext dbContext) =>
{
    var tenantBooks = await dbContext.Books
        .IgnoreQueryFilters([ApplicationDbContext.SoftDeleteFilter])
        .ToListAsync();

    return Results.Ok(tenantBooks);
});
```

---

## Summary

Global query filters in EF Core enforce data access rules consistently across the application. They are particularly useful in **multi-tenant** architectures and **soft deletion** scenarios.  

In EF Core 10, they were updated to **Named Query Filters**, allowing multiple filters per entity and selective ignoring when needed.
