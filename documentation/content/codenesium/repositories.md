+++
title = "Repositories"
description = ""
weight = 1
alwaysopen = true
+++

Our repositories look like a standard repository. 

We do generate a separate repository for each object. This is a violation of DRY. However, a generic repository can be an anti-pattern
and I like the flexibility of a repository per object.

Some design choices I made.

1. We don't expose a method that takes a Func like you will see on a lot of repositories using Entity Framework. I made this choice because I don't 
want the repository to be so coupled to Entity Framework is can never be changed. Parsing a Func into something Dapper can understand would be challenging if
not impossible. To accomplish this we only except objects or primitives as parameters. This creates a strong contract for the repository instead of a 
generic method that takes a Func. This does require adding methods for everything you want to add.

This could me mitigated by using the Strategy pattern and moving the queries to a separate, testable class. It is something I'm going to explore in the future. 

You can add a generic where method to the derived class to add this yourself.

Repositories are by default broken out by table. As your business domain starts making sense you should move things around to fit your domain. A domain would rarely just need access to one table. Luckily with Entity Framework we have access to the entire context so you can access anything you need from one repository. Repositories 
Can be combined or molded to fit your needs and I would expect what is initially generated to change. 

```
public interface IDeviceRepository
{
		Task<Device> Create(Device item);

		Task Update(Device item);

		Task Delete(int id);

		Task<Device> Get(int id);

		Task<List<Device>> All(int limit = int.MaxValue, int offset = 0);

		Task<Device> ByPublicId(Guid publicId);

		Task<List<DeviceAction>> DeviceActions(int deviceId, int limit = int.MaxValue, int offset = 0);
}
```