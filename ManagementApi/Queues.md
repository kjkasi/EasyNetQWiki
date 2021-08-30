## General

**GetQueues()** Will get a list of queues declared on the broker. For example:

    var queues = managementClient.GetQueues();

    foreach (Queue queue in queues)
    {
        Console.Out.WriteLine("queue.Name = {0}", queue.Name);
    }

Will output:

    queue.name = EasyNetQ_Default_Error_Queue
    queue.name = EasyNetQ_Tests_MyMessage:EasyNetQ_Tests_test
    queue.name = EasyNetQ_Tests_TestPerformanceMessage:EasyNetQ_Tests_Messages_consumer
    queue.name = management_api_test_queue

***

**CreateQueue(QueueInfo, Vhost)** will create (declare) a new queue:

    var vhost = managementClient.GetVhost("/");
    var queueInfo = new QueueInfo(testQueue);
    var queue = managementClient.CreateQueue(queueInfo, vhost);

***

**DeleteQueue(Queue)** will delete a queue:

    managementClient.DeleteQueue(queue);

***

**GetBindingsForQueue(Queue)** will return all the bindings that have the given queue as their destination:

    var bindings = managementClient.GetBindingsForQueue(queue);

***

**Purge(Queue)** will purge all the messages from a queue:

    managementClient.Purge(queue);

***

**GetMessagesFromQueue(Queue, GetMessageCriteria)** will return messages from a queue. 

The **GetMessagesCriteria** constructor takes two arguments: 

* **count** controls the number of messages to get. You may get fewer messages than this if the queue cannot immediately provide them.

* **requeue** determines whether the messages will be removed from the queue. If requeue is true they will be requeued - but their position in the queue may change and their redelivered flag will be set.

For example:

    var criteria = new GetMessagesCriteria(1, true);
    var messages = managementClient.GetMessagesFromQueue(queue, criteria);

    foreach (var message in messages)
    {
        Console.Out.WriteLine("message.payload = {0}", message.Payload);
    }

## Quorum queues

To use quorum queues just add the parameter QueueType to the message class.

    [Queue("QuorumQueue", QueueType = QueueType.Quorum)]
    public class QuorumQueueMessage
    {

    }