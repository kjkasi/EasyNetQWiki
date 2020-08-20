[[images/logo_design_150.png|EasyNetQ logo]]

EasyNetQ is a simple to use, opinionated, .NET API for [[RabbitMQ|http://www.rabbitmq.com/]].

If you just want to get up and running as quickly as possible, go to the **[[Quick Start]]** guide.

The goal of EasyNetQ is to provide a library that makes working with RabbitMQ in .NET as simple as possible. In order to do this, it has to take an opinionated view of how you should use RabbitMQ with .NET. It trades flexibility for simplicity by enforcing some simple conventions. These include:

* Messages should be represented by .NET types. 
* Messages should be routed by their .NET type.

This means that messages are defined by .NET classes. Each distinct message type that you want to send is represented by a class. The class should be public with a default constructor and public read/write properties. You would not normally implement any functionality in a message, but treat it as a simple data container or Data Transfer Object (DTO). Here is a simple message:

    public class MyMessage
    {
        public string Text { get; set; }
    }

EasyNetQ routes messages by their type. When you publish a message, EasyNetQ examines its type and gives it a routing key based on the type name, namespace and assembly. On the consuming side, subscribers subscribe to a type. After subscribing to a type, messages of that type get routed to the subscriber.

By default, EasyNetQ serializes .NET types as JSON using the Newtonsoft.Json library. This has the advantage that messages are human readable, so you can use tools such as the RabbitMQ management application to debug message problems.

## API Design

![API Design](images/EasyNetQ_API.png)

EasyNetQ is a collection of components that provide services on top of the RabbitMQ.Client library. These do things like serialization, error handling, thread marshalling, connection management, etc. They are composed by a mini-IoC container. You can replace any component with your own implementation quite easily. So if you’d like XML serialization instead of the built in JSON, just write an implementation of ISerializer and register it with the container.

These components are fronted by the IAdvancedBus API. This looks a lot like the AMQP specification, and indeed you can run most AMQP methods from this API. The only AMQP concept that this API hides from you is channels. This is because channels are a confusing low-level concept that should never have been part of the AMQP specification in the first place. ‘Advanced’ is not a very good name for this API to be honest, ‘Iamqp’ would be much better.

Layered on top of the advanced API are a set of messaging patterns: Publish/Subscribe, Request/Response, and Send/Receive. This is the ‘opinionated’ part of EasyNetQ. It is our take on how such patterns should be implemented. There is very little flexibility; either you accept our way of doing things, or you don’t use it. The intention is that you, the user, don’t have to expend mental bandwidth re-inventing the same patterns; you don’t have to make choices every time you simply want to publish a message and subscribe to it. It’s designed to achieve EasyNetQ’s core goal of making working with RabbitMQ as easy as possible.

The patterns sit behind the IBus API. Once again, this is a poor name, it’s got very little to do with the concept of a message bus. A better name would be IPackagedMessagePatterns.

IBus is intended to work for 80% of users, 80% of the time. It’s not exhaustive. If the pattern you want to implement is not provided by IBus, then you should use IAdvancedBus. There’s no problem with doing this, and it’s how EasyNetQ is designed to be used.

## Why do I need EasyNetQ?

Doesn’t RabbitMQ already have a .NET client? 

Yes it does. You can download the .NET AMQP client library [[here|http://www.rabbitmq.com/dotnet.html]].

So why do I need EasyNetQ? The RabbitMQ .NET client implements the client side of the AMQP protocol (and RabbitMQ implements the server side). AMQP is intended as the HTTP of messaging. It is designed to be cross platform and language agnostic. It is also designed to flexibly support a wide range of messaging patterns based on the Exchange/Binding/Queue model. 

It’s great having this flexibility, but with flexibility comes complexity. It means that you will need to write a significant amount of code in order to implement a RabbitMQ client. Typically this code would include:

* Implementing messaging patterns such as Publish/Subscribe or Request/Response. Although, to be fair, the .NET client does provide some support here. 
* Implement a routing strategy. How will you design your exchange-queue bindings, and how will you route messages between producers and consumers? 
* Implement message serialization/deserialization. How will you convert the binary representation of messages in AMQP to something your programming language understands? 
* Implement a consumer thread for subscriptions. You will need to have a dedicated consumer loop waiting for messages you have subscribed to. How will you deal with multiple subscribers, or transient subscribers, like those waiting for responses from a request? 
* Implement subscriber reconnection. If the connection is disrupted or the RabbitMQ server bounces, how do you detect it and make sure all your subscriptions are rebuilt? 
* Understand and implement quality of service settings. What settings do you need to make to ensure that you have a reliable client. 
* Implement an error handling strategy. What should your client do if it receives a malformed message, or if an unexpected exception is thrown? 
* Implement publisher confirms for reliable messaging.

EasyNetQ aims to encapsulate all these concerns in a simple to use library that sits on top of the existing AMQP client. It is the culmination of several year's experience of RabbitMQ use in a high volume commercial environment.

## Performance Expectations

The performance of EasyNetQ is directly related to the performance of the RabbitMQ broker. This can vary with network and server performance. In tests on a developer machine with a local instance of RabbitMQ, sustained over-night performance of around 5000 2K messages per second was achieved. Memory use for all the EasyNetQ endpoints was stable for the overnight run.