## Packages
```
dotnet add package Quartz
dotnet add package Quartz.Extensions.Hosting
```

## appsettings.json
```
{
  "ConnectionStrings": {
    "QuartzDb": "Server=localhost;Database=QuartzDb;Trusted_Connection=True;"
  },
  "Quartz": {
    "quartz.scheduler.instanceName": "MyQuartzScheduler",
    "quartz.scheduler.instanceId": "AUTO",
    "quartz.jobStore.type": "Quartz.Impl.AdoJobStore.JobStoreTX, Quartz",
    "quartz.jobStore.useProperties": "true",
    "quartz.jobStore.dataSource": "default",
    "quartz.jobStore.tablePrefix": "QRTZ_",
    "quartz.jobStore.driverDelegateType": "Quartz.Impl.AdoJobStore.SqlServerDelegate, Quartz",
    "quartz.jobStore.clustered": "true",
    "quartz.dataSource.default.provider": "SqlServer",
    "quartz.dataSource.default.connectionString": "Server=localhost;Database=QuartzDb;Trusted_Connection=True;"
  }
}

```

## PrintDateJob.cs
```
public class PrintDateJob : IJob
{
    private readonly ILogger<PrintDateJob> _logger;

    public PrintDateJob(ILogger<PrintDateJob> logger)
    {
        _logger = logger;
    }

    public async Task Execute(IJobExecutionContext context)
    {
        var token = context.CancellationToken;
        _logger.LogInformation("Job started at: {Time}", DateTime.Now);
        try
        {
            for (int i = 0; i < 300; i++) // simulate 5 minutes of work
            {
                token.ThrowIfCancellationRequested();
                _logger.LogInformation("Working... {Time}", DateTime.Now);
                await Task.Delay(1000, token); // Delay 1 second
            }
            _logger.LogInformation("Job finished normally at: {Time}", DateTime.Now);
        }
        catch (OperationCanceledException)
        {
            _logger.LogWarning("Job was interrupted and cancelled at: {Time}", DateTime.Now);
        }
    }
}

```

## Program.cs

```
var builder = WebApplication.CreateBuilder(args);

// Add Quartz with SQL Server-backed job store
builder.Services.AddQuartz(q =>
{
    q.UseMicrosoftDependencyInjectionJobFactory();

    var jobKey = new JobKey("PrintDateJob");
    q.AddJob<PrintDateJob>(opts => opts.WithIdentity(jobKey));

    q.AddTrigger(opts => opts
        .ForJob(jobKey)
        .WithIdentity("PrintDateTrigger")
        .WithCronSchedule("0 0/5 * * * ?")); // every 5 minutes
});

builder.Services.AddQuartzHostedService(q => q.WaitForJobsToComplete = true);
builder.Services.AddLogging();

```

## Stop (interrupt) a Quartz job
```
[ApiController]
[Route("api/[controller]")]
public class QuartzController : ControllerBase
{
    private readonly ISchedulerFactory _schedulerFactory;

    public QuartzController(ISchedulerFactory schedulerFactory)
    {
        _schedulerFactory = schedulerFactory;
    }

    [HttpPost("interrupt")]
    public async Task<IActionResult> InterruptJob(string jobName)
    {
        var scheduler = await _schedulerFactory.GetScheduler();
        var jobKey = new JobKey(jobName);

        if (!await scheduler.CheckExists(jobKey))
        {
            return NotFound($"Job '{jobName}' not found.");
        }

        await scheduler.Interrupt(jobKey);
        return Ok($"Job '{jobName}' was interrupted.");
    }
}
```


## Quartz requires the creation of a set of tables
[DB Schemas](https://github.com/quartznet/quartznet/tree/main/database/tables)
