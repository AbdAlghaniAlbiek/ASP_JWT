# Logging Using `Serilog`

* [Description](#description)
* [Setup](#Setup)
* [Scenarios](#scenarios)
  
## Desription
`ILogger` that provided by Asp.Net core have cool features to log many types of logs like information, warning, danger ...etc, 
but there is some limitation like it can't log to specified file or log to DB or get some deeper info like thread/process info, 
and here `Serilog` it comes to solve these problems

## Setup
Please follow these steps:
1. Install these packages
    1. `Serilog.AspNetCore`
    2. `Serilog.Enrichers.Environment`
    3. `Serilog.Enrichers.Process`
    4. `Serilog.Enrichers.Thread`
    5. `Serilog.Settings.Configuration`
    
2. Go to `appsettings.json` add this row json
```json
"Serilog": {
    "Using": [],
    "MinimalLevel": {
        "Default": "Information",
        "override": {
            "Microsoft": "Warning",
            "System": "Warning"
        }
    },
    "WriteTo": [
        {
            "Name": "Console"
        },
        {
            "Name": "File",
            "Args": {
               "path": "D:\\Logs\log.txt",
               "outputTemplate": "{Timestamp} {Message}{NewLine:1}{Exception:1}"
            }
        }
    ],
    "Enrich": [
        "FromLogContext",
        "WithMachineName",
        "WithProcessId",
        "WithThreadId"
    ],
    "Properties": {
        "ApplicationName": "Serilog.WebApplication"
    }
}
```

3. Go to `Program` class and modify on `Main` method to be like this
```csharp
public static void Main(string[] args)
{
    var config = new ConfigurationBuilder()
        .AddJsonFile("appsettings.json")
        .Build();

    Log.Logger = new LoggerConfiguration()
        .ReadFrom.Configuration(config)
        .CreateLogger();

    try
    {
        Log.Information("Application Starting");
        CreateHostBuilder(args).Build().Run();
    }
    catch (Exception ex)
    {
        Log.Fatal(ex, "The application failed to start!");
    }
    finally
    {
        Log.CloseAndFlush();
    }
}
```

4. You need also add `UseSerilog()` method inside `CreateHostBuilder` method in `Program` class
```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
Host.CreateDefaultBuilder(args)
    .UseSerilog()
    .ConfigureWebHostDefaults(webBuilder =>
    {
        webBuilder.UseStartup<Startup>();
    });
```

5. Then you can inject it in any class want, so let's assume that we have this controller
```csharp
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    private readonly ILogger<WeatherForecastController> _logger;

    public WeatherForecastController(ILogger<WeatherForecastController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        _logger.LogInformation("Action has worked");

        var rng = new Random();
        return Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateTime.Now.AddDays(index),
            TemperatureC = rng.Next(-20, 55),
            Summary = Summaries[rng.Next(Summaries.Length)]
        })
        .ToArray();

        _logger.LogInformation("Action has finished");
    }
}
```


## Scenario
We have here 3 scenarios we'll discussed:
1. The example above `ILogger` will logs to console and to D:\\logs\log.txt file that has been created automatically by your asp.net core project

2. In this scenario `ILogger` will log the information to log.json file in the same directory, but we need to change our previous raw json to this new one.
```json
"Serilog": {
    "Using": [],
    "MinimalLevel": {
        "Default": "Information",
        "override": {
            "Microsoft": "Warning",
            "System": "Warning"
        }
    },
    "WriteTo": [
        {
            "Name": "Console"
        },
        {
            "Name": "File",
            "Args": {
               "path": "D:\\Logs\log.json",
               "formatter": "Serilog.Formatting.Json.JsonFormatter, Serilog"
            }
        }
    ],
    "Enrich": [
        "FromLogContext",
        "WithMachineName",
        "WithProcessId",
        "WithThreadId"
    ],
    "Properties": {
        "ApplicationName": "Serilog.WebApplication"
    }
}
```

3. In this scenario `ILogger` will log the information to MSSqlServer DB, but we need to do these two steps:
   1. Install this package `Serilog.Sinks.MSSqlServer`.
   2. Change your previous raw json to this new one.
    ```csharp
    "Serilog": {
        "Using": [],
        "MinimalLevel": {
            "Default": "Information",
            "override": {
                "Microsoft": "Warning",
                "System": "Warning"
            }
        },
        "WriteTo": [
            { 
                "Name": "Console"
            },
            {
                "Name": "MSSqlServer",
                "Args": {
                    "ConnectionString": "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=SerilogDB;Integrated Security=True",
                    "sinkOptionsSection": {
                        "tableName": "Logs",
                        "schemaName": "Logging",
                        "autoCreateSqlTables": true
                    },
                    "restrictedMinimalLevel": "Warning"
                }
            }
        ],
        "Enrich": [
            "FromLogContext",
            "WithMachineName",
            "WithProcessId",
            "WithThreadId"
        ],
        "Properties": {
            "ApplicationName": "Serilog.WebApplication"
        }
    }
    ```
