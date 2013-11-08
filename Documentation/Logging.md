EasyNetQ provides a logger interface IEasyNetQLogger:

    public interface IEasyNetQLogger
    {
       void DebugWrite(string format, params object[] args);
       void InfoWrite(string format, params object[] args);
       void ErrorWrite(string format, params object[] args);
       void ErrorWrite(Exception exception);
    }

By default EasyNetQ logs to the console, which is probably not what you want in a production system. The debug level logging is _very_ verbose and will not make much sense to someone without an intimate knowledge of AMQP and EasyNetQ. You should provide your own implementation of IEasyNetQLogger that logs info and error messages to your application's log. The RabbitHutch.CreateBus method provides overloads that allow you to replace any of the EasyNetQ components. See [[Replacing EasyNetQ Components]]. You can use this to register your custom logger with the bus. For example:

    var logger = new MyLogger() // implements IEasyNetQLogger
    var bus = RabbitHutch.CreateBus(“my connection string”, x => x.Register<IEasyNetQLogger>(_ => logger));

