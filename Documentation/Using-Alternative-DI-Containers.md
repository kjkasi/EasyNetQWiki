EasyNetQ is made up of a collection of independent components. Internally it uses LightInject container internally, which is wrapped by DefaultServiceProvider. If you look at the code for the static RabbitHutch class that you use to create instances of the core IBus interface, you will see that it simply creates a new DefaultServiceProvider, registers all of EasyNetQ’s components, and then calls container.Resolve<IBus>() creating a new instance of IBus with its tree of dependencies supplied by the container:

  public static IBus CreateBus(Func<IServiceResolver, ConnectionConfiguration> connectionConfigurationFactory, Action<IServiceRegister> registerServices)
        {
            var container = new DefaultServiceContainer();
            RegisterBus(container, connectionConfigurationFactory, registerServices);
            return container.Resolve<IBus>();
        }

But what if you want EasyNetQ to use your container of choice? From version 3 the RabbitHutch class provides a static method, RegisterBus, that allows you to register EasyNetQ components in `IServiceRegister`,  which you can implement for any container you want. There are several ready adapters for [Castle Windsor](https://www.nuget.org/packages/EasyNetQ.DI.Windsor), [Autofac](https://www.nuget.org/packages/EasyNetQ.DI.Autofac), [LightInject](https://www.nuget.org/packages/EasyNetQ.DI.LightInject), [NInject](https://www.nuget.org/packages/EasyNetQ.DI.NInject), [StructureMap](https://www.nuget.org/packages/EasyNetQ.DI.StructureMap), [SimpleInjector](https://www.nuget.org/packages/EasyNetQ.DI.SimpleInjector).

In this example we are using the Castle Windsor IoC container:

    // register our alternative container factory
    RabbitHutch.SetContainerFactory(() =>
        {
            // create an instance of Windsor
            var windsorContainer = new WindsorContainer();

            // wrap it in our implementation of EasyNetQ.IContainer
            return new WindsorContainerWrapper(windsorContainer);
        });

    // now we can create an IBus instance, but it's resolved from
    // windsor, rather than EasyNetQ's default service provider.
    var bus = RabbitHutch.CreateBus("host=localhost");

Here is WindsorContainerWrapper:

    public class WindsorContainerWrapper : IContainer, IDisposable
    {
        private readonly IWindsorContainer windsorContainer;

        public WindsorContainerWrapper(IWindsorContainer windsorContainer)
        {
            this.windsorContainer = windsorContainer;
        }

        public TService Resolve<TService>() where TService : class
        {
            return windsorContainer.Resolve<TService>();
        }

        public IServiceRegister Register<TService>(System.Func<IServiceProvider, TService> serviceCreator) 
            where TService : class
        {
            windsorContainer.Register(
                Component.For<TService>().UsingFactoryMethod(() => serviceCreator(this)).LifeStyle.Singleton
                );
            return this;
        }

        public IServiceRegister Register<TService, TImplementation>() 
            where TService : class 
            where TImplementation : class, TService
        {
            windsorContainer.Register(
                Component.For<TService>().ImplementedBy<TImplementation>().LifeStyle.Singleton
                );
            return this;
        }

        public void Dispose()
        {
            windsorContainer.Dispose();
        }
    }

Note that all EasyNetQ services should be registered as singletons.

It’s important that you dispose of Windsor correctly. EasyNetQ doesn’t provide a Dispose method on IContainer, but you can access the container via the advanced bus (yes, this is new too), and dispose of windsor that way:

    ((WindsorContainerWrapper)bus.Advanced.Container).Dispose();
    bus.Dispose();