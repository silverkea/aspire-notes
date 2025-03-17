
.NET Aspire provides three integrations that you can use to interact with a Redis service:

- **.NET Aspire StackExchange Redis integration**: Use this integration to interact directly with a Redis server. For example, you can use it to save and retrieve values in Redis database or subscribe to messages sent to a channel. Since this integration isn't focused on caching, we won't consider it further in this module.
- **.NET Aspire StackExchange Redis output caching integration**: Use this integration to cache complete HTTP responses.
- **.NET Aspire StackExchange Redis distributed caching integration**: Use this caching integration to store and retrieve data from a distributed cache. Distributed caching describes an architecture where multiple microservices or other client services share a single cache.

## Redis Distributed Cache
A distributed cache is one that is shared between several calling services.

### Setting Up Redis Distributed Caching in Aspire

#### App Host

Add the hosting package

```
dotnet add package Aspire.Hosting.Redis --prerelease
```

Register the cache and pass it to projects

```c#
// Register the cache
var redis = builder.AddRedis("redis");

// Initiate the consuming project and pass the cache
builder.AddProject<Projects.ConsumingProject>()
       .WithReference(redis);
```


#### Consuming Projects

Add the package
```
dotnet add package Aspire.StackExchange.Redis.DistributedCaching
```

Wire up the cache in application builder
```c#
builder.AddRedisDistributedCache("redis")
```

Using the cache
```c#
public class MyService(IDistributedCache cache)
{
   public async Task InitializeAsync()
   {
      // Check if there is cached content
      var cachedInfo = await cache.GetAsync("myinfo")

      if (cachedInfo is null)
      {
         // There's no content in the cache so formulate it here
         // For example, query databases.

        // Store the content in the cache
        await cache.SetAsync("myinfo", cachedInformation, new()
           { AbsoluteExpiration = DateTime.Now.AddSeconds(60) }
        );
      }
   }
}
```

#### Configuration

Connection string in appsettings.json
```json
{ 
	"ConnectionStrings": { 
		"redis": "redis.contoso.com:6379" 
	} 
}
```

Configure the behavior  in appsettings.json
```json
{ 
	"Aspire": { 
		"StackExchange": { 
			"Redis": { 
				"ConfigurationOptions": { 
					"ConnectTimeout": 5000, 
					"ConnectRetry": 3 
				} 
			} 
		} 
	} 
}
```

Using inline delegates
```c#
builder.AddRedisDistributedCache("redis", 
	configureOptions: options => options.ConnectTimeout = 5000 );
```

## Redis Output Cache
Store complete HTML pages in web apps or, in APIs, smaller portions of output.

### Setting Up Redis Output Caching in Aspire

## App Host

Add the hosting package
```
dotnet add package Aspire.Hosting.Redis --prerelease
```

Register the cache
``` c#
// Register the cache
var redis = builder.AddRedis("redis");

// Initiate the consuming project and pass the cache
builder.AddProject<Projects.ConsumingProject>()
       .WithReference(redis);
```

### Consuming Projects

Add the package
```
dotnet add package Aspire.StackExchange.Redis.OutputCaching
```

Wire up the output cache on the application
```c#
// Add the output cache
builder.AddRedisOutputCache();

// Build the app
var app = builder.Build();

// Add the middleware
app.UseOutputCache();
```


### Caching Complete Pages
Use the `OutputCache` attribute, e.g. in Razor page

```razor
@page "/"
@attribute [OutputCache(Duration = 10)]

<PageTitle>Welcome to Contoso</PageTitle>

<h1>Welcome to Contoso</h1>

This is our homepage. The time is: @DateTime.Now
```

### Minimal APIs
To cache this response, either call the `CacheOutput()` method, or apply the `OutputCache` attribute in the `MapGet` call:

``` c#
app
	.MapGet(
		"/products/{ProdId}", 
		(int ProdId) => $"The product ID is {ProdId}."
	)
	.CacheOutput();

app.MapGet(
	"/products/{ProdId}", 
	[OutputCache] (int ProdId) => $"The product ID is {ProdId}."
);
```

## Learn more

- [Overview of caching in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/overview)
- [Tutorial: Implement caching with .NET Aspire integrations](https://learn.microsoft.com/en-us/dotnet/aspire/caching/caching-integrations)
- [StackExchange.Redis Basic Usage](https://stackexchange.github.io/StackExchange.Redis/Basics)
- [.NET Aspire StackExchange Redis distributed caching integration](https://learn.microsoft.com/en-us/dotnet/aspire/caching/stackexchange-redis-distributed-caching-integration)
- [.NET Aspire StackExchange Redis output caching integration](https://learn.microsoft.com/en-us/dotnet/aspire/caching/stackexchange-redis-output-caching-integration)
- [StackExchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration.html)