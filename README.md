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
