RabbitMQ has a very cool feature, [topic based routing](http://www.rabbitmq.com/tutorials/tutorial-five-python.html), that allows a subscriber to filter messages based on multiple criteria. A topic is a list of words delimited by dots that are published along with the message. Examples would be, "stock.usd.nyse" or "book.uk.london" or "a.b.c", the words can be anything you like, but would usually be some attributes of the message. The topic string has a limit of 255 characters.

To publish with a topic, simply use the overloaded Publish method with a topic:

    bus.Publish(message, "X.A");
  
Subscribers can filter messages by specifying a topic to match to. These can include the wildcard characters:

\* (star) to match to exactly one word.

\# (hash) to match to zero or more words.

So a message that is published with the topic "X.A.2" would match "\#", "X.\#", "\*.A.\*" but not "X.B.\*" or "A". To subscribe with a topic, use the overloaded Subscribe method with configuration:

    bus.Subscribe("my_id", handler, x => x.WithTopic("X.*"));
  
**A warning**. Two separate subscriptions with the same subscriberId but different topic strings probably will not have the effect you are expecting. A subscriberId effectively identifies an individual AMQP queue. Two subscribers with the same subscriptionId will both connect to the same queue and both will add their own topic binding. So, for example, if you do this:

    bus.Subscribe("my_id", handlerOfXDotStar, x => x.WithTopic("X.*"));
    bus.Subscribe("my_id", handlerOfStarDotB, x => x.WithTopic("*.B"));
  
All messages that match "x.\*" or "\*.B" will be delivered to the 'XXX_my_id' queue. RabbitMQ will then deliver the messages round-robbined to the two consumers, with handlerOfXDotStar and handlerOfStarDotB getting each message in turn.

Now, if you want to match on multiple topics ("X.\*" OR "\*.B") you can use another overload of the Subscribe method that takes multiple topics, like this:

    bus.Subscribe("my_id", handler, x => x.WithTopic("X.*").WithTopic("*.B"));

There are topic overloads for the SubscribeAsync method that work in exactly the same way.

For some more cautionary tales on topics, please see this blog post [Topic Confusion](http://mikehadlow.blogspot.co.uk/2013/09/easynetq-topic-confusion.html), although note that the example uses an older version of the API where you had to open a publish channel before calling Publish.