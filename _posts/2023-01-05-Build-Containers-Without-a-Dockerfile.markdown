---
layout: post
title: Build Containers Without a Dockerfile
date: 2023-01-05T00:00:00.0000000-06:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
Containers are rapidly becoming the primary deployment model for server-side code, whether that be a web site, an app server, or other server-side code that you need to run in your environment.

Containers offer substantial value, because a container image is created at build time, and once created, _the same container image_ can be used for dev testing, QA testing, and ultimately pushed into production. No need to keep rebuilding the project, or tweaking configuration, or other things that can cause inconsistencies in the CI/CD pipeline between a developer and production deployment.

As a general rule, container images are created using the Docker tool, and that tool uses a definition stored in a Dockerfile.

There's a new .NET capability that allows the `dotnet` command line tool to directly create a container image without Docker or a Dockerfile.

First, I will review how Docker has been used to create container images.

## Quick Overview of a Dockerfile

Here is an example of a Dockerfile that the `docker build` CLI command uses to create a Linux container image for a .NET project:

```text
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["webtest/webtest.csproj", "webtest/"]
RUN dotnet restore "webtest/webtest.csproj"
COPY . .
WORKDIR "/src/webtest"
RUN dotnet build "webtest.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "webtest.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "webtest.dll"]
```

The specification defines a `base` image using the ASP.NET 6.0 runtime, which will expose ports 80 and 443 when deployed. 

It then defines a temporary container image named `build` based on the .NET 6.0 SDK. This temporary image has access to the .NET compilers and other build tools necessary to build and publish the project. This image is used to run the `dotnet build` command to compile the solution in release mode.

The next step is to define another temporary container image named `publish`, based off the `build` image. This image is used to run the `dotnet publish` command to create the published output for the .NET project. The result is that the `/app/publish` directory contains only the files necessary for the app to execute at runtime.

Finally, a `final` image is created based on `base`, and the contents from the temporary `publish` image's `/app/publish` directory are copied into the final image's `/app` directory. The `ENTRYPOINT` statement indicates that when this container image is executed, a `dotnet webtest.dll` command will be run to start the app.

## Existing Tooling

Visual Studio, Visual Studio Code, and .NET have supported the creation of container images for several years, starting with .NET Core. To build a container image from a .NET project, you add a `Dockerfile` file to your project, and then your tooling (or you manually) runs the `docker build` command to build your project based on the contents of that Dockerfile specification.

Visual Studio has the ability to create the `Dockerfile` file for you, or you can create it by hand. Visual Studio is aware of your project's support for containers, and so will execute a `docker build` command for you, or you can run the command yourself at a command line interface (CLI) prompt.

## Creating Images Without Docker or a Dockerfile

Starting with .NET 7 Microsoft has introduced a new feature that allows you to build a .NET project into a container image without adding a `Dockerfile` file to your project.

This is nice, because it means you don't need to worry about maintaining the `Dockerfile` file, storing it in source control, seeing it in your project, or otherwise thinking about it.

On the other hand, it is not ideal because cloud-native infrastructure tooling you might use with images, such as Docker or docker-compose rely on the contents of a well-formed Dockerfile. What this means, in practice, is that you may need to continue to maintain a `Dockerfile` file in your project.

For scenarios where you want to create a single microservice, web, or web API project, build it into a container image, and push the image into Azure or Kubernetes, this new .NET capability is useful.

I will walk through the steps you can follow to use this new feature.

### Prerequisites and Constraints

As with the existing functionality from .NET Core to today, your development workstation must have Docker Desktop installed and running.

> ℹ️ Docker is not used to create the image, but the tooling does attempt to push the image into the local Docker image repository once it has been created, and this is why Docker Desktop is required.

The current tooling only supports Linux container images. 

I don't find this to be a serious limitation, as my view on Windows containers is that they exist to support legacy software. I assume that any server-side code I create using modern .NET will end up running in a Linux environment.

This new feature requires .NET 7, so you must have the .NET 7 SDK installed. 

If you are using Visual Studio, Visual Studio 2022 is required to work with .NET 7. This new tooling doesn't yet work in Visual Studio, but you can still use Visual Studio as an editor as long as you are willing to use the CLI to run the `dotnet publish` command.

If you are already using the CLI with something like Visual Studio Code, then your existing build process will be largely unaffected.

### Create the Project

Create a .NET 7 Web project named `webtest`, either in Visual Studio or using the CLI:

```text
dotnet new web -o webtest
```

If using Visual Studio _do not_ check the box in the project creation wizard that allows you to use Docker. Checking the box during the project creation causes Visual Studio to add a `Dockerfile` file to the project, which is not what you want in this example.

Instead of adding a `Dockerfile` file to the project, you will need to add a NuGet package to the project and edit the `csproj` file to specify things like the container image name, tag, and ports used by the container.

### Add a NuGet Reference

If using Visual Studio, right-click on the project's Dependencies node in Solution Explorer and choose the option to manage NuGet references. Add a reference to the `Microsoft.NET.Build.Containers` package.

If you are not using Visual Studio, change your directory to the `webtest` folder and run the following CLI command:

```text
dotnet add package Microsoft.NET.Build.Containers
```

The result is that your project now references the NuGet package required to support creating a container image directly from the `dotnet` CLI.

### Edit the csproj File

When building a container image you can optionally specify the name of the image file, and the tag and port values. With this new technique, these values are included in the `csproj` file.

Open the `csproj` file in your editor and add a `ContainerImageName` element to the `PropertyGroup` node:

```xml
    <ContainerImageName>webtest</ContainerImageName>
```

This specifies that the container image will be named `webtest`. If you don't provide this element, the container image name will default to the assembly name of the .NET project.

Optionally, you can also provide one or more tag values. By default the tooling will create a tag based on the version number of the project. If you want to override the default, add a `ContainerTag` element or `ContainerTags` element for multiple tags:

```xml
    <ContainerImageTag>latest</ContainerImageTag>
```

The example shown here indicates that the container image will be labeled `webtest:latest`.

Because the project is a web app, also define the ports needed by the container with a `ContainerPort` element. This element needs to be in its own `ItemGroup` node:

```xml
  <ItemGroup>
    <ContainerPort Include="443" Type="tcp" />
  </ItemGroup>
```

> ℹ️ For full details about the options you can set in your `csproj` file, check out the Microsoft Learn [publish as container](https://learn.microsoft.com/en-us/dotnet/core/docker/publish-as-container) document.

To recap, you now have a .NET 7 project that references the `Microsoft.NET.Build.Containers` package, and which specifies container image information in the `csproj` file. You can now build the project to create the container image.

### Create the Container Image

To build the container image you must use the CLI. Change directory to the location of the `csproj` file and run the following command (for web projects):

```text
dotnet publish --os linux --arch x64 -p:PublishProfile=DefaultContainer -c Release
```

Notice that the target operating system is Linux, and the architecture is set to x64.

If your project is a console app or other non-web project, then the `dotnet publish` command would be:

```text
dotnet publish --os linux --arch x64 /t:PublishContainer -c Release
```

The output from the `dotnet publish` command will look like this:

```text
MSBuild version 17.4.0+18d5aef85 for .NET
  Determining projects to restore...
  Restored /home/rockylhotka/src/webtest/webtest.csproj (in 141 ms).
  webtest -> /home/rockylhotka/src/webtest/bin/Release/net7.0/linux-x64/webtest.dll
  webtest -> /home/rockylhotka/src/webtest/bin/Release/net7.0/linux-x64/publish/
  Building image 'webtest' with tags latest on top of base image mcr.microsoft.com/dotnet/aspnet:7.0
  Pushed container 'webtest:latest' to Docker daemon
```

Once the command has finished you can use the `docker image ls` command to see the image:

```text
REPOSITORY      TAG     IMAGE ID       CREATED         SIZE
webtest         latest  4e3ff779ff69   2 seconds ago   216MB
```

This is the same result as if you'd created a `Dockerfile` file for the project and run the `docker build` command, but without the need to create and maintain the Dockerfile asset. And the image was created without using Docker itself, so that dependency has been eliminated (except for the automatic push to the local Docker image repository).

### Inspecting the Container Image

You can inspect the resulting image to see its configuration. This is done using the `docker` CLI command:

```text
docker image inspect 4e3ff779ff69
```

Notice that I'm using the `IMAGE ID` value for the image shown by the `docker image ls` command to specify which image I want to inspect. The output is JSON, and provides you with all the information about the image, such as `"Os": "linux"` and many other details.

### Running the Container Image

Because the `dotnet publish` command pushed the image into the local Docker repository, you can use the `docker` CLI command to easily run the image as a container on your local workstation:

```text
docker run -d -P 4e3ff779ff69
```

Again, I'm using the image id to specify the image I want to run. 

The `-d` flag tells Docker to detach my CLI from the container. By default when you run a container your CLI window will remain connected to the console output from that container. As a result, your CLI window is no longer useful expect for watching debug output. The `-d` flag frees up your CLI window for further use.

The `-P` flag tells Docker to automatically map the ports listed in the container image to open ports on `localhost`. In the `csproj` file the `ContainerPort` specified port 443, and so the `-P` flag will map that port to some open port on my local computer.

You can find the port mapping information by using the `docker ps` command. The result should look similar to this:

```text
$ docker ps
CONTAINER ID   IMAGE          COMMAND          CREATED         STATUS         PORTS                    NAMES
083cccfcb1ad   4e3ff779ff69   "/app/webtest"   2 minutes ago   Up 2 minutes   0.0.0.0:49155->443/tcp   cool_lumiere
```

Notice the port mapping: `0.0.0.0:49155->443/tcp`. This means that the container's port 443 is mapped to my computer's `localhost:49155` and I can use a browser to navigate to that address.

## Credits

Thank you to [@timheuer](https://mastodon.social/@timheuer) and [@ChetHusk](https://hachyderm.io/@chethusk) for their help in troubleshooting some issues I encountered, and also explaining some behind-the-scenes details.

## Resources

The code shown here is available in GitHub at [rockfordlhotka/webtest](https://github.com/rockfordlhotka/webtest).

The documentation for this feature is available on Microsoft Learn: [Publish As Container](https://learn.microsoft.com/en-us/dotnet/core/docker/publish-as-container).

## Conclusion

I am a strong advocate for the use of container images to publish server-side (and increasingly some client-side) software. Images provide many benefits to developers, IT operations, DevOps, QA, and other roles involved in software development, deployment, and management.

The ability to create container images directly using `dotnet publish`, without the need for Docker or a Dockerfile asset, is quite powerful in some scenarios.

This new feature is, I believe, another small step toward overall better tooling and workflows for developers and IT that we are seeing from Microsoft and the .NET ecosystem.