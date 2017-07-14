**GetBindings()** returns a list of all bindings on a broker:

    var bindings = managementClient.GetBindings();

    foreach (var binding in bindings)
    {
        Console.Out.WriteLine("binding.Destination = {0}", binding.Destination);
        Console.Out.WriteLine("binding.Source = {0}", binding.Source);
        Console.Out.WriteLine("binding.Properties_key = {0}", binding.PropertiesKey);
    }

Outputs something like this:

    binding.Destination = EasyNetQ_Tests_MyMessage:EasyNetQ_Tests_test
    binding.Source = EasyNetQ_Tests_MyMessage:EasyNetQ_Tests
    binding.PropertiesKey = %23
    binding.Destination = EasyNetQ_Tests_TestPerformanceMessage:EasyNetQ_Tests_Messages_consumer
    binding.Source = EasyNetQ_Tests_TestPerformanceMessage:EasyNetQ_Tests_Messages
    binding.PropertiesKey = %23
    binding.Destination = EasyNetQ_Default_Error_Queue

***

**GetBindings(Exchange, Queue)** returns all the bindings between an exchange and a queue:

    var bindings = managementClient.GetBindings(exchange, queue);

***

**CreateBinding(Exchange, Queue, BindingInfo)** creates a binding between an exchange and a queue. A single exchange-queue pair can have multiple bindings. Very useful for complex routing scenarios.

    var vhost = managementClient.GetVhost("/");
    var queue = managementClient.GetQueue("my_queue", vhost);
    var exchange = managementClient.GetExchange("my_exchange", vhost);

    var bindingInfo = new BindingInfo("my.routing.key");

    managementClient.CreateBinding(exchange, queue, bindingInfo);

***

**DeleteBinding(Binding)** will delete a binding. For example, to delete all the bindings between an exchange and a queue:

    var vhost = managementClient.GetVhost("/");
    var queue = managementClient.GetQueue(testQueue, vhost);
    var exchange = managementClient.GetExchange(testExchange, vhost);

    var bindings = managementClient.GetBindings(exchange, queue);

    foreach (var binding in bindings)
    {
        managementClient.DeleteBinding(binding);
    }

