Whereas the Publish/Subscribe and Request/Response patterns are location transparent, in that you don't need to specify where the consumer of the message is located, the Send/Receive pattern is specifically designed for communication via a named queue. It also makes no assumptions about the types of message that can be sent via the queue. This means that you can send different types of message via the same queue.

The send/receive pattern is ideal for creating 'command pipelines', where you want a buffered channel to a single command processor.

To send a message, use the Send method on IBus, specifying the name of the queue you wish to sent the message to and the message itself:

    bus.Send("my.queue", new MyMessage{ Text = "Hello Widgets!" });

To setup a message receiver for a particular message type, use the Receive method on IBus:

    bus.Receive<MyMessage>(queue, message => Console.WriteLine("MyMessage: {0}", message.Text));

You can set up multiple receivers for different message types on the same queue, for example:

    bus.Receive<MyOtherMessage>(queue, message => Console.WriteLine("MyOtherMessage: {0}", message.Text));

If a message arrives on a receive queue that doesn't have a matching receiver, EasyNetQ will write the message to the EasyNetQ error queue with an exception saying 'No handler found for message type <message type>'.