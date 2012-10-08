If you are used to handling connections to a relational database like SQL Server, you might find the way EasyNetQ handles connections a little odd. Communication with a relational database is always initiated by the client. The client opens a connection, issues a SQL command, processes the results if necessary, and then closes the connection. The general advice is that you should hold an open connection for as little time as possible and leave connection pooling to the API. 

Talking to a message broker such as RabbitMQ is a little different because the connection tends to last the lifetime of the application. Typically you will open a connection, create a subscription and then wait for any messages to arrive on the open connection. EasyNetQ does not assume that the broker will be available at all times. Instead it employs a lazy connection approach, polling the given endpoint on a background thread until it can connect. If the server disconnects for any reason (maybe a network fault, maybe the RabbitMQ server itself has been bounced), EasyNetQ will revert to polling the endpoint until it can reconnect.

**Standard practice is to create a single IBus instance for the lifetime of your application. Dispose it when your application closes.**

A lazy connection to a RabbitMQ server is represented by an IBus interface. Most EasyNetQ operations are methods on IBus. You create an IBus instance like this:

    var bus = RabbitHutch.CreateBus(“host=myServer;virtualHost=myVirtualHost;username=mike;password=topsecret”);

The connection string is made up of key/value pairs in the format key=value, each one separated by a semicolon (;). The only required field is 'host'. The possible connection string values are:

* **host** (e.g. host=localhost or host=192.168.2.56 or host=myhost.mydomain.com) this field is required. To specify the port you want to connect to, you can use the standard format host:port (e.g. host=myhost.com:5673). If you omit the port number, the default AMQP port is used (5672). To connect to a RabbitMQ cluster, specify each cluster node separated by commas (e.g. host=myhost1.com,myhost2.com,myhost3.com). See [[Cluster Support]] for more details.
* **virtualHost** (e.g. virtualHost=myVirtualHost) default is the default virtual host '/'
* **username** (e.g. username=mike) default is 'guest'
* **password** (e.g. password=mysecret) default is 'guest'
* **requestedHeartbeat** (e.g. requestedHeartbeat=10) default is zero for no heartbeat.
* **prefetchcount** (e.g. prefetchcount=1) default is 50. This is the number of messages that will be delivered by RabbitMQ before an ack is sent by EasyNetQ. Set to 0 for infinite prefetch (not recommended). Set to 1 for fair work balancing among a farm of consumers.

To close the connection, simply dispose the bus like this:

    bus.Dispose();

## Logging

EasyNetQ provides a logger interface IEasyNetQLogger:

    public interface IEasyNetQLogger
    {
       void DebugWrite(string format, params object[] args);
       void InfoWrite(string format, params object[] args);
       void ErrorWrite(string format, params object[] args);
       void ErrorWrite(Exception exception);
    }

By default EasyNetQ logs to the console, which is probably not what you want in a production system. You should provide your own implementation of IEasyNetQLogger. The RabbitHutch.CreateBus method provides overloads that allow you to replace any of the EasyNetQ components. You can use this to provide your custom logger to the bus. For example:

    var logger = new MyLogger() // implements IEasyNetQLogger
    var bus = RabbitHutch.CreateBus(“my connection string”, x => x.Register<IEasyNetQLogger>(_ => logger));

For more detail on replacing EasyNetQ components with your own versions, see [[Replacing EasyNetQ Components]].