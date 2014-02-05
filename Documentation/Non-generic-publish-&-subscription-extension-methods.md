Since it’s very first version, EasyNetQ has allowed you to subscribe to, and publish a message simply by providing a handler for a given message type (and a subscription id, but that’s another discussion).

    bus.Subscribe<MyMessage>("subscriptionId", x => Console.WriteLine(x.Text));
		
and

	bus.Publish<MyMessage>(theMessage);

But what do you do if you are discovering the message type at runtime? For example, you might have some system which loads add-ins and wants to subscribe to message types on their behalf. EasyNetQ provides you with non-generic publish and subscription methods just for this purpose.

Just add this using statement:

    using EasyNetQ.NonGeneric;

Which provides you with these subscription extension methods:

    public static IDisposable Subscribe(this IBus bus, Type messageType, string subscriptionId, Action<object> onMessage)
    public static IDisposable Subscribe(
        this IBus bus,
        Type messageType,
        string subscriptionId,
        Action<object> onMessage,
        Action<ISubscriptionConfiguration> configure)
    public static IDisposable SubscribeAsync(    
        this IBus bus,    
        Type messageType,    
        string subscriptionId,    
        Func<object, Task> onMessage)
    public static IDisposable SubscribeAsync(    
        this IBus bus,     
        Type messageType,     
        string subscriptionId,     
        Func<object, Task> onMessage,     
        Action<ISubscriptionConfiguration> configure)

...and these publish extension methods:

	 public static void Publish(this IBus bus, Type messageType, object message)
	 public static void Publish(this IBus bus, Type messageType, object message, string topic)
	 public static Task PublishAsync(this IBus bus, Type messageType, object message)
	 public static Task PublishAsync(this IBus bus, Type messageType, object message, string topic)
	 
	 
They are just like the Publish and Subscribe methods on IBus except that you provide a Type argument instead of the generic argument, and the message handler is an Action&lt;object&gt; instead of an Action&lt;T&gt;.

Here’s an example of the non-generic subscribe in use:

    var messageType = typeof(MyMessage);
    bus.Subscribe(messageType, "my_subscriptionId", x =>    
        {        
            var message = (MyMessage)x;        
            Console.Out.WriteLine("Got Message: {0}", x.Text);    
        });

Here's an example of the non-generic publish in use:

	var messageType = typeof(MyMessage);
	bus.Publish(messageType, theMessage);