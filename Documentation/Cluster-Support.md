EasyNetQ supports RabbitMQ clusters without any need to deploy a load balancer.

Simply list the nodes of the cluster in the connection string ...

    var bus = RabbitHutch.CreateBus("host=ubuntu:5672,ubuntu:5673");

In this example we have a cluster set up on a single machine, 'ubuntu', with node 1 on port 5672 and node 2 on port 5673. When the CreateBus statement executes, EasyNetQ will attempt to connect to the first host listed (ubuntu:5672). If it fails to connect it will attempt to connect to the second host listed (ubuntu:5673). If neither node is available it will sit in a re-try loop attempting to connect to both servers every five seconds. It logs all this activity to the registered IEasyNetQLogger. You might see something like this if the first node was unavailable:

    DEBUG: Trying to connect
    ERROR: Failed to connect to Broker: 'ubuntu', Port: 5672 VHost: '/'. ExceptionMessage: 'None of the specified endpoints were reachable'
    DEBUG: OnConnected event fired
    INFO: Connected to RabbitMQ. Broker: 'ubuntu', Port: 5674, VHost: '/'

If the node that EasyNetQ is connected to fails, EasyNetQ will attempt to connect to the next listed node. Once connected, it will re-declare all the exchanges and queues and re-start all the consumers. You get automatic fail-over out of the box.

If you have multiple services using EasyNetQ to connect to a RabbitMQ cluster, they will all initially connect to the first listed node in their respective connection strings. For this reason the EasyNetQ cluster support is not really suitable for load balancing high throughput systems. We recommend that you use a dedicated hardware or software load balancer instead.