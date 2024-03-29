From v0.7.1.30 EasyNetQ comes with a simple `AutoSubscriber`. You can use it to easily scan a specific assembly for classes that implement either of the interfaces `IConsume<T>` or `IConsumeAsync<T>`, and then let the auto subscriber subscribe these consumers to your bus. You can of course let your consumers handle multiple messages. Let's have a look at some samples.

**Note**: From version 0.13.0 all the AutoSubscriber classes are in the EasyNetQ.AutoSubscribe namespace, so please add the following using statement:

    using EasyNetQ.AutoSubscribe;

Lets define a simple consumer, handling three messages: `MessageA`, `MessageB` and `MessageC`.

```c#
public class MyConsumer : IConsume<MessageA>, IConsume<MessageB>, IConsumeAsync<MessageC>
{
    public void Consume(MessageA message) {...}

    public void Consume(MessageB message) {...}

    public Task ConsumeAsync(MessageC message, CancellationToken cancellationToken) {...}
}
```

First create a new instance of AutoSubscriber, passing your IBus instance and a subscriptionId prefix to the constructor. The subscriptionId prefix is prefixed to all auto-generated subscriptionIds, but not to custom subscriptionIds (see below).

To register this, _and all other consumers in the same Assembly_, we just need to pass the types from assembly that contains your consumers to: `AutoSubscriber.Subscribe(assembly.types)`. **Note!** This is something you only should do **ONCE**, preferably on application start up.

```c#
var subscriber = new AutoSubscriber(bus, "my_applications_subscriptionId_prefix");
subscriber.Subscribe(Assembly.GetExecutingAssembly().GetTypes());
```

## Subscribe by topic(s)
By default the `AutoSubscriber` will bind without topics. In the example below MessageA is registered with two topics.

**Note!** If you run the code without ForTopic attribute it will have a routing key of "#" which will pick up any subscribe for the message type. Assuming default ports and admin plugin installed, simply visit http://localhost:15672/#/queues and unbind routing if needed.

```c#
public class MyConsumer : IConsume<MessageA>, IConsume<MessageB>, IConsumeAsync<MessageC>
{
    [ForTopic("Topic.Foo")]
    [ForTopic("Topic.Bar")]
    public void Consume(MessageA message) {...}

    public void Consume(MessageB message) {...}

    public Task Consume(MessageC message) {...}
}

//To publish by topic
var bus = RabbitHutch.CreateBus("host=localhost");

var msg1 = new MessageA(msg1, "Topic.Foo");   //picked up
var msg2 = new MessageA(msg2, "Topic.Bar");   //picked up
var msg3 = new MessageA(msg3);                //not picked up
```

## Specify A Specific SubscriptionId
By default the `AutoSubscriber` will generate a unique `SubscriptionId` per each Message/Consumer combination. This means that you will start multiple instances of the same consumer and they will read from the same queues in a round-robin fashion (worker pattern). 

If you would like subscription id's to be fixed, you can decorate the `Consume` method with the `AutoSubscriberConsumerAttribute`. Why you would make it fixed, is something you can [read up about here](subscribe).

Lets say, the consumer above should have a fixed `SubscriptionId` for the consumer method of `MessageB`. Just decorated it and define a value for `SubscriptionId`.

```c#
[AutoSubscriberConsumer(SubscriptionId = "MyExplicitId")]
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

## Taking Control Of The Consumer Configuration Setting
When using the autosubscriber to subscribe to queues you can set ISubscriptionConfiguration values, such as AutoDelete, Priority etc.

Either by setting an action when you create the AutoSubscriber.
```c#
var subscriber = new AutoSubscriber(bus)
{    
    ConfigureSubscriptionConfiguration = c => c.WithAutoDelete()
                                               .WithPriority(10)
};
subscriber.Subscribe(Assembly.GetExecutingAssembly().GetTypes());
```

Or alternatively you can apply an attribute to the consume method which would take precedence over any configuration values set by a ConfigureSubscriptionConfiguration action.

```c#
public class MyConsumer : IConsume<MessageA>
{
    [SubscriptionConfiguration(CancelOnHaFailover = true, PrefetchCount = 10)]
    public void Consume(MessageA message) {...}
}
```

## Using an IoC container with AutoSubscriber

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

autoSubscriber.SubscribeAsync(GetType().Assembly);
```

Now each time a message arrives a new instance of our consumer will be resolved from our container.