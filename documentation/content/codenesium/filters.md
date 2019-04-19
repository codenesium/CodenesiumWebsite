+++
title = "Filters"
description = ""
weight = 1
alwaysopen = true
+++
Filters in ASP.NET Core are useful for making changes in the request pipeline. The ReadOnlyAttribute for example can be applied to controller methods that should only be reading data. It disables the change tracking on the controller.

Filters can be used to make changes to requests or apply security rules. Basically anything that needs to happen before the request enters the controller or the response after it leave.

Filters can also be applied to controllers. Filters can be applied globally in the startup. 

The order of filters does matter. 

See [documentation](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-2.1) from Microsoft.

```
public class ReadOnlyAttribute : ActionFilterAttribute
{
	public override void OnActionExecuting(ActionExecutingContext actionContext)
	{
		if(!(actionContext.Controller is AbstractApiController))
		{
			throw new Exception("ReadOnlyFilter can only be applied to controllers that inherit from AbstractApiController");
		}

		AbstractApiController controller = (AbstractApiController)actionContext.Controller;
		controller.TransactionCooordinator.DisableChangeTracking();
	}

	public override void OnActionExecuted(ActionExecutedContext actionExecutedContext)
	{
		base.OnActionExecuted(actionExecutedContext);
	}
}
```
	