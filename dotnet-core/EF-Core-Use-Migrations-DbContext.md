# Entity Framework Setup and Usage Guide for .NET 6 to .NET 10

This guide explains how to set up and use **Entity Framework Core (EF Core)** in ASP.NET Core or general .NET applications targeting **.NET 6 through .NET 10**.

It covers:
- what EF Core is
- which packages to install
- how to configure `DbContext`
- how to create entities
- how to run migrations
- how to query and save data
- how to scaffold from an existing database
- practical recommendations for real projects

---

## 1. What Entity Framework Core is

Entity Framework Core is Microsoft’s cross-platform object-relational mapper (ORM) for .NET. It lets you work with databases using C# classes and LINQ instead of writing most SQL by hand.

Common EF Core capabilities include:
- mapping C# classes to database tables
- querying with LINQ
- tracking changes
- inserting, updating, and deleting data
- managing schema changes with migrations

References:
- EF Core overview: <https://learn.microsoft.com/en-us/ef/core/>
- EF docs hub: <https://learn.microsoft.com/en-us/ef/>

---

## 2. Version guidance for .NET 6 to .NET 10

For older projects, the simplest rule is:

- **.NET 6** → usually use **EF Core 6**
- **.NET 7** → usually use **EF Core 7**
- **.NET 8** → use **EF Core 8** or **EF Core 9** depending on your upgrade strategy
- **.NET 9** → use **EF Core 9**
- **.NET 10** → use **EF Core 10**

Important notes:
- Microsoft states that **EF Core 8 and EF Core 9 both target .NET 8**, which means EF Core 9 can be used by apps targeting .NET 8 or .NET 9.
- If you are working in a **.NET 10** project, use the EF Core 10 package line.
- In general, keep all EF Core packages on the **same major version**.

References:
- Supported .NET implementations: <https://learn.microsoft.com/en-us/ef/core/miscellaneous/platforms>
- EF Core 9 target framework note: <https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-9.0/whatsnew>
- EF Core 10 breaking changes / upgrade path: <https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-10.0/breaking-changes>

---

## 3. Install the required packages

EF Core is delivered through NuGet packages. At minimum, you install:

1. the **provider** package for your database
2. usually the **Design** package for migrations and scaffolding

### Common packages

#### SQL Server
```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
```

#### SQLite
```bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Microsoft.EntityFrameworkCore.Design
```

#### PostgreSQL
```bash
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Microsoft.EntityFrameworkCore.Design
```

> Replace package versions with the EF Core major version that matches your project strategy, for example `6.*`, `8.*`, `9.*`, or `10.*`.

References:
- Installing EF Core: <https://learn.microsoft.com/en-us/ef/core/get-started/overview/install>
- EF Core NuGet packages: <https://learn.microsoft.com/en-us/ef/core/what-is-new/nuget-packages>
- Database providers: <https://learn.microsoft.com/en-us/ef/core/providers/>

---

## 4. Install the EF Core CLI tools

To create and apply migrations from the command line, install the EF Core CLI tool.

### Global tool
```bash
dotnet tool install --global dotnet-ef
```

To update it later:
```bash
dotnet tool update --global dotnet-ef
```

You can verify installation with:
```bash
dotnet ef
```

References:
- EF Core CLI tools: <https://learn.microsoft.com/en-us/ef/core/cli/dotnet>
- EF Core tools overview: <https://learn.microsoft.com/en-us/ef/core/cli/>

---

## 5. Define your entity classes

Entities are normal C# classes that represent database tables.

Example:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public bool IsActive { get; set; } = true;
}
```

By convention:
- `Id` becomes the primary key
- the class name usually maps to a table
- scalar properties map to columns

Reference:
- Modeling in EF Core: <https://learn.microsoft.com/en-us/ef/core/modeling/>

---

Use `OnModelCreating` for Fluent API configuration when conventions are not enough.

References:
- DbContext configuration: <https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/>
- Model configuration: <https://learn.microsoft.com/en-us/ef/core/modeling/>

---

## 6. Add your connection string

In `appsettings.json`:

### SQL Server
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyAppDb;Trusted_Connection=True;TrustServerCertificate=True"
  }
}
```

