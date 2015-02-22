Connecting via SSL is possible with EasyNetQ. This guide, by Gordon Coulter, was originally written in reply to a raised issue.

Firstly, you have to follow the steps at https://www.rabbitmq.com/ssl.html carefully. I spent a lot of time trying to get the openssl part working and then a lot more time getting it working the way I needed it and not just the canned demo.

Even once you get EasyNetQ working with SSL, having that little bit of DotNet example code they show on that page available for testing is a big help. I have a simple console app which contains both Rabbit and the EasyNetQ code below. Also use the Rabbit logs. They’re sometimes a bit more specific than what gets emitted at the client when something’s wrong.

When you do get connected, the management screen shows a small SSL under the protocol label on the connection screen. You should also see port 443 (assuming that’s what you bind to) in the listening ports table on the Overview tab.

![ssl](https://cloud.githubusercontent.com/assets/8321491/3799065/8d6ac506-1bea-11e4-84ac-9a9d71830b2e.png)

The sample code to make this work is as follows:
```C#
var connection = new ConnectionConfiguration();

connection.Port = 443;
connection.UserName = "user";
connection.Password = "pass";
connection.Product = "SSLTest";

var host1 = new HostConfiguration();
host1.Host = "rmq1.contoso.com";
host1.Port = 443;
host1.Ssl.Enabled = true;
host1.Ssl.ServerName = "rmq1.contoso.com";
host1.Ssl.CertPath = "c:\\tmp\\myclient.p12";
host1.Ssl.CertPassphrase = "secret";

var host2 = new HostConfiguration();
host2.Host = "rmq2.contoso.com";
host2.Port = 443;
host2.Ssl.Enabled = true;
host2.Ssl.ServerName = "rmq2.contoso.com";
host2.Ssl.CertPath = "c:\\tmp\\myclient.p12";
host2.Ssl.CertPassphrase = "secret";

connection.Hosts = new List<HostConfiguration> { host1, host2 };

connection.Validate();        //VERY IMPORTANT - DOES CONFIG AS WELL AS VALIDATION!

var Bus = RabbitHutch.CreateBus(connection, services => services.Register<IEasyNetQLogger>(logger => new DoNothingLogger()));
```
The appropriate place to set the SslOption properties is on the HostConfiguration object even though there is an SslOption property on the ConnectionConfiguration. Setting the SSL options on the HostConfiguration object enables support for clustering scenarios. Note in the above example we specified two HostConfiguration objects. If one becomes unavailable, EasyNetQ's PersistentConnection feature will automatically connect to the next available host. Having SSL settings configured on the host will allow it to connect without any errors.

If you only specify one host then you can choose to set the SslOptions on the HostConfiguration object or the ConnectionConfiguration object directly.

Don’t forget the call to Validate(). I initially skipped that (on the basis I was hard coding everything, so there could be nothing wrong which required validation). However, that method call actually applies various settings that are required to make the connection work.