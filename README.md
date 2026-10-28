# CampusCare

CampusCare is an ASP.NET Core MVC application for campus complaints, improvement suggestions, status tracking, and administrator reporting. It uses ASP.NET Core Identity, Entity Framework Core, and SQL Server.

## Prerequisites

- .NET 8 SDK
- SQL Server LocalDB, SQL Server Express, or another SQL Server instance
- Optional: `dotnet-ef` (`dotnet tool install --global dotnet-ef --version 8.0.11`)

## Configure SQL Server

The default connection string in `appsettings.json` uses Windows LocalDB. Update `ConnectionStrings:DefaultConnection` for another instance. Keep production credentials in user secrets or environment variables, not in source control. For example:

```powershell
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=YOUR_SERVER;Database=CampusCare;Trusted_Connection=True;TrustServerCertificate=True"
```

For deployments using SQL authentication, store the password in a secret provider and use an encrypted connection. Do not commit credentials.

## Create and apply the initial migration

From the directory containing `CampusCare.csproj`:

```powershell
dotnet restore
dotnet tool install --global dotnet-ef --version 8.0.11
dotnet ef migrations add InitialCreate
dotnet ef database update
dotnet run
```

On a fresh local database, the app can also create its schema with `EnsureCreated` when no migrations exist. Once you add `InitialCreate`, startup applies migrations. If you already launched once using `EnsureCreated`, point the migration workflow at a new empty database (or remove the local demo database) before running `dotnet ef database update`, because `EnsureCreated` does not create EF migration history. For later schema changes, create a new migration and apply it with `dotnet ef migrations add <MigrationName>` and `dotnet ef database update`.

Browse to the HTTPS URL printed by `dotnet run`.

## Development demo accounts

When `ASPNETCORE_ENVIRONMENT=Development`, the startup seed creates clearly marked demo identities and one sample complaint.

| Role | Email | Demo password |
| --- | --- | --- |
| Administrator | `info@iare.ac.in` | `iare2026` |
| Student | `student@iare.ac.in` | `studentiare` |
| Faculty | `faculty@campuscare.demo` | `CampusCare!2026` |
| College staff | `staff@campuscare.demo` | `CampusCare!2026` |

These accounts are for local demonstrations only. Seeding of demo identities is disabled outside Development. Change or remove them before using a shared development database. The static localhost preview has separate role-selected demo credentials (Student: `student@iare.ac.in` / `studentiare`; Administrator: `info@iare.ac.in` / `iare2026`) and stores its demo activity in browser storage, not SQL Server.

The Administrator and Student demo passwords are hashed with ASP.NET Core Identity's password hasher and seeded only in Development. The application's normal password rules remain unchanged; these short demo credentials are for the isolated local demonstration only.

## Main flows

- Register as a student; administrators can assign verified accounts to faculty and college staff roles. Sign in and sign out.
- Submit and track complaints, attach optional JPG/PNG/WebP photos (5 MB maximum), comment, and submit improvement ideas.
- Administrators search and filter reports, approve or mark complaints not approved, manage user roles, and export a CSV report. An approved complaint enters the Under Review tracking status; the submitter receives an in-app notification for either decision.
- Status updates create a timeline entry and an in-app notification for the reporter. Resolved complaints can receive feedback.

Uploaded images are stored outside the public web root in `App_Data/uploads` using generated filenames and are served only after complaint ownership checks. Back up this directory along with the SQL database.

## Security notes

Password storage and sign-in use ASP.NET Core Identity. MVC POST forms use antiforgery tokens, administrator endpoints require the Administrator role, complaint details and comments enforce reporter ownership, and image uploads check size, declared type, and file signature. Configure HTTPS, secret storage, database backups, and upload retention for the deployment environment.
