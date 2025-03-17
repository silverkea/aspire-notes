## [[Orchestration]]

- Specify the .NET projects, containers, executables, cloud resources that make up the application
- Automatic discovery of endpoints and services, connection string management and injection
- Aspire project -`[SolutionName].AppHost` - set as start up project for solution

## [[Integrations]]

Opinionated stack that includes integrations to backing services for common functional requirements - e.g.

- Data storage - e.g. PostgreSQL, SQL Server, Azure Cosmos DB, Mongo DB
- Caching - e.g. Redis
- Messaging - e.g. RabbitMQ, Azure Service Bus

Community support for developing other integrations

## [[Tooling]]

- New project templates for Visual Studio / Rider
- .NET Aspire dashboard - displays all microservices and backing services, supports testing and monitoring, shows performance
- Extendable 