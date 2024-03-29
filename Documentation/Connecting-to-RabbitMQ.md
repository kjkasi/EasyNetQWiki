If you are used to handling connections to a relational database like SQL Server, you might find the way EasyNetQ handles connections a little odd. Communication with a relational database is always initiated by the client. The client opens a connection, issues a SQL command, processes the results if necessary, and then closes the connection. The general advice is that you should hold an open connection for as little time as possible and leave connection pooling to the API. 

Talking to a message broker such as RabbitMQ is a little different because the connection tends to last the lifetime of the application. Typically you will open a connection, create a subscription and then wait for any messages to arrive on the open connection. EasyNetQ does not assume that the broker will be available at all times. Instead it employs a lazy connection approach, polling the given endpoint on a background thread until it can connect. If the server disconnects for any reason (maybe a network fault, maybe the RabbitMQ server itself has been bounced), EasyNetQ will revert to polling the endpoint until it can reconnect.

**Standard practice is to create a single IBus instance for the lifetime of your application. Dispose it when your application closes.**

A lazy connection to a RabbitMQ server is represented by an IBus interface. Most EasyNetQ operations are methods on IBus. You create an IBus instance like this:

    var bus = RabbitHutch.CreateBus(“host=myServer;virtualHost=myVirtualHost;username=mike;password=topsecret”);

The connection string is made up of key/value pairs in the format key=value, each one separated by a semicolon (;). The only required field is 'host'. The possible connection string values are:

* **host** (e.g. host=localhost or host=192.168.2.56 or host=myhost.mydomain.com) this field is required. To specify the port you want to connect to, you can use the standard format host:port (e.g. host=myhost.com:5673). If you omit the port number, the default AMQP port is used (5672). To connect to a RabbitMQ cluster, specify each cluster node separated by commas (e.g. host=myhost1.com,myhost2.com,myhost3.com). See [[Cluster Support]] for more details.
* **virtualHost** (e.g. virtualHost=myVirtualHost) default is the default virtual host '/'
* **username** (e.g. username=mike) default is 'guest' (for non 'localhost' host you need other user than 'guest')
* **password** (e.g. password=mysecret) default is 'guest'
* **requestedHeartbeat** (e.g. requestedHeartbeat=10) default is 10 seconds. Set to zero for no heartbeat.
* **prefetchcount** (e.g. prefetchcount=1) default is 50. This is the number of messages that will be delivered by RabbitMQ before an ack is sent by EasyNetQ. Set to 0 for infinite prefetch (not recommended). Set to 1 for fair work balancing among a farm of consumers.
* **publisherConfirms** (e.g. publisherConfirms=true) default is false. This turns on [[Publisher Confirms]].
* **persistentMessages** (e.g. persistentMessages=false) default is true. This determines how the delivery_mode in basic.properties is set when a message is published. false=1, true=2. When set to true, messages will be persisted to disk by RabbitMQ and survive a server restart. Performance gains can be expected when set to false.
* **product** (e.g. product=My really important service) was introduced in EasyNetQ 0.27.3. default value is the name of the executable which instantiates the bus. The value entered here will show up in the administrative interface for RabbitMQ. 
* **platform** (e.g. platform=my.fully.qualified.domain.name) was introduced in EasyNetQ 0.27.3. default value is the hostname of the machine running the client process instantiating the bus. The value entered here will show up in the administrative interface for RabbitMQ. 
* **timeout** (e.g timeout=60) default is 10 seconds. Was introduced in EasyNetQ 0.17. Parsed to type System.UInt16. Range from 0 to 65535. Format is in seconds. For infinite timeout please use 0. Throws System.TimeoutException when value exceeded.

To close the connection, simply dispose the bus like this:

    bus.Dispose();

This will close the connection, channels, consumers and all other resources used by EasyNetQ.