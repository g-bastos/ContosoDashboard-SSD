# Copilot Instructions — ContosoDashboard

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
<!-- SPECKIT END -->

## Project Overview

ContosoDashboard is a **Blazor Server** application built with **.NET 8** for Contoso Corporation. It is an internal employee dashboard for managing tasks, projects, notifications, and team information. The app uses **Entity Framework Core** with **SQLite** as the database and **cookie-based authentication** (mock, for training purposes).

## Technology Stack

| Layer | Technology |
|---|---|
| Framework | ASP.NET Core 8 + Blazor Server |
| UI | Razor Components (`.razor`), Razor Pages (`.cshtml`) |
| ORM | Entity Framework Core 8 (SQLite) |
| Auth | Cookie authentication (`Microsoft.AspNetCore.Authentication.Cookies`) |
| Identity | Microsoft Identity Web (configured for future OpenID Connect) |
| Styling | Bootstrap 5, custom CSS (`wwwroot/css/site.css`) |

## Project Structure

```
ContosoDashboard/
├── Data/               # EF Core DbContext (ApplicationDbContext)
├── Models/             # Domain model classes
├── Pages/              # Razor Pages (.cshtml) and Blazor pages (.razor)
├── Services/           # Business logic – always injected via interfaces
├── Shared/             # Shared Blazor layout components (NavMenu, MainLayout)
└── wwwroot/            # Static assets
```

## Architecture & Coding Conventions

### Service Layer
- All business logic must live in `Services/`. Never put business logic in Razor components.
- Every service must have an interface (e.g., `IUserService` → `UserService`).
- Register services as **scoped** in `Program.cs` using `AddScoped<IFoo, Foo>()`.
- Services receive `ApplicationDbContext` via constructor injection.

### Models
- All model classes live in `ContosoDashboard.Models`.
- Use **Data Annotations** (`[Required]`, `[MaxLength]`, `[EmailAddress]`, etc.) for validation.
- Navigation properties should be marked `virtual` and initialized to empty collections.
- Primary keys follow the pattern `{ModelName}Id` (e.g., `UserId`, `ProjectId`).

### Blazor Components
- Pages that require authentication must inject `AuthenticationStateProvider` or use `<AuthorizeView>`.
- Use `@inject` for service injection in `.razor` files.
- Keep component code-behind minimal; delegate to services.
- Prefer `@code { }` blocks at the bottom of `.razor` files.

### Database
- Use `ApplicationDbContext` to access data. Never use raw SQL strings.
- Relationships and indexes are configured in `OnModelCreating` inside `ApplicationDbContext.cs`.
- In development, `context.Database.EnsureCreated()` is used. Use EF migrations for production.
- Use `DeleteBehavior.Restrict` for FK relationships where cascading deletes are undesirable.

### Authorization
- Four roles are defined: `Employee`, `TeamLead`, `ProjectManager`, `Administrator`.
- Roles are hierarchical: each higher role includes all permissions of lower ones.
- Authorization policies are registered in `Program.cs` and referenced by name on pages/components.
- Protect pages with `[Authorize(Policy = "...")]` or `<AuthorizeView Roles="...">`.

## Roles Reference

| Role | Description |
|---|---|
| `Employee` | Standard user — access to own tasks and assigned projects |
| `TeamLead` | Manages team members and their tasks |
| `ProjectManager` | Full project management access |
| `Administrator` | Full access including audit and admin operations |

## Key Services

| Interface | Implementation | Responsibility |
|---|---|---|
| `IUserService` | `UserService` | User CRUD and profile management |
| `ITaskService` | `TaskService` | Task creation, updates, comments |
| `IProjectService` | `ProjectService` | Project and member management |
| `INotificationService` | `NotificationService` | In-app notifications |
| `IDashboardService` | `DashboardService` | Dashboard summary data |

## Planned Feature: Document Upload and Management

The next feature to implement is document upload and management. Key design decisions:

- **Storage**: Files stored in `AppData/uploads/` (outside `wwwroot`) — never serve files directly from `wwwroot`.
- **Interface**: Use `IFileStorageService` with methods `UploadAsync()`, `DeleteAsync()`, `DownloadAsync()`, `GetUrlAsync()` to allow future Azure Blob Storage migration.
- **File path pattern**: `{userId}/{projectId-or-"personal"}/{guid}.{extension}` — always use GUIDs, never user-supplied filenames.
- **Upload sequence**: Generate path → Save file to disk → Save metadata to database (prevents orphaned DB records).
- **Supported types**: PDF, Word, Excel, PowerPoint, text files, JPEG, PNG. Max 25 MB per file.
- **Metadata fields**: title, description, category, associated project, tags, upload date, uploader, file size, MIME type.
- **MIME type field**: must accommodate 255 characters (Office document MIME types are long).
- **Download endpoint**: Must be a controller endpoint (not static file) to enforce authorization checks.

## Shell Commands

```bash
# Restore and build
cd ContosoDashboard && dotnet build

# Run the app
dotnet run --project ContosoDashboard

# Add EF migration
dotnet ef migrations add <MigrationName> --project ContosoDashboard

# Apply migrations
dotnet ef database update --project ContosoDashboard
```

## Security Guidelines

- Never store user-supplied filenames in file paths; always use GUID-based names.
- Validate file extensions against an explicit allowlist before saving.
- Serve protected files through controller endpoints with authorization checks.
- Do not expose internal exception details to the UI; log them server-side.
- Sensitive configuration (connection strings, secrets) must use `appsettings.json` + environment variables — never hardcode.

