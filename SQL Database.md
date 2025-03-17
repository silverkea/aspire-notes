
### Configuring the app host

To install the SQL database hosting integration, use this command:

```
dotnet add package Aspire.Hosting.SqlServer --prerelease
```

To register the container and database, add this code to the app host's _Program.cs_ file:

``` c#
var sql = builder.AddSqlServer("sql");
var sqldb = sql.AddDatabase("sqldb");
```

Then, pass a reference to database service to any projects that consume it:

``` c#
var northernTradersCatalogAPI = builder.AddProject<Projects.NorthernTraders_CatalogAPI>()
                                       .WithReference(sqldb);
```

### Configuring the consuming projects

To install the .NET Aspire SQL Database integration, use a command like this one in your .NET Aspire projects:

```
dotnet add package Aspire.Microsoft.Data.SqlClient --prerelease
```

Or to use the .NET Aspire SqlServer Entity Framework Core integration, use this command instead:

``` c#
dotnet add package Aspire.Microsoft.EntityFrameworkCore.SqlServer --prerelease
```

These NuGet packages can also be added using the **Add > .NET Aspire package** shortcut in Visual Studio.

The _*.AppHost_ project's _Program.cs_ file code to access the database is similar to the PostgreSQL example:

``` c#
var sqlServer = builder.AddSqlServer("sql")
                       .AddDatabase("sqldata");

var myService = builder.AddProject<Projects.MyService>()
                       .WithReference(sqlServer);
```

## Using a SQL Server database

In the projects that need SQL access, in the _Program.cs_ file, this code registers the Entity Framework database context:

``` c#
builder.AddSqlServerDbContext<YourDbContext>("sqldata");
```

Once the database is registered in the consuming project, you can interact with the database context `YourDbContext` using dependency injection. This example code retrieves weather forecasts from a database and selects one at random to return:


``` c#
app.MapGet("/weatherforecast", async (YourDbContext context) =>
{
  var rng = new Random();
  var forecasts = await context.Forecasts.ToListAsync();
  var forecast = forecasts[rng.Next(forecasts.Count)];
  return forecast;
});
```

## Configuring the SQL Server integration

### Using configuration providers

The SQL Server integration also supports `Microsoft.Extensions.Configuration`. By default, it looks for settings using the `Aspire:SqlServer:SqlClient` key. In projects using _appsettings.json_, an example configuration might look like this:

JSONCopy

``` c#
{
  "Aspire": {
    "SqlServer": {
      "SqlClient": {
        "ConnectionString": "YOUR_CONNECTIONSTRING",
        "HealthChecks": true,
        "Tracing": false,
        "Metrics": false
      }
    }
  }
}
```

### Using inline configurations

When you add the SQL Server integration, you can pass a `configureSettings` inline delegate to the `AddSqlServerClient` method. This delegate allows you to configure the settings for the database integration directly with code:

``` c#
builder.AddSqlServerClient("sqldata", static settings => settings.HealthChecks = false);
```

You can pass any of the supported options:

- `ConnectionString`: The connection string of the SQL Server database
- `HealthChecks`: A boolean value that indicates whether the database health check is enabled
- `Tracing`: A boolean value that indicates whether the OpenTelemetry tracing is enabled
- `Metrics`: A boolean value that indicates whether the OpenTelemetry metrics are enabled

### Connect to multiple databases

The SQL Server integration supports multiple connections through named instances. For example, you can connect to two different SQL Server databases in the same project:

``` json
{
  "Aspire": {
    "SqlServer": {
      "SqlClient": {
        "INSTANCE_1": {
          "ServiceUri": "YOUR_URI",
          "HealthChecks": false
        },
        "INSTANCE_2": {
          "ServiceUri": "YOUR_URI",
          "HealthChecks": false
        }
      }
    }
  }
}
```

Using this configuration, you can connect to the two different databases in the same project:

``` c#
builder.AddSqlServerClient("INSTANCE_1");
builder.AddSqlServerClient("INSTANCE_2");
```