# Background jobs Using `Hangfire`

* [Description](#description)
* [Setup](#setup)
* [Scenarios](#scenarios)

## Description
If you want to make some **lightweight** operations that works in the background you can use Hangfire, It gives you a flexibility in working with background jobs, so you can enqueue, schedule, delete or recurring them.


## Setup
Please follow these step:
1. Install this package `Hangfire`
2. Go to `appssettings.json` file add connection string to MSSqlserver
```csharp
"ConnectionStrings": {
    "DefaultConnection": "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=Hangfire;Integrated Security=True"
}
```

3. You need add these lines in services
```csharp
builder.Services.AddHangfire(x => x.UseSqlServerStorage(builder.Configuration.GetConnectionString("DefaultConnection")));
builder.Services.AddHangfireServer();
```

4. Add this line to app object
```csharp
app.UseHangfireDashboard("/dashboard");
```
> If you want to go to the dashboard of Hangfire, just put the base url of your domain then put this endpoint '/dashboard'.


## Scenarios
We will discuss the major operations on background jobs and how to use them.

### Enqueue
It's used to make background job works directly after initialize it.
```csharp
BackgroundJob.Enqueue(() => { Console.WriteLine("Hello world") });
```

### Schedule
It makes background job works but after specified time you determine.
```csharp
BackgroundJob.Schedule(() => { Console.WriteLine("Hello world") }, TimeSpan.FromMinutes(1));
```

### Recurring
It makes background job works recurrently and it doesn't stop until shut down the backend web server.
```csharp
RecurringJob.AddOrUpdate(() => { Console.WriteLine("Hello world") }, Cron.Minutly());
// or
RecurringJob.AddOrUpdate(() => Console.WriteLine("Hello world") }, Cron.Monthly(1));
```

### Delete
You can delete any background job you have created in anytime you want.
```csharp
string bjId = BackgroundJob.Schedule(() => { Console.WriteLine("Hello world") }, TimeSpan.FromMinutes(10));

await Task.Delay(TimeSpan.FromMinutes(5));

BackgroundJob.Delete(bjId);
```


