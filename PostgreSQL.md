
## Configuring the app host

Start by installing the appropriate hosting integration to the app host:

```
dotnet add package Aspire.Hosting.PostgreSQL --prerelease
```

Next, to register a database and create a container for it, add this code to the app host's _Program.cs_ file:

```c#
var postgres = builder.AddPostgres("postgres");
var postgresdb = postgres.AddDatabase("postgresdb");
```

You must also pass a reference to the database service to any projects that consume it:

``` c#
var northernTradersCatalogAPI = builder
	.AddProject<Projects.NorthernTraders_CatalogAPI>()
    .WithReference(postgresdb);
```

## Configuring the consuming projects

To install the .NET Aspire PostgreSQL integration, use a command like this one in your .NET Aspire projects:

```
dotnet add package Aspire.Npgsql --prerelease
```

Or to use the .NET Aspire PostgreSQL Entity Framework Core integration, use this command instead:

```
dotnet add package Aspire.Npgsql.EntityFrameworkCore.PostgreSQL --prerelease
```


## Admin Tools
Some of the .NET Aspire database integrations also allow you to create a container for database management tools. To add **PgAdmin** to your solution to manage the PostgreSQL database, use this code:

```c#
var postgresdb = builder.AddPostgres("pg")
                        .AddDatabase("postgresdb")
                        .WithPgAdmin();
```

The advantage of letting .NET Aspire create the container is that you don't need to do any configuration to connect PgAdmin to the PostgreSQL database, it's all automatic.


## Using Data Source 

In the _Program.cs_ file, this code registers the database:

```
builder.AddNpgsqlDataSource("postgresdb");
```

Once the database is registered in the consuming project, you can interact with the data source anytime you need it by using dependency injection:

```c#
public class YourService(NpgsqlDataSource dataSource)
{
    public async Task<IEnumerable<Catalog>> GetCatalog()
	{
        const string query = "SELECT * FROM catalog";
        using var dbConnection = dataSource.OpenConnection();
        var results = await dbConnection.QueryAsync<Catalog>(command);
        return queryResult.ToArray();
	}
}
```

## Using EF Core DbContext

Register the database context:

```
builder.AddNpgsqlDbContext<YourDbContext>("postgresdb");
```

Retrieve the database context `YourDbContext` to interact with the database:

```c#
public class YourService(YourDbContext context)
{
    public async Task<IEnumerable<Catalog>> GetCatalog()
	{
        var items = await context.ObjectItems;
        if (item is null)
        {
            return Results.NotFound();
        }
		else
		{
		    return items;
		}
	}
}
```


## Configuration Options

### Using a Connection String
In the project that requires the database, you use a connection string to connect to the database. This approach is useful when you need to connect to a database that isn't registered in the app host.

```c#
builder.AddNpgsqlDataSource("NpgsqlConnectionString");
```

Then in the configuration file, you can add the connection string:

```json
{
  "ConnectionStrings": {
    "NpgsqlConnectionString": "Host=myserver;Database=postgresdb;User id=myuser;Password=mypassword"
  }
}
```

### Using configuration providers

.NET Aspire has a feature of integrations that allows them to support a `Microsoft.Extensions.Configuration`. The PostgreSQL integration supports this feature, and by default it looks for settings using the `Aspire:Npgsql` key. In projects using _appsettings.json_, an example configuration might look like this:

```json
{
  "Aspire": {
    "Npgsql": {
      "ConnectionString": "Host=myserver;Database=postgresdb;User id=myuser;Password=mypassword",
      "HealthChecks": true,
      "Tracing": true,
      "Metrics": true
    }
  }
}
```

The previous configuration is setting the connection string, enabling health checks, tracing, and metrics for the PostgreSQL integration. You code then no longer needs to specify the connection string, just use `builder.AddNpgsqlDataSource();`.

If you're using the PostgreSQL Entity Framework Core integration, you can use the `Aspire:Npgsql:EntityFrameworkCore:PostgreSQL` key to configure the database context:

``` json
{
  "Aspire": {
    "Npgsql": {
      "EntityFrameworkCore": {
        "PostgreSQL": {
          "ConnectionString": "Host=myserver;Database=postgresdb;User id=myuser;Password=mypassword",
          "MaxRetryCount": 0,
          "HealthChecks": false,
          "Tracing": false
        }
      }
    }
  }
}
```

For more information on the Entity Framework configuration options, see the [.NET Aspire documentation](https://learn.microsoft.com/en-us/dotnet/aspire/database/postgresql-entity-framework-integration?tabs=dotnet-cli#configuration).

### Using inline delegates

The last option is to pass a `configureSettings` inline delegate to the `AddNpgsqlDataSource` method. This delegate allows you to configure the settings for the database integration directly with code:

``` c#
builder.AddNpgsqlDataSource(
    "postgresdb", static settings => settings.HealthChecks = false);
```




