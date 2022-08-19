## v7.x

EasyNetQ provides a logger interface [ILogger](https://github.com/EasyNetQ/EasyNetQ/blob/42b96b1e0180c0db200b4de0f3bf22545c0632ac/Source/EasyNetQ/Logging/ILogger.cs#L8).

There are specific packages `EasyNetQ.Logging.*` to support different logging framework. The most commonly used is `EasyNetQ.Logging.Microsoft` that enables the Microsoft logging system where nowadays all the other providers can be consumed (Serilog, Log4Net, NLog, etc...).

The easiest way to jump in and start logging in a modern [.NET Generic Host](https://docs.microsoft.com/en-us/dotnet/core/extensions/generic-host) project is to install the packages [EasyNetQ.DI.Microsoft](https://www.nuget.org/packages/EasyNetQ.DI.Microsoft) and [EasyNetQ.Logging.Microsoft](https://www.nuget.org/packages/EasyNetQ.Logging.Microsoft) and setup the host services in the following way:

```csharp
using EasyNetQ;

IHost host = Host.CreateDefaultBuilder(args)
    .ConfigureServices(services =>
    {
        services.RegisterEasyNetQ("host=localhost", register => register.EnableMicrosoftLogging());
    })
    .Build();

await host.RunAsync();
```

If you need basic console logging for testing or debugging purposes you can use a built-in [ConsoleLogger](https://github.com/EasyNetQ/EasyNetQ/blob/42b96b1e0180c0db200b4de0f3bf22545c0632ac/Source/EasyNetQ/Logging/ConsoleLogger.cs#L7) in the following way:

```csharp
var bus = RabbitHutch.CreateBus("host=localhost", register => register.EnableConsoleLogger());
```

However, this is probably not what you want in a production system. The debug level logging is very verbose and logging all this information may have a performance impact on your application. Also, it will not make much sense to someone without an intimate knowledge of AMQP and EasyNetQ.

You can register a custom logger in the following way:

```csharp
var bus = RabbitHutch.CreateBus("host=localhost", register => register.Register(typeof(EasyNetQ.Logging.ILogger<>), typeof(MyCustomLogger<>)));
```

```csharp
IHost host = Host.CreateDefaultBuilder(args)
    .ConfigureServices(services =>
    {
        services.RegisterEasyNetQ("host=localhost", register => register.Register(typeof(EasyNetQ.Logging.ILogger<>), typeof(MyCustomLogger<>)));
    })
    .Build();

await host.RunAsync();
```

```csharp
public class MyCustomLogger : EasyNetQ.Logging.ILogger
{
    public bool Log(LogLevel logLevel, Func<string> messageFunc, Exception exception = null, params object[] formatParameters)
    {
        if (messageFunc == null)
        {
            return true;
        }

        // this is the structured message template and should be formatted
        var message = messageFunc();

        Console.ForegroundColor = ConsoleColor.Cyan;
        Console.WriteLine(message);

        return true;
    }
}

public class MyCustomLogger<TCategory> : MyCustomLogger, EasyNetQ.Logging.ILogger<TCategory>
{
}
```

## v6.x

EasyNetQ provides a logger interface using [LibLog](https://github.com/damianh/LibLog). 

It contains transparent built-in support for NLog, Log4Net, EntLib Logging, Serilog and Loupe, but you should to configure appropriate logging library first. For example, if you are using [Serilog](https://github.com/serilog/serilog) you should configure `Log.Logger` and LibLog will use it. 
Also you can disable logging by setting `LogProvider.IsDisabled` to `true`.

If you need basic console logging for testing or debugging purposes you can use a built-in [ConsoleLogProvider](https://github.com/Pliner/EasyNetQ/blob/8b58d9163741af2cdb092ce51ca27537c7f8b05d/Source/EasyNetQ/Logging/ConsoleLoggingProvider.cs) in the following way:

```csharp
LogProvider.SetCurrentLogProvider(ConsoleLogProvider.Instance);
```

However, this is probably not what you want in a production system. The debug level logging is _very_ verbose and logging all this information may have a performance impact on your application. Also, it will not make much sense to someone without an intimate knowledge of AMQP and EasyNetQ.