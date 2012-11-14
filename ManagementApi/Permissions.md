**GetPermissions()** will return a list of all the permissions on the broker. For example:

    var permissions = managementClient.GetPermissions();

    foreach (var permission in permissions)
    {
        Console.Out.WriteLine("permission.user = {0}", permission.user);
        Console.Out.WriteLine("permission.vhost = {0}", permission.vhost);
        Console.Out.WriteLine("permission.configure = {0}", permission.configure);
        Console.Out.WriteLine("permission.read = {0}", permission.read);
        Console.Out.WriteLine("permission.write = {0}", permission.write);
    }

Will output something like this:

    permission.user = guest
    permission.vhost = /
    permission.configure = .*
    permission.read = .*
    permission.write = .*
    permission.user = guest
    permission.vhost = management_test_virtual_host
    permission.configure = .*
    permission.read = .*
    permission.write = .*
    permission.user = mike
    permission.vhost = my_virtual_host
    permission.configure = .*
    permission.read = .*
    permission.write = .*

***

**CreatePermission(PermissionInfo)** will create a new permission on the broker.

A permission is a set of access values between a user and a virtual host. RabbitMQ distinguishes between _configure_, _write_ and _read_ operations on exchanges and queues. The exchanges or queues that a permission applies to is expressed by a regular expression. So, for example '^$', matching nothing but an empty string denies access to any exchange or queue. '.*', matching any string, gives access to all exchanges and queues. 'mike\..*' would give access to any exchange or queue beginning with 'mike.'.

The **PermissionInfo** class represents a single permission. The default permission gives access to everything:

    user = new User {name = "mikey"};
    vhost = new Vhost {name = "theVHostName"};
    var permissionInfo = new PermissionInfo(user, vhost);

Use the **DenyAllConfigure**, **DenyAllRead** and **DenyAllWrite** methods to remove all permissions for these operations:

    var permissionInfo = new PermissionInfo(user, vhost)
        .DenyAllConfigure()
        .DenyAllRead()
        .DenyAllWrite();

To deny user configure permissions on all objects, but read and write on any the begin with 'mike.', do the following:

    var permissionInfo = new PermissionInfo(user, vhost)
        .DenyAllConfigure()
        .Read(@"mike\..*")
        .Write(@"mike\..*");

Once you've created a **PermissionInfo** object, you can create the permission like this:

    managementClient.CreatePermission(permissionInfo);

***

**DeletePermission(Permission)** deletes the given permission. For example, to delete the permission between a virtual host and a user, do this:

    var permission = managementClient.GetPermissions()
        .SingleOrDefault(x => x.user == "mikey" && x.vhost == "my_virtual_host");

    if (permission == null)
    {
        throw new ApplicationException(string.Format("No permission for vhost: {0} and user: {1}",
            testVHost, testUser));
    }

    managementClient.DeletePermission(permission);

