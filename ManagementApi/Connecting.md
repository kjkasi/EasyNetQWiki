Create a new instance of **EasyNetQ.Management.Client.ManagementClient**:

    var client = new ManagementClient("http://localhost", "my_user_name", "my_password");

This doesn't actually connect to RabbitMQ, an HTTP request (or two) is made for each method that you call on the managementClient instance.

If you want to connect with a port number other than the default (15672), use the optional portNumber parameter:

    var client = new ManagementClient(url, username, password, portNumber: 8080);

If you are using Mono, you must set the optional runningOnMono parameter to true:

    var client = new ManagementClient(url, username, password, runningOnMono: true);

If you need to do extra configuration of the HttpWebRequest (to configure a proxy for example) use the configureRequest action optional parameter:

    var client = new ManagementClient(url, username, password, configureRequest: request => 
            request.Headers.Add("x-not-used", "some_value"));