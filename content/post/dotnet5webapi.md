---
title:      "Creating a .NET 5 Web API project with MySQL connection"
date: 2020-12-01T01:37:56+08:00
lastmod: 2020-12-01T01:37:56+08:00
author:     "Adrian"
tags: [".NET 5"]
categories: ["Coding"]
draft: false
---

1. Create a web API project using .NET Core CLI:

```linux
dotnet new webapi -o MyWebApi
```

2. Add the following packages in .csproj. (Or using .NET Core CLI command`dotnet add package` )

```c#
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="5.0.1">
    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    <PrivateAssets>all</PrivateAssets>
</PackageReference>
<PackageReference Include="MySql.Data.EntityFrameworkCore" Version="8.0.22" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="5.6.3" />
<PackageReference Include="Pomelo.EntityFrameworkCore.MySql" Version="5.0.0-alpha.2" />
```

3. Install Entity Framework Core CLI Tools if haven't already

```linux
dotnet tool install --global dotnet-ef
```

4. If database already exists, use `dotnet ef dbcontext scaffold` command to generate code for a `DbContext` and entity types

```linux
dotnet ef dbcontext scaffold "Server=localhost;Database=yourdb;User=root;Password=yourpwd;TreatTinyAsBoolean=true;" "Pomelo.EntityFrameworkCore.MySql" --context-dir "DB" -o "Models"
```

5. Add connection string to appsettings.json

```json
  "ConnectionStrings": {
    "DefaultConnection": "Server=192.168.129.82;Database=lgdb_examination;Trusted_Connection=True;MultipleActiveResultSets=true"
 }
```

6. Register dbcontext to the services configuration in your the `Startup.cs` file.

```c#
public void ConfigureServices(IServiceCollection services)
{
    // Replace "YourDbContext" with the name of your own DbContext derived class.
    services.AddDbContextPool<YourDbContext>(
        dbContextOptions => dbContextOptions
        .UseMySql(
            Configuration.GetConnectionString("DefaultConnection"),
            // Replace with your server version and type.
            new MySqlServerVersion(new Version(5, 7, 28)),
            mySqlOptions => mySqlOptions
            .CharSetBehavior(CharSetBehavior.NeverAppend)
        )
        // Everything from this point on is optional but helps with debugging.
        .EnableSensitiveDataLogging()
        .EnableDetailedErrors()
    );
}
```

