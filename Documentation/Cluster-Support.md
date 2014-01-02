EasyNetQ supports RabbitMQ clusters without any need to deploy a load balancer.

Simply list the nodes of the cluster in the connection string ...

    var bus = RabbitHutch.CreateBus("host=ubuntu:5672,ubuntu:5673");

In this example we have a cluster set up on a single machine, 'ubuntu', with node 1 on port 5672 and node 2 on port 5673. When the CreateBus statement executes, EasyNetQ will attempt to connect to the first host listed (ubuntu:5672). If it fails to connect it will attempt to connect to the second host listed (ubuntu:5673). If neither node is available it will sit in a re-try loop attempting to connect to both servers every five seconds. It logs all this activity to the registered IEasyNetQLogger. You might see something like this if the first node was unavailable:

    DEBUG: Trying to connect
    ERROR: Failed to connect to Broker: 'ubuntu', Port: 5672 VHost: '/'. ExceptionMessage: 'None of the specified endpoints were reachable'
    DEBUG: OnConnected event fired
    INFO: Connected to RabbitMQ. Broker: 'ubuntu', Port: 5674, VHost: '/'

If the node that EasyNetQ is connected to fails, EasyNetQ will attempt to connect to the next listed node. Once connected, it will re-declare all the exchanges and queues and re-start all the consumers. Here's an example log record showing one node failing then EasyNetQ connecting to the other node and recreating the subscribers:

    INFO: Disconnected from RabbitMQ Broker
    DEBUG: Trying to connect
    DEBUG: OnConnected event fired
    DEBUG: Re-creating subscribers
    INFO: Connected to RabbitMQ. Broker: 'ubuntu', Port: 5674, VHost: '/'

You get automatic fail-over out of the box.

### Random Hosts Selection
If you have multiple services using EasyNetQ to connect to a RabbitMQ cluster, they will all initially connect to the first listed node in their respective connection strings. If you are planning to use the load balancing feature, you should consider switching to the RandomClusterHostSelectionStrategy. Configure it like this:

    var bus = RabbitHutch.CreateBus("host=myfirsthost,mysecondhost", x => x.Register<IClusterHostSelectionStrategy<ConnectionFactoryInfo>, RandomClusterHostSelectionStrategy<ConnectionFactoryInfo>>());

In this snippet we have replaced the default IClusterHostSelectionStrategy for the RandomClusterHostSelectionStrategy, which will select hosts randomly. You can find more information about replacement EasyNetQ's components [here](https://github.com/mikehadlow/EasyNetQ/wiki/Replacing-EasyNetQ-Components).

### Consider using a dedicated load balancer
An alternative to the EasyNetQ cluster support is to use a dedicated front-side proxy. See the end of the RabbitMQ clustering guide for details: http://www.rabbitmq.com/clustering.html

