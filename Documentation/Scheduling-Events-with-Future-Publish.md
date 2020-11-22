Many business processes require that events be scheduled for some future date. For example, after an initial sales contact with a customer, we may want to schedule a follow up call at some time in the future. EasyNetQ can help you implement this functionality with its Future Publish feature. For example, here we are using the FuturePublish extension method to schedule a follow-up sales call for a week in the future.
```c#
    var followUpCallMessage = new FollowUpCallMessage( .. );
    bus.Scheduler.FuturePublish(TimeSpan.FromDays(7), followUpCallMessage);
```
Three months from now the message will be published by EasyNetQ and any subscribers of FollowUpCallMessage will receive a copy of the original message.


## How Does It Work?

Default `IScheduler` implementation uses [per queue TTL](https://www.rabbitmq.com/ttl.html#queue-ttl) + [DLX exchange](https://www.rabbitmq.com/dlx.htmlhttps://www.rabbitmq.com/dlx.html).


