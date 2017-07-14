The **GetOverview()** method returns an Overview object that has a number of properties describing various aspects of the RabbitMQ broker you are connecting to. For example:

   var overview = management.GetOverview();

            Console.Out.WriteLine("exchangeType.name = {0}", overview.ManagementVersion);

            foreach (var exchangeType in overview.ExchangeTypes)
            {
                Console.Out.WriteLine("exchangeType.name = {0}", exchangeType.Name);
            }

            foreach (var listener in overview.Listeners)
            {
                Console.Out.WriteLine("listener.ip_address = {0}", listener.IpAddress);
            }

            Console.Out.WriteLine("overview.queue_totals = {0}", overview.QueueTotals.Messages);

            foreach (var context in overview.Contexts)
            {
                Console.Out.WriteLine("context.description = {0}", context.Description);
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