EasyNetQ's mission is to provide the simplest possible API for RabbitMQ based messaging. The core IBus interface purposefully avoids exposing AMQP concepts such as exchanges, bindings, and queues, instead it implements a default exchange-binding-queue topology based on message type.

For some scenarios it's useful to be able to configure your own exchange-binding-queue topology; the advanced EasyNetQ API allows you to do that.

The advanced API is implemented with the IAdvancedBus interface. An instance of this interface can be accessed via the Advanced property of IBus:

    var advancedBus = RabbitHutch.CreateBus("host=localhost").Advanced;

