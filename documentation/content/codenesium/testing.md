+++
title = "Testing"
description = ""
weight = 1
alwaysopen = true
+++
Codenesium ships with a unit/integration test suite. Our philosophy is that tests are key to making software than can evolve with changing requirements.

All projects have 80%+ coverage. Where the coverage is missing is by design. 

We use popular libraries for testing. Testing with this stack is fun!

[xUnit](https://xunit.github.io/)
This is unit testing framework.

[Moq](https://github.com/moq/moq)
Mocking library for testing.

[Fluent Assertions](https://fluentassertions.com/)
Assertion library that allows checking of assertions with fluent syntax.

[Coverlet](https://github.com/tonerdo/coverlet)
App that generated a code coverage report.

We are also a huge fan of [NCrunch](https://www.ncrunch.net/). If you haven't checked it out you should.


A sample test looks like this
```
[Trait("Type", "Unit")]
[Trait("Table", "Device")]
[Trait("Area", "Repositories")]
public partial class DeviceRepositoryTests
{
	[Fact]
	public async void All()
	{
		Mock<ILogger<DeviceRepository>> loggerMoc = DeviceRepositoryMoc.GetLoggerMoc();
		ApplicationDbContext context = DeviceRepositoryMoc.GetContext();
		var repository = new DeviceRepository(loggerMoc.Object, context);

		Device entity = new Device();
		context.Set<Device>().Add(entity);
		await context.SaveChangesAsync();

		var record = await repository.All();

		record.Should().NotBeEmpty();
	}
}
```