An EasyNetQ subscriber subscribes to a message type (the .NET type of the message class). Once the subscription to a type has been setup by calling the Subscribe method, a persistent queue will be created on the RabbitMQ broker and any messages of that type will be placed on the queue. RabbitMQ will send any messages from the queue to the subscriber whenever it is connected.

To subscribe to a message we need to give EasyNetQ an action to perform whenever a message arrives. We do this by passing subscribe a delegate:

    bus.Subscribe<MyMessage>("my_subscription_id", msg => Console.WriteLine(msg.Text));

Now every time that an instance of MyMessage is published, EasyNetQ will call our delegate and print the message’s Text property to the console.

The subscription id that you pass to Subscribe is important. EasyNetQ will create a unique queue on the RabbitMQ broker for each unique combination of message type and subscription id. The general rule you should follow is that each call to subscribe should have its own id. Indeed, this is a case where you shouldn’t follow normal good practice and where a hard-coded string is probably the best option.

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

## Cancelling subscriptions

All the subscribe methods return an IDisposable. You can cancel a subscriber at any time by calling Dispose on the IDisposable instance:

    var consumer = bus.Subscribe<MyMessage>("sub_id", MyHandler);

    ...

    consumer.Dispose();

This will stop EasyNetQ consuming from the queue and close the consumer's channel.

## Distributed processing out-of-the-box

EasyNetQ and RabbitMQ provide distributed processing out-of-the-box. Say we have written a windows service with a single call to subscribe just like the one above. We deploy it on a server and start it up. When the Subscribe call is run EasyNetQ creates a queue called something like 'someNamespace_myMessage:someAssembly_mySubscriptionId' on the RabbitMQ broker. As instances of MyMessage are published they are routed to this queue and our windows service gets a copy of every message. This is exactly what we want.

Now, what if we deploy a second instance of our windows service on a second server and start it up? When the Subscribe call runs, EasyNetQ will find that there’s already a queue that matches the subscriber id / message type combination, so instead of creating a new queue it will simply start consuming from the existing queue created by the first instance. When RabbitMQ has two consumers consuming from the same queue, it sends messages to the consumers, round-robin style, in turn. So the first message will be sent the the first instance, the second to the second instance and then the third to the first instance, and so on. We get distributed processing out-of-the-box, with no need for any special programming techniques when writing our subscribers, or special software or hardware load balancers.