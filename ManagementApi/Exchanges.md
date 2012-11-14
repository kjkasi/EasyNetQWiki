**GetExchanges()** will return a list of exchanges. For example:

    var exchanges = managementClient.GetExchanges();

    foreach (Exchange exchange in exchanges)
    {
        Console.Out.WriteLine("exchange.name = {0}", exchange.name);
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

