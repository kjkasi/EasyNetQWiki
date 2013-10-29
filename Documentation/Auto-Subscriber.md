From v0.7.1.30 EasyNetQ comes with a simple `AutoSubscriber`. You can use it to easily scan a specific assembly for classes that implement either of the interfaces `IConsume<T>` or `IConsumeAsync<T>`, and then let the auto subscriber subscribe these consumers to your bus. An implementation of `IConsume<T>` will use the buses Subscribe method whilst implementations of `IConsumeAsync<T>` will use the buses SubscribeAsync method, see [[Subscribe]] for details. You can of course let your consumers handle multiple messages. Lets have a look at some samples.

**Note**: From version 0.13.0 all the AutoSubscriber classes are in the EasyNetQ.AutoSubscribe namespace, so please add the following using statement:

    using EasyNetQ.AutoSubscribe;

Lets define a simple consumer, handling three messages: `MessageA`, `MessageB` and `MessageC`.

```c#
public class MyConsumer : IConsume<MessageA>, IConsume<MessageB>, IConsumeAsync<MessageC>
{
    public void Consume(MessageA message) {...}

    public void Consume(MessageB message) {...}

    public Task Consume(MessageC message) {...}
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

##Using an IoC container with AutoSubscriber

AutoSubscriber has a property, MessageDispatcher, which allows you to plug in your own message dispatching code. This allows you to resolve your consumers from an IoC container or do other custom dispatch time tasks.

Let's write a custom IAutoSubscriberMessageDispatcher to resolve consumers from the [Windsor IoC container](http://docs.castleproject.org/Windsor.MainPage.ashx)

```C#
public class WindsorMessageDispatcher : IAutoSubscriberMessageDispatcher
{
    private readonly IWindsorContainer container;

    public WindsorMessageDispatcher(IWindsorContainer container)
    {
        this.container = container;
    }

    public void Dispatch<TMessage, TConsumer>(TMessage message) where TMessage : class where TConsumer : IConsume<TMessage>
    {
        var consumer = container.Resolve<TConsumer>();
        try
        {
            consumer.Consume(message);
        }
        finally
        {
            container.Release(consumer);
        }
    }

    public Task DispatchAsync<TMessage, TConsumer>(TMessage message) where TMessage: class where TConsumer: IConsumeAsync<TMessage>
    {
        var consumer = _container.Resolve<TConsumer>();           
        return consumer.Consume(message).ContinueWith(t=>_container.Release(consumer));            
    }
}
```

Now we need to register our consumer with our IoC container:

```c#
var container = new WindsorContainer();
container.Register(
    Component.For<MyConsumer>().ImplementedBy<MyConsumer>()
    );

```

Next setup the AutoSubscriber with our custom IMessageDispatcher:

```c#
var bus = RabbitHutch.CreateBus("host=localhost");

var autoSubscriber = new AutoSubscriber(bus, "My_subscription_id_prefix")
{
    MessageDispatcher = new WindsorMessageDispatcher(container)
};
autoSubscriber.Subscribe(GetType().Assembly);
autoSubscriber.SubscribeAsync(GetType().Assembly);
```

Now each time a message arrives a new instance of our consumer will be resolved from our container.