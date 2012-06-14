Many business processes require that events be scheduled for some future date. For example, after an initial sales contact with a customer, we may want to schedule a follow up call at some time in the future. EasyNetQ can help you implement this functionality with its Future Publish feature. For example, here we are using the FuturePublish extension method to schedule a follow-up sales call for a month in the future. Note that FuturePublish uses UTC time.

    var followUpCallMessage = new FollowUpCallMessage( .. );

    using (var publishChannel = bus.OpenPublishChannel())
    {
        publishChannel.FuturePublish(DateTime.UtcNow.AddMonths(3), followUpCallMessage);
    }

One month from now the message will be published by EasyNetQ and any subscribers of FollowUpCallMessage will receive a copy of the original message.

FuturePublish requires that the **EasyNetQ.Scheduler** service is running.

## How Does It Work?

When you call bus.FuturePublish(publishDate, message), EasyNetQ wraps your message in a system message ‘ScheduleMe’ and publishes it to RabbitMQ. The scheduler service subscribes to this message. When it receives a ScheduleMe message, it stores it in its local database. The scheduler service polls its database for messages where the schedule date has become due, when it finds any due messages, it unwraps the original message from the ScheduleMe message and publishes it to the bus.
