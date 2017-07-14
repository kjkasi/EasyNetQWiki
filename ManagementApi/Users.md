**GetUsers()** will return a list of all users on the broker. For example:

    var users = managementClient.GetUsers();

    foreach (var user in users)
    {
        Console.Out.WriteLine("user.name = {0}", user.Name);
    }

Will output something like:

    user.name = guest
    user.name = mike
    user.name = mikey

*** 

**CreateUser(UserInfo)** will create a new user on the broker. For example, to create a new administrator:

    var userInfo = new UserInfo("mikey", "topSecret").AddTag("administrator");
    var user = managementClient.CreateUser(userInfo);

Possible tag values are: "administrator", "monitoring" and "management"

***

**DeleteUser(User)** will delete the user:

    var user = managementClient.GetUser(testUser);
    managementClient.DeleteUser(user);
