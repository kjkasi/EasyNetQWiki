EasyNetQ's mission is to provide the simplest possible API for RabbitMQ based messaging. The core IBus interface purposefully avoids exposing [AMQP](http://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) concepts such as exchanges, bindings, and queues, instead it implements a default exchange-binding-queue topology based on message type.

For some scenarios it's useful to be able to configure your own exchange-binding-queue topology; the advanced EasyNetQ API allows you to do that. The advanced API assumes a good understanding of AMQP.

The advanced API is implemented with the [IAdvancedBus](https://github.com/mikehadlow/EasyNetQ/blob/master/Source/EasyNetQ/IAdvancedBus.cs) interface. An instance of this interface can be accessed via the Advanced property of IBus:

    var advancedBus = RabbitHutch.CreateBus("host=localhost").Advanced;

## Declaring Exchanges

To declare an exchange use the IAdvancedBus's ExchangeDeclare method:

    IExchange ExchangeDeclare(
        string name, 
        string type, 
        bool passive = false, 
        bool durable = true, 
        bool autoDelete = false, 
        bool @internal = false);

What the parameters mean:

    name: The name of the exchange you want to create
    type: The type of the exchange. It must be a valid AMQP exchange type. Use the static
        properties of the ExchangeType class to safely declare exchanges.
    passive: Do not create an exchange. If the named exchange doesn't exist, throw an exception.
        (default false)
    durable: Survive server restarts. If this parameter is false, the exchange will be removed
        when the server restarts.
        (default true)
    autoDelete: Delete this exchange when the last queue is unbound.
        (default false)
    internal: This exchange can not be directly used by publishers, but only used by exchange to
        exchange bindings.
        (default false)

Some examples:

    // create a direct exchange
    var exchange = advancedBus.ExchangeDeclare("my_exchange", ExchangeType.Direct);
    
    // create a topic exchange
    var exchange = advancedBus.ExchangeDeclare("my_exchange", ExchangeType.Topic);
    
    // create a fanout exchange
    var exchange = advancedBus.ExchangeDeclare("my_exchange", ExchangeType.Fanout);

To get the RabbitMQ default exchange do this:

    var exchange = Exchange.GetDefault();

##Declaring Queues

To declare a queue use the IAdvancedBus's QueueDeclare method:

    IQueue QueueDeclare(
        string name,
        bool passive = false,
        bool durable = true,
        bool exclusive = false,
        bool autoDelete = false,
        uint perQueueTtl = uint.MaxValue,
        uint expires = uint.MaxValue);

What the parameters mean:

    name:		The name of the queue	
    passive:	Do not create the queue if it doesn't exist, instead, throw an exception.
    	(default false)
    durable:	Can survive a server restart. If this is false the queue will be deleted when
    			the server restarts.
		(default true)
    exclusive:	Can only be accessed by the current connection.
    	(default false)
    autoDelete:	Delete the queue once all consumers have disconnected.
    	(default false)
    perQueueTtl: How long in milliseconds a message should remain on the queue before it is discarded.
    	(default not set)
    expires:	How long in milliseconds the queue should remain unused before it is automatically deleted.
    	(default not set)

Some examples:

    // declare a durable queue
    var queue = advancedBus.QueueDeclare("my_queue");

    // declare a queue with message TTL of 10 seconds:
    var queue = advancedBus.QueueDeclare("my_queue", perQueueTtl:10000);

To declare an 'unnamed' exclusive queue, where RabbitMQ provides the queue name, use the QueueDeclare overload with no parameters:

    var queue = advancedBus.QueueDeclare();

Note that EasyNetQ's automatic consumer reconnection logic is turned off for exclusive queues.

##Bindings

You bind a queue to an exchange like this:

    var queue = advancedBus.QueueDeclare("my.queue");
    var exchange = advancedBus.ExchangeDeclare("my.exchange", ExchangeType.Topic);
    var binding = advancedBus.Bind(exchange, queue, "A.*");

To specify multiple bindings between a queue and an exchange, simply make multiple bind calls:

    var queue = advancedBus.QueueDeclare("my.queue");
    var exchange = advancedBus.ExchangeDeclare("my.exchange", ExchangeType.Topic);
    advancedBus.Bind(exchange, queue, "A.B");
    advancedBus.Bind(exchange, queue, "A.C");

You can also bind exchanges to exchanges in a chain:

    var sourceExchange = advancedBus.ExchangeDeclare("my.exchange.1", ExchangeType.Topic);
    var destinationExchange = advancedBus.ExchangeDeclare("my.exchange.2", ExchangeType.Topic);
    var queue = advancedBus.QueueDeclare("my.queue");

    advancedBus.Bind(sourceExchange, destinationExchange, "A.*");
    advancedBus.Bind(destinationExchange, queue, "A.C");

## Publishing

The advanced Publish method allows you to specify the exchange you want to publish your message to. It also allows access to the message's AMQP basic properties.

Create your message. The advanced API requires that your message is wrapped in a Message<T>:

    var myMessage = new MyMessage {Text = "Hello from the publisher"};
    var message = new Message<MyMessage>(myMessage);

The [Message<T>](https://github.com/mikehadlow/EasyNetQ/blob/master/Source/EasyNetQ/IMessage.cs) class gives you access to the AMQP basic properties, for example:

    message.Properties.AppId = "my_app_id";
    message.Properties.ReplyTo = "my_reply_queue";

Finally, open a new channel and publish your message. Here we are publishing to the default exchange:

    using (var channel = advancedBus.OpenPublishChannel())
    {
        channel.Publish(Exchange.GetDefault, queueName, message);
    }

An overload of Publish allows you to bypass EasyNetQ's message serialization and create your own byte array messages:

    using (var channel = advancedBus.OpenPublishChannel())
    {
        var properties = new MessageProperties();
        var body = Encoding.UTF8.GetBytes("Hello World!");
        channel.Publish(Exchange.GetDefault, queueName, properties, body);
    }

## Subscribing

The advanced subscribe method allows you to specify the queue you wish to subscribe to and its exchange binding. It also gives you access to the AMQP basic properties and additional information about the subscription.

Before subscribing, declare the queue you wish to subscribe to and bind it to an exchange:

    var exchange = Exchange.DeclareDirect("advanced_test_exchange");
    var queue = Queue.DeclareDurable("advanced_test_queue");
    queue.BindTo(exchange, routingKey);

The subscription handler has the following type:

    Func<IMessage<T>, MessageRecievedInfo, Task>

It takes a Message<T>, as described above in the publish section; a [MessageReceivedInfo](https://github.com/mikehadlow/EasyNetQ/blob/master/Source/EasyNetQ/MessageReceivedInfo.cs), which contains information about the subscription and the message received; and returns a Task, which allows you to write non-blocking asynchronous handlers.

Subscribe by supplying the queue and the subscription handler:

    advancedBus.Subscribe<MyMessage>(queue, (msg, messageReceivedInfo) => 
        Task.Factory.StartNew(() =>
        {
            Console.WriteLine("Got Message: {0}", msg.Body.Text);
            Console.WriteLine("ConsumerTag: {0}", messageReceivedInfo.ConsumerTag);
            Console.WriteLine("DeliverTag: {0}", messageReceivedInfo.DeliverTag);
            Console.WriteLine("Redelivered: {0}", messageReceivedInfo.Redelivered);
            Console.WriteLine("Exchange: {0}", messageReceivedInfo.Exchange);
            Console.WriteLine("RoutingKey: {0}", messageReceivedInfo.RoutingKey);
        }));

## Message types must match

The EasyNetQ advanced API expects subscribers to only receive messages of the type provided by the generic type parameter. In the example above, only messages of type MyMessage should be received. However, EasyNetQ does not protect you from publishing messages of the wrong type to a subscriber. I could easily set up an exchange-binding-queue topology to publish messages of type NotMyMessage that would be received by the handler above. If a message of the wrong type is received, EasyNetQ will throw an **EasyNetQInvalidMessageTypeException** something like this:

    EasyNetQ.EasyNetQInvalidMessageTypeException: Message type is incorrect. Expected 'EasyNetQ_Tests_MyMessage:EasyNetQ_Tests', but was 'EasyNetQ_Tests_MyOtherMessage:EasyNetQ_Tests'
       at EasyNetQ.RabbitAdvancedBus.CheckMessageType[TMessage](MessageProperties properties) in D:\Source\EasyNetQ\Source\EasyNetQ\RabbitAdvancedBus.cs:line 217
       at EasyNetQ.RabbitAdvancedBus.<>c__DisplayClass1`1.<Subscribe>b__0(Byte[] body, MessageProperties properties, MessageReceivedInfo messageRecievedInfo) in D:\Source\EasyNetQ\Source\EasyNetQ\RabbitAdvancedBus.cs:line 131
       at EasyNetQ.RabbitAdvancedBus.<>c__DisplayClass6.<Subscribe>b__5(String consumerTag, UInt64 deliveryTag, Boolean redelivered, String exchange, String routingKey, IBasicProperties properties, Byte[] body) in D:\Source\EasyNetQ\Source\EasyNetQ\RabbitAdvancedBus.cs:line 176
       at EasyNetQ.QueueingConsumerFactory.HandleMessageDelivery(BasicDeliverEventArgs basicDeliverEventArgs) in D:\Source\EasyNetQ\Source\EasyNetQ\QueueingConsumerFactory.cs:line 85