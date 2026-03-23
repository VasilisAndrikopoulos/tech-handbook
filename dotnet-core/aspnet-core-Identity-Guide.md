# ASP.NET Core Identity Guide (.NET 6 to .NET 10)

This brief guide explains how to set up **ASP.NET Core Identity**, how it works with **Entity Framework Core**, and how to generate **scaffolded Identity UI pages** similar to the experience in **Visual Studio 2022**.

---

## 1) What ASP.NET Core Identity is

ASP.NET Core Identity provides:

- User registration and login
- Password hashing and validation
- Roles and claims
- Email confirmation and password reset
- Two-factor authentication
- External login providers
- User/profile management UI

Identity is commonly backed by **Entity Framework Core** using SQL Server.

---

## 2) Easiest way to start

The simplest starting point is to create a new project with **Individual Accounts** enabled.

### Visual Studio 2022
When creating an ASP.NET Core MVC or Razor Pages app:
- Choose **Authentication Type**
- Select **Individual Accounts**

### .NET CLI
```bash
dotnet new mvc -au Individual
```

or

```bash
dotnet new webapp -au Individual
```

This creates the base Identity setup for you.

---

## 3) Main packages commonly used

Most Identity-enabled templates already include what you need, but these are the important packages/concepts:

- `Microsoft.AspNetCore.Identity.EntityFrameworkCore`
- `Microsoft.EntityFrameworkCore.SqlServer`
- `Microsoft.EntityFrameworkCore.Tools`

For scaffolding UI with the CLI, you may also need:

- `dotnet-aspnet-codegenerator` tool

Install the tool globally:

```bash
dotnet tool install -g dotnet-aspnet-codegenerator
```

If already installed:

```bash
dotnet tool update -g dotnet-aspnet-codegenerator
```

---

## 4) Typical Identity database objects

When using the default EF Core Identity model, SQL tables commonly include:

- `AspNetUsers`
- `AspNetRoles`
- `AspNetUserRoles`
- `AspNetUserClaims`
- `AspNetUserLogins`
- `AspNetUserTokens`
- `AspNetRoleClaims`

These are created through EF Core migrations.

---

## 5) Basic setup in Program.cs

A common setup looks like this:

```csharp
...

builder.Services
    .AddDefaultIdentity<IdentityUser>(options =>
    {
        options.SignIn.RequireConfirmedAccount = false;
    })
    .AddEntityFrameworkStores<ApplicationDbContext>();

builder.Services.AddControllersWithViews();

...
```

### Important points
- `AddDefaultIdentity<IdentityUser>()` sets up the common Identity services and UI scenario
- `AddEntityFrameworkStores<ApplicationDbContext>()` connects Identity to EF Core
- `UseAuthentication()` must come before `UseAuthorization()`
- `MapRazorPages()` is required because the default Identity UI uses Razor Pages

---

## 6) Example ApplicationDbContext

```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

namespace MyApp.Data;

public class ApplicationDbContext : IdentityDbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
}
```

If you need a custom user type:

```csharp
using Microsoft.AspNetCore.Identity;

public class ApplicationUser : IdentityUser
{
    public string? FirstName { get; set; }
    public string? LastName { get; set; }
}
```

Then register it like this:

```csharp
builder.Services
    .AddDefaultIdentity<ApplicationUser>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

And update the context:

```csharp
public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
}
```

---

## 7) How scaffolded Identity UI works

ASP.NET Core Identity includes a **default UI**.  
You do **not** need to scaffold pages unless you want to customize them.

Common pages include:

- Register
- Login
- Logout
- Forgot Password
- Reset Password
- Confirm Email
- Access Denied
- Manage Account pages
- Two-factor pages

### Without scaffolding
The app uses the built-in Identity UI from the framework.

### With scaffolding
The generated Razor Pages are copied into your project so you can edit them directly.

Typical scaffold output location:

```txt
Areas/Identity/Pages/Account/
Areas/Identity/Pages/Account/Manage/
```

---

## 8) Scaffold Identity pages in Visual Studio 2022

This is the Visual Studio-style experience most teams use.

### Steps
1. Right-click the project
2. Select **Add**
3. Select **New Scaffolded Item**
4. Choose **Identity**
5. Select the pages you want to override
6. Choose your `DbContext`
7. Choose the layout page if prompted
8. Click **Add**

Visual Studio then generates the selected pages inside:

```txt
Areas/Identity/Pages/
```

### Best practice
Only scaffold the pages you actually need to customize.  
Do not scaffold everything unless necessary, because it increases maintenance work.

---

## 9) Scaffold Identity pages with the CLI

You can also scaffold Identity without Visual Studio.

### Basic example
```bash
dotnet aspnet-codegenerator identity -dc ApplicationDbContext
```

### Example scaffolding selected pages
```bash
dotnet aspnet-codegenerator identity -dc ApplicationDbContext --files "Account.Login;Account.Register;Account.Logout"
```

### Common options
- `-dc` -> DbContext class
- `--files` -> scaffold only selected pages
- `--useDefaultUI` -> keep default UI for pages not scaffolded
- `--force` -> overwrite existing files

Note: exact available options can vary slightly by SDK/tool version, so use help if needed:

```bash
dotnet aspnet-codegenerator identity -h
```

---

## 10) Common pages people scaffold first

These are the most common customization targets:

- `Account/Login`
- `Account/Register`
- `Account/ForgotPassword`
- `Account/ResetPassword`
- `Account/Manage/Index`
- `Account/Manage/Email`
- `Account/Manage/ChangePassword`

Reasons teams scaffold them:
- Add branding
- Add extra registration fields
- Change validation/UI text
- Add profile fields
- Modify account management pages

---

## 11) Adding custom user fields

A very common requirement is to add fields like:

- First name
- Last name
- Department
- Job title
- Tenant ID

### Example custom user
```csharp
public class ApplicationUser : IdentityUser
{
    public string? FirstName { get; set; }
    public string? LastName { get; set; }
}
```

After that:

1. Update Identity registration to use `ApplicationUser`
2. Add a migration
3. Update scaffolded Register / Manage pages to read and save the new fields

Migration example:

```bash
dotnet ef migrations add AddCustomFieldsToApplicationUser
dotnet ef database update
```

---

## 12) Configuring Identity options

You can customize password, lockout, and user settings.

```csharp
builder.Services
    .AddDefaultIdentity<IdentityUser>(options =>
    {
        options.Password.RequireDigit = true;
        options.Password.RequireLowercase = true;
        options.Password.RequireUppercase = true;
        options.Password.RequireNonAlphanumeric = false;
        options.Password.RequiredLength = 8;

        options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(5);
        options.Lockout.MaxFailedAccessAttempts = 5;

        options.User.RequireUniqueEmail = true;

        options.SignIn.RequireConfirmedEmail = false;
    })
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

### Common option areas
- `Password`
- `Lockout`
- `User`
- `SignIn`

---

## 13) Roles example

If you need role-based authorization, switch to `AddIdentity` or add role support.

```csharp
builder.Services
    .AddIdentity<IdentityUser, IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();
```

Example role checks:

```csharp
[Authorize(Roles = "Admin")]
public IActionResult AdminOnly()
{
    return View();
}
```

Create roles/users through seed code or admin screens.

---

## 14) Why scaffolded views sometimes differ from templates

Scaffolded Identity pages generated by Visual Studio 2022 may not exactly match:
- Your site layout
- Bootstrap version
- Existing shared partials
- Custom validation styling
- Project-specific structure

That is normal. After scaffolding, teams usually:
- Update `_Layout`
- Adjust CSS/classes
- Update shared imports
- Rename labels/text
- Add extra fields and validation

---

## 15) Common files created by scaffolding

You may see files like:

```txt
Areas/
  Identity/
    Pages/
      Account/
        Login.cshtml
        Login.cshtml.cs
        Register.cshtml
        Register.cshtml.cs
        ForgotPassword.cshtml
        ForgotPassword.cshtml.cs
      Manage/
        Index.cshtml
        Index.cshtml.cs
        ChangePassword.cshtml
        ChangePassword.cshtml.cs
```

You may also get:
- `_ViewImports.cshtml`
- `_ValidationScriptsPartial.cshtml`
- `_ManageNav.cshtml`

---

## 16) Typical workflow for customizing Register page

A common real-world flow:

1. Scaffold `Account/Register`
2. Add fields to `ApplicationUser`
3. Add fields to the `InputModel`
4. Update the `.cshtml` form
5. Update the registration logic in `Register.cshtml.cs`
6. Add a migration
7. Update the database

Example input model addition:

```csharp
public class InputModel
{
    [Required]
    [Display(Name = "First name")]
    public string FirstName { get; set; } = string.Empty;

    [Required]
    [Display(Name = "Last name")]
    public string LastName { get; set; } = string.Empty;

    [Required]
    [EmailAddress]
    public string Email { get; set; } = string.Empty;

    [Required]
    [StringLength(100, MinimumLength = 6)]
    [DataType(DataType.Password)]
    public string Password { get; set; } = string.Empty;

    [DataType(DataType.Password)]
    [Compare("Password")]
    public string ConfirmPassword { get; set; } = string.Empty;
}
```

Then map those fields into `ApplicationUser` during registration.

---

## 17) Common issues

### `MapRazorPages()` missing
Identity pages may 404 if Razor Pages are not mapped.

### `UseAuthentication()` missing
Users can't log in correctly if authentication middleware is not added.

### Wrong DbContext selected during scaffolding
Generated pages may reference the wrong context or fail at runtime.

### Migrations not applied
Identity tables won't exist until migrations are created and applied.

### Layout mismatch
Scaffolded pages may look broken if the selected layout file is wrong.

### Missing tooling
CLI scaffolding requires the ASP.NET code generator tool.

---

## 18) Recommended setup for most teams

For most MVC or Razor Pages apps:

- Start with **Individual Accounts**
- Use SQL Server + EF Core
- Keep default Identity UI initially
- Scaffold only the pages you truly need
- Create a custom `ApplicationUser` only when needed
- Use migrations for all schema changes
- Require secure password and sign-in settings for production

---

## 19) Summary

- ASP.NET Core Identity handles authentication and user management
- It commonly uses EF Core with SQL Server
- Visual Studio 2022 can scaffold Identity pages through **Add > New Scaffolded Item > Identity**
- Scaffolded pages are copied into `Areas/Identity/Pages`
- Use scaffolding only when you need to customize the default UI
- For custom profile data, create `ApplicationUser`, update pages, then run migrations

---

## 20) Handy commands reference

### Create project with Identity
```bash
dotnet new mvc -au Individual
```

### Install/update code generator
```bash
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet tool update -g dotnet-aspnet-codegenerator
```

### Scaffold Identity
```bash
dotnet aspnet-codegenerator identity -dc ApplicationDbContext
```

### Create migration
```bash
dotnet ef migrations add CreateIdentitySchema
```

### Update database
```bash
dotnet ef database update
```
