Architecting a system around a message bus involves writing many small focussed components that sit on the bus waiting for messages they care about to arrive. These are best implemented as Windows services. A great way of implementing Windows services is to use [TopShelf](http://topshelf-project.com/). This very nice open source library grew out of the excellent MassTransit project. It makes writing windows services super easy; you simply create a console project, 'install-package Topshelf', and use the neat fluent API to describe your service. An IoC container should be at the heart of any but the simplest .NET application. If you've not used an IoC container [read this for why you should](http://mikehadlow.blogspot.co.uk/2007/08/castle-projects-windsor-container-and.html). There are quite a few IoC containers to choose from, Here we use [Castle Windsor](http://docs.castleproject.org/Windsor.MainPage.ashx).

This section shows how to wire up TopShelf, Windsor and EasyNetQ. 

The Zen of IoC says that [any application that uses an IoC container should reference it in only two places](http://mikehadlow.blogspot.co.uk/2010/11/first-commandment-thou-shalt-not.html). First to create an instance of the container that lasts the lifetime of the application. Second to resolve a root service from the container. All the other dependencies are supplied invisibly by the container itself. This magic is what makes IoC containers such awesome and essential frameworks. Following this rule, in the Main() function we create the IoC container and resolve the root service of our application, in this case IVaultService, all within the fluent API provided by TopShelf.

    public class Program
    {
        public static void Main(string[] args)
        {
            var container = new WindsorContainer().Install(FromAssembly.This());

            HostFactory.Run(x =>
            {
                x.Service<IVaultService>(s =>
                {
                    s.ConstructUsing(name => container.Resolve<IVaultService>());
                    s.WhenStarted(tc => tc.Start());
                    s.WhenStopped(tc =>
                    {
                        tc.Stop();
                        container.Release(tc);
                        container.Dispose();
                    });
                });

                x.RunAsLocalSystem();

                x.SetDescription("Vault service.");
                x.SetDisplayName("Vault.Service");
                x.SetServiceName("Vault.Service");
            });
        }
    }

[A cardinal rule of Windsor is that you must release any components that you resolve](http://nexussharp.wordpress.com/2012/04/21/castle-windsor-avoid-memory-leaks-by-learning-the-underlying-mechanics/). Windsor tracks any components that implement IDisposable and ensures that Dispose is called no matter where the component gets resolved in the dependency graph, but you need to call Release for this to happen correctly. The 'tc' variable is the instance of IVaultService that gets resolved in the 'ConstructUsing' call, so we can use it in the Release call.

What about EasyNetQ? The Zen of EasyNetQ says that [you should create a single instance of IBus that lasts the lifetime of the application](Connecting-to-RabbitMQ). Now we could have created our IBus instance in Main() alongside the TopShelf setup and newing up the container, but since we've got a container we want it to manage the lifetimes of all the components used in the application. First let's create a simple factory method that gets the connection string for EasyNetQ and creates a new instance of IBus:

    public class BusBuilder
    {
        public static IBus CreateMessageBus()
        {
            var connectionString = ConfigurationManager.ConnectionStrings["easynetq"];
            if (connectionString == null || connectionString.ConnectionString == string.Empty)
            {
                throw new VaultServiceException("easynetq connection string is missing or empty");
            }

            return RabbitHutch.CreateBus(connectionString.ConnectionString);
        }
    }

Now we can write our [IWindsorInstaller](http://docs.castleproject.org/Windsor.Installers.ashx) to register our services:

    public class Installer : IWindsorInstaller
    {
        public void Install(IWindsorContainer container, IConfigurationStore store)
        {
            container.Register(
                Component.For<IVaultService>().ImplementedBy<VaultService>().LifestyleTransient(),
                Component.For<IBus>().UsingFactoryMethod(BusBuilder.CreateMessageBus).LifestyleSingleton()
                );
        }
    }

Note that we tell Windsor to create our IBus instance using our factory with 'UsingFactoryMethod' rather than 'Instance'. The Instance method tells Windsor that we will take responsibility for the lifetime of the service, but we want Windsor to call Dispose when the application shuts down, UsingFactoryMethod tells Windsor that it needs to manage the IBus lifestyle. We declare it as LifestyleSingleton because we only want a single instance of IBus for the entire lifetime of the application.

Now we can reference IBus in our IVaultService implementation:

    public interface IVaultService
    {
        void Start();
        void Stop();
    }

    public class VaultService : IVaultService
    {
        private readonly IBus bus;

        public VaultService(IBus bus)
        {
            this.bus = bus;
        }

        public void Start()
        {
            bus.SubscribeAsync<MyMessage>("vault_handler", msg => {  });

        }

        public void Stop()
        {
            // shutdown code
        }
    }

Here we are simply subscribing to MyMessage in the Start method of VaultService. I would probably also have an IMyMessageHandler service referenced by IVaultService to do the message handling itself.

So there you have it, a simple recipe for using these three excellent OSS projects together.

## Example application

EasyNetQ example showing Request Response and Autosubcriber, wired up using Windsor IOC

[https://bitbucket.org/philipogorman/createrequestservice/src](https://bitbucket.org/philipogorman/createrequestservice/src)
