## Using the MySQL integration

To install the .NET Aspire MySQL integration, use a command like this one in your .NET Aspire projects that require data access:

``` 
dotnet add package Aspire.MySqlConnector --prerelease
```

Or use the **Add > .NET Aspire package** shortcut in Visual Studio to install the integration from the NuGet package manager.

The _*.AppHost_ project's _Program.cs_ file code to access the database is similar to the PostgreSQL example:


``` c#
var mysqldb = builder.AddMySql("mysql")
                     .AddDatabase("mysqldb")
                     .WithPhpMyAdmin();

var myService = builder.AddProject<Projects.MyService>()
                       .WithReference(mysqldb);
```

The MySQL integration also allows you to create a container for database management tools. The previous example adds **PhpMyAdmin** to the solution.

## Using a MySQL database

The pattern is the same in the projects that need MySQL access. In the _Program.cs_ file, this code registers the database:

```
builder.AddMySqlDataSource("mysqldb");
```

Once the database is registered in the consuming project, you can interact with the data source anytime you need it by using dependency injection:

``` c#
app.MapGet("/catalog", async (MySqlConnection db) =>
{
    const string sql = """
        SELECT Id, Name, Description, Price
        FROM catalog
        """;

    // the db object is a connection to the MySQL database registered with AddMySqlDataSource
    return await db.QueryAsync<CatalogItem>(sql);
});
```

## Configuring the MySQL integration

The MySQL integration supports the same three options to manage the configuration.

### Connection strings

The _appsettings.json_ file can contain the connection string for the MySQL database:


``` json
{
  "ConnectionStrings": {
    "MySqConnection": "Server=myserver;Database=mysqldb;Uid=myuser;Pwd=mypassword"
  }
}
```

Then, in your project, you can connect to the database with the connection string using code like this:

``` c#
builder.AddMySqlDataSource("MySqConnection");
```

### Configuration providers

The `Aspire:MySqlConnector` key is used to configure the MySQL integration.


``` json
{
  "Aspire": {
    "MySqlConnector": {
      "ConnectionString": "Server=myserver;Database=mysqldb;Uid=myuser;Pwd=mypassword",
      "HealthChecks": true,
      "Tracing": false,
      "Metrics": false
    }
  }
}
```

### Inline configurations


``` c#
builder.AddMySqlDataSource("mysqldb", static settings => settings.HealthChecks = false);
```