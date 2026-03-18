# Adding Scalar or Swagger UI to ASP.NET Core (.NET 6 to .NET 10)

This guide shows the simplest way to add **Swagger UI** or **Scalar** to an ASP.NET Core Web API across **.NET 6, 7, 8, 9, and 10**.

## Quick version matrix

| Target framework | Recommended OpenAPI setup | Swagger UI | Scalar |
|---|---|---|---|
| .NET 6 | Swashbuckle | Yes | Not recommended with current `Scalar.AspNetCore` |
| .NET 7 | Swashbuckle | Yes | Not recommended with current `Scalar.AspNetCore` |
| .NET 8 | Built-in OpenAPI or Swashbuckle | Yes | Yes |
| .NET 9 | Built-in OpenAPI | Yes | Yes |
| .NET 10 | Built-in OpenAPI | Yes | Yes |

## Important note about Scalar support

The current `Scalar.AspNetCore` package targets **.NET 8 and above**. That means:

- Use **Swagger UI** for **.NET 6 and .NET 7**.
- Use **Scalar** for **.NET 8, .NET 9, and .NET 10**.

---

## Option 1: Swagger UI

## .NET 6 and .NET 7

For .NET 6 and .NET 7, the standard approach is **Swashbuckle**.

### Install package

```bash
dotnet add package Swashbuckle.AspNetCore
```

### Program.cs

```csharp
using Microsoft.OpenApi.Models;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "My API",
        Version = "v1"
    });
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.MapControllers();
app.Run();
```

### URLs

- Swagger UI: `/swagger`
- Swagger JSON: `/swagger/v1/swagger.json`

---

## .NET 8, .NET 9, and .NET 10

For newer ASP.NET Core versions, you can use the built-in OpenAPI support and add the Swagger UI package on top.

### Install packages

```bash
dotnet add package Microsoft.AspNetCore.OpenApi
dotnet add package Swashbuckle.AspNetCore.SwaggerUI
```

### Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOpenApi();
builder.Services.AddControllers();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();

    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/openapi/v1.json", "v1");
    });
}

app.MapControllers();
app.Run();
```

### URLs

- Swagger UI: `/swagger`
- OpenAPI JSON: `/openapi/v1.json`

> `MapOpenApi()` exposes the document at `/openapi/{documentName}.json` by default, so the default `v1` document is available at `/openapi/v1.json`.

---

## Option 2: Scalar

## .NET 8, .NET 9, and .NET 10

Scalar is the modern UI option and works well with the ASP.NET Core OpenAPI endpoint.

### Install packages

```bash
dotnet add package Microsoft.AspNetCore.OpenApi
dotnet add package Scalar.AspNetCore
```

### Program.cs

```csharp
using Scalar.AspNetCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOpenApi();
builder.Services.AddControllers();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference();
}

app.MapControllers();
app.Run();
```

### URLs

- Scalar UI: `/scalar`
- OpenAPI JSON: `/openapi/v1.json`

---

## Can I use Scalar on .NET 6 or .NET 7?

Not with the current `Scalar.AspNetCore` package line.

If your project targets **.NET 6** or **.NET 7**, the safest recommendation is:

1. Use **Swagger UI** now.
2. Upgrade the API to **.NET 8+** if you want first-party-style Scalar integration.

---

## Controller-based APIs vs Minimal APIs

The examples above work for both common API styles:

- **Controller-based APIs**: keep `builder.Services.AddControllers()` and `app.MapControllers()`.
- **Minimal APIs**: you can remove controller registration and map endpoints directly with `app.MapGet`, `app.MapPost`, and so on.

Example minimal API with Scalar:

```csharp
using Scalar.AspNetCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOpenApi();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference();
}

app.MapGet("/hello", () => Results.Ok(new { message = "Hello" }));

app.Run();
```

---

## Customizing the route

### Change the OpenAPI JSON route

```csharp
app.MapOpenApi("/openapi/{documentName}/openapi.json");
```

Then your `v1` document becomes:

```text
/openapi/v1/openapi.json
```

If you change the OpenAPI route, also update Swagger UI or any custom UI configuration to point to the new JSON endpoint.

---

## Security recommendation

Keep interactive API documentation UIs enabled only in development unless you intentionally want them public.

Typical pattern:

```csharp
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference();
}
```

or:

```csharp
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/openapi/v1.json", "v1");
    });
}
```

---

## Which one should you choose?

### Choose Swagger UI if:

- You target **.NET 6 or .NET 7**.
- You already use **Swashbuckle**.
- You want the most common and widely documented setup.

### Choose Scalar if:

- You target **.NET 8+**.
- You want a cleaner, more modern API reference UI.
- You already use ASP.NET Core's built-in OpenAPI support.

---

## Recommended defaults

### Best default for .NET 6 and .NET 7

Use **Swashbuckle**:

```bash
dotnet add package Swashbuckle.AspNetCore
```

### Best default for .NET 8, .NET 9, and .NET 10

Use built-in OpenAPI plus **Scalar**:

```bash
dotnet add package Microsoft.AspNetCore.OpenApi
dotnet add package Scalar.AspNetCore
```

---

## References

- [ASP.NET Core OpenAPI document generation](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/aspnetcore-openapi?view=aspnetcore-10.0)
- [Use generated OpenAPI documents in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/using-openapi-documents?view=aspnetcore-10.0)
- [Swashbuckle and ASP.NET Core (.NET versions earlier than 9)](https://learn.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-8.0)
- [Scalar.AspNetCore on NuGet](https://www.nuget.org/packages/Scalar.AspNetCore)

