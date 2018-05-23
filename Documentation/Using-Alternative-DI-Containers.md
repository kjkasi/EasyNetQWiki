EasyNetQ is made up of a collection of independent components. Internally it uses LightInject container, which is wrapped by DefaultServiceContainer. If you look at the code for the static RabbitHutch class that you use to create instances of the core IBus interface, you will see that it simply creates a new DefaultServiceContainer, registers all of EasyNetQ’s components, and then calls container.Resolve<IBus>() creating a new instance of IBus with its tree of dependencies supplied by the container:

```
public static IBus CreateBus(Func<IServiceResolver, ConnectionConfiguration> connectionConfigurationFactory, Action<IServiceRegister> registerServices)
{
    var container = new DefaultServiceContainer();
    RegisterBus(container, connectionConfigurationFactory, registerServices);
    return container.Resolve<IBus>();
}
```

But what if you want EasyNetQ to use your container of choice? From version 3 the RabbitHutch class provides a static method, RegisterBus, that allows you to register EasyNetQ components in `IServiceRegister`,  which you can implement for any container you want. There are several already implemented adapters for [Castle Windsor](https://www.nuget.org/packages/EasyNetQ.DI.Windsor), [Autofac](https://www.nuget.org/packages/EasyNetQ.DI.Autofac), [LightInject](https://www.nuget.org/packages/EasyNetQ.DI.LightInject), [NInject](https://www.nuget.org/packages/EasyNetQ.DI.NInject), [StructureMap](https://www.nuget.org/packages/EasyNetQ.DI.StructureMap), [SimpleInjector](https://www.nuget.org/packages/EasyNetQ.DI.SimpleInjector).

In this example we are using Autofac and EasyNetQ.DI.Autofac package. It provides `RegisterEasyNetQ` extension method on top of `ContainerBuilder`.  

```
var containerBuilder = new ContainerBuilder();
containerBuilder.RegisterEasyNetQ("host=localhost", c => {/* override services here */});
var container = containerBuilder.Build();
```

After calling `RegisterEasyNetQ`, all components will be registered in `ContainerBuilder`.