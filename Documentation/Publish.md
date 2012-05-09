The simplest messaging pattern that EasyNetQ supports is Publish/Subscribe. This pattern is an excellent way of de-coupling providers of information from consumers. The publisher simply says, ‘This has happened’ or ‘I now have this information’, to the world. It doesn’t care if anyone is listening or who they might be or where they are located. We can add and remove subscribers for a particular message type without any reconfiguration of the publisher. We can also have many publishers publishing the same message, again adding and removing publishers without having to reconfigure any of the other publishers or subscribers.

To publish with EasyNetQ (assuming you've already created an IBus instance):

1. Create an instance of your message, it can be any serializable .NET type.
2. Open an IPublishChannel by calling OpenPublishChannel. Do not share IPublishChannel between threads. You should also dispose of IPublishChannel when you have finished using it. The easiest way to do this is to wrap it in a using statement. Note that if you are publishing a lot of messages in a tight loop, it's better to create a single IPublishChannel for all your Publish calls. Creating a channel has some overhead.
3. Call the Publish method on IPublishChannel passing it your message instance.

    var message = new MyMessage { Text = “Hello Rabbit” };
    try 
    {
        using (var publishChannel = bus.OpenPublishChannel())
        {
            publishChannel.Publish(message);
        }    
    }
    catch(EasyNetQException) 
    {
        // the server is not connected
    }

Note that we have wrapped the call to Publish in a try-catch block. As explained in the previous section, EasyNetQ employs a lazy connection approach so that it can automatically recover from connection failure, this means that a connection might not be available when you call publish. You should handle the EasyNetQException appropriately.
