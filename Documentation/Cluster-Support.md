EasyNetQ supports RabbitMQ clusters without the need to deploy a load balancer and relies on an implementation of RabbitMQ.Client.

Simply list the nodes of the cluster in the connection string ...
```с#
    var bus = RabbitHutch.CreateBus("host=ubuntu:5672,ubuntu:5673");
```
In this example we have a cluster set up on a single machine, `ubuntu`, with node 1 on port 5672 and node 2 on port 5673. After the `CreateBus` and a first request(for instance, `Publish`) are executed, EasyNetQ will attempt to connect to the random host listed (for example, ubuntu:5672). If it fails to connect it will attempt to connect to the unused random host listed (`ubuntu:5673`). If neither node is available it will sit in a re-try loop attempting to connect to both servers every five seconds by default.

If the node that EasyNetQ is connected to fails, EasyNetQ will attempt to connect to the next listed node. Once connected, it will re-declare all the exchanges and queues and re-start all the consumers.

### Mirroring
If a consumer is consuming a queue which is not mirrored (i.e. it exists only on one of the cluster nodes) then it is possible to arrive at a situation where subscribers are not automatically recreated if a broker node goes down & comes back up. For this reason, it is recommended to use [mirrored queues](https://www.rabbitmq.com/ha.html#what-is-mirroring) when running a RabbitMQ cluster. 

For more detail on a specific failure scenario, see https://github.com/EasyNetQ/EasyNetQ/issues/1135

### Consider using a dedicated load balancer
An alternative to the EasyNetQ cluster support is to use a dedicated front-side proxy. See the end of the RabbitMQ clustering guide for details: http://www.rabbitmq.com/clustering.html
