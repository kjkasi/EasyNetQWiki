If you are used to handling connections to a relational database like SQL Server, you might find the way EasyNetQ handles connections a little odd. Communication with a relational database is always initiated by the client. The client opens a connection, issues a SQL command, processes the results if necessary, and then closes the connection. The general advice is that you should hold an open connection for as little time as possible and leave connection pooling to the API. 

Talking to a message broker such as RabbitMQ is a little different because the connection tends to last the lifetime of the application. Typically you will open a connection, create a subscription and then wait for any messages to arrive on the open connection. EasyNetQ does not assume that the broker will be available at all times. Instead it employs a lazy connection approach, polling the given endpoint on a background thread until it can connect. If the server disconnects for any reason (maybe a network fault, maybe the RabbitMQ server itself has been bounced), EasyNetQ will revert to polling the endpoint until it can reconnect.

**Standard practice is to create a single IBus instance for the lifetime of your application. Dispose it when your application closes.**

A lazy connection to a RabbitMQ server is represented by an IBus interface. Most EasyNetQ operations are methods on IBus. You create an IBus instance like this:
```c#
var bus = RabbitHutch.CreateBus("myServer”, "1234", ”myVHost", "myUsername", "myPassword");
```
Alternatively, EasyNetQ supports ADO.NET style connection strings in the format:

    host=MyServer;port=ThePortToConnectWith;virtualHost=MyVirtualHost;username=MyUsername;password=MyPassword

For example:
```c#
var bus = RabbitHutch.CreateBus(“host=a;port=1234;virtualHost=b;username=c;password=d”);
```
The only required field is 'host'. If you leave out the port, EasyNetQ will connect with the default port, 5672. If you leave out the virtualHost and simply give a server name, such as ‘localhost’, EasyNetQ will use the default RabbitMQ vHost ‘/’. You can also omit the username and password, in which case EasyNetQ will use the default guest account to connect.

To close the connection, simply dispose the bus like this:
```c#
bus.Dispose();
```
## Logging

EasyNetQ provides a logger interface IEasyNetQLogger:
```c#
public interface IEasyNetQLogger
{
   void DebugWrite(string format, params object[] args);
   void InfoWrite(string format, params object[] args);
   void ErrorWrite(string format, params object[] args);
   void ErrorWrite(Exception exception);
}
```
By default EasyNetQ logs to the console, which is probably not what you want in a production system. You should provide your own implementation of IEasyNetQLogger. The RabbitHutch.CreateBus method provides overloads that take an IEasyNetQLogger which you can use to provide your custom logger to the bus. For example:
```c#
var logger = new MyLogger() // implements IEasyNetQLogger
var bus = RabbitHutch.CreateBus(“my connection string”, logger);
```