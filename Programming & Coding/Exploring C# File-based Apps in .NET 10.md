# Exploring C# File-based Apps in .NET 10

C# has always been a bit ceremony-heavy, and we all know it. Even the simplest "Hello World" program traditionally needed a solution file, a project file, and enough boilerplate to make you wonder if you should've just used a scripting language instead.

Well, Microsoft finally heard us. With .NET 10, they've introduced [**file-based apps**](run-csharp-scripts-with-dotnet-run-app-no-project-files-needed). And honestly, it's about time.

## [What are File-based Apps?](#what-are-file-based-apps)

The idea is simple: write your C# code in a single `.cs` file and run it directly. That's it.

No need to set up a whole project structure for that quick utility script you need to parse some CSV files or test an API endpoint.

You get to keep everything that makes C# great: the **type safety**, the **performance**, the **rich standard library**. But now you can use it for those throwaway scripts where setting up a full project would've been overkill. And yes, you can still **reference NuGet packages**, **reference other C# projects**, target specific SDKs, and configure project properties, all from within that single file using special directives that start with `#:`.

The feature builds on **top-level statements** (remember those from C# 9?) and takes them to their logical conclusion. If we're already letting people skip the class and `Main` method ceremony, why not let them skip the project ceremony too?

## [Getting Started with File-based Apps](#getting-started-with-file-based-apps)

Let's say you want to quickly check what day of the week a specific date falls on. Create a file called `date-checker.cs`:

```csharp
var targetDate = new DateTime(2025, 12, 31);
Console.WriteLine($"New Year's 2025 falls on a {targetDate.DayOfWeek}");
Console.WriteLine($"That's {(targetDate - DateTime.Today).Days} days from now");
```

Run it with:

```bash
dotnet run date-checker.cs
```

The first time you run this, the CLI does some behind-the-scenes magic. It creates a virtual project, compiles your code, and caches everything. Subsequent runs are nearly instant because it's smart enough to know when nothing has changed.

## [Real-world Example: Quick Data Processing](#real-world-example-quick-data-processing)

Here's where things get interesting. Say you need to quickly process some JSON data and generate a report.

Let's do something practical with `System.Text.Json` and `CsvHelper`:

```csharp
#:package CsvHelper@33.1.0
using System.Text.Json;
using CsvHelper;
using System.Globalization;

var json = await File.ReadAllTextAsync("sales_data.json");
var sales = JsonSerializer.Deserialize<List<SaleRecord>>(json);

var topProducts = sales
    .GroupBy(s => s.Product)
    .Select(g => new {
        Product = g.Key,
        TotalRevenue = g.Sum(s => s.Amount),
        UnitsSold = g.Count()
    })
    .OrderByDescending(p => p.TotalRevenue)
    .Take(10);

using var writer = new StreamWriter("top_products.csv");
using var csv = new CsvWriter(writer, CultureInfo.InvariantCulture);
csv.WriteRecords(topProducts);

Console.WriteLine("Report generated! Check top_products.csv");

record SaleRecord(string Product, decimal Amount, DateTime Date);
```

Notice how we're mixing package references, async operations, LINQ, and record types, all in a single file that reads like a cohesive script. This is the kind of thing you'd typically reach for Python for, but now you can stay in C# land.

## [Building Something More Ambitious with Aspire](#building-something-more-ambitious-with-aspire)

You can actually create an **Aspire AppHost** in a single file, which is pretty wild when you think about it:

```csharp
#:sdk Aspire.AppHost.Sdk@13.0.0
#:package Aspire.Hosting.AppHost@13.0.0

var builder = DistributedApplication.CreateBuilder(args);

var cache = builder.AddRedis("cache")
    .WithDataVolume();

var postgres = builder.AddPostgres("postgres")
    .WithDataVolume()
    .AddDatabase("tododb");

var todoApi = builder.AddProject<Projects.TodoApi>("api")
    .WithReference(cache)
    .WithReference(postgres);

builder.AddNpmApp("frontend", "../TodoApp")
    .WithReference(todoApi)
    .WithReference("api")
    .WithHttpEndpoint(env: "PORT")
    .WithExternalHttpEndpoints();

builder.Build().Run();
```

With the `#:sdk Aspire.AppHost.Sdk@13.0.0` directive, your single file becomes a full orchestrator for a distributed application. You're defining infrastructure, wiring up dependencies, and setting up a complete development environment, all without creating a project file. It's particularly useful when you're prototyping architectures or need to quickly spin up a test environment.

## [Migrating to a Full Project](#migrating-to-a-full-project)

Eventually, some scripts outgrow their single-file roots. Maybe you need to split things into multiple files, or maybe you want proper IDE support for debugging. The transition is painless:

```bash
dotnet project convert MyUtility.cs
```

This generates a proper project structure while preserving all your package references and SDK choices. Your code moves to `Program.cs`, and you get a `.csproj` that reflects all those `#:` directives you were using.

## [Current Limitations](#current-limitations)

Right now, it's strictly single-file. If you need **multiple files**, you'll have to [wait for .NET 11](https://github.com/dotnet/sdk/issues/48174) or convert to a full project. Of course, you can still reference other projects or packages, but the main script has to be in one file.

The caching mechanism can occasionally get confused if you're rapidly iterating on package versions. And while the IDE support is getting better, it's not quite at the same level as full projects yet, especially for IntelliSense with dynamically referenced packages.

But for what it's designed to do (make C# approachable for scripting scenarios), it works remarkably well. You can use it for **build scripts**, **data migration one-offs**, quick API tests, or even teaching C# without overwhelming beginners with project structure.

## [Where This Leaves Us](#where-this-leaves-us)

**File-based apps** feel like C# finally acknowledging that not every piece of code needs to be an enterprise application. Sometimes you just need to parse a log file, sometimes you want to quickly test an algorithm, and sometimes you're teaching someone programming and don't want to explain what a solution file is on day one.

The feature doesn't revolutionize C# development, but it **fills a gap** that's been annoying developers for years. If you've been keeping a folder of scripts for quick tasks because C# felt too heavy, maybe it's time to give your favorite language another shot.

After all, the best language for a quick script is the one you already know, and now C# makes that argument a lot more compelling.

* * *