### SQLite
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=myapp.db"
  }
}
```

### PostgreSQL
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Database=MyAppDb;Username=postgres;Password=yourpassword"
  }
}
```

For production, do not hardcode secrets in source control. Prefer:
- environment variables
- user secrets for local development
- a secret manager such as Azure Key Vault

---

## 7. Create your first migration

Once your entity classes and `DbContext` are ready, create a migration.

```bash
dotnet ef migrations add InitialCreate
```

This generates migration files that describe how to create the database schema.

If your startup project and data project are different, use:

```bash
dotnet ef migrations add InitialCreate --project MyApp.Data --startup-project MyApp.Api
```

Reference:
- EF Core CLI tools: <https://learn.microsoft.com/en-us/ef/core/cli/dotnet>

---

## 8. Apply the migration to the database

```bash
dotnet ef database update
```

This applies pending migrations and creates or updates the database.

Reference:
- Applying migrations: <https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying>

---

## 9. Basic CRUD usage

You normally inject `AppDbContext` into services, endpoints, or controllers.

### Insert data

```csharp
app.MapPost("/products", async (AppDbContext db, Product product) =>
{
    db.Products.Add(product);
    await db.SaveChangesAsync();
    return Results.Created($"/products/{product.Id}", product);
});
```

### Read data

```csharp
app.MapGet("/products", async (AppDbContext db) =>
    await db.Products
        .Where(p => p.IsActive)
        .OrderBy(p => p.Name)
        .ToListAsync());
```

### Read one row

```csharp
app.MapGet("/products/{id:int}", async (AppDbContext db, int id) =>
{
    var product = await db.Products.FindAsync(id);
    return product is null ? Results.NotFound() : Results.Ok(product);
});
```

### Update data

```csharp
app.MapPut("/products/{id:int}", async (AppDbContext db, int id, Product input) =>
{
    var product = await db.Products.FindAsync(id);
    if (product is null) return Results.NotFound();

    product.Name = input.Name;
    product.Price = input.Price;
    product.IsActive = input.IsActive;

    await db.SaveChangesAsync();
    return Results.NoContent();
});
```

### Delete data

```csharp
app.MapDelete("/products/{id:int}", async (AppDbContext db, int id) =>
{
    var product = await db.Products.FindAsync(id);
    if (product is null) return Results.NotFound();

    db.Products.Remove(product);
    await db.SaveChangesAsync();
    return Results.NoContent();
});
```

---

## 10. Querying tips

EF Core uses LINQ for most queries.

Example:

```csharp
var products = await db.Products
    .AsNoTracking()
    .Where(p => p.Price > 100)
    .OrderByDescending(p => p.Price)
    .ToListAsync();
```

Useful practices:
- use `AsNoTracking()` for read-only queries
- project only the columns you need with `Select(...)`
- avoid loading huge object graphs unnecessarily
- use async methods like `ToListAsync`, `FirstOrDefaultAsync`, and `SaveChangesAsync`

Reference:
- EF Core overview and querying guidance: <https://learn.microsoft.com/en-us/ef/core/>

---

## 11. Relationships example

Example one-to-many relationship:

```csharp
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public List<Product> Products { get; set; } = new();
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;

    public int CategoryId { get; set; }
    public Category Category { get; set; } = null!;
}
```

Optional Fluent API configuration:

```csharp
modelBuilder.Entity<Product>()
    .HasOne(p => p.Category)
    .WithMany(c => c.Products)
    .HasForeignKey(p => p.CategoryId);
```

Reference:
- Modeling: <https://learn.microsoft.com/en-us/ef/core/modeling/>

---

## 12. Scaffolding from an existing database

If the database already exists, EF Core can generate the `DbContext` and entity classes.

