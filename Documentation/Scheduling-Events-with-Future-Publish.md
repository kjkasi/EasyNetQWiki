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

## Installing the Scheduler Service

1. In SQL Server, create a new database EasyNetQ.Scheduler

2. Get the source code for EasyNetQ

    git clone git@github.com:mikehadlow/EasyNetQ.git

3. Open the EasyNetQ.2012 solution in Visual Studio. In the folder DatabaseScripts -> EasyNetQ.Scheduler you will find a number of SQL scripts. Open and run them in the EasyNetQ.Scheduler database. You will need to run CreateWorkTables.sql first, the others are stored procedure scripts and can be run in any order.

4. Build the solution.

5. Find the <clone path>\Source\EasyNetQ.Scheduler\bin\Debug and copy the contents to a deployment folder of your choice.

6. Open EasyNetQ.Scheduler.exe.config in a text editor and change the 'rabbit' and 'scheduleDb' connection strings to point to your RabbitMQ broker and SQL Server instance respectively.

7. Open a console window and change the path to the folder where you deployed EasyNetQ.Scheduler.

8. Run the following command to install EasyNetQ.Scheduler as a windows service:

    EasyNetQ.Scheduler.exe install

    Configuration Result:
    [Success] Name EasyNetQ.Scheduler
    [Success] ServiceName EasyNetQ.Scheduler
    Topshelf v3.1.106.0, .NET Framework v4.0.30319.18051

    Running a transacted installation.

    Beginning the Install phase of the installation.
    Installing EasyNetQ.Scheduler service
    Installing service EasyNetQ.Scheduler...
    Service EasyNetQ.Scheduler has been successfully installed.
    Creating EventLog source EasyNetQ.Scheduler in log Application...

    The Install phase completed successfully, and the Commit phase is beginning.

    The Commit phase completed successfully.

    The transacted install has completed.

You should now be able to call FuturePublish and see the messages appear at the specified time.

To uninstall EasyNetQ.Scheduler run:

    EasyNetQ.Scheduler.exe uninstall

