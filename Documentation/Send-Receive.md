Whereas the Publish/Subscribe and Request/Response patterns are location transparent, in that you don't need to specify where the consumer of the message is located, the Send/Receive pattern is specifically designed for communication via a named queue. It also makes no assumptions about the types of message that can be sent via the queue. This means that you can send different types of message via the same queue.

The send/receive pattern is ideal for creating 'command pipelines', where you want a buffered channel to a single command processor.

To send a message, use the Send method on IBus.SendReceive, specifying the name of the queue you wish to send the message to and the message itself:
```c#
    bus.SendReceive.Send("my.queue", new MyMessage{ Text = "Hello Widgets!" });
```
To setup a message receiver for a particular message type, use the Receive method on IBus:
```c#
    bus.SendReceive.Receive<MyMessage>("my.queue", message => Console.WriteLine("MyMessage: {0}", message.Text));
```
You can set up multiple receivers for different message types on the same queue by using the Receive overload that takes an Action&lt;IReceiveRegistration&gt;, for example:
```c#
    bus.SendReceive.Receive("my.queue", x => x
        .Add<MyMessage>(message => deliveredMyMessage = message)
        .Add<MyOtherMessage>(message => deliveredMyOtherMessage = message));
```
If a message arrives on a receive queue that doesn't have a matching receiver, EasyNetQ will write the message to the EasyNetQ error queue with an exception saying 'No handler found for message type &lt;message type&gt;'.

_Note: You probably do not want to call bus.Receive more than once for the same queue. This will create a new consumer on the queue and RabbitMQ will round-robin between them. If you are consuming different types on different Receive calls (and thus different consumers), some of your messages will end up on the error queue because EasyNetQ will not find a handler for your message type associated with the consumer on which it is consumed._