### SQL Server example
```bash
dotnet ef dbcontext scaffold \
  "Server=localhost;Database=MyAppDb;Trusted_Connection=True;TrustServerCertificate=True" \
  Microsoft.EntityFrameworkCore.SqlServer \
  --output-dir Models \
  --context AppDbContext
```

### PostgreSQL example
```bash
dotnet ef dbcontext scaffold \
  "Host=localhost;Database=MyAppDb;Username=postgres;Password=yourpassword" \
  Npgsql.EntityFrameworkCore.PostgreSQL \
  --output-dir Models \
  --context AppDbContext
```

Reference:
- Scaffolding / reverse engineering: <https://learn.microsoft.com/en-us/ef/core/managing-schemas/scaffolding/>

---

## 13. Recommended project structure

A clean layout for medium or large projects is:

```text
MyApp.Api/
MyApp.Application/
MyApp.Domain/
MyApp.Infrastructure/
```

Typical EF placement:
- entities in `Domain` or `Infrastructure`
- `DbContext` in `Infrastructure`
- migrations usually in the same project as `DbContext`

If your `DbContext` lives in a class library, remember that `dotnet ef` may need a startup project because design-time tools must execute application code.

Reference:
- EF Core CLI design-time behavior: <https://learn.microsoft.com/en-us/ef/core/cli/dotnet>


## 14. Production recommendations

For production systems:

- avoid calling `EnsureCreated()` for apps that will use migrations long term
- prefer **migrations** for schema evolution
- review generated migration SQL before production rollout
- keep provider and EF Core package versions aligned
- test upgrades carefully because EF Core major versions can include breaking changes
- store secrets outside source control
- use explicit projections for performance-sensitive endpoints

Microsoft notes that applying schema changes to production databases can be risky, and generating SQL scripts is often safer because they can be reviewed and archived before execution.

Reference:
- Applying migrations in production: <https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying>
- Breaking changes in EF Core 8: <https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-8.0/breaking-changes>
- Breaking changes in EF Core 9: <https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-9.0/breaking-changes>
- Breaking changes in EF Core 10: <https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-10.0/breaking-changes>

---

## 15. Minimal end-to-end example

### `Product.cs`
```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
}
```

### `AppDbContext.cs`
```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products => Set<Product>();
}
```

### `Program.cs`
```csharp
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

app.MapGet("/products", async (AppDbContext db) =>
    await db.Products.AsNoTracking().ToListAsync());

app.MapPost("/products", async (AppDbContext db, Product product) =>
{
    db.Products.Add(product);
    await db.SaveChangesAsync();
    return Results.Created($"/products/{product.Id}", product);
});

app.Run();
```

### Commands
```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---
---

## Migrations & Db Update commands

```bash
# Add a migration
dotnet ef migrations add MigrationName

# (create and) Apply migrations to the database
dotnet ef database update

# Rollback to a previous migration
dotnet ef database update PreviousMigrationName

# Rollback all migrations (Empty database)
dotnet ef database update 0

# Remove last(newest) migration
dotnet ef migrations remove

# Remove all migrations (rollback all & remove)
dotnet ef database update 0
dotnet ef migrations remove

# List migrations
dotnet ef migrations list

# Generate SQL script
dotnet ef migrations script

```

---
---

## Db Context - Guide to Create and Register

### Create your DbContext

`DbContext` is the main EF Core class used to access the database.

```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<Product> Products => Set<Product>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<Product>(entity =>
        {
            entity.Property(p => p.Name)
                  .HasMaxLength(200)
                  .IsRequired();

            entity.Property(p => p.Price)
                  .HasColumnType("decimal(18,2)");
        });
    }
}
```

Use `OnModelCreating` for Fluent API configuration when conventions are not enough.


### Register DbContext in `Program.cs`

SQL Server example

```csharp
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddControllers();

var app = builder.Build();

app.MapControllers();
app.Run();
```



