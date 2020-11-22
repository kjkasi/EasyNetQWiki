The default AMQP publish is not transactional and doesn't guarantee that your message will actually reach the broker. AMQP does specify a transactional publish, but with RabbitMQ it is extremely slow and we haven't supported it via the EasyNetQ API. For high-performance guaranteed delivery it's recommended that you use 'Publisher Confirms'. Simply speaking, this is an extension to AMQP that provides a callback when your message has been successfully received by the broker.

What does 'successfully received' mean? It depends ...

* A transient message is confirmed the moment it is enqueued.
* A persistent message is confirmed as soon as it is persisted to disk, or when it is consumed on every queue.
* An unroutable transient message is confirmed directly it is published.

For more information on publisher confirms, [please read the announcement on the RabbitMQ blog](http://www.rabbitmq.com/blog/2011/02/10/introducing-publisher-confirms/)

Enable publisher confirms by setting publisherConfirms=true on the connection string:

    bus = RabbitHutch.CreateBus("host=localhost;publisherConfirms=true;timeout=10");

The synchronous bus.Publish(..) method will wait for the confirm before returning. A failure to confirm before the timeout period (also configured in the connection string) will cause an exception to be thrown. With publisher confirms on the synchronous publish method will slow down significantly. If performance is a concern, you should consider using the PublishAsync method:

    bus.PubSub.PublishAsync(new MyMessage
        {
            Text = "Hello World"
        }).ContinueWith(task =>
            {
                // this only checks that the task finished
                // IsCompleted will be true even for tasks in a faulted state
                // we use if (task.IsCompleted && !task.IsFaulted) to check for success
                if (task.IsCompleted) 
                {
                    //Console.Out.WriteLine("{0} Completed", count);
                }
                if (task.IsFaulted)
                {
                    Console.Out.WriteLine("\n\n");
                    Console.Out.WriteLine(task.Exception);
                    Console.Out.WriteLine("\n\n");
                }
            });

This will return before the confirmation is received. The task will complete in a faulted state if no confirmation or a NACK confirmation is received.