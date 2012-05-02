RabbitMQ has a very cool feature, [http://www.rabbitmq.com/tutorials/tutorial-five-python.html|topic based routing], that allows a subscriber to filter messages based on multiple criteria. A topic is a list of words delimited by dots that are published along with the message. Examples would be, "stock.usd.nyse" or "book.uk.london" or "a.b.c", the words can be anything you like, but would usually be some attributes of the message. The topic string has a limit of 255 characters.

To publish with a topic, simply use the overloaded Publish method:

  bus.Publish("X.A", message);
  
Subscribers can filter messages by specifying a topic to match to. These can include the wildcard characters:

* (star) to match to exactly one word.

# (hash) to match to zero or more words.

So a message that is published with the topic "X.A.2" would match "#", "X.#", "*.A.*" but not "X.B.*" or "A". To subscribe with a topic, use the overloaded Subscribe method:

  bus.Subscribe("my_id", "X.*", handler);
  
A warning. Two separate subscriptions with the same subscriberId but different topic strings probably will not have the effect you are expecting. A subscriberId effectively identifies an individual AMQP queue. Two subscribers with the same subscriptionId will both connect to the same queue and both will add their own topic. So, for example, if you do this:

  bus.Subscribe("my_id", "X.*", handler);
  bus.Subscribe("my_id", "*.B", handler);
  
Each handler will be round-robbined with any messages that match "x.*" or "*.B".

Now, if you want to match on multiple topics ("X.*" OR "*.B") you can use another override of the Subscribe method that takes multiple topics, like this:

  bus.Subscribe("my_id", new[]{"X.*", "*.B"}, handler);