Sagas are a way of implementing long running business processes that sit on the message bus and respond to and orchestrate various business events. The core architectural intention behind Sagas is that they help us to separate business logic from infrastructure. The framework for hosting and maintaining the Saga(s) is all contained within the SagaHost windows service.

Sagas are small assemblies of business process specific code that are hosted in EasyNetQ’s SagaHost service. EasyNetQ.SagaHost.exe uses the Managed Extensibility Framework to load and run user Sagas.

Writing a Saga is easy. Simply create a new C# library project with a reference to the EasyNetQ assembly and the System.ComponentModel.Composition assembly (This is the Managed Extensibility Framework assembly).

The Saga itself is a class that implements and exports (With the System.ComponentModel.Composition.ExportAttribute) the EasyNetQ.ISaga interface. The ISaga interface has a single method, Initialize, which is where you create your subscriptions for your Saga. A Saga always starts with a subscription. Inside the subscription callback, you might want to make a request / response call, or publish further messages. Here’s an example:

    using System; 
    using System.ComponentModel.Composition;  
    namespace EasyNetQ.Tests.SimpleSaga 
    {     
        [Export(typeof(ISaga))]     
        public class TestSaga : ISaga     
        {         
            public void Initialize(IBus bus)         
            {             
                bus.Subscribe<StartMessage>("simpleSaga", startMessage =>             
                {                 
                    Console.WriteLine("StartMessage: {0}", startMessage.Text);                 
                    var firstProcessedMessage = startMessage.Text + " - initial process ";                 
                    var request = new TestRequestMessage { Text = firstProcessedMessage };   
                    
                    using (var publishChannel = bus.OpenPublishChannel())
                    {
                        publishChannel.Request<TestRequestMessage, TestResponseMessage>(request, response =>                 
                        {                     
                            Console.WriteLine("TestResponseMessage: {0}", response.Text);                     
                            var secondProcessedMessage = response.Text + " - final process ";                     
                            var endMessage = new EndMessage { Text = secondProcessedMessage };                     
                            using (var publishChannel2 = bus.OpenPublishChannel())
                            {
                                publishChannel2.Publish(endMessage);                 
                            }
                        });             
                    }
                });         
            }     
        } 
    }

This Saga starts when a StartMessage is received; it then creates a TestRequestMessage and makes a request expecting a TestResponseMessage. When the response is received, it creates an EndMessage and publishes it.

## Installing SagaHost

To install SagaHost, open a command prompt in the directory where you’ve placed the EasyNetQ.SagaHost.exe and type:

    EasyNetQ.SagaHost.exe install

## Deploying Sagas

To deploy your Saga(s) simply copy the Assembly that contains them into the ‘Sagas’ directory below the location of EasyNetQ.SagaHost.exe and restart the service. You should see the Sagas loaded in the log file:

    DEBUG - MEF DirectoryCatalog is probing 
    'C:\SOURCE\MIKE.AMQPSPIKE\EASYNETQ.SAGAHOST\BIN\DEBUG\SAGAS' 
    DEBUG - Listing *.dll files in Saga directory .... 
    DEBUG - C:\SOURCE\MIKE.AMQPSPIKE\EASYNETQ.SAGAHOST\BIN\DEBUG\SAGAS\EasyNetQ.Tests.Messages.dll 
    DEBUG - C:\SOURCE\MIKE.AMQPSPIKE\EASYNETQ.SAGAHOST\BIN\DEBUG\SAGAS\EasyNetQ.Tests.SimpleSaga.dll 
    DEBUG - 
    DEBUG - Loading Sagas ... 
    DEBUG - Loading Saga: EasyNetQ.Tests.SimpleSaga.TestSaga 
    DEBUG - Loaded 1 Sagas 
    DEBUG - MEF container initialized
