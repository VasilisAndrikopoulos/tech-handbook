Create your DbContext
---------------------

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
