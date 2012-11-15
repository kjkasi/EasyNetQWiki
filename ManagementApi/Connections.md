**GetConnections()** will return a list of currently open connections to the broker. For example:

    var connections = managementClient.GetConnections();

    foreach (var connection in connections)
    {
        Console.Out.WriteLine("connection.name = {0}", connection.name);
        Console.WriteLine("user:\t{0}", connection.client_properties.user);
        Console.WriteLine("application:\t{0}", connection.client_properties.application);
        Console.WriteLine("client_api:\t{0}", connection.client_properties.client_api);
        Console.WriteLine("application_location:\t{0}", connection.client_properties.application_location);
        Console.WriteLine("connected:\t{0}", connection.client_properties.connected);
        Console.WriteLine("easynetq_version:\t{0}", connection.client_properties.easynetq_version);
        Console.WriteLine("machine_name:\t{0}", connection.client_properties.machine_name);
    }

Outputs the following on a system with a single consumer:

    connection.name = [::1]:64754 -> [::1]:5672
    user:	guest
    application:	EasyNetQ.Tests.Performance.Consumer.exe
    client_api:	EasyNetQ
    application_location:	D:\Source\EasyNetQ\Source\EasyNetQ.Tests.Performance.Consumer\bin\Debug
    connected:	14/11/2012 15:06:19
    easynetq_version:	0.9.0.0
    machine_name:	THOMAS

**CloseConnection()** will close a connection. For example, this code closes the first connection found by the GetConnections() method:

    var connections = managementClient.GetConnections();
    managementClient.CloseConnection(connections.First());

