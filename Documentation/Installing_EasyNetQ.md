Simply create a reference to the EasyNetQ.dll assembly. EasyNetQ has a dependency on the Newtonsoft.Json.dll and RabbitMQ.Client.dll so these should also be available.

To use FuturePublish you should also install EasyNetQ.Scheduler.exe. At a command prompt in the directory that contains the EasyNetQ.Scheduler.exe type:

    EasyNetQ.Scheduler.exe install
