> The RabbitMQ Delayed Message Plugin is still in experimental phase.<br/>
> You use it at your own risk.


The [RabbitMQ Delayed Message Plugin](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/) adds new exchange type to RabbitMQ which allows to delay message delivery.

EasyNetQ provides support for using that exchange by defining new scheduler type: `DelayedExchangeScheduler`. 

This allows you to use the same Future Publish interface as before, with the exception of cancelling future messages. Because Delayed Message Plugin doesn't support messages cancellation, whenever you call `FuturePublish` specifying `cancellationKey`, or when you call `CancelFuturePublish` the scheduler will throw `NotImplementedException`.

Below example shows how to publish a message which will get delivered in three months from now:
```
bus = RabbitHutch.CreateBus("host=localhost", x => x.Register<IScheduler, DelayedExchangeScheduler>());

var followUpCallMessage = new FollowUpCallMessage( .. );

bus.FuturePublish(DateTime.UtcNow.AddMonths(3), followUpCallMessage);
```

First line instructs EasyNetQ to use new scheduler which supports Delayed Message Exchanges. Next the message is created and published with delivery time set to three months. Note that FuturePublish uses UTC time.

DelayedExchangeScheduler requires **Delayed Message Plugin** to be installed.

# Plugin installation

Delayed Message Plugin can be found on [Community Plugins page](http://www.rabbitmq.com/community-plugins.html). Download the corresponding .ez file for your RabbitMQ installation, copy it into RabbitMQ's plugin folder and then enable it by running the following command:
```
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```
The plugin requires RabbitMQ version 3.4 or newer.

# How it works

When you call bus.FuturePublish(...), EasyNetQ automatically creates new `x-delayed-message` exchange along with normal exchange and binds them together. The message is published on the delayed exchange which will store the message until the time comes to deliver it. At that point, the message is routed to normal exchange and from there to bound queue(s).

When you call Publish(...) method, the messages are published to normal exchange which prevents any performance degradation related to using new `x-delayed-message` exchange.

The delayed exchange persists messages using Mnesia. That prevents messages loss in case of server downtime. Once the server restores, all eligible messages will be delivered on schedule.

# Cancelling Future messages

As mentioned earlier, the Delayed Message Plugin doesn't support messages cancellation, hence this functionality is not supported. Any call to `FuturePublish` specifying `cancellationKey`, or to `CancelFuturePublish` will throw `NotImplementedException`.

If you require that functionality, please use Scheduler Service as described in [Scheduling Events with Future Publish](https://github.com/EasyNetQ/EasyNetQ/wiki/Scheduling-Events-with-Future-Publish).