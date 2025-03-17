
## Benefits
- **App composition**: .NET Aspire specifies the .NET projects, containers, executables, and cloud resources that make up the application.
- **Service discovery and connection string management**: The app host manages injecting the right connection strings and service discovery information to simplify the developer experience.

## Supported Resource Types
- **ProjectResource**: A .NET project, such as ASP.NET Core web apps.
- **ContainerResource**: A container image, such as a Docker image containing Redis.
- **ExecutableResource**: An executable file.


## Enlisting an existing app in .NET Aspire orchestration

Visual Studio provides menus to enlist an existing project in .NET Aspire orchestration.

![[add-orchestration-to-project.png]]


## Changes Aspire makes to an existing solution

When you add .NET Aspire orchestration to your solution, the following changes happen:

- An **AppHost** project is added. The project contains the orchestration code. It becomes the entry point for your app and is responsible for starting and stopping your app. It also manages the service discovery and connection string management.
- A **ServiceDefaults** project is added. The project configures OpenTelemetry, adds default health check endpoints, and enables service discovery through `HttpClient`.
- The solution's default startup project is changed to **AppHost**.
- Dependencies on the projects enrolled in orchestration are added to the **AppHost** project.
- The .NET Aspire Dashboard is added to your solution, which enables shortcuts to access all the project endpoints in your solution.
- The dashboard adds logs, traces, and metrics for the projects in your solution.

If you add orchestration to a web app project, .NET Aspire automatically adds a reference to the **ServiceDefaults** project. It then makes the following changes to the code in _Program.cs_:

- Adds a call to `AddServiceDefaults` that enables the default OpenTelemetry, meters, and service discovery.
- Adds a call to `MapDefaultEndpoints` that enables the default endpoints, such as `/health` and `/alive`.