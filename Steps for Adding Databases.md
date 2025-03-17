
## App Host project

- Install the .NET Aspire hosting integration to the app host project.
- Register a database and create a container for it in the solution's app host.
- Pass a reference to the projects that needs access to the created container hosting the database.

## Projects that Use Database

- Add the .NET Aspire integration with a NuGet package to the projects that require data access. Optionally, if there's a .NET Core Entity Framework (EF) integration, you can use that instead.
- Register the data source, or database context for EF, in the project's _Program.cs_ file.
- Use dependency injection to inject the data source into your services.