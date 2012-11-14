The GetOverview method returns an Overview object that has a number of properties describing various aspects of the RabbitMQ broker you are connecting to. For example:

    var overview = managementClient.GetOverview();

    Console.Out.WriteLine("overview.management_version = {0}", overview.management_version);
    foreach (var exchangeType in overview.exchange_types)
    {
        Console.Out.WriteLine("exchangeType.name = {0}", exchangeType.name);
    }
    foreach (var listener in overview.listeners)
    {
        Console.Out.WriteLine("listener.ip_address = {0}", listener.ip_address);
    }

    Console.Out.WriteLine("overview.queue_totals = {0}", overview.queue_totals.messages);

    foreach (var context in overview.contexts)
    {
        Console.Out.WriteLine("context.description = {0}", context.description);
    }

Will output ...

    overview.management_version = 2.8.6
    exchangeType.name = topic
    exchangeType.name = fanout
    exchangeType.name = direct
    exchangeType.name = headers
    listener.ip_address = 0.0.0.0
    listener.ip_address = ::
    overview.queue_totals = 0
    context.description = RabbitMQ Management