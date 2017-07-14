**GetExchanges()** will return a list of exchanges. For example:

    var exchanges = managementClient.GetExchanges();

    foreach (Exchange exchange in exchanges)
    {
        Console.Out.WriteLine("exchange.Name = {0}", exchange.Name);
    }

Will output:

    exchange.name = 
    exchange.name = EasyNetQ_Tests_MyMessage:EasyNetQ_Tests
    exchange.name = EasyNetQ_Tests_TestPerformanceMessage:EasyNetQ_Tests_Messages
    exchange.name = ErrorExchange_originalRoutingKey
    exchange.name = amq.direct
    exchange.name = amq.fanout
    exchange.name = amq.headers
    exchange.name = amq.match
    exchange.name = amq.rabbitmq.log
    exchange.name = amq.rabbitmq.trace
    exchange.name = amq.topic
    exchange.name = management_api_test_exchange
    exchange.name = 
    exchange.name = amq.direct
    exchange.name = amq.fanout
    exchange.name = amq.headers
    exchange.name = amq.match
    exchange.name = amq.rabbitmq.trace
    exchange.name = amq.topic

***

**CreateExchange(ExchangeInfo, VHost)*** Will create an exchange on the given virtual host. For example:

    var vhost = managementClient.GetVhost("my_vhost");
    var exchangeInfo = new ExchangeInfo("my_exchange", "direct");

    var exchange = managementClient.CreateExchange(exchangeInfo, vhost);

***

**DeleteExchange(Exchange)** will delete an exchange. For example (assuming we still have the vhost and exchange from the above example):

    managementClient.DeleteExchange(exchange);

***

**GetBindingsWithSource(Exchange)** will get a list of the bindings that have the exchange as their source. For example:

    var bindings = managementClient.GetBindingsWithSource(exchange);

***

**GetBindingsWithDestination(Exchange)** will get a list of the bindings that have the exchange as their destination (for exchange to exchange bindings)

    var bindings = managementClient.GetBindingsWithDestination(exchange);

***

**Publish(Exchange, PublishInfo)** will publish a test message to an exchange. 

Please note that the publish / get paths in the HTTP API are intended for injecting test messages, diagnostics etc - they do not implement reliable delivery and so should be treated as a sysadmin's tool rather than a general API for messaging. For a general messaging API please use EasyNetQ.

    var publishInfo = new PublishInfo(testQueue, "Hello World");
    var result = managementClient.Publish(exchange, publishInfo);

The result.routed property will be true if the messages was successfully routed to a queue.
