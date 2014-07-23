The EasyNetQ queue management utility. Use it to grab messages from queues and re-publish them. Also use it to inspect error queue messages and retry them.

###Usage:

    EasyNetQ.Hosepipe.exe <command> [<option:value> ..]

###Commands

	dump	Dump all the messages in a queue to the given directory
			Note: this creates three files for each message:

			The message body:
			<queue_name>.n.message.txt

			The basic properties for the message:
			<queue_name>.n.properties.txt

			The infomation needed to republish the message, including the exchange name
			and routing key:
			<queue_name>.n.info.txt

	insert	Republish all the messages in a given directory

	err		Dump all the EasyNetQ error messages to the given directory

	retry	Retry any EasyNetQ error messages in the given directory
			Note this ignores the *.message.txt and *.info.txt files
			because the properties and info are contained in the error message
			itself

	?		Output this usage message

###Options

	s	the RabbitMQ broker (server) to connect to. Default is 'localhost'
	v	the virtual host. Default is '/'
	u	the username to connect with. Default is 'guest'
	p	the password to connect with. Default is 'guest'
	q	the queue name to take messages from, or publish them to.
	o	the directory to output messages to. Default is current directory.
	n	the maximum number of messages to retrieve. Default is 1000.

###Examples
	
1. To output all the messages in a queue called 'my_queue' as text files 
   to a directory 'C:\temp\messages':

	EasyNetQ.Hosepipe.exe dump s:localhost u:guest p:guest q:my_queue o:C:\temp\messages

2. To insert (re-publish) all the messages in directory 'C:\temp\messages':

	EasyNetQ.Hosepipe.exe insert s:localhost u:guest p:guest o:C:\temp\messages

3. Dump all the EasyNetQ messages queued in the broker localhost to a directory
   'C:\temp\messages'

	EasyNetQ.Hosepipe.exe err s:localhost o:C:\temp\messages

4. To re-publish all error messages in directory 'C:\temp\messages':

	EasyNetQ.Hosepipe.exe retry s:localhost u:guest p:guest o:C:\temp\messages

###Notes

Neither of the commands 'dump' and 'err' remove messages from queues, they simply
iterate the queue and copy the messages to the given directory, leaving the original 
messages on the queue. Take care when retrying error messages that you purge the error 
queue first (using the RabbitMQ management interface), because if the messages fail 
again, they too will result in new error messages being published to the error queue 
and it is possible that	duplicate messages could be created.

###Installing

Currently Hosepipe is only available as source code. You can find the project here:

https://github.com/mikehadlow/EasyNetQ/tree/master/Source/EasyNetQ.Hosepipe