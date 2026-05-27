<!--
SYNC IMPACT REPORT
==================
Version change: (none — initial adoption) → 1.0.0
Modified principles: N/A (first ratification)
Added sections: Core Principles, Technology Constraints, Development Workflow, Governance
Removed sections: N/A
Templates updated:
  - .specify/templates/plan-template.md ✅ (Constitution Check gates aligned)
  - .specify/templates/spec-template.md ✅ (security & role-based scope validated)
  - .specify/templates/tasks-template.md ✅ (task phase structure confirmed compatible)
Deferred TODOs: none
-->

# ContosoDashboard Constitution

## Core Principles

### I. Service-First Architecture (NON-NEGOTIABLE)

All business logic MUST live in `Services/`. Razor components and pages MUST NOT contain
business logic — they are presentation-only.

- Every service MUST declare a matching interface (e.g., `IUserService` → `UserService`).
- Services MUST be registered as **scoped** via `AddScoped<IInterface, Implementation>()` in
  `Program.cs`.
- Services receive `ApplicationDbContext` via constructor injection — no service locator pattern.
- Rationale: enforces testability, separation of concerns, and enables mock injection in future
  test projects.

### II. Security-by-Design (NON-NEGOTIABLE)

Every feature MUST satisfy all of the following before implementation is considered complete:

- Authorization checks MUST be enforced at three layers: middleware, page/component attribute,
  and service method.
- Files MUST be stored outside `wwwroot`; protected resources MUST be served through controller
  endpoints with authorization checks — never as static files.
- User-supplied filenames MUST never be used in file paths; GUID-based names are mandatory.
- File extension allowlisting MUST be validated server-side before any write operation.
- Internal exception details MUST NOT be exposed to the UI; errors MUST be logged server-side.
- Secrets and connection strings MUST use `appsettings.json` + environment variables — never
  hardcoded.
- Rationale: OWASP Top 10 compliance; training code must still model real security patterns.

### III. Role-Based Authorization (NON-NEGOTIABLE)

All pages and services MUST enforce the four-tier hierarchical role model:

| Role | Permissions |
|---|---|
| `Employee` | Own tasks and assigned projects |
| `TeamLead` | Team member tasks + Employee permissions |
| `ProjectManager` | All project management + TeamLead permissions |
| `Administrator` | Full access including audit operations |

- Policies MUST be registered in `Program.cs` using `RequireRole()` with explicit role lists.
- Components MUST use `<AuthorizeView>` or inject `AuthenticationStateProvider` — never trust
  client-supplied identity claims directly.
- Rationale: prevents privilege escalation and IDOR vulnerabilities.

### IV. Data Integrity via EF Core

All data access MUST go through `ApplicationDbContext`. Raw SQL strings are forbidden.

- Relationships and indexes MUST be configured in `OnModelCreating` — not in model annotations
  alone.
- Foreign key relationships where cascading deletes are undesirable MUST use
  `DeleteBehavior.Restrict`.
- Primary keys MUST follow the `{ModelName}Id` convention (e.g., `UserId`, `ProjectId`).
- Production code MUST use EF migrations; `EnsureCreated()` is allowed only in development.
- Rationale: prevents silent data loss, ensures schema integrity across environments.

### V. Simplicity (YAGNI)

This is a training project — over-engineering is a defect.

- Implement only what is required by the current feature specification. Do not add speculative
  abstractions.
- Do not add docstrings, comments, or type annotations to code that was not changed in the
  current task.
- Do not create helper utilities or abstractions for one-time operations.
- Prefer editing existing files over creating new ones whenever the existing structure supports
  the change.
- Rationale: maximizes learner clarity; complexity must be pedagogically justified.

## Technology Constraints

The technology stack is fixed for this project and MUST NOT be changed without a constitutional
amendment:

| Layer | Technology |
|---|---|
| Framework | ASP.NET Core 8 + Blazor Server |
| UI | Razor Components (`.razor`), Razor Pages (`.cshtml`) |
| ORM | Entity Framework Core 8 (SQLite) |
| Auth | Cookie authentication (`Microsoft.AspNetCore.Authentication.Cookies`) |
| Styling | Bootstrap 5, custom CSS (`wwwroot/css/site.css`) |

- New NuGet packages MUST be justified against an existing capability gap before being added.
- Azure Blob Storage integration is planned; new file-storage code MUST implement
  `IFileStorageService` to enable future migration without breaking changes.
- The project is intentionally offline-capable and requires no external service dependencies.

## Development Workflow

All feature work MUST follow this sequence:

1. `speckit.specify` — create or update the feature spec in `/specs/<###-feature-name>/spec.md`
2. `speckit.clarify` — resolve ambiguities before planning
3. `speckit.plan` — produce design artifacts (research, data model, contracts)
4. `speckit.tasks` — generate dependency-ordered task list
5. `speckit.implement` — execute tasks

**Shell commands** (run from workspace root):

```bash
cd ContosoDashboard && dotnet build
dotnet run --project ContosoDashboard
dotnet ef migrations add <MigrationName> --project ContosoDashboard
dotnet ef database update --project ContosoDashboard
```

**Constitution Check gates** (enforced in `plan-template.md`):

- [ ] Every new service has a matching interface
- [ ] Authorization enforced at middleware + page + service (Principle II & III)
- [ ] No raw SQL introduced (Principle IV)
- [ ] No features beyond the spec scope (Principle V)
- [ ] File storage uses `IFileStorageService` pattern if files are involved (Principle II)

## Governance

This constitution supersedes all other coding conventions and practices for ContosoDashboard.

- **Amendments** require: (1) documenting the change with rationale, (2) bumping the version
  per semantic versioning rules, (3) propagating updates to all dependent templates.
- **Versioning policy**:
  - MAJOR: removal or redefinition of a non-negotiable principle
  - MINOR: new principle or section added
  - PATCH: clarifications, wording, or non-semantic refinements
- **Compliance review**: every PR description MUST include a "Constitution Check" checklist
  confirming compliance with all five principles.
- **Training context**: this project is for educational purposes only. Production deployments
  require proper identity providers, password hashing, MFA, TLS, and audit logging beyond
  what is implemented here.
- Runtime guidance is documented in `.github/copilot-instructions.md`.

**Version**: 1.0.0 | **Ratified**: 2026-05-27 | **Last Amended**: 2026-05-27
