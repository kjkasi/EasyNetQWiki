Some ideas about a V2 Advanced API proposal. The intention is that the V2 API should be much closer to AMQP. But still with useful features such as:

* A simple consumer loop, so that users don't have to worry about implementing it.
* A persistent connection that automatically reconnects.
* A default error handling strategy.
* A serializer.

Here's an example of the API in use ...

    IConnectionFactory connectionFactory = new RabbitMQConnectionFactory();

    IPersistentConnection connection = connectionFactory.OpenConnection("host=localhost"); // is IDisposable

    var channelSettings = new ChannelSettings
    {
        PrefetchCount = 10,
        ConfirmsSelect = true
    };

    IChannel channel = connection.OpenChannel(channelSettings); // is IDisposable

    IExchange exchange = Exchange.Direct("my_exchange", exchangeSettings);
    IExchange exchange = Exchange.Custom("my_exchange", "exchange_type", exchangeSettings); 
    IExchange exchange = Exchange.Default();

    IQueue queue = Queue.Create("my_queue", queueSettings);

    channel.Declare(exchange);
    channel.Declare(queue);

    var binding = channel.Bind(exchange, queue, "routing_key");

    byte[] message = SomeCreateMessageMethod();

    channel.ConfirmPublication += PublisherConfirmationHandler;

    channel.Publish(exchange, "my_routing_key", properties, message);

    // or typed message publication
    var message = new Message<MyMessage>(new MyMessage(), properties);
    var contentType = "application/json"; // EasyNetQ looks up the correct serializer for the content type
    var publishSettings = new PublishSettings(exchange, "my_routing_key", contentType);
    channel.Publish<MyMessage>(message, publishSettings);

    IConsumer consumer = new Consumer(connection.ComsumptionLoop, consumerSettings);
    IConsumer consumer = new ResolvingConsumer(connection.ConsumptionLoop, consumerSettings, windsorResolver);

    IHandler<MyMessage> handler = new MyHandler(); // implements IHandler<MyMessage> { void Handle(Message<MyMessage>); }

    consumer.AddHandler(handler); // consumer can have multiple handlers switched on message type.

    IConsumerHandle consumerHandle = channel.StartConsuming(queue, consumer);
    consumerHandle.Dispose();