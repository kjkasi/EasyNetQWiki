**GetChannels()** will return the list of channels currently open on the broker. For example:

    var channels = managementClient.GetChannels();

    foreach (var channel in channels)
    {
        Console.Out.WriteLine("channel.name = {0}", channel.name);
        Console.Out.WriteLine("channel.user = {0}", channel.user);
        Console.Out.WriteLine("channel.prefetch_count = {0}", channel.prefetch_count);
    }

Will output:

    channel.name = [::1]:64754 -> [::1]:5672 (1)
    channel.user = guest
    channel.prefetch_count = 50