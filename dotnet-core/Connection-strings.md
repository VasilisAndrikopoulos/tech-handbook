# SQL Server Connection Strings Guide (.NET / EF Core)

This brief guide explains how to configure and understand SQL Server connection strings in .NET (ASP.NET Core / EF Core).

---

## 📌 Basic Connection String

```txt
Server=localhost;Database=MyAppDb;User Id=sa;Password=Your_password123;
```

### Common aliases:
- `Server` = `Data Source`
- `Database` = `Initial Catalog`
- `User Id` = `UID`
- `Password` = `PWD`

---

## 🔐 Windows Authentication (Trusted Connection)

```txt
Server=localhost;Database=MyAppDb;Trusted_Connection=True;
```

or

```txt
Server=localhost;Database=MyAppDb;Integrated Security=True;
```

Use this when running on Windows with domain/local authentication.

---

## 🔒 SQL Server Authentication

```txt
Server=localhost;Database=MyAppDb;User Id=myuser;Password=mypassword;
```

---

## 🌐 Remote Server / Named Instance

```txt
Server=192.168.1.10;Database=MyAppDb;User Id=myuser;Password=mypassword;
```

```txt
Server=localhost\SQLEXPRESS;Database=MyAppDb;Trusted_Connection=True;
```

---

## 🚪 Using a Specific Port

```txt
Server=localhost,1433;Database=MyAppDb;User Id=myuser;Password=mypassword;
```

---

## 🔐 Encryption & Security Options

```txt
Server=localhost;Database=MyAppDb;User Id=myuser;Password=mypassword;
Encrypt=True;TrustServerCertificate=True;
```

### Important properties:
- `Encrypt=True` → Enables TLS encryption
- `TrustServerCertificate=True` → Skips certificate validation (dev only)
- `Persist Security Info=False` → Prevents sensitive info from being returned

---

## ⚙️ Advanced Options

```txt
Server=localhost;Database=MyAppDb;
Trusted_Connection=True;
MultipleActiveResultSets=True;
Connection Timeout=30;
Application Name=MyApp;
```

### Key options:
- `MultipleActiveResultSets=True` (MARS)
- `Connection Timeout=30`
- `Command Timeout=30` (set in code)
- `Application Name=MyApp`
- `Pooling=True` (default)
- `Min Pool Size=0`
- `Max Pool Size=100`

---

## ⚡ Connection String with EF Core (appsettings.json)

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyAppDb;Trusted_Connection=True;"
  }
}
```

---

## 🧩 Using in Program.cs

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));
```

---

## 🧪 Development vs Production Tips

### Development:
- Use `TrustServerCertificate=True`
- Use local SQL Server or Docker
- Keep credentials simple

### Production:
- Always use `Encrypt=True`
- Use secure secrets storage (Azure Key Vault, env vars)
- Avoid hardcoding credentials
- Use managed identity when possible

---

## 🧱 Common Mistakes

- ❌ Wrong server name or instance
- ❌ SQL Server not allowing remote connections
- ❌ Firewall blocking port 1433
- ❌ Using Windows auth on Linux containers
- ❌ Missing `Encrypt=True` in cloud environments

---

## 📚 Quick Reference Table

| Property | Description |
|--------|-------------|
| Server / Data Source | SQL Server location |
| Database / Initial Catalog | Database name |
| User Id / UID | Username |
| Password / PWD | Password |
| Trusted_Connection | Windows auth |
| Encrypt | Enable SSL |
| TrustServerCertificate | Skip cert validation |
| Connection Timeout | Seconds to connect |
| MultipleActiveResultSets | Allow multiple queries |
| Application Name | App identifier |
| Pooling | Enable connection pooling |

---

## ✅ Example (Production-ready)

```txt
Server=tcp:myserver.database.windows.net,1433;
Initial Catalog=MyAppDb;
User Id=myuser;
Password=mypassword;
Encrypt=True;
TrustServerCertificate=False;
Connection Timeout=30;
```

---

## 🚀 Summary

- Use **Trusted_Connection** for local/dev Windows setups
- Use **SQL authentication** for cross-platform and production
- Always enable **encryption in production**
- Store secrets securely (never commit to Git)

--- 
