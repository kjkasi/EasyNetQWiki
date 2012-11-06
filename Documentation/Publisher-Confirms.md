The default AMQP publish is not transactional and doesn't guarantee that your message will actually reach the broker. AMQP does specify a transactional publish, but with RabbitMQ it is extremely slow and we haven't supported it via the EasyNetQ API. For high-performance guaranteed delivery it's recommended that you use 'Publisher Confirms'. Simply speaking, this an extension to AMQP that provides a callback when your message has been successfully received by the broker.

What does 'successfully received mean? It depends ...

* A transient message is confirmed the moment it is enqueued.
* A persistent message is confirmed as soon as it is persisted to disk, or when it is consumed on every queue.
* An unroutable transient message is confirmed directly it is published.

For more information on publisher confirms, [please read the announcement on the RabbitMQ blog](http://www.rabbitmq.com/blog/2011/02/10/introducing-publisher-confirms/)

To use publisher confirms, you must first create the publish channel with publisher confirms on:

     var channel = bus.OpenPublishChannel(x => x.WithPublisherConfirms())

Next you must specify success and failure callbacks when you publish your message:

    channel.Publish(message, x => 
        x.OnSuccess(() =>
        {
            // do success processing here
        })
        .OnFailure(() => 
        {
            // do failure processing here
        }));

Be careful not to dispose the publish channel before your callbacks have had a chance to execute.
