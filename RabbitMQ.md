RabbitMQ is one of the most popular message brokers and has many features that make it an ideal candidate to handle communications in a cloud-native app. It includes:

- The RabbitMQ server, which hosts the queues. The server supports clustering and failover for high availability and can run in containers.
- Implementations of Advanced Message Queuing Protocol (AMQP), Simple Text Oriented Message Protocol (STOMP), and Message Queuing Telemetry Transport (MQTT).
- AMQP client libraries that you can use in .NET, Java, and Erlang.

In RabbitMQ terms, your microservices, which send and receive messages, are **clients**. Clients that send messages are termed **message producers**. Clients that receive messages are **message consumers**. The RabbitMQ service is the **message broker**.

# Communication Patterns

### Single Producer and Single Consumer
- Producer client sends to queue
- One client as consumer

### Competing Consumers
- Producer client sends to queue
- Multiple competing clients as consumer
- Messages processed by only one consumer - at most once processing

### Publishing and Subscribing

- Producer client sends via fanout exchange
- Queue per consumer type
- Single / competing consumers

### Routing messages and topics

- Producer client sends via direct exchange, message includes routing key
- Each subscription has a binding key
- Messages sent to subscriptions by matching routing key to binding key
- For more flexibility use a topic exchange - routing key with terms separated by dots, and binding key uses wild cards `*` to substitute for one word, or `#` to substitute zero or more words


## .NET Aspire RabbitMQ integration


## AppHost

Install the package
```
dotnet add package Aspire.Hosting.RabbitMQ
```

Register the service
```c#
// Service registration
var rabbit = builder.AddRabbitMQ("messaging");

// Service consumption
builder.AddProject<Projects.CatalogAPI>()
       .WithReference(rabbit);
```


## Consuming Projects

Install the package
```
dotnet add package Aspire.RabbitMQ.Client
```

Wire up in application builder
```
builder.AddRabbitMQClient("messaging");
```

Consume via Dependency Injection
``` c#
public class CatalogAPI(IConnection rabbitConnection)
{
    // Send and receive messages here
}
```

Create message channel
``` c#
var channel = connection.CreateModel();
```

### Sending

Create queue
```
channel.QueueDeclare(queue: "catalogEvents",
    durable: false,
    exclusive: false,
    autoDelete: false,
    arguments: null);
```

Publish as byte array


``` c#
var body = Encoding.UTF8.GetBytes("Getting all items in the catalog.");

channel.BasicPublish(exchange: string.Empty,
    routingKey: "catalogEvents",
    basicProperties: null,
    body: body);
```


### Receiving

Create consumer and handle the `Received` event

``` c#
var consumer = new EventingBasicConsumer(channel);
consumer.Received += ProcessMessageAsync;
```

Handle message and deserialize the body
``` c#
private void ProcessMessageAsync(object? sender, BasicDeliverEventArgs args)
{
    string messagetext = Encoding.UTF8.GetString(args.Body.ToArray());
    logger.LogInformation("The message is: {text}", messagetext);
}
```

Check the queue using `BasicConsume()` method

``` c#
channel.BasicConsume(queue:  queueName,
                    autoAck: true, 
                    consumer: consumer);
```


## Learn more

- [RabbitMQ Tutorials](https://www.rabbitmq.com/tutorials)
- [.NET Aspire RabbitMQ integration](https://learn.microsoft.com/en-us/dotnet/aspire/messaging/rabbitmq-client-integration)
- [.NET Aspire Apache Kafka integration](https://learn.microsoft.com/en-us/dotnet/aspire/messaging/kafka-integration)
- [Tutorial: Use .NET Aspire messaging integrations in ASP.NET Core](https://learn.microsoft.com/en-us/dotnet/aspire/messaging/messaging-integrations)
- [RabbitMQ Documentation](https://www.rabbitmq.com/docs)