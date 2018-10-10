**TL;DR**

To enable the legacy type naming convention simply create the bus with `EnableLegacyTypeNaming()` option:

```csharp
var bus = RabbitHutch.CreateBus("host=localhost", x => x.EnableLegacyTypeNaming());
```

**Long story**

Before v3 the queue and exchange name were something like this:

```
Exchange: EasyNetQ.Tests.TestPerformanceMessage:EasyNetQ.Tests.Common
Queue: EasyNetQ.Tests.TestPerformanceMessage:EasyNetQ.Tests.Common_consumer
```

The new format has become this:

```
Exchange: EasyNetQ.Tests.TestPerformanceMessage, EasyNetQ.Tests.Common
Queue: EasyNetQ.Tests.TestPerformanceMessage, EasyNetQ.Tests.Common_consumer
```

There isn't only the comma difference, to see all the difference check both serializers [DefaultTypeNameSerializer >=v3](https://github.com/EasyNetQ/EasyNetQ/blob/master/Source/EasyNetQ/DefaultTypeNameSerializer.cs) and [LegacyTypeNameSerializer <=v2](https://github.com/EasyNetQ/EasyNetQ/blob/master/Source/EasyNetQ/LegacyTypeNameSerializer.cs)