EasyNetQ is alpha software. It is under heavy development with frequent changes both to the internals and to the public API. The developers follow the following versioning scheme:

&lt;major&gt;.&lt;minor&gt;.&lt;patch&gt;.&lt;build-number&gt;

So for example, the version at the time this was written was:

0.25.2.174

We use the following, somewhat semantic, versioning policy:

    major           - 0 to indicate that this is alpha software.
    minor           - increments when there is a breaking change to the API.
    patch           - indicates a bug fix or some internal change.
    build-number    - increments on each build on the CI server (http://teamcity.codebetter.com/)

Each is build is automatically published to NuGet. You should regression test your software when upgrading your NuGet EasyNetQ package.