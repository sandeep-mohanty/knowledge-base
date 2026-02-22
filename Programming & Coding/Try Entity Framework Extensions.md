# Why Every EF Core Developer Needs to Try Entity Framework Extensions

When building data-intensive applications with Entity Framework Core, you will eventually face a critical performance challenge: inserting, updating, or deleting thousands of records efficiently.

The standard EF Core approach using `SaveChanges()` works perfectly for small datasets or even a few hundred rows, but it becomes a significant bottleneck when you need to process thousands of records.

There is a better solution: [Entity Framework Extensions](https://entityframework-extensions.net?utm_source=antondevtips&utm_medium=newsletter&utm_campaign=january-2026) provides high-performance bulk operations that can process thousands of records in less than a second. You still use the same code you have in EF Core with a new set of DbContext extension methods.

In this post, we will explore:

- Why inserting 10,000+ rows with EF Core quickly becomes a performance bottleneck  
- What happens under the hood when you call `SaveChanges()` and why it slows down at scale  
- How Entity Framework Extensions solve bulk data problems without rewriting your EF Core code  
- How to import data at scale using `BulkInsert` to insert thousands of records in a single operation  
- How to apply mass updates with `BulkUpdate` to efficiently modify large datasets  
- How to clean up data with `BulkDelete` to remove large volumes of records in one database round-trip  
- How to perform upserts and sync external systems using `BulkMerge` with minimal code  
- How to run full data synchronization jobs using `BulkSynchronize` to keep databases in sync reliably  

---

## Why Inserting 10,000+ Rows with EF Core Becomes a Performance Bottleneck

You are building an IoT platform that collects telemetry data from thousands of devices every minute. Each device sends temperature readings, humidity levels, and status updates.

Your system needs to insert 50,000 telemetry records into the database every few minutes.

You start with the standard Entity Framework Core approach:

```csharp
app.MapPost("/telemetries", async (IotDbContext dbContext, InsertTelemetryRequest request) => {
    var telemetryEntries = request.MapToTelemetryEntries();
    dbContext.Telemetries.AddRange(telemetryEntries);
    await dbContext.SaveChangesAsync();
    return Results.Ok();
});
```

---

## What Happens Under the Hood When You Call SaveChanges() and Why It's a Performance Bottleneck

For inserts, EF Core generates individual `INSERT` statements:

```sql
INSERT INTO Telemetries (TelemetryId, DeviceId, ComponentId, Value, Quality, CollectedAt, ReceivedAt)
VALUES (@p0, @p1, @p2, @p3, @p4, @p5, @p6);

INSERT INTO Telemetries (TelemetryId, DeviceId, ComponentId, Value, Quality, CollectedAt, ReceivedAt)
VALUES (@p7, @p8, @p9, @p10, @p11, @p12, @p13);

-- ... repeated 10,000 times
```

**Raw SQL Example:**

```csharp
var sql = "INSERT INTO Telemetries (TelemetryId, DeviceId, ComponentId, Value, Quality, CollectedAt, ReceivedAt) VALUES ";
var values = string.Join(", ", telemetryBatch.Select(t =>
    $"('{t.TelemetryId}', '{t.DeviceId}', '{t.ComponentId}', {t.Value}, {(int)t.Quality}, '{t.CollectedAt:yyyy-MM-dd HH:mm:ss}', '{t.ReceivedAt:yyyy-MM-dd HH:mm:ss}')"));
await dbContext.Database.ExecuteSqlRawAsync(sql + values);
```

---

## How Entity Framework Extensions Solve Bulk Data Problems Without Rewriting Your EF Core Code

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var telemetryBatch = await ReceiveTelemetryFromDevices(); // 10,000 records
await dbContext.BulkInsertAsync(telemetryBatch); // Execution time: 250ms
```

---

## Bulk Operation Examples

```bash
dotnet add package Z.EntityFramework.Extensions.EFCore
```

```csharp
using Z.EntityFramework.Extensions;

// Bulk insert
await dbContext.BulkInsertAsync(entities);

// Bulk update
await dbContext.BulkUpdateAsync(entities);

// Bulk delete
await dbContext.BulkDeleteAsync(entities);

// Bulk merge (upsert)
await dbContext.BulkMergeAsync(entities);

// Bulk synchronize
await dbContext.BulkSynchronizeAsync(entities);
```

---

## BulkInsert Example

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var telemetryBatch = await ReceiveTelemetryFromDevices(); // 10,000 records
await dbContext.BulkInsertAsync(telemetryBatch);
```

**Working with Related Entities:**

```csharp
public class Device {
    public Guid DeviceId { get; set; }
    public string Name { get; set; } = string.Empty;
    public string DeviceType { get; set; } = string.Empty;
    public string Manufacturer { get; set; } = string.Empty;
    public string SerialNumber { get; set; } = string.Empty;
    public string FirmwareVersion { get; set; } = string.Empty;
    public string HardwareVersion { get; set; } = string.Empty;
    public DeviceStatus Status { get; set; }
    public DateTime LastSeenAt { get; set; }
    public DateTime RegisteredAt { get; set; }
    public string Configuration { get; set; } = string.Empty;
}

public class Component {
    public Guid ComponentId { get; set; }
    public Guid DeviceId { get; set; }
    public ComponentType ComponentType { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Capability { get; set; } = string.Empty;
    public string Unit { get; set; } = string.Empty;
    public string StateValue { get; set; } = string.Empty;
    public ComponentState State { get; set; }
    public bool IsActive { get; set; }
    public DateTime LastUpdatedAt { get; set; }
}
```

```csharp
// Bulk insert with IncludeGraph
var devices = await ImportDevicesWithComponents(); // 5,000 devices
await dbContext.BulkInsertAsync(devices, options => options.IncludeGraph = true);
```

---

## BulkUpdate Example

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

// Receive updated device data from external system
var externalDeviceData = await FetchDeviceDataFromExternalApi(); // 10,000 devices

var devicesToUpdate = externalDeviceData.Select(ext => new Device {
    DeviceId = ext.Id,
    FirmwareVersion = ext.Firmware,
    HardwareVersion = ext.Hardware,
    Configuration = ext.Config,
    LastSeenAt = ext.LastContact
}).ToList();

await dbContext.BulkUpdateAsync(devicesToUpdate, options => {
    options.ColumnPrimaryKeyExpression = d => d.DeviceId;
    options.ColumnInputExpression = d => new {
        d.FirmwareVersion,
        d.HardwareVersion,
        d.Configuration,
        d.LastSeenAt
    };
});
```

---

## BulkDelete Example

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var deviceIdsToDelete = await GetDecommissionedDeviceIds(); // 100,000 device IDs
var devicesToDelete = deviceIdsToDelete.Select(id => new Device { DeviceId = id }).ToList();

await dbContext.BulkDeleteAsync(devicesToDelete, options => {
    options.ColumnPrimaryKeyExpression = d => d.DeviceId;
});
```

---

## BulkMerge Example

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var externalDevices = await FetchDevicesFromExternalApi(); // 10,000 devices
var devicesToMerge = externalDevices.Select(ext => new Device {
    DeviceId = ext.DeviceId,
    Name = ext.Name,
    DeviceType = ext.Type,
    Manufacturer = ext.Manufacturer,
    Status = ext.Status,
    LastSeenAt = ext.LastContact
}).ToList();

await dbContext.BulkMergeAsync(devicesToMerge);
```

---

## BulkSynchronize Example

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var externalDevices = await FetchAllDevicesFromExternalApi(); // 10,000 devices
var devicesToSync = externalDevices.Select(ext => new Device {
    DeviceId = ext.DeviceId,
    Name = ext.Name,
    DeviceType = ext.Type,
    Manufacturer = ext.Manufacturer,
    SerialNumber = ext.SerialNumber,
    Status = ext.Status,
    LastSeenAt = ext.LastContact
}).ToList();

await dbContext.BulkSynchronizeAsync(devicesToSync);
```

---

## Summary
Entity Framework Extensions provides optimized bulk operations (`BulkInsert`, `BulkUpdate`, `BulkDelete`, `BulkMerge`, `BulkSynchronize`) that solve EF Core’s performance bottlenecks when working with large datasets.
