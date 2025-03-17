
Manifests describe the contents of the .NET Aspire solution, including its microservices, backing services, and configuration.


To create the manifest file, both Visual Studio and the Azure Developer CLI run a .NET CLI `run` command with a specific target. You can manually run the same command to create your own manifest file like this:


## Creating from AppHost project
```
dotnet run --project eShop.AppHost\eShop.AppHost.csproj `
    --publisher manifest `
    --output-path ../aspire-manifest.json
```

Make sure you specify the app host project with the `--project` option.


## Creating from Launch Profile

Add a launch profile to create a manifest file with JSON like this:
``` json
{
	"$schema": "http://json.schemastore.org/launchsettings.json",
	"profiles": {
	...
	  "generate-manifest": {
	    "commandName": "Project",
	    "launchBrowser": false,
	    "dotnetRunMessages": true,
	    "commandLineArgs": "--publisher manifest --output-path aspire-manifest.json"
	  }
}
```

In Visual Studio, you can then choose the **generate-manifest** profile when you start debugging. At the command line, use the `--launch-profile` option:

```
dotnet run --launch-profile generate-manifest
```


## Example Manifest
Includes a web front end, API service and cache

``` json
{
  "resources": {
    "cache": {
      "type": "container.v0",
      "connectionString": "{cache.bindings.tcp.host}:{cache.bindings.tcp.port}",
      "image": "docker.io/library/redis:7.2",
      "bindings": {
        "tcp": {
          "scheme": "tcp",
          "protocol": "tcp",
          "transport": "tcp",
          "targetPort": 6379
        }
      }
    },
    "apiservice": {
      "type": "project.v0",
      "path": "AspireStarter.ApiService/AspireStarter.ApiService.csproj",
      "env": {
        "OTEL_DOTNET_EXPERIMENTAL_OTLP_EMIT_EXCEPTION_LOG_ATTRIBUTES": "true",
        "OTEL_DOTNET_EXPERIMENTAL_OTLP_EMIT_EVENT_LOG_ATTRIBUTES": "true",
        "OTEL_DOTNET_EXPERIMENTAL_OTLP_RETRY": "in_memory",
        "ASPNETCORE_FORWARDEDHEADERS_ENABLED": "true"
      },
      "bindings": {
        "http": {
          "scheme": "http",
          "protocol": "tcp",
          "transport": "http"
        },
        "https": {
          "scheme": "https",
          "protocol": "tcp",
          "transport": "http"
        }
      }
    },
    "webfrontend": {
      "type": "project.v0",
      "path": "AspireStarter.Web/AspireStarter.Web.csproj",
      "env": {
        "OTEL_DOTNET_EXPERIMENTAL_OTLP_EMIT_EXCEPTION_LOG_ATTRIBUTES": "true",
        "OTEL_DOTNET_EXPERIMENTAL_OTLP_EMIT_EVENT_LOG_ATTRIBUTES": "true",
        "OTEL_DOTNET_EXPERIMENTAL_OTLP_RETRY": "in_memory",
        "ASPNETCORE_FORWARDEDHEADERS_ENABLED": "true",
        "ConnectionStrings__cache": "{cache.connectionString}",
        "services__apiservice__http__0": "{apiservice.bindings.http.url}",
        "services__apiservice__https__0": "{apiservice.bindings.https.url}"
      },
      "bindings": {
        "http": {
          "scheme": "http",
          "protocol": "tcp",
          "transport": "http",
          "external": true
        },
        "https": {
          "scheme": "https",
          "protocol": "tcp",
          "transport": "http",
          "external": true
        }
      }
    }
  }
}
```


In the app host's _Program.cs_ file, the `webfrontend` project depends on both the `cache` and the `apiservice`
``` c#
var cache = builder.AddRedis("cache");

var apiService = builder.AddProject<Projects.AspireStarter_ApiService>("apiservice");

builder.AddProject<Projects.AspireStarter_Web>("webfrontend")
    .WithExternalHttpEndpoints()
    .WithReference(cache)
    .WithReference(apiService);
```

In the manifest these dependencies are expressed as environment variables
``` json 
"env": {
  "ConnectionStrings__cache": "{cache.connectionString}",
  "services__apiservice__http__0": "{apiservice.bindings.http.url}",
  "services__apiservice__https__0": "{apiservice.bindings.https.url}"
}
```




## Learn more

- [.NET Aspire manifest format for deployment tool builders](https://learn.microsoft.com/en-us/dotnet/aspire/deployment/manifest-format)
- [.NET Aspire and launch profiles](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/launch-profiles)