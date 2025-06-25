## Add Pacakges
dotnet add package Microsoft.Extensions.Http.Resilience
dotnet add package Microsoft.Extensions.Resilience


## Configure a Named HttpClient with Resilience In Program.cs
```
builder.Services.AddHttpClient("MyApi")
    .AddResilienceHandler("my-api-pipeline", builder =>
    {
        builder.AddRetry(new()
        {
            MaxRetryAttempts = 3,
            Delay = TimeSpan.FromSeconds(2)
        });

        builder.AddTimeout(TimeSpan.FromSeconds(5));
    });
```
### Use the Named Client

```
public class MyService
{
    private readonly HttpClient _httpClient;

    public MyService(IHttpClientFactory factory)
    {
        _httpClient = factory.CreateClient("MyApi");
    }

    public async Task<string> GetDataAsync()
    {
        var response = await _httpClient.GetAsync("/data");
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadAsStringAsync();
    }
}
```

# Reactive Strategies

## Retry
```
.AddRetry(new RetryStrategyptions
{
    MaxRetryAttempts = 3,
    Delay = TimeSpan.FromMilliseconds(200),
    BackoffType = DelayBackoffType.Exponential
})
```
* Transient network errors (e.g., HTTP 500, timeouts)
* Temporary unavailability of a downstream service
* Flaky third-party APIs

## Circuit Breaker
```
.AddCircuitBreaker(new CircuitBreakerStrategyOptions
{
    FailureRatio = 0.5,
    MinimumThroughput = 5,
    SamplingDuration = TimeSpan.FromSeconds(30),
    BreakDuration = TimeSpan.FromSeconds(15)
})
```
* Prevent hammering a failing system
* Avoid exhausting resources on repeated errors
* Enable graceful recovery and health checks

## Fallback
```
.AddFallback(new FallbackStrategyOptions
{
    ShouldHandle = new PredicateBuilder().Handle<TimeoutRejectedException>(),
    FallbackAction = static _ => ValueTask.FromResult(new HttpResponseMessage(HttpStatusCode.OK)
    {
        Content = new StringContent("This is a fallback response.")
    })
})

* Gracefully handle complete failure
* Return cached/stubbed data instead of throwing errors
* Maintain user experience even when services are down
```

## Timeout
```
.AddTimeout(new TimeoutStrategyOptions
{
    Timeout = TimeSpan.FromSeconds(3)
});
```
* Protect against long-hanging requests
* Prevent a system from freezing due to stuck dependencies
* Maintain snappy UX and system responsiveness

## Hedging
```
.AddHedging(new HedgingStrategyOptions
{
    MaxHedgedAttempts = 2,
    HedgingDelay = TimeSpan.FromMilliseconds(100),
    ShouldHandle = new PredicateBuilder().Handle<Exception>(),
    HedgingActionGenerator = args => ValueTask.FromResult<HedgingAction>(
        async token =>
        {
            Console.WriteLine("Hedged action triggered");
            await Task.Delay(200, token);
        })
});
```
* High-latency or unreliable endpoints
* Mission-critical systems where waiting too long is costly
* Redundant endpoints or mirror services


## Rate Limiter
```
.AddRateLimiter(new RateLimiterStrategyOptions
{
    PermitLimit = 5,
    Window = TimeSpan.FromSeconds(10),
    QueueLimit = 2
});
```
* Protect services from being overwhelmed by traffic

-----------------------------------------------------------------

## Strong Resilience Pipeline Setup

1 Timeout – Fail fast if too slow.
2 Retry – Retry on transient errors.
3 Circuit Breaker – Stop calling a broken service.
4 Fallback – Provide default or cached response if all else fails.

```
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Http.Resilience;
using Microsoft.Extensions.Resilience;
using Polly;
using Polly.Retry;
using Polly.CircuitBreaker;
using Polly.Timeout;
using Polly.Fallback;
using System.Net;

var builder = WebApplication.CreateBuilder(args);

// Register HttpClient with a strong resilience pipeline
builder.Services
    .AddHttpClient("MyApi", client =>
    {
        client.BaseAddress = new Uri("https://httpbin.org/");
    })
    .AddResilienceHandler("strong-pipeline", static pipelineBuilder =>
    {
        // Timeout strategy (first line of defense)
        pipelineBuilder.AddTimeout(TimeSpan.FromSeconds(3));

        // Retry strategy
        pipelineBuilder.AddRetry(new RetryStrategyOptions<HttpResponseMessage>
        {
            MaxRetryAttempts = 3,
            Delay = TimeSpan.FromMilliseconds(500),
            BackoffType = DelayBackoffType.Exponential,
            ShouldHandle = args =>
            {
                if (args.Outcome.Result is HttpResponseMessage response)
                {
                    return ValueTask.FromResult(
                        response.StatusCode == HttpStatusCode.InternalServerError ||
                        response.StatusCode == HttpStatusCode.RequestTimeout ||
                        response.StatusCode == HttpStatusCode.ServiceUnavailable ||
                        response.StatusCode == HttpStatusCode.BadGateway ||
                        response.StatusCode == HttpStatusCode.GatewayTimeout
                    );
                }

                return ValueTask.FromResult(false);
            }
        });

        // Circuit Breaker strategy
        pipelineBuilder.AddCircuitBreaker(new CircuitBreakerStrategyOptions<HttpResponseMessage>
        {
            FailureRatio = 0.5,
            MinimumThroughput = 4,
            SamplingDuration = TimeSpan.FromSeconds(10),
            BreakDuration = TimeSpan.FromSeconds(15),
            ShouldHandle = args =>
            {
                if (args.Outcome.Result is HttpResponseMessage response)
                {
                    return ValueTask.FromResult(!response.IsSuccessStatusCode);
                }

                return ValueTask.FromResult(false);
            }
        });

        // Fallback strategy
        pipelineBuilder.AddFallback(new FallbackStrategyOptions<HttpResponseMessage>
        {
            ShouldHandle = args =>
            {
                if (args.Outcome.Result is HttpResponseMessage response)
                {
                    return ValueTask.FromResult(!response.IsSuccessStatusCode);
                }

                return ValueTask.FromResult(false);
            },
            FallbackAction = _ =>
            {
                var fallback = new HttpResponseMessage(HttpStatusCode.OK)
                {
                    Content = new StringContent("Fallback response from resilience pipeline.")
                };

                return ValueTask.FromResult(Outcome.FromResult(fallback));
            }
        });
    });

var app = builder.Build();

// Sample endpoint to simulate transient failure (e.g., status/500)
app.MapGet("/", async (IHttpClientFactory httpClientFactory) =>
{
    var client = httpClientFactory.CreateClient("MyApi");

    var response = await client.GetAsync("status/500");
    var content = await response.Content.ReadAsStringAsync();
    return Results.Content(content);
});

app.Run();
```



