### Add Migration
```
dotnet ef migrations add InitialCreate
// or
dotnet ef migrations add InitialCreate -c MyCustomDbContext
```

### Apply Migrations to Database
```
dotnet ef database update
// or
dotnet ef database update -c YourDbContextName
```

### Rollback
```
dotnet ef database update PreviousMigrationName -c MyCustomDbContext
// This will revert the database to its initial state (no tables).
dotnet ef database update 0 -c MyCustomDbContext
```

### Querying Data (LINQ)
```
var users = await context.Users.ToListAsync(); // Get all
var user = await context.Users.FindAsync(1);   // Find by PK
var filtered = await context.Users
    .Where(u => u.Name.Contains("vas"))
    .ToListAsync();
```

### Insert, Update, Delete
```
// Insert
context.Users.Add(new User { Name = "Vasilios" });
await context.SaveChangesAsync();

// Update
var user = await context.Users.FindAsync(1);
user.Name = "Updated";
await context.SaveChangesAsync();

// Delete
context.Users.Remove(user);
await context.SaveChangesAsync();
```

### Check Connection

```
bool canConnect = await context.Database.CanConnectAsync();
```

## API

### MigrateDB
```
public static async Task MigrateDBAsync(this WebApplication app)
{
    using var scope = app.Services.CreateScope();
    try
    {
        var context = scope.ServiceProvider.GetService<myInterlifeDbContext>();
        if (context != null)
        {
            //await context.Database.EnsureCreatedAsync();
            await context.Database.MigrateAsync();
        }
    }
    catch (Exception ex)
    {
        //add logging here later on
    }
}
```

### ResetDatabase

```
public static async Task ResetDatabaseAsync(this WebApplication app)
{
    using var scope = app.Services.CreateScope();
    try
    {
        var context = scope.ServiceProvider.GetService<myInterlifeDbContext>();
        if (context != null)
        {
            await context.Database.EnsureDeletedAsync();
            await context.Database.MigrateAsync();
        }
    }
    catch (Exception ex)
    {
        //add logging here later on
    }
}
```

