using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

var host = Host.CreateDefaultBuilder(args)
    .ConfigureServices((context, services) =>
    {
        services.AddTransient<IMyService, MyService>();
        services.AddSingleton<Application>();
    })
    .Build();

var app = host.Services.GetRequiredService<Application>();
app.Run();


public class Application
{
    private readonly IMyService _myService;

    public Application(IMyService myService)
    {
        _myService = myService;
    }

    public void Run()
    {
        _myService.DoWork();
    }
}

public interface IMyService
{
    void DoWork();
}

public class MyService : IMyService
{
    public void DoWork()
    {
        Console.WriteLine("Service is working!");
    }
}
