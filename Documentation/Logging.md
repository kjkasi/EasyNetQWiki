EasyNetQ provides a logger interface using [LibLog](https://github.com/damianh/LibLog). 

It contains transparent built-in support for NLog, Log4Net, EntLib Logging, Serilog and Loupe, but you should to configure appropriate logging library first. For example, if you are using [Serilog](https://github.com/serilog/serilog) you should configure `Log.Logger` and LibLog will use it.

If you need basic console logging for testing or debugging purposes you can use build in [ConsoleLogProvider](https://github.com/Pliner/EasyNetQ/blob/8b58d9163741af2cdb092ce51ca27537c7f8b05d/Source/EasyNetQ/Logging/ConsoleLoggingProvider.cs) in following way:

`LogProvider.SetCurrentLogProvider(ConsoleLogProvider.Instance);            `

However, this is probably not what you want in a production system. The debug level logging is _very_ verbose and logging all this information may have a performance impact on your application. Also, it will not make much sense to someone without an intimate knowledge of AMQP and EasyNetQ.
