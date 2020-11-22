Since it’s very first version, EasyNetQ has allowed you publish a message:
```c#
    bus.Publish<MyMessage>(theMessage);
```
But what do you do if you are discovering the message type at runtime?EasyNetQ provides you with non-generic extensions methods just for this purpose(for more information please look at `NonGenericRpcExtensions`, `NonGenericPubSubExtensions`, `NonGenericSendReceiveExtensions`, `NonGenericSchedulerExtensions`).
	 
They are just like the Publish() method on IBus.PubSub except that you provide a Type argument instead of the generic argument.

Here's an example of the non-generic publish in use:
```c#
    bus.Publish(theMessage, typeof(MyMessage));
```