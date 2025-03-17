
## Using .NET Aspire Integrations

### Adding Packages

#### Visual Studio

- Add -> .NET Aspire package menu item

#### Visual Studio Code

- Install C# Dev Kit for .NET Aspire integrations

#### CLI

```
dotnet add package Aspire.StackExchange.Redis --prerelease
```

### Registering and Consuming Packages

Each .NET Aspire integration has an equivalent hosting package installed in `AppHost` to configure resources and dependencies.
#### AppHost project

```c#
var cache = builder.AddRedis("cache"); 
builder
	.AddProject<Projects.AspireSample_Web>("webfrontend")
	.WithReference(cache);
```


#### Consuming project
```c#
build.AddRedisClient("cache");
```


## Types of Integrations

### Database

#### [[Relational Databases]]

- [[PostgreSQL]]
- [[MySQL]]
- [[SQL Database]]
- Oracle Database

#### [[NoSQL Databases]]

- [[MongoDB]]
- Azure Cosmos DB

### Storage

- Azure Blob Storage
- Azure Table Storage
- Azure Queue Storage

### Messaging

- [[RabbitMQ]]
- Apache Kafka
- Azure Service Bus

### Caching

#### [[Redis]]
- Redis
- Redis Output Caching
- Redis Distributed Cache

### Security

- Azure Key Vault

## [[Observability]]

- OpenTelemetry




