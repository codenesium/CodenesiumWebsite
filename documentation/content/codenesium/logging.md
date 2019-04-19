+++
title = "Logging"
description = ""
weight = 1
alwaysopen = true
+++


[MSDN](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-2.1&tabs=aspnetcore2x)


There are two places to configure logging. This section is only used by Entity Framework and is set by default
to write the Entity Framework SQL queries to the console.
```
public static readonly LoggerFactory LoggerFactory = new LoggerFactory(new List<ILoggerProvider>()
{
	new ConsoleLoggerProvider((category, level)
	=> 
	category == DbLoggerCategory.Database.Command.Name
	   && level == LogLevel.Information, true),

	 new DebugLoggerProvider((category, level)
	=> category == DbLoggerCategory.Database.Command.Name
	   && level == LogLevel.Information)
});
```

The second place sets up the logging from the config file.

```
public virtual void SetupLogging(IServiceCollection services)
{
	services.AddLogging(logBuilder => logBuilder
		.AddConfiguration(this.Configuration.GetSection("Logging"))
		.AddConsole()
		.AddDebug());
}
```

Loggers are injected all over the generated system. If you need to add a logger to a class simply add
```
ILogger<YOUR_CLASS_NAME> logger
```
to any constructor. 