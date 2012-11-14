To connect the management API, create a new instance of EasyNetQ.Management.Client.ManagementClient:

    IManagementClient managementClient = new ManagementClient("http://localhost", "my_user_name", "my_password");

This doesn't actually connect to RabbitMQ. An HTTP request (or two) is made for each method that you call on the managementClient instance.