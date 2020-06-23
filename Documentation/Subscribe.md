An EasyNetQ subscriber subscribes to a message type (the .NET type of the message class). Once the subscription to a type has been setup by calling the Subscribe method, a persistent queue will be created on the RabbitMQ broker and any messages of that type will be placed on the queue. RabbitMQ will send any messages from the queue to the subscriber whenever it is connected.

To subscribe to a message we need to give EasyNetQ an action to perform whenever a message arrives. We do this by passing subscribe a delegate:

    bus.Subscribe<MyMessage>("my_subscription_id", msg => Console.WriteLine(msg.Text));

Now every time that an instance of MyMessage is published, RabbitMQ will deliver the message to EasyNetQ, which will call our delegate and print the message’s Text property to the console. Note that the message delivery is only 'acked' once your delegate completes, if it's a long running operation, you will notice that it is marked unacked in the RabbitMQ management UI while the operation is in process. If there is some interruption, such as a power failure or a network interruption the unacked message will remain at the head of the queue. For this reason you should be prepared for multiple copies of the same message to be delivered in some circumstances.

**The subscription id that you pass to Subscribe is important.** EasyNetQ will create a unique queue on the RabbitMQ broker for each unique combination of message type and subscription id.

Each call to Subscribe creates a new queue consumer. If you call Subscribe two times with the same message type and subscription id, you will create two consumers consuming from the same queue. RabbitMQ will then round-robin successive messages to each consumer in turn. This is great for scaling and work-sharing. Say you've created a service that processes a particular message, but it's getting overloaded with work. Simply start a new instance of that service (on the same machine, or a different one) and without having to configure anything, you get automatic scaling.

If you call Subscribe two times with different subscription ids but the same message type, you will create two queues, each with its own consumer. A copy of each message of the given type will be routed to each queue, so each consumer will get all the messages (of that type). This is great if you've got several different services that all care about the same message type.

## Considerations when writing the subscribe callback delegate

As messages are received from queues subscribed to via EasyNetQ, they are placed on an in-memory queue. A single thread sits in a loop taking messages from the queue and calling their Action<TMessage> delegates. Since the delegates are processed one at a time on a single thread, you should avoid long-running synchronous IO operations. Return control from the delegate as soon as possible.

## Use SubscribeAsync

SubscribeAsync allows your subscriber delegate to return a Task immediately and then asynchronously execute long-running IO operations. Once the long-running subscription is complete, simply complete the Task. In the example below we are making a request to a web service using an asynchronous IO operation (DownloadStringTask). When the task completes, we write a line to the console.

    bus.SubscribeAsync<MyMessage>("subscribe_async_test", message => 
        new WebClient().DownloadStringTask(new Uri("http://localhost:1338/?timeout=500"))
            .ContinueWith(task => 
                Console.WriteLine("Received: '{0}', Downloaded: '{1}'", 
                    message.Text, 
                    task.Result)));

**Note that SubscribeAsync will ack the message once the Task completes. Not when the Task is returned from your handler.**

Another example that will result in an exception being thrown if there is a fault which will then result in the message being placed on the default error queue:

    _bus.SubscribeAsync<MessageType>("Queue_Identifier",
                message => Task.Factory.StartNew(() =>
                {
                    // Perform some actions here
                    // If there is a exception it will result in a task complete but task faulted which
                    // is dealt with below in the continuation
                }).ContinueWith(task =>
                {
                    if (task.IsCompleted && !task.IsFaulted)
                    {
                        // Everything worked out ok
                    }
                    else
                    {                        
                        // Don't catch this, it is caught further up the hierarchy and results in being sent to the default error queue
                        // on the broker
                        throw new EasyNetQException("Message processing exception - look in the default error queue (broker)");
                    }
                }));

## Cancelling subscriptions

All the subscribe methods return an [ISubscriptionResult](https://github.com/EasyNetQ/EasyNetQ/blob/master/Source/EasyNetQ/ISubscriptionResult.cs). It contains properties that describe the `IExchange` and `IQueue` used by the underlying [IConsumer](https://github.com/EasyNetQ/EasyNetQ/blob/master/Source/EasyNetQ/Consumer/IConsumer.cs), these can be further manipulated using the advanced API `IAdvancedBus` if required.

You can cancel a subscriber at any time by calling Dispose on the `ISubscriptionResult` instance or on its `ConsumerCancellation` property:

    var subscriptionResult = bus.Subscribe<MyMessage>("sub_id", MyHandler);

    ...

    subscriptionResult.Dispose();
    // this is equivalent to subscriptionResult.ConsumerCancellation.Dispose();

This will stop EasyNetQ consuming from the queue and close the consumer's channel.

Note that disposing of the `IBus` or `IAdvancedBus` instance will also cancel all consumers and close the connection to RabbitMQ.

Do _**not**_ call `subscriptionResult.Dispose()` inside a message handler. This will create a race condition between EasyNetQ ACK'ing the message on the consumer's channel and the `subscriptionResult.Dispose()` call to close that channel. Because of EasyNetQ's internal architecture these calls will be invoked on different threads and the timing is not deterministic.