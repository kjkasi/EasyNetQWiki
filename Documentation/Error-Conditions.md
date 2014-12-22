In this section we take a look at various error conditions that can occur in any messaging system and look at how EasyNetQ handles them.

## My Subscription Service Dies

You’ve written a windows service that subscribes to my NewCustomerMessage. What happens if the service fails? For efficiency, EasyNetQ implements an internal in-memory queue for subscriptions. Messages are received over the network from RabbitMQ and placed on this queue. A single subscription thread takes the messages off the queue in turn and passes them to the callback(s) you have provided. Once the callback completes EasyNetQ sends an ‘Ack’ back to RabbitMQ. Until the ‘Ack’ is received the message is not removed from the RabbitMQ queue. If your service dies while processing a message, the message (and all the messages in the EasyNetQ in-memory queue) will stay on the RabbitMQ queue. Once your service reconnects the messages will be resent. 

## My Subscriber Consumes Messages Slower Than They Are Published

EasyNetQ uses RabbitMQ’s quality of service settings to set the prefectch-count to some reasonable setting (currently 50). This means that there will never be more than 50 messages in the subscriber’s in-memory queue. This prevents out-of-memory exceptions occurring in your subscribing application. Once the count of un-acked messages gets to this level, RabbitMQ will stop sending them and simply leave them on its internal queue. Of course, eventually this queue will eat all your disk space on your RabbitMQ server machine. You should have some monitoring in place to make sure you are alerted before this happens.

## There Is a Network Failure Between My Subscriber And The RabbitMQ Broker

As described in the section ‘Connecting to RabbitMQ’, EasyNetQ implements a lazy connection strategy. It assumes that the broker will not always be available. When you first connect to a broker using RabbitHutch.CreateBus, EasyNetQ enters a connection try loop and if no broker is available at the address you specify in the connection string, you will see information messages logged saying ‘Trying to Connect’. Subscribers can subscribe using bus.Subscribe even when a broker is not available. The subscription details are cached by EasyNetQ. When a broker becomes available the connection loop succeeds, a connection with the broker is established, and all the cached subscriptions are created. 

Similarly when EasyNetQ loses a connection to the broker, it returns to the connection-loop and you will see ‘Trying to Connect’ messages in the log. Once the connection is re-established the cached subscribers are once again created. The upshot of this is that you can leave your subscribers running in an environment where the network connection is unreliable or where you need to bounce your RabbitMQ broker.

## My Subscription Callback Throws an Exception While Consuming a Message

If your subscription callback throws an exception, EasyNetQ will take the message that was being consumed and wrap it in a special error message. The error message will be published to the EasyNetQ error queue (named EasyNetQ_Default_Error_Queue). You should monitor the error queue for any messages. The error message includes all the information necessary to re-publish the original message as well as the exception’s type, message and stack trace. You can use the EasyNetQ.Hosepipe utility to re-publish error messages. See the section on EasyNetQ.Hosepipe below.
