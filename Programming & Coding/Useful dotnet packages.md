# I Was Writing Too Much Code Until I Found These .NET Packages
**Another carefully curated batch of hidden C# libraries That Do the Work, So You Can Take the credit.**


### Introduction
Let’s be honest for a second.

A lot of .NET codebases are filled with things we don’t *actually* want to write: mapping layers, background workers, retry logic, test scaffolding, parsing glue… the list keeps growing.

And the worst part is, most of this isn’t even “hard” work. It’s just repetitive.

The ecosystem already has solutions for a lot of this, but unless you go looking, you’ll probably never find them.

In this post, I’m listing the **actual packages I use** to cut through repetitive tasks and speed up development. If you’re working in .NET, whether it’s APIs, desktop, or mobile, these tools will make your life a lot easier.

This is Part 5 in an ongoing series on .NET packages.
* [Underrated .NET packages that will save you 100s of lines of code](https://medium.com/gitconnected/underrated-net-packages-that-will-save-you-100s-of-lines-of-code-9f9f9d9f9f9f)
* [Game-Changing .NET Packages You’re Probably Not Using Yet](https://medium.com/gitconnected/game-changing-net-packages-youre-probably-not-using-yet-8e8e8e8e8e8e)
* [Stop Wasting Time: These .NET Packages Do the Hard Work for You](https://medium.com/gitconnected/stop-wasting-time-these-net-packages-do-the-hard-work-for-you-7d7d7d7d7d7d)
* [Stop Reinventing the Wheel: These .NET Packages Work Smarter, So You Don’t Have To](https://medium.com/gitconnected/stop-reinventing-the-wheel-these-net-packages-work-smarter-so-you-don-t-have-to-6c6c6c6c6c6c)
* **I Was Writing Too Much Code Until I Found These .NET Packages**

Let’s get right into it.

---

### 1. FreakyKit.Forge
If you’ve ever written mapping code in C#, you already know how quickly it gets out of hand.
DTOs, domain models, API contracts… and suddenly you’re maintaining dozens of mapping methods that don’t actually add any business value.

This is where **FreakyKit.Forge** stands out.
It uses Roslyn source generators to handle object mapping at compile time, which means:
* No reflection
* No runtime overhead
* No hidden magic

You define your mappings, and it generates everything for you during compilation. Compared to traditional mappers, this feels a lot more predictable. You get strong typing, better performance, and no runtime surprises.

[FreakyKit.Forge 1.0.1](https://www.nuget.org/packages/FreakyKit.Forge/)
*Compile-time object mapping for C# powered by Roslyn source generators. Zero reflection, zero runtime overhead. This…*

---

### 2. ICSharpCode.Decompiler
Sometimes you don’t have the source code.
And when that happens, debugging, understanding behavior, or even verifying assumptions becomes painful.

**ICSharpCode.Decompiler** is the engine behind ILSpy, and it lets you decompile .NET assemblies back into readable C# code.
This isn’t just for curiosity. It’s incredibly useful when:
* Investigating third-party libraries
* Debugging edge cases
* Understanding legacy binaries

It turns a black box into something you can actually reason about.

[ICSharpCode.Decompiler 10.0.0.8330](https://www.nuget.org/packages/ICSharpCode.Decompiler/)
*ICSharpCode.Decompiler is the decompiler engine used in ILSpy.*

---

### 3. BusyBee
Background processing doesn’t always need a full-blown infrastructure setup.
If all you need is simple, in-memory job execution, queues, retries, lightweight scheduling, **BusyBee** gets the job done without pulling in heavy dependencies.

It’s the kind of library you use when:
* You don’t want Redis, queues, or external workers
* You just need something that works inside your app

Clean, minimal, and surprisingly practical.

[BusyBee 0.0.3](https://www.nuget.org/packages/BusyBee/)
*BusyBee is a lightweight library for in-memory background processing in .NET applications. It provides a simple way to…*

---

### 4. RateLimitHeaders
Rate limiting is one of those things you usually react to after things start breaking.
**RateLimitHeaders** flips that around.
It parses IETF rate limit headers and gives your application awareness of:
* Remaining requests
* Reset windows
* Limits imposed by APIs

Instead of blindly firing requests, your client can adapt intelligently. That’s a small change with a big impact in production systems.

[RateLimitHeaders 1.0.0](https://www.nuget.org/packages/RateLimitHeaders/)
*A .NET library for parsing IETF RateLimit headers and enabling proactive rate limit awareness in HTTP clients. Optional…*

---

### 5. Deedle
Working with structured data in .NET can feel… clunky.
**Deedle** brings a more data-frame-like approach, similar to Pandas.

It lets you:
* Work with time series
* Manipulate tabular data
* Perform data transformations more naturally

If you’re doing analytics or data-heavy processing in .NET, this removes a lot of friction.

[Deedle 5.0.0](https://www.nuget.org/packages/Deedle/)
*Package Description*

---

### 6. Snappier
Compression is one of those things you don’t think about until performance becomes a problem.
**Snappier** is a high-performance implementation of Google’s Snappy algorithm in C#.
It’s designed for speed, not maximum compression ratio.

Which makes it ideal for:
* Network transport
* Logging pipelines
* High-throughput systems

You get near-native performance without leaving the .NET ecosystem.

[Snappier 1.3.0](https://www.nuget.org/packages/Snappier/)
*A near-C++ performance implementation of the Snappy compression algorithm for .NET. Snappier is ported to C# directly…*

---

### 7. sly
Parsing text properly is harder than it looks.
At some point, regex just stops being enough.

**sly** sits in an interesting middle ground between parser generators and combinators. It gives you structure without forcing you into overly complex tooling like ANTLR.

Great for:
* DSLs
* Expression parsing
* Structured input processing

[sly 3.7.7](https://www.nuget.org/packages/sly/)
*SLY is a parser generator halfway between parser combinators and parser generator like ANTLR*

---

### 8. Superpower
If you prefer a more composable approach to parsing, **Superpower** is worth looking at.
It’s a parser combinator library that lets you build parsers in a very declarative way.

Compared to traditional parsing approaches, this feels:
* More readable
* Easier to test
* Easier to evolve

Especially useful when parsing logic grows over time.

[Superpower 3.1.0](https://www.nuget.org/packages/Superpower/)
*A parser combinator library for C#*

---

### 9. NaturalCron
Cron expressions are powerful… but not exactly readable.
**NaturalCron** lets you define schedules using expressions that are much closer to natural language.

Instead of cryptic strings, you get something you can actually understand at a glance. That alone reduces mistakes significantly.

[NaturalCron 1.0.0](https://www.nuget.org/packages/NaturalCron/)
*NaturalCron is a C# library for scheduling recurring events using easy-to-read, natural language-inspired expressions…*

---

### 10. Occurify
Time-based logic gets complicated fast.
Recurring events, filters, edge cases around time windows, it all adds up.

**Occurify** provides a structured way to:
* Define time periods
* Filter occurrences
* Transform schedules

It’s one of those libraries you don’t think you need, until you try to build this yourself.

[Occurify 0.10.0](https://www.nuget.org/packages/Occurify/)
*A comprehensive and intuitive .NET library for defining, filtering, transforming, and scheduling time periods.*

---

### 11. Facet
If you’re already leaning into source generators, **Facet** fits right in.
It focuses on generating projections and model transformations at compile time.

That means:
* Less manual mapping
* Cleaner models
* Better performance than runtime approaches

It pairs well with architectures that separate concerns aggressively.

[Facet 6.0.0](https://www.nuget.org/packages/Facet/)
*A Roslyn source generator for models and projections.*

---

### 12. OctaneEngineCore
Downloading large files efficiently isn’t trivial.
**OctaneEngineCore** splits downloads into chunks and processes them asynchronously.

The result:
* Faster downloads
* Better resource utilization
* More control over the process

This is especially useful for high-performance or large-scale file operations.

[OctaneEngineCore 8.5.1](https://www.nuget.org/packages/OctaneEngineCore/)
*A high Performance C# file downloader that asyncrounously downloads files as pieces. Made as a faster, more efficent…*

---

### 13. FakeItEasy
Mocking libraries can get complicated fast.
**FakeItEasy** takes a different approach; it keeps things simple.

You don’t need to learn a complex API to start writing tests. It just works.
For teams that want:
* Readable tests
* Minimal setup
* Less friction

This is a solid choice.

[FakeItEasy 9.0.1](https://www.nuget.org/packages/FakeItEasy/)
*It’s faking amazing! The easy mocking library for .NET that works great in C# and VB.NET alike. No need to know the…*

---

### 14. ScottPlot
Visualization in .NET is often overlooked.
**ScottPlot** makes it easy to generate:
* Charts
* Graphs
* Data visualizations

It’s lightweight, fast, and doesn’t require a massive setup. Perfect for dashboards, diagnostics, or internal tooling.

[ScottPlot 5.1.58](https://www.nuget.org/packages/ScottPlot/)
*ScottPlot is a free and open-source plotting library for .NET. This package can be used to create static plots, and…*

---

### 15. GenFu
Test data generation is another silent time sink.
**GenFu** helps you generate realistic data quickly.

Instead of manually crafting test objects, you can:
* Auto-generate meaningful data
* Reduce boilerplate in tests
* Focus on actual test logic

It’s simple, but incredibly useful.

[GenFu 1.6.0](https://www.nuget.org/packages/GenFu/)
*GenFu is a library you can use to generate realistic test data.*

---

### Conclusion
Most developers don’t have a tooling problem.
They have a **discovery problem.**

The .NET ecosystem already has solutions for a lot of the repetitive work we deal with every day, but they’re often hidden under the surface.

The right libraries don’t just save you time. They remove entire categories of code you shouldn’t be writing in the first place. And once you start using them, it’s hard to go back.