Some ideas about a V2 Advanced API proposal.

    IConnectionFactory connectionFactory = new RabbitMQConnectionFactory();

    IPersistentConnection connection = connectionFactory.OpenConnection("host=localhost"); // is IDisposable

    IChannel channel = connection.OpenChannel(); // is IDisposable
    channel.PrefetchCount = 10;
    channel.ConfirmsSelect();

    IExchange exchange = Exchange.Direct("my_exchange");
    IExchange exchange = Exchange.Default();

    IQueue queue = Queue.Create("my_queue").Shared().;

    channel.Declare(exchange);
    channel.Declare(queue);

    var binding = channel.Bind(exchange, queue, "routing_key");

    byte[] message = SomeCreateMessageMethod();

    channel.ConfirmPublication += PublisherConfirmationHandler;

    channel.Publish(exchange, "my_routing_key", properties, message);

    // or typed message publication
    channel.Publish<MyMessage>(exchange, "my_routing_key", properties, new MyMessage());

    IConsumer consumer = new Consumer(connection.ConsumerQueue);

    IHandler<MyMessage> handler = new MyHandler(); // implements IHandler<MyMessage>

    consumer.AddHandler(handler); // consumer can have multiple handlers switched on message type.

    IConsumerHandle consumerHandle = channel.StartConsuming(queue, consumer);

    consumerHandle.StopConsuming();