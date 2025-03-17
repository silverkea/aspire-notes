
### Configuring the app host

Install the MongoDB hosting integration:

```
dotnet add package Aspire.Hosting.MongoDB --prerelease
```

Then, in the app host's _Program.cs_ file, add this code to register the container and create a database:


``` C#
var mongo = builder.AddMongoDB("mongo");
var mongodb = mongo.AddDatabase("mongodb");
```

Pass the database service to any projects that consume it:

``` c#
var northernTradersCatalogAPI = builder.AddProject<Projects.NorthernTraders_CatalogAPI>()
                                       .WithReference(mongodb);
```

### Configuring the consuming projects

You need to add the .NET Aspire MongoDB integration to any microservice project that uses the database. You add the `Aspire.MongoDB.Driver` NuGet package to the project that needs data access, using either .NET CLI or the Visual Studio NuGet package manager.

```
dotnet add package Aspire.MongoDB.Driver --prerelease
```

In the **AppHost** _Program.cs_ file, register the MongoDB driver. For example your code might look like this:


``` c#
var mongoDB = builder.AddMongoDB("mongo")
                     .WithMongoExpress()
                     .AddDatabase("BasketDB");

var myService = builder.AddProject<Projects.MyService>()
                       .WithReference(mongoDB);
```

The MongoDB integration, like PostgeSQL and MySQL, provides a database management tool called Mongo Express. The above code adds a container for it.

## Using a MongoDB database

All the projects in your solution that need access to the MongoDB database must add a MongoDB client. You then use the MongoDB client to read and write data to the database.

``` c#
builder.AddMongoDBClient("BasketDB");
```

The `AddMongoDBClient` method adds a client to the project. See this example code that uses it:


``` c#
using MongoDB.Driver;
using MongoDB.Driver.Linq;

public class MongoBasketStore
{
  public async Task<CustomerBasket?> GetBasketAsync(IMongoClient mongoClient, string customerId)
  {
    var basketCollection = mongoClient.GetDatabase("BasketDB").GetCollection<CustomerBasket>("basketitems");
    var filter = Builders<CustomerBasket>.Filter.Eq(r => r.BuyerId, customerId);
  
    return await basketCollection.Find(filter).FirstOrDefaultAsync();
  }
}
```

The above code creates a MongoDB collection that enables the method to query over all the `CustomerBasket` objects to find the one with the matching `BuyerId`.

## Configuring the MongoDB integration

### Using a connection string

In your projects _appsettings.json_ file, add a connection string to the MongoDB database. For example:

JSONCopy

```
{
  ...
  "ConnectionStrings": {
    "MongoConnectionString": "mongodb://localhost:27017/test",
  }
}
```

The previous connection string adds support for a database named **test** on a local MongoDB database server listening on port 27017. You use this connection string in your _Program.cs_ file when you create the MongoDB client:

``` c#
builder.AddMongoDBClient("MongoConnectionString");
```

### Other options

The tag for the `Microsoft.Extensions.Configuration` version of MongoDB is `Aspire:MongoDB:Driver`. So you can connect to a MongoDB database using the following JSON configuration:

``` json
{
  "Aspire": {
    "MongoDB": {
      "Driver": {
        "ConnectionString": "mongodb://localhost:27017/test",
        "HealthChecks": true,
        "HealthCheckTimeout": 10000,
        "Tracing": true
      },
    }
  }
}
```

With the previous configuration, you no longer need to add the connection string, just use `builder.AddMongoDBClient();`.

The final option is to configure the connection in code, with inline configurations. For example:

### Using inline delegates

``` c#
builder.AddMongoDBClient("mongodb", static settings => 
  { 
    settings.ConnectionString = "mongodb://localhost:27017/test"; 
    settings.HealthChecks = true; 
  });
```