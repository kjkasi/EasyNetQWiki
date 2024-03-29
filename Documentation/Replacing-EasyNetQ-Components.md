EasyNetQ is a library composed of small components. When you write:

    var bus = RabbitHutch.CreateBus("host=localhost");

... the static method CreateBus assembles these components using a [tiny internal IoC container](../blob/master/Source/EasyNetQ/DI/DefaultServiceContainer.cs). An overload of the CreateBus method allows you to access the component registration so that you can provide your own versions of any of the EasyNetQ dependencies. The signature looks like this:

    public static IBus CreateBus(string connectionString, Action<IServiceRegister> registerServices)

The IServiceRegister interface provides a single method:

    public interface IServiceRegister
    {
        IServiceRegister Register<TService>(Func<IServiceProvider, TService> serviceCreator) where TService : class;
    }

So to register your own logger, based on IEasyNetQLogger, you'd write this code:

    IEasyNetQLogger logger = new MyLogger(); // note the use of IEasyNetQLogger not var.
    var bus = RabbitHutch.CreateBus(connectionString, 
        serviceRegister => serviceRegister.Register(serviceProvider => logger));

The Register method's argument, Func&lt;IServiceProvider, TService&gt;, is a function that's run when CreateBus pulls together the components to make an IBus instance. IServiceProvider looks like this:

    public interface IServiceProvider
    {
        TService Resolve<TService>() where TService : class;
    }

This allows you to access other services that EasyNetQ provides. If for example you wanted to replace the default serializer with your own implementation of ISerializer, and you wanted to construct it with a reference to the logger, you could do this:

    var bus = RabbitHutch.CreateBus(connectionString, serviceRegister => 
        serviceRegister.Register<ISerializer>(
        serviceProvider => new MySerializer(serviceProvider.Resolve<IEasyNetQLogger>())));

Note that we have to use an explicit type parameter on the Register method so that the internal IoC container knows which service we are replacing.

To see the complete list of components that make up the IBus instance, and how they are assembled, take a look at the [ComponentRegistration](../blob/master/Source/EasyNetQ/DI/DefaultServicesRegistration.cs) class.

You can access the container via the IAdvancedBus's Container property. This allows you to access internal components:

    var serializer = bus.Advanced.Container.Resolve<ISerializer>();

To replace the internal container with your own choice of IoC container, see [[Using Alternative DI Containers]]
    