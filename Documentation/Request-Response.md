EasyNetQ also supports a Request/Response messaging pattern. This makes it easy to implement client/server applications where the client makes a request to a server which then processes the request and returns a response. Unlike traditional RPC mechanisms, an EasyNetQ request/response operation doesn’t have a name, but is simply defined by the request/response message type pair.

Also, unlike traditional RPC mechanisms, including most web service toolkits, EasyNetQ’s request/response pattern is based on messaging, so it is asynchronous out-of-the-box.

## Making a request, and handling the response

To make a request with EasyNetQ, call the Request method on IBus:
```c#
    var myRequest = new MyRequest { Text = “Hello Server” };
    var response = bus.Rpc.Request<MyRequest, MyResponse>(myRequest);
    Console.WriteLine(response.Text);
```
Here we create a new request of type MyRequest and then call the Request method with the message as the argument. When the response returns the response message’s Text property is output to the console.

## Asynchronous request

Messaging is by nature asynchronous. You send a message, then allow your program to continue with its other tasks. At some point in the future, you receive the response. With the synchronous Request method shown above, your thread will block until the response is returned. It is usually a better choice to use the RequestAsync method that returns a task:
```c#
    var task = bus.Rpc.RequestAsync<TestRequestMessage, TestResponseMessage>(request)
    task.ContinueWith(response => {
        Console.WriteLine("Got response: '{0}'", response.Result.Text);
    });
```
## Responding to requests

To write a server that responds to requests, simply use the IBus.Respond method like this:
```c#
    bus.Rpc.Respond<MyRequest, MyResponse>(request => new MyResponse { Text = “Responding to “ + request.Text});
```
Respond takes a single argument, a `Func<TRequest, TResponse>`, that takes a request and returns a response. The same advice that applies to Subscription callbacks also applies to responders. Do not block on long-running IO operations. If you want to do long-running IO, use RespondAsync instead.

## Responding Asynchronously

EasyNetQ also provides a RespondAsync method that takes a `Func<TRequest, Task<TResponse>>` delegate. This allows you to execute long-running IO-bound operations without blocking the EasyNetQ subscription handling loop.
```c#
    static void Main(string[] args)
        {
            // create a group of worker objects
            var workers = new BlockingCollection<MyWorker>();
            for (int i = 0; i < 10; i++)
            {
                workers.Add(new MyWorker());
            }
            // create the bus
            var bus = RabbitHutch.CreateBus("host=localhost");
            // respond to requests
            bus.RespondAsync<RequestServerTime, ResponseServerTime>(request =>
                Task.Factory.StartNew(() =>
                {
                    var worker = workers.Take();
                    try
                    {
                        return worker.Execute(request);
                    }
                    finally
                    {
                        workers.Add(worker);
                    }
                }));
            Console.ReadLine();
            bus.Dispose();
        }
```
## Example application
EasyNetQ example showing Request Response and Autosubcriber, wired up using Windsor IOC

https://bitbucket.org/philipogorman/createrequestservice/src