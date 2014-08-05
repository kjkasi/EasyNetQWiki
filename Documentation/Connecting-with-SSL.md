Connecting via SSL is already possible with EasyNetQ.

Firstly, you have to follow the steps at https://www.rabbitmq.com/ssl.html carefully. I spent a lot of time trying to get the openssl part working and then a lot more time getting it working the way I needed it and not just the canned demo.

Even once you get EasyNetQ working with SSL, having that little bit of DotNet example code they show on that page available for testing is a big help. I have a simple console app which contains both Rabbit and the EasyNetQ code below. Also use the Rabbit logs. They’re sometimes a bit more specific than what gets emitted at the client when something’s wrong.

When you do get connected, the management screen shows a small SSL under the protocol label on the connection screen. You should also see port 443 (assuming that’s what you bind to) in the listening ports table on the Overview tab.
![ssl](https://cloud.githubusercontent.com/assets/8321491/3799065/8d6ac506-1bea-11e4-84ac-9a9d71830b2e.png)

The sample code to make this work is as follows:
```C#
string serverName = "q1.domain.com";

HostConfiguration host = new HostConfiguration();
host.Host = serverName;

ConnectionConfiguration connection = new ConnectionConfiguration();
connection.Hosts = new List<IHostConfiguration>
{
    new HostConfiguration {Host=serverName, Port=443}
};

connection.Port = 443;
connection.UserName = "user";
connection.Password = "pass";
connection.Product = "SSLTest";

connection.Ssl.Enabled = true;
connection.Ssl.ServerName = serverName;


connection.Port = 443;
connection.Ssl.CertPath = "c:\\tmp\\myclient.p12";
connection.Ssl.CertPassphrase = "secret";
connection.Ssl.Enabled = true;
connection.Ssl.ServerName = serverName;

connectionConfig.Validate();        //VERY IMPORTANT - DOES CONFIG AS WELL AS VALIDATION!

var logger = new LogNothingLogger();
IAdvancedBus Bus = RabbitHutch.CreateBus(connectionConfig, reg => reg.Register<IEasyNetQLogger>(log => logger)).Advanced;
```
There’s some duplication in that you have to pass the server name and port into the main ConnectionConfiguration and then again on the SSL settings.

The SSL settings (which are exposed from the underlying Rabbit code) don’t allow for multiple servers. From what I can see, that’s an EasyNetQ feature. I’m guessing that it would be a case of having to wrap the SSL options into each of the HostConfiguration objects.

Don’t forget the call to Validate(). I initially skipped that (on the basis I was hard coding everything, so there could be nothing wrong which required validation). However, that method call actually applies various settings that are required to make the connection work.