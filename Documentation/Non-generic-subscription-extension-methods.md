Since it’s very first version, EasyNetQ has allowed you to subscribe to a message simply by providing a handler for a given message type (and a subscription id, but that’s another discussion).

    bus.Subscribe<MyMessage>("subscriptionId", x => Console.WriteLine(x.Text));

But what do you do if you are discovering the message type at runtime? For example, you might have some system which loads add-ins and wants to subscribe to message types on their behalf. EasyNetQ provides you with non-generic subscription methods just for this purpose.

Just add the this using statement:

using EasyNetQ.NonGeneric;

Which provides you with these extension methods:

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

They are just like the Subscribe methods on IBus except that you provide a Type argument instead of the generic argument, and the message handler is an Action<object> instead of an Action<T>.

Here’s an example of the non-generic subscribe in use:

    var messageType = typeof(MyMessage);
    bus.Subscribe(messageType, "my_subscriptionId", x =>    
        {        
            var message = (MyMessage)x;        
            Console.Out.WriteLine("Got Message: {0}", x.Text);    
        });
