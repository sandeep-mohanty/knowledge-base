# EF Core Bulk Data Retrieval: 5 Methods You Should Know

When working with Entity Framework Core, developers often need to retrieve multiple entities from the database based on a collection of IDs or values.

The typical approach is to use the `Contains` method:

```csharp
var productIds = GetProductIdsForSync();
var products = await dbContext.Products
    .Where(p => productIds.Contains(p.Id))
    .ToListAsync();
```

This approach works fine for small collections, but it creates significant problems when dealing with large datasets:

-   **Performance degradation** - Even with proper indexes, the SQL `WHERE IN` clause becomes slower as the number of parameters increases.
-   **Parameter limit exceeded** - SQL Server has a hard limit of 2,100 parameters per query. While you can work around this by splitting queries into batches in SQL, this is not always practical when using EF Core, especially in complex scenarios with additional filters and joins.
-   **Memory and connection issues** - Multiple round-trips to the database consume more memory and keep database connections open longer, potentially causing connection pool exhaustion.

This becomes a real problem in scenarios like synchronizing product catalogs, processing bulk orders, or updating inventory across thousands of items.

There is a better solution: [Entity Framework Extensions](https://entityframework-extensions.net?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) library provides specialized methods for bulk data retrieval that solve all these problems.

In this post, you will explore 5 methods that allow you to efficiently retrieve large datasets without hitting parameter limits or sacrificing performance.

Let's dive in.

[](#the-problem-with-contains-method)

## The Problem with Contains Method

All examples in this post were tested on the SQL Server database.

The Contains method in EF Core translates to a SQL `WHERE IN` clause.

When you pass a collection of 10,000 product IDs, EF Core generates a query with 10,000 parameters:

```csharp
var productIds = await GetProductIdsFromExternalSystem();
var products = await dbContext.Products
    .Where(p => productIds.Contains(p.Id))
    .ToListAsync();
```

```csharp
SELECT * FROM Products WHERE Id IN (@p0, @p1, @p2, ... @p9999)
```

This approach has critical limitations: **SQL Server Parameter Limit** - SQL Server allows a maximum of 2,100 parameters in a single query. If you try to query more than 2,100 products, EF Core will throw an exception:

```csharp
System.Data.SqlClient.SqlException: The incoming request has too many parameters.  The server supports a maximum of 2100 parameters.
```

As a workaround, you could split the query into batches:

```csharp
var batchSize = 2000;
var allProducts = new List<Product>();

for (int i = 0; i < productIds.Count; i += batchSize)
{
    var batch = productIds.Skip(i).Take(batchSize).ToList();

    var products = await dbContext.Products
        .Where(p => batch.Contains(p.Id))
        .ToListAsync();

    allProducts.AddRange(products);
}
```

However, this solution creates multiple round-trips to the database, keeps connections open longer, and makes it difficult to apply additional filters or includes across all batches.

In SQL, you could use a temporary table or table-valued parameter to work around the limit. Still, these approaches require raw SQL and lose the benefits of EF Core's strongly-typed queries, change tracking, and navigation property support.

For product synchronization scenarios where you need to retrieve 50,000 products from your database to compare with an external system, this becomes a significant bottleneck.

[Entity Framework Extensions](https://entityframework-extensions.net?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) is a library that extends EF Core with high-performance bulk operations. While it's well-known for [bulk inserts](https://entityframework-extensions.net/bulk-insert?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025), [updates](https://entityframework-extensions.net/bulk-update?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025), and [deletes](https://entityframework-extensions.net/bulk-delete?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025), it also provides powerful methods for bulk data retrieval.

To get started with Entity Framework Extensions, install the following NuGet package:

```csharp
dotnet add package Z.EntityFramework.Extensions.EFCore
```

The library provides specialized methods that use temporary tables internally to bypass the parameter limit and improve performance. Instead of passing thousands of parameters in a `WHERE IN` clause, these methods:

-   Create a temporary table in the database
-   Insert your filter values into that temporary table
-   Join your entity table with the temporary table
-   Return the filtered results
-   Clean up the temporary table automatically

This approach significantly improves query performance for large datasets.

The Entity Framework Extensions library offers five main methods for bulk data retrieval:

-   [WhereBulkContains](https://entityframework-extensions.net/where-bulk-contains?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) - filter entities with a LINQ query using all items from an existing list. Only the database items that match the items in your list will be returned.
-   [WhereBulkNotContains](https://entityframework-extensions.net/where-bulk-not-contains?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) - filter entities with a LINQ query by excluding all items from an existing list.
-   [BulkRead](https://entityframework-extensions.net/bulk-read?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) - filter entities with LINQ query by including all items from an existing list and then return the result immediately
-   [WhereBulkContainsFilterList](https://entityframework-extensions.net/where-bulk-contains-filter-list?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) - returns items from the list that already exist in the database.
-   [WhereBulkNotContainsFilterList](https://entityframework-extensions.net/where-bulk-not-contains-filter-list?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) - returns items from the list that don't exist in the database. It's the inverse of the `WhereBulkContainsFilterList` method.

Let's explore these methods in more detail.

[](#method-1-wherebulkcontains)

## Method 1: WhereBulkContains

The [WhereBulkContains](https://entityframework-extensions.net/where-bulk-contains?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) method is the most commonly used bulk retrieval method. It retrieves entities where a specified property matches any value in a collection.

This method is particularly useful for product synchronization scenarios where you need to retrieve thousands of products from your database to compare with data from an external system, API, or file import.

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var productIds = await GetProductIdsFromExternalSystem();

var products = await dbContext.Products
    .WhereBulkContains(productIds)
    .ToListAsync();
```

The method automatically uses the entity's primary key to match items. In this example, it joins on `Id`.

You can also filter by specifying any property as the 2nd parameter:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var products = await GetProductsFromExternalSystem();

var products = await dbContext.Products
    .WhereBulkContains(products, x => x.Id)
    .ToListAsync();
```

The first parameter is the collection of values to filter by, and the second parameter is a lambda expression that specifies which property to match.

All kinds of lists are supported. The only requirement is that your list is a basic type or contains a property with the same name as the key:

-   Basic type such as List<int> and List<Guid>
-   Entity Type such as List<Customer>
-   Anonymous Type
-   Expando Object list

**Filtering by Non-Primary Key Properties:**

You can use `WhereBulkContains` with any property, not just the primary key:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var productCodes = new List<string>
{
    "SKU-001", "SKU-002", "SKU-003", // ...
};

var products = await dbContext.Products
    .WhereBulkContains(productCodes, x => x.ProductCode)
    .ToListAsync();
```

You can filter by multiple properties simultaneously:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var products = context.Products
    .WhereBulkContains(deserializedProducts, x => new { x.SupplierCode, x.ProductCode })
    .ToListAsync();
```

WhereBulkContains returns `IQueryable<T>` that can be chained with other LINQ methods like `Where`, `Select`, and `Include`:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var productIds = GetProductIdsForSync();

var activeProducts = await dbContext.Products
    .Where(p => p.IsActive && p.Stock > 0)
    .WhereBulkContains(productIds, x => x.Id)
    .Select(p => new
    {
        p.Id,
        p.Name,
        p.Price,
        CategoryName = p.Category.Name
    })
    .ToListAsync();
```

This is much more flexible than splitting queries into batches, where applying additional filters and includes across batches becomes complex and error-prone.

[](#method-2-wherebulknotcontains)

## Method 2: WhereBulkNotContains

The [WhereBulkNotContains](https://entityframework-extensions.net/where-bulk-not-contains?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) method is the opposite of `WhereBulkContains`. It retrieves entities where a specified property does not match any value in the collection.

This method is useful when you need to find entities that are missing from an external system or when you want to exclude a large set of items from your query results.

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var discontinuedProductIds = await GetDiscontinuedProductIdsFromExternalSystem();

var activeProducts = await dbContext.Products
    .WhereBulkNotContains(discontinuedProductIds, x => x.Id)
    .ToListAsync();
```

This retrieves all products except those in the `discontinuedProductIds` collection, without hitting the parameter limit.

Like `WhereBulkContains`, you can chain this method with other LINQ operations:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var processedProductIds = await GetProcessedProductIds();

var remainingProducts = await dbContext.Products
    .Where(p => p.RequiresProcessing)
    .WhereBulkNotContains(processedProductIds, x => x.Id)
    .OrderBy(p => p.Priority)
    .Take(1000)
    .ToListAsync();
```

[](#method-3-bulkread)

## Method 3: BulkRead

The [BulkRead](https://entityframework-extensions.net/bulk-read?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) method is a convenience method that combines `WhereBulkContains` with immediate execution.

It retrieves entities from the database that match items in your list and returns the results immediately as a `List<T>`.

It works the same as `WhereBulkContains` but doesn't require you to chain with other methods to return the results:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var deserializedProducts = DeserializeProductsFromJson(); // Returns 10,000 products

// Retrieve matching products from the database immediately
var products = context.Products.BulkRead(deserializedProducts);
```

The difference is that `BulkRead` returns results immediately, while `WhereBulkContains` returns an `IQueryable<T>` that you can chain with additional LINQ operations.

The async version is also available:

```csharp
var deserializedProducts = DeserializeProductsFromJson();
var products = await context.Products.BulkReadAsync(deserializedProducts);
```

[](#method-4-wherebulkcontainsfilterlist)

## Method 4: WhereBulkContainsFilterList

The [WhereBulkContainsFilterList](https://entityframework-extensions.net/where-bulk-contains-filter-list?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) method filters your input list by returning only the items that already exist in the database.

This is different from `WhereBulkContains`, which returns entities from the database. This method returns items from your input list (not database entities).

This method is particularly useful when you need to split your data into existing and non-existent items, such as separating records that need to be inserted from those that need to be updated.

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var deserializedProducts = DeserializeProductsFromJson(); // Returns 20,000 products

// Returns items from deserializedProducts that exist in the database
var existingProducts = context.Products
    .WhereBulkContainsFilterList(deserializedProducts)
    .ToList();
```

The method returns items from your input list (not database entities) that match rows in the database.

You can specify which property to use for matching:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var existingProducts = context.Products
    .WhereBulkContainsFilterList(deserializedProducts, x => x.ProductCode)
    .ToList();

// Or use a string
var existingProducts = context.Products
    .WhereBulkContainsFilterList(deserializedProducts, "ProductCode")
    .ToList();

// Or use a list of strings
var existingProducts = context.Products
    .WhereBulkContainsFilterList(deserializedProducts, new List<string> { "ProductCode" })
    .ToList();
```

This method supports filtering by multiple properties.

[](#method-5-wherebulknotcontainsfilterlist)

## Method 5: WhereBulkNotContainsFilterList

The [WhereBulkNotContainsFilterList](https://entityframework-extensions.net/where-bulk-not-contains-filter-list?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=december-2025) method is the inverse of `WhereBulkContainsFilterList`. It filters your input list by returning only the items that don't exist in the database.

This method returns items from your input list (not database entities).

This method is useful when you want to identify new records that need to be inserted or when you need to find items from your import that are missing from the database.

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var deserializedProducts = DeserializeProductsFromJson(); // Returns 20,000 products

// Returns items from deserializedProducts that don't exist in the database
var notExistingProducts = context.Products
    .WhereBulkNotContainsFilterList(deserializedProducts)
    .ToList();
```

The method returns items from your input list (not database entities) that don't match rows in the database.

You can specify which property to use for matching.

This method supports filtering by multiple properties.

[](#summary)

## Summary

**When to Use BulkRead vs WhereBulkContains:**

Use `BulkRead` when:

-   You want immediate results as a `List<T>`
-   You don't need to apply additional LINQ operations
-   You're working with deserialized data that needs to be matched against the database

Use `WhereBulkContains` when:

-   You need to chain additional LINQ operations like `Where`, `Select`, or `Include`
-   You want to build a query before executing it
-   You need the flexibility of `IQueryable<T>`

**When to Use WhereBulkContainsFilterList vs WhereBulkContains:**

-   Use `WhereBulkContainsFilterList` when you want to filter your list.
-   Use `WhereBulkContains` when you want to retrieve database entities.

**When to Use WhereBulkNotContainsFilterList vs WhereBulkNotContains:**

-   Use `WhereBulkNotContainsFilterList` when you want to filter your list
-   Use `WhereBulkNotContains` when you want to retrieve database entities.

Entity Framework Extensions is a commercial library with a free trial, which you can use to see if it's right for your project.

