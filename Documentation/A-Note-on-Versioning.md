EasyNetQ is Open Source Software. It has been used successfully in production by many companies, but be aware that it is under development with frequent changes both to the internals and to the public API. 

**EasyNetQ comes with absolutely no warranty.**

It is your responsibility to regression test each update. 

EasyNetQ follows [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html)

> Given a version number MAJOR.MINOR.PATCH, increment the:

> MAJOR version when you make incompatible API changes,
> MINOR version when you add functionality in a backwards compatible manner, and
> PATCH version when you make backwards compatible bug fixes.

> Additional labels for pre-release and build metadata are available as extensions to the MAJOR.MINOR.PATCH format.

Every commit pushed to the develop branch is built and released to NuGet with the -alphaNNNN postfix, where NNNN is the build number. For example, the current development release (at time of writing) is 3.7.0-alpha0038.

When the development team deem that develop is ready for release, a GitHub release is created (https://github.com/EasyNetQ/EasyNetQ/releases), and published to NuGet.

To browse the code version for a particular release, simply find the release on [the releases page](https://github.com/EasyNetQ/EasyNetQ/releases), and click the commit number. You can then browse the code as desired.