To enable support for versioned messages you need to ensure the required components are configured. The simplest way to achieve this is:

```csharp
var bus = RabbitHutch.CreateBus( "host=localhost", services => services.EnableMessageVersioning() )
```
	
Once support for versioned messages is enabled, you must explicitly opt-in any messages you want to be treated as versioned.

```csharp
// This message is not versioned and will be treated in the same way as any other when it is published
public class MyMessage
{
	public string Text { get; set; }
}

// This message is versioned and will find it's way to both MyMessageV2 and MyMessage subscribers
public class MyMessageV2 : MyMessage, ISupersede<MyMessage>
{
	public int Number { get; set; }
}
```
	
## How Does It Work

When you publish a message, EasyNetQ usually creates an exchange for the message type and publishes the message to that exchange. Subscribers create queues that are bound to the exchange and therefore receive any messages published to it.

With message versioning enabled, EasyNetQ will create an exchange for each message type in the version hierarchy and bind those exchanges together. When you publish the MyMessageV2 message, it will be sent to the MyMessageV2 exchange which will automatically forward it to the MyMessage exchange.

When messages are serialized, EasyNetQ stores the message type name in the Type property of the message properties. This metadata is sent along with your message to any subscribers who can then use it to deserialize the message.

With message versioning enabled, EasyNetQ will also store all of the superseded message types in a header in the message properties. Subscribers will use this to find the first available type that the message can be deserialised into meaning that even if an endpoint does not have the latest version
of a message, so long as it has a version, it can be deserialized and handled.

## Message Versioning Guidance

1. If the change cannot be implemented by extending the original message type then it is not a new version of the message, it is a new message type.
1. If you are unsure, prefer to create a new message type rather than version an existing message.
1. Versioned messages should not be used with request / response as the message types are part of the request / response contract and `Request<V1,Response>` is not the same as `Request<V2,Response>` even if `V2` extends `V1` (i.e. `public class V2 : V1 {}`)
1. Versioned messages should not be used with send / receive as this is targeted sending and therefore there is a declared dependency between the sender and the receiver.


## Here Be Dragons

1. Versioned message support has been developed and tested in publish-subscribe scenarios. It has not be tested in either send-receive or request-response scenarios; use it outside of publish-subscribe at your own risk!
1. Versioned message support has not been extended to future publish scenarios at this time, additional work is planned to enable this, however due to the potential for breaking changes, some discussion with the project owner and community is required.