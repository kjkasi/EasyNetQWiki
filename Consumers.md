**GetConsumersAsync()** will return a list of active consumers on currently open connections to the broker. For example:

    var consumers = await managementClient.GetConsumersAsync();

    foreach (var consumer in consumers)
    {
       Console.Out.WriteLine("ConsumerTag = {0}", consumer.ConsumerTag);
    }

Outputs the tags assigned by the broker to each consumer. For example, assuming there are 4 active consumers, the output will be similar to:

    e7680fcc-b493-49dd-be3c-65d98b7039d3
    e8317d79-1b4b-4b46-9e1f-bc338571365a
    7973b9d2-5241-4ceb-ba24-de5e68e341a2
    0144b5c8-a11e-49ec-ac54-7c582699f050

Each consumer, with its properties, is deserialized from the following json data:

```
  {
    "arguments": {
      "x-cancel-on-ha-failover": false,
      "x-priority": 0
    },
    "ack_required": true,
    "active": true,
    "activity_status": "up",
    "channel_details": {
      "connection_name": "[::1]:65216 -> [::1]:5672",
      "name": "[::1]:65216 -> [::1]:5672 (4)",
      "node": "rabbit@localhost",
      "number": 4,
      "peer_host": "::1",
      "peer_port": 65216,
      "user": "guest"
    },
    "consumer_tag": "60479aa8-c8b8-43d1-baee-e45731b53024",
    "exclusive": false,
    "prefetch_count": 50,
    "queue": {
      "name": "semple_queue",
      "vhost": "/"
    }
  }
```