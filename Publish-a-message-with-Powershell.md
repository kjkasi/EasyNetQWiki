Publishing a message to an exchange with Microsoft PowerShell v3+ and the EasyNetQ library is simple to perform.   

Below are two methods to publish a string message.  

**************Method #1************** Basic publish  (Lacks the ability to specify the exchange. ) 


```````$EasyNetQDll =[Reflection.Assembly]::LoadFile($path2EasyNetqdll)```````

```````# requires an older version of the RabbitMQ.client.dll Version 3.6.0```````

```````$RabbitMQDll =[Reflection.Assembly]::LoadFile($path2RabbitClientDll)```````

```````$myhost=[EasynetQ.RabbitHutch]::CreateBus("host=$RabbitServerHostName;virtualHost=$RabbitVirtualHostName;username=$RabbitUserName;password=$RabbitPassWord")```````

```````$myhost.publish("Hello World")```````

```````$myhost.Dispose()```````

**************Method #2************** Specify exchange 

```````#############Variable Declaration Area Modify variables in this area ###################```````

```````$RabbitServerHostName="DNS_Name_of_Rabbit_Server"```````

```````$RabbitVirtualHostName="Rabbit_VirtualHost_Name"```````

```````$RabbitUserName="Rabbit_User_Name"```````

```````$RabbitPassWord="Rabbit_UserName_Password"```````

```````$RoutingKey="Rabbit_Routing_Key"```````

```````$ExchangeName="Rabbit_Exchange_Name"```````

```````$ExchangeType="Topic"```````

```````$path2EasyNetqdll="C:\Path_To_DLL\EasyNetQ.dll"```````

```````#needs to be the 3.6.0 version of the RabbitMQ.Client.dll.```````

```````$path2RabbitClientDll="C:\Path_To_DLL\RabbitMQ.Client.dll"```````

```````#prepare a sample message to send to Rabbit for testing.```````

```````$currentTime= get-date```````

```````$MsgToSend= "Hello World it is now $currentTime"```````

```````####################################```````

```````$EasyNetQDll =[Reflection.Assembly]::LoadFile($path2EasyNetqdll)```````

```````# requires an older version of the RabbitMQ.client.dll Version 3.6.0```````

```````$RabbitMQDll =[Reflection.Assembly]::LoadFile($path2RabbitClientDll)```````


```````$myhost=[EasynetQ.RabbitHutch]::CreateBus("host=$RabbitServerHostName;virtualHost=$RabbitVirtualHostName;username=$RabbitUserName;password=$RabbitPassWord")```````

```````$myexch=New-Object EasyNetQ.Topology.Exchange -ArgumentList $ExchangeName```````

```````$mymsgProperty=new-object EasyNetQ.MessageProperties ```````

```````#Set the Delivery mode to Persistent. Non-persistent=1 persistent=2```````

```````$mymsgProperty.DeliveryMode=2```````

```````$myhost.Advanced.Publish($myexch, $RoutingKey, $true, $mymsgProperty,[system.text.encoding]::UTF8.GetBytes($MsgToSend))```````

```````$myhost.Dispose()```````
