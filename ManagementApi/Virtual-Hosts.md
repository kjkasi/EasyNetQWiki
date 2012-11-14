**GetVHosts()** returns all the virtual hosts on a broker:

    var vhosts = managementClient.GetVHosts();

    foreach (var vhost in vhosts)
    {
        Console.Out.WriteLine("vhost.name = {0}", vhost.name);
    }

Outputs something like this:

    vhost.name = /
    vhost.name = management_test_virtual_host
    vhost.name = my_virtual_host

***

**CreateVirtualHost(string name)** will create a virtual host with the given name:

    var vhost = managementClient.CreateVirtualHost("my_virtual_host");

***

**DeleteVirtualHost(VHost)** will delete the given virtual host:

    var vhost = managementClient.GetVhost("my_virtual_host");
    managementClient.DeleteVirtualHost(vhost);

