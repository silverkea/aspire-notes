.NET provides APIs that you can use to log customized telemetry data. [[OpenTelemetry]] can export that data.
## Efficient logging

Use the following techniques every time you log:

- Check that the logging level you want to use is enabled. Available levels include information, warning, error, and critical. Adminstrators can enable different levels when testing, staging, and deploying to production. Log output is controlled through `IConfiguration`, typically using `appsettings.json` or environment variables.
- Avoid string interpolation in your logged message. Interpolated strings are defined with the `$` symbol and are evaluated even if your chosen logging level is not enabled. Instead, use a log method such as `LogInformation()` or `LogDebug()` and pass parameters in the argument list.
- Use the compile-time source generation to further optimize the logging performance and create a unique identifier for each log message, which is useful when querying for log messages in an APM.
## Compile time source generation

Compile time source generation with `ILogger` objects reduces the cost of logging by doing the string analysis once, rather than on each logging request. It also includes an ID for each type of log message. To use this technique, define partial logging methods with the logging parameters and apply the `LoggerMessageAttribute` to them. .NET automatically generates the complete logging method when the code is compiled.

Remember that in .NET Aspire, you don't need to create an `ILogger` but instead you can get it from dependency injection:

```
public partial class BasketService(
    IBasketRepository repository,
    ILogger<BasketService> logger) : Basket.BasketBase
{
    [LoggerMessage(
        EventId = 0,
        Level = LogLevel.Information,
        Message = "Obtaining a basket from method {Method} for basket {basketId}")]
    public partial void LogGetBasket(string Method, int basketId);
}
```

## Learn more

- [Logging guidance for .NET library authors](https://learn.microsoft.com/en-us/dotnet/core/extensions/logging-library-authors)
- [Compile-time logging source generation](https://learn.microsoft.com/en-us/dotnet/core/extensions/logger-message-generator)