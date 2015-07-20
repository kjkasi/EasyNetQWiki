The default behaviour of EasyNetQ, when generating names for queues, is to use message type name and append it with subscription Id. For example the `PartyInvitation` message type from namespace `EasyNetQ.Tests.Integration` will use queue name **EasyNetQ.Tests.Integration.PartyInvitation:EasyNetQ.Tests_schedulingTest1**, assuming that subscription Id is `schedulingTest1`.

## Controlling queue name

To control the name of the queue, annotate the message class with `Queue` attribute:

	[Queue("TestMessagesQueue", ExchangeName = "MyTestExchange")]
    public class TestMessage
    {
	   public string Text { get; set; }
    }
	
	// ...
	
	bus.Subscribe<TestMessage>(string.Empty, msg => Console.WriteLine(msg.Text));

Here we tell EasyNetQ to use TestMessagesQueue as a queue name, and MyTestExchange as an exchange name. Notice that subscriptionId passed to Subscribe method is empty. If you specify subscriptionId then it will be appended to the end and used as queue name.

## Working with messages not published by EasyNetQ

Using QueueAttribute allows to consume messages from any queue. This can be used to consume messages which are published by frameworks other than EasyNetQ as long as one condition is met - queue message has property `type` set. The value of `type` property is used during message de-serialization to determine type of the message. As long as this property is set to something meaningful, the messages may be consumed. Decoding type name is done in `ITypeNameSerializer.Deserialize` method.

If you do decide to implement your own custom `ITypeNameSerializer` then be careful of how you implement the `Deserialize` method. If your implementation is processor intensive then you are in danger of constraining the speed at which you can dequeue messages. e.g. Assembly scanning is a bad idea without some kind of type caching.

## Considerations for naming queues

Setting queue name to empty string will use default naming behaviour. The maximum length of queue name is 255 characters (this is enforced by RabbitMQ client library). The name can be a sequence of letters, digits, hyphen, underscore, period, or colon. 
Queue names starting with "amq." are reserved for [pre-declared and standardised queues]("https://www.rabbitmq.com/amqp-0-9-1-reference.html#queue.declare.queue").
