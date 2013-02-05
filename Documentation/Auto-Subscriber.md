From v0.7.1.30 EasyNetQ comes with a simple `AutoSubscriber`. You can use it to easily scan a specific assembly for classes that implements the interface `IConsume<T>`, and then let the auto subscriber subscribe these consumers to your bus. You can of course let your consumers handle multiple messages. Lets have a look at some samples.

Lets define a simple consumer, handling two messages: `MessageA` and `MessageB`.

```c#
public class MyConsumer : IConsume<MessageA>, IConsume<MessageB>
{
    public void Consume(MessageA message) { }

    public void Consume(MessageB message) { }
}
```

First create a new instance of AutoSubscriber, passing your IBus instance and a subscriptionId prefix to the constructor. The subscriptionId prefix is prefixed to all auto-generated subscriptionIds, but not to custom subscriptionIds (see below).

To register this, _and all other consumers in the same Assembly_, we just need to pass the assembly that contains your consumers to: `AutoSubscriber.Subscribe(assembly)`. **Note!** This is something you only should do **ONCE**, preferably on application start up.

```c#
var subscriber = new AutoSubscriber(bus, "my_applications_subscriptionId_prefix");
subscriber.Subscribe(Assembly.GetExecutingAssembly());
```

## Specify A Specific SubscriptionId
By default the `AutoSubscriber` will generate a unique `SubscriptionId`. If you would like it to be fixed, you can decorate the `Consume` method with the `ConsumerAttribute`. Why you would make it fixed, is something you can [read up about here](subscribe).

Lets say, the consumer above should have a fixed `SubscriptionId` for the consumer method of `MessageB`. Just decorated it and define a value for `SubscriptionId`.

```c#
[Consumer(SubscriptionId = "MyExplicitId")]
public void Consume(MessageB message) { }
```

## Taking Control Of The SubscriptionId Generation
You could of course also take control of the actual `SubscriptionId` generation. Just replace the `AutoSubscriber.GenerateSubscriptionId : Func<ConsumerInfo, string>`.

```c#
var subscriber = new AutoSubscriber(bus)
{
    CreateConsumer = t => objectResolver.Resolve(t),
    GenerateSubscriptionId = c => AppDomain.CurrentDomain.FriendlyName + c.ConcreteType.Name
};
subscriber.Subscribe(Assembly.GetExecutingAssembly());
```

**Note!** Just a sample implementation. Ensure you have [read and understood](subscribe) the importance of the `SubscriptionId` value.