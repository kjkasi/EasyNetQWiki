**EasyNetQ.Client.Management** is a thin C# client for the [RabbitMQ Management HTTP API](http://hg.rabbitmq.com/rabbitmq-management/raw-file/3646dee55e02/priv/www-api/help.html). Before you can use it, you have to enable the [RabbitMQ Management Plugin](http://www.rabbitmq.com/management.html).

Using the management API it's possible to automate various management, monitoring and set-up tasks including:

* Declare, list and delete exchanges, queues, bindings, users, virtual hosts and permissions.
* Monitor queue length, message rates globally and per channel, data rates per connection, etc.
* Force close connections, purge queues.

To install **EasyNetQ.Client.Management** use NuGet:

    PM> Install-Package EasyNetQ.Management.Client

To give an overview of the sort of things you can do with **EasyNetQ.Client.Management**, have a look at this code which walks through a scenario creating a new virtual host, user, some permissions and then creating an exchange, a queue, and binding them together. Finally it publishes a test message to the exchange and checks that it has arrived on the queue:

    var initial = new ManagementClient("http://localhost", "guest", "guest");

	// first create a new virtual host
	var vhost = await initial.CreateVhostAsync("my_virtual_host");

	// next create a user for that virutal host
	var user = await initial.CreateUserAsync(new UserInfo("mike", "topSecret"));

	// give the new user all permissions on the virtual host
	await initial.CreatePermissionAsync(new PermissionInfo(user, vhost));

	// now log in again as the new user
	var management = new ManagementClient("http://localhost", user.Name, "topSecret");

	// test that everything's OK
	await management.IsAliveAsync(vhost);

	// create an exchange
	var exchange = await management.CreateExchangeAsync(new ExchangeInfo("my_exchagne", "direct"), vhost);

	// create a queue
	var queue = await management.CreateQueueAsync(new QueueInfo("my_queue"), vhost);

	// bind the exchange to the queue
	await management.CreateBindingAsync(exchange, queue, new BindingInfo("my_routing_key"));

	// publish a test message
	await management.PublishAsync(exchange, new PublishInfo("my_routing_key", "Hello World!"));

	// get any messages on the queue
	var messages = await management.GetMessagesFromQueueAsync(queue, new GetMessagesCriteria(1, Ackmodes.ack_requeue_false));

	foreach (var message in messages)
	{
	    Console.Out.WriteLine("message.payload = {0}", message.Payload);
	}