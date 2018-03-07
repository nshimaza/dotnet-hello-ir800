# .NET Hello World on IR800 IOx

This Git repo demonstrates procedure and Dockerfiles to build an .NET Core
application running on IOx framework coming with
Cisco 829 / 809 Industrial Integrated Services Routers.

Building process consists of following steps.

1. Build .NET Core Hello World binaries using Microsoft provided Docker image.
1. Extract the built executable from the image.
1. Build Docker image for runtime.
1. Convert the runtime image to IOx application package.

## Prepare your environment

Before you start, you need to install `ioxclient` tool from Cisco.

`ioxclient` is a developer tool used for manipulating IOx environment.
It will be used for creating IOx application from a Docker image.
You can find `ioxclient` [here](https://developer.cisco.com/docs/iox/#downloads).

Consult to [Cisco DevNet site](https://developer.cisco.com/docs/iox/#what-is-ioxclient) for more detail.

## Build executable in Docker image

Build a .NET Core Hello World binaries using Docker image from Microsoft.
The Dockerfile under build directory does it for you.
Execute following command.

```shell-session
$ cd build
$ docker build -t dotnet-ir800-build .
```

You will get a Docker image tagged with "dotnet-ir800-build".

## Extract executable from the image

The binaries are built under `/hello/bin/Debug/netcoreapp2.0/publish/`.
You can extract it by executing following command.
`extract.sh` in the directory does it for you.

```shell-session
$ cd ../runtime
$ docker run --rm -ti --volume "`pwd`:/vol" dotnet-ir800-build /bin/sh /copy.sh
```

You will get a DLL file, PDB file and two JSON files on your working directory.
Those must be `hello.deps.json`, `hello.dll`, `hello.pdb`
and `hello.runtimeconfig.json`.

## Build runtime Docker image

Make sure you already have four built files under runtime directory.

The Dockerfile under the runtime directory builds a Docker image which
contains Hello World application binaries, .NET Core runtime and
all libraries the runtime depends on.

To get the image, execute following command under runtime directory.

```shell-session
$ docker build -t dotnet-ir800 .
```

You will get a Docker image tagged with "dotnet-ir800".
The image contains Hello World app and a shell script for keeping
container running.

## Convert Docker image to IOx app

Once a Docker image for runtime has been built, you can convert it to
IOx application package.

Navigate to the directory `app` and execute following command.

```shell-session
$ cd ../app
$ ioxclient docker package dotnet-ir800 .
```

You will get `packaget.tar` which is IOx application package installable to IR809 or IR829.

## Verify the app

Install the IOx app to IR8x9, activate then start.

When the app is started, `loop.sh` is called.  This sample app doesn't
start .NET Hello World because it cannot keep the app container running.
Like Docker container, IOx app container stops when its first process
terminated.  If we started Hello World as the first app process, it shows
Hello World message then immediately finishes.  As a result of this,
the entire app container stops immediately so we cannot verify if
the Hello World is really working.

So we starts a small dummy process which only keeps running so that we can
keep the app container running until we stop it intentionally.
The `loop.sh` does this.

While IOx app is running, we can access to console of the container
using `ioxclient` tool with following command.

NOTE: Before you access to the app console, make sure your `ioxclient` has
correct profile created for your IR8x9.  Use `ioxclient profile create` command
to create new profile.  Use port number 22 for SSH instead of default 2222.

```shell-session
$ ioxclient application console <app-ID>
```
Once you get shell prompt inside of the app container, run `dotnet` command
to execute the Hello World.

```shell-session
# dotnet /hello/hello.dll
Hello World!
```

You will see "Hello World!" on the console.
