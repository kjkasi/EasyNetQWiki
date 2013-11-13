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

Queues can be set as single use. This is an EasyNetQ feature that deletes the queue and closes the queue's channel once the first message has been consumed. Set a queue to single use by calling:

    queue.SetAsSingleUse();

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

Finally publish your message using the Publish method. Here we are publishing to the default exchange:

    bus.Publish(Exchange.GetDefault, queueName, message);

An overload of Publish allows you to bypass EasyNetQ's message serialization and create your own byte array messages:

    var properties = new MessageProperties();
    var body = Encoding.UTF8.GetBytes("Hello World!");
    bus.Publish(Exchange.GetDefault, queueName, properties, body);

## Consuming

Use the IAdvancedBus's Consume method to consume messages from a queue.

    IDisposable Consume<T>(IQueue queue, Func<IMessage<T>, MessageReceivedInfo, Task> onMessage) where T : class;

The onMessage delegate is the handler you provide for message delivery. Its parameters are as follows:

As described in the publish section above, IMessage<T> gives you access to the message and its MessageProperties. MessageRecivedInfo gives you extra information about the context in which the message was consumed:

    public class MessageReceivedInfo
    {
        public string ConsumerTag { get; set; }
        public ulong DeliverTag { get; set; }
        public bool Redelivered { get; set; }
        public string Exchange { get; set; }
        public string RoutingKey { get; set; }         
    }

You return a Task which allows you to write a non-blocking asynchronous handler.

If you only need a synchronous handler you can use the synchronous overload:

    IDisposable Consume<T>(IQueue queue, Action<IMessage<T>, MessageReceivedInfo> onMessage) where T : class;

To bypass EasyNetQ's message serializer, use the consume overload that provides the raw byte array message:

    void Consume(IQueue queue, Func<Byte[], MessageProperties, MessageReceivedInfo, Task> onMessage);

In this example we are consuming the raw message bytes from the queue 'my_queue':

    var queue = advancedBus.QueueDeclare("my_queue");
    advancedBus.Consume(queue, (body, properties, info) => Task.Factory.StartNew(() =>
        {
            var message = Encoding.UTF8.GetString(body);
            Console.WriteLine("Got message: '{0}'", message);
        }));

You can optionally register multiple handlers with a single consumer using this overload of the Consume method:

    IDisposable Consume(IQueue queue, Action<IHandlerRegistration> addHandlers);

The IHandlerRegistration interface looks like this:

    public interface IHandlerRegistration
    {
        /// <summary>
        /// Add an asynchronous handler
        /// </summary>
        /// <typeparam name="T">The message type</typeparam>
        /// <param name="handler">The handler</param>
        /// <returns></returns>
        IHandlerRegistration Add<T>(Func<IMessage<T>, MessageReceivedInfo, Task> handler)
            where T : class;

        /// <summary>
        /// Add a synchronous handler
        /// </summary>
        /// <typeparam name="T">The message type</typeparam>
        /// <param name="handler">The handler</param>
        /// <returns></returns>
        IHandlerRegistration Add<T>(Action<IMessage<T>, MessageReceivedInfo> handler)
            where T : class;

        /// <summary>
        /// Set to true if the handler collection should throw an EasyNetQException when no
        /// matching handler is found, or false if it should return a noop handler.
        /// Default is true.
        /// </summary>
        bool ThrowOnNoMatchingHandler { get; set; }
    }

In this example we are registering two different handlers, one that handles messages of type MyMessage, and the other which handles messages of type MyOtherMessage:

    bus.Advanced.Consume(queue, x => x
            .Add<MyMessage>((message, info) => 
                { 
                    Console.WriteLine("Got MyMessage {0}", message.Body.Text);
                    countdownEvent.Signal();
                })
            .Add<MyOtherMessage>((message, info) =>
                {
                    Console.WriteLine("Got MyOtherMessage {0}", message.Body.Text);
                    countdownEvent.Signal();
                })
        );

See this blog post for more information: 

http://mikehadlow.blogspot.co.uk/2013/11/easynetq-multiple-handlers-per-consumer.html

## Message types must match

The EasyNetQ advanced API expects subscribers to only receive messages of the type provided by the generic type parameter. In the example above, only messages of type MyMessage should be received. However, EasyNetQ does not protect you from publishing messages of the wrong type to a subscriber. I could easily set up an exchange-binding-queue topology to publish messages of type NotMyMessage that would be received by the handler above. If a message of the wrong type is received, EasyNetQ will throw an **EasyNetQInvalidMessageTypeException** something like this:

    EasyNetQ.EasyNetQInvalidMessageTypeException: Message type is incorrect. Expected 'EasyNetQ_Tests_MyMessage:EasyNetQ_Tests', but was 'EasyNetQ_Tests_MyOtherMessage:EasyNetQ_Tests'
       at EasyNetQ.RabbitAdvancedBus.CheckMessageType[TMessage](MessageProperties properties) in D:\Source\EasyNetQ\Source\EasyNetQ\RabbitAdvancedBus.cs:line 217
       at EasyNetQ.RabbitAdvancedBus.<>c__DisplayClass1`1.<Subscribe>b__0(Byte[] body, MessageProperties properties, MessageReceivedInfo messageRecievedInfo) in D:\Source\EasyNetQ\Source\EasyNetQ\RabbitAdvancedBus.cs:line 131
       at EasyNetQ.RabbitAdvancedBus.<>c__DisplayClass6.<Subscribe>b__5(String consumerTag, UInt64 deliveryTag, Boolean redelivered, String exchange, String routingKey, IBasicProperties properties, Byte[] body) in D:\Source\EasyNetQ\Source\EasyNetQ\RabbitAdvancedBus.cs:line 176
       at EasyNetQ.QueueingConsumerFactory.HandleMessageDelivery(BasicDeliverEventArgs basicDeliverEventArgs) in D:\Source\EasyNetQ\Source\EasyNetQ\QueueingConsumerFactory.cs:line 85