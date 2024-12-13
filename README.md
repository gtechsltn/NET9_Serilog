# Serilog Write to Database in .NET 9.0

## Nuget Packages
```
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.MSSqlServer
```

## SQL Script
```
CREATE TABLE [Logs] (
    [Id] INT IDENTITY(1,1) PRIMARY KEY,
    [Timestamp] DATETIMEOFFSET NOT NULL,
    [Level] NVARCHAR(128) NOT NULL,
    [Message] NVARCHAR(MAX) NOT NULL,
    [Exception] NVARCHAR(MAX) NULL,
    [Properties] NVARCHAR(MAX) NULL
);
```

## Program.cs
```
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.Console()
    .WriteTo.MSSqlServer(
        connectionString: "YourConnectionStringHere",
        tableName: "Logs",
        autoCreateTable: true)
    .CreateLogger();

builder.Host.UseSerilog(); // Use Serilog as the logging provider

// Add services to the container
builder.Services.AddControllers();

var app = builder.Build();

app.UseRouting();

app.UseAuthorization();

app.MapControllers();

app.Run();
```

## SampleController.cs
```
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;

[ApiController]
[Route("api/[controller]")]
public class SampleController : ControllerBase
{
    private readonly ILogger<SampleController> _logger;

    public SampleController(ILogger<SampleController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IActionResult Get()
    {
        _logger.LogInformation("Sample log message at {Time}", DateTime.UtcNow);
        return Ok("Hello, World!");
    }
}
```
