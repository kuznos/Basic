# Kafka in Clean Architecture  
---------------------------------------------

# Producer
```
src/
├── Domain/
├── Application/
│   └── Interfaces/
│       └── IEventPublisher.cs
├── Infrastructure/
│   └── Messaging/
│       └── KafkaEventPublisher.cs
├── API/
```

### Define Kafka Interfaces in Application Layer

```
public interface IEventPublisher
{
    Task PublishAsync<T>(string topic, T message);
}

```

### Implement Kafka in Infrastructure Layer
```
dotnet add package Confluent.Kafka
```
### Implement the interface
```
public class KafkaEventPublisher : IEventPublisher
{
    private readonly IProducer<Null, string> _producer;

    public KafkaEventPublisher(IConfiguration config)
    {
        var producerConfig = new ProducerConfig
        {
            BootstrapServers = config["Kafka:BootstrapServers"]
        };
        _producer = new ProducerBuilder<Null, string>(producerConfig).Build();
    }

    public async Task PublishAsync<T>(string topic, T message)
    {
        var json = JsonSerializer.Serialize(message);
        await _producer.ProduceAsync(topic, new Message<Null, string> { Value = json });
    }
}

```

### Register in DI Container in Infrastructure project

```
services.AddSingleton<IEventPublisher, KafkaEventPublisher>();
```

### Use in Application or API

```
public class OrderService
{
    private readonly IEventPublisher _publisher;

    public OrderService(IEventPublisher publisher)
    {
        _publisher = publisher;
    }

    public async Task PlaceOrder(Order order)
    {
        // Business logic...
        await _publisher.PublishAsync("orders", order);
    }
}
```

# Cosnumer

### Define an Interface in the Application Layer

```
public interface IEventConsumer
{
    Task ConsumeAsync(CancellationToken cancellationToken);
}
```

### Implement the Kafka Consumer
> In a .NET Clean Architecture setup, you’ll typically place the consumer logic in the Infrastructure layer and expose it via a background service.
```
public class KafkaEventConsumer : BackgroundService, IEventConsumer
{
    private readonly IConfiguration _config;
    private readonly ILogger<KafkaEventConsumer> _logger;

    public KafkaEventConsumer(IConfiguration config, ILogger<KafkaEventConsumer> logger)
    {
        _config = config;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var consumerConfig = new ConsumerConfig
        {
            BootstrapServers = _config["Kafka:BootstrapServers"],
            GroupId = "my-consumer-group",
            AutoOffsetReset = AutoOffsetReset.Earliest
        };

        using var consumer = new ConsumerBuilder<Ignore, string>(consumerConfig).Build();
        consumer.Subscribe("orders");

        try
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                var result = consumer.Consume(stoppingToken);
                _logger.LogInformation("Received message: {Message}", result.Message.Value);

                // TODO: Handle the message (e.g., call Application layer)
            }
        }
        catch (OperationCanceledException)
        {
            _logger.LogWarning("Kafka consumer cancelled.");
        }
        finally
        {
            consumer.Close();
        }
    }

    public Task ConsumeAsync(CancellationToken cancellationToken) => ExecuteAsync(cancellationToken);
}
```

### Register the Consumer in DI
```
services.AddHostedService<KafkaEventConsumer>();
```
> This ensures the consumer runs as a background service when the app starts.







