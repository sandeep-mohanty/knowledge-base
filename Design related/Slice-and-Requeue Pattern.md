# Slice-and-Requeue Pattern: The Cleanest Way to Process Massive Workloads in .NET
**Build fault-tolerant, scalable job pipelines using queues, idempotency, and smart retries.**

Learn how to build resilient, fault-tolerant background jobs in .NET using the Slice-and-Requeue strategy, ensuring your system handles massive workloads without memory pressure or data loss.

Every .NET developer has been there: You queue a background job to process 100,000 records, it runs for three hours, and then fails at record 99,999. Now what? Do you retry the whole thing, waste expensive compute, and risk data duplication?

This is exactly where most queue-based systems break down. In this guide, I’ll show you how the **Slice-and-Requeue** strategy transforms fragile background jobs into resilient, scalable pipelines with real C# examples you can apply immediately.

### The High Cost of “Big Batch” processing
Most background tasks start with a simple loop. It looks clean, passes code review, and works perfectly in your local development environment.

C#
```csharp
// ❌ The Anti-Pattern: The "Big Loop"
public async Task ProcessAllAsync() 
{
    // Loading 100k records into memory is a recipe for disaster
    var records = await _repo.GetAllAsync(); 
    
    foreach (var record in records) 
    {
        await ProcessRecord(record); 
    } 
}
```

**Why this fails in Production:**
* **Memory Pressure:** Loading massive datasets causes Garbage Collection (GC) spikes and `OutOfMemory` exceptions.
* **Timeouts:** Most orchestrators (like Azure Functions or AWS Lambda) have execution limits.
* **The “All-or-Nothing” Trap:** A single transient error (like a 503 from an external API) crashes the entire job, forcing a full restart.

⚠️ **Pro Tip:** If your background job takes more than 5 minutes to run, you aren’t writing a “task” — you’re building a bottleneck.

---

### What is the Slice-and-Requeue Pattern?
Instead of attempting to swallow the whole whale, we process one bite at a time. The logic is simple: **Process a small slice, then put the rest back on the queue.**

**The Conceptual Flow:**
1.  **Receive** a command containing the full list of IDs (or a pointer to them).
2.  **Take** a small “slice” (e.g., 100 items).
3.  **Process** only those 100 items.
4.  **Requeue** a new message with the remaining IDs or an updated offset.

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*0EDYiC4sz-dfYI5Zr2THRQ.png)
---

### Implementation: C# in Action
Let’s look at how to implement this using a modern, record-based approach in .NET.

#### Step 1: Define the Command
We need to track our progress. Using an `Offset` and `BatchSize` allows us to move through the data systematically.

C#
```csharp
public record ProcessSliceCommand(
    List<int> ItemIds, 
    int Offset = 0, 
    int BatchSize = 100
);
```

#### Step 2: The Resilient Handler
This handler processes its specific chunk and intelligently decides if more work is needed.

C#
```csharp
public async Task Handle(ProcessSliceCommand cmd) 
{
    // 1. Extract the current slice
    var slice = cmd.ItemIds
        .Skip(cmd.Offset)
        .Take(cmd.BatchSize)
        .ToList();

    // 2. Process items (ensure Process is idempotent!)
    foreach (var id in slice) 
    {
        await _service.Process(id); 
    }
    
    // 3. Check if there is remaining work
    var nextOffset = cmd.Offset + cmd.BatchSize;
    if (nextOffset < cmd.ItemIds.Count) 
    {
        // 4. Requeue the next slice
        var nextCommand = cmd with { Offset = nextOffset };
        await _queue.PublishAsync(nextCommand);
    }
}
```

---

### Performance Comparison: Why It Wins
![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*C4kGrfbNse65jH66kugu9A.png)
---

### Production-Grade Enhancements
To make this truly “Senior-Level” code, you need to account for the realities of distributed systems.

#### 1. Idempotency is Non-Negotiable
Since messages can be delivered more than once, your processing logic must be safe to run twice.

C#
```csharp
if (await _processedRepo.Exists(id)) return;
await _service.Execute(id);
await _processedRepo.MarkProcessed(id);
```

#### 2. Distributed Locking
If you are worried about multiple workers picking up the same large job simultaneously, use a distributed lock (via Redis or SQL) based on the `OriginId` of the master job.

#### 3. Observability
Don’t let your slices vanish into the void. Use structured logging (like Serilog) to track progress:
`Log.Information("Processed slice {Offset} of {Total} for Job {JobId}", cmd.Offset, cmd.ItemIds.Count, cmd.JobId);`

---

### Conclusion
The goal of a resilient .NET system isn’t to build a job that **never** fails — it’s to build a job that is **safe to fail**. By slicing your workloads and using the queue as your state machine, you eliminate timeouts, reduce memory pressure, and make your system infinitely more scalable.

How are you currently handling massive background jobs? Are you retrying everything from scratch, or are you slicing intelligently? Let’s discuss in the comments.