You can subscribe to an interface, then publish implementations of that interface.

Let's look at an example. I have an interface IAnimal and two implementations Cat and Dog:
```c#
    public interface IAnimal
    {
        string Name { get; set; }
    }

    public class Cat : IAnimal
    {
        public string Name { get; set; }
        public string Meow { get; set; }
    }

    public class Dog : IAnimal
    {
        public string Name { get; set; }
        public string Bark { get; set; }
    }
```
I can subscribe to IAnimal and receive both Cat and Dog classes:
```c#
    bus.PubSub.Subscribe<IAnimal>("polymorphic_test", @interface =>
        {
            var cat = @interface as Cat;
            var dog = @interface as Dog;

            if (cat != null)
            {
                Console.Out.WriteLine("Name = {0}", cat.Name);
                Console.Out.WriteLine("Meow = {0}", cat.Meow);
            }
            else if (dog != null)
            {
                Console.Out.WriteLine("Name = {0}", dog.Name);
                Console.Out.WriteLine("Bark = {0}", dog.Bark);
            }
            else
            {
                Console.Out.WriteLine("message was not a dog or a cat");
            }
        });
```
Let's publish a cat and a dog:
```c#
    var cat = new Cat
    {
        Name = "Gobbolino",
        Meow = "Purr"
    };

    var dog = new Dog
    {
        Name = "Rover",
        Bark = "Woof"
    };

    bus.PubSub.Publish<IAnimal>(cat);
    bus.PubSub.Publish<IAnimal>(dog);
```
Note that I have to explicitly specify that I am publishing IAnimal. EasyNetQ uses the generic type specified in the Publish and Subscribe methods to route the publications to the subscriptions.