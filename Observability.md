
A system that emits rich telemetry is 
	- Observable - allows understanding of behavior from outside it
	- Instrumented - emits all data developers need to troubleshoot an issue
## Pillars of Observability

- [[Logging]] - timestamped text record of what happened during processing
- Metrics - measurement of a service that can be aggregated into statistics
- Tracing - distributed trace record of all units of work often referred to as Spans

## Using telemetry in .NET
.NET has built-in types to represent the three pillars:

- **ILogger**: This interface provides a standard interface you can use when you what to log events from any .NET code.
- **Meter**: You can use this class to create a group of instruments, each of which measures a value associated with the performance or behavior of your code. For example, you could add an instrument that counts the sales of a product, or another that measures the length of time a message waited in a queue. .NET also provides many built-in metrics.
- **Activity**: You can use this class to record the tracing information. Start by creating an `ActivitySource` object for your namespace. Then call the `StartActivity` method to begin recording data.

## OpenTelemetry configuration in .NET Aspire

The code that adds and configures OpenTelemetry in a .NET Aspire solution is in the **ServiceDefaults** project. In the _Extensions.cs_ file you find:

- The `ConfigureOpenTelemetry()` method, which adds logging, metrics, and tracing services.
- The `AddOpenTelemetryExporters()` method, which adds OpenTelemetry Protocol (OTLP) exporters.
- The `AddBuiltInMeters()` method, which adds all the metrics that are built into .NET.

You should extend this code when you want to:

- Add additional sources of metrics or distributed tracing data, such as custom metrics.
- Add exporters to send telemetry data to an Application Performance Management (APM) system such as Application Insights or Grafana.
## Exporting telemetry
Common telemetry export destinations include:

- The [[Aspire Dashboard]]. You'll learn more about the dashboard later in this module.
- Other Application Performance Management (APM) tools such as Prometheus and Grafana.
- Azure Application Insights. This feature of Azure Monitor can analyze and display behavior data from many sources, both within Azure and from other sources, such as cloud-native apps.

## Learn more

- [.NET Aspire telemetry](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/telemetry)
- [.NET observability with OpenTelemetry](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel)
- [What diagnostic tools are available in .NET Core?](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/)
- [Creating custom metrics](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/metrics-instrumentation)
- [Adding distributed tracing instrumentation](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/distributed-tracing-instrumentation-walkthroughs)