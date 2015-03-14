EasyNetQ provides a logger interface [IEasyNetQLogger](https://github.com/mikehadlow/EasyNetQ/blob/master/Source/EasyNetQ/IEasyNetQLogger.cs):

    public interface IEasyNetQLogger
    {
       void DebugWrite(string format, params object[] args);
       void InfoWrite(string format, params object[] args);
       void ErrorWrite(string format, params object[] args);
       void ErrorWrite(Exception exception);
    }

Logging is disabled by default, [NullLogger](https://github.com/EasyNetQ/EasyNetQ/blob/master/Source/EasyNetQ/Loggers/NullLogger.cs) is registered as the concrete implementation of `IeasyNetQLogger`.

A logger that logs to the console ([ConsoleLogger](https://github.com/EasyNetQ/EasyNetQ/blob/master/Source/EasyNetQ/Loggers/ConsoleLogger.cs)) exists and can be used for testing or debugging purposes. However, this is probably not what you want in a production system. The debug level logging is _very_ verbose and logging all this information may have a performance impact on your application. Also, it will not make much sense to someone without an intimate knowledge of AMQP and EasyNetQ.

You should provide your own implementation of IEasyNetQLogger that logs info and error messages to your application's log. The RabbitHutch.CreateBus method provides overloads that allow you to replace any of the EasyNetQ components. See [[Replacing EasyNetQ Components]]. You can use this to register your custom logger with the bus. For example:

    var logger = new MyLogger() // implements IEasyNetQLogger
    var bus = RabbitHutch.CreateBus(“my connection string”, x => x.Register<IEasyNetQLogger>(_ => logger));