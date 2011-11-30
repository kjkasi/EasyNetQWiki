The simplest messaging pattern that EasyNetQ supports is Publish/Subscribe. This pattern is an excellent way of de-coupling providers of information from consumers. The publisher simply says, ‘This has happened’ or ‘I now have this information’, to the world. It doesn’t care if anyone is listening or who they might be or where they are located. We can add and remove subscribers for a particular message type without any reconfiguration of the publisher. We can also have many publishers publishing the same message, again adding and removing publishers without having to reconfigure any of the other publishers or subscribers.

Publishing is easy with EasyNetQ. Just create an instance of your message, it can be any serializable .NET type, and call Publish like this:

    var message = new MyMessage { Text = “Hello Rabbit” };
    try {
        bus.Publish(message);
    }
    catch(EasyNetQException) {
        // the server is not connected
    }

Note that we have wrapped the call to Publish in a try-catch block. As explained in the previous section, EasyNetQ employs a lazy connection approach so that it can automatically recover from connection failure, this means that a connection might not be available when you call publish. You should handle the EasyNetQException appropriately.
