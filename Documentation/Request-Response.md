EasyNetQ also supports a Request/Response messaging pattern. This makes it easy to implement client/server applications where the client makes a request to a server which then processes the request and returns a response. Unlike traditional RPC mechanisms, an EasyNetQ request/response operation doesn’t have a name, but is simply defined by the request/response message type pair.

Also, unlike traditional RPC mechanisms, including most web service toolkits, EasyNetQ’s request/response pattern is based on messaging, so it is asynchronous out-of-the-box.

## Making a request, and handling the response

To make a request with EasyNetQ, simply do the following:

    var myRequest = new MyRequest { Text = “Hello Server” };
    bus.Request<MyRequest, MyResponse>(myRequest, response => Console.WriteLine(“Got response: {0}”, response.Text));

Here we create a new request of type MyMessage and then call the IBus.Request method with it as the first argument. When the response returns, at some later time, on some later thread, the response message’s Text property is output to the console.

## Beware of closures!

It’s a common pattern to subscribe to some message, and then during the processing of that message make a request/response call.

    bus.Subscribe<MyInitialMessage>(“myid”,  msg => 
    {
        var myRequest = new MyRequest { Text = “blah” };
        bus.Request<MyRequest, MyResponse>(myRequest, response => 
        {
            DoSomeProcessing(response, msg);
        }
    }

Note that the method DoSomeProcessing takes msg (in bold) from the outer scope; it ‘closes over’ msg. This will not work as expected. You will notice that the msg variable used inside the response callback is always the first msg that arrives. The reason for this is that EasyNetQ does not create a new response subscription every time Request is called. Instead it caches a single instance of response callback and uses it to handle every response.

## Responding to requests

To write a server that responds to requests, simply use the IBus.Respond method like this:

    bus.Respond<MyRequest, MyResponse>(request => new MyResponse { Text = “Responding to “ + request.Text});

Respond takes a single argument, a Func<TRequest, TResponse>, that takes a request and returns a response. The same advice that applies to Subscription callbacks also applies to responders. Do not block on long-running IO operations. If you want to do long-running IO, use RespondAsync instead.

## Responding Asynchronously

EasyNetQ also provides a RespondAsync method that takes a Func<TRequest, Task<TResponse>> delegate. This allows you to execute long-running IO-bound operations without blocking the EasyNetQ subscription handling loop.

    needs an example using RespondAsync
