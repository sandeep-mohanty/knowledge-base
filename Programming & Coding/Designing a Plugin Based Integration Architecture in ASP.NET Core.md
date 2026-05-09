# Designing a Plugin Based Integration Architecture in ASP.NET Core

When building modern SaaS platforms, integrating with external services is almost inevitable. Whether it’s **Google Drive**, **OneDrive**, or **Dropbox**, users expect seamless connectivity with the tools they already use.

At first, the implementation seems straightforward. You define an interface for drive operations and create separate implementations for each provider. Then, somewhere in your application logic, you determine which provider to execute:

```csharp
if (provider == "GoogleDrive")
{
    // Google Drive logic
}
else if (provider == "OneDrive")
{
    // OneDrive logic
}
else if (provider == "Dropbox")
{
    // Dropbox logic
}
```

It works, Until it doesn’t.

As more integrations are added, this approach quickly becomes problematic:
* The application becomes tightly coupled to specific providers.
* Every new integration requires modifying existing logic.
* Testing becomes harder.
* The codebase becomes messy with conditional branching.
* The system violates the Open/Closed Principle.

This is where a **plugin based integration architecture** becomes essential.

In this article, we’ll explore how to design a clean, extensible, and production ready plugin architecture in ASP.NET Core that allows you to add new integrations such as Google Drive, OneDrive, or Dropbox without touching the core application logic.

### Okey… How can we do this?
A plugin based integration architecture can be implemented in multiple ways, depending on the overall system design, extensibility requirements, and complexity of the integration layer.

In practice, the most common design approaches include:
1. Strategy Pattern
2. Factory Pattern
3. A combination of Strategy and Factory patterns

Each approach has its own trade offs. In this article, I will focus specifically on implementing a **Factory based plugin resolution approach in ASP.NET Core**, while still leveraging interface driven design.

---

### Let’s Dive In
Imagine there is a feature in Application X, when a user uploads a file to the application X, it should automatically be uploaded to connected drive (GoogleDrive, DropBox, OneDrive,..etc). Currently, application X has support only for **GoogleDrive** and **DropBox**.

To achieve the goal, we should mainly have:
* **DriveFactory**
* **IDriveService**
* **GoogleDriveService**
* **DropBoxService**

#### Basic DTOs and Type Enum:
Before implementing the factory layer, we need to establish a small set of core abstractions that remain independent of any specific drive provider.

```csharp
public class DriveSettings
{
    public string FolderName { get; set; }
    public string AccessToken { get; set; }
}
```
**DriveSettings** represents the minimal configuration required to interact with a drive provider. Instead of creating provider specific configuration classes inside the application core, we define a **generic configuration object** that can support any drive implementation.

```csharp
public class DriveResponse
{
    public bool Success { get; set; }
    public string Message { get; set; }
}
```
When integrating multiple external systems, one common challenge is **inconsistent response handling**. Each provider returns errors and success responses differently. To avoid leaking those inconsistencies into the application core, we define a standardized response wrapper.

```csharp
public enum DriveType
{
    GOOGLE_DRIVE,
    DROPBOX
}
```
The **DriveType** enum provides a strongly typed mechanism for identifying which drive implementation should be resolved by the factory.

#### IDriveService:
```csharp
public interface IDriveService
{
    DriveType DriveType { get; }
    DriveResponse UploadFile(Stream file, DriveSettings driveSettings);
}
```
The **IDriveService** abstraction enforces a consistent behavior across all drive implementations (Google Drive, Dropbox, OneDrive, etc.). At the heart of a plugin based architecture lies a **stable contract** that all integrations must implement.

#### GoogleDriveService:
```csharp
public class GoogleDriveService : IDriveService
{
    public DriveType DriveType => DriveType.GOOGLE_DRIVE;
    public DriveResponse UploadFile(FileStream file, DriveSettings driveSettings)
    {
        // Google Drive Specific Implementation
    }
}
```
This is where, the implementation for google drive uploading files, takes place.

#### DropBoxService:
```csharp
public class DropBoxService : IDriveService
{
    public DriveType DriveType => DriveType.DROPBOX;
    public DriveResponse UploadFile(FileStream file, DriveSettings driveSettings)
    {
        // DropBox Specific Implementation
    }
}
```
This is the class for DropBox implementation.

---

### Finally, the most important parts of plugin based architecture,

```csharp
public interface IDriveFactory
{
    IDriveService Resolve(DriveType driveType);
}

public class DriveFactory : IDriveFactory
{
    private readonly Dictionary<DriveType, IDriveService> _driveServices;

    public DriveFactory(IEnumerable<IDriveService> driveServices)
    {
        _driveServices = driveServices
            .ToDictionary(s => s.DriveType, s => s);
    }

    public IDriveService Resolve(DriveType driveType)
    {
        if (!_driveServices.TryGetValue(driveType, out var service))
            throw new NotSupportedException($"Drive type '{driveType}' is not supported.");

        return service;
    }
}
```
The **DriveFactory** is responsible for giving you the correct drive service (Google Drive, Dropbox, etc.) based on the **DriveType**.
* When the application starts, all implementations of **IDriveService** are injected into the factory.
* The constructor stores them in a **dictionary**, mapping each **DriveType** to its corresponding service.
* When **Resolve()** is called, the factory quickly looks up the correct service from the dictionary and returns it.
* If no matching service exists, it throws an error.

So instead of using `if-else` or `switch` statements, the factory cleanly and efficiently selects the correct drive implementation.

#### Final Step: Adding services in Program.cs.
```csharp
// Program.cs

builder.Services.AddScoped<IDriveService, GoogleDriveService>();
builder.Services.AddScoped<IDriveService, DropboxDriveService>();
builder.Services.AddSingleton<IDriveFactory, DriveFactory>();
```

#### Example Usage:
```csharp
app.MapPost("/upload", async (
        IFormFile file,
        DriveType driveType,
        IDriveFactory driveFactory,
        CancellationToken cancellationToken) =>
    {
        if (file == null || file.Length == 0)
            return Results.BadRequest("File is required.");

        await using var stream = file.OpenReadStream();

        var driveService = driveFactory.Resolve(driveType);

        var response = driveService.UploadFile((FileStream)stream, driveSettings);

        return response.Success
            ? Results.Ok(response)
            : Results.BadRequest(response);
    })
    .WithOpenApi();
```
In this example, a single endpoint is created to upload a file to a given drive. No need of creating separate if-else logics or separate endpoints for each drive.

---

### Architecture Diagram:
Press enter or click to view image in full size


**Plugin Architecture using Factory Pattern**

---

### How to integrate a new Drive in the Future?
Imagine you want to integrate OneDrive or any other drive to this application. When you already have the base architecture ready, all you have to do,
1. Create **OneDriveService** which implements **IDriveService**.
2. Add an enum member to **DriveType** enum.
3. Register the newly created service in **Program.cs**.

No need of any business logic change :-)

---

### What can be improved in this architecture?
In very large systems, even enum based factories can become limiting because:
* Enums require modification when adding new providers.
* Plugins may need to be loaded dynamically (via assemblies).

At that level, you evolve toward:
* Metadata based resolution.
* Attribute based mapping.
* Reflection based plugin loading.
* Modular architecture (separate integration assemblies).

But for most SaaS systems, the DI-backed factory approach provides the perfect balance between simplicity and scalability.