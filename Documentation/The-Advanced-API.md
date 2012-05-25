EasyNetQ's mission is to provide the simplest possible API for RabbitMQ based messaging. The core IBus interface purposefully avoids exposing AMQP concepts such as exchanges, bindings, and queues, instead it implements a default exchange-binding-queue topology based on message type.

For some scenarios it's useful to be able to configure your own exchange-binding-queue topology; the advanced EasyNetQ API allows you to do that.

The advanced API is implemented with the IAdvancedBus interface. An instance of this interface can be accessed via the Advanced property of IBus:

    var advancedBus = RabbitHutch.CreateBus("host=localhost").Advanced;

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