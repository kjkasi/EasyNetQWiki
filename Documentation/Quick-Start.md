Welcome to EasyNetQ. This guide shows you how to get up and running with EasyNetQ in about 10 minutes.

EasyNetQ is a simple to use client API for RabbitMQ. So first install RabbitMQ:

1. Install Erlang: http://www.erlang.org/download.html
2. Install the RabbitMQ server: http://www.rabbitmq.com/download.html
3. You'll probably want to enable the RabbitMQ management API too: http://www.rabbitmq.com/management.html

Now you should be able to navigate to the RabbitMQ management URL:

    http://localhost:15672/

Next open Visual Studio and create a new solution named EasyNetQTest with three C# projects:

    Messages   (Class Library)
    Publisher  (Console Application)
    Subscriber (Console Application)

Open the NuGet Package Manager Console and install EasyNetQ in both the Publisher and Subscriber projects by typing:

    Install-Package EasyNetQ -ProjectName Publisher
    Install-Package EasyNetQ -ProjectName Subscriber

In the Messages project create a new class TextMessage.cs:

    namespace Messages
    {
        public class TextMessage
        {
            public string Text { get; set; } 
        }
    }

Add a reference to the Messages project to both the Publisher and Subscriber projects.

Open the Program.cs class in the Publisher project and type in the following code:

    using System;
    using EasyNetQ;
    using Messages;
    
    namespace Publisher
    {
        class Program
        {
            static void Main(string[] args)
            {
                using (var bus = RabbitHutch.CreateBus("host=localhost"))
                {
                    var input = "";
                    Console.WriteLine("Enter a message. 'Quit' to quit.");
                    while ((input = Console.ReadLine()) != "Quit")
                    {
                        bus.Publish(new TextMessage
                            {
                                Text = input
                            });
                    }
                }
            }
        }
    }

Open the other Program.cs class in the Subscriber project and type this code:

    using System;
    using EasyNetQ;
    using Messages;
    
    namespace Subscriber
    {
        class Program
        {
            static void Main(string[] args)
            {
                using (var bus = RabbitHutch.CreateBus("host=localhost"))
                {
                    bus.Subscribe<TextMessage>("test", HandleTextMessage);
    
                    Console.WriteLine("Listening for messages. Hit <return> to quit.");
                    Console.ReadLine();
                }
            }
    
            static void HandleTextMessage(TextMessage textMessage)
            {
                Console.WriteLine("Got message: {0}", textMessage);
            }
        }
    }

Now right click on the Subscriber project and select 'Set As StartUp project', then hit ctrl-F5 (start without debugging) to launch the Subscriber console application. Repeat the same steps with the Publish project.

You should now have two console applications running with a lot of debugging information showing that EasyNetQ has successfully connected to your RabbitMQ server. Now type some messages into the publisher console application. You should see the Subscriber application report that it has received them.