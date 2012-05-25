EasyNetQ's mission is to provide the simplest possible API for RabbitMQ based messaging. The core IBus interface purposefully avoids exposing [AMQP](http://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) concepts such as exchanges, bindings, and queues, instead it implements a default exchange-binding-queue topology based on message type.

For some scenarios it's useful to be able to configure your own exchange-binding-queue topology; the advanced EasyNetQ API allows you to do that. The advanced API assumes a good understanding of AMQP.

The advanced API is implemented with the [IAdvancedBus](https://github.com/mikehadlow/EasyNetQ/blob/master/Source/EasyNetQ/IAdvancedBus.cs) interface. An instance of this interface can be accessed via the Advanced property of IBus:

    var advancedBus = RabbitHutch.CreateBus("host=localhost").Advanced;

## Creating Exchanges, Bindings, and Queues

You can configure your Exchanges, Bindings and Queues using the classes in the EasyNetQ.Topology namespace. To declare an exchange use one of the static methods of the Exchange class like this:

    // create a direct exchange
    var myDirectExchange = Exchange.DeclareDirect("my.direct.exchange");
    
    // create a topic exchange
    var myTopicExchange = Exchange.DeclareTopic("my.topic.exchagne");
    
    // create a fanout exchange
    var myFanoutExchange = Exchange.DeclareFanout("my.fanout.exchange");

To get the RabbitMQ default exchange do this:

    var exchange = Exchange.GetDefault();

Queues are created in a similar way. To declare a durable queue do this:

    var queue = Queue.DeclareDurable(queueName);

To declare a transient queue do this:

    var queue = Queue.DeclareTransient(queueName);

To declare an 'unnamed' transient queue, where RabbitMQ provides the queue name, simply leave out the queue name:

    var queue = Queue.DeclareTransient();

You bind a queue to an exchange like this:

    var queue = Queue.DeclareDurable("my.queue.name");
    var exchange = Exchange.DeclareDirect("my.exchange.name");
    queue.BindTo(exchange, "my.routing.key");

To specify multiple bindings between a queue and an exchange, simply provide multiple routing keys:

    var queue = Queue.DeclareDurable(queueName);
    var exchange = Exchange.DeclareDirect(exchangeName);
    queue.BindTo(exchange, "a", "b", "c");

You can also bind exchanges to exchanges in a chain:

    var sourceExchange = Exchange.DeclareDirect("source");
    var destinationExchange = Exchange.DeclareDirect("destination");
    var queue = Queue.DeclareDurable(queueName);

    destinationExchange.BindTo(sourceExchange, routingKey);
    queue.BindTo(destinationExchange, routingKey);

## Publishing

The advanced Publish method allows you to specify the exchange you want to publish your message to. It also allows access to the message's AMQP basic properties.

To publish you must first create an exchange (as explained above):

    var exchange = Exchange.DeclareDirect("advanced_test_exchange");

Next create your message. The advanced API requires that your message is wrapped in a Message<T>:

    var myMessage = new MyMessage {Text = "Hello from the publisher"};
    var message = new Message<MyMessage>(myMessage);

The [Message<T>](https://github.com/mikehadlow/EasyNetQ/blob/master/Source/EasyNetQ/IMessage.cs) class gives you access to the AMQP basic properties, for example:

    message.Properties.AppId = "my_app_id";
    message.Properties.ReplyTo = "my_reply_queue";

Finally, open a new channel and publish your message:

    using (var channel = advancedBus.OpenPublishChannel())
    {
        channel.Publish(exchange, routingKey, message);
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