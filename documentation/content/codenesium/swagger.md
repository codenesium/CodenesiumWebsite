+++
title = "Swagger"
description = ""
weight = 1
alwaysopen = true
+++
We use [SwashBuckle](https://github.com/domaindrivendev/Swashbuckle) to provide swagger support.


There is a lot of configuration options available for swagger.
 
Look in Startup.cs to see everything Swagger related.

For a demo view http://www.codenesium.com:8080/user7303b0f5161f4149bf2959a488d359feNebula/swagger/

To generate a client for a swagger endpoint in many languages see http://editor.swagger.io/

```
services.AddSwaggerGen(o =>
{
	o.ResolveConflictingActions(apiDescriptions => apiDescriptions.First());

	// resolve the IApiVersionDescriptionProvider service
	// note: that we have to build a temporary service provider here because one has not been created yet
	var provider = services.BuildServiceProvider().GetRequiredService<IApiVersionDescriptionProvider>();

	// add a swagger document for each discovered API version
	// note: you might choose to skip or document deprecated API versions differently
	foreach (var description in provider.ApiVersionDescriptions)
	{
		o.SwaggerDoc(description.GroupName, CreateInfoForApiVersion(description));
	}

	// add a custom operation filter which sets default values
	o.OperationFilter<SwaggerDefaultValues>();

	// integrate xml comments
	// o.IncludeXmlComments( XmlCommentsFilePath );
	if (this.Configuration.GetValue<bool>("SecurityEnabled"))
	{
	   var security = new Dictionary<string, IEnumerable<string>>
	   {
			{"Bearer", new string[] { }},
	   };
	   o.AddSecurityRequirement(security);
	   o.AddSecurityDefinition("Bearer", new ApiKeyScheme() { In = "header", Description = "Please insert JWT prefixed with Bearer", Name = "Authorization", Type = "apiKey" });
	}
});
```