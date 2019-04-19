+++
title = "Deployment"
description = ""
weight = 1
alwaysopen = true
+++

Deployment in .NET Core is very easy.

This [article](https://docs.microsoft.com/en-us/dotnet/core/deploying/) from Microsoft will give you all of the information you need.

For our applications you can open a command prompt and navigate to the web directory in the solution.

From there you can run
```dotnet publish -c release```

The app wil be built and the output will be in 
```bin\release\netcoreapp2.1\publish```

You can also build self contained applications with arguments like these

```dotnet publish -c release --self-contained -r win-x64``` for Windows 

```dotnet publish -c release --self-contained -r linux-x64``` for Linux

```dotnet publish -c release --self-contained -r osx-x64``` for Mac 

See [.NET Core RID Catalog](https://docs.microsoft.com/en-us/dotnet/core/rid-catalog) for a complete list of targets.


