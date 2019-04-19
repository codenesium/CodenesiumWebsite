+++
title = "Autofac"
description = ""
weight = 1
alwaysopen = true
+++

We use Autofac for our Inversion of Control container. ASP.NET Core does have a build in IOC container but we found it had some limitaitons.
Mainly it doesn't support registration by convention which means you have to make an explicit registration for everything you add which is a pain.
Autofac is only used in the Startup.cs file. Look there for documenation on how we use Autofac.

[Autofac Documentation](https://autofac.readthedocs.io/en/latest/)

[Dependency injection in .NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-2.